# Modelo de Datos — Career Companion Platform (Supabase)

> **Estado:** Definido. Listo para implementar.
> **Última actualización:** 2026-02-24

---

## Principios de Diseño

1. **RLS desde el origen** — Toda tabla con datos de usuario tiene `user_id UUID REFERENCES auth.users(id)` y política `USING (user_id = auth.uid())`. No es opcional.
2. **Pydantic como SSOT** — Este documento es la referencia humana. El código real vive en los modelos Pydantic del backend (apps/profiles/models.py, etc.). Si hay conflicto, el código gana.
3. **JSONB para datos variables** — Bullets de experiencia, clasificación de requisitos, resultados de scoring: JSONB con tipos validados en Python. No tablas de 30 columnas para datos que varían por instancia.
4. **Normalización selectiva** — Skills y experiencias son filas propias (se consultan individualmente). Achievements dentro de una experiencia son JSONB (siempre se consultan en bloque con su experiencia).
5. **pgvector diferido** — Vector embeddings para RAG no están en el MVP. El perfil cabe en el context window. Agregar cuando la escala lo requiera.

---

## Diagrama de Relaciones

```
auth.users (Supabase managed)
    │
    ├── profiles (1:1) — identidad profesional, voz, non-negotiables
    ├── work_preferences (1:1) — modalidad, salario, disponibilidad
    ├── work_experiences (1:N) — trayectoria profesional
    ├── skills (1:N) — power stack con niveles
    ├── projects (1:N) — proyectos con tier 1/2/3
    ├── professional_strengths (1:N) — input para MVP scorer
    ├── education (1:N) — títulos y certificaciones
    │
    ├── applications (1:N) — postulaciones
    │       ├── scoring_history (1:N) — resultados de scoring
    │       ├── outputs (1:N) — CVs generados con versiones
    │       └── learnings (1:N) — insights capturados en Fase 7
    │
    └── usage_quotas (1:1) — control de costos por usuario

platform_schemas (global, compartido) — schemas de plataformas de postulación
```

---

## Tablas

### `profiles`

Mapa 1:1 con `auth.users`. Contiene la identidad profesional del usuario (sección PERFIL PROFESIONAL y VOICE PROFILE de `perfil_base.md`).

```sql
CREATE TABLE profiles (
  id            UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,

  -- Sección: PERFIL PROFESIONAL
  primary_pillar          TEXT,         -- "Full Stack Developer", "Data Engineer"
  professional_identity   TEXT,         -- 2-3 líneas de propuesta de valor
  motivation_ethos        TEXT,         -- motivaciones y valores
  differentiator          TEXT,         -- qué lo hace único vs. otros
  leadership_soft_power   TEXT,         -- experiencia liderando (si aplica)

  -- Sección: VOICE PROFILE (resumen — doc completo en brand_voice)
  voice_tone              TEXT,
  voice_structure         TEXT,
  voice_balance           TEXT,
  voice_personality       TEXT,
  voice_adaptation        TEXT,

  -- Sección: VALUE HIERARCHY
  value_hierarchy         JSONB,        -- [{rank: 1, value: "Integridad técnica", description: "..."}]

  -- Sección: NON-NEGOTIABLES
  non_negotiables         JSONB,        -- {experience: [], metrics: [], positioning: [], tone: []}

  -- Sección: KILLER SUMMARY
  killer_summary          TEXT,

  -- Metadata de completitud
  completeness_pct        INTEGER DEFAULT 0 CHECK (completeness_pct BETWEEN 0 AND 100),

  created_at  TIMESTAMPTZ DEFAULT NOW(),
  updated_at  TIMESTAMPTZ DEFAULT NOW()
);

ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;
CREATE POLICY "Users access own profile"
  ON profiles FOR ALL USING (id = auth.uid());
```

---

### `work_preferences`

Preferencias laborales para auto-completar campos de plataformas y alimentar el SAS (Dimensión 5: Autonomía/Rol).

