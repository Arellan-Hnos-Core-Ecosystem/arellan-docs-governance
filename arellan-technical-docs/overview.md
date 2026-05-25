# Technical Docs — Arellan Backend

Documentación técnica de implementación del sistema. Referencia para desarrolladores que trabajan en `arellan-backend-core` y servicios de hardware.

## Estructura

```
arellan-technical-docs/
├── database/
│   ├── schema-overview.md        ← Tablas, tipos, relaciones
│   ├── tables-reference.md       ← Referencia completa de campos
│   ├── migrations-guide.md       ← Cómo crear y ejecutar migrations
│   ├── indexes-and-performance.md ← Índices críticos y explain plans
│   └── seed-data.md              ← Datos iniciales de desarrollo
├── infrastructure/
│   ├── environments.md           ← dev / staging / production
│   ├── enviroment-variables.md   ← Variables requeridas por servicio
│   ├── docker-setup.md           ← Entorno local con Docker Compose
│   ├── deployment-guide.md       ← Deploy en Railway y AWS
│   └── ssl-and-domains.md        ← Dominios y certificados SSL
└── integrations/
    ├── culqi-integration.md      ← Pagos QR (webhooks, idempotencia)
    ├── sunat-integration.md      ← Facturación electrónica (Nubefact)
    ├── whatsapp-business.md      ← Notificaciones Meta Business API
    └── zkteco-bioemtric.md       ← ADMS bridge biométrico
```

## Stack Técnico

| Capa | Tecnología | Versión |
|------|-----------|---------|
| Runtime | Node.js | 20 LTS |
| Framework | NestJS | 10 |
| Lenguaje | TypeScript | 5 strict |
| ORM | Prisma | 5 |
| Base de datos | PostgreSQL | 15 |
| Cache / Queue | Redis + BullMQ | 7 / 4 |
| Auth (MVP) | Supabase Auth | — |
| Auth (Prod) | JWT RS256 + Passport | — |
| Deploy (MVP) | Railway | — |
| Deploy (Prod) | AWS ECS + RDS | — |
| Fronted admin | Next.js | 14 |
| Frontend mecánico | React 18 + Vite PWA | — |
| Mobile dueños | Expo React Native | — |

## Principios de Arquitectura

1. **Monolito modular** — Un proceso NestJS con módulos con fronteras claras. Sin microservicios en MVP.
2. **Audit log inmutable** — `audit_log` con `RULE` PostgreSQL. Ningún `DELETE` ni `UPDATE` permitido, ni siquiera por el owner del DB.
3. **Zero Trust** — Todos los endpoints autenticados (excepto `GET /vehicles/:plate/status`). MFA obligatorio para módulos financieros.
4. **Data masking** — Mecánicos nunca ven teléfono, email, DNI ni dirección de clientes. Solo placa y modelo.
5. **NUMERIC para dinero** — Nunca `FLOAT` ni `DOUBLE` en campos monetarios. `NUMERIC(10,2)` siempre.

## Decisiones Clave

Ver `arellan-system-architecture/decisions/` para los 5 ADRs:

- **ADR-001:** Multi-repo con backend monolítico vs microservicios
- **ADR-002:** PostgreSQL sobre MongoDB
- **ADR-003:** PWA sobre React Native para mecánicos
- **ADR-004:** NestJS sobre Express
- **ADR-005:** Turborepo para frontends compartidos
