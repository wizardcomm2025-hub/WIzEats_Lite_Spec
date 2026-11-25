# 04 — See orders and items — developer cases

Use case: [04-see-orders-and-items.md](../04-see-orders-and-items.md). Contract: [api-contract.md](../api-contract.md#restaurant-pages). Fixtures: [README.md](./README.md#fixtures) (SESS).

`GET /admin/orders/<id>` or `/admin/items/<id>` with SESS; `<id>` = REST-A hex.

| ID | Category | Request / manipulation | Expected |
|---|---|---|---|
| ADM-04-D01 | — (happy path) / Restaurant isolation — *Fails until ADM-D2 is fixed* | ORDER-LIVE, ORDER-PAID, ORDER-B-PAID (REST-B) today; `/admin/orders/<REST-A>` | 200; exactly the two REST-A orders, by `created_at` DESC; `Order No.` = `orderNumber`; IDs = `HEX(orderId)`; times = `created_at` converted to IST, as `d M Y, h:i A` (today: no order number; UTC times) |
| ADM-04-D02 | Business rule — *Fails until ADM-D2 is fixed* | Two REST-A items both named `Paneer Tikka` (different IDs), one line each in paid orders today, plus ORDER-LIVE; `/admin/items/<REST-A>` | One row `Paneer Tikka` with the summed quantity and amount of the two paid lines only (same-name merge accepted) (today: ORDER-LIVE's lines added too) |
| ADM-04-D03 | Input validation / Error hygiene — *Fails until ADM-D2 is fixed* | `/admin/items/<REST-A>?sort_by=quantity&sort_order=` (a) `up` (b) `ASC` (c) `desc;DROP TABLE order_items` (d) `asc` with `sort_by=menuItemName` (e) `sort_by=price` | (a)(c) 422 page `The selected sort order is invalid.` (e) 422 `The selected sort by is invalid.` (b) 200 sorted ascending (d) 200 alphabetical; `order_items` intact (today: (a)(c) 500 `Server Error` — with `APP_DEBUG=true` `Order direction must be "asc" or "desc".` — (e) 200 default order) |
| ADM-04-D04 | Authentication / Rate limiting / Injection | (a) both pages with no cookie (b) 100 × each in a minute (c) an item named `<img src=x onerror=alert(1)>` sold today | (a) 302 → `/` (b) all 200, no 429 (c) the name shown escaped; no script runs |
| ADM-04-D05 | Input validation — *Fails until ADM-D2 is fixed* | ORDERS-30 and ORDER-OLD; `/admin/orders/<REST-A>` with (a) no query (b) `?page=2` (c) `?filter=yesterday` (d) `?page=abc` (e) `?page=9` | (a) 25 newest of today's 30 (b) the other 5 (c) ORDER-OLD only (d) page 1 (e) an empty table with the footer (today: the query is ignored; all 30 every time) |
