# Protocolo Digital de Recepción de Vehículos y Custodia de Activos (Anti-Joyride)

## 1. Fase de Recepción en Zona de Trabajo (Surquillo)
Al ingresar un vehículo al taller, el operario o el practicante está obligado a abrir el módulo de recepción en la tablet del taller (`arellan-mechanic-ui`). El proceso físico-digital consta de los siguientes pasos obligatorios:

1. **Captura Automatizada de Placa:** Mediante la cámara de la tablet, el sistema procesa la imagen con un algoritmo OCR local para registrar la placa sin errores de digitación manual.
2. **Registro de Kilometraje y Combustible:** Se ingresa el kilometraje exacto del odómetro y el nivel del tanque de combustible.
3. **Evidencia Fotográfica Inmutable (4 Ángulos):** El sistema obliga a tomar 4 fotografías panorámicas del auto (Frontal, Posterior, Lateral Izquierdo, Lateral Derecho) y 1 fotografía del tablero de instrumentos. Estas imágenes se suben de inmediato a un bucket privado de AWS S3 y se les genera un hash criptográfico (SHA-256) en la base de datos para evitar alteraciones posteriores.
4. **Firma de Conformidad del Cliente:** El cliente lee los términos de internamiento y firma digitalmente sobre la pantalla táctil de la tablet.

## 2. Activación del Perímetro de Seguridad (Geofencing GPS)
Una vez guardada la orden de ingreso, el sistema vincula automáticamente el vehículo con el identificador del dispositivo OBD-II / GPS de cortesía que el mecánico conecta al puerto del auto.

- **Regla de Monitoreo:** El servicio en segundo plano `arellan-vehicle-tracking` recibe pings de geolocalización cada 30 segundos de las unidades activas en taller.
- **Límites Geográficos:** El sistema establece un radio de geovalla (Geofence) de exactamente 200 metros a la redonda del local de Surquillo.
- **Excepción de Prueba de Ruta (Road Test):** Un auto solo puede salir del perímetro si un hermano (Edgar/Juan) activa el estado `PRUEBA_DE_RUTA` desde su aplicativo móvil, asignando un mecánico responsable y un tiempo límite (máximo 45 minutos).
- **Activación de Alarma Silenciosa:** Si un vehículo (como el carro asignado provisionalmente a Ricardo para el colegio de sus hijos) sale del perímetro fuera del horario escolar autorizado o sin el estado de prueba de ruta activo:
    1. El sistema cambia el estado del activo a `ALERTA_DE_FUGAS`.
    2. Bloquea el acceso del mecánico responsable a la aplicación del taller.
    3. Envía alertas sonoras continuas a los teléfonos de los dueños indicando la ubicación en tiempo real mediante mapas en la aplicación gerencial.