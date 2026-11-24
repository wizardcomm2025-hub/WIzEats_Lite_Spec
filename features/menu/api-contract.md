# Menu — API contract

Shared shapes: [project-overview.md → API conventions](../../project-overview.md#api-conventions). Facts from the code on 2026-11-10, re-checked 2026-11-24 (API commit d54d6c8; `ItemService` changed 2026-11-05 and 2026-11-18). Rows marked **Target** describe agreed fixes and are not in the code yet.

## Endpoints

| Endpoint | Middleware | Rate limit | Used by | Roles — target (X-D1 / MENU-D4) | Use case |
|---|---|---|---|---|---|
| `POST /api/item/getItemsCategory` | `auth:sanctum`, `restaurant.tenant` | none | Category Management, Item Management picker, ordering screen | owner, waiter | [01](./01-manage-categories.md) |
| `POST /api/item/categorySave` | same | none | Category Management | owner | [01](./01-manage-categories.md) |
| `POST /api/item/showCategoryMenuItems` | same | none | Item Management | owner | [02](./02-manage-menu-items.md) |
| `POST /api/item/getMenuItems` | same | none | Ordering screen (full menu) | owner, waiter | [02](./02-manage-menu-items.md) |
| `POST /api/item/menuItemSave` | same | none | Item Management | owner | [02](./02-manage-menu-items.md) |
| `POST /api/image/upload` | same | none | — | owner | [04](./04-upload-image.md) |
| Panel routes | `web`, `auth:admin` | none | Panel | super admin | [03](./03-menu-in-panel.md) |

Today no role check anywhere (MENU-D4). **Target:** roles as above, otherwise 403 `{ "status": false, "message": "You do not have permission to perform this action", "data": null }`. The common API failures: no or invalid token → 401 `{ "message": "Unauthenticated." }`; no `restaurantId` → 403 `Restaurant context missing`; `GET` → 405. Today a switched-off user and a Pending restaurant still get through (AUTH-D2); **target (X-D2)** 403 `Your account is inactive.` / `Your restaurant is not approved.` / `Your restaurant is inactive.`

Service refusals (the 422s and 404s below that are not Laravel validation) use `{ "status": false, "message": "<text>", "data": null }`. Laravel validation uses `{ "status": false, "message": "<first error>", "errors": {…} }`.

---

## `POST /api/item/getItemsCategory`

| Field | Rule |
|---|---|
| `itemCategoryId` | optional string; today compared as sent, so a hex ID returns `[]`. **Target (MENU-D4):** 32-char hex (dashes allowed) → that category; other values → 422 `Validation error` |

200 `{ "status": true, "message": "Items Category retrieved successfully", "data": [ { itemCategoryId, restaurantId, itemCategoryName, itemCategoryDescription, isActive, createdBy, isDelete, created_at, updated_at, items: [ …menu items ] } ] }` — not-deleted categories, active and inactive. `items`: today all the category's items, inactive **and deleted**; **target** not deleted. A field that isn't a string → 422 `{ "status": false, "message": "Validation error", "errors": {…} }`. Database error → 500 `Failed to retrieve tables` (sic).

## `POST /api/item/categorySave`

| Field | Create (no `itemCategoryId`) | Update |
|---|---|---|
| `itemCategoryId` | — | string — 32-char hex (since 2026-11-18). Dashes not accepted today (500); **target** allowed |
| `itemCategoryName` | required string — trimmed; unique per restaurant, any case | sometimes, string — same check, excluding itself |
| `itemCategoryDescription` | nullable string | nullable string (`""` / `null` clears it) |
| `isActive`, `isDelete` | nullable boolean (defaults true / false) | nullable boolean; `isActive` also sets all the category's items |

| Situation | HTTP | Body |
|---|---|---|
| Created | 200 | `{ "status": true, "message": "Item category created successfully", "data": { …category } }` |
| Updated (since 2026-11-18) | 200 | `{ "status": true, "message": "Item category updated successfully", "data": { …category } }` |
| Name already used by another category of the restaurant (any case, trimmed) | 422 | `{ "status": false, "message": "Category name already exists", "data": null }` — today deleted categories count too; **target** they don't |
| Unknown / other restaurant's / malformed `itemCategoryId`; a name over 255 characters | 500 | `{ "status": false, "message": "Failed to save item category", "data": null }` |
| **Target (MENU-D4)** unknown / deleted / other restaurant's / malformed `itemCategoryId` | 404 | `{ "status": false, "message": "Category not found", "data": null }` |
| **Target (MENU-D4)** name over 255 | 422 | `The item category name field must not be greater than 255 characters.` |
| Validation | 422 | `The item category name field is required.` · `… must be a string.` · `The is active field must be true or false.` |

## `POST /api/item/getMenuItems` and `/showCategoryMenuItems`

| Field | Rule |
|---|---|
| `categoryId`, `menuItemId` | optional strings — 32-char hex, converted since 2026-11-05; a malformed value gives 500 today (**target** 422 `Validation error`) |
| `page` | optional integer ≥ 1 (default 1) |
| `perPage` | optional integer 1–100 (default 10) |

| Endpoint | Items |
|---|---|
| `getMenuItems` | active **and** not deleted |
| `showCategoryMenuItems` | not deleted (active and inactive) |

200 `{ "status": true, "message": "Menu items retrieved successfully", "data": [ { menuItemId, restaurantId, categoryId, name, description, price: "250.00", isVeg, isActive, imageUrl, version, createdBy, isDelete, created_at, updated_at } ], "pagination": { currentPage, perPage, total, lastPage, from, to } }`. Ordered by `created_at` (since 2026-11-18). Validation failure → 422 `{ "status": false, "message": "Validation error", "errors": {…} }` (for example `perPage: 101`).

## `POST /api/item/menuItemSave`

The update rules apply whenever `menuItemId` is **filled** (`0` counts as filled). Checks run in this order after Laravel validation.

| Field | Create (`menuItemId` absent / null / "") | Update (`menuItemId` filled) |
|---|---|---|
| `menuItemId` | — | nullable string; must be 32-char hex (no dashes) → else 422 `Invalid menu item ID` |
| `categoryId` | required string; 32-char hex → else 422 `Invalid category ID`; a non-deleted category of the restaurant → else 422 `Invalid category. Category not found` | sometimes, same checks (omitted → the item's category) |
| `name` | required string; trimmed; blank → 422 `Menu item name is required`; unique per category among non-deleted items, any case → else 422 `Menu item name already exists in this category` | sometimes, same checks, excluding itself |
| `price` | required numeric, `gt:0` (since 2026-11-18) | sometimes, numeric, `gt:0` |
| `description` | nullable string | nullable string |
| `isActive`, `isVeg`, `isDelete` | nullable boolean (defaults true / **false** / false) | nullable boolean |
| `image` | nullable string — `data:image/png|jpg|jpeg;base64,…` | same; replaces `imageUrl` |

| Situation | HTTP | Body |
|---|---|---|
| Created | 200 | `{ "status": true, "message": "Menu item created successfully", "data": { …item } }` |
| Updated (since 2026-11-18) | 200 | `{ "status": true, "message": "Menu item updated successfully", "data": { …item } }` |
| Laravel validation | 422 | first message, e.g. `The menu item id field must be a string.`, `The category id field is required.`, `The name field is required.`, `The price field is required.`, `The price field must be a number.`, `The price field must be greater than 0.` |
| Service checks (above) | 422 | `{ "status": false, "message": "<Invalid menu item ID / Invalid category ID / Invalid category. Category not found / Menu item name is required / Menu item name already exists in this category / Menu item price must be greater than zero>", "data": null }` |
| Unknown / other restaurant's `menuItemId` (valid hex); a bad image; a name over 255 characters; a price ≥ 100 000 000 | 500 | `{ "status": false, "message": "Failed to save menu item", "data": null }` |
| **Target (MENU-D4)** unknown / deleted / other restaurant's `menuItemId` | 404 | `{ "status": false, "message": "Menu item not found", "data": null }` |
| **Target (MENU-D4)** name over 255; price over 99 999 999.99 | 422 | `The name field must not be greater than 255 characters.` · `The price field must not be greater than 99999999.99.` |
| **Target (MENU-D5)** bad image | 422 | `{ "status": false, "message": "The image must be a PNG or JPEG of at most 2 MB.", "data": null }` |

## `POST /api/image/upload`

| Field | Rule |
|---|---|
| `image` | required string |

| Situation | HTTP | Body |
|---|---|---|
| Saved | 201 | `{ "status": true, "message": "Image uploaded successfully", "data": { "status": true, "message": "Image uploaded successfully", "imagePath": "storage/image/<uuid>.<type>" } }` |
| Bad prefix, type or base64 | 201 | `{ "status": true, "message": "Image uploaded successfully", "data": { "status": false, "message": "Failed to upload image:", "code": 500 } }` (MENU-D5). **Target:** 422 `{ "status": false, "message": "The image must be a PNG or JPEG of at most 2 MB.", "data": null }` |
| **Target (MENU-D5)** content not PNG / JPEG, or over 2 MB decoded | 422 | same (today saved) |
| **Target (MENU-D5)** disk write fails | 500 | `{ "status": false, "message": "Failed to upload image", "data": null }` |
| Missing / not a string | 422 | `The image field is required.` / `The image field must be a string.` |

---

## Panel routes (web)

| Route | Body | Result today | Target (MENU-D3) |
|---|---|---|---|
| `GET /admin/categories-management/{restaurantId}` | — | Page; 10 per page; hex IDs | unknown restaurant → 404 |
| `POST /admin/categories/store` | `restaurantId, name, isActive` | Insert; `{ "success": true }`; empty name → 500 | name required, max 255, unique per restaurant (non-deleted) → 422 |
| `POST /admin/categories/update` | `id, name, isActive` | Update; no cascade | same rules; `isActive` cascades to items; unknown `id` → 404 |
| `POST /admin/categories/toggle` | `id, isActive` | Update `isActive`; no cascade | cascades to items; unknown `id` → 404 |
| `GET /admin/items-management/{restaurantId}` | — | Page; 10 per page; raw binary IDs; unknown restaurant → 500 | hex IDs; unknown restaurant → 404 |
| `POST /admin/items/store` | `restaurantId, categoryId, menuItemName, price, isActive` | Always 500 (column `menuItemName` doesn't exist) | writes `name`; API item rules → 422 |
| `POST /admin/items/update` | `id, menuItemName?, price?, isActive?, categoryId?` | 500 when `menuItemName` is sent (the page always sends it); without it, `price` / `isActive` / `categoryId` update by `id` | writes `name`; API item rules; unknown `id` → 404 |
| `POST /admin/item-toggle-status` | `id, isActive` | Updates by `id`; with the page's mangled IDs it matches nothing and still answers `{ "success": true }` | works with the page's hex IDs; unknown `id` → 404 |

Common failures: no session → 302 to `/` (JSON 401); bad CSRF → 419; a POS bearer token counts as no session. Target failures answer JSON 422 `{ "message": "<first error>", "errors": {…} }` or 404.
