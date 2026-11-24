# 01 — See ready orders — tester cases

Use case: [01-see-ready-orders.md](../01-see-ready-orders.md). Screen text: [ui-ux.md §2](../ui-ux.md#2-ready-orders). Fixtures: [README.md](./README.md#fixtures).

WTR-A signed in; CHF-A on another device moves KOTs (KITCH), or a developer sets them Ready by SQL.

| ID | Covers | Steps | Expected |
|---|---|---|---|
| WAIT-01-T01 | AC1 | Phone: drawer → `Operations` → `Ready Orders`. REST-A has no Ready KOT. | `Ready Orders` · `Orders prepared by kitchen`, count bubble `0`; `Pickup` tab: `No Pending Pickups` · `Kitchen hasn't marked any orders as ready yet. New ones appear here automatically.` |
| WAIT-01-T02 | AC1 | Tablet: tap the `Ready` rail button. | Same screen as T01. |
| WAIT-01-T03 | AC2 — *Fails until WAIT-D1 is fixed* | Send a Dine In KOT on Table 1 (`Paneer Tikka` × 2, note `less spicy`); the chef marks it Ready (or a developer by SQL). Stay on `Pickup` for 20 s. | A green card appears by itself: `KOT #<kot number>`, `READY`, `Main Hall  ·  Table 1`, `Paneer Tikka` × 2, `📝 less spicy`, `Delivered to Customer`; count bubble `1`; `Pickup` badge `1`. (Today: still `No Pending Pickups`.) |
| WAIT-01-T04 | AC2 — *Fails until WAIT-D1 is fixed* | A second KOT (Takeaway) becomes Ready; pull down on `Pickup`. | Two cards, newest first; the Takeaway card's place is `Takeaway`; count `2`. |
| WAIT-01-T05 | AC1 | Tap `Delivered` with nothing delivered today. | `No Deliveries Yet` · `Orders you deliver will appear here.` |
