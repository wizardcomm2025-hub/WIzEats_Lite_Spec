# 02 — Move a KOT through its stages — developer cases

Use case: [02-move-kot-through-stages.md](../02-move-kot-through-stages.md). Contract: [api-contract.md](../api-contract.md#post-apichefupdate-statuskotid). Fixtures: [README.md](./README.md#fixtures).

`POST /api/chef/update-status/<id>`, JSON, CHF-A's token unless stated. "Updated(L)" = 200 `{ status: true, message: "KOT status updated successfully", data: { kotStatus: "L" } }`. "Fail-500" = 500 `{ status: false, message: "Something went wrong", data: null }`. "Times" lists `preparing_at / rejected_at / completed_at`.

| ID | Category | Request / manipulation | Expected | Database after |
|---|---|---|---|---|
| KITCH-02-D01 | — (happy path, as the app; accepted KITCH-D1) | `<id>` = KOT-P hex; `{ "status": 1 }` | Updated(Preparing) | KOT-P unchanged: status 0, `chefId` NULL, `updated_at` unchanged |
| KITCH-02-D02 | — (raw ID) | RAW(KOT-P); `{ "status": 1 }` | Updated(Preparing) | status 1, `chefId` CHF-A, times: now / NULL / NULL |
| KITCH-02-D03 | Business-rule bypass (accepted KITCH-D2) | RAW(KOT-P), in turn: 3, 4, 0, 2, 1 | Updated(Completed), (Ready), (Pending), (Rejected), (Preparing) | after 3: NULL / NULL / now; after 4 and 0: all NULL; after 2: NULL / now / NULL; after 1: now / NULL / NULL |
| KITCH-02-D04 | Restaurant isolation (accepted KITCH-D2) | RAW(KOT-B) with CHF-A; `{ "status": 2 }` | Updated(Rejected) | KOT-B status 2, `chefId` CHF-A (a REST-A user) |
| KITCH-02-D05 | Role (accepted KITCH-D2) | RAW(KOT-P) `{ "status": 4 }` with WTR-A's token; then OWN-A's | Updated(Ready) both | `chefId` WTR-A, then OWN-A |
| KITCH-02-D06 | Input validation (accepted KITCH-D2) | (a) `{}` (b) `{ "status": "x" }` (c) RAW(KOT-P) `9` (d) RAW(KOT-P) `-1` (e) RAW(KOT-P) `300` (f) KOT-P hex `9` | (a) 422 `The status field is required.` (b) 422 `The status field must be an integer.` (c)–(f) Fail-500 | (c) status 9, all times NULL (d) status −1 (e) unchanged (f) unchanged |
| KITCH-02-D07 | Injection / Input validation (accepted KITCH-D1) | `<id>` = `xyz`; `' OR 1=1 --` (URL-encoded); `00000000000000000000000000000000`; `{ "status": 1 }` each | Updated(Preparing) each | no `kots` row changed |
| KITCH-02-D08 | Over-posting | RAW(KOT-P) `{ "status": 3, "chefId": "<OWN-B hex>", "kotNote": "x", "isPickedUp": 1, "kotNumber": "999" }` | Updated(Completed) | only status, `chefId` (= CHF-A), times, `updated_at` change |
| KITCH-02-D09 | Concurrency | Two parallel RAW(KOT-P) calls: `1` and `2` | both Updated | status and times match whichever ran last; never both `preparing_at` and `rejected_at` set |
| KITCH-02-D10 | Authentication / Account state / Super-admin | no token → 401; panel cookie only → 401; `GET` → 405; NOREST → 403; WTR-A-OFF old token, RAW(KOT-P) `1` → Updated (AUTH-D2) | as listed | — |
| KITCH-02-D11 | Rate limiting | 60 calls in a minute (KOT-P hex) | all Updated; no 429 | — |
| KITCH-02-D12 | Error hygiene | Inspect the D06 (c) body | `Something went wrong` only; no exception text, no `error` field | — |
| KITCH-02-D13 | Business-rule bypass (accepted KITCH-D2) | KOT-R with `isPickedUp` 1 (set by SQL); RAW(KOT-R) `{ "status": 0 }` | Updated(Pending) | status 0, `isPickedUp` still 1 |
