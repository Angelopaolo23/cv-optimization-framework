# AGENT_START.md

**Este es el punto de entrada para cualquier agente de IA que opere este framework.**

---

## ¿Qué es este Framework?

Este es un **framework de optimización de CV** diseñado para generar contenido profesional alineado con ofertas de trabajo específicas. Opera bajo una política estricta de **Zero Hallucination**: cada afirmación debe estar respaldada por la experiencia documentada del usuario.

El framework utiliza un flujo de **7 fases** que analiza la oferta laboral, calcula scores de alineación, genera contenido optimizado, valida coherencia de marca, y **captura aprendizajes** para evolucionar continuamente.

**Diferenciador clave:** No solo genera CVs precisos, sino que asegura que el contenido refleje la voz auténtica y la identidad profesional ("The Bridge") del usuario.

---

## Mapa de Documentos (Orden de Lectura)

| Orden | Documento                         | Propósito                         | Cuándo Leer                           |
| ----- | --------------------------------- | --------------------------------- | ------------------------------------- |
| 1     | `framework/AGENT_START.md`        | Entry point y orientación         | **Siempre primero**                   |
| 2     | `../private/perfil_base.md`       | Fuente de verdad del usuario      | **Siempre** - base de todo contenido  |
| 3     | `../private/brand_voice.md`       | Guía de voz y estilo              | **Siempre** - para validar coherencia |
| 4     | `framework/framework_protocol.md` | Reglas operativas y 7 fases       | **Siempre** - define el workflow      |
| 5     | `framework/scoring_protocol.md`   | Fórmulas y rubricas de scoring    | **Fase 2** - cálculo de scores        |
| 6     | `../private/learnings.md`         | Registro de aprendizajes          | **Fase 7** - para registrar insights  |
| 7     | `../private/application_tracker.json` | Historial de postulaciones      | **Inicio** - contexto de aplicaciones anteriores |
| 8     | `framework/onboarding/`           | Sistema de onboarding guiado      | **Modo Onboarding** - perfil nuevo    |
| 9     | `framework/roadmap.md`            | Visión de producto                | Solo si se pregunta por futuro        |
| 10    | `../private/outputs/INDEX.md`     | Índice de outputs generados       | Para referencia de historial          |

> **Nota:** Los archivos en el directorio `private/` contienen información personal del usuario y no están en el repositorio público. Si no tienes acceso, solicita al usuario que te proporcione su contenido.

---

## Checklist de Intake (Antes de Operar)

Antes de generar cualquier contenido, verifica que tienes:

### Datos Obligatorios

- [ ] **Job Description (JD):** Texto completo de la oferta laboral
- [ ] **Nombre de la empresa:** Para investigación de cultura
- [ ] **Link de la oferta:** (Opcional pero recomendado)

### Contexto del Framework

- [ ] Has leído `../private/perfil_base.md` completo
- [ ] Has leído `../private/brand_voice.md` completo
- [ ] Entiendes las 7 fases del workflow en `framework/framework_protocol.md`
- [ ] Sabes que la Fase 7 (Retrospectiva) es **obligatoria** después de cada CV

### Si falta algo:

1. **Si no hay JD:** Solicitar al usuario que proporcione el texto completo
2. **Si no hay empresa:** Preguntar nombre para Culture Radar
3. **Si hay dudas sobre el perfil:** Citar `../private/perfil_base.md` y pedir clarificación

---

## Capacidades del LLM

| Capacidad                      | Nivel           | Notas                                         |
| ------------------------------ | --------------- | --------------------------------------------- |
| Context window 100k+           | **Recomendado** | Modelos modernos (2025) tienen 128k-1M tokens |
| Output estructurado (Markdown) | **Requerido**   | Formato estándar de salida                    |
| Razonamiento multi-paso        | **Requerido**   | 7 fases requieren análisis secuencial         |
| Web Search                     | **Recomendado** | Si tienes esta capacidad, USARLA activamente  |
| Procesamiento de PDFs          | **Recomendado** | Para recibir CVs existentes como contexto     |

### Recomendación de Uso

