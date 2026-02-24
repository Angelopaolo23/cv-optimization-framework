# Stack Técnico — Career Companion Platform

> **Estado:** Definido. Decisiones finales tomadas 2026-02-23.
> **Última actualización:** 2026-02-23

---

## Visión del Producto

Transformar el CV Optimization Framework (sistema de archivos markdown + agente LLM) en una plataforma web de Career Companion. El usuario interactúa con un agente de IA que lo ayuda a optimizar su perfil profesional, evaluar ofertas laborales, y gestionar su búsqueda de empleo — con integridad, determinismo en scoring, y adaptabilidad a múltiples plataformas de CV.

**Modelo de negocio:** SaaS con suscripción mensual (precio objetivo ~$15 USD/mes).

---

## Filosofía de Desarrollo: Geometría Sagrada

Este proyecto se rige por principios de Geometría Sagrada aplicados al desarrollo de software. Estos principios son **obligatorios** para cualquier agente o desarrollador que trabaje en el código:

### 1. El Centro (Single Source of Truth)
Todo sistema tiene un centro desde el cual emana la información. Antes de codificar, identifica el "núcleo" de la tarea. No permitas que la lógica se disperse. Mantén simetría total entre el Design System y la implementación. Si el centro (contexto) está desalineado, el resto de los nodos (componentes) fallarán.

**En este proyecto:** Pydantic schemas son el SSOT del backend. OpenAPI es el contrato. Los tipos TS generados son derivados, nunca fuente.

### 2. Estructura de Nodos (Modularidad Atómica)
Cada componente debe ser una entidad autosuficiente, con interfaces claras y sin dependencias innecesarias que rompan la simetría del sistema.

**En este proyecto:** Cada plugin de SK, cada service de Django, cada componente React — autosuficiente, testeable en aislamiento, con inputs/outputs explícitos.

### 3. Iteración Fractal (Macro-to-Micro)
"Como es en el Roadmap, es en el Commit". Cada tarea sigue el ciclo: **Planificar → Ejecutar → Validar → Documentar**. No se considera terminada una tarea si no ha cumplido el ciclo completo.

**En este proyecto:** Ningún PR se mergea sin que la feature esté planificada (spec o test), ejecutada (código), validada (tests pasan, funciona), y documentada (CLAUDE.md actualizado si cambia la estructura).

### 4. Eficiencia de Flujo (Menor Resistencia)
Implementa siempre la solución más elegante y simple. Si una solución requiere demasiada explicación o código redundante, está rompiendo la simetría. Nombres de archivos, estructuras de carpetas y lógica de datos deben ser predecibles y armoniosos.

**En este proyecto:** No over-engineer. Si `fetch` nativo funciona, no agregar TanStack Query. Si un `if/else` es claro, no crear un Strategy pattern.

### 5. Deuda de Coherencia
A diferencia de la deuda técnica, la Deuda de Coherencia mide qué tan lejos estamos del diseño original y la armonía del sistema. El objetivo es mantener esta deuda en cero. El código debe "sentirse" como una sola pieza, sin parches ni inconsistencias.

**En este proyecto:** Antes de cada commit, verificar que el código nuevo se integra con el estilo, convenciones, y estructura existente. Un archivo que "se siente" diferente al resto del proyecto es deuda de coherencia.

---

## Stack Definido

### Decisiones Clave Resueltas

| Decisión | Resultado | Razón |
|---|---|---|
| **¿Monorepo o repos separados?** | Monorepo simple (sin Nx/Turborepo) | Un solo git log, un solo PR cross-stack. Path filters en CI. Más fácil separar después que juntar. |
| **¿Backend Python o TypeScript?** | Python (Django) | Bootcamp, ecosistema AI nativo, Semantic Kernel, certificación AI-102. |
| **¿Django Ninja o DRF?** | Django Ninja | Pydantic nativo (mismo mental model que Zod), OpenAPI auto, menos verbose. |
| **¿NestJS monorepo full TS?** | No | Pierde Python para AI + bootcamp. OpenAPI codegen resuelve shared types. |
| **¿Vite?** | No | Next.js tiene Turbopack. Son alternativas, no complementos. |
| **¿Deploy backend?** | Azure App Service | Synergy con AI-102, narrativa para Startup Chile, enterprise-grade. Railway como fallback para dev rápido. |
| **¿Sync tipos Pydantic ↔ Zod?** | OpenAPI codegen | Pydantic → Django Ninja → openapi.json → openapi-typescript → types.ts |

