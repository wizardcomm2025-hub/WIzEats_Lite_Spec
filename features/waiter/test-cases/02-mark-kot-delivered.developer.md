# 02 — Mark a KOT delivered — developer cases

Use case: [02-mark-kot-delivered.md](../02-mark-kot-delivered.md). Contract: [api-contract.md](../api-contract.md#post-apiwaiterupdatepickedupstatuskotid). Fixtures: [README.md](./README.md#fixtures).

`POST /api/waiter/updatePickedUpStatus/<id>`, body `{}`, WTR-A's token unless stated; `<id>` is the KOT's hex unless stated. "Picked" = 200 `{ status: true, message: "KOT picked up status updated successfully", data: { kotId, kotNumber, pickedUp_at } }`. Today's bodies: "OK-empty" = 200 with `data: []`; "Pick-500" = 500 `{ status: false, message: "An error occurred while update kot picked up status ", error: "Malformed UTF-8 characters, possibly incorrectly encoded" }` (trailing space). Cases tagged *Fails until … is fixed* assert the target; today's result is in brackets.

| ID | Category | Request / manipulation | Expected | Database after |
|---|---|---|---|---|
| WAIT-02-D01 | — (happy path, as the app) — *Fails until WAIT-D1 is fixed* | KOT-R | Picked; `kotId` = KOT-R's hex (today, from PICK-NONE: OK-empty) | KOT-R `isPickedUp` 1, `pickedUp_at` now, `status` still 4 (today: unchanged) |
| WAIT-02-D02 | Error hygiene / Restaurant isolation — *Fails until WAIT-D2 is fixed* | KOT-BR (REST-B) already has `isPickedUp` 1; pick up KOT-R | Picked — the reply holds KOT-R only (today: Pick-500, and if it ever encoded it would list REST-B's KOT) | KOT-R picked up |
| WAIT-02-D03 | Restaurant isolation — *Fails until WAIT-D2 is fixed* | KOT-BR's hex with WTR-A | 404 `{ status: false, message: "KOT not found", data: null }` | KOT-BR unchanged |
| WAIT-02-D04 | Restaurant isolation — *Fails until WAIT-D2 is fixed* | RAW(KOT-BR) with WTR-A | 404 `{ status: false, message: "KOT not found", data: null }` (today: Pick-500 and REST-B's KOT-BR marked picked up) | KOT-BR unchanged |
| WAIT-02-D05 | Business-rule bypass — *Fails until WAIT-D2 is fixed* | KOT-P (status 0) | 409 `{ status: false, message: "KOT is not ready for pickup", data: null }` (today, RAW: Pick-500, KOT-P marked with status still 0) | KOT-P unchanged |
| WAIT-02-D06 | Role — *Fails until WAIT-D2 is fixed* | KOT-R with CHF-A's token; with OWN-A's | 403 `{ status: false, message: "You do not have permission to perform this action", data: null }` both (today, RAW: marked each time) | KOT-R unchanged |
| WAIT-02-D07 | Input validation / Injection — *Fails until WAIT-D1 is fixed* | `<id>` = `xyz`; `' OR 1=1 --` (URL-encoded); `00000000000000000000000000000000` | 404 `{ status: false, message: "KOT not found", data: null }` each (today: OK-empty each) | no row changed |
| WAIT-02-D08 | Over-posting | KOT-R with body `{ "isPickedUp": 0, "status": 1, "restaurantId": "<REST-B hex>", "pickedUp_at": "2020-01-01 00:00:00" }` | Picked | only `isPickedUp` (1) and `pickedUp_at` (now) change |
| WAIT-02-D09 | Concurrency — *Fails until WAIT-D2 is fixed* | Two parallel calls for KOT-R | one Picked, the other 409 `KOT already picked up` (today: both answered; `pickedUp_at` from the later call) | `isPickedUp` 1, one `pickedUp_at` |
| WAIT-02-D10 | Authentication / Account state / Super-admin — switched-off part *Fails until AUTH-D2 is fixed* | no token → 401; panel cookie only → 401; `GET` → 405; NOREST → 403; WTR-A-OFF old token → 403 `Your account is inactive.` (today OK-empty) | as listed | — |
| WAIT-02-D11 | Rate limiting | 60 calls in a minute on an unknown hex ID | all answered (404 target); no 429 | — |
| WAIT-02-D12 | Error hygiene | Inspect the bodies of D03, D05, D07 | fixed messages only; no exception text, no `error` field, no other restaurant's data | — |
| WAIT-02-D13 | Input validation — *Fails until WAIT-D1 is fixed* | KOT-R's ID with dashes | Picked | KOT-R picked up |
| WAIT-02-D14 | Business rule — *Fails until WAIT-D2 is fixed* | KOT-RP (already picked up) | 409 `{ status: false, message: "KOT already picked up", data: null }` (today, RAW: `pickedUp_at` moved again) | `pickedUp_at` unchanged |
| WAIT-02-D15 | Business rule — *Fails until WAIT-D2 is fixed* | KOT-RP picked up today, KOT-R not; a REST-B KOT picked up today; `ready-kots` `{ "pickedUp": true }` | 200 `Picked up KOTs fetched successfully`, `data` = [KOT-RP] with `pickedUp_at` — no KOT-R, nothing from REST-B (today: the field is ignored → Encode-500 for KOT-R) | — |
