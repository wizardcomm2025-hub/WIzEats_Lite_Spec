# 01 — See ready orders — developer cases

Use case: [01-see-ready-orders.md](../01-see-ready-orders.md). Contract: [api-contract.md](../api-contract.md#post-apiwaiterready-kots). Fixtures: [README.md](./README.md#fixtures).

`POST /api/waiter/ready-kots`, no body, WTR-A's token unless stated. "Empty" = 200 `{ status: true, message: "No ready KOTs found", data: [] }`. "Listed" = 200 `{ status: true, message: "Pending KOTs fetched successfully", data: [ … ] }` with hex IDs. "Encode-500" (today) = 500 `{ status: false, message: "An error occurred while fetching pending kots", error: "Malformed UTF-8 characters, possibly incorrectly encoded" }`. Cases tagged *Fails until … is fixed* assert the target; today's result is in brackets.

| ID | Category | Request / manipulation | Expected |
|---|---|---|---|
| WAIT-01-D01 | — (happy path) | REST-A has no Ready KOT | Empty |
| WAIT-01-D02 | — (happy path) — *Fails until WAIT-D1 is fixed* | KOT-R exists (status 4, not picked up) | Listed [KOT-R]: `kotId` 32-char hex, `kotNumber`, `orderNumber`, `status` 4, `isPickedUp` 0, items; no raw bytes (today: Encode-500) |
| WAIT-01-D03 | Business rule | Only KOT-RP (picked up); then only KOT-P (status 0) | Empty both |
| WAIT-01-D04 | Restaurant isolation | Only KOT-BR (REST-B) is Ready | Empty |
| WAIT-01-D05 | Role — *Fails until WAIT-D2 is fixed* | D02 with CHF-A's token, then OWN-A's | 403 `{ status: false, message: "You do not have permission to perform this action", data: null }` both (today: same as D02) |
| WAIT-01-D06 | Authentication / Account state / Super-admin — switched-off part *Fails until AUTH-D2 is fixed* | no token → 401; panel cookie only → 401; `GET` → 405; NOREST → 403 `Restaurant context missing`; WTR-A-OFF's old token → 403 `Your account is inactive.` (today Empty) | as listed |
| WAIT-01-D07 | Over-posting / Injection / Restaurant isolation | Only KOT-BR Ready; body `{ "restaurantId": "<REST-B hex>", "status": "4 OR 1=1" }` | Empty (body ignored) |
| WAIT-01-D08 | Rate limiting | 60 calls in a minute | all answered; no 429 |
| WAIT-01-D09 | Error hygiene — *Fails until WAIT-D1 is fixed* | force a failure (e.g. drop DB read rights) | 500 `{ status: false, message: "Something went wrong", data: null }` — no `error` key (today: the encoder message in `error`) |
| WAIT-01-D10 | Business rule — *Fails until WAIT-D1 is fixed* | KOT-R on Table 1 and a Takeaway KOT set Ready by SQL | Listed, newest first; KOT-R has `tableName` `Table 1`, `zoneName` `Main Hall`; the Takeaway KOT has no place names (today: Encode-500) |
