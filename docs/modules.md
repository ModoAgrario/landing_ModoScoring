# ModoScoring — Mapa de Módulos

## 1) Superficies de producto
```text
Landing Pública
Portal Cliente (self-service)
Backoffice Interno
API Empresas
```

## 2) Módulos funcionales y responsabilidades

## 2.1 Landing pública
- Propuesta de valor, planes, FAQ, compliance, CTA registro/login.
- SEO, performance, analítica de conversión.
- Integración con formularios comerciales (enterprise/API access).

## 2.2 Identidad y acceso
- Registro email + password.
- Login social Google OAuth.
- Verificación de email, recuperación de contraseña.
- Gestión de sesiones (logout global, expiración, revocación).
- MFA opcional para perfiles administrativos.

## 2.3 Cuentas y membresías
- Cuenta individual/empresa.
- Invitaciones multiusuario.
- Roles: Owner, Admin, Billing, Analyst/Operator, Developer/API.
- Políticas RBAC por recurso y acción.

## 2.4 Planes, suscripciones y pricing
- Catálogo de planes (pack, mensual, anual, ilimitado con fair use).
- Cambio de plan, upgrade inmediato con prorrateo.
- Downgrade diferido al próximo ciclo.
- Conversión mensual↔anual según política.

## 2.5 Créditos y consumo
- Wallet de créditos por ciclo + packs extendidos.
- Reglas de consumo: primero suscripción, luego packs.
- Estados de saldo y alertas (80/90/100%).
- Bloqueo operativo por saldo agotado + CTA de upgrade.

## 2.6 Consultas / Informes
- Alta de consulta web/API.
- Historial y detalle por cuenta/usuario/canal.
- Idempotencia para API.
- Trazabilidad completa de input, outcome y consumo.

## 2.7 API empresas
- Habilitación por feature flag.
- Gestión de credenciales (emitir, rotar, revocar).
- Rate limit por plan/cuenta.
- Webhooks de eventos operativos y billing.
- Métricas de uso y errores por integración.

## 2.8 Billing y facturación
- Métodos de pago tokenizados.
- Cobros recurrentes, reintentos, dunning.
- Facturas/recibos/ajustes.
- Prorrateo de upgrades y notas de crédito operativas.

## 2.9 Backoffice interno
- Gestión de cuentas, planes, créditos y pagos.
- Operaciones manuales auditadas (ajustes, bloqueos, correcciones).
- Soporte (timeline + notas + tickets).
- Riesgo/fraude/abuso y gobierno de API.
- Integraciones externas (estado, certificados, jobs).

## 2.10 Observabilidad y auditoría
- Logs estructurados por request.
- Auditoría de eventos sensibles.
- Trazas distribuidas para consultas de punta a punta.
- Dashboards SRE/negocio.

## 3) Dependencias entre módulos
```text
Auth -> Account/RBAC -> (Portal, Backoffice, API)
Plans/Billing <-> Credits
Reports -> Credits -> External Scoring Engine
API Mgmt -> Rate Limits -> Reports
Backoffice -> todos los dominios (con permisos internos)
```

## 4) Priorización por fases
- **Fase 1**: landing, auth, planes/pagos básicos, dashboard saldo, consultas web, backoffice básico.
- **Fase 2**: multiusuario, RBAC completo, API self-service, webhooks, prorrateo automático, auditoría ampliada.
- **Fase 3**: dunning avanzado, anti-fraude, integraciones verticales (ARCA), automatizaciones enterprise.
