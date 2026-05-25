# Vehicles Endpoints

## Vehículos de Clientes (en OTs)

### POST /vehicles/intake

Registrar el ingreso de un vehículo de cliente.

**Roles:** MECHANIC, ADMIN

**Request (multipart/form-data):**
```
workOrderId: uuid
plate: ABC-123
model: Toyota Corolla
year: 2019
mileage: 45230
fuelLevel: HALF     # EMPTY | QUARTER | HALF | THREE_QUARTER | FULL
clientDescription: "Hace ruido al frenar"
photos[]: [archivo-frontal.jpg]    # 5 fotos obligatorias
photos[]: [archivo-posterior.jpg]
photos[]: [archivo-lateral-izq.jpg]
photos[]: [archivo-lateral-der.jpg]
photos[]: [archivo-tablero.jpg]
clientSignature: [firma-digital-base64]
```

**Response 201:**
```json
{
  "intakeId": "uuid",
  "plate": "ABC-123",
  "photos": [
    {
      "angle": "FRONT",
      "s3Url": "https://s3.amazonaws.com/...",
      "sha256Hash": "abc123..."   // Hash inmutable de la foto
    }
  ],
  "geofenceActive": true,
  "intakeCompletedAt": "2024-01-15T08:30:00.000Z"
}
```

---

### GET /vehicles/:plate/status

Consultar el estado actual de un vehículo (endpoint público para portal de clientes).

**Auth requerida:** No (público)
**Rate limit:** 30 req/min por IP

**Response 200:**
```json
{
  "plate": "ABC-123",
  "vehicleModel": "Toyota Corolla 2019",
  "currentStatus": "EN_PROCESO",
  "statusLabel": "Trabajo en progreso, tiempo estimado: 2 horas",
  "estimatedDelivery": "2024-01-15T17:00:00.000Z",
  "lastUpdate": "2024-01-15T11:30:00.000Z"
}
```

**Nota:** Este endpoint NO expone datos del cliente (nombre, teléfono, etc.) — solo estado del vehículo.

---

## Vehículos del Taller (Propiedad de Arellan)

### GET /workshop-vehicles

Listar vehículos propios del taller.

**Roles:** OWNER, ADMIN

**Response 200:**
```json
{
  "data": [
    {
      "id": "uuid",
      "plate": "XYZ-789",
      "model": "Toyota Hilux 2020",
      "status": "DISPONIBLE",
      "lastLocation": { "lat": -12.1095, "lng": -77.0282 },
      "insideFence": true,
      "assignedTo": null
    }
  ]
}
```

---

### POST /workshop-vehicles/:id/authorize-use

Autorizar el uso de un vehículo del taller.

**Roles:** OWNER

**Request:**
```json
{
  "assignedTo": "employee-uuid",
  "destination": "Recoger cliente - Miraflores",
  "estimatedReturnAt": "2024-01-15T12:00:00.000Z",
  "maxDurationMinutes": 45
}
```

**Response 200:**
```json
{
  "authorizationId": "uuid",
  "vehicleId": "uuid",
  "status": "EN_USO_AUTORIZADO",
  "authorizedBy": "Edgar Arellan",
  "expiresAt": "2024-01-15T12:00:00.000Z"
}
```

---

### GET /workshop-vehicles/:id/tracking

Obtener el historial de tracking GPS de un vehículo.

**Roles:** OWNER

**Query params:** `?from=2024-01-15T00:00:00Z&to=2024-01-15T23:59:59Z`

**Response 200:**
```json
{
  "vehicleId": "uuid",
  "plate": "XYZ-789",
  "trackingPoints": [
    {
      "lat": -12.1095,
      "lng": -77.0282,
      "insideFence": true,
      "timestamp": "2024-01-15T08:00:00.000Z"
    }
  ],
  "joyrideAlerts": []   // Lista de alertas de joyride en el período
}
```
