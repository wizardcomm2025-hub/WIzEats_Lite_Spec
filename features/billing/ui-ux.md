# Bill & payment — UI / UX

Screen text exactly as in the code on 2026-11-11. The bill screens can't be opened today (BILL-D2), so only the entry points and what testers see there are listed; the screens themselves are summarised.

## 1. Entry points

| Where | Control | Today |
|---|---|---|
| Takeaway KOT sheet (phone) / KOT banner (tablet) | `Collect Payment` (green) | Tapping does nothing |
| Cart with a loaded order | `View Bill` | Never shown: the cart can't load an existing order (ORD-D2) |
| Order History → Live | bill button on a row | Spinner on the row, then nothing (the order can't be loaded) |

## 2. Bill Breakdown (summary)

The order's lines, subtotal, tax at the order's rate, and the discount entry (when the discount feature is on in Profile). Continues to Bill Payment.

## 3. Bill Payment (summary)

| Part | Content |
|---|---|
| Receipt | Lines, subtotal, discount, tax, round-off, grand total; bill number once saved; UPI QR while money is owed or a UPI leg was used |
| Payment dock | Methods `CASH`, `UPI`; amount pre-filled with what is owed; quick-cash amounts; change due for cash; split legs list |
| States | `Already paid`, `Bill not up to date` (`Retry`), `Bill has no items`, `Bill not ready` |
| Save mismatch | `The bill was saved at ₹<x>, not ₹<y>. Check the amount and confirm again.` |
| Success | Payment success dialog; `Done` (or `Next Order` for takeaway) |
