# Estándares de Código

Convenciones y reglas técnicas que aplican a todos los repositorios del ecosistema Arellan. El objetivo es tener un codebase coherente, mantenible y seguro independientemente de quién haya escrito cada parte.

## Backend — NestJS + TypeScript

### TypeScript Estricto (Obligatorio)

```json
// tsconfig.json — configuración base para todos los backends
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "exactOptionalPropertyTypes": true
  }
}
```

### Estructura de Módulos NestJS

```
src/
├── modules/
│   ├── auth/
│   │   ├── auth.module.ts
│   │   ├── auth.controller.ts   ← Solo routing y respuesta HTTP
│   │   ├── auth.service.ts      ← Lógica de negocio
│   │   ├── auth.guard.ts        ← Guards de autorización
│   │   ├── dto/
│   │   │   ├── login.dto.ts
│   │   │   └── mfa-verify.dto.ts
│   │   └── __tests__/
│   │       └── auth.service.spec.ts
│   └── finance/
│       └── (misma estructura)
├── common/
│   ├── decorators/
│   ├── filters/
│   ├── guards/
│   └── interceptors/
└── main.ts
```

### Reglas de Controladores

```typescript
// ✅ Correcto: controller solo hace routing
@Controller('finance/expenses')
export class ExpensesController {
  constructor(private readonly expensesService: ExpensesService) {}

  @Post()
  @UseGuards(JwtAuthGuard, RolesGuard)
  @Roles(UserRole.FINANCE, UserRole.ADMIN)
  async create(
    @Body() dto: CreateExpenseDto,
    @CurrentUser() user: AuthUser,
  ) {
    return this.expensesService.create(dto, user.id)
  }
}

// ❌ Incorrecto: lógica de negocio en el controller
@Post()
async create(@Body() dto: CreateExpenseDto) {
  const expense = await this.prisma.expense.create({ data: dto })  // NO
  if (expense.amount > 500) {
    await this.pushService.notify(...)  // NO
  }
  return expense
}
```

### Validación con class-validator (Obligatorio)

```typescript
// Todos los DTOs usan class-validator
export class CreateExpenseDto {
  @IsNumber()
  @IsPositive()
  @Max(100_000)  // Prevenir montos absurdos
  amount: number

  @IsString()
  @IsNotEmpty()
  @MaxLength(500)
  description: string

  @IsUUID()
  providerId: string

  @IsEnum(ExpenseCategory)
  category: ExpenseCategory

  @IsOptional()
  @IsUrl()
  quotationUrl?: string
}
```

### Prohibiciones Absolutas en Backend

```typescript
// ❌ NUNCA: console.log en producción
console.log('Debug info')  // Usar logger.debug()

// ❌ NUNCA: credenciales hardcodeadas
const apiKey = 'sk_live_xxxxx'  // Usar process.env + Secrets Manager

// ❌ NUNCA: SQL raw sin parámetros preparados
prisma.$executeRaw(`SELECT * FROM users WHERE id = ${userId}`)  // SQL injection

// ✅ SIEMPRE: SQL raw con parámetros
prisma.$executeRaw`SELECT * FROM users WHERE id = ${userId}`  // Tagged template = seguro

// ❌ NUNCA: any en TypeScript
const data: any = response.data  // Tipar correctamente

// ❌ NUNCA: catch sin log
try { ... } catch (e) { }  // Siempre loguear el error
```

## Frontend — Next.js + TypeScript

### Estructura de Componentes

```
components/
├── ui/                    ← Primitivos de @arellan/ui
├── features/
│   ├── orders/
│   │   ├── OrderList.tsx
│   │   ├── OrderCard.tsx
│   │   └── hooks/
│   │       └── useOrders.ts
│   └── finance/
│       └── (misma estructura)
└── layout/
    ├── AppLayout.tsx
    └── Sidebar.tsx
```

### Hooks y Estado

```typescript
// ✅ Usar TanStack Query para fetching
function useOrders() {
  return useQuery({
    queryKey: ['orders', 'active'],
    queryFn: () => api.orders.getActive(),
    staleTime: 30_000,  // 30 segundos
  })
}

// ✅ Usar Zustand para estado UI global (no para server state)
const useCashboxStore = create<CashboxStore>((set) => ({
  isOpen: false,
  openCashbox: () => set({ isOpen: true }),
  closeCashbox: () => set({ isOpen: false }),
}))

// ❌ No usar useState para datos del servidor
const [orders, setOrders] = useState([])  // Usar useQuery en su lugar
```

### Formularios

```typescript
// ✅ React Hook Form + Zod para validación
const schema = z.object({
  amount: z.number().positive().max(100_000),
  description: z.string().min(5).max(500),
  category: z.nativeEnum(ExpenseCategory),
})

function ExpenseForm() {
  const { register, handleSubmit, formState: { errors } } = useForm({
    resolver: zodResolver(schema),
  })
  ...
}
```

## Seguridad en Código

### Lo que NUNCA debe aparecer en el código (ni en comentarios)

- Contraseñas o tokens reales
- API keys de producción
- Datos reales de clientes (DNI, teléfonos)
- Credenciales de base de datos
- Secretos JWT

El secret scanning de GitHub detectará automáticamente muchos de estos y bloqueará el push.

### Guards en Todos los Endpoints

```typescript
// ✅ Todo endpoint nuevo debe tener guards explícitos
@Get('finance/reports')
@UseGuards(JwtAuthGuard, MfaRequiredGuard, RolesGuard)
@Roles(UserRole.OWNER, UserRole.FINANCE)
async getFinancialReports() { ... }

// ❌ Nunca dejar un endpoint sin autenticación (a menos que sea público y anotado)
@Get('internal/debug')
async getDebugInfo() { ... }  // Sin guard = endpoint público = vulnerabilidad
```

## Linting y Formato (Automático)

```json
// .eslintrc — reglas importantes
{
  "rules": {
    "@typescript-eslint/no-explicit-any": "error",
    "@typescript-eslint/no-unused-vars": "error",
    "no-console": "warn",  // warn en dev, error en CI
    "no-secrets/no-secrets": "error"  // Detectar secretos en código
  }
}
```

```json
// .prettierrc — formato estándar
{
  "semi": false,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "es5",
  "printWidth": 100
}
```

El pre-commit hook con Husky corre ESLint + Prettier antes de cada commit. El CI también verifica esto.
