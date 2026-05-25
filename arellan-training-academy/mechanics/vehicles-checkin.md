# Registrar Ingreso de Vehículo — Mecánicos

Paso a paso para registrar correctamente la entrada de un vehículo al taller.

## Por Qué es Importante

Las 5 fotos y la firma del cliente son **evidencia legal**. Si el cliente reclama luego que el taller rayó su vehículo, las fotos con timestamp y hash inmutable prueban el estado real al ingreso. Esto protege al taller **y a ti como mecánico**.

## Antes de Empezar

- La OT debe estar ya creada por Ana (aparece en tu tablet como "RECIBIDO")
- El vehículo debe estar en el espacio de recepción
- Necesitas la tablet con cámara funcionando

## Paso 1: Abrir la OT

1. En tu tablet → OT con estado `RECIBIDO`
2. Clic en **Iniciar Ingreso de Vehículo**

## Paso 2: Verificar Datos del Vehículo

El sistema muestra los datos que ingresó Ana:
- Placa
- Modelo del vehículo
- Año
- Kilometraje estimado (lo confirmas ahora con el odómetro real)

Corrije el kilometraje si es diferente al registrado.

## Paso 3: Registrar Combustible

Indica el nivel de combustible actual:
- VACÍO / CUARTO / MITAD / TRES CUARTOS / LLENO

Hazlo honestamente — si el cliente reclama que entró con el tanque lleno y salió medio, esto es tu respaldo.

## Paso 4: Las 5 Fotos Obligatorias

**El sistema bloquea el registro sin las 5 fotos.** No hay excepción.

| # | Ángulo | Qué capturar |
|---|--------|-------------|
| 1 | Frontal | Parachoque, faros, capó |
| 2 | Posterior | Parachoque trasero, luces, placa |
| 3 | Lateral Izquierdo | Puerta conductor, molduras |
| 4 | Lateral Derecho | Puerta copiloto, molduras |
| 5 | Tablero | Odómetro visible, indicadores de luz |

### Cómo Tomar Buenas Fotos

- **Buena luz** — si hay sombra, usa el flash de la tablet
- **Todo el ángulo en la foto** — no solo el centro del auto
- **Si hay daño preexistente** (rayón, abolladura) — acércate a esa zona y toma una foto adicional (puedes agregar más de 5)
- **La foto del tablero** debe mostrar claramente el número del odómetro

## Paso 5: Daños Preexistentes

Si el vehículo tiene daños que no causó el taller:
1. Toca **Agregar Daño Preexistente**
2. Describe: "Rayón en puerta trasera derecha, largo aprox. 15cm"
3. Toma foto de cerca del daño
4. Esto queda registrado y protege al taller de reclamaciones falsas

## Paso 6: Firma Digital del Cliente

1. El sistema muestra pantalla de firma al cliente
2. El cliente firma con el dedo directamente en la tablet
3. La firma confirma que el cliente vio el registro de condición del vehículo
4. Si el cliente no quiere firmar → registra "Cliente no quiso firmar" en Notas y procede

## Paso 7: Confirmar Ingreso

1. Revisa el resumen: placa, fotos (miniaturas), combustible, daños
2. Clic **Confirmar Ingreso**
3. La OT cambia a estado `EN_PROCESO` automáticamente
4. El sistema activa el geofencing del vehículo (si aplica)

## Errores Comunes

| Error | Solución |
|-------|---------|
| "Cámara no disponible" | Dar permiso de cámara a Chrome en ajustes del tablet |
| "Foto rechazada — muy oscura" | Encender más luz o usar flash |
| "Debes subir 5 fotos" | Tomar las fotos que faltan — el sistema indica cuáles |
| Foto subida pero tarda mucho | Verificar que hay WiFi — en offline, se sube cuando haya conexión |

## En Modo Offline

Si el WiFi está caído:
- Las fotos se guardan temporalmente en la tablet (IndexedDB)
- El sistema muestra "Guardado offline — se subirá cuando haya conexión"
- Las fotos se sincronizan automáticamente cuando regresa el internet
- Puedes continuar trabajando normalmente

**Importante:** No reinicies la tablet mientras hay fotos pendientes de sincronizar en offline.
