# Menu — data model

Sources:

- `db_script/tables/tables.sql`
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
| `itemCategoryName` | varchar(255) NOT NULL | no uniqueness |
| `itemCategoryDescription` | varchar(255) null | |
| `isActive` | tinyint NOT NULL, 1 | |
| `createdBy` | varchar null | never set |
| `isDelete` | tinyint NOT NULL, 0 | API only |

## `menu_items`

| Column | Type | Notes |
|---|---|---|
| `menuItemId` | binary(16) PK | new GUID on create |
| `restaurantId` | binary(16) null | the caller's restaurant |
| `categoryId` | binary(16) null | `hex2bin` of the posted ID on create; **not checked** against `item_categories` (MENU-D4). Invalid hex → 500 |
| `name` | varchar(255) NOT NULL | |
| `description` | text null | |
| `price` | decimal(10,2) NOT NULL | any number from −99 999 999.99 to 99 999 999.99; more decimals are rounded |
| `isVeg` | tinyint NOT NULL, 1 (the API defaults to **false** when not sent) | |
| `isActive` | tinyint NOT NULL, 1 | |
| `imageUrl` | varchar(255) null | full URL `asset('storage/image/<uuid>.<type>')` |
| `version`, `createdBy` | | not set |
| `isDelete` | tinyint NOT NULL, 0 | API only |

## Reads

| Reader | Returns |
|---|---|
| `getItemsCategory` | the restaurant's categories, not deleted (active and inactive), each with `items` = **all** its items (inactive and deleted too) |
| `getMenuItems` | the restaurant's items, active and not deleted, ordered by `menuItemId`, paged |
| `showCategoryMenuItems` | the restaurant's items, not deleted (active and inactive), ordered by `menuItemId`, paged |
| Panel Categories page | all of the restaurant's categories (deleted too), with hex IDs |
| Panel Items page | all of the restaurant's items, with the category name and **raw binary** IDs (MENU-D3) |

## Rules

- In code, a category's `isActive` update cascades to its items. It is unreachable today (MENU-D2), and the panel's switch doesn't cascade.
- Image files are never deleted: replacing or deleting an item leaves its file.
- IDs are returned as 32-character hex.
