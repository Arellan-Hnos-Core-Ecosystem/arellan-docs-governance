# Directorio de Proveedores

Registro de proveedores activos del taller. Los datos de contacto reales se mantienen en el sistema digital (protegidos, no en este documento público). Este documento describe las categorías y el protocolo de homologación.

**IMPORTANTE:** Los datos reales de proveedores (precios, contactos específicos, cuentas bancarias) están en el sistema digital y solo son visibles para OWNER, ADMIN y FINANCE. Los datos sensibles están cifrados con AES-256.

## Categorías de Proveedores

### Proveedores Nacionales (Lima)

| Categoría | Descripción |
|-----------|-------------|
| Repuestos generales | Aceites, filtros, bujías, pastillas — disponibles en Lima |
| Repuestos especializados | Por marca/modelo específico |
| Herramientas | Equipos y herramientas de taller |
| Consumibles | Trapos, desengrasantes, lubricantes |
| Servicios | Grúa, chapistería externa, pintura |

### Proveedores Internacionales

| País de origen | Especialidad típica |
|---------------|-------------------|
| EE.UU. | Repuestos para vehículos americanos, herramientas especializadas |
| China | Repuestos económicos, piezas de carrocería |
| Europa (Alemania/España) | Repuestos OEM para vehículos europeos |
| Japón (vía Miami) | Repuestos originales para marcas japonesas |

## Protocolo de Homologación de Proveedores

### Para Proveedores Nacionales

Antes de agregar un nuevo proveedor al directorio:
1. Verificar RUC activo y habido en SUNAT (`sunat.gob.pe`)
2. Verificar que emite comprobantes electrónicos válidos
3. Obtener al menos 1 referencia de otro taller
4. Aprobación de ADMIN

### Para Proveedores Internacionales

Solo OWNER puede agregar proveedores internacionales:
1. Verificar Tax ID en base de datos pública del país de origen
2. Verificar que el proveedor existe físicamente (búsqueda en Google, web oficial)
3. Solicitar primera muestra/pedido pequeño antes de pedidos grandes
4. Registrar en el sistema con: nombre legal exacto, país, Tax ID, cuenta bancaria para pagos

**Regla crítica:** Ningún proveedor internacional puede ser "contacto personal" de un empleado. Todos deben ser verificables públicamente y registrados en el sistema por el owner.

## Proveedores Prohibidos

Un proveedor queda automáticamente bloqueado si:
- Solicita pago a cuenta de tercero diferente al proveedor registrado
- Sus precios resultaron consistentemente 30%+ por encima del mercado
- Hay evidencia de que pagó comisiones a empleados del taller
- Su RUC/Tax ID fue dado de baja o está en estado irregular

## Revisión del Directorio

El directorio de proveedores se revisa cada 6 meses:
- Proveedores sin compras en 12 meses → marcados como INACTIVO
- Precios de referencia actualizados con cotizaciones del mercado
- Verificación de RUC/Tax ID vigente
