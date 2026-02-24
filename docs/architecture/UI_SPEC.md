# UI Spec — Career Companion Platform

> **Estado:** Definido. Listo para generar con v0.dev.
> **Última actualización:** 2026-02-24

---

## Restricciones de Diseño (para v0.dev)

| Restricción | Valor |
|---|---|
| **Framework** | Next.js 15 App Router |
| **Componentes** | shadcn/ui exclusivamente |
| **Estilos** | Tailwind CSS 4 |
| **Idioma de UI** | Español |
| **Dark mode** | Sí — shadcn lo soporta out of the box. Toggle en header. |
| **Responsive** | Desktop-first (el uso principal es escribir CVs), pero funcional en mobile |
| **Tipografía** | Inter (default de shadcn) |
| **Iconos** | Lucide React (incluido con shadcn) |
| **Paleta** | Neutral base + azul/indigo para acciones primarias. Profesional, no corporativo. |
| **Tono visual** | Limpio, espaciado generoso, sin ruido. El contenido es el protagonista. |
| **Emojis** | No en la UI. Solo en contenido generado si el framework lo produce. |

---

## Mapa de Páginas

```
/                          Landing (pública, SSG)
/login                     Login con Google/GitHub OAuth
/onboarding                Primera vez: setup de perfil multi-formato
/dashboard                 Home del usuario autenticado
/profile                   Editor de perfil completo
/chat                      Chat con el agente (nueva postulación)
/applications              Historial de postulaciones (tabla)
/applications/[id]         Vista de resultados de una postulación
```

---

## Páginas y Componentes

### 1. Landing (`/`)

**Propósito:** Convertir visitantes en signups. SSG, rápida, SEO optimizada.

**Layout:**
```
┌──────────────────────────────────────────────────┐
│  Logo                              [Iniciar sesión] │
├──────────────────────────────────────────────────┤
│                                                    │
│  "Tu abogado defensor para                         │
│   la búsqueda de empleo"                           │
│                                                    │
│  Párrafo: propuesta de valor en 2 líneas           │
│                                                    │
│  [Comenzar gratis]                                 │
│                                                    │
├──────────────────────────────────────────────────┤
│  Cómo funciona (3 pasos)                           │
│  ┌────────┐  ┌────────┐  ┌────────┐               │
│  │ 1.Perfil│  │2.Pega JD│  │3.CV     │              │
│  │ Crea tu │  │ El agente│  │optimizado│             │
│  │ perfil  │  │ analiza  │  │+ scores │              │
│  └────────┘  └────────┘  └────────┘               │
├──────────────────────────────────────────────────┤
│  Screenshot/demo del flujo                         │
├──────────────────────────────────────────────────┤
│  Diferenciadores (3 cards)                         │
│  - Scoring determinista                            │
│  - Perfil que mejora con cada uso                  │
│  - Abogado defensor, no auditor                    │
├──────────────────────────────────────────────────┤
│  CTA final + Footer                                │
└──────────────────────────────────────────────────┘
```

**Componentes shadcn:** Button, Card, Badge
**SEO:** meta title/description para "optimizar CV con IA", "career companion IA español"

---

### 2. Login (`/login`)

**Propósito:** Autenticación con OAuth. Minimalista.

```
┌──────────────────────────────────────┐
│                                        │
│        Career Companion                │
│                                        │
│    [🔵 Continuar con Google]           │
│    [⚫ Continuar con GitHub]           │
│                                        │
│    Texto legal pequeño                 │
│                                        │
└──────────────────────────────────────┘
```

**Componentes shadcn:** Button, Card
**Lógica:** Post-login → si no tiene profile completado → `/onboarding`. Si ya tiene → `/dashboard`.

---

### 3. Onboarding (`/onboarding`)

**Propósito:** Poblar el perfil del usuario en su primera sesión. Multi-formato.