```sql
CREATE TABLE work_preferences (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id     UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE UNIQUE,

  modality                TEXT,         -- 'remote' | 'hybrid' | 'onsite' | 'flexible'
  modality_is_dealbreaker BOOLEAN DEFAULT FALSE,
  location                TEXT,
  willing_to_relocate     BOOLEAN DEFAULT FALSE,
  acceptable_timezones    TEXT[],

  salary_min              INTEGER,
  salary_ideal            INTEGER,
  salary_currency         TEXT DEFAULT 'CLP',

  contract_types          TEXT[],       -- ['indefinido', 'freelance', 'contractor']
  availability            TEXT,         -- 'immediate' | 'two_weeks' | 'employed'
  availability_weeks      INTEGER,

  created_at  TIMESTAMPTZ DEFAULT NOW(),
  updated_at  TIMESTAMPTZ DEFAULT NOW()
);

ALTER TABLE work_preferences ENABLE ROW LEVEL SECURITY;
CREATE POLICY "Users access own preferences"
  ON work_preferences FOR ALL USING (user_id = auth.uid());
```

---

### `work_experiences`

Trayectoria profesional. Los `achievements` son JSONB porque siempre se leen con la experiencia completa.

```sql
CREATE TABLE work_experiences (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id     UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,

  title           TEXT NOT NULL,
  company         TEXT NOT NULL,
  start_date      DATE NOT NULL,
  end_date        DATE,               -- NULL si es el trabajo actual
  is_current      BOOLEAN DEFAULT FALSE,
  context         TEXT,               -- breve descripción del rol

  achievements    JSONB NOT NULL DEFAULT '[]',
  -- Schema de cada achievement:
  -- {
  --   "description": "Lideré migración de sistema legacy...",
  --   "has_metric": true,
  --   "metric_value": "60% reducción de incidencias",
  --   "action": "lideré",
  --   "impact_area": "sistema" | "equipo" | "negocio" | "proceso"
  -- }

  display_order   INTEGER DEFAULT 0,  -- orden de aparición en CV

  created_at  TIMESTAMPTZ DEFAULT NOW(),
  updated_at  TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_work_experiences_user ON work_experiences(user_id);

ALTER TABLE work_experiences ENABLE ROW LEVEL SECURITY;
CREATE POLICY "Users access own experiences"
  ON work_experiences FOR ALL USING (user_id = auth.uid());
```

---

### `skills`

Power Stack. Una fila por skill para poder filtrar/ordenar por categoría y nivel.

```sql
CREATE TABLE skills (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id     UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,

  category    TEXT NOT NULL CHECK (category IN (
                'languages', 'frontend', 'backend_db', 'cloud_devops', 'ai_agents', 'other'
              )),
  name        TEXT NOT NULL,
  level       INTEGER CHECK (level BETWEEN 1 AND 5),
  -- 1=awareness, 2=learning, 3=skilled, 4=experienced, 5=expert
  is_learning BOOLEAN DEFAULT FALSE,  -- está en sección "En Aprendizaje"
  notes       TEXT,                   -- contexto adicional

  created_at  TIMESTAMPTZ DEFAULT NOW(),
  updated_at  TIMESTAMPTZ DEFAULT NOW(),

  UNIQUE(user_id, name)               -- un usuario no puede tener el mismo skill dos veces
);

CREATE INDEX idx_skills_user_category ON skills(user_id, category);

ALTER TABLE skills ENABLE ROW LEVEL SECURITY;
CREATE POLICY "Users access own skills"
  ON skills FOR ALL USING (user_id = auth.uid());
```

---

### `projects`

Proyectos personales con sistema de 3 tiers (Tier 1=Producción, Tier 2=Portafolio, Tier 3=Aprendizaje).

