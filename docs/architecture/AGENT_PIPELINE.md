# Pipeline del Agente — Semantic Kernel

> **Estado:** Definido. Listo para implementar.
> **Última actualización:** 2026-02-24

---

## Principio de Diseño

**El LLM clasifica. El código calcula.**

| Responsabilidad | Quién | Por qué |
|---|---|---|
| Clasificar requisitos del JD (Tipo A-D) | LLM (GPT-4o) | Requiere juicio semántico |
| Evaluar match candidato-requisito | LLM (GPT-4o) | Requiere interpretación contextual |
| Calcular scores (Confidence, SAS, MVP) | Código Python | Determinismo garantizado |
| Generar contenido (Summary, Bullets) | LLM (GPT-4o) | Requiere lenguaje natural |
| Validar Non-Negotiables | Código Python | Reglas binarias, sin interpretación |
| Intent classification (¿es career-related?) | LLM (GPT-4o-mini) | Barato, rápido, suficiente |

---

## Arquitectura del Pipeline

```
HTTP Request (usuario pega JD)
        │
        ▼
┌─────────────────────┐
│  IntentClassifier   │  GPT-4o-mini: ¿Es una tarea de career companion?
│  (SecurityLayer)    │  Si no → rechazar antes de gastar tokens
└─────────────────────┘
        │ pass
        ▼
┌─────────────────────┐
│   AgentOrchestrator │  Django view → construye ProfileSnapshot → inicia pipeline
│   (Django)          │  Modo: 'screening' | 'full' | 'onboarding'
└─────────────────────┘
        │
        ▼
┌─────────────────────────────────────────────────────────────┐
│                  Semantic Kernel Pipeline                    │
│                                                              │
│  Phase 1: CultureRadarPlugin                                 │
│  Phase 2: RequirementClassifierPlugin → ScoringEngine (Py)  │
│  Phase 3: ContentDraftingPlugin                              │
│  Phase 4: ApplicationSupportPlugin  (triggered by user)     │
│  Phase 5: VerificationPlugin                                 │
│  Phase 6: BrandCoherencePlugin                               │
│  Phase 7: RetrospectivePlugin       (triggered at close)     │
└─────────────────────────────────────────────────────────────┘
        │
        ▼
  Output guardado en DB (scoring_history + outputs)
  Respuesta streameada al frontend via Vercel AI SDK
```

---

## Context Object

El `AgentContext` es el objeto que pasa entre fases. Se construye una vez y se enriquece en cada fase. Nunca se accede a la DB dentro de un plugin — el contexto ya tiene todo.

```python
# backend/apps/agent/schemas.py

class AgentContext(BaseModel):
    # Inputs del usuario
    jd_text: str
    mode: Literal['screening', 'full', 'onboarding']
    platform_name: str | None = None

    # Perfil del usuario (construido antes de iniciar el pipeline)
    profile: ProfileSnapshot

    # Resultados que se van completando por fase
    culture_radar: CultureRadarResult | None = None          # Fase 1
    classified_requirements: list[ClassifiedRequirement] = []# Fase 2
    scoring_result: ScoringResult | None = None              # Fase 2
    cv_content: CVContent | None = None                      # Fase 3
    application_qa: list[QAPair] = []                        # Fase 4
    verification_result: VerificationResult | None = None    # Fase 5
    brand_audit: BrandAuditResult | None = None              # Fase 6
    learnings: list[Learning] = []                           # Fase 7
```

---

## Plugins de Semantic Kernel

### Plugin 1: `CultureRadarPlugin`

**Fase:** 1 — Research & Culture Radar
**Ejecutor:** LLM (GPT-4o)
**Cuándo:** Siempre, al inicio del pipeline

```python
# backend/plugins/culture_radar.py

class CultureRadarPlugin:
    @kernel_function(
        name="analyze_culture",
        description="Analyzes a job description to extract company pain points, "
                    "culture signals, and tech stack maturity"
    )
    async def analyze_culture(
        self,
        jd_text: Annotated[str, "The full job description text"],
        company_name: Annotated[str, "The company name"]
    ) -> CultureRadarResult:
        ...

class CultureRadarResult(BaseModel):
    pain_points: list[str]          # 3 problemas que el rol busca resolver
    tech_stack_detected: list[str]  # tecnologías mencionadas
    tech_maturity: Literal['legacy', 'mixed', 'modern', 'cutting_edge']
    culture_signals: list[str]      # valores, frases de cultura detectadas
    red_flags: list[str]            # señales de alerta (si las hay)
    company_size_signal: Literal['startup', 'mid', 'enterprise', 'unknown']
```

---

