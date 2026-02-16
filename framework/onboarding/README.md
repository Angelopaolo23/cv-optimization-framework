# Sistema de Onboarding

Este directorio contiene el flujo de configuración inicial para nuevos usuarios del framework. El objetivo es poblar `private/perfil_base.md` y `private/brand_voice.md` con la información necesaria para que el framework funcione.

---

## Cuándo Usar Onboarding

- **Primera vez:** El usuario no tiene `perfil_base.md` ni `brand_voice.md` poblados
- **Actualización mayor:** El usuario cambió de carrera, sumó experiencia significativa, o quiere rehacer su perfil desde cero
- **Perfil incompleto:** Existe `perfil_base.md` pero le faltan secciones clave (Power Stack, Fortalezas Profesionales, Non-Negotiables)

## Modos de Onboarding

| Modo | Archivo | Para Quién | Duración Estimada |
|------|---------|------------|-------------------|
| **A: Guiado** | `mode_a_guided.md` | Usuarios que prefieren estructura — el agente hace preguntas fase por fase | 30-45 min |
| **B: Freeform** | `mode_b_freeform.md` | Usuarios que prefieren contar su historia libremente — el agente escucha y organiza | 20-30 min |

## Cómo Empezar

Al inicio de sesión, cuando el usuario elige "Onboarding", el agente debe:

1. Preguntar: **"¿Prefieres que te guíe con preguntas paso a paso (Modo A) o prefieres contarme tu historia y yo organizo? (Modo B)"**
2. Leer el protocolo del modo elegido
3. Copiar los templates de `templates/` a `private/` si no existen
4. Ejecutar el flujo conversacional
5. Al finalizar, verificar completitud del perfil

## Output Esperado

Al terminar el onboarding, el usuario debe tener:

- [x] `private/perfil_base.md` con todas las secciones obligatorias pobladas
- [x] `private/brand_voice.md` con al menos Identidad Central, Tono Base, y Red Lines
- [x] Confianza de que el framework tiene suficiente contexto para generar CVs de calidad

## Conexión con el Resto del Framework

- El onboarding alimenta el **Confidence Score** (via Power Stack + Trayectoria)
- El onboarding alimenta el **MVP** (via Fortalezas Profesionales)
- El onboarding alimenta el **SAS** (via Value Hierarchy, Motivaciones, Metas)
- Después del onboarding, el usuario puede ir directo a **Screening** o **Proceso Completo**
