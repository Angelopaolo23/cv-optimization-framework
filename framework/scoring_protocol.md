# 📊 Scoring Protocol: Cálculo Estructurado de Métricas

Este documento define las fórmulas y criterios objetivos para calcular los scores del framework. **Cualquier agente debe seguir este protocolo para asegurar consistencia y transparencia.**

---

## Resumen de Scores

| Score | Qué Mide | Pregunta Central |
|-------|----------|------------------|
| **Confidence Score** | Fortaleza de la posición de partida | "¿Qué tan fuerte es mi caso?" |
| **Strategic Alignment Score (SAS)** | Alineación con motivaciones y valores | "¿Esto me conviene a MÍ?" |

---

## 1. CONFIDENCE SCORE (0-100%)

### Definición
Qué tan fuerte es la posición de partida del candidato basado en el match entre sus habilidades documentadas y los requisitos de la oferta. No es una "probabilidad de contratación" — es una medida de qué tan sólido es el caso inicial.

### Fórmula

```
Confidence Score = (Obligatorios × 0.60) + (Deseables × 0.40)

Donde:
- Obligatorios = Suma de puntos (Tipo A + Tipo B) / Total puntos posibles (A + B) × 100
- Deseables = Suma de puntos (Tipo C + Tipo D) / Total puntos posibles (C + D) × 100
- Los requisitos Tipo D tienen peso máximo de 0.5 (ver tabla de clasificación)
```

> **Nota sobre Tipo D:** Los requisitos inflados pesan la mitad porque su ausencia rara vez afecta la candidatura real. Si un JD tiene muchos Tipo D, el score refleja mejor la posición real del candidato al no penalizarlo por wishlist irreal.

### Proceso de Cálculo

#### Paso 1: Extraer y Clasificar Requisitos del JD

Cada requisito del JD se clasifica en uno de 4 tipos. Esta clasificación reemplaza la distinción binaria obligatorio/deseable, porque **los JDs mezclan filtros reales con wishlists infladas** — y un buen abogado sabe distinguirlos.

| Tipo | Nombre | Qué Significa | Peso en Fórmula |
|------|--------|---------------|-----------------|
| **A** | Filtro Duro | Requisito eliminatorio real. Sin esto, el CV no pasa el ATS o el primer filtro humano. | × 1.0 (crítico) |
| **B** | Obligatorio Estándar | Requisito importante pero negociable. La mayoría de contratados probablemente lo tienen, pero no es knockout. | × 1.0 |
| **C** | Deseable Real | Genuinamente valioso para el rol. Diferencia candidatos pero no elimina. | × 1.0 |
| **D** | Deseable Inflado | Padding del JD. Wishlist aspiracional, certificaciones decorativas, o requisitos que contradicen el nivel del rol. | × 0.5 |

##### Heurísticas de Clasificación

**Para identificar Tipo A (Filtro Duro):**
- El JD dice explícitamente "excluyente", "eliminatorio", "requisito mínimo indispensable"
- Certificaciones legales o regulatorias (ej: título profesional para ejercer, visa de trabajo)
- Años mínimos de experiencia cuando el rol es senior/lead y la empresa es grande o regulada
- El requisito aparece en el título del puesto (ej: "React Developer" → React es Tipo A)
- Sin este skill, el candidato literalmente no podría hacer el trabajo el día 1

**Para identificar Tipo B (Obligatorio Estándar):**
- "Required", "Must have", "Requisito" sin énfasis adicional
- Skills del stack principal mencionados en la descripción del trabajo diario
- Años de experiencia en roles junior/mid (frecuentemente negociables)
- Aparece en sección de requisitos pero sin lenguaje eliminatorio

**Para identificar Tipo C (Deseable Real):**
- "Nice to have", "Preferred", "Deseable", "Plus", "Bonus"
- Skills complementarios que agregan valor genuino al rol
- Experiencia en la industria específica (cuando no es regulada)
- Herramientas específicas que tienen alternativas equivalentes (ej: "Jira" cuando cualquier PM tool sirve)

**Para identificar Tipo D (Deseable Inflado):**
- El requisito contradice el nivel del rol (ej: pedir 10 años de experiencia para un mid-level)
- Lista de 15+ tecnologías donde nadie domina todas
- Certificaciones que no son estándar de la industria para ese rol
- "Experiencia en startup Y enterprise Y consulting" — pedir todo a la vez
- El requisito parece copiado de otro JD o es genérico (ej: "passionate about technology")
- Skill que apareció hace menos tiempo del que piden de experiencia

