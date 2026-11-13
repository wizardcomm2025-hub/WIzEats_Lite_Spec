# 01 — See live orders — developer cases

Use case: [01-see-live-orders.md](../01-see-live-orders.md). Contract: [api-contract.md](../api-contract.md#post-apiordersorderhistorylive). Fixtures: [README.md](./README.md#fixtures).

`POST /api/orders/orderHistoryLive`, body BODY-TODAY, OWN-A's token unless stated. "Live-OK" = 200 `{ status: true, message: "Live order history retrieved successfully", data: [...] }`.

| ID | Category | Request / manipulation | Expected |
|---|---|---|---|
| HIST-01-D01 | — (happy path) | ORDER-LIVE, ORDER-LIVE-BILLED, ORDER-PAID exist | Live-OK with ORDER-LIVE (`orderStatus` 0) and ORDER-LIVE-BILLED (1), not ORDER-PAID; each row: hex `orderId`, `orderChannel` `Takeaway`, `paymentStatus` `Unpaid`, `menuItems` `[ { menuItemName: "Paneer Tikka", quantity: 2, price: "250.00", … } ]`; no `orderDateTime`, `bill`, `kots` or `pagination` |
| HIST-01-D02 | Business rule (accepted HIST-D2) | ORDER-EARLY and ORDER-OLD; BODY-TODAY, then dates = yesterday | Today: neither; yesterday: both |
| HIST-01-D03 | Business-rule bypass / Over-posting (accepted HIST-D2) | BODY-TODAY with `search: "zzz"`, `paymentType: "UPI"`, `perPage: 1`, `page: 5`, `restaurantId: "<REST-B hex>"` | Same rows as D01 |
| HIST-01-D04 | Restaurant isolation | ORDER-B-RUN exists | Not in REST-A's list |
| HIST-01-D05 | Input validation / Error hygiene (accepted HIST-D1) | `orderChannelId`: (a) `0` (b) `1` (c) `3` (d) `"<CH-TAKE hex>"` | (a) 500 `{ status: false, message: "Failed to retrieve live order history", data: null }` (b)(c) 422 `The selected order channel id is invalid.` (d) 422 `The order channel id field must be an integer.` |
| HIST-01-D06 | Input validation | (a) `endDate` before `startDate` (b) `startDate: "abc"` (c) `perPage: 101` (d) `paymentType: "CARD"` | 422: (a) `The end date field must be a date after or equal to start date.` (b) `The start date field must be a valid date.` (c) `The per page field must not be greater than 100.` (d) `The selected payment type is invalid.` |
| HIST-01-D07 | Role (accepted HIST-D3) | CHF-A | Live-OK, same rows as D01 |
| HIST-01-D08 | Authentication / Account state / Super-admin | no token → 401; panel cookie only → 401; `GET` → 405; NOREST → 403; WTR-A-OFF old token → Live-OK (AUTH-D2) | as listed |
| HIST-01-D09 | Injection / Rate limiting | `search: "' OR 1=1 --"` (ignored); then 60 calls in a minute | Same rows as D01; no 429 |
