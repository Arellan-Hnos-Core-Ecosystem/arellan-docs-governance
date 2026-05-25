# Manual de Caja Diaria — Finance

Procedimiento operativo diario para Valeria Arellan (rol FINANCE). Leer completo antes del primer día de operación.

## Por Qué Este Módulo Existe

Antes del sistema, el taller manejaba los pagos de forma mixta: clientes pagaban directamente con QR personal de Yape de Ricardo (un mecánico), y parte del dinero nunca llegaba a la caja del taller. Este módulo garantiza que **todo pago pasa por la cuenta empresarial de Arellan** y queda registrado con timestamp, monto exacto y método de pago.

---

## 1. Apertura de Caja — 8:00 AM

### Pasos

1. Enciende la computadora de la oficina administrativa
2. Abre Chrome e ingresa a **https://app.arellan.pe**
3. Ingresa con tu email `hija@arellan.pe` y contraseña
4. El sistema pide el **código MFA** — abre Google Authenticator en tu celular y escribe los 6 dígitos
5. Ve a **Módulo Finanzas → Control de Caja → Apertura del Día**
6. El sistema muestra el saldo con que cerró ayer
7. **Cuenta físicamente** los billetes y monedas de la gaveta
8. Ingresa el monto real contado en el campo "Monto de Apertura"
9. Si coincide con lo que cerró ayer → clic **Confirmar Apertura**
10. Si hay diferencia → escribe la diferencia en "Notas" y confirma igualmente (esto queda registrado)

**Importante:** Si la caja ya fue abierta hoy por Ana antes de que llegaras → el sistema mostrará error 409 "Caja ya abierta". Eso es correcto. Ve directo al monitoreo.

---

## 2. Monitoreo Durante el Día

Durante el horario del taller (8 AM - 6 PM):

### Pagos QR (principal método)

- Cuando un vehículo está listo para entrega, **tú o Ana** generan el QR desde el sistema
- Ve a la OT → botón **Generar QR de Pago** → seleccionar método (Yape / Plin / Tarjeta)
- El QR aparece en pantalla — el cliente lo escanea con su app
- El sistema confirma el pago automáticamente en 1-2 minutos via webhook de Culqi
- **NO aceptes Yape a número de celular personal** — solo QR generado por el sistema

### Regla de Egresos

Si un mecánico viene a pedir dinero en efectivo para un repuesto:

1. Pídele que muestre la solicitud en el sistema con estado **APROBADO** (check verde)
2. ¿Quién aprobó?
   - Si el gasto es ≤ S/.100 → tú puedes aprobarlo desde tu rol
   - Si es S/.101-500 → necesita aprobación de Ana
   - Si es > S/.500 → necesita aprobación de Edgar o Juan
3. Solo si tiene aprobación digital → entregar el efectivo
4. Registrar el egreso como "DISBURSED" en el sistema

**Nunca entregues dinero sin aprobación digital.** El sistema detectará el faltante al cierre.

---

## 3. Cierre de Caja — 6:00 PM

### Pasos

1. Ve a **Módulo Finanzas → Control de Caja → Cierre del Día**
2. El sistema muestra la "Matriz de Balance":

```
┌──────────────────────────────────────────────────────────────┐
│  BALANCE AL CIERRE                                           │
├────────────────────┬─────────────────┬───────────────────────┤
│ Ingresos Digitales │ Ingresos POS    │ Total Esperado Caja   │
│ (QR Yape/Plin)     │ (Tarjeta)       │ (Efectivo)            │
│    S/. 1,200       │    S/. 500      │    S/. 850            │
├────────────────────┴─────────────────┴───────────────────────┤
│ Monto de Apertura + Ingresos Efectivo - Egresos = S/. 850   │
└──────────────────────────────────────────────────────────────┘
```

3. **Vacía la gaveta y cuenta físicamente** todos los billetes y monedas
4. Ingresa el monto real en "Efectivo Real en Caja"
5. El sistema calcula la diferencia automáticamente

---

## 4. Manejo de Descuadres

### Diferencia ≤ S/.5

Probablemente error de conteo. Vuelve a contar. Si persiste:
- Agrega nota: "Diferencia menor, posible error de redondeo en vuelto"
- El sistema permite cerrar normalmente con nota

### Diferencia entre S/.5 y S/.50

- El botón de cierre normal se **desactiva**
- Debes escribir una justificación en el campo obligatorio
- Ejemplos de justificación válida:
  - "Cliente pagó S/.200, no había vuelto exacto, diferencia de S/.20"
  - "Pago en efectivo sin recibo registrado por error — ya se corrigió"
- Al confirmar: sistema cierra con estado `CLOSED_WITH_DISCREPANCY`
- **Edgar y Juan reciben notificación automática** por WhatsApp

### Diferencia > S/.50

**No cierres la caja todavía.** Llama a Edgar antes de proceder. Una diferencia grande puede indicar:
- Un pago que no se registró en el sistema
- Un egreso no autorizado
- Un error de registro

Edgar puede decidir revisar las cámaras del taller antes de cerrar.

---

## 5. Reporte Automático al Cierre

Al cerrar caja, el sistema envía automáticamente a los teléfonos de Edgar y Juan:

```
📊 Resumen de Caja — 15 enero 2024

Ingresos: S/. 2,450
  • Yape/QR: S/. 1,200
  • Plin: S/. 750
  • Tarjeta POS: S/. 500

Gastos aprobados: S/. 350
Balance neto: S/. 2,100

Estado: ✓ Sin diferencias
```

Si hubo descuadre:
```
Estado: ⚠️ Diferencia detectada: -S/. 30
Justificación registrada: "..."
```

---

## Checklist Diario

**Apertura (8 AM):**
- [ ] Login con MFA completado
- [ ] Caja abierta con monto físico contado
- [ ] Diferencia de apertura registrada (si hubo)

**Durante el día:**
- [ ] Pagos solo via QR del sistema (nunca Yape personal)
- [ ] Egresos solo con aprobación digital
- [ ] OTs listos con QR generado para entrega

**Cierre (6 PM):**
- [ ] Todos los pagos del día registrados
- [ ] Gaveta contada físicamente
- [ ] Caja cerrada con justificación si hay descuadre
- [ ] Reporte enviado a Edgar y Juan (automático)

---

## Errores Comunes y Soluciones

| Error | Causa | Solución |
|-------|-------|---------|
| "Caja ya abierta" al querer abrir | Ana ya la abrió | Normal — ir directo al monitoreo |
| "MFA inválido" | Código ingresado tarde (expiró) | Ingresar el siguiente código (cambia cada 30 seg) |
| "QR expirado" | Cliente tardó más de 7 minutos en escanear | Generar nuevo QR desde la OT |
| "La caja no puede cerrar" | Hay diferencia > S/.5 sin justificación | Escribir justificación en el campo |
| Pago confirmado pero OT no se marca como pagada | Webhook de Culqi demoró | Esperar 2-3 minutos — si persiste, avisar al Tech Lead |
