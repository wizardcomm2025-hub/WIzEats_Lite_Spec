# 03 — Add a KOT to a running order — developer cases

Use case: [03-add-kot.md](../03-add-kot.md). Contract: [api-contract.md](../api-contract.md#post-apiorderskotsorderid). Fixtures: [README.md](./README.md#fixtures).

`POST /api/orders/kots/<orderId>`, OWN-A's token. Body "add" = `{ "kotNote": "", "items": [ { "menuItemId": "<ITEM-2>", "menuItemName": "Chicken 65", "quantity": 1, "price": 300 } ] }`. "Not found" (target) = 404 `{ status: false, message: "Order not found", data: null }`. Cases tagged *Fails until … is fixed* assert the target; today's result is in brackets.

| ID | Category | Request / manipulation | Expected | Database after |
|---|---|---|---|---|
| ORD-03-D01 | — (happy path) | ORDER-RUN, add | 201 `KOT added successfully`; `subTotal` raised by 300; new `kotNumber` | new KOT + line (tax at the order's %); `orders.subTotal` updated |
| ORD-03-D02 | Business rule | ORDER-BILLED, add | 400 `{ status: false, message: "Bill already generated, cannot add items to this order" }` | unchanged |
| ORD-03-D03 | Restaurant isolation — *Fails until ORD-D4 is fixed* | ORDER-B (REST-B), add | Not found (today: 400 `Order not found`) | ORDER-B unchanged |
| ORD-03-D04 | Input validation / Injection / Error hygiene — *Fails until ORD-D4 is fixed* | `<orderId>` = `xyz`; = `NaN`; = 32 hex of no order; = dashed form of ORDER-RUN | `xyz` / `NaN` / unknown: Not found, no PHP text (today: `xyz` / `NaN` 400 with the `hex2bin()` text, unknown 400 `Order not found`); dashed: 201 | — |
| ORD-03-D05 | Input validation — (a) *Fails until ORD-D4 is fixed* | ORDER-RUN, (a) add without the `kotNote` key (b) `items: []` | (a) 201, KOT with no note (today: 400 `Undefined array key "kotNote"`) (b) 422 `The items field is required.` | (a) new KOT |
| ORD-03-D06 | Business rule | Change REST-A tax to 12 %; ORDER-RUN, add | 201 | new line `taxPercent` = the order's original 5.00 |
| ORD-03-D07 | Over-posting | add + `tableId`, `orderChannelId`, `orderStatus: 2`, `subTotal: 1` | 201 | only items added; other fields unchanged |
| ORD-03-D08 | Authentication / Account state / Role / Super-admin — chef and WTR-A-OFF parts *Fails until ORD-D4 is fixed* / *Fails until AUTH-D2 is fixed* | no token → 401; panel cookie → 401; `GET` → 405; CHF-A → 403 `{ status: false, message: "You do not have permission to perform this action", data: null }` (today 201); WTR-A-OFF old token → 403 `Your account is inactive.` (today 201); NOREST → 403 | as listed | — |
| ORD-03-D09 | Concurrency | 5 parallel adds on ORDER-RUN | all 201 | subtotal increased by exactly 5 × 300 (row lock); KOT numbers distinct (target ORD-D3; today may repeat) |
| ORD-03-D10 | Rate limiting | 30 adds/min | no 429 | — |
| ORD-03-D11 | Concurrency — *Fails until ORD-D4 is fixed* | ORDER-RUN, add twice with the same `Idempotency-Key` | both 201 with the same body | one new KOT (today: two) |
