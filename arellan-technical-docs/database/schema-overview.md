# Database Schema Overview

PostgreSQL 15. ORM: Prisma 5. Todas las PKs son UUID v4.

## Módulos y Tablas

```
accounts              ← Usuarios del sistema
account_sessions      ← JWT / refresh tokens activos
mfa_configs           ← Configuración TOTP por usuario

work_orders           ← Órdenes de Trabajo (OTs)
work_order_services   ← Líneas de servicio dentro de una OT
vehicle_intakes       ← Registro fotográfico de ingreso de vehículos
vehicle_intake_photos ← 5 fotos obligatorias con SHA-256

cashbox_sessions      ← Apertura/cierre de caja diaria
financial_transactions ← Ingresos y egresos
payments              ← Pagos QR + webhook Culqi
expense_requests      ← Solicitudes de gasto
expense_approvals     ← Historial de aprobaciones

inventory_items       ← Catálogo de repuestos e insumos
inventory_movements   ← Entradas, salidas, ajustes
purchase_orders       ← Órdenes de compra nacionales
import_orders         ← Órdenes de importación con validación de margen
providers             ← Proveedores (cuenta bancaria AES-256)

employees             ← Datos laborales
attendance_records    ← Registros biométricos ZKTeco
employee_incidents    ← Incidencias disciplinarias (Ley 728)

workshop_vehicles     ← Vehículos propios del taller
vehicle_authorizations ← Autorizaciones de uso
vehicle_tracking_logs ← Historial GPS (30s interval)

audit_log             ← INMUTABLE — toda acción crítica
```

## Convenciones

```sql
-- UUID en todas las PKs
id UUID DEFAULT gen_random_uuid() PRIMARY KEY

-- Timestamps con timezone Lima (UTC-5)
created_at TIMESTAMPTZ DEFAULT NOW()
updated_at TIMESTAMPTZ DEFAULT NOW()

-- Soft delete (nunca DELETE físico excepto audit_log que prohíbe DELETE)
deleted_at TIMESTAMPTZ NULL  -- NULL = activo

-- Dinero: NUNCA FLOAT
amount NUMERIC(10,2)
```

## Diagrama de Relaciones Clave

```
accounts (1) ────────── (N) work_orders
    │                           │
    │                           └── (N) work_order_services
    │                           └── (1) vehicle_intake
    │                                       │
    │                                       └── (N) vehicle_intake_photos
    │
    └── (N) attendance_records (via biometric ZKTeco)
    └── (N) expense_requests
    └── (N) audit_log (every action)

cashbox_sessions (1) ── (N) financial_transactions
cashbox_sessions (1) ── (N) payments

inventory_items (1) ─── (N) inventory_movements
inventory_items (1) ─── (N) work_order_services (consumed)

workshop_vehicles (1) ── (N) vehicle_authorizations
workshop_vehicles (1) ── (N) vehicle_tracking_logs
```

## Audit Log Inmutable

```sql
CREATE TABLE audit_log (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  account_id  UUID REFERENCES accounts(id),
  action      VARCHAR(100) NOT NULL,
  entity_type VARCHAR(50)  NOT NULL,
  entity_id   UUID,
  before_data JSONB,
  after_data  JSONB,
  ip_address  INET,
  user_agent  TEXT,
  created_at  TIMESTAMPTZ DEFAULT NOW()
);

-- Inmutabilidad: REGLAS de PostgreSQL (no triggers, que pueden deshabilitarse)
CREATE RULE audit_log_no_delete AS
  ON DELETE TO audit_log DO INSTEAD NOTHING;

CREATE RULE audit_log_no_update AS
  ON UPDATE TO audit_log DO INSTEAD NOTHING;
```

Ni el superusuario de la aplicación puede borrar o modificar registros del audit_log. Solo el DBA con acceso directo al servidor puede hacerlo — y eso queda registrado a nivel de sistema operativo.

## Tipos ENUM

```sql
-- Roles del sistema
CREATE TYPE user_role AS ENUM (
  'OWNER', 'ADMIN', 'FINANCE', 'MECHANIC', 'TRAINEE', 'CLIENT'
);

-- Estados de OT
CREATE TYPE order_status AS ENUM (
  'RECIBIDO', 'EN_PROCESO', 'EN_ESPERA_REPUESTO',
  'LISTO', 'ENTREGADO', 'CANCELADO'
);

-- Nivel de combustible al ingreso
CREATE TYPE fuel_level AS ENUM (
  'EMPTY', 'QUARTER', 'HALF', 'THREE_QUARTER', 'FULL'
);

-- Métodos de verificación biométrica
CREATE TYPE verify_method AS ENUM (
  'HUELLA', 'FACIAL', 'PIN', 'CARD'
);

-- Estados de caja
CREATE TYPE cashbox_status AS ENUM (
  'OPEN', 'CLOSED_OK', 'CLOSED_WITH_DISCREPANCY'
);

-- Métodos de pago
CREATE TYPE payment_method AS ENUM (
  'YAPE', 'PLIN', 'TARJETA', 'EFECTIVO', 'TRANSFERENCIA'
);
```

## Campos Encriptados

Datos sensibles encriptados con AES-256 a nivel de aplicación antes de guardar:

| Tabla | Campo | Por qué |
|-------|-------|---------|
| `providers` | `bank_account` | Datos bancarios de proveedores |
| `mfa_configs` | `totp_secret` | Secreto TOTP del autenticador |
| `accounts` | `password_hash` | bcrypt (hash, no encriptación) |

Ver `arellan-platform-governance/standards/security-standards.md` para implementación.
