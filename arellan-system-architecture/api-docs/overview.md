
# API Documentation Overview

## Base URL

```
Producción:  https://api.arellan.pe/api/v1
Staging:     https://staging.api.arellan.pe/api/v1
Local:       http://localhost:3001/api/v1
```

## Autenticación

Todos los endpoints (excepto los marcados como públicos) requieren el header:

```
Authorization: Bearer <access_token>
```

Para obtener un access token: ver `auth-endpoints.md`.

## Módulos de la API

| Módulo | Prefijo | Descripción |
|--------|---------|-------------|
| Auth | `/auth` | Login, MFA, logout, refresh token |
| Orders | `/orders` | Órdenes de trabajo completas |
| Finance | `/finance` | Caja, transacciones, gastos, QR de pago |
| Inventory | `/inventory` | Stock, ítems, movimientos |
| Clients | `/clients` | Clientes del taller |
| Vehicles | `/vehicles` | Vehículos en taller + vehículos del taller |
| Employees | `/employees` | Gestión de personal y asistencia |
| Audit | `/audit` | Solo lectura del audit log |
| Reports | `/reports` | Generación de reportes PDF/Excel |
| Webhooks | `/webhooks` | Webhooks de terceros (Culqi, ZKTeco) |

## Formato de Respuestas

### Respuesta Exitosa

```json
{
  "id": "uuid",
  "status": "EN_PROCESO",
  "createdAt": "2024-01-15T08:05:32.000Z"
}
```

O para listas paginadas:

```json
{
  "data": [...],
  "pagination": {
    "total": 150,
    "limit": 20,
    "cursor": "uuid-del-ultimo",
    "hasNextPage": true
  }
}
```

### Respuesta de Error

```json
{
  "statusCode": 403,
  "message": "Este módulo requiere verificación MFA activa",
  "code": "MFA_REQUIRED",
  "timestamp": "2024-01-15T08:05:32.000Z",
  "path": "/api/v1/finance/transactions"
}
```

## Rate Limiting

| Módulo | Límite |
|--------|--------|
| General | 100 req/min por usuario |
| Finance | 10 req/min por usuario |
| Auth (login) | 5 intentos/min por IP |
| Portal público | 30 req/min por IP |

Al superar el límite: `429 Too Many Requests` con header `Retry-After`.

## WebSocket (Tiempo Real)

Conexión en: `wss://api.arellan.pe/events`

Requiere el access token en el query param:
```
wss://api.arellan.pe/events?token=<access_token>
```

Ver `websocket-events.md` para el listado de eventos.
