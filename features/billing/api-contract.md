# Bill & payment — API contract

Shared shapes: [project-overview.md → API conventions](../../project-overview.md#api-conventions). Facts from the code on 2026-11-11.

| Endpoint | Middleware | Rate limit | Use case |
|---|---|---|---|
| `POST /api/orders/generateBill/{orderId}` | `auth:sanctum`, `restaurant.tenant` | none | [01](./01-generate-bill.md) |
| `POST /api/payments/create2` | same | none | [02](./02-take-payment.md) |
| `POST /api/payments/create` | same | none | [02](./02-take-payment.md) |
| `POST /api/payments/bill/{orderId}` | same | none | [03](./03-get-saved-bill.md) |

No role check; `Idempotency-Key` ignored (BILL-D3). The common failures: no or invalid token → 401 `{ "message": "Unauthenticated." }`; no `restaurantId` → 403 `Restaurant context missing`; `GET` → 405. AUTH-D2 applies. Validation failures: 422 `{ "status": false, "message": "<first error>", "errors": { … } }`.

**Bill object** (all three endpoints): `{ billId, billNumber, financialYear, billStatus, orderId, orderNumber, orderChannelId, subTotal, taxPercent, taxAmount, isDiscountPercent, discountPercent, discountAmount, roundOff, grandTotal, customerName, customerPhone, createdAt, items: [ { menuItemName, quantity, price, total, taxPercent, taxAmount } ] }` — IDs hex, money as decimal strings.

---

## `POST /api/orders/generateBill/{orderId}`

| Field | Rule |
|---|---|
| `subTotal`, `taxPercent`, `taxAmount`, `discountPercent`, `discountAmount`, `grandTotal` | required numeric ≥ 0 |
| `roundOff` | required numeric (may be negative) |
| `isDiscountPercent` | required, any value |

| Situation | HTTP | Body |
|---|---|---|
| OK | 201 | `{ "status": true, "message": "Bill generated successfully", "data": <bill> }` |
| Order not the restaurant's / unknown | 400 | `{ "status": false, "message": "Order not found" }` |
| Order status ≠ 0 | 400 | `{ "status": false, "message": "Bill already generated for this order" }` |
| Database error (null item name, out-of-range figure, bad `isDiscountPercent`) | 400 | `{ "status": false, "message": "Failed to generate bill" }` |
| Bad ID | 400 | `{ "status": false, "message": "hex2bin(): …" }` |

## `POST /api/payments/create2`

| Field | Rule |
|---|---|
| `orderId` | required string, 32-char hex (no dashes) |
| 8 totals | nullable, same types as above; used only when the order is at status 0 (all eight needed then) |
| `payments` | nullable array, ≥ 1 entry |
| `payments.*.amount` | required with `payments`, numeric (any sign); **not stored** |
| `payments.*.paymentType` | required with `payments`, string |
| `payments.*.receivedAmount`, `payments.*.changeAmount` | nullable numeric ≥ 0 |
| `payments.*.paymentNote` | nullable string |
| `amount`, `paymentType` | required when `payments` is absent or empty (old payload) |

| Situation | HTTP | Body |
|---|---|---|
| OK | 200 | `{ "status": true, "message": "Payment created successfully", "data": { "payments": [ { paymentId, restaurantId, transactionId, orderId, paymentType, receivedAmount, changeAmount, paymentStatus, paymentNote, createdBy, created_at, updated_at } ], "bill": <bill with billStatus 1> } }` |
| Business failure | 500 | `{ "status": false, "message": "Order not found" \| "Order is already paid" \| "Bill not generated for this order" \| "Bill not found for this order" \| "Payment details are required" \| "<exception text>", "data": null }` |
| Database failure | 500 | `{ "status": false, "message": "Failed to create payment", "data": null }` |

## `POST /api/payments/create`

Same as `create2`, but one payment from `amount` (required ≥ 0.01, stored as `receivedAmount`), `paymentType`, `paymentNote`; dashes in `orderId` are removed. 200 `data: { payment: {…}, bill: <bill> }`. Failures as above (also with `data: null`).

## `POST /api/payments/bill/{orderId}`

| Situation | HTTP | Body |
|---|---|---|
| Bill found | 200 | `{ "status": true, "message": "Bill retrieved successfully", "data": <bill> }` |
| No bill / another restaurant's | 404 | `{ "status": false, "message": "Bill not found", "data": null }` |
| Bad ID (BILL-D2) | 500 | `{ "message": "Server Error" }` (debug off) |
| Database error | 500 | `{ "status": false, "message": "Failed to retrieve bill", "data": null }` |
