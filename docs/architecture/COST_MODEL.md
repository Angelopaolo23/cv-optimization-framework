# Modelo de Costos — Career Companion Platform

> **Estado:** Estimación inicial. Requiere validación con datos reales post-MVP.
> **Última actualización:** 2026-02-22

---

## Principio: El Backend es el Proxy de Costos

Todo request que involucre un LLM pasa por Django. El frontend NUNCA habla directo con Azure OpenAI. Esto permite:
- Medir tokens exactos por usuario
- Enforcer quotas antes de gastar
- Rutear al modelo correcto según la tarea
- Cortar si un usuario excede su presupuesto

---

## Costos por Operación (Estimados con Azure OpenAI)

> Precios aproximados de Azure OpenAI a Feb 2026. Sujetos a cambio.

### Modelo de Routing por Operación

| Operación | Modelo | Input tokens (est.) | Output tokens (est.) | Costo estimado |
|---|---|---|---|---|
| **Intent classification** | GPT-4o-mini | ~300 | ~50 | ~$0.0003 |
| **Clasificación A-D de requisitos** | GPT-4o | ~3,000 | ~1,000 | ~$0.02 |
| **Scoring (cálculo numérico)** | Código Python | 0 | 0 | $0.00 |
| **Generación Killer Summary** | GPT-4o | ~2,000 | ~500 | ~$0.01 |
| **Generación Impact Bullets** | GPT-4o | ~3,000 | ~1,500 | ~$0.025 |
| **Brand Coherence Audit** | GPT-4o | ~2,000 | ~800 | ~$0.015 |
| **Mensaje de chat (onboarding, Q&A)** | GPT-4o | ~2,000 | ~500 | ~$0.01 |
| **Validación Non-Negotiables** | Código Python | 0 | 0 | $0.00 |
| **Research de empresa (Bing Search)** | Bing Search API | N/A | N/A | ~$0.005 (3 queries) |
| **Refinamiento iterativo (por turno)** | GPT-4o | ~3,000 (context) | ~500 | ~$0.015 |
| **Embedding para RAG** | text-embedding-3-small | ~500 | N/A | ~$0.00001 |

### Costo por Flujo Completo

| Flujo | Operaciones | Costo total estimado |
|---|---|---|
| **Screening Rápido** (Fases 1-2) | Intent + Clasificación + Scoring + Resumen | ~$0.05 |
| **Screening + Research** (Sprint 2) | Screening + 3 Bing queries + cultura enriquecida | ~$0.06 |
| **CV Completo** (7 fases) | Intent + Clasificación + Scoring + Summary + Bullets + Audit + Retro | ~$0.15 |
| **CV Completo + Research** (Sprint 2) | CV Completo + Bing Search | ~$0.16 |
| **Refinamiento** (3 turnos promedio) | 3x (Intent + Corrección con contexto) | ~$0.05 |
| **Sesión de Onboarding** (~20 mensajes) | 20x (Intent + Chat) | ~$0.20 |
| **Sesión de chat general** (~10 mensajes) | 10x (Intent + Chat) | ~$0.10 |

### Costo Mensual por Usuario Tipo (post-Sprint 2)

| Tipo de usuario | Uso estimado/mes | Costo AI/mes |
|---|---|---|
| **Light** | 2 screenings + 1 CV + 3 refinamientos + 10 chats | ~$0.50 |
| **Normal** | 5 screenings + 3 CVs + 10 refinamientos + 30 chats | ~$1.50 |
| **Heavy** | 15 screenings + 8 CVs + 25 refinamientos + 100 chats | ~$5.00 |
| **Abuser (sin protección)** | 500+ mensajes off-topic | ~$50+ ⚠️ |

> **Impacto de Sprint 2:** Bing Search agrega ~$0.005/screening (despreciable). El refinamiento iterativo agrega ~$0.05 por sesión de 3 turnos (aceptable). El costo por usuario sube ~30-40%, pero el margen sigue siendo >80% para uso normal.

### Margen con $15/mes

| Tipo | Costo AI | Costo infra | Total costo | Margen |
|---|---|---|---|---|
| Light | $0.50 | ~$0.50 | ~$1.00 | **93%** |
| Normal | $1.50 | ~$0.50 | ~$2.00 | **87%** |
| Heavy | $5.00 | ~$0.50 | ~$5.50 | **63%** |
| Abuser | $50+ | ~$0.50 | $50+ | **NEGATIVO** |