### Plugin 2: `RequirementClassifierPlugin`

**Fase:** 2 — Alignment Score (parte LLM)
**Ejecutor:** LLM (GPT-4o)
**Cuándo:** Siempre en modo full; en screening usa versión simplificada

```python
class RequirementClassifierPlugin:
    @kernel_function(
        name="classify_requirements",
        description="Classifies each requirement from a JD into types A/B/C/D "
                    "and evaluates match against the candidate profile"
    )
    async def classify_requirements(
        self,
        jd_text: Annotated[str, "The full JD text"],
        profile_snapshot: Annotated[str, "JSON string of ProfileSnapshot"]
    ) -> list[ClassifiedRequirement]:
        ...

class ClassifiedRequirement(BaseModel):
    requirement: str
    type: Literal['A', 'B', 'C', 'D']
    # A = Filtro Duro, B = Obligatorio, C = Deseable Real, D = Deseable Inflado
    type_justification: str
    match: Literal['cumple', 'transferible', 'parcial', 'no_cumple']
    match_justification: str
    score: float  # 1.0 | 0.7 | 0.5 | 0.0
```

**Nota:** La clasificación A-D y la evaluación de match se hacen en **una sola llamada al LLM** para minimizar latencia y costo. El sistema prompt incluye las heurísticas completas de `scoring_protocol.md`.

---

### Scoring Engine (Python puro — no SK plugin)

**Fase:** 2 — Alignment Score (parte determinista)
**Ejecutor:** Código Python
**Cuándo:** Siempre, después del RequirementClassifierPlugin

Este no es un plugin de SK — es una función Python pura que recibe los outputs del LLM y calcula scores. **No hace llamadas LLM.**

```python
# backend/apps/scoring/engine.py

class ScoringEngine:
    def calculate_confidence_score(
        self,
        requirements: list[ClassifiedRequirement],
        strengths: list[StrengthSchema]  # para MVP
    ) -> ConfidenceScoreResult:
        """
        Implementa la fórmula exacta de scoring_protocol.md.
        Si mismo input → mismo output. Siempre.
        """
        # Paso 3: calcular base
        type_ab = [r for r in requirements if r.type in ('A', 'B')]
        type_cd = [r for r in requirements if r.type in ('C', 'D')]

        # Los tipo D tienen peso máximo 0.5 en puntos posibles
        ab_possible = sum(1.0 for r in type_ab)
        ab_earned   = sum(r.score for r in type_ab)
        obligatorios = (ab_earned / ab_possible * 100) if ab_possible > 0 else 0

        cd_possible = sum(0.5 if r.type == 'D' else 1.0 for r in type_cd)
        cd_earned   = sum(min(r.score, 0.5 if r.type == 'D' else 1.0) for r in type_cd)
        deseables = (cd_earned / cd_possible * 100) if cd_possible > 0 else 0

        base = (obligatorios * 0.60) + (deseables * 0.40)

        # Paso 4: MVP (el LLM NO evalúa esto — el scoring engine evalúa las strengths)
        mvp = self._calculate_mvp(strengths)

        final = min(base + mvp.total, 100.0)
        return ConfidenceScoreResult(
            obligatorios_score=obligatorios,
            deseables_score=deseables,
            base=base,
            mvp=mvp,
            final=final
        )

    def calculate_sas(
        self,
        profile: ProfileSnapshot,
        culture_radar: CultureRadarResult
    ) -> SASResult:
        """
        Las 5 dimensiones se evalúan con reglas de puntuación definidas.
        Si la dimensión no tiene datos suficientes en el perfil → 10 puntos (neutral).
        """
        ...

class ScoringResult(BaseModel):
    confidence: ConfidenceScoreResult
    sas: SASResult
    gap_analysis: list[GapItem]
    # gap_analysis: [{requirement, gap_type, mitigation_strategy, is_mitigable}]
    decision_matrix: Literal['max_priority', 'consider', 'strategic_only', 'not_recommended']
```

**Tests:** El scoring engine tiene cobertura TDD completa. Todo cambio en las fórmulas requiere que los tests pasen primero.

```python
# backend/tests/test_scoring_engine.py
def test_type_d_weight_halved():
    """Tipo D debe tener peso 0.5 en puntos posibles"""
    ...

def test_transferible_scores_0_7():
    """Estado transferible = 0.7 siempre"""
    ...

def test_mvp_cap_at_100():
    """Score final nunca supera 100"""
    ...

def test_same_input_same_output():
    """Determinismo garantizado"""
    req1 = [...]
    req2 = [...]  # mismos datos
    assert engine.calculate_confidence_score(req1) == engine.calculate_confidence_score(req2)
```