```sql
CREATE TABLE projects (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id     UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,

  name            TEXT NOT NULL,
  tier            INTEGER NOT NULL CHECK (tier BETWEEN 1 AND 3),
  description     TEXT,
  problem_solved  TEXT,
  users_impact    TEXT,               -- quién lo usa, cuántos (NULL para tier 2-3)
  stack_summary   TEXT,               -- "Next.js (4), FastAPI (3), PostgreSQL"
  demonstrates    TEXT,               -- qué capacidad evidencia para un empleador

  technical_achievements JSONB DEFAULT '[]',
  -- [{area: "Auth & pagos", description: "Sistema de auth propio + Stripe"}]

  link            TEXT,               -- GitHub URL o URL de producción
  start_date      DATE,
  end_date        DATE,               -- NULL si ongoing
  is_current      BOOLEAN DEFAULT FALSE,

  created_at  TIMESTAMPTZ DEFAULT NOW(),
  updated_at  TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_projects_user_tier ON projects(user_id, tier);

ALTER TABLE projects ENABLE ROW LEVEL SECURITY;
CREATE POLICY "Users access own projects"
  ON projects FOR ALL USING (user_id = auth.uid());
```

---

### `professional_strengths`

Fuente de datos para el MVP (Modificador de Valor Profesional) del scoring engine. Cada fila es un ejemplo concreto de una fortaleza.

```sql
CREATE TABLE professional_strengths (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id     UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,

  category    TEXT NOT NULL CHECK (category IN (
                'leadership',          -- Liderazgo y Gestión de Personas
                'industry_business',   -- Experiencia de Industria/Negocio
                'learning_speed'       -- Velocidad de Aprendizaje Demostrada
              )),
  description TEXT NOT NULL,           -- ejemplo concreto con contexto
  -- Ej: "Lideré equipo de 5 personas en proyecto ERP — 8 meses, entregado a tiempo"

  created_at  TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_strengths_user_category ON professional_strengths(user_id, category);

ALTER TABLE professional_strengths ENABLE ROW LEVEL SECURITY;
CREATE POLICY "Users access own strengths"
  ON professional_strengths FOR ALL USING (user_id = auth.uid());
```

---

### `education`

Títulos, certificaciones, y cursos en progreso.

```sql
CREATE TABLE education (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id     UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,

  type        TEXT NOT NULL CHECK (type IN ('degree', 'certification', 'course')),
  institution TEXT NOT NULL,
  title       TEXT NOT NULL,
  summary     TEXT,
  year        INTEGER,
  is_in_progress BOOLEAN DEFAULT FALSE,
  display_order  INTEGER DEFAULT 0,

  created_at  TIMESTAMPTZ DEFAULT NOW()
);

ALTER TABLE education ENABLE ROW LEVEL SECURITY;
CREATE POLICY "Users access own education"
  ON education FOR ALL USING (user_id = auth.uid());
```

---

### `applications`

Tracker de postulaciones. Una fila por oferta evaluada.

```sql
CREATE TABLE applications (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id     UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,

  company     TEXT NOT NULL,
  role        TEXT NOT NULL,
  jd_text     TEXT,                   -- JD original pegado por el usuario
  platform    TEXT,                   -- 'getOnBoard' | 'linkedin_jobs' | 'computrabajo' | etc.
  job_url     TEXT,

  status      TEXT DEFAULT 'screening' CHECK (status IN (
                'screening',           -- solo scoring, no se generó CV
                'cv_generated',        -- CV generado, pendiente de postular
                'applied',             -- postulado
                'in_process',          -- entrevistas en curso
                'rejected',
                'offer',
                'declined'
              )),

  applied_at  TIMESTAMPTZ,
  created_at  TIMESTAMPTZ DEFAULT NOW(),
  updated_at  TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_applications_user_status ON applications(user_id, status);
CREATE INDEX idx_applications_user_created ON applications(user_id, created_at DESC);

ALTER TABLE applications ENABLE ROW LEVEL SECURITY;
CREATE POLICY "Users access own applications"
  ON applications FOR ALL USING (user_id = auth.uid());
```

---

### `scoring_history`

Resultados de cada cálculo de scoring. Permite auditoría y drift detection. El `input_hash` garantiza determinismo.