> **Conclusión:** El margen es excelente para uso legítimo. El intent classifier + quotas son críticos para prevenir el escenario de abuse.

---

## Costos de Infraestructura (Estimados)

### MVP (0-100 usuarios)

| Servicio | Tier | Costo/mes |
|---|---|---|
| Supabase | Free tier | $0 |
| Vercel | Free tier (hobby) | $0 |
| Railway (backend) | Starter | ~$5 |
| Redis (Railway addon o Upstash) | Free tier | $0 |
| Azure OpenAI | Pay-as-you-go | Variable (ver arriba) |
| Azure Content Safety | Free tier (1k calls/mes) luego pay-as-you-go | $0-5 |
| Bing Search API (Sprint 2) | S1 tier (1k calls/mes gratis, luego $7/1k) | $0-7 |
| **Total infra fijo** | | **~$5-17/mes** |

### Growth (100-1,000 usuarios)

| Servicio | Tier | Costo/mes |
|---|---|---|
| Supabase | Pro ($25) | $25 |
| Vercel | Pro ($20) | $20 |
| Railway | Pro | ~$20 |
| Redis | Upstash Pro | ~$10 |
| Azure OpenAI | Pay-as-you-go | ~$100-500 (depende de usuarios) |
| Azure Content Safety | Pay-as-you-go | ~$20 |
| **Total infra fijo** | | **~$195-595/mes** |

### Break-even

| Usuarios pagando $15/mes | Revenue | Costo estimado (Normal use) | Profit |
|---|---|---|---|
| 10 | $150 | ~$25 (infra) + $10 (AI) = $35 | +$115 |
| 50 | $750 | ~$50 (infra) + $50 (AI) = $100 | +$650 |
| 100 | $1,500 | ~$100 (infra) + $100 (AI) = $200 | +$1,300 |
| 500 | $7,500 | ~$300 (infra) + $500 (AI) = $800 | +$6,700 |

---

## Estrategia de Optimización de Costos

### 1. Model Routing (ya descrito en STACK.md)

El ahorro principal viene de NO usar GPT-4o para todo:

```
Ahorro estimado del intent classifier:
- Sin classifier: 100% de mensajes van a GPT-4o → $0.01/msg
- Con classifier: ~20% de mensajes son off-topic y se rechazan con GPT-4o-mini → $0.0003/msg
- Si un usuario manda 100 mensajes off-topic: ahorro de ~$0.97
```

### 2. Caching Inteligente

- **Cache de scoring:** Si el JD no cambió y el perfil no cambió, devolver score anterior
- **Cache de embeddings:** Generar embedding del perfil una vez, regenerar solo cuando se edita
- **Cache de análisis de empresa:** Si otro usuario ya evaluó la misma empresa, reusar research

### 3. Scoring Determinista en Código

El scoring era la operación más frecuente (cada screening y cada CV lo calcula). Moverlo a código ahorra ~$0.02 por scoring × miles de scorings/mes.

### 4. Batch Processing

Para operaciones que no necesitan respuesta inmediata (ej: Brand Coherence Audit, Retrospectiva), usar Azure OpenAI Batch API con ~50% de descuento.

---

## Métricas a Trackear Post-Launch

| Métrica | Por qué importa | Alerta |
|---|---|---|
| Tokens/usuario/día | Detectar abusers | >5,000 tokens/día sostenido |
| % mensajes off-topic | Eficacia del intent classifier | >30% indica UX confusa |
| Costo/usuario/mes promedio | Margen real | >$5 = revisar quotas |
| Ratio intent blocked/total | Calibrar classifier | >50% = demasiado restrictivo |
| Latencia P95 del agente | UX | >10s = optimizar pipeline |

---

## Temas Pendientes

- [ ] Precios exactos de Azure OpenAI para la región que usaremos
- [ ] ¿Ofrecer tier gratuito? (riesgo de abuse vs. adquisición de usuarios)
- [ ] ¿Cobrar por features premium adicionales? (voice-to-text, adaptadores premium)
- [ ] Evaluar modelos open source (Llama, Mistral) como alternativa para tareas simples
- [ ] Definir política de fair use clara para los usuarios