### Frontend (MVP)

| Tecnología | Versión | Propósito |
|---|---|---|
| **Next.js** | 15+ (App Router) | Framework React con SSR/SSG |
| **TypeScript** | 5.x | Tipado estático |
| **shadcn/ui** | latest | Componentes copiables + Tailwind |
| **Tailwind CSS** | 4.x | Utilidades CSS |
| **Vercel AI SDK** | latest | Streaming de chat con agente |
| **Zod** | 3.x | Validación de datos en runtime |
| **React Hook Form** | latest | Formularios con integración Zod |

**Agregar cuando sea necesario:**
- **TanStack Query** — cuando el data fetching client-side se complique y `fetch` + Server Components no alcancen
- **Framer Motion** — cuando la UI necesite animaciones

### Backend (MVP)

| Tecnología | Versión | Propósito |
|---|---|---|
| **Python** | 3.12+ | Lenguaje backend |
| **Django** | 5.x | Framework web + ORM + Admin |
| **Django Ninja** | 1.x | API REST con Pydantic + OpenAPI auto |
| **Pydantic** | 2.x | Schemas tipados (SSOT del backend) |

**Agregar cuando sea necesario:**
- **Celery + Redis** — cuando haya jobs en background que no puedan ser síncronos (generación de CV larga, batch processing)
- **Redis solo** — cuando necesitemos rate limiting más sofisticado o cache

### Agente de IA

| Tecnología | Propósito | Sprint |
|---|---|---|
| **Semantic Kernel (Python)** | Orquestación del agente con plugins tipados | 1 |
| **Azure OpenAI** | Hosting de modelos LLM | 1 |
| **Azure Content Safety** | Protección contra prompt injection y jailbreak | 1 |
| **Bing Search API** | Web search para CultureRadar (research de empresa, cultura real) | 2 |

**Model routing:**

| Tarea | Modelo | Costo/call |
|---|---|---|
| Intent classification (¿es career-related?) | GPT-4o-mini | ~$0.0003 |
| Clasificación de requisitos A-D | GPT-4o | ~$0.02 |
| Generación de contenido (Summary, Bullets) | GPT-4o | ~$0.01-0.025 |
| Conversación de onboarding | GPT-4o | ~$0.01 |
| Cálculo de scores | **Código Python** | $0.00 |
| Validación de Non-Negotiables | **Código Python** | $0.00 |

### Infraestructura

| Tecnología | Propósito |
|---|---|
| **Supabase** | PostgreSQL + Auth (OAuth) + Storage + pgvector |
| **Vercel** | Deploy frontend (automático desde git) |
| **Azure App Service** | Deploy backend (producción). Railway para dev/staging si se necesita rapidez. |

> **Nota sobre Azure:** Azure App Service es la opción de producción por tres razones: (1) synergy con Azure OpenAI y Content Safety, (2) narrativa para Startup Chile y potencial funding, (3) demo para certificación AI-102. Si el setup inicial es demasiado fricción, Railway como staging hasta que el deploy Azure esté configurado.

### Testing (MVP)

| Herramienta | Capa | Propósito |
|---|---|---|
| **Vitest** | Frontend unit | Tests de componentes, hooks, utils |
| **Testing Library** | Frontend integration | Tests de componentes con interacción de usuario |
| **pytest** | Backend unit/integration | Scoring engine, API endpoints, plugins SK |
| **pytest-django** | Backend Django | Modelos, views, middleware |

**Agregar cuando sea necesario:**
- **Playwright** — E2E tests cuando haya flujos críticos multi-paso (onboarding completo, postulación end-to-end)
- **Hypothesis** — property-based testing cuando el scoring engine esté estable y se necesiten tests de propiedades
- **Factory Boy** — cuando haya muchos modelos y los fixtures manuales no escalen

### Herramientas de Desarrollo

| Herramienta | Propósito | Configuración |
|---|---|---|
| **Biome** | Lint + format TypeScript | Una vez, se olvida |
| **Ruff** | Lint + format Python | Una vez, se olvida |
| **Husky + lint-staged** | Pre-commit hooks | Una vez, previene código sucio |
| **openapi-typescript** | Codegen tipos TS desde OpenAPI | Script automático |
| **GitHub Actions** | CI/CD con path filters | Una vez, evoluciona con el proyecto |

