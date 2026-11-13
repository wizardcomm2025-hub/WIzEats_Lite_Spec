# 03 — View a restaurant's overview

## Summary

`View Analytics` opens one restaurant's **Overview** page. It shows four figures, a revenue chart, an order-channel chart, Popular Items and the 10 most recent orders, for the period chosen in the header. A sidebar leads to the restaurant's other pages.

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
3. **Figures** (ADM-D2):
   - `Total Revenue`: Σ `grandTotal` of paid orders, shown as `525.00` (no `₹`);
   - `Total Orders`: the count of all orders;
   - `Total Items Sold`: Σ `quantity` of all lines;
   - `Avg Order Value`: `₹` + revenue ÷ all orders, to 2 decimals.
4. **`Revenue Overview`** (subtitle `Daily revenue for the current month`): a line chart with one point per UTC date with paid revenue.
5. **`Order Channel`** (subtitle `Distribution of order channels`): a pie chart with `Total Orders` in the middle. Below it is a list of `<channel name> <percent>%` (percent to 2 decimals) over all orders.
6. **`Popular Items`** (subtitle `Top performing menu items this week`): every item, grouped by item ID and name, sorted by quantity, as `0<rank>`, the name, `<quantity> orders` and `₹<amount>`.
7. **`Recent Orders`** (`Latest incoming orders`, `Live` dot): up to 10 orders, newest first, with these columns:
   - `Order ID`: upper-case hex;
   - `Items`: `<n> items`, the number of lines;
   - `Total`: `₹<grandTotal>`;
   - `Payment Status`: `Paid` / `Unpaid`.
8. Sidebar links open Order History / Item History ([04](./04-see-orders-and-items.md)) or the pages owned by other features.

## Alternative flows

- **No orders in the period:** all figures `0` / `0.00` / `₹0.00`, empty charts and lists.
- **Unknown or malformed restaurant ID:** the same empty page, not a 404 (ADM-D3).
- **Unknown or empty `filter`** (`?filter=`): all-time figures, while the dropdown shows `Today` (ADM-D2).
- **Periods:** UTC days. `Last 7 Days` includes the date 7 days ago, and likewise for 15 and 30 (ADM-D2).
- **`Export Report`, bell:** nothing happens (ADM-D3).

## Acceptance criteria

1. Given ORDER-LIVE and ORDER-PAID today, then the page shows:
   - `Total Revenue` `525.00`, `Total Orders` `2`, `Total Items Sold` `4`, `Avg Order Value` `₹262.50`;
   - `Order Channel` `Takeaway 100%`;
   - `Popular Items` `01` `Paneer Tikka` `4 orders` `₹1,000.00`;
   - `Recent Orders` two rows: ORDER-PAID `Paid` `₹525.00`, ORDER-LIVE `Unpaid`.
2. Given only ORDER-LIVE, then `Total Revenue` `0.00`, `Total Orders` `1`, `Avg Order Value` `₹0.00`, with `Paneer Tikka` still under Popular Items.
3. Given an order paid at 14:00 IST seven days ago, then `Last 7 Days` includes it, and an order eight days ago is not included.
4. Given an order created at 00:30 IST today, then it counts under `Yesterday`, not `Today`.

## Platform notes

### Super-admin panel

`AdminController::analytics`, `applyDateFilter`; `resources/views/admin/analytics.blade.php`, `layouts/app`, `layouts/header`, `layouts/sidebar`; the texts are in `lang/<locale>/analytics.php`. Charts use Chart.js.

### Data — MySQL

Read only; see [data-model.md](./data-model.md).

### POS app, POS API

None.

## Configuration

`config/app.php` timezone `UTC`.

## Out of scope

- Exporting. Comparison with previous periods.

## Open questions

- None open.