---

### Plugin 3: `ContentDraftingPlugin`

**Fase:** 3 — Context Mapping & Drafting
**Ejecutor:** LLM (GPT-4o)
**Cuándo:** Solo en modo `full`

```python
class ContentDraftingPlugin:
    @kernel_function(name="generate_killer_summary")
    async def generate_killer_summary(
        self,
        profile: Annotated[str, "ProfileSnapshot JSON"],
        scoring: Annotated[str, "ScoringResult JSON — para enfatizar fortalezas"],
        culture_radar: Annotated[str, "CultureRadarResult JSON — para alinear tono"],
        brand_voice: Annotated[str, "Brand voice guidelines"]
    ) -> str:
        """3-4 líneas. Refuerza el posicionamiento principal del candidato.
        Anti-impostorsindrome activo: si el scoring muestra fortalezas subestimadas, empujarlas."""
        ...

    @kernel_function(name="generate_impact_bullets")
    async def generate_impact_bullets(
        self,
        experience: Annotated[str, "WorkExperience JSON with achievements"],
        jd_requirements: Annotated[str, "ClassifiedRequirements JSON — para priorizar bullets"],
        brand_voice: Annotated[str, "Brand voice guidelines"]
    ) -> list[str]:
        """Estructura: [Acción] + [Contexto/Complejidad] + [Resultado Cuantificable].
        Solo incluir datos verificados en achievements. Sin inventar métricas."""
        ...

    @kernel_function(name="generate_skills_matrix")
    async def generate_skills_matrix(
        self,
        profile_skills: Annotated[str, "Skills JSON from ProfileSnapshot"],
        requirements: Annotated[str, "ClassifiedRequirements JSON"]
    ) -> list[SkillMatrixEntry]:
        """Lista skills relevantes al JD con nivel documentado en perfil.
        No incluir skills que no estén en el perfil — integrity check."""
        ...

class CVContent(BaseModel):
    killer_summary: str
    impact_bullets: dict[str, list[str]]  # {experience_id: [bullets]}
    skills_matrix: list[SkillMatrixEntry]
```

---

### Plugin 4: `ApplicationSupportPlugin`

**Fase:** 4 — Application Support
**Ejecutor:** LLM (GPT-4o)
**Cuándo:** Solo si el usuario proporciona preguntas del portal (triggered, no automático)

```python
class ApplicationSupportPlugin:
    @kernel_function(name="answer_platform_question")
    async def answer_platform_question(
        self,
        question: Annotated[str, "The exact question from the application portal"],
        cv_content: Annotated[str, "CVContent JSON — for narrative coherence"],
        profile: Annotated[str, "ProfileSnapshot JSON"],
        platform_context: Annotated[str, "Platform schema context if available"]
    ) -> str:
        """Respuesta coherente con el CV ya generado.
        Si el CV dice 'Full Stack con background en negocios', la respuesta refuerza ese ángulo.
        No inventar — solo usar datos del perfil."""
        ...

    @kernel_function(name="get_platform_questions_preview")
    async def get_platform_questions_preview(
        self,
        platform_name: Annotated[str, "e.g. 'getOnBoard', 'linkedin_jobs'"]
    ) -> PlatformQuestionsPreview:
        """Consulta platform_schemas (layer 2) para mostrar al usuario
        las preguntas típicas de esa plataforma y pedir confirmación."""
        ...

class PlatformQuestionsPreview(BaseModel):
    platform_name: str
    layer_2_fields: list[PlatformField]  # campos fijos de la plataforma
    confirmation_message: str            # "GetOnBoard generalmente pide: [lista]. ¿Te aparecen estas mismas?"
```

---

### Plugin 5: `VerificationPlugin`

**Fase:** 5 — Verification & Review
**Ejecutor:** LLM (GPT-4o) + Código Python para Non-Negotiables

```python
class VerificationPlugin:
    @kernel_function(name="verify_factual_integrity")
    async def verify_factual_integrity(
        self,
        cv_content: Annotated[str, "CVContent JSON"],
        profile: Annotated[str, "ProfileSnapshot JSON — source of truth"]
    ) -> VerificationResult:
        """Cross-check: cada afirmación del CV debe tener respaldo en el perfil.
        Non-Negotiables se validan en código (reglas binarias), no por LLM."""
        ...

    @kernel_function(name="generate_interview_intel")
    async def generate_interview_intel(
        self,
        cv_content: Annotated[str, "CVContent JSON"],
        culture_radar: Annotated[str, "CultureRadarResult JSON"],
        scoring: Annotated[str, "ScoringResult JSON — gaps se vuelven preguntas probables"]
    ) -> InterviewIntel:
        """3 preguntas técnicas probables + 2 de cultura.
        Los gaps del scoring se convierten en preguntas de presión."""
        ...

class VerificationResult(BaseModel):
    passed: bool
    violations: list[str]  # afirmaciones sin respaldo
    non_negotiables_check: dict[str, bool]  # {rule: passed}
    advocacy_check: list[str]  # "¿Se posicionaron transferibles correctamente?"

class InterviewIntel(BaseModel):
    technical_questions: list[str]
    culture_questions: list[str]
```

