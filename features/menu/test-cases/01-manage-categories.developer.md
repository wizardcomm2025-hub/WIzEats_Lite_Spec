# 01 — Manage categories — developer cases

Use case: [01-manage-categories.md](../01-manage-categories.md). Contract: [api-contract.md](../api-contract.md#post-apiitemcategorysave). Fixtures: [README.md](./README.md#fixtures).

JSON with OWN-A's bearer token unless stated. "Failed" = 500 `{ status: false, message: "Failed to save item category", data: null }`.

| ID | Category | Request / manipulation | Expected | Database after |
|---|---|---|---|---|
| MENU-01-D01 | — (happy path) | `getItemsCategory` `{}` | 200 `Items Category retrieved successfully`; `Starters` with `items` ITEM-1, ITEM-2, ITEM-3 (inactive included); `Drinks` with ITEM-4; hex IDs | — |
| MENU-01-D02 | — (happy path) | `categorySave` `{ "itemCategoryName": "Desserts", "itemCategoryDescription": "Sweet" }` | 200 `Item category created successfully`; `isActive` true | one row in REST-A |
| MENU-01-D03 | Business rule (accepted MENU-D2) | `categorySave` `{ "itemCategoryId": "<CAT-S hex>", "itemCategoryName": "X" }` (also dashed) | Failed | unchanged |
| MENU-01-D04 | Business rule (accepted MENU-D2) | `categorySave` `{ "itemCategoryId": "<CAT-S hex>", "isActive": false }` | Failed | category and its items unchanged |
| MENU-01-D05 | Input validation | (a) `{}` (b) `{ "itemCategoryName": "" }` (c) `{ "itemCategoryName": 5 }` (d) `{ "itemCategoryName": "X", "isActive": "yes" }` | 422 (a)(b) `The item category name field is required.` (c) `The item category name field must be a string.` (d) `The is active field must be true or false.` | none |
| MENU-01-D06 | Input validation / Error hygiene | create with a 256-character name | Failed — no `error` key, no SQL | none |
| MENU-01-D07 | Over-posting / Injection | create with `restaurantId: "<REST-B>"`, `createdBy: "x"`, name `'); DROP TABLE item_categories; --` | 200 | row in REST-A; `createdBy` NULL; name stored exactly |
| MENU-01-D08 | Role (accepted MENU-D4) | WTR-A creates `Waiter Cat`; CHF-A lists | 200 both | row created |
| MENU-01-D09 | Restaurant isolation | (a) `getItemsCategory {}` (b) `categorySave` update with CAT-B's hex | (a) no CAT-B (b) Failed | CAT-B unchanged |
| MENU-01-D10 | Authentication | both endpoints: no token / `Bearer garbage` → 401; `GET` → 405 | as listed | — |
| MENU-01-D11 | Account state (AUTH-D2) | OWN-P creates a category; NOREST lists | 200; 403 `Restaurant context missing` | row in REST-P |
| MENU-01-D12 | Input validation / Injection | `getItemsCategory` `itemCategoryId` = (a) CAT-S hex (b) `' OR 1=1 --` (c) `[1]` | (a)(b) 200 `data: []` (c) 422 `Validation error` | — |
| MENU-01-D13 | Super-admin boundary | panel cookie only | 401 | — |
| MENU-01-D14 | Rate limiting | 60 calls in a minute | no 429 | — |
