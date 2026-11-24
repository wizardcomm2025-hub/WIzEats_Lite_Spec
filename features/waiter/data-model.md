# Waiter — data model

Sources:

- `db_script/pos_stage.sql` (`kots`, `orders`, `order_items`, `tables`; re-checked 2026-11-24: unchanged)
- `app/Services/Staff/StaffService.php` (`getReadyKots`, `updateKOTPickedUpStatus`, `getDashboardData`)

## `kots` (columns WAIT reads or writes)

| Column | WAIT |
|---|---|
| `kotId` (`binary(16)`) | lookup key; sent raw in replies today (WAIT-D1); target hex |
| `kotNumber` | target: in the lists and the pick-up reply |
| `status` | read: 4 = Ready. Never changed by WAIT. Target: pick-up only from 4 |
| `isPickedUp` (tinyint, default 0) | set to 1 (target: once) |
| `pickedUp_at` | today now on every call that matches (WAIT-D2); target set once |

`kots` has no `restaurantId`. The ready list scopes through `order_items.restaurantId`; today the pick-up update and its reply have no scope at all (WAIT-D2); target: the same `order_items` scope.

## Ready list row (per KOT)

Today: `kotId` (raw), `kotNote`, `status` (number 4), `tableName`, `zoneName`, `roomName`, `floorName`, `created_at`, `isPickedUp` (0), `items: [ { menuItemName, quantity } ]`. Target: `kotId` hex plus `kotNumber` and `orderNumber`; the `pickedUp: true` list adds `pickedUp_at`. Place names are filled for app orders with a table or room since the ORD-D2 fix; Takeaway and Delivery KOTs show `Takeaway` in the app.

## Pick-up reply rows

Today: full `kots` rows with `isPickedUp` 1, from all restaurants: `kotId`, `kotNumber`, `kotNote`, `createdBy`, `chefId`, `status`, `preparing_at`, `rejected_at`, `completed_at`, `isPickedUp`, `pickedUp_at`, `created_at`, `updated_at`. Target: `{ kotId, kotNumber, pickedUp_at }` of the KOT just picked up.

## Dashboard sources

| Value | Today (WAIT-D3) | Target |
|---|---|---|
| `totalOrders` | `COUNT(orders)` where restaurant and `createdByUserId` = caller, any date | same, `created_at` today |
| `totalRevenue` | `SUM(grandTotal)` of those with `orderStatus` 2, any date | of today's |
| `averageOrderValue` | `totalRevenue / totalOrders` | same, over today's |
| `tableTurnoverRate` | count of those with status 2 and `created_at` today ÷ `COUNT(tables)` of the restaurant (all tables, active or not) | unchanged |
| `topSellingItems` | `order_items` joined to those orders, status 2, created today; `SUM(quantity)`, `SUM(total)`; top 10 | unchanged |
