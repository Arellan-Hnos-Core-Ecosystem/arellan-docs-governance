# ADR 005: Turborepo para el Monorepo de Frontends

## Estado

`Aprobado`

## Contexto

Arellan tiene tres aplicaciones frontend con código compartido:

- `arellan-frontend-web` — Next.js 14 (OWNER, ADMIN, FINANCE)
- `arellan-mechanic-ui` — React 18 + Vite PWA (mecánicos en tablets)
- `arellan-mobile-app` — Expo React Native (dueños en móvil)

Estas tres apps comparten:
- Componentes UI (botones, modales, formularios de OT)
- Tipos TypeScript (interfaces de API, enums de estado)
- Utilidades (formateo de moneda peruana PEN, validación de placas, fechas Lima timezone)
- Config de ESLint/Prettier/TypeScript

**Sin Turborepo:** Cada app mantiene su propia copia de estos elementos → inconsistencias, bugs que se arreglan en una app pero no en las otras, y trabajo duplicado.

**Restricción:** El backend ya es un repo separado. El debate no es mono-repo total (backend + frontend juntos) sino si los tres frontends deberían compartir un workspace.

## Alternativas Consideradas

### Opción 1: Tres repos completamente independientes

Cada frontend en su propio repo sin package compartido.

**Pros:**
- Setup más simple
- Deploy completamente independiente

**Contras:**
- Componente `<OrderStatusBadge>` → duplicado en 3 repos → 3 bugs distintos cuando cambia el diseño
- Tipos TypeScript de la API → duplicados → inconsistencias cuando el backend cambia un campo
- ESLint/Prettier config → 3 copias → drifting rules entre repos
- `formatCurrency(amount: number)` (S/. con separador de miles peruano) → 3 implementaciones
- Cualquier cambio de diseño global = 3 PRs separados

### Opción 2: npm workspaces sin Turborepo

Monorepo simple con npm workspaces.

**Pros:**
- Sin dependencia externa (solo npm nativo)
- Package hoisting automático

**Contras:**
- Sin task orchestration → `npm run build` en raíz no sabe qué construir primero
- Sin remote caching → cada CI rebuild completo aunque nada cambió
- Sin dependency graph → `mechanic-ui` se rebuil aunque solo `frontend-web` cambió
- Sin pipeline tasks → parallelización manual

### Opción 3: Turborepo ← Seleccionada

Build system con cache inteligente y orquestación de tasks para monorepos.

**Pros:**
- **Remote cache:** Si `@arellan/ui` no cambió, Turborepo devuelve el resultado cacheado en segundos (no minutos)
- **Task pipeline:** Define que `build` de apps depende del `build` de `@arellan/ui` → orden automático
- **Parallel builds:** `frontend-web` y `mechanic-ui` buildean en paralelo cuando no hay dependencias entre ellos
- **Affected-only:** Solo rebuil lo que cambió y sus dependientes
- Compatible con Vercel Remote Cache (gratis para proyectos Vercel)
- Sin lock-in: si se necesita salir, el monorepo sigue siendo npm workspaces estándar

**Contras:**
- Configuración inicial (`turbo.json`) de ~20 líneas
- Aprendizaje de conceptos de pipeline (mínimo)

## Decisión

Seleccionamos **Turborepo** con la siguiente estructura:

```
arellan-frontend/ (o arellan-mechanic-ui como repo raíz)
├── apps/
│   ├── web/           ← arellan-frontend-web (Next.js 14)
│   ├── mechanic-ui/   ← React 18 + Vite PWA
│   └── mobile/        ← Expo React Native (Fase 3)
├── packages/
│   ├── ui/            ← @arellan/ui (componentes compartidos)
│   ├── types/         ← @arellan/types (interfaces TypeScript)
│   ├── utils/         ← @arellan/utils (formatCurrency, etc.)
│   └── config/        ← ESLint, Prettier, TypeScript base configs
├── turbo.json
└── package.json
```

### turbo.json

```json
{
  "$schema": "https://turbo.build/schema.json",
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": [".next/**", "dist/**"]
    },
    "lint": {
      "dependsOn": ["^build"]
    },
    "test": {
      "dependsOn": ["^build"],
      "outputs": ["coverage/**"]
    },
    "dev": {
      "cache": false,
      "persistent": true
    }
  },
  "remoteCache": {
    "enabled": true
  }
}
```

### Packages Compartidos

**`@arellan/ui`** — Componentes Tailwind compartidos:
```typescript
export { OrderStatusBadge } from './components/OrderStatusBadge';
export { CurrencyDisplay } from './components/CurrencyDisplay';
export { MobileButton } from './components/MobileButton';  // 56px touch target
```

**`@arellan/types`** — Tipos de la API:
```typescript
export type { OrderStatus, Role, PaymentMethod } from './api';
export type { WorkOrder, Employee, InventoryItem } from './models';
```

**`@arellan/utils`** — Utilidades peruanas:
```typescript
export function formatPEN(amount: number): string {
  return new Intl.NumberFormat('es-PE', {
    style: 'currency',
    currency: 'PEN',
  }).format(amount);
}

export function formatLimaDate(isoString: string): string {
  return new Intl.DateTimeFormat('es-PE', {
    timeZone: 'America/Lima',
    dateStyle: 'long',
    timeStyle: 'short',
  }).format(new Date(isoString));
}
```

## Consecuencias

### Positivas

- Un cambio en `@arellan/ui` se propaga a las 3 apps automáticamente en el next build
- CI/CD: Vercel Remote Cache → builds 3-5x más rápidos después del primer build
- Consistencia: mismo `formatPEN()` en web, mechanic-ui y mobile
- TypeScript: el compilador detecta si la app usa un campo eliminado de `@arellan/types`

### Negativas / Trade-offs

- `packages/ui` tiene que ser building correctamente antes de cada app build → gestionar con Turborepo pipeline (resuelto con `dependsOn: ["^build"]`)
- Si se necesita actualizar solo `mechanic-ui` en emergencia → `turbo run build --filter=mechanic-ui` funciona
- Mobile (Expo) puede tener incompatibilidades con algunos componentes web → separar `packages/ui-native` cuando se necesite

### Neutras

- `arellan-shared-types` (el repo separado mencionado en ADR-001) se reemplaza/complementa con `@arellan/types` en el monorepo frontend → backend sigue publicando su propio package para que el backend pueda tipar sus responses

## Notas de Implementación

- Remote cache: Vercel Remote Cache en CI (`TURBO_TOKEN` + `TURBO_TEAM` env vars)
- Dev: `turbo run dev` inicia todas las apps en paralelo
- Ver `arellan-platform-governance/standards/frontend-standards.md` para convenciones de componentes
- Components naming: PascalCase, archivos kebab-case (`order-status-badge.tsx`)

---

*Fecha de decisión: 2025-Q1*
*Autor: Diego Soto (Tech Lead)*
*Revisado por: Edgar Arellan (Owner)*
