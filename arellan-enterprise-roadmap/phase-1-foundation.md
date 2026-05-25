# Fase 1 — MVP Foundation (Meses 1-3)

## Objetivo

Detener las pérdidas financieras activas mediante la digitalización de los flujos de cobro y autorización de gastos. El sistema debe estar en producción en el taller físico de Surquillo con los mecánicos y Ana trabajando en él diariamente.

## Criterio de Éxito

- **Primario:** Cero pagos fuera del sistema durante 30 días consecutivos
- **Secundario:** Diferencia de caja mensual < S/.100
- **Técnico:** 80% de cobertura de tests, zero downtime en horario laboral

## Semanas 1-2: Preparación de Infraestructura

- [ ] Cuenta Railway creada + proyecto desplegado
- [ ] Supabase proyecto creado (auth + storage)
- [ ] Vercel proyectos para frontend-web y mechanic-ui
- [ ] Cloudflare DNS configurado (`*.arellan.pe`)
- [ ] GitHub org `arellan-tech` configurada, 13 repos creados
- [ ] Branch protection en `main` y `develop` en todos los repos
- [ ] Secret scanning habilitado en todos los repos
- [ ] Design system `@arellan/ui` publicado en GitHub Packages

## Semanas 3-5: Backend Core

### `arellan-auth-service`
- [ ] Supabase Auth integrado
- [ ] MFA TOTP obligatorio para OWNER/ADMIN/FINANCE
- [ ] JWT RS256 con Passport
- [ ] Roles: OWNER, ADMIN, FINANCE, MECHANIC, TRAINEE
- [ ] Guards de RBAC en todos los endpoints
- [ ] Audit log de logins/logouts

### `arellan-backend-api` — Módulos Base
- [ ] Módulo Auth (guards, decorators)
- [ ] Módulo Accounts (CRUD usuarios, gestión de roles)
- [ ] Módulo Orders (ciclo completo de OT)
- [ ] Módulo Finance (caja, transacciones, gastos)
- [ ] Módulo Inventory (stock básico, movimientos)
- [ ] Módulo Audit (log inmutable)

### Base de Datos
- [ ] Prisma schema completo + migraciones
- [ ] Reglas PostgreSQL de inmutabilidad para `audit_logs`
- [ ] Índices en campos de búsqueda frecuente
- [ ] Seed data: roles, usuario OWNER inicial

## Semanas 6-8: Frontends

### `arellan-frontend-web` (Panel Admin)
- [ ] Login con MFA
- [ ] Dashboard de OTs del día
- [ ] Apertura/cierre de caja
- [ ] Módulo de gastos con flujo de aprobación
- [ ] Gestión de inventario básica
- [ ] Módulo de empleados

### `arellan-mechanic-ui` (Tablet del Taller)
- [ ] Login con PIN (sesión 12h)
- [ ] Ver lista de OTs asignadas
- [ ] Registrar ingreso de vehículo (4 fotos + km)
- [ ] Cambiar estado de OT
- [ ] Solicitar repuestos
- [ ] Modo offline con IndexedDB queue
- [ ] UI touch-friendly (botones 56px mínimo)

### `arellan-mobile-app` (App Gerencial)
- [ ] Dashboard de métricas en tiempo real
- [ ] Notificaciones push de alertas
- [ ] Aprobar/rechazar gastos
- [ ] Ver ubicación GPS de vehículos del taller

## Semanas 9-10: Integraciones Críticas

### QR de Pago (Culqi/Yape Business)
- [ ] Generación de QR dinámico por OT
- [ ] Webhook de confirmación de pago
- [ ] Idempotencia en procesamiento de webhooks
- [ ] QR expira en 7 minutos
- [ ] Fallback manual documentado

### Notificaciones
- [ ] Web Push (VAPID) para admins en desktop
- [ ] Push nativo para app gerencial
- [ ] Alertas de diferencia de caja > S/.50
- [ ] Alertas de gastos pendientes de aprobación

## Semanas 11-12: Testing y Lanzamiento

- [ ] Tests unitarios: 80% cobertura en módulos finance y auth
- [ ] Tests e2e: flujo completo de OT, caja, autorización de gastos
- [ ] Test de inmutabilidad de audit_log
- [ ] Runbook de despliegue documentado
- [ ] Manual de usuario completado (training-academy)
- [ ] Capacitación a Ana (admin), hija (finance), mecánicos
- [ ] Lanzamiento en soft opening (1 semana paralelo con sistema manual)
- [ ] Go-live completo

## Módulos Excluidos del MVP (Fase 2+)

| Módulo | Razón de exclusión |
|--------|-------------------|
| Portal cliente | Requiere registro MINJUS primero |
| SUNAT facturas electrónicas | Requiere OSE contratado |
| ZKTeco biométrico | Hardware puede llegar después |
| Importaciones avanzadas | Sistema base debe estabilizarse primero |
| IA/ML | Necesita datos históricos primero |

## Riesgos del MVP

| Riesgo | Mitigación |
|--------|-----------|
| Resistencia al cambio del personal | Capacitación + soporte intensivo primera semana |
| Falla del internet en el taller | Modo offline en mechanic-ui (8h de capacidad) |
| Webhook de Culqi sin respuesta | Fallback manual documentado en training-academy |
| Tablet rota o perdida | Sesión revocable remotamente |
