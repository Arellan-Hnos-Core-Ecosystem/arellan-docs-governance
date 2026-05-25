# Guía de Onboarding Técnico

Para nuevos miembros del equipo técnico del ecosistema digital Arellan. Cubre la configuración del entorno local, accesos necesarios y primeras tareas.

## Antes de Empezar

Necesitas que Edgar o Juan te den acceso a:
- Organización GitHub `arellan-tech` (acceso al repositorio específico que trabajarás)
- Proyecto Railway (si trabajas en backend/infra)
- Proyecto Supabase (si trabajas en auth/storage)
- Canal de comunicación del equipo técnico

**Regla de seguridad:** Los owners son los únicos con acceso de Admin a la organización GitHub. Como developer, tendrás rol Write solo en los repos que trabajes.

## Configuración del Entorno Local

### Prerequisitos

```bash
# Versiones requeridas
node --version    # v20 LTS o v24
npm --version     # v10+
docker --version  # 24+
git --version     # 2.40+

# Opcional pero recomendado
psql --version    # Para consultas directas a DB
redis-cli         # Para inspeccionar colas BullMQ
```

### 1. Clonar Repositorios

```bash
# Clonar el repo principal que trabajarás
git clone git@github.com:arellan-tech/arellan-platform.git

# Configurar identidad Git (usar email corporativo)
git config user.email "tu-nombre@arellan.pe"
git config user.name "Tu Nombre"
```

### 2. Levantar Entorno Local con Docker

```bash
# En arellan-infra-devops/ o en el repo principal
docker compose up -d

# Servicios que levanta:
# PostgreSQL 15 en localhost:5432
# Redis en localhost:6379
```

### 3. Variables de Entorno

```bash
# Copiar template de variables de entorno
cp .env.example .env

# Editar con los valores de desarrollo (NO usar valores de producción)
# Los valores de desarrollo te los da el tech lead
# NUNCA commitear el .env
```

### 4. Migraciones y Seed

```bash
# Aplicar migraciones de Prisma
npx prisma migrate dev

# Seed: crear datos de prueba
npx prisma db seed

# Esto crea:
# - Usuario OWNER: edgar@arellan.pe / password: Dev123!
# - Usuario ADMIN: ana@arellan.pe / password: Dev123!
# - Usuario MECHANIC: mecanico1@arellan.pe / password: Dev123!
```

### 5. Iniciar el Backend

```bash
npm run start:dev

# El servidor corre en http://localhost:3001
# Hot-reload activado (NestJS watch mode)
```

## Herramientas Recomendadas

| Herramienta | Uso | Instalación |
|-------------|-----|-------------|
| **VS Code** | Editor principal | code.visualstudio.com |
| **TablePlus** | GUI para PostgreSQL | tableplus.com |
| **Another Redis Desktop** | GUI para Redis/BullMQ | No obligatorio |
| **Insomnia** o **Hoppscotch** | Testing de APIs | Preferencia personal |
| **Prisma Extension (VS Code)** | Syntax highlighting de Prisma schema | VS Code marketplace |

## Estructura de Carpetas del Backend

```
src/
├── modules/          ← Un módulo por dominio de negocio
│   ├── auth/
│   ├── orders/
│   ├── finance/
│   ├── inventory/
│   ├── clients/
│   ├── vehicles/
│   ├── personnel/
│   └── audit/
├── common/           ← Guards, decorators, filters compartidos
│   ├── guards/
│   ├── decorators/
│   └── interceptors/
├── prisma/           ← Schema, migraciones, seed
└── main.ts
```

## Tu Primera Tarea

1. **Leer la arquitectura:** `arellan-system-architecture/overview.md`
2. **Leer el schema de DB:** `arellan-technical-docs/database/schema-overview.md`
3. **Correr los tests:** `npm test` — deben pasar al 100%
4. **Hacer un cambio pequeño:** Agregar un campo al DTO de creación de OT y su validación
5. **Abrir un PR:** Seguir el proceso documentado en `git-workflow.md`

## Reglas de Seguridad (Obligatorio)

- **Nunca** commitear el `.env` ni ningún secreto
- **Nunca** hacer push directamente a `main` o `develop`
- **Nunca** compartir credenciales del entorno de desarrollo
- **Siempre** usar SSH para autenticarte en GitHub (no HTTPS con contraseña)
- **Activar 2FA** en tu cuenta de GitHub antes de que te den acceso al org

## Preguntas Frecuentes

**¿Por qué 13 repositorios separados?**
Isolación por dominio de negocio y confidencialidad. Los mecánicos no tienen acceso al código de finanzas. Ver `arellan-system-architecture/decisions/001-monorepo-vs-microservices.md`.

**¿Por qué NestJS y no Express puro?**
Ver `arellan-system-architecture/decisions/004-nestjs-over-express.md`.

**¿Puedo usar `any` en TypeScript?**
No. ESLint lo detecta como error. Siempre tipificar correctamente.

**¿Cuál es el flujo para llevar un feature a producción?**
feature/* → PR → code review → develop → staging → release/* → main. Ver `git-workflow.md`.
