# ROADMAP: CV Optimization Framework

Este roadmap define la evolución del framework desde su núcleo actual hacia una herramienta integral de gestión de carrera técnica y búsqueda activa.

> **Nota para el agente planificador:** Este roadmap contiene tanto el historial de lo construido como la visión futura. Las versiones V1-V1.5 están implementadas o en progreso. V2+ son visión documentada para futuras sesiones de planificación.

---

## V1: CORE ARCHITECTURE — Completado

**Lo que se construyó:**
- Protocolo Maestro (`framework_protocol.md`): System prompt con 7 fases, reglas de integridad, modos de operación
- Cápsula de Contexto (`perfil_base.md`): Fuente de verdad única del candidato
- Brand Voice (`brand_voice.md`): Especificaciones de voz y estilo
- Scoring Protocol (`scoring_protocol.md`): Fórmulas de Confidence Score + SAS
- Generación de CV: Optimización basada en pain points de JDs específicas
- Modo Screening: Evaluación rápida en Fases 1-2
- 6 outputs generados: 3 CVs completos (Fintoc, KPaz, Xepelin) + 3 screenings (KPaz, Falabella, Strider)

---

## V1.1: DATA ENTRY & APPLICATION QA — Parcialmente Completado

_Objetivo: Facilitar la ejecución de la postulación._

**Completado:**
- Plantillas Copy-Paste: Formateo directo para herramientas como CVMaker.cl (Ready-to-Copy Sections en output)
- Application Support (Fase 4): Resolución de preguntas de cribado en portales laborales
- Templates para nuevos usuarios (`templates/`)

**Pendiente:**
- Interview Playbooks: Guías de preparación específicas post-entrevista
  - _Técnica:_ Preguntas basadas en el stack del CV optimizado
  - _Cultura/Recruiter:_ Cómo vender el perfil y la experiencia
- Feedback Loop estructurado: Registro de respuestas de empresas para ajustar perfil_base.md
- Control de Versiones de outputs (convención `_v[N]` definida, falta implementar INDEX.md)

---

## V1.5: FRAMEWORK HARDENING — En Progreso

_Objetivo: Madurar el framework basado en la auditoría de los primeros 6 outputs reales._

**Contexto:** Después de generar 6 outputs, se detectaron problemas de filosofía (tono de auditor), bugs (walkthrough desactualizado), puntos ciegos en scoring, templates vacíos, y un feedback loop que nunca se ejecutó. Esta versión corrige todo eso.

**Completado (Fases 0-2):**
- Cambio de filosofía: De auditor a abogado defensor del candidato
- Principios reescritos: Candidate Advocacy, Integridad Estratégica, Anti Síndrome del Impostor
- Scoring reinterpretado: "Fortaleza de posición" en vez de "probabilidad de contratación"
- Walkthrough reescrito con tono pro-usuario y sección sobre JDs como wish-lists
- AGENT_START actualizado: Mapa de documentos completo, modo Onboarding, versionado de outputs
- Limpieza: `implementation_plan.md` eliminado, `projects/` movido fuera del repo

**Pendiente (Fases 3-11):**
- **Fase 3 — Scoring Overhaul:** Sistema de clasificación de requisitos (Tipo A: filtro duro, B: obligatorio estándar, C: deseable real, D: deseable inflado). Estado `Transferible (0.7)` para skills análogos. Desacoplar valores personales hardcodeados del SAS
- **Fase 4 — Templates enriquecidos:** Agregar ejemplos concretos a `perfil_base.template.md` y `brand_voice.template.md`
- **Fase 5 — Versionado + INDEX:** Crear `private/outputs/INDEX.md` con tabla de todos los outputs
- **Fase 6 — Learnings reales:** Poblar `private/learnings.md` con los 4 learnings de la auditoría (contradicción KPaz, métrica inventada Xepelin, fases nunca ejecutadas, scoring sin filtros duros)
- **Fase 7 — Application Tracker:** Crear `private/application_tracker.md` (o `session-state.json`, ver nota abajo) con historial de las 6 postulaciones
- **Fase 8 — Onboarding:** Crear `framework/onboarding/` con modo guiado (AI pregunta) y modo libre (usuario guía)
- **Fase 9 — Patrones públicos:** Crear `framework/common_patterns.md` con anti-patrones y patrones generalizados para forkers
- **Fase 10 — Este roadmap:** Actualizar visión (completado con este documento)
- **Fase 11 — Sync CLAUDE.md + README:** Reflejar estructura final en documentación raíz

