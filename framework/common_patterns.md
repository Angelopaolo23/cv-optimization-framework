# 🔁 Patrones Comunes del Framework

Este documento captura patrones observados en múltiples sesiones de optimización de CV. Está pensado para usuarios que ya tienen experiencia con el framework y quieren mejorar su uso, y para personas que forkearon el repo y quieren aprender de casos reales.

**Fuente:** Combinación de aprendizajes documentados en `../private/learnings.md` + observaciones de diseño del protocolo.

---

## Anti-Patrones (Lo Que Falla)

### ❌ Anti-Patrón 1: El CV Sin Retrospectiva

**Qué pasa:** Se genera el CV completo (Fases 1-5) y se cierra la sesión sin ejecutar la Fase 6 (Brand Coherence Audit) ni la Fase 7 (Retrospectiva).

**Por qué es un problema:** Los errores se repiten en el siguiente CV. Las métricas incorrectas no se detectan. Los patrones de voz que no funcionaron no se documentan. El framework no evoluciona.

**Señales de que está pasando:**
- El agente dice "aquí está tu CV optimizado" sin hacer preguntas de retrospectiva
- No hay entrada en `../private/learnings.md` después de la sesión
- El Brand Coherence Audit no aparece en el output

**Solución:** El checklist de AGENT_START.md incluye explícitamente que Fases 6 y 7 son obligatorias. Si el usuario quiere cerrar antes, el agente debe al menos capturar el learning más importante en 2 líneas.

---

### ❌ Anti-Patrón 2: La Métrica Inventada

**Qué pasa:** El agente genera un número específico ("reduje los tiempos en un 40%", "ahorré $50k") que no tiene respaldo en `perfil_base.md`.

**Por qué es un problema:** Si el candidato llega a entrevista, no puede defender el número. El reclutador puede preguntar "¿cómo mediste eso?". La credibilidad se destruye con un solo dato inventado.

**Señales de que está pasando:**
- Un porcentaje exacto aparece en el CV sin que el usuario lo haya mencionado
- La métrica es "redonda" (exactamente 30%, 50%, 2x) sin contexto de cómo se midió
- Buscar en `perfil_base.md` y el número no aparece en ningún lado

**Solución:** Si no hay métrica verificable, usar lenguaje cualitativo preciso: "redujo significativamente", "eliminó completamente", "procesó en minutos lo que antes tomaba horas". Es menos impresionante pero 100% defendible.

**Regla:** Si el usuario quiere usar un número aproximado, debe estar cómodo diciéndole a un entrevistador "estimé conservadoramente basado en [fuente]".

---

### ❌ Anti-Patrón 3: El Score Inflado

**Qué pasa:** El Confidence Score sale alto (80%+) porque el agente clasificó habilidades parciales o transferibles como "cumple completo", sin usar el estado Transferible (0.7) ni la clasificación A-D.

**Por qué es un problema:** El candidato entra a un proceso con expectativas incorrectas. Si el score real debería ser 50%, ir preparado como si fuera 80% genera desalineación en la entrevista técnica.

**Señales de que está pasando:**
- Scores de 88%+ en roles donde el candidato claramente tiene gaps significativos
- No hay tabla de clasificación A-D en el output
- El estado "Transferible" nunca aparece (todos son ✅ o ❌)

**Solución:** Seguir el Paso 1 del `scoring_protocol.md` fielmente. Cuando hay duda de clasificación de un requisito, errar hacia Tipo C en vez de Tipo B. Cuando hay duda de match, usar Transferible (0.7) en vez de Cumple (1.0).

---

### ❌ Anti-Patrón 4: El JD Tomado Literalmente

**Qué pasa:** El agente trata todos los requisitos del JD como igualmente importantes y penaliza al candidato por no cumplir ítems inflados o contradictorios.

**Por qué es un problema:** Un JD típico tiene 3-5 requisitos reales y 10-12 de relleno. Tratar los 15 igual significa que un candidato que cumple los 5 críticos parece 33% calificado cuando debería ser 100%.

**Señales de que está pasando:**
- El score es bajo pero el candidato tiene todas las habilidades del stack principal
- Se están penalizando certificaciones opcionales o experiencia en startup + enterprise + consulting "todo a la vez"
- Piden 8 años de experiencia en una tecnología que tiene 6 años de existencia

**Solución:** Clasificación Tipo A-D antes de calcular cualquier score. Ver heurísticas en `scoring_protocol.md:Paso 1`.

---

### ❌ Anti-Patrón 5: El Perfil Invisible

**Qué pasa:** El usuario tiene fortalezas no-técnicas significativas (liderazgo, experiencia de negocio, transiciones de carrera) pero `perfil_base.md` solo tiene el Power Stack técnico. El score refleja solo el match técnico y el MVP es 0.

