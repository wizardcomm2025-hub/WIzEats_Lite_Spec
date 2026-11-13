# Waiter — data model

Sources:

- `db_script/tables/tables.sql` (`kots`, `orders`, `order_items`, `tables`)
- `app/Services/Staff/StaffService.php` (`getReadyKots`, `updateKOTPickedUpStatus`, `getDashboardData`)

## `kots` (columns WAIT reads or writes)

| Column | WAIT |
|---|---|
| `kotId` (`binary(16)`) | lookup key; sent raw in replies (WAIT-D1) |
| `status` | read: 4 = Ready. Never changed by WAIT. |
| `isPickedUp` (tinyint, default 0) | set to 1 |
| `pickedUp_at` | now, on every call that matches (WAIT-D2) |

`kots` has no `restaurantId`. The ready list scopes through `order_items.restaurantId`; the pick-up update and its reply have no scope at all (WAIT-D2).

## Ready list row (per KOT)

`kotId`, `kotNote`, `status` (number 4), `tableName`, `zoneName`, `roomName`, `floorName`, `created_at`, `isPickedUp` (0), `items: [ { menuItemName, quantity } ]`.

## Pick-up reply rows

Full `kots` rows with `isPickedUp` 1, from all restaurants: `kotId`, `kotNumber`, `kotNote`, `createdBy`, `chefId`, `status`, `preparing_at`, `rejected_at`, `completed_at`, `isPickedUp`, `pickedUp_at`, `created_at`, `updated_at`.

## Dashboard sources

| Value | From |
|---|---|
| `totalOrders` | `COUNT(orders)` where restaurant and `createdByUserId` = caller |
| `totalRevenue` | `SUM(grandTotal)` of those with `orderStatus` 2 |
| `averageOrderValue` | `totalRevenue / totalOrders` |
| `tableTurnoverRate` | count of those with status 2 and `created_at` today ÷ `COUNT(tables)` of the restaurant (all tables, active or not) |
| `topSellingItems` | `order_items` joined to those orders, status 2, created today; `SUM(quantity)`, `SUM(total)`; top 10 |
