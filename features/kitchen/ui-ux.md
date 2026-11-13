# Kitchen (chef) — UI / UX

Screen text exactly as in the code on 2026-11-11. Only the parts testers check are listed.

## 1. Chef shell

| Part | Content |
|---|---|
| After sign-in | Opens **Kitchen Orders** (role `chef` in any case) |
| Footer | All buttons greyed out (half opacity); tapping does nothing |
| Drawer | Section `Kitchen`: `KOT`, `Dashboard` |

## 2. Kitchen Orders

| Part | Content |
|---|---|
| Title | `Kitchen Orders` · `Manage and track food preparation` · flame icon |
| Tabs | `New` · `Preparing` · `Completed` · `Ready` — no count badges (always 0) |
| Loading (tab change) | `Loading KOTs...` |
| Empty / failed list | `No Orders Here` · `No orders in this section. Pull down to refresh.` |
| Refresh | Pull down; also automatic every 15 s while open and in the foreground |

### KOT card (never shown today — KITCH-D1)

| Part | Content |
|---|---|
| Title | `KOT #<kotId>` · status pill (`Pending` blue, `Preparing` amber, `Rejected` red, `Completed` accent, `Ready` green) · `Just now` / `<n>m ago` / `<h>h <m>m ago` |
| Place | `Zone : <zone>  <table>`, or `Room: <room>`, or `N/A` |
| Items | quantity bubble + item name, one row per line |
| Note | `📝 <note>` when there is one |
| Pending | `Reject` (red) and `Accept` |
| Preparing / Completed | `Mark Completed` / `Mark Ready` with an arrow |
| Ready | no button |
| While sending | spinner on the tapped button; all buttons disabled, other cards dimmed |
| After success | the card leaves the tab |
| On failure | nothing shown; the card stays |

## 3. Chef dashboard

| Part | Content |
|---|---|
| Header | `Hello, <name>!` · `Here's your performance summary` · `Today's Overview` · `CHEF-<userId hex>` |
| Cards (role exactly `Chef`) | `Received` · `Accepted` · `Rejected` |
| Role `chef` / `CHEF` | Waiter dashboard: `Total Orders`; header `WAIT-<userId>` (KITCH-D3) |
| Loading | `Loading your dashboard...` |
| Refresh | Pull down |
