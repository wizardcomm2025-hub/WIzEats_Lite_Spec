# Take an order & send KOTs — UI / UX

Screen text exactly as in the code on 2026-11-11. Only the parts testers check are listed. The ordering menu and the cart have more visual detail than this.

## 1. Home

Channel tiles: `DINE IN`, `TAKEAWAY`, `ROOM SERVICE`, `DELIVERY` (all four always — RST-D1).

## 2. Table grid (Dine In)

| Part | Content |
|---|---|
| Header | Zone picker (active zones) |
| Filter chips | `All` + one per status present: `Available`, `Occupied`, `Ordered` — each with a count, in the status colour (green / amber / red from the server) |
| Card | Table name, capacity, status colour |
| Free table → sheet | Table name · `Capacity: <n>` · button `Ready to Occupy` |
| Occupied table, no order | Opens the menu straight away (a new order) |
| Long press, occupied, no order | Leave-table confirmation |
| Empty filter | `No <filter> tables found` |

## 3. Cart and KOT

| Event | What staff see |
|---|---|
| Nothing ticked | Toast `Tick at least one item to send on this KOT.` |
| First KOT sent | Toast `KOT generated successfully!` (plus ` <n> held item(s) still in the cart.` when some are held back); the KOT card / modal with the order number, KOT number, note and items |
| Further KOT (same visit) | Toast `Items added to KOT successfully!` |
| Server error | Toast with the server's message (e.g. `Something went wrong`, `Bill already generated, cannot add items to this order`), or `Failed to generate KOT` |
| No reply | Toast `No reply from the server. The KOT may already have reached the kitchen — check before sending it again.` |
| Unexpected | Toast `Something went wrong. Please try again.` |
| Takeaway, tablet `New Order` | Toast `Order #<number> saved — collect it from Order History.` |
| Channel change with items | `Switch to <channel>?` confirmation |
| Re-opened order (today) | The earlier KOTs aren't shown; `Failed to fetch order` (ORD-D2) |
