# Environments

Diferencias entre development, staging y production.

## Resumen

| Aspecto | Development | Staging | Production |
|---------|-------------|---------|------------|
| Host | localhost | Railway (branch) | Railway / AWS |
| DB | PostgreSQL local (Docker) | Supabase staging project | Supabase prod / AWS RDS |
| Redis | Redis local (Docker) | Railway Redis | Railway Redis Pro / ElastiCache |
| Auth | Supabase Auth staging | Supabase Auth staging | Supabase Auth prod |
| Culqi | Sandbox keys | Sandbox keys | Live keys |
| SUNAT | Nubefact demo | Nubefact demo | Nubefact producción |
| WhatsApp | Sin envío real | Sin envío real | Meta Business API live |
| ZKTeco ADMS | Simulado (mock endpoint) | Simulado | Real (hardware en taller) |
| MFA obligatorio | Opcional | Opcional | OBLIGATORIO |
| Logs | Console + archivo | Railway logs | Railway logs + CloudWatch |
| SSL | Auto (localhost:3000) | Railway auto-SSL | Railway / CloudFront |

## Development

### Cómo levantar

```bash
# Clonar y setup inicial
git clone https://github.com/arellan-tech/arellan-backend-core
cd arellan-backend-core
cp .env.example .env.local

# Levantar servicios con Docker Compose
docker compose up -d postgres redis

# Instalar y migrar
npm install
npx prisma migrate dev
npx prisma db seed

# Iniciar backend
npm run start:dev
```

### Accesos locales

| Servicio | URL |
|---------|-----|
| Backend API | http://localhost:3000 |
| Swagger UI | http://localhost:3000/api |
| PostgreSQL | postgresql://localhost:5432/arellan_dev |
| Redis | redis://localhost:6379 |
| Bull Board (queues) | http://localhost:3000/admin/queues |

## Staging

Entorno de validación pre-producción. Deploy automático desde rama `develop` via GitHub Actions.

### Accesos

| Servicio | URL |
|---------|-----|
| Backend API | https://api-staging.arellan.pe |
| Frontend Web | https://staging.arellan.pe |
| Mechanic UI | https://mechanics-staging.arellan.pe |

### Características

- Datos de prueba (seed ejecutado en cada deploy de staging)
- Culqi sandbox → pagos reales NO se procesan
- SUNAT Nubefact demo → facturas NO van al sistema real
- WhatsApp → mensajes no se envían (endpoint silenciado)
- ZKTeco → endpoint mock que acepta cualquier payload

### Deploy a staging

```bash
# Automático: merge a develop → GitHub Actions despliega a staging
git checkout develop
git merge feature/mi-feature
git push origin develop
# → CI corre, Railway despliega automáticamente
```

## Production

Entorno real del taller Arellan en Surquillo.

### Accesos

| Servicio | URL |
|---------|-----|
| Backend API | https://api.arellan.pe |
| Frontend Web | https://app.arellan.pe |
| Mechanic UI | https://mecanicos.arellan.pe |
| Mobile App | Expo / PWA instalada |
| Portal Clientes | https://estado.arellan.pe |

### Deploy a producción

```bash
# Manual: merge de release/* → main → aprobación manual en GitHub Actions
git checkout main
git merge release/1.2.0
git push origin main
# → CI corre, ESPERA aprobación manual de Edgar o Tech Lead
# → Luego Railway despliega
```

**Ventana de deploy:** Domingos 9 PM o Lunes 5 AM. Nunca Lunes-Sábado 7 AM - 6 PM (horario del taller).

Ver `arellan-governance/release-process.md` para proceso completo.

### Acceso a producción

Solo Edgar Arellan (OWNER) y el Tech Lead tienen acceso directo a variables de producción. Los desarrolladores trabajan solo en staging.

```bash
# Railway CLI (requiere acceso al proyecto)
railway login
railway environment production
railway logs --follow
```

## Diferencias de Configuración por Ambiente

### JWT Expiry

| Ambiente | Token Admin | Token Mecánico |
|---------|-------------|----------------|
| Development | 24h | 72h |
| Staging | 1h | 12h |
| Production | 1h | 12h |

### Rate Limits

| Ambiente | Login | Finance endpoints | Portal público |
|---------|-------|-------------------|----------------|
| Development | Sin límite | Sin límite | Sin límite |
| Staging | 5/min | 10/min | 30/min |
| Production | 5/min | 10/min | 30/min |

### Logs

```typescript
// Development
Logger.log('Cashbox opened', 'CashboxService');   // Console colorido

// Production
// Winston → JSON → Railway Logs → CloudWatch (Fase 5)
{
  "level": "info",
  "message": "Cashbox opened",
  "service": "CashboxService",
  "accountId": "uuid",
  "timestamp": "2024-01-15T07:00:00.000Z"
}
```
