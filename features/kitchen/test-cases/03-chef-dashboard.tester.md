# 03 — Chef dashboard — tester cases

Use case: [03-chef-dashboard.md](../03-chef-dashboard.md). Screen text: [ui-ux.md §3](../ui-ux.md#3-chef-dashboard). Fixtures: [README.md](./README.md#fixtures).

CHF-A on DEV-1; a waiter on DEV-2 sends KOTs.

| ID | Covers | Steps | Expected |
|---|---|---|---|
| KITCH-03-T01 | AC1 | No KOTs today. Sign in as CHF-A; open the dashboard. | `Hello, <name>!` · `Here's your performance summary` · `Today's Overview` · `CHEF-<32 hex>`; cards `Received 0`, `Accepted 0`, `Rejected 0`. |
| KITCH-03-T02 | AC2 — *Fails until KITCH-D3 is fixed* | A waiter sends two KOTs; pull down on the dashboard. | `Received 2`, `Accepted 0`, `Rejected 0`. (Today: `0`, `0`, `0`.) |
| KITCH-03-T03 | AC2 — *Fails until KITCH-D1 is fixed*, *Fails until KITCH-D3 is fixed* | Accept the first KOT on Kitchen Orders; pull down on the dashboard. | `Received 2`, `Accepted 1`, `Rejected 0`. |
| KITCH-03-T04 | AC2 — *Fails until KITCH-D1 is fixed*, *Fails until KITCH-D3 is fixed* | Move that KOT on to Completed and Ready; pull down. | Still `Received 2`, `Accepted 1`, `Rejected 0` (it stays accepted). |
| KITCH-03-T05 | AC2 — *Fails until KITCH-D1 is fixed*, *Fails until KITCH-D3 is fixed* | Reject the second KOT; pull down. | `Received 2`, `Accepted 1`, `Rejected 1`. |
| KITCH-03-T06 | AC3 — *Fails until KITCH-D3 is fixed* | Sign in as CHF-A-LC; note the first screen; open the dashboard. | First screen is Kitchen Orders. The dashboard shows the chef cards `Received` / `Accepted` / `Rejected` and `CHEF-<32 hex>`. (Today: the waiter version — a single `Total Orders` card and `WAIT-<32 hex>`.) |
