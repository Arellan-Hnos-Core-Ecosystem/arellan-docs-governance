# WhatsApp Business Integration

Notificaciones automáticas a clientes y dueños via Meta Business API.

## Contexto

El taller ya usa WhatsApp para comunicarse con clientes — es el canal principal en Perú. La integración automatiza mensajes sin que el empleado tenga acceso al teléfono de los clientes (data masking).

**Fase:** Fase 4 completo. En Fase 1-2 se usan los templates básicos solamente.

## Templates Aprobados

Templates pre-aprobados por Meta (requieren aprobación antes de usar):

| Template | Trigger | Destinatario |
|---------|---------|-------------|
| `vehicle_received` | OT creada | Cliente |
| `work_in_progress` | OT → EN_PROCESO | Cliente |
| `vehicle_ready` | OT → LISTO | Cliente |
| `payment_confirmed` | Pago confirmado | Cliente |
| `cashbox_summary` | Cierre de caja | OWNER (Edgar, Juan) |
| `joyride_alert` | Vehículo fuera geofence | OWNER solo |
| `expense_approval_request` | Gasto >S/.500 | OWNER |
| `low_stock_alert` | Stock crítico | OWNER, ADMIN |

## Envío de Mensajes

```typescript
// notifications/whatsapp.service.ts
@Injectable()
export class WhatsAppService {
  private readonly apiUrl = `https://graph.facebook.com/v19.0/${this.config.phoneNumberId}/messages`;

  async sendTemplate(
    to: string,
    templateName: string,
    components: TemplateComponent[],
  ): Promise<void> {
    await this.httpClient.post(
      this.apiUrl,
      {
        messaging_product: 'whatsapp',
        to: this.formatPhone(to),
        type: 'template',
        template: {
          name: templateName,
          language: { code: 'es' },
          components,
        },
      },
      {
        headers: { Authorization: `Bearer ${this.config.whatsappApiToken}` },
      },
    );
  }

  async notifyVehicleReady(plate: string, clientPhone: string, orderNumber: string): Promise<void> {
    await this.sendTemplate(clientPhone, 'vehicle_ready', [
      {
        type: 'body',
        parameters: [
          { type: 'text', text: plate },
          { type: 'text', text: orderNumber },
        ],
      },
    ]);
  }

  async notifyCashboxSummary(summary: CashboxSummary): Promise<void> {
    const owners = await this.prisma.account.findMany({
      where: { role: 'OWNER', status: 'ACTIVE' },
    });

    for (const owner of owners) {
      await this.sendTemplate(owner.phone, 'cashbox_summary', [
        {
          type: 'body',
          parameters: [
            { type: 'text', text: formatPEN(summary.totalIncome) },
            { type: 'text', text: formatPEN(summary.totalExpenses) },
            { type: 'text', text: formatPEN(summary.netBalance) },
            {
              type: 'text',
              text: summary.discrepancy !== 0
                ? `⚠️ Diferencia: ${formatPEN(summary.discrepancy)}`
                : '✓ Sin diferencias',
            },
          ],
        },
      ]);
    }
  }

  private formatPhone(phone: string): string {
    // Asegurar formato internacional con código Perú (+51)
    const clean = phone.replace(/\D/g, '');
    if (clean.startsWith('51')) return clean;
    return `51${clean}`;  // Agrega código país Perú
  }
}
```

## Template: vehicle_ready

Contenido aprobado por Meta (no modificar sin re-aprobación):

```
Hola, su vehículo de placa *{{1}}* ya está listo para recojo en Clínica Automotriz Arellan Hnos.

OT: {{2}}
Horario: Lunes a Sábado 7am - 6pm
Dirección: Av. ejemplo 123, Surquillo

Gracias por su preferencia 🔧
```

Variables: `{{1}}` = placa, `{{2}}` = número de OT.

## Template: cashbox_summary (para dueños)

```
📊 *Resumen de Caja — {{fecha}}*

Ingresos: {{1}}
Gastos: {{2}}
Balance neto: {{3}}

{{4}}

— Sistema Arellan
```

## Template: joyride_alert (prioridad máxima)

```
⚠️ *ALERTA: Vehículo fuera del taller*

Placa: {{1}}
Hora: {{2}}
Última ubicación: {{3}}

Este vehículo salió del perímetro sin autorización.
Acceda al sistema para más detalles.
```

## Configuración

```env
WHATSAPP_API_TOKEN=EAABsbCS...       # Token permanente de Meta
WHATSAPP_PHONE_NUMBER_ID=1234567890  # ID del número en Meta Business
WHATSAPP_BUSINESS_ACCOUNT_ID=0987   # ID de la cuenta de negocio
```

### Obtener Token

1. Meta Business Suite → WhatsApp → API Setup
2. Generar token de sistema con permiso `whatsapp_business_messaging`
3. El token de sistema no expira (a diferencia del token de usuario de 60 días)

## Rate Limits de Meta

| Tipo | Límite | Acción si se supera |
|------|--------|-------------------|
| Mensajes por número destino | 1 msg/seg al mismo número | Implementar delay entre mensajes al mismo usuario |
| Mensajes totales por hora | Varía por nivel de cuenta | En Tier 1: 1,000 conversaciones/día |
| Templates de negocio | Sin límite si el cliente inició chat | |

Para los dueños (Edgar y Juan): máximo 5-10 mensajes por día → sin problemas de rate limit.

## Testing

```typescript
// En development/staging: el servicio loguea el mensaje en lugar de enviarlo
if (this.config.environment !== 'production') {
  this.logger.log(`[WhatsApp Mock] To: ${to}, Template: ${templateName}`, JSON.stringify(components));
  return;
}
// En producción: envío real
```

## Errores Comunes

| Error | Causa | Solución |
|-------|-------|---------|
| `131030` | Template no existe o no aprobado | Verificar nombre exacto del template en Meta |
| `131047` | Número no en WhatsApp | Validar que el cliente usa WhatsApp |
| `131000` | Token expirado | Renovar token en Meta Business |
| `130429` | Rate limit superado | Implementar backoff exponencial |

## Privacidad (Ley 29733)

Los teléfonos de clientes:
- Se guardan encriptados (AES-256) en DB
- El `WhatsAppService` recibe el teléfono como parámetro desde el servicio que tiene acceso
- Los mecánicos NO tienen acceso al teléfono del cliente — el envío de WhatsApp lo hace el sistema, no un empleado
- Los dueños sí tienen acceso para comunicación directa cuando sea necesario
