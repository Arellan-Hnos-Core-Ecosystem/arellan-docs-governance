# Estándares de Testing

Define qué se testea, cómo y cuánto en el ecosistema Arellan. El objetivo es tener confianza en el código sin testing excesivo que ralentice el desarrollo.

## Pirámide de Tests

```
              ┌─────────────┐
              │  E2E Tests  │  15% — Flujos completos de negocio (Playwright)
              ├─────────────┤
              │ Integration │  35% — Servicios + DB real (Testcontainers)
              ├─────────────┤
              │  Unit Tests │  50% — Guards, validators, lógica pura (Jest)
              └─────────────┘
```

## Cobertura Mínima por Módulo

| Módulo | Cobertura mínima | Justificación |
|--------|-----------------|---------------|
| `auth` | 95% | Seguridad crítica |
| `finance` | 90% | Módulo de dinero real |
| `audit` | 85% | Trazabilidad legal |
| `orders` | 80% | Flujo principal del negocio |
| `inventory` | 80% | |
| `clients` | 80% | Datos personales (Ley 29733) |
| `vehicles` | 75% | |
| `personnel` | 75% | |
| Frontend components | 60% | UI tests menos críticos |

## Tests Unitarios — Qué Testear

```typescript
// ✅ Testear: guards, validators, transformaciones de datos, lógica de negocio pura
describe('MfaRequiredGuard', () => {
  it('should block OWNER without MFA', ...)
  it('should allow MECHANIC without MFA (not required)', ...)
  it('should allow OWNER with MFA verified', ...)
})

// ✅ Testear: boundary conditions (exactamente en el límite)
describe('ExpenseService.getRequiredApprovalLevel', () => {
  it.each([
    [100, 'FINANCE'],    // Límite inferior de FINANCE
    [101, 'ADMIN'],      // Límite inferior de ADMIN
    [500, 'ADMIN'],      // Límite superior de ADMIN
    [501, 'OWNER'],      // Límite inferior de OWNER
    [2000, 'OWNER'],
    [2001, 'DUAL_OWNER'], // Límite de doble aprobación
  ])('amount %i → level %s', (amount, expectedLevel) => {
    expect(service.getRequiredApprovalLevel(amount)).toBe(expectedLevel)
  })
})

// ❌ No testear: getters simples, constantes, código trivial
describe('UserRole enum', () => {
  it('should have OWNER role', () => {
    expect(UserRole.OWNER).toBe('OWNER')  // No value en testear esto
  })
})
```

## Tests de Integración — Qué Testear

```typescript
// ✅ Testear: flujos completos con DB real (Testcontainers)
describe('CashboxService integration', () => {
  // Testcontainers levanta PostgreSQL 15 real para estos tests
  it('should correctly compute discrepancy and log to audit_trail', ...)
  it('should reject duplicate cashbox open on same day', ...)
  it('should enforce immutability of audit_logs via DB rules', ...)
})

// ✅ Testear: integraciones con Redis/BullMQ
describe('NotificationService integration', () => {
  it('should enqueue push job when expense requires owner approval', ...)
  it('should not enqueue duplicate jobs (idempotency)', ...)
})

// ✅ Testear: endpoints HTTP completos con Supertest
describe('POST /api/v1/finance/expenses', () => {
  it('should return 403 when mechanic tries to create expense', ...)
  it('should return 400 when amount is negative', ...)
  it('should return 201 and queue notification when amount > 500', ...)
})
```

## Tests E2E — Flujos Críticos Obligatorios

Los siguientes flujos DEBEN tener cobertura E2E con Playwright:

```
1. Login con MFA → acceder a módulo de finanzas
2. Crear OT → asignar mecánico → completar → pagar con QR → estado ENTREGADO
3. Crear gasto S/.750 → owner recibe push → owner aprueba → estado PAID
4. Apertura caja → registrar ingreso → cierre con diferencia → alerta a owner
5. Mecánico intenta acceder a /finance → 403 esperado
6. Crear, intentar borrar audit_log → error esperado
```

## Configuración de Jest

```typescript
// jest.config.ts
export default {
  moduleNameMapper: {
    '^@/(.*)$': '<rootDir>/src/$1',
  },
  coverageThreshold: {
    global: { lines: 80, functions: 80, branches: 80 },
    './src/modules/finance/': { lines: 90, functions: 90, branches: 90 },
    './src/modules/auth/': { lines: 95, functions: 95, branches: 95 },
  },
  testEnvironment: 'node',
  setupFilesAfterFramework: ['./jest.setup.ts'],
}
```

## Datos de Test

```typescript
// ✅ Usar faker-js para datos realistas
import { faker } from '@faker-js/faker/locale/es'

const testExpense = {
  amount: faker.number.float({ min: 50, max: 2000, fractionDigits: 2 }),
  description: faker.commerce.productDescription(),
  providerId: faker.string.uuid(),
}

// ✅ Fixtures para datos que deben ser específicos
const fixtures = {
  ownerUser: { id: 'test-owner-id', role: UserRole.OWNER, mfaVerified: true },
  mechanicUser: { id: 'test-mechanic-id', role: UserRole.MECHANIC, mfaVerified: false },
}

// ❌ No hardcodear datos que aparecen en producción (DNIs, teléfonos, nombres reales)
const testClient = { dni: '12345678', phone: '999888777' }  // No hacer esto
```

## CI/CD

Los tests se ejecutan automáticamente en GitHub Actions en dos etapas:

1. **En PR hacia develop:** Unit + integration tests (< 5 minutos)
2. **En merge a develop:** Unit + integration + E2E tests (< 15 minutos)

Un PR no puede mergearse si los tests fallan o la cobertura baja del mínimo.
