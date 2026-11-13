# 03 — Menu in the panel — developer cases

Use case: [03-menu-in-panel.md](../03-menu-in-panel.md). Contract: [api-contract.md → Panel routes](../api-contract.md#panel-routes-web). Fixtures and how to call the panel: [README.md](./README.md#panel-requests-use-case-03).

JSON with ADMIN-1's session and `X-CSRF-TOKEN` unless stated.

| ID | Category | Request / manipulation | Expected | Database after |
|---|---|---|---|---|
| MENU-03-D01 | — (happy path) | `POST /admin/categories/store` `{ restaurantId: <REST-A>, name: "Soups", isActive: 1 }` | 200 `{ "success": true }` | row |
| MENU-03-D02 | — (happy path) | `POST /admin/categories/update` `{ id: <CAT-S hex>, name: "Starters 2", isActive: 1 }` | 200 | renamed |
| MENU-03-D03 | Business rule | `POST /admin/categories/toggle` `{ id: <CAT-S>, isActive: 0 }` | 200 | category 0; ITEM-1/2 still 1 |
| MENU-03-D04 | Business rule (accepted MENU-D3) | `POST /admin/items/store` `{ restaurantId: <REST-A>, categoryId: <CAT-S>, menuItemName: "X", price: 10, isActive: 1 }` | 500 (unknown column `menuItemName`) | none |
| MENU-03-D05 | Business rule (accepted MENU-D3) | `POST /admin/items/update` (a) `{ id: <ITEM-1 hex>, menuItemName: "Y", price: 260 }` (b) `{ id: <ITEM-1 hex>, price: 260 }` | (a) 500 (b) 200 | (a) unchanged (b) price 260.00 |
| MENU-03-D06 | Business rule (accepted MENU-D3) | `GET /admin/items-management/<REST-A>`; read a row's `data-id`; post it to `/admin/item-toggle-status` with `isActive: 0` | page has non-hex `data-id`; toggle 200 `{ "success": true }` | nothing changed. (With ITEM-1's real hex instead: item switched off.) |
| MENU-03-D07 | Input validation / Error hygiene | categories/store with `name: ""`; `GET /admin/items-management/<32 hex of no restaurant>` | 500 each; no SQL with `APP_DEBUG=false` | none |
| MENU-03-D08 | Restaurant isolation (by design) | categories/update with CAT-B's hex from REST-A's page | 200 — the super admin may edit any restaurant | CAT-B changed |
| MENU-03-D09 | Authentication / Super-admin boundary | any POST: no session → 302 / JSON 401; no CSRF → 419; POS bearer only → 302 / 401 | as listed | unchanged |
| MENU-03-D10 | Injection / Over-posting / Rate limit | categories/store `name: "<img src=x onerror=alert(1)>"`, plus `isDelete: 1`; 30 posts/min | 200; the page shows the name escaped; `isDelete` ignored; no 429 | stored; `isDelete` 0 |
