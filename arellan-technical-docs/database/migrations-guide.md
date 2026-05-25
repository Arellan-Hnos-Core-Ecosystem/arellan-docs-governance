# Migrations Guide

Cómo crear, revisar y ejecutar migraciones de base de datos con Prisma Migrate.

## Flujo Estándar

### 1. Editar el Schema

```prisma
// prisma/schema.prisma
model InventoryItem {
  id           String   @id @default(uuid())
  sku          String   @unique
  name         String
  currentStock Int      @default(0)
  minimumStock Int
  // ... campos nuevos aquí
}
```

### 2. Crear la Migration

```bash
# Genera SQL en prisma/migrations/{timestamp}_{nombre}/migration.sql
npx prisma migrate dev --name add_inventory_supplier_field

# Ejemplo de nombre descriptivo:
# add_expense_quotation_url
# alter_accounts_add_mfa_enabled
# create_vehicle_tracking_logs
```

### 3. Revisar el SQL Generado

**CRÍTICO:** Siempre leer el SQL antes de aplicar.

```bash
# El archivo está en:
cat prisma/migrations/20240115120000_add_inventory_supplier_field/migration.sql
```

Verificar:
- `ALTER TABLE ... ADD COLUMN` con `DEFAULT` si la columna es `NOT NULL` (evita lock en tablas grandes)
- Nunca `DROP COLUMN` sin confirmar con el equipo primero
- Indexes están incluidos si se agregaron con `@@index`
- Reglas de audit_log siguen presentes si la migration toca esa tabla

### 4. Aplicar en Staging Primero

```bash
# En staging:
DATABASE_URL="postgresql://..." npx prisma migrate deploy

# Verificar que la app funciona en staging
# Luego aplicar en producción
```

### 5. Aplicar en Producción

```bash
# Railway (producción):
railway run npx prisma migrate deploy

# AWS RDS:
DATABASE_URL=$PROD_DATABASE_URL npx prisma migrate deploy
```

## Comandos de Referencia

```bash
# Desarrollo — aplica y regenera Prisma Client
npx prisma migrate dev

# Producción — solo aplica migrations pendientes (sin reset)
npx prisma migrate deploy

# Ver estado de migrations
npx prisma migrate status

# Reset completo (SOLO DEV — destruye datos)
npx prisma migrate reset

# Regenerar Prisma Client sin migration nueva
npx prisma generate

# Introspect DB existente → schema Prisma
npx prisma db pull
```

## Reglas de la Equipo

### NO HACER

```bash
# Nunca en producción:
npx prisma migrate reset        # Destruye todos los datos
npx prisma db push              # Bypass de migrations (no queda historial)
npx prisma migrate dev          # En prod, usar solo "deploy"

# Nunca modificar archivos dentro de prisma/migrations/
# Son parte del historial inmutable del schema
```

### SIEMPRE HACER

- Migrations en rama `feature/` → merge a `develop` → staging → `main`
- Nombre descriptivo de la migration (`add_`, `alter_`, `create_`, `drop_`)
- Para `DROP COLUMN`: crear una migration de "soft deprecation" primero (NULL + ignored), luego drop en siguiente release
- Para tablas > 100K rows: usar `CONCURRENTLY` para indexes, `ADD COLUMN ... DEFAULT NULL` (evitar lock)

## Migrations Especiales

### Agregar Index sin Lock (tablas grandes)

```sql
-- NO hacer (bloquea tabla):
CREATE INDEX idx_orders_plate ON work_orders(plate);

-- SÍ hacer (no bloquea, más lento):
CREATE INDEX CONCURRENTLY idx_orders_plate ON work_orders(plate);
```

En Prisma, esto requiere SQL raw en la migration (Prisma no genera `CONCURRENTLY`):

```prisma
// schema.prisma
@@index([plate])  // Prisma genera el index

// Pero en la migration SQL, editar manualmente para agregar CONCURRENTLY
```

### Columna NOT NULL en Tabla con Datos

```sql
-- Paso 1: Agregar nullable con default
ALTER TABLE work_orders ADD COLUMN priority VARCHAR(10) DEFAULT 'NORMAL';

-- Paso 2: Rellenar datos existentes
UPDATE work_orders SET priority = 'NORMAL' WHERE priority IS NULL;

-- Paso 3: Hacer NOT NULL (en migration separada)
ALTER TABLE work_orders ALTER COLUMN priority SET NOT NULL;
```

### Migration de Audit Log (no modificar las RULES)

Si se necesita agregar columnas al audit_log:

```sql
-- Solo ADD COLUMN permitido
ALTER TABLE audit_log ADD COLUMN session_id UUID;

-- NUNCA:
-- DROP RULE audit_log_no_delete;   ← PROHIBIDO
-- DROP RULE audit_log_no_update;   ← PROHIBIDO
```

## Seed Data

```bash
# Poblar datos iniciales (dev/staging):
npx prisma db seed

# El seed crea:
# - edgar@arellan.pe (OWNER)
# - juan@arellan.pe (OWNER)
# - ana@arellan.pe (ADMIN)
# - hija@arellan.pe (FINANCE)
# - mecanico1@arellan.pe (MECHANIC)
# Ver database/seed-data.md para detalle completo
```

## Historial de Migrations Críticas

| Migration | Descripción | Impacto |
|-----------|-------------|---------|
| `20240101_initial_schema` | Schema base completo | Creación de todas las tablas |
| `20240115_immutable_audit_log` | RULE NO DELETE/UPDATE en audit_log | **Seguridad crítica** |
| `20240201_expense_approval_levels` | Columna `required_approval_level` | Flujo de autorización |
| `20240301_vehicle_geofencing` | Tablas tracking GPS | Sistema antijoyride |
| `20240401_inventory_movements` | Historial de movimientos + avg cost | Control de inventario |
