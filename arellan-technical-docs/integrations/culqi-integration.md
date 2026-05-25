# Culqi Integration

Procesamiento de pagos QR (Yape, Plin, Tarjeta) via Culqi. Este es el módulo que reemplaza el fraude del QR personal de Ricardo.

## Por Qué Culqi y No Yape Directo

El fraude original: Ricardo tenía su QR personal de Yape pegado en el taller. Los clientes pagaban a Ricardo directamente, él no registraba el pago en el sistema y se quedaba con el dinero.

**Solución:** El sistema genera QR dinámicos desde la API de Culqi. El pago va directo a la cuenta empresarial de Arellan. El mecánico/admin nunca ve ni controla el QR — lo genera el sistema, el sistema lo confirma, el sistema lo registra.

## Flujo de Pago QR

```
1. Admin/Finance → POST /finance/payment/qr { workOrderId, paymentMethod }
2. Backend → Culqi API → genera QR dinámico
3. Backend → guarda paymentId en Redis con TTL 7 minutos
4. Cliente escanea QR con Yape/Plin/App
5. Culqi → webhook POST /finance/webhooks/culqi { event: "charge.succeeded", ... }
6. Backend → valida firma HMAC del webhook
7. Backend → verifica idempotencia en Redis (clave: culqi:event:{chargeId})
8. Backend → marca OT como pagada → crea financial_transaction → audit_log
9. Backend → WebSocket → notifica a admin en tiempo real
```

## Generación de QR

```typescript
// finance/payments.service.ts
async generatePaymentQR(dto: GenerateQRDto): Promise<PaymentQRResponse> {
  const workOrder = await this.prisma.workOrder.findUniqueOrThrow({
    where: { id: dto.workOrderId },
  });

  const charge = await this.culqiClient.charges.create({
    amount: Math.round(workOrder.totalAmount * 100),  // Culqi usa centavos
    currency_code: 'PEN',
    description: `OT ${workOrder.orderNumber} - ${workOrder.plate}`,
    email: 'pagos@arellan.pe',
    source_id: await this.generateQRSource(dto.paymentMethod),
  });

  // Guardar en Redis para idempotencia del webhook
  await this.redis.set(
    `culqi:charge:${charge.id}`,
    JSON.stringify({ workOrderId: dto.workOrderId, amount: workOrder.totalAmount }),
    'EX', 3600,  // 1 hora
  );

  return {
    qrCode: charge.metadata.qr_code_base64,
    amount: workOrder.totalAmount,
    expiresAt: new Date(Date.now() + 7 * 60 * 1000),  // 7 minutos
    paymentId: charge.id,
  };
}
```

## Webhook Handler con Idempotencia

```typescript
// finance/culqi-webhook.controller.ts
@Post('webhooks/culqi')
async handleCulqiWebhook(
  @Body() payload: CulqiWebhookPayload,
  @Headers('Culqi-Signature') signature: string,
  @Req() req: Request,
): Promise<void> {
  // 1. Validar firma HMAC
  this.validateWebhookSignature(req.rawBody, signature);

  if (payload.type !== 'charge.succeeded') return;

  const chargeId = payload.data.object.id;

  // 2. Idempotencia: si ya procesamos este evento, ignorar
  const alreadyProcessed = await this.redis.get(`culqi:event:${chargeId}`);
  if (alreadyProcessed) {
    this.logger.warn(`Duplicate Culqi event: ${chargeId}`);
    return;
  }

  // 3. Marcar como procesado ANTES de la operación (evita race condition)
  await this.redis.set(`culqi:event:${chargeId}`, '1', 'EX', 86400);  // 24h

  // 4. Recuperar la OT asociada
  const chargeData = await this.redis.get(`culqi:charge:${chargeId}`);
  if (!chargeData) throw new NotFoundException(`Charge ${chargeId} not found in cache`);

  const { workOrderId } = JSON.parse(chargeData);

  // 5. Procesar el pago en transacción atómica
  await this.prisma.$transaction([
    this.prisma.workOrder.update({
      where: { id: workOrderId },
      data: { isPaid: true, paidAt: new Date() },
    }),
    this.prisma.financialTransaction.create({
      data: {
        workOrderId,
        type: 'INCOME',
        amount: payload.data.object.amount / 100,
        method: this.mapCulqiMethod(payload.data.object.payment_method_type),
        culqiChargeId: chargeId,
      },
    }),
    this.prisma.auditLog.create({
      data: {
        action: 'PAYMENT_CONFIRMED',
        entityType: 'WorkOrder',
        entityId: workOrderId,
        afterData: { chargeId, amount: payload.data.object.amount / 100 },
      },
    }),
  ]);

  // 6. Notificar en tiempo real
  this.eventsGateway.notifyPaymentConfirmed(workOrderId);
}

private validateWebhookSignature(rawBody: Buffer, signature: string): void {
  const expectedSig = crypto
    .createHmac('sha256', this.config.culqiWebhookSecret)
    .update(rawBody)
    .digest('hex');

  if (!crypto.timingSafeEqual(Buffer.from(signature), Buffer.from(expectedSig))) {
    throw new UnauthorizedException('Invalid Culqi webhook signature');
  }
}
```

## Configuración Culqi

```typescript
// culqi.module.ts
@Module({
  providers: [
    {
      provide: 'CULQI_CLIENT',
      useFactory: (config: ConfigService) => {
        const Culqi = require('culqi-node');
        return new Culqi({
          apiKey: config.get('CULQI_SECRET_KEY'),
        });
      },
      inject: [ConfigService],
    },
  ],
})
export class CulqiModule {}
```

## Testing con Sandbox

```bash
# Claves sandbox (en .env.local o staging)
CULQI_PUBLIC_KEY=pk_test_...
CULQI_SECRET_KEY=sk_test_...

# Tarjetas de prueba Culqi:
# Aprobada:  4111111111111111 (Visa)
# Declinada: 4000000000000002
# Yape test: usar el simulador de Culqi sandbox
```

## Simular Webhook en Dev

```bash
# Instalar Culqi CLI o usar ngrok para recibir webhooks en local
ngrok http 3000

# Configurar en Culqi dashboard el webhook URL:
# https://XXXX.ngrok.io/finance/webhooks/culqi

# O simular manualmente:
curl -X POST http://localhost:3000/finance/webhooks/culqi \
  -H "Content-Type: application/json" \
  -H "Culqi-Signature: sha256=..." \
  -d '{"type":"charge.succeeded","data":{"object":{"id":"chr_...","amount":35000}}}'
```

## Errores Conocidos

| Error | Causa | Solución |
|-------|-------|---------|
| `QR expired` | Cliente demoró más de 7 minutos | Generar nuevo QR |
| `Duplicate payment` | Cliente escaneó QR dos veces | Idempotencia en Redis lo bloquea |
| `Webhook signature invalid` | Clave incorrecta o payload modificado | Verificar `CULQI_WEBHOOK_SECRET` |
| `Amount mismatch` | OT sin `totalAmount` calculado | Cerrar líneas de servicio antes de generar QR |
