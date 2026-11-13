# 03 — Get a saved bill — developer cases

Use case: [03-get-saved-bill.md](../03-get-saved-bill.md). Contract: [api-contract.md](../api-contract.md#post-apipaymentsbillorderid). Fixtures: [README.md](./README.md#fixtures).

`POST /api/payments/bill/<order hex>`, no body, OWN-A's token unless stated.

| ID | Category | Request / manipulation | Expected |
|---|---|---|---|
| BILL-03-D01 | — (happy path) | ORDER-BILLED | 200 `{ status: true, message: "Bill retrieved successfully", data: <bill> }`, `billStatus` 0, `grandTotal` `"525.00"`, 1 item |
| BILL-03-D02 | Restaurant isolation | ORDER-RUN (not billed); ORDER-B-BILLED | 404 `{ status: false, message: "Bill not found", data: null }` both |
| BILL-03-D03 | Input validation / Injection / Error hygiene (accepted BILL-D2) | `xyz`; `' OR 1=1 --` (URL-encoded); ORDER-BILLED in dashed UUID form | 500 `{ "message": "Server Error" }`; 500 `Server Error`; 200 as D01 |
| BILL-03-D04 | Role / Authentication / Account state / Super-admin | CHF-A → 200; no token → 401; panel cookie only → 401; `GET` → 405; NOREST → 403 | as listed |
| BILL-03-D05 | Business rule | ORDER-BILLED after BILL-02-D01 | 200, `billStatus` 1 |
| BILL-03-D06 | Rate limiting | 60 calls in a minute | all 200; no 429 |
