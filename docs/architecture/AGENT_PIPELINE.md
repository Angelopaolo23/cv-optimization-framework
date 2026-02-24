# Pipeline del Agente — Semantic Kernel

> **Estado:** Definido. Revisado con Opus 4.6. Listo para implementar.
> **Última actualización:** 2026-02-24

---

## Principio de Diseño

**El LLM clasifica. El código calcula.**

| Responsabilidad | Quién | Por qué |
|---|---|---|
| Clasificar requisitos del JD (Tipo A-D) | LLM (GPT-4o) | Requiere juicio semántico |
| Evaluar match candidato-requisito | LLM (GPT-4o) | Requiere interpretación contextual |
| **Evaluar dimensiones del SAS (5 rubricas)** | **LLM (GPT-4o)** | Las rubricas son cualitativas — requieren juicio semántico |
| Calcular Confidence Score + MVP + suma SAS | Código Python | Aritmética pura — determinismo garantizado |
| Generar contenido (Summary, Bullets) | LLM (GPT-4o) | Requiere lenguaje natural |
| Validar Non-Negotiables | Código Python | Reglas binarias, sin interpretación |
| Intent classification (¿es career-related?) | LLM (GPT-4o-mini) | Barato, rápido, suficiente |

> **Nota sobre el SAS:** Las 5 dimensiones del SAS (Metas, Motivaciones, Cultura, Crecimiento, Autonomía) se puntúan mediante un `SASEvaluatorPlugin` que usa GPT-4o para evaluar cada rubrica contra el perfil. El ScoringEngine luego **solo suma** los puntos. Esto mantiene el determinismo del cálculo numérico mientras reconoce que la evaluación dimensional es semántica.

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
│  Phase 2: RequirementClassifierPlugin → SASEvaluatorPlugin   │
│           → ScoringEngine (Py)                               │
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
    sas_evaluation: SASDimensionEvaluation | None = None     # Fase 2 (LLM)
    scoring_result: ScoringResult | None = None              # Fase 2 (Python)
    cv_content: CVContent | None = None                      # Fase 3
    application_qa: list[QAPair] = []                        # Fase 4
    verification_result: VerificationResult | None = None    # Fase 5
    brand_audit: BrandAuditResult | None = None              # Fase 6
    learnings: list[Learning] = []                           # Fase 7

    # Control de recuperación de sesión
    last_completed_phase: int = 0      # 0=ninguna, 1=CultureRadar, ..., 7=Retro
    application_id: str | None = None  # FK a DB para persistir estado por fase
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

> **Sprint 1 — Sin web search (degradación explícita):** En Sprint 1, `CultureRadarPlugin` opera únicamente con el texto del JD. Cuando no hay datos externos, la dimensión "Valores y Cultura" del SAS se marca con **baja confianza** y el sistema muestra al usuario: *"La evaluación de cultura está basada solo en el texto de la oferta. Para un análisis más completo, necesitaría investigar la empresa."* No se asigna un puntaje falso — se informa la limitación.
>
> **Sprint 2 — Bing Search API como tool de SK:** Se integra Bing Search API como tool del plugin. El agente busca: blog técnico, Glassdoor/reviews, noticias recientes, GitHub organizacional. La dimensión "Cultura" del SAS pasa a ser una evaluación real. Esto es parte del diferenciador core del producto: no solo optimiza el CV, evalúa si la oportunidad le *conviene* al usuario.

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

**Nota sobre una sola llamada:** La clasificación A-D y la evaluación de match se hacen en **una sola llamada al LLM** para minimizar latencia. Si el JD tiene >12 requisitos, evaluar si chunking en dos llamadas mejora la calidad de clasificación en los últimos items. Medir en las primeras 50 clasificaciones reales antes de decidir.

**Cache de clasificaciones:** Antes de invocar el plugin, el Orchestrator hace lookup en `scoring_history` por `input_hash`. Si existe una clasificación previa para el mismo JD + perfil, la reutiliza. Si re-clasifica, compara con la anterior y alerta si la diferencia en score es >5 puntos.

---

### Plugin 2b: `SASEvaluatorPlugin`

**Fase:** 2 — Alignment Score (evaluación dimensional del SAS)
**Ejecutor:** LLM (GPT-4o)
**Cuándo:** Siempre, después de `RequirementClassifierPlugin`

