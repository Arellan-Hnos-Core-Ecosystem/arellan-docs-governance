eventos en tiempo real para mobile-app
# WebSocket Events — Tiempo Real

Eventos emitidos por el servidor vía WebSocket (Socket.io) para actualizar interfaces en tiempo real. Principalmente consumidos por `arellan-mobile-app` y `arellan-frontend-web`.

## Conexión

```javascript
const socket = io('wss://api.arellan.pe', {
  auth: { token: accessToken },
  transports: ['websocket'],
})

socket.on('connect', () => console.log('Conectado al servidor de eventos'))
socket.on('disconnect', () => console.log('Desconectado'))
```

## Eventos del Servidor → Cliente

### `order:status_changed`

Cuando una OT cambia de estado.

```typescript
interface OrderStatusChangedEvent {
  type: 'order:status_changed'
  orderId: string
  plate: string
  previousStatus: OrderStatus
  newStatus: OrderStatus
  changedBy: { id: string; name: string }
  timestamp: string
}
```

**Quién lo recibe:** Todos los usuarios con acceso a la OT (según rol)

---

### `cashbox:discrepancy_detected`

Cuando se detecta una diferencia de caja al cerrar.

```typescript
interface CashboxDiscrepancyEvent {
  type: 'cashbox:discrepancy_detected'
  cashboxSessionId: string
  discrepancy: number   // Negativo = faltante, positivo = sobrante
  closedBy: string
  timestamp: string
}
```

**Quién lo recibe:** OWNER (siempre), ADMIN (si discrepancy > S/.10)

---

### `vehicle:joyride_alert`

Cuando un vehículo del taller sale del geofence sin autorización.

```typescript
interface VehicleJoyrideAlertEvent {
  type: 'vehicle:joyride_alert'
  vehicleId: string
  plate: string
  currentLocation: { lat: number; lng: number }
  assignedEmployeeId: string | null
  timestamp: string
}
```

**Quién lo recibe:** Solo OWNER — prioridad máxima

---

### `expense:approval_required`

Cuando se crea un gasto que requiere aprobación.

```typescript
interface ExpenseApprovalRequiredEvent {
  type: 'expense:approval_required'
  expenseId: string
  amount: number
  description: string
  requestedBy: string
  requiredApprovalLevel: 'FINANCE' | 'ADMIN' | 'OWNER' | 'DUAL_OWNER'
  timestamp: string
}
```

**Quién lo recibe:** El nivel de aprobación requerido

---

### `inventory:low_stock`

Cuando un ítem de inventario cae al nivel mínimo.

```typescript
interface InventoryLowStockEvent {
  type: 'inventory:low_stock'
  itemId: string
  itemName: string
  currentStock: number
  minimumStock: number
  timestamp: string
}
```

**Quién lo recibe:** OWNER, ADMIN

---

### `attendance:after_hours_presence`

Cuando el sensor IoT detecta presencia en zonas restringidas fuera de horario.

```typescript
interface AfterHoursPresenceEvent {
  type: 'attendance:after_hours_presence'
  zone: 'storage' | 'accounting_office'
  detectedAt: string
  timestamp: string
}
```

**Quién lo recibe:** Solo OWNER — push silencioso

---

### `vehicle:location_update`

Actualización de ubicación GPS en tiempo real de vehículos del taller.

```typescript
interface VehicleLocationUpdateEvent {
  type: 'vehicle:location_update'
  vehicleId: string
  plate: string
  lat: number
  lng: number
  insideFence: boolean
  timestamp: string
}
```

**Quién lo recibe:** OWNER, ADMIN (si tienen el mapa abierto)
**Frecuencia:** Cada 30 segundos

---

## Eventos del Cliente → Servidor

### `join:vehicle_tracking`

Suscribirse a los updates de tracking GPS de un vehículo específico.

```javascript
socket.emit('join:vehicle_tracking', { vehicleId: 'uuid' })
```

### `leave:vehicle_tracking`

```javascript
socket.emit('leave:vehicle_tracking', { vehicleId: 'uuid' })
```

## Rooms (Grupos de WebSocket)

| Room | Miembros | Eventos recibidos |
|------|---------|-------------------|
| `owners` | Edgar, Juan | Todos los eventos de alerta |
| `admins` | Ana + owners | cashbox, inventory, orders |
| `finance` | Hija + admins | cashbox, expenses |
| `mechanics` | Mecánicos activos | order:status_changed (sus OTs) |
| `vehicle:{id}` | Suscriptores al vehículo | vehicle:location_update |
