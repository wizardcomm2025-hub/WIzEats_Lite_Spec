# 04 — See orders and items — tester cases

Use case: [04-see-orders-and-items.md](../04-see-orders-and-items.md). Screen text: [ui-ux.md §5](../ui-ux.md#5-order-history--item-history-english-only). Fixtures: [README.md](./README.md#fixtures).

Signed in as ADM-1, on REST-A's pages (sidebar).

| ID | Covers | Steps | Expected |
|---|---|---|---|
| ADM-04-T01 | AC1 — *Fails until ADM-D2 is fixed* | ORDER-LIVE and ORDER-PAID today. Open `Order History`. | Title `Dashboard Overview`. Two rows, newest first: `Order No.` (`CODE-A/<n>`), upper-case hex `Order ID`, `Channel` `Takeaway`, `Total Amount` `₹ 525.00` for ORDER-PAID, `Payment Status` `Paid` (green) / `Unpaid` (red); `Date & Time` like `12 Nov 2026, 02:00 PM` in IST (the time the order was placed). (Today: no `Order No.`; times 5½ hours behind IST.) |
| ADM-04-T02 | AC2 — *Fails until ADM-D2 is fixed* | Open `Item History`. | One row: `1`, `Paneer Tikka`, `2`, `₹ 500.00` (ORDER-PAID only); `⬇️` on `Quantity`, `↕️` on `Amount`. (Today: `4`, `₹ 1000.00`; both arrows `↕️`.) |
| ADM-04-T03 | AC4 — *Fails until ADM-D2 is fixed* | Add ORDER-OLD. Reload both pages with `Today`; then pick `Yesterday` in the header. | `Today`: ORDER-OLD appears on neither. `Yesterday`: Order History lists ORDER-OLD, and the page stays Order History. (Today: `Yesterday` opens the Overview.) |
| ADM-04-T04 | AC3 | Today: one order with ITEM-1 × 1 and ITEM-2 × 3. On Item History click the `Quantity` arrow, then click it again; then click the `Amount` arrow. | 1st click: `Paneer Tikka` first, arrow `⬆️`. 2nd: `Chicken 65` first, `⬇️`. Amount click: sorted by amount, descending, `⬇️` on `Amount`. |
| ADM-04-T05 | Alt — *Fails until ADM-D2 is fixed* | On Order History pick `Last 7 Days` in the header dropdown; then open Item History from the sidebar. | Order History reloads with `?filter=7days` and lists the last 7 IST days' orders; the dropdown shows `Last 7 Days`. Item History opens with the same period. (Today: the **Overview** opens with `Last 7 Days`.) |
| ADM-04-T06 | Alt (ADM-D3) | Switch the language to `Deutsch`; open both pages. | The sidebar and header are German; the table headings stay `Order ID`, `Channel`, …, `SL No`, `Item Name`, `Quantity`, `Amount`; the title stays `Dashboard Overview`. |
| ADM-04-T07 | AC5 — *Fails until ADM-D2 is fixed* | ORDERS-30. Open `Order History`; go to page 2. | Page 1: 25 rows and `Showing 1 to 25 of 30 results`; page 2: 5 rows, `Showing 26 to 30 of 30 results`. (Today: all 30 on one page, no footer.) |