> **Decisión arquitectónica pendiente — JSON vs Markdown para tracker:**
> Se evaluó usar `session-state.json` en vez de markdown para el application tracker. Ventajas: menos tokens al parsear (JSON es nativo para LLMs), ediciones quirúrgicas sin romper formato, extensibilidad trivial. Approach recomendado: JSON como source of truth + render markdown bajo demanda para lectura humana. Evaluar en implementación de Fase 7.

---

## V2: PLATFORM ADAPTERS & LEARNING PATHS — Visión

_Objetivo: Que el framework se adapte a múltiples plataformas de CV y acompañe el desarrollo profesional del usuario._

### 2.1 — Adaptadores por Plataforma de CV

**Motivación:** Hoy el framework genera "Ready-to-Copy Sections" genéricos, pero cada plataforma de creación de CV (CVMaker.cl, GetOnBoard, LinkedIn, etc.) tiene campos específicos con restricciones de longitud, formato, y estructura distintas. El usuario pierde tiempo adaptando manualmente.

**Visión:** Un sistema de adaptadores/conectores donde cada plataforma tiene un schema de inputs definido, y el agente mapea automáticamente su output a ese schema.

**Por qué es factible:**
- Los campos entre plataformas son ~95% iguales (nombre, resumen, experiencia, skills, educación)
- Las diferencias son predecibles: longitud máxima de summary, número de bullets por experiencia, campos custom, formato de fechas
- Un adaptador es esencialmente un archivo de configuración (JSON schema), no código complejo
- El agente ya genera todo el contenido — solo falta el "último tramo" de formateo específico

**Modelo de contribución comunitaria:**
- Un usuario configura un adaptador para una plataforma nueva (ej: "campos de Indeed Chile")
- Lo sube como PR o a través de un mecanismo simplificado (formulario → PR automático)
- Se revisa, se aprueba, y queda disponible para todos los usuarios del framework
- Esto tiene más tracción que contribuir learnings porque el valor es inmediato y concreto

**Implementación conceptual:**
```
framework/adapters/
├── README.md                    # Cómo crear un adaptador
├── schema.json                  # Schema base que todos comparten
├── cvmaker_cl.json              # Adaptador CVMaker.cl
├── getonboard.json              # Adaptador GetOnBoard
└── linkedin.json                # Adaptador LinkedIn
```

Cada adaptador define:
- Campos disponibles y sus restricciones (maxLength, required, format)
- Mapping desde el output estándar del framework a los campos de la plataforma
- Notas sobre particularidades ("GetOnBoard no tiene campo de summary separado, va en About")

### 2.2 — Learning Paths (Roadmaps de Aprendizaje)

**Motivación:** Cuando el framework detecta un gap en una postulación (ej: "piden Winston.js y no lo tienes"), hoy solo dice "gap no mitigable" o "mencionar en aprendizaje activo". No propone un camino para cerrarlo.

**Visión:** El agente propone plazos razonables y un mini-roadmap para aprender lo que falta, conectado directamente con postulaciones reales.

**Cómo funciona:**
1. El scoring detecta gap en skill X para la postulación Y
2. El agente evalúa: ¿es un gap "aprendible" en tiempo razonable? (ej: Winston.js sí, 5 años de ML no)
3. Si es aprendible, propone: "Con 2 semanas de práctica enfocada, podrías subir Winston de nivel 1 a nivel 3. Esto mejoraría tu Confidence Score de 65% a 78%"
4. Incluye recursos sugeridos y un checkpoint verificable
5. El progreso se registra en el perfil base y se refleja en futuros scorings

**Valor estratégico:**
- Transforma gaps de "debilidades" en "oportunidades con timeline"
- Conecta el aprendizaje con resultados concretos (mejor score en postulaciones reales)
- Motiva al usuario: no es "estudia por estudiar", es "aprende esto y tu caso para Empresa X mejora un 13%"
- Es el puente natural entre "framework de CV" y "Career Companion"

