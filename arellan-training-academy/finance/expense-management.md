# Gestión de Gastos — Finance

Registrar, seguir y conciliar egresos del taller.

## Tipos de Gasto

| Categoría | Ejemplos | Quién aprueba (según monto) |
|-----------|----------|----------------------------|
| REPUESTOS_LOCALES | Pastillas de freno, aceite, filtros | Finance ≤100 / Admin ≤500 / Owner >500 |
| IMPORTACION | Repuestos de USA/China via Ricardo... sistema reemplaza eso | Owner siempre (margen validado) |
| SERVICIOS | Electricidad, agua, internet del taller | Admin ≤500 / Owner >500 |
| MANTENIMIENTO | Reparación de equipos del taller | Admin ≤500 / Owner >500 |
| OTROS | Casos especiales | Según monto |

## Registrar Solicitud de Gasto

1. **Módulo Finanzas → Gastos → Nueva Solicitud**
2. Completar campos:
   - Monto (S/.)
   - Descripción detallada (qué se compra y para qué)
   - Categoría
   - Proveedor (seleccionar del directorio o crear nuevo)
   - Adjuntar cotización (PDF o foto de la cotización) — obligatorio si > S/.50
3. Clic **Enviar Solicitud**
4. El sistema asigna automáticamente el nivel de aprobación según el monto
5. Recibirás notificación cuando sea aprobado o rechazado

**Nota:** No puedes aprobar tu propio gasto. El sistema lo bloquea con código `SELF_APPROVAL_FORBIDDEN`.

## Gastos Pequeños (≤ S/.100) — Puedes Aprobar Tú Misma

Si el gasto es tuyo Y es ≤ S/.100:
- Vas a **Gastos → Pendientes de mi aprobación**
- El sistema te mostrará las solicitudes de otros usuarios (no las tuyas) que estén en tu rango

Si el gasto lo creaste tú → automáticamente sube al siguiente nivel (Ana para ≤S/.500, Edgar para >S/.500).

## Seguimiento de un Gasto

En **Módulo Finanzas → Gastos** puedes ver el estado de cada solicitud:

| Estado | Significado |
|--------|-------------|
| PENDING_FINANCE | Esperando aprobación de Finance |
| PENDING_ADMIN | Esperando aprobación de Admin (Ana) |
| PENDING_OWNER | Esperando aprobación de Edgar/Juan |
| PENDING_DUAL_OWNER | Esperando ambos Owners (>S/.2,000) |
| APPROVED | Aprobado — puedes proceder al pago |
| REJECTED | Rechazado — ver el motivo en el detalle |
| DISBURSED | Pagado y conciliado |

## Después de la Aprobación

Cuando un gasto tiene estado `APPROVED`:

1. Realiza el pago al proveedor (transferencia, efectivo, tarjeta)
2. Guarda el comprobante (boleta/factura del proveedor)
3. En el sistema: ve al gasto → **Marcar como Pagado** → adjunta foto del comprobante
4. El gasto pasa a `DISBURSED` y se registra en el audit log

**El sistema no paga directamente** (aún). Tú haces el pago físicamente y luego lo registras.

## Alertas de Importación

Si registras una solicitud de importación y el sistema muestra:

- **"Margen implícito mayor a 35%"** → el precio de venta que se aplicaría sobre el costo de importación es sospechosamente alto → podría haber una comisión oculta al proveedor. Avisar a Edgar antes de proceder.
- **"Margen implícito menor a 5%"** → el costo es inusualmente bajo → verificar que el proveedor sea legítimo.

Esta validación existe porque antes del sistema, el proceso de importaciones tenía comisiones no declaradas que el taller nunca vio.

## Conciliación de Egresos

Al final del mes, para el cierre mensual:

1. **Gastos → Exportar → Mes actual → Excel**
2. Comparar cada ítem `DISBURSED` con los comprobantes físicos guardados
3. Si hay gasto `APPROVED` pero sin `DISBURSED` → pendiente de pago → gestionar antes del cierre
4. Si hay pago realizado pero no registrado → registrarlo con fecha retroactiva + justificación

Ver [monthly-close.md](monthly-close.md) para el proceso completo de cierre mensual.
