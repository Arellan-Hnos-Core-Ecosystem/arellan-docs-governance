# Employees Endpoints

## GET /employees

Listar empleados activos.

**Roles:** OWNER, ADMIN

**Response 200:**
```json
{
  "data": [
    {
      "id": "uuid",
      "fullName": "Carlos Quispe",
      "email": "carlos@arellan.pe",
      "role": "MECHANIC",
      "status": "ACTIVE",
      "lastCheckInAt": "2024-01-15T07:05:00.000Z",
      "isCurrentlyAtWork": true
    }
  ]
}
```

---

## POST /employees

Crear nuevo empleado (cuenta en el sistema).

**Roles:** OWNER

**Request:**
```json
{
  "fullName": "Carlos Quispe",
  "email": "carlos@arellan.pe",
  "role": "MECHANIC",
  "phone": "999888777",
  "zktecoBiometricId": "E004"   // ID asignado en el reloj ZKTeco
}
```

**Response 201:**
```json
{
  "id": "uuid",
  "email": "carlos@arellan.pe",
  "temporaryPassword": "TempPass123!",   // Debe cambiarla en primer login
  "status": "ACTIVE"
}
```

---

## PATCH /employees/:id/status

Activar o desactivar un empleado.

**Roles:** OWNER

**Request:**
```json
{
  "status": "TERMINATED",
  "reason": "Fin de contrato"
}
```

**Response 200:**
```json
{
  "id": "uuid",
  "status": "TERMINATED",
  "terminatedAt": "2024-01-15T18:00:00.000Z",
  "sessionsRevoked": 2,         // Sesiones activas revocadas automáticamente
  "auditLogEntry": "uuid"       // ID del registro en audit_log
}
```

**Nota importante:** La desactivación revoca automáticamente todas las sesiones activas del empleado. Este endpoint debe ejecutarse el mismo día que termina la relación laboral.

---

## GET /employees/:id/attendance

Obtener historial de asistencia de un empleado.

**Roles:** OWNER, ADMIN

**Query params:** `?from=2024-01-01&to=2024-01-31`

**Response 200:**
```json
{
  "employeeId": "uuid",
  "employeeName": "Carlos Quispe",
  "period": { "from": "2024-01-01", "to": "2024-01-31" },
  "summary": {
    "totalDaysWorked": 22,
    "totalHoursWorked": 176,
    "lateArrivals": 2,
    "forcedCheckouts": 1
  },
  "records": [
    {
      "date": "2024-01-15",
      "checkIn": "07:05",
      "checkOut": "18:10",
      "hoursWorked": 11.08,
      "verifyMethod": "HUELLA",
      "forcedCheckout": false
    }
  ]
}
```

---

## POST /employees/:id/incidents

Registrar una incidencia disciplinaria.

**Roles:** OWNER, ADMIN

**Request:**
```json
{
  "type": "LATE_ARRIVAL",   // LATE_ARRIVAL | UNAUTHORIZED_ACTION | POLICY_VIOLATION | OTHER
  "description": "Llegó 45 minutos tarde sin justificación",
  "date": "2024-01-15",
  "evidence": "https://s3.amazonaws.com/..."   // URL de evidencia adjunta (opcional)
}
```

**Response 201:**
```json
{
  "id": "uuid",
  "employeeId": "uuid",
  "type": "LATE_ARRIVAL",
  "registeredBy": "Ana García",
  "createdAt": "2024-01-15T08:00:00.000Z",
  "auditLogEntry": "uuid"   // Referencia en audit_log inmutable
}
```

**Nota legal:** Las incidencias registradas aquí constituyen evidencia técnica para procesos laborales bajo la Ley 728. Sin embargo, siempre debe complementarse con asesoría legal externa antes de tomar decisiones de desvinculación.
