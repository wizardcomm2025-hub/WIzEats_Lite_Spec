# 01 — Manage floors — developer cases

Use case: [01-manage-floors.md](../01-manage-floors.md). Contract: [api-contract.md](../api-contract.md#post-apiroomsfloorsave). Fixtures: [README.md](./README.md#fixtures).

JSON with OWN-A's bearer token unless stated. "Failed" = 500 `{ status: false, message: "Failed to save floor", data: null }`. Cases tagged *Fails until … is fixed* assert the target; today's result is in brackets.

| ID | Category | Request / manipulation | Expected | Database after |
|---|---|---|---|---|
| ROOM-01-D01 | — (happy path) | `getFloorDetails` `{}` | 200 `Room floors retrieved successfully`; `Ground Floor` with `roomCount` 3, `First Floor` with 0; hex IDs | — |
| ROOM-01-D02 | — (happy path) | `floorSave` `{ "floorName": "Upper Floor" }` | 200 `floor created successfully`; `data.isActive` true, `createdBy` `owner`, no `roomCount` | one row in REST-A |
| ROOM-01-D03 | — (happy path; ROOM-D1 fixed) | `floorSave` `{ "floorId": "<FLOOR-G hex>", "floorName": "Level 0" }`; repeat with the dashed and the upper-case form of the ID | 200 `floor updated successfully` each | name `Level 0` |
| ROOM-01-D04 | Business rule (ROOM-D1 fixed) | `floorSave` `{ "floorId": "<FLOOR-G hex>", "isActive": false }`; then `true` | 200 both | after `false`: floor and `101`, `102`, `103` all `isActive` 0; after `true`: all 1 (`103` too) |
| ROOM-01-D05 | Input validation | `floorSave` (a) `{}` (b) `{ "floorName": 123 }` (c) `{ "floorName": "X", "isActive": "yes" }` | 422 (a) `The floor name field is required.` (b) `The floor name field must be a string.` (c) `The is active field must be true or false.` | none |
| ROOM-01-D06 | Input validation / Error hygiene — *Fails until ROOM-D3 is fixed* | `floorSave` `{ "floorName": <256 × "a"> }` | 422 `The floor name field must not be greater than 255 characters.` (today: Failed 500 — no `error` key, no SQL) | none |
| ROOM-01-D07 | Over-posting | create with `restaurantId: "<REST-B>"`, `createdBy: "admin"`, `floorId: ""` | 200 created | in REST-A; `createdBy` `owner` |
| ROOM-01-D08 | Role — *Fails until ROOM-D2 is fixed* | (a) WTR-A creates a floor (b) CHF-A creates a floor (c) WTR-A lists floors (d) CHF-A lists floors | (a)(b)(d) 403 `{ status: false, message: "You do not have permission to perform this action", data: null }` (today: 200; rows with `createdBy` `waiter` / `chef`) (c) 200 | no rows |
| ROOM-01-D09 | Restaurant isolation / Injection — (b)(c) *Fails until ROOM-D4 is fixed* | `getFloorDetails` (a) `{}` (b) `{ "floorId": "<FLOOR-G hex>" }` (c) `{ "floorId": "' OR 1=1 --" }` | (a) no REST-B floor (b) 200, [Ground Floor] (c) 422 `Validation error` (today: (b)(c) 200 `data: []`) | — |
| ROOM-01-D10 | Authentication / Account state — OWN-P part *Fails until AUTH-D2 is fixed* | both endpoints: no token / `Bearer garbage` → 401; `GET` → 405; OWN-P → 403 `Your restaurant is not approved.` (today 200); NOREST → 403 `Restaurant context missing` | as listed | — |
| ROOM-01-D11 | Injection | create `floorName` = `'); DROP TABLE floors; --` | 200 | stored exactly; tables intact |
| ROOM-01-D12 | Super-admin boundary / Rate limiting | panel cookie only → 401; 60 calls in a minute → no 429 | as listed | — |
| ROOM-01-D13 | Business-rule bypass (accepted — ROOM-D2) | Room Service **off**; create `Hidden Floor` | 200 | floor created though the screens are hidden |
| ROOM-01-D14 | Business rule — *Fails until ROOM-D3 is fixed* | (a) create `ground floor` (b) rename FLOOR-1 to ` GROUND FLOOR ` (c) delete FLOOR-G, then create `Ground Floor` | (a)(b) 422 `{ status: false, message: "Floor name already exists.", data: null }` (today: 200) (c) 200 — deleted floors don't count | (a)(b) unchanged (c) one new floor |
| ROOM-01-D15 | Restaurant isolation / Error hygiene — *Fails until ROOM-D3 is fixed* | `floorSave` update with `floorId` = (a) FLOOR-B (b) 32 hex of no floor (c) `xyz` | 404 `{ status: false, message: "Floor not found", data: null }` each (today: Failed 500) | FLOOR-B unchanged |
| ROOM-01-D16 | Business rule — *Fails until ROOM-D3 is fixed* | `floorSave` `{ "floorId": "<FLOOR-G>", "isDelete": true }`; then `getFloorDetails`, `getRoomsDetails {}` | 200; Ground Floor gone from the floor list; `101`, `102` gone from the room list (today: still listed) | floor `isDelete` 1; its rooms `isDelete` 1 (today: unchanged) |
