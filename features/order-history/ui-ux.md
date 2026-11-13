# Order history — UI / UX

Screen text exactly as in the code on 2026-11-11. Only the parts testers check are listed.

## 1. Screen

| Part | Content |
|---|---|
| Entry | Phone drawer `Order History`; tablet rail `Orders` (owner, waiter) |
| Tabs | `Live` · `History`, each with the server's count |
| Search | `Search by order number or amount` |
| Date chips | `Today` · `7 days` · `15 days` · `30 days`; `Reset` when filters are on |
| Filters button | badge with the number of active filters |
| Errors | Toast with the server's message, or `Failed to fetch orders` |
| Empty Live | `No Live Orders` · `Orders still awaiting payment will show up here. Pull down to refresh.` |
| Empty History | `No Past Orders` · `Settled and closed orders will appear here. Pull down to refresh.` |
| Paging | `Load More` / automatic at the end of the list (History) |
| Tablet | List on the left, the selected order's details beside it |

## 2. Order card

| Part | Live | History |
|---|---|---|
| Title | `Order No. <number>` · `Pending` (amber) · `Bill generated` for a billed order | `Order No. <number>` · `Paid` (green) |
| Subtitle | `<channel> · <n> item(s)` | `<channel> · <n> item(s) · <dd/Mon/yyyy hh:mm AM/PM>` |
| Total | `₹<subTotal>` `Sub Total` (HIST-D2) | `₹<grandTotal>` `Grand Total` |
| Expanded | lines (`ITEM` / `AMOUNT`, price × quantity), `Discount`, `Tax and round-off are added when the bill is generated.` | payment chips (`CASH`, `UPI`; amounts when split), lines, `Sub Total`, `Discount (<n>%)`, `Tax`, `Round Off` (signed), `Change Returned`, `Grand Total`, `Print Bill No. <n>` (PRN) |
| Button | `Bill Payment` → `Loading Bill…` (never opens — BILL-D2) | — |

## 3. Filter sheet

| Part | Content |
|---|---|
| Channel | `Order Channel`: `All Channels`, then each active channel |
| Payment | `All`, `Cash`, `UPI` (one at a time) |
| Dates | `Today`, `Last 7 days`, `Last 15 days`, `Last 30 days`, `Custom` (calendar) |
| Footer | `Showing <summary>` · `Reset` · `Apply` |
| Half-picked custom range | Toast `Pick both a start and an end date.` |
