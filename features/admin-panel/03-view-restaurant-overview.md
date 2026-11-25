# 03 — View a restaurant's overview

## Summary

`View Analytics` opens one restaurant's **Overview** page. It shows four figures, a revenue chart, an order-channel chart, Popular Items and the 10 most recent orders, for the period chosen in the header. A sidebar leads to the restaurant's other pages. **Today revenue counts paid orders while the other figures count all orders, days are UTC and "Last 7 Days" spans 8 days (ADM-D2 — To fix: paid only, IST days, as RPT).**

## Prerequisites

- **Backend:** `GET /admin/analytics/{restaurantId}?filter=<period>`; see [api-contract.md](./api-contract.md#restaurant-pages)
- **Data:** `orders`, `order_items`, `order_channels`; see [data-model.md](./data-model.md)

## Actors & preconditions

- **Actor:** a signed-in super admin. Any restaurant (ADM-D1).

## Main flow

1. **Layout:**
   - the sidebar shows `WizardComm` / `Analytics Dashboard` and these links: `Overview`, `Order History`, `Item History`, `Zone Management`, `Table Management`, `Category Management`, `Item Management`, `Restaurant Settings`, `User Management`;
   - the header shows the page title `Dashboard Overview` / `Welcome back! Here's what's happening today.`;
   - the header controls are: the period dropdown (`Today`, `Yesterday`, `Last 7 Days`, `Last 15 Days`, `Last 30 Days`), the language dropdown, a bell, `Export Report` and `Logout`.
2. Choosing a period reloads the page with `?filter=today|yesterday|7days|15days|30days`. With no filter the page shows today.
3. **Figures** — *target (ADM-D2):* paid orders only (`orderStatus` 2):
   - `Total Revenue`: Σ `grandTotal`, shown as `₹525.00`;
   - `Total Orders`: the count of paid orders;
   - `Total Items Sold`: Σ `quantity` of their lines;
   - `Avg Order Value`: `₹` + revenue ÷ paid orders, to 2 decimals.

   *Today:* revenue paid only and shown without `₹`; orders and items count every order; Avg = revenue ÷ all orders.
4. **`Revenue Overview`**: a line chart of paid revenue. *Target:* subtitle `Daily revenue — <period>`, one point per IST date of the period (0 when none). *Today:* subtitle `Daily revenue for the current month`, one point per UTC date that has paid revenue.
5. **`Order Channel`** (subtitle `Distribution of order channels`): a pie chart with `Total Orders` in the middle. Below it is a list of `<channel name> <percent>%` (percent to 2 decimals) — *target* over paid orders; today over all orders.
6. **`Popular Items`**: items grouped by item ID and name, sorted by quantity, as `<rank>`, the name, `<quantity> orders` and `₹<amount>`. *Target:* subtitle `Top performing menu items — <period>`; paid orders' lines only; the top 10, ranked `01`–`10`. *Today:* subtitle `Top performing menu items this week`; every line, every item, ranked `0<rank>` (`010` for the tenth).
7. **`Recent Orders`** (`Latest incoming orders`, `Live` dot): up to 10 orders, newest first, with these columns:
   - `Order ID`: upper-case hex;
   - `Items`: `<n> items`, the number of lines;
   - `Total`: `₹<grandTotal>`;
   - `Payment Status`: `Paid` / `Unpaid`.
8. Sidebar links open Order History / Item History ([04](./04-see-orders-and-items.md)) or the pages owned by other features.

## Alternative flows

- **No orders in the period:** all figures `0` / `0.00` / `₹0.00`, empty charts and lists.
- **Unknown or malformed restaurant ID:** the same empty page, not a 404 (ADM-D3).
- **Unknown or empty `filter`** (`?filter=`): *target* `Today`; today all-time figures, while the dropdown shows `Today` (ADM-D2).
- **Periods:** *target* IST days; `Last 7 Days` = today and the 6 days before (likewise 15, 30). Today UTC days, and `Last 7 Days` includes the date 7 days ago (8 days) (ADM-D2).
- **Header subtitle:** *target* `Welcome back! Here's what happened: <period>.`; today always `… happening today.`
- **`Export Report`, bell:** nothing happens (ADM-D3).

## Acceptance criteria

1. *(Target, ADM-D2)* Given ORDER-LIVE and ORDER-PAID today, then the page shows:
   - `Total Revenue` `₹525.00`, `Total Orders` `1`, `Total Items Sold` `2`, `Avg Order Value` `₹525.00`;
   - `Order Channel` `Takeaway 100%`;
   - `Popular Items` `01` `Paneer Tikka` `2 orders` `₹500.00`;
   - `Recent Orders` two rows: ORDER-PAID `Paid` `₹525.00`, ORDER-LIVE `Unpaid`.
   (Today: `525.00`, `2`, `4`, `₹262.50`; `4 orders` `₹1,000.00`.)
2. *(Target)* Given only ORDER-LIVE, then all figures are zero and Popular Items is empty; `Recent Orders` lists ORDER-LIVE `Unpaid`.
3. *(Target)* Given orders paid at 14:00 IST six and seven days ago, then `Last 7 Days` includes only the first. (Today: both.)
4. *(Target)* Given an order paid at 00:30 IST today, then it counts under `Today`, not `Yesterday`. (Today: `Yesterday`.)
5. *(Target)* Given 12 different items sold in paid orders, then Popular Items shows 10 rows ranked `01`–`10`.

## Platform notes

### Super-admin panel

`AdminController::analytics`, `applyDateFilter`; `resources/views/admin/analytics.blade.php`, `layouts/app`, `layouts/header`, `layouts/sidebar`; the texts are in `lang/<locale>/analytics.php`. Charts use Chart.js.

### Data — MySQL

Read only; see [data-model.md](./data-model.md).

### POS app, POS API

None.

## Configuration

`config/app.php` timezone `UTC` (stays; the target converts IST day boundaries in the queries).

## Out of scope

- Exporting. Comparison with previous periods.

## Open questions

- None open. Related decisions: ADM-D2, ADM-D3.
