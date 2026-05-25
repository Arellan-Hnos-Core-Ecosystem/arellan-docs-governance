# ZKTeco Biometric Integration

Integración con el reloj biométrico ZKTeco via protocolo ADMS (Automatic Data Master Server).

## Arquitectura

```
[Reloj ZKTeco en el taller, Surquillo LAN]
    │
    │ HTTP POST (ADMS protocol)
    │ cada punche de huella/facial/PIN
    ▼
[arellan-iot-hardware-bridge] ← proceso Node.js separado
    │ Recibe el evento
    │ Valida y transforma
    │ HTTP interno
    ▼
[arellan-backend-core]
    │ Procesa asistencia
    │ Detecta anomalías (PIN en vez de huella, fuera de horario)
    ▼
[PostgreSQL: attendance_records]
```

El hardware está en la LAN del taller (IP dinámica, ISP residencial). El reloj envía eventos al servidor cloud via ADMS reverse-push → no necesitamos conectarnos al reloj, él se conecta a nosotros.

## Protocolo ADMS

El reloj ZKTeco hace `POST /adms/push` al endpoint configurado. El handler **debe** responder con `GetStamp` para que el reloj sincronice su tiempo:

```typescript
// iot-bridge/adms.controller.ts
@Controller('adms')
export class AdmsController {
  @Post('push')
  async receiveAttendance(@Body() body: string): Promise<string> {
    // ADMS envía como x-www-form-urlencoded, no JSON
    const params = new URLSearchParams(body);
    const event = {
      serialNumber: params.get('SN'),
      userId: params.get('UserID'),     // ID del empleado en el reloj (ej: "E003")
      timestamp: params.get('Stamp'),   // "2024-01-15 07:05:22"
      attendState: params.get('AttState'), // 0=Check-in, 1=Check-out
      verifyMethod: params.get('Verify'),  // 1=PIN, 4=Huella, 15=Facial
    };

    await this.processEvent(event);

    // CRÍTICO: El reloj necesita esta respuesta para sincronizar su reloj interno
    // Sin esta respuesta, el reloj se desconecta y no envía más eventos
    const now = new Date();
    return `GET STAMP\r\nStamp=${format(now, "yyyy-MM-dd HH:mm:ss")}\r\n`;
  }
}
```

## Payload ADMS

```
POST /adms/push HTTP/1.1
Content-Type: application/x-www-form-urlencoded

SN=ZK-TERMINAL-SURQUILLO-01&UserID=E003&Stamp=2024-01-15+07%3A05%3A22&AttState=0&Verify=4
```

| Campo | Descripción | Valores |
|-------|-------------|---------|
| `SN` | Serial number del reloj | "ZK-TERMINAL-SURQUILLO-01" |
| `UserID` | ID del empleado en el reloj | "E001", "E002", ... |
| `Stamp` | Timestamp del evento | "2024-01-15 07:05:22" |
| `AttState` | Tipo de evento | 0=Check-in, 1=Check-out |
| `Verify` | Método de verificación | 1=PIN, 4=Huella digital, 15=Facial, 6=Tarjeta |

## Procesamiento en Backend

```typescript
// employees/attendance.service.ts
async processAdmsEvent(event: AdmsAttendanceEvent): Promise<void> {
  const account = await this.prisma.account.findFirst({
    where: {
      zktecoBiometricId: event.userId,
      status: 'ACTIVE',
    },
  });

  if (!account) {
    this.logger.warn(`Unknown ZKTeco user: ${event.userId}`);
    return;
  }

  const attendanceRecord = await this.prisma.attendanceRecord.create({
    data: {
      accountId: account.id,
      zktecoBiometricId: event.userId,
      zktecDeviceSn: event.serialNumber,
      eventType: event.attendState === '0' ? 'CHECK_IN' : 'CHECK_OUT',
      verifyMethod: this.mapVerifyMethod(event.verifyMethod),
      recordedAt: new Date(event.timestamp),
      isForcedCheckout: false,
    },
  });

  // Detectar uso de PIN en vez de huella (señal de fraude potencial)
  if (event.verifyMethod === '1') {  // PIN
    await this.alertsService.sendPinVerificationAlert(account, attendanceRecord);
  }

  // Audit log
  await this.auditService.log({
    accountId: account.id,
    action: 'ATTENDANCE_RECORDED',
    entityType: 'AttendanceRecord',
    entityId: attendanceRecord.id,
    afterData: { eventType: attendanceRecord.eventType, verifyMethod: attendanceRecord.verifyMethod },
  });
}

private mapVerifyMethod(verifyCode: string): VerifyMethod {
  const map: Record<string, VerifyMethod> = {
    '1': 'PIN',
    '4': 'HUELLA',
    '15': 'FACIAL',
    '6': 'CARD',
  };
  return map[verifyCode] ?? 'PIN';
}
```

## Forzar Checkout a las 11:59 PM

Si un empleado no registra salida, el cron job lo fuerza para no dejar registros abiertos:

```typescript
// employees/attendance.service.ts
@Cron('59 23 * * *', { timeZone: 'America/Lima' })
async forceEndOfDayCheckout(): Promise<void> {
  const today = startOfDay(new Date());

  // Empleados con check-in pero sin check-out hoy
  const openCheckIns = await this.prisma.attendanceRecord.findMany({
    where: {
      eventType: 'CHECK_IN',
      recordedAt: { gte: today },
      // Sin un CHECK_OUT posterior para el mismo empleado hoy
    },
  });

  for (const checkIn of openCheckIns) {
    await this.prisma.attendanceRecord.create({
      data: {
        accountId: checkIn.accountId,
        eventType: 'CHECK_OUT',
        verifyMethod: 'PIN',  // Marcado como sistema
        recordedAt: endOfDay(new Date()),
        isForcedCheckout: true,  // Flag importante para reportes
      },
    });

    this.logger.warn(`Forced checkout for account ${checkIn.accountId}`);
  }
}
```

## Configuración del Reloj ZKTeco

En la pantalla del reloj (o via ZKTeco Pro App):

```
COMM → ADMS → Server Address: https://iot.arellan.pe
               Port: 443 (HTTPS)
               Enable ADMS: Yes
               Push Data: Attendance

# El endpoint en el bridge:
POST https://iot.arellan.pe/adms/push
```

## Gestión de Empleados en el Reloj

Cuando se crea un empleado en el sistema:
1. Backend crea el usuario → genera `zktecoBiometricId` (ej: "E007")
2. Admin va al reloj físico → agrega usuario con ese ID
3. Registra huella dactilar del empleado en el reloj
4. El reloj empieza a enviar sus eventos con ese UserID

Cuando se termina un empleado:
1. Backend → `PATCH /employees/:id/status { status: "TERMINATED" }` → sesiones revocadas
2. Admin va al reloj → elimina al usuario del dispositivo
3. Si por alguna razón el reloj aún envía eventos del empleado terminado → backend los descarta (`status != ACTIVE`)

## Troubleshooting

Ver `arellan-knowledge-base/troubleshooting/biometric-issues.md` para problemas comunes (no sincroniza, huella rechazada, tiempo equivocado).

## Offline Buffer

Si el internet del taller falla, el reloj almacena hasta 50,000 eventos localmente. Cuando se recupera la conexión, los envía todos en orden cronológico. El backend los procesa con el timestamp del hardware (`recordedAt`), no el de inserción en DB (`createdAt`).

Esto garantiza que los registros de asistencia sean precisos incluso con cortes de internet.
