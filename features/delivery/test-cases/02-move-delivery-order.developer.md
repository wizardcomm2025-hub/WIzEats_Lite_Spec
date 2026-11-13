# 02 — Move a delivery order on — developer cases

Use case: [02-move-delivery-order.md](../02-move-delivery-order.md). Contract: [api-contract.md](../api-contract.md#post-apiordersupdate-statuskotid). Fixtures: [README.md](./README.md#fixtures) (WA-CATCH and SNAP for every case).

`POST /api/orders/update-status/<id>`, JSON, WTR-A's token unless stated. "Not-found" = 400 `{ status: false, message: "KOT not found" }`. "Cast-0 error" = 400 `{ status: false, message: "SQLSTATE[22007]: … 1292 Truncated incorrect DOUBLE value … update `kots` …" }`, or 500 `{ message: "Server Error" }` if that text isn't valid UTF-8 — record which. "Unchanged" = every `kots` row matches SNAP, and WA-CATCH received nothing.

| ID | Category | Request / manipulation | Expected | After |
|---|---|---|---|---|
| DLV-02-D01 | — (happy path; accepted DLV-D2) | `<id>` = KOT-D hex; `{ "status": 1 }` | Not-found | Unchanged |
| DLV-02-D02 | Restaurant isolation / Error hygiene (accepted DLV-D2) | `<id>` = KOT-L hex; `{ "status": 1 }` | Cast-0 error | Unchanged (no KOT of any restaurant changed) |
| DLV-02-D03 | Business-rule bypass (accepted DLV-D2) | `<id>` = `0`; `NaN`; `00000000000000000000000000000000` with `{ "status": 4 }` | Cast-0 error each | Unchanged |
| DLV-02-D04 | Input validation | KOT-D hex with (a) `{}` (b) `{ "status": "x" }` (c) `{ "status": 99 }` | (a)(b) 422 `{ status: false, message: "Validation error", errors: { status: […] } }` (c) Not-found | Unchanged |
| DLV-02-D05 | Role (accepted DLV-D2) | D01 and D02 with CHF-A's token, then OWN-A's | Same replies as D01 / D02 (no 403) | Unchanged |
| DLV-02-D06 | Over-posting | KOT-D hex `{ "status": 1, "kotId": "<KOT-L hex>", "customerPhone": "9999999999" }` | Not-found | Unchanged |
| DLV-02-D07 | Authentication / Account state / Super-admin | no token → 401; panel cookie only → 401; `GET` → 405; NOREST → 403; WTR-A-OFF old token, D01 → Not-found (AUTH-D2) | as listed | Unchanged |
| DLV-02-D08 | Rate limiting | 60 × D01 in a minute | all Not-found; no 429 | Unchanged |
| DLV-02-D09 | Injection | `<id>` = `1 OR 1=1` (URL-encoded); `1; DROP TABLE kots` | Not-found (cast to 1) | Unchanged; `kots` intact |
| DLV-02-D10 | Error hygiene (accepted DLV-D2, AUTH-D5) | Inspect the D02 body; repeat D02 with `APP_DEBUG=true` | D02 body exposes the SQL statement and table name; with debug on, a 500 also carries `exception` and `trace` | Unchanged |
