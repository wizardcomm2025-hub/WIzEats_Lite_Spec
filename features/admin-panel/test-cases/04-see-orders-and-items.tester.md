# 04 — See today's orders and items — tester cases

Use case: [04-see-orders-and-items.md](../04-see-orders-and-items.md). Screen text: [ui-ux.md §5](../ui-ux.md#5-order-history--item-history-english-only). Fixtures: [README.md](./README.md#fixtures).

Signed in as ADM-1, on REST-A's pages (sidebar).

| ID | Covers | Steps | Expected |
|---|---|---|---|
| ADM-04-T01 | AC1 (ADM-D2) | ORDER-LIVE and ORDER-PAID today. Open `Order History`. | Title `Dashboard Overview`. Two rows, newest first: upper-case hex `Order ID`, `Channel` `Takeaway`, `Total Amount` `₹ 525.00` for ORDER-PAID, `Payment Status` `Paid` (green) / `Unpaid` (red); `Date & Time` like `12 Nov 2026, 08:30 AM`, 5½ hours behind IST. |
| ADM-04-T02 | AC2 | Open `Item History`. | One row: `1`, `Paneer Tikka`, `4`, `₹ 1000.00`; both sort arrows `↕️`. |
| ADM-04-T03 | AC4 | Add ORDER-OLD. Reload both pages. | ORDER-OLD appears on neither. |
| ADM-04-T04 | AC3 | Today: one order with ITEM-1 × 1 and ITEM-2 × 3. On Item History click the `Quantity` arrow, then click it again; then click the `Amount` arrow. | 1st click: `Paneer Tikka` first, arrow `⬆️`. 2nd: `Chicken 65` first, `⬇️`. Amount click: sorted by amount, descending, `⬇️` on `Amount`. |
| ADM-04-T05 | Alt (ADM-D2) | On Order History pick `Yesterday` in the header dropdown. | The **Overview** page opens with `Yesterday`. |
| ADM-04-T06 | Alt (ADM-D3) | Switch the language to `Deutsch`; open both pages. | The sidebar and header are German; the table headings stay `Order ID`, `Channel`, …, `SL No`, `Item Name`, `Quantity`, `Amount`; the title stays `Dashboard Overview`. |
