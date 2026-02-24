# Sprint Breakdown — Career Companion MVP

> **Estado:** Definido. Nivel intermedio (scope por milestone, no day-by-day).
> **Última actualización:** 2026-02-24

---

## Estructura General

El MVP se construye en **2 sprints** con **9 milestones** internos. Cada milestone entrega algo funcional end-to-end (vertical slice).

```
Sprint 1: Pipeline Funcional (lineal)     ~20-30 días
├── M1: Foundation (repo + infra + schema)
├── M2: Auth (signup/login funcional)
├── M3: Profile (CRUD + ProfileSnapshot)
├── M4: Scoring Engine (determinista, TDD)
├── M5: Agent Core (plugins SK + orchestrator + streaming)
├── M6: Chat + Resultados (UI completa de postulación)
├── M7: Application Flow (plataformas + historial)
├── M8: Security (intent classifier + quotas)
└── M9: Landing + Polish

Sprint 2: Companion Features                ~6-9 días
├── M10: Iterative Refinement (ChatHistory + correcciones → perfil)
└── M11: Web Research (Bing Search + SAS Culture upgrade)
```

---

## Paralelización: Backend vs Frontend (v0.dev)

Una ventaja clave: **el frontend se puede generar en paralelo con v0.dev** mientras el backend se construye. Esto comprime el timeline real.

```
Timeline simplificado:

Semana 1-2:  Backend M1-M3  |  Frontend: v0.dev genera páginas con interfaces TS (SP-014)
Semana 2-3:  Backend M4-M5  |  Frontend: integrar auth real + conectar API de profiles
Semana 3-4:  Backend M5-M6  |  Frontend: chat UI + streaming + resultados
Semana 4-5:  M7-M8-M9       |  Frontend: historial + landing + polish
Semana 5-6:  Sprint 2        |  Frontend: refinamiento UI iterativo
```

**Requisito para que funcione:** SP-014 (UI Spec) debe estar listo antes de empezar. v0.dev necesita: interfaces TS, descripción de cada página, y restricciones de diseño.

---

## Sprint 1: Pipeline Funcional

### M1: Foundation
**Entrega:** Repo inicializado con ambos stacks, DB creada, CI básico.

| Componente | Scope |
|---|---|
| Repo | Crear `career-companion/` con estructura de STACK.md (frontend/ + backend/ + docs/ + supabase/) |
| Backend | Django project + Django Ninja configurado + pyproject.toml + Ruff |
| Frontend | Next.js 15 App Router + shadcn/ui + Biome + Vitest config |
| DB | Supabase project creado + todas las migrations de DATABASE.md ejecutadas |
| CI | GitHub Actions: lint frontend + lint backend (path filters) |
| Dev tools | Husky + lint-staged, script de OpenAPI codegen |
| Docs | CLAUDE.md del nuevo repo con convenciones, /session skill (SP-013) |

**Depende de:** Nada. Es el punto de partida.
**Desbloquea:** Todo lo demás.
**Paralelizable con v0.dev:** Mientras se hace M1, se puede estar generando UI en v0.dev con las interfaces TS de SP-014.

---

### M2: Auth
**Entrega:** Usuario puede hacer signup/login con Google o GitHub. RLS funcional.

| Componente | Scope |
|---|---|
| Supabase Auth | Google + GitHub OAuth configurados |
| Backend | Middleware Django que valida JWT de Supabase, extrae user_id |
| Frontend | Páginas /login y /signup, redirect post-auth, session management |
| Trigger | `handle_new_user()` crea profile + usage_quotas automáticamente |
| Test | Login funcional end-to-end en dev |

**Depende de:** M1
**Desbloquea:** M3, M8

---

### M3: Profile
**Entrega:** Usuario puede ver y editar su perfil completo. ProfileSnapshot funcional.

| Componente | Scope |
|---|---|
| API | CRUD endpoints para profiles, skills, work_experiences, projects, professional_strengths, education, work_preferences |
| ProfileSnapshot | Función que hace JOIN de todas las tablas del usuario y construye el snapshot Pydantic |
| OpenAPI codegen | Generar tipos TS desde los schemas Pydantic |
| Frontend | Editor de perfil multi-sección (puede ser tabs o acordeón) |
| Completeness | Calcular % de completitud del perfil |
| Test | Crear perfil, editar skills, verificar que ProfileSnapshot refleja cambios |

**Depende de:** M2 (necesita auth para saber quién es el usuario)
**Desbloquea:** M4, M5 (el scoring y el agente necesitan el ProfileSnapshot)
**v0.dev:** Generar el editor de perfil con las interfaces TS. Es la página más compleja en formularios.

