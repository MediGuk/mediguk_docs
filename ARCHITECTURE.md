# 🏗️ Arquitectura del Sistema Mediguk

## Índice

- [Visión General](#visión-general)
- [Sistema de 3 Etapas](#sistema-de-3-etapas)
- [Arquitectura de Microservicios](#arquitectura-de-microservicios)
- [Flujo de Datos](#flujo-de-datos)
- [Decisiones de Diseño](#decisiones-de-diseño)

---

## Visión General

Mediguk utiliza una arquitectura de microservicios con un sistema de análisis de 3 etapas condicional. El objetivo es proporcionar triaje médico inteligente manteniendo seguridad, escalabilidad y transparencia.

### Principios de Diseño

1. **Safety-First**: En caso de duda, siempre escalar urgencia
2. **Transparencia Total**: Health workers ven el razonamiento completo de la IA
3. **Microservicios**: Componentes independientes y escalables
4. **GDPR Compliant**: Datos procesados localmente, nunca en cloud público
5. **Consensus-Based**: Múltiples validaciones antes de decisión final

---

## Sistema de 3 Etapas

### Flujo Completo
```
┌─────────────────────────────────────────────────────────────┐
│                    PATIENT INPUT                            │
│  • Photo of condition                                       │
│  • Text description (symptoms, duration, severity)          │
│  • Audio explanation (optional)                             │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│               STAGE 1: VLM INITIAL ANALYSIS                 │
│                                                             │
│  Vision-Language Model analyzes:                            │
│  • Visual features (color, size, shape, location)           │
│  • Text symptoms (what, when, how long, severity)           │
│  • Medical context extraction                               │
│                                                             │
│  May ask clarifying questions:                              │
│  • "Is there pain or just itching?"                         │
│  • "How long have you had this?"                            │
│  • "Please take a closer photo in better light"             │
│                                                             │
│  Outputs:                                                   │
│  • Preliminary condition guess                              │
│  • Initial urgency score (0-10)                             │
│  • Confidence level (0-100%)                                │
│  • Suggested routing (Auxiliary/Nurse/Doctor/Specialist)    │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│             STAGE 2: DATA ANALYSIS ENGINE                   │
│                                                             │
│  Takes VLM output and:                                      │
│  • Searches clinic historical database                      │
│  • Finds similar past cases (pattern matching)              │
│  • Analyzes treatment outcomes                              │
│  • Compares demographics, symptoms, presentations           │
│                                                             │
│  Outputs:                                                   │
│  • Top N matching conditions with percentages               │
│    Example: 67% dermatitis, 25% fungal, 8% allergic        │
│  • Historical urgency patterns for similar cases            │
│  • Data-based urgency score (0-10)                          │
│  • Data-based routing recommendation                        │
│  • Confidence level (0-100%)                                │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│              DECISION POINT: DISAGREEMENT CHECK             │
│                                                             │
│  Compare Stage 1 vs Stage 2:                                │
│  ✓ Urgency scores differ by >2 points?                     │
│  ✓ Different primary condition suggestions?                 │
│  ✓ Different routing recommendations?                       │
│  ✓ Either confidence <70%?                                  │
│  ✓ Any "red flags" detected?                                │
└─────────────────────────────────────────────────────────────┘
         ↓                                      ↓
        YES                                    NO
   (20-30% cases)                        (70-80% cases)
         ↓                                      ↓
┌──────────────────────────┐          ┌─────────────────┐
│  STAGE 3: VLM REVIEW     │          │  SKIP STAGE 3   │
│  & RECONCILIATION        │          │                 │
│                          │          │  Proceed with   │
│  VLM receives:           │          │  2-stage data   │
│  • Data analysis results │          │                 │
│  • Similar case patterns │          │  Agreement =    │
│  • Confidence metrics    │          │  High confidence│
│                          │          └─────────────────┘
│  VLM re-examines:        │                   ↓
│  • Original photo with   │                   │
│    new context           │                   │
│  • Data-driven insights  │                   │
│                          │                   │
│  Outputs:                │                   │
│  • Reconciled urgency    │                   │
│  • Explanation of changes│                   │
│  • Additional flags      │                   │
└──────────────────────────┘                   │
         ↓                                      │
         └──────────────┬───────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────────┐
│          FINAL SYSTEM RECOMMENDATION                        │
│                   (Safety-First Logic)                      │
│                                                             │
│  IF 3 stages ran:                                           │
│  • final_urgency = MAX(stage1, stage2, stage3)              │
│  • Show complete 3-stage reasoning chain                    │
│  • Highlight what triggered Stage 3                         │
│                                                             │
│  IF 2 stages only:                                          │
│  • final_urgency = MAX(stage1, stage2)                      │
│  • Show 2-stage agreement confirmation                      │
│  • Note high confidence                                     │
│                                                             │
│  Routing Decision:                                          │
│  • If any stage suggests higher level → escalate           │
│  • If urgency ≥7 → Always to Doctor minimum                │
│  • If urgency 4-6 → Nurse or General Doctor                │
│  • If urgency ≤3 → Auxiliary or Nurse                      │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│              HEALTH WORKER QUEUE                            │
│                                                             │
│  Health worker sees:                                        │
│  • Final urgency level (LOW/MEDIUM/HIGH)                    │
│  • Suggested condition(s) with confidence                   │
│  • Similar past cases from database                         │
│  • Recommended care level                                   │
│  • Complete AI reasoning (all stages visible)               │
│  • Patient photo and description                            │
│                                                             │
│  Actions available:                                         │
│  • Accept recommendation                                    │
│  • Override/modify decision                                 │
│  • Escalate to colleague                                    │
│  • Request more info from patient                           │
│  • Schedule appointment / video call                        │
└─────────────────────────────────────────────────────────────┘
```

### Performance Metrics

| Metric | Target | Rationale |
|--------|--------|-----------|
| 2-stage cases | 70-80% | Most cases are straightforward |
| 3-stage cases | 20-30% | Complex/uncertain cases |
| 2-stage latency | 3-5 sec | Fast enough for good UX |
| 3-stage latency | 8-12 sec | Acceptable for complex cases |
| Cost savings | ~60% | vs always using 3 stages |

---

## Arquitectura de Microservicios
```
┌─────────────────────────────────────────────────────────────┐
│                      FRONTEND LAYER                         │
│                                                             │
│  ┌─────────────────────┐      ┌──────────────────────┐    │
│  │  Patient Web App    │      │  Health Worker       │    │
│  │  (Next.js)          │      │  Dashboard (Next.js) │    │
│  │                     │      │                      │    │
│  │  • Upload symptoms  │      │  • Review queue      │    │
│  │  • Track status     │      │  • View AI analysis  │    │
│  │  • Get results      │      │  • Make decisions    │    │
│  └─────────────────────┘      └──────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                    API GATEWAY LAYER                        │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │        Spring Boot API Gateway                       │  │
│  │                                                      │  │
│  │  • Authentication & Authorization (OAuth2/JWT)       │  │
│  │  • Rate limiting                                     │  │
│  │  • Request routing                                   │  │
│  │  • Load balancing                                    │  │
│  │  • API versioning                                    │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                  MICROSERVICES LAYER                        │
│                                                             │
│  ┌─────────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │   Image     │  │     ML/AI    │  │   Orchestration  │  │
│  │  Service    │  │   Service    │  │     Service      │  │
│  │   (Go)      │  │  (Python)    │  │  (Spring Boot)   │  │
│  │             │  │              │  │                  │  │
│  │ • Upload    │  │ • VLM Stage1 │  │ • 3-stage logic  │  │
│  │ • Process   │  │ • Data Stage2│  │ • Decision point │  │
│  │ • Validate  │  │ • VLM Stage3 │  │ • Queue mgmt     │  │
│  │ • Store     │  │ • Consensus  │  │ • Notifications  │  │
│  └─────────────┘  └──────────────┘  └──────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                      DATA LAYER                             │
│                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌─────────────────┐  │
│  │  PostgreSQL  │  │   S3 Storage │  │     Redis       │  │
│  │              │  │              │  │     Cache       │  │
│  │ • Patients   │  │ • Medical    │  │                 │  │
│  │ • Cases      │  │   images     │  │ • AI responses  │  │
│  │ • Historics  │  │ • Audit logs │  │ • Session data  │  │
│  │ • Queue      │  │              │  │ • Rate limits   │  │
│  └──────────────┘  └──────────────┘  └─────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                  MESSAGE QUEUE LAYER                        │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │              RabbitMQ / Kafka                        │  │
│  │                                                      │  │
│  │  • Async processing between microservices            │  │
│  │  • Event-driven architecture                         │  │
│  │  • Guaranteed message delivery                       │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

---

## Flujo de Datos

### 1. Patient Submission Flow
```
Patient → Frontend → API Gateway → Image Service
                                       ↓
                              Validate & Store
                                       ↓
                              Queue for processing
                                       ↓
                              Orchestration Service
                                       ↓
                              Trigger ML Service (Stage 1)
```

### 2. Analysis Flow
```
ML Service (VLM Stage 1)
    ↓
Extract features & preliminary analysis
    ↓
Send to Data Analysis (Stage 2)
    ↓
Compare with historical cases
    ↓
Decision Point: Disagreement?
    ↓
YES → Trigger VLM Stage 3
NO → Finalize with 2 stages
    ↓
Orchestration Service combines results
    ↓
Apply Safety-First logic
    ↓
Create entry in Health Worker Queue
    ↓
Notify health worker
```

### 3. Health Worker Interaction Flow
```
Health Worker Dashboard → API Gateway → Orchestration Service
                                              ↓
                                    Fetch case from queue
                                              ↓
                                    Load patient data
                                              ↓
                                    Load AI analysis (all stages)
                                              ↓
                                    Load similar historical cases
                                              ↓
                                    Display complete information
                                              ↓
Health Worker makes decision → Update case status
                                              ↓
                                    Notify patient
                                              ↓
                                    Log to audit trail
```

---

## Decisiones de Diseño

### ¿Por qué 3 etapas condicionales?

**Problema**: Usar siempre 3 etapas es costoso (3x API calls) y lento

**Solución**: Solo ejecutar Stage 3 cuando hay desacuerdo o incertidumbre

**Beneficios**:
- 60% reducción en costes
- Latencia promedio menor
- Mantiene precisión en casos complejos

### ¿Por qué microservicios?

**Ventajas**:
- Escalabilidad independiente (e.g., más instancias de Image Service en horas pico)
- Despliegue independiente (actualizar ML sin tocar frontend)
- Tolerancia a fallos (si Image Service cae, resto sigue funcionando)
- Tech stack apropiado para cada servicio (Go para imágenes, Python para ML)

**Desventajas aceptadas**:
- Mayor complejidad operacional
- Requiere orquestación cuidadosa

### ¿Por qué procesamiento local?

**GDPR/LOPD Requirements**:
- Datos médicos no pueden salir de servidores controlados
- Derecho al olvido debe ser garantizado
- Auditoría completa requerida

**Implementación**:
- Todo procesamiento en VPS europeo
- Modelos ML corriendo localmente
- No uso de APIs cloud para datos sensibles

### ¿Por qué safety-first?

**En medicina, es mejor sobre-tratar que sub-tratar**

**Implementación**:
- `final_urgency = MAX(all_stages)` - siempre el valor más alto
- Si cualquier stage detecta red flag → escalada automática
- Umbral conservador para routing (urgency ≥7 → Doctor siempre)

---

## Seguridad

### Autenticación

- OAuth2 / JWT tokens
- 2FA para health workers
- Session timeout after 15min inactivity

### Autorización

- Role-based access control (RBAC)
  - Patient: Solo ve sus propios casos
  - Auxiliary: Ve casos asignados, solo lectura
  - Nurse: Ve casos, puede modificar
  - Doctor: Ve todos, puede reasignar
  - Admin: Gestión completa

### Encriptación

- TLS 1.3 for all communications
- Database encryption at rest
- End-to-end encryption for medical images

### Audit Trail

- Cada acción logueada con:
  - User ID
  - Timestamp
  - Action performed
  - Data accessed/modified
- Logs inmutables (append-only)
- Retention: 7 años (requirement legal en España)

---

## Escalabilidad

### Horizontal Scaling

Servicios que pueden escalarse horizontalmente:
- Image Service (Go) - CPU bound
- ML Service (Python) - GPU/CPU bound
- API Gateway - Stateless

### Vertical Scaling

Servicios que requieren más recursos:
- PostgreSQL - RAM para cache
- ML Service - GPU para inferencia

### Caching Strategy
```
┌─────────────┐
│   Request   │
└──────┬──────┘
       ↓
   Check Redis cache
       ↓
   Cache hit? ──YES──→ Return cached result
       │
       NO
       ↓
   Call ML Service
       ↓
   Store in Redis (TTL: 1 hour)
       ↓
   Return result
```

---

## Monitoreo

### Métricas Clave

- Latency por servicio (p50, p95, p99)
- Error rates
- Throughput (requests/sec)
- Queue depth
- ML model accuracy drift

### Alertas

- Latency p99 > 15sec
- Error rate > 5%
- Queue depth > 100
- Service down
- Database connections > 80% capacity

### Herramientas

- Prometheus para métricas
- Grafana para dashboards
- ELK stack para logs
- Sentry para error tracking

---

## Próximos Pasos

Ver [ROADMAP.md](ROADMAP.md) para el plan de implementación detallado.

---

<p align="center">
  <i>Última actualización: Febrero 2025</i>
</p>
