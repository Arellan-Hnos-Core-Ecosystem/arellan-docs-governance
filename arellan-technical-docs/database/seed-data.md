# Seed Data

Datos iniciales para entornos de desarrollo y staging.

## Ejecutar Seed

```bash
# Desde arellan-backend-core/
npx prisma db seed

# O via npm script
npm run db:seed
```

## Usuarios Creados

### Owners (Edgar y Juan Arellan)

```typescript
{
  email: 'edgar@arellan.pe',
  fullName: 'Edgar Arellan',
  role: 'OWNER',
  password: 'DevPass123!',   // Solo en dev/staging — cambiar en primer login
  mfaEnabled: false,          // Configurar TOTP en primer ingreso
  status: 'ACTIVE',
}

{
  email: 'juan@arellan.pe',
  fullName: 'Juan Arellan',
  role: 'OWNER',
  password: 'DevPass123!',
  mfaEnabled: false,
  status: 'ACTIVE',
}
```

### Admin (Ana García)

```typescript
{
  email: 'ana@arellan.pe',
  fullName: 'Ana García',
  role: 'ADMIN',
  password: 'DevPass123!',
  mfaEnabled: false,
  zktecoBiometricId: 'E001',
  status: 'ACTIVE',
}
```

### Finance (hija de Edgar)

```typescript
{
  email: 'hija@arellan.pe',
  fullName: 'Valeria Arellan',
  role: 'FINANCE',
  password: 'DevPass123!',
  mfaEnabled: false,
  zktecoBiometricId: 'E002',
  status: 'ACTIVE',
}
```

### Mecánicos

```typescript
{
  email: 'mecanico1@arellan.pe',
  fullName: 'Carlos Quispe',
  role: 'MECHANIC',
  password: 'DevPass123!',
  zktecoBiometricId: 'E003',
  status: 'ACTIVE',
}

{
  email: 'mecanico2@arellan.pe',
  fullName: 'Luis Torres',
  role: 'MECHANIC',
  password: 'DevPass123!',
  zktecoBiometricId: 'E004',
  status: 'ACTIVE',
}
```

### Ex-empleado (para tests de seguridad)

```typescript
{
  email: 'ricardo@arellan.pe',
  fullName: 'Ricardo Mendoza',
  role: 'MECHANIC',
  password: 'DevPass123!',
  zktecoBiometricId: 'E005',
  status: 'TERMINATED',  // Acceso revocado — para test de RBAC
  deletedAt: new Date('2024-01-01'),
}
```

## Inventario Inicial

Items de muestra para pruebas del módulo de inventario:

```typescript
[
  { sku: 'ACE-10W40-1L', name: 'Aceite Motor 10W-40 1L', category: 'ACEITES', currentStock: 24, minimumStock: 10, unitCost: 25.00 },
  { sku: 'ACE-15W40-1L', name: 'Aceite Motor 15W-40 1L', category: 'ACEITES', currentStock: 18, minimumStock: 8, unitCost: 22.00 },
  { sku: 'FIL-ACE-001', name: 'Filtro de Aceite Universal', category: 'FILTROS', currentStock: 15, minimumStock: 10, unitCost: 12.00 },
  { sku: 'FIL-AIR-001', name: 'Filtro de Aire Toyota', category: 'FILTROS', currentStock: 8, minimumStock: 5, unitCost: 18.00 },
  { sku: 'PAD-FRONT-001', name: 'Pastillas Freno Delanteras', category: 'FRENOS', currentStock: 3, minimumStock: 4, unitCost: 45.00 },  // LOW STOCK
  { sku: 'LIQ-FRENO-500ML', name: 'Líquido de Frenos DOT4 500ml', category: 'FRENOS', currentStock: 6, minimumStock: 5, unitCost: 15.00 },
]
```

El ítem `PAD-FRONT-001` tiene stock por debajo del mínimo → dispara alerta `inventory:low_stock` via WebSocket en el seed.

## Vehículos del Taller

```typescript
[
  {
    plate: 'XYZ-789',
    model: 'Toyota Hilux 2020',
    status: 'DISPONIBLE',
    lastLat: -12.1095,
    lastLng: -77.0282,
    insideFence: true,
    gpsDeviceId: 'TELTONIKA-IMEI-001',
  }
]
```

## Caja del Día (seed para testing)

```typescript
// No se crea en seed — se crea manualmente para cada test
// Ver arellan-e2e-tests para tests de caja completos
```

## OT de Muestra

```typescript
{
  orderNumber: 'OT-2024-0001',
  plate: 'ABC-123',
  vehicleModel: 'Toyota Corolla 2019',
  mileageIn: 45230,
  fuelLevel: 'HALF',
  clientDescription: 'Hace ruido al frenar — pastillas posiblemente gastadas',
  status: 'EN_PROCESO',
  assignedMechanicId: 'carlos-quispe-uuid',
  estimatedDeliveryAt: new Date('2024-01-15T17:00:00.000Z'),
}
```

## Notas

- Las contraseñas `DevPass123!` son solo para desarrollo. En staging usar contraseñas distintas.
- En producción el seed **no** se ejecuta automáticamente. Los usuarios de producción se crean manualmente por el Owner con MFA.
- El seed limpia (`deleteMany()`) todas las tablas antes de insertar → destructivo, solo para dev.
- Archivo: `arellan-backend-core/prisma/seed.ts`
