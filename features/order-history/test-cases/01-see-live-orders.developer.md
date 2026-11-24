# 01 — See live orders — developer cases

Use case: [01-see-live-orders.md](../01-see-live-orders.md). Contract: [api-contract.md](../api-contract.md#post-apiordersorderhistorylive). Fixtures: [README.md](./README.md#fixtures).

`POST /api/orders/orderHistoryLive`, body BODY-TODAY, OWN-A's token unless stated. "Live-OK" = 200 `{ status: true, message: "Live order history retrieved successfully", data: [...] }`. Cases tagged *Fails until … is fixed* assert the target; today's result is in brackets.

| ID | Category | Request / manipulation | Expected |
|---|---|---|---|
| HIST-01-D01 | — (happy path) | ORDER-LIVE, ORDER-LIVE-BILLED, ORDER-PAID exist | Live-OK with ORDER-LIVE (`orderStatus` 0) and ORDER-LIVE-BILLED (1), newest first, not ORDER-PAID; each row: hex `orderId`, `orderChannel` `Takeaway`, `paymentStatus` `Unpaid`, `menuItems` `[ { menuItemName: "Paneer Tikka", quantity: 2, price: "250.00", … } ]`; no `bill`, `kots` or `pagination` (target also `orderDateTime` — HIST-D2) |
| HIST-01-D02 | Business rule — *Fails until HIST-D2 is fixed* | ORDER-EARLY and ORDER-OLD; BODY-TODAY, then dates = last week only | both listed every time, with `orderDateTime` (today: under `Today` neither; under yesterday's date both) |
| HIST-01-D03 | Business-rule bypass / Over-posting — *Fails until HIST-D2 is fixed* | BODY-TODAY with (a) `search: "zzz"` (b) `paymentType: "UPI"`, `perPage: 1`, `page: 5`, `restaurantId: "<REST-B hex>"` | (a) `data: []` (today: same rows as D01) (b) same rows as D01 — payment mode and paging don't apply on Live |
| HIST-01-D04 | Restaurant isolation | ORDER-B-RUN exists | Not in REST-A's list |
| HIST-01-D05 | Business rule (HIST-D1 fixed) / Input validation | (a) `channelCode: "TAKEAWAY"` (b) `"DINE_IN"` (c) `"takeaway"` (d) `"FOO"` (e) the old `orderChannelId: 3` | (a)(c) Live-OK with ORDER-LIVE and ORDER-LIVE-BILLED only (b) ORDER-DINE only (d) 422 `The selected channel code is invalid.` (e) ignored — same rows as D01 |
| HIST-01-D06 | Input validation | (a) `endDate` before `startDate` (b) `startDate: "abc"` (c) `perPage: 101` (d) `paymentType: "CARD"` | 422: (a) `The end date field must be a date after or equal to start date.` (b) `The start date field must be a valid date.` (c) `The per page field must not be greater than 100.` (d) `The selected payment type is invalid.` |
| HIST-01-D07 | Role — *Fails until HIST-D3 is fixed* | CHF-A; WTR-A | CHF-A: 403 `{ status: false, message: "You do not have permission to perform this action", data: null }` (today Live-OK); WTR-A: Live-OK |
| HIST-01-D08 | Authentication / Account state / Super-admin — switched-off part *Fails until AUTH-D2 is fixed* | no token → 401; panel cookie only → 401; `GET` → 405; NOREST → 403; WTR-A-OFF old token → 403 `Your account is inactive.` (today Live-OK) | as listed |
| HIST-01-D09 | Injection / Rate limiting | `search: "' OR 1=1 --"`; then 60 calls in a minute | `data: []` for the injection (target; today ignored — same rows as D01); no error; no 429 |
| HIST-01-D10 | Business rule — *Fails until HIST-D2 is fixed* | ORDER-LIVE number noted (e.g. `CODE-A/8`); `search: "8"`; then `search: "500"` (no running order's grand total is 500 until billed) | `#8` match: ORDER-LIVE listed; `500`: only orders whose number contains `500` (today: same rows as D01 both times) |
