# Waiter (ready orders & dashboard)

A waiter takes orders (ORD) and serves the food the kitchen has finished. **Ready Orders** lists the KOTs the chef marked Ready; the waiter taps `Delivered to Customer` once the food reaches the table. The waiter also has a dashboard: Total Orders and their top-selling items today.

The spec records today's behaviour plus targets (re-synced 2026-11-24 against API commit d54d6c8 and `POS_APP-kot`; no WAIT code changed). **Today no KOT ever appears in Ready Orders: a KOT can't reach Ready from the app (KITCH-D1), and the list fails with a 500 whenever one is Ready (WAIT-D1).** WAIT-D1 to WAIT-D3 are *To fix*; most cases assert the target.

**Feature code:** `WAIT` (test IDs `WAIT-01-T01`, `WAIT-02-D04`, …)

## Use cases

| # | Use case | Actor | Tester file | Developer file |
|---|---|---|---|---|
| 01 | [See ready orders](./01-see-ready-orders.md) | Waiter (today any role through the API — WAIT-D2) | [tester](./test-cases/01-see-ready-orders.tester.md) | [developer](./test-cases/01-see-ready-orders.developer.md) |
| 02 | [Mark a KOT delivered](./02-mark-kot-delivered.md) | same | [tester](./test-cases/02-mark-kot-delivered.tester.md) | [developer](./test-cases/02-mark-kot-delivered.developer.md) |
| 03 | [Waiter dashboard](./03-waiter-dashboard.md) | same | [tester](./test-cases/03-waiter-dashboard.tester.md) | [developer](./test-cases/03-waiter-dashboard.developer.md) |

## Key concepts

