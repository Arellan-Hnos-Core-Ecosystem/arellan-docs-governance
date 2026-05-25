# Solicitar Repuestos — Mecánicos

Cómo pedir una pieza o repuesto para tu orden de trabajo.

## La Regla

**Todo pedido de repuesto va por el sistema.** No le digas a Ana verbalmente — regístralo en la tablet. Esto asegura que el repuesto se descuenta del inventario correcto y se vincula a tu OT.

## Solicitar un Repuesto del Inventario

### Si el repuesto está en el taller

1. En tu tablet → abre la OT a la que necesitas el repuesto
2. Clic en **Solicitar Repuesto**
3. Busca el ítem: escribe el nombre o SKU (ej: "pastillas freno", "ACE-10W40")
4. Ingresa la cantidad necesaria
5. Selecciona urgencia: **NORMAL** o **ALTA**
6. Agrega nota si es necesario: "Para cambio de frenos delanteros"
7. Clic **Enviar Solicitud**

**Si hay stock disponible:** El sistema reserva las unidades para tu OT. Estado: `RESERVADO`. Ana te da la pieza físicamente.

**Si no hay stock:** Estado: `PENDIENTE`. Ana recibe la alerta para hacer el pedido de compra. Tu OT cambia a `EN_ESPERA_REPUESTO`.

### Cómo Saber si Hay Stock

En la búsqueda de repuestos, el sistema muestra:
- **Verde: XX unidades** — hay stock
- **Amarillo: X unidades (bajo stock)** — hay pero poco
- **Rojo: Sin stock** — hay que pedir

## Ver el Estado de tu Solicitud

1. En la OT → sección **Repuestos Solicitados**
2. Estado posibles:

| Estado | Qué significa |
|--------|---------------|
| RESERVADO | Hay stock, pasa por Ana para que te lo entregue |
| PENDIENTE | No hay stock, se está gestionando la compra |
| ENTREGADO | Ana ya te lo dio físicamente — confirmar en el sistema |

## Confirmar que Recibiste el Repuesto

Cuando Ana te entregue la pieza físicamente:
1. En la OT → Repuesto con estado `RESERVADO`
2. Clic **Confirmar Recepción**
3. El sistema descuenta del inventario y registra en tu OT

**No confirmes si no recibiste la pieza físicamente.** El inventario se descontará aunque no tengas el repuesto.

## Si Necesitas una Pieza que No Está en el Catálogo

Si buscas un repuesto y no aparece en el sistema:
1. Avísale a Ana verbalmente
2. Ella puede registrar el ítem nuevo en el catálogo de inventario
3. Luego tú procedes con la solicitud normal

## Urgencia ALTA — Cuándo Usarla

Solo cuando la pieza es crítica para completar la OT hoy y el cliente espera el vehículo. No marques todo como ALTA — si abusas, Ana no puede priorizar correctamente.

| Urgencia | Cuándo |
|---------|--------|
| NORMAL | Tienes 1-2 días de margen |
| ALTA | El cliente necesita el vehículo hoy |

## Si tu OT está en EN_ESPERA_REPUESTO

Significa que solicitaste un repuesto que no hay en stock y estamos esperando que llegue.

- No puedes hacer mucho más hasta que llegue la pieza
- Ana gestiona la compra y te notifica cuando llega
- La OT queda pausada automáticamente (no se cuenta en tus métricas de tiempo)
