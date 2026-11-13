# 03 — View a restaurant's overview — tester cases

Use case: [03-view-restaurant-overview.md](../03-view-restaurant-overview.md). Screen text: [ui-ux.md §3–4](../ui-ux.md#4-overview). Fixtures: [README.md](./README.md#fixtures).

Signed in as ADM-1; restaurant list → REST-A → `View Analytics`. Ask a developer to set up the order fixtures; "today" is the UTC date.

| ID | Covers | Steps | Expected |
|---|---|---|---|
| ADM-03-T01 | AC1 (ADM-D2) | With ORDER-LIVE and ORDER-PAID today, open the Overview (period `Today`). | `Total Revenue` `525.00`; `Total Orders` `2`; `Total Items Sold` `4`; `Avg Order Value` `₹262.50`. `Order Channel`: centre `2` `Total Orders`, row `Takeaway` `100%`. `Popular Items`: `01` `Paneer Tikka` `4 orders` `₹1,000.00`. `Recent Orders`: two rows, newest first; ORDER-PAID `1 items` `₹525.00` `Paid`; ORDER-LIVE `1 items` `Unpaid`. Revenue chart: one point for today, 525. |
| ADM-03-T02 | AC2 (ADM-D2) | Only ORDER-LIVE today. | `Total Revenue` `0.00`; `Total Orders` `1`; `Total Items Sold` `2`; `Avg Order Value` `₹0.00`; `Paneer Tikka` `2 orders` still listed; revenue chart empty. |
| ADM-03-T03 | AC3 (ADM-D2) | PAID-AT(7 days ago, 14:00 IST) and PAID-AT(8 days ago, 14:00 IST); no orders today. Pick each period. | `Today` and `Yesterday`: 0 orders. `Last 7 Days`: 1 order (the 7-days-ago one). `Last 15 Days` and `Last 30 Days`: 2 orders. |
| ADM-03-T04 | AC4 (ADM-D2) | PAID-AT(today 00:30 IST). | `Today`: 0 orders; `Yesterday`: 1 order. |
| ADM-03-T05 | Alt (ADM-D2) | With T01's data, pick `Last 30 Days`; read the subtitles. | Still `Daily revenue for the current month`, `Top performing menu items this week`, `Welcome back! Here's what's happening today.` |
| ADM-03-T06 | Alt (ADM-D3) | Click `Export Report`; click the bell. | Nothing happens. |
| ADM-03-T07 | Alt | Click each sidebar link, then `Overview`. | `Order History` / `Item History` open the ADM-04 pages; the zone, table, category, item, settings and user links open those features' panel pages for REST-A; `Overview` returns here. The `WizardComm` logo returns to the restaurant list. |
