# ModoScoring — Seguridad, RBAC y Trazabilidad

## 1) Objetivo
Implementar seguridad por capas para portal, backoffice y API, protegiendo datos sensibles de scoring, identidad, pagos y trazabilidad operativa.

## 2) Identidad y autenticación
- Email/password con políticas robustas.
- Google OAuth (OIDC) coexistiendo con login tradicional.
- Verificación de email para altas no sociales.
- Recuperación de contraseña segura (token corto + single-use).
- MFA recomendado para Owner/Admin y usuarios internos.

## 3) RBAC por cuenta
Roles funcionales:
- `owner`: control total cuenta + billing + API + usuarios.
- `admin`: operación y configuración general.
- `billing`: pagos, facturas, métodos de pago.
- `analyst`: consultas y reportes sin acceso financiero.
- `developer_api`: credenciales API/webhooks sin privilegios billing.

Permisos recomendados por dominio:
- `accounts:*` (owner/admin)
- `billing:*` (owner/billing)
- `reports:read|write` (owner/admin/analyst)
- `api_keys:*` (owner/developer_api)
- `users:*` (owner/admin)

## 4) Seguridad API
- API keys con hash, expiración opcional, revocación inmediata.
- Rotación periódica asistida.
- Rate limits multinivel (key/account/plan).
- HMAC opcional en enterprise.
- Allowlist IP opcional por cuenta.

## 5) Seguridad de datos
- TLS 1.2+ en tránsito.
- Cifrado en reposo (DB, backups, object storage).
- Tokenización de tarjeta vía pasarela (sin PAN completo local).
- Secret management centralizado (KMS/Secrets Manager).
- Minimización y retención controlada de datos personales.

## 6) Auditoría y trazabilidad
- `audit_logs` para acciones sensibles:
  - cambios de plan,
  - ajustes de créditos,
  - gestión de usuarios/roles,
  - emisión/revocación de API keys,
  - intervenciones de backoffice.
- Trazabilidad de consultas:
  - quién consultó,
  - cuándo,
  - por qué canal,
  - qué crédito consumió,
  - correlación con request externo.

## 7) Monitoreo y respuesta
- Alertas de seguridad:
  - intentos fallidos repetidos,
  - geolocalización anómala,
  - picos de consumo API,
  - elevación de privilegios.
- Playbooks de respuesta:
  - revocación masiva de sesiones,
  - rotación forzada de claves,
  - suspensión temporal de cuenta.

## 8) Cumplimiento y privacidad
- Consentimiento/base legal documentada.
- Mecanismos para acceso/rectificación/supresión.
- Registro de fuentes de datos y propósito de uso.
- Términos y privacidad visibles en landing y onboarding.

## 9) Backoffice interno seguro
- Separación estricta entre portal cliente y admin interno.
- SSO corporativo recomendado para staff.
- Permisos granulares para operaciones manuales.
- Doble confirmación en acciones de alto impacto (ej. debitar créditos).
