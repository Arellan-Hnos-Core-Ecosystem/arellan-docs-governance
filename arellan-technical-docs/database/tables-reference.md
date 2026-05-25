# Tables Reference

Referencia completa de campos por tabla. Generado desde Prisma schema.

## accounts

| Campo | Tipo | Nullable | Descripción |
|-------|------|----------|-------------|
| id | UUID | NO | PK |
| email | VARCHAR(255) | NO | UNIQUE — email corporativo `@arellan.pe` |
| password_hash | TEXT | NO | bcrypt rounds=12 |
| full_name | VARCHAR(255) | NO | Nombre completo |
| role | user_role ENUM | NO | OWNER/ADMIN/FINANCE/MECHANIC/TRAINEE/CLIENT |
| status | account_status | NO | ACTIVE/INACTIVE/TERMINATED |
| zkTeco_biometric_id | VARCHAR(20) | YES | ID en reloj ZKTeco (ej: "E004") |
| phone | VARCHAR(20) | YES | AES-256 encriptado |
| mfa_enabled | BOOLEAN | NO | DEFAULT false |
| last_login_at | TIMESTAMPTZ | YES | |
| created_at | TIMESTAMPTZ | NO | DEFAULT NOW() |
| updated_at | TIMESTAMPTZ | NO | DEFAULT NOW() |
| deleted_at | TIMESTAMPTZ | YES | NULL = activo, soft delete |

## work_orders

| Campo | Tipo | Nullable | Descripción |
|-------|------|----------|-------------|
| id | UUID | NO | PK |
| order_number | VARCHAR(20) | NO | UNIQUE — formato "OT-2024-0001" |
| client_id | UUID | NO | FK → accounts (role=CLIENT) |
| plate | VARCHAR(10) | NO | Placa del vehículo |
| vehicle_model | VARCHAR(100) | NO | "Toyota Corolla 2019" |
| mileage_in | INT | NO | Kilometraje al ingreso |
| fuel_level | fuel_level | NO | ENUM |
| client_description | TEXT | NO | Síntoma descrito por el cliente |
| status | order_status | NO | DEFAULT 'RECIBIDO' |
| assigned_mechanic_id | UUID | YES | FK → accounts (role=MECHANIC) |
| estimated_delivery_at | TIMESTAMPTZ | YES | |
| completed_at | TIMESTAMPTZ | YES | Cuando status → LISTO |
| delivered_at | TIMESTAMPTZ | YES | Cuando status → ENTREGADO |
| total_amount | NUMERIC(10,2) | YES | Calculado al cerrar |
| is_paid | BOOLEAN | NO | DEFAULT false |
| created_at | TIMESTAMPTZ | NO | DEFAULT NOW() |

## cashbox_sessions

| Campo | Tipo | Nullable | Descripción |
|-------|------|----------|-------------|
| id | UUID | NO | PK |
| date | DATE | NO | UNIQUE — una caja por día |
| opened_by_id | UUID | NO | FK → accounts |
| closed_by_id | UUID | YES | FK → accounts |
| initial_amount | NUMERIC(10,2) | NO | Monto de apertura |
| expected_final_amount | NUMERIC(10,2) | YES | Calculado al cierre |
| actual_final_amount | NUMERIC(10,2) | YES | Contado físicamente |
| discrepancy | NUMERIC(10,2) | YES | actual - expected (negativo = faltante) |
| status | cashbox_status | NO | DEFAULT 'OPEN' |
| notes | TEXT | YES | |
| opened_at | TIMESTAMPTZ | NO | DEFAULT NOW() |
| closed_at | TIMESTAMPTZ | YES | |

## expense_requests

