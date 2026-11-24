# Bill & payment — API contract

Shared shapes: [project-overview.md → API conventions](../../project-overview.md#api-conventions). Facts from the code on 2026-11-11, re-checked 2026-11-24 (API commit d54d6c8 — no change). Rows marked **Target** describe agreed fixes (BILL-D1, BILL-D3) and are not in the code yet.

| Endpoint | Middleware | Rate limit | Roles — target (X-D1) | Use case |
|---|---|---|---|---|
| `POST /api/orders/generateBill/{orderId}` | `auth:sanctum`, `restaurant.tenant` | none | owner, waiter | [01](./01-generate-bill.md) |
| `POST /api/payments/create2` | same | none | owner, waiter | [02](./02-take-payment.md) |
| `POST /api/payments/create` | same | none | owner, waiter | [02](./02-take-payment.md) |
| `POST /api/payments/bill/{orderId}` | same | none | owner, waiter | [03](./03-get-saved-bill.md) |

Today no role check; `Idempotency-Key` ignored (BILL-D3). **Target:** chefs → 403 `{ "status": false, "message": "You do not have permission to perform this action", "data": null }`; `Idempotency-Key` honoured on the three writes (same key + same body → the first answer, nothing written; same key + different body → 422 `Idempotency-Key reused with a different request`). The common failures: no or invalid token → 401 `{ "message": "Unauthenticated." }`; no `restaurantId` → 403 `Restaurant context missing`; `GET` → 405. Today AUTH-D2 applies; **target (X-D2)** 403 `Your account is inactive.` / `Your restaurant is not approved.` / `Your restaurant is inactive.` Validation failures: 422 `{ "status": false, "message": "<first error>", "errors": { … } }`.

**Bill object** (all endpoints): `{ billId, billNumber, financialYear, billStatus, orderId, orderNumber, orderChannelId, subTotal, taxPercent, taxAmount, isDiscountPercent, discountPercent, discountAmount, roundOff, grandTotal, customerName, customerPhone, createdAt, items: [ { menuItemName, quantity, price, total, taxPercent, taxAmount } ] }` — IDs hex, money as decimal strings.

**Target bill calculation (BILL-D1)** — the app's own rules: `subTotal` = Σ order lines; `discountAmount` = `subTotal × discountPercent / 100` or the amount sent, capped at `subTotal`; `taxAmount` = round((`subTotal` − `discountAmount`) × order `taxPercent` / 100, 2); `grandTotal` = that sum rounded to the nearest rupee (.50 up); `roundOff` = `grandTotal` − the unrounded sum.

---

## `POST /api/orders/generateBill/{orderId}`

| Field | Rule today | Target |
|---|---|---|
| `subTotal`, `taxPercent`, `taxAmount`, `discountPercent`, `discountAmount`, `grandTotal` | required numeric ≥ 0 — stored as sent | only `discountPercent` (0–100) / `discountAmount` (≥ 0) used; the others optional and ignored |
| `roundOff` | required numeric (may be negative) — stored as sent | optional, ignored |
| `isDiscountPercent` | required, any value | optional boolean (default false) |

| Situation | HTTP | Body |
|---|---|---|
| OK | 201 | `{ "status": true, "message": "Bill generated successfully", "data": <bill> }` — today with the posted totals; **target** with the server's |
| Order not the restaurant's / unknown | 400 | `{ "status": false, "message": "Order not found" }`. **Target:** 404, `data: null` |
| Order status ≠ 0 | 400 | `{ "status": false, "message": "Bill already generated for this order" }`. **Target:** 409 |
| Database error (null item name, out-of-range figure, bad `isDiscountPercent`) | 400 | `{ "status": false, "message": "Failed to generate bill" }`. **Target:** 500 `Something went wrong` (these inputs no longer reach the database) |
| Bad ID | 400 | `{ "status": false, "message": "hex2bin(): …" }`. **Target:** 404 `Order not found` |
| **Target** discount above the subtotal | 422 | `The discount can't be more than the subtotal` |
| **Target** non-zero discount with the discount switch off | 422 | `Discounts are turned off for this restaurant` |

## `POST /api/payments/create2`

| Field | Rule today | Target |
|---|---|---|
| `orderId` | required string, 32-char hex (no dashes) | dashes allowed |
| 8 totals | nullable; used only when the order is at status 0 (all eight needed then) | only the discount fields, as in generateBill |
| `payments` | nullable array, ≥ 1 entry | required (≥ 1) |
| `payments.*.amount` | required with `payments`, numeric (any sign); **not stored** | > 0 (0 only for a ₹0 bill); stored |
| `payments.*.paymentType` | required with `payments`, string (any text) | `CASH` or `UPI` |
| `payments.*.receivedAmount`, `payments.*.changeAmount` | nullable numeric ≥ 0 | CASH: `receivedAmount` ≥ `amount`, `changeAmount` = the difference |
| `payments.*.paymentNote` | nullable string | same |
| `amount`, `paymentType` | required when `payments` is absent or empty (old payload) | not accepted here → 422 `Payment details are required` |

| Situation | HTTP | Body |
|---|---|---|
| OK | 200 | `{ "status": true, "message": "Payment created successfully", "data": { "payments": [ { paymentId, restaurantId, transactionId, orderId, paymentType, receivedAmount, changeAmount, paymentStatus, paymentNote, createdBy, created_at, updated_at } ], "bill": <bill with billStatus 1> } }` (target: each payment also has `amount`) |
| Business failure | 500 | `{ "status": false, "message": "Order not found" \| "Order is already paid" \| "Bill not generated for this order" \| "Bill not found for this order" \| "Payment details are required" \| "<exception text>", "data": null }` |
| Database failure | 500 | `{ "status": false, "message": "Failed to create payment", "data": null }` |
| **Target** unknown / other restaurant's / malformed order | 404 | `Order not found` |
| **Target** already paid | 409 | `Order is already paid` |
| **Target** amounts don't add up to the grand total | 422 | `Payment total does not match the bill` |
| **Target** payment details missing | 422 | `Payment details are required` |
| **Target** unexpected | 500 | `Failed to create payment` — no exception text |

## `POST /api/payments/create`

Same as `create2`, but one payment from `amount` (required ≥ 0.01, stored as `receivedAmount`), `paymentType`, `paymentNote`; dashes in `orderId` are removed. 200 `data: { payment: {…}, bill: <bill> }`. Failures as above (also with `data: null`). **Target:** the same rules as `create2` — `amount` must equal the grand total, `paymentType` `CASH` / `UPI`, the same codes.

## `POST /api/payments/bill/{orderId}`

| Situation | HTTP | Body |
|---|---|---|
| Bill found | 200 | `{ "status": true, "message": "Bill retrieved successfully", "data": <bill> }` |
| No bill / another restaurant's | 404 | `{ "status": false, "message": "Bill not found", "data": null }` |
| Bad ID | 500 | `{ "message": "Server Error" }` (debug off). **Target:** 404 `Bill not found` |
| Database error | 500 | `{ "status": false, "message": "Failed to retrieve bill", "data": null }` |
