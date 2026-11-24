# 03 — Export the report — developer cases

Use case: [03-export-report.md](../03-export-report.md). Contract: [api-contract.md](../api-contract.md#post-apireportsget-all-items). Fixtures: [README.md](./README.md#fixtures).

`POST /api/reports/get-all-items` or `/get-all-orders`, body `{}`, OWN-A's token unless stated. Cases tagged *Fails until … is fixed* assert the target; today's result is in brackets.

| ID | Category | Request / manipulation | Expected |
|---|---|---|---|
| RPT-03-D01 | — (happy path) — *Fails until RPT-D1 is fixed* | ORDER-LIVE and ORDER-PAID; `get-all-items` | 200 `{ status: true, message: "items fetched successfully", data: [ { menuItemName: "Paneer Tikka", quantity: "2", total: "500.00", last_created: <ORDER-PAID's> } ] }` — only the paid order (today: `quantity` "4", `total` "1000.00") |
| RPT-03-D02 | Business rule (orders export fixed 2026-11-18/19) | (a) ORDER-LIVE and ORDER-PAID; `get-all-orders` (b) no orders today; `get-all-orders` (c) no orders today; `get-all-items` | (a) 200 `Orders fetched successfully`, 2 rows newest first, lower-case hex `orderId`, `tableId` / `roomId` null, `grandTotal` `"525.00"` / `"500.00"`-style, `paymentStatus` `Paid` / `Unpaid`, `created_at` in IST (b)(c) 200 `data: []` |
| RPT-03-D03 | Input validation | (a) `{ "startDate": "2026-11-12", "endDate": "2026-11-01" }` (b) `{ "startDate": "x" }` | 422 (a) `The end date field must be a date after or equal to start date.` (b) `The start date field must be a valid date.` |
| RPT-03-D04 | Restaurant isolation | Only ORDER-B-PAID; `get-all-items`, `get-all-orders` | `data: []` both |
| RPT-03-D05 | Role / Authentication / Account state / Super-admin / Over-posting / Rate limiting — role part *Fails until RPT-D3 is fixed* | WTR-A and CHF-A → 403 `{ status: false, message: "You do not have permission to perform this action", data: null }` (today as D01); no token → 401; panel cookie → 401; `GET` → 405; NOREST → 403; body `{ "restaurantId": "<REST-B hex>" }` → REST-A's data; 60 calls a minute → no 429 | as listed |
| RPT-03-D06 | Business rule | Two REST-A items both named `Paneer Tikka` (different IDs and prices 250 / 300), one line each today | one row `Paneer Tikka`, `quantity` `"2"`, `total` `"550.00"` |
| RPT-03-D07 | Injection | ITEM-FORMULA in a paid order today; `get-all-items` | 200; `menuItemName` `=1+1` returned as is (the CSV neutralising is the app's job — RPT-D3) |
| RPT-03-D08 | Business rule — *Fails until RPT-D1 is fixed* | An order billed at 525.00 and paid with one CASH leg `receivedAmount` 100 (possible until BILL-D1 is fixed); `get-all-items` | not counted (payments don't cover the total) (today: counted) |
| RPT-03-D09 | Business rule — *Fails until RPT-D2 is fixed* / *Fails until RPT-D3 is fixed* | (a) PAID-AT(today 00:30 IST); `get-all-items` (b) ORDER-1050; `get-all-orders` | (a) counted under today (today: not — UTC date) (b) `grandTotal` `"1050.00"` (today: `"1,050.00"`) |
