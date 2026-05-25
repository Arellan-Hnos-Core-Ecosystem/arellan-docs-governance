# Arellan Training Academy

Manuales de usuario en español para el sistema Arellan. Organizados por rol.

## Para Quién es Cada Sección

| Sección | Para | Rol en el sistema |
|---------|------|------------------|
| `admins/` | Ana García | ADMIN |
| `finance/` | Valeria Arellan (hija) | FINANCE |
| `mechanics/` | Carlos, Luis y mecánicos nuevos | MECHANIC |
| `videos/` | Todos | — |

Edgar y Juan (OWNER) pueden leer cualquier sección. Su acceso al sistema es de supervisión: ven todo, pero rara vez operan directamente.

Este repositorio centraliza toda la documentación estratégica, técnica y operativa del ecosistema digital diseñado para la transformación digital y mitigación de fraudes internos de la **Clínica Automotriz Arellan Hnos.** El sistema está concebido bajo una arquitectura genérica de marca blanca y multi-inquilino para su posterior comercialización como SaaS.

## Estructura del Ecosistema
- `arellan-enterprise-roadmap`: Hitos multianuales de desarrollo y deuda técnica.
- `arellan-governance`: Reglas de automatización y directrices para agentes de IA (Cursor/Cline).
- `arellan-knowledge-base`: Mapeo de procesos de negocio en Surquillo y resolución de problemas.
- `arellan-platform-governance`: Plantillas operativas y estándares globales de desarrollo.
- `arellan-system-architecture`: Arquitectura de software, diagramas técnicos y registros de decisiones (ADRs).
- `arellan-technical-docs`: Esquemas de bases de datos, despliegues y contratos de integración.
- `arellan-training-academy`: Manuales paso a paso en español segmentados por roles para la familia y el staff.

## Navegación Rápida

### Si eres Administrador (Ana)

1. **[Primeros pasos](admins/quick-start-guide.md)** — login, MFA, vista general del sistema
2. **[Dashboard](admins/dashboard-guide.md)** — qué significan los números en pantalla
3. **[Aprobar gastos](admins/approval-guide.md)** — aprobar/rechazar solicitudes
4. **[Alertas](admins/alerts-guide.md)** — qué hacer cuando suena una alerta
5. **[Reportes](admins/reports-guide.md)** — exportar datos para Edgar y Juan

### Si eres Finance (Valeria)

1. **[Caja diaria](finance/cashbox-daily.md)** ← el más importante, leer primero
2. **[Gestión de gastos](finance/expense-management.md)** — registrar y seguir egresos
3. **[Cierre mensual](finance/monthly-close.md)** — procedimiento de fin de mes
4. **[Preparar auditoría](finance/audit-preparation.md)** — cuando Edgar pide reportes

### Si eres Mecánico

1. **[Guía rápida de tablet](mechanics/tablet-quick-guide.md)** ← una página, leer primero
2. **[Registrar ingreso de vehículo](mechanics/vehicles-checkin.md)** — las 5 fotos obligatorias
3. **[Solicitar repuestos](mechanics/parts-request.md)** — pedir piezas del inventario
4. **[Marcar trabajo terminado](mechanics/work-order-completion.md)** — cerrar tu OT

## Reglas de Oro del Sistema

Estas reglas no son negociables. El sistema las hace cumplir automáticamente.

**1. El QR lo genera el sistema, nunca tú.**
Cuando un cliente va a pagar, genera el QR desde la app. Nunca des tu número personal de Yape o Plin.

**2. Sin aprobación digital, sin dinero.**
Ningún gasto sale de caja sin la aprobación verde en el sistema. No importa qué diga el mecánico.

**3. Las 5 fotos son obligatorias.**
Al recibir un vehículo, debes tomar las 5 fotos (frontal, posterior, lateral izquierdo, lateral derecho, tablero). El sistema bloquea el registro sin ellas.

**4. El sistema ve todo.**
Cada acción queda registrada con tu usuario, la hora y tu IP. No hay forma de borrar ese registro.

**5. MFA es obligatorio.**
El código de 6 dígitos del Google Authenticator nunca se comparte. Si alguien te lo pide, reporta a Edgar.

## Videos de Capacitación

Ver [videos/video-links.md](videos/video-links.md) para links a las grabaciones de entrenamiento.
