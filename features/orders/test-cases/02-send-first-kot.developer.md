# 02 — Send the first KOT — developer cases

Use case: [02-send-first-kot.md](../02-send-first-kot.md). Contract: [api-contract.md](../api-contract.md#post-apiorderscreateorder). Fixtures: [README.md](./README.md#fixtures).

`POST /api/orders/createOrder`, JSON, OWN-A's token unless stated. "Created" = 201 `Order created successfully` with `data { orderId, orderNumber, kotId, kotNumber, subTotal }`. "500-text" = 500 `{ status: false, message: "Something went wrong", error: "<exception text>" }`.

| ID | Category | Request / manipulation | Expected | Database after |
|---|---|---|---|---|
| ORD-02-D01 | — (happy path) | basic | Created; `orderNumber` `CODE-A/1` (first today); `kotNumber` 1; `subTotal` 500 | order status 0, tax % 5.00, subtotal 500.00, grandTotal 0.00; 1 KOT (status 0); 1 line: price 250.00, qty 2, total 500.00, tax 25.00 |
| ORD-02-D02 | Business rule (accepted ORD-D2) | basic with `orderChannelId: CH-DINE`, `tableId: "<Table 1 hex>"`; then `showDetails` | Created | order `tableId` = Table 1; **Table 1 `status` unchanged** (update matched nothing); `showDetails` gives Table 1 this `orderId` |
| ORD-02-D03 | Business rule (accepted ORD-D2) | what the app sends: `tableId: null` (from `Number(hex)`) on CH-DINE | Created | `tableId` NULL; Table 1 shows no order |
| ORD-02-D04 | Business rule (accepted ORD-D3) | (a) basic + `customerName: "Ravi"`, `customerPhone: "9876543210"` (b) `orderChannelId: "2"` with no customer fields (c) `orderChannelId: "2"` with both | (a) Created (b) 422 `Customer name is required for room-service orders` (c) 500-text (`hex2bin` odd length) | (a) customer fields NULL |
| ORD-02-D05 | Business-rule bypass (accepted ORD-D1) | items `price: 0`; then `price: 1` with `menuItemName: "Free Lobster"` | Created both | lines at 0.00 and 1.00 with the sent names |
| ORD-02-D06 | Business-rule bypass (accepted ORD-D1) | `menuItemId` = ITEM-3 (inactive); = ITEM-B1 (REST-B); = 32 hex of no item | Created all | lines with those IDs |
| ORD-02-D07 | Business-rule bypass | `quantity: 100000`, `price: 99999` (total 9 999 900 000) | 500-text (out of range) | nothing (rolled back) |
| ORD-02-D08 | Input validation | (a) no `items` (b) `items: []` (c) `quantity: 0` (d) `price: -1` (e) no `orderChannelId` (f) `quantity: 1.5` | 422: (a)(b) `The items field is required.` (c) `The items.0.quantity field must be at least 1.` (d) `The items.0.price field must be at least 0.` (e) `The order channel id field is required.` (f) `The items.0.quantity field must be an integer.` | none |
| ORD-02-D09 | Input validation / Error hygiene (accepted ORD-D4) | `orderChannelId: "1"` (what the app sends with no channel); `menuItemId: "xyz"` | 500-text with the `hex2bin()` message in `error` | none |
| ORD-02-D10 | Restaurant isolation / Error hygiene | `orderChannelId` = CH-B (REST-B's channel) | Created | order in REST-A with REST-B's channel ID |
| ORD-02-D11 | Restaurant isolation | `tableId` = a REST-B table | Created | order in REST-A pointing at REST-B's table; REST-B's table unchanged |
| ORD-02-D12 | Role (accepted ORD-D4) | CHF-A sends basic | Created; `createdBy` `chef` | — |
| ORD-02-D13 | Business-rule bypass (accepted ORD-D4) | two orders on Table 1 (D02 twice) | Created both | two running orders with `tableId` Table 1; `showDetails` shows the higher `orderId` |
| ORD-02-D14 | Over-posting | basic + `orderStatus: 2`, `restaurantId: "<REST-B>"`, `grandTotal: 1`, `orderNumber: "X"`, `taxPercent: 0` | Created | status 0, REST-A, own number, tax 5.00, grandTotal 0.00 |
| ORD-02-D15 | Concurrency (accepted ORD-D3) | 10 parallel basic orders | all Created | record the `orderNumber`s and `kotNumber`s — duplicates possible |
| ORD-02-D16 | Authentication / Super-admin | no token → 401; panel cookie only → 401; `GET` → 405 | as listed | none |
| ORD-02-D17 | Account state (AUTH-D2) | OWN-P (Pending) → Created; WTR-A-OFF old token → Created; NOREST → 403 | as listed | — |
| ORD-02-D18 | Concurrency (accepted ORD-D4) | the same basic twice with the same `Idempotency-Key` header | Created twice | two orders |
| ORD-02-D19 | Rate limiting | 30 orders in a minute | all Created; no 429 | 30 orders |
| ORD-02-D20 | Injection | `kotNote` / `menuItemName` = `'); DROP TABLE orders; --` and `<script>alert(1)</script>` | Created | stored exactly; tables intact |
