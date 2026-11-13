# 03 — Filter and search — developer cases

Use case: [03-filter-and-search.md](../03-filter-and-search.md). Contract: [api-contract.md](../api-contract.md#request-fields-both-endpoints). Fixtures: [README.md](./README.md#fixtures) (ORDER-PAID, ORDER-UPI, a paid order from yesterday).

`POST /api/orders/orderHistory`, BODY-TODAY plus the fields given, OWN-A's token. Live's handling of the same fields: 01-D03, 01-D05.

| ID | Category | Request / manipulation | Expected |
|---|---|---|---|
| HIST-03-D01 | — (happy path) | `search`: (a) the last digit(s) of ORDER-PAID's number (b) `"525"` (c) `"CODE-B"` | (a) ORDER-PAID among the rows (b) ORDER-PAID and ORDER-UPI (`grandTotal` 525) (c) `data: []` |
| HIST-03-D02 | Business-rule bypass (accepted HIST-D3) | `search: "%"`; `search: "_"` | every paid order of the range (wildcards not escaped) |
| HIST-03-D03 | — (payment mode) | `paymentType: "CASH"`; `"UPI"` | ORDER-PAID only; ORDER-UPI only |
| HIST-03-D04 | Input validation / Error hygiene (accepted HIST-D1) | `orderChannelId`: (a) `0` (b) `2` (c) `"<CH-DINE hex>"` | (a) 500 `{ status: false, message: "Failed to retrieve order history", data: null, pagination: null }` (b) 422 `The selected order channel id is invalid.` (c) 422 `The order channel id field must be an integer.` |
| HIST-03-D05 | Business rule | `startDate` = yesterday, no `endDate` | no date filter: yesterday's and today's paid orders |
| HIST-03-D06 | Injection | `search: "' OR 1=1 --"`; `search: "<script>alert(1)</script>"` | `data: []` both; no error |
| HIST-03-D07 | Over-posting / Restaurant isolation | `restaurantId: "<REST-B hex>"`, `orderStatus: 0` | same as BODY-TODAY alone (REST-A paid orders) |
| HIST-03-D08 | Input validation | `search` of 256 characters | 422 `The search field must not be greater than 255 characters.` |