> **Principio:** Ante la duda entre B y C, clasificar como C (beneficia al candidato). Ante la duda entre C y D, clasificar como C (ser conservador con desinflar). Documentar siempre la justificación de cada Tipo A.

#### Paso 2: Evaluar Match contra `../private/perfil_base.md`

Para cada requisito, asignar uno de estos estados:

| Estado | Criterio | Valor |
|--------|----------|-------|
| ✅ **Cumple** | Documentado en Power Stack con puntaje ≥3, o experiencia directa verificable | 1.0 |
| 🔄 **Transferible** | No tiene la skill exacta, pero tiene una habilidad equivalente o directamente transferible con evidencia concreta | 0.7 |
| 🟡 **Parcial** | Documentado con puntaje 2, o exposición limitada sin evidencia de aplicación independiente | 0.5 |
| ❌ **No cumple** | No documentado o puntaje 1 | 0.0 |

##### Cuándo usar "Transferible" vs "Parcial"

**Transferible (0.7)** — el candidato puede hacer el trabajo con mínima rampa:
- Piden PostgreSQL, tiene MySQL con experiencia en producción → la skill SQL es transferible
- Piden experiencia en fintech, tiene 3 años en operaciones financieras en retail → el conocimiento de dominio transfiere
- Piden React, tiene Vue.js con proyectos complejos → frameworks SPA son transferibles
- Piden liderazgo de equipo de desarrollo, tiene liderazgo de equipos multidisciplinarios → la gestión de personas transfiere

**Parcial (0.5)** — hay exposición pero no evidencia de aplicación sólida:
- Piden PostgreSQL, ha hecho tutoriales de SQL pero nunca en producción
- Piden liderazgo, ha coordinado tareas pero nunca liderado un equipo
- Tiene la skill en Power Stack con puntaje 2 (en aprendizaje)

> **Principio:** El abogado defensor argumenta transferibilidad cuando hay evidencia real. "Sabe Vue, por lo tanto puede React" es un argumento válido. "Leyó un artículo sobre React" no lo es.

#### Paso 3: Calcular Score

**Ejemplo práctico:**

```
JD: Backend Developer en Fintech

TIPO A — Filtro Duro (2):
- Node.js (3+ años)        → ✅ Cumple (Power Stack: Node.js en uso)     = 1.0
  [Tipo A: aparece en título del puesto, skill central del rol]
- APIs REST                → ✅ Cumple (ExpressJS, Swagger)              = 1.0
  [Tipo A: sin esto no puede hacer el trabajo el día 1]

TIPO B — Obligatorio Estándar (3):
- PostgreSQL               → ✅ Cumple (Power Stack: PostgreSQL)         = 1.0
  [Tipo B: requerido pero hay alternativas SQL equivalentes]
- TypeScript               → ✅ Cumple (Power Stack: TypeScript en uso)  = 1.0
  [Tipo B: en sección "requisitos" sin lenguaje eliminatorio]
- Experiencia en Fintech   → 🔄 Transferible (No fintech, pero CGE es ops financieras reguladas) = 0.7
  [Tipo B: experiencia en operaciones financieras reguladas transfiere a fintech]

Subtotal Obligatorios (A+B) = (4.7 / 5) × 100 = 94%

TIPO C — Deseable Real (2):
- Redis                    → ✅ Cumple (Power Stack: Redis)              = 1.0
  [Tipo C: "nice to have" genuino, agrega valor al rol]
- CI/CD                    → ✅ Cumple (GitHub Actions)                  = 1.0
  [Tipo C: "preferred", complementario al desarrollo]

TIPO D — Deseable Inflado (1):
- Docker + Kubernetes + Terraform → ❌ No cumple                        = 0.0
  [Tipo D: piden 3 herramientas DevOps para un rol Backend mid — inflado]
  Peso máximo: 0.5 → puntos posibles = 0.5

Subtotal Deseables (C+D) = (2.0 / 2.5) × 100 = 80%

CONFIDENCE SCORE (Base) = (94 × 0.60) + (80 × 0.40) = 56.4 + 32 = 88.4%
```

> **Comparación:** Con clasificación binaria antigua y sin estado Transferible, el score habría sido 80.7%. El sistema de 4 tipos + Transferible reconoce que (1) el requisito DevOps estaba inflado y (2) la experiencia en operaciones financieras es transferible a fintech, dando un score más representativo de la posición real.

### Paso 4: Aplicar Modificador de Valor Profesional (MVP)

