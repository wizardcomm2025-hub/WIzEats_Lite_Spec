# Kitchen (chef)

A chef signs in and lands on **Kitchen Orders**: the KOTs sent from ordering (ORD), in four tabs by stage (New, Preparing, Completed, Ready). The chef accepts or rejects a new KOT, then moves it on to Completed and Ready. When a KOT is Ready, the waiter picks it up (WAIT). The chef also has a small dashboard with today's Received / Accepted / Rejected counts.

The spec records today's behaviour plus targets (re-synced 2026-11-24 against API commit d54d6c8 and `POS_APP-kot`; no KITCH code changed). **Today the chef's KOT list still fails with a 500 whenever at least one KOT exists, and a status change never reaches the database (KITCH-D1), so in the app the kitchen never sees a KOT.** KITCH-D1 to KITCH-D3 are *To fix*; most cases assert the target.

**Feature code:** `KITCH` (test IDs `KITCH-01-T01`, `KITCH-02-D04`, …)

## Use cases

| # | Use case | Actor | Tester file | Developer file |
|---|---|---|---|---|
| 01 | [See KOTs by stage](./01-see-kots-by-stage.md) | Chef (today any role through the API — KITCH-D2) | [tester](./test-cases/01-see-kots-by-stage.tester.md) | [developer](./test-cases/01-see-kots-by-stage.developer.md) |
| 02 | [Move a KOT through its stages](./02-move-kot-through-stages.md) | same | [tester](./test-cases/02-move-kot-through-stages.tester.md) | [developer](./test-cases/02-move-kot-through-stages.developer.md) |
| 03 | [Chef dashboard](./03-chef-dashboard.md) | same | [tester](./test-cases/03-chef-dashboard.tester.md) | [developer](./test-cases/03-chef-dashboard.developer.md) |

## Key concepts

- **KOT status** — `kots.status`: 0 Pending, 1 Preparing, 2 Rejected, 3 Completed, 4 Ready. The server sends the label text (`Pending` …) in lists; the app sends the number on updates.
- **Stage path in the app** — Pending → (Accept) Preparing → (Mark Completed) Completed → (Mark Ready) Ready. Reject only on a Pending KOT. Today the server enforces none of this; target it does (KITCH-D2).
- **Chef on a KOT** — `kots.chefId`, today set to whoever last changed the status (target: whoever accepted or rejected it). Stage times: `preparing_at`, `rejected_at`, `completed_at`.
- **Picked up** — `kots.isPickedUp`, set by the waiter (WAIT), not by the chef.

## Documents

- [data-model.md](./data-model.md) · [api-contract.md](./api-contract.md) · [ui-ux.md](./ui-ux.md) · [test-cases/README.md](./test-cases/README.md) · [developer-guide.html](./developer-guide.html)
- Implementation: `laravel-pos-api/api/specs/kitchen/` (created with the first plan)

## Depends on

- **ORD** — creates the KOTs and their `order_items` (the list reaches the restaurant only through `order_items.restaurantId`).
- **STAFF** — chef users; the app saves the role as `Chef` (capital C).
- **AUTH** — PIN sign-in; the app routes a chef to Kitchen Orders.
- **WAIT** — reads Ready KOTs and marks them picked up.

## Decisions

Statuses per [conventions.md → Decision status](../../conventions.md#decision-status), set 2026-11-24. Cross-cutting items: [project-overview.md → Known gaps and targets](../../project-overview.md#known-gaps-and-targets).

| ID | Status | Short | Tests |
|---|---|---|---|
| KITCH-D1 | **To fix** (with X-D4, X-D5) | KOT list 500s; status updates never match; card shows the KOT ID | 01-D02, 01-D03, 01-D12, 01-D13, 02-D01, 02-D07, 02-D14; 01-T03–T05, 01-T07, 02-T01–T03, 03-T03–T05 |
| KITCH-D2 | **To fix** (with X-D1) | No role, restaurant, value or stage rules on status changes | 01-D05–D07, 02-D02–D06, 02-D09, 02-D13, 03-D06 |
| KITCH-D3 | **To fix** | Dashboard counts only KOTs the chef last changed; case-sensitive chef check | 03-D01–D03, 03-D05; 03-T02–T06 |
| AUTH-D2 (inherited) | **To fix** (X-D2) | Switched-off / unapproved callers keep access | 01-D08, 02-D10, 03-D07 |

- **KITCH-D1 — To fix (2026-11-24, was accepted 2026-11-11).** *Target:* `pending-kots` returns every ID as 32-char hex (no raw bytes) and adds `kotNumber` and `orderNumber` to each KOT; `update-status/{kotId}` finds the KOT by hex ID (dashes allowed) among the KOTs of the caller's restaurant (through `order_items`) → an unknown, other restaurant's or malformed ID → 404 `{ "status": false, "message": "KOT not found", "data": null }`; 500s carry no `error` key (X-D5). The app's card title shows `KOT #<kotNumber>`.
- **KITCH-D2 — To fix (2026-11-24, was accepted 2026-11-11).** *Target:* all three `/api/chef/*` endpoints are chef only (X-D1: 403 `You do not have permission to perform this action`). `status` must be 0–4 → else 422 `The selected status is invalid.` (on both `pending-kots` and `update-status`). Only these changes are allowed: Pending → Preparing, Pending → Rejected, Preparing → Completed, Completed → Ready; any other → 409 `{ "status": false, "message": "Invalid status change", "data": null }`, nothing written. The new stage's time is set and earlier ones are kept; `chefId` is set on accept or reject only.
- **KITCH-D3 — To fix (2026-11-24, was accepted 2026-11-11).** *Target:* the dashboard counts the restaurant's KOTs created today (server date): `total_received` all of them, `total_accepted` those in Preparing, Completed or Ready, `total_rejected` those Rejected. The app shows the chef cards for role `chef` in any letter case.
- **Inherited:** AUTH-D2 (To fix, X-D2); STAFF-D3 (role text — To fix); ORD-D2 fixed (app orders now carry their table or room, so the card shows the place); AUTH-D5 / X-D5.

Original decision texts (history):

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
- **Inherited (2026-11-11):** AUTH-D2 (old tokens keep working), STAFF-D3 (any role text), ORD-D2 (orders created from the app have no table, so a KOT shows `N/A` for its place), AUTH-D5.

## Cross-cutting open questions

- None open.

## Feature-wide out of scope

- Ready KOTs and pick-up — WAIT.
- `POST /api/orders/update-status/{kotId}` (online / delivery order status with a WhatsApp message) — DLV.
- Printing a KOT in the kitchen — PRN.
