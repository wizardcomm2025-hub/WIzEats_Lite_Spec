# 01 — View the dashboard — tester cases

Use case: [01-view-dashboard.md](../01-view-dashboard.md). Screen text: [ui-ux.md §1](../ui-ux.md#1-dashboard). Fixtures: [README.md](./README.md#fixtures).

Signed in as OWN-A; drawer `Analytics` → `Dashboard` (phone) or rail `Dashboard` (tablet).

| ID | Covers | Steps | Expected |
|---|---|---|---|
| RPT-01-T01 | AC1 (RPT-D1) | Send a Takeaway KOT `Paneer Tikka` × 2 from the app. Open the Dashboard (`Today`). | `Total Revenue ₹0`, `Total Orders 0`, `Avg Order Value ₹0.00`, `Table Turnover Rate 0.0x`; `No Payments Yet`; `No Channel Data`; Top Selling: `Best Seller` `Paneer Tikka`, `2 units sold`, `₹500`. |
| RPT-01-T02 | AC3 | Tap `Yesterday`, then `Today`. | `Yesterday`: `Loading analytics data...`, then Top Selling `No Sales Data` (no orders yesterday). Back on `Today`: T01's figures again. |
| RPT-01-T03 | AC2 | Ask a developer to bill and pay T01's order (TOT-525, CASH received 600, change 75). Pull down. | `Total Revenue ₹525.00`, `Total Orders 1`, `Avg Order Value ₹525.00`; Revenue Breakdown Cash 525 (100 %), UPI 0; Order Channels: Takeaway 100 %. |
| RPT-01-T04 | Owner only | Sign in as WTR-A; open the drawer (phone) / look at the rail (tablet). | No `Analytics` group, and the waiter `Dashboard` opens the waiter dashboard (WAIT), not this one. |
