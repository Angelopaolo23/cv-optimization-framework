# CV Optimization Framework

Un framework de **Context Engineering** para optimizar CVs y perfiles profesionales mediante agentes de IA. Diseñado para construir el caso más fuerte posible para un candidato — con integridad, coherencia de marca, y scoring estructurado.

**Filosofía central:** El agente actúa como abogado defensor del candidato, no como auditor. Los JDs son listas aspiracionales — el framework encuentra el mejor encuadre legítimo de la experiencia real.

## Características

- **Candidate Advocacy** - El agente defiende activamente al candidato, busca transferibilidad, y combate el síndrome del impostor
- **Scoring Integral** - Confidence Score (técnico, con clasificación A-D + MVP no-técnico) + SAS (alineación estratégica)
- **Clasificación de Requisitos** - Tipo A (filtro duro), B (obligatorio), C (deseable real), D (inflado) — no todos los requisitos pesan igual
- **Modificador de Valor Profesional** - Captura liderazgo, experiencia de negocio, y velocidad de aprendizaje que el JD no pregunta pero que importa
- **3 Modos de Operación** - Screening Rápido, Proceso Completo (7 fases), Onboarding
- **LLM-Agnóstico** - Funciona con Claude, GPT-4, Gemini, o cualquier modelo capaz
- **Feedback Loop** - Learnings privados que escalan a patrones públicos reutilizables

## Flujo de Trabajo

```
┌─────────────────┐
│   Primera vez?  │──▶ Onboarding (poblar perfil_base.md)
└────────┬────────┘
         │ No (perfil existente)
         ▼
┌─────────────────┐
│   JD + Empresa  │
└────────┬────────┘
         │
         ▼
┌─────────────────┐     ┌─────────────────┐
│    Screening    │────▶│    Veredicto    │──▶ ¿Continuar?
│   (Fases 1-2)   │     │  🟢 🟡 🔴      │
└─────────────────┘     └─────────────────┘
         │                      │
         │ Sí                   │ No
         ▼                      ▼
┌─────────────────┐     ┌─────────────────┐
│ Proceso Completo│     │ Siguiente Oferta│
│  (7 Fases)      │     └─────────────────┘
└────────┬────────┘
         │
         ▼
┌─────────────────────────────┐
│  CV Optimizado + Interview  │
│  Intel + Brand Audit +      │
│  Retrospectiva (Learnings)  │
└─────────────────────────────┘
```

## Las 7 Fases

1. **Research & Culture Radar** - Investigar empresa, pain points, stack
2. **Alignment Score** - Clasificar requisitos (A-D), calcular Confidence + MVP + SAS
3. **Context Mapping & Drafting** - Generar Killer Summary, Impact Bullets, Skills Matrix
4. **Application Support** - Responder preguntas del portal (bajo demanda)
5. **Verification & Review** - Validar integridad + Advocacy Checklist + Interview Intel
6. **Brand Coherence Audit** - Asegurar autenticidad y coherencia de voz
7. **Retrospectiva** - Capturar learnings y consolidar en documentos base

## Estructura del Repositorio

```
/
├── README.md                       # Este archivo
├── framework/                      # Lógica del sistema (público)
│   ├── AGENT_START.md              # Entry point para agentes — leer primero
│   ├── framework_protocol.md       # Reglas de operación y 7 fases del workflow
│   ├── scoring_protocol.md         # Fórmulas: Confidence Score, MVP, SAS
│   ├── walkthrough.md              # Guía de uso para el candidato
│   ├── common_patterns.md          # Anti-patrones y patrones validados
│   ├── roadmap.md                  # Visión de producto V1-V4
│   └── onboarding/                 # Sistema de onboarding para nuevos usuarios
│       ├── README.md               # Selección de modo
│       ├── mode_a_guided.md        # Flujo guiado (5 fases de preguntas)
│       └── mode_b_freeform.md      # Flujo libre (escucha reactiva)
│
├── templates/                      # Plantillas para personalizar (punto de partida)
│   ├── perfil_base.template.md     # Con guía de uso y ejemplos concretos
│   ├── brand_voice.template.md     # Con ejemplos de tono y killer summary
│   └── learnings.template.md       # Estructura de feedback loop
│
└── private/                        # Tu información personal (en .gitignore)
    ├── perfil_base.md              # Fuente de verdad (experiencia, skills, restricciones)
    ├── brand_voice.md              # Voz y estilo documentados
    ├── learnings.md                # Registro de aprendizajes por sesión
    ├── application_tracker.json    # Historial de postulaciones
    └── outputs/
        ├── INDEX.md                # Índice de todos los CVs generados
        └── cv_*.md / screening_*.md
```

