# Estándares de Frontend — Next.js + React

Convenciones para el desarrollo del panel admin (`arellan-frontend-web`) y el portal cliente (`arellan-client-portal`) con Next.js 14 + TypeScript + Tailwind CSS.

## Estructura de Componentes

```
src/
├── app/                    ← Next.js App Router (páginas)
│   ├── (auth)/
│   │   └── login/page.tsx
│   ├── (dashboard)/
│   │   ├── orders/page.tsx
│   │   ├── finance/page.tsx
│   │   └── layout.tsx
│   └── layout.tsx
├── components/
│   ├── ui/                 ← Solo primitivos de @arellan/ui
│   ├── features/           ← Componentes de dominio
│   │   ├── orders/
│   │   │   ├── OrderList.tsx
│   │   │   ├── OrderCard.tsx
│   │   │   └── OrderStatusBadge.tsx
│   │   └── finance/
│   │       ├── CashboxPanel.tsx
│   │       └── ExpenseForm.tsx
│   └── layout/
│       ├── Sidebar.tsx
│       └── TopNav.tsx
├── hooks/                  ← Custom hooks compartidos
├── lib/                    ← Utilitarios, configuración de API
├── stores/                 ← Estado global con Zustand
└── types/                  ← TypeScript types del frontend
```

## Gestión de Estado

### Server State → TanStack Query (Obligatorio)

```typescript
// ✅ Para datos del servidor, siempre TanStack Query
function useOrders(filters?: OrderFilters) {
  return useQuery({
    queryKey: ['orders', filters],
    queryFn: () => api.orders.getAll(filters),
    staleTime: 30_000,        // 30 segundos antes de refetch
    gcTime: 5 * 60_000,       // 5 minutos en cache
    refetchOnWindowFocus: true,
  })
}

// ✅ Para mutaciones
function useApproveExpense() {
  const queryClient = useQueryClient()

  return useMutation({
    mutationFn: (id: string) => api.expenses.approve(id),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['expenses'] })
      queryClient.invalidateQueries({ queryKey: ['cashbox'] })
      toast.success('Gasto aprobado')
    },
    onError: (error) => {
      toast.error(getErrorMessage(error))
    },
  })
}

// ❌ No usar useState para datos del servidor
const [orders, setOrders] = useState<Order[]>([])
useEffect(() => { fetchOrders().then(setOrders) }, [])  // No hacer esto
```

### UI State → Zustand

```typescript
// ✅ Zustand solo para estado de UI que persiste entre componentes
const useSidebarStore = create<SidebarStore>((set) => ({
  isCollapsed: false,
  toggle: () => set(state => ({ isCollapsed: !state.isCollapsed })),
}))

// ✅ Estado local de UI simple → useState
const [isModalOpen, setIsModalOpen] = useState(false)

// ❌ Zustand no es para server state (usar TanStack Query)
const useOrdersStore = create(() => ({
  orders: [],  // ❌ Esto debería ser useQuery
}))
```

## Formularios

```typescript
// ✅ React Hook Form + Zod (siempre)
const createExpenseSchema = z.object({
  amount: z.number({ message: 'Ingresa un monto válido' })
    .positive('El monto debe ser positivo')
    .max(100_000, 'Monto máximo: S/.100,000'),
  description: z.string()
    .min(5, 'Mínimo 5 caracteres')
    .max(500, 'Máximo 500 caracteres'),
  category: z.nativeEnum(ExpenseCategory),
  providerId: z.string().uuid('Selecciona un proveedor válido'),
})

type CreateExpenseForm = z.infer<typeof createExpenseSchema>

function ExpenseForm() {
  const { register, handleSubmit, formState: { errors, isSubmitting } } = useForm<CreateExpenseForm>({
    resolver: zodResolver(createExpenseSchema),
  })

  // ...
}
```

## Componentes — Reglas

```typescript
// ✅ Componentes funcionales siempre (no class components)
// ✅ TypeScript explícito en props
interface OrderCardProps {
  order: WorkOrder
  onStatusChange?: (id: string, status: OrderStatus) => void
  className?: string
}

function OrderCard({ order, onStatusChange, className }: OrderCardProps) {
  // ...
}

// ❌ No usar any en props
function OrderCard({ order }: any) { ... }

// ✅ Exportar componentes de la forma más simple (no default exports en features)
export { OrderCard }
export type { OrderCardProps }
```

## Tailwind CSS

```tsx
// ✅ Usar tokens del design system cuando existan
<div className="bg-brand-500 text-brand-foreground">

// ✅ cn() utility para clases condicionales
import { cn } from '@/lib/utils'

<div className={cn(
  'rounded-lg border p-4',
  isHighlighted && 'border-brand-500 bg-brand-50',
  className
)}>

// ❌ No usar estilos inline cuando Tailwind puede hacer lo mismo
<div style={{ backgroundColor: '#3b82f6' }}>  // No si hay clase de Tailwind equivalente
```

## mechanic-ui (Tablet) — Reglas Adicionales

```tsx
// ✅ Botones touch-friendly (mínimo 56px de alto)
<button className="min-h-[56px] px-6 text-xl font-semibold">
  Iniciar trabajo
</button>

// ✅ Texto grande para lectura con guantes
<p className="text-2xl font-medium">Placa: {plate}</p>

// ✅ Espaciado generoso entre elementos táctiles
<div className="grid grid-cols-1 gap-4">  // No gap-1 en tablet UI
```
