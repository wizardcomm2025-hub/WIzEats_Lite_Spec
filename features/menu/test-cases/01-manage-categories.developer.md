# 01 — Manage categories — developer cases

Use case: [01-manage-categories.md](../01-manage-categories.md). Contract: [api-contract.md](../api-contract.md#post-apiitemcategorysave). Fixtures: [README.md](./README.md#fixtures).

JSON with OWN-A's bearer token unless stated. "Failed" = 500 `{ status: false, message: "Failed to save item category", data: null }`. "Not found" = 404 `{ status: false, message: "Category not found", data: null }`. Cases tagged *Fails until … is fixed* assert the target; today's result is in brackets.

| ID | Category | Request / manipulation | Expected | Database after |
|---|---|---|---|---|
| MENU-01-D01 | — (happy path) | `getItemsCategory` `{}` | 200 `Items Category retrieved successfully`; `Starters` with `items` ITEM-1, ITEM-2, ITEM-3 (inactive included); `Drinks` with ITEM-4; no `Specials` (deleted); hex IDs | — |
| MENU-01-D02 | — (happy path) | `categorySave` `{ "itemCategoryName": "  Desserts ", "itemCategoryDescription": "Sweet" }` | 200 `Item category created successfully`; `isActive` true; name `Desserts` | one row in REST-A, name trimmed |
| MENU-01-D03 | — (happy path; MENU-D2 fixed) | `categorySave` `{ "itemCategoryId": "<CAT-S hex>", "itemCategoryName": "Starters & Snacks" }`; then `{ "itemCategoryId": "<CAT-S hex>", "itemCategoryName": "starters & snacks" }` | 200 `Item category updated successfully` both — a case change of its own name is not a duplicate | name `starters & snacks`; description unchanged (not sent) |
| MENU-01-D04 | Business rule (MENU-D2 fixed) | `categorySave` `{ "itemCategoryId": "<CAT-S hex>", "isActive": false }`; then `getMenuItems {}`; then `true` | 200 both; while off `getMenuItems` has no ITEM-1 / ITEM-2 | off: category and ITEM-1, ITEM-2, ITEM-3 `isActive` 0; on: all 1 (ITEM-3 too) |
| MENU-01-D05 | Input validation | (a) `{}` (b) `{ "itemCategoryName": "" }` (c) `{ "itemCategoryName": 5 }` (d) `{ "itemCategoryName": "X", "isActive": "yes" }` | 422 (a)(b) `The item category name field is required.` (c) `The item category name field must be a string.` (d) `The is active field must be true or false.` | none |
| MENU-01-D06 | Input validation / Error hygiene — *Fails until MENU-D4 is fixed* | create with a 256-character name | 422 `The item category name field must not be greater than 255 characters.` (today: Failed 500 — no `error` key, no SQL) | none |
| MENU-01-D07 | Over-posting / Injection | create with `restaurantId: "<REST-B>"`, `createdBy: "x"`, name `'); DROP TABLE item_categories; --` | 200 | row in REST-A; `createdBy` NULL; name stored exactly |
| MENU-01-D08 | Role — *Fails until MENU-D4 is fixed* | (a) WTR-A creates `Waiter Cat` (b) WTR-A lists (c) CHF-A lists | (a)(c) 403 `{ status: false, message: "You do not have permission to perform this action", data: null }` (b) 200 (today: all 200, row created) | no row |
| MENU-01-D09 | Restaurant isolation — (b) *Fails until MENU-D4 is fixed* | (a) `getItemsCategory {}` (b) `categorySave` update with CAT-B's hex | (a) no CAT-B (b) Not found (today: Failed 500) | CAT-B unchanged |
| MENU-01-D10 | Authentication | both endpoints: no token / `Bearer garbage` → 401; `GET` → 405 | as listed | — |
| MENU-01-D11 | Account state — OWN-P part *Fails until AUTH-D2 is fixed* | OWN-P creates a category; NOREST lists | OWN-P: 403 `Your restaurant is not approved.` (today 200, row in REST-P); NOREST: 403 `Restaurant context missing` | no row |
| MENU-01-D12 | Input validation / Injection — *Fails until MENU-D4 is fixed* | `getItemsCategory` `itemCategoryId` = (a) CAT-S hex (b) CAT-S dashed (c) `' OR 1=1 --` (d) `[1]` | (a)(b) 200, [Starters] with `items` ITEM-1, ITEM-2, ITEM-3 — **not** ITEM-DEL (c)(d) 422 `Validation error` (today: (a)–(c) 200 `data: []`; (d) 422; ITEM-DEL included in `{}`) | — |
| MENU-01-D13 | Super-admin boundary | panel cookie only | 401 | — |
| MENU-01-D14 | Rate limiting | 60 calls in a minute | no 429 | — |
| MENU-01-D15 | Business rule — (d)–(f) *Fails until MENU-D4 is fixed* | (a) create `starters` (b) rename CAT-D to ` STARTERS ` (c) create `Specials` (CAT-X is deleted) (d) rename CAT-S with its dashed ID (e) update with `itemCategoryId` = `xyz` (f) update with 32 hex of no category | (a)(b) 422 `{ status: false, message: "Category name already exists", data: null }` (c) 200 (today: 422 `Category name already exists`) (d) 200 `Item category updated successfully` (today: Failed 500) (e)(f) Not found (today: Failed 500) | (a)(b) unchanged (c) one new row |
| MENU-01-D16 | Business rule (MENU-D6 context) | `categorySave` `{ "itemCategoryId": "<CAT-S hex>", "itemCategoryName": "Starters", "itemCategoryDescription": "" }` (today's app edit body) | 200 | description NULL — a field sent empty is cleared by design; the fix is in the app (01-T09) |
