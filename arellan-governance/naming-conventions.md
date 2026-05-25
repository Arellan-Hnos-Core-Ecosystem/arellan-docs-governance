# Convenciones de Nombres

Estándares de nomenclatura aplicables en todo el ecosistema de 13 repositorios. La consistencia en los nombres hace el código más legible y facilita el trabajo entre repositorios.

## Código TypeScript/JavaScript

### Variables y Funciones → camelCase

```typescript
// Variables
const totalAmount = 1500.00
const isPendingApproval = true
const workOrderId = 'uuid-...'
const currentCashboxSession = await getCashboxSession()

// Funciones
function calculateDynamicInvoiceQr(orderId: string): Promise<QrData>
function validateExpenseApprovalLevel(amount: number): ApprovalLevel
function getOffHoursAttendanceAlerts(): Alert[]

// Métodos de clase
class ExpenseService {
  async createExpense(dto: CreateExpenseDto): Promise<Expense>
  async approveExpense(id: string, approverId: string): Promise<Expense>
  getRequiredApprovalLevel(amount: number): ApprovalLevel
}
```

### Clases e Interfaces → PascalCase

```typescript
// Clases NestJS
class ExpenseAuthorizationService
class MfaRequiredGuard
class AuditTrailInterceptor
class CashboxReportWorker

// Interfaces / Types
interface AuthUser {
  id: string
  role: UserRole
  mfaVerified: boolean
}

interface WorkOrder {
  id: string
  status: OrderStatus
  assignedMechanicId: string
}

// DTOs
class CreateExpenseDto
class UpdateOrderStatusDto
class GenerateQrPaymentDto
```

### Enums → PascalCase (nombre) + SCREAMING_SNAKE_CASE (valores)

```typescript
enum UserRole {
  OWNER = 'OWNER',
  ADMIN = 'ADMIN',
  FINANCE = 'FINANCE',
  MECHANIC = 'MECHANIC',
  TRAINEE = 'TRAINEE',
  CLIENT = 'CLIENT',
}

enum OrderStatus {
  RECIBIDO = 'RECIBIDO',
  EN_DIAGNOSTICO = 'EN_DIAGNOSTICO',
  PRESUPUESTADO = 'PRESUPUESTADO',
  EN_PROCESO = 'EN_PROCESO',
  EN_REVISION = 'EN_REVISION',
  LISTO = 'LISTO',
  ENTREGADO = 'ENTREGADO',
  CANCELADO = 'CANCELADO',
}

enum ExpenseStatus {
  PENDING = 'PENDING',
  PENDING_APPROVAL = 'PENDING_APPROVAL',
  APPROVED = 'APPROVED',
  REJECTED = 'REJECTED',
  PAID = 'PAID',
}
```

### Constantes → SCREAMING_SNAKE_CASE

```typescript
const MAX_QR_EXPIRY_MINUTES = 7
const MIN_CASHBOX_DISCREPANCY_ALERT = 10
const GEOFENCE_RADIUS_METERS = 200
const JWT_ACCESS_TOKEN_EXPIRY_HOURS = 1
const MECHANIC_SESSION_EXPIRY_HOURS = 12
```

## Base de Datos (PostgreSQL + Prisma)

### Tablas → snake_case, plural

```sql
accounts           -- usuarios del sistema
work_orders        -- órdenes de trabajo
inventory_items    -- ítems del inventario
inventory_movements
financial_transactions
cashbox_sessions
expense_authorizations
audit_logs
refresh_tokens
push_subscriptions
attendance_records
workshop_vehicles
vehicle_tracking_logs
```

### Columnas → snake_case

```sql
id, created_at, updated_at
employee_id, account_id, work_order_id
total_amount, parts_cost, labor_cost
check_in_at, check_out_at
is_active, has_discrepancy
```

### Índices → idx_[tabla]_[columna(s)]

```sql
idx_audit_logs_entity
idx_work_orders_mechanic_status
idx_inventory_items_sku
idx_financial_transactions_cashbox_date
```

## API REST

### Endpoints → kebab-case, plural, sustantivos

```
GET    /api/v1/work-orders
POST   /api/v1/work-orders
GET    /api/v1/work-orders/:id
PATCH  /api/v1/work-orders/:id/status

GET    /api/v1/finance/transactions
POST   /api/v1/finance/expenses
POST   /api/v1/finance/cashbox/open
POST   /api/v1/finance/cashbox/close

GET    /api/v1/inventory/items
POST   /api/v1/inventory/movements

GET    /api/v1/auth/profile
POST   /api/v1/auth/login
POST   /api/v1/auth/mfa/verify
DELETE /api/v1/auth/logout
```

### Acciones en Recursos → verbos después del recurso

```
POST /work-orders/:id/assign-mechanic   ← asignar mecánico
POST /work-orders/:id/complete          ← completar OT
POST /expenses/:id/approve              ← aprobar gasto
POST /expenses/:id/reject               ← rechazar gasto
POST /cashbox/open                      ← abrir caja
POST /cashbox/close                     ← cerrar caja
```

## Archivos y Carpetas

### Código → kebab-case

```
arellan-backend-api/
  src/
    modules/
      work-orders/
        work-orders.module.ts
        work-orders.controller.ts
        work-orders.service.ts
        dto/
          create-work-order.dto.ts
          update-order-status.dto.ts
```

### Documentación → kebab-case.md

```
arellan-docs-governance/
  arellan-technical-docs/
    database/
      schema-overview.md
      migrations-guide.md
      indexes-and-performance.md
```

## BullMQ Jobs

### Queues → kebab-case

```typescript
'notifications'       // Envío de notificaciones (push/email/WhatsApp)
'inventory-alerts'    // Alertas de stock crítico
'reports'             // Generación de reportes PDF/Excel
'audit-anomalies'     // Detección de anomalías en audit log
'backups'             // Backups automáticos
'cashbox-reports'     // Reportes diarios de caja
```

### Job names → dominio.acción.complemento

```typescript
'notification.push.send'
'notification.email.send'
'inventory.alert.low-stock'
'report.pdf.generate'
'cashbox.report.daily'
'audit.anomaly.detect'
```

## WebSocket Events

### Eventos → dominio:acción

```typescript
'order:status_changed'
'cashbox:discrepancy_detected'
'vehicle:joyride_alert'
'expense:approval_required'
'inventory:low_stock'
'attendance:after_hours_presence'
```
