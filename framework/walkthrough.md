# Walkthrough: Usando el CV Optimization Framework

Este framework es tu **abogado defensor en la búsqueda laboral**. Analiza ofertas de trabajo, calcula qué tan fuerte es tu posición, y genera contenido optimizado que presenta tu experiencia real de la mejor manera posible.

**Filosofía:** Los JDs son wish-lists infladas. Las empresas piden el candidato ideal sabiendo que no existe. Este framework busca activamente el mejor encuadre legítimo de tu experiencia — sin inventar nada, pero sin quedarse pasivo ante gaps menores.

---

## Sobre los JDs (Job Descriptions)

Antes de empezar, algo importante:

- Los JDs son **listas aspiracionales**, no checklists eliminatorios
- Las empresas rutinariamente piden 5+ años para roles que alguien con 2 años puede hacer bien
- "Requisitos obligatorios" son a menudo negociables, especialmente si demuestras capacidad de aprendizaje
- El candidato promedio aplica cumpliendo ~60% de los requisitos

Este framework te ayuda a evaluar con honestidad dónde estás y construir el mejor caso posible con lo que tienes.

---

## Los 3 Modos de Operación

| Modo | Fases | Duración | Resultado |
|------|-------|----------|-----------|
| **Screening Rápido** | 1-2 | ~5 min | Veredicto + scores para decidir si vale la pena |
| **Proceso Completo** | 1-7 | Variable | CV optimizado + Interview Intel + Learnings |
| **Onboarding** | — | ~30-45 min | Perfil base y brand voice configurados |

---

## Las 7 Fases del Proceso Completo

1. **Research & Culture Radar** — Investigar empresa, pain points, stack tecnológico
2. **Alignment Score** — Calcular Confidence Score + SAS (Strategic Alignment Score)
3. **Context Mapping & Drafting** — Generar Killer Summary, Impact Bullets, Skills Matrix
4. **Application Support** — Responder preguntas del portal de postulación (bajo demanda)
5. **Verification & Review** — Validar integridad factual + preparar Interview Intel
6. **Brand Coherence Audit** — Asegurar que el output refleja tu voz auténtica
7. **Retrospectiva** — Capturar learnings para evolucionar el framework

---

## Cómo Ejecutar

### Opción A: Sesión Nueva con Cualquier LLM

1. **Copia el contenido de `AGENT_START.md`** y pégalo en una nueva sesión de IA
2. **Adjunta `../private/perfil_base.md`** y **`../private/brand_voice.md`** para dar contexto
3. **Proporciona la Job Description** que te interesa (texto completo + link)
4. El agente ejecutará las 7 fases automáticamente

### Opción B: Sesión con Acceso a Archivos (Claude Code, Cursor, etc.)

1. Indica que deseas optimizar tu CV para una oferta
2. Proporciona el texto de la JD y el nombre de la empresa
3. El agente leerá automáticamente los archivos necesarios

---

## Archivos del Framework

| Archivo | Propósito | Cuándo Usar |
|---------|-----------|-------------|
| `AGENT_START.md` | Entry point para agentes | Siempre primero |
| `framework_protocol.md` | Reglas detalladas del workflow (7 fases) | Siempre |
| `scoring_protocol.md` | Fórmulas de Confidence Score y SAS | Fase 2 |
| `../private/perfil_base.md` | Fuente de verdad del candidato | Siempre |
| `../private/brand_voice.md` | Referencia de tono y estilo | Fases 3, 6 |
| `../private/learnings.md` | Registro de aprendizajes | Fase 7 |
| `../private/application_tracker.json` | Historial de postulaciones | Contexto |
| `../private/outputs/INDEX.md` | Índice de CVs generados | Referencia |
| `onboarding/` | Sistema de onboarding para perfiles nuevos | Primera vez |
| `common_patterns.md` | Patrones generalizados para la comunidad | Referencia |
| `roadmap.md` | Visión de producto | Si se pregunta |

---

## ¿Primera Vez?

Si es tu primera vez usando el framework:

1. Copia las plantillas de `templates/` a `private/`
2. Usa el **modo Onboarding** (opción 3 al iniciar) para llenar tu perfil con preguntas guiadas
3. O llena manualmente `perfil_base.md` y `brand_voice.md` siguiendo los ejemplos en las plantillas

---

## ¿Listo?

Si tienes tu perfil actualizado y una Job Description, podemos empezar ahora mismo.
