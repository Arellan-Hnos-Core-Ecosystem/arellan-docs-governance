# Base de Conocimiento — Clínica Automotriz Arellan Hnos

Documentación de los procesos de negocio reales del taller, directorio de proveedores y guías de resolución de problemas. Esta sección es la memoria operativa del negocio — lo que el equipo necesita saber para operar y mantener el sistema.

## Contenido

### Procesos de Negocio

| Documento | Descripción |
|-----------|-------------|
| `business-processes/cash-management-flow.md` | Control de caja, flujo de cobros digitales, manejo de diferencias |
| `business-processes/expense-authorization.md` | Flujo de autorización de gastos por nivel de monto |
| `business-processes/vehicle-intake-flow.md` | Recepción de vehículos, evidencia fotográfica, geofencing |
| `business-processes/import-processes.md` | Importación de repuestos con control de márgenes y comisiones |
| `business-processes/inventory-replenishment.md` | Proceso de reposición de inventario |

### Proveedores

| Documento | Descripción |
|-----------|-------------|
| `suppliers/supplier-directory.md` | Directorio de proveedores activos con datos de contacto |
| `suppliers/import-contacts.md` | Contactos específicos para importaciones internacionales |
| `suppliers/pricing-notes.md` | Notas sobre precios de referencia y márgenes aceptados |

### Troubleshooting

| Documento | Descripción |
|-----------|-------------|
| `troubleshooting/biometric-issues.md` | Problemas con el lector ZKTeco y soluciones |
| `troubleshooting/common-errors.md` | Errores frecuentes del sistema y cómo resolverlos |
| `troubleshooting/database-issues.md` | Problemas de conexión y migraciones fallidas |
| `troubleshooting/deployment-issues.md` | Errores de deploy en Railway/Vercel y soluciones |

## Contexto del Negocio

**Ubicación:** Surquillo, Lima, Perú
**Horario:** Lunes a Sábado, 7 AM a 6 PM
**Personal operativo:** Mecánicos (3-5), Practicantes (1-2), Administradora (Ana), Finance (hija de Edgar)
**Owners:** Edgar y Juan (hermanos, supervisan remotamente desde app móvil)

## Reglas de Negocio No Negociables

1. **Ningún pago se hace con QR personal** — siempre desde el sistema
2. **Todo gasto > S/.500 requiere aprobación push del owner** — sin excepción
3. **Las OTs no se cierran sin pago registrado** en el sistema
4. **El mecánico nunca ve datos del cliente** (solo placa y modelo)
5. **Al desvincular a un empleado: desactivar cuenta el mismo día**
# Manual de Conocimiento Operativo de la Clínica Automotriz

Este repositorio almacena las descripciones funcionales de los procesos comerciales del taller de Surquillo. Sirve como fuente de verdad para mapear los requisitos lógicos antes de codificar reglas dentro del ERP.