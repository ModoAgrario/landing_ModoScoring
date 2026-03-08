# ModoScoring — Roadmap de Implementación

## 1) Estrategia general
Construcción por fases con entregables verticales (funcional + técnico + operativo), priorizando time-to-market y reducción de riesgo en billing/créditos.

## 2) Fase 0 — Foundations (2-4 semanas)
- Decisiones de stack e infraestructura base.
- Setup CI/CD, observabilidad y gestión de secretos.
- Esqueleto de módulos de dominio y migraciones iniciales.
- Definición final de contratos con motor externo de scoring.

**Entregables**
- Arquitectura deployable `dev/stg/prod`.
- Catálogo de eventos y esquema de auditoría.
- Primer set de tablas núcleo.

## 3) Fase 1 — MVP comercial y operación básica (6-10 semanas)
- Landing pública con CTA y páginas legales.
- Auth: email/password + Google.
- Portal cliente básico: dashboard, consultas web, saldo, plan actual.
- Planes/pagos con tarjeta y facturación básica.
- Backoffice básico: cuentas, suscripciones, créditos, soporte inicial.
- Integración productiva con motor de scoring externo.

**KPIs de salida**
- registro->pago,
- tiempo a primera consulta,
- tasa de éxito de scoring,
- reclamos de saldo/cobro.

## 4) Fase 2 — Escala self-service B2B (6-8 semanas)
- Multiusuario + invitaciones + RBAC completo.
- API empresas v1 (keys, usage, rate limits).
- Webhooks y docs API en portal.
- Upgrades con prorrateo automático.
- Auditoría expandida y trazabilidad end-to-end.

**KPIs de salida**
- cuentas con API habilitada,
- errores por integración,
- upgrades/mes,
- recuperación dunning.

## 5) Fase 3 — Operación avanzada LatAm (8-12 semanas)
- Dunning avanzado y automatismos de cobranza.
- Motor antifraude/abuso (reglas + scoring de riesgo operativo).
- Packs extra/top-ups autogestionables.
- Verticalización de integraciones (ARCA y otras fuentes regionales).
- Hardening de performance y alta disponibilidad.

**KPIs de salida**
- MRR/ARR,
- churn,
- fraude evitado,
- disponibilidad y latencia p95.

## 6) Hitos transversales técnicos
- Hito A: ledger de créditos confiable y reconciliado.
- Hito B: idempotencia completa (billing + scoring API).
- Hito C: auditoría y trazabilidad certificables.
- Hito D: operación SRE con SLOs y alertas.

## 7) Riesgos del roadmap y mitigación
- Ambigüedad funcional de prorrateo -> especificación y test de casos límite.
- Dependencia de proveedor de pago -> abstracción + fallback.
- Cuellos de botella del motor externo -> timeouts, retries, colas, circuit breaker.
- Deuda de permisos -> matriz RBAC temprana + pruebas de autorización.

## 8) Recomendaciones de ejecución
- Mantener un RFC corto por decisión estructural (billing, créditos, API auth).
- QA automatizado por dominio crítico (credits/billing/auth).
- Rollout progresivo con feature flags por segmento/país.
- Comité semanal de métricas: producto + tech + revenue + riesgo.
