# 02 — Manage rooms — developer cases

Use case: [02-manage-rooms.md](../02-manage-rooms.md). Contract: [api-contract.md](../api-contract.md#post-apiroomsroomsave). Fixtures: [README.md](./README.md#fixtures).

JSON with OWN-A's bearer token unless stated. "Failed" = 500 `{ status: false, message: "Failed to save room", data: null }`.

| ID | Category | Request / manipulation | Expected | Database after |
|---|---|---|---|---|
| ROOM-02-D01 | — (happy path) | `getRoomsDetails` `{}` | 200 `Rooms retrieved successfully`; rooms `101`, `102` only (`103` inactive is absent); `101` has RUN-R's `orderId`; `status_key` / `status_color` per status | — |
| ROOM-02-D02 | Input validation (accepted ROOM-D1) | `getRoomsDetails` `{ "floorId": "<FLOOR-G hex>" }` (what the app sends) | 200 `data: []` | — |
| ROOM-02-D03 | Input validation / Injection (accepted ROOM-D1) | `getRoomsDetails` `{ "roomId": "<101 hex>" }`; `{ "roomId": "' OR 1=1 --" }` | 500 `{ status: false, message: "Failed to retrieve rooms", data: null }` both | — |
| ROOM-02-D04 | Business rule (accepted ROOM-D1) | `roomSave` `{ "roomName": "201", "floorId": "<FLOOR-G hex>", "capacity": 2 }` | Failed | no row |
| ROOM-02-D05 | Business-rule bypass / Injection (accepted ROOM-D3) | `roomSave` with `floorId` = (a) `"abc"` (b) `"' OR 1=1 --"` (11 chars) | 200 `room created successfully` both | orphan rows with `floorId` = those bytes padded; they appear in `getRoomsDetails {}` |
| ROOM-02-D06 | Business rule (accepted ROOM-D1) | `roomSave` `{ "roomId": "<101 hex>", "isActive": false }`; `{ "roomId": "<103 hex>", "isActive": true }` | Failed both | unchanged — `103` stays inactive and unlisted |
| ROOM-02-D07 | Input validation | create with (a) no `roomName` (b) no `floorId` (c) no `capacity` (d) `capacity: "two"` (e) `status: 5` | 422: (a) `The room name field is required.` (b) `The floor id field is required.` (c) `The capacity field is required.` (d) `The capacity field must be an integer.` (e) `The selected status is invalid.` | none |
| ROOM-02-D08 | Business-rule bypass (accepted ROOM-D3) | create with `floorId: "abc"` and `capacity` 0, then -3 | 200 both | stored |
| ROOM-02-D09 | Input validation | create with `floorId: "abc"`, `roomName` 256 × `a` | Failed | none |
| ROOM-02-D10 | Role (accepted ROOM-D2) | WTR-A: create with `floorId: "abc"`; CHF-A: `getRoomsDetails {}` | 200 both | one row |
| ROOM-02-D11 | Restaurant isolation | (a) `getRoomsDetails {}` (b) `roomSave` update with `roomId` = ROOM-B1 hex | (a) no REST-B room (b) Failed | ROOM-B1 unchanged |
| ROOM-02-D12 | Authentication / Account state / Super-admin / Rate limit | both endpoints: no token → 401; `GET` → 405; OWN-P → 200; NOREST → 403; panel cookie only → 401; 60 calls/min → no 429 | as listed | — |
| ROOM-02-D13 | Over-posting | create with `floorId: "abc"`, plus `restaurantId: "<REST-B>"`, `version: 9`, `createdBy: "x"` | 200 | row in REST-A; `version` 0; `createdBy` NULL |
| ROOM-02-D14 | — (duplicate rows, accepted ROOM-D1) | Add a second running order on `101`; `getRoomsDetails {}` | `101` appears twice, once per `orderId` | — |
| ROOM-02-D15 | Error hygiene | the 500 bodies of D03, D04, D09 | No `error` key, no SQL | — |
