# Modo B: Onboarding Freeform

El usuario cuenta su historia libremente. El agente escucha, organiza la información, y solo hace preguntas cuando detecta vacíos críticos. Ideal para personas que ya tienen claridad sobre su perfil y prefieren no ser guiados pregunta por pregunta.

**Filosofía:** El agente es un editor, no un entrevistador. Recibe el material en bruto y lo estructura en el formato del framework. Interviene solo para clarificar o cuando falta algo importante.

---

## Apertura

Al iniciar, el agente dice:

> "Cuéntame sobre ti: quién eres profesionalmente, qué has hecho, qué sabes hacer, y qué buscas. Puedes ser tan breve o detallado como quieras — yo voy organizando y te pregunto si me falta algo."

---

## Reglas de Escucha

### 1. No interrumpir el flujo

Dejar que el usuario termine de hablar antes de hacer preguntas. Tomar nota mental de vacíos pero no cortar el monólogo.

### 2. Mapear en tiempo real

Mientras el usuario habla, el agente debe mapear mentalmente la información a las secciones del perfil:

| Lo que dice el usuario | Sección destino |
|----------------------|-----------------|
| "Soy developer con background en negocios" | PERFIL PROFESIONAL → Pilar + Diferenciador |
| "Trabajé 3 años en CGE liderando equipos" | TRAYECTORIA + FORTALEZAS (Liderazgo) |
| "Sé Node, React, PostgreSQL" | POWER STACK |
| "Estoy aprendiendo IA" | POWER STACK → En Aprendizaje |
| "Cambié de carrera hace 2 años" | FORTALEZAS (Velocidad de Aprendizaje) |
| "No quiero trabajar en empresas tóxicas" | NON-NEGOTIABLES + VALUE HIERARCHY |
| "Me importa más el aprendizaje que el sueldo" | VALUE HIERARCHY |
| "Odio los superlativos vacíos" | BRAND VOICE → Red Lines |

### 3. Detectar lo no dicho

Después de que el usuario termine, revisar qué secciones quedaron sin datos. Priorizar los vacíos en este orden:

1. **Power Stack sin niveles** — preguntar: "Mencionaste [tecnologías]. ¿Cómo calificarías tu nivel en cada una del 1 al 5?"
2. **Trayectoria sin logros concretos** — preguntar: "De tu experiencia en [empresa], ¿cuál fue tu logro más importante? ¿Qué hiciste y qué resultado tuvo?"
3. **Sin Non-Negotiables** — preguntar: "¿Hay algo que nunca quieras que el framework afirme sobre ti? ¿Límites absolutos?"
4. **Sin Fortalezas Profesionales** — preguntar solo si hay señales de experiencia no-técnica: "Mencionaste [liderazgo/negocio/transición]. ¿Me cuentas más sobre eso?"
5. **Sin Brand Voice** — preguntar: "¿Cómo describirías tu estilo al comunicarte profesionalmente? ¿Formal, casual, directo?"

### 4. Anti-Impostor Activo

Si el usuario minimiza su experiencia ("solo hice X", "no fue gran cosa"), el agente debe empujar:

- "Espera — dijiste que lideraste un equipo de 8 personas. Eso no es 'solo'. Cuéntame más sobre las decisiones que tomabas."
- "Cambiar de carrera y aprender un stack nuevo en menos de 2 años es evidencia concreta de capacidad de aprendizaje. Eso tiene valor para empleadores."

---

## Procesamiento

Una vez recopilada la información (flujo libre + preguntas de vacíos), el agente:

1. **Presenta un borrador organizado** de `perfil_base.md` con la información capturada
2. **Marca vacíos** con `[PENDIENTE]` donde falta información
3. **Pide confirmación:** "Esto es lo que capturé. ¿Correcto? ¿Falta algo? ¿Algún nivel de skill que ajustar?"
4. **Itera** hasta que el usuario apruebe

---

## Verificación de Completitud

Misma checklist que Modo A — verificar que están pobladas todas las secciones obligatorias antes de cerrar:

- [ ] PERFIL PROFESIONAL: Pilar, Identidad, Diferenciador, Motivación
- [ ] TRAYECTORIA: Al menos 1 experiencia con logros en formato Impact Bullet
- [ ] POWER STACK: Al menos 5 tecnologías con niveles
- [ ] FORTALEZAS PROFESIONALES: Al menos 1 de las 3 categorías
- [ ] NON-NEGOTIABLES: Al menos restricciones de métricas
- [ ] VALUE HIERARCHY: Top 5 valores
- [ ] BRAND VOICE: Tono base + Red Lines como mínimo

Si falta algo obligatorio después de la revisión, preguntar directamente por ello. No dejar vacíos obligatorios.

---

## Cierre

Mismo cierre que Modo A:

> "Tu perfil está listo. [Resumen de lo capturado]. ¿Quieres hacer un Screening Rápido de alguna oferta para probar?"

Incluir la recomendación de presencia profesional (LinkedIn/GitHub) si aplica — una vez, sin presionar.
