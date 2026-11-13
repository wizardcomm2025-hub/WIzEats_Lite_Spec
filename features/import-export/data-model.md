# Import & export — data model

Sources:

- `app/Services/Import/ImportService.php`
- `app/Services/Export/ExportService.php`
- the models in TBL, MENU and ROOM

IMPEX adds no tables or columns.

## What one row creates

| File | Parent row (find or create) | Child row (always created) |
|---|---|---|
| Zone | `zones` by `restaurantId` + `zoneName` (any case) + `isDelete = 0` | `tables`: `zoneId`, `tableName`, `capacity = (int)col3`; `isActive` 1, `status` 0 |
| Menu | `item_categories` by `restaurantId` + `itemCategoryName` + `isDelete = 0` | `menu_items`: `categoryId`, `name`, `description` (empty → empty string, not NULL), `isVeg = (int)col4`, `price = (float)col5`; `isActive` 1, `imageUrl` NULL |
| Floor | `floors` by `restaurantId` + `floorName` + `isDelete = 0`; `createdBy` = the caller's role | `rooms`: `floorId`, `roomName`, `capacity = (int)col3`; `isActive` 1, `status` 0 |

New parents are active. A matched parent keeps its own `isActive`, so children may land in an inactive zone, category or floor.

## Casting examples

| Text | `(int)` | `(float)` |
|---|---|---|
| `4` | 4 | 4.0 |
| `four` | 0 | 0.0 |
| `-3` | -3 | -3.0 |
| `2.9` | 2 | 2.9 |
| `₹120` | 0 | 0.0 |
| `1e9` | 1000000000 (fits a capacity) | 1000000000.0 → price out of range → row skipped |

## Rules

- All rows run in one transaction. A row whose insert throws is caught, logged and skipped (IMPEX-D2).
- There are no uniqueness checks on tables, items or rooms.
- Export reads no data.
