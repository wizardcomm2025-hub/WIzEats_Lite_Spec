# 01 — Manage zones in the app — developer cases

Use case: [01-manage-zones.md](../01-manage-zones.md). Contract: [api-contract.md](../api-contract.md#post-apitableszonesave). Fixtures: [README.md](./README.md#fixtures).

JSON with OWN-A's bearer token unless stated. "Saved" = 200 with `status: true`. "Failed" = 500 `{ status: false, message: "Failed to save zone", data: null }`.

| ID | Category | Request / manipulation | Expected | Database after |
|---|---|---|---|---|
| TBL-01-D01 | — (happy path) | `getTablesZoneDetails` `{}` | 200 `Tables Zone retrieved successfully`; `Main Hall` (`tableCount` 4, `isActive` true) and `Patio` (`isActive` false, `tableCount` 1); hex IDs | — |
| TBL-01-D02 | — (happy path) / Over-posting | `ZoneSave` `{ "zoneName": "Garden", "restaurantId": "<REST-B hex>", "createdBy": "x" }` | Saved, `Zone created successfully`; `data.zoneName` `Garden`, `isActive` true, no `tableCount` | zone in **REST-A**; `createdBy` NULL |
| TBL-01-D03 | — (happy path) / Injection | `ZoneSave` `{ "zoneId": "<MAIN-A hex>", "zoneName": "'); DROP TABLE zones; --" }` | Saved, `Zone updated successfully` | name stored exactly; tables intact |
| TBL-01-D04 | Business rule | `ZoneSave` `{ "zoneId": "<MAIN-A>", "isActive": false }`, then `showDetails` | Saved; `showDetails.zone` has no Main Hall | zone 0; **all** its tables `isActive` 0 |
| TBL-01-D05 | Business rule / Concurrency | (a) Main Hall active with `Table 2` alone inactive; send `isActive: true` for MAIN-A (b) two parallel `isActive` true / false | (a) Saved (b) both Saved | (a) zone and all four tables 1 (b) zone and its tables all share one value |
| TBL-01-D06 | Input validation | `ZoneSave` (a) `{}` (b) `{ "zoneName": "" }` (c) `{ "zoneName": 123 }` (d) `{ "zoneId": "<MAIN-A>", "isActive": "yes" }` | 422: (a)(b) `The zone name field is required.` (c) `The zone name field must be a string.` (d) `The is active field must be true or false.` | unchanged |
| TBL-01-D07 | Restaurant isolation / Error hygiene | `ZoneSave` with `zoneId` = (a) ZONE-B (b) 32 hex of no zone (c) `xyz` | Failed (all) — no `error` key, no SQL | ZONE-B unchanged |
| TBL-01-D08 | Input validation | `ZoneSave` `{ "zoneName": <256 × "a"> }` | Failed | no zone |
| TBL-01-D09 | Role (accepted TBL-D3) | WTR-A: `ZoneSave` `{ "zoneId": "<MAIN-A>", "isActive": false }`; CHF-A: create `Chef Zone` | Saved both | changes applied |
| TBL-01-D10 | Business rule (accepted TBL-D2) | `ZoneSave` `{ "zoneId": "<MAIN-A>", "isDelete": true }`; then `getTablesZoneDetails`, `getTablesDetails`, `showDetails` | Saved; zone gone from the zone list and from `showDetails`; its four tables still in `getTablesDetails` | zone `isDelete` 1; tables unchanged (active) |
| TBL-01-D11 | Authentication | Both endpoints: (a) no token (b) `Bearer garbage` (c) `GET` | (a)(b) 401 `{ "message": "Unauthenticated." }` (c) 405 | unchanged |
| TBL-01-D12 | Account state (AUTH-D2) | (a) OWN-P creates a zone (b) WTR-A-OFF's token from before the switch-off lists zones (c) NOREST | (a)(b) 200 (c) 403 `Restaurant context missing` | (a) zone created in REST-P |
| TBL-01-D13 | Restaurant isolation | OWN-A `getTablesZoneDetails` `{}` | No REST-B zone in `data` | — |
| TBL-01-D14 | Input validation / Injection (accepted TBL-D4) | `getTablesZoneDetails` `zoneId` = (a) MAIN-A hex (b) `' OR 1=1 --` (c) `[1]` | (a)(b) 200 `data: []` (c) 422 `Validation error` | — |
| TBL-01-D15 | Rate limiting | 60 `getTablesZoneDetails` + 30 `ZoneSave` in a minute | All processed; no 429 | — |
| TBL-01-D16 | Super-admin boundary | No bearer; ADMIN-1's panel cookie | 401 | unchanged |
