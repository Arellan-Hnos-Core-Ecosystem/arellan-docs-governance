# SUNAT Integration

Facturación electrónica via Nubefact (OSE autorizado por SUNAT).

## Contexto

Arellan Hnos emite boletas y facturas electrónicas. En el MVP (Fase 2), la integración con Nubefact permite:
- Emitir boletas al cerrar una OT pagada
- Emitir facturas para clientes empresa con RUC
- Enviar comprobantes por email o WhatsApp al cliente

**OSE seleccionado:** Nubefact (`nubefact.com`) — API REST simple, tarifa por comprobante (~S/.0.10-0.20 c/u), ideal para volumen de taller pequeño.

## Flujo de Emisión

```
1. OT cerrada + pagada → admin confirma emisión
2. Backend → valida datos del comprobante
3. Backend → POST /api/v1/fe/json a Nubefact
4. Nubefact → valida con SUNAT (OSE)
5. SUNAT → acepta o rechaza
6. Nubefact → responde con XML firmado + PDF
7. Backend → guarda URL del PDF en S3
8. Backend → envía PDF por WhatsApp/email al cliente
```

## Tipos de Comprobante

| Tipo | Cuando | Identificación cliente |
|------|--------|----------------------|
| Boleta (B001-XXXX) | Persona natural, DNI o sin documento | DNI o "00000000" |
| Factura (F001-XXXX) | Empresa con RUC | RUC validado (11 dígitos) |

## API Nubefact

### Emisión de Boleta

```typescript
// sunat/nubefact.service.ts
async emitirBoleta(workOrderId: string): Promise<ComprobantResponse> {
  const workOrder = await this.prisma.workOrder.findUniqueOrThrow({
    where: { id: workOrderId },
    include: {
      client: true,
      services: { include: { inventoryItem: true } },
      payment: true,
    },
  });

  const payload: NubefactBoletaPayload = {
    operacion: 'generar_comprobante',
    tipo_de_comprobante: 2,  // 1=Factura, 2=Boleta
    serie: this.config.nubefactSerieBoleta,
    numero: await this.getNextBoleta(),
    sunat_transaction: 1,  // Venta interna
    cliente_tipo_de_documento: 1,  // 1=DNI, 6=RUC
    cliente_numero_de_documento: workOrder.client.dni ?? '00000000',
    cliente_denominacion: workOrder.client.fullName,
    fecha_de_emision: format(new Date(), 'dd-MM-yyyy'),
    fecha_de_vencimiento: format(new Date(), 'dd-MM-yyyy'),
    moneda: 1,  // 1=PEN
    tipo_de_cambio: '',
    porcentaje_de_igv: 18.0,
    descuento_global: 0,
    total_descuento: 0,
    total_anticipo: 0,
    total_gravada: Math.round(workOrder.totalAmount / 1.18 * 100) / 100,
    total_inafecta: 0,
    total_exonerada: 0,
    total_igv: Math.round((workOrder.totalAmount - workOrder.totalAmount / 1.18) * 100) / 100,
    total_gratuita: 0,
    total_otros_cargos: 0,
    total: workOrder.totalAmount,
    percepcion_tipo: '',
    percepcion_base_imponible: 0,
    total_percepcion: 0,
    total_incluido_percepcion: 0,
    detraccion: false,
    observaciones: `OT ${workOrder.orderNumber} - ${workOrder.plate}`,
    items: workOrder.services.map(service => ({
      unidad_de_medida: 'ZZ',
      codigo: service.inventoryItem?.sku ?? 'SRV',
      descripcion: service.description,
      cantidad: service.quantity,
      valor_unitario: service.unitPrice / 1.18,
      precio_unitario: service.unitPrice,
      descuento: 0,
      subtotal: service.totalPrice / 1.18,
      tipo_de_igv: 1,
      igv: service.totalPrice - service.totalPrice / 1.18,
      total: service.totalPrice,
      anticipo_regularizacion: false,
    })),
    adicionales: [
      {
        codigo: 1000,
        nombre: 'Placa vehiculo',
        valor: workOrder.plate,
      },
    ],
  };

  const response = await this.httpClient.post(
    `${this.config.nubefactBaseUrl}/api/v1/fe/json`,
    payload,
    { headers: { Token: this.config.nubefactApiToken } },
  );

  if (response.data.errors) {
    throw new BadRequestException(`SUNAT error: ${JSON.stringify(response.data.errors)}`);
  }

  // Guardar URL del PDF
  await this.prisma.workOrder.update({
    where: { id: workOrderId },
    data: { invoiceUrl: response.data.enlace_del_pdf },
  });

  return response.data;
}
```

### Validación de RUC

```typescript
async validateRUC(ruc: string): Promise<boolean> {
  if (ruc.length !== 11) return false;

  // Llamar a API de validación SUNAT (via Nubefact o APIS.net)
  const response = await this.httpClient.get(
    `https://api.nubefact.com/api/v1/ruc/${ruc}`,
    { headers: { Token: this.config.nubefactApiToken } },
  );

  return response.data.estado === 'ACTIVO';
}
```

## Configuración

```env
NUBEFACT_API_TOKEN=eyJ...
NUBEFACT_RUC=20XXXXXXXXX
NUBEFACT_SERIE_BOLETA=B001
NUBEFACT_SERIE_FACTURA=F001
NUBEFACT_ENVIRONMENT=demo    # demo | produccion
# demo: comprobantes de prueba, no van a SUNAT real
# produccion: comprobantes reales, van a SUNAT
```

```typescript
// URL por ambiente
const NUBEFACT_BASE_URL = {
  demo: 'https://demo-fact.nubefact.com',
  produccion: 'https://fact.nubefact.com',
};
```

## Errores SUNAT Comunes

| Código | Descripción | Solución |
|--------|-------------|---------|
| 2017 | El número de RUC no existe | Validar RUC antes de emitir factura |
| 2329 | El producto o servicio no puede ser deducible | Usar descripción correcta del servicio |
| 0152 | Certificado digital vencido | Renovar certificado en Nubefact |
| 4269 | El tipo de documento del cliente es inválido | Verificar que DNI sea 8 dígitos |

## Notas Legales

- Boletas con monto > S/.700 deben incluir DNI del cliente
- Facturas sin RUC válido son rechazadas por SUNAT
- Los XML firmados se guardan por 5 años (obligación legal)
- En Nubefact, los XML firmados se almacenan en su sistema — descargables cuando se necesite

## Fase de Implementación

Esta integración es **Fase 2** — no está activa en el MVP. En MVP, el taller emite comprobantes manuales o usa el sistema de Nubefact directamente. La integración automática se activa cuando:
1. Se tiene RUC registrado en Nubefact
2. Se tiene certificado digital vigente
3. Se completa el módulo `InvoiceModule` en el backend
