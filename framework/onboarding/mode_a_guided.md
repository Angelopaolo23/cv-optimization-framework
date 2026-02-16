# Modo A: Onboarding Guiado

El agente lleva al usuario por 5 fases de preguntas para poblar su perfil completo. Cada fase tiene preguntas obligatorias y opcionales. El tono es conversacional — no es un interrogatorio, es una entrevista colaborativa.

**Filosofía:** El agente actúa como un buen entrevistador que saca lo mejor del candidato. Muchas personas subestiman su experiencia — el agente debe empujar cuando detecta potencial no expresado.

---

## Fase 1: Identidad y Propósito (5-8 min)

**Objetivo:** Poblar sección PERFIL PROFESIONAL de `perfil_base.md`

### Preguntas Obligatorias

1. **"¿A qué te dedicas hoy? En una línea, ¿cómo te describirías profesionalmente?"**
   - Buscamos: Pilar Principal e Identidad Profesional
   - Si la respuesta es vaga ("soy desarrollador"), empujar: "¿De qué tipo? ¿Backend, frontend, full stack? ¿En qué industria?"

2. **"¿Qué te hace diferente de otros profesionales con skills similares?"**
   - Buscamos: Diferenciador
   - Si dice "nada especial", empujar: "¿Tienes una combinación inusual de habilidades? ¿Vienes de otra carrera? ¿Resuelves problemas de una forma particular?"

3. **"¿Qué te mueve? ¿Por qué haces lo que haces?"**
   - Buscamos: Motivación & Ethos
   - No aceptar "me gusta programar" — ir más profundo: "¿Qué parte del trabajo te da energía? ¿Resolver problemas, construir cosas, ayudar personas?"

### Preguntas Opcionales (si hay tiempo)

4. "¿Cómo te ves en 2-3 años?"
   - Buscamos: Metas de carrera (alimenta SAS Dimensión 1)

5. "¿Hay algo que nunca harías profesionalmente? ¿Valores no negociables?"
   - Buscamos: Semilla para Non-Negotiables

### Al Cerrar Esta Fase

Resumir lo capturado y confirmar: "Entonces tu identidad profesional es [X], te diferencia [Y], y te mueve [Z]. ¿Correcto o ajustamos algo?"

---

## Fase 2: Trayectoria y Logros (10-15 min)

**Objetivo:** Poblar TRAYECTORIA PROFESIONAL y detectar Fortalezas Profesionales

### Preguntas Obligatorias

1. **"Cuéntame tu trayectoria profesional. ¿Dónde has trabajado y qué hacías?"**
   - Capturar: empresa, rol, período, contexto
   - Para cada experiencia, preguntar: "¿Cuál fue tu logro más importante ahí?"

2. **"Para cada logro que mencionaste: ¿Qué hiciste específicamente, con qué herramientas, y qué resultado tuvo?"**
   - Buscamos: Impact Bullets con estructura Acción + Contexto + Resultado
   - Si dice "mejoré un proceso", empujar: "¿Cuánto mejoró? ¿De qué a qué? ¿Cuántas personas se beneficiaron?"

3. **"¿Tienes métricas concretas de algún logro? Números, porcentajes, tiempos, dinero."**
   - Buscamos: Métricas verificables para Non-Negotiables
   - Si no tiene números exactos: "Está bien no tener números exactos. ¿Puedes dar una estimación conservadora que estés cómodo defendiendo en una entrevista?"

### Preguntas de Detección de Fortalezas (MVP)

4. **"¿Has liderado equipos o proyectos? ¿Cuántas personas, por cuánto tiempo, qué tipo de decisiones tomabas?"**
   - Buscamos: Liderazgo y Gestión (MVP categoría 1)
   - Incluir: coordinación, mentoría, stakeholder management

5. **"¿Tienes formación o experiencia en negocio? ¿Has trabajado con clientes, definido producto, o tomado decisiones con impacto comercial?"**
   - Buscamos: Experiencia de Industria/Negocio (MVP categoría 2)

6. **"¿Has hecho una transición de carrera o aprendido algo nuevo en poco tiempo? ¿Cambiaste de área, aprendiste un stack nuevo, sacaste certificaciones?"**
   - Buscamos: Velocidad de Aprendizaje (MVP categoría 3)

### Al Cerrar Esta Fase

"Identifiqué [N] experiencias con [M] logros documentados. Tus fortalezas profesionales principales parecen ser [X, Y, Z]. ¿Falta algo importante?"

---

## Fase 3: Stack Técnico (5-8 min)

**Objetivo:** Poblar THE POWER STACK

### Preguntas Obligatorias

1. **"¿Qué tecnologías usas hoy en tu trabajo o proyectos? Nombra las principales."**
   - Capturar por categoría: lenguajes, frontend, backend, DB, cloud, DevOps, otros

2. **"Para cada tecnología: ¿cómo la calificarías del 1 al 5?"**
   - Recordar la escala: 5=experto diario, 4=producción frecuente, 3=independiente, 2=con ayuda, 1=teórico
   - Si alguien se pone 5 en todo, empujar: "¿Podrías mentorar a alguien en [X]? ¿Lo usas a diario para resolver problemas complejos?"

