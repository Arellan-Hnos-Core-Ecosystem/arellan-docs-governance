# Errores Frecuentes y Soluciones

Errores que el equipo (mecánicos, Ana, hija de Edgar) puede encontrar en el uso diario del sistema, y cómo resolverlos sin necesidad de llamar al equipo técnico.

## Errores del Sistema para Usuarios

### Error: "Sesión expirada, vuelve a iniciar sesión"

**Causa:** El token de sesión venció. Para Admin/Finance/Owner, la sesión dura 1 hora. Para mecánicos, 12 horas.

**Solución:** Cerrar el navegador/app y volver a iniciar sesión. Si tienes MFA, tendrás que ingresar el código de Google Authenticator nuevamente.

**Si se repite constantemente:** Verificar que la hora del dispositivo esté sincronizada correctamente con internet.

### Error: "Este módulo requiere verificación MFA activa"

**Causa:** Intentas acceder al módulo de finanzas o administración y tu sesión MFA expiró.

**Solución:** Hacer logout y login completo, incluido el código MFA de Google Authenticator. Esto siempre resuelve el problema.

**Si no tienes el teléfono con Google Authenticator:** Contactar a Edgar o Juan para que generen un código de recuperación temporal.

### Error: "No tienes permisos para realizar esta acción" (403)

**Causa:** Tu rol no tiene acceso a esa funcionalidad.

**Solución:** Verificar con Ana o Edgar qué rol tienes asignado. Si necesitas acceso adicional, Edgar o Juan deben autorizarlo explícitamente.

### Error: "El pago no pudo procesarse" en el QR de cobro

**Causas posibles:**
1. El QR expiró (dura solo 7 minutos) → Regenerar el QR
2. El cliente no tiene saldo suficiente → El cliente debe verificar su app Yape/Plin
3. Fallo temporal de la pasarela de pagos → Esperar 2 minutos y reintentar

**Si ninguna solución funciona:** Usar el fallback manual: el cliente hace la transferencia a la cuenta bancaria del taller y Ana confirma manualmente el pago con el número de operación bancaria.

### Error: "El stock es insuficiente para esta OT"

**Causa:** No hay suficientes unidades del repuesto en inventario para completar la OT.

**Solución:**
1. Verificar si hay en almacén pero no registrado en sistema (desajuste físico vs digital)
2. Si no hay físicamente: crear solicitud de compra de emergencia desde la tablet
3. Coordinar con Ana para compra urgente al proveedor local

### Error: "La orden de trabajo ya fue cerrada"

**Causa:** La OT pasó a estado `ENTREGADO` y ya no se puede modificar.

**Solución:** Si hay una corrección legítima, solo ADMIN u OWNER puede reabrirla desde el panel de administración. Todos los cambios quedan en audit_log.

## Errores de la Tablet del Taller

### La Tablet No Tiene Conexión a Internet

**Síntomas:** El sistema muestra un banner amarillo "Modo Offline Activo".

**Lo que puedes hacer:**
- Seguir trabajando normalmente — las acciones se guardan localmente
- El sistema sincroniza automáticamente cuando vuelve internet
- Capacidad offline: hasta 8 horas de trabajo

**Lo que NO puedes hacer offline:**
- Cobrar servicios (el QR necesita internet)
- Ver notificaciones en tiempo real
- Sincronizar fotos de vehículos (se suben al reconectar)

**Solución de red:** Verificar que el router del taller esté funcionando. Si la internet del taller está caída, llamar a Claro/Movistar.

### La Cámara de la Tablet No Abre para Tomar Fotos

**Solución:** Ir a Configuración del teléfono → Aplicaciones → Chrome (o Safari) → Permisos → Cámara → Permitir. Recargar la página web.

### Las Fotos del Vehículo No Se Están Subiendo

**Síntomas:** Las fotos aparecen en la pantalla pero con ícono de "subiendo...".

**Causa probable:** Problema de internet.

**Solución:** Las fotos se suben automáticamente cuando hay conexión estable. No cerrar la página mientras suben. Si se cerró la página, las fotos se perdieron y deben tomarse nuevamente.

## Errores Técnicos (Para el Equipo Técnico)

### Error: "Cannot connect to database"

Ver `database-issues.md` para el proceso de diagnóstico.

### Error: Deploy fallido en Railway

Ver `deployment-issues.md` para el proceso de rollback.

### Error: Webhook de Culqi no llegando

```bash
# Verificar logs del API Gateway
railway logs --service api-gateway | grep "culqi"

# Verificar que el endpoint está activo
curl https://api.arellan.pe/health

# Verificar Culqi dashboard: últimos webhooks enviados y status
# https://dashboard.culqi.com → Webhooks → Logs
```
