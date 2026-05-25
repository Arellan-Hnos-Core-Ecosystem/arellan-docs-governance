# Kiro CLI — Reglas de Scaffolding

Configuración y mandatos para el agente Kiro CLI cuando genera estructura de directorios y código boilerplate en los repositorios del ecosistema Arellan.

## Principios de Generación

### 1. Clean Architecture en NestJS

Todo módulo generado debe seguir la estructura de Clean Architecture:

```
modules/[nombre-modulo]/
├── [nombre-modulo].module.ts       ← Registro de providers e imports
├── [nombre-modulo].controller.ts   ← Solo routing HTTP y respuestas
├── [nombre-modulo].service.ts      ← Lógica de negocio y acceso a DB
├── dto/
│   ├── create-[nombre-modulo].dto.ts
│   ├── update-[nombre-modulo].dto.ts
│   └── [nombre-modulo]-response.dto.ts
└── __tests__/
    ├── [nombre-modulo].service.spec.ts
    └── [nombre-modulo].controller.spec.ts
```

### 2. Guards por Defecto

Cada nuevo controller generado debe incluir por defecto:

```typescript
@Controller('[nombre-modulo]')
@UseGuards(JwtAuthGuard, RolesGuard)  // SIEMPRE incluir guards
export class [NombreModulo]Controller {
  // ...
}
```

Nunca generar controllers sin autenticación a menos que sea explícitamente solicitado como endpoint público.

### 3. Inyección de AuditService

Módulos que modifican datos de negocio deben inyectar `AuditService`:

```typescript
constructor(
  private readonly prisma: PrismaService,
  private readonly auditService: AuditService,  // Siempre presente en módulos de negocio
) {}
```

### 4. Convenciones de Nombres (Obligatorias)

```
Archivos:         kebab-case.ts
Clases/Interfaces: PascalCase
Funciones:        camelCase
Variables:        camelCase
Constantes:       SCREAMING_SNAKE_CASE
DB tables:        snake_case plural
DB columns:       snake_case
```

## Templates de Código Base

### Template Controller

```typescript
import { Controller, Get, Post, Body, Param, UseGuards } from '@nestjs/common'
import { JwtAuthGuard } from '../auth/guards/jwt-auth.guard'
import { RolesGuard } from '../auth/guards/roles.guard'
import { Roles } from '../auth/decorators/roles.decorator'
import { CurrentUser } from '../auth/decorators/current-user.decorator'
import { UserRole } from '../shared-types/enums'
import { [NombreModulo]Service } from './[nombre-modulo].service'
import { Create[NombreModulo]Dto } from './dto/create-[nombre-modulo].dto'
import { AuthUser } from '../auth/interfaces/auth-user.interface'

@Controller('[nombre-modulo]')
@UseGuards(JwtAuthGuard, RolesGuard)
export class [NombreModulo]Controller {
  constructor(private readonly [nombreModulo]Service: [NombreModulo]Service) {}

  @Post()
  @Roles(UserRole.ADMIN, UserRole.OWNER)
  async create(
    @Body() dto: Create[NombreModulo]Dto,
    @CurrentUser() user: AuthUser,
  ) {
    return this.[nombreModulo]Service.create(dto, user.id)
  }

  @Get()
  @Roles(UserRole.ADMIN, UserRole.OWNER, UserRole.FINANCE)
  async findAll() {
    return this.[nombreModulo]Service.findAll()
  }

  @Get(':id')
  async findOne(@Param('id') id: string) {
    return this.[nombreModulo]Service.findOne(id)
  }
}
```

### Template Service

```typescript
import { Injectable, NotFoundException } from '@nestjs/common'
import { PrismaService } from '../../prisma/prisma.service'
import { AuditService } from '../audit/audit.service'
import { Create[NombreModulo]Dto } from './dto/create-[nombre-modulo].dto'

@Injectable()
export class [NombreModulo]Service {
  constructor(
    private readonly prisma: PrismaService,
    private readonly auditService: AuditService,
  ) {}

  async create(dto: Create[NombreModulo]Dto, userId: string) {
    const entity = await this.prisma.[nombreModulo].create({ data: dto })

    await this.auditService.log({
      action: '[NOMBRE_MODULO]_CREATED',
      entityType: '[nombre_modulo]s',
      entityId: entity.id,
      performedBy: userId,
    })

    return entity
  }

  async findAll() {
    return this.prisma.[nombreModulo].findMany({
      orderBy: { createdAt: 'desc' },
    })
  }

  async findOne(id: string) {
    const entity = await this.prisma.[nombreModulo].findUnique({ where: { id } })
    if (!entity) throw new NotFoundException(`[NombreModulo] ${id} no encontrado`)
    return entity
  }
}
```

## Lo que Kiro NO debe generar

```
❌ Archivos .env con valores reales
❌ Claves privadas o secretos
❌ Endpoints de DELETE o UPDATE sobre audit_logs
❌ Controllers sin guards
❌ Cualquier uso de `any` en TypeScript
❌ console.log en código de producción
❌ SQL con concatenación de strings (injection risk)
❌ Migraciones que modifiquen audit_logs
```
