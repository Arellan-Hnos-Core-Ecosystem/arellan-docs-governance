# Guía de Primeros Pasos — Administrador

Para Ana García (y cualquier usuario con rol ADMIN). Esta guía te lleva desde el primer login hasta tener el sistema funcionando.

## Paso 1: Primer Login

1. Abre el navegador (Chrome o Edge recomendado) e ingresa a **https://app.arellan.pe**
2. Ingresa tu email: `ana@arellan.pe`
3. Ingresa la contraseña temporal que te dio el sistema
4. El sistema te pedirá que **cambies la contraseña** inmediatamente — elige una contraseña de al menos 12 caracteres que solo tú sepas

## Paso 2: Configurar Google Authenticator (MFA)

Este paso es obligatorio para poder abrir/cerrar caja y aprobar gastos.

1. Descarga **Google Authenticator** en tu celular (App Store o Play Store)
2. En el sistema, ve a **Mi Cuenta → Configurar Autenticación de Dos Factores**
3. Aparecerá un código QR en pantalla — ábrelo con Google Authenticator
4. Google Authenticator mostrará un código de 6 dígitos que cambia cada 30 segundos
5. Ingresa ese código en el sistema para confirmar la configuración
6. El sistema te mostrará **10 códigos de respaldo** — guárdalos en un lugar seguro (si pierdes el celular, los necesitas)

**Importante:** Este código nunca se comparte. Si alguien te lo pide, avisa a Edgar.

## Paso 3: Explorar el Dashboard

Al ingresar, verás el **Dashboard Principal** con:

- **OTs Activas:** Órdenes de trabajo en proceso hoy
- **Caja:** Estado actual (Abierta/Cerrada + saldo)
- **Alertas:** Notificaciones pendientes (stock bajo, discrepancias, etc.)
- **Empleados en el taller:** Quién fichó entrada hoy (según ZKTeco)

## Paso 4: Abrir Caja por Primera Vez

Ver [finance/cashbox-daily.md](../finance/cashbox-daily.md) — aunque es guía de Finance, el Admin también puede abrir la caja.

## Paso 5: Crear tu Primera OT

1. Dashboard → **Nueva Orden de Trabajo** (botón azul)
2. Busca o crea el cliente (nombre, DNI, teléfono)
3. Ingresa los datos del vehículo (placa, modelo, año, kilometraje)
4. Selecciona el mecánico asignado
5. El mecánico recibirá la OT en su tablet automáticamente

## Cosas que Puedes Hacer (Rol ADMIN)

| Función | Dónde |
|---------|-------|
| Crear/editar OTs | Módulo Órdenes |
| Asignar mecánicos | Dentro de cada OT |
| Abrir/cerrar caja | Módulo Finanzas → Caja |
| Aprobar gastos ≤ S/.500 | Módulo Finanzas → Gastos |
| Registrar movimientos de inventario | Módulo Inventario |
| Ver reportes | Módulo Reportes |
| Ver empleados y asistencia | Módulo Empleados |
| Registrar incidencia disciplinaria | Módulo Empleados → Incidencias |

## Cosas que NO puedes hacer (requieren OWNER)

- Crear o desactivar empleados
- Aprobar gastos > S/.500
- Ver historial de tracking GPS de vehículos del taller
- Revocar sesiones de otros usuarios

## Si Algo Falla

1. Revisa la sección de [alertas](alerts-guide.md)
2. Si es un error del sistema: anota el mensaje exacto y avisa al Tech Lead
3. Para problemas urgentes que bloqueen la operación del taller: llama directamente a Edgar

## Flujo Diario Típico de Ana

```
7:00 AM  → Login + abrir caja (si Finance no llegó)
7:00-8:00 → Recibir primeros vehículos con mecánicos
8:00-5:00 → Gestionar OTs, aprobar gastos pequeños, coordinar inventario
5:30 PM  → Generar QRs de pago para vehículos listos
6:00 PM  → Apoyar cierre de caja si Finance lo necesita
```
