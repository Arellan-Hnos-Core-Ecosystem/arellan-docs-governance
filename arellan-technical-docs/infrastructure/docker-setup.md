# Docker Setup

Cómo levantar el entorno local de desarrollo con Docker Compose.

## Prerequisitos

- Docker Desktop 4.x+
- Node.js 20 LTS
- npm 10+

## Levantar Entorno Local

### Paso 1: Clonar y configurar variables

```bash
git clone https://github.com/arellan-tech/arellan-backend-core
cd arellan-backend-core
cp .env.example .env.local
# Editar .env.local con los valores de development (ver environments.md)
```

### Paso 2: Levantar servicios con Docker Compose

```bash
docker compose up -d
# Levanta: PostgreSQL 15, Redis 7, MailHog (email dev)
```

### Paso 3: Verificar servicios

```bash
docker compose ps
# Debe mostrar los 3 contenedores en estado "Up"
```

### Paso 4: Migrations y seed

```bash
npm install
npx prisma migrate dev
npx prisma db seed
# Crea usuarios edgar@arellan.pe, ana@arellan.pe, mecanico1@arellan.pe
```

### Paso 5: Iniciar backend

```bash
npm run start:dev
# Servidor en http://localhost:3000
# Swagger en http://localhost:3000/api
# Bull Board en http://localhost:3000/admin/queues
```

## docker-compose.yml

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15-alpine
    container_name: arellan_postgres
    environment:
      POSTGRES_USER: arellan
      POSTGRES_PASSWORD: arellan_dev_pass
      POSTGRES_DB: arellan_dev
    ports:
      - '5432:5432'
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ['CMD-SHELL', 'pg_isready -U arellan']
      interval: 5s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    container_name: arellan_redis
    command: redis-server --requirepass arellan_redis_pass
    ports:
      - '6379:6379'
    volumes:
      - redis_data:/data

  mailhog:
    image: mailhog/mailhog
    container_name: arellan_mail
    ports:
      - '1025:1025'   # SMTP
      - '8025:8025'   # Web UI para ver emails de dev

volumes:
  postgres_data:
  redis_data:
```

## Comandos Útiles

```bash
# Ver logs de PostgreSQL
docker compose logs postgres -f

# Ver logs de Redis
docker compose logs redis -f

# Conectar a PostgreSQL directamente
docker exec -it arellan_postgres psql -U arellan -d arellan_dev

# Acceder a Redis CLI
docker exec -it arellan_redis redis-cli -a arellan_redis_pass

# Detener todos los servicios (conserva datos)
docker compose stop

# Destruir y recrear (limpia datos — útil si DB en estado inconsistente)
docker compose down -v
docker compose up -d
npx prisma migrate dev
npx prisma db seed
```

## Troubleshooting

### PostgreSQL no inicia

```bash
# Error: port 5432 already in use
sudo lsof -ti:5432 | xargs kill -9
docker compose up -d postgres
```

### Prisma no puede conectar

```bash
# Verificar que DATABASE_URL en .env.local coincide con docker-compose
# Local: postgresql://arellan:arellan_dev_pass@localhost:5432/arellan_dev

# Verificar que el contenedor está corriendo
docker compose ps postgres
```

### Redis connection refused

```bash
# Verificar REDIS_URL y REDIS_PASSWORD en .env.local
# Local:
# REDIS_URL=redis://localhost:6379
# REDIS_PASSWORD=arellan_redis_pass
```

### Reset completo de datos

```bash
docker compose down -v             # Borra volúmenes
docker compose up -d               # Recrea contenedores
npx prisma migrate reset --force   # Reset de migrations + seed
```

## Servicios del Hardware (opcional)

Para testing del módulo IoT sin hardware real:

```bash
# ADMS mock (simula ZKTeco enviando punch events)
cd arellan-iot-hardware-bridge
npm run mock:adms

# Envía un evento de check-in de ejemplo cada 30 segundos
# Ver logs en arellan-backend-core para confirmar procesamiento
```

## Entorno de Frontend

```bash
# Frontend Web (Next.js)
cd arellan-frontend-web
npm install
cp .env.example .env.local
# NEXT_PUBLIC_API_URL=http://localhost:3000
npm run dev   # http://localhost:3001

# Mechanic UI (Vite)
cd arellan-mechanic-ui
npm install
cp .env.example .env.local
# VITE_API_URL=http://localhost:3000
npm run dev   # http://localhost:5173
```