El Confidence Score base mide match técnico contra el JD. Pero un candidato es más que su stack — y un buen abogado presenta **toda** la evidencia relevante, no solo la que la contraparte pidió.

El MVP captura fortalezas profesionales documentadas en `../private/perfil_base.md` que **no están listadas en el JD pero son relevantes para el rol**.

#### Fórmula

```
Confidence Score Final = min(Base Score + MVP, 100)

MVP = Liderazgo + Negocio + Aprendizaje

Cada categoría: 0-5 puntos
Máximo MVP: +15 puntos
Cap: el score final nunca supera 100%
```

#### Categorías del MVP

##### 1. Liderazgo y Gestión de Personas (0-5)

Evalúa experiencia liderando equipos, gestionando stakeholders, coordinando proyectos cross-functional. Relevante para casi cualquier rol que no sea puramente individual contributor junior.

| Puntos | Criterio |
|--------|----------|
| **5** | Liderazgo formal de equipos + gestión de stakeholders senior documentados |
| **3** | Coordinación de proyectos o equipos pequeños con evidencia |
| **1** | Experiencia colaborativa pero sin rol de liderazgo |
| **0** | No relevante para este rol o sin evidencia |

**Fuente de datos:** Sección de experiencia laboral en `perfil_base.md` — buscar roles con responsabilidad sobre personas, proyectos, o decisiones.

##### 2. Experiencia de Industria/Negocio (0-5)

Evalúa conocimiento del sector, visión de negocio, capacidad de conectar decisiones técnicas con impacto comercial. Especialmente relevante para roles que piden "business acumen" o están en industrias específicas.

| Puntos | Criterio |
|--------|----------|
| **5** | Experiencia directa en la industria del rol + visión de negocio demostrada |
| **3** | Experiencia en industria adyacente o conocimiento de negocio genérico fuerte |
| **1** | Exposición limitada al mundo de negocio |
| **0** | No relevante para este rol o sin evidencia |

**Fuente de datos:** Historial laboral, formación académica (ej: Ingeniería Comercial, MBA), y sección de identidad profesional en `perfil_base.md`.

##### 3. Velocidad de Aprendizaje Demostrada (0-5)

Evalúa evidencia concreta de que el candidato aprende rápido y se adapta a contextos nuevos. No es "creo que aprendo rápido" — es track record verificable.

| Puntos | Criterio |
|--------|----------|
| **5** | Múltiples transiciones exitosas documentadas (cambio de carrera, nuevo stack adoptado, certificaciones en poco tiempo) |
| **3** | Al menos 1 transición o adopción rápida documentada |
| **1** | Evidencia indirecta de adaptabilidad |
| **0** | No relevante para este rol o sin evidencia |

**Fuente de datos:** Trayectoria laboral, transiciones de carrera, certificaciones con fechas, proyectos personales que muestren stacks nuevos.

#### Cuándo aplicar el MVP

- **SIEMPRE evaluar** las 3 categorías, pero asignar 0 cuando no son relevantes para el rol específico
- **Ejemplo de relevancia:** Para un rol de "Junior React Developer" en una startup de 5 personas, Liderazgo probablemente puntúa 0 (no relevante). Para un "Full Stack Developer" en empresa grande que coordina con múltiples equipos, puede puntuar 3-5
- **El MVP no es un regalo** — cada punto debe tener evidencia en `perfil_base.md` y ser argumentablemente relevante para el rol

#### Datos a Solicitar al Usuario

Para que el MVP funcione, `perfil_base.md` debe contener esta información. Si no está documentada, el agente debe **preguntar proactivamente** durante el onboarding o al inicio de la primera sesión:

1. **Liderazgo:** "¿Has liderado equipos, coordinado proyectos, o gestionado stakeholders? Cuéntame sobre personas a tu cargo, decisiones que tomaste, conflictos que resolviste."
2. **Negocio:** "¿Tienes formación o experiencia en negocio, gestión, o estrategia? ¿Has trabajado directamente con clientes, definido producto, o tomado decisiones con impacto comercial?"
3. **Aprendizaje:** "¿Has cambiado de carrera, aprendido un stack nuevo en poco tiempo, o sacado certificaciones? Dame ejemplos concretos con contexto y tiempos."

> **Importante:** Esta información se documenta en `perfil_base.md` en las secciones de experiencia laboral y/o en una sección dedicada de "Fortalezas Profesionales". No va en `brand_voice.md` — es evidencia factual, no estilo.

#### Ejemplo de MVP aplicado

