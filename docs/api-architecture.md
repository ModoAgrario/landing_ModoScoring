# ModoScoring — Arquitectura API (Empresas + Interna)

## 1) Principios API
- Versionado explícito (`/api/v1`).
- Contratos estables, cambios breaking sólo en nueva versión.
- Seguridad por API keys con rotación, scopes y revocación.
- Idempotencia en operaciones de scoring para evitar doble consumo.
- Observabilidad por endpoint, cuenta y credencial.

## 2) Topología
```text
[Client Systems]
   |
   v
[API Gateway]
  - authN/authZ
  - rate limits
  - request validation
  - correlation-id
   |
   v
[Public API Service]
   |--> [Credits Service]
   |--> [Reports Orchestrator] --> [External Scoring Engine]
   |--> [Usage Service]
   |--> [Webhook Service]
```

## 3) Endpoints objetivo (MVP+)
- `POST /api/v1/reports/score`
- `GET /api/v1/reports/{id}`
- `GET /api/v1/usage`
- `GET /api/v1/plans`
- `POST /api/v1/webhooks/test`

Extensiones:
- `GET /api/v1/credits/balance`
- `POST /api/v1/api-keys/rotate` (portal/backoffice).

## 4) Autenticación y autorización
- Header `Authorization: Bearer <api_key>` o esquema `X-API-Key`.
- Clave almacenada hasheada; sólo se muestra una vez en creación.
- Scopes recomendados:
  - `reports:write`
  - `reports:read`
  - `usage:read`
  - `webhooks:write`
- Opción enterprise: firma HMAC con timestamp para anti-replay.

## 5) Consumo de créditos e idempotencia
- Regla: 1 scoring exitoso = 1 crédito.
- Errores de autenticación/validación no consumen.
- Soporte `Idempotency-Key` en `POST /reports/score`:
  - misma key + mismo payload => misma respuesta/lógica.
  - evita doble cobro ante retries de red.

## 6) Rate limits
- Capas:
  1. Por API key.
  2. Por cuenta.
  3. Por plan.
- Dimensiones:
  - requests/min
  - requests/hora
  - requests/día
  - burst corto.
- Respuesta estándar `429` con headers de cuota restante.

## 7) Errores y contrato de respuesta
- Estructura uniforme:
```json
{
  "error": {
    "code": "CREDITS_EXHAUSTED",
    "message": "No hay créditos disponibles.",
    "correlation_id": "..."
  }
}
```
- Catálogo mínimo: `UNAUTHORIZED`, `FORBIDDEN`, `RATE_LIMITED`, `INVALID_INPUT`, `CREDITS_EXHAUSTED`, `UPSTREAM_TIMEOUT`.

## 8) Webhooks
Eventos:
- `subscription.renewed`
- `subscription.payment_failed`
- `credits.low`
- `credits.exhausted`
- `plan.changed`
- `api_key.rotated`

Buenas prácticas:
- Firma HMAC del payload.
- Retries exponenciales con DLQ.
- Reenvío manual desde portal/backoffice.

## 9) Entornos
- `sandbox` para integración y pruebas.
- `production` con límites y auditoría completa.
- Claves separadas por entorno.

## 10) Seguridad operativa API
- TLS obligatorio.
- Allowlist IP opcional para enterprise.
- WAF y detección de patrones anómalos.
- Trazabilidad completa por `correlation_id`.