**Por qué es un problema:** Para muchos roles, especialmente en startups y empresas medianas, el candidato que lidera y entiende el negocio es más valioso que el técnico puro. Si esa información no está documentada, el framework no puede argumentarla.

**Solución:** Completar la sección FORTALEZAS PROFESIONALES en `perfil_base.md`. Ver guía en el template. Si estás en onboarding, el Modo A (Fase 2) tiene preguntas específicas para capturar esta información.

---

## Patrones (Lo Que Funciona)

### ✅ Patrón 1: El Killer Summary Contextualizado

**Qué es:** Adaptar el Killer Summary para cada empresa en lugar de usar siempre el Universal del perfil.

**Cómo funciona:** El Universal es la base. Para cada postulación, ajustar 1-2 líneas que resuenen con el pain point específico del empleador detectado en Fase 1. Mismo candidato, distinto ángulo de entrada.

**Ejemplo:**
- Universal: "Full Stack Developer con background en negocios..."
- Para fintech: "...con experiencia en operaciones financieras reguladas y automatización de procesos críticos"
- Para startup de producto: "...con foco en shipping rápido y conexión entre técnico y roadmap de producto"

---

### ✅ Patrón 2: Los Gaps Como Narrative Asset

**Qué es:** En vez de esconder gaps técnicos, convertirlos en parte del argumento del candidato.

**Cómo funciona:** Un gap técnico + evidencia de aprendizaje rápido = argumento para contratación. "No tengo X pero adoptié Y y Z en 3 meses mientras trabajaba full-time" es más convincente que pretender que X no existe.

**Cuándo aplica:** Gaps en Tipo B o C donde hay evidencia de velocidad de aprendizaje (MVP categoría 3). No aplica para Tipo A (filtros duros) — esos hay que resolverlos o ser honesto sobre el desafío.

---

### ✅ Patrón 3: La Transferibilidad Argumentada

**Qué es:** No solo marcar una habilidad como Transferible en el score — construir el argumento explícito de por qué transfiere.

**Cómo funciona:** En el Gap Analysis del output, incluir: "No tengo X, pero tengo Y que es equivalente porque [razón específica]." Preparar al candidato para decir esto en entrevista.

**Ejemplo:** "No tengo experiencia en fintech pero trabajé 3 años en operaciones de distribución eléctrica — ambos son entornos regulados con datos sensibles, procesos críticos de negocio, y tolerancia cero a errores en producción."

---

### ✅ Patrón 4: El Screening Como Filtro Temprano

**Qué es:** Hacer Screening Rápido (Fases 1-2) antes de invertir en el Proceso Completo.

**Cómo funciona:** 5 minutos de screening revelan si vale la pena. Un SAS < 50% con Confidence < 40% es señal clara de no proceder. Evita gastar 45 minutos en un CV que no debería existir.

**Cuándo hacer Proceso Completo directamente:** Cuando el screening rápido da Confidence ≥ 60% y SAS ≥ 65%, o cuando hay razón estratégica específica para postular a pesar de los scores.

---

### ✅ Patrón 5: El Perfil Base Como Documento Vivo

**Qué es:** Actualizar `perfil_base.md` después de cada CV, no solo al inicio.

**Cómo funciona:** En la Fase 7, cuando el candidato articula algo nuevo sobre su experiencia (un logro que nunca había documentado, un skill que subió de nivel), agregarlo inmediatamente. El perfil debe crecer con cada sesión.

**Por qué importa:** La segunda postulación siempre es mejor que la primera porque el perfil está más completo. Si nunca se actualiza, la segunda es igual de incompleta.

---

## Patrones por Tipo de Rol

### Para Roles Técnicos Puros (Backend, Frontend, Data)
- Priorizar Power Stack sobre Trayectoria narrativa
- Impact Bullets con lenguaje técnico específico (no "usé herramientas de backend")
- SAS Dimensión 4 (Crecimiento Técnico) tiene peso alto — asegurarse de que hay match

### Para Roles Híbridos (Tech Lead, Product Engineer, DevOps)
- El MVP de Liderazgo/Gestión tiene impacto significativo en el score
- Balance 50/50 técnico-negocio en el Killer Summary
- Destacar experiencia de coordinación aunque no fuera formal

### Para Roles de Producto/Estrategia
- SAS importa más que Confidence — el fit estratégico es lo primero
- MVP de Industria/Negocio es crítico
- Habilidades técnicas son contexto, no el argumento principal

### Para Roles en Startups Early-Stage
- Clasificar más requisitos como Tipo C o D (wishlist extensa es señal de startup)
- El patrón "aprendo rápido" es más valioso que el match técnico exacto
- Cultura y autonomía (SAS Dimensiones 3 y 5) son cruciales — un 0 en cultura es deal-breaker
