# Fase 6 — Expansión y SaaS (Año 3 — 2028)

## Objetivo

Evaluar la viabilidad de escalar el sistema como producto SaaS para otros talleres mecánicos en Lima y Perú, manteniendo el negocio original de Arellan como cliente principal y caso de éxito.

## Contexto

El sistema desarrollado para Arellan Hnos es inherentemente genérico en su arquitectura. Los módulos de OTs, caja, inventario, RBAC y audit trail son aplicables a cualquier taller mecánico que tenga los mismos problemas de fraude interno y falta de trazabilidad.

**Decisión a evaluar en 2027-2028:** ¿Tiene sentido comercializar el sistema como SaaS?

## Prerequisitos para Evaluar SaaS

- Sistema Arellan estable en producción por 2+ años
- Evidencia cuantificable de reducción de pérdidas (ROI demostrado)
- Equipo técnico con capacidad de mantener múltiples tenants
- Demanda real de otros talleres (al menos 3 interesados confirmados)
- Capital para desarrollo de multi-tenancy (~$10,000-20,000 USD)

## Arquitectura Multi-Tenant (Si se decide)

### Opción 1: Schema por Tenant (Recomendada para MVP SaaS)

```sql
-- Cada taller tiene su propio schema en la misma DB
CREATE SCHEMA tenant_taller_123;
CREATE SCHEMA tenant_taller_456;

-- Tablas replicadas por schema
SET search_path TO tenant_taller_123;
SELECT * FROM work_orders;  -- Solo OTs de ese taller
```

**Pro:** Simple de implementar. **Contra:** Hasta ~100 tenants por DB.

### Opción 2: Row-Level Security (RLS)

```sql
-- Todas las tablas tienen tenant_id
ALTER TABLE work_orders ADD COLUMN tenant_id UUID NOT NULL;

-- RLS policy: cada usuario solo ve sus datos
CREATE POLICY tenant_isolation ON work_orders
  USING (tenant_id = current_setting('app.current_tenant')::UUID);
```

**Pro:** Escala a miles de tenants. **Contra:** Mayor complejidad operativa.

### Opción 3: Database por Tenant

```
Cada taller tiene su propia base de datos PostgreSQL
Railway/RDS: instancia por tenant
```

**Pro:** Aislamiento total. **Contra:** Costo elevado (~$20/mes por tenant).

## Modelo de Negocio SaaS (Propuesta)

| Plan | Precio | Usuarios | OTs/mes | Soporte |
|------|--------|---------|---------|---------|
| Básico | S/.150/mes | 5 usuarios | 100 OTs | Email |
| Estándar | S/.350/mes | 15 usuarios | 500 OTs | WhatsApp |
| Premium | S/.800/mes | Ilimitado | Ilimitado | Dedicado |

**Break-even:** ~10 talleres en plan Estándar para cubrir costos de infraestructura y mantenimiento.

## Hitos de la Fase 6

| Hito | Descripción | Estimado |
|------|-------------|---------|
| M6.1 | Validar demanda (entrevistar 10 talleres) | Q1 2028 |
| M6.2 | Prototipo multi-tenant con 1 taller piloto | Q2 2028 |
| M6.3 | Landing page SaaS + onboarding automatizado | Q3 2028 |
| M6.4 | 5 talleres pagando en producción | Q4 2028 |

## Riesgos

| Riesgo | Mitigación |
|--------|-----------|
| Mercado no dispuesto a pagar | Validar con entrevistas antes de desarrollar |
| Soporte abrumador | Documentación + video tutoriales + onboarding guiado |
| Regulación SUNAT diferente por taller | Configuración flexible de comprobantes |
| Copia del sistema por competidores | IP protegida, licencia comercial |
