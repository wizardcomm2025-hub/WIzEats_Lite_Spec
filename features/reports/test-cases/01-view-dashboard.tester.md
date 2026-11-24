# 01 — View the dashboard — tester cases

Use case: [01-view-dashboard.md](../01-view-dashboard.md). Screen text: [ui-ux.md §1](../ui-ux.md#1-dashboard). Fixtures: [README.md](./README.md#fixtures).

Signed in as OWN-A; drawer `Analytics` → `Dashboard` (phone) or rail `Dashboard` (tablet).

| ID | Covers | Steps | Expected |
|---|---|---|---|
| RPT-01-T01 | AC1 | Send a Takeaway KOT `Paneer Tikka` × 2 from the app (don't pay). Open the Dashboard (`Today`). | `Total Revenue ₹0`, `Total Orders 0`, `Avg Order Value ₹0.00`, `Table Turnover Rate 0.0x`; `No Payments Yet`; `No Channel Data`; Top Selling `No Sales Data`. |
| RPT-01-T02 | AC3 | Tap `Yesterday`, then `Today`. | `Yesterday`: `Loading analytics data...`, then Top Selling `No Sales Data` (no orders yesterday). Back on `Today`: T01's figures again. |
| RPT-01-T03 | AC2 | Pay T01's order in the app (BILL 02): CASH, received ₹600. Pull down on the Dashboard. | `Total Revenue ₹525.00`, `Total Orders 1`, `Avg Order Value ₹525.00`; Revenue Breakdown Cash 525 (100 %), UPI 0; Top Selling `Paneer Tikka` `2 units sold`; Order Channels: Takeaway 100 %. |
| RPT-01-T04 | Owner only | Sign in as WTR-A; open the drawer (phone) / look at the rail (tablet). | No `Analytics` group, and the waiter `Dashboard` opens the waiter dashboard (WAIT), not this one. (The API refusal is RPT-01-D09.) |
