# Menu — API contract

Shared shapes: [project-overview.md → API conventions](../../project-overview.md#api-conventions). Facts from the code on 2026-11-10.

## Endpoints

| Endpoint | Middleware | Rate limit | Used by | Use case |
|---|---|---|---|---|
| `POST /api/item/getItemsCategory` | `auth:sanctum`, `restaurant.tenant` | none | App (categories, item pickers) | [01](./01-manage-categories.md) |
| `POST /api/item/categorySave` | same | none | App | [01](./01-manage-categories.md) |
| `POST /api/item/showCategoryMenuItems` | same | none | App (Item Management) | [02](./02-manage-menu-items.md) |
| `POST /api/item/getMenuItems` | same | none | App (ordering menu) | [02](./02-manage-menu-items.md) |
| `POST /api/item/menuItemSave` | same | none | App | [02](./02-manage-menu-items.md) |
| `POST /api/image/upload` | same | none | — | [04](./04-upload-image.md) |
| Panel routes | `web`, `auth:admin` | none | Panel | [03](./03-menu-in-panel.md) |

No role check anywhere (MENU-D4). The common API failures: no or invalid token → 401 `{ "message": "Unauthenticated." }`; no `restaurantId` → 403 `Restaurant context missing`; `GET` → 405. A switched-off user and a Pending restaurant still get through (AUTH-D2).

---

## `POST /api/item/getItemsCategory`

| Field | Rule |
|---|---|
| `itemCategoryId` | optional string; compared as sent, so a hex ID returns `[]` (MENU-D2) |

200 `{ "status": true, "message": "Items Category retrieved successfully", "data": [ { itemCategoryId, restaurantId, itemCategoryName, itemCategoryDescription, isActive, createdBy, isDelete, created_at, updated_at, items: [ …menu items ] } ] }`. A field that isn't a string → 422 `{ "status": false, "message": "Validation error", "errors": {…} }`. Database error → 500 `Failed to retrieve tables` (sic).

## `POST /api/item/categorySave`

| Field | Create (no `itemCategoryId`) | Update |
|---|---|---|
| `itemCategoryId` | — | string — compared as sent (MENU-D2) |
| `itemCategoryName` | required string | sometimes, string |
| `itemCategoryDescription` | nullable string | nullable string |
| `isActive`, `isDelete` | nullable boolean (defaults true / false) | nullable boolean; `isActive` cascades to the items |

| Situation | HTTP | Body |
|---|---|---|
| Created | 200 | `{ "status": true, "message": "Item category created successfully", "data": { …category } }` |
| Any update by hex ID; a name over 255 characters | 500 | `{ "status": false, "message": "Failed to save item category", "data": null }` |
| Validation | 422 | `The item category name field is required.` · `… must be a string.` · `The is active field must be true or false.` |

## `POST /api/item/getMenuItems` and `/showCategoryMenuItems`

| Field | Rule |
|---|---|
| `menuItemId`, `categoryId` | optional strings; compared as sent, so hex IDs return `[]` (MENU-D2) |
| `page` | optional integer ≥ 1 (default 1) |
| `perPage` | optional integer 1–100 (default 10) |

| Endpoint | Items |
|---|---|
| `getMenuItems` | active **and** not deleted |
| `showCategoryMenuItems` | not deleted (active and inactive) |

200 `{ "status": true, "message": "Menu items retrieved successfully", "data": [ { menuItemId, restaurantId, categoryId, name, description, price: "250.00", isVeg, isActive, imageUrl, version, createdBy, isDelete, created_at, updated_at } ], "pagination": { currentPage, perPage, total, lastPage, from, to } }`. The items are ordered by `menuItemId`. Validation failure → 422 `{ "status": false, "message": "Validation error", "errors": {…} }` (for example `perPage: 101`).

## `POST /api/item/menuItemSave`

The update rules apply whenever `menuItemId` is **filled**, and `0` counts as filled (MENU-D1).

| Field | Create (`menuItemId` absent / null / "") | Update (`menuItemId` filled) |
|---|---|---|
| `menuItemId` | — | nullable **string** (`0` → 422) |
| `categoryId` | required string (32-char hex; any category — MENU-D4) | sometimes, string |
| `name` | required string | sometimes, string |
| `price` | required numeric | sometimes, numeric |
| `description` | nullable string | nullable string |
| `isActive`, `isVeg`, `isDelete` | nullable boolean (defaults true / **false** / false) | nullable boolean |
| `image` | nullable string — `data:image/png|jpg|jpeg;base64,…` | same; replaces `imageUrl` |

| Situation | HTTP | Body |
|---|---|---|
| Created | 200 | `{ "status": true, "message": "Menu item created successfully", "data": { …item } }` |
| Validation | 422 | first message, e.g. `The menu item id field must be a string.`, `The category id field is required.`, `The name field is required.`, `The price field is required.`, `The price field must be a number.` |
| Any update by hex ID; `categoryId` not hex; a bad image; a name over 255 characters; a price out of range | 500 | `{ "status": false, "message": "Failed to save menu item", "data": null }` |

## `POST /api/image/upload`

| Field | Rule |
|---|---|
| `image` | required string |

| Situation | HTTP | Body |
|---|---|---|
| Saved | 201 | `{ "status": true, "message": "Image uploaded successfully", "data": { "status": true, "message": "Image uploaded successfully", "imagePath": "storage/image/<uuid>.<type>" } }` |
| Bad prefix, type or base64 | 201 | `{ "status": true, "message": "Image uploaded successfully", "data": { "status": false, "message": "Failed to upload image:", "code": 500 } }` (MENU-D4) |
| Missing / not a string | 422 | `The image field is required.` / `The image field must be a string.` |

---

## Panel routes (web)

| Route | Body | Result |
|---|---|---|
| `GET /admin/categories-management/{restaurantId}` | — | Page; 10 per page; hex IDs |
| `POST /admin/categories/store` | `restaurantId, name, isActive` | Insert; `{ "success": true }`; empty name → 500 |
| `POST /admin/categories/update` | `id, name, isActive` | Update; no cascade |
| `POST /admin/categories/toggle` | `id, isActive` | Update `isActive`; no cascade |
| `GET /admin/items-management/{restaurantId}` | — | Page; 10 per page; raw binary IDs; unknown restaurant → 500 |
| `POST /admin/items/store` | `restaurantId, categoryId, menuItemName, price, isActive` | Always 500 (column `menuItemName` doesn't exist — MENU-D3) |
| `POST /admin/items/update` | `id, menuItemName?, price?, isActive?, categoryId?` | 500 when `menuItemName` is sent (the page always sends it); without it, `price` / `isActive` / `categoryId` update by `id` |
| `POST /admin/item-toggle-status` | `id, isActive` | Updates by `id`; with the page's mangled IDs it matches nothing and still answers `{ "success": true }` |

Common failures: no session → 302 to `/` (JSON 401); bad CSRF → 419; a POS bearer token counts as no session.
