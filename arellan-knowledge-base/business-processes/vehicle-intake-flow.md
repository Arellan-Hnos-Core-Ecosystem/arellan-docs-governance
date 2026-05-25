# Protocolo Digital de Recepción de Vehículos y Custodia de Activos

## Contexto

Los vehículos de clientes son activos bajo custodia del taller una vez que ingresan. El sistema provee evidencia inmutable del estado del vehículo al ingreso y durante toda su permanencia en el taller, protegiendo tanto al cliente como a Arellan Hnos ante cualquier reclamo.

## 1. Fase de Recepción (Tablet del Taller)

Al ingresar un vehículo, el operario abre el módulo de recepción en la tablet (`arellan-mechanic-ui`). El proceso físico-digital es:

### Paso 1: Captura de Placa

El sistema procesa la imagen de la placa con OCR (Tesseract) para registrar la placa sin errores de digitación manual. El mecánico confirma la lectura.

```
Formato de placas peruanas soportadas:
  Antigua: ABC-123 (3 letras + 3 dígitos)
  Nueva: A1B-234 (alfanumérico)
  Diplomático: CD-XXX-XX
```

### Paso 2: Datos del Vehículo y Cliente

- Modelo y año del vehículo
- Kilometraje exacto del odómetro
- Nivel del tanque de combustible (1/4, 1/2, 3/4, lleno)
- Nombre del cliente (para la OT — no visible para mecánicos después)
- Teléfono del cliente (solo para Admin/Finance — mecánicos no lo ven)
- Descripción del problema reportado por el cliente

### Paso 3: Evidencia Fotográfica Inmutable (5 fotos obligatorias)

El sistema **obliga** a tomar exactamente estas 5 fotos:
1. Frontal del vehículo
2. Posterior del vehículo
3. Lateral izquierdo
4. Lateral derecho
5. Tablero de instrumentos (km + combustible visibles)

Cada foto se sube inmediatamente a AWS S3 y se genera un **hash SHA-256** que queda en la base de datos. Si alguien intenta reemplazar la foto después, el hash no coincidirá — evidencia de manipulación.

```typescript
// El sistema calcula y almacena:
const photoHash = createHash('sha256')
  .update(photoBuffer)
  .digest('hex')

await prisma.vehicleIntakePhoto.create({
  data: {
    workOrderId: order.id,
    s3Key: `vehicles/${order.id}/${angle}.jpg`,
    sha256Hash: photoHash,  // Inmutable en audit_log
    angle: 'FRONT' | 'REAR' | 'LEFT' | 'RIGHT' | 'DASHBOARD',
  }
})
```

### Paso 4: Firma de Conformidad del Cliente

El cliente lee los términos de internamiento en la pantalla y firma digitalmente sobre la pantalla táctil de la tablet. La firma queda almacenada como imagen PNG con hash SHA-256.

**Los términos incluyen:**
- Tiempo estimado de entrega (si se puede estimar)
- Política de almacenaje (> 30 días sin retirar → cargos)
- Acuerdo de presupuesto previo a ejecución

## 2. Activación del Perímetro de Seguridad (Geofencing GPS)

Una vez guardada la orden de ingreso, el sistema vincula el vehículo con el taller activo:

### Para vehículos de CLIENTES (vehículos en reparación)
- El geofencing aplica si el vehículo tiene GPS OBD-II instalado opcionalmente
- Si el vehículo sale del perímetro de 200m sin una OT activa de `PRUEBA_DE_RUTA`: alerta inmediata
- **Prueba de ruta:** Un owner activa el estado `PRUEBA_DE_RUTA` desde la app móvil, asignando mecánico responsable y tiempo límite (máximo 45 minutos)

### Para vehículos PROPIOS del taller
- Geofencing 24/7 activo (ver `arellan-vehicle-tracking/arellan_vehicle_tracking.md`)
- Cualquier movimiento sin autorización → alerta P1 instantánea

### Regla Anti-Joyride

Si un vehículo (como el que Ricardo usaba para fines personales) sale del perímetro sin autorización:
1. El sistema cambia el estado del activo a `ALERTA_DE_FUGAS`
2. Bloquea el acceso del mecánico responsable a la app del taller
3. Envía push de alta prioridad a Edgar y Juan con ubicación GPS en tiempo real
4. La app gerencial muestra el mapa con la posición del vehículo en tiempo real

## 3. Durante la Permanencia en el Taller

Cualquier movimiento del vehículo dentro del taller que requiera sacarlo del espacio asignado debe registrarse:
- `MOVIMIENTO_INTERNO`: mover de bahía (sin salir del taller)
- `PRUEBA_DE_RUTA`: salida autorizada por owner (< 45 min)
- `TRASLADO_EXTERNO`: a otro taller o servicio (raro — requiere autorización owner + cliente)

## 4. Entrega del Vehículo

Al momento de la entrega:

1. **Fotos de entrega obligatorias** (mismos ángulos que al ingreso)
2. **Comparación visual** del estado de entrega vs ingreso
3. **Registro del km** al momento de la entrega
4. **Cobro del servicio** (QR dinámico del sistema — nunca Yape personal)
5. **Cambio de estado** a `ENTREGADO` — inmutable en audit_log

### Evidencia Legal

Si un cliente reclama daños en el vehículo:
- El sistema tiene 5 fotos del ingreso con hash SHA-256 (inmutables)
- El sistema tiene 5 fotos de la entrega con hash SHA-256
- El sistema tiene el registro de todos los movimientos autorizados
- Todo queda en audit_log con timestamps exactos

**Recomendación legal:** Contratar seguro de Responsabilidad Civil para talleres automotrices (póliza RC Profesional) como capa adicional de protección.
