# Plan de Escalabilidad — Arquitectura Cloud

## Estrategia de Migración por Etapas

### Etapa 1: MVP (Railway + Supabase) — ~$65-120 USD/mes

```
Componente          Servicio              Costo
──────────────────────────────────────────────────
Backend NestJS      Railway ($20)         $20
PostgreSQL 15       Supabase ($25)        $25
Auth + Storage      Supabase (incluido)   $0
Redis               Railway add-on ($10)  $10
Frontends           Vercel ($0)           $0
CDN + DNS           Cloudflare ($0)       $0
Email               Resend ($0-5)         $5
──────────────────────────────────────────────────
Total estimado:                           $60-80
```

**Capacidad:** Hasta ~500 OTs/mes, 20 usuarios concurrentes.

### Etapa 2: Crecimiento (Railway escalado) — ~$120-200 USD/mes

Cuando el tráfico lo justifique (> 200 OTs/mes):
- Railway Pro plan: más RAM + CPU
- Supabase Pro: más storage, backups point-in-time
- Agregar Redis Redis Cloud dedicated

### Etapa 3: Producción AWS — ~$150-300 USD/mes

Migración a AWS cuando el volumen justifique la complejidad operativa adicional:

```
Componente          Servicio AWS          Costo estimado
──────────────────────────────────────────────────────────
Backend NestJS      ECS Fargate (t3.small) $30-50
PostgreSQL 15       RDS Multi-AZ (db.t3)  $50-80
Redis               ElastiCache (t3.small) $25-35
Archivos            S3 + CloudFront        $5-15
Balanceador         ALB                    $20-25
Secrets             Secrets Manager        $1
──────────────────────────────────────────────────────────
Total estimado:                            $131-206
```

**Capacidad:** Hasta 2,000 OTs/mes, auto-scaling, Multi-AZ HA.

## Separación Read/Write (Fase 3+)

Para soportar queries analíticas pesadas sin afectar operaciones del taller:

```
PostgreSQL Primary (Write)
  └── Transacciones: OTs, pagos, inventario
  └── Audit log writes
  └── Máximo: 100 writes/segundo

PostgreSQL Read Replica (Read)
  └── Dashboards y reportes (ETL queries)
  └── Búsquedas históricas
  └── Data pipeline / BI
```

```sql
-- En Prisma, configurar read/write splitting
datasource db {
  provider  = "postgresql"
  url       = env("DATABASE_URL")          // Primary (write)
  directUrl = env("DATABASE_READ_REPLICA") // Read replica
}
```

## Auto-Scaling en AWS ECS

```hcl
# Terraform: auto-scaling para backend NestJS
resource "aws_appautoscaling_policy" "ecs_cpu_policy" {
  name               = "arellan-cpu-autoscaling"
  policy_type        = "TargetTrackingScaling"
  
  target_tracking_scaling_policy_configuration {
    target_value = 70.0  # Scale up si CPU > 70%
    
    ecs_service_namespace {
      metric_type = "ECSServiceAverageCPUUtilization"
    }
  }
}

# Mínimo 2 tasks, máximo 5
resource "aws_appautoscaling_target" "ecs_target" {
  min_capacity = 2
  max_capacity = 5
}
```

## Caché con Redis

```typescript
// Cachear consultas frecuentes para reducir carga en DB
@Injectable()
export class CacheService {
  // Dashboard principal: cachear 5 minutos
  @Cacheable({ key: 'dashboard:summary', ttl: 300 })
  async getDashboardSummary(tenantId: string) { ... }

  // Inventario: cachear 1 minuto (alta frecuencia de lectura)
  @Cacheable({ key: 'inventory:list', ttl: 60 })
  async getInventoryList() { ... }

  // Invalidar cache cuando hay mutaciones
  @CacheInvalidate(['dashboard:summary', 'inventory:list'])
  async createInventoryMovement(data: CreateMovementDto) { ... }
}
```

## Plan de Capacidad (36 meses)

| Mes | OTs/mes estimadas | Usuarios | Infraestructura |
|-----|------------------|---------|----------------|
| 1-6 | 50-100 | 8 | Railway básico |
| 7-12 | 100-200 | 10 | Railway Pro |
| 13-18 | 200-400 | 12 | AWS (migración) |
| 19-36 | 400+ | 20+ | AWS Multi-AZ + Read Replica |