**Paso 1: Selector de modo**
```
┌──────────────────────────────────────────────────┐
│  ¿Cómo prefieres empezar?                         │
│                                                    │
│  ┌─────────────────┐  ┌─────────────────┐         │
│  │ 📄 Tengo un CV   │  │ 💬 Prefiero que  │        │
│  │ o texto sobre mí │  │ me preguntes     │        │
│  └─────────────────┘  └─────────────────┘         │
│                                                    │
│  ┌─────────────────────────────────────┐           │
│  │ 🔄 Las dos: pego algo y después     │           │
│  │    me preguntas lo que falta         │           │
│  └─────────────────────────────────────┘           │
└──────────────────────────────────────────────────┘
```

**Paso 2: Interfaz según modo**
- **Modo "Tengo CV":** Textarea grande + botón "Analizar". El agente extrae datos y muestra lo que entendió.
- **Modo "Pregúntame":** Chat guiado fase por fase (identidad → trayectoria → stack → voz → prioridades).
- **Modo "Ambos":** Textarea → análisis → chat de follow-up.

**Paso 3: Resumen del perfil**
```
┌──────────────────────────────────────────────────┐
│  Tu perfil está listo                              │
│                                                    │
│  Completitud: ████████░░ 78%                       │
│                                                    │
│  ✅ Identidad profesional                          │
│  ✅ 3 experiencias laborales                       │
│  ✅ 12 skills documentados                         │
│  ⚠️ Sin fortalezas profesionales (MVP será 0)     │
│  ⚠️ Sin preferencias laborales                    │
│                                                    │
│  [Completar lo que falta]  [Ir al dashboard]       │
└──────────────────────────────────────────────────┘
```

**Componentes shadcn:** Card, Textarea, Button, Progress, Badge, Tabs
**Nota:** El chat de onboarding usa el mismo componente de chat que `/chat`, pero con el modo `onboarding` del agente.

---

### 4. Dashboard (`/dashboard`)

**Propósito:** Home del usuario. Vista rápida del estado + acceso a acciones principales.

```
┌──────────────────────────────────────────────────────────┐
│  Header: Logo  |  Dashboard  Perfil  Historial  |  Avatar │
├──────────────────────────────────────────────────────────┤
│                                                            │
│  ┌─────────────────────┐  ┌──────────────────────────┐    │
│  │ Tu Perfil            │  │ + Nueva Postulación       │   │
│  │ ████████░░ 78%       │  │                            │   │
│  │ [Editar perfil]      │  │ Pega una oferta laboral    │   │
│  └─────────────────────┘  │ para empezar               │   │
│                            │ [Ir al chat]               │   │
│                            └──────────────────────────┘   │
│                                                            │
│  Postulaciones Recientes                                   │
│  ┌────────────────────────────────────────────────────┐   │
│  │ Empresa    │ Rol          │ Score │ SAS │ Estado    │   │
│  │ Fintoc     │ Backend Dev  │  88%  │ 87% │ Postulado │   │
│  │ Mercado L. │ Full Stack   │  72%  │ 65% │ Screening │   │
│  │ Cornershop │ Sr. Engineer │  45%  │ 82% │ CV listo  │   │
│  └────────────────────────────────────────────────────┘   │
│  [Ver todo el historial]                                   │
│                                                            │
│  ┌───────────────────────────────────────┐                │
│  │ Estadísticas                           │               │
│  │ 8 postulaciones | Score prom: 71%      │               │
│  │ SAS prom: 74%   | 3 en proceso         │               │
│  └───────────────────────────────────────┘                │
└──────────────────────────────────────────────────────────┘
```

**Componentes shadcn:** Card, Progress, Table, Button, Badge (para status), Avatar, DropdownMenu
**Datos:** `DashboardData` (ver interfaces TS)

---

### 5. Editor de Perfil (`/profile`)

**Propósito:** CRUD completo del perfil. Cada sección se edita y guarda independientemente.

