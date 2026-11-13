# 01 — Manage floors — developer cases

Use case: [01-manage-floors.md](../01-manage-floors.md). Contract: [api-contract.md](../api-contract.md#post-apiroomsfloorsave). Fixtures: [README.md](./README.md#fixtures).

JSON with OWN-A's bearer token unless stated. "Failed" = 500 `{ status: false, message: "Failed to save floor", data: null }`.

| ID | Category | Request / manipulation | Expected | Database after |
|---|---|---|---|---|
| ROOM-01-D01 | — (happy path) | `getFloorDetails` `{}` | 200 `Room floors retrieved successfully`; `Ground Floor` with `roomCount` 3, hex IDs | — |
| ROOM-01-D02 | — (happy path) | `floorSave` `{ "floorName": "Upper Floor" }` | 200 `floor created successfully`; `data.isActive` true, `createdBy` `owner`, no `roomCount` | one row in REST-A |
| ROOM-01-D03 | Business rule (accepted ROOM-D1) | `floorSave` `{ "floorId": "<FLOOR-G hex>", "floorName": "Level 0" }`; repeat with the dashed and the upper-case form | Failed (all) | unchanged |
| ROOM-01-D04 | Business rule (accepted ROOM-D1) | `floorSave` `{ "floorId": "<FLOOR-G hex>", "isActive": false }` | Failed | floor and rooms unchanged |
| ROOM-01-D05 | Input validation | `floorSave` (a) `{}` (b) `{ "floorName": 123 }` (c) `{ "floorName": "X", "isActive": "yes" }` | 422 (a) `The floor name field is required.` (b) `The floor name field must be a string.` (c) `The is active field must be true or false.` | none |
| ROOM-01-D06 | Input validation / Error hygiene | `floorSave` `{ "floorName": <256 × "a"> }` | Failed — no `error` key, no SQL | none |
| ROOM-01-D07 | Over-posting | create with `restaurantId: "<REST-B>"`, `createdBy: "admin"`, `floorId: ""` | 200 created | in REST-A; `createdBy` `owner` |
| ROOM-01-D08 | Role (accepted ROOM-D2) | WTR-A and CHF-A each create a floor; WTR-A lists floors | 200 each; `createdBy` `waiter` / `chef` | two rows |
| ROOM-01-D09 | Restaurant isolation / Injection | `getFloorDetails` (a) `{}` (b) `{ "floorId": "<FLOOR-G hex>" }` (c) `{ "floorId": "' OR 1=1 --" }` | (a) no REST-B floor (b)(c) 200 `data: []` | — |
| ROOM-01-D10 | Authentication / Account state | both endpoints: no token / `Bearer garbage` → 401; `GET` → 405; OWN-P → 200; NOREST → 403 | as listed | — |
| ROOM-01-D11 | Injection | create `floorName` = `'); DROP TABLE floors; --` | 200 | stored exactly; tables intact |
| ROOM-01-D12 | Super-admin boundary / Rate limiting | panel cookie only → 401; 60 calls in a minute → no 429 | as listed | — |
| ROOM-01-D13 | Business-rule bypass (accepted ROOM-D2) | Room Service **off**; create `Hidden Floor` | 200 | floor created though the screens are hidden |
