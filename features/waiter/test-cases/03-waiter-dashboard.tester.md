# 03 — Waiter dashboard — tester cases

Use case: [03-waiter-dashboard.md](../03-waiter-dashboard.md). Screen text: [ui-ux.md §3](../ui-ux.md#3-waiter-dashboard). Fixtures: [README.md](./README.md#fixtures) (DASH-A, but with ORD-T1 **not yet paid** at the start).

Signed in as WTR-A. Open the dashboard from the drawer (`Operations` → `Dashboard`) or the tablet rail.

| ID | Covers | Steps | Expected |
|---|---|---|---|
| WAIT-03-T01 | AC1 (WAIT-D3) | Open the dashboard. | `Hello, <name>!` · `Here's your performance summary` · `Today's Overview` · `WAIT-<32 hex>`; `Total Orders` **5** (yesterday's orders included). Only that one card. |
| WAIT-03-T02 | AC2 | Look at `Your Top Selling Items` · `Today`. | `No items sold yet today` (no paid order today). |
| WAIT-03-T03 | AC2 | Ask a developer to mark ORD-T1 paid (status 2, `grandTotal` 525.00). On WTR-A's dashboard, pull down. | One row: `1` · `Paneer Tikka` · `2`. `Chicken 65` (ORD-T2, unpaid) isn't listed. No revenue column. |
| WAIT-03-T04 | AC1 | WTR-A sends a new Takeaway KOT; pull down on the dashboard. | `Total Orders` 6. |
| WAIT-03-T05 | AC3 | OWN-A sends a Takeaway KOT; pull down on WTR-A's dashboard. | `Total Orders` unchanged. |
