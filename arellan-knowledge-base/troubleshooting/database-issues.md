# Troubleshooting: Problemas de Base de Datos

Guía para el equipo técnico ante problemas de conexión, migraciones fallidas o inconsistencias en la base de datos PostgreSQL.

## Diagnóstico Inicial

```bash
# 1. Verificar que la base de datos está activa
# En Supabase dashboard (MVP): Project → Database → Status
# En Railway (si está en Railway): Service → Metrics

# 2. Verificar conectividad desde el backend
railway logs --service backend-api | tail -50 | grep -i "database\|prisma\|connect"

# 3. Probar conexión directamente (requiere acceso de equipo técnico)
psql $DATABASE_URL -c "SELECT NOW()"
```

## Problema: Backend No Puede Conectarse a la DB

### Causa 1: Variable de entorno DATABASE_URL incorrecta

```bash
# Verificar la variable de entorno en Railway
railway variables | grep DATABASE_URL

# Comparar con la URL de Supabase (Supabase → Settings → Database → Connection String)
# Debe ser la "Transaction Pooler" URL (puerto 6543) para Railway
```

### Causa 2: Supabase en mantenimiento

Verificar `status.supabase.com` para ver si hay un incidente activo.

**Mitigación:** Activar el modo de operación manual documentado en `arellan-disaster-recovery`.

### Causa 3: Demasiadas conexiones abiertas

```sql
-- Verificar conexiones activas
SELECT count(*) FROM pg_stat_activity WHERE state = 'active';

-- Si > 80% del límite (Supabase Free: 20 conexiones):
-- El backend tiene connection pool con Prisma → verificar configuración
```

```typescript
// prisma.service.ts — configuración de pool
const prisma = new PrismaClient({
  datasources: {
    db: { url: process.env.DATABASE_URL },
  },
  log: ['error'],
})

// Para Railway/Supabase Free (20 conexiones):
// connection_limit=3&pool_timeout=10 en la DATABASE_URL
// Ejemplo: postgresql://...?connection_limit=3&pool_timeout=10
```

## Problema: Migración de Prisma Falló

### Ver el estado de las migraciones

```bash
npx prisma migrate status
```

### Migración aplicada parcialmente

Si una migración se interrumpió a la mitad:

```bash
# 1. Ver el estado exacto
npx prisma migrate status

# 2. Si dice "failed":
# NO ejecutar migrate deploy de nuevo — puede dejar inconsistencias

# 3. Resolver manualmente:
# a) Conectar a la DB y ver qué parte se aplicó
psql $DATABASE_URL

# b) Completar manualmente la parte que faltó
# c) Marcar la migración como aplicada
npx prisma migrate resolve --applied "20240115_nombre_de_la_migracion"
```

### Migración con errores de schema incompatible

```bash
# Ver el error exacto
npx prisma migrate deploy 2>&1

# Error común: columna NOT NULL en tabla con datos existentes
# Solución: agregar DEFAULT en la migración o migrar en dos pasos:
# Paso 1: agregar columna NULLABLE
# Paso 2: llenar valores
# Paso 3: migración separada: agregar NOT NULL
```

## Problema: Query Lenta Bloqueando el Sistema

```sql
-- Ver queries activas con duración
SELECT
  pid,
  now() - query_start AS duration,
  query,
  state
FROM pg_stat_activity
WHERE state != 'idle'
  AND query_start < now() - INTERVAL '30 seconds'
ORDER BY duration DESC;

-- Si hay una query bloqueada > 5 minutos:
-- Terminar el proceso (con cuidado — verificar que no es una migración legítima)
SELECT pg_terminate_backend(pid)
FROM pg_stat_activity
WHERE pid = [PID_DEL_PROCESO];
```

## Problema: Audit Log Tiene Registros Incompletos

Los audit_logs son append-only. Si faltan registros no es por borrado (eso es imposible por las reglas de PostgreSQL). Las causas posibles son:

1. **El servicio estuvo caído** durante ese período (verificar Railway logs)
2. **El código no estaba llamando** a `auditService.log()` en algún módulo nuevo
3. **Rollback de transacción** — si la transacción principal falló, el audit_log de ese intento también se revirtió

```sql
-- Verificar gaps en audit_log
SELECT
  DATE_TRUNC('hour', created_at) as hour,
  COUNT(*) as registros
FROM audit_logs
WHERE created_at > NOW() - INTERVAL '7 days'
GROUP BY hour
ORDER BY hour;

-- Si hay horas con 0 registros en horario laboral → investigar
```

## Backup y Restauración

### Tomar Backup Manual (Antes de Migraciones Riesgosas)

```bash
# Con psql
pg_dump $DATABASE_URL > backup_$(date +%Y%m%d_%H%M%S).sql

# Subir a S3 como evidencia
aws s3 cp backup_YYYYMMDD_HHMMSS.sql s3://arellan-backups/manual/
```

### Restaurar desde Backup

```bash
# PELIGRO: esto reemplaza todos los datos actuales
# Solo hacer con aprobación explícita de OWNER

psql $DATABASE_URL < backup_YYYYMMDD.sql
```

Ver `arellan-disaster-recovery` para el runbook completo de restauración.
