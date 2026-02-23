# MVP Scope — Career Companion Platform

> **Estado:** Definido. Pendiente validación final de prioridades.
> **Última actualización:** 2026-02-22

---

## Principio del MVP

El mínimo producto que demuestre que un Career Companion con IA es significativamente mejor que "pedirle a ChatGPT que mejore mi CV". Tres cosas que debe demostrar:

1. **Scoring determinista** — Mismo JD + mismo perfil = mismo score. Siempre.
2. **Perfil persistente que mejora** — La segunda postulación es mejor que la primera.
3. **Agente como abogado defensor** — No es un chatbot, es un flujo estructurado con advocacy.

---

## Features del MVP

### 1. Autenticación

- **Supabase Auth** con Google y GitHub OAuth
- Row Level Security en todas las tablas
- Sin planes diferenciados en el MVP (todos tienen el mismo acceso con quotas)

### 2. Onboarding Inteligente (Multi-formato)

El usuario puede poblar su perfil de múltiples formas. Todas convergen al mismo resultado: un perfil estructurado en la DB.

**Fuentes de entrada soportadas:**

| Fuente | Qué hace el agente |
|---|---|
| Formulario guiado | Preguntas fase por fase (Modo A del framework) |
| CV existente (texto pegado o PDF) | Extrae datos, evalúa la voz, identifica gaps |
| Párrafo libre / descripción informal | Extrae señales de identidad, skills, experiencia |
| LinkedIn URL (futuro, no MVP) | Scraping + extracción |

**Principio crítico: el input del usuario no es verdad completa.**

El agente trata cualquier input como **materia prima** que necesita:

1. **Extracción** — Sacar datos estructurados (skills, logros, experiencia)
2. **Evaluación de calidad** — ¿Los datos son completos? ¿La voz es profesional o genérica?
3. **Curiosidad activa** — Descubrir logros ocultos, cuantificar experiencia vaga
4. **Pulido de voz** — Mejorar la forma sin perder la esencia y marca personal del usuario

**Niveles de confianza por fuente:**

| Fuente | Confianza en datos | Confianza en voz | Acción del agente |
|---|---|---|---|
| CV existente | Media-alta | Baja (puede ser genérico, pasivo) | Extraer datos, cuestionar la voz, preguntar más |
| Párrafo libre | Media (subjetiva) | Media (más auténtica pero sin pulir) | Usar como semilla de brand_voice, profundizar |
| Conversación directa | Alta | Alta (se calibra en vivo) | Fuente principal de calidad |

**Feedback de voz al usuario:**

Cuando el agente detecta que la voz del CV existente es débil, ofrece feedback constructivo:
> "Tu experiencia es sólida pero la forma en que está escrita no le hace justicia. Por ejemplo, 'participé en el proyecto de migración' podría ser 'lideré la migración de X sistema que impactó a Y usuarios'. ¿Trabajamos la voz juntos manteniendo tu estilo?"

No impone una voz corporativa genérica — busca profesionalizar respetando la autenticidad.

### 3. Chat con Agente (Core)

- Interfaz de chat con streaming (Vercel AI SDK)
- El agente ejecuta el pipeline del framework: screening → scoring → CV generation
- Sistema de plugins via Semantic Kernel
- El scoring se calcula en código Python (determinista), no por el LLM

**Capacidades del agente en el chat:**

| Capacidad | Descripción |
|---|---|
| Screening rápido | Pegar JD → score + veredicto en ~30 segundos |
| CV completo | 7 fases del framework, interactivo |
| Campos adicionales de plataforma | "Me piden [X, Y, Z]" → el agente genera contenido para esos campos |
| Edición de perfil | "Acabo de aprender React" → actualiza el skill en la DB |
| Preguntas de carrera | "¿Debería postular a este rol?" → análisis basado en el perfil |

### 4. Adaptadores de Plataforma (Integrado al Chat)

**No es una feature separada — es parte del flujo de conversación.**

Cuando el usuario genera contenido para un CV, puede decir: "la plataforma donde voy a armar el CV me pide también estos campos: [lista]". El agente genera contenido para esos campos adicionales.

**Schema Crowdsourced:**

```
1. Usuario dice "CVMaker me pide 'Logro principal en 1 línea'"
2. El agente genera el contenido
3. Backend registra: { platform_hint: "CVMaker", field: "Logro principal en 1 línea", type: "short_text" }
4. Admin (tú) valida manualmente entrando a CVMaker → confirma que el campo existe
5. El campo se agrega al schema oficial de CVMaker
6. Futuros usuarios que mencionen CVMaker obtienen esos campos proactivamente
```

**Por qué funciona:** No hay miles de plataformas. En Latam, el universo relevante es ~10-15 (CVMaker, GetOnBoard, LinkedIn, Computrabajo, Laborum, Indeed, etc.). La validación manual es viable y necesaria.

**Impacto:** El producto se adapta orgánicamente a lo que el mercado usa, sin necesidad de mantener integraciones.

### 5. Scoring Engine (Determinista)

- Función Python que recibe clasificación de requisitos + perfil → score numérico
- El LLM clasifica los requisitos (juicio: A/B/C/D, estado de match)
- El código calcula (aritmética: pesos, promedios, MVP modifier, cap a 100)
- Resultado almacenado en `scoring_history` para drift detection

**Garantía:** Si el perfil no cambió y el JD no cambió, el score numérico es idéntico. La clasificación del LLM puede variar levemente, pero las variaciones se mitigan con:
- Cache de clasificaciones previas
- Alerta si una re-clasificación difiere significativamente de una anterior

### 6. Vista de Resultados

No solo chat — una vista estructurada del output:

