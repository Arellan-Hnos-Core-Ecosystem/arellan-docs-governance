# Manual de Usuario: Procedimiento Operativo Diario para el Cierre y Cuadre de Caja

## 1. Flujo de Apertura del Día (08:00 AM)
1. Enciende la computadora de la oficina administrativa, abre el navegador web e ingresa al portal `arellan-frontend-web`.
2. Introduce tu correo electrónico institucional corporativo (`administracion.arellan.hnos@gmail.com`) y tu contraseña privada. El sistema solicitará el código numérico de 6 dígitos de tu celular (Google Authenticator). **Nunca compartas este código con ningún trabajador del taller.**
3. Dirígete a la sección **Módulos Financieros -> Control de Caja Chica**.
4. Haz clic en el botón verde **"Apertura del Día"**. El sistema te mostrará el saldo de efectivo con el que cerró la caja la noche anterior. Cuenta físicamente los billetes y monedas en la gaveta metálica e ingresa el monto real. Si coincide, haz clic en **Confirmar Apertura**.

## 2. Monitoreo y Conciliación Automatizada Durante el Día
A lo largo de la jornada laboral en Surquillo, los clientes pagarán escaneando los códigos QR dinámicos generados desde las tablets o mediante la lectora de tarjetas (POS). 
- El sistema sumará de forma invisible estos ingresos directo a las cuentas bancarias corporativas de la clínica. No necesitas usar cuadernos ni hojas de cálculo manuales.
- **Regla Inquebrantable de Egresos:** Si un mecánico viene a solicitar dinero en efectivo para un repuesto urgente, verifica tu pantalla en el módulo **Autorizaciones de Gasto**. Si la solicitud no tiene el estado de **"APROBADO POR DUEÑO"** con el check verde digitalizado de tu papá (Edgar) o tu tío (Juan), **no entregues ni un solo sol de la gaveta.** El software bloqueará el cuadre si falta dinero no justificado digitalmente.

## 3. Procedimiento Obligatorio de Cierre de Caja (06:00 PM)
1. Al terminar el turno de la tarde, haz clic en **"Iniciar Cierre de Caja"**. El sistema congelará la entrada de nuevas órdenes de trabajo en taller.
2. El ERP desplegará la matriz de balance cruzado de tres columnas:
    - **Columna 1:** Total Esperado en Medios Digitales (Pasarelas QR/Webhooks).
    - **Columna 2:** Total Esperado en POS de Tarjetas (Vouchers físicos agregados).
    - **Columna 3:** Total Esperado en Efectivo de Caja Chica.
3. Vacía la gaveta física y cuenta el dinero. Digita el monto exacto en el campo "Efectivo Real en Caja".
4. **Manejo de Desfases o Descuadres (Complejidad Crítica):**
    - Si el sistema detecta un faltante mayor a S/. 5.00, el botón de cierre regular se deshabilitará.
    - El software te obligará a abrir un campo de justificación de discrepancia.
    - Al enviar este informe, el sistema aplicará un cierre con bandera de error (`CLOSED_WITH_DISCREPANCY`) e inyectará de forma automática una notificación de auditoría forense en el repositorio `arellan-logging-centralizado` para que los hermanos revisen las cámaras de seguridad del taller en esa hora exacta.