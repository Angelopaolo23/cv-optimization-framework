# 🗺️ ROADMAP: CV Optimization Framework

Este roadmap define la evolución del framework desde su núcleo actual hacia una herramienta integral de gestión de carrera técnica y búsqueda activa.

## 🟢 V1: CORE ARCHITECTURE (Estado Actual)

- **Protocolo Maestro:** Definición del System Prompt y reglas de integridad.
- **Cápsula de Contexto:** `perfil_base.md` como fuente de verdad única.
- **Generación de CV:** Optimización basada en "dolores técnicos" de JDs específicas.

---

## 🟡 V1.1: DATA ENTRY & APPLICATION QA (En Progreso)

_Objetivo: Facilitar la ejecución de la postulación._

- **Plantillas Copy-Paste:** Formateo directo para herramientas como CVMaker.cl.
- **Application Support:** Resolución de preguntas de cribado (pre-screening questions) en portales laborales.

_Objetivo: Acompañar al candidato más allá del CV._

- **Interview Playbooks:** Generación de guías de preparación específicas tras obtener una entrevista.
  - _Técnica:_ Preguntas basadas en el stack mencionado en el CV optimizado.
  - _Cultura/Recruiter:_ Cómo vender el perfil "The Bridge" y la experiencia en CGE.
- **Feedback Loop:** Registro de respuestas de empresas para ajustar el `perfil_base.md` (ej: si rechazan por falta de X, subir prioridad a ese aprendizaje).
- **Control de Versiones:** Historial de CVs generados por empresa para coherencia en las entrevistas.

---

## 🟠 V3: RADAR & AUTO-DISCOVERY (Exploración Avanzada)

_Objetivo: Proactividad en la búsqueda de oportunidades._

- **Scraping Estratégico:** Uso del "Culture Radar" para buscar JDs que coincidan con el stack y la filosofía de "The Bridge".
- **Análisis de Mercado en Tiempo Real:** Identificar tendencias en startups de Latam/Global para sugerir nuevas certificaciones o proyectos.
- **Auto-Matching:** Notificaciones sobre ofertas donde el "Confidence Score" sea > 85%.

---

## 🔴 V4: CAREER AGENT & PRODUCTIZATION (Visión Final)

_Objetivo: Transformar la herramienta en un SaaS escalable para terceros._

- **Interfaz Gráfica (Landing & Dashboard):** Migrar de archivos `.md` a una UI moderna y reactiva para facilitar el acceso a usuarios no técnicos.
- **Motor de Inferencia Optimizado:** Implementación de **Gemini 3 Flash** como motor principal, o modelos **Open Source** equivalentes, para asegurar velocidad y eficiencia de costos.
- **Abstracción del Protocolo:** Creación de formularios intuitivos que alimenten el **Master Protocol** sin que el usuario necesite conocer ingeniería de prompts.
- **Application CRM (Solicitud Tracker):** Tablero para gestionar el estado de cada postulación, rastreando fechas, SAS scores y respuestas.
- **Simulador de Entrevistas con Agentes:** Práctica interactiva con roles de Recruiter, CTO y Product Manager basada en el CV específico generado.
- **Estrategia "Build in Public":** Documentación del proceso en LinkedIn/GitHub como diario de desarrollo (**Developer Diary**) para generar tracción y validar el producto.
