# Proceso de Autorización de Gastos, Compras e Importaciones

## 1. Propósito del Sistema de Control

Evitar la salida discrecional de dinero en efectivo de la caja chica para compras de repuestos locales o pagos de comisiones no autorizadas en procesos de importación.

**El problema específico:** El empleado encargado de importaciones (Ricardo) coordinaba directamente con proveedores extranjeros cobrando comisiones del 20-30% que el taller pagaba sin saberlo. El costo real de los repuestos era S/.70-80 pero se facturaba S/.100, la diferencia iba al bolsillo del intermediario.

## 2. Ciclo de Vida y Estados de una Solicitud de Gasto

Toda salida de dinero de la clínica automotriz sigue estrictamente esta máquina de estados:

```
[REGISTRADO] ──> [NOTIFICADO al nivel de aprobación] ──> [APROBADO] ──> [DESEMBOLSADO] ──> [SUSTENTADO con XML/PDF]
                                                      └──> [RECHAZADO]
```

## 3. Niveles de Autorización

| Monto | Autorizador | Método | Plazo |
|-------|-------------|--------|-------|
| ≤ S/.100 | Finance (hija de Edgar) | Aprobación directa en sistema | Inmediato |
| S/.101 – S/.500 | Admin (Ana) | Aprobación en panel admin | < 2 horas |
| S/.501 – S/.2,000 | Owner (Edgar o Juan) | Push en arellan-mobile-app | < 4 horas |
| > S/.2,000 | Ambos Owners | Doble aprobación push | < 8 horas |

## 4. Reglas del Proceso

### Regla 1: Petición con Sustento

Cuando un mecánico requiera una pieza local o se procese el arancel de una importación, se registra una solicitud indicando obligatoriamente:
- Categoría del gasto: Repuestos Locales / Importaciones / Herramientas / Servicios Tercerizados
- Monto exacto en Soles o Dólares (con tipo de cambio si aplica)
- Proveedor seleccionado del directorio homologado (`supplier-directory.md`)
- Enlace obligatorio al PDF de la cotización o proforma del proveedor

### Regla 2: El Sistema Bloquea el Desembolso

El botón de "Pagar" en el sistema está deshabilitado hasta recibir la aprobación del nivel correspondiente. Ana no puede entregar dinero físico sin el check verde digital de aprobación. Si lo hace, el cierre de caja marcará una diferencia sin justificación.

### Regla 3: Aprobación Remota por Biometría

El software no libera la transacción hasta que el owner ingrese a la app móvil, verifique el sustento y presione "Aprobar" con su PIN biométrico (Face ID / Touch ID del teléfono). Esta aprobación queda en `audit_logs` con el `userId` del owner y el timestamp exacto.

### Regla 4: Auditoría de Importaciones

Para repuestos importados, el sistema calcula automáticamente el margen implícito:

```
margen = (precio_venta_histórico - precio_compra_importado) / precio_venta_histórico

Si margen > 35%: BLOQUEADO — requiere justificación escrita del owner
Si margen < 5%: ALERTA — precio anormalmente alto (posible sobrecosto)
```

Antes de este sistema: el empleado podía presentar un precio de $100 CIF cuando el precio real era $70, cobrando $30 de "comisión" del proveedor.

### Regla 5: Sustentación con SUNAT

Todo gasto autorizado tiene un plazo máximo de **48 horas** para ser sustentado con comprobante electrónico. Ana sube el XML o PDF de la Factura Electrónica. El backend valida automáticamente con la API de SUNAT que el documento esté activo, aprobado y corresponda al RUC del proveedor registrado.

**Gastos sin sustentación a las 72h:** Notificación automática a OWNER + flag de incumplimiento en el reporte mensual.

## 5. Categorías de Gasto

| Categoría | Descripción | Autorizador mínimo |
|-----------|-------------|-------------------|
| Repuestos locales | Compras en Lima/Surquillo | Finance |
| Importaciones | Repuestos EE.UU./China/Europa | Owner siempre |
| Herramientas | Equipo de trabajo del taller | Admin |
| Servicios tercerizados | Grúa, pintura externa, etc. | Admin |
| Gastos operativos | Agua, luz, internet, limpieza | Finance |
| Gastos de personal | Anticipos, préstamos | Owner siempre |
| Inversiones de capital | Equipos, mejoras de local | Ambos owners |

## 6. Gastos de Emergencia (< S/.50, horario nocturno)

Si surge una necesidad urgente fuera del horario de atención:
1. Ana o el mecánico registran el gasto en el sistema con categoría `EMERGENCIA`
2. Fotografía del ticket/recibo obligatoria al momento de la compra
3. Al día siguiente: registro del sustento formal
4. El owner recibe notificación de emergencia y confirma retroactivamente

Este flujo de emergencia se usa máximo 2 veces por mes. Si se usa más, el sistema alerta que hay un problema de gestión de inventario.
