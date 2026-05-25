# Git Workflow y Gobernanza de Branches

## Topología de Branches

Cada uno de los 13 repositorios mantiene la misma estructura de branches:

```
main          ← Producción. Solo recibe merges de release/* o hotfix/*
  │
develop       ← Integración. Recibe merges de feature/*
  │
  ├── feature/[dominio]/[nombre]   ← Trabajo nuevo
  ├── hotfix/[incidente-id]        ← Parches urgentes de producción
  └── release/[versión]            ← Preparación de release
```

### Descripción de Cada Branch

| Branch | Descripción | Quién puede mergear a él |
|--------|-------------|--------------------------|
| `main` | Estado de producción. Código en el taller físico | Solo via release/* o hotfix/* con review |
| `develop` | Trunk de integración continua | Feature branches con PR aprobado |
| `feature/*` | Feature en desarrollo | El developer (se borra al mergear) |
| `hotfix/*` | Parche urgente de producción | Solo para P0/P1 confirmados |
| `release/*` | Staging pre-producción | Solo desde develop con checklist completo |

## Convenciones de Nombres de Branches

```
feature/finance/dynamic-qr-webhook
feature/iot/zkteco-log-parser
feature/auth/mfa-totp-implementation
feature/orders/offline-sync-queue
hotfix/P0-cashbox-webhook-timeout
release/1.2.0
```

## Ciclo de Vida de un Feature

```
1. Crear branch desde develop
   git checkout develop && git pull
   git checkout -b feature/finance/dynamic-qr-webhook

2. Desarrollar con commits semánticos
   git commit -m "feat(finance): add dynamic QR generation"
   git commit -m "test(finance): add QR idempotency tests"
   git commit -m "fix(finance): handle culqi webhook timeout"

3. Abrir PR hacia develop
   - Título descriptivo: "[Finance] Dynamic QR webhook flow"
   - Descripción: qué cambia, por qué, cómo testar
   - Link a issue/task si existe

4. Code review (1 aprobación obligatoria)
   - Reviewer verifica: cobertura 80%+, sin secretos, guards en nuevos endpoints
   - APPROVED → merge con --no-ff para preservar bubble histórico

5. Auto-deploy a staging via GitHub Actions

6. Release a producción (manual, con aprobación)
```

## Convenciones de Commits (Conventional Commits)

```
<tipo>(<dominio>): <descripción corta en español>

feat(finance):       Nueva funcionalidad de negocio
fix(auth):           Corrección de bug
security(auth):      Cambio de seguridad / criptografía
test(orders):        Agregar o modificar tests
chore(infra):        DevOps, config, dependencias
docs(governance):    Documentación
refactor(inventory): Refactoring sin cambio de comportamiento
```

**Ejemplos:**
```
feat(finance): agregar QR dinámico con expiración de 7 minutos
fix(inventory): corregir race condition en decremento de stock
security(auth): rotar claves RS256 para JWT
test(finance): cubrir flujo completo de autorización de gastos
chore(deps): actualizar NestJS a 10.4.1
```

## Reglas de Protección de Branches (Branch Protection)

Configurado en GitHub para `main` y `develop` en todos los repos:

```
main:
  ✅ Require pull request reviews (1 aprovación mínima)
  ✅ Dismiss stale reviews when new commits are pushed
  ✅ Require status checks (CI must pass)
  ✅ Require branches to be up to date before merging
  ❌ Allow force pushes → NUNCA
  ❌ Allow deletions → NUNCA

develop:
  ✅ Require pull request reviews (1 aprobación)
  ✅ Require status checks (lint + test)
  ❌ Allow force pushes → NUNCA
```

## Hotfix: Proceso de Emergencia

Solo para incidentes P0/P1 confirmados en producción:

```
1. Branch desde main (no desde develop)
   git checkout main && git pull
   git checkout -b hotfix/P0-cashbox-webhook-timeout

2. Corregir el bug (scope mínimo)

3. PR con labels: [HOTFIX] [P0]
   → Code review urgente (puede ser verbal + asíncrono si es P0)
   → CI debe pasar

4. Merge a main → deploy inmediato

5. Merge también a develop (back-merge)
   → OBLIGATORIO dentro de 12 horas
   → Para que develop no quede detrás de main
```

## Freelancers y Acceso Externo

Los freelancers externos que trabajen puntualmente:
- **Nunca** tienen acceso a `main` ni `develop` directamente
- Trabajan en branches `feature/[dominio]/ext-[nombre]-[tarea]`
- PR revisado y aprobado por miembro del equipo interno antes del merge
- Acceso revocado inmediatamente al terminar el contrato
- Se rotan secretos relevantes al terminar cualquier colaboración externa
