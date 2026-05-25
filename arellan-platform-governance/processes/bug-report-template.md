# Plantilla para Reporte de Bug

Usar esta plantilla al reportar un bug en GitHub Issues. Completar todos los campos para acelerar el diagnóstico.

---

## Reporte de Bug: [Título descriptivo del problema]

**Fecha:** YYYY-MM-DD
**Reportado por:** [Nombre del reportador]
**Severidad:** P0 (crítico) / P1 (alto) / P2 (medio) / P3 (bajo)
**Módulo afectado:** auth / finance / orders / inventory / vehicles / personnel / audit / frontend-web / mechanic-ui / mobile-app

### Descripción

[Descripción clara del problema en 2-3 oraciones. ¿Qué esperabas que pasara? ¿Qué pasó en su lugar?]

### Pasos para Reproducir

1. [Paso 1]
2. [Paso 2]
3. [Paso 3]
4. Ver error

### Comportamiento Esperado

[Describe qué debería pasar]

### Comportamiento Actual

[Describe qué está pasando]

### Evidencia

```
[Mensaje de error exacto / stack trace / código de error]
```

Capturas de pantalla: [Adjuntar si aplica]

### Contexto del Entorno

- **Entorno:** Producción / Staging / Local
- **Navegador/App:** Chrome XX / Safari XX / App iOS/Android
- **Rol del usuario:** OWNER / ADMIN / FINANCE / MECHANIC
- **URL afectada:** `https://...`

### Impacto en el Negocio

[¿El taller puede seguir operando? ¿Hay un workaround manual? ¿Cuántos usuarios están afectados?]

### Logs Relevantes

```
[Pegar logs de Railway/Sentry/Axiom si están disponibles]
```

---

## Criterios de Severidad

| Severidad | Criterio | Tiempo de respuesta |
|-----------|---------|---------------------|
| **P0** | Sistema caído, fraude activo, pagos no funcionan | < 15 minutos |
| **P1** | Módulo crítico caído (caja, auth), pérdida de datos | < 1 hora |
| **P2** | Funcionalidad degradada pero hay workaround | < 4 horas |
| **P3** | Bug menor, cosmético o de usabilidad | < 24 horas |
