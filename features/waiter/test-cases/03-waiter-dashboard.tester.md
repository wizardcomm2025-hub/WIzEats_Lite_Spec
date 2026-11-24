# 03 — Waiter dashboard — tester cases

Use case: [03-waiter-dashboard.md](../03-waiter-dashboard.md). Screen text: [ui-ux.md §3](../ui-ux.md#3-waiter-dashboard). Fixtures: [README.md](./README.md#fixtures).

DASH-A set up; WTR-A signed in.

| ID | Covers | Steps | Expected |
|---|---|---|---|
| WAIT-03-T01 | AC1 — *Fails until WAIT-D3 is fixed* | Open the dashboard. | `Hello, <name>!` · `Here's your performance summary` · `Today's Overview` · `WAIT-<32 hex>`; `Total Orders` **2** (today's only). Only that one card. (Today: 5 — yesterday's included.) |
| WAIT-03-T02 | AC2 | Ask a developer to set ORD-T1 back to unpaid first; look at `Your Top Selling Items` · `Today`. | `No items sold yet today`. |
| WAIT-03-T03 | AC2 | Ask a developer to mark ORD-T1 paid again (status 2, `grandTotal` 525.00). Pull down. | One row: `1` · `Paneer Tikka` · `2`. `Chicken 65` (ORD-T2, unpaid) isn't listed. No revenue column. |
| WAIT-03-T04 | AC1 — *Fails until WAIT-D3 is fixed* | WTR-A sends a new Takeaway KOT; pull down on the dashboard. | `Total Orders` 3. (Today: 6.) |
| WAIT-03-T05 | AC3 | OWN-A sends a Takeaway KOT; pull down on WTR-A's dashboard. | `Total Orders` unchanged. |
