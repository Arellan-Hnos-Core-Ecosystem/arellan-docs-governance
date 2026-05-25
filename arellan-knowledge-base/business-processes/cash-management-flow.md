# Proceso Operativo Estándar: Control de Caja y Flujo de Recaudación Digital

## 1. Diagnóstico del Dolor Financiero

La operación manual previa permitía que trabajadores con roles de confianza mostraran sus aplicativos personales (Yape/Plin personal) a los clientes para el cobro de mano de obra y repuestos, desviando el capital de la clínica hacia cuentas de terceros.

**El problema específico:** Ricardo (empleado de confianza) presentaba su Yape personal a clientes para cobros de servicios, impidiendo que el taller registrara esos ingresos. Estimado de pérdidas: S/.1,500-3,000 mensuales.

## 2. Arquitectura de Mitigación (Cero Intervención Humana en el Cobro)

Queda estrictamente prohibido que cualquier mecánico, practicante o administrador muestre un código QR físico o su número telefónico personal para procesar un pago dentro del local de Surquillo. El flujo de recaudación se automatiza por completo:

```
[Módulo de Liquidación (Frontend Admin)]
    │
    │ POST /api/v1/finance/qr/generate
    ▼
[API NestJS → Pasarela Corporativa (Culqi/Yape Business)]
    │
    │ QR dinámico generado con monto exacto de la OT
    ▼
[Tablet del Taller — QR visible solo en pantalla]
    │
    │ Cliente escanea con su teléfono
    ▼
[Pago del Cliente]
    │
    │ Webhook firmado criptográficamente → arellan-api-gateway
    ▼
[Cierre automático de OT + Registro en audit_log]
```

## 3. Reglas Críticas del Flujo de Pago

### Regla 1: Generación de QR Dinámico
Al finalizar la reparación, Ana (admin) o la hija de Edgar (finance) hace clic en "Liquidar Orden". El sistema calcula el monto total (Mano de obra + Repuestos del inventario) y genera un token único para la pasarela corporativa. El QR **expira automáticamente en 7 minutos** — si el cliente no paga en ese tiempo, se debe regenerar.

### Regla 2: Exhibición Controlada
El QR dinámico se muestra EXCLUSIVAMENTE en la pantalla de la tablet de liquidación orientada al cliente. La tablet no tiene la app de Yape personal instalada. Solo tiene acceso a `taller.arellan.pe`.

### Regla 3: Procesamiento de Webhooks con Idempotencia
La pasarela de pagos envía una notificación asíncrona (Webhook) firmada criptográficamente. El sistema procesa la transacción utilizando una clave de idempotencia (`payment_id`) para evitar duplicaciones en caso de que Culqi reenvíe el mismo evento múltiples veces.

### Regla 4: Fallback por Falla de Red

Si el webhook de la pasarela falla por microcortes de internet en Surquillo:

1. La OT permanece en estado `PENDIENTE_VERIFICACION`
2. Ana ingresa al portal bancario corporativo (con MFA físico o Google Authenticator)
3. Contrasta el número de operación bancaria con el campo temporal generado por el sistema
4. Ingresa manualmente el código de verificación bancaria
5. El sistema cambia el estado a `PAGADO` y genera un log de auditoría inmutable con la acción manual de Ana, su firma digital y timestamp

**Esta acción manual queda en audit_log como `MANUAL_PAYMENT_CONFIRMATION` — más restrictivo que el pago automático.**

## 4. Apertura y Cierre de Caja

### Apertura (7:00 AM)

1. Ana ingresa al sistema con MFA
2. Navega a **Módulos Financieros → Control de Caja**
3. Hace clic en **"Apertura del Día"**
4. El sistema muestra el saldo con el que cerró el día anterior
5. Ana cuenta físicamente los billetes y monedas de la gaveta
6. Ingresa el monto real → si coincide, confirma la apertura

### Durante el Día

- Los pagos por QR se registran automáticamente (no requiere acción de Ana)
- Los pagos en efectivo se registran manualmente en el sistema inmediatamente después de recibirlos
- **Regla inquebrantable de egresos:** Si un mecánico pide dinero de caja para un repuesto, Ana verifica el módulo de Autorizaciones de Gasto. Si no tiene el estado `APROBADO_POR_DUEÑO`, **no entrega ningún sol**. El sistema bloqueará el cuadre si hay dinero salido sin justificación digital.

### Cierre de Caja (6:00 PM)

1. Ana hace clic en **"Iniciar Cierre de Caja"** → el sistema congela nuevas órdenes
2. El sistema muestra la matriz de balance cruzado en tres columnas:
   - **Columna 1:** Total esperado en medios digitales (QR + webhooks)
   - **Columna 2:** Total esperado en POS de tarjetas (vouchers físicos)
   - **Columna 3:** Total esperado en efectivo
3. Ana vacía la gaveta física y cuenta el dinero con cuidado
4. Ingresa el monto exacto en "Efectivo Real en Caja"

### Manejo de Diferencias (Crítico)

| Diferencia | Acción del sistema |
|-----------|-------------------|
| < S/.5 | Cierre normal. Log informativo |
| S/.5 - S/.50 | Cierre con flag DISCREPANCY. Ana debe escribir justificación. Notificación informativa a owners |
| > S/.50 | Cierre bloqueado. Campo obligatorio de justificación. Alerta P1 a owners. Log forense en audit_log con timestamp exacto para cruzar con cámaras de seguridad |

**Al enviar un cierre con diferencia:** El sistema inyecta automáticamente una notificación de auditoría forense con la hora exacta, para que los owners puedan revisar las cámaras de seguridad de esa franja horaria.

## 5. Reporte Diario a Owners

Al cierre de caja, el sistema envía automáticamente a Edgar y Juan vía WhatsApp:

```
📊 Cierre de Caja — [Fecha]
Ingresos del día: S/.X,XXX
Gastos del día: S/.XXX
Saldo neto: S/.X,XXX
Diferencia de caja: ±S/.XX
OTs completadas: XX
Estado: ✅ Sin diferencias / ⚠️ Diferencia detectada
```