```
Continuación del ejemplo anterior (Backend Developer en Fintech):

CONFIDENCE SCORE BASE = 88.4%

MVP:
1. Liderazgo y Gestión: 4/5
   - Lideró equipos multidisciplinarios de 8 personas en CGE
   - Gestionó stakeholders C-level en proyectos de optimización
   - Relevante: el JD menciona "colaboración cross-team"

2. Experiencia de Industria/Negocio: 3/5
   - Ingeniería Comercial + experiencia en operaciones financieras
   - No es fintech directo, pero entiende regulación y procesos financieros
   - Relevante: fintech valora entender el negocio, no solo el código

3. Velocidad de Aprendizaje: 4/5
   - Transición de negocios internacionales a desarrollo Full Stack
   - Múltiples tecnologías adoptadas en <1 año (Node, React, PostgreSQL)
   - Relevante: señala que cualquier gap técnico tiene vida corta

MVP = 4 + 3 + 4 = +11 puntos

CONFIDENCE SCORE FINAL = min(88.4 + 11, 100) = 99.4% → 99%
```

> **Nota:** Un MVP de +11 es alto. Esto refleja un candidato con perfil integral fuerte que va más allá de lo técnico. El score final de 99% no significa "candidato perfecto" — significa que el caso que podemos construir es muy sólido considerando toda la evidencia disponible.

### Tabla de Interpretación

| Rango | Interpretación | Recomendación |
|-------|----------------|---------------|
| **90-100%** | Match excepcional | Postular con alta prioridad |
| **75-89%** | Match fuerte | Postular, gaps menores mitigables |
| **60-74%** | Match moderado | Postular si SAS es alto, preparar mitigación de gaps |
| **40-59%** | Match desafiante | Gaps significativos. Evaluar habilidades transferibles y valor estratégico |
| **<40%** | Gap significativo | Evaluar valor estratégico con el usuario. Un score bajo no significa "no postular" |

---

## 2. STRATEGIC ALIGNMENT SCORE (SAS) (0-100%)

### Definición
Qué tanto se alinea esta oportunidad con las motivaciones intrínsecas, valores, y metas de carrera del usuario. **No mide si el usuario califica, sino si el trabajo le CONVIENE.**

### Dimensiones de Evaluación

El SAS se compone de 5 dimensiones, cada una vale 20 puntos (máximo 100):

| Dimensión | Qué Evalúa | Referencia |
|-----------|------------|------------|
| **1. Metas de Carrera** | ¿Acerca al usuario a sus objetivos? | Roadmap personal |
| **2. Motivaciones Intrínsecas** | ¿Conecta con lo que lo mueve internamente? | Perfil ETHOS |
| **3. Valores y Cultura** | ¿Hay fit con los valores del usuario? | Non-Negotiables |
| **4. Crecimiento Técnico** | ¿Permite aprender lo que quiere aprender? | Power Stack "En Aprendizaje" |
| **5. Autonomía y Rol** | ¿El tipo de trabajo es el que busca? | Identidad "The Bridge" |

### Fórmula

```
SAS = Σ (Puntuación por dimensión)

Cada dimensión: 0-20 puntos
Total máximo: 100 puntos
```

### Rubrica de Evaluación por Dimensión

Cada dimensión se evalúa consultando la sección correspondiente de `../private/perfil_base.md`. Las rubricas son genéricas — los datos específicos del usuario vienen del perfil.

#### Dimensión 1: Metas de Carrera (0-20)

**Fuente de datos:** Sección "Metas de Carrera" o "Career Roadmap" en `perfil_base.md`.

| Puntuación | Criterio |
|------------|----------|
| **20** | El rol acelera directamente una meta documentada |
| **15** | El rol contribuye indirectamente a una meta |
| **10** | Neutral — no acerca ni aleja de las metas |
| **5** | El rol podría distraer de las metas principales |
| **0** | El rol va en dirección opuesta a las metas |

#### Dimensión 2: Motivaciones Intrínsecas (0-20)

**Fuente de datos:** Sección "Motivaciones" o "Ethos" en `perfil_base.md`.

| Puntuación | Criterio |
|------------|----------|
| **20** | El rol satisface 3+ motivaciones documentadas |
| **15** | El rol satisface 2 motivaciones |
| **10** | El rol satisface 1 motivación claramente |
| **5** | Neutral — sin conexión clara con motivaciones |
| **0** | El rol va contra las motivaciones (ej: trabajo aislado cuando valora conexión) |

#### Dimensión 3: Valores y Cultura (0-20)

