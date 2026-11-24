# 01 — Choose a channel and a table — developer cases

Use case: [01-choose-channel-and-table.md](../01-choose-channel-and-table.md). Fixtures: [README.md](./README.md#fixtures). Cases tagged *Fails until … is fixed* assert the target; today's result is in brackets.

| ID | Category | Request / manipulation | Expected | Database after |
|---|---|---|---|---|
| ORD-01-D01 | — (happy path) | OWN-A `getOrderChannels` `{}` | 200 `Order channels retrieved successfully`; 4 rows `DINE_IN`, `TAKEAWAY`, `ROOM_SERVICE`, `DELIVERY`, hex IDs | — |
| ORD-01-D02 | — (occupy) | `tableSave` `{ tableId: <Table 1>, status: 1 }`; `showDetails` | 200; Table 1 `status` 1, `status_key` `occupied`, `orderId` null | Table 1 status 1 |
| ORD-01-D03 | — (leave) | `tableSave` `{ tableId: <Table 1>, status: 0 }` | 200 | status 0 |
| ORD-01-D04 | Input validation / Injection | `getOrderChannels` `{ "channelId": "<CH-DINE>" }`; `{ "channelId": "' OR 1=1 --" }`; `{ "channelId": [1] }` | first two: all 4 channels (filter unused); third: 422 `Validation error` | — |
| ORD-01-D05 | Authentication / Super-admin | no token → 401; panel cookie only → 401; `GET` → 405 | as listed | — |
| ORD-01-D06 | Account state — *Fails until AUTH-D2 is fixed* | OWN-P; WTR-A-OFF's old token; NOREST | OWN-P → 403 `Your restaurant is not approved.`; WTR-A-OFF → 403 `Your account is inactive.` (today both 200); NOREST → 403 `Restaurant context missing` | — |
| ORD-01-D07 | Restaurant isolation / Error hygiene | OWN-A `getOrderChannels` | no REST-B channel IDs | — |
| ORD-01-D08 | Role / Rate limiting — chef part *Fails until ORD-D4 is fixed* | CHF-A; WTR-A; 60 calls/min | CHF-A → 403 `{ status: false, message: "You do not have permission to perform this action", data: null }` (today 200); WTR-A → 200; no 429 | — |
| ORD-01-D09 | Business rule (RST-D1 fixed) | switch Delivery off in Profile (RST); `getOrderChannels` | 3 channels; no `DELIVERY` | — |
