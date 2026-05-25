# Guía de Alertas — Administrador

Qué significa cada tipo de alerta y qué hacer cuando aparece.

## Cómo Llegan las Alertas

Las alertas llegan por dos canales:
1. **Dashboard del sistema** — en la sección Alertas (número rojo en la campana)
2. **WhatsApp** — para los dueños (Edgar y Juan) directamente en su celular

Las alertas de prioridad crítica también generan un sonido en el sistema si tienes la pestaña abierta.

## Tipos de Alerta

### 🔴 CRÍTICA: Vehículo Fuera del Taller (Joyride)

**¿Qué pasó?**
Un vehículo propio del taller (camioneta o auto de servicio) salió de los 200 metros alrededor del taller sin autorización previa.

**¿Qué hacer?**
1. Ir a **Módulo Vehículos → Vehículos del Taller**
2. Ver el estado del vehículo alertado
3. ¿Tiene autorización `EN_USO_AUTORIZADO`?
   - **SÍ** → probablemente el radio fue superado por la ruta. Monitorear.
   - **NO** → llamar a Edgar de inmediato. No confrontes al empleado solo.
4. El sistema guarda la ubicación GPS con timestamp — es evidencia.

**Nota:** Solo los dueños pueden autorizar el uso de vehículos del taller. Tú como Admin puedes ver el estado pero no autorizar.

---

### 🟠 ALTA: Discrepancia en Cierre de Caja

**¿Qué pasó?**
Al cerrar la caja, el efectivo contado no coincide con lo que debería haber según el sistema.

- Número **negativo** (ej: -S/.50) = hay menos dinero del esperado (faltante)
- Número **positivo** (ej: +S/.20) = hay más dinero del esperado (sobrante)

**¿Qué hacer?**
1. Si la diferencia es ≤ S/.5: puede ser error de conteo — volver a contar. Si persiste, registrar con justificación.
2. Si la diferencia es > S/.5 y ≤ S/.50: registrar con justificación detallada. Edgar recibe notificación automática.
3. Si la diferencia es > S/.50: **no cierres la caja aún** — llama a Edgar o Juan antes de proceder.

Ver [finance/cashbox-daily.md](../finance/cashbox-daily.md) sección "Manejo de Descuadres" para el procedimiento completo.

---

### 🟡 MEDIA: Gasto Pendiente de Aprobación

**¿Qué pasó?**
Valeria (Finance) u otro usuario registró una solicitud de gasto que requiere tu aprobación como Admin.

**Cuándo te corresponde aprobar a ti:**
- Gastos entre S/.101 y S/.500 → el Admin aprueba

**¿Qué hacer?**
1. Ir a **Módulo Finanzas → Gastos → Pendientes**
2. Ver los detalles: monto, descripción, categoría, cotización adjunta
3. ¿El gasto tiene sustento? (cotización, factura, descripción clara)
   - **SÍ** → clic en Aprobar → requiere tu código MFA
   - **NO** → clic en Rechazar con motivo
4. Si el gasto es > S/.500 → **no puedes aprobarlo tú** — Edgar lo recibe directamente por WhatsApp

**Regla anti-fraude:** No puedes aprobar un gasto que tú misma creaste.

---

### 🟡 MEDIA: Stock Bajo en Inventario

**¿Qué pasó?**
Un ítem del inventario llegó al nivel mínimo o lo superó.

**¿Qué hacer?**
1. Ir a **Módulo Inventario → Alertas de Stock**
2. Ver el ítem con stock bajo, la cantidad actual y la mínima
3. Iniciar proceso de compra nacional (ver [knowledge-base/inventory-replenishment.md](../../arellan-knowledge-base/business-processes/inventory-replenishment.md))
4. Si es urgente (hay OTs esperando ese ítem) → hay mecánicos con OT en estado `EN_ESPERA_REPUESTO`

---

### 🟡 MEDIA: Orden de Trabajo Sin Movimiento

**¿Qué pasó?**
Una OT lleva más de 24 horas sin cambio de estado o actualización del mecánico.

**¿Qué hacer?**
1. Verificar físicamente con el mecánico qué está pasando
2. Si el mecánico terminó pero no cerró la OT en el sistema → pedirle que lo haga desde su tablet
3. Si hay un problema técnico (tablet sin batería, sin WiFi) → tú puedes actualizar el estado desde el dashboard

---

### ⚪ BAJA: Presencia Fuera de Horario (Solo Owners ven esto)

**¿Qué pasó?**
El sensor IoT detectó movimiento en la zona de almacén u oficina contable fuera del horario laboral (después de 7 PM o antes de 6 AM).

**Esta alerta es solo para Edgar y Juan.** Como Admin no la recibes directamente, pero los dueños pueden consultarte si la investigación lo requiere.

---

## Si No Sabes Qué Hacer

Ante cualquier alerta que no entiendas:
1. **No ignores la alerta** — registra que la viste (timestamp queda en el sistema)
2. **No intentes "arreglar" algo que no entiendes** — puedes empeorar la situación
3. **Llama a Edgar** — él siempre tiene acceso total al sistema desde su celular
