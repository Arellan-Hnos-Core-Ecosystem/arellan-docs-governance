# Guía para Agentes de IA (Cline / Claude Code)

Instrucciones para agentes de IA autónomos que operen dentro de los repositorios del ecosistema Arellan. Define lo que está permitido, lo que está prohibido y el contexto necesario para asistir sin causar daño.

## Contexto del Proyecto

Este es el ecosistema digital de la **Clínica Automotriz Arellan Hnos** (Surquillo, Lima, Perú). El sistema resuelve fraude interno en una empresa familiar de mecánica automotriz.

**Roles del sistema:**
- `OWNER` — Edgar y Juan (hermanos, dueños del taller)
- `ADMIN` — Ana (administradora)
- `FINANCE` — Hija de Edgar (finanzas)
- `MECHANIC` — Mecánicos del taller
- `TRAINEE` — Practicantes
- `CLIENT` — Clientes del portal

**Stack:**
- Backend: NestJS 10 + TypeScript 5 + Prisma ORM + PostgreSQL 15
- Frontend: Next.js 14 + Tailwind + Zustand + TanStack Query
- Auth: Supabase Auth (MVP) → JWT RS256 (producción)

## Acciones Permitidas

```
✅ Scaffolding de módulos NestJS siguiendo la estructura existente
✅ Implementar DTOs con class-validator
✅ Agregar guards de autenticación y autorización
✅ Escribir tests unitarios y de integración
✅ Leer y escribir archivos de documentación markdown
✅ Instalar dependencias npm (sin bajar versiones existentes)
✅ Crear migraciones de Prisma (sin aplicarlas a producción)
✅ Refactorizar código siguiendo los estándares de este repo
✅ Generar seed data para desarrollo local
```

## Acciones Prohibidas (Nunca sin confirmación explícita del owner)

```
❌ Commitear o pushear código directamente a main o develop
❌ Modificar archivos .env o generar nuevos secretos
❌ Ejecutar migraciones en base de datos de producción
❌ Agregar endpoints SIN guards de autorización
❌ Introducir any en TypeScript
❌ Agregar console.log que no sean temporales de debug
❌ Instalar dependencias con vulnerabilidades conocidas
❌ Crear users o modificar roles en la base de datos
❌ Tocar la tabla audit_logs (append-only, nunca modificar)
❌ Exponer datos de clientes (phone, email, DNI) a roles de mecánico
```

## Reglas de Seguridad Críticas

### 1. Audit Log es Sagrado

La tabla `audit_logs` en PostgreSQL es append-only. Nunca generes código que:
- Tenga `DELETE FROM audit_logs`
- Tenga `UPDATE audit_logs`
- Tenga un endpoint `DELETE /audit-logs/:id`
- Tenga una migración que borre o modifique audit_logs

Si necesitas "corregir" un audit log, la respuesta correcta es agregar un nuevo registro de corrección, nunca modificar el existente.

### 2. Guards en Todo Endpoint Nuevo

Cada nuevo `@Controller` o método de controlador debe tener:
```typescript
@UseGuards(JwtAuthGuard, RolesGuard)
@Roles(UserRole.ADMIN)  // O el rol apropiado
```

Si el endpoint es público (sin auth), debe estar explícitamente marcado:
```typescript
@Public()  // Decorator custom que bypass JwtAuthGuard
@Get('status/:plate')  // Solo para consultas públicas justificadas
```

### 3. Data Masking para Mecánicos

Nunca generes código que exponga al rol MECHANIC:
- `client.phone` — teléfono del cliente
- `client.email` — email del cliente
- `client.dni` — documento de identidad
- `client.address` — dirección
- Datos financieros de ningún tipo

Los mecánicos solo necesitan: placa del vehículo, modelo del año, descripción del trabajo a hacer.

### 4. Validación en DTOs

Todo nuevo DTO debe usar class-validator:
```typescript
// ✅ Correcto
export class CreateOrderDto {
  @IsString()
  @IsNotEmpty()
  @MaxLength(10)
  plate: string

  @IsNumber()
  @Min(0)
  @Max(999999)
  mileage: number
}
```

## Contexto de Módulos

| Módulo | Descripción | Sensibilidad |
|--------|-------------|-------------|
| `auth` | Login, MFA, sesiones, JWT | Crítico |
| `finance` | Caja, gastos, transacciones | Crítico |
| `orders` | Órdenes de trabajo | Alto |
| `inventory` | Stock de repuestos | Medio |
| `clients` | Datos de clientes | Alto (Ley 29733) |
| `vehicles` | Vehículos en taller | Alto |
| `personnel` | Empleados, asistencia | Alto |
| `audit` | Registro inmutable | Crítico (solo append) |
