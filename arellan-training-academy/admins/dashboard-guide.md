# Guía del Dashboard — Administrador y Dueños

Cómo leer el dashboard ejecutivo del sistema.

## Pantalla Principal

Al ingresar verás cuatro secciones principales:

### 1. Tarjetas de Resumen (parte superior)

```
┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
│  OTs Activas    │ │  Ingresos Hoy   │ │  Caja Actual    │ │  Alertas        │
│      12         │ │   S/. 2,450     │ │   S/. 850       │ │     3           │
│  (en proceso)   │ │                 │ │   ABIERTA       │ │  (pendientes)   │
└─────────────────┘ └─────────────────┘ └─────────────────┘ └─────────────────┘
```

| Tarjeta | Qué significa | Acción si algo parece mal |
|---------|---------------|--------------------------|
| OTs Activas | Órdenes en proceso ahora | Si hay OTs viejas (>2 días), revisar con mecánico |
| Ingresos Hoy | Total cobrado (digital + efectivo) | Comparar con lo esperado |
| Caja Actual | Saldo digital calculado | Si está en 0 y hay OTs pagadas, avisa al sistema |
| Alertas | Notificaciones sin resolver | Revisar inmediatamente si hay número rojo |

### 2. Lista de OTs del Día

Muestra todas las órdenes de trabajo activas con su estado:

| Estado | Color | Significado |
|--------|-------|-------------|
| RECIBIDO | Gris | Vehículo ingresó, mecánico aún no empezó |
| EN_PROCESO | Azul | Mecánico trabajando |
| EN_ESPERA_REPUESTO | Amarillo | Falta pieza, OT pausada |
| LISTO | Verde | Trabajo terminado, esperando pago y entrega |
| ENTREGADO | Verde oscuro | Vehículo entregado y pagado |

**Qué hacer si una OT lleva demasiado tiempo en un estado:**
- `EN_PROCESO` por más de 1 día sin actualización → pregunta al mecánico qué pasa
- `EN_ESPERA_REPUESTO` por más de 3 días → revisa si se hizo el pedido de la pieza
- `LISTO` por más de 1 día → el cliente no ha recogido → llamar al cliente

### 3. Panel de Empleados

Muestra quién está en el taller ahora según el reloj biométrico ZKTeco.

- **Verde = En el taller** (registró entrada, no ha registrado salida)
- **Gris = Fuera** (no ha registrado entrada hoy)
- **Naranja = Llegada tarde** (entró después de las 7:15 AM)

Si un mecánico aparece como "fuera" pero está físicamente en el taller → puede haber un problema con el lector biométrico. Ver [troubleshooting/biometric-issues.md](../../arellan-knowledge-base/troubleshooting/biometric-issues.md).

### 4. Panel de Alertas

Las alertas aparecen aquí y también como notificaciones en el celular de los dueños.

#### Tipos de Alerta

| Ícono | Tipo | Prioridad | Qué hacer |
|-------|------|-----------|-----------|
| 🔴 | Joyride detectado | CRÍTICA | Ver sección joyride abajo |
| 🟠 | Discrepancia de caja | ALTA | Ver [cierre de caja](../finance/cashbox-daily.md) |
| 🟡 | Gasto pendiente aprobación | MEDIA | Ir a Gastos y revisar |
| 🟡 | Stock bajo | MEDIA | Ver módulo Inventario |
| ⚪ | Presencia fuera de horario | BAJA | Solo visible para Owners |

#### Alerta de Joyride (vehículo fuera del taller)

Si ves esta alerta:
1. **No entres en pánico** — puede ser una salida autorizada
2. Ve a **Módulo Vehículos → Vehículos del Taller**
3. Verifica si hay una autorización activa (`EN_USO_AUTORIZADO`)
4. Si NO hay autorización → llama inmediatamente a Edgar o Juan
5. El sistema registra la ubicación GPS — se puede ver en el mapa

## Vista de Dueños (Edgar y Juan)

Los Owners ven el mismo dashboard pero con columnas adicionales:

- **Ingresos del mes** comparado con el mes anterior
- **Gastos del mes** con desglose por categoría
- **Mecánico más productivo** (OTs completadas)
- **Anomalías detectadas** (pagos fuera del sistema, si BI está activo)

## Filtros Útiles

En la lista de OTs, puedes filtrar por:
- **Por mecánico:** Ver solo las OTs de Carlos
- **Por estado:** Ver solo las OTs LISTAS pendientes de pago
- **Por fecha:** Ver OTs de la semana pasada
- **Por placa:** Buscar una OT específica

## Actualización en Tiempo Real

El dashboard se actualiza automáticamente via WebSocket. No necesitas recargar la página. Si ves que el contador de OTs no cambia aunque estés creando nuevas → recarga la página una vez para reconectar.
