# 03 — Export the report — developer cases

Use case: [03-export-report.md](../03-export-report.md). Contract: [api-contract.md](../api-contract.md#post-apireportsget-all-items). Fixtures: [README.md](./README.md#fixtures).

`POST /api/reports/get-all-items` or `/get-all-orders`, body `{}`, OWN-A's token unless stated.

| ID | Category | Request / manipulation | Expected |
|---|---|---|---|
| RPT-03-D01 | — (happy path; accepted RPT-D1) | ORDER-LIVE and ORDER-PAID; `get-all-items` | 200 `{ status: true, message: "items fetched successfully", data: [ { menuItemName: "Paneer Tikka", quantity: "4", total: "1000.00", last_created: <latest> } ] }` |
| RPT-03-D02 | Business rule / Error hygiene (accepted RPT-D3) | (a) ORDER-LIVE; `get-all-orders` (b) the same with `APP_DEBUG=true` (c) no orders today; `get-all-orders` (d) no orders today; `get-all-items` | (a) 500 `{ "message": "Server Error" }` (b) 500 debug JSON with `Malformed UTF-8 characters, possibly incorrectly encoded` (c) 200 `{ status: true, message: "Orders fetched successfully", data: [] }` (d) 200 `data: []` |
| RPT-03-D03 | Input validation | (a) `{ "startDate": "2026-11-12", "endDate": "2026-11-01" }` (b) `{ "startDate": "x" }` | 422 (a) `The end date field must be a date after or equal to start date.` (b) `The start date field must be a valid date.` |
| RPT-03-D04 | Restaurant isolation | Only ORDER-B-PAID; `get-all-items`, `get-all-orders` | `data: []` both |
| RPT-03-D05 | Role / Authentication / Account state / Super-admin / Over-posting / Rate limiting | WTR-A and CHF-A → as D01; no token → 401; panel cookie → 401; `GET` → 405; NOREST → 403; body `{ "restaurantId": "<REST-B hex>" }` → REST-A's data; 60 calls a minute → no 429 | as listed |
| RPT-03-D06 | Business rule | Two REST-A items both named `Paneer Tikka` (different IDs and prices 250 / 300), one line each today | one row `Paneer Tikka`, `quantity` `"2"`, `total` `"550.00"` |
| RPT-03-D07 | Injection | An order line with `menuItemName` `=1+1` (ORD-D1 allows any name); `get-all-items` | 200; `menuItemName` `=1+1` returned as is (the CSV risk is in the app — RPT-D3) |