---

### Plugin 6: `BrandCoherencePlugin`

**Fase:** 6 — Brand Coherence Audit
**Ejecutor:** LLM (GPT-4o)
**Cuándo:** Solo en modo `full`

```python
class BrandCoherencePlugin:
    @kernel_function(name="audit_brand_coherence")
    async def audit_brand_coherence(
        self,
        cv_content: Annotated[str, "CVContent JSON"],
        brand_voice: Annotated[str, "Brand voice guidelines"],
        profile: Annotated[str, "ProfileSnapshot JSON — pillars, ethos"]
    ) -> BrandAuditResult:
        """Evalúa 6 criterios del checklist de framework_protocol.md Fase 6."""
        ...

class BrandAuditResult(BaseModel):
    tone_match: bool           # ¿El lenguaje refleja la voz documentada?
    pillar_reinforced: bool    # ¿Cada logro refuerza el posicionamiento principal?
    ethos_visible: bool        # ¿Se evidencia la motivación/valores del candidato?
    tech_business_balance: bool # ¿Hay balance técnico/negocio según brand_voice?
    red_lines_respected: bool  # ¿No se violó ningún Non-Negotiable?
    narrative_coherent: bool   # ¿El CV cuenta UNA historia?
    notes: list[str]           # observaciones por criterio
    overall_passed: bool       # todos los criterios passed
```

---

### Plugin 7: `RetrospectivePlugin`

**Fase:** 7 — Retrospectiva
**Ejecutor:** LLM (GPT-4o)
**Cuándo:** Al cerrar cualquier sesión completa (triggered por el usuario)

```python
class RetrospectivePlugin:
    @kernel_function(name="capture_learnings")
    async def capture_learnings(
        self,
        session_summary: Annotated[str, "JSON resumen de la sesión completa"],
        user_responses: Annotated[str, "Respuestas del usuario a las 4 preguntas de Fase 7"]
    ) -> list[LearningDraft]:
        """Formula learnings estructurados de las respuestas del usuario."""
        ...

    @kernel_function(name="evaluate_generalizability")
    async def evaluate_generalizability(
        self,
        learnings: Annotated[str, "List of LearningDraft JSON"]
    ) -> list[GeneralizabilityAssessment]:
        """Evalúa si algún learning aplica a cualquier candidato.
        Criterios: se repite 2+ veces, es generalizable, tiene solución."""
        ...

class LearningDraft(BaseModel):
    category: Literal['non_negotiables', 'brand_voice', 'personal_context', 'process', 'general']
    description: str
    trigger_context: str
    solution: str
    is_generalizable: bool
    generalizability_justification: str
```

---

## Orchestration: Flujo del Pipeline

El `AgentOrchestrator` en Django controla el flujo. **No usa SK AutoPlanner** en el MVP — la secuencia es explícita y determinista.

```python
# backend/apps/agent/orchestrator.py

class AgentOrchestrator:
    def __init__(self, kernel: Kernel, scoring_engine: ScoringEngine):
        self.kernel = kernel
        self.scoring_engine = scoring_engine

    async def run_screening(self, jd_text: str, profile: ProfileSnapshot) -> AgentContext:
        """Modo screening: Fases 1-2 condensadas."""
        ctx = AgentContext(jd_text=jd_text, mode='screening', profile=profile)

        # Fase 1: Culture Radar (simplificado)
        ctx.culture_radar = await self.kernel.invoke(
            "CultureRadarPlugin", "analyze_culture",
            jd_text=jd_text, company_name=self._extract_company(jd_text)
        )

        # Fase 2: Clasificación + Scoring
        ctx.classified_requirements = await self.kernel.invoke(
            "RequirementClassifierPlugin", "classify_requirements",
            jd_text=jd_text, profile_snapshot=ctx.profile.model_dump_json()
        )
        ctx.scoring_result = self.scoring_engine.calculate_confidence_score(
            requirements=ctx.classified_requirements,
            strengths=profile.professional_strengths
        )
        ctx.scoring_result.sas = self.scoring_engine.calculate_sas(profile, ctx.culture_radar)

        return ctx

    async def run_full_pipeline(self, jd_text: str, profile: ProfileSnapshot) -> AsyncIterator[str]:
        """Modo completo: 7 fases con streaming de respuesta."""
        ctx = AgentContext(jd_text=jd_text, mode='full', profile=profile)

        # Fases 1-2 (igual que screening)
        yield "[Fase 1/7] Analizando empresa y cultura..."
        ctx = await self._run_phase_1_2(ctx)

        yield "[Fase 3/7] Generando contenido del CV..."
        ctx.cv_content = await self._run_phase_3(ctx)

        # Fases 4-6 son triggered por interacción del usuario
        # Se pausan y esperan input antes de continuar

        yield "[Fase 5/7] Verificando integridad..."
        ctx.verification_result = await self._run_phase_5(ctx)

        yield "[Fase 6/7] Auditando coherencia de marca..."
        ctx.brand_audit = await self._run_phase_6(ctx)

        return ctx
```

