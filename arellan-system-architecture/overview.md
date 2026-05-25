# Arquitectura del Sistema — Visión General

## Diagrama de Alto Nivel

```
                    ┌────────────────────────────────────────┐
                    │          Clientes del Sistema           │
                    └──────┬────────┬────────┬───────────────┘
                           │        │        │
              ┌────────────▼──┐  ┌──▼─────┐  ┌▼──────────────────┐
              │arellan-frontend│  │mechanic│  │arellan-mobile-app │
              │   -web         │  │  -ui   │  │(PWA/gerencial)    │
              └────────────┬──┘  └──┬─────┘  └┬──────────────────┘
                           │        │          │
                           └────────▼──────────┘
                                    │
                    ┌───────────────▼───────────────┐
                    │     arellan-api-gateway        │
                    │  Rate limiting · CORS · Auth  │
                    │  Data masking · Helmet        │
                    └───────────────┬───────────────┘
                                    │
                    ┌───────────────▼───────────────┐
                    │    arellan-backend-api          │
                    │  (NestJS Monolithic Modular)   │
                    │                               │
                    │  auth · orders · finance      │
                    │  inventory · clients          │
                    │  vehicles · personnel · audit │
                    └──────┬──────┬────────┬────────┘
                           │      │        │
                    ┌──────▼──┐ ┌─▼──┐ ┌──▼──────────┐
                    │Postgres │ │Redis│ │ BullMQ Jobs │
                    │   15    │ │Cache│ │  (Workers)  │
                    └─────────┘ └────┘ └─────────────┘
                                              │
                        ┌─────────────────────┤
                        │                     │
                 ┌──────▼──────┐      ┌───────▼───────┐
                 │Notifications│      │ Reports/ETL   │
                 │Push/Email/WA│      │ Engine        │
                 └─────────────┘      └───────────────┘
```

## Decisión de Arquitectura: Monolito Modular vs Microservicios

El backend es un **monolito modular** (no microservicios). Ver ADR `decisions/001-monorepo-vs-microservices.md` para el razonamiento completo.

**Resumen:** Los dominios de negocio (OTs, inventario, finanzas, personal) están profundamente acoplados en el contexto de un taller mecánico. Separarlos en microservicios añadiría complejidad operativa sin beneficio real para un equipo de 2 personas.

## Los 13 Repositorios

```
PRODUCTOS (7 repos — código deployable):
  arellan-platform/          ← Backend: API, Auth, Notifications, Workers, etc.
  arellan-frontend-web/      ← Panel admin web (Next.js)
  arellan-mobile-app/        ← App gerencial owners (Next.js PWA)
  arellan-mechanic-ui/       ← Tablet del taller (React Vite, offline-first)
  arellan-client-portal/     ← Portal público de clientes
  arellan-status-dashboard/  ← Status page (Upptime)
  arellan-design-system/     ← Componentes UI compartidos (@arellan/ui)

MACRO-REPOSITORIOS (6 repos — documentación y configuración):
  arellan-hardware-iot/      ← ZKTeco, GPS, sensores IoT
  arellan-business-ops/      ← Lógica de negocio, tests, marketing
  arellan-data-intelligence/ ← DB design, pipelines, BI, IA
  arellan-infrastructure/    ← Terraform, DevSecOps, observabilidad, DR
  arellan-security-compliance/ ← Gobernanza, riesgos, incidents, compliance
  arellan-docs-governance/   ← Este repositorio: toda la documentación
```

## Stack Tecnológico

| Capa | Tecnología | Razón |
|------|-----------|-------|
| Backend | NestJS 10 + TypeScript 5 | Decorators, DI, módulos — escala sin microservices |
| ORM | Prisma 5 | Type-safe, migraciones, Postgres-native |
| Base de datos | PostgreSQL 15 | ACID, RLS, triggers para audit log |
| Cache / Queue | Redis 7 + BullMQ | Jobs resilientes, rate limit, sesiones |
| Auth (MVP) | Supabase Auth | Zero config, MFA + Storage incluidos |
| Auth (prod) | JWT RS256 + Passport | Control total, rotación de claves |
| Frontend admin | Next.js 14 (App Router) | SSR, RSC, excelente DX |
| Mechanic UI | React 18 + Vite + PWA | Bundle pequeño, offline-first con Service Workers |
| Estado servidor | TanStack Query | Cache inteligente, optimistic updates |
| Estado UI | Zustand | Minimal, no boilerplate |
| Estilos | Tailwind CSS | Utility-first, consistencia con design tokens |
| Formularios | React Hook Form + Zod | Type-safe, validación en cliente y servidor |

## Flujo de Autenticación y Autorización

```
1. Usuario hace POST /api/v1/auth/login con email + password
2. Si role requiere MFA → response con mfaPending: true
3. Usuario envía código TOTP → POST /api/v1/auth/mfa/verify
4. Backend devuelve: { accessToken, refreshToken }
5. Access token se almacena en memoria del cliente (no localStorage)
6. Cada request incluye: Authorization: Bearer <accessToken>
7. API Gateway valida JWT, extrae { userId, role, mfaVerified }
8. Guards de RBAC verifican que el rol tiene acceso al endpoint
9. Data masking interceptor filtra datos sensibles según el rol
```

## Flujo de Pago (QR Dinámico)

```
1. Admin hace POST /api/v1/finance/payment/qr con workOrderId
2. Backend calcula total (mano de obra + repuestos) de la OT
3. Backend llama a API de Culqi para generar QR con monto y ID único
4. QR con TTL de 7 minutos se muestra en la tablet del taller
5. Cliente escanea con su teléfono
6. Culqi envía webhook firmado a POST /api/v1/webhooks/culqi/payment
7. Backend verifica firma HMAC del webhook
8. Backend verifica idempotencia (Redis: culqi:event:{id})
9. Backend actualiza OT a PAGADO + crea FinancialTransaction
10. Todo queda en audit_log (inmutable)
```

## Audit Log — Principio de Inmutabilidad

```sql
-- PostgreSQL enforces immutability at DB level
-- La tabla audit_logs no tiene endpoints de UPDATE ni DELETE
-- Las reglas de PostgreSQL previenen cualquier modificación

CREATE RULE no_update_audit_logs AS ON UPDATE TO audit_logs DO INSTEAD NOTHING;
CREATE RULE no_delete_audit_logs AS ON DELETE TO audit_logs DO INSTEAD NOTHING;

-- Incluso con acceso directo a la DB, estas reglas aplican
```