Las rubricas del SAS son cualitativas ("¿El rol acelera directamente una meta documentada?" = 20 pts). Requieren juicio semántico del LLM — no son reglas binarias ejecutables en código. El ScoringEngine solo suma los puntos.

```python
class SASEvaluatorPlugin:
    @kernel_function(
        name="evaluate_sas_dimensions",
        description="Evaluates the 5 SAS dimensions against the candidate profile "
                    "and the job opportunity"
    )
    async def evaluate_sas_dimensions(
        self,
        profile: Annotated[str, "ProfileSnapshot JSON — goals, motivations, values, preferences"],
        culture_radar: Annotated[str, "CultureRadarResult JSON — company signals"],
        jd_text: Annotated[str, "The full JD text"]
    ) -> SASDimensionEvaluation:
        ...

class SASDimensionScore(BaseModel):
    score: int          # 0, 5, 10, 15, or 20
    justification: str  # explicación de por qué ese puntaje

class SASDimensionEvaluation(BaseModel):
    career_goals: SASDimensionScore     # Dimensión 1: Metas de Carrera
    intrinsic_motivations: SASDimensionScore  # Dimensión 2: Motivaciones Intrínsecas
    values_culture: SASDimensionScore   # Dimensión 3: Valores y Cultura
    tech_growth: SASDimensionScore      # Dimensión 4: Crecimiento Técnico
    autonomy_role: SASDimensionScore    # Dimensión 5: Autonomía y Tipo de Rol
    # El ScoringEngine suma los 5 scores → sas_total
```

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
        sas_evaluation: SASDimensionEvaluation
    ) -> SASResult:
        """
        Solo suma las puntuaciones ya evaluadas por el SASEvaluatorPlugin (LLM).
        No hace juicio semántico — aritmética pura.
        """
        total = (
            sas_evaluation.career_goals.score +
            sas_evaluation.intrinsic_motivations.score +
            sas_evaluation.values_culture.score +
            sas_evaluation.tech_growth.score +
            sas_evaluation.autonomy_role.score
        )
        return SASResult(dimensions=sas_evaluation, total=total)

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

El `AgentOrchestrator` en Django controla el flujo. **No usa SK AutoPlanner** en el MVP — la secuencia es explícita y determinista. AutoPlanner se puede reconsiderar en V2 si se agregan modos dinámicos.

### Validación pre-pipeline (siempre, antes de invocar plugins)

```python
def _validate_profile_completeness(self, profile: ProfileSnapshot) -> list[str]:
    """
    Verifica datos mínimos necesarios. Si faltan, devuelve mensajes amigables
    en lugar de dejar que el pipeline falle silenciosamente con scores de 0.
    """
    errors = []
    if not profile.skills:
        errors.append("Tu perfil necesita al menos un skill en el Power Stack para calcular el score.")
    if not profile.work_experiences and not profile.projects:
        errors.append("Agrega al menos una experiencia laboral o proyecto para generar Impact Bullets.")
    # professional_strengths vacío → MVP = 0, no es error pero se avisa
    return errors
```

### Flujo del Orchestrator

