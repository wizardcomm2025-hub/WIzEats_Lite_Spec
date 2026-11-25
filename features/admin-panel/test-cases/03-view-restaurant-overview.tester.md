# 03 — View a restaurant's overview — tester cases

Use case: [03-view-restaurant-overview.md](../03-view-restaurant-overview.md). Screen text: [ui-ux.md §3–4](../ui-ux.md#4-overview). Fixtures: [README.md](./README.md#fixtures).

Signed in as ADM-1; restaurant list → REST-A → `View Analytics`. Ask a developer to set up the order fixtures; "today" is the IST date (see the [README](./README.md#fixtures)).

| ID | Covers | Steps | Expected |
|---|---|---|---|
| ADM-03-T01 | AC1 — *Fails until ADM-D2 is fixed* | With ORDER-LIVE and ORDER-PAID today, open the Overview (period `Today`). | `Total Revenue` `₹525.00`; `Total Orders` `1`; `Total Items Sold` `2`; `Avg Order Value` `₹525.00`. `Order Channel`: centre `1` `Total Orders`, row `Takeaway` `100%`. `Popular Items`: `01` `Paneer Tikka` `2 orders` `₹500.00`. `Recent Orders`: two rows, newest first; ORDER-PAID `1 items` `₹525.00` `Paid`; ORDER-LIVE `1 items` `Unpaid`. Revenue chart: one point for today, 525. (Today: `525.00`, `2`, `4`, `₹262.50`; centre `2`; `4 orders` `₹1,000.00`.) |
| ADM-03-T02 | AC2 — *Fails until ADM-D2 is fixed* | Only ORDER-LIVE today. | `Total Revenue` `₹0.00`; `Total Orders` `0`; `Total Items Sold` `0`; `Avg Order Value` `₹0.00`; `Popular Items` and `Order Channel` empty; revenue chart one point at 0; `Recent Orders` lists ORDER-LIVE `Unpaid`. (Today: `Total Orders` `1`, `Total Items Sold` `2`, `Paneer Tikka` `2 orders` listed, chart empty.) |
| ADM-03-T03 | AC3 — *Fails until ADM-D2 is fixed* | PAID-AT(6 days ago, 14:00 IST) and PAID-AT(7 days ago, 14:00 IST); no orders today. Pick each period. | `Today` and `Yesterday`: 0 orders. `Last 7 Days`: 1 order (the 6-days-ago one); the chart has 7 points. `Last 15 Days` and `Last 30 Days`: 2 orders. (Today: `Last 7 Days` 2 orders.) |
| ADM-03-T04 | AC4 — *Fails until ADM-D2 is fixed* | PAID-AT(today 00:30 IST). | `Today`: 1 order; `Yesterday`: 0. (Today: `Today` 0, `Yesterday` 1.) |
| ADM-03-T05 | Alt — *Fails until ADM-D2 is fixed* | With T01's data, pick `Last 30 Days`; read the subtitles. Then switch to `Deutsch`. | `Welcome back! Here's what happened: Last 30 Days.`, `Daily revenue — Last 30 Days`, `Top performing menu items — Last 30 Days`; in German the same texts translated, with the German period label. (Today: always `Welcome back! Here's what's happening today.`, `Daily revenue for the current month`, `Top performing menu items this week`.) |
| ADM-03-T06 | Alt (ADM-D3) | Click `Export Report`; click the bell. | Nothing happens. |
| ADM-03-T07 | Alt | Click each sidebar link, then `Overview`. | `Order History` / `Item History` open the ADM-04 pages; the zone, table, category, item, settings and user links open those features' panel pages for REST-A; `Overview` returns here. The `WizardComm` logo returns to the restaurant list. |
| ADM-03-T08 | AC5 — *Fails until ADM-D2 is fixed* | ITEMS-12. Open the Overview (`Today`). | `Popular Items` shows 10 rows ranked `01` … `10`, the item with quantity 12 first; the two smallest are not listed. (Today: 12 rows, ranked `01` … `09`, `010`, `011`, `012`.) |
