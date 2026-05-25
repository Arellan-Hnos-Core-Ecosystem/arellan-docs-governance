# Fase 4 — Mobile Ejecutivo y Portal de Clientes (Meses 10-12)

## Objetivo

Optimizar la experiencia móvil de los owners con capacidades avanzadas de supervisión remota y lanzar el portal de clientes completo con registro, historial y notificaciones push.

## Prerequisitos de Entrada

- Registro banco de datos ante MINJUS completado (Ley 29733)
- WhatsApp Business API contratada y templates aprobados por Meta
- Sistema estable con < 3 bugs críticos/mes
- Al menos 6 meses de datos históricos

## Módulos a Desarrollar

### `arellan-mobile-app` — Features Avanzados

```
Nuevas funcionalidades para owners (Edgar y Juan):
- Mapa GPS en tiempo real de vehículos del taller
- Aprobación de gastos con biometría del teléfono (Face ID / Touch ID)
- Vista de cámaras IP del taller (si se instalan — opcional)
- Dashboard offline (datos cacheados 24h sin internet)
- Historial completo de incidentes de seguridad
- Control de personal: check-in/out biométrico en tiempo real
- Exportación de reportes directamente desde mobile a PDF
- Modo "viaje": acceso completo aunque estés fuera de Lima
```

### Portal Cliente Completo (`arellan-client-portal`)

```
Features con registro:
- Registro con email + verificación OTP
- Historial completo: todos los vehículos, todas las OTs
- Notificaciones push de avance de OT
- Galería de fotos del ingreso y entrega del vehículo
- Descarga de proformas y comprobantes SUNAT
- Solicitud de cita / mantenimiento preventivo
- Sistema de calificación del servicio
- Chat con el taller (Fase 4+)
```

### WhatsApp Business API Completa

Templates aprobados en Meta Business Manager:

| Template | Trigger | Destinatario |
|---------|---------|-------------|
| `ot_recibida` | OT creada | Cliente |
| `ot_en_proceso` | OT → EN_PROCESO | Cliente |
| `ot_lista` | OT → LISTO | Cliente |
| `presupuesto_pendiente` | OT presupuestada | Cliente |
| `recordatorio_mantenimiento` | 90 días post-servicio | Cliente |
| `gasto_aprobacion` | Gasto > S/.500 | Owner |
| `reporte_caja_diario` | Cierre de caja | Owner |
| `alerta_anomalia` | Fraude detectado | Owner (silencioso) |

### Observability Avanzada

```
- Grafana Cloud: dashboards de negocio en tiempo real
- TV en taller: métricas del día (OTs activas, caja, stock crítico)
- SLO tracking: uptime, latencia p95, errores
- Alertas de performance: p95 > 500ms → notificación al equipo técnico
```

## KPIs de Éxito

| KPI | Meta |
|-----|------|
| Clientes registrados en portal | > 50 en primer mes |
| Notificaciones WhatsApp de OT | > 80% de OTs |
| App gerencial uso diario | Ambos owners cada día |
| Tiempo aprobación de gastos | < 10 minutos |
| Valoraciones portal cliente | Promedio > 4 estrellas |
