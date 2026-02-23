# Stack Técnico — Career Companion Platform

> **Estado:** En definición. Este documento evoluciona con cada sesión de planificación.
> **Última actualización:** 2026-02-22

---

## Visión del Producto

Transformar el CV Optimization Framework (sistema de archivos markdown + agente LLM) en una plataforma web de Career Companion. El usuario interactúa con un agente de IA que lo ayuda a optimizar su perfil profesional, evaluar ofertas laborales, y gestionar su búsqueda de empleo — con integridad, determinismo en scoring, y adaptabilidad a múltiples plataformas de CV.

**Modelo de negocio:** SaaS con suscripción mensual (precio objetivo ~$15 USD/mes).

---

## Stack Definido

### Frontend

| Tecnología | Versión | Propósito | Justificación |
|---|---|---|---|
| **Next.js** | 15+ (App Router) | Framework React con SSR/SSG | SEO para landing, Server Components para performance, deploy nativo en Vercel |
| **TypeScript** | 5.x | Tipado estático | Type safety end-to-end, mejor DX con Claude Code |
| **shadcn/ui** | latest | Componente library | Componentes copiables (no dependencia), theming con Tailwind, generación rápida con v0.dev |
| **Tailwind CSS** | 4.x | Utilidades CSS | Estándar con shadcn, productivo para prototipar |
| **Vercel AI SDK** | latest | Integración con agente | Streaming de respuestas, hooks para chat UI, soporte nativo para Azure OpenAI |
| **Zod** | 3.x | Validación de datos | Validación en runtime + inferencia de tipos TS. Compartir schemas entre frontend y API |
| **React Hook Form** | latest | Formularios | Integración nativa con Zod via resolvers |
| **TanStack Query** | 5.x | Data fetching + cache | Cache inteligente, optimistic updates para el Kanban, deduplicación de requests |

**Bundler:** Next.js usa Turbopack (sucesor de Webpack) por defecto en dev, no necesitamos Vite. En producción usa el bundler de Next.js.

> **Decisión: ¿Por qué no Vite?** Vite es excelente para SPAs con React puro, pero Next.js ya trae su propio bundler optimizado. Usar Vite + Next.js no tiene sentido — son alternativas, no complementos. Si en algún momento necesitamos una SPA pura sin SSR (ej: widget embebible), ahí sí Vite sería la opción.

### Backend

| Tecnología | Versión | Propósito | Justificación |
|---|---|---|---|
| **Python** | 3.12+ | Lenguaje backend | Bootcamp Django, ecosistema AI/ML, Semantic Kernel nativo |
| **Django** | 5.x | Framework web | ORM maduro, admin panel gratis, auth robusta, enorme ecosistema |
| **Django Ninja** | 1.x | API REST | Tipado con Pydantic, OpenAPI automático, más rápido que DRF para APIs modernas |
| **Pydantic** | 2.x | Validación de datos (backend) | Schemas tipados para inputs/outputs del agente, serialización JSON |
| **Celery** | 5.x | Tareas asíncronas | Procesamiento de scoring en background, generación de CV, jobs largos del agente |
| **Redis** | 7.x | Broker para Celery + cache | Rate limiting, cache de scoring, session storage |

> **Decisión: ¿Django Ninja vs DRF (Django REST Framework)?** Django Ninja usa Pydantic (mismo modelo mental que Zod en frontend), genera OpenAPI automáticamente, y es más conciso. DRF es más maduro pero más verbose. Para un proyecto nuevo con Claude Code, Ninja es mejor DX.

> **Decisión: ¿Por qué no NestJS + monorepo full TypeScript?**
> - Django es requerimiento de aprendizaje (bootcamp)
> - Python es el lenguaje nativo de Semantic Kernel y el ecosistema AI
> - Un monorepo TS tendría ventaja en shared types, pero con OpenAPI codegen entre Django Ninja y el frontend, obtenemos type safety cross-stack sin monorepo
> - El costo real de Python + TS es la duplicación de validación (Pydantic + Zod), que se mitiga con codegen desde OpenAPI

### Agente de IA

| Tecnología | Propósito | Justificación |
|---|---|---|
| **Semantic Kernel (Python)** | Orquestación del agente | Plugins tipados, planificador, memoria, nativo Azure. Certificación AI-102 |
| **Azure OpenAI** | Hosting de modelos LLM | Enterprise-grade, content safety integrado, región Latam disponible |
| **Azure Content Safety** | Protección contra prompt injection | Detecta jailbreak attempts, contenido off-topic, abuso del modelo |

**Modelos y routing (ver sección Optimización de Costos):**

