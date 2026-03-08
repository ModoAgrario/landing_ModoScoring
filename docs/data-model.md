# ModoScoring — Modelo de Datos

## 1) Principios
- Multi-tenant estricto por `account_id`.
- Ledger auditable para créditos y billing.
- Soft delete en entidades sensibles.
- Timestamps consistentes (`created_at`, `updated_at`, `deleted_at`).

## 2) Entidades núcleo

## 2.1 Identidad y organización
- `accounts`: tipo, razón social, cuit, estado, owner_user_id, billing_email.
- `users`: nombre, apellido, email único, auth_provider, email_verified, estado.
- `memberships`: account_id, user_id, role, estado.
- `invitations`: email invitado, rol, token, expiración, estado.

## 2.2 Planes y suscripciones
- `plans`: nombre, tipo (`pack|monthly|yearly|unlimited`), créditos, precio, moneda, activo.
- `subscriptions`: account_id, plan_id, start_at, end_at, renewal_at, auto_renew, estado.
- `subscription_changes`: tipo (`upgrade|downgrade|billing_period_change`), aplicado_en, metadata.

## 2.3 Créditos
- `credit_wallets`: account_id, policy_version, estado.
- `credit_cycles`: subscription_id, cycle_start, cycle_end, assigned, consumed, available.
- `credit_packs`: account_id, plan_id, purchased_at, expires_at, total, consumed, available.
- `credit_movements` (ledger):
  - tipo (`allocation|consumption|upgrade_extra|bonus|adjustment|reversal`)
  - cantidad (+/-)
  - source (`web|api|backoffice|billing_job`)
  - reference_type/reference_id
  - idempotency_key opcional

## 2.4 Consultas y scoring
- `consultations`: account_id, user_id nullable, channel (`web|api`), input_ref, status, credits_consumed, requested_at.
- `consultation_results`: consultation_id, provider_name, provider_request_id, normalized_payload, score_value, delivered_at.
- `consultation_traces`: consultation_id, stage, latency_ms, error_code, retry_count.

## 2.5 API empresas
- `api_clients`: account_id, nombre, estado.
- `api_keys`: client_id, key_prefix, secret_hash, scopes, expires_at, last_used_at, revoked_at.
- `api_requests`: account_id, api_key_id, endpoint, status_code, latency_ms, idempotency_key, created_at.
- `webhook_endpoints`: account_id, url, secret_hash, eventos, estado.
- `webhook_deliveries`: endpoint_id, event_id, attempt, status, response_code.

## 2.6 Billing
- `payment_methods`: account_id, provider, token_ref, brand, last4, exp_month, exp_year, is_default.
- `invoices`: account_id, subscription_id, amount, currency, tax_amount, status, issued_at, due_at.
- `payments`: invoice_id, provider_reference, amount, status, paid_at, failure_reason.
- `refunds`: payment_id, amount, reason, status.

## 2.7 Seguridad y auditoría
- `audit_logs`: actor_type, actor_id, account_id, action, entity, entity_id, payload_before, payload_after, ip, user_agent, timestamp.
- `security_events`: tipo, severidad, account_id, user_id, metadata, detected_at.
- `rate_limit_counters` (Redis/materialización temporal).

## 3) Relaciones principales
```text
Account 1---N Membership N---1 User
Account 1---N Subscription N---1 Plan
Subscription 1---N CreditCycle
Account 1---N CreditPack
Account 1---N CreditMovement
Account 1---N Consultation 1---1 ConsultationResult
Account 1---N ApiClient 1---N ApiKey
Account 1---N Invoice 1---N Payment
```

## 4) Reglas de integridad recomendadas
- `users.email` único global, normalizado.
- `memberships` único por (`account_id`,`user_id`) activo.
- `credit_movements` siempre balanceable contra saldos derivados.
- `consultations` sólo confirma consumo cuando status final es exitoso.
- `api_keys.secret_hash` irreversible (nunca guardar secreto plano).

## 5) Particionado y retención
- Particionar `consultations`, `api_requests`, `audit_logs` por mes.
- Retención sugerida:
  - auditoría: 24 meses mínimo;
  - requests API detalladas: 6-12 meses hot + archive cold;
  - eventos de seguridad: 24+ meses.