3. **"¿Qué estás aprendiendo o quieres aprender?"**
   - Buscamos: Columna "En Aprendizaje" del Power Stack
   - Esto alimenta SAS Dimensión 4 (Crecimiento Técnico)

### Pregunta Anti-Impostor

4. **"¿Hay algo que usas regularmente pero no consideras 'skill'? A veces damos por hecho cosas que otros no saben hacer."**
   - Empujar: "¿Git avanzado? ¿Testing? ¿CI/CD? ¿Documentación? ¿Debugging? ¿Prompt engineering?"

### Al Cerrar Esta Fase

Mostrar la tabla armada y confirmar niveles. "¿Alguno te parece muy alto o muy bajo?"

---

## Fase 4: Voz y Estilo (5-8 min)

**Objetivo:** Poblar `brand_voice.md`

### Preguntas Obligatorias

1. **"Cuando escribes profesionalmente (emails, documentos, presentaciones), ¿cómo describirías tu estilo?"**
   - Buscamos: Tono Base
   - Opciones guía: "¿Formal o casual? ¿Técnico o accesible? ¿Directo o diplomático?"

2. **"Si tuvieras que presentarte en 3 líneas a un reclutador, ¿qué dirías?"**
   - Buscamos: Semilla para Killer Summary
   - Si le cuesta: "Imagina que tienes 10 segundos en un elevador con el hiring manager de tu trabajo ideal. ¿Qué le dices?"

3. **"¿Hay cosas que NUNCA dirías en un CV o entrevista? ¿Palabras, afirmaciones, estilos que te incomodan?"**
   - Buscamos: Red Lines de brand_voice.md
   - Ejemplos: "¿Te molestan los superlativos? ¿El lenguaje corporativo vacío? ¿Las promesas vagas?"

### Pregunta Opcional

4. "¿Tienes CVs anteriores que funcionaron bien? ¿Qué te gustó de ellos?"
   - Buscamos: Patrones validados para brand_voice.md

### Al Cerrar Esta Fase

"Tu voz es [X]: [resumen de 1 línea]. ¿Suena como tú?"

---

## Fase 5: Prioridades y Límites (5 min)

**Objetivo:** Poblar VALUE HIERARCHY y NON-NEGOTIABLES

### Preguntas Obligatorias

1. **"¿Qué es lo más importante para ti en un trabajo? Ordena: compensación, aprendizaje, autonomía, impacto, equilibrio, equipo, ubicación."**
   - Buscamos: Value Hierarchy (top 5)

2. **"¿Qué tipo de empresas o roles descartarías sin pensarlo?"**
   - Buscamos: Non-Negotiables de posicionamiento
   - Empujar: "¿Hay industrias, culturas, o prácticas que son deal-breaker para ti?"

3. **"¿Hay algo sobre tu experiencia que nunca quieras exagerar o que sea sensible?"**
   - Buscamos: Non-Negotiables de experiencia
   - Ejemplo: "¿Skills en progreso que no quieres que se presenten como dominados?"

### Al Cerrar Esta Fase

"Tus prioridades son [1, 2, 3, 4, 5] y tus límites absolutos son [X, Y, Z]. ¿Correcto?"

---

## Cierre del Onboarding

### Verificación de Completitud

Antes de cerrar, verificar que están pobladas todas las secciones obligatorias:

- [ ] PERFIL PROFESIONAL: Pilar, Identidad, Diferenciador, Motivación
- [ ] TRAYECTORIA: Al menos 1 experiencia con logros en formato Impact Bullet
- [ ] POWER STACK: Al menos 5 tecnologías con niveles
- [ ] FORTALEZAS PROFESIONALES: Al menos 1 de las 3 categorías (Liderazgo, Negocio, Aprendizaje)
- [ ] NON-NEGOTIABLES: Al menos restricciones de métricas
- [ ] VALUE HIERARCHY: Top 5 valores
- [ ] BRAND VOICE: Tono base + Red Lines como mínimo

### Mensaje de Cierre

> "Tu perfil está listo. Tienes [N] experiencias documentadas, [M] tecnologías en tu stack, y tus fortalezas profesionales están capturadas para el scoring.
>
> Ahora puedes usar el framework para optimizar CVs. ¿Quieres hacer un Screening Rápido de alguna oferta para probar?"

### Recomendación de Presencia Profesional

Si durante el onboarding se detecta que el usuario:
- No tiene LinkedIn optimizado
- No tiene GitHub con proyectos visibles
- No tiene portafolio o presencia online relevante para su campo

El agente debe sugerir (sin presionar):

> "Un dato: muchos reclutadores revisan LinkedIn y GitHub antes de leer el CV. Si quieres que el framework te ayude a optimizar tu presencia en estas plataformas en una sesión futura, podemos hacerlo. No es obligatorio, pero multiplica el impacto del CV."

Esta sugerencia se hace **una vez**, al cierre del onboarding. No se repite en sesiones posteriores a menos que el usuario lo pida.