```
┌──────────────────────────────────────────────────────────┐
│  Mi Perfil                                    78% completo │
├──────────────────────────────────────────────────────────┤
│                                                            │
│  [Identidad] [Skills] [Experiencia] [Proyectos]           │
│  [Fortalezas] [Educación] [Preferencias] [Non-Negotiables]│
│                                                            │
│  ── Tab activo: Skills ──────────────────────────────      │
│                                                            │
│  Power Stack                          [+ Agregar Skill]    │
│                                                            │
│  Lenguajes                                                 │
│  ┌──────────────────────────────────────┐                 │
│  │ Python      ████░  4/5    [Editar][X]│                 │
│  │ JavaScript  ████░  4/5    [Editar][X]│                 │
│  │ SQL         ███░░  3/5    [Editar][X]│                 │
│  └──────────────────────────────────────┘                 │
│                                                            │
│  En Aprendizaje                                            │
│  ┌──────────────────────────────────────┐                 │
│  │ Rust         (en aprendizaje)        │                 │
│  │ Kubernetes   (en aprendizaje)        │                 │
│  └──────────────────────────────────────┘                 │
│                                                            │
│  Frontend                                                  │
│  ┌──────────────────────────────────────┐                 │
│  │ React       ████░  4/5    [Editar][X]│                 │
│  │ ...                                   │                 │
│  └──────────────────────────────────────┘                 │
│                                                            │
└──────────────────────────────────────────────────────────┘
```

**Tabs y sus formularios:**

| Tab | Campos principales | Componentes |
|---|---|---|
| Identidad | Pilar, identidad profesional, motivación, diferenciador, liderazgo, killer summary | Textarea, Input |
| Skills | Tabla por categoría con nivel 1-5, toggle learning | Table, Select, Slider, Switch |
| Experiencia | Lista de experiencias con achievements editables | Card, Input, Textarea, DatePicker, drag-to-reorder |
| Proyectos | Lista con tier selector, stack, link | Card, Select (tier 1/2/3), Input, Textarea |
| Fortalezas | 3 categorías (liderazgo, negocio, aprendizaje) con ejemplos | Textarea por categoría |
| Educación | Lista de títulos/certs/cursos | Card, Input, Select, Switch (en progreso) |
| Preferencias | Modalidad, ubicación, salario, contrato, disponibilidad | Select, Input, Switch, RadioGroup |
| Non-Negotiables | 4 categorías de restricciones (experiencia, métricas, posicionamiento, tono) | Textarea por categoría |

**Componentes shadcn:** Tabs, Card, Input, Textarea, Select, Switch, Slider, Button, DatePicker, Badge, Dialog (para edición modal), Toast (para confirmación de guardado)

---

### 6. Chat (`/chat`)

**Propósito:** Interacción principal con el agente. Screening, CV generation, Q&A.

