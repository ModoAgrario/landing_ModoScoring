# ModoScoring — Billing, Suscripciones y Créditos

## 1) Modelo comercial soportado
- Packs por única vez (vigencia extendida sugerida: 12 meses).
- Suscripciones mensuales.
- Suscripciones anuales (descuento por compromiso).
- Plan ilimitado (con política de uso justo y rate limits).

## 2) Reglas de ciclo
- Ciclo de suscripción definido por fecha aniversario.
- Créditos de suscripción:
  - se asignan al inicio de ciclo,
  - no se acumulan,
  - se renuevan al ciclo siguiente.
- Packs extra:
  - vigencia larga,
  - consumo posterior al cupo de suscripción.

## 3) Política de consumo (obligatoria)
Orden:
1. Consumir créditos de `credit_cycle` activo.
2. Al agotar, consumir `credit_packs` disponibles.
3. Si no hay saldo, bloquear consulta con CTA upgrade.

## 4) Upgrade con prorrateo (inmediato)
Variables:
- `P1` precio actual, `P2` precio nuevo
- `C1` créditos actuales, `C2` créditos nuevos
- `f = días_restantes / días_totales`

Fórmulas:
- `cargo_prorrateado = (P2 - P1) * f`
- `creditos_extra = ceil((C2 - C1) * f)`
- `saldo_resultante = max(0, C1 - usados) + creditos_extra`

Regla de aplicación:
- Cobro inmediato del prorrateo.
- Acreditación inmediata de créditos extra prorrateados.
- Siguiente ciclo: plan nuevo completo.

## 5) Downgrade y cambios de período
- Downgrade: siempre diferido al próximo ciclo.
- Mensual -> anual:
  - por defecto, aplicar próximo ciclo.
  - excepción operativa mediante backoffice.

## 6) Cobranza y dunning
- Reintentos sugeridos: D+1, D+3, D+5, D+7.
- Notificaciones en cada estado (`pending`, `failed`, `paid`).
- Grace period configurable antes de suspensión.
- Si falla renovación:
  - marcar `past_due`,
  - restringir uso progresivamente,
  - suspender si no recupera cobro.

## 7) UI de transparencia (portal)
Mostrar siempre:
- plan actual y modalidad,
- fecha de renovación,
- saldo (`x de y`),
- historial de consumo,
- cargo inmediato al confirmar upgrade,
- precio normal desde próximo ciclo.

## 8) Jobs y colas de billing/créditos
```text
Queue: billing.renewal
- generar invoice
- ejecutar cobro
- emitir evento subscription.renewed|payment_failed

Queue: billing.dunning
- reintentos programados
- notificaciones
- transición de estado

Queue: credits.reconciliation
- validar ledger vs saldos materializados
- alertar inconsistencias
```

## 9) Riesgos frecuentes y controles
- Doble cobro por retry: idempotencia en cobros.
- Doble consumo por timeout: reserva de crédito + commit/rollback.
- Reclamos por prorrateo: desglose visible + comprobante de cálculo.
- Inconsistencia de saldos: reconciliación nocturna y alarmas.