| Tarea | Modelo | Razón |
|---|---|---|
| Clasificación de intent (¿es relevante al career companion?) | GPT-4o-mini | Rápido, barato, suficiente para clasificación binaria |
| Clasificación de requisitos A-D | GPT-4o | Requiere juicio sobre JDs |
| Generación de contenido (Killer Summary, Impact Bullets) | GPT-4o | Calidad de escritura |
| Cálculo de scores | **Código Python (no LLM)** | Determinista, testeable, sin costo de tokens |
| Validación de Non-Negotiables | **Código Python (no LLM)** | Determinista |
| Conversación de onboarding | GPT-4o | Requiere empatía y curiosidad activa |
| Análisis de cultura/empresa | GPT-4o + web search | Requiere razonamiento + datos frescos |

> **Nota sobre modelos:** Azure OpenAI permite cambiar modelos sin cambiar código. Si mañana GPT-5 es mejor y más barato, se cambia la config. Semantic Kernel abstrae esto.

### Infraestructura y Datos

| Tecnología | Propósito | Justificación |
|---|---|---|
| **Supabase** | Base de datos, auth, storage, vector DB | Todo-en-uno, tier gratis generoso, pgvector nativo |
| **PostgreSQL** (via Supabase) | Base de datos relacional | Profiles, applications, learnings, scoring_history, outputs |
| **pgvector** (via Supabase) | Base de datos vectorial | Embeddings de perfil, patterns, JDs para RAG |
| **Supabase Auth** | Autenticación | OAuth social (GitHub, Google), JWT, Row Level Security |
| **Supabase Storage** | Archivos | PDFs exportados, attachments del usuario |
| **Vercel** | Deploy frontend | Deploy automático desde git, preview deploys, edge functions |
| **Azure App Service** o **Railway** | Deploy backend | Django + Celery. Railway es más simple para empezar, Azure para AI-102 |

> **Decisión pendiente: ¿Azure App Service vs Railway para backend?**
> Railway: setup en 5 minutos, pricing predecible, ideal para MVP.
> Azure: necesario para AI-102 demo, más complejo pero más enterprise.
> **Propuesta:** Railway para MVP, migrar a Azure cuando sea relevante para la certificación.

### Testing

| Herramienta | Capa | Propósito |
|---|---|---|
| **Vitest** | Frontend - Unit | Tests de componentes, hooks, utils. Compatible con Jest API pero más rápido |
| **Playwright** | Frontend - E2E | Tests end-to-end del flujo completo (onboarding, scoring, chat) |
| **Testing Library** | Frontend - Integration | Tests de componentes React con interacción de usuario |
| **pytest** | Backend - Unit/Integration | Tests de scoring engine, plugins SK, API endpoints |
| **pytest-django** | Backend - Django | Tests específicos de modelos, views, middleware |
| **Factory Boy** | Backend - Fixtures | Factories para generar datos de test (perfiles, applications) |
| **Hypothesis** | Backend - Property-based | Tests de propiedades para scoring engine (ej: score siempre entre 0-100) |

> **Tests críticos del scoring engine:**
> - Determinismo: mismo input → mismo output, 100 veces
> - Boundaries: score nunca < 0 ni > 100
> - Monotonicidad: agregar un skill match nunca reduce el score
> - Clasificación: requisitos Tipo A siempre pesan más que Tipo D

### Herramientas de Desarrollo

| Herramienta | Propósito |
|---|---|
| **Biome** | Linter + formatter para TypeScript (reemplaza ESLint + Prettier, más rápido) |
| **Ruff** | Linter + formatter para Python (reemplaza flake8 + black + isort) |
| **Husky + lint-staged** | Pre-commit hooks (lint, format, type-check) |
| **GitHub Actions** | CI/CD (tests, lint, deploy) |
| **openapi-typescript** | Codegen: genera tipos TS desde el OpenAPI schema de Django Ninja |

---

## Estructura del Repositorio

```
career-companion/
├── README.md
├── .github/
│   └── workflows/
│       ├── ci-frontend.yml
│       ├── ci-backend.yml
│       └── deploy.yml
│
├── frontend/                     # Next.js app
│   ├── package.json
│   ├── next.config.ts
│   ├── biome.json
│   ├── vitest.config.ts
│   ├── playwright.config.ts
│   ├── src/
│   │   ├── app/                  # App Router pages
│   │   ├── components/           # shadcn + custom components
│   │   ├── lib/                  # Utils, API client, Zod schemas
│   │   ├── hooks/                # Custom React hooks
│   │   └── types/                # Generated types from OpenAPI
│   └── tests/
│       ├── unit/
│       └── e2e/
│
├── backend/                      # Django project
│   ├── pyproject.toml            # Deps con uv o poetry
│   ├── manage.py
│   ├── config/                   # Django settings, urls, wsgi
│   ├── apps/
│   │   ├── profiles/             # Perfil del usuario (perfil_base migrado)
│   │   ├── applications/         # Tracker de postulaciones
│   │   ├── scoring/              # Scoring engine (determinista)
│   │   ├── agent/                # Semantic Kernel integration
│   │   └── adapters/             # Platform adapters (CVMaker, LinkedIn, etc.)
│   ├── plugins/                  # Semantic Kernel plugins
│   │   ├── score_calculator.py
│   │   ├── profile_analyzer.py
│   │   ├── content_generator.py
│   │   └── validator.py
│   └── tests/
│       ├── test_scoring.py
│       ├── test_plugins.py
│       └── factories.py
│
├── docs/                         # Documentación del proyecto
│   ├── architecture/
│   │   ├── STACK.md              # Este archivo
│   │   ├── DECISIONS.md          # ADRs (Architecture Decision Records)
│   │   ├── SECURITY.md           # Modelo de seguridad y anti-abuse
│   │   ├── COST_MODEL.md         # Proyección de costos AI/cloud
│   │   └── ADAPTERS.md           # Sistema de adaptadores de plataforma
│   └── api/
│       └── openapi.yaml          # Schema generado por Django Ninja
│
├── framework/                    # Framework original (markdown) — referencia
│   └── ...                       # Se mantiene como documentación y fallback
│
└── supabase/
    ├── migrations/               # SQL migrations
    └── seed.sql                  # Datos iniciales
```

