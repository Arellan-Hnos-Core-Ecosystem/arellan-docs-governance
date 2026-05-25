# Environment Variables

Variables de entorno requeridas por cada servicio. Sin valores reales — ver secrets en Railway/AWS.

## arellan-backend-core

```env
# ─────────────────────────────
# BASE DE DATOS
# ─────────────────────────────
DATABASE_URL=postgresql://USER:PASS@HOST:5432/arellan
# Con PgBouncer (Supabase transaction mode):
# DATABASE_URL=postgresql://USER:PASS@HOST:6543/arellan?pgbouncer=true
DIRECT_DATABASE_URL=postgresql://USER:PASS@HOST:5432/arellan  # Sin PgBouncer para migrations

# ─────────────────────────────
# REDIS
# ─────────────────────────────
REDIS_URL=redis://HOST:6379
REDIS_PASSWORD=STRONG_PASS

# ─────────────────────────────
# AUTENTICACIÓN
# ─────────────────────────────
# MVP: Supabase Auth
SUPABASE_URL=https://PROJECT_ID.supabase.co
SUPABASE_SERVICE_ROLE_KEY=eyJ...  # SECRETO — nunca en frontend

# Producción: JWT RS256
JWT_PRIVATE_KEY=-----BEGIN RSA PRIVATE KEY-----...
JWT_PUBLIC_KEY=-----BEGIN PUBLIC KEY-----...
JWT_EXPIRY_ADMIN=1h
JWT_EXPIRY_MECHANIC=12h

# ─────────────────────────────
# MFA (TOTP)
# ─────────────────────────────
MFA_TOTP_SECRET_ENCRYPTION_KEY=32_BYTE_AES_KEY_BASE64  # AES-256 para secrets TOTP
MFA_ISSUER=Arellan

# ─────────────────────────────
# CULQI (PAGOS QR)
# ─────────────────────────────
CULQI_PUBLIC_KEY=pk_test_...   # pk_live_... en producción
CULQI_SECRET_KEY=sk_test_...   # sk_live_... en producción
CULQI_WEBHOOK_SECRET=wh_...    # Para validar firma del webhook

# ─────────────────────────────
# SUNAT / NUBEFACT
# ─────────────────────────────
NUBEFACT_API_TOKEN=eyJ...
NUBEFACT_RUC=20XXXXXXXXX         # RUC de Arellan Hnos
NUBEFACT_SERIE_BOLETA=B001
NUBEFACT_SERIE_FACTURA=F001
NUBEFACT_ENVIRONMENT=demo        # demo | produccion

# ─────────────────────────────
# WHATSAPP BUSINESS
# ─────────────────────────────
WHATSAPP_API_TOKEN=EAABsbCS...
WHATSAPP_PHONE_NUMBER_ID=1234567890
WHATSAPP_BUSINESS_ACCOUNT_ID=0987654321

# ─────────────────────────────
# ALMACENAMIENTO S3
# ─────────────────────────────
AWS_S3_BUCKET=arellan-assets
AWS_S3_REGION=us-east-1
AWS_ACCESS_KEY_ID=AKIA...
AWS_SECRET_ACCESS_KEY=SECRET...
S3_SIGNED_URL_EXPIRY=3600  # 1 hora para URLs de fotos de ingreso

# ─────────────────────────────
# ENCRIPTACIÓN DE DATOS SENSIBLES
# ─────────────────────────────
ENCRYPTION_KEY=32_BYTE_AES256_KEY_BASE64  # Para bank_account, teléfonos

# ─────────────────────────────
# APP
# ─────────────────────────────
NODE_ENV=development  # development | staging | production
PORT=3000
APP_URL=https://api.arellan.pe
FRONTEND_URL=https://app.arellan.pe
CORS_ORIGINS=https://app.arellan.pe,https://mecanicos.arellan.pe

# ─────────────────────────────
# GEOFENCING
# ─────────────────────────────
GEOFENCE_LAT=-12.1095    # Latitud del taller (Surquillo)
GEOFENCE_LNG=-77.0282    # Longitud del taller
GEOFENCE_RADIUS_METERS=200

# ─────────────────────────────
# QUEUE (BullMQ)
# ─────────────────────────────
BULL_BOARD_PASSWORD=ADMIN_PASSWORD  # Acceso a /admin/queues
```

## arellan-frontend-web

```env
# ─────────────────────────────
# API
# ─────────────────────────────
NEXT_PUBLIC_API_URL=https://api.arellan.pe
NEXT_PUBLIC_WS_URL=wss://api.arellan.pe

# ─────────────────────────────
# SUPABASE AUTH (MVP)
# ─────────────────────────────
NEXT_PUBLIC_SUPABASE_URL=https://PROJECT_ID.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJ...  # Anon key — segura en frontend

# ─────────────────────────────
# FEATURE FLAGS
# ─────────────────────────────
NEXT_PUBLIC_FEATURE_IMPORTACIONES=false    # Fase 2
NEXT_PUBLIC_FEATURE_GPS_MAP=false          # Fase 4
```

## arellan-mechanic-ui

```env
VITE_API_URL=https://api.arellan.pe
VITE_WS_URL=wss://api.arellan.pe
VITE_SUPABASE_URL=https://PROJECT_ID.supabase.co
VITE_SUPABASE_ANON_KEY=eyJ...
```

## arellan-iot-hardware-bridge

```env
# Backend interno
BACKEND_INTERNAL_URL=https://api.arellan.pe
BACKEND_API_KEY=INTERNAL_SERVICE_KEY  # No es JWT — es service-to-service key

# ADMS endpoint config
ADMS_LISTEN_PORT=8080
ADMS_SECRET_TOKEN=TOKEN_DEL_RELOJ_ZKTECO

# Forzar checkout cron
FORCED_CHECKOUT_TIME=23:59   # Lima timezone
TIMEZONE=America/Lima
```

## arellan-vehicle-tracking

```env
BACKEND_INTERNAL_URL=https://api.arellan.pe
BACKEND_API_KEY=INTERNAL_SERVICE_KEY

# GPS polling
GPS_POLLING_INTERVAL_SECONDS=30
GEOFENCE_LAT=-12.1095
GEOFENCE_LNG=-77.0282
GEOFENCE_RADIUS_METERS=200

# Teltonika API (si se usa gestión remota)
TELTONIKA_API_KEY=KEY
```

## Gestión de Secrets

| Ambiente | Herramienta |
|---------|-------------|
| Development | `.env.local` (no commiteado, en `.gitignore`) |
| Staging | Railway Variables (cifradas en Railway) |
| Production | Railway Variables + AWS Secrets Manager (Fase 5) |

### Reglas

- Nunca commitear valores reales en ningún `.env` o `.env.example`
- `.env.example` solo contiene nombres de variables, no valores
- `SUPABASE_SERVICE_ROLE_KEY` y `CULQI_SECRET_KEY` → jamás en frontend ni en logs
- Rotar claves cada 90 días en producción (mínimo)
- Ver `arellan-platform-governance/standards/security-standards.md` para más
