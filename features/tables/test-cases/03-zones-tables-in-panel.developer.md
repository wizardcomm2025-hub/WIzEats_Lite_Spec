# 03 — Zones and tables in the panel — developer cases

Use case: [03-zones-tables-in-panel.md](../03-zones-tables-in-panel.md). Contract: [api-contract.md → Panel routes](../api-contract.md#panel-routes-web). Fixtures and how to call the panel: [README.md](./README.md#panel-requests-use-case-03).

JSON with ADMIN-1's session and `X-CSRF-TOKEN` unless stated.

| ID | Category | Request / manipulation | Expected | Database after |
|---|---|---|---|---|
| TBL-03-D01 | — (happy path) | `POST /admin/zones/store` `{ restaurantId: <REST-A>, name: "Terrace", isActive: 1 }` | 200 `{ "success": true }` | zone in REST-A |
| TBL-03-D02 | — (happy path) | `POST /admin/zones/update` `{ id: <MAIN-A>, name: "Hall A", isActive: 1 }` | 200 | renamed |
| TBL-03-D03 | Business rule | `POST /admin/zones/toggle` `{ id: <MAIN-A>, isActive: 0 }` | 200 | zone 0; tables **unchanged** (1) |
| TBL-03-D04 | — (happy path) | `POST /admin/tables/store` `{ restaurantId: <REST-A>, zoneId: <MAIN-A>, tableName: "Table 9", capacity: 2, isActive: 1 }` | 200 | row; `status` 0 (default) |
| TBL-03-D05 | — (happy path) | `POST /admin/tables/update` `{ id: <Table 2>, tableName: "T2", capacity: 6, isActive: 0, zoneId: <PATIO-A> }` | 200 | updated incl. zone |
| TBL-03-D06 | Input validation / Error hygiene | (a) tables/store with `tableName: ""` (b) with `capacity: "x"` (c) zones/store with `name: ""` | 500 error page each; no SQL with `APP_DEBUG=false` | nothing inserted |
| TBL-03-D07 | Input validation | tables/update with `capacity: -3`; zones/update with an unknown 32-hex `id` | 200 both | capacity -3; nothing else changed |
| TBL-03-D08 | Business-rule bypass (accepted TBL-D2) | tables/update `{ id: <Table 3>, status: 5 }`; then OWN-A `showDetails` and `getTablesDetails` | 200; then both API calls → 500 (`Failed to fetch restaurant details` / `Failed to retrieve tables`) until status is reset | Table 3 status 5 |
| TBL-03-D09 | Over-posting | tables/update `{ id: <Table 4>, isDelete: 1, version: 7, restaurantId: <REST-B> }`; then `{ id: <Table 4>, nonColumn: 1 }` | first 200 (`restaurantId` ignored); second 500 | Table 4 `isDelete` 1, `version` 7, still REST-A |
| TBL-03-D10 | Authentication / Super-admin boundary | any POST: (a) no session (b) no CSRF (c) POS bearer only | (a)(c) 302 to `/` / JSON 401 (b) 419 | unchanged |
| TBL-03-D11 | Restaurant isolation (by design) | tables/update `{ id: <TABLE-B1>, tableName: "B1x" }` from REST-A's page | 200 — the super admin may edit any restaurant | TABLE-B1 renamed |
| TBL-03-D12 | Injection / Rate limiting | zones/store `name: "<img src=x onerror=alert(1)>"`; 30 updates/min | 200; the page shows the text escaped (cell and `data-name`); no 429 | stored exactly |