```python
# backend/apps/agent/orchestrator.py

class AgentOrchestrator:
    def __init__(self, kernel: Kernel, scoring_engine: ScoringEngine):
        self.kernel = kernel
        self.scoring_engine = scoring_engine

    async def run_screening(self, jd_text: str, profile: ProfileSnapshot) -> AgentContext:
        """Modo screening: Fases 1-2 condensadas."""

        # Validación pre-pipeline
        errors = self._validate_profile_completeness(profile)
        if errors:
            raise ProfileIncompleteError(errors)

        ctx = AgentContext(jd_text=jd_text, mode='screening', profile=profile)

        # Fase 1: Culture Radar
        ctx.culture_radar = await self._invoke_with_retry(
            "CultureRadarPlugin", "analyze_culture",
            jd_text=jd_text, company_name=self._extract_company(jd_text)
        )
        ctx.last_completed_phase = 1
        await self._persist_context(ctx)

        # Fase 2a: Cache lookup antes de clasificar
        cached = await self._lookup_classification_cache(ctx)
        if cached:
            ctx.classified_requirements = cached
        else:
            ctx.classified_requirements = await self._invoke_with_retry(
                "RequirementClassifierPlugin", "classify_requirements",
                jd_text=jd_text, profile_snapshot=ctx.profile.model_dump_json()
            )
            await self._check_classification_drift(ctx)

        # Fase 2b: SAS dimensional evaluation (LLM)
        ctx.sas_evaluation = await self._invoke_with_retry(
            "SASEvaluatorPlugin", "evaluate_sas_dimensions",
            profile=ctx.profile.model_dump_json(),
            culture_radar=ctx.culture_radar.model_dump_json(),
            jd_text=jd_text
        )

        # Fase 2c: Score calculation (Python puro)
        ctx.scoring_result = self.scoring_engine.calculate_confidence_score(
            requirements=ctx.classified_requirements,
            strengths=profile.professional_strengths
        )
        ctx.scoring_result.sas = self.scoring_engine.calculate_sas(ctx.sas_evaluation)
        ctx.last_completed_phase = 2
        await self._persist_context(ctx)

        return ctx

    async def run_full_pipeline(self, jd_text: str, profile: ProfileSnapshot) -> AsyncIterator[str]:
        """Modo completo: 7 fases con streaming. Persiste por fase para recuperación."""

        errors = self._validate_profile_completeness(profile)
        if errors:
            yield f"[ERROR] {'; '.join(errors)}"
            return

        ctx = AgentContext(jd_text=jd_text, mode='full', profile=profile)

        yield "[Fase 1/7] Analizando empresa y cultura..."
        ctx = await self._run_phase_1_2(ctx)  # incluye SAS + persistencia
        yield self._format_scoring_segment(ctx)  # stream de scores al frontend

        # Anti-Síndrome del Impostor: pausar si hay requisitos A/B sin match
        hard_gaps = [r for r in ctx.classified_requirements
                     if r.type in ('A', 'B') and r.match == 'no_cumple']
        if len(hard_gaps) >= 2:
            yield self._format_anti_impostor_prompt(hard_gaps)
            # El Orchestrator espera la respuesta del usuario antes de continuar
            # (implementado como evento en el WebSocket / SSE stream)
            return  # frontend retoma con /api/agent/continue

        yield "[Fase 3/7] Generando contenido del CV..."
        ctx.cv_content = await self._run_phase_3(ctx)
        ctx.last_completed_phase = 3
        await self._persist_context(ctx)

        # Fases 4-6 son triggered por interacción del usuario
        yield "[Fases 4-6] Listo para soporte de postulación, verificación y auditoría de marca."
        yield "[Fase 7] Cuando termines la postulación, activa la retrospectiva para capturar learnings."

    async def _invoke_with_retry(self, plugin: str, func: str, **kwargs) -> Any:
        """
        Wrapper de retry para todas las llamadas a plugins LLM.
        - Intenta 2 veces con backoff exponencial
        - Valida respuesta contra el schema Pydantic esperado
        - Si falla 3 veces → raise AgentCallError (el view lo convierte en mensaje amigable)
        """
        ...

    async def _persist_context(self, ctx: AgentContext) -> None:
        """Persiste el AgentContext en DB al completar cada fase. Permite reanudar."""
        ...
```

### Protocolo de Fase 7 (Orchestrator responsibility)

El Orchestrator emite las 4 preguntas obligatorias del `framework_protocol.md` antes de invocar el `RetrospectivePlugin`. No es el plugin quien pregunta — es el flujo:

```
1. "¿Hubo algún momento donde casi afirmamos algo que no deberíamos?"
2. "¿El tono funcionó bien para este tipo de empresa?"
3. "¿Surgió algo nuevo que debería documentarse en tu perfil?"
4. "¿El workflow funcionó bien o hay fricciones que mejorar?"
```

Solo después de recibir respuestas del usuario se invoca `RetrospectivePlugin.capture_learnings`.

### SK ChatHistory — Sprint 2 (Core para iteración)

**Sprint 1:** El pipeline es lineal — el usuario pega JD, el agente ejecuta las fases, genera output. El contexto se maneja con `AgentContext` persistido en DB. No hay refinamiento multi-turno.

**Sprint 2:** Se integra SK `ChatHistory` para habilitar **refinamiento iterativo**. Sin esto, el producto es un generador de texto, no un Companion.

**Por qué es core (no nice-to-have):**
- Si el Killer Summary tiene el tono incorrecto, el usuario necesita poder decir "hazlo menos formal" sin repetir todo el flujo
- Si un Impact Bullet contiene un dato impreciso ("lideré" cuando en realidad "coordiné"), el usuario necesita corregirlo y que el agente aprenda
- Si las respuestas del portal (Fase 4) necesitan ajustes, el usuario debe poder iterar en varios turnos
- El principio "la segunda postulación es mejor que la primera" depende de que las correcciones alimenten el perfil

