# 03 — Chef dashboard — tester cases

Use case: [03-chef-dashboard.md](../03-chef-dashboard.md). Screen text: [ui-ux.md §3](../ui-ux.md#3-chef-dashboard). Fixtures: [README.md](./README.md#fixtures).

Chef device DEV-1. Open the dashboard from the drawer: `Kitchen` → `Dashboard`. T03–T05 need a developer to change KOT-P with the raw ID (KITCH-02-D02 style) today, then the tester pulls down to refresh.

| ID | Covers | Steps | Expected |
|---|---|---|---|
| KITCH-03-T01 | AC1 | Sign in as CHF-A; open the dashboard. | `Hello, <name>!` · `Here's your performance summary` · `Today's Overview` · `CHEF-<32 hex>`; cards `Received 0`, `Accepted 0`, `Rejected 0`. |
| KITCH-03-T02 | AC1 (KITCH-D1) | A waiter sends two KOTs; pull down on the dashboard. | Still `0`, `0`, `0`. |
| KITCH-03-T03 | AC2 (KITCH-D3) | Developer moves KOT-P to Preparing as CHF-A; pull down. | `Received 1`, `Accepted 1`, `Rejected 0`. |
| KITCH-03-T04 | AC2 (KITCH-D3) | Developer moves KOT-P on to Completed as CHF-A; pull down. | `Received 1`, `Accepted 0`, `Rejected 0`. |
| KITCH-03-T05 | AC2 (KITCH-D3) | Developer moves KOT-P to Rejected as CHF-A; pull down. | `Received 1`, `Accepted 0`, `Rejected 1`. |
| KITCH-03-T06 | AC3 (KITCH-D3) | Sign in as CHF-A-LC; note the first screen; open the dashboard. | First screen is Kitchen Orders. The dashboard shows the waiter version: a single `Total Orders` card and `WAIT-<32 hex>`. |