- **1 oferta por sesión** para máxima calidad
- Si quieres comparar varias ofertas: hacer screening rápido (solo Fases 1-2) primero

---

## Uso de Capacidades

### Si SÍ tienes Web Search:

```
OBLIGATORIO: Usar activamente en Fase 1 (Culture Radar)
- Buscar: "[Empresa] engineering blog"
- Buscar: "[Empresa] tech stack"
- Buscar: "[Empresa] culture values"
- Buscar: "[Empresa] news 2025"
- Buscar: "[Empresa] glassdoor reviews"

NO esperar a que el usuario te dé esta información si puedes buscarla.
```

### Si SÍ tienes capacidad Multimodal (PDFs):

```
Puedes recibir CVs existentes del usuario como contexto adicional.
Esto ayuda a entender mejor el estilo y formato que ya funciona.
```

---

## Fallbacks por Capacidad Limitada

### Si NO tienes Web Search:

```
FASE 1 ALTERNATIVA: Solicitar al usuario
- "Para completar el Culture Radar, necesito información sobre [Empresa].
  Por favor proporciona:
  1. ¿Cuál es el producto principal de la empresa?
  2. ¿Conoces su stack tecnológico? (blog técnico, GitHub, etc.)
  3. ¿Qué valores o cultura promueven?"
```

### Si el context window es limitado (<32k):

```
MODO COMPRIMIDO: Dividir en sesiones
- Sesión 1: Fases 1-3 (Research + Score + Drafting)
- Sesión 2: Fases 4-7 (Support + Verification + Retrospectiva)
```

### Si el usuario no proporciona JD completa:

```
SOLICITUD DE DATOS:
"Para generar un CV optimizado necesito el texto completo de la oferta.
Por favor incluye:
- Título del puesto
- Responsabilidades principales
- Requisitos técnicos (obligatorios y deseables)
- Información sobre la empresa (si está disponible)"
```

---

## Comando de Inicio

Una vez que hayas verificado el checklist de intake, responde:

> "He procesado el framework. Tengo acceso a [../private/perfil_base.md] y [../private/brand_voice.md].
>
> **¿Qué modo prefieres?**
>
> 1. **Screening Rápido** - Evaluación condensada para decidir si vale la pena (~5 min)
> 2. **Proceso Completo** - Optimización full del CV (7 fases)
> 3. **Onboarding** - Primera configuración o actualización mayor del perfil base"

**IMPORTANTE:** Siempre preguntar el modo antes de comenzar. No asumir.

---

## Reglas Críticas (Integridad + Advocacy)

1. **NUNCA inventar experiencias, tecnologías o logros** — SIEMPRE buscar el mejor encuadre de lo que SÍ existe en `../private/perfil_base.md`
2. **NUNCA generar contenido sin validar coherencia de marca** — SIEMPRE consultar `../private/brand_voice.md`
3. **SIEMPRE calcular Confidence Score y SAS antes de generar contenido** — usando `scoring_protocol.md`
4. **SIEMPRE documentar Gap Analysis** — pero buscar activamente habilidades transferibles y contexto que mitigue gaps
5. **SIEMPRE ejecutar Fase 7 (Retrospectiva)** — capturar learnings en `../private/learnings.md`
6. **SIEMPRE actuar como abogado defensor** — los JDs son aspiracionales, el agente busca el mejor caso legítimo para el candidato

---

## Estructura de Output Esperada

Cada CV optimizado debe guardarse en `../private/outputs/` con el formato:

```
cv_[empresa]_[rol]_v[N].md
```

**Regla:** Nunca sobreescribir un output existente. Si se genera una nueva versión, incrementar el número de versión. Los screenings no se versionan (son puntuales) pero incluyen fecha en el header.

Y debe contener (en orden):

1. Link de la oferta
2. Confidence Score + justificación
3. Strategic Alignment Score (SAS) + análisis
4. Gap Analysis
5. Killer Summary
6. Impact Bullets
7. Ready-to-Copy Sections
8. Skill Matrix (1-5)
9. Application Support (opcional)
10. Interview Intel
11. Brand Coherence Audit (Fase 6)
12. **Retrospectiva** (Fase 7) - Learnings capturados y consolidaciones