- **Ready KOT** — `kots.status` 4 and `isPickedUp` 0 (stages: [KITCH](../kitchen/README.md#key-concepts)).
- **Picked up / delivered** — `kots.isPickedUp` 1 with `pickedUp_at`. The app calls it "Delivered to Customer"; the API calls it "picked up".
- **Delivered tab** — today the app's own list of KOTs confirmed on this device since the app started, not loaded from the server; target: today's picked-up KOTs of the restaurant, loaded from the server (WAIT-D2).
- **Fulfilled order** — `orders.orderStatus` 2, set when the bill is paid (BILL).

## Documents

- [data-model.md](./data-model.md) · [api-contract.md](./api-contract.md) · [ui-ux.md](./ui-ux.md) · [test-cases/README.md](./test-cases/README.md) · [developer-guide.html](./developer-guide.html)
- Implementation: `laravel-pos-api/api/specs/waiter/` (created with the first plan)

## Depends on

- **KITCH** — moves KOTs to Ready.
- **ORD** — creates the orders and KOTs (`createdByUserId` drives the dashboard).
- **BILL** — sets orders to fulfilled, which the revenue and top-selling counts need.
- **AUTH / STAFF** — the waiter role; the app shows the waiter menus for role `waiter` in any case.

## Decisions

Statuses per [conventions.md → Decision status](../../conventions.md#decision-status), set 2026-11-24. Cross-cutting items: [project-overview.md → Known gaps and targets](../../project-overview.md#known-gaps-and-targets).

| ID | Status | Short | Tests |
|---|---|---|---|
| WAIT-D1 | **To fix** (with KITCH-D1, X-D4, X-D5) | Ready list 500s; pick-up never matches; card shows the KOT ID | 01-D02, 01-D09, 01-D10, 02-D01, 02-D07, 02-D13; 01-T03, 01-T04, 02-T01 |
| WAIT-D2 | **To fix** (with X-D1) | Pick-up has no rules and leaks every restaurant's KOTs; Delivered tab in memory only | 01-D05, 02-D02–D06, 02-D09, 02-D14, 02-D15, 03-D04; 02-T02, 02-T03 |
| WAIT-D3 | **To fix** | `Total Orders` counts every order ever | 03-D01, 03-D02; 03-T01, 03-T04 |
| AUTH-D2 (inherited) | **To fix** (X-D2) | Switched-off / unapproved callers keep access | 01-D06, 02-D10, 03-D06 |

- **WAIT-D1 — To fix (2026-11-24, was accepted 2026-11-11), with KITCH-D1.** *Target:* `ready-kots` returns hex IDs (no raw bytes) and adds `kotNumber` and `orderNumber`; `updatePickedUpStatus/{kotId}` finds the KOT by hex ID (dashes allowed) among the caller's restaurant's KOTs → an unknown, other restaurant's or malformed ID → 404 `{ "status": false, "message": "KOT not found", "data": null }`; 500s carry no `error` key (X-D5). The app's cards show `KOT #<kotNumber>`.
- **WAIT-D2 — To fix (2026-11-24, was accepted 2026-11-11).** *Target:* the three `/api/waiter/*` endpoints are waiter only (X-D1: 403 `You do not have permission to perform this action`). Pick-up accepts only a Ready (status 4) KOT that isn't picked up yet → a KOT in another stage → 409 `KOT is not ready for pickup`; one already picked up → 409 `KOT already picked up`; nothing written. Its reply is that KOT only: `data: { kotId, kotNumber, pickedUp_at }`. `ready-kots` with `{ "pickedUp": true }` returns the restaurant's KOTs picked up today (newest pick-up first); the app's `Delivered` tab loads it, so it is shared by all devices and survives a restart.
- **WAIT-D3 — To fix (2026-11-24, was accepted 2026-11-11).** *Target:* every dashboard value covers this waiter's orders **created today** (server date): `totalOrders` = their orders today, any status; `totalRevenue` = `grandTotal` of those fulfilled; `averageOrderValue` = `totalRevenue / totalOrders`; `tableTurnoverRate` unchanged; `topSellingItems` unchanged (today's fulfilled orders — the same paid-only rule as RPT).
- **Inherited:** KITCH-D1 (To fix — nothing reaches Ready from the app until then); KITCH-D3 (target: a chef saved as `chef` gets the chef dashboard); ORD-D2 fixed (app orders carry their table or room, so a ready card shows its place); AUTH-D2 (To fix, X-D2); AUTH-D5 / X-D5.

Original decision texts (history):

- **WAIT-D1 (2026-11-11):** The GUID faults stay:
  - `POST /api/waiter/ready-kots` puts raw 16-byte `kotId`s into the JSON, so any non-empty list answers 500 `An error occurred while fetching pending kots` with `error` `Malformed UTF-8 characters, possibly incorrectly encoded`. An empty list answers 200.
  - `POST /api/waiter/updatePickedUpStatus/{kotId}` compares the hex ID with the binary column and updates no row.
  - So Ready Orders always shows `No Pending Pickups`, and `Delivered to Customer` can't be tapped in practice.

  Accepted.
- **WAIT-D2 (2026-11-11):** The pick-up endpoint has no rules:
  - no role, restaurant or status check: with a matching ID (the 16 raw bytes, percent-encoded) any KOT of any restaurant, in any stage, is marked picked up, again and again (`pickedUp_at` moves each time);
  - its reply lists **every** picked-up KOT of **every** restaurant, as full `kots` rows. Today that list contains raw IDs, so whenever any picked-up KOT exists anywhere the reply is 500 `An error occurred while update kot picked up status ` (the update itself has already happened). If the encoding ever succeeds, other restaurants' KOTs are disclosed;
  - the app's Delivered tab lives only in memory: it is lost when the app restarts or the user signs out, and it isn't shared between devices.

  Accepted risk.
- **WAIT-D3 (2026-11-11):** The waiter dashboard stays as it is:
  - `Total Orders`, shown under `Today's Overview`, counts **all** orders this user ever created in the restaurant, not just today's;
  - `Your Top Selling Items · Today` counts only today's **fulfilled** orders created by this user (up to 10 items);
  - revenue, average order value and table turnover are returned but hidden in the app. Revenue counts fulfilled orders only; the average divides it by all orders.

  Accepted.
- **Inherited (2026-11-11):** KITCH-D1 (nothing reaches Ready from the app), KITCH-D3 (a chef saved as `chef` sees this dashboard), ORD-D2 (app orders have no table, so a ready KOT would show `Takeaway`), AUTH-D2, AUTH-D5.

## Cross-cutting open questions

- None open.

## Feature-wide out of scope

- Taking orders — ORD. The owner's dashboard — RPT. Order history — HIST.
- The owner has no Ready Orders screen; today the API doesn't stop an owner calling these endpoints (target: 403 — WAIT-D2).
