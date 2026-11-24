# 03 — Menu in the panel — developer cases

Use case: [03-menu-in-panel.md](../03-menu-in-panel.md). Contract: [api-contract.md → Panel routes](../api-contract.md#panel-routes-web). Fixtures and how to call the panel: [README.md](./README.md#panel-requests-use-case-03).

JSON with ADMIN-1's session, `X-CSRF-TOKEN` and `Accept: application/json` unless stated. "422 `<msg>`" = Laravel's `{ "message": "<msg>", "errors": {…} }`. Cases tagged *Fails until … is fixed* assert the target; today's result is in brackets.

| ID | Category | Request / manipulation | Expected | Database after |
|---|---|---|---|---|
| MENU-03-D01 | — (happy path) | `POST /admin/categories/store` `{ restaurantId: <REST-A>, name: "Soups", isActive: 1 }` | 200 `{ "success": true }` | row |
| MENU-03-D02 | — (happy path) | `POST /admin/categories/update` `{ id: <CAT-S hex>, name: "Starters 2", isActive: 1 }` | 200 | renamed |
| MENU-03-D03 | Business rule — *Fails until MENU-D3 is fixed* | `POST /admin/categories/toggle` `{ id: <CAT-S>, isActive: 0 }` | 200 | category 0; ITEM-1, ITEM-2, ITEM-3 also 0 (today: still 1) |
| MENU-03-D04 | Business rule — *Fails until MENU-D3 is fixed* | `POST /admin/items/store` `{ restaurantId: <REST-A>, categoryId: <CAT-S>, menuItemName: "Samosa", price: 40, isActive: 1 }` | 200 `{ "success": true }` (today: 500, unknown column `menuItemName`) | one row, `name` `Samosa` |
| MENU-03-D05 | Business rule — (a) *Fails until MENU-D3 is fixed* | `POST /admin/items/update` (a) `{ id: <ITEM-1 hex>, menuItemName: "Paneer Tikka 2", price: 260 }` (b) `{ id: <ITEM-1 hex>, price: 270 }` | 200 both (today: (a) 500) | (a) name and price changed (b) price 270.00 |
| MENU-03-D06 | Business rule — *Fails until MENU-D3 is fixed* | `GET /admin/items-management/<REST-A>`; read `Lime Soda`'s `data-id`; post it to `/admin/item-toggle-status` with `isActive: 0` | `data-id` is ITEM-4's 32-char hex; toggle 200 | ITEM-4 `isActive` 0 (today: non-hex `data-id`, nothing changed) |
| MENU-03-D07 | Input validation / Error hygiene — *Fails until MENU-D3 is fixed* | (a) categories/store with `name: ""` (b) `GET /admin/items-management/<32 hex of no restaurant>` (c) items/update with an unknown 32-hex `id` | (a) 422 `The name field is required.` (b)(c) 404 (today: (a)(b) 500 error page, no SQL with `APP_DEBUG=false`; (c) 200) | none |
| MENU-03-D08 | Restaurant isolation (by design) | categories/update with CAT-B's hex from REST-A's page | 200 — the super admin may edit any restaurant | CAT-B changed |
| MENU-03-D09 | Authentication / Super-admin boundary | any POST: no session → 302 / JSON 401; no CSRF → 419; POS bearer only → 302 / 401 | as listed | unchanged |
| MENU-03-D10 | Injection / Over-posting / Rate limit | categories/store `name: "<img src=x onerror=alert(1)>"`, plus `isDelete: 1`; 30 posts/min | 200; the page shows the name escaped; `isDelete` ignored; no 429 | stored; `isDelete` 0 |
| MENU-03-D11 | Input validation / Business rule — *Fails until MENU-D3 is fixed* | items/store in CAT-S with (a) `price: 0` (b) `menuItemName: "paneer tikka"` (c) `categoryId: <CAT-B>` (d) a 256-character name; categories/store `name: "drinks"` | 422 each — (a) price, (b) duplicate name, (c) category, (d) length; `drinks` → duplicate category name (today: items/store 500; categories/store 200) | none |