---

## Approach de Desarrollo

### Metodología Híbrida: Spec-Driven + Test-Driven

No una sola metodología — la correcta para cada tipo de código:

| Tipo de código | Approach | Por qué |
|---|---|---|
| **Scoring engine** | **Test-Driven** | Es matemática pura. Los tests definen el comportamiento exacto. Se escriben ANTES del código. |
| **Seguridad (intent classifier, quotas)** | **Test-Driven** | Comportamiento crítico que debe ser exacto y verificable. |
| **Pipeline del agente (7 fases)** | **Spec-Driven** | La spec ya existe en los markdowns del framework. Se traduce a acceptance criteria. |
| **Flujos de usuario (onboarding, postulación)** | **Spec-Driven** | Flujos definidos en MVP.md. Se traducen a criterios verificables. |
| **CRUD, API endpoints, UI** | **Directo** | Claude Code genera + tests después si es necesario. No vale la pena especificar "create profile endpoint". |

### Vertical Slices (No Capas Horizontales)

Cada sprint construye una feature completa de punta a punta, no "todo el backend primero":

```
Sprint 1: Auth funcional (Supabase + Django + Next.js login)
Sprint 2: Perfil CRUD (API + formulario + DB)
Sprint 3: Scoring engine (Python + tests + endpoint)
Sprint 4: Chat básico (SK + streaming + UI)
Sprint 5: Flujo de postulación completo
Sprint 6: Seguridad (intent classifier + quotas)
Sprint 7: Landing page + polish
```

### Iteración Fractal en Cada Sprint

Cada tarea sigue el ciclo completo. Una tarea NO está terminada si no cumple los 4 pasos:

```
1. Planificar  → ¿Qué vamos a construir? ¿Spec o tests primero?
2. Ejecutar    → Claude Code implementa
3. Validar     → Tests pasan, funciona en dev, no rompe nada existente
4. Documentar  → CLAUDE.md actualizado si cambia estructura, commit descriptivo
```

---

## Convenciones (para CLAUDE.md del nuevo repo)

Estas convenciones se definirán en detalle cuando se cree el repositorio. Principios base:

### Backend
- Lógica de negocio en `services.py`, nunca en views
- Views solo orquestan: reciben request → llaman service → retornan response
- El frontend NUNCA habla directo con Azure OpenAI — siempre pasa por Django
- Pydantic schemas son la fuente de verdad de la API

### Frontend
- Server Components por defecto, `"use client"` solo donde hay interactividad
- Componentes en carpetas: `ComponentName/index.tsx` + `types.ts` + opcionalmente `hooks.ts`
- Zod schemas para validación de formularios, tipos generados desde OpenAPI para API calls

### Cross-Stack
- OpenAPI es el contrato entre frontend y backend
- Cuando se cambia un schema Pydantic, se regeneran los tipos TS antes de commitear

---

## Estructura del Repositorio

```
career-companion/
├── README.md
├── CLAUDE.md                     # Filosofía + convenciones para Claude Code
├── .github/
│   └── workflows/
│       ├── ci-frontend.yml       # Lint + test + build (path: frontend/**)
│       ├── ci-backend.yml        # Lint + test (path: backend/**)
│       └── deploy.yml
│
├── frontend/                     # Next.js app
│   ├── package.json
│   ├── next.config.ts
│   ├── biome.json
│   ├── vitest.config.ts
│   ├── src/
│   │   ├── app/                  # App Router pages
│   │   ├── components/           # shadcn + custom components
│   │   ├── lib/                  # Utils, API client, Zod schemas
│   │   ├── hooks/                # Custom React hooks
│   │   └── types/                # Generated types from OpenAPI
│   └── tests/
│
├── backend/                      # Django project
│   ├── pyproject.toml
│   ├── manage.py
│   ├── config/                   # Django settings, urls, wsgi
│   ├── apps/
│   │   ├── profiles/             # Perfil del usuario
│   │   ├── applications/         # Tracker de postulaciones
│   │   ├── scoring/              # Scoring engine (determinista)
│   │   ├── agent/                # Semantic Kernel integration
│   │   └── adapters/             # Platform adapters
│   ├── plugins/                  # Semantic Kernel plugins
│   └── tests/
│
├── docs/                         # Documentación de arquitectura
│   └── architecture/
│
├── framework/                    # Framework markdown original (referencia)
│
└── supabase/
    ├── migrations/
    └── seed.sql
```

