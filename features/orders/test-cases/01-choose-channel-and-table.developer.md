# 01 — Choose a channel and a table — developer cases

Use case: [01-choose-channel-and-table.md](../01-choose-channel-and-table.md). Contract: [api-contract.md](../api-contract.md#post-apiordersgetorderchannels). Fixtures: [README.md](./README.md#fixtures).

| ID | Category | Request / manipulation | Expected | Database after |
|---|---|---|---|---|
| ORD-01-D01 | — (happy path) | OWN-A `getOrderChannels` `{}` | 200 `Order channels retrieved successfully`; 4 rows `DINE_IN`, `TAKEAWAY`, `ROOM_SERVICE`, `DELIVERY`, hex IDs | — |
| ORD-01-D02 | — (occupy) | `tableSave` `{ tableId: <Table 1>, status: 1 }`; `showDetails` | 200; Table 1 `status` 1, `status_key` `occupied`, `orderId` null | Table 1 status 1 |
| ORD-01-D03 | — (leave) | `tableSave` `{ tableId: <Table 1>, status: 0 }` | 200 | status 0 |
| ORD-01-D04 | Input validation / Injection | `getOrderChannels` `{ "channelId": "<CH-DINE>" }`; `{ "channelId": "' OR 1=1 --" }`; `{ "channelId": [1] }` | first two: all 4 channels (filter unused); third: 422 `Validation error` | — |
| ORD-01-D05 | Authentication / Super-admin | no token → 401; panel cookie only → 401; `GET` → 405 | as listed | — |
| ORD-01-D06 | Account state (AUTH-D2) | OWN-P → 200; WTR-A-OFF old token → 200; NOREST → 403 | as listed | — |
| ORD-01-D07 | Restaurant isolation / Error hygiene | OWN-A `getOrderChannels` | no REST-B channel IDs | — |
| ORD-01-D08 | Role / Rate limiting | CHF-A → 200; 60 calls/min → no 429 | as listed | — |
