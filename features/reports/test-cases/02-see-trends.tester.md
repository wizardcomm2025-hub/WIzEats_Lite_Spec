# 02 — See revenue and order trends — tester cases

Use case: [02-see-trends.md](../02-see-trends.md). Screen text: [ui-ux.md §1](../ui-ux.md#1-dashboard). Fixtures: [README.md](./README.md#fixtures).

Signed in as OWN-A on the Dashboard.

| ID | Covers | Steps | Expected |
|---|---|---|---|
| RPT-02-T01 | AC1 | No paid orders this year. Scroll to the trends; tap `Daily`, `Weekly`, `Monthly`. | Revenue Trend: `No Data Available` · `Transactions will appear here once recorded.`; Order Trend: `No Order Data` · `Order trends will appear here once orders are placed.` — for all three. |
| RPT-02-T02 | AC2 / AC3 | Pay one order today in the app. Pull down; tap `Daily`, `Weekly`, `Monthly`. | `Daily`: one point for today's day name; `Weekly`: one point `Week <CEIL(day/7)>`; `Monthly`: one point for this month; each with the order's revenue and 1 order. |
