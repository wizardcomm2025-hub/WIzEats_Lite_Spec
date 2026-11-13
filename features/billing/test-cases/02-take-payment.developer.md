# 02 — Take payment — developer cases

Use case: [02-take-payment.md](../02-take-payment.md). Contract: [api-contract.md](../api-contract.md#post-apipaymentscreate2). Fixtures: [README.md](./README.md#fixtures).

`POST /api/payments/create2` with PAY-525 for ORDER-BILLED, OWN-A's token, unless stated. "Paid" = 200 `{ status: true, message: "Payment created successfully", data: { payments: [...], bill: <bill, billStatus 1> } }`; database: bill `billStatus` 1, order `orderStatus` 2 and `paymentStatus` 1. "Fail(m)" = 500 `{ status: false, message: "m", data: null }`.

| ID | Category | Request / manipulation | Expected | Database after |
|---|---|---|---|---|
| BILL-02-D01 | — (happy path) | PAY-525 | Paid; one payment: `paymentType` `CASH`, `receivedAmount` 600 and `changeAmount` 75 (echoed as sent), `paymentStatus` `COMPLETED`, `transactionId` `TXN-` + 12 characters, `paymentId` / `orderId` hex | 1 `payments` row: 600.00 / 75.00 |
| BILL-02-D02 | — (split) | `payments`: CASH `amount` 300 `receivedAmount` 300; UPI `amount` 225 | Paid; 2 payments | 2 rows; the UPI row's `receivedAmount` NULL |
| BILL-02-D03 | Business-rule bypass (accepted BILL-D1) | (a) one entry `amount: 1` (b) `amount: -500` (c) `paymentType: "BITCOIN"` | Paid each (fresh ORDER-BILLED each) | no amount stored; (c) `paymentType` `BITCOIN` |
| BILL-02-D04 | Business rule (pay-and-bill) | ORDER-RUN; PAY-525 + TOT-525 | Paid; `bill.billNumber` N+1 | bill created and paid; order status 2 |
| BILL-02-D05 | Business rule / Error hygiene (accepted BILL-D3) | ORDER-RUN with (a) PAY-525 only (b) PAY-525 + `grandTotal: 525` only | (a) Fail(`Bill not generated for this order`) (b) Fail(`Undefined array key "subTotal"`) | nothing (rolled back) |
| BILL-02-D06 | Business rule / Concurrency (accepted BILL-D3) | D01, then D01 again with the same `Idempotency-Key` | Paid, then Fail(`Order is already paid`) | one payment |
| BILL-02-D07 | Restaurant isolation | ORDER-B-BILLED | Fail(`Order not found`) | nothing |
| BILL-02-D08 | Input validation | (a) `{ "orderId": "<hex>" }` (b) `payments: [ {} ]` (c) `receivedAmount: -1` (d) no `orderId` | 422: (a) `The amount field is required when payments is not present.` (b) `The payments.0.amount field is required when payments is present.` (c) `The payments.0.receivedAmount field must be at least 0.` (d) `The order id field is required.` | nothing |
| BILL-02-D09 | Input validation / Error hygiene (accepted BILL-D2/D3) | (a) `{ orderId, amount: 525, paymentType: "CASH" }` (old payload on create2) (b) PAY-525 with ORDER-BILLED in dashed UUID form (c) `orderId: "xyz"` | (a) Fail(`Payment details are required`) (b) Fail(`hex2bin(): Input string must be hexadecimal string`) (c) Fail(`hex2bin(): Hexadecimal input must have an even length`) | nothing |
| BILL-02-D10 | Business rule (accepted BILL-D2) | (a) ORDER-T1 billed then paid (b) an order created as the app does (`tableId: null`) on Dine In, with Table 2 occupied | Paid both | (a) Table 1 `status` 0 (b) no `tables` row changed; Table 2 still 1 |
| BILL-02-D11 | Role (accepted BILL-D3) | CHF-A pays | Paid | `payments.createdBy` `chef` |
| BILL-02-D12 | Concurrency | 2 parallel D01 calls | one Paid, one Fail(`Order is already paid`) | one payment |
| BILL-02-D13 | Over-posting | PAY-525 entry + `paymentStatus: "FAILED"`, `transactionId: "X"`, `createdBy: "admin"`; top-level `restaurantId: "<REST-B hex>"`, `billStatus: 0` | Paid | `COMPLETED`, generated `TXN-…`, `createdBy` `owner`, REST-A |
| BILL-02-D14 | Authentication / Account state / Super-admin | no token → 401; panel cookie only → 401; `GET` → 405; NOREST → 403; WTR-A-OFF old token → Paid (AUTH-D2) | as listed | — |
| BILL-02-D15 | — (legacy `payments/create`) / Input validation | `POST /api/payments/create` (a) `{ orderId, amount: 525, paymentType: "CASH" }` (b) `amount: 0` | (a) 200 `data: { payment: {…, receivedAmount: 525}, bill }` (b) 422 `The amount field must be at least 0.01.` | (a) order paid |
| BILL-02-D16 | Rate limiting | 30 payment calls in a minute (already-paid order) | all Fail(`Order is already paid`); no 429 | — |
| BILL-02-D17 | Injection | `paymentNote` = `'); DROP TABLE payments; --` and `<script>alert(1)</script>` | Paid | note stored exactly; tables intact |
