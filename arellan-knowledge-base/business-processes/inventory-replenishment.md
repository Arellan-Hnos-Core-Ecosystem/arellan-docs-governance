# Proceso de Reposición de Inventario

## Cuándo se Repone el Inventario

El sistema tiene dos mecanismos de reposición:

### 1. Automático (Sistema → Alerta)

Cuando el stock de un ítem cae al nivel mínimo configurado, el sistema genera automáticamente:
- Notificación push a ADMIN
- Tarea de compra en el sistema con el ítem y cantidad sugerida
- Si es ítem crítico (aceite, filtros, bujías): alerta a OWNER también

```
Ítems críticos con reposición automática:
- Aceite motor 10W-40 → mínimo 10 litros
- Filtro de aceite genérico → mínimo 5 unidades
- Pastillas de freno delanteras (genéricas) → mínimo 3 pares
- Bujías NGK estándar → mínimo 8 unidades
- Líquido de frenos DOT4 → mínimo 3 litros
- Refrigerante 50/50 → mínimo 5 litros
```

### 2. Manual (Mecánico → Solicitud)

El mecánico, desde la tablet, puede solicitar un repuesto específico para una OT:
1. Busca el ítem en el inventario del sistema
2. Si hay stock: el sistema lo "reserva" para la OT
3. Si no hay stock: crea solicitud de compra automáticamente
4. La solicitud llega a Admin para gestionar la compra

## Proceso de Compra Nacional

Para repuestos que se consiguen en Lima:

```
Mecánico identifica necesidad
    │
    ▼
Solicitud de compra en sistema
    │ → Ítem, cantidad, urgencia
    ▼
Admin busca proveedor en directorio
    │ → Comparar 2-3 cotizaciones si monto > S/.200
    ▼
Autorización por nivel de monto
    │ → ≤ S/.100: Finance
    │ → S/.101-500: Admin
    │ → > S/.500: Owner
    ▼
Compra efectuada + registro de comprobante
    │
    ▼
Ingreso al inventario (Admin confirma recepción)
    │ → Precio actualiza el promedio ponderado
    │ → Stock aumenta
    ▼
Ítem disponible para la OT
```

## Código QR/Barras por Ítem (Fase 2)

Cada ítem físico del inventario tendrá un código QR adherido:
- Para verificar stock: escanear con tablet del taller
- Para registrar salida: escanear al asignar a una OT
- Para registrar entrada: escanear al recibir mercadería

Esto elimina el error humano en el registro de movimientos.

## Stock Mínimo por Categoría

| Categoría | Stock mínimo | Reposición sugerida |
|-----------|-------------|-------------------|
| Aceites y lubricantes | 5 unidades | Semanal |
| Filtros | 3 unidades por tipo | Quincenal |
| Pastillas y discos | 2 juegos por tipo | Mensual |
| Bujías | 8 unidades | Mensual |
| Correas y mangueras comunes | 2 unidades | Mensual |
| Repuestos importados | 1 unidad | Según OT siguiente |

## Toma de Inventario Físico

**Mensual:** Admin verifica al azar 20% del inventario y confirma que el stock físico coincide con el sistema.

**Trimestral:** Inventario completo — cada ítem físico verificado contra el sistema. Diferencias quedan en audit_log.

**Si hay diferencia sin explicación:** Alert a OWNER. Posible hurto de repuestos — se cruza con el audit_log de accesos al almacén.