---

## Modelo de Streaming

El frontend usa **Vercel AI SDK** para recibir el stream. El backend emite el output en chunks:

```
[Segmento 1] Culture Radar → stream del análisis de empresa
[Segmento 2] Scores → Confidence + SAS con tabla de desglose
[Segmento 3] Gap Analysis → lista de gaps con estrategias
[Segmento 4] Killer Summary → texto generado
[Segmento 5] Impact Bullets → bullets por experiencia
[Segmento 6] Skills Matrix → tabla
[Segmento 7] Verification + Interview Intel
[Segmento 8] Brand Coherence Audit
```

Cada segmento se marca con un prefijo estandarizado para que el frontend pueda renderizar secciones progresivamente.

---

## Estructura de Archivos

```
backend/
├── apps/
│   ├── agent/
│   │   ├── orchestrator.py    # AgentOrchestrator — lógica de secuencia
│   │   ├── schemas.py         # AgentContext + todos los tipos de datos del pipeline
│   │   └── views.py           # Endpoints: /api/agent/chat, /api/agent/screening
│   └── scoring/
│       ├── engine.py          # ScoringEngine (Python puro, sin LLM)
│       └── schemas.py         # ScoringResult, ConfidenceScore, SASResult, etc.
│
└── plugins/                   # Semantic Kernel plugins
    ├── culture_radar.py       # CultureRadarPlugin
    ├── requirement_classifier.py # RequirementClassifierPlugin
    ├── content_drafting.py    # ContentDraftingPlugin
    ├── application_support.py # ApplicationSupportPlugin
    ├── verification.py        # VerificationPlugin
    ├── brand_coherence.py     # BrandCoherencePlugin
    └── retrospective.py       # RetrospectivePlugin
```

---

## Tabla Resumen: LLM vs Código

| Fase | Tarea | Ejecutor | Modelo |
|------|-------|----------|--------|
| Pre-pipeline | Intent classification | LLM | GPT-4o-mini |
| 1 | Análisis de empresa y cultura | LLM | GPT-4o |
| 2 | Clasificación de requisitos A-D | LLM | GPT-4o |
| 2 | Evaluación de match candidato-req | LLM | GPT-4o |
| 2 | **Cálculo de Confidence Score** | **Código Python** | — |
| 2 | **Cálculo del MVP (Modificador)** | **Código Python** | — |
| 2 | **Cálculo del SAS** | **Código Python** | — |
| 2 | **Validación de Non-Negotiables** | **Código Python** | — |
| 3 | Generación de Killer Summary | LLM | GPT-4o |
| 3 | Generación de Impact Bullets | LLM | GPT-4o |
| 3 | Generación de Skills Matrix | LLM | GPT-4o |
| 4 | Respuestas a preguntas del portal | LLM | GPT-4o |
| 4 | Preview de campos de plataforma | **Código (DB lookup)** | — |
| 5 | Verificación de integridad factual | LLM | GPT-4o |
| 5 | Generación de Interview Intel | LLM | GPT-4o |
| 6 | Auditoría de coherencia de marca | LLM | GPT-4o |
| 7 | Captura de learnings | LLM | GPT-4o |
| 7 | Evaluación de generalizabilidad | LLM | GPT-4o |

---

## Documentos Relacionados

- `scoring_protocol.md` — Fórmulas exactas que implementa el ScoringEngine
- `framework_protocol.md` — Las 7 fases originales del framework markdown
- `DATABASE.md` — Schema de DB que alimenta el ProfileSnapshot
- `SECURITY.md` — IntentClassifier como primera capa del pipeline
- `STACK.md` — Semantic Kernel como framework de orquestación
