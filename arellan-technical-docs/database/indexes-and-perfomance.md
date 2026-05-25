# Indexes and Performance

Índices críticos del sistema y guía de performance para PostgreSQL 15.

## Índices por Tabla

### work_orders

```sql
-- Búsqueda por placa (frecuente: portal de clientes)
CREATE INDEX idx_work_orders_plate ON work_orders(plate);

-- Filtro por estado (dashboard admin)
CREATE INDEX idx_work_orders_status ON work_orders(status);

-- OTs por mecánico (vista del mecánico)
CREATE INDEX idx_work_orders_mechanic ON work_orders(assigned_mechanic_id);

-- OTs activas (excluye soft-deleted)
CREATE INDEX idx_work_orders_active ON work_orders(status)
  WHERE status NOT IN ('ENTREGADO', 'CANCELADO');

-- Portal cliente: placa + estado (query frecuente sin auth)
CREATE INDEX idx_work_orders_plate_status ON work_orders(plate, status);
```

### audit_log

```sql
-- Búsqueda por entidad (investigar acción específica)
CREATE INDEX idx_audit_entity ON audit_log(entity_type, entity_id);

-- Búsqueda por usuario (historial de acciones de un empleado)
CREATE INDEX idx_audit_account ON audit_log(account_id);

-- Filtro por tipo de acción (detectar patrones de fraude)
CREATE INDEX idx_audit_action ON audit_log(action);

-- Rango de fechas (reportes diarios/mensuales)
CREATE INDEX idx_audit_created ON audit_log(created_at DESC);
```

### cashbox_sessions

```sql
-- Una caja por día (query más frecuente del módulo finance)
CREATE UNIQUE INDEX idx_cashbox_date ON cashbox_sessions(date);

-- Caja abierta actual
CREATE INDEX idx_cashbox_open ON cashbox_sessions(status)
  WHERE status = 'OPEN';
```

### attendance_records

```sql
-- Historial por empleado + fecha
CREATE INDEX idx_attendance_account_date
  ON attendance_records(account_id, recorded_at DESC);

-- Detección de PIN en lugar de huella (alerta de fraude)
CREATE INDEX idx_attendance_verify ON attendance_records(verify_method)
  WHERE verify_method = 'PIN';
```

### vehicle_tracking_logs

```sql
-- Historial de vehículo en rango de tiempo
CREATE INDEX idx_tracking_vehicle_time
  ON vehicle_tracking_logs(vehicle_id, recorded_at DESC);

-- Alertas de joyride
CREATE INDEX idx_tracking_outside ON vehicle_tracking_logs(inside_fence)
  WHERE inside_fence = false;
```

### inventory_items

```sql
-- Stock crítico (dashboard alertas)
CREATE INDEX idx_inventory_low_stock ON inventory_items(current_stock, minimum_stock)
  WHERE current_stock <= minimum_stock AND is_active = true;

-- Búsqueda por SKU
CREATE UNIQUE INDEX idx_inventory_sku ON inventory_items(sku);
```

## Explain Plan — Queries Críticos

### Verificar índice usado

```sql
EXPLAIN ANALYZE
SELECT * FROM work_orders
WHERE plate = 'ABC-123' AND status != 'ENTREGADO';

-- Resultado esperado:
-- Index Scan using idx_work_orders_plate on work_orders
-- (cost=0.43..8.45 rows=1 width=...)
-- NOT Seq Scan (señal de alerta si aparece)
```

### Detectar Seq Scans en producción

```sql
-- Queries sin índice (requieren atención)
SELECT schemaname, tablename, seq_scan, idx_scan,
       seq_tup_read, idx_tup_fetch
FROM pg_stat_user_tables
WHERE seq_scan > 100
ORDER BY seq_scan DESC;
```

### Queries lentos (pg_stat_statements)

```sql
-- Habilitar extensión (una vez):
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;

-- Top 10 queries más lentos
SELECT query, calls, mean_exec_time, total_exec_time
FROM pg_stat_statements
ORDER BY mean_exec_time DESC
LIMIT 10;
```

## Connection Pool

### Config en Prisma (production)

```typescript
// prisma/schema.prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
  // directUrl para migrations (evita PgBouncer)
  directUrl = env("DIRECT_DATABASE_URL")
}
```

### PgBouncer (Railway / Supabase)

```env
# Transaction mode (compatible con Prisma)
DATABASE_URL="postgresql://user:pass@db.supabase.co:6543/postgres?pgbouncer=true"

# Direct connection (para migrations solamente)
DIRECT_DATABASE_URL="postgresql://user:pass@db.supabase.co:5432/postgres"
```

### Pool size recomendado

| Environment | Pool Size | Justificación |
|-------------|-----------|---------------|
| Development | 5 | Local, sin carga |
| Staging | 10 | Testing con carga simulada |
| Production MVP (Railway) | 15 | Máximo Railway tier |
| Production (AWS RDS) | 25 | db.t3.medium permite ~100 conn |

## Mantenimiento

### VACUUM y ANALYZE automáticos

PostgreSQL 15 tiene autovacuum activado por defecto. Verificar que esté funcionando:

```sql
SELECT schemaname, tablename, last_autovacuum, last_autoanalyze
FROM pg_stat_user_tables
WHERE last_autovacuum IS NULL OR last_autovacuum < NOW() - INTERVAL '7 days'
ORDER BY n_dead_tup DESC;
```

### VACUUM manual en audit_log

El audit_log crece constantemente. Nunca se borran filas (inmutable), pero el autovacuum es suficiente. No se necesita VACUUM manual periódico.

### Tabla de tamaños

```sql
SELECT
  table_name,
  pg_size_pretty(pg_total_relation_size(table_name::regclass)) AS total_size,
  pg_size_pretty(pg_relation_size(table_name::regclass)) AS table_size,
  pg_size_pretty(pg_indexes_size(table_name::regclass)) AS index_size
FROM information_schema.tables
WHERE table_schema = 'public'
ORDER BY pg_total_relation_size(table_name::regclass) DESC;
```

## Alertas de Performance

| Señal | Umbral | Acción |
|-------|--------|--------|
| Seq scan en `work_orders` > 1000/min | Prod | Revisar índice faltante |
| Query P95 > 500ms | Prod | `EXPLAIN ANALYZE`, agregar índice |
| Connections activas > 80% del pool | Prod | Escalar connection pool o instancia |
| `n_dead_tup` > 1,000,000 en una tabla | Cualquiera | `VACUUM ANALYZE nombre_tabla` |
| DB size > 5GB | Prod | Planificar partitioning de `audit_log` (Fase 5) |
