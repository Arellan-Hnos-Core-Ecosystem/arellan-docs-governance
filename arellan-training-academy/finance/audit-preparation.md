# Preparar Auditoría — Finance

Cómo preparar reportes cuando Edgar, un contador externo, o una entidad oficial (SUNAT, MTPE) pide información.

## Tipos de Auditoría

| Tipo | Quién pide | Qué necesitan |
|------|-----------|---------------|
| Interna (Edgar) | Edgar o Juan | Resumen financiero + detalle de gastos |
| Contable externa | Contador del taller | Transacciones, comprobantes, gastos |
| SUNAT | Fiscalización tributaria | Comprobantes de venta, gastos con sustento |
| MTPE / Ley 728 | Ministerio de Trabajo | Registros de asistencia, incidencias laborales |
| Seguros | RC Profesional u otro seguro | Foto de ingreso del vehículo, registro de la OT |

## Exportar Datos del Sistema

### Para Auditoría Financiera

```
Módulo Reportes → Historial de Transacciones
  → Rango: período auditado
  → Exportar: Excel + PDF

Módulo Finanzas → Gastos → Historial
  → Rango: período auditado
  → Incluir: aprobador + comprobante adjunto
  → Exportar: Excel
```

### Para Auditoría Laboral (MTPE)

```
Módulo Empleados → Asistencia
  → Seleccionar empleado (o todos)
  → Rango: período auditado
  → Exportar: Excel con detalle de cada día

Módulo Empleados → Incidencias
  → Empleado específico
  → Exportar: PDF con evidencia adjunta
```

El sistema registra cada entrada con el **método de verificación** (huella dactilar, facial, PIN). Esto es importante para demostrar que el registro biométrico es auténtico.

### Para Auditoría de Vehículos

```
Módulo Vehículos → [placa] → Historial de OT
  → Ver fotos de ingreso (5 obligatorias, con SHA-256 hash)
  → Ver firma digital del cliente
  → Ver estado de la OT en cada fecha
```

El hash SHA-256 de las fotos prueba que no fueron modificadas después del ingreso.

## El Audit Log del Sistema

El audit log registra **cada acción crítica** con timestamp, usuario, IP y datos antes/después del cambio. Este registro:
- Es inmutable — nadie puede borrarlo ni modificarlo
- Está disponible para Edgar en **Módulo Seguridad → Audit Log**
- Solo accesible para OWNER (Edgar y Juan) — tú como Finance no lo ves directamente

Si una auditoría necesita el audit log, Edgar lo exporta.

## Documentos Físicos a Tener Listos

Complementar los datos del sistema con documentos físicos:

- Boletas/facturas de todos los gastos del período
- Vouchers de pago con tarjeta (si se usó POS)
- Contratos de proveedores (si aplica)
- Planillas o recibos de pago a empleados

Guardar en carpetas mensualmente → ver [monthly-close.md](monthly-close.md).

## Auditoría SUNAT

Si SUNAT fiscaliza:

1. **No entregues nada directamente** — primero avisar a Edgar y al contador externo
2. El contador prepara la respuesta oficial
3. Tú aportas los datos del sistema según lo que el contador necesite
4. Plazo habitual: SUNAT da 10-15 días hábiles para responder

**Nota legal:** Las boletas y facturas electrónicas emitidas via Nubefact ya están en el servidor de SUNAT — no puedes borrarlas ni modificarlas aunque quisieras. Eso en realidad protege al taller.

## Respuesta a Consultas de Edgar

Cuando Edgar pide: "¿Qué pasó el día X con la caja?"

1. **Módulo Finanzas → Caja → Historial → [fecha]**
2. Ver: apertura, todos los ingresos del día, egresos, cierre, discrepancia (si hubo)
3. Clic en cualquier transacción → ver detalle: método de pago, OT asociada, quien la registró

Si Edgar quiere más detalle (quién hizo qué hora por hora) → eso está en el Audit Log → solo él puede verlo con su cuenta OWNER.

## Preparar Informe para Edgar (formato sugerido)

```
INFORME MENSUAL — [Mes Año]
Preparado por: Valeria Arellan (Finance)
Fecha: [fecha]

RESUMEN:
- Ingresos del mes: S/. ______
- Gastos del mes: S/. ______
- Balance neto: S/. ______

NOTAS:
- Días con descuadre de caja: [N] (detalle en Excel adjunto)
- Gastos inusuales: [describir si hubo algo fuera de lo normal]
- Incidencias con empleados: [si las hubo]

ADJUNTOS:
- Excel transacciones del mes
- Excel gastos aprobados del mes
- Extracto bancario del mes
```
