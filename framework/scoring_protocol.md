# 📊 Scoring Protocol: Cálculo Estructurado de Métricas

Este documento define las fórmulas y criterios objetivos para calcular los scores del framework. **Cualquier agente debe seguir este protocolo para asegurar consistencia y transparencia.**

---

## Resumen de Scores

| Score | Qué Mide | Pregunta Central |
|-------|----------|------------------|
| **Confidence Score** | Probabilidad técnica de ser contratado | "¿Cumplo con lo que piden?" |
| **Strategic Alignment Score (SAS)** | Alineación con motivaciones y valores | "¿Esto me conviene a MÍ?" |

---

## 1. CONFIDENCE SCORE (0-100%)

### Definición
Probabilidad de que el usuario sea considerado un candidato viable basado en el match entre sus habilidades documentadas y los requisitos de la oferta.

### Fórmula

```
Confidence Score = (Obligatorios × 0.60) + (Deseables × 0.40)

Donde:
- Obligatorios = (Requisitos obligatorios cumplidos / Total obligatorios) × 100
- Deseables = (Requisitos deseables cumplidos / Total deseables) × 100
```

### Proceso de Cálculo

#### Paso 1: Extraer Requisitos del JD

Clasificar cada requisito en:

| Categoría | Identificadores Típicos |
|-----------|------------------------|
| **Obligatorio** | "Required", "Must have", "Imprescindible", "Requisito", sin calificador |
| **Deseable** | "Nice to have", "Preferred", "Deseable", "Plus", "Bonus" |

#### Paso 2: Evaluar Match contra `../private/perfil_base.md`

Para cada requisito, asignar uno de estos estados:

| Estado | Criterio | Valor |
|--------|----------|-------|
| ✅ **Cumple** | Documentado en Power Stack con puntaje ≥3 | 1.0 |
| 🟡 **Parcial** | Documentado con puntaje 2, o habilidad transferible clara | 0.5 |
| ❌ **No cumple** | No documentado o puntaje 1 | 0.0 |

#### Paso 3: Calcular Score

**Ejemplo práctico:**

```
JD: Backend Developer en Fintech

OBLIGATORIOS (5):
- Node.js (3+ años)        → ✅ Cumple (Power Stack: Node.js en uso)     = 1.0
- PostgreSQL               → ✅ Cumple (Power Stack: PostgreSQL)         = 1.0
- APIs REST                → ✅ Cumple (ExpressJS, Swagger)              = 1.0
- TypeScript               → ✅ Cumple (Power Stack: TypeScript en uso)  = 1.0
- Experiencia en Fintech   → 🟡 Parcial (No fintech, pero CGE es ops)   = 0.5

Subtotal Obligatorios = (4.5 / 5) × 100 = 90%

DESEABLES (3):
- Redis                    → ✅ Cumple (Power Stack: Redis)              = 1.0
- Docker/Kubernetes        → ❌ No cumple                                = 0.0
- CI/CD                    → ✅ Cumple (GitHub Actions)                  = 1.0

Subtotal Deseables = (2 / 3) × 100 = 66.7%

CONFIDENCE SCORE = (90 × 0.60) + (66.7 × 0.40) = 54 + 26.7 = 80.7%
```

### Tabla de Interpretación

| Rango | Interpretación | Recomendación |
|-------|----------------|---------------|
| **90-100%** | Match excepcional | Postular con alta prioridad |
| **75-89%** | Match fuerte | Postular, gaps menores mitigables |
| **60-74%** | Match moderado | Postular si SAS es alto, preparar mitigación de gaps |
| **40-59%** | Match débil | Evaluar si vale la pena, gaps significativos |
| **<40%** | No recomendado | Advertir al usuario antes de continuar |

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

#### Dimensión 1: Metas de Carrera (0-20)

**Metas documentadas del usuario:**
- Profundizar en IA/ML y arquitectura de agentes
- Transición hacia roles de Producto/Arquitectura
- Eventualmente emprender (SaaS propio)

| Puntuación | Criterio |
|------------|----------|
| **20** | El rol acelera directamente una meta (ej: AI Engineer, Product role) |
| **15** | El rol contribuye indirectamente (ej: exposición a IA en el stack) |
| **10** | Neutral - no acerca ni aleja de las metas |
| **5** | El rol podría distraer de las metas principales |
| **0** | El rol va en dirección opuesta a las metas |

