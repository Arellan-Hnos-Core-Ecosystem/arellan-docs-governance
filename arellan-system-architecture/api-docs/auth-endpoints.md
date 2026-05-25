# Auth Endpoints

## POST /auth/login

Iniciar sesión con email y contraseña.

**Auth requerida:** No
**Rate limit:** 5 intentos/min por IP

**Request:**
```json
{
  "email": "ana@arellan.pe",
  "password": "MiContraseña123!"
}
```

**Response 200 — Login exitoso (rol sin MFA requerida):**
```json
{
  "accessToken": "eyJhbGciOiJSUzI1NiJ9...",
  "refreshToken": "rt-uuid-hash...",
  "user": {
    "id": "uuid",
    "email": "ana@arellan.pe",
    "fullName": "Ana García",
    "role": "ADMIN",
    "mfaVerified": false
  }
}
```

**Response 200 — MFA pendiente (roles OWNER/ADMIN/FINANCE):**
```json
{
  "mfaPending": true,
  "mfaToken": "temp-token-uuid",  // Token temporal, expira en 5 minutos
  "message": "Ingresa el código de Google Authenticator"
}
```

**Response 401:** Credenciales inválidas
**Response 429:** Demasiados intentos

---

## POST /auth/mfa/verify

Verificar código TOTP para completar el login.

**Auth requerida:** No (usa mfaToken del paso anterior)

**Request:**
```json
{
  "mfaToken": "temp-token-uuid",
  "totpCode": "123456"
}
```

**Response 200:**
```json
{
  "accessToken": "eyJhbGciOiJSUzI1NiJ9...",
  "refreshToken": "rt-uuid-hash...",
  "user": { "id": "uuid", "role": "ADMIN", "mfaVerified": true }
}
```

**Response 401:** Código TOTP inválido o expirado

---

## POST /auth/refresh

Renovar el access token usando el refresh token.

**Auth requerida:** No (usa refreshToken en body)

**Request:**
```json
{
  "refreshToken": "rt-uuid-hash..."
}
```

**Response 200:**
```json
{
  "accessToken": "nuevo-eyJhbGciOiJSUzI1NiJ9...",
  "refreshToken": "nuevo-rt-uuid-hash..."
}
```

**Response 401:** Refresh token inválido o revocado

---

## DELETE /auth/logout

Cerrar sesión y revocar el refresh token.

**Auth requerida:** Sí

**Response 204:** Sesión cerrada correctamente

---

## GET /auth/profile

Obtener el perfil del usuario autenticado.

**Auth requerida:** Sí

**Response 200:**
```json
{
  "id": "uuid",
  "email": "ana@arellan.pe",
  "fullName": "Ana García",
  "role": "ADMIN",
  "mfaEnabled": true,
  "mfaVerified": true,
  "lastLoginAt": "2024-01-15T08:00:00.000Z"
}
```

---

## POST /auth/mfa/setup

Iniciar configuración de MFA (obtener QR para Google Authenticator).

**Auth requerida:** Sí (solo el propio usuario puede hacer setup de su MFA)

**Response 200:**
```json
{
  "qrCodeUrl": "data:image/png;base64,...",   // QR para escanear con Google Authenticator
  "backupCodes": ["xxxx-xxxx", "yyyy-yyyy"],  // Códigos de respaldo (mostrar una sola vez)
  "secret": "JBSWY3DPEHPK3PXP"               // Secret para ingreso manual
}
```

---

## POST /auth/sessions/revoke-all

Revocar todas las sesiones activas del usuario (útil si se sospecha de acceso no autorizado).

**Auth requerida:** Sí (OWNER puede revocar sesiones de cualquier usuario)

**Request (si OWNER revoca otro usuario):**
```json
{
  "targetUserId": "uuid-del-usuario"
}
```

**Response 200:**
```json
{
  "revokedSessions": 3,
  "message": "Todas las sesiones han sido revocadas"
}
```
