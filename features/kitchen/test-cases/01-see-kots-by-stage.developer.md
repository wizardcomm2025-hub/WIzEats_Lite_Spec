# 01 — See KOTs by stage — developer cases

Use case: [01-see-kots-by-stage.md](../01-see-kots-by-stage.md). Contract: [api-contract.md](../api-contract.md#post-apichefpending-kots). Fixtures: [README.md](./README.md#fixtures).

`POST /api/chef/pending-kots`, JSON, CHF-A's token unless stated. "Empty" = 200 `{ status: true, message: "No pending KOTs found", data: [] }`. "Listed" = 200 `{ status: true, message: "Pending KOTs fetched successfully", data: [ … ] }` with hex IDs. "Encode-500" (today) = 500 `{ status: false, message: "An error occurred while fetching pending kots", error: "Malformed UTF-8 characters, possibly incorrectly encoded" }`. Cases tagged *Fails until … is fixed* assert the target; today's result is in brackets.

| ID | Category | Request / manipulation | Expected |
|---|---|---|---|
| KITCH-01-D01 | — (happy path) | REST-A has no KOTs; `{ "status": 0 }` | Empty |
| KITCH-01-D02 | — (happy path) — *Fails until KITCH-D1 is fixed* | KOT-P exists; `{ "status": 0 }` | Listed: one KOT — `kotId` = KOT-P's 32-char hex, `kotNumber`, `orderNumber`, `kotNote` `less spicy`, `status` `Pending`, `tableName` `Table 1`, `zoneName` `Main Hall`, `items` [`Paneer Tikka` × 2], `created_at`; no raw bytes anywhere (today: Encode-500) |
| KITCH-01-D03 | Business rule — *Fails until KITCH-D1 is fixed* | KOT-R and KOT-X exist; `{ "status": 4 }`; then `{ "status": 2 }` | Listed [KOT-R]; then Listed [KOT-X] (today: Encode-500 both) |
| KITCH-01-D04 | Restaurant isolation | Only KOT-B exists (REST-B); `{ "status": 0 }` | Empty |
| KITCH-01-D05 | Input validation — (e) *Fails until KITCH-D2 is fixed* | (a) `{}` (b) `{ "status": "abc" }` (c) `{ "status": 1.5 }` (d) `{ "status": "0" }` with no KOTs (e) `{ "status": 99 }` | (a) 422 `The status field is required.` (b)(c) 422 `The status field must be an integer.` (d) Empty (e) 422 `The selected status is invalid.` (today: Empty) |
| KITCH-01-D06 | Business-rule bypass — *Fails until KITCH-D2 is fixed* | Set a REST-A KOT's status to 7 by SQL; `{ "status": 7 }` | 422 `The selected status is invalid.` (target; today 500 `{ status: false, message: "Something went wrong", data: null }`) |
| KITCH-01-D07 | Role — *Fails until KITCH-D2 is fixed* | D02 with WTR-A's token, then OWN-A's | 403 `{ status: false, message: "You do not have permission to perform this action", data: null }` both (today: same as D02) |
| KITCH-01-D08 | Authentication / Account state / Super-admin — WTR-A-OFF / OWN-P parts *Fails until AUTH-D2 is fixed* | no token → 401; panel cookie only → 401; `GET` → 405; NOREST → 403 `Restaurant context missing`; a chef token taken before that chef was switched off → 403 `Your account is inactive.` (today Empty) | as listed |
| KITCH-01-D09 | Injection | `{ "status": "0 OR 1=1" }`; `{ "status": [0] }` | 422 `The status field must be an integer.` both |
| KITCH-01-D10 | Over-posting / Restaurant isolation | Only KOT-B exists; `{ "status": 0, "restaurantId": "<REST-B hex>" }` | Empty (the field is ignored) |
| KITCH-01-D11 | Rate limiting | 60 calls in a minute | all answered; no 429 |
| KITCH-01-D12 | Error hygiene — *Fails until KITCH-D1 is fixed* | force a failure (e.g. drop DB read rights); `{ "status": 0 }` | 500 `{ status: false, message: "Something went wrong", data: null }` — no `error` key, no encoder message (today: D02's body carries the PHP encoder message in `error`) |
| KITCH-01-D13 | Business rule — *Fails until KITCH-D1 is fixed* | KOT-P plus a Takeaway KOT and a Room Service KOT on room `101`; `{ "status": 0 }` | Listed, newest first: the room KOT has `roomName` `101` and `floorName`; the Takeaway KOT has all place names null; KOT-P has its table (today: Encode-500) |
