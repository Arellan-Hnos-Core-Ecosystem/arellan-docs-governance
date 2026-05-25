# ADR 004: NestJS 10 sobre Express Puro

## Estado

`Aprobado`

## Contexto

El backend de Arellan (`arellan-backend-core`) necesita:

- **RBAC robusto:** 6 roles (OWNER, ADMIN, FINANCE, MECHANIC, TRAINEE, CLIENT) con permisos granulares por endpoint
- **MFA obligatorio:** Guards que bloqueen acceso sin TOTP verificado para módulos financieros
- **Audit log automático:** Interceptores que registren toda acción crítica sin contaminar código de negocio
- **Validación de DTOs:** Todas las entradas validadas y transformadas automáticamente
- **Queue processing:** BullMQ para notificaciones, webhooks Culqi, jobs de ZKTeco
- **WebSocket:** Socket.io para eventos en tiempo real (alertas joyride, cambios de OT, discrepancias de caja)
- **Cron jobs:** Forzar checkout a las 11:59 PM para registros ZKTeco sin salida
- **Módulos escalables:** 8+ módulos (Auth, Orders, Finance, Inventory, Employees, Vehicles, Audit, Notifications)

**Restricción de equipo:** El equipo tiene experiencia en Node.js pero no en arquitectura enterprise a escala. Se necesita un framework que imponga estructura, no solo una librería.

## Alternativas Consideradas

### Opción 1: Express.js Puro

Framework minimalista, solo maneja routing básico.

**Pros:**
- Máxima flexibilidad
- Bundle size mínimo
- Equipo ya conoce Express básico

**Contras:**
- RBAC, guards, interceptores → implementar todo desde cero
- Sin DI (Dependency Injection) → testing difícil, clases con dependencias hard-coded
- Sin decoradores nativos → decorar endpoints con roles = boilerplate manual
- Sin estructura de módulos → a los 3 meses el proyecto se convierte en spaghetti
- Validación de DTOs = instalar y configurar express-validator o joi por separado
- Sin CLI para generar módulos/servicios/controladores con estructura consistente
- Para el nivel de complejidad de Arellan, Express requeriría construir un framework encima

### Opción 2: Fastify

Framework Node.js de alto rendimiento.

**Pros:**
- ~2x más rápido que Express en benchmarks de throughput
- Schema validation nativo con JSON Schema
- TypeScript support mejorado vs Express

**Contras:**
- Sin DI nativo → mismo problema que Express para módulos complejos
- Sin guards/interceptores declarativos como NestJS
- Ecosistema de plugins menos maduro para RBAC + WebSocket + Queues integradas
- El performance advantage no justifica el costo: Arellan tiene <100 req/min de pico

### Opción 3: NestJS 10 ← Seleccionada

Framework Node.js enterprise con arquitectura inspirada en Angular, usando decoradores y DI.

**Pros:**
- DI nativo → servicios fácilmente testables con mocks/stubs
- Guards declarativos: `@UseGuards(JwtAuthGuard, RolesGuard, MfaRequiredGuard)` en el controller
- Interceptores: `AuditInterceptor` registra automáticamente sin modificar controllers
- Pipes: `ValidationPipe` valida y transforma DTOs automáticamente en cada request
- Módulos con encapsulamiento real → imposible que `OrdersModule` acceda a internals de `FinanceModule` sin importar explícitamente
- `@nestjs/bull` para BullMQ integrado
- `@nestjs/schedule` para cron jobs declarativos (`@Cron('59 23 * * *')`)
- `@nestjs/websockets` para Socket.io nativo
- CLI: `nest generate module finance` → estructura correcta garantizada
- Testing utilities: `Test.createTestingModule()` con DI real para unit tests
- TypeScript first desde el diseño

**Contras:**
- Más boilerplate que Express para casos simples (irrelevante: el sistema no es simple)
- Curva de aprendizaje inicial (decoradores, DI, módulos) → 1-2 semanas
- Bundle size mayor → irrelevante en backend

## Decisión

Seleccionamos **NestJS 10 con TypeScript 5 strict**.

### Patrón de Módulo

```
src/
├── modules/
│   ├── auth/
│   │   ├── auth.module.ts
│   │   ├── auth.controller.ts    ← Solo routing y decoradores
│   │   ├── auth.service.ts       ← Lógica de negocio
│   │   ├── dto/
│   │   │   ├── login.dto.ts
│   │   │   └── verify-mfa.dto.ts
│   │   └── guards/
│   │       ├── jwt-auth.guard.ts
│   │       └── mfa-required.guard.ts
│   ├── finance/
│   │   ├── finance.module.ts
│   │   ├── cashbox.controller.ts
│   │   ├── cashbox.service.ts
│   │   ├── expenses.service.ts
│   │   └── dto/
│   └── ...
├── common/
│   ├── guards/
│   │   └── roles.guard.ts
│   ├── interceptors/
│   │   └── audit.interceptor.ts
│   ├── decorators/
│   │   └── roles.decorator.ts
│   └── pipes/
│       └── parse-uuid.pipe.ts
└── main.ts
```

### Guards en Cascada

```typescript
// Orden de guards: autenticación → roles → MFA
@Controller('finance')
@UseGuards(JwtAuthGuard, RolesGuard)
export class CashboxController {

  @Post('cashbox/close')
  @Roles(Role.ADMIN, Role.FINANCE, Role.OWNER)
  @UseGuards(MfaRequiredGuard)   // Adicional para endpoints financieros
  async closeCashbox(@Body() dto: CloseCashboxDto) {
    return this.cashboxService.close(dto);
  }
}
```

### DI para Testing

```typescript
// Unit test con DI real de NestJS
const module = await Test.createTestingModule({
  providers: [
    ExpenseService,
    { provide: PrismaService, useValue: mockPrisma },
    { provide: AuditService, useValue: mockAudit },
  ],
}).compile();

const service = module.get<ExpenseService>(ExpenseService);
```

## Consecuencias

### Positivas

- Guards declarativos → sin posibilidad de olvidar proteger un endpoint nuevo
- AuditInterceptor automático → audit log siempre registra sin acoplamiento con controllers
- DI → unit tests limpios con mocks de PrismaService
- Módulos con fronteras claras → escalar a 10 devs sin caos de imports circular

### Negativas / Trade-offs

- Boilerplate inicial mayor que Express → justificado por la complejidad del sistema
- NestJS tiene su propia forma de hacer las cosas → devs Express necesitan 1-2 semanas de adaptación

### Neutras

- Performance: NestJS overhead es ~10ms/request en cold → irrelevante para <100 req/min
- Bundle: NestJS + TypeScript compilado → deployment en Docker sin issues

## Notas de Implementación

- Entry: `src/main.ts` con `ValidationPipe` global y `ClassSerializerInterceptor`
- Global config: `app.useGlobalPipes(new ValidationPipe({ whitelist: true, transform: true }))`
- Guards: `JwtAuthGuard` usa Passport JWT, `RolesGuard` usa `Reflector`
- Ver `arellan-platform-governance/standards/backend-standards.md` para patrones
- Ver `arellan-governance/code-standards.md` para TypeScript config

---

*Fecha de decisión: 2025-Q1*
*Autor: Diego Soto (Tech Lead)*
*Revisado por: Edgar Arellan (Owner)*
