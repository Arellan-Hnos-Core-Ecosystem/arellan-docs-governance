# SSL and Domains

Dominios por producto y gestión de certificados SSL.

## Dominios del Sistema

| Servicio | Dominio | Provider | SSL |
|---------|---------|----------|-----|
| Backend API | `api.arellan.pe` | Railway | Auto (Let's Encrypt) |
| Frontend Admin | `app.arellan.pe` | Vercel | Auto (Let's Encrypt) |
| Mechanic UI | `mecanicos.arellan.pe` | Vercel | Auto (Let's Encrypt) |
| Portal Clientes | `estado.arellan.pe` | Vercel | Auto (Let's Encrypt) |
| API Staging | `api-staging.arellan.pe` | Railway | Auto (Let's Encrypt) |
| Staging Web | `staging.arellan.pe` | Vercel | Auto (Let's Encrypt) |

## DNS (Cloudflare)

Todos los dominios pasan por Cloudflare para:
- DDoS protection
- CDN para assets estáticos
- Rate limiting de primera capa
- SSL/TLS termination

### Records DNS

```
# API Backend → Railway
api.arellan.pe          CNAME   xxxxx.railway.app    (Proxied)
api-staging.arellan.pe  CNAME   yyyyy.railway.app    (Proxied)

# Frontends → Vercel
app.arellan.pe          CNAME   cname.vercel-dns.com (Proxied)
mecanicos.arellan.pe    CNAME   cname.vercel-dns.com (Proxied)
estado.arellan.pe       CNAME   cname.vercel-dns.com (Proxied)
staging.arellan.pe      CNAME   cname.vercel-dns.com (Proxied)
```

### Configuración Cloudflare

```
SSL Mode: Full (Strict)   ← NO "Flexible" — siempre Full Strict
TLS min version: TLS 1.2
Automatic HTTPS Rewrites: ON
HSTS: enabled (max-age=31536000, includeSubDomains)
```

## SSL en el Backend

### Helmet HSTS (NestJS)

```typescript
// main.ts
import helmet from 'helmet';

app.use(
  helmet({
    hsts: {
      maxAge: 31536000,      // 1 año
      includeSubDomains: true,
      preload: true,
    },
    contentSecurityPolicy: {
      directives: {
        defaultSrc: ["'self'"],
        scriptSrc: ["'self'"],
        styleSrc: ["'self'", "'unsafe-inline'"],  // Tailwind inline styles
        imgSrc: ["'self'", 'data:', 'https://s3.amazonaws.com'],
        connectSrc: ["'self'", 'wss://api.arellan.pe'],
      },
    },
  }),
);
```

### CORS

```typescript
// main.ts
app.enableCors({
  origin: [
    'https://app.arellan.pe',
    'https://mecanicos.arellan.pe',
    'https://estado.arellan.pe',
    // Solo en development:
    process.env.NODE_ENV === 'development' && 'http://localhost:3001',
    process.env.NODE_ENV === 'development' && 'http://localhost:5173',
  ].filter(Boolean),
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE', 'OPTIONS'],
  allowedHeaders: ['Authorization', 'Content-Type', 'X-Request-ID'],
});
```

## SSL para PWA (Mechanic UI)

La PWA en `mecanicos.arellan.pe` requiere HTTPS para:
- Service Worker (Workbox) — solo funciona en HTTPS o localhost
- `getUserMedia()` (cámara) — requiere contexto seguro
- PWA install prompt — requiere HTTPS

Vercel provee SSL automático → sin configuración adicional.

## Renovación de Certificados

| Provider | Renovación | Acción requerida |
|---------|-----------|-----------------|
| Railway (Let's Encrypt) | Automática | Ninguna |
| Vercel (Let's Encrypt) | Automática | Ninguna |
| Cloudflare | Manejado por CF | Ninguna |

**Alerta:** Si un certificado falla → los WebSockets (WSS) se caen → mecánicos pierden actualizaciones en tiempo real. Monitorear con el uptime checker de Cloudflare.

## Migración a AWS (Fase 3+)

En AWS, CloudFront maneja el SSL:

```
CloudFront → ACM Certificate (us-east-1)
           → ALB (HTTP internamente, CloudFront termina HTTPS)
```

```bash
# Solicitar certificado en ACM
aws acm request-certificate \
  --domain-name api.arellan.pe \
  --validation-method DNS \
  --region us-east-1

# Agregar registro DNS de validación en Cloudflare
# Luego asignar el certificado ARN al CloudFront distribution
```

## WebSocket con SSL

```javascript
// Frontend: siempre WSS en producción
const socket = io('wss://api.arellan.pe', {
  auth: { token: accessToken },
  transports: ['websocket'],  // Sin polling — más eficiente
  secure: true,
});

// Development: WS sin SSL
const socket = io('ws://localhost:3000', {
  auth: { token: accessToken },
  transports: ['websocket'],
});
```

Railway y Cloudflare manejan el upgrade HTTP→WebSocket automáticamente con SSL.