```
┌─────────────────────────────────────────────────────┐
│  SCREENING: [Empresa] - [Rol]                       │
│                                                      │
│  Confidence Score    ████████░░  72%                 │
│  SAS                 ██████░░░░  58%                 │
│  MVP Modifier        +8%                             │
│  Score Final         ████████░░  80%                 │
│                                                      │
│  ┌───────────┐  ┌───────────┐  ┌───────────────────┐│
│  │ Gap       │  │ CV        │  │ Campos            ││
│  │ Analysis  │  │ Content   │  │ Adicionales       ││
│  └───────────┘  └───────────┘  └───────────────────┘│
│                                                      │
│  [Copiar Todo] [Guardar] [Nueva Postulación]        │
└─────────────────────────────────────────────────────┘
```

### 7. Historial de Postulaciones (Tabla Simple)

- Tabla con: empresa, rol, fecha, score, estado (screening/cv_generado/postulado/en_proceso)
- Filtros básicos por fecha y estado
- Click para ver el resultado completo
- **No es Kanban en el MVP** — una tabla ordenable es suficiente

### 8. Seguridad y Control de Costos

- Intent classifier (GPT-4o-mini) antes de cada mensaje
- Quotas por usuario (mensajes/día, scorings/día, CVs/mes)
- Azure Content Safety para prompt injection
- Rate limiting por IP y por usuario

### 9. Landing Page

- Next.js SSG (estática, rápida)
- Propuesta de valor clara, demo visual del flujo, CTA a signup
- SEO básico para "optimizar CV con IA" en español

---

## Lo que NO Hace el MVP

| Feature | Por qué no | Cuándo |
|---|---|---|
| Kanban de postulaciones | Tabla simple es suficiente | Cuando usuarios pidan mejor organización |
| Voice-to-text | Requiere Azure Speech, no es core | Cuando onboarding por texto sea friction point |
| RAG con vector DB | Perfiles pequeños caben en el prompt | Cuando haya escala |
| Exportar PDF/DOCX | Usuarios ya tienen herramientas para eso | Si lo piden mucho |
| Auto-discovery de ofertas | Requiere scraping infrastructure | V3 |
| Simulador de entrevistas | Feature compleja, no core | V3+ |
| Learning paths | Interesante pero no urgente | V2-V3 |
| Multi-idioma | Español primero (Latam) | Cuando haya demanda en inglés |
| Mobile app | Web responsive es suficiente | Si hay tracción mobile |
| API pública | Solo la web consume el backend | Si hay demanda de integraciones |

---

## Flujo del Usuario (MVP)

```
1. LANDING
   "Tu abogado defensor para la búsqueda de empleo"
   → [Comenzar gratis]

2. SIGNUP
   → Google o GitHub OAuth via Supabase

3. ONBOARDING (primera vez)
   → "¿Cómo prefieres empezar?"
     a) "Tengo un CV o texto sobre mí" → Pegar → Agente extrae + evalúa + pregunta
     b) "Prefiero que me preguntes" → Formulario guiado
     c) "Las dos: pego algo y después me preguntas lo que falta"
   → El agente SIEMPRE hace preguntas adicionales (curiosidad activa)
   → El agente evalúa la voz y ofrece feedback
   → Resultado: perfil poblado en DB

4. DASHBOARD
   → Perfil (% completitud) + Historial de postulaciones + [+ Nueva Postulación]

5. NUEVA POSTULACIÓN
   → Pegar JD (texto)
   → Agente hace screening automático → Score + Veredicto
   → "¿Genero el CV optimizado?" → Sí → Chat interactivo para iterar
   → "La plataforma me pide también [X]" → Agente genera campos adicionales

6. RESULTADO
   → Vista estructurada: scores + gap analysis + CV content por sección
   → [Copiar cada sección] para pegar en la plataforma de CV del usuario
   → Se guarda en historial automáticamente

7. SIGUIENTES USOS
   → Perfil ya poblado → solo pegar JD y obtener resultado
   → Cada sesión mejora el perfil (nuevos logros, learnings capturados)
```

---

## Métricas de Éxito del MVP

| Métrica | Target | Cómo medir |
|---|---|---|
| Onboarding completado | >60% de signups | % que llegan a perfil >50% completo |
| Segunda postulación | >40% de usuarios | % que hacen >1 scoring |
| Retención semana 2 | >30% | % que vuelve después de 7 días |
| NPS informal | >8 | Pregunta in-app después del 3er uso |
| Tiempo a primer resultado | <5 min | Desde signup hasta primer screening |

---

## Estimación de Esfuerzo

| Componente | Esfuerzo estimado | Dependencias |
|---|---|---|
| Supabase setup (schema + auth + RLS) | 1-2 días | Ninguna |
| Django proyecto + API base | 2-3 días | Schema definido |
| Scoring engine (Python) | 2-3 días | Schema de scoring |
| Semantic Kernel plugins | 3-5 días | Scoring engine + API base |
| Intent classifier + security | 1-2 días | Backend funcional |
| Next.js setup + auth flow | 1-2 días | Supabase auth |
| Chat UI con streaming | 2-3 días | Backend agent endpoint |
| Editor de perfil | 2-3 días | API de profiles |
| Vista de resultados | 2-3 días | Agent genera output |
| Historial (tabla) | 1 día | API de applications |
| Landing page | 1-2 días | Ninguna |
| **Total estimado** | **~20-30 días de desarrollo** | |

> **Nota:** Estos son días de desarrollo efectivo con Claude Code como copiloto. No incluye QA profundo, iteración de diseño, ni imprevistos. Factor realista: multiplicar por 1.5-2x.
