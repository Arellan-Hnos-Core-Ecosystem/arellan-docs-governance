# Guía de Aprobación de Gastos — Administrador

Cómo aprobar o rechazar solicitudes de gasto desde el sistema.

## ¿Quién Aprueba Qué?

| Monto del Gasto | Quién Aprueba |
|----------------|---------------|
| Hasta S/.100 | Finance (Valeria) |
| S/.101 — S/.500 | Admin (Ana) ← tú |
| Más de S/.500 | Owner (Edgar o Juan) |
| Más de S/.2,000 | Los dos Owners juntos |

**Regla importante:** Si Valeria registró un gasto de S/.80, ella puede aprobarlo sola. Si registró uno de S/.300, te llegará a ti para aprobar.

## Cómo Aprobar un Gasto

### Desde el Dashboard

1. En la campana de notificaciones verás un número si hay gastos pendientes
2. Clic en la notificación o ve a **Finanzas → Gastos → Pendientes**

### Revisar el Gasto

Antes de aprobar, verifica:

- [ ] ¿El monto es razonable para lo que describe?
- [ ] ¿Tiene cotización adjunta (PDF o imagen)?
- [ ] ¿El proveedor es conocido o tiene historial?
- [ ] ¿La categoría es correcta (repuesto, servicio, etc.)?
- [ ] ¿No es el mismo usuario que solicitó el gasto? (auto-aprobación está bloqueada)

### Aprobar

1. Clic en **Aprobar**
2. El sistema te pedirá el **código MFA** (6 dígitos del Google Authenticator)
3. Ingrésalo y confirma
4. El gasto pasa a estado `APROBADO` — Valeria puede proceder con el pago

### Rechazar

1. Clic en **Rechazar**
2. Escribe el motivo en el campo de texto (obligatorio)
   - Ejemplo: "Falta cotización del proveedor" o "Monto excesivo para el servicio descrito"
3. Confirma el rechazo
4. Valeria recibirá notificación con tu motivo

## Situaciones Especiales

### El gasto supera tu nivel de aprobación

Si llega un gasto de S/.600 a tus pendientes **por error del sistema**, no lo apruebes. Escala a Edgar y notifica al Tech Lead — el sistema no debería enviarte gastos que superan tu nivel.

### Importaciones con margen sospechoso

Si el gasto es una importación y el sistema muestra una advertencia de margen:
- **"Margen posiblemente excesivo (>35%)"** → revisar con Edgar antes de aprobar
- **"Margen muy bajo (<5%)"** → puede haber comisión oculta → reportar a Edgar

El sistema de importaciones fue diseñado específicamente para detectar el fraude de Ricardo (quien cobraba comisiones no declaradas a los proveedores). Si ves estas advertencias, es el sistema haciendo su trabajo.

### Gasto de emergencia urgente

Si hay una emergencia y el mecánico necesita un repuesto urgente que cuesta S/.200:
1. El mecánico o Valeria registra la solicitud
2. Te llega la notificación en menos de 1 minuto
3. Apruebas con MFA
4. Valeria puede proceder

**No existe** aprobación verbal o "te pago después". Todo debe pasar por el sistema. Esto protege a todos, incluyéndote a ti.

## Historial de Aprobaciones

Para ver todas las aprobaciones que hiciste:
- **Finanzas → Gastos → Historial**
- Filtra por "Aprobado por mí" + rango de fechas

Este historial es inmutable — nadie puede borrarlo, incluyendo el sistema y los dueños. Es evidencia contable y legal.

## En el Celular

Si Edgar instaló la app móvil, también puedes aprobar desde el celular:
1. Abre la app Arellan
2. Ve a **Aprobaciones pendientes**
3. El flujo es el mismo — requiere MFA también desde móvil

Si no tienes la app, puedes abrir `app.arellan.pe` desde el navegador de tu celular.
