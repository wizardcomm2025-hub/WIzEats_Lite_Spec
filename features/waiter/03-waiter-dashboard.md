# 03 — Waiter dashboard

## Summary

From the drawer (`Operations` → `Dashboard`) or the tablet rail, the waiter sees **Today's Overview**: a `Total Orders` card and **Your Top Selling Items** for today. Today `Total Orders` counts all their orders ever (WAIT-D3); target today's only.

## Prerequisites

- **Backend:** `POST /api/waiter/dashboard` — [api-contract.md](./api-contract.md#post-apiwaiterdashboard)
- **Data:** `orders`, `order_items`, `tables`

## Actors & preconditions

- **Actor:** waiter. Today also a chef saved as `chef` (KITCH-D3) and any role through the API; target waiter only.
- **Preconditions:** signed in.

## Main flow

1. The waiter opens the dashboard. The app posts `/waiter/dashboard` once per sign-in; pull down asks again.
2. The server computes, for the caller's restaurant and `createdByUserId` = the caller:
   - `totalOrders` — count of their orders, any status; today any date, *target* created today;
   - `totalRevenue` — sum of `grandTotal` of those fulfilled (status 2); today any date, *target* today's;
   - `averageOrderValue` — `totalRevenue / totalOrders`, 2 decimals;
   - `tableTurnoverRate` — their orders fulfilled and created today ÷ the restaurant's table count, 2 decimals;
   - `topSellingItems` — up to 10 `{ menuItemName, quantity, revenue }` from their fulfilled orders created today, grouped by item ID and name, most quantity first.
3. The app shows `Hello, <name>!` · `Here's your performance summary` · `Today's Overview` · `WAIT-<userId hex>`, the `Total Orders` card, and the top-selling table (rank, item, quantity; the revenue column is hidden).

## Alternative flows

- **No fulfilled orders today.** `No items sold yet today`.
- **Server error.** 500 `Failed to fetch dashboard data`; the app keeps its last data.
- **Same item sent under two names.** Today two rows (ORD-D1); after the ORD-D1 fix names come from the menu, so one row.

## Acceptance criteria

1. *(Target, WAIT-D3)* Given WTR-A created 3 orders yesterday and 2 today, when the dashboard opens, then `Total Orders` is 2. (Today: 5.)
2. Given one of today's orders is paid (status 2) with `Paneer Tikka` × 2, then the top-selling table shows `Paneer Tikka` with quantity 2; unpaid orders add nothing.
3. Given OWN-A created orders, then WTR-A's counts don't include them.

## Platform notes

### POS API

`StaffController::getWaiterDashboardDetails` → `StaffService::getDashboardData`. "Today" uses the server date on `created_at`.

### Data — MySQL

Read only.

### POS app

`waiterDashboard/waiterScreen.tsx` (`fetchWaiterDashboard`), `StatsOverview`, `TopSellingItems`. Quantity is shown with `parseInt`.

### Super-admin panel

None.

### Integrations

None.

## Configuration

None.

## Out of scope

- The owner's dashboard — RPT. The chef's dashboard — KITCH.

## Open questions

- None open. Related decision: WAIT-D3.
