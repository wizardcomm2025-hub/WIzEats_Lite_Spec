# Super-admin panel — data model

Sources:

- `db_script/tables/tables.sql`
- `app/Http/Controllers/Admin/AdminAuthController.php`, `AdminController.php` (`index`, `analytics`, `applyDateFilter`, `orderhistory`, `itemhistory`)
- `config/auth.php`, `config/session.php`

## `super_admins`

| Column | Use |
|---|---|
| `id` | bigint |
| `name`, `email` | not shown anywhere |
| `username` | sign-in name |
| `password` | bcrypt hash; set by SQL only (ADM-D1) |
| `status` | 1 can sign in, 0 can't. Checked only at sign-in (ADM-D1) |
| `last_login_at` | set on each successful sign-in |
| `remember_token` | for "remember me" |

Sessions are rows in `sessions` (driver `database`, lifetime 120 min). The remember cookie `remember_admin_<hash>` lasts about 400 days (Laravel default).

## Restaurant list

`restaurants` where `isOnboardingDone` = 1, by `created_at` DESC, 10 per page. The counters count `status` = 1 / 0 among those.

## Overview queries (all filtered by the restaurant and the period on `created_at`)

| Figure | Query |
|---|---|
| Total Revenue | `SUM(orders.grandTotal)` where `orderStatus` = 2 |
| Total Orders | `COUNT(orders)` |
| Avg Order Value | revenue ÷ total orders, rounded to 2 |
| Total Items Sold | `SUM(order_items.quantity)` (period on the line's `created_at`) |
| Order Channel | orders ⟕ `order_channels` (same restaurant), grouped by channel; percent = count ÷ total × 100, rounded to 2 |
| Popular Items | `order_items` grouped by `menuItemId`, `menuItemName`; `SUM(quantity)`, `SUM(total)`; by quantity DESC |
| Recent Orders | orders ⟕ `order_items`, grouped by order: `HEX(orderId)`, line count, `grandTotal`, `orderStatus = 2` → Paid; by `created_at` DESC, 10 |
| Revenue chart | `DATE(created_at)`, `SUM(grandTotal)` where `orderStatus` = 2, by date |

Periods (`applyDateFilter`, UTC):

| `filter` | Condition |
|---|---|
| `today` (default) | `DATE(col) = today` |
| `yesterday` | `DATE(col) = today − 1` |
| `7days` / `15days` / `30days` | `DATE(col) >= (now − n days)`, i.e. n + 1 calendar days |
| anything else | no condition (all time) |

## Order History / Item History

- Orders where `DATE(created_at)` = today (UTC), newest first. Columns: `HEX(orderId)`, `channelName`, `grandTotal`, Paid = `orderStatus = 2`, `created_at`.
- Lines where `DATE(created_at)` = today, grouped by `menuItemName`: `SUM(quantity)`, `SUM(total)`, `MAX(orderItemId)`.
  - Sorts: `menuItemName`, `quantity`, `total`, `last_id` (default; the largest line ID).
  - Direction: `sort_order` is passed unchecked, and anything other than `asc` / `desc` throws.

## Restaurant ID in the URL

`binId()`:
- dashes are removed;
- 16 characters are used as raw bytes;
- 32 hex characters go through `hex2bin`;
- anything else is used as given (so it matches nothing).