---

## Logging y Manejo de Errores

### Logging — Principios

**Herramientas MVP:** `logging` de Python (formato JSON estructurado) + Vercel logs (frontend). Sin infraestructura adicional (ELK, Datadog) hasta que haya escala.

**Qué loggear en backend:**

| Evento | Qué registrar | Para qué |
|---|---|---|
| Llamada al agente | Modelo, tokens input/output, latencia, user_id | Control de costos, performance |
| Errores de Azure OpenAI | Tipo de error, retry count, request_id | Debugging, detectar outages |
| Intent classifier | Mensaje resumido, clasificación, razón si bloqueado | Calibrar el classifier |
| Scoring | Input hash, output score, timestamp | Auditoría, drift detection |
| Auth/quotas | User_id, quota actual, límite | Detectar abusers |

**Qué NO loggear:**
- Contenido completo de mensajes del usuario (privacidad)
- Datos del perfil (PII)
- API keys o tokens de sesión

**Frontend:** Mínimo. Errores de API no manejados + Vercel analytics. No agregar librerías de logging client-side.

**Formato backend (JSON estructurado):**
```python
# Cada log entry tiene estos campos
{
    "timestamp": "2026-02-23T10:30:00Z",
    "level": "info",
    "event": "agent_call",
    "user_id": "uuid",
    "model": "gpt-4o",
    "tokens_in": 2000,
    "tokens_out": 500,
    "latency_ms": 3200
}
```

### Manejo de Errores — Principios

**Regla de oro:** El usuario NUNCA ve un error técnico. Ve un mensaje humano que le dice qué pasó y qué puede hacer.

**Categorías de errores y comportamiento esperado:**

| Error | Qué ve el usuario | Qué hace el backend |
|---|---|---|
| Azure OpenAI timeout | "El agente está tardando más de lo normal. ¿Reintentamos?" | Retry 1x con backoff, si falla → mensaje al usuario |
| Azure rate limit | "Hay mucha demanda en este momento. Intenta en unos minutos." | Log + no retry (esperar) |
| LLM genera formato inesperado | Nada — el agente reintenta internamente | Retry con prompt más estricto, máx 2 intentos |
| Scoring recibe datos incompletos | "Tu perfil necesita [X] para calcular el score. ¿Lo completamos?" | Validación Pydantic antes de calcular, error descriptivo |
| Quota excedida | "Alcanzaste tu límite de [X] por hoy. Se renueva mañana." | Respuesta inmediata sin llamar al agente (ahorra tokens) |
| Error interno no manejado | "Algo salió mal de nuestro lado. Estamos revisando." | Log completo con stack trace, alerta si se repite |

**Principios de implementación:**
1. **Validar antes de procesar** — Pydantic valida inputs antes de que lleguen a la lógica de negocio. Los errores de validación son mensajes amigables, no excepciones.
2. **Fallar rápido, comunicar claro** — Si algo va a fallar, que falle antes de gastar tokens o tiempo.
3. **Retry con límite** — Máximo 2 retries para llamadas al LLM. Si falla 3 veces, es un problema real, no transitorio.
4. **Errores del agente vs errores del sistema** — Si el LLM no genera buen contenido, es un problema del prompt (fix interno). Si Azure se cae, es un problema de infra (comunicar al usuario).
5. **Las convenciones específicas se definen en el CLAUDE.md del nuevo repo** — Qué excepciones custom, qué middleware de error handling, qué formato de respuesta de error API.

---

## Decisiones Arquitectónicas Pendientes

| # | Decisión | Estado |
|---|---|---|
| ADR-001 | Deploy: Azure App Service como producción, Railway como dev/staging opcional | **Resuelto** |
| ADR-002 | Sync tipos: OpenAPI codegen (Pydantic → openapi.json → types.ts) | **Resuelto** |
| ADR-003 | ¿Celery o Azure Functions para jobs async? | **Diferido** — MVP sin jobs async |
| ADR-004 | Estado del chat: Vercel AI SDK streaming | **Resuelto** |
| ADR-005 | Multi-tenant: Supabase RLS desde el inicio | **Resuelto** |

---

## Documentos Relacionados

- `MVP.md` — Scope del MVP con 9 features core
- `SECURITY.md` — Modelo de seguridad y 5 capas anti-abuse
- `COST_MODEL.md` — Proyección de costos AI y cloud
