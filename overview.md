# arellan-docs-governance — Documentación y Gobernanza

Repositorio central de toda la documentación estratégica, técnica y operativa del ecosistema digital de la **Clínica Automotriz Arellan Hnos** (Surquillo, Lima, Perú). Cubre la arquitectura del sistema, estándares de código, roadmap, manuales de usuario y base de conocimiento operativo.

## Estructura del Repositorio

```
arellan-docs-governance/
├── arellan-enterprise-roadmap/   Roadmap 2026-2028, deuda técnica, visión
├── arellan-governance/           Git workflow, estándares, convenciones, onboarding
├── arellan-knowledge-base/       Procesos de negocio, proveedores, troubleshooting
├── arellan-platform-governance/  Templates, procesos, estándares de desarrollo
├── arellan-system-architecture/  Arquitectura, ADRs, documentación de API
├── arellan-technical-docs/       DB schema, infraestructura, integraciones
└── arellan-training-academy/     Manuales de usuario por rol en español
```

## Cómo Navegar esta Documentación

| Si buscas... | Ve a... |
|--------------|---------|
| Entender el proyecto desde cero | `arellan-system-architecture/overview.md` |
| Configurar el entorno local | `arellan-technical-docs/infrastructure/docker-setup.md` |
| Aprender el flujo de Git | `arellan-governance/git-workflow.md` |
| Entender el schema de la base de datos | `arellan-technical-docs/database/schema-overview.md` |
| Manual para mecánicos (tablet) | `arellan-training-academy/mechanics/tablet-quick-guide.md` |
| Manual para administración | `arellan-training-academy/admins/quick-start-guide.md` |
| Manual para finanzas | `arellan-training-academy/finance/cashbox-daily.md` |
| Documentación de endpoints API | `arellan-system-architecture/api-docs/overview.md` |
| Decisiones de arquitectura | `arellan-system-architecture/decisions/` |
| Procesos de negocio | `arellan-knowledge-base/business-processes/` |
| Plan de despliegue | `arellan-technical-docs/infrastructure/deployment-guide.md` |
| Roadmap y fases | `arellan-enterprise-roadmap/overview.md` |

## Contexto del Proyecto

El sistema resuelve tres problemas críticos de fraude interno:
1. **Cobros fuera del sistema** — empleados usaban Yape personal para desviar pagos
2. **Comisiones no declaradas en importaciones** — 20-30% de sobrecosto oculto
3. **Uso no autorizado de vehículos** — vehículos del taller usados sin permiso

La solución es un ecosistema digital de 13 repositorios que hace todas las transacciones trazables y requiere aprobación digital para cualquier movimiento de dinero o vehículo.

## Audiencia

- **Equipo técnico:** Arquitectura, ADRs, estándares de desarrollo
- **Edgar y Juan (Owners):** Roadmap, procesos de negocio, manuales de owner
- **Ana (Apoyo):** Manuales de administración
- **Hija (Finance):** Manuales de caja y finanzas
- **Mecánicos:** Guías rápidas de tablet

## Stack Tecnológico Resumido

| Capa | Tecnología |
|------|-----------|
| Backend | NestJS 10 + TypeScript 5 + Prisma ORM + PostgreSQL 15 |
| Frontend Web | Next.js 14 + TypeScript + Tailwind + Zustand |
| Mechanic UI | React 18 + Vite (offline-first, PWA) |
| Mobile App | Next.js 14 PWA → React Native Expo (Fase 3) |
| Auth | Supabase Auth (MVP) → JWT RS256 custom (producción) |
| Cloud MVP | Railway + Supabase + Vercel + Cloudflare (~$65-120 USD/mes) |
| Cloud Prod | AWS (EC2/RDS/ElastiCache/S3/CloudFront, ~$150-300 USD/mes) |
