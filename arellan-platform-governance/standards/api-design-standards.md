REST: status codes, paginación, errores
# Estándares de Diseño de API REST

Convenciones para el diseño de todos los endpoints del ecosistema Arellan. Seguir estos estándares garantiza que el frontend y los clientes puedan integrarse de forma predecible.

## Códigos HTTP

| Situación | Código | Cuándo usarlo |
|-----------|--------|--------------|
| Éxito con datos | `200 OK` | GET, PUT/PATCH que retornan el recurso actualizado |
| Creado | `201 Created` | POST que crea un recurso nuevo |
| Sin contenido | `204 No Content` | DELETE, PUT/PATCH sin body de respuesta |
| Petición inválida | `400 Bad Request` | Validación de DTO fallida |
| No autenticado | `401 Unauthorized` | Token ausente o inválido |
| Sin permisos | `403 Forbidden` | Token válido pero rol insuficiente o MFA requerida |
| No encontrado | `404 Not Found` | Recurso no existe |
| Conflicto | `409 Conflict` | Duplicado, estado incompatible |
| Rate limit | `429 Too Many Requests` | Throttling activo |
| Error interno | `500 Internal Server Error` | Error no controlado (nunca debería llegar al cliente) |

## Formato de Respuesta de Error

Todos los errores devuelven el mismo formato:

```typescript
interface ErrorResponse {
  statusCode: number
  message: string          // Descripción legible para el humano
  code: string             // Código de error para el cliente (para lógica condicional)
  timestamp: string        // ISO 8601
  path: string             // Endpoint que generó el error
}
```

**Ejemplos de `code`:**
```
'VALIDATION_FAILED'      → DTO inválido
'UNAUTHORIZED'           → Sin token
'MFA_REQUIRED'           → MFA necesaria para este módulo
'INSUFFICIENT_ROLE'      → Rol no tiene permisos
'RESOURCE_NOT_FOUND'     → Recurso no existe
'SELF_APPROVAL_FORBIDDEN' → Empleado no puede aprobar su propio gasto
'CASHBOX_ALREADY_OPEN'   → Caja ya fue abierta hoy
'STOCK_INSUFFICIENT'     → No hay suficiente stock
'RATE_LIMIT_EXCEEDED'    → Demasiadas peticiones
```

## Paginación

Todos los endpoints que devuelven listas usan paginación cursor-based:

```typescript
// Query params
GET /api/v1/orders?limit=20&cursor=uuid-del-ultimo-item

// Respuesta
interface PaginatedResponse<T> {
  data: T[]
  pagination: {
    total: number
    limit: number
    cursor: string | null      // null si no hay más páginas
    hasNextPage: boolean
  }
}
```

**Por qué cursor-based y no offset:**
- Offset tiene el problema de "registro saltado" si se agregan items entre páginas
- Para el audit_log con miles de registros, el cursor es más eficiente

## Estructura de URL

```
Base URL: https://api.arellan.pe/api/v1

Recursos:
GET    /orders                    → listar OTs (paginado)
POST   /orders                    → crear OT
GET    /orders/:id                → obtener OT por ID
PATCH  /orders/:id                → actualizar OT parcialmente
DELETE /orders/:id                → solo soft-delete (status CANCELADO)

Sub-recursos:
GET    /orders/:id/photos         → fotos de la OT
POST   /orders/:id/status         → cambiar estado de la OT
POST   /orders/:id/payment/qr     → generar QR de pago

Acciones (verbos):
POST   /cashbox/open
POST   /cashbox/close
POST   /expenses/:id/approve
POST   /expenses/:id/reject
POST   /auth/mfa/verify
DELETE /auth/logout
```

## Filtros y Búsqueda

```
GET /orders?status=EN_PROCESO&mechanicId=uuid&from=2024-01-01&to=2024-01-31
GET /inventory?category=ACEITES&lowStock=true
GET /audit-logs?action=CASHBOX_DISCREPANCY&from=2024-01-01
```

Los filtros van como query params. Nunca en el body de un GET.

## Versionado

```
/api/v1/...   → versión actual
/api/v2/...   → próxima versión (cuando haya breaking changes)
```

Cuando se introduce `/v2`, `/v1` se mantiene activo por al menos 6 meses.

## Headers Requeridos

```
Authorization: Bearer <jwt_access_token>
Content-Type: application/json     (en requests con body)
X-Request-ID: <uuid>               (para trazabilidad, generado por el cliente)
```

## Responses que Nunca Deben Ocurrir

```typescript
// ❌ Nunca exponer stack traces al cliente
{
  "error": "TypeError: Cannot read property 'id' of undefined\n    at ExpenseService.approve...",
}

// ✅ Correcto: mensaje genérico + log interno del error real
{
  "statusCode": 500,
  "message": "Error interno del servidor",
  "code": "INTERNAL_ERROR",
  "timestamp": "2024-01-15T08:05:32.000Z",
  "path": "/api/v1/expenses/uuid/approve"
}
```

El stack trace se registra en Sentry/logs pero nunca se expone al cliente.