```sql
CREATE TABLE scoring_history (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id         UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
  application_id  UUID REFERENCES applications(id) ON DELETE SET NULL,

  -- Hash para determinismo: SHA256(jd_text + profile_snapshot_version)
  input_hash      TEXT NOT NULL,

  -- Clasificación de requisitos (output del LLM, input del scoring engine)
  requirements_classification JSONB NOT NULL,
  -- Schema:
  -- {
  --   "type_a": [{"req": "Node.js (3+ años)", "match": "cumple", "score": 1.0, "justification": "..."}],
  --   "type_b": [...],
  --   "type_c": [...],
  --   "type_d": [{"req": "Docker+K8s", "match": "no_cumple", "score": 0.0, "justification": "..."}]
  -- }

  -- Confidence Score desglosado
  obligatorios_score  DECIMAL(5,2),   -- % de A+B
  deseables_score     DECIMAL(5,2),   -- % de C+D
  confidence_base     DECIMAL(5,2),   -- (A+B * 0.60) + (C+D * 0.40)

  -- MVP (Modificador de Valor Profesional)
  mvp_leadership              INTEGER,          -- 0-5
  mvp_leadership_justification TEXT,
  mvp_business                INTEGER,          -- 0-5
  mvp_business_justification  TEXT,
  mvp_learning                INTEGER,          -- 0-5
  mvp_learning_justification  TEXT,
  mvp_total                   INTEGER,          -- suma 0-15
  confidence_final            DECIMAL(5,2),     -- min(base + mvp, 100)

  -- Strategic Alignment Score
  sas_career_goals    INTEGER,          -- 0-20
  sas_motivations     INTEGER,          -- 0-20
  sas_culture         INTEGER,          -- 0-20
  sas_tech_growth     INTEGER,          -- 0-20
  sas_autonomy        INTEGER,          -- 0-20
  sas_total           INTEGER,          -- 0-100
  sas_justification   JSONB,            -- {dimension: justification_text}

  -- Metadata
  model_used          TEXT,             -- "gpt-4o", "gpt-4o-mini"
  created_at          TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_scoring_user ON scoring_history(user_id, created_at DESC);
CREATE INDEX idx_scoring_input_hash ON scoring_history(input_hash);  -- para cache/dedup

ALTER TABLE scoring_history ENABLE ROW LEVEL SECURITY;
CREATE POLICY "Users access own scoring"
  ON scoring_history FOR ALL USING (user_id = auth.uid());
```

---

### `outputs`

CVs generados. Versionado por application: cada regeneración es una nueva fila, nunca se sobreescribe.

```sql
CREATE TABLE outputs (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id         UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
  application_id  UUID NOT NULL REFERENCES applications(id) ON DELETE CASCADE,
  scoring_id      UUID REFERENCES scoring_history(id) ON DELETE SET NULL,
  version         INTEGER DEFAULT 1,

  -- Contenido generado por sección (coherente con las 7 fases)
  killer_summary      TEXT,
  gap_analysis        JSONB,    -- [{requirement, gap_type, mitigation_strategy, is_mitigable}]
  impact_bullets      JSONB,    -- [{experience_id, experience_title, bullets: [str]}]
  skills_matrix       JSONB,    -- [{skill, score, justification}]
  application_qa      JSONB,    -- [{question, answer, platform}]
  interview_intel     JSONB,    -- {technical: [str], culture: [str]}
  brand_coherence_audit JSONB,  -- {tone: bool, pillar: bool, ethos: bool, balance: bool, red_lines: bool, narrative: bool}

  -- Output completo renderizado (para copiar/pegar)
  full_content        TEXT,

  created_at  TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_outputs_application ON outputs(application_id, version DESC);

ALTER TABLE outputs ENABLE ROW LEVEL SECURITY;
CREATE POLICY "Users access own outputs"
  ON outputs FOR ALL USING (user_id = auth.uid());
```

---

### `learnings`

Insights capturados en Fase 7. Pueden promovarse a `common_patterns.md` si son generalizables.

