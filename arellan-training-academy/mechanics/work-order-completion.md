# Completar Orden de Trabajo — Mecánicos

Cómo marcar tu trabajo como terminado y dejar la OT lista para entrega.

## El Flujo Completo

```
RECIBIDO → EN_PROCESO → [EN_ESPERA_REPUESTO →] LISTO → ENTREGADO
                ↑ tú aquí                              ↑ Ana o Finance hacen esto
```

Tu trabajo termina en **LISTO**. La entrega y el cobro los hace Ana o Valeria.

## Cambiar Estado a EN_PROCESO

Cuando empiezas a trabajar en el vehículo:
1. Abre la OT en tu tablet
2. Clic **Iniciar Trabajo**
3. Estado cambia a `EN_PROCESO`
4. El sistema registra la hora de inicio

Hazlo **cuando realmente empiezas**, no cuando recibes el vehículo. Esto afecta el tiempo de trabajo registrado.

## Registrar Avance (opcional pero recomendado)

Puedes agregar notas de avance durante el trabajo:
1. OT abierta → sección **Notas del Mecánico**
2. Escribe el avance: "Cambié pastillas delanteras. Discos en buen estado. Continuando con frenos traseros."
3. Estas notas las ve Ana y los dueños — úsalas para comunicar problemas o hallazgos

Si encuentras un problema adicional (algo que el cliente no mencionó):
1. Agregar nota detallada
2. Ana contacta al cliente para aprobación antes de continuar
3. **No hagas trabajo extra sin aprobación del cliente** — aunque sea "obvio" que hay que hacerlo

## Marcar como LISTO

Cuando el trabajo está 100% terminado:

1. Abre la OT → clic **Marcar como Listo**
2. El sistema verifica:
   - Todos los repuestos solicitados están confirmados como recibidos
   - Hay al menos una nota de trabajo registrada
3. Si hay repuestos pendientes sin confirmar → el sistema te avisa para que los confirmes
4. Clic **Confirmar — Trabajo Terminado**
5. Estado cambia a `LISTO`
6. **Ana y los dueños reciben notificación automática**

## Qué Pasa Después

Cuando la OT está en `LISTO`:
- Ana o Valeria generan el QR de pago para el cliente
- El cliente paga escaneando el QR
- Sistema confirma el pago automáticamente
- Ana entrega el vehículo y registra la entrega
- OT pasa a `ENTREGADO`

**Tú no manejas el cobro.** El sistema lo hace para garantizar que el pago vaya a la cuenta del taller.

## Fotos de Entrega (si te piden tomarlas)

Algunos talleres documentan el estado del vehículo al salir también. Si Edgar decide activar esto:
1. OT en LISTO → sección Fotos de Entrega
2. Mismos 4 ángulos exteriores
3. Esto demuestra que el vehículo salió en el mismo estado o mejor

Por ahora esto es opcional — pregunta a Ana si se necesita.

## Si Marcaste LISTO por Error

Si cometiste un error y el trabajo no está terminado:
1. Ana puede revertir el estado desde su dashboard
2. Díselo de inmediato — no esperes

No intentes "arreglarlo" tú solo. El cambio de estado queda en el audit log y si lo cambias de vuelta tú, queda raro en el historial.

## Métricas que el Sistema Registra de Tu Trabajo

El sistema calcula automáticamente:
- Tiempo desde EN_PROCESO hasta LISTO
- Número de OTs completadas en el mes
- Repuestos solicitados por OT

Edgar y los dueños ven estas métricas en su dashboard. Esto es para reconocer el buen trabajo, no para penalizarte. Si una OT tardó mucho, hay contexto (esperando repuesto, problema complejo) que también queda registrado.
