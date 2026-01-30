# 🚀 Walkthrough: Usando el CV Optimization Framework

Este documento explica cómo usar el framework para optimizar tu CV hacia una oferta laboral específica.

---

## Fase 1: Preparación

1. Asegúrate de que `perfil_base.md` esté actualizado con tu experiencia más reciente.
2. Revisa `brand_voice.md` para entender tu voz de marca establecida.

---

## Fase 2: Ejecución (La Rutina)

Cuando encuentres una oferta de trabajo interesante:

### Opción A: Sesión Nueva con Cualquier LLM

1. **Copia el contenido de `AGENT_START.md`** y pégalo en una nueva sesión de IA.
2. **Adjunta `perfil_base.md`** y **`brand_voice.md`** para que el agente tenga contexto.
3. **Proporciona la Job Description** que te interesa (texto completo + link si lo tienes).
4. El agente ejecutará las 6 fases automáticamente.

### Opción B: Sesión con Acceso a Archivos

Si el LLM tiene acceso al filesystem (como Claude Code):

1. Indica que deseas optimizar tu CV para una oferta.
2. Proporciona el texto de la JD y el nombre de la empresa.
3. El agente leerá automáticamente los archivos necesarios.

---

## Fase 3: El Resultado

El agente actuará como un **Estratega de Carrera** y generará:

- **Confidence Score:** Probabilidad de match con la oferta
- **Strategic Alignment Score:** Alineación con tus metas de carrera
- **Gap Analysis:** Qué te falta y cómo mitigarlo
- **Killer Summary:** Gancho optimizado para la empresa
- **Impact Bullets:** Logros reescritos para el contexto específico
- **Skills Matrix:** Ranking de habilidades relevantes
- **Interview Intel:** Preguntas probables para preparar
- **Brand Coherence Audit:** Verificación de autenticidad

---

## Archivos del Framework

| Archivo | Cuándo Leer |
|---------|-------------|
| `AGENT_START.md` | Entry point para agentes nuevos |
| `perfil_base.md` | Fuente de verdad - siempre actualizado |
| `brand_voice.md` | Referencia de tono y estilo |
| `framework_protocol.md` | Reglas detalladas del workflow |
| `outputs/` | CVs generados anteriormente |

---

## ¿Listo para una prueba?

Si tienes tu perfil actualizado y una Job Description específica, podemos ejecutar el framework ahora mismo.
