# Take an order & send KOTs — UI / UX

Screen text exactly as in the code on 2026-11-11; re-checked in `POS_APP-kot` 2026-11-24. Only the parts testers check are listed. The ordering menu and the cart have more visual detail than this.

## 1. Home

Channel tiles for the **active** channels: `DINE IN` always; `TAKEAWAY`, `ROOM SERVICE`, `DELIVERY` when switched on in the settings (RST-D1 fixed).

## 2. Table grid (Dine In) and room grid (Room Service)

| Part | Content |
|---|---|
| Header | Zone picker (active zones) / floor picker |
| Filter chips | `All` + one per status present: `Available`, `Occupied`, `Ordered` — each with a count, in the status colour (green / amber / red from the server) |
| Card | Table / room name, capacity, status colour |
| Free → sheet | Name · `Capacity: <n>` · button `Ready to Occupy` |
| Occupied, no order | Opens the menu straight away (a new order) |
| Ordered (running order) | Opens the cart with the order's KOTs (ORD-D2 fixed) |
| Long press, occupied, no order | Leave confirmation |
| Empty filter | `No <filter> tables found` |

## 3. Cart and KOT

| Event | What staff see |
|---|---|
| Nothing ticked | Toast `Tick at least one item to send on this KOT.` |
| First KOT sent | Toast `KOT generated successfully!` (plus ` <n> held item(s) still in the cart.` when some are held back); the KOT card / modal with the order number, KOT number, note and items |
| Further KOT | Toast `Items added to KOT successfully!` |
| Server error | Toast with the server's message (e.g. `Something went wrong`, `Bill already generated, cannot add items to this order`; target also `Menu item not available`, `This table already has a running order`, `Customer name is required for delivery orders`), or `Failed to generate KOT` |
| No reply | Toast `No reply from the server. The KOT may already have reached the kitchen — check before sending it again.` |
| Unexpected | Toast `Something went wrong. Please try again.` |
| Takeaway, tablet `New Order` | Toast `Order #<number> saved — collect it from Order History.` |
| Channel change with items | `Switch to <channel>?` confirmation |
| Re-opened order | The order's earlier KOTs and lines are shown (ORD-D2 fixed); an unknown order → `Failed to fetch order` |
| Customer fields | Today never shown (the app checks channel IDs `0`–`2`). *Target (ORD-D3):* the existing customer block (placeholders `Enter customer name...` / `Enter phone number...`, compact `Name` / `10-digit phone`) for Takeaway (optional) and Delivery (required: `Customer name is required`, `Customer phone is required`) |
