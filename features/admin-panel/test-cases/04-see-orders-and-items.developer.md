# 04 — See today's orders and items — developer cases

Use case: [04-see-orders-and-items.md](../04-see-orders-and-items.md). Contract: [api-contract.md](../api-contract.md#restaurant-pages). Fixtures: [README.md](./README.md#fixtures) (SESS).

`GET /admin/orders/<id>` or `/admin/items/<id>` with SESS; `<id>` = REST-A hex.

| ID | Category | Request / manipulation | Expected |
|---|---|---|---|
| ADM-04-D01 | — (happy path) / Restaurant isolation | ORDER-LIVE, ORDER-PAID, ORDER-B-PAID (REST-B) today; `/admin/orders/<REST-A>` | 200; exactly the two REST-A orders, by `created_at` DESC; IDs = `HEX(orderId)`; times = `created_at` (UTC) as `d M Y, h:i A` |
| ADM-04-D02 | Business rule (accepted ADM-D2) | Two REST-A items both named `Paneer Tikka` (different IDs), one line each today; `/admin/items/<REST-A>` | One row `Paneer Tikka` with the summed quantity and amount |
| ADM-04-D03 | Input validation / Error hygiene (accepted ADM-D2) | `/admin/items/<REST-A>?sort_by=quantity&sort_order=` (a) `up` (b) `ASC` (c) `desc;DROP TABLE order_items` (d) `asc` with `sort_by=menuItemName` (e) `sort_by=price` | (a)(c) 500 error page `Server Error` (with `APP_DEBUG=true`: `Order direction must be "asc" or "desc".`) (b) 200 sorted ascending (case-insensitive) (d) 200 alphabetical (e) 200 default order; `order_items` intact |
| ADM-04-D04 | Authentication / Rate limiting / Injection | (a) both pages with no cookie (b) 100 × each in a minute (c) an item named `<img src=x onerror=alert(1)>` sold today | (a) 302 → `/` (b) all 200, no 429 (c) the name shown escaped; no script runs |