```
┌──────────────────────────────────────────────────────────┐
│  Nueva Postulación                                        │
├──────────────────────────────────────────────────────────┤
│                                                            │
│  ┌────────────────────────────────────────────────────┐   │
│  │                                                      │  │
│  │  [Agente] Pega la oferta laboral y te hago un        │  │
│  │  screening rápido. Si te interesa, generamos el      │  │
│  │  CV optimizado.                                      │  │
│  │                                                      │  │
│  │  [Usuario] *pega JD largo*                           │  │
│  │                                                      │  │
│  │  [Agente] [Fase 1/7] Analizando empresa...           │  │
│  │  ████████████░░░░░ streaming...                      │  │
│  │                                                      │  │
│  │  ── Resultados del Screening ──                      │  │
│  │  ┌─────────────────────────────┐                     │  │
│  │  │ Confidence  ████████░░  88% │                     │  │
│  │  │ SAS         ███████░░░  72% │                     │  │
│  │  │ Veredicto:  🟢 SEGUIR       │                     │  │
│  │  └─────────────────────────────┘                     │  │
│  │                                                      │  │
│  │  [Agente] ¿Genero el CV optimizado?                  │  │
│  │                                                      │  │
│  │  ── Anti-Impostor (si aplica) ──                     │  │
│  │  ┌─────────────────────────────────────────┐         │  │
│  │  │ ⚠️ Clasifiqué 2 requisitos como          │        │  │
│  │  │ "no cumple" pero quiero verificar:        │        │  │
│  │  │                                           │        │  │
│  │  │ - "Experiencia con microservicios"        │        │  │
│  │  │   ¿Has trabajado con servicios separados  │        │  │
│  │  │   que se comunican entre sí?              │        │  │
│  │  │                                           │        │  │
│  │  │ - "CI/CD pipelines"                       │        │  │
│  │  │   ¿Has configurado GitHub Actions o       │        │  │
│  │  │   algo similar?                           │        │  │
│  │  └─────────────────────────────────────────┘         │  │
│  │                                                      │  │
│  └────────────────────────────────────────────────────┘   │
│                                                            │
│  ┌────────────────────────────────────────────────────┐   │
│  │ Escribe tu mensaje...                    [Enviar]   │  │
│  └────────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────┘
```

**Componentes especiales dentro del chat:**
- `ScoreCard` — muestra Confidence + SAS inline con barras de progreso
- `AntiImpostorPrompt` — card destacada con preguntas de verificación
- `PhaseProgress` — indicador "[Fase 3/7] Generando contenido..."
- `PlatformConfirmation` — "GetOnBoard pide estos campos: [lista]. ¿Son los mismos?"

**Componentes shadcn:** ScrollArea, Input, Button, Card, Progress, Alert, Badge, Skeleton (loading)

---

### 7. Vista de Resultados (`/applications/[id]`)

**Propósito:** Output estructurado de una postulación. Para copiar secciones al CV.

```
┌──────────────────────────────────────────────────────────┐
│  Fintoc — Backend Developer            Estado: [Postulado ▼] │
├──────────────────────────────────────────────────────────┤
│                                                            │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐     │
│  │Confidence│ │   SAS    │ │   MVP    │ │ Veredicto│     │
│  │   88%    │ │   72%    │ │   +11    │ │ 🟢 SEGUIR│     │
│  │ ████████ │ │ ███████  │ │          │ │          │     │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘     │
│                                                            │
│  [Gap Analysis] [CV Content] [Plataforma] [Entrevista]     │
│                                                            │
│  ── Tab: CV Content ─────────────────────────────────      │
│                                                            │
│  Killer Summary                              [Copiar 📋]  │
│  ┌────────────────────────────────────────────────────┐   │
│  │ "Full Stack Developer con background en negocios    │   │
│  │  internacionales. Combino visión técnica con..."    │   │
│  └────────────────────────────────────────────────────┘   │
│                                                            │
│  Impact Bullets — Empresa X                  [Copiar 📋]  │
│  ┌────────────────────────────────────────────────────┐   │
│  │ • Diseñé e implementé pipeline de automatización... │   │
│  │ • Lideré migración de sistema legacy a...           │   │
│  └────────────────────────────────────────────────────┘   │
│                                                            │
│  Skills Matrix                               [Copiar 📋]  │
│  ┌────────────────────────────────────────────────────┐   │
│  │ Skill      │ Nivel │ Justificación                  │   │
│  │ Node.js    │ 4/5   │ Producción diaria, 3+ años     │   │
│  │ PostgreSQL │ 4/5   │ Diseño de schema, queries...   │   │
│  └────────────────────────────────────────────────────┘   │
│                                                            │
│  [Copiar Todo]                   [Volver al chat]         │
└──────────────────────────────────────────────────────────┘
```

**Tabs de resultados:**

| Tab | Contenido |
|---|---|
| Gap Analysis | Tabla: requisito, tipo A-D, match, estrategia de mitigación |
| CV Content | Killer Summary + Impact Bullets + Skills Matrix (cada uno con copy button) |
| Plataforma | Respuestas a preguntas del portal (si se generaron) |
| Entrevista | Interview Intel: preguntas técnicas + culturales probables |

