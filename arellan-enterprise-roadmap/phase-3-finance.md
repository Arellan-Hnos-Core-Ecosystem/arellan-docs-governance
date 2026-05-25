# Fase 3 — Finanzas Avanzadas e Inteligencia (Meses 7-9)

## Objetivo

Convertir los datos del sistema en inteligencia accionable para los owners. Implementar reportes financieros profundos, análisis de rentabilidad y las primeras capacidades de automatización inteligente.

## Prerequisitos de Entrada

- Fase 2 estable en producción
- Al menos 3 meses de datos históricos en el sistema
- SUNAT integración operativa
- ZKTeco integrado con datos confiables

## Módulos a Desarrollar

### Dashboard Financiero Avanzado

```
Métricas nuevas:
- P&L mensual detallado (ingresos - costos directos - gastos operativos)
- Margen bruto por tipo de servicio (mecánica, electricidad, llaves, importaciones)
- Comparativa mes a mes con tendencia
- Proyección de ingresos (basado en OTs en proceso)
- Flujo de caja semanal (real vs proyectado)
- Top 10 clientes por facturación
- Análisis de horas mecánico: facturadas vs presencia biométrica
```

### Análisis de Rentabilidad por Servicio

```sql
-- Rentabilidad por tipo de servicio (últimos 3 meses)
SELECT
  service_type,
  COUNT(*) as total_ots,
  AVG(total_amount) as ticket_promedio,
  SUM(total_amount) as facturación_total,
  SUM(parts_cost) as costo_repuestos,
  SUM(total_amount - parts_cost - labor_cost) as utilidad_bruta,
  ROUND((SUM(total_amount - parts_cost - labor_cost) / SUM(total_amount)) * 100, 2) as margen_pct
FROM work_orders
WHERE status = 'ENTREGADO'
  AND delivered_at >= now() - INTERVAL '3 months'
GROUP BY service_type
ORDER BY utilidad_bruta DESC;
```

### Módulo de Conciliación Bancaria

```
Funcionalidad:
- Upload semanal del extracto bancario (CSV/PDF)
- Cruce automático vs transacciones del sistema
- Detección de diferencias > 5%
- Historial de conciliaciones con evidencia
- Alerta si suma sistema ≠ depósitos bancarios reales
```

### Multi-Firma para Gastos de Capital

```
Regla nueva (Fase 3):
- Gastos > S/.2,000: requieren aprobación de AMBOS owners (Edgar Y Juan)
- Push a los dos teléfonos
- Timer de 24h: si uno no responde, escalada automática
- Evidencia de doble aprobación en audit_log
```

### Reportes Automatizados Mensuales

El día 1 de cada mes, enviado por email + WhatsApp a owners:

1. **P&L completo** — ingresos/costos/utilidad vs mes anterior + tendencia 6 meses
2. **Productividad de mecánicos** — OTs, tiempos, horas facturadas vs presencia
3. **Análisis de inventario** — rotación por categoría, valor total, ítems con baja rotación
4. **Reporte de compliance** — MFA adoption, intentos fallidos, alertas activas
5. **Análisis de clientes** — nuevos vs recurrentes, ticket promedio, top clientes por facturación
6. **Estado de deuda** — gastos pendientes de sustentación, comprobantes vencidos

### Módulo de Personal Completo

```
Funcionalidades:
- Historial de incidencias disciplinarias (evidencia Ley 728)
- Evaluaciones de desempeño por mecánico (mensual)
- Control de vacaciones y permisos
- Cruce: horas en ZKTeco vs OTs completadas
- Exportación de evidencia para procesos laborales en PDF firmado
```

## KPIs de Éxito

| KPI | Meta |
|-----|------|
| Margen bruto conocido por servicio | Todos los servicios mapeados |
| Conciliación bancaria vs sistema | Diferencia < 2% mensual |
| Reportes mensuales automatizados | Entregados el día 1 de cada mes |
| Tiempo promedio de aprobación de gastos | < 10 minutos |
| Gastos de capital con doble firma | 100% |
