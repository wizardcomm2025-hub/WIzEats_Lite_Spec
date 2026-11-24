# 01 — Choose a channel and a table — tester cases

Use case: [01-choose-channel-and-table.md](../01-choose-channel-and-table.md). Screen text: [ui-ux.md](../ui-ux.md). Fixtures: [README.md](./README.md#fixtures).

Signed in as WTR-A (phone) or OWN-A (tablet) on Home.

| ID | Covers | Steps | Expected |
|---|---|---|---|
| ORD-01-T01 | AC1 | Look at Home. Then switch Delivery off in Profile (as OWN-A) and look again. | Tiles `DINE IN`, `TAKEAWAY`, `ROOM SERVICE`, `DELIVERY`; after switching Delivery off, no `DELIVERY` tile. |
| ORD-01-T02 | AC2 | Tap `DINE IN`. | Zone `Main Hall`; chips `All 4`, `Available 4`; four green table cards. |
| ORD-01-T03 | AC3 | Tap `Table 1`; check the sheet; tap `Ready to Occupy`. Go back to the grid and wait for the refresh. | Sheet `Table 1` / `Capacity: 4` / `Ready to Occupy`; the menu opens. Grid: Table 1 amber, chips include `Occupied 1`. |
| ORD-01-T04 | AC4 | Long-press occupied `Table 1`; confirm leaving. | Confirmation shown; Table 1 green / `Available` again. |
| ORD-01-T05 | AC5 | Tap `TAKEAWAY`. | The ordering menu opens with no table. |
| ORD-01-T06 | AC7 (ORD-D2 fixed) | Occupy Table 2; send a KOT (02); go back to the grid; tap Table 2 again. | Table 2 is `Ordered` (red); tapping opens the **cart with that order** and its KOT. |
| ORD-01-T07 | Alt: switch guard | Put an item in the cart on Dine In; on the tablet rail tap Takeaway. | `Switch to Takeaway?` asks first; cancelling keeps the cart. |
| ORD-01-T08 | AC7 — Room Service (ROOM-D1, ORD-D2 fixed) | Tap `ROOM SERVICE`; pick `Ground Floor`; tap `101`; `Ready to Occupy`; send a KOT; go back; tap `101`. | Room grid with `101`, `102`; after the KOT `101` is `Ordered`; tapping it opens the cart with that order. |