#### Dimensión 2: Motivaciones Intrínsecas (0-20)

**Motivaciones documentadas:**
- Curiosidad sistemática y resolución de problemas
- Conexión humana y trabajo con gente diversa
- Optimización de procesos y eficiencia
- Impacto tangible (no solo código por código)

| Puntuación | Criterio |
|------------|----------|
| **20** | El rol satisface 3+ motivaciones directamente |
| **15** | El rol satisface 2 motivaciones |
| **10** | El rol satisface 1 motivación claramente |
| **5** | Neutral - no hay conexión clara con motivaciones |
| **0** | El rol va contra las motivaciones (ej: trabajo aislado, sin impacto visible) |

#### Dimensión 3: Valores y Cultura (0-20)

**Valores documentados:**
- Integridad y honestidad
- Autonomía con responsabilidad
- Equilibrio vida-trabajo
- Ambiente colaborativo (no tóxico)

| Puntuación | Criterio |
|------------|----------|
| **20** | La empresa demuestra alineación cultural fuerte (evidence-based) |
| **15** | Señales positivas de cultura compatible |
| **10** | Información insuficiente para evaluar cultura |
| **5** | Algunas red flags pero no críticas |
| **0** | Red flags claras (cultura tóxica, valores opuestos) |

#### Dimensión 4: Crecimiento Técnico (0-20)

**Áreas de aprendizaje prioritarias (de Power Stack):**
- AI-102, RAG, Fine-tuning, Vectores
- NestJS, Django
- Arquitectura de Software avanzada
- AZ-400 (DevOps avanzado)

| Puntuación | Criterio |
|------------|----------|
| **20** | El rol trabaja directamente con tecnologías que quiero aprender |
| **15** | Exposición parcial a tecnologías objetivo |
| **10** | Stack conocido, poco aprendizaje nuevo pero consolida |
| **5** | Stack que no aporta al roadmap técnico |
| **0** | Stack legacy o tecnologías que no quiero tocar |

#### Dimensión 5: Autonomía y Tipo de Rol (0-20)

**Preferencias de rol ("The Bridge"):**
- Roles que combinen técnico + negocio
- Autonomía para tomar decisiones
- Visibilidad de impacto
- No solo ejecutor - también diseño/estrategia

| Puntuación | Criterio |
|------------|----------|
| **20** | Rol híbrido técnico-producto con autonomía |
| **15** | Rol técnico con exposición a decisiones de negocio |
| **10** | Rol técnico puro pero con ownership |
| **5** | Rol de ejecución con poca autonomía |
| **0** | Rol puramente operativo/repetitivo |

### Ejemplo de Cálculo SAS

```
JD: Product Operations Engineer en Fintoc

Dimensión 1 (Metas): 18/20
- Rol de producto ✅, exposición a fintech ✅, path a Product Manager
- No es AI directamente, pero Product es meta secundaria

Dimensión 2 (Motivaciones): 20/20
- Resolución de problemas operativos ✅
- Trabajo cross-functional ✅
- Impacto directo en producto ✅
- Optimización de procesos ✅

Dimensión 3 (Cultura): 17/20
- Fintoc conocida por buena cultura startup
- Autonomía valorada según JD
- Información limitada sobre balance vida-trabajo

Dimensión 4 (Crecimiento): 12/20
- SQL y herramientas de producto (Retool) - útil pero no prioritario
- No hay AI/ML en el stack
- Aprende operaciones de fintech

Dimensión 5 (Autonomía/Rol): 20/20
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
### 🎯 Confidence Score: 80%

**Desglose:**
| Categoría | Cumplidos | Total | % |
|-----------|-----------|-------|---|
| Obligatorios | 4.5 | 5 | 90% |
| Deseables | 2 | 3 | 67% |

**Cálculo:** (90% × 0.60) + (67% × 0.40) = **80%**

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

- **Metas de carrera:** Actualizar si cambian las prioridades
- **Motivaciones:** Refinar basado en learnings de Fase 7
- **Tecnologías objetivo:** Sincronizar con Power Stack "En Aprendizaje"
- **Ponderación 60/40:** Ajustable si el usuario lo solicita

**Última actualización:** 2025-01-30
