# Troubleshooting: Lector Biométrico ZKTeco

Guía de resolución de problemas con el reloj biométrico ZKTeco BioTime 8.0 y su integración con el sistema digital Arellan.

## Problemas Comunes

### 1. El Reloj No Envía los Fichajes al Sistema

**Síntomas:** Los empleados fichan en el reloj físico pero no aparecen en el sistema.

**Diagnóstico:**
```bash
# Verificar que el bridge ADMS está recibiendo requests
# Revisar logs de Railway:
railway logs --service iot-hardware-bridge --tail 50

# Buscar en logs:
# ✅ "POST /adms/attendance - 200 OK" → el reloj está enviando
# ❌ Sin ningún log de /adms/attendance → el reloj no está conectado
```

**Soluciones:**

| Causa | Solución |
|-------|---------|
| El taller no tiene internet | Esperar reconexión. Los fichajes se sincronizan automáticamente cuando hay internet |
| URL del servidor mal configurada en el reloj | En el panel ZKTeco: Device → Communication → ADMS → verificar `api.arellan.pe` |
| Puerto bloqueado por el router | Verificar que el router permite tráfico saliente en puerto 443 (HTTPS) |
| SN del dispositivo no está en la whitelist | Verificar `ZKTECO_DEVICE_SN_WHITELIST` en las variables de entorno del bridge |

### 2. El Empleado No Puede Fichar (Huella No Reconocida)

**Síntomas:** El reloj rechaza la huella del empleado.

**Soluciones:**

1. Pedir al empleado que limpie el dedo (grasa del taller causa falsos rechazos)
2. Probar con el mismo dedo en diferentes ángulos
3. Si persiste: registrar la huella nuevamente en el panel admin del ZKTeco
4. Fallback temporal: el empleado puede fichar con PIN de 4 dígitos hasta que se resuelva

**Registro en sistema:** Si el empleado usa PIN en lugar de huella, el sistema lo registra con `VerifyMethod=4` (PIN) en lugar de `VerifyMethod=1` (huella). 3+ veces consecutivas con PIN genera alerta automática a OWNER.

### 3. Fichajes Duplicados

**Síntomas:** El mismo empleado aparece con múltiples check-ins en la misma hora.

**Causa:** El reloj reentreg intentó sincronizar y envió el mismo evento dos veces.

**Solución automática:** El bridge tiene idempotencia por `SN + UserID + Stamp`. Si llega el mismo evento dos veces, el segundo se ignora silenciosamente.

**Si los duplicados ya están en DB:**
```sql
-- Identificar duplicados
SELECT employee_id, DATE_TRUNC('minute', check_in_at), COUNT(*)
FROM attendance_records
GROUP BY employee_id, DATE_TRUNC('minute', check_in_at)
HAVING COUNT(*) > 1;

-- El equipo técnico limpia con supervisión del owner (registrar acción en audit_log)
```

### 4. Empleado Tiene Check-in pero Sin Check-out

**Síntomas:** Al día siguiente, el empleado aparece como "aún en taller".

**Solución automática:** El sistema ejecuta un job automático a las 11:59 PM que cierra todos los check-ins sin check-out del día, marcados como `forced_checkout=true`.

**Si es recurrente:** El empleado no está fichando la salida. El supervisor debe recordarle la obligación de fichar al salir.

### 5. El Reloj No Está Sincronizando la Hora

**Síntomas:** Los timestamps de los fichajes tienen varios minutos de diferencia con la hora real.

**Solución:**
```
En el panel admin del ZKTeco:
Device → Date/Time → Sync with NTP Server
NTP Server: time.cloudflare.com
```

El bridge también sincroniza el timestamp al responder al ZKTeco: la respuesta `GetStamp: 2024-01-15 08:05:32` le dice al reloj cuál es la hora correcta del servidor.

## Contacto de Soporte ZKTeco

Si ninguna solución funciona, contactar al distribuidor local de ZKTeco en Lima para soporte en sitio. El modelo es BioTime 8.0 — tener el número de serie (SN) a mano.