**Al fondo:** Brand Coherence Audit como checklist colapsable.

**Componentes shadcn:** Card, Tabs, Table, Button, Badge, Collapsible, Tooltip (para justificaciones), Toast (al copiar)

---

### 8. Historial (`/applications`)

**Propósito:** Tabla de todas las postulaciones. Filtrable y ordenable.

```
┌──────────────────────────────────────────────────────────┐
│  Historial de Postulaciones              [+ Nueva]        │
├──────────────────────────────────────────────────────────┤
│                                                            │
│  Filtros: [Todos ▼]  [Ordenar por: Fecha ▼]              │
│                                                            │
│  ┌──────────────────────────────────────────────────┐    │
│  │ Empresa     │ Rol          │ Score│ SAS │ Estado  │ Fecha     │
│  │─────────────│──────────────│──────│─────│─────────│───────────│
│  │ Fintoc      │ Backend Dev  │  88% │ 72% │ 🟢 Post │ 24 feb    │
│  │ Mercado L.  │ Full Stack   │  72% │ 65% │ 🟡 Scr  │ 22 feb    │
│  │ Cornershop  │ Sr. Engineer │  45% │ 82% │ 🔵 CV   │ 20 feb    │
│  │ Buk         │ React Dev    │  91% │ 78% │ 🟢 Post │ 18 feb    │
│  │ NotCo       │ Data Eng     │  38% │ 45% │ 🔴 Desc │ 15 feb    │
│  └──────────────────────────────────────────────────┘    │
│                                                            │
│  Click en fila → /applications/[id]                        │
└──────────────────────────────────────────────────────────┘
```

**Columnas:** empresa, rol, confidence score, SAS, status (badge con color), fecha
**Filtros:** por status (dropdown)
**Orden:** por fecha (default desc), por score, por SAS
**Colores de status:**

| Status | Color | Label corto |
|---|---|---|
| screening | gris | Scr |
| cv_generated | azul | CV |
| applied | verde | Post |
| in_process | amarillo | Proc |
| rejected | rojo | Rech |
| offer | verde fuerte | Oferta |
| declined | gris | Decl |

**Componentes shadcn:** Table, Badge, Select (filtros), Button, DropdownMenu

---

## Layout Global

```
┌──────────────────────────────────────────────────────────┐
│  Logo    │  Dashboard  Perfil  Historial  │  🌙/☀️  Avatar │
├──────────────────────────────────────────────────────────┤
│                                                            │
│                    Contenido de página                     │
│                                                            │
└──────────────────────────────────────────────────────────┘
```

- **Header:** Logo + nav links + dark mode toggle + avatar con dropdown (Settings, Logout)
- **No sidebar:** Layout simple con nav en header. No necesita sidebar para 4 páginas.
- **Max width:** `max-w-6xl mx-auto` para contenido. Chat puede ser más ancho.
- **Padding:** `p-6` consistente en todas las páginas.

**Componentes shadcn:** NavigationMenu, Avatar, DropdownMenu, ThemeToggle

---

## Interfaces TypeScript

Estas interfaces son el contrato entre frontend y backend. Se generan automáticamente desde Pydantic via OpenAPI codegen, pero v0.dev las necesita para generar componentes con los tipos correctos.

