# Delivery board — UI / UX

Screen text exactly as in the code on 2026-11-11. Only the parts testers check are listed.

## 1. Delivery Orders

| Part | Content |
|---|---|
| Entry | Home tile `DELIVERY` (owner and waiter); on a tablet the rail keeps `Home` highlighted |
| Title | `Delivery Orders` · `Accept, reject and track home delivery` (or `<n> order(s) in this list`) · bike icon |
| First load | `Loading delivery orders...` |
| Empty | `No Delivery Orders` · `No delivery orders in this section. Pull down to refresh.` |
| First-load failure | Red toast with the server's message (today `hex2bin(): Hexadecimal input must have an even length` — DLV-D1) |
| Refresh | Pull down; automatic every 15 s while open and in the foreground; failures after the first load are silent |

## 2. Delivery card (never shown today — DLV-D1)

| Part | Content |
|---|---|
| Header | `Order #<orderNumber>` · stage pill: `Pending` (amber), `Accepted` (blue), `Rejected` (red), `Completed` (green), `Out for Delivery` (primary) |
| Customer | Initial bubble · name or `Guest customer` · phone (tap to call) or `No phone number` |
| Address | Map pin + address (only when `orderChannel` is present in the row) |
| Items | quantity bubble + name |
| Total | `Order total` · `₹<grandTotal>` |
| Actions | Pending: `Reject`, `Accept` · Accepted: `Out For Delivery` · Out for Delivery: `Complete Payment` · Rejected / Completed: none |
| Toasts | `Order #<orderId> accepted` / `rejected` (info) / `is out for delivery` / `payment completed`; `Order #<orderId> has no KOT to update`; failure: the server's message or `Failed to update delivery status` |

`<orderId>` in toasts is `Number(<hex>)`, i.e. `NaN` for most IDs.
