# 02 — See past orders — developer cases

Use case: [02-see-past-orders.md](../02-see-past-orders.md). Contract: [api-contract.md](../api-contract.md#post-apiordersorderhistory). Fixtures: [README.md](./README.md#fixtures).

`POST /api/orders/orderHistory`, body BODY-TODAY, OWN-A's token unless stated. "Hist-OK" = 200 `{ status: true, message: "Order history retrieved successfully", data: [...], pagination: {...} }`.

| ID | Category | Request / manipulation | Expected |
|---|---|---|---|
| HIST-02-D01 | — (happy path) | Only ORDER-PAID is paid | Hist-OK, 1 row: `paymentStatus` `Paid`, `orderStatus` 2, `grandTotal` `"525.00"`, `orderDateTime` today in IST (`YYYY-MM-DD HH:MM:SS`), `payments` `[ { paymentType: "CASH", receivedAmount: "600.00", changeAmount: "75.00" } ]`, `bill.billNumber` set, `kots` 1 entry; `pagination` `{ currentPage: 1, perPage: 10, total: 1, lastPage: 1, from: 1, to: 1 }` |
| HIST-02-D02 | Business rule | PAID-25 only; pages 1–4 | 10, 10, 5 rows; `total` 25, `lastPage` 3; page 4: `data: []`, `pagination` `{ currentPage: 4, perPage: 10, total: 25, lastPage: 3 }`; the 25 IDs all different |
| HIST-02-D03 | Business rule | ORDER-LIVE and ORDER-LIVE-BILLED exist | not listed |
| HIST-02-D04 | Restaurant isolation | ORDER-B-PAID exists | not listed |
| HIST-02-D05 | Business rule | A paid REST-A order with `created_at` = today 00:30 IST (SQL) | listed under BODY-TODAY |
| HIST-02-D06 | Business-rule bypass (accepted HIST-D3) | A paid order whose payment was posted with `paymentType: "UP:I"` (BILL-D1), received 600 | its `payments` entry is `{ paymentType: "UP", receivedAmount: "I", changeAmount: "600.00" }` |
| HIST-02-D07 | Role (accepted HIST-D3) | CHF-A | Hist-OK as D01 |
| HIST-02-D08 | Authentication / Account state / Super-admin | no token → 401; panel cookie only → 401; `GET` → 405; NOREST → 403; WTR-A-OFF old token → Hist-OK (AUTH-D2) | as listed |
| HIST-02-D09 | Input validation | (a) `page: 0` (b) `perPage: 0` (c) `perPage: 101` | 422: (a) `The page field must be at least 1.` (b) `The per page field must be at least 1.` (c) `The per page field must not be greater than 100.` |
| HIST-02-D10 | Rate limiting | 60 calls in a minute with `perPage: 100` and PAID-25 | all Hist-OK; no 429 |