**Fuente de datos:** Sección "Non-Negotiables" y "Valores" en `perfil_base.md`.

| Puntuación | Criterio |
|------------|----------|
| **20** | La empresa demuestra alineación cultural fuerte (evidence-based) |
| **15** | Señales positivas de cultura compatible |
| **10** | Información insuficiente para evaluar cultura |
| **5** | Algunas red flags pero no críticas |
| **0** | Red flags claras (cultura tóxica, valores opuestos a los documentados) |

#### Dimensión 4: Crecimiento Técnico (0-20)

**Fuente de datos:** Sección "Power Stack — En Aprendizaje" y tecnologías marcadas como objetivo en `perfil_base.md`.

| Puntuación | Criterio |
|------------|----------|
| **20** | El rol trabaja directamente con tecnologías que el usuario quiere aprender |
| **15** | Exposición parcial a tecnologías objetivo |
| **10** | Stack conocido — poco aprendizaje nuevo pero consolida |
| **5** | Stack que no aporta al roadmap técnico del usuario |
| **0** | Stack legacy o tecnologías que el usuario no quiere tocar |

#### Dimensión 5: Autonomía y Tipo de Rol (0-20)

**Fuente de datos:** Sección "Identidad Profesional" y preferencias de rol en `perfil_base.md`.

| Puntuación | Criterio |
|------------|----------|
| **20** | Match perfecto con el tipo de rol buscado (ej: híbrido, autónomo, estratégico) |
| **15** | Match parcial — el rol cubre la mayoría de preferencias |
| **10** | Rol aceptable pero no ideal en estructura |
| **5** | Rol con poca alineación al tipo de trabajo buscado |
| **0** | Rol opuesto a las preferencias (ej: puramente operativo cuando busca estratégico) |

### Ejemplo de Cálculo SAS

> **Nota:** Este ejemplo usa datos del usuario actual para ilustrar el proceso. Cada usuario tendrá sus propias metas, motivaciones, valores y preferencias documentadas en su `perfil_base.md`.

```
JD: Product Operations Engineer en Fintoc
Usuario: Perfil "The Bridge" (técnico + negocio)

Dimensión 1 (Metas): 18/20
- Metas documentadas: transición a Producto, IA/ML, emprendimiento
- Rol de producto ✅, path a Product Manager ✅
- No es AI directamente, pero Product es meta secundaria

Dimensión 2 (Motivaciones): 20/20
- Motivaciones documentadas: curiosidad, conexión humana, optimización, impacto
- Resolución de problemas operativos ✅, cross-functional ✅
- Impacto directo en producto ✅, optimización de procesos ✅

Dimensión 3 (Cultura): 17/20
- Non-negotiables: integridad, autonomía, equilibrio, colaboración
- Fintoc conocida por buena cultura startup ✅
- Autonomía valorada según JD ✅
- Información limitada sobre balance vida-trabajo

Dimensión 4 (Crecimiento): 12/20
- Tecnologías objetivo: AI-102, RAG, NestJS, Django, Arquitectura avanzada
- SQL y Retool — útil pero no prioritario
- No hay AI/ML en el stack
- Aprende operaciones de fintech (valor indirecto)

Dimensión 5 (Autonomía/Rol): 20/20
- Preferencias: híbrido técnico-negocio, autonomía, visibilidad de impacto
- Rol híbrido operaciones-producto ✅
- Alta autonomía según descripción ✅
- Impacto visible en el negocio ✅

SAS = 18 + 20 + 17 + 12 + 20 = 87%
```

### Tabla de Interpretación SAS

| Rango | Interpretación | Recomendación |
|-------|----------------|---------------|
| **85-100%** | Alineación excepcional | Priorizar esta oportunidad |
| **70-84%** | Buena alineación | Vale la pena, considerar trade-offs |
| **50-69%** | Alineación parcial | Evaluar si los gaps son aceptables |
| **30-49%** | Alineación débil | Solo postular si hay razones estratégicas |
| **<30%** | No alineado | No recomendado - advertir al usuario |

---

## 3. MATRIZ DE DECISIÓN COMBINADA

Usar ambos scores para guiar la recomendación:

| | SAS Alto (≥70%) | SAS Medio (50-69%) | SAS Bajo (<50%) |
|---|---|---|---|
| **Confidence Alto (≥75%)** | 🟢 **Prioridad máxima** | 🟡 Postular, considerar trade-offs | 🟠 Solo si hay razón estratégica |
| **Confidence Medio (50-74%)** | 🟡 Postular, preparar gaps | 🟠 Evaluar esfuerzo vs. beneficio | 🔴 Probablemente no vale la pena |
| **Confidence Bajo (<50%)** | 🟠 Solo si es "stretch role" deseado | 🔴 No recomendado | 🔴 **Descartar** |

