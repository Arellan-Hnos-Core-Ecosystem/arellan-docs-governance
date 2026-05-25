# Troubleshooting: Errores de Deploy

Guía para el equipo técnico ante fallos en el pipeline de despliegue en Railway (backend) y Vercel (frontends).

## Diagnóstico Rápido

```bash
# Ver el estado del último deploy
railway status
railway logs --tail 100

# Para Vercel (frontends)
vercel logs [deployment-url]
```

## Errores en Railway (Backend)

### Deploy Fallido: Build Error

```bash
# Ver logs completos del build
railway logs --deployment [deployment-id]

# Error común 1: Error de TypeScript
# TypeScript compilation failed
# Solución: correr localmente primero
npm run build
# Corregir errores de TypeScript antes del deploy
```

```bash
# Error común 2: Falta variable de entorno
# Error: process.env.JWT_SECRET is undefined
# Solución: agregar la variable en Railway
railway variables set JWT_SECRET=valor
```

### Deploy Exitoso pero el Servicio No Responde

```bash
# Verificar que el proceso está corriendo
railway logs | grep "Application is running"

# Si no aparece ese mensaje, el proceso crasheó al iniciar
# Buscar el error en los logs:
railway logs | grep -i "error\|exception\|failed"
```

### Rollback Inmediato (< 2 minutos)

Si el deploy rompió algo en producción:

```bash
# Opción 1: Rollback desde Railway CLI
railway rollback

# Opción 2: Rollback desde el dashboard de Railway
# Railway Dashboard → Service → Deployments → [deployment anterior] → Redeploy

# DESPUÉS del rollback: investigar qué causó el fallo antes del próximo deploy
```

### El Servicio Se Reinicia Constantemente (Crash Loop)

```bash
# Ver el error que causa el crash
railway logs | tail -200

# Causas comunes:
# 1. Prisma no puede conectarse a la DB al iniciar
#    → Verificar DATABASE_URL
#    → Verificar que la DB de Supabase está activa

# 2. Variable de entorno faltante
#    → El código lanza una excepción al leer process.env.VARIABLE_INEXISTENTE

# 3. Puerto incorrecto
#    → Railway expone el puerto definido en PORT env variable
#    → El backend debe escuchar en process.env.PORT
```

## Errores en Vercel (Frontends)

### Build Fallido en Vercel

```bash
# Ver los logs de build en Vercel Dashboard:
# Project → Deployments → [deployment] → Build Logs

# Error común: módulo no encontrado
# Module not found: Error: Can't resolve '@arellan/ui'
# Solución: verificar que GitHub Packages está configurado en .npmrc
# echo "@arellan:registry=https://npm.pkg.github.com" > .npmrc
# Agregar GITHUB_TOKEN a las variables de entorno de Vercel
```

### Frontend Desplegado pero Devuelve 500

```bash
# Ver los Function logs en Vercel (para las API routes de Next.js)
vercel logs [url-del-deployment]

# Causas comunes en Next.js:
# 1. Variable de entorno NEXT_PUBLIC_API_URL incorrecta
#    → Vercel Dashboard → Settings → Environment Variables
# 2. El backend al que apunta el frontend no está disponible
#    → Verificar que el backend en Railway está running
```

## Pipeline CI/CD: GitHub Actions Fallido

```bash
# Ver el fallo en GitHub:
# Repositorio → Actions → [workflow fallido] → [job fallido] → Ver logs

# Fallo común 1: Tests fallaron
# FAIL src/modules/finance/finance.service.spec.ts
# Solución: correr los tests localmente, corregir antes del PR

# Fallo común 2: Lint error
# 15:3  error  'any' is not allowed  @typescript-eslint/no-explicit-any
# Solución: correr npm run lint:fix localmente

# Fallo común 3: Build de TypeScript falla en CI pero no localmente
# Causa probable: diferencia de versión de Node
# Solución: verificar que .nvmrc o engines en package.json están actualizados
```

## Proceso de Deploy Manual de Emergencia

Si GitHub Actions está caído y hay un hotfix crítico:

```bash
# 1. Asegurarse de estar en la rama correcta con el fix
git status

# 2. Build manual
npm run build

# 3. Deploy directo a Railway
railway up --detach

# 4. Verificar que el deploy fue exitoso
railway logs | tail -30 | grep "Application is running"

# 5. DESPUÉS: documentar el deploy manual en el audit trail del equipo
```

## Monitoreo Post-Deploy

Después de cualquier deploy a producción, verificar durante los primeros 10 minutos:

```bash
# Health check
curl https://api.arellan.pe/health

# Logs en tiempo real
railway logs --tail 50 --follow

# En Sentry: verificar que no aparecen nuevos errores
# https://sentry.io → proyecto arellan → Issues
```

Si aparecen errores nuevos en Sentry dentro de los 10 minutos post-deploy: rollback inmediato.
