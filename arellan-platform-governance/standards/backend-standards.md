NestJS: estructura de modulos, naming
# Estándares de Backend — NestJS

Convenciones específicas para el desarrollo del backend del ecosistema Arellan con NestJS + TypeScript + Prisma.

## Estructura de Módulos

Cada dominio de negocio es un módulo NestJS independiente:

```
src/modules/[nombre-modulo]/
├── [nombre-modulo].module.ts       ← Registra providers e imports
├── [nombre-modulo].controller.ts   ← HTTP layer únicamente
├── [nombre-modulo].service.ts      ← Business logic
├── dto/                            ← Data Transfer Objects
│   ├── create-[nombre-modulo].dto.ts
│   ├── update-[nombre-modulo].dto.ts
│   └── [nombre-modulo]-filter.dto.ts
└── __tests__/
    ├── [nombre-modulo].service.spec.ts
    └── [nombre-modulo].controller.spec.ts
```

## Inyección de Dependencias

```typescript
// ✅ Constructor injection (siempre)
@Injectable()
export class ExpenseService {
  constructor(
    private readonly prisma: PrismaService,
    private readonly auditService: AuditService,      // Obligatorio en módulos de negocio
    private readonly notificationsService: NotificationsService,
  ) {}
}

// ❌ No usar property injection
@Inject()
private prisma: PrismaService  // No hacer esto
```

## Manejo de Errores

```typescript
// ✅ Usar excepciones de NestJS con mensajes en español y código de error
throw new NotFoundException(`Orden de trabajo ${id} no encontrada`)
throw new ForbiddenException('No puedes aprobar tu propio gasto')
throw new ConflictException('La caja ya fue abierta hoy')
throw new BadRequestException('El monto debe ser mayor a 0')

// Para errores con código específico (para el frontend):
throw new ForbiddenException({
  message: 'Segregación de funciones: el solicitante no puede aprobar',
  code: 'SELF_APPROVAL_FORBIDDEN',
})
```

## Logging

```typescript
// ✅ Usar Logger de NestJS, nunca console.log
private readonly logger = new Logger(ExpenseService.name)

this.logger.log(`Gasto ${id} aprobado por ${approverId}`)
this.logger.error(`Error al procesar gasto ${id}`, error.stack)
this.logger.warn(`Diferencia de caja detectada: S/.${discrepancy}`)
this.logger.debug(`Query params recibidos: ${JSON.stringify(filter)}`)  // Solo en dev
```

## Transacciones de Base de Datos

Para operaciones que modifican múltiples tablas:

```typescript
// ✅ Usar transacciones para mantener consistencia
async approveExpenseAndPay(expenseId: string, approverId: string) {
  return this.prisma.$transaction(async (tx) => {
    const expense = await tx.expense.update({
      where: { id: expenseId },
      data: { status: 'PAID', approvedBy: approverId, approvedAt: new Date() },
    })

    await tx.financialTransaction.create({
      data: {
        amount: expense.amount,
        type: 'EXPENSE',
        referenceId: expenseId,
      },
    })

    await tx.auditLog.create({
      data: {
        action: 'EXPENSE_PAID',
        entityType: 'expenses',
        entityId: expenseId,
        performedBy: approverId,
      },
    })

    return expense
  })
}
// Si cualquier paso falla, TODA la transacción se revierte
```

## Guards y Decoradores

```typescript
// Siempre en este orden en los controllers
@Controller('finance/expenses')
@UseGuards(JwtAuthGuard, RolesGuard)  // Auth primero, luego roles
export class ExpensesController {

  @Post()
  @Roles(UserRole.FINANCE, UserRole.ADMIN)  // Roles permitidos
  async create(@Body() dto: CreateExpenseDto, @CurrentUser() user: AuthUser) { ... }

  @Post(':id/approve')
  @Roles(UserRole.ADMIN, UserRole.OWNER)
  @UseGuards(JwtAuthGuard, MfaRequiredGuard, RolesGuard)  // MFA para aprobaciones
  async approve(@Param('id') id: string, @CurrentUser() user: AuthUser) { ... }
}
```

## Validación y Transformación

```typescript
// En main.ts — global pipes
app.useGlobalPipes(new ValidationPipe({
  whitelist: true,          // Remover campos no declarados en el DTO
  forbidNonWhitelisted: true, // Error si vienen campos no permitidos
  transform: true,          // Transformar automáticamente tipos (string → number)
  transformOptions: {
    enableImplicitConversion: true,
  },
}))
```

## Prisma — Buenas Prácticas

```typescript
// ✅ Usar select para no exponer campos sensibles
const expense = await this.prisma.expense.findUnique({
  where: { id },
  select: {
    id: true,
    amount: true,
    status: true,
    description: true,
    // NO incluir: providerBankAccount, internalNotes, etc.
  },
})

// ✅ Incluir relaciones solo cuando se necesitan
const order = await this.prisma.workOrder.findUnique({
  where: { id },
  include: {
    photos: true,
    assignedMechanic: { select: { id: true, fullName: true } },
    // NO incluir: client.phone, client.dni (se enmascaran en el gateway)
  },
})

// ❌ Nunca hacer N+1 queries
const orders = await this.prisma.workOrder.findMany()
for (const order of orders) {
  const mechanic = await this.prisma.account.findUnique(...)  // N+1 ❌
}

// ✅ Incluir relaciones en la query principal
const orders = await this.prisma.workOrder.findMany({
  include: { assignedMechanic: true },  // Un solo query ✅
})
```
