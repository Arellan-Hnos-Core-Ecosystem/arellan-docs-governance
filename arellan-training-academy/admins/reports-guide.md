# Guía de Reportes — Administrador

Cómo generar y exportar reportes desde el sistema.

## Reportes Disponibles

### Módulo Reportes → Seleccionar tipo:

| Reporte | Qué muestra | Cuándo usarlo |
|---------|-------------|---------------|
| Resumen Diario | Ingresos, gastos, OTs del día | Cada noche antes de salir |
| OTs por Período | Lista de órdenes en rango de fechas | Cuando Edgar pide ver actividad |
| Asistencia de Empleados | Entradas/salidas, horas trabajadas, tardanzas | Fin de quincena / fin de mes |
| Inventario Actual | Stock de todos los ítems + alertas | Cuando se planifica compras |
| Gastos por Categoría | Egresos agrupados por tipo | Revisión de costos mensual |
| Historial de Pagos | Todos los pagos QR + método | Conciliación bancaria |

## Cómo Generar un Reporte

1. **Módulo Reportes** → clic en el tipo de reporte
2. Seleccionar **rango de fechas** (o elegir: Hoy / Esta semana / Este mes)
3. Filtros opcionales: por mecánico, por categoría, por estado
4. Clic en **Generar Reporte**
5. El sistema muestra la vista previa en pantalla
6. Para exportar: botón **Descargar PDF** o **Descargar Excel**

## Reportes Frecuentes que Pide Edgar

### "¿Cuánto hicimos esta semana?"

**Reporte:** Resumen Diario × 5 días → o usar Resumen Semanal directo
- Módulo Reportes → Resumen → Esta semana → PDF
- Enviar por WhatsApp o imprimir

### "¿Cómo va la asistencia de los mecánicos?"

**Reporte:** Asistencia de Empleados
- Módulo Empleados → Asistencia → Seleccionar empleado (o Todos)
- Rango: quincena actual
- Ver columnas: días trabajados, horas, tardanzas, salidas forzadas

### "¿Cuánto gastamos en repuestos este mes?"

**Reporte:** Gastos por Categoría
- Módulo Finanzas → Reportes → Gastos
- Filtrar por categoría: REPUESTOS_LOCALES + IMPORTACION
- Rango: mes actual
- Excel para que Edgar revise en detalle

## Exportar Datos para Auditoría

Si Edgar o un contador externo pide los datos:

1. **Reporte de Transacciones Financieras** → todos los pagos del período
2. **Reporte de Gastos Aprobados** → con quién aprobó cada gasto
3. **Reporte de Audit Log** (solo Owners) → historial de todas las acciones del sistema

Para el audit log, Edgar lo descarga desde su cuenta OWNER — tú como Admin no tienes acceso completo a ese reporte por seguridad.

## Si el Reporte Tarda Mucho

Reportes con más de 3 meses de datos pueden demorar 10-30 segundos en generarse. Normal. No recargues la página.

Si después de 1 minuto no carga → anota el rango de fechas que pediste y avisa al Tech Lead.

## Reportes Automáticos (Fase 4)

En Fase 4 del sistema, Edgar y Juan recibirán automáticamente por WhatsApp:
- Resumen diario: 6:30 PM cada día
- Resumen semanal: domingo 8 PM
- Reporte mensual: primer día del mes

Por ahora (MVP), los reportes se generan manualmente cuando se necesitan.