**Relación con el scoring:**
- El Confidence Score actual es estático (snapshot). Con learning paths se vuelve dinámico: "hoy 65%, en 2 semanas podrías estar en 78%"
- Esto cambia la conversación de "no cumples" a "cuánto te tomaría cumplir"

---

## V3: RADAR, AUTO-DISCOVERY & COMMUNITY — Visión

_Objetivo: Proactividad en la búsqueda y construcción de comunidad._

### 3.1 — Auto-Discovery de Ofertas

- **Scraping Estratégico:** Usar el Culture Radar para buscar JDs que coincidan con el stack y las metas del usuario
- **Análisis de Mercado:** Identificar tendencias en startups Latam/Global para sugerir nuevas certificaciones o proyectos
- **Auto-Matching:** Alertas sobre ofertas donde el Confidence Score sea > 75% y SAS > 70%
- **Integración con plataformas:** Conectar con APIs de GetOnBoard, LinkedIn Jobs, etc. para pull automático de JDs

### 3.2 — Community Contributions & Interfaz

**Motivación:** El framework vive en GitHub, lo que limita las contribuciones a personas que saben usar git, PRs, y agentes de IA. Para escalar la comunidad, necesitamos reducir la barrera de entrada.

**Niveles de contribución (de menor a mayor fricción):**

1. **GitHub Issues/Discussions:** Reportar patrones o anti-patrones en texto libre (barrera: tener cuenta GitHub)
2. **Formulario web simple:** Google Form o similar que genera un Issue automáticamente (barrera: ninguna)
3. **PR asistido por agente:** El usuario describe su learning, el agente genera el PR formateado (barrera: mínima)
4. **Interfaz web dedicada:** Dashboard donde los usuarios ven patrones, votan los más útiles, y contribuyen los suyos (barrera: ninguna, pero requiere infraestructura)

**Estrategia de validación:**
- Empezar con nivel 1-2 (cero costo de desarrollo)
- Medir: ¿la gente contribuye? ¿Qué tipo de contenido aportan?
- Si hay tracción, construir nivel 3-4
- La interfaz visual es parte de V4 (SaaS), no de V3

**Qué se contribuye:**
- Patrones de JDs por industria (fintech pide X, startups piden Y)
- Anti-patrones de postulación (errores comunes que el framework debería prevenir)
- Adaptadores de plataforma (ver V2.1)
- Learnings generalizados y anonimizados

---

## V4: CAREER COMPANION & PRODUCTIZATION — Visión

_Objetivo: Evolucionar de "framework de CV" a plataforma integral de gestión de carrera._

### La Visión Grande

El perfil base (`perfil_base.md`) no es solo para CVs. Es la **single source of truth profesional** del usuario — un asset que sirve para:
- Optimización de CV (lo que ya hace el framework)
- Actualización de LinkedIn
- Preparación de entrevistas
- Negociaciones salariales ("estos son mis logros cuantificados")
- Networking ("este es mi elevator pitch actualizado")
- Autoevaluación de carrera ("¿dónde estoy vs. dónde quiero estar?")

**El Career Companion es un agente que:**
- Mantiene tu perfil profesional siempre actualizado
- Te propone qué aprender basado en el mercado real (no en teoría)
- Lleva registro de tus postulaciones, entrevistas, y resultados
- Cuantifica tu progreso (skills que subieron de nivel, gaps cerrados, entrevistas conseguidas)
- Te da una visión clara de tu carrera: dónde estás, hacia dónde vas, qué te falta

**Filosofía:** Siempre es bueno tener una cuantificación escrita de tu carrera laboral. Si alguien te pregunta tus skills, tecnologías, logros — lo tienes a mano, estructurado, y actualizado. A través del agente creas un perfil fuerte de información que puedes aprovechar para tus propios fines.

### 4.1 — Interfaz Gráfica (Dashboard)

- Migrar de archivos `.md`/`.json` a una UI moderna y reactiva
- Accesible para usuarios no técnicos (no necesitan saber git ni markdown)
- Visualización del pipeline de postulaciones, scores, y progreso
- El dashboard es la puerta de entrada a toda la funcionalidad