| Campo | Tipo | Nullable | Descripción |
|-------|------|----------|-------------|
| id | UUID | NO | PK |
| amount | NUMERIC(10,2) | NO | |
| description | TEXT | NO | |
| category | expense_category | NO | REPUESTOS_LOCALES/IMPORTACION/SERVICIOS/etc |
| provider_id | UUID | YES | FK → providers |
| quotation_url | TEXT | YES | URL de cotización en S3 |
| requested_by_id | UUID | NO | FK → accounts |
| required_approval_level | user_role | NO | Nivel requerido según monto |
| status | expense_status | NO | PENDING/APPROVED/REJECTED/DISBURSED |
| approved_by_id | UUID | YES | FK → accounts |
| approved_at | TIMESTAMPTZ | YES | |
| created_at | TIMESTAMPTZ | NO | DEFAULT NOW() |

**Regla:** `approved_by_id ≠ requested_by_id` — auto-aprobación imposible (validado en DB y en servicio)

## inventory_items

| Campo | Tipo | Nullable | Descripción |
|-------|------|----------|-------------|
| id | UUID | NO | PK |
| sku | VARCHAR(50) | NO | UNIQUE — ej: "ACE-10W40-1L" |
| name | VARCHAR(255) | NO | |
| category | inventory_category | NO | ACEITES/FILTROS/FRENOS/etc |
| current_stock | INT | NO | DEFAULT 0 |
| minimum_stock | INT | NO | Umbral de alerta |
| unit | VARCHAR(20) | NO | litros/unidades/kg |
| unit_cost | NUMERIC(10,2) | NO | Precio promedio ponderado |
| is_active | BOOLEAN | NO | DEFAULT true |
| created_at | TIMESTAMPTZ | NO | DEFAULT NOW() |
| updated_at | TIMESTAMPTZ | NO | DEFAULT NOW() |

## attendance_records

| Campo | Tipo | Nullable | Descripción |
|-------|------|----------|-------------|
| id | UUID | NO | PK |
| account_id | UUID | NO | FK → accounts |
| zkteco_device_sn | VARCHAR(50) | NO | Serial del reloj |
| event_type | attendance_event | NO | CHECK_IN/CHECK_OUT |
| verify_method | verify_method | NO | HUELLA/FACIAL/PIN/CARD |
| recorded_at | TIMESTAMPTZ | NO | Timestamp del evento (hardware time) |
| is_forced_checkout | BOOLEAN | NO | DEFAULT false — sistema lo marca a las 11:59 PM |
| created_at | TIMESTAMPTZ | NO | Insertion time (≠ recorded_at si hubo buffer) |

## audit_log

| Campo | Tipo | Nullable | Descripción |
|-------|------|----------|-------------|
| id | UUID | NO | PK |
| account_id | UUID | YES | NULL si acción de sistema |
| action | VARCHAR(100) | NO | CASHBOX_OPENED / EXPENSE_APPROVED / etc |
| entity_type | VARCHAR(50) | NO | WorkOrder / CashboxSession / etc |
| entity_id | UUID | YES | ID del registro afectado |
| before_data | JSONB | YES | Snapshot antes del cambio |
| after_data | JSONB | YES | Snapshot después del cambio |
| ip_address | INET | YES | IP del cliente |
| user_agent | TEXT | YES | |
| created_at | TIMESTAMPTZ | NO | DEFAULT NOW() |

**NO DELETE. NO UPDATE.** — PostgreSQL RULE. Ver `schema-overview.md`.

## workshop_vehicles

| Campo | Tipo | Nullable | Descripción |
|-------|------|----------|-------------|
| id | UUID | NO | PK |
| plate | VARCHAR(10) | NO | UNIQUE |
| model | VARCHAR(100) | NO | |
| status | vehicle_status | NO | DISPONIBLE/EN_USO_AUTORIZADO/ALERTA_JOYRIDE |
| last_lat | NUMERIC(10,7) | YES | Última latitud GPS |
| last_lng | NUMERIC(10,7) | YES | Última longitud GPS |
| inside_fence | BOOLEAN | NO | DEFAULT true |
| assigned_to_id | UUID | YES | FK → accounts (empleado con autorización activa) |
| gps_device_id | VARCHAR(50) | YES | ID del dispositivo GPS (ej: Teltonika IMEI) |
| is_active | BOOLEAN | NO | DEFAULT true |
| created_at | TIMESTAMPTZ | NO | DEFAULT NOW() |
