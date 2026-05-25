# Registro de Deuda Técnica

Atajos y compromisos técnicos tomados en el MVP para acelerar el lanzamiento. Cada ítem incluye el impacto, cuándo debe resolverse y el esfuerzo estimado.

## Deuda Activa

### TD-001 — Supabase Auth vs JWT RS256 Custom

| Campo | Detalle |
|-------|---------|
| **Decisión original** | Usar Supabase Auth para el MVP (zero config, MFA incluido) |
| **Deuda** | Supabase gestiona los tokens, no tenemos control total sobre expiración ni revocación |
| **Impacto** | Bajo en MVP. Alto si migramos a AWS (Supabase no corre on-premise) |
| **Cuando resolver** | Fase 2 o al migrar a AWS |
| **Esfuerzo** | 2-3 semanas (implementar JWT RS256 + Passport + refresh token store en Redis) |

### TD-002 — Sin Read Replica en PostgreSQL

| Campo | Detalle |
|-------|---------|
| **Decisión original** | Single-node PostgreSQL en Railway para el MVP |
| **Deuda** | Las queries analíticas de reportes compiten con las transacciones operativas |
| **Impacto** | Bajo con < 100 OTs/mes. Crece con el volumen |
| **Cuando resolver** | Fase 3 o cuando tiempo de respuesta de dashboard > 3 segundos |
| **Esfuerzo** | 1 semana (provisionar read replica + configurar Prisma directUrl) |

### TD-003 — Logs en Contenedor vs Sistema Centralizado

| Campo | Detalle |
|-------|---------|
| **Decisión original** | Logs de aplicación en stdout del contenedor Railway |
| **Deuda** | Si el contenedor reinicia, se pierden los logs de aplicación (no los audit_logs — esos están en DB) |
| **Impacto** | Bajo para audit trail (audit_logs en DB son permanentes). Medio para debugging |
| **Cuando resolver** | Fase 2 |
| **Esfuerzo** | 1 semana (configurar Axiom o Grafana Loki, integrar Winston transport) |

### TD-004 — Sin Versionado de API

| Campo | Detalle |
|-------|---------|
| **Decisión original** | API en `/api/v1/` pero sin versioning real implementado |
| **Deuda** | Si el mobile-app en producción tiene `v1` hardcodeado y cambiamos el contrato, hay breaking changes |
| **Impacto** | Bajo en MVP (solo 1 cliente: el taller). Alto cuando haya múltiples clientes |
| **Cuando resolver** | Antes de lanzar portal cliente o SaaS |
| **Esfuerzo** | 1 semana |

### TD-005 — Tests de Integración sin Testcontainers

| Campo | Detalle |
|-------|---------|
| **Decisión original** | Tests de integración usan base de datos en memoria (better-sqlite3) |
| **Deuda** | SQLite no tiene las mismas características que PostgreSQL 15 (RLS, JSONB, full-text search) |
| **Impacto** | Falsos positivos en tests. Bugs que solo aparecen en producción con PostgreSQL |
| **Cuando resolver** | Fase 1 sprint 2 |
| **Esfuerzo** | 3 días (migrar a Testcontainers PostgreSQL) |

### TD-006 — IndexedDB Sin Cifrado en Tablet

| Campo | Detalle |
|-------|---------|
| **Decisión original** | La queue offline de mechanic-ui almacena en IndexedDB sin cifrado |
| **Deuda** | Si alguien roba la tablet y la analiza, puede ver las acciones encoladas |
| **Impacto** | Bajo (datos operativos de OTs, no financieros) |
| **Cuando resolver** | Fase 2 |
| **Esfuerzo** | 2 días (integrar dexie-encrypted para IndexedDB) |

## Deuda Resuelta

| ID | Descripción | Resuelta en |
|----|-------------|------------|
| TD-007 | Webhook Culqi sin idempotencia → implementado con Redis | Fase 1 sprint 3 |
| TD-008 | audit_logs sin reglas RLS → implementado con PostgreSQL rules | Fase 1 sprint 2 |

## Principio de Gestión de Deuda

- **Deuda de seguridad:** Resolver inmediatamente (no se acumula)
- **Deuda de performance:** Resolver cuando el impacto sea medible (> 3s respuesta)
- **Deuda de arquitectura:** Resolver en la siguiente fase antes de agregar nuevas features
- **Deuda de DX:** Resolver en tiempo de capacidad disponible

La deuda técnica se registra aquí con issue en GitHub correspondiente para que no se pierda en el backlog.
