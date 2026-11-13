# 02 — Manage tables in the app — developer cases

Use case: [02-manage-tables.md](../02-manage-tables.md). Contract: [api-contract.md](../api-contract.md#post-apitablestablesave). Fixtures: [README.md](./README.md#fixtures).

JSON with OWN-A's bearer token unless stated. "Saved" = 200 with `status: true`. "Failed" = 500 `{ status: false, message: "Failed to save table", data: null }`.

| ID | Category | Request / manipulation | Expected | Database after |
|---|---|---|---|---|
| TBL-02-D01 | — (happy path) | `getTablesDetails` `{}` | 200 `Tables retrieved successfully`; `Table 1`–`4` and `P1` with hex `tableId` / `zoneId`, `status_key` / `status_color` matching each row's `status`, `Table 1` `orderId` = RUN-1's hex, others null | — |
| TBL-02-D02 | — (happy path) | `tableSave` `{ "tableName": "Table 5", "zoneId": "<MAIN-A>", "capacity": 4 }`; then `showDetails` | Saved `Table created successfully`; `data.isActive` true, `status` 0, `status_key` `available`; `Table 5` in `showDetails` under Main Hall | one row |
| TBL-02-D03 | Business rule | `tableSave` create `table 1` (and `Table 1 ` with a trailing space) in MAIN-A; then `Table 1` in PATIO-A | 422 `{ status: false, message: "Table name already exists.", data: null }` for both in MAIN-A; Saved in PATIO-A | one new row in Patio only |
| TBL-02-D04 | Input validation | create with (a) no `tableName` (b) no `zoneId` (c) no `capacity` (d) `capacity: "four"` (e) `status: 5` (f) `isActive: "yes"` | 422: (a) `The table name field is required.` (b) `The zone id field is required.` (c) `The capacity field is required.` (d) `The capacity field must be an integer.` (e) `The selected status is invalid.` (f) `The is active field must be true or false.` | no row |
| TBL-02-D05 | — (happy path) | `tableSave` `{ "tableId": "<Table 2>", "isActive": false }`; then `showDetails` | Saved `Table updated successfully`; Table 2 not in `showDetails` | `isActive` 0 |
| TBL-02-D06 | Business rule (accepted TBL-D2) | `tableSave` `{ "tableId": "<Table 2>", "tableName": "Table 1" }` | Saved | two rows named `Table 1` in Main Hall |
| TBL-02-D07 | Restaurant isolation | `tableSave` update with `tableId` = TABLE-B1 | Failed | TABLE-B1 unchanged |
| TBL-02-D08 | Restaurant isolation (accepted TBL-D2) | (a) create `X1` with `zoneId` = ZONE-B (b) update `Table 3` with `zoneId` = ZONE-B; then OWN-A and OWN-B `showDetails` | Saved both; neither restaurant's `showDetails` shows `X1` or `Table 3` | rows in REST-A pointing at ZONE-B |
| TBL-02-D09 | Role (accepted TBL-D3) | WTR-A: switch `Table 4` off; CHF-A: create `Chef T` | Saved both | applied |
| TBL-02-D10 | Input validation | (a) create with `zoneId` = `xyz` (b) create with `zoneId` = 32 hex of no zone (c) `tableName` 256 × `a` | (a) Failed (b) Saved (TBL-D2) (c) Failed | (b) orphan row; (a)(c) none |
| TBL-02-D11 | Business-rule bypass (accepted TBL-D2) | `capacity` = (a) `0` (b) `-3` (c) `2147483648` | (a)(b) Saved (c) Failed | (a)(b) stored |
| TBL-02-D12 | Business-rule bypass | `tableSave` `{ "tableId": "<Table 1>", "status": 0 }` while RUN-1 is running | Saved | Table 1 `status` 0 (manual override of the order's status) |
| TBL-02-D13 | Business rule (accepted TBL-D2) | `tableSave` `{ "tableId": "<Table 4>", "isDelete": true }`; then `getTablesDetails`, `showDetails` | Saved; Table 4 in neither list | `isDelete` 1 |
| TBL-02-D14 | Over-posting | create with `restaurantId: "<REST-B>"`, `version: 9`, `createdBy: "x"`, `tableId` empty | Saved | row in REST-A; `version` 0; `createdBy` NULL |
| TBL-02-D15 | Input validation / Injection / Error hygiene (accepted TBL-D4) | `getTablesDetails` (a) `{ "zoneId": "<MAIN-A>" }` (b) `{ "tableId": "<Table 1>" }` (c) `{ "tableId": "' OR 1=1 --" }` (d) `{ "zoneId": [1] }` | (a) 200 `data: []` (b)(c) 500 `{ status: false, message: "Failed to retrieve tables", data: null }` — no SQL (d) 422 `Validation error` | — |
| TBL-02-D16 | Concurrency | Two parallel creates `Table 7` in MAIN-A | Both may be Saved (no lock) | one or two `Table 7` rows — record which |
| TBL-02-D17 | Authentication / Account state / Role / Super-admin / Rate limit | Both endpoints: no token → 401; `GET` → 405; OWN-P → 200; NOREST → 403; WTR-A reads `getTablesDetails` → 200; panel cookie only → 401; 60 calls/min → no 429 | as listed | — |
| TBL-02-D18 | Injection | create `tableName` = `<script>alert(1)</script>` | Saved | stored exactly; panel shows it as text |
| TBL-02-D19 | Error hygiene | The 500 bodies of D07, D10 | No `error` key, no SQL | — |
| TBL-02-D20 | — (duplicate rows, accepted TBL-D4) | Add a second running order on `Table 1`; `getTablesDetails` `{}` | `Table 1` appears twice, once per `orderId`; `showDetails` lists it once with the higher `orderId` | — |
