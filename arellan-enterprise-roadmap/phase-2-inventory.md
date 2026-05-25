# Fase 2 — Inventario Inteligente e Importaciones (Meses 4-6)

## Objetivo

Eliminar el control monopolizado de información sobre importaciones y gestión de stock. Implementar el sistema de importaciones con auditoría de márgenes para prevenir comisiones ocultas del 20-30%.

## Prerequisitos de Entrada

- MVP Fase 1 estable en producción por al menos 4 semanas
- Diferencias de caja < S/.100/mes sostenidas
- Equipo capacitado y usando el sistema diariamente
- Datos de inventario inicial ingresados

## Módulos a Desarrollar

### Sistema de Importaciones (`arellan-procurement-system`)

**Problema a resolver:** Ricardo negociaba con proveedores extranjeros y cobraba comisiones del 20-30% sin declarar, pagadas directamente por los proveedores.

```
Funcionalidades:
- Registro de proveedores internacionales con Tax ID
- Cotizaciones con precio fijo y proveedor fijo antes de aprobación
- Validación automática de márgenes (alerta si > 35%)
- Flujo de aprobación: Finance → Admin → Owner según monto
- Registro de costos aduaneros (flete + desaduanaje + aranceles)
- Comprobante de pago obligatorio (upload a S3)
- Validación SUNAT del proveedor nacional
```

### Inventario Avanzado

```
Funcionalidades nuevas vs MVP:
- Código QR/barras para cada ítem del inventario
- Scanner con cámara de tablet para registrar movimientos
- Historial completo de movimientos por ítem
- Precio promedio ponderado actualizado en cada ingreso
- Alertas de stock mínimo configurables por categoría
- Solicitudes de reposición desde tablet del mecánico
- Dashboard de inventario con valor total y rotación
```

### Integración ZKTeco Biométrica

```
- Bridge ADMS: recibir fichajes en tiempo real
- Mapeo UserID ZKTeco → employeeId interno
- Historial de asistencia por empleado
- Checkout forzado automático a las 11:59 PM
- Alertas si asistencia no coincide con actividad en sistema
- Cruce: acciones en sistema cuando empleado no fichó check-in
```

### Portal de Cliente (Consulta Básica)

**Prerequisito:** Registro ante MINJUS (Ley 29733) completado.

```
- Página pública en cliente.arellan.pe
- Consulta por número de placa (sin login)
- Estado de OT visible: RECIBIDO → LISTO
- Tiempo estimado de entrega
- Sin datos personales del cliente expuestos
```

### SUNAT Integración (Fase 2)

```
- Contrato con Nubefact (OSE homologado, ~S/.29/mes)
- Emisión de facturas electrónicas (tipo 01)
- Emisión de boletas electrónicas (tipo 03)
- Validación de comprobantes de proveedores
- XML firmado + CDR de SUNAT
```

## Milestones

| Hito | Semana | Criterio |
|------|--------|---------|
| M2.1: Importaciones en producción | 4 | Primer pedido de importación aprobado por el sistema |
| M2.2: ZKTeco integrado | 5 | 100% del personal fichando por sistema |
| M2.3: Inventario con QR | 6 | 100% del inventario escaneado y en sistema |
| M2.4: Portal cliente live | 8 | Clientes pueden consultar estado de OT por placa |
| M2.5: SUNAT habilitado | 10 | Primera factura electrónica emitida |

## KPIs de Éxito

| KPI | Meta |
|-----|------|
| Importaciones con comisiones > 35% detectadas | 0 (o justificadas) |
| Precisión del inventario | > 95% coincidencia físico/digital |
| Personal registrado en ZKTeco | 100% |
| Consultas de estado de OT por portal | > 20 por semana |

## Deuda Técnica de Fase 1 a Saldar

- Migrar de Supabase Auth a JWT RS256 custom (si el tráfico lo justifica)
- Optimizar queries de reportes (agregar vistas materializadas)
- Implementar Redis para cache de sesiones
- Completar cobertura de tests al 85%
