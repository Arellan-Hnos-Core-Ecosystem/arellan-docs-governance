# Proceso Operativo Estándar: Control de Caja y Flujo de Recaudación Digital

## 1. Diagnóstico del Dolor Financiero
La operación manual previa permitía que trabajadores con roles de confianza mostraran sus aplicativos personales (Yape/Plin personal) a los clientes para el cobro de mano de obra y repuestos, desviando el capital de la clínica hacia cuentas de terceros.

## 2. Arquitectura de Mitigación (Cero Intervención Humana)
Queda estrictamente prohibido que cualquier mecánico, practicante o administrador muestre un código QR físico o su número telefónico personal para procesar un pago dentro del local de Surquillo. El flujo de recaudación se automatiza por completo mediante el sistema central de la siguiente manera:

[Módulo de Liquidación (Frontend)] ──> [Solicitud API NestJS] ──> [Pasarela local (Culqi/Niubiz API)]
│
[Visualización de QR Único en Tablet] <── [Generación de QR Dinámico] <────────┘
│
[Pago del Cliente con Smartphone]
│
▼
[Webhook de la Pasarela] ──> [API Gateway (arellan-api-gateway)] ──> [Cierre de Orden y Desbloqueo de Auto]

## 3. Reglas Críticas del Flujo de Pago
1. **Generación de QR Dinámico:** Al finalizar la reparación, la encargada de finanzas hace clic en "Liquidar Orden" desde el portal administrativo. El sistema calcula el monto total (Mano de obra + Repuestos del inventario) y genera un token único que se comunica con la pasarela corporativa.
2. **Exhibición Controlada:** El sistema genera un código QR dinámico que se muestra exclusivamente en la pantalla de la tablet de liquidación orientada al cliente. Este QR expira automáticamente en 7 minutos.
3. **Procesamiento de Webhooks e Idempotencia:** La pasarela de pagos envía una notificación asíncrona (Webhook) firmada criptográficamente al repositorio `arellan-backend-core`. El sistema procesa la transacción utilizando una clave de idempotencia (`payment_id`) para evitar duplicaciones de transacciones.
4. **Contingencia por Falla de Red (Fallback Manual):** Si el webhook de la pasarela falla debido a microcortes de internet en Surquillo:
    - El sistema mantendrá la Orden de Trabajo como `PENDIENTE_VERIFICACIÓN`.
    - La encargada de finanzas ingresará al portal bancario corporativo utilizando su Token físico/MFA.
    - Deberá contrastar el número de operación bancaria con el campo temporal generado por el ERP.
    - Solo tras ingresar el código de verificación bancaria manual, el sistema cambiará el estado a `PAGADO` emitiendo un log forense inmutable de auditoría con la firma digital de la administradora.