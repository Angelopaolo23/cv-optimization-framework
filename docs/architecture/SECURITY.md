# Seguridad y Anti-Abuse — Career Companion Platform

> **Estado:** En definición.
> **Última actualización:** 2026-02-22

---

## Modelo de Amenazas

### Amenaza Principal: Abuso del Agente LLM

**Escenario:** Un usuario paga $15/mes y usa el agente como chatbot general (programación, tareas escolares, conversación libre) en vez de para career optimization. Costo real del usuario: potencialmente $50-100+/mes en tokens.

**Variantes:**
1. **Uso off-topic legítimo:** El usuario simplemente no sabe que el agente es especializado y le pide cosas genéricas
2. **Jailbreak intencional:** El usuario intenta saltar las instrucciones del sistema para usar el modelo sin restricciones
3. **Prompt injection via JD:** Un JD malicioso contiene instrucciones ocultas para manipular al agente

### Otras Amenazas

| Amenaza | Riesgo | Impacto |
|---|---|---|
| Scraping masivo de la API | Medio | Costos inflados, DoS |
| Acceso a datos de otros usuarios | Alto | Breach de privacidad |
| Exfiltración del system prompt | Bajo | Pérdida de IP (menor, el framework es open source) |
| Inyección SQL via inputs del usuario | Bajo | Supabase RLS + Django ORM mitigan |

---

## Arquitectura de Seguridad: Defensa en Profundidad

```
Usuario
  │
  ▼
┌─────────────────────────────────────┐
│  Capa 1: Rate Limiting (Vercel/CF) │  ← Requests/min por IP y por usuario
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│  Capa 2: Auth + Quotas (Django)    │  ← JWT de Supabase, quotas por plan
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│  Capa 3: Intent Classifier         │  ← GPT-4o-mini: ¿es career-related?
│  (modelo barato, ~0.001 USD/req)   │
└──────────────┬──────────────────────┘
               │ Solo si pasa
               ▼
┌─────────────────────────────────────┐
│  Capa 4: Azure Content Safety      │  ← Detecta jailbreak, contenido dañino
└──────────────┬──────────────────────┘
               │ Solo si pasa
               ▼
┌─────────────────────────────────────┐
│  Capa 5: Agente (Semantic Kernel)  │  ← System prompt + plugins restringidos
│  con modelo principal (GPT-4o)     │
└─────────────────────────────────────┘
```

### Capa 1: Rate Limiting

- **Por IP:** Max 60 requests/minuto (previene scraping)
- **Por usuario autenticado:** Max 30 mensajes de chat/hora, 10 scorings/día, 5 CVs/día
- **Implementación:** Redis en Django middleware, o Vercel Edge Middleware para el frontend

### Capa 2: Auth + Quotas por Plan

```python
# Ejemplo de quotas por plan
PLAN_QUOTAS = {
    "free": {
        "messages_per_day": 20,
        "scorings_per_day": 3,
        "cv_generations_per_month": 2,
        "max_tokens_per_message": 2000,
    },
    "pro": {  # $15/mes
        "messages_per_day": 200,
        "scorings_per_day": 20,
        "cv_generations_per_month": 20,
        "max_tokens_per_message": 4000,
    }
}
```

- Tracking en tabla `usage` de Supabase
- El backend verifica quotas ANTES de llamar al agente
- Si se excede la quota, respuesta inmediata sin costo de tokens

### Capa 3: Intent Classifier (Guardian)

El componente más importante para controlar costos. Un modelo barato que decide si el mensaje del usuario es relevante antes de pasarlo al modelo caro.

```python
# Pseudocódigo del Intent Classifier
SYSTEM_PROMPT_CLASSIFIER = """
Eres un clasificador de intención. Analiza el mensaje del usuario y clasifícalo:

ALLOWED: El mensaje está relacionado con:
- Optimización de CV o perfil profesional
- Evaluación de ofertas de trabajo
- Preguntas sobre carrera profesional
- Onboarding de perfil
- Gestión de postulaciones
- Preparación de entrevistas
- Skills y desarrollo profesional

BLOCKED: El mensaje NO está relacionado con carrera/empleo:
- Programación general, tareas escolares
- Conversación casual no relacionada
- Solicitudes de contenido creativo no profesional
- Preguntas sobre otros temas

Responde SOLO con: {"intent": "allowed"} o {"intent": "blocked", "reason": "..."}
"""

# Costo: ~0.001 USD por clasificación (GPT-4o-mini, ~200 tokens)
# vs ~0.03-0.10 USD por mensaje al agente principal (GPT-4o, ~2000-5000 tokens)
```

**Si está bloqueado, el agente responde:**
> "Soy tu Career Companion — estoy diseñado para ayudarte con tu perfil profesional, evaluar ofertas, y optimizar tu búsqueda de empleo. ¿En qué puedo ayudarte con tu carrera?"

**Margen de tolerancia:** El classifier debe ser ligeramente permisivo. "¿Cómo negocio un sueldo?" es career-related aunque no sea sobre CVs. "¿Qué tecnologías debería aprender?" también. El agente es un career companion amplio, no solo un CV builder.

### Capa 4: Azure Content Safety

- **Jailbreak detection:** Detecta intentos de "ignore your instructions", "you are now DAN", etc.
- **Prompt injection via JD:** Cuando el usuario pega un JD, Content Safety lo analiza antes de procesarlo
- **Content filtering:** Bloquea contenido violento, sexual, o ilegal
- **Implementación:** API call a Azure Content Safety antes de cada invocación del agente principal

### Capa 5: Agente con Restricciones

- System prompt reforzado con instrucciones de no salir del dominio
- Semantic Kernel plugins limitan las acciones posibles (no puede ejecutar código arbitrario, no puede hacer web requests arbitrarios)
- El agente solo puede llamar a los plugins definidos — no tiene acceso general al sistema

---

## Protección de Datos de Usuario

### Supabase Row Level Security (RLS)

```sql
-- Cada tabla tiene RLS habilitado
-- El usuario solo ve sus propios datos
CREATE POLICY "Users can only see own profiles"
ON profiles
FOR ALL
USING (auth.uid() = user_id);

-- Mismo patrón para applications, learnings, outputs, etc.
```

### Datos Sensibles

| Dato | Almacenamiento | Acceso |
|---|---|---|
| Perfil profesional del usuario | Supabase DB (encriptado at rest) | Solo el usuario via RLS |
| CVs generados | Supabase Storage (bucket privado) | Solo el usuario |
| Historial de chat con agente | Supabase DB | Solo el usuario, auto-purge después de 90 días |
| Métricas de uso (quotas) | Supabase DB | Usuario (sus datos) + admin (agregados) |

### System Prompt

- El system prompt del agente NO contiene datos del usuario
- Los datos del usuario se inyectan como contexto en cada conversación, no como parte del system prompt
- Si alguien extrae el system prompt, obtiene las instrucciones del framework (que ya es open source) — no datos personales

---

## Temas Pendientes de Definir

- [ ] Política de retención de datos (GDPR-like para Latam)
- [ ] Manejo de API keys de Azure (¿Django env vars? ¿Azure Key Vault?)
- [ ] Logging de intentos de abuse para detección de patrones
- [ ] ¿Ofrecer tier gratuito o solo trial?
- [ ] Respuesta del agente cuando detecta prompt injection — ¿transparente o silencioso?
