# 01 — Generate the bill — developer cases

Use case: [01-generate-bill.md](../01-generate-bill.md). Contract: [api-contract.md](../api-contract.md#post-apiordersgeneratebillorderid). Fixtures: [README.md](./README.md#fixtures).

`POST /api/orders/generateBill/<order hex>`, body TOT-525, OWN-A's token unless stated. "Billed" = 201 `{ status: true, message: "Bill generated successfully", data: <bill> }`.

| ID | Category | Request / manipulation | Expected | Database after |
|---|---|---|---|---|
| BILL-01-D01 | — (happy path) | ORDER-RUN | Billed: `billNumber` N+1, `financialYear` `2026-27`, `billStatus` 0, `grandTotal` `"525.00"`, `items` = 1 (`Paneer Tikka`, quantity 2, price `"250.00"`, total `"500.00"`, tax `"25.00"`) | 1 `bills` row, 1 `bill_items` row; order status 1, `grandTotal` 525.00, `taxAmount` 25.00 |
| BILL-01-D02 | Business rule | D01 again; ORDER-BILLED; a paid order | 400 `Bill already generated for this order` each | no new bill |
| BILL-01-D03 | Restaurant isolation | ORDER-B | 400 `Order not found` | nothing |
| BILL-01-D04 | Business-rule bypass (accepted BILL-D1) | ORDER-RUN with `{ subTotal: 1, taxPercent: 0, taxAmount: 0, isDiscountPercent: 0, discountPercent: 0, discountAmount: 0, roundOff: 0, grandTotal: 1 }` | Billed, `grandTotal` `"1.00"` | bill and order store 1.00; order `subTotal` now 1.00; `bill_items` still 2 × 250.00 |
| BILL-01-D05 | Input validation / Error hygiene | ORDER-RUN with (a) `{}` (b) `subTotal: -1` (c) `roundOff: -999` (d) `isDiscountPercent: "abc"` (e) `grandTotal: 1000000000` | (a) 422 `The sub total field is required.` (b) 422 `The sub total field must be at least 0.` (c) Billed (d)(e) 400 `Failed to generate bill` | (c) `roundOff` −999.00; (d)(e) nothing |
| BILL-01-D06 | Business rule (accepted BILL-D1) | ORDER-NONAME, twice | 400 `Failed to generate bill` both | no `bills` / `bill_items` row; order status 0 |
| BILL-01-D07 | Input validation / Injection / Error hygiene (accepted BILL-D3) | `<id>` = `xyz`; `zz`; `' OR 1=1 --` (URL-encoded); ORDER-RUN in dashed UUID form | 400 `hex2bin(): Hexadecimal input must have an even length`; 400 `hex2bin(): Input string must be hexadecimal string`; 400 with one of those two; Billed | only the last creates a bill |
| BILL-01-D08 | Role (accepted BILL-D3) | CHF-A bills ORDER-RUN | Billed | `bills.createdBy` `chef` |
| BILL-01-D09 | Concurrency | (a) 2 parallel D01 calls on one order (b) parallel calls on two different REST-A orders | (a) one Billed, one 400 `Bill already generated for this order` (b) both Billed | (a) one bill (b) numbers N+1 and N+2, no duplicate |
| BILL-01-D10 | Over-posting | TOT-525 + `billNumber: 1`, `financialYear: "1999-00"`, `billStatus: 1`, `restaurantId: "<REST-B hex>"`, `orderStatus: 2`, `customerName: "X"` | Billed | number N+1, `2026-27`, `billStatus` 0, REST-A, order status 1, customer NULL |
| BILL-01-D11 | Authentication / Account state / Super-admin | no token → 401; panel cookie only → 401; `GET` → 405; NOREST → 403; WTR-A-OFF old token → Billed (AUTH-D2) | as listed | — |
| BILL-01-D12 | Rate limiting / Concurrency (accepted BILL-D3) | D01 twice with the same `Idempotency-Key`; then 30 calls in a minute | Billed, then 400 `Bill already generated for this order`; no 429 | one bill |
| BILL-01-D13 | Business rule | Insert by SQL a REST-A bill `financialYear` `2025-26`, `billNumber` 999, and a REST-B bill `2026-27` number 500; then D01 | Billed, `billNumber` N+1 | neither inserted bill affects the number |
