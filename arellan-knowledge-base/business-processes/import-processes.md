# Proceso de Importación de Repuestos

## Contexto

El taller importa repuestos especializados de EE.UU., China y Europa que no se consiguen en el mercado local peruano. Históricamente, el empleado encargado (Ricardo) cobraba comisiones del 20-30% de los proveedores extranjeros sin declararlas al taller. Este proceso digitalizado cierra ese riesgo.

## Flujo Completo de Importación

### Paso 1: Identificación de la Necesidad

Puede originarse de:
- Mecánico solicita repuesto que no hay en stock y no hay equivalente local
- Owner o Admin identifica oportunidad de importar para stock estratégico
- El sistema genera alerta de stock crítico para ítem importado

### Paso 2: Búsqueda de Proveedor y Cotización

1. **Seleccionar proveedor del directorio homologado** (`supplier-directory.md`)
   - Si es proveedor nuevo: debe ser registrado primero con Tax ID verificable
   - Solo Edgar o Juan pueden agregar proveedores internacionales nuevos
2. **Solicitar cotización formal** con precio CIF (Cost, Insurance, Freight)
3. **Registrar cotización en el sistema:**
   - Precio CIF declarado (en USD)
   - Tipo de cambio del día (BCP oficial)
   - Proveedor exacto (no se puede cambiar post-aprobación)
   - URL o adjunto de la cotización del proveedor

### Paso 3: Validación de Márgenes (Sistema Automático)

```
El sistema calcula automáticamente:

Precio CIF en soles = precio_usd × tipo_cambio_bcp
Costo total estimado = precio_CIF_soles + flete_aéreo + desaduanaje + aranceles

Margen implícito = (precio_venta_histórico - costo_total) / precio_venta_histórico

SI margen > 35%: BLOQUEADO
  → Requiere justificación escrita del OWNER explicando por qué el costo es mayor
  → Alerta a Edgar y Juan antes de aprobar

SI margen < 5%: ALERTA
  → El precio parece inflado (posible sobrecosto)
  → Owner debe revisar antes de aprobar

SI 5% ≤ margen ≤ 35%: APROBABLE
```

### Paso 4: Aprobación del Owner

- **Siempre** requiere aprobación del owner (sin excepción para importaciones)
- Owner recibe push en app móvil con: proveedor, ítem, precio CIF, costo total estimado, margen calculado
- Owner aprueba o rechaza con biometría del teléfono
- El precio aprobado queda **fijo** — no se puede modificar post-aprobación

### Paso 5: Pago al Proveedor

**Restricciones de pago:**
- Solo por transferencia bancaria al proveedor registrado (no efectivo)
- El destinatario de la transferencia debe coincidir exactamente con el proveedor en el sistema
- Comprobante de pago obligatorio: pantallazo del banco + número de transferencia
- El comprobante se adjunta en el sistema dentro de las 24 horas

**Si el proveedor pide pago a cuenta de tercero:** ALERTA DE FRAUDE. No se puede procesar sin investigación y aprobación especial de ambos owners.

### Paso 6: Seguimiento del Embarque

```
En el sistema se registra:
- Número de tracking del courier (DHL/FedEx/USPS/etc)
- Nombre del agente de aduana en Lima
- Fecha estimada de llegada
- Costos adicionales reales (al conocerse):
  - Flete real (si difiere del estimado)
  - Gastos de aduana real
  - Aranceles reales (según clasificación arancelaria)
```

### Paso 7: Recepción y Verificación

Al llegar el paquete:
1. Verificar que el contenido coincide con la orden de compra (ítem, cantidad, marca)
2. Si hay discrepancia > 5% en precio: investigación + notificación a owner
3. Ingresar los ítems al inventario con el precio de costo real (no el estimado)
4. Adjuntar la factura del proveedor en el sistema
5. El sistema valida que el total pagado + costos aduaneros = costo registrado

## Costos Aduaneros Típicos (Referencia)

| Concepto | Estimado |
|---------|---------|
| Flete aéreo (EE.UU. → Lima) | $15-50 USD/kg |
| Comisión agente de aduana | 2-5% del valor CIF |
| Aranceles SUNAT | 0-12% según partida arancelaria |
| IGV de importación | 18% sobre valor en aduana |
| Almacenaje aeropuerto | S/.30-100 por día de demora |

## Señales de Alerta de Comisiones Ocultas

El sistema genera alerta automática ante:
- Precio CIF > 30% por encima del precio de mercado de referencia
- Proveedor solicita pago a cuenta diferente al proveedor registrado
- Proveedor "nuevo" propuesto por el empleado encargado (no por el owner)
- Órdenes de compra que el empleado quiere "acelerar" sin seguir el proceso
- Descuentos del proveedor que no se reflejan en el precio final cobrado al taller