> **Decisión: ¿Monorepo o repos separados?**
> Monorepo en un solo repositorio Git, pero NO con herramientas de monorepo pesadas (Nx, Turborepo). Razones:
> - Frontend y backend son stacks distintos (TS + Python) — Turborepo no aporta mucho
> - Un solo repo simplifica: un solo git log, un solo PR para features cross-stack
> - GitHub Actions con path filters para CI separado (`frontend/**` y `backend/**`)
> - Si crece demasiado, se puede separar después — es más fácil split que merge

---

## Patrones de Diseño

### Backend (Django + Semantic Kernel)

| Patrón | Dónde aplica | Beneficio |
|---|---|---|
| **Service Layer** | `apps/scoring/services.py`, `apps/agent/services.py` | Lógica de negocio separada de views/serializers. Las views solo orquestan. |
| **Repository Pattern** | Acceso a Supabase desde Django | Abstrae queries de DB, facilita testing con mocks |
| **Strategy Pattern** | Scoring engine, model routing | Diferentes estrategias de cálculo o modelo según contexto |
| **Plugin Architecture** | Semantic Kernel plugins | Cada capacidad del agente es un plugin independiente y testeable |
| **Pipeline Pattern** | Las 7 fases del framework | Cada fase es un step con input/output definido, ejecutable en secuencia |
| **Proxy Pattern** | Backend como proxy del agente | El frontend nunca habla directo con Azure OpenAI — siempre pasa por Django |
| **Observer Pattern** | Scoring history, drift detection | Cuando se genera un score, se registra automáticamente para comparar |

### Frontend (Next.js)

| Patrón | Dónde aplica | Beneficio |
|---|---|---|
| **Server Components** (default) | Páginas, layouts, data fetching | Menos JS al cliente, mejor performance |
| **Client Components** (opt-in) | Chat, formularios interactivos, Kanban | Solo donde se necesita interactividad |
| **Compound Components** | Editor de perfil, scoring display | Componentes complejos descompuestos en partes reutilizables |
| **Custom Hooks** | `useChat`, `useScoring`, `useProfile` | Lógica reutilizable separada de UI |
| **Optimistic Updates** | Kanban de applications, edición de perfil | UX fluida — actualiza UI antes de confirmar servidor |

### Cross-Stack

| Patrón | Propósito |
|---|---|
| **Contract-First API** | Django Ninja genera OpenAPI → codegen genera tipos TS → frontend y backend siempre están sincronizados |
| **Feature Flags** | Activar/desactivar features por usuario (ej: voice-to-text, adaptadores beta) sin deploy |
| **Event Sourcing (ligero)** | Scoring history guarda cada cálculo, no solo el último. Permite auditoría y drift detection |

---

## Decisiones Arquitectónicas Pendientes

| # | Decisión | Opciones | Estado |
|---|---|---|---|
| ADR-001 | Deploy backend: Railway vs Azure App Service | Railway (simple) vs Azure (AI-102) | Pendiente |
| ADR-002 | ¿Cómo sincronizar tipos Pydantic ↔ Zod? | OpenAPI codegen vs manual vs shared schema | Pendiente |
| ADR-003 | ¿Celery worker o Azure Functions para jobs? | Celery (más control) vs Functions (serverless) | Pendiente |
| ADR-004 | ¿Cómo manejar el estado del chat del agente? | Supabase realtime vs polling vs Vercel AI SDK streams | Pendiente |
| ADR-005 | ¿Multi-tenant desde el inicio o single-tenant? | RLS en Supabase (multi) vs instancias separadas | Pendiente |

---

## Documentos Relacionados

- `DECISIONS.md` — Architecture Decision Records detallados (por crear)
- `SECURITY.md` — Modelo de seguridad, anti-abuse, prompt injection (por crear)
- `COST_MODEL.md` — Proyección de costos AI y cloud (por crear)
- `ADAPTERS.md` — Sistema de adaptadores de plataforma CV (por crear)
