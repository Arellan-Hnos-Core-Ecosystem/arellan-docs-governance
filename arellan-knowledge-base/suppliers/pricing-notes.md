# Notas sobre Precios y Márgenes

Guía de referencia para validar que los precios de repuestos están dentro de rangos aceptables. El sistema aplica estas reglas automáticamente, pero este documento explica el razonamiento detrás de cada límite.

## Márgenes Objetivo por Categoría

| Categoría | Margen bruto objetivo | Alerta si margen es |
|-----------|----------------------|---------------------|
| Repuestos nacionales | 30-50% | < 15% o > 60% |
| Repuestos importados | 35-55% | < 20% o > 65% |
| Mano de obra | 60-80% (sobre costo de hora) | < 40% |
| Herramientas especializadas | 20-40% | < 10% |

**Ejemplo de cálculo:**
```
Repuesto importado:
  Precio CIF: $50 USD = S/.190 (tipo de cambio 3.80)
  Costos adicionales (flete+aduana): S/.30
  Costo total: S/.220
  Precio de venta al cliente: S/.330

  Margen = (330 - 220) / 330 = 33.3%  ✅ Dentro del rango
```

## Fuentes de Precios de Referencia

### Para Validar Precios de Importación

| Fuente | Uso |
|--------|-----|
| Amazon USA / RockAuto | Precio de referencia de repuestos americanos |
| AliExpress | Referencia para repuestos chinos |
| Mercado Libre Perú | Precio de mercado local en Lima |
| SUNAT aduanet.gob.pe | Valor en aduana declarado por otros importadores |

### Para Verificar Tipo de Cambio

Usar siempre el tipo de cambio del BCP (Banco de Crédito del Perú) al momento de registrar la importación. El sistema tiene un campo para ingresar el tipo de cambio del día.

## Señales de Precios Inflados (Comisión Oculta)

El sistema alerta automáticamente cuando detecta:

1. **Margen implícito < 5%:** El precio de compra es tan alto que casi no hay ganancia. Indica posible sobrecosto.

2. **Variación vs historial > 30%:** El mismo ítem costó S/.100 en los últimos 6 pedidos y ahora cuesta S/.150. Sin justificación de alza de mercado.

3. **Proveedor nuevo + precio 20%+ más alto que el proveedor habitual:** El cambio de proveedor debería traer precios similares o mejores, no más altos.

4. **Descuento del proveedor no reflejado:** Si un proveedor ofrece 10% de descuento por volumen pero el precio en el sistema es el mismo de siempre.

## Comisiones Declaradas vs No Declaradas

**Comisión declarada (PERMITIDA):**
El proveedor paga una comisión de 5% al agente de compras por facilitar la transacción. Esto está declarado en el sistema como `comision_agente`, visible para el owner, y descontado del precio base.

**Comisión no declarada (PROHIBIDA y DETECTADA):**
El proveedor paga directamente al empleado del taller una comisión no registrada. El sistema detecta esto cuando el precio registrado es consistentemente más alto que el precio de mercado para el mismo ítem.

## Actualización de Precios de Referencia

El OWNER actualiza los precios de referencia en el sistema cada 3 meses, basándose en:
- Cotizaciones actuales del mercado
- Variación del tipo de cambio
- Cambios en aranceles SUNAT