---

## 4. DOCUMENTACIÓN EN OUTPUT

Al reportar scores en el CV optimizado, incluir:

```markdown
### 🎯 Confidence Score: 99%

**Clasificación de Requisitos:**
| Requisito | Tipo | Match | Puntos |
|-----------|------|-------|--------|
| Node.js (3+ años) | A — Filtro Duro | ✅ Cumple | 1.0/1.0 |
| APIs REST | A — Filtro Duro | ✅ Cumple | 1.0/1.0 |
| PostgreSQL | B — Obligatorio | ✅ Cumple | 1.0/1.0 |
| TypeScript | B — Obligatorio | ✅ Cumple | 1.0/1.0 |
| Exp. Fintech | B — Obligatorio | 🔄 Transferible | 0.7/1.0 |
| Redis | C — Deseable Real | ✅ Cumple | 1.0/1.0 |
| CI/CD | C — Deseable Real | ✅ Cumple | 1.0/1.0 |
| Docker+K8s+Terraform | D — Inflado | ❌ No cumple | 0.0/0.5 |

**Desglose:**
| Categoría | Puntos | Posible | % |
|-----------|--------|---------|---|
| Obligatorios (A+B) | 4.7 | 5.0 | 94% |
| Deseables (C+D) | 2.0 | 2.5 | 80% |

**Base:** (94% × 0.60) + (80% × 0.40) = **88.4%**

**Modificador de Valor Profesional (MVP):**
| Categoría | Puntos | Justificación |
|-----------|--------|---------------|
| Liderazgo y Gestión | 4/5 | Lideró equipos de 8, stakeholders C-level |
| Industria/Negocio | 3/5 | Ing. Comercial + ops financieras reguladas |
| Velocidad de Aprendizaje | 4/5 | Transición negocio→dev, stack adoptado en <1 año |

**MVP:** +11 puntos

**Confidence Score Final:** min(88.4 + 11, 100) = **99%**

### 🧠 Strategic Alignment Score (SAS): 87%

**Desglose por dimensión:**
| Dimensión | Puntos | Justificación |
|-----------|--------|---------------|
| Metas de Carrera | 18/20 | Rol de producto alineado con transición deseada |
| Motivaciones | 20/20 | Satisface curiosidad, conexión, optimización |
| Cultura | 17/20 | Señales positivas, info limitada en balance |
| Crecimiento Técnico | 12/20 | SQL útil, pero no IA/ML |
| Autonomía/Rol | 20/20 | Híbrido producto-ops, alta autonomía |

**Total:** 87/100
```

---

## 5. ACTUALIZACIÓN DE CRITERIOS

Los criterios de este protocolo deben evolucionar junto con `../private/perfil_base.md`:

### Confidence Score
- **Clasificación A-D:** Las heurísticas de clasificación son genéricas, pero el agente puede refinarlas basado en patrones observados en `../private/learnings.md`
- **Estado Transferible (0.7):** El umbral de "transferibilidad" se calibra con experiencia — si un Transferible resulta ser más fuerte o más débil en la práctica, documentar en learnings
- **Ponderación 60/40:** Ajustable si el usuario lo solicita (ej: para roles donde los deseables son más relevantes)

### MVP (Modificador de Valor Profesional)
- **Categorías:** Las 3 categorías actuales (Liderazgo, Negocio, Aprendizaje) cubren los casos más comunes. Si se identifica una categoría nueva recurrente, agregarla (máximo 4 categorías, ajustar cap)
- **Datos del usuario:** Si `perfil_base.md` no tiene información suficiente para evaluar el MVP, el agente debe solicitarla proactivamente (ver "Datos a Solicitar al Usuario" en Paso 4)
- **Cap de +15:** Ajustable si se demuestra que es insuficiente para perfiles muy senior o muy multidisciplinarios

### SAS
- **Metas de carrera:** Actualizar si cambian las prioridades del usuario
- **Motivaciones:** Refinar basado en learnings de Fase 7
- **Tecnologías objetivo:** Sincronizar con Power Stack "En Aprendizaje"
- **Dimensiones:** Las 5 dimensiones son genéricas. Los datos específicos se leen de `perfil_base.md`

**Última actualización:** 2026-02-16
