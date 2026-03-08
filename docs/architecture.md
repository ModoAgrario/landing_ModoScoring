# ModoScoring — Arquitectura de Plataforma SaaS

## 1) Objetivo y alcance
Diseñar una arquitectura escalable para Argentina y LatAm con cuatro superficies de producto:
1. Landing pública (adquisición).
2. Portal cliente (autogestión B2C/B2B).
3. Backoffice interno (operación, soporte, riesgo, billing).
4. API para empresas (integración programática).

Supuestos clave:
- El motor de scoring vive fuera de esta plataforma y se consume por API externa.
- Login con email/contraseña y Google OAuth.
- Suscripciones, packs, créditos por ciclo, upgrades con prorrateo, cobro con tarjeta.
- Multiusuario, RBAC, auditoría, rate limits y trazabilidad.

## 2) Arquitectura lógica (alto nivel)
```text
[Web Landing] ----\
[Web Portal] ------> [API Gateway/BFF] ---> [Auth Service]
[Backoffice Web] --/          |             [Account & RBAC Service]
                              |             [Plans/Billing Service] ---> [Payment Gateway]
[Clientes API] -------------> [Public API] --> [Credits & Usage Service]
                                              --> [Reports Orchestrator] --> [External Scoring Engine API]
                                              --> [Webhook Service]
                                              --> [Notification Service]

                           [Event Bus / Queue]
                              |        |      
                      [Billing Jobs] [Usage Jobs] [Risk/Fraud Jobs]

                                  [PostgreSQL]
                                  [Redis]
                                  [Object Storage]
                                  [Observability Stack]
```

## 3) Estilo arquitectónico recomendado
- **Modular monolith evolutivo** al inicio (MVP/Fase 1 y 2), con límites de dominio claros.
- Evolución progresiva a microservicios para dominios de mayor carga:
  - Reports Orchestrator
  - Billing/Cobranza
  - API Publica & rate limiting
  - Notificaciones/Webhooks
- Ventaja: menor complejidad inicial + ruta clara de escalabilidad.

## 4) Componentes principales
- **Frontend**:
  - Landing (SSR/estática optimizada para conversión y SEO).
  - Portal cliente (SPA/SSR con dashboard, consultas, billing, API keys).
  - Backoffice interno con permisos fuertes.
- **Capa API**:
  - API Gateway unifica auth, límites y observabilidad.
  - BFF para portal/backoffice (endpoints orientados a UI).
  - Public API versionada (`/api/v1`).
- **Servicios de dominio**:
  - Identidad y acceso.
  - Cuentas, membresías y RBAC.
  - Planes/suscripciones/facturación.
  - Créditos y consumo.
  - Consultas y trazabilidad (web/api).
  - Integraciones (scoring externo, ARCA opcional por vertical).
- **Infraestructura de datos**:
  - PostgreSQL (transaccional y auditoría).
  - Redis (sesiones, rate limiting, caché temporal).
  - Cola/event bus (SQS/Rabbit/Kafka según etapa).
  - Object storage para comprobantes y exportaciones.

## 5) Flujos críticos
### 5.1 Registro y onboarding
1. Usuario se registra por email o Google.
2. Se crea `Account` (individual/empresa) + `User` + `Membership(Owner)`.
3. Se valida email (si aplica).
4. Se habilita trial o plan inicial.

### 5.2 Consulta de scoring
1. Request web/API entra por gateway.
2. Auth + RBAC + rate limit.
3. Credits service reserva 1 crédito (hold transaccional).
4. Reports orchestrator invoca motor externo.
5. Si respuesta válida: consume crédito definitivo y persiste consulta.
6. Si error técnico/no exitoso: libera hold según política.

### 5.3 Upgrade con prorrateo
1. Usuario selecciona nuevo plan.
2. Billing calcula cargo prorrateado + créditos extra prorrateados.
3. Se cobra método principal.
4. Si cobro exitoso: se acredita saldo proporcional y audit trail.
5. Próximo ciclo aplica plan completo.

## 6) Escalabilidad para Argentina y LatAm
- **Multi-tenant por cuenta** desde día 1.
- **Moneda/impuestos configurables por país** (iniciar ARS, extender a otras monedas).
- **Feature flags por país/segmento** (API, ARCA, planes enterprise).
- **Rate limits por plan y por cuenta** con configuración dinámica.
- **Desacople async** para tareas de cobranza, webhooks, alertas, reconciliaciones.
- **Observabilidad integral**: métricas, logs estructurados, trazas distribuidas.

## 7) NFRs y SLOs propuestos
- Disponibilidad API pública: 99.9% mensual.
- p95 endpoint scoring sync: < 2.5s (sin contar timeouts externos extremos).
- RTO/RPO inicial: 4h/1h; objetivo enterprise: 1h/15min.
- Auditoría inmutable para acciones sensibles (retención mínima 24 meses).

## 8) Riesgos y mitigaciones
- **Dependencia del motor externo**: timeouts, circuit breakers, colas de retry.
- **Reclamos por créditos/prorrateo**: ledger explícito + UX transparente + auditoría.
- **Fraude/abuso API**: rate limit, IP allowlist opcional, detección anómala.
- **Cobros fallidos**: dunning automático, reintentos escalonados, suspensión gradual.
- **Complejidad regulatoria regional**: capa fiscal desacoplada por país.

## 9) Recomendaciones técnicas concretas
- Backend: TypeScript (NestJS/Fastify) o Kotlin (Spring) con arquitectura hexagonal por módulos.
- Frontend: Next.js para landing/portal/backoffice (repos separados o monorepo con apps).
- DB: PostgreSQL 15+, migraciones versionadas y constraints fuertes.
- Cola: iniciar con SQS/Rabbit; evaluar Kafka al escalar eventos.
- Infra: Docker + IaC (Terraform), despliegue en AWS/GCP con entornos `dev/stg/prod`.
