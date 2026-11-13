# 02 — Manage menu items — developer cases

Use case: [02-manage-menu-items.md](../02-manage-menu-items.md). Contract: [api-contract.md](../api-contract.md#post-apiitemmenuitemsave). Fixtures: [README.md](./README.md#fixtures).

JSON with OWN-A's bearer token unless stated. "Failed" = 500 `{ status: false, message: "Failed to save menu item", data: null }`.

| ID | Category | Request / manipulation | Expected | Database after |
|---|---|---|---|---|
| MENU-02-D01 | — (happy path) | `getMenuItems` `{}` | 200 `Menu items retrieved successfully`; ITEM-1, ITEM-2, ITEM-4 (not ITEM-3); `price` like `"250.00"`; `pagination` `{ currentPage: 1, perPage: 10, total: 3, lastPage: 1, from: 1, to: 3 }`; ordered by `menuItemId` | — |
| MENU-02-D02 | — (paging) | `getMenuItems` `{ "page": 2, "perPage": 2 }`; then `{ "page": 5, "perPage": 2 }` | 1 item, `from` 3, `to` 3; then `data: []`, `from` 9, `to` 3 (as computed) | — |
| MENU-02-D03 | Input validation (accepted MENU-D2) | `showCategoryMenuItems` `{ "categoryId": "<CAT-S hex>", "page": 1, "perPage": 10 }` (what the app sends) | 200 `data: []`, `total` 0 | — |
| MENU-02-D04 | Input validation / Injection | `showCategoryMenuItems` `{}`; `getMenuItems` `{ "menuItemId": "' OR 1=1 --" }` | first: ITEM-1–4 (inactive ITEM-3 included); second: `data: []` | — |
| MENU-02-D05 | Business rule (accepted MENU-D1) | app payload `{ name, price: 280, description, categoryId: "<CAT-S>", isActive: true, isVeg: true, menuItemId: 0, image: "" }` | 422 `{ status: false, message: "The menu item id field must be a string.", errors: {…} }` | none |
| MENU-02-D06 | — (happy path, API) | same without `menuItemId`, with `image` = IMG-PNG | 200 `Menu item created successfully`; `imageUrl` = `<APP_URL>/storage/image/<uuid>.png`, reachable | row; file exists |
| MENU-02-D07 | Input validation | create with (a) no `categoryId` (b) no `name` (c) no `price` (d) `price: "abc"` (e) `isVeg: "yes"` | 422 (a) `The category id field is required.` (b) `The name field is required.` (c) `The price field is required.` (d) `The price field must be a number.` (e) `The is veg field must be true or false.` | none |
| MENU-02-D08 | Business rule (accepted MENU-D2) | `menuItemSave` `{ "menuItemId": "<ITEM-1 hex>", "price": 260 }`; `{ "menuItemId": "<ITEM-1 hex>", "isActive": false }` | Failed both | unchanged |
| MENU-02-D09 | Input validation | create with `categoryId` = (a) `xyz` (b) 31 hex chars | Failed | none |
| MENU-02-D10 | Restaurant isolation (accepted MENU-D4) | create with `categoryId` = CAT-B hex; then OWN-A `getItemsCategory`, OWN-B `getItemsCategory` | 200 created; the item appears under neither restaurant's categories but in OWN-A's `getMenuItems` | row in REST-A with CAT-B's ID |
| MENU-02-D11 | Input validation | create with (a) a 256-character name (b) `price: 100000000` (c) `price: 12.345` | (a)(b) Failed (c) 200 | (c) `12.35` |
| MENU-02-D12 | Business-rule bypass (accepted MENU-D4) | create with `price: -50` | 200 | price `-50.00`; it shows on the ordering screen |
| MENU-02-D13 | Over-posting | create with `restaurantId: "<REST-B>"`, `version: 9`, `createdBy: "x"`, `imageUrl: "https://evil.example/x.png"` | 200 | row in REST-A; `version` 0; `createdBy` NULL; `imageUrl` NULL (`imageUrl` only from `image`) |
| MENU-02-D14 | Input validation | `getMenuItems` `{ "perPage": 101 }`; `{ "page": 0 }` | 422 `Validation error` both | — |
| MENU-02-D15 | Restaurant isolation | `getMenuItems {}`; `showCategoryMenuItems {}` | no ITEM-B1 | — |
| MENU-02-D16 | Role (accepted MENU-D4) | WTR-A creates an item (API path); CHF-A reads `getMenuItems` | 200 both | row |
| MENU-02-D17 | Authentication / Account state / Super-admin / Rate limit | all three endpoints: no token → 401; `GET` → 405; OWN-P → 200; NOREST → 403; panel cookie only → 401; 60/min → no 429 | as listed | — |
| MENU-02-D18 | Injection | create `name` = `<script>alert(1)</script>`, `description` = `'); DROP TABLE menu_items; --` | 200 | stored exactly; the panel page shows text only |
| MENU-02-D19 | Concurrency | two parallel creates with the same name | both 200 | two rows (no uniqueness) |
| MENU-02-D20 | Error hygiene / Business rule | create with `image` = `data:image/gif;base64,R0lGOD…` | Failed — no `error` key | no row, no file |
