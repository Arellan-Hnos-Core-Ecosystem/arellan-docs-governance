# Platform Governance — Gobernanza de la Plataforma

Estándares, procesos y plantillas que gobiernan el desarrollo y operación de todos los repositorios del ecosistema digital Arellan.

## Contenido

### Procesos Operativos

| Documento | Propósito |
|-----------|-----------|
| `processes/bug-report-template.md` | Plantilla para reportar bugs de forma estructurada |
| `processes/feature-request-template.md` | Plantilla para solicitar nuevas funcionalidades |
| `processes/incident-template.md` | Plantilla para documentar incidentes de producción |
| `processes/sprint-retrospective.md` | Formato de retrospectivas quincenales del equipo |

### Estándares de Desarrollo

| Documento | Propósito |
|-----------|-----------|
| `standards/api-design-standards.md` | REST API: códigos HTTP, paginación, formato de errores |
| `standards/backend-standards.md` | NestJS: estructura de módulos, patterns obligatorios |
| `standards/frontend-standards.md` | Next.js: componentes, hooks, gestión de estado |
| `standards/security-standards.md` | JWT, MFA, RBAC, data masking, rate limiting |
| `standards/testing-standards.md` | Cobertura mínima por módulo, tipos de tests |

## Filosofía de Gobernanza

El ecosistema Arellan es mantenido por un equipo técnico pequeño (2 personas). Los estándares aquí documentados tienen el objetivo de:

1. **Consistencia:** El código escrito en módulo X se parece al código de módulo Y, reduciendo la curva de contexto
2. **Seguridad por defecto:** Los estándares incluyen seguridad desde el diseño (secure by default)
3. **Trazabilidad:** Todo cambio tiene un audit trail — en el código (git log) y en el sistema (audit_logs)
4. **Minimizar deuda técnica:** Los estándares previenen atajos que se convierten en problemas futuros

## Ciclo de Revisión

- **Estándares de código:** Revisión semestral o cuando hay un cambio de tecnología
- **Procesos:** Revisión trimestral
- **Templates:** Se actualizan según la experiencia acumulada con incidentes reales
