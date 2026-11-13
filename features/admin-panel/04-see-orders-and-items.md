# 04 — See today's orders and items

## Summary

From a restaurant's sidebar, `Order History` lists today's orders and `Item History` lists today's sales per item name. Item History can be sorted by quantity or amount. Both pages show today (UTC) only and ignore the header's period (ADM-D2).

## Prerequisites

- **Backend:** `GET /admin/orders/{restaurantId}`, `GET /admin/items/{restaurantId}?sort_by=&sort_order=`; see [api-contract.md](./api-contract.md#restaurant-pages)

## Actors & preconditions

- **Actor:** a signed-in super admin.

## Main flow

1. **Order History:** the page title stays `Dashboard Overview` (ADM-D3). The table has these columns, always in English:
   - `Order ID`: upper-case hex;
   - `Channel`: the channel name;
   - `Total Amount`: `₹ <grandTotal>`, as stored, e.g. `₹ 525.00`;
   - `Payment Status`: `Paid` (green) or `Unpaid` (red);
   - `Date & Time`: `d M Y, h:i A` in UTC.

   It lists every order created today (UTC), newest first, with no paging.
2. **Item History:** columns `SL No`, `Item Name`, `Quantity ↕️`, `Amount ↕️`. There is one row per item name for today's lines, with summed quantity and `₹ <sum>`.
3. Clicking the arrow next to `Quantity` or `Amount` sorts by it. The arrow then shows ⬆️ or ⬇️, and the next click reverses it. The first click sorts ascending when the current order is descending (the default).

## Alternative flows

- **Default Item History order:** by the largest line ID, a random GUID, so effectively random (ADM-D2).
- **`?sort_order=` other than `asc` / `desc`:** a 500 error page (ADM-D2).
- **`?sort_by=menuItemName`:** sorts by name; there is no link for it in the UI. An unknown `sort_by` uses the default.
- **Header period dropdown:** opens the **Overview** with that period (ADM-D2).
- **Orders before today (UTC)**, including those made before 05:30 IST today, are not shown.
- **Two different items with the same name:** one row (ADM-D2).
- **Unknown restaurant ID:** empty tables (ADM-D3).

## Acceptance criteria

1. Given ORDER-LIVE and ORDER-PAID today, then Order History lists both, newest first:
   - ORDER-PAID: `Takeaway`, `₹ 525.00`, `Paid`;
   - ORDER-LIVE: `Unpaid`;
   - each with a UTC time.
2. Given those orders, then Item History shows one row: `Paneer Tikka`, `4`, `₹ 1000.00`.
3. Given ITEM-1 × 1 and ITEM-2 × 3 today, when `Quantity` is clicked, then `Paneer Tikka` comes first (ascending) with ⬆️, and a second click puts `Chicken 65` first with ⬇️.
4. Given ORDER-OLD (yesterday), then it appears on neither page.

## Platform notes

### Super-admin panel

`AdminController::orderhistory`, `itemhistory` (`orderBy($column, $sortOrder)`); `admin/order-history.blade.php`, `admin/item-history.blade.php`.

### Data — MySQL

Read only.

### POS app, POS API

None.

## Configuration

None.

## Out of scope

- Order details, bills, refunds.

## Open questions

- None open.