---

### M4: Scoring Engine
**Entrega:** Scoring determinista funcional con tests. Endpoint expuesto.

| Componente | Scope |
|---|---|
| ScoringEngine | `calculate_confidence_score()` + `calculate_sas()` en Python puro |
| Tests TDD | Todos los tests de scoring_protocol.md: tipo D peso 0.5, transferible 0.7, MVP cap 100, determinismo |
| Schemas | `ClassifiedRequirement`, `ConfidenceScoreResult`, `SASResult`, `ScoringResult` |
| Endpoint | POST `/api/scoring/calculate` que recibe requirements clasificados + strengths → score |
| Validación | Non-Negotiables checker (reglas binarias en código) |

**Depende de:** M1 (solo necesita el proyecto Django, no necesita auth ni profile para los tests)
**Desbloquea:** M5 (el orchestrator necesita el scoring engine)
**Nota:** Este milestone se puede hacer en paralelo con M2 y M3 — solo necesita M1.

---

### M5: Agent Core
**Entrega:** Pipeline completo ejecutable. Screening y CV generation funcionan end-to-end.

| Componente | Scope |
|---|---|
| SK Setup | Semantic Kernel configurado con Azure OpenAI |
| Plugins | CultureRadarPlugin, RequirementClassifierPlugin, SASEvaluatorPlugin, ContentDraftingPlugin, VerificationPlugin, BrandCoherencePlugin |
| Orchestrator | `run_screening()` y `run_full_pipeline()` con persistencia por fase |
| AgentContext | Schema completo con `last_completed_phase` |
| Retry wrapper | `_invoke_with_retry()` con validación Pydantic |
| Cache | Classification cache lookup + drift detection |
| Anti-Impostor | Pausa interactiva si 2+ requisitos A/B son no_cumple |
| Streaming | Endpoint SSE/WebSocket que emite segmentos del pipeline |
| Tests | Spec-driven: screening de un JD real contra un perfil de prueba → score coherente |

**Depende de:** M3 (ProfileSnapshot) + M4 (ScoringEngine)
**Desbloquea:** M6 (el chat UI necesita el backend de agente)
**Es el milestone más pesado** (~3-5 días). Aquí se conectan todas las piezas.

---

### M6: Chat + Resultados
**Entrega:** Usuario pega JD, ve scores, obtiene CV generado. La experiencia core.

| Componente | Scope |
|---|---|
| Chat UI | Interfaz de chat con Vercel AI SDK, streaming de respuesta |
| Vista de resultados | Componente estructurado: scores + gap analysis + CV content por sección + Interview Intel |
| Copy buttons | Copiar cada sección al clipboard |
| Guardar | Output se guarda automáticamente en DB (tabla outputs) |
| Frontend | Páginas /chat y /results/[id] |

**Depende de:** M5 (backend de agente funcional)
**Desbloquea:** M7
**v0.dev:** Chat UI y vista de resultados son buenos candidatos para v0.dev — shadcn tiene componentes de chat.

---

### M7: Application Flow
**Entrega:** Soporte de plataformas de postulación + historial.

| Componente | Scope |
|---|---|
| ApplicationSupportPlugin | Respuestas a preguntas de portal coherentes con CV |
| Platform preview | `get_platform_questions_preview()` + DB lookup de platform_schemas |
| RetrospectivePlugin | Captura de learnings + evaluación de generalizabilidad |
| Seed data | 5 platform schemas iniciales (GetOnBoard, LinkedIn, Computrabajo, Indeed, Laborum) — depende de SP-012 (tarea del owner) |
| Historial | Tabla de postulaciones: empresa, rol, fecha, score, estado. Filtros básicos. Click → resultado. |
| Dashboard | Página /dashboard: perfil (% completitud) + historial + [+ Nueva Postulación] |

**Depende de:** M6 (necesita que el flujo de chat funcione)
**Desbloquea:** M8
**Nota:** SP-012 (análisis manual de plataformas) debe estar hecho para tener seed data real.

---

### M8: Security
**Entrega:** El sistema rechaza mensajes off-topic y respeta quotas.

| Componente | Scope |
|---|---|
| Intent classifier | GPT-4o-mini como primer filtro antes del pipeline |
| Quotas | Middleware que verifica usage_quotas antes de procesar (mensajes/día, screenings/día, CVs/mes) |
| Rate limiting | Por IP y por usuario |
| Azure Content Safety | Integración para detectar prompt injection |
| Mensajes amigables | Cada tipo de error tiene mensaje humano (ver STACK.md) |
| Tests TDD | Intent classifier acepta career queries, rechaza off-topic. Quotas bloquean al límite. |

