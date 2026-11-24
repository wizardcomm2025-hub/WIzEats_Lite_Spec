# 03 — Filter and search

## Summary

Staff narrow either tab by date range, channel and payment mode (filter sheet and quick date chips) and search by order number or amount. Since 2026-11-18 the channel filter works (HIST-D1 fixed). Today dates and channel apply to both tabs, search and payment mode to History only; target: on Live, channel and search apply and the date and payment filters are hidden (HIST-D2).

## Prerequisites

- **Backend:** both history endpoints — [api-contract.md](./api-contract.md#request-fields-both-endpoints)

## Actors & preconditions

- **Actor:** owner or waiter.
- **Preconditions:** Order History open.

## Main flow

1. **Date chips** under the search box: `Today`, `7 days`, `15 days`, `30 days`. The filter sheet adds `Custom` with a calendar; a custom range needs both ends (`Pick both a start and an end date.` otherwise). *Target:* shown on History only.
2. **Filter sheet:** `Order Channel` (`All Channels` + each active channel), payment mode (`All`, `Cash`, `UPI`, one at a time — target History only), date range, a `Showing …` summary, `Reset`, `Apply`. The `Filters` button shows how many are on. `Reset` returns to Today / all channels / all modes.
3. **Search:** `Search by order number or amount`; `#`, `₹` and `,` are removed and spaces collapsed before sending.
4. Any change empties both tabs and reloads the visible one from page 1.
5. The server applies:
   - **dates** — History: whole IST days; Live: today whole UTC days, target none (HIST-D2);
   - **channel** — the channel's code, both tabs;
   - **payment mode** — History only: orders with a completed payment of that type;
   - **search** — History (target also Live): `orderNumber LIKE %text%`, or `grandTotal = text` when numeric.

## Alternative flows

- **Channel picked.** Only that channel's orders, on both tabs.
- **Unknown channel code** (API). 422 `The selected channel code is invalid.`
- **`endDate` before `startDate`** (API). 422 `The end date field must be a date after or equal to start date.`
- **Only one date sent** (API). The date filter is skipped.
- **`paymentType` other than CASH / UPI** (API). 422 `The selected payment type is invalid.`

## Acceptance criteria

1. Given ORDER-PAID (number `CODE-A/7`, grand total 525.00), when History is searched for `#7` or `₹525`, then it is found; searching `CODE-B` finds nothing.
2. *(Target, HIST-D2)* Given Live with a running order numbered `CODE-A/8`, when `#8` is searched, then it stays listed; when `zzz` is searched, then the Live list is empty. (Today: the list doesn't change.)
3. Given the channel `Takeaway` is applied, then only Takeaway orders show, on both tabs (HIST-D1 fixed); `Reset` brings back all.
4. Given `UPI` is picked on History and ORDER-PAID was paid in CASH, then it is hidden.

## Platform notes

### POS API

`orderHistoryRequest` rules; filters in `OrderService::getOrderHistory` / `getOrderHistoryLive`.

### Data — MySQL

Read only.

### POS app

`OrderHistoryFilterBar`, `OrderHistoryFilterSheet`, `orderHistory.filters.ts` (`channelFilterCode` — the channel's `channelCode`, `toOrderHistoryPayload`, `normalizeOrderSearch`), `DateRangeCalendar`.

### Super-admin panel

None.

### Integrations

None.

## Configuration

None.

## Out of scope

- Report-style totals — RPT.

## Open questions

- None open. Related decisions: HIST-D1 (fixed), HIST-D2.
