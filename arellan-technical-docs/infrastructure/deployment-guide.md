# Deployment Guide

Paso a paso para desplegar en Railway (MVP) y AWS (producción).

## Railway — MVP

### Servicios en Railway

| Servicio | Repo | Deploy trigger |
|---------|------|---------------|
| `arellan-api` | arellan-backend-core | Push a `main` (previa aprobación) |
| `arellan-iot-bridge` | arellan-iot-hardware-bridge | Push a `main` |
| PostgreSQL | Railway add-on | Permanente |
| Redis | Railway add-on | Permanente |

### Primer Deploy (setup)

```bash
# Instalar Railway CLI
npm install -g @railway/cli

# Login
railway login

# Vincular proyecto
railway link

# Deploy manual inicial
railway up
```

### Variables de Entorno en Railway

```bash
# Configurar desde CLI (o desde el dashboard)
railway variables set DATABASE_URL="postgresql://..."
railway variables set REDIS_URL="redis://..."
railway variables set CULQI_SECRET_KEY="sk_live_..."
# ... resto de variables de enviroment-variables.md
```

### Deploy Automático (GitHub Actions)

El CI/CD corre en cada PR y push a `main`:

```yaml
# .github/workflows/deploy.yml
name: Deploy to Railway

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production   # Requiere aprobación manual
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm ci
      - run: npm run build
      - run: npx prisma migrate deploy
        env:
          DATABASE_URL: ${{ secrets.DIRECT_DATABASE_URL }}
      - uses: bervProject/railway-deploy@main
        with:
          railway_token: ${{ secrets.RAILWAY_TOKEN }}
          service: arellan-api
```

### Rollback en Railway

```bash
# Ver deployments anteriores
railway deployments

# Rollback al anterior
railway rollback

# O especificar deployment ID
railway rollback DEPLOYMENT_ID
```

### Verificar Deploy

```bash
# Ver logs en tiempo real
railway logs --follow

# Health check
curl https://api.arellan.pe/health
# Esperado: {"status":"ok","database":"connected","redis":"connected"}
```

## Vercel — Frontends

### arellan-frontend-web (Next.js)

```bash
# Instalar Vercel CLI
npm install -g vercel

# Deploy staging
vercel

# Deploy producción
vercel --prod
```

Variables en Vercel Dashboard:
- `NEXT_PUBLIC_API_URL` = `https://api.arellan.pe`
- `NEXT_PUBLIC_SUPABASE_URL` = `https://...supabase.co`
- `NEXT_PUBLIC_SUPABASE_ANON_KEY` = `eyJ...`

### arellan-mechanic-ui (Vite PWA)

```bash
# Build
npm run build

# Deploy a Vercel
vercel --prod
# O configurar en Vercel Dashboard con build command: npm run build, output: dist
```

## AWS — Producción Escalada (Fase 3+)

Ver `arellan-enterprise-roadmap/scaling-plan.md` para plan completo de migración Railway → AWS.

### Stack AWS Target

```
Internet → CloudFront → ALB → ECS Fargate (arellan-api)
                              ECS Fargate (arellan-iot-bridge)

ECS → RDS PostgreSQL 15 (Multi-AZ)
ECS → ElastiCache Redis (cluster mode)
ECS → S3 (fotos, documentos)
```

### Dockerfile (arellan-backend-core)

```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npx prisma generate
RUN npm run build

FROM node:20-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/prisma ./prisma
EXPOSE 3000
CMD ["node", "dist/main.js"]
```

### Docker Build y Push a ECR

```bash
# Build imagen
docker build -t arellan-api .

# Tag para ECR
docker tag arellan-api:latest AWS_ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com/arellan-api:latest

# Push a ECR
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin AWS_ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com

docker push AWS_ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com/arellan-api:latest
```

### Migrations en AWS

```bash
# Correr migrations antes del nuevo deployment (ECS task one-off)
aws ecs run-task \
  --cluster arellan-cluster \
  --task-definition arellan-migrations \
  --overrides '{"containerOverrides": [{"name": "api", "command": ["npx", "prisma", "migrate", "deploy"]}]}'
```

## Post-Deploy Checklist

Después de cada deploy a producción:

- [ ] `GET /health` retorna `{"status":"ok"}`
- [ ] Login funciona con edgar@arellan.pe
- [ ] Crear OT de prueba y cambiar estado
- [ ] Abrir caja → verificar que no hay error 409
- [ ] WebSocket conecta (verificar en frontend)
- [ ] BullMQ queues vacías (sin jobs stuck en `/admin/queues`)
- [ ] Ver Railway logs: sin errores en los primeros 2 minutos
- [ ] Notificar a Edgar vía WhatsApp: "Deploy completado ✓"

## Monitoreo

```bash
# Railway logs en tiempo real
railway logs --follow

# PostgreSQL conexiones activas
SELECT count(*), state FROM pg_stat_activity GROUP BY state;

# Redis memoria
redis-cli -u $REDIS_URL INFO memory | grep used_memory_human

# BullMQ jobs atascados
# Ver /admin/queues en Bull Board dashboard
```
