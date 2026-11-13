# 03 — Filter and search

## Summary

Staff narrow either tab by date range, channel and payment mode (filter sheet and quick date chips) and search by order number or amount. **The date filter works, search and payment mode work on History only, and any channel filter answers with an error (HIST-D1, HIST-D2).**

## Prerequisites

- **Backend:** both history endpoints — [api-contract.md](./api-contract.md#request-fields-both-endpoints)

## Actors & preconditions

- **Actor:** owner or waiter.
- **Preconditions:** Order History open.

## Main flow

1. **Date chips** under the search box: `Today`, `7 days`, `15 days`, `30 days`. The filter sheet adds `Custom` with a calendar; a custom range needs both ends (`Pick both a start and an end date.` otherwise).
2. **Filter sheet:** `Order Channel` (`All Channels` + each active channel), payment mode (`All`, `Cash`, `UPI`, one at a time), date range, a `Showing …` summary, `Reset`, `Apply`. The `Filters` button shows how many are on. `Reset` returns to Today / all channels / all modes.
3. **Search:** `Search by order number or amount`; `#`, `₹` and `,` are removed and spaces collapsed before sending.
4. Any change empties both tabs and reloads the visible one from page 1.
5. The server applies:
   - **dates** — History: whole IST days; Live: whole UTC days (HIST-D2);
   - **channel** — see HIST-D1;
   - **payment mode** — History only: orders with a completed payment of that type;
   - **search** — History only: `orderNumber LIKE %text%`, or `grandTotal = text` when numeric (HIST-D3).

## Alternative flows

- **Channel picked.** `Takeaway` / `Room Service` / `Delivery`: toast `The selected order channel id is invalid.` `Dine In`: toast `Failed to retrieve order history` (History) or `Failed to retrieve live order history` (Live). The list is empty until the channel is cleared (HIST-D1).
- **`endDate` before `startDate`** (API). 422 `The end date field must be a date after or equal to start date.`
- **Only one date sent** (API). The date filter is skipped.
- **`paymentType` other than CASH / UPI** (API). 422 `The selected payment type is invalid.`

## Acceptance criteria

1. Given ORDER-PAID (number `CODE-A/7`, grand total 525.00), when History is searched for `#7` or `₹525`, then it is found; searching `CODE-B` finds nothing.
2. Given Live with today's running order, when `Paneer` is searched, then the Live list is unchanged (HIST-D2).
3. Given the channel `Takeaway` is applied, then a toast `The selected order channel id is invalid.` shows and the list is empty; `Reset` brings the orders back.
4. Given `UPI` is picked on History and ORDER-PAID was paid in CASH, then it is hidden.

## Platform notes

### POS API

`orderHistoryRequest` rules; filters in `OrderService::getOrderHistory` / `getOrderHistoryLive`.

### Data — MySQL

Read only.

### POS app

`OrderHistoryFilterBar`, `OrderHistoryFilterSheet`, `orderHistory.filters.ts` (`channelFilterCode`, `toOrderHistoryPayload`, `normalizeOrderSearch`), `DateRangeCalendar`.

### Super-admin panel

None.

### Integrations

None.

## Configuration

None.

## Out of scope

- Report-style totals — RPT.

## Open questions

- None open.
