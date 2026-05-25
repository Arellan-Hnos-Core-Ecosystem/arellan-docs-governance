# Inventory Endpoints

## GET /inventory/items

Listar ítems del inventario.

**Roles:** OWNER, ADMIN, FINANCE, MECHANIC, TRAINEE

**Query params:** `?category=ACEITES&lowStock=true&search=filtro&limit=20&cursor=uuid`

**Response 200:**
```json
{
  "data": [
    {
      "id": "uuid",
      "sku": "ACE-10W40-1L",
      "name": "Aceite Motor 10W-40 1L",
      "category": "ACEITES",
      "currentStock": 8,
      "minimumStock": 5,
      "unit": "litros",
      "unitCost": 25.00,
      "isLowStock": false
    }
  ],
  "pagination": { "total": 120, "hasNextPage": true, "cursor": "uuid" }
}
```

---

## GET /inventory/items/:id

Obtener detalle de un ítem.

**Roles:** Todos (mecánicos no ven el costo unitario)

**Response 200:**
```json
{
  "id": "uuid",
  "sku": "ACE-10W40-1L",
  "name": "Aceite Motor 10W-40 1L",
  "category": "ACEITES",
  "currentStock": 8,
  "minimumStock": 5,
  "unitCost": 25.00,       // Oculto para mecánicos
  "lastMovementAt": "2024-01-14T15:00:00.000Z",
  "movements": [...]        // Últimos 10 movimientos
}
```

---

## POST /inventory/movements

Registrar un movimiento de inventario (entrada o salida).

**Roles:** ADMIN, OWNER

**Request:**
```json
{
  "itemId": "uuid",
  "type": "IN",           // IN | OUT | ADJUSTMENT
  "quantity": 10,
  "unitCost": 24.50,      // Requerido para tipo IN
  "reason": "PURCHASE",   // PURCHASE | WORK_ORDER | ADJUSTMENT | LOSS
  "workOrderId": null,    // Requerido si reason = WORK_ORDER
  "notes": "Compra mensual Proveedor X"
}
```

**Response 201:**
```json
{
  "id": "uuid",
  "itemId": "uuid",
  "newStock": 18,
  "newAverageCost": 24.75,   // Precio promedio ponderado actualizado
  "createdAt": "2024-01-15T09:00:00.000Z"
}
```

---

## POST /inventory/items/:id/request

Solicitar un ítem para una OT (mecánico).

**Roles:** MECHANIC, TRAINEE

**Request:**
```json
{
  "workOrderId": "uuid",
  "quantity": 2,
  "urgency": "HIGH",
  "notes": "Necesario para completar el cambio de aceite"
}
```

**Response 201:**
```json
{
  "id": "uuid",
  "status": "PENDING",       // PENDING si no hay stock → genera solicitud de compra
  "status": "RESERVED",      // RESERVED si hay stock disponible
  "reservedQuantity": 2
}
```

---

## GET /inventory/alerts

Obtener ítems con stock crítico.

**Roles:** OWNER, ADMIN, FINANCE

**Response 200:**
```json
{
  "data": [
    {
      "id": "uuid",
      "name": "Pastillas de freno delanteras",
      "currentStock": 1,
      "minimumStock": 2,
      "daysUntilStockout": 3,       // Estimado basado en consumo histórico
      "suggestedOrderQuantity": 5
    }
  ],
  "totalAlerts": 3
}
```
