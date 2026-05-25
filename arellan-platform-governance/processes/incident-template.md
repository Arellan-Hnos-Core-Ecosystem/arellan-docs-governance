# Plantilla de Post-Mortem de Incidente

Completar esta plantilla dentro de las 48 horas siguientes a cualquier incidente P0 o P1. El post-mortem se guarda en este repositorio como evidencia y fuente de aprendizaje.

---

## Post-Mortem: [Nombre del Incidente]

**Fecha del incidente:** YYYY-MM-DD HH:MM (hora Perú)
**Duración:** X horas Y minutos
**Severidad:** P0 / P1
**Módulos afectados:** [Lista]
**Autor del post-mortem:** [Nombre]
**Revisado por:** [Nombre]

### Resumen Ejecutivo

[2-3 oraciones describiendo qué pasó, cuánto tiempo duró y cuál fue el impacto en el negocio]

### Línea de Tiempo

| Hora | Evento |
|------|--------|
| HH:MM | Primera detección del problema |
| HH:MM | Notificación a owners |
| HH:MM | Equipo técnico comienza diagnóstico |
| HH:MM | Causa raíz identificada |
| HH:MM | Solución implementada |
| HH:MM | Sistema restaurado completamente |
| HH:MM | Notificación de resolución a owners |

### Causa Raíz

[Descripción técnica de qué causó el incidente. Ser específico: no "había un bug" sino "la función X en el módulo Y no manejaba el caso Z cuando el valor era null"]

### Impacto

- **Usuarios afectados:** [Número y roles]
- **Transacciones fallidas:** [Si aplica]
- **Datos en riesgo:** Sí / No (si Sí, describir)
- **Pérdida financiera estimada:** S/. [si aplica]
- **Tiempo fuera de servicio:** HH:MM

### ¿Cómo se Detectó?

- [ ] Alerta automática del sistema (Sentry / Railway)
- [ ] Reporte de un usuario
- [ ] Revisión proactiva del equipo
- [ ] Edgar o Juan notificaron

### Acciones de Remediación

**Acciones tomadas durante el incidente:**
1. [Acción 1 — quién, cuándo]
2. [Acción 2]

**Acciones preventivas (para evitar que vuelva a pasar):**

| Acción | Responsable | Plazo |
|--------|-------------|-------|
| [Acción preventiva 1] | [Nombre] | [Fecha] |
| [Acción preventiva 2] | [Nombre] | [Fecha] |

### Lecciones Aprendidas

**¿Qué salió bien en la respuesta?**
- [Cosa 1]
- [Cosa 2]

**¿Qué mejorar en la respuesta?**
- [Cosa 1]
- [Cosa 2]

**¿El playbook de incidents fue suficiente?** Sí / No
Si No: ¿qué faltó? Actualizar `arellan-incident-response/arellan_incident_response.md`

### Evidencia

- [ ] Export de audit_logs del período del incidente adjunto
- [ ] Logs de Railway/Sentry capturados
- [ ] Screenshots relevantes adjuntos
- [ ] Matriz de riesgos actualizada si aplica (`arellan-risk-management`)

---

*Este documento es de acceso restringido: OWNER + equipo técnico únicamente*
