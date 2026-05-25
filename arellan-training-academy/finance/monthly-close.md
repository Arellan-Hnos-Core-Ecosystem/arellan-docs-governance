# Cierre Mensual — Finance

Procedimiento de fin de mes. Hacerlo el último día hábil del mes o el primero del mes siguiente.

## Checklist de Cierre Mensual

### Semana Previa al Cierre

- [ ] Verificar que todas las OTs del mes están en estado `ENTREGADO` o `CANCELADO`
- [ ] Verificar que no hay gastos en estado `APPROVED` sin `DISBURSED` (pendientes de pago)
- [ ] Descargar resumen de caja de los últimos 30 días (ver cualquier `CLOSED_WITH_DISCREPANCY`)
- [ ] Recopilar comprobantes físicos: boletas, facturas, vouchers de Culqi

### Día del Cierre

#### 1. Cierre de Caja Normal del Día

Igual que siempre. El sistema no hace nada especial en el cierre mensual del día.

#### 2. Generar Reporte Mensual

1. **Módulo Reportes → Reporte Mensual → [mes y año]**
2. El reporte incluye:
   - Ingresos totales (desglosados por método: QR Yape, Plin, Tarjeta, Efectivo)
   - Gastos totales (desglosados por categoría)
   - Balance neto
   - OTs completadas vs canceladas
   - Mecánico con más OTs completadas
   - Días con discrepancia de caja (si hubo)
3. Descargar en PDF y Excel
4. Enviar PDF a Edgar via WhatsApp (o imprimir para él)

#### 3. Conciliación con Extracto Bancario

Comparar el reporte del sistema con el extracto bancario de la cuenta de Arellan:

1. Solicitar extracto bancario del mes al banco (o descargarlo si tienen banca por internet)
2. Comparar cada pago QR del sistema vs depósito en cuenta
3. Los pagos Culqi demoran 1-2 días hábiles en liquidarse → tener en cuenta para pagos de fin de mes

Si hay diferencia entre sistema y banco → anotar y revisar con Edgar. No ajustar sin coordinación.

#### 4. Inventario al Cierre del Mes

1. **Módulo Inventario → Exportar Stock Actual → Excel**
2. Comparar con stock físico (conteo manual de los ítems críticos)
3. Si hay diferencias → registrar ajuste de inventario con motivo `ADJUSTMENT`
4. Ítems con discrepancia recurrente → reportar a Edgar (puede indicar pérdida no registrada)

#### 5. Gastos Pendientes

1. **Módulo Finanzas → Gastos → Filtrar: estado APPROVED, mes actual**
2. Gestionar los pagos pendientes antes del cierre
3. Si un gasto ya no se va a ejecutar → rechazarlo con motivo "No ejecutado en el período"

### Reporte para Contador Externo (si aplica)

Si Edgar tiene contador externo para declaraciones SUNAT:

1. **Reportes → Historial de Pagos** → mes → Excel
2. **Reportes → Gastos Aprobados** → mes → Excel (con comprobantes adjuntos)
3. **Reportes → OTs Completadas** → mes → PDF

Estos datos no reemplazan la asesoría del contador — son el insumo para que él trabaje.

## Archivo de Documentos

Crear carpeta en Google Drive o USB: `Cierre [Mes] [Año]`

Guardar:
- PDF reporte mensual del sistema
- Excel de transacciones y gastos
- Fotos/PDFs de comprobantes físicos
- Extracto bancario del mes

Guardar mínimo 5 años (requisito SUNAT para sustentación).

## Señales de Alerta en el Cierre Mensual

| Señal | Posible causa | Acción |
|-------|---------------|--------|
| Ingresos del sistema ≠ depósitos bancarios | Pagos fuera del sistema o demora Culqi | Investigar con Edgar |
| Muchos días con `CLOSED_WITH_DISCREPANCY` | Pagos en efectivo no registrados | Revisar procedimiento de caja |
| Stock físico < stock sistema | Pérdida o uso sin registrar | Ajuste + conversación con mecánicos |
| Gasto APPROVED de > 30 días sin DISBURSED | Proveedor no cobró o se olvidó pagar | Contactar proveedor |
