# Roadmap Estratégico — Visión 2026-2028

## Resumen Ejecutivo

El ecosistema digital Arellan transforma la Clínica Automotriz de operaciones manuales y vulnerables al fraude interno, hacia un sistema digital auditable, seguro y escalable en tres años.

**El problema que resuelve:** Tres vectores de fraude activo que causaban pérdidas mensuales estimadas de S/.2,000-5,000:
1. Cobros desviados a Yape personal de empleados
2. Comisiones del 20-30% no declaradas en importaciones de repuestos
3. Uso no autorizado de vehículos del taller

## Estrategia por Año

| Año | Foco | Objetivo |
|-----|------|---------|
| 2026 | Control y seguridad | Detener las pérdidas activas. MVP en producción |
| 2027 | Inteligencia operativa | Optimizar rentabilidad con datos reales |
| 2028 | Escalabilidad | Portal de clientes masivo + IA + expansión |

## Fases de Desarrollo (2026)

### Fase 0 — Preparación (2 semanas)
- Setup de infraestructura (Railway + Supabase + Vercel)
- Configuración de repositorios y CI/CD
- Design system base (`@arellan/ui`)

### Fase 1 — MVP Core (10-12 semanas)
**Objetivo:** Detener el fraude activo. Sistema en producción en el taller.

Módulos incluidos:
- Auth con MFA obligatorio para roles críticos
- Órdenes de trabajo completas
- Control de caja con QR dinámico (elimina Yape personal)
- Inventario básico
- Autorización de gastos por niveles
- Audit log inmutable
- Notificaciones push a owners

Criterio de éxito: Cero pagos fuera del sistema durante 30 días consecutivos.

### Fase 2 — Consolidación (8-10 semanas)
**Objetivo:** Optimizar operaciones y cerrar riesgos secundarios.

Módulos incluidos:
- Sistema de importaciones con control de márgenes
- Inventario avanzado con QR/código de barras
- Integración ZKTeco biométrica completa
- Portal cliente básico (consulta de estado por placa)
- Reportes financieros mensuales automatizados
- SUNAT integración (comprobantes electrónicos)

### Fase 3 — Escala (10-14 semanas)
**Objetivo:** Crecimiento digital del negocio.

Módulos incluidos:
- Portal cliente completo con login y historial
- Marketing digital y captación de clientes
- IA: predicción de demanda de repuestos
- IA: scoring de riesgo de fraude
- WhatsApp Business API completa
- Dashboard de inteligencia de negocio avanzado

## Roadmap 2027

- **Q1:** Migración a AWS (Railway → EC2/RDS)
- **Q2:** React Native Expo (si PWA no cubre necesidades iOS)
- **Q3:** Módulo de fidelización de clientes + CRM básico
- **Q4:** Análisis predictivo de mantenimiento preventivo

## Roadmap 2028

- **Q1-Q2:** Evaluación de SaaS multi-tenant (otros talleres en Lima)
- **Q3-Q4:** IA generativa para diagnóstico asistido

## Indicadores de Éxito del MVP

| Indicador | Meta |
|-----------|------|
| Diferencias de caja mensuales | < S/.100/mes |
| Cobros fuera del sistema | 0 en 30 días |
| Tiempo de apertura de OT | < 3 minutos |
| Adopción MFA en roles críticos | 100% |
| Uptime del sistema | > 99% en horario laboral |