**Puntos de iteración en el pipeline:**

| Fase | Ejemplo de iteración |
|------|---------------------|
| 3 (Content) | "Este bullet está mal — no lideré el equipo, solo coordiné" → corrige bullet + actualiza perfil |
| 4 (Application) | "La respuesta a '¿por qué te interesa?' no refleja mi motivación real" → refina |
| 5 (Verification) | "Ese dato no es correcto, la reducción fue del 40%, no del 60%" → corrige |

**Complejidad de implementación:**

| Componente | Esfuerzo | Notas |
|---|---|---|
| Instanciar SK ChatHistory | Trivial | 5-10 líneas |
| Persistir historial entre requests HTTP | 1-2 días | Guardar en DB con session_id, cargar al retomar |
| Gestión de tokens (historial largo) | Medio | Solo incluir últimas N interacciones relevantes, no todo |
| Conectar correcciones con actualización de perfil | 1-2 días | Cuando el usuario corrige un dato, actualizar `profiles`/`work_experiences` |
| **Total estimado** | **2-3 días** | |

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
    ├── culture_radar.py       # CultureRadarPlugin (Fase 1)
    ├── requirement_classifier.py # RequirementClassifierPlugin (Fase 2 — LLM)
    ├── sas_evaluator.py       # SASEvaluatorPlugin (Fase 2 — LLM)
    ├── content_drafting.py    # ContentDraftingPlugin (Fase 3)
    ├── application_support.py # ApplicationSupportPlugin (Fase 4)
    ├── verification.py        # VerificationPlugin (Fase 5)
    ├── brand_coherence.py     # BrandCoherencePlugin (Fase 6)
    └── retrospective.py       # RetrospectivePlugin (Fase 7)
```

---

## Tabla Resumen: LLM vs Código

| Fase | Tarea | Ejecutor | Modelo |
|------|-------|----------|--------|
| Pre-pipeline | Validación de completitud de perfil | **Código Python** | — |
| Pre-pipeline | Intent classification | LLM | GPT-4o-mini |
| 1 | Análisis de empresa y cultura (solo texto JD, sin web) | LLM | GPT-4o |
| 2 | Cache lookup + drift check de clasificaciones previas | **Código Python** | — |
| 2 | Clasificación de requisitos A-D + match (una sola llamada) | LLM | GPT-4o |
| 2 | **Evaluación de las 5 dimensiones del SAS** | **LLM (GPT-4o)** | GPT-4o |
| 2 | **Cálculo de Confidence Score + MVP** | **Código Python** | — |
| 2 | **Suma del SAS total** | **Código Python** | — |
| 2 | **Validación de Non-Negotiables** | **Código Python** | — |
| 2 | Anti-Impostor: pausa interactiva si hay 2+ A/B "no_cumple" | Orchestrator (lógica) | — |
| 3 | Generación de Killer Summary | LLM | GPT-4o |
| 3 | Generación de Impact Bullets | LLM | GPT-4o |
| 3 | Generación de Skills Matrix | LLM | GPT-4o |
| 4 | Preview de campos de plataforma | **Código (DB lookup)** | — |
| 4 | Confirmación de campos con usuario | Orchestrator (flujo) | — |
| 4 | Respuestas a preguntas del portal | LLM | GPT-4o |
| 5 | Verificación de integridad factual | LLM | GPT-4o |
| 5 | Generación de Interview Intel | LLM | GPT-4o |
| 6 | Auditoría de coherencia de marca | LLM | GPT-4o |
| 7 | Las 4 preguntas de retrospectiva | Orchestrator (flujo) | — |
| 7 | Captura y estructuración de learnings | LLM | GPT-4o |
| 7 | Evaluación de generalizabilidad | LLM | GPT-4o |

---

## Documentos Relacionados

- `scoring_protocol.md` — Fórmulas exactas que implementa el ScoringEngine
- `framework_protocol.md` — Las 7 fases originales del framework markdown
- `DATABASE.md` — Schema de DB que alimenta el ProfileSnapshot
- `SECURITY.md` — IntentClassifier como primera capa del pipeline
- `STACK.md` — Semantic Kernel como framework de orquestación