## Cómo Usar

### Primera Vez (Onboarding)

```bash
# Clona el repositorio
git clone [url]

# Crea tu carpeta privada
mkdir -p private/outputs

# Copia las plantillas
cp templates/perfil_base.template.md private/perfil_base.md
cp templates/brand_voice.template.md private/brand_voice.md
cp templates/learnings.template.md private/learnings.md
```

Abre una sesión con tu LLM y di:
> "Lee `framework/AGENT_START.md` y quiero hacer onboarding"

El agente guiará la configuración inicial de tu perfil.

### Uso Regular

Abre una sesión con tu LLM y di:
> "Lee `framework/AGENT_START.md` — tengo una oferta para analizar"

El agente preguntará: **Screening Rápido, Proceso Completo, u Onboarding**.

### Iterar y Mejorar

Después de cada sesión, los learnings se capturan en `private/learnings.md`. Los que se repiten escalan a `framework/common_patterns.md` para beneficiar a todos los usuarios.

## Scoring

### Confidence Score (0-100%)

Qué tan fuerte es el caso del candidato considerando match técnico + valor profesional integral.

```
Confidence Final = min(Base Score + MVP, 100)

Base Score = (Obligatorios [Tipo A+B] × 60%) + (Deseables [Tipo C+D] × 40%)

Clasificación de requisitos:
  Tipo A — Filtro Duro  (knockout real)              × 1.0
  Tipo B — Obligatorio  (importante, negociable)     × 1.0
  Tipo C — Deseable Real (diferenciador)             × 1.0
  Tipo D — Deseable Inflado (wishlist irreal)        × 0.5

Estados de match:
  ✅ Cumple       (skill ≥ nivel 3 en Power Stack)  → 1.0
  🔄 Transferible (equivalente con evidencia)        → 0.7
  🟡 Parcial      (skill nivel 2, exposición)        → 0.5
  ❌ No cumple                                       → 0.0

MVP (Modificador de Valor Profesional): +0 a +15 pts
  Liderazgo y Gestión          0-5 pts
  Experiencia Industria/Negocio 0-5 pts
  Velocidad de Aprendizaje     0-5 pts
```

### Strategic Alignment Score — SAS (0-100%)

Qué tanto te conviene esta oportunidad según tus metas y valores documentados en `perfil_base.md`.

5 dimensiones × 20 pts cada una:
- Metas de Carrera
- Motivaciones Intrínsecas
- Valores y Cultura
- Crecimiento Técnico
- Autonomía y Tipo de Rol

## Roadmap

- [x] V1.0: Core Architecture — 7 fases, scoring básico, templates
- [x] V1.5: Hardening — filosofía abogado defensor, scoring integral (A-D + MVP), onboarding, tracker, patrones
- [ ] V2.0: Interview Playbooks + Multi-plataforma (LinkedIn, GitHub)
- [ ] V3.0: Auto-Discovery de ofertas + adapters (LinkedIn Jobs, GetOnBoard)
- [ ] V4.0: Career Companion — paths de aprendizaje, seguimiento de carrera

## Tecnologías Utilizadas

Este framework es un ejemplo de:
- **Context Engineering** - Diseño de contexto estructurado para LLMs
- **Agentic Workflows** - Flujos de trabajo ejecutados por agentes de IA
- **Prompt Engineering** - Instrucciones optimizadas para consistencia

## Licencia

MIT - Siéntete libre de usar y adaptar para tu propio proceso de búsqueda laboral.

---

*Desarrollado como proyecto de Context Engineering para demostrar el uso de agentes de IA en optimización de procesos personales.*
