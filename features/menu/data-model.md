# Menu — data model

Sources:

- `db_script/pos_stage.sql` (re-checked 2026-11-24: no MENU table changed; `menu_item_variants` is still missing — X-D7)
- `app/Services/Menu/ItemService.php`
- `app/Services/ImageUpload/ImageUploadService.php`
- `app/Models/ItemCategory.php`, `MenuItem.php`
- `AdminController` (category and item methods)

MENU adds no tables or columns.

## `item_categories`

| Column | Type | Notes |
|---|---|---|
| `itemCategoryId` | binary(16) PK | new GUID on create |
| `restaurantId` | binary(16) null | the caller's restaurant |
| `itemCategoryName` | varchar(255) NOT NULL | stored trimmed; unique per restaurant, any case, through the API since 2026-11-18 (deleted categories count today; target they don't) |
| `itemCategoryDescription` | varchar(255) null | cleared by every app edit today (MENU-D6) |
| `isActive` | tinyint NOT NULL, 1 | |
| `createdBy` | varchar null | never set |
| `isDelete` | tinyint NOT NULL, 0 | API only |

## `menu_items`

| Column | Type | Notes |
|---|---|---|
| `menuItemId` | binary(16) PK | new GUID on create |
| `restaurantId` | binary(16) null | the caller's restaurant |
| `categoryId` | binary(16) null | since 2026-11-18 a non-deleted category of the restaurant (API) |
| `name` | varchar(255) NOT NULL | stored trimmed; unique per category among non-deleted items, any case (API, since 2026-11-18) |
| `description` | text null | |
| `price` | decimal(10,2) NOT NULL | > 0 through the API since 2026-11-18; up to 99 999 999.99 (above → 500 today, target 422); more decimals are rounded |
| `isVeg` | tinyint NOT NULL, 1 (the API defaults to **false** when not sent) | |
| `isActive` | tinyint NOT NULL, 1 | follows the category when the category is switched through the API |
| `imageUrl` | varchar(255) null | full URL `asset('storage/image/<uuid>.<type>')` |
| `version`, `createdBy` | | not set |
| `isDelete` | tinyint NOT NULL, 0 | API only |

## Reads

| Reader | Returns |
|---|---|
| `getItemsCategory` | the restaurant's categories, not deleted (active and inactive), each with `items` = all its items (inactive and deleted today; target not deleted) |
| `getMenuItems` | the restaurant's items, active and not deleted, ordered by `created_at`, paged |
| `showCategoryMenuItems` | the restaurant's items, not deleted (active and inactive), ordered by `created_at`, paged |
| Panel Categories page | all of the restaurant's categories (deleted too), with hex IDs |
| Panel Items page | all of the restaurant's items, with the category name and **raw binary** IDs today (target hex — MENU-D3) |

## Rules

- A category's `isActive` update through the API cascades to its items. The panel's switch doesn't (target: it does — MENU-D3).
- Image files are never deleted: replacing or deleting an item leaves its file (target: a replaced image's file is deleted — MENU-D5).
- IDs are returned as 32-character hex.