**Depende de:** M5 (necesita el pipeline para poner el classifier delante)
**Desbloquea:** M9
**Nota:** Se puede empezar en paralelo con M7 — no se bloquean mutuamente.

---

### M9: Landing + Polish
**Entrega:** Landing page pública + pulido general.

| Componente | Scope |
|---|---|
| Landing | Next.js SSG: propuesta de valor, demo visual del flujo, CTA a signup, SEO básico |
| Polish | Responsive check, error states, loading states, empty states |
| Onboarding flow | Flujo de primera vez: "¿Cómo prefieres empezar?" → formulario/texto/CV existente |
| Docs | README del nuevo repo actualizado |

**Depende de:** M6+ (necesita el producto funcionando para la demo visual)
**v0.dev:** Landing page es ideal para v0.dev.

---

## Sprint 2: Companion Features

### M10: Iterative Refinement
**Entrega:** Usuario puede corregir y refinar el output. Las correcciones actualizan el perfil.

| Componente | Scope |
|---|---|
| SK ChatHistory | Integrar con el pipeline, persistir en DB con session_id |
| Refinement flow | Usuario dice "este bullet está mal" → agente corrige output + pregunta si actualizar perfil |
| Profile update | Cuando el usuario corrige un dato factual → actualizar tabla correspondiente (work_experiences, skills, etc.) |
| Token management | Solo incluir últimas N interacciones relevantes en el historial para no explotar tokens |
| Tests | Spec-driven: usuario corrige un bullet → output cambia + perfil se actualiza |

**Depende de:** Sprint 1 completo
**Desbloquea:** M11

---

### M11: Web Research
**Entrega:** CultureRadar investiga la empresa de verdad. SAS Cultura es una evaluación real.

| Componente | Scope |
|---|---|
| Bing Search API | Configurar como tool de SK, disponible para CultureRadarPlugin |
| CultureRadar upgrade | Buscar: blog técnico, Glassdoor, noticias, GitHub org |
| SAS Cultura upgrade | La dimensión "Valores y Cultura" ya no es low-confidence — evalúa con datos reales |
| Fallback | Si Bing Search falla o no encuentra nada → degradar a modo texto-only con aviso |
| Cost tracking | Loggear queries de Bing para monitorear costo |

**Depende de:** M10 (orden lógico, aunque técnicamente podría ser paralelo)

---

## Diagrama de Dependencias

```
M1 (Foundation)
├── M2 (Auth) ──── M3 (Profile) ──┐
│                                   ├── M5 (Agent Core) ── M6 (Chat+Results) ── M7 (App Flow) ── M9 (Landing)
└── M4 (Scoring) ─────────────────┘                       │
                                                            └── M8 (Security) ─────────────────┘
                                                                                    │
                                                                              Sprint 2:
                                                                         M10 (Refinement) ── M11 (Web Research)

Paralelo con v0.dev:
M1 → v0 genera todas las páginas con interfaces TS
M2 → v0 login/signup ya generado, solo conectar auth real
M3 → v0 editor de perfil ya generado, solo conectar API
M6 → v0 chat UI ya generado, solo conectar streaming
M9 → v0 landing ya generada, solo ajustar copy
```

---

## Criterio de "Done" por Milestone

Cada milestone sigue el ciclo fractal (Geometría Sagrada):

1. **Planificar** — Leer spec del milestone, identificar archivos a crear/modificar
2. **Ejecutar** — Implementar con Claude Code
3. **Validar** — Tests pasan, funciona en dev, no rompe nada existente
4. **Documentar** — Commit descriptivo, CLAUDE.md actualizado si cambia estructura

Un milestone NO está terminado si:
- Hay tests fallando
- Faltan error states o loading states en la UI
- El código no pasa lint (Biome/Ruff)
- No hay commit

---

## Documentos Relacionados

- `MVP.md` — Features del MVP y flujo del usuario
- `DATABASE.md` — Schema que M1 implementa
- `AGENT_PIPELINE.md` — Plugins que M5 implementa
- `STACK.md` — Stack y convenciones que todo milestone sigue
- `SECURITY.md` — Modelo de seguridad que M8 implementa
- `UI_SPEC.md` — (Pendiente SP-014) Spec de páginas y componentes para v0.dev
