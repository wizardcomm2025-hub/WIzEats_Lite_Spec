# 01 — View the restaurant profile — developer cases

Use case: [01-view-restaurant-profile.md](../01-view-restaurant-profile.md). Contract: [api-contract.md](../api-contract.md#post-apirestaurantsshowdetails). Fixtures: [README.md](./README.md#fixtures).

All requests: `POST /api/restaurants/showDetails`, body `{}`, the fixture user's bearer token unless stated. REST-A at REST-A-BASE.

| ID | Category | Request / manipulation | Expected | Database after |
|---|---|---|---|---|
| RST-01-D01 | — (happy path) | OWN-A | 200 `{ status: true, message: "Restaurant details fetched successfully", data }`. `data.restaurantId` = REST-A's 32-char hex. Name, owner, e-mail and so on are as stored. `data.restaurantConfig.taxPercent` = `"5.00"`, `qr` = `"resta@upi"`, `enableTakeAway`, `enableDiscount` and `showKotInBill` = `true`, `kotDisplayName` = `"KOT"`. `data.zone` holds REST-A's active zones, each with its active tables. | unchanged |
| RST-01-D02 | — (happy path) | OWN-FRESH | 200. `restaurantConfig`: `taxPercent` null, `serviceChargePercent` null, `kotDisplayName` `"kot"`, `enableDiscount` false, `showKotInBill` false, `enableTakeAway` false, `enableKOT` true, `qr` null, `restaurantCode` 6 characters. `zone` = one zone `Main Hall` with `Table 1`–`Table 4`, each `orderId` null. | unchanged |
| RST-01-D03 | Role (X-D1: every role may read) | (a) WTR-A (b) CHF-A | Both 200 with the same `data` as D01 — today and after X-D1. | unchanged |
| RST-01-D04 | Restaurant isolation / Over-posting / Injection | OWN-A with body (a) `{ "restaurantId": "<REST-B hex>" }` (b) `{ "restaurantId": "' OR 1=1 --" }` (c) `{ "restaurantId": null }` | All 200 with **REST-A's** data only; nothing from REST-B. | unchanged |
| RST-01-D05 | Authentication | (a) no `Authorization` (b) `Bearer garbage` (c) OWN-A's token after `POST /api/auth/logout` (d) `GET` with a valid token | (a)–(c) 401 `{ "message": "Unauthenticated." }` · (d) 405 | unchanged |
| RST-01-D06 | Account state — (c) *Fails until AUTH-D2 is fixed* | (a) OWN-P (Pending restaurant) (b) OWN-R (Rejected) (c) OWN-OFF, with a token taken **before** REST-OFF was switched off | (a)(b) 200 with their own restaurant's data (`showDetails` is exempt from the approval check — today and target) · (c) 403 `Your restaurant is inactive.` (today: 200) | unchanged |
| RST-01-D07 | Account state — (a) *Fails until AUTH-D2 is fixed* | (a) token of WTR-A-OFF taken **before** the switch-off (b) NOREST | (a) 403 `Your account is inactive.` (today: 200) · (b) 403 `Restaurant context missing` | unchanged |
| RST-01-D08 | Error hygiene | D01's response | No `pin`, `password`, `token` or user fields anywhere; IDs are hex, not raw bytes. | — |
| RST-01-D09 | Error hygiene (finding F2) | Set REST-A's `restaurantName` to NAME-HEX in the database, then call as OWN-A | 200; `data.restaurantName` = `"436166c3a92044c3a96c696365203132"`. Same for `kotDisplayName` = NAME-HEX inside `restaurantConfig`. | the stored name is still `Café Délice 12` |
| RST-01-D10 | — (data rules) | Give REST-A: one zone with `isActive = 0`, one with `isDelete = 1`; in an active zone one inactive table; table T1 with a running order (`orderStatus` 0) and table T2 whose only order is finished (2). | 200. The inactive and deleted zones and the inactive table are absent. T1 `orderId` = that order's ID; T2 `orderId` null. With two running orders on T1, the higher `orderId` is shown. | unchanged |
| RST-01-D11 | Super-admin boundary | No bearer; send ADMIN-1's panel session cookie instead | 401 `{ "message": "Unauthenticated." }` | unchanged |
| RST-01-D12 | Rate limiting | 60 calls in one minute as OWN-A | All 200; no 429. | unchanged |
