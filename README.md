# CV Optimization Framework

Un framework de **Context Engineering** para optimizar CVs y perfiles profesionales mediante agentes de IA. Diseñado para acelerar postulaciones a trabajos técnicos manteniendo integridad y coherencia de marca personal.

## Características

- **Zero Hallucination Policy** - Todo contenido debe estar respaldado por experiencia documentada
- **LLM-Agnóstico** - Funciona con Claude, GPT-4, Gemini, o cualquier modelo capaz
- **Modo Screening** - Evaluación rápida de ofertas antes de invertir tiempo
- **Modo Completo** - Optimización full con 7 fases de análisis
- **Scoring Estructurado** - Confidence Score (técnico) + SAS (alineación estratégica)
- **Feedback Loop** - Sistema de learnings para evolución continua

## Flujo de Trabajo

```
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
┌─────────────────┐
│  CV Optimizado  │
│  + Interview    │
│    Intel        │
└─────────────────┘
```

## Las 7 Fases

1. **Research & Culture Radar** - Investigar empresa, pain points, stack
2. **Alignment Score** - Calcular Confidence Score + SAS
3. **Context Mapping & Drafting** - Generar Killer Summary, Impact Bullets
4. **Application Support** - Responder preguntas del portal (bajo demanda)
5. **Verification & Review** - Validar integridad + preparar Interview Intel
6. **Brand Coherence Audit** - Asegurar autenticidad de voz
7. **Retrospectiva** - Capturar learnings para evolucionar

## Estructura del Repositorio

```
/
├── README.md                 # Este archivo
├── framework/                # Lógica del sistema (público)
│   ├── AGENT_START.md        # Entry point para agentes
│   ├── framework_protocol.md # Reglas y fases del workflow
│   ├── scoring_protocol.md   # Fórmulas de cálculo de scores
│   ├── walkthrough.md        # Guía de uso
│   └── roadmap.md            # Visión de producto
│
├── templates/                # Plantillas para personalizar
│   ├── perfil_base.template.md
│   ├── brand_voice.template.md
│   └── learnings.template.md
│
└── private/                  # Tu información (en .gitignore)
    ├── perfil_base.md
    ├── brand_voice.md
    ├── learnings.md
    └── outputs/
```

## Cómo Usar

### 1. Configuración Inicial

```bash
# Clona el repositorio
git clone [url]

# Crea tu carpeta privada
mkdir -p private/outputs

# Copia las plantillas y personalízalas
cp templates/perfil_base.template.md private/perfil_base.md
cp templates/brand_voice.template.md private/brand_voice.md
cp templates/learnings.template.md private/learnings.md

# Edita tus archivos personales
# - private/perfil_base.md (tu experiencia, skills, restricciones)
# - private/brand_voice.md (tu tono y estilo de comunicación)
```

### 2. Ejecutar con un Agente

Abre una sesión con tu LLM preferido (Claude, GPT-4, Gemini) y di:

> "Lee el archivo `framework/AGENT_START.md` y luego te paso una oferta laboral"

El agente:
1. Preguntará si quieres **Screening Rápido** o **Proceso Completo**
2. Solicitará tus archivos de `private/` si no tiene acceso
3. Ejecutará el flujo correspondiente

### 3. Iterar y Mejorar

Después de cada sesión, los learnings se capturan en `private/learnings.md` y pueden consolidarse en tus documentos base.

## Scoring

### Confidence Score (0-100%)
Probabilidad técnica de ser considerado candidato viable.

```
Confidence = (Obligatorios × 60%) + (Deseables × 40%)
```

### Strategic Alignment Score - SAS (0-100%)
Qué tanto te conviene esta oportunidad según tus metas y valores.

5 dimensiones × 20 pts cada una:
- Metas de Carrera
- Motivaciones Intrínsecas
- Valores y Cultura
- Crecimiento Técnico
- Autonomía y Tipo de Rol

## Roadmap

- [x] V1: Core Architecture
- [x] V1.1: Screening Mode + Scoring Protocol
- [ ] V2: Interview Playbooks
- [ ] V3: Auto-Discovery de ofertas
- [ ] V4: SaaS con UI

## Tecnologías Utilizadas

Este framework es un ejemplo de:
- **Context Engineering** - Diseño de contexto estructurado para LLMs
- **Agentic Workflows** - Flujos de trabajo ejecutados por agentes de IA
- **Prompt Engineering** - Instrucciones optimizadas para consistencia

## Licencia

MIT - Siéntete libre de usar y adaptar para tu propio proceso de búsqueda laboral.

---

*Desarrollado como proyecto de Context Engineering para demostrar el uso de agentes de IA en optimización de procesos personales.*
