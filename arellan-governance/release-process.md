# Proceso de Release a Producción

Protocolo para llevar código de `develop` a `main` (producción en el taller físico). Se ejecuta máximo una vez por semana, en horario de baja actividad (domingos por la noche o lunes muy temprano antes del turno).

## Flujo Normal de Release

```
develop (integración continua)
    │
    │ 1. Code freeze: no mergear features nuevos
    ▼
release/[versión]    ← Branch de staging
    │
    │ 2. Deploy automático a Railway staging
    │ 3. QA manual (Ana + owners verifican en staging)
    │ 4. Si todo OK: abrir PR release/* → main
    ▼
main (producción)
    │
    │ 5. Deploy automático a producción
    │ 6. Smoke tests post-deploy
    └── Tag: v1.2.0
```

## Paso a Paso

### 1. Preparación (Día antes del release)

```bash
# Crear branch de release desde develop
git checkout develop && git pull
git checkout -b release/1.2.0

# Actualizar versión en package.json
npm version minor  # o patch si es bug fix

# Generar changelog de los commits desde el último release
git log v1.1.0..HEAD --oneline --no-merges
```

### 2. QA en Staging (Mismo día)

Deploy automático a staging al pushear el branch `release/*`.

El equipo técnico + Ana verifican manualmente:
- [ ] Login con MFA funciona
- [ ] Crear una OT de prueba: ingreso → proceso → listo → pago
- [ ] Apertura y cierre de caja sin diferencias
- [ ] Aprobar un gasto desde la app móvil
- [ ] Notificaciones push llegan correctamente
- [ ] Modo offline de la tablet funciona

### 3. Aprobación del Release

```
Checklist de aprobación antes del merge a main:
- [ ] Todos los checks de CI pasan (lint + test + build)
- [ ] QA manual en staging aprobado
- [ ] No hay diferencias de caja en staging
- [ ] Backup de la base de datos de producción tomado
- [ ] Owners notificados del horario de deploy
```

### 4. Deploy a Producción

```bash
# PR: release/1.2.0 → main (1 aprobación requerida)
# Railway hace deploy automático al mergear a main

# Verificar deploy exitoso
railway logs --tail 100

# Smoke tests inmediatos (< 5 minutos):
curl https://api.arellan.pe/health  # → { "status": "ok" }
```

### 5. Post-Deploy

```bash
# Tag en Git
git tag -a v1.2.0 -m "Release 1.2.0: descripción breve"
git push origin v1.2.0

# Back-merge a develop (importante: main siempre debe estar en develop)
git checkout develop && git merge main --no-ff

# Notificar al equipo
# WhatsApp a Edgar y Juan: "Sistema actualizado v1.2.0. Todo OK."
```

## Hotfix: Emergencias P0/P1

Solo para incidentes que afecten la operación del taller activamente:

```bash
# Branching desde main (no desde develop)
git checkout main && git pull
git checkout -b hotfix/P0-cashbox-webhook-timeout

# Hacer el fix (scope mínimo — solo lo necesario)
# Abrir PR hacia main con label [HOTFIX]

# Review urgente puede ser asíncrono si es P0
# Criterio de aprobación: fix es correcto y no introduce nuevos bugs

# Post-hotfix: back-merge a develop OBLIGATORIO (< 12h)
git checkout develop && git merge main --no-ff
```

## Rollback

Si el deploy de producción falla:

```bash
# Railway rollback al deploy anterior (< 2 minutos)
railway rollback

# O manualmente:
git checkout main && git reset --hard v1.1.0
git push --force-with-lease origin main

# IMPORTANTE: solo hacer force push si el rollback de Railway falló
# Notificar a owners del rollback
```

## Calendario de Releases

| Tipo | Frecuencia | Horario |
|------|-----------|---------|
| Feature release | Máximo 1 vez/semana | Domingo 9 PM o Lunes 5 AM |
| Bug fix release | Según necesidad | Mismos horarios |
| Hotfix P0/P1 | Inmediato | Cualquier hora |
| Security patch | Inmediato | Cualquier hora |

**Ventana prohibida:** Lunes a Sábado 7 AM - 6 PM (horario activo del taller). Ningún deploy en producción en ese horario a menos que sea hotfix P0.
