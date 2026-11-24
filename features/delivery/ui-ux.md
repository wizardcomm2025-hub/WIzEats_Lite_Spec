# Delivery board — UI / UX

Screen text exactly as in the code on 2026-11-11; re-checked in `POS_APP-kot` 2026-11-24. Only the parts testers check are listed.

## 1. Delivery Orders

| Part | Content |
|---|---|
| Entry | Home tile `DELIVERY` (owner and waiter; only while the Delivery channel is on); on a tablet the rail keeps `Home` highlighted. No `New Order` button (DLV-D4) |
| Title | `Delivery Orders` · `Accept, reject and track home delivery` (or `<n> order(s) in this list`) · bike icon |
| First load | `Loading delivery orders...` |
| Empty | `No Delivery Orders` · `No delivery orders in this section. Pull down to refresh.` |
| Load failure | Red toast with the server's message when the server answers with an HTTP error. Today, with the real channel ID, the server's failure comes back as a 200 and the board is just empty (DLV-D1) |
| Refresh | Pull down; automatic every 15 s while open and in the foreground |

## 2. Delivery card (never shown today — DLV-D1; shown once it is fixed)

| Part | Content |
|---|---|
| Header | `Order #<orderNumber>` · stage pill: `Pending` (amber), `Accepted` (blue), `Rejected` (red), `Completed` (green), `Out for Delivery` (primary) |
| Customer | Initial bubble · name or `Guest customer` · phone (tap to call) or `No phone number` |
| Address | Map pin + address (only when `orderChannel` is present in the row) |
| Items | quantity bubble + name |
| Total | `Order total` · `₹<grandTotal>` |
| Actions | Pending: `Reject`, `Accept` · Accepted: `Out For Delivery` · Out for Delivery: `Complete Payment` (target: opens Bill Payment — DLV-D3) · Rejected / Completed: none |
| Toasts | `Order #<orderId> accepted` / `rejected` (info) / `is out for delivery` / `payment completed`; `Order #<orderId> has no KOT to update`; failure: the server's message or `Failed to update delivery status` |

`<orderId>` in toasts is the order's 32-char hex ID (the app no longer converts it with `Number()`).
