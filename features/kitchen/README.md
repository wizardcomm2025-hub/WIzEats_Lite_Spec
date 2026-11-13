# Kitchen (chef)

A chef signs in and lands on **Kitchen Orders**: the KOTs sent from ordering (ORD), in four tabs by stage (New, Preparing, Completed, Ready). The chef accepts or rejects a new KOT, then moves it on to Completed and Ready. When a KOT is Ready, the waiter picks it up (WAIT). The chef also has a small dashboard with today's Received / Accepted / Rejected counts.

The spec records today's behaviour. Decisions KITCH-D1 to KITCH-D3 keep the current rules as accepted risks. **Today the chef's KOT list fails with a 500 whenever at least one KOT exists, and a status change never reaches the database (KITCH-D1). So in the app the kitchen never sees a KOT.**

**Feature code:** `KITCH` (test IDs `KITCH-01-T01`, `KITCH-02-D04`, …)

## Use cases

| # | Use case | Actor | Tester file | Developer file |
|---|---|---|---|---|
| 01 | [See KOTs by stage](./01-see-kots-by-stage.md) | Chef (any role through the API — KITCH-D2) | [tester](./test-cases/01-see-kots-by-stage.tester.md) | [developer](./test-cases/01-see-kots-by-stage.developer.md) |
| 02 | [Move a KOT through its stages](./02-move-kot-through-stages.md) | same | [tester](./test-cases/02-move-kot-through-stages.tester.md) | [developer](./test-cases/02-move-kot-through-stages.developer.md) |
| 03 | [Chef dashboard](./03-chef-dashboard.md) | same | [tester](./test-cases/03-chef-dashboard.tester.md) | [developer](./test-cases/03-chef-dashboard.developer.md) |

## Key concepts

- **KOT status** — `kots.status`: 0 Pending, 1 Preparing, 2 Rejected, 3 Completed, 4 Ready. The server sends the label text (`Pending` …) in lists; the app sends the number on updates.
- **Stage path in the app** — Pending → (Accept) Preparing → (Mark Completed) Completed → (Mark Ready) Ready. Reject only on a Pending KOT. The server enforces none of this (KITCH-D2).
- **Chef on a KOT** — `kots.chefId`, set to whoever last changed the status. Stage times: `preparing_at`, `rejected_at`, `completed_at`.
- **Picked up** — `kots.isPickedUp`, set by the waiter (WAIT), not by the chef.

## Documents

- [data-model.md](./data-model.md) · [api-contract.md](./api-contract.md) · [ui-ux.md](./ui-ux.md) · [test-cases/README.md](./test-cases/README.md) · [developer-guide.html](./developer-guide.html)
- Implementation: `laravel-pos-api-dev_guid/specs/kitchen/` (created with the first plan)

## Depends on

- **ORD** — creates the KOTs and their `order_items` (the list reaches the restaurant only through `order_items.restaurantId`).
- **STAFF** — chef users; the app saves the role as `Chef` (capital C).
- **AUTH** — PIN sign-in; the app routes a chef to Kitchen Orders.
- **WAIT** — reads Ready KOTs and marks them picked up.

## Decisions

- **KITCH-D1 (2026-11-11):** The GUID faults in the kitchen stay:
  - `POST /api/chef/pending-kots` puts the raw 16-byte `kotId` into the JSON. Laravel can't encode it as UTF-8, so any non-empty list answers 500 `An error occurred while fetching pending kots` with `error` `Malformed UTF-8 characters, possibly incorrectly encoded`. An empty list answers 200. In the very rare case that every listed ID happens to be valid UTF-8, a 200 comes back with garbled IDs.
  - `POST /api/chef/update-status/{kotId}` compares the hex ID with the binary column, updates no row, and still answers 200 `KOT status updated successfully`.
  - So the app's KOT tabs only ever show `No Orders Here`, and the chef's dashboard counts stay at 0.

  Accepted.
- **KITCH-D2 (2026-11-11):** The status endpoint has no rules:
  - no role check (an owner or waiter can call it) and no restaurant check: the lookup is by `kotId` alone, so if the ID matched (for example, the 16 raw bytes sent percent-encoded in the URL), any restaurant's KOT would change;
  - any integer is accepted, and the order of stages isn't enforced (Ready → Pending, Ready → Rejected, Rejected → Preparing);
  - a value outside 0–4 is written first and then answers 500 `Something went wrong` (the label lookup fails); a value outside −128…127 fails in the database (500, nothing written);
  - every update sets the three stage times again: the one for the new stage is now, the other two become NULL. `chefId` becomes the caller.

  Accepted risk.
- **KITCH-D3 (2026-11-11):** The chef dashboard stays as it is:
  - "Received" counts KOTs created today whose `chefId` is this user, i.e. the ones they last changed — not the KOTs the kitchen received;
  - "Accepted" counts only those now in Preparing (status 1); "Rejected" those now Rejected;
  - the app shows the chef cards only when `role` is exactly `Chef`. A chef saved as `chef` (panel or API) gets the waiter dashboard instead.

  Accepted.
- **Inherited:** AUTH-D2 (old tokens keep working), STAFF-D3 (any role text), ORD-D2 (orders created from the app have no table, so a KOT shows `N/A` for its place), AUTH-D5.

## Cross-cutting open questions

- None open.

## Feature-wide out of scope

- Ready KOTs and pick-up — WAIT.
- `POST /api/orders/update-status/{kotId}` (online / delivery order status with a WhatsApp message) — DLV.
- Printing a KOT in the kitchen — PRN.
