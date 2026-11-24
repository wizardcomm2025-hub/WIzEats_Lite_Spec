# Kitchen (chef) — data model

Sources:

- `db_script/pos_stage.sql` (`kots`; re-checked 2026-11-24: unchanged)
- `app/Services/Staff/StaffService.php` (`getPendingKots`, `updateKOTStatus`, `getChefKotStats`)

## `kots` (columns KITCH reads or writes)

| Column | Type | KITCH |
|---|---|---|
| `kotId` | `binary(16)` | lookup key; sent in lists as raw bytes today (KITCH-D1); target hex |
| `kotNumber` | varchar | not read today (the app shows `kotId`); target: in the list and on the card |
| `kotNote` | varchar | shown on the card |
| `chefId` | `binary(16)` | today set to the caller on every status change; target on accept / reject only |
| `status` | `tinyint(1)`, default 0 | 0 Pending, 1 Preparing, 2 Rejected, 3 Completed, 4 Ready; today any −128…127 can be stored (KITCH-D2); target 0–4 along the allowed path |
| `preparing_at` | timestamp | today now if the new status is 1, else **NULL**; target set on → 1 and kept |
| `rejected_at` | timestamp | today now if 2, else **NULL**; target set on → 2 and kept |
| `completed_at` | timestamp | today now if 3, else **NULL**; target set on → 3 and kept |
| `isPickedUp`, `pickedUp_at` | — | WAIT |
| `created_at` | timestamp | list order (newest first); dashboard "today" |
| `updated_at` | timestamp | now on every status change |

There is no "ready at" time. Today moving to Ready (4) clears all three stage times; target keeps them.

## Restaurant scope

`kots` has no `restaurantId`. The list reaches the restaurant through `order_items.restaurantId` (inner-join behaviour, because of the `WHERE`). Today the status update and the dashboard have no restaurant filter at all (KITCH-D2, KITCH-D3); target: both scope through `order_items` the same way.

## List row shape (per KOT)

Today: `kotId` (raw), `kotNote`, `status` (label), `tableName`, `zoneName`, `roomName`, `floorName`, `created_at`, `items: [ { menuItemName, quantity } ]` — and the raw `tableId` / `roomId` are selected too, which also breaks the JSON. Target: `kotId` hex, plus `kotNumber` and `orderNumber`, no raw IDs. One item entry per `order_items` row, so the same dish sent twice appears twice. Place names are filled for app orders with a table or room since the ORD-D2 fix; Takeaway and Delivery KOTs have none (`N/A`).

## Dashboard counts

Today: `SELECT COUNT(*), SUM(status = 1), SUM(status = 2) FROM kots WHERE chefId = <caller> AND DATE(created_at) = today` (KITCH-D3).
Target: over the restaurant's KOTs created today (through `order_items`): `total_received` = all; `total_accepted` = status 1, 3 or 4; `total_rejected` = status 2.
