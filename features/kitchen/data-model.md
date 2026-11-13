# Kitchen (chef) — data model

Sources:

- `db_script/tables/tables.sql` (`kots`)
- `app/Services/Staff/StaffService.php` (`getPendingKots`, `updateKOTStatus`, `getChefKotStats`)

## `kots` (columns KITCH reads or writes)

| Column | Type | KITCH |
|---|---|---|
| `kotId` | `binary(16)` | lookup key; sent in lists as raw bytes (KITCH-D1) |
| `kotNumber` | varchar | not read by KITCH (the app shows `kotId` instead) |
| `kotNote` | varchar | shown on the card |
| `chefId` | `binary(16)` | set to the caller on every status change |
| `status` | `tinyint(1)`, default 0 | 0 Pending, 1 Preparing, 2 Rejected, 3 Completed, 4 Ready; any −128…127 can be stored (KITCH-D2) |
| `preparing_at` | timestamp | now if the new status is 1, else **NULL** |
| `rejected_at` | timestamp | now if 2, else **NULL** |
| `completed_at` | timestamp | now if 3, else **NULL** |
| `isPickedUp`, `pickedUp_at` | — | WAIT |
| `created_at` | timestamp | list order (newest first); dashboard "today" |
| `updated_at` | timestamp | now on every status change |

There is no "ready at" time: moving to Ready (4) clears all three stage times.

## Restaurant scope

`kots` has no `restaurantId`. The list reaches the restaurant through `order_items.restaurantId` (inner-join behaviour, because of the `WHERE`). The status update and the dashboard have no restaurant filter at all (KITCH-D2, KITCH-D3).

## List row shape (per KOT)

`kotId`, `kotNote`, `status` (label), `tableName`, `zoneName`, `roomName`, `floorName`, `created_at`, `items: [ { menuItemName, quantity } ]`. One item entry per `order_items` row, so the same dish sent twice appears twice. Place names are NULL for orders without a table or room (all app-created orders — ORD-D2).

## Dashboard counts

`SELECT COUNT(*), SUM(status = 1), SUM(status = 2) FROM kots WHERE chefId = <caller> AND DATE(created_at) = today`.