### 4.2 — Motor de Inferencia

- Modelos optimizados para costo/velocidad: Claude Haiku, Gemini Flash, o modelos open source
- Opus/Sonnet para planificación y análisis profundo
- Haiku/Flash para tareas repetitivas (formatting, adaptadores, updates de estado)
- El usuario no elige modelo — el sistema rutea automáticamente según la tarea

### 4.3 — Application CRM

- Evolución del `session-state.json` a un tablero visual completo
- Pipeline: Screening → CV Generado → Postulado → En Proceso → Oferta → Cerrado
- Métricas de conversión (% de screenings que pasan a CV, % de CVs que consiguen entrevista)
- Historial completo con learnings por postulación
- Precursor: el application tracker de V1.5 es el MVP de esto

### 4.4 — Simulador de Entrevistas

- Práctica interactiva con agentes que simulan Recruiter, CTO, Product Manager
- Basado en el CV específico generado y la investigación de empresa
- Feedback estructurado después de cada simulación

### 4.5 — Abstracción del Protocolo

- Formularios intuitivos que alimentan el protocolo sin necesidad de conocer prompt engineering
- El onboarding de V1.5 es el precursor de esto

### 4.6 — Build in Public

- Documentación del proceso en LinkedIn/GitHub como Developer Diary
- Generar tracción y validar el producto antes de invertir en desarrollo pesado

---

## Notas para Futuras Sesiones de Planificación

### Sobre prioridades de implementación
- V2.1 (Adaptadores) es la feature con mejor ratio valor/esfuerzo después de V1.5
- V2.2 (Learning Paths) es la feature que más diferencia al producto de ser "otro optimizador de CV"
- V3.2 (Community) necesita validación antes de inversión — empezar con GitHub Issues
- V4 requiere tracción demostrada antes de construir infraestructura

### Sobre arquitectura de datos
- Evaluado `session-state.json` vs markdown para datos estructurados (tracker, estado de sesión)
- JSON recomendado como source of truth: menos tokens, menos errores de parsing, ediciones quirúrgicas
- Markdown como capa de presentación bajo demanda
- Esto aplica para: application tracker, adaptadores de plataforma, índice de outputs, y eventualmente el perfil base mismo

### Punto a evaluar: formato de archivos según su especialidad
- **Hipótesis:** Usar markdown para todo puede ser ineficiente en tokens. Cada archivo debería usar el formato más adecuado a su contenido — formatos LLM-friendly como JSON donde corresponda.
- **Evaluación inicial:** Markdown es eficiente para instrucciones y prosa (formato nativo de LLMs). La ineficiencia está en usar markdown para datos tabulares que se parsean repetidamente. No es urgente migrar ahora — el impacto real en tokens es marginal hasta que el framework escale.
- **Regla propuesta para futuras iteraciones:** cada archivo debería usar el formato que mejor se ajusta a su contenido — markdown para prosa/instrucciones, JSON para datos estructurados y configuración, YAML frontmatter para archivos mixtos (prosa + metadata).
- **Candidatos a migración (cuando se evalúe):** application tracker → JSON, INDEX de outputs → JSON, Power Stack/Skills → YAML frontmatter en perfil_base, adaptadores de plataforma → JSON schemas
- **No migrar:** framework_protocol, walkthrough, AGENT_START, brand_voice, learnings (son prosa, markdown es óptimo)
- **Status:** A considerar en futuras iteraciones, no prioritario para V1.5

### Sobre el modelo de contribución comunitaria
- Los adaptadores de plataforma (V2.1) son el contenido más probable que la gente quiera contribuir — valor inmediato y concreto
- Los learnings generalizados (V1.5 Fase 9) son más difíciles de motivar pero más valiosos a largo plazo
- La barrera técnica (git, PRs) filtra contribuidores innecesariamente — explorar mecanismos más accesibles
- No asumir que la gente contribuirá — diseñar para que el framework sea valioso sin contribuciones externas, y las contribuciones sean bonus

### Última actualización
2026-02-13 — Sesión de refactorización V1.5 + brainstorming de visión
