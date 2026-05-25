# Contactos para Importaciones

**AVISO DE CONFIDENCIALIDAD:** Este documento describe el protocolo para gestionar los contactos de importación. Los datos reales de contacto (nombres, teléfonos, emails, cuentas bancarias) se almacenan ÚNICAMENTE en el sistema digital, cifrados con AES-256, y solo son visibles para OWNER.

## Por Qué los Contactos Están Solo en el Sistema

Históricamente, el empleado a cargo de las importaciones (Ricardo) mantenía los contactos de proveedores extranjeros exclusivamente en su teléfono personal, creando una dependencia que usó para cobrar comisiones no declaradas del 20-30%.

**La solución:** Todos los contactos de proveedores internacionales se almacenan en el sistema con acceso controlado por RBAC. Si un empleado se va, el taller no pierde los contactos.

## Protocolo para Nuevos Contactos de Importación

1. **Solo Edgar o Juan** pueden agregar contactos de proveedores internacionales
2. El contacto debe tener una empresa verificable (no individuos sin empresa)
3. Registro en el sistema con: nombre empresa, país, Tax ID, URL web oficial, persona de contacto, email corporativo, método de pago aceptado
4. El contacto queda asociado al proveedor en el directorio — no a un empleado específico
5. Cualquier descuento o comisión ofrecida por el proveedor debe declararse en el sistema

## Tipos de Contactos de Importación

| Tipo | Descripción |
|------|-------------|
| Proveedor directo | Fabricante o distribuidor autorizado |
| Agente de compras | Intermediario en el país de origen (solo si es transparente) |
| Agente de aduana Lima | Encargado del despacho aduanero en Perú |
| Courier de carga | DHL/FedEx/UPS para envíos pequeños |
| Naviera | Para contenedores (importaciones grandes, futuro) |

## Agente de Aduana

El agente de aduana en Lima debe estar registrado en el sistema con:
- Empresa y RUC
- Código de agente de aduana SUNAT
- Teléfono y WhatsApp
- Tarifa estándar de servicio

El agente de aduana no debe coincidir con familiares o conocidos del empleado encargado de importaciones — conflicto de interés.

## Comunicación con Proveedores Internacionales

Toda comunicación relevante (cotizaciones, órdenes de compra, confirmaciones) debe:
- Adjuntarse en el sistema como documento
- Ser visible para el OWNER
- Nunca hacerse solo por teléfono sin respaldo escrito

Los emails corporativos (`@arellan.pe`) deben usarse para todas las comunicaciones de negocio con proveedores internacionales.
