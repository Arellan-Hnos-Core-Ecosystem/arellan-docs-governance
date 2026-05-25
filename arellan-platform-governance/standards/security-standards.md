# Estándares de Seguridad

Estándares de seguridad de la información aplicables a todos los repositorios del ecosistema Arellan. Basados en las políticas definidas en `arellan-security-compliance/arellan-security-governance`.

## Autenticación y Sesiones

### JWT RS256

```
Access Token:
  Algoritmo: RS256 (clave privada en AWS Secrets Manager)
  Duración:  1 hora (roles admin/finance/owner)
             12 horas (mechanic — tablet compartida)
  Payload:   { sub: userId, role: UserRole, mfaVerified: boolean, iat, exp }

Refresh Token:
  Duración:  7 días
  Storage:   Hash bcrypt en tabla refresh_tokens (nunca texto plano)
  Revocación: Inmediata al logout, cambio de contraseña, o forced logout por owner
```

### MFA (Obligatorio para Roles Críticos)

```typescript
// Roles que REQUIEREN MFA verificada para acceder a módulos sensibles
const MFA_REQUIRED_ROLES = [UserRole.OWNER, UserRole.ADMIN, UserRole.FINANCE]

@Injectable()
export class MfaRequiredGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const user: AuthUser = context.switchToHttp().getRequest().user
    if (MFA_REQUIRED_ROLES.includes(user.role) && !user.mfaVerified) {
      throw new ForbiddenException({
        message: 'Este módulo requiere verificación MFA',
        code: 'MFA_REQUIRED',
      })
    }
    return true
  }
}
```

## Control de Acceso (RBAC)

### Matriz de Permisos

| Módulo | OWNER | ADMIN | FINANCE | MECHANIC | TRAINEE | CLIENT |
|--------|-------|-------|---------|---------|---------|--------|
| Órdenes (ver) | ✅ | ✅ | ✅ | Solo asignadas | Solo asignadas | Solo propias |
| Órdenes (crear) | ✅ | ✅ | ❌ | ✅ | Con supervisión | ❌ |
| Caja (ver) | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |
| Caja (operar) | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |
| Gastos (crear) | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |
| Gastos (aprobar) | ✅ (>S/.100) | ✅ (S/.101-500) | ✅ (≤S/.100) | ❌ | ❌ | ❌ |
| Inventario (ver) | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ |
| Inventario (ajustar) | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ |
| Clientes (datos completos) | ✅ | ✅ | ✅ | ❌ (solo placa) | ❌ | Solo propios |
| Personal | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ |
| Audit log | ✅ (solo lectura) | ✅ (solo lectura) | ❌ | ❌ | ❌ | ❌ |
| Configuración del sistema | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |

## Data Masking para Mecánicos

```typescript
// En el API Gateway, antes de enviar la respuesta a mechanic-ui
@Injectable()
export class MechanicDataMaskInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest()
    const user: AuthUser = request.user

    if (user.role !== UserRole.MECHANIC && user.role !== UserRole.TRAINEE) {
      return next.handle()
    }

    return next.handle().pipe(
      map(data => this.maskSensitiveData(data))
    )
  }

  private maskSensitiveData(data: any): any {
    if (data?.client) {
      const { phone, email, dni, address, ...safeClientData } = data.client
      return { ...data, client: safeClientData }
    }
    return data
  }
}
```

## Rate Limiting

```typescript
// Configuración de ThrottlerModule por módulo
const RATE_LIMITS = {
  global: { ttl: 60_000, limit: 100 },      // 100 req/min general
  finance: { ttl: 60_000, limit: 10 },       // 10 req/min módulo financiero
  auth: { ttl: 60_000, limit: 5 },           // 5 intentos/min (brute force protection)
  public: { ttl: 60_000, limit: 30 },        // 30 req/min portal público
}
```

## Datos Sensibles Cifrados (AES-256)

Campos que se almacenan cifrados en la base de datos:

```typescript
const ENCRYPTED_FIELDS = [
  'mfa_secret',             // Secreto TOTP del usuario
  'account_number_encrypted', // Cuentas bancarias del taller
  'provider_bank_account',  // Cuentas bancarias de proveedores
]

// Implementación con @nestjs/config + crypto-js
function encryptField(value: string): string {
  return CryptoJS.AES.encrypt(value, process.env.FIELD_ENCRYPTION_KEY).toString()
}

function decryptField(encrypted: string): string {
  return CryptoJS.AES.decrypt(encrypted, process.env.FIELD_ENCRYPTION_KEY).toString(CryptoJS.enc.Utf8)
}
```

## Prohibiciones Absolutas en el Código

```
❌ Contraseñas, tokens o API keys hardcodeadas en el código
❌ Datos de clientes (phone, DNI, email) en logs
❌ Tokens JWT en logs
❌ console.log con datos sensibles
❌ SQL raw con concatenación (SQL injection)
❌ eval() o Function() constructor
❌ Disable SSL/TLS verification
❌ process.env sin validación en startup
```

## HTTPS y Headers de Seguridad

```typescript
// main.ts — headers de seguridad con Helmet
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      styleSrc: ["'self'", "'unsafe-inline'"],  // Tailwind requiere esto
      imgSrc: ["'self'", 'data:', 'https://s3.amazonaws.com'],
    },
  },
  hsts: {
    maxAge: 31536000,  // 1 año
    includeSubDomains: true,
    preload: true,
  },
}))
```
