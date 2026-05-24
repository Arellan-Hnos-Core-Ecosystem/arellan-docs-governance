# Proceso de Autorización de Gastos, Compras e Importaciones

## 1. Propósito del Sistema de Control
Evitar la salida discrecional de dinero en efectivo de la caja chica para compras de repuestos locales o pagos de comisiones no autorizadas en procesos de importación (frenando el sobrecosto del 20-30% por conocimiento informal).

## 2. Ciclo de Vida y Estados de una Solicitud de Gasto
Toda salida de dinero de la clínica automotriz debe seguir estrictamente la siguiente máquina de estados integrada en el backend:

[REGISTRADO] ──> [NOTIFICADO (Push/MFA)] ──> [APROBADO POR DUEÑO] ──> [DESEMBOLSADO] ──> [Sustentado con XML/PDF]

## 3. Reglas del Negocio y Complejidades Operativas
1. **Petición con Sustento:** Cuando un mecánico requiera una pieza local o se procese el arancel de una importación, se registra una solicitud en el sistema indicando:
    - Categoría del gasto (Repuestos Locales, Importaciones, Herramientas, Servicios Tercerizados).
    - Monto exacto en Soles o Dólares.
    - Proveedor seleccionado del directorio homologado (`supplier-directory.md`).
    - Enlace obligatorio al PDF de la cotización o proforma del proveedor.
2. **Límites de Autorización y Flujo Asíncrono:**
    - **Gastos Menores (Menor o igual a S/. 100.00):** La encargada de finanzas tiene un fondo de contingencia diario automatizado. Puede autorizar el desembolso directamente si el gasto está dentro del presupuesto asignado al día.
    - **Gastos Mayores (Superior a S/. 100.00 o Importaciones):** El sistema bloquea el desembolso y dispara de inmediato una notificación push de alta prioridad y un mensaje automatizado mediante la API de WhatsApp a los celulares de Edgar y Juan.
3. **Aprobación Remota por MFA:** El software no liberará la transacción hasta que uno de los dos hermanos ingrese a la aplicación móvil (`arellan-mobile-app`), verifique el sustento y presione "Aprobar" ingresando su pin biométrico celular.
4. **Auditoría de Compras de Importación:** Para las piezas importadas, el sistema calcula automáticamente el costo de importación base (flete, desaduanaje, aranceles cargados en la configuración). Si el porcentaje solicitado por intermediarios supera los márgenes de tolerancia del sistema (máximo 5%), el software bloquea la orden de compra y emite una alerta de riesgo financiero por intento de cobro abusivo de comisiones.
5. **Cierre de Ciclo con SUNAT:** Todo gasto autorizado tiene un plazo máximo de 48 horas para ser sustentado en el sistema. La administradora debe subir el archivo XML o PDF de la Factura Electrónica. El backend se conecta con el módulo de consulta de validez de comprobantes de la SUNAT para verificar que el documento esté activo, aprobado y corresponda al RUC del proveedor.