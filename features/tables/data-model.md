# Zones & tables — data model

Sources:

- `db_script/tables/tables.sql`
- `app/Services/Table/TableService.php`
- `app/Models/Table.php`, `TableZone.php`
- `AdminController` (zone and table methods)

TBL adds no tables or columns.

## `zones`

| Column | Type | Notes |
|---|---|---|
| `zoneId` | binary(16) PK | new GUID on create |
| `restaurantId` | binary(16) null | the caller's restaurant (API); the posted one (panel) |
| `zoneName` | varchar(255) NOT NULL | no uniqueness |
| `isActive` | tinyint NOT NULL, 1 | |
| `isDelete` | tinyint NOT NULL, 0 | API only |
| `createdBy` | varchar null | never set |
| `created_at`, `updated_at` | | |

## `tables`

| Column | Type | Notes |
|---|---|---|
| `tableId` | binary(16) PK | new GUID on create |
| `restaurantId` | binary(16) null | |
| `zoneId` | binary(16) null | **not checked** against `zones` (TBL-D2) |
| `tableName` | varchar(255) NOT NULL | duplicate check on API create only: same restaurant + zone, not deleted, case-insensitive |
| `capacity` | int NOT NULL | any integer through the API or panel |
| `isActive` | tinyint NOT NULL, 1 | follows the zone when the zone is switched through the API |
| `status` | tinyint NOT NULL, 0 | 0 free · 1 occupied · 2 ordered — set by ORD / BILL; the API also accepts 0–2. Outside 0–2 (possible only through the panel), building the `status_key` / `status_color` attributes throws, and every API response that includes the table fails |
| `version`, `createdBy` | | not set by TBL |
| `isDelete` | tinyint NOT NULL, 0 | API only (and panel over-posting) |

## Reads

| Reader | Zones | Tables |
|---|---|---|
| `getTablesZoneDetails` | restaurant, not deleted (active **and** inactive), plus `tableCount` = all tables of the zone | — |
| `getTablesDetails` | — | restaurant, not deleted (active and inactive, any zone — including deleted or foreign zones), left join `orders` with status 0 / 1 → `orderId` (one row per running order) |
| `showDetails` (RST) | restaurant, active, not deleted | active, not deleted, in that zone; latest running order |
| Panel pages | restaurant, **all** (deleted too) | restaurant, all, with the zone name |

## Rules

- `ZoneSave` with `isActive` updates `tables.isActive` for every table of the zone, including deleted ones. The panel's zone switch doesn't.
- Deleting a zone (`isDelete = 1`) doesn't touch its tables (TBL-D2).
- API updates find the row within the caller's restaurant. Panel updates find it by ID alone.
- IDs are returned as 32-character hex. API input accepts hex with or without dashes.
