# Bill & payment — UI / UX

Screen text exactly as in the code on 2026-11-11; re-checked in `POS_APP-kot` 2026-11-24. The bill screens open from the app since the BILL-D2 fix; only the parts testers check are listed.

## 1. Entry points

| Where | Control | Opens |
|---|---|---|
| Cart with a loaded order (Dine In / Room Service) | `View Bill` | Bill Breakdown, then Bill Payment (a saved bill goes straight to Bill Payment) |
| Takeaway KOT sheet (phone) / KOT banner (tablet) | `Collect Payment` (green) | Bill Payment (with the discount on the page) |
| Order History → Live | bill button on a row (phone) / `Bill Payment` in the detail pane (tablet) | Bill Payment; spinner on the row while the order loads |
| Delivery board | `Complete Payment` | today a status change only; target Bill Payment (DLV-D3) |

## 2. Bill Breakdown

The order's lines, subtotal, tax at the order's rate, and the discount entry (when the discount switch is on in Profile). Applying a discount shows a short celebration: `Discount Applied!` · `−₹<amount>` with confetti. Continues to Bill Payment.

## 3. Bill Payment

| Part | Content |
|---|---|
| Receipt | Lines, subtotal, discount, tax, round-off, grand total; bill number once saved; UPI QR while money is owed or a UPI leg was used |
| Payment dock | Methods `CASH`, `UPI`; amount pre-filled with what is owed; quick-cash amounts; change due for cash; split legs list |
| States | `Already paid`, `Bill not up to date` (`Retry`), `Bill has no items`, `Bill not ready` |
| Save mismatch | `The bill was saved at ₹<x>, not ₹<y>. Check the amount and confirm again.` |
| Server refusal | Toast with the server's message (target e.g. `Payment total does not match the bill`, `Order is already paid`) |
| Success | Payment success dialog; `Done` (or `Next Order` for takeaway) |
| Customer display (terminals that have one) | The total to pay while the bill is open; the change after a cash payment with change; blank when the page closes |
