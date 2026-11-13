# 01 — See ready orders — developer cases

Use case: [01-see-ready-orders.md](../01-see-ready-orders.md). Contract: [api-contract.md](../api-contract.md#post-apiwaiterready-kots). Fixtures: [README.md](./README.md#fixtures).

`POST /api/waiter/ready-kots`, no body, WTR-A's token unless stated. "Empty" = 200 `{ status: true, message: "No ready KOTs found", data: [] }`. "Encode-500" = 500 `{ status: false, message: "An error occurred while fetching pending kots", error: "Malformed UTF-8 characters, possibly incorrectly encoded" }`.

| ID | Category | Request / manipulation | Expected |
|---|---|---|---|
| WAIT-01-D01 | — (happy path) | REST-A has no Ready KOT | Empty |
| WAIT-01-D02 | Business rule (accepted WAIT-D1) | KOT-R exists (status 4, not picked up) | Encode-500. (A 200 would mean every ID happened to be valid UTF-8 — record the body.) |
| WAIT-01-D03 | Business rule | Only KOT-R with `isPickedUp` 1; then only KOT-P (status 0) | Empty both |
| WAIT-01-D04 | Restaurant isolation | Only KOT-BR (REST-B) is Ready | Empty |
| WAIT-01-D05 | Role (accepted WAIT-D2) | D02 with CHF-A's token, then OWN-A's | Same as D02 (no 403) |
| WAIT-01-D06 | Authentication / Account state / Super-admin | no token → 401; panel cookie only → 401; `GET` → 405; NOREST → 403 `Restaurant context missing`; WTR-A-OFF old token with no Ready KOT → Empty (AUTH-D2) | as listed |
| WAIT-01-D07 | Over-posting / Injection / Restaurant isolation | Only KOT-BR Ready; body `{ "restaurantId": "<REST-B hex>", "status": "4 OR 1=1" }` | Empty (body ignored) |
| WAIT-01-D08 | Rate limiting | 60 calls in a minute | all answered; no 429 |
| WAIT-01-D09 | Error hygiene (accepted WAIT-D1, AUTH-D5) | Inspect the D02 body | encoder message in `error`; no stack trace, no SQL |