```sql
CREATE TABLE learnings (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id         UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
  application_id  UUID REFERENCES applications(id) ON DELETE SET NULL,

  category        TEXT CHECK (category IN (
                    'non_negotiables', 'brand_voice', 'personal_context',
                    'process', 'general'
                  )),
  description     TEXT NOT NULL,
  trigger_context TEXT,               -- qué situación generó el learning
  solution        TEXT,               -- qué se decidió hacer
  is_generalizable   BOOLEAN DEFAULT FALSE,
  promoted_to_patterns BOOLEAN DEFAULT FALSE,
  promoted_at     TIMESTAMPTZ,

  created_at  TIMESTAMPTZ DEFAULT NOW()
);

ALTER TABLE learnings ENABLE ROW LEVEL SECURITY;
CREATE POLICY "Users access own learnings"
  ON learnings FOR ALL USING (user_id = auth.uid());
```

---

### `platform_schemas`

Schemas de plataformas de postulación. **Tabla compartida (no por usuario)** — datos curados por admin + crowdsourced. RLS permite lectura a todos los usuarios autenticados, escritura solo admin.

```sql
CREATE TABLE platform_schemas (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),

  platform_name TEXT NOT NULL,         -- 'getOnBoard' | 'linkedin_jobs' | 'computrabajo' | 'indeed' | 'laborum'
  platform_url  TEXT,

  layer         INTEGER NOT NULL CHECK (layer IN (1, 2, 3)),
  -- 1 = Datos estándar (nombre, email, LinkedIn) — siempre los mismos
  -- 2 = Campos fijos de plataforma (pretensión de renta, modalidad, "¿por qué te interesa?")
  -- 3 = Preguntas del empleador — ad-hoc, acumuladas del uso real

  field_key     TEXT NOT NULL,         -- snake_case, ej: "salary_expectation"
  field_label   TEXT NOT NULL,         -- "Pretensión de renta"
  field_type    TEXT,                  -- 'text' | 'select' | 'boolean' | 'salary' | 'number' | 'textarea'
  field_options TEXT[],                -- para campos select

  is_required        BOOLEAN DEFAULT FALSE,
  is_crowdsourced    BOOLEAN DEFAULT FALSE,
  contributed_by     UUID REFERENCES auth.users(id) ON DELETE SET NULL,
  validated_by_admin BOOLEAN DEFAULT FALSE,
  last_verified_at   TIMESTAMPTZ,
  version            INTEGER DEFAULT 1,
  notes              TEXT,

  created_at  TIMESTAMPTZ DEFAULT NOW(),
  updated_at  TIMESTAMPTZ DEFAULT NOW(),

  UNIQUE(platform_name, layer, field_key)
);

CREATE INDEX idx_platform_schemas_name ON platform_schemas(platform_name, layer);

ALTER TABLE platform_schemas ENABLE ROW LEVEL SECURITY;

-- Lectura: todos los usuarios autenticados
CREATE POLICY "Authenticated users can read schemas"
  ON platform_schemas FOR SELECT USING (auth.role() = 'authenticated');

-- Escritura: solo admin (service_role o rol admin explícito)
CREATE POLICY "Only admin can write schemas"
  ON platform_schemas FOR INSERT WITH CHECK (auth.jwt() ->> 'role' = 'admin');
CREATE POLICY "Only admin can update schemas"
  ON platform_schemas FOR UPDATE USING (auth.jwt() ->> 'role' = 'admin');
```

---

### `usage_quotas`

Control de costos por usuario. Los contadores se resetean automáticamente (daily via cron job o lazy reset al leer).

