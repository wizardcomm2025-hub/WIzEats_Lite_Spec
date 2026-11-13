# Waiter — UI / UX

Screen text exactly as in the code on 2026-11-11. Only the parts testers check are listed.

## 1. Waiter navigation

| Device | Entries |
|---|---|
| Phone drawer | `Operations`: `Dashboard`, `Ready Orders`, `Order History`, `Printer`, `User Management` |
| Tablet rail | Home, Tables (when enabled), Menu, Orders, `Dashboard`, `Ready`; footer Printer, Users |

## 2. Ready Orders

| Part | Content |
|---|---|
| Header | Back arrow · `Ready Orders` · `Orders prepared by kitchen` · count bubble (number of ready KOTs) |
| Tabs | `Pickup` · `Delivered`, each with a count badge when above 0 |
| Pickup empty / failed | `No Pending Pickups` · `Kitchen hasn't marked any orders as ready yet. New ones appear here automatically.` |
| Delivered empty | `No Deliveries Yet` · `Orders you deliver will appear here.` |
| Refresh | Pull down (Pickup); automatic every 15 s while open and in the foreground |

### Cards (never shown today — WAIT-D1)

| Part | Pickup card | Delivered card |
|---|---|---|
| Title | `KOT #<kotId>` · green `READY` · time ago | `KOT #<kotId>` · blue `DELIVERED` |
| Place | `<zone>  ·  <table>`, `Room: <room>` or `Takeaway` | same |
| Items | quantity bubble + name | same |
| Note | `📝 <note>` | same |
| Button | `Delivered to Customer`; spinner while sending; all buttons disabled meanwhile | — |

## 3. Waiter dashboard

| Part | Content |
|---|---|
| Header | `Hello, <name>!` · `Here's your performance summary` · `Today's Overview` · `WAIT-<userId hex>` |
| Card | `Total Orders` (WAIT-D3) |
| Table | `Your Top Selling Items` · `Today`; columns `ITEM`, `QUANTITY`; rank bubble 1–10 |
| Empty table | `No items sold yet today` |
| Loading | `Loading your dashboard...` |
| Refresh | Pull down |
