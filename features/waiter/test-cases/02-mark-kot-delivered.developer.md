# 02 — Mark a KOT delivered — developer cases

Use case: [02-mark-kot-delivered.md](../02-mark-kot-delivered.md). Contract: [api-contract.md](../api-contract.md#post-apiwaiterupdatepickedupstatuskotid). Fixtures: [README.md](./README.md#fixtures).

`POST /api/waiter/updatePickedUpStatus/<id>`, body `{}`, WTR-A's token unless stated. Start each case from PICK-NONE unless it says otherwise. "OK-empty" = 200 `{ status: true, message: "KOT picked up status updated successfully", data: [] }`. "Pick-500" = 500 `{ status: false, message: "An error occurred while update kot picked up status ", error: "Malformed UTF-8 characters, possibly incorrectly encoded" }` (note the trailing space).

| ID | Category | Request / manipulation | Expected | Database after |
|---|---|---|---|---|
| WAIT-02-D01 | — (happy path, as the app; accepted WAIT-D1) | `<id>` = KOT-R hex | OK-empty | KOT-R unchanged (`isPickedUp` 0) |
| WAIT-02-D02 | — (raw ID; accepted WAIT-D2) | RAW(KOT-R) | Pick-500 | KOT-R `isPickedUp` 1, `pickedUp_at` set, `status` still 4 |
| WAIT-02-D03 | Restaurant isolation (accepted WAIT-D2) | Not from PICK-NONE: KOT-B already has `isPickedUp` 1 (SQL). WTR-A sends KOT-R hex. | Pick-500 — the reply includes REST-B's KOT | nothing changed |
| WAIT-02-D04 | Restaurant isolation (accepted WAIT-D2) | RAW(KOT-BR) with WTR-A | Pick-500 | REST-B's KOT-BR marked picked up |
| WAIT-02-D05 | Business-rule bypass (accepted WAIT-D2) | RAW(KOT-P) (status 0); again 1 minute later | Pick-500 both | KOT-P `isPickedUp` 1, status still 0; `pickedUp_at` = the second call's time |
| WAIT-02-D06 | Role (accepted WAIT-D2) | RAW(KOT-R) with CHF-A's token; from PICK-NONE again with OWN-A's | Pick-500 both | KOT-R marked each time |
| WAIT-02-D07 | Input validation / Injection | `<id>` = `xyz`; `' OR 1=1 --` (URL-encoded); `00000000000000000000000000000000` | OK-empty each | no row changed |
| WAIT-02-D08 | Over-posting | RAW(KOT-R) with body `{ "isPickedUp": 0, "status": 1, "restaurantId": "<REST-B hex>", "pickedUp_at": "2020-01-01 00:00:00" }` | Pick-500 | only `isPickedUp` (1) and `pickedUp_at` (now) change |
| WAIT-02-D09 | Concurrency | Two parallel RAW(KOT-R) calls | Pick-500 both | `isPickedUp` 1; `pickedUp_at` from the later call |
| WAIT-02-D10 | Authentication / Account state / Super-admin | no token → 401; panel cookie only → 401; `GET` → 405; NOREST → 403; WTR-A-OFF old token, KOT-R hex → OK-empty (AUTH-D2) | as listed | — |
| WAIT-02-D11 | Rate limiting | 60 calls in a minute (KOT-R hex, PICK-NONE) | all OK-empty; no 429 | — |
| WAIT-02-D12 | Error hygiene (accepted WAIT-D2, AUTH-D5) | Inspect the D02 body | encoder message in `error`; no stack trace, no SQL; no other restaurant's data in the body | — |
