# Finance Endpoints

**Rate limit:** 10 req/min por usuario (módulo financiero)
**MFA requerida:** Sí, para todos los endpoints de este módulo (roles OWNER/ADMIN/FINANCE)

## Caja

### POST /finance/cashbox/open

Abrir la caja del día.

**Roles:** ADMIN, FINANCE, OWNER

**Request:**
```json
{
  "initialAmount": 500.00,
  "notes": "Saldo de ayer confirmado"
}
```

**Response 201:**
```json
{
  "id": "uuid",
  "openedAt": "2024-01-15T07:00:00.000Z",
  "openedBy": "Ana García",
  "initialAmount": 500.00,
  "status": "OPEN"
}
```

**Response 409:** La caja ya fue abierta hoy

---

### POST /finance/cashbox/close

Cerrar la caja del día.

**Roles:** ADMIN, FINANCE, OWNER

**Request:**
```json
{
  "finalAmount": 850.00,
  "notes": "Cierre sin observaciones"
}
```

**Response 200:**
```json
{
  "id": "uuid",
  "closedAt": "2024-01-15T18:00:00.000Z",
  "initialAmount": 500.00,
  "expectedFinalAmount": 900.00,
  "actualFinalAmount": 850.00,
  "discrepancy": -50.00,
  "hasDiscrepancy": true,
  "status": "CLOSED_WITH_DISCREPANCY"
}
```

---

### GET /finance/cashbox/current

Obtener el estado de la caja del día actual.

**Roles:** ADMIN, FINANCE, OWNER

**Response 200:**
```json
{
  "id": "uuid",
  "status": "OPEN",
  "openedAt": "2024-01-15T07:00:00.000Z",
  "initialAmount": 500.00,
  "currentDigitalBalance": 650.00,
  "currentCashBalance": null,  // null hasta que se cierre
  "totalIncome": 1500.00,
  "totalExpenses": 350.00
}
```

---

## Gastos

### POST /finance/expenses

Crear solicitud de gasto.

**Roles:** FINANCE, ADMIN, OWNER

**Request:**
```json
{
  "amount": 750.00,
  "description": "Compra de aceite por mayor",
  "category": "REPUESTOS_LOCALES",
  "providerId": "uuid",
  "quotationUrl": "https://..."
}
```

**Response 201:**
```json
{
  "id": "uuid",
  "amount": 750.00,
  "status": "PENDING_OWNER",   // Sistema asignó nivel de aprobación automáticamente
  "requiredApprovalLevel": "OWNER",
  "notificationSent": true
}
```

---

### POST /finance/expenses/:id/approve

Aprobar un gasto.

**Roles:** Según el monto (FINANCE ≤100, ADMIN ≤500, OWNER >500)

**Response 200:**
```json
{
  "id": "uuid",
  "status": "APPROVED",
  "approvedBy": "Edgar Arellan",
  "approvedAt": "2024-01-15T08:30:00.000Z"
}
```

**Response 403 (código SELF_APPROVAL_FORBIDDEN):** El solicitante no puede aprobar su propio gasto

---

## Pagos y QR

### POST /finance/payment/qr

Generar QR de pago para una OT.

**Roles:** ADMIN, FINANCE

**Request:**
```json
{
  "workOrderId": "uuid",
  "paymentMethod": "YAPE"  // YAPE | PLIN | TARJETA
}
```

**Response 201:**
```json
{
  "qrCode": "data:image/png;base64,...",
  "amount": 350.00,
  "expiresAt": "2024-01-15T08:12:00.000Z",  // 7 minutos
  "paymentId": "culqi-payment-uuid"
}
```

---

### GET /finance/transactions

Listar transacciones financieras.

**Roles:** OWNER, FINANCE (con MFA)

**Query params:** `?from=2024-01-01&to=2024-01-31&type=INCOME&limit=20&cursor=uuid`

**Response 200:**
```json
{
  "data": [
    {
      "id": "uuid",
      "amount": 350.00,
      "type": "INCOME",
      "method": "YAPE",
      "workOrderId": "uuid",
      "createdAt": "2024-01-15T08:05:00.000Z"
    }
  ],
  "pagination": { "total": 45, "hasNextPage": false, "cursor": null }
}
```

---

### GET /finance/summary

Resumen financiero del día/mes.

**Roles:** OWNER, FINANCE, ADMIN (con MFA para FINANCE/ADMIN)

**Query params:** `?period=today|week|month|custom&from=&to=`

**Response 200:**
```json
{
  "period": "today",
  "income": { "total": 2500.00, "yape": 1200.00, "cash": 800.00, "card": 500.00 },
  "expenses": { "total": 450.00, "approved": 350.00, "pending": 100.00 },
  "netBalance": 2050.00,
  "cashboxDiscrepancy": -5.00
}
```
