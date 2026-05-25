# ADR 002: PostgreSQL 15 sobre MongoDB

## Estado

`Aprobado`

## Contexto

El sistema Arellan maneja datos con características críticas:

- **Audit log inmutable:** Cada acción financiera debe registrarse y ser imposible de borrar o modificar
- **Transacciones multi-tabla:** Cerrar una OT implica actualizar estado, registrar pago, mover inventario, y crear entrada en audit_log — todo atómicamente
- **Relaciones complejas:** OT → líneas de servicio → repuestos → movimientos de inventario → proveedor
- **Integridad referencial:** Un mecánico no puede ser eliminado si tiene OTs activas
- **Datos financieros:** Caja diaria, gastos, pagos — requieren exactitud absoluta (NUMERIC, no Float)
- **Compliance SUNAT:** Facturas electrónicas con estructura fija y validable
- **Compliance Ley 728:** Registros laborales como evidencia legal (asistencia, incidencias)

**Restricción:** El equipo tiene experiencia sólida en SQL y Prisma ORM. Experiencia mínima con MongoDB y esquemas flexibles.

## Alternativas Consideradas

### Opción 1: MongoDB

Base de datos documental con esquemas flexibles.

**Pros:**
- Esquemas flexibles para datos no estructurados
- JSON nativo (sin mapeo objeto-documento)
- Horizontal scaling nativo

**Contras:**
- Sin ACID multi-documento real en versiones anteriores a 4.x (y aun limitado)
- Sin foreign keys → integridad referencial por aplicación (frágil)
- Sin `NUMERIC` exacto → problemas con decimales financieros (0.1 + 0.2 ≠ 0.3)
- Imposible implementar audit log inmutable a nivel de DB (sin DDL constraints equivalentes a `RULE`)
- Prisma ORM tiene soporte MongoDB experimental, no production-grade
- Para datos tan relacionales como OTs + inventario + pagos, el modelo documental genera queries complejas y costosas

### Opción 2: MySQL / MariaDB

**Pros:**
- Ampliamente conocido
- Soporte Prisma production-grade

**Contras:**
- Sin `RULE` nativa para audit log inmutable (solo triggers, que pueden desactivarse)
- Tipos ENUM menos flexibles
- Sin soporte nativo para JSON avanzado (JSONB en PostgreSQL es superior)
- Row-level security (RLS) menos robusto que PostgreSQL

### Opción 3: PostgreSQL 15 ← Seleccionada

Base de datos relacional con ACID completo, extensible y con características avanzadas de seguridad.

**Pros:**
- ACID completo con transacciones multi-tabla reales
- `RULE` para hacer audit_log verdaderamente inmutable a nivel de motor
- `NUMERIC(10,2)` para exactitud financiera absoluta
- Row-Level Security (RLS) para multi-tenancy futuro
- JSONB nativo para datos semiestructurados (cuando se necesite)
- Supabase usa PostgreSQL → mismo motor en MVP y producción
- Prisma soporte production-grade maduro
- `pg_cron` para jobs periódicos en DB (backup, limpieza)
- Extensions: `uuid-ossp`, `pgcrypto`, `pg_stat_statements`

**Contras:**
- Horizontal write scaling más complejo (mitigado: carga de escritura del taller es baja, <100 req/min)
- Schema migrations requieren gestión cuidadosa → resuelto con Prisma Migrate

## Decisión

Seleccionamos **PostgreSQL 15** gestionado inicialmente en Supabase (MVP) y migrable a AWS RDS en producción.

### Patrón de Audit Log Inmutable

La decisión crítica es usar `RULE` de PostgreSQL para hacer el audit_log imposible de modificar, incluso para el usuario de la aplicación:

```sql
-- Inmutabilidad total: ni DELETE ni UPDATE permitidos
CREATE RULE audit_log_no_delete AS
  ON DELETE TO audit_log DO INSTEAD NOTHING;

CREATE RULE audit_log_no_update AS
  ON UPDATE TO audit_log DO INSTEAD NOTHING;

-- Solo INSERT es permitido
-- Aplicado en schema: 20240115000000_immutable_audit_log.sql
```

Esto es imposible de replicar con la misma seguridad en MongoDB.

### Tipos de Datos Críticos

```sql
-- Financiero: exactitud absoluta
amount       NUMERIC(10,2)   -- NO FLOAT ni DOUBLE
discrepancy  NUMERIC(10,2)

-- Identidad
id           UUID DEFAULT gen_random_uuid()

-- Timestamps con timezone (Lima = UTC-5)
created_at   TIMESTAMPTZ DEFAULT NOW()

-- Enums en PostgreSQL (validación a nivel de motor)
CREATE TYPE order_status AS ENUM ('RECIBIDO', 'EN_PROCESO', 'LISTO', 'ENTREGADO', 'CANCELADO');
```

## Consecuencias

### Positivas

- Audit log inmutable garantizado a nivel de motor DB, no solo por código
- Transacciones ACID para operaciones financieras críticas (caja, pagos, inventario)
- Exactitud financiera con `NUMERIC(10,2)` — sin errores de punto flotante
- Integridad referencial automática con foreign keys
- RLS ready para migración SaaS multi-tenant

### Negativas / Trade-offs

- Schema migrations requieren coordinación → proceso en `database/migrations-guide.md`
- No horizontal write scaling out-of-the-box → aceptable para el volumen del taller
- Para datos de tracking GPS (alto volumen, append-only) se considera TimescaleDB en Fase 5

### Neutras

- Supabase y Railway PostgreSQL usan el mismo motor → zero-friction migration

## Notas de Implementación

- Migrations: `prisma/migrations/` con `prisma migrate deploy` en CI/CD
- Schema: `arellan-technical-docs/database/schema-overview.md`
- Indexes: `arellan-technical-docs/database/indexes-and-performance.md`
- Seed: `prisma/seed.ts` crea usuarios Edgar, Ana, mecánico demo
- Audit log SQL rule: `prisma/migrations/20240115000000_immutable_audit_log/migration.sql`

---

*Fecha de decisión: 2025-Q1*
*Autor: Diego Soto (Tech Lead)*
*Revisado por: Edgar Arellan (Owner)*
