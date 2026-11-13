# 01 — Import CSV files — developer cases

Use case: [01-import-csv.md](../01-import-csv.md). Contract: [api-contract.md](../api-contract.md#post-apidataimport). Fixtures: [README.md](./README.md#fixtures).

`POST /api/data/import`, multipart, OWN-A's bearer token unless stated. "Done" = 200 `{ status: true, message: "Import completed successfully" }`.

| ID | Category | Request / manipulation | Expected | Database after |
|---|---|---|---|---|
| IMPEX-01-D01 | — (happy path) | `zone_file` = ZONE-OK; then `showDetails` | Done; `Main Hall` lists `Table 5`; one `Garden` zone (case-insensitive match) with `G1`, `G2` | 3 tables; 1 new zone |
| IMPEX-01-D02 | — (happy path) | `menu_file` = MENU-OK; then `getMenuItems` | Done; `Veg Roll` (Veg, 120.00, description `Crispy`) in Starters; `Butter Chicken` (Non-Veg, 350.00, description empty, `imageUrl` null) in new `Mains` | 2 items; 1 new category |
| IMPEX-01-D03 | — (happy path) | `floor_file` = FLOOR-OK; then `getFloorDetails`, `getRoomsDetails {}` | Done; `Ground Floor` with `roomCount` 2; rooms `101`, `102` listed | 1 floor (`createdBy` `owner`), 2 rooms with real binary `floorId` |
| IMPEX-01-D04 | — (all three) | all three OK files in one request | Done | everything from D01–D03 |
| IMPEX-01-D05 | Input validation (accepted IMPEX-D2/D3) | `zone_file` = ZONE-BAD | Done | `Lobby` zone with `L1` (capacity 0) and `L2` (capacity -3); `P9` in the inactive Patio (stays hidden from Home); the empty-name rows and the 256-character row are absent; skipped rows in the log |
| IMPEX-01-D06 | Input validation / Business-rule bypass (accepted IMPEX-D3) | `menu_file` = MENU-BAD | Done | `Free Item` Non-Veg price 0.00; `Neg Item` price -50.00 (on the ordering screen); `Huge` absent (out of range, logged); `No Cat` absent; the SQL-text and `=1+1` names stored literally |
| IMPEX-01-D07 | Business-rule bypass (accepted IMPEX-D2) | `zone_file` = ZONE-OK twice | Done both times | `Table 5`, `G1`, `G2` each twice; `Garden` once |
| IMPEX-01-D08 | Input validation | no fields; then only a text field `zone_file=abc` | 422 `At least one CSV file is required.`; then 422 `The zone file field must be a file.` (plus the at-least-one error) | none |
| IMPEX-01-D09 | Input validation | `zone_file` = NOT-CSV; `menu_file` = BIG | 422 `The zone file field must be a file of type: csv, txt.`; 422 `The menu file field must not be greater than 10240 kilobytes.` | none |
| IMPEX-01-D10 | Error hygiene (accepted IMPEX-D3) | `menu_file` = HEADER-ONLY; then a zone file of one line with no newline and no header (so its only row is skipped as the header) | Done both; nothing created. (A forced failure — e.g. revoke DB insert rights — answers 400 with the raw exception text; record it) | none |
| IMPEX-01-D11 | Role (accepted IMPEX-D3) | WTR-A and CHF-A each import ZONE-OK | Done both | rows created in REST-A |
| IMPEX-01-D12 | Restaurant isolation | OWN-B imports ZONE-OK | Done | rows only in REST-B; REST-A's `Main Hall` untouched |
| IMPEX-01-D13 | Authentication / Super-admin boundary | no token / `Bearer garbage` / panel cookie only → 401; `GET` → 405 | as listed | none |
| IMPEX-01-D14 | Account state (AUTH-D2) | OWN-P imports; NOREST imports | Done; 403 `Restaurant context missing` | rows in REST-P |
| IMPEX-01-D15 | Over-posting | add text fields `restaurantId=<REST-B>`, `createdBy=admin` with ZONE-OK | Done | rows in REST-A; `createdBy` not set from the field |
| IMPEX-01-D16 | Concurrency | two parallel imports of FLOOR-OK | both Done | `Ground Floor` once or twice (no lock) — record; rooms 4 |
| IMPEX-01-D17 | Rate limiting | 20 imports of HEADER-ONLY in a minute | all Done; no 429 | none |
| IMPEX-01-D18 | Injection | a zone file with `Main Hall' OR '1'='1,T9,2` and a 5 MB single field | Done (or 422 if over size) | zone named literally; no other zone touched |