```typescript
// ============================================================
// PROFILE TYPES (maps to DATABASE.md profile tables)
// ============================================================

interface Profile {
  id: string
  primary_pillar: string | null
  professional_identity: string | null
  motivation_ethos: string | null
  differentiator: string | null
  leadership_soft_power: string | null
  voice_tone: string | null
  voice_structure: string | null
  voice_balance: string | null
  voice_personality: string | null
  voice_adaptation: string | null
  value_hierarchy: ValueHierarchyItem[] | null
  non_negotiables: NonNegotiables | null
  killer_summary: string | null
  completeness_pct: number
  created_at: string
  updated_at: string
}

interface ValueHierarchyItem {
  rank: number
  value: string
  description: string
}

interface NonNegotiables {
  experience: string[]
  metrics: string[]
  positioning: string[]
  tone: string[]
}

interface WorkPreferences {
  id: string
  modality: 'remote' | 'hybrid' | 'onsite' | 'flexible' | null
  modality_is_dealbreaker: boolean
  location: string | null
  willing_to_relocate: boolean
  acceptable_timezones: string[]
  salary_min: number | null
  salary_ideal: number | null
  salary_currency: string
  contract_types: string[]
  availability: 'immediate' | 'two_weeks' | 'employed' | null
  availability_weeks: number | null
}

interface Skill {
  id: string
  category: 'languages' | 'frontend' | 'backend_db' | 'cloud_devops' | 'ai_agents' | 'other'
  name: string
  level: 1 | 2 | 3 | 4 | 5
  is_learning: boolean
  notes: string | null
}

interface WorkExperience {
  id: string
  title: string
  company: string
  start_date: string       // ISO date
  end_date: string | null
  is_current: boolean
  context: string | null
  achievements: Achievement[]
  display_order: number
}

interface Achievement {
  description: string
  has_metric: boolean
  metric_value: string | null
  action: string | null
  impact_area: 'sistema' | 'equipo' | 'negocio' | 'proceso' | null
}

interface Project {
  id: string
  name: string
  tier: 1 | 2 | 3
  description: string | null
  problem_solved: string | null
  users_impact: string | null
  stack_summary: string | null
  demonstrates: string | null
  technical_achievements: { area: string; description: string }[]
  link: string | null
  start_date: string | null
  end_date: string | null
  is_current: boolean
}

interface ProfessionalStrength {
  id: string
  category: 'leadership' | 'industry_business' | 'learning_speed'
  description: string
}

interface Education {
  id: string
  type: 'degree' | 'certification' | 'course'
  institution: string
  title: string
  summary: string | null
  year: number | null
  is_in_progress: boolean
  display_order: number
}

// ============================================================
// APPLICATION & SCORING TYPES
// ============================================================

interface Application {
  id: string
  company: string
  role: string
  jd_text: string | null
  platform: string | null
  job_url: string | null
  status: ApplicationStatus
  applied_at: string | null
  created_at: string
  updated_at: string
}

type ApplicationStatus =
  | 'screening'
  | 'cv_generated'
  | 'applied'
  | 'in_process'
  | 'rejected'
  | 'offer'
  | 'declined'

interface ClassifiedRequirement {
  requirement: string
  type: 'A' | 'B' | 'C' | 'D'
  type_justification: string
  match: 'cumple' | 'transferible' | 'parcial' | 'no_cumple'
  match_justification: string
  score: number
}

interface ConfidenceScoreResult {
  obligatorios_score: number
  deseables_score: number
  base: number
  mvp: {
    leadership: number
    leadership_justification: string
    business: number
    business_justification: string
    learning: number
    learning_justification: string
    total: number
  }
  final: number
  requirements: ClassifiedRequirement[]
}

interface SASDimensionScore {
  score: number
  justification: string
  confidence: 'high' | 'low'  // 'low' when no web search data (Sprint 1)
}

interface SASResult {
  career_goals: SASDimensionScore
  intrinsic_motivations: SASDimensionScore
  values_culture: SASDimensionScore
  tech_growth: SASDimensionScore
  autonomy_role: SASDimensionScore
  total: number
}

interface GapItem {
  requirement: string
  gap_type: string
  mitigation_strategy: string
  is_mitigable: boolean
}

interface ScoringResult {
  confidence: ConfidenceScoreResult
  sas: SASResult
  gap_analysis: GapItem[]
  decision_matrix: 'max_priority' | 'consider' | 'strategic_only' | 'not_recommended'
}

// ============================================================
// OUTPUT TYPES (generated CV content)
// ============================================================

interface Output {
  id: string
  application_id: string
  scoring_id: string | null
  version: number
  killer_summary: string | null
  gap_analysis: GapItem[]
  impact_bullets: { experience_id: string; experience_title: string; bullets: string[] }[]
  skills_matrix: { skill: string; score: number; justification: string }[]
  application_qa: { question: string; answer: string; platform: string | null }[]
  interview_intel: { technical_questions: string[]; culture_questions: string[] }
  brand_coherence_audit: BrandAuditResult
  full_content: string | null
  created_at: string
}

interface BrandAuditResult {
  tone_match: boolean
  pillar_reinforced: boolean
  ethos_visible: boolean
  tech_business_balance: boolean
  red_lines_respected: boolean
  narrative_coherent: boolean
  notes: string[]
  overall_passed: boolean
}

// ============================================================
// DASHBOARD & AGGREGATION TYPES
// ============================================================

interface DashboardData {
  profile_completeness: number
  recent_applications: ApplicationSummary[]
  stats: {
    total_applications: number
    average_confidence: number
    average_sas: number
    in_process_count: number
  }
}

interface ApplicationSummary {
  id: string
  company: string
  role: string
  confidence_final: number
  sas_total: number
  status: ApplicationStatus
  created_at: string
}

// ============================================================
// CHAT & STREAMING TYPES
// ============================================================

type AgentStreamSegmentType =
  | 'phase_progress'     // "[Fase 2/7] Clasificando requisitos..."
  | 'culture_radar'      // Resultado del análisis de empresa
  | 'scores'             // ScoreCard inline
  | 'gap_analysis'       // Lista de gaps
  | 'anti_impostor'      // Pausa interactiva con preguntas
  | 'killer_summary'     // Texto del summary
  | 'impact_bullets'     // Bullets por experiencia
  | 'skills_matrix'      // Tabla de skills
  | 'platform_confirm'   // "GetOnBoard pide estos campos..."
  | 'verification'       // Resultado de verificación
  | 'brand_audit'        // Checklist de coherencia
  | 'interview_intel'    // Preguntas probables
  | 'error'              // Mensaje de error amigable

interface AgentStreamSegment {
  type: AgentStreamSegmentType
  content: string          // markdown o JSON string según el tipo
  phase: number            // 1-7
  metadata?: Record<string, unknown>
}
```

