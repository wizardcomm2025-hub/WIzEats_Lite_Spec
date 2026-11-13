# 01 — See KOTs by stage — developer cases

Use case: [01-see-kots-by-stage.md](../01-see-kots-by-stage.md). Contract: [api-contract.md](../api-contract.md#post-apichefpending-kots). Fixtures: [README.md](./README.md#fixtures).

`POST /api/chef/pending-kots`, JSON, CHF-A's token unless stated. "Empty" = 200 `{ status: true, message: "No pending KOTs found", data: [] }`. "Encode-500" = 500 `{ status: false, message: "An error occurred while fetching pending kots", error: "Malformed UTF-8 characters, possibly incorrectly encoded" }`.

| ID | Category | Request / manipulation | Expected |
|---|---|---|---|
| KITCH-01-D01 | — (happy path) | REST-A has no KOTs; `{ "status": 0 }` | Empty |
| KITCH-01-D02 | Business rule (accepted KITCH-D1) | KOT-P exists; `{ "status": 0 }` | Encode-500. (If a 200 ever comes back, record the raw body: every ID happened to be valid UTF-8.) |
| KITCH-01-D03 | Business rule (accepted KITCH-D1) | KOT-R exists; `{ "status": 4 }`; then `{ "status": 2 }` | Encode-500; then Empty |
| KITCH-01-D04 | Restaurant isolation | Only KOT-B exists (REST-B); `{ "status": 0 }` | Empty |
| KITCH-01-D05 | Input validation | (a) `{}` (b) `{ "status": "abc" }` (c) `{ "status": 1.5 }` (d) `{ "status": "0" }` with no KOTs (e) `{ "status": 99 }` | (a) 422 `The status field is required.` (b)(c) 422 `The status field must be an integer.` (d) Empty (e) Empty |
| KITCH-01-D06 | Business-rule bypass | Set a REST-A KOT's status to 7 by SQL; `{ "status": 7 }` | 500 `{ status: false, message: "Something went wrong", data: null }` |
| KITCH-01-D07 | Role (accepted KITCH-D2) | D02 with WTR-A's token, then OWN-A's | Same as D02 (no 403) |
| KITCH-01-D08 | Authentication / Account state / Super-admin | no token → 401; panel cookie only → 401; `GET` → 405; NOREST → 403 `Restaurant context missing`; WTR-A-OFF's old token with no KOTs → Empty (AUTH-D2) | as listed |
| KITCH-01-D09 | Injection | `{ "status": "0 OR 1=1" }`; `{ "status": [0] }` | 422 `The status field must be an integer.` both |
| KITCH-01-D10 | Over-posting / Restaurant isolation | Only KOT-B exists; `{ "status": 0, "restaurantId": "<REST-B hex>" }` | Empty (the field is ignored) |
| KITCH-01-D11 | Rate limiting | 60 calls in a minute | all answered; no 429 |
| KITCH-01-D12 | Error hygiene (accepted KITCH-D1, AUTH-D5) | Inspect the D02 body | `error` carries the PHP encoder message; no stack trace, no SQL |