```sql
CREATE TABLE usage_quotas (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id     UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE UNIQUE,

  -- Contadores diarios
  messages_today      INTEGER DEFAULT 0,
  screenings_today    INTEGER DEFAULT 0,
  last_daily_reset    DATE DEFAULT CURRENT_DATE,

  -- Contadores mensuales
  cvs_this_month      INTEGER DEFAULT 0,
  last_monthly_reset  DATE DEFAULT DATE_TRUNC('month', CURRENT_DATE)::DATE,

  -- Límites (ajustables por tier en el futuro)
  messages_per_day    INTEGER DEFAULT 50,
  screenings_per_day  INTEGER DEFAULT 5,
  cvs_per_month       INTEGER DEFAULT 10,

  created_at  TIMESTAMPTZ DEFAULT NOW(),
  updated_at  TIMESTAMPTZ DEFAULT NOW()
);

ALTER TABLE usage_quotas ENABLE ROW LEVEL SECURITY;
CREATE POLICY "Users access own quotas"
  ON usage_quotas FOR ALL USING (user_id = auth.uid());
```

---

## Índices Adicionales

```sql
-- Para el historial de postulaciones (tabla principal del dashboard)
CREATE INDEX idx_applications_dashboard
  ON applications(user_id, created_at DESC, status);

-- Para cache de scoring (evitar re-calcular si el input no cambió)
CREATE UNIQUE INDEX idx_scoring_cache
  ON scoring_history(application_id, input_hash);
```

---

## Migrations y Seed

### Estructura de archivos

```
supabase/
├── migrations/
│   ├── 20260224_001_create_profiles.sql
│   ├── 20260224_002_create_work_preferences.sql
│   ├── 20260224_003_create_work_experiences.sql
│   ├── 20260224_004_create_skills.sql
│   ├── 20260224_005_create_projects.sql
│   ├── 20260224_006_create_professional_strengths.sql
│   ├── 20260224_007_create_education.sql
│   ├── 20260224_008_create_applications.sql
│   ├── 20260224_009_create_scoring_history.sql
│   ├── 20260224_010_create_outputs.sql
│   ├── 20260224_011_create_learnings.sql
│   ├── 20260224_012_create_platform_schemas.sql
│   └── 20260224_013_create_usage_quotas.sql
└── seed.sql           -- platform_schemas iniciales para 5 plataformas principales
```

### Trigger: auto-crear registros 1:1 al hacer signup

```sql
-- Se ejecuta automáticamente cuando Supabase Auth crea un usuario
CREATE OR REPLACE FUNCTION handle_new_user()
RETURNS TRIGGER AS $$
BEGIN
  INSERT INTO profiles (id) VALUES (NEW.id);
  INSERT INTO usage_quotas (user_id) VALUES (NEW.id);
  RETURN NEW;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

CREATE TRIGGER on_auth_user_created
  AFTER INSERT ON auth.users
  FOR EACH ROW EXECUTE FUNCTION handle_new_user();
```

---

## Snapshot de Perfil para el Agente

Cuando el agente necesita el perfil completo para calcular scoring o generar contenido, el backend construye un `ProfileSnapshot` via JOIN. Este snapshot es el equivalente a `perfil_base.md` pero en forma estructurada:

```python
# backend/apps/profiles/schemas.py (Pydantic)
class ProfileSnapshot(BaseModel):
    # Identity
    primary_pillar: str | None
    professional_identity: str | None
    differentiator: str | None
    killer_summary: str | None
    non_negotiables: NonNegotiablesSchema

    # Skills (para el scoring engine)
    skills: list[SkillSchema]

    # Experience (para impact bullets)
    work_experiences: list[WorkExperienceSchema]
    projects: list[ProjectSchema]

    # MVP inputs (para scoring engine)
    professional_strengths: list[StrengthSchema]

    # Voice (para brand coherence)
    voice_profile: VoiceProfileSchema | None

    # Preferences (para SAS Dimensión 5)
    work_preferences: WorkPreferencesSchema | None
```

Este snapshot se pasa como contexto a todos los plugins de Semantic Kernel. **Nunca** se pasa la DB connection al agente directamente.

---

## Documentos Relacionados

- `STACK.md` — Pydantic como SSOT del backend
- `SECURITY.md` — RLS como primera capa de seguridad
- `AGENT_PIPELINE.md` — cómo el pipeline consume el ProfileSnapshot
- `MVP.md` — features que usan cada tabla