---

## Guía para v0.dev

### Orden sugerido de generación

1. **Layout global** (header + nav + dark mode toggle)
2. **Landing page** — la más independiente, no necesita tipos del backend
3. **Login page** — simple, 2 botones
4. **Dashboard** — usa `DashboardData`, `ApplicationSummary`
5. **Profile editor** — la más compleja en formularios, usa todos los types de perfil
6. **Chat UI** — usa `AgentStreamSegment` para renderizar segmentos
7. **Results view** — usa `Output`, `ScoringResult`
8. **Application history** — usa `ApplicationSummary`

### Prompt template para v0.dev

```
Genera un componente React con shadcn/ui y Tailwind CSS.

Framework: Next.js 15 App Router
Idioma de UI: Español
Soporte dark mode: Sí (usar clases de shadcn)
Tipografía: Inter

[Descripción de la página/componente]

Interfaces TypeScript:
[Pegar interfaces relevantes]

Restricciones:
- Solo usar componentes de shadcn/ui
- No emojis en labels/buttons
- Responsive: funcional en mobile, optimizado para desktop
- Usar Lucide React para iconos
```

---

## Documentos Relacionados

- `DATABASE.md` — Schema que define las interfaces TS
- `AGENT_PIPELINE.md` — AgentStreamSegment types y flujo del pipeline
- `MVP.md` — Features y flujo del usuario
- `SPRINTS.md` — En qué milestone se construye cada página
