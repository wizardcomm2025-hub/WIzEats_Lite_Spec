# 02 — Edit profile and settings in the app — developer cases

Use case: [02-edit-profile-and-settings.md](../02-edit-profile-and-settings.md). Contract: [api-contract.md](../api-contract.md#post-apirestaurantsupdatedetails). Fixtures: [README.md](./README.md#fixtures).

All requests: `POST /api/restaurants/updateDetails`, JSON, OWN-A's bearer token unless stated. REST-A reset to REST-A-BASE before each case. "Saved" = 200 `{ status: true, message: "Restaurant details updated successfully", data: { restaurant, restaurantConfig } }`. "Failed" = 500 `{ status: false, message: "Failed to update restaurant details", data: null }`.

| ID | Category | Request / manipulation | Expected | Database after |
|---|---|---|---|---|
| RST-02-D01 | — (happy path) | app-full with `restaurantName: "Rest A2"`, `taxPercent: "7"`, `qr: "UPI-X"`, `enableDiscount: false`, `showKotInBill: true`, `kotDisplayName: "Token"` | Saved; `data.restaurant.restaurantName` `Rest A2`; `data.restaurantConfig.taxPercent` `"7.00"` | `restaurants` and `restaurant_configs` hold the sent values; `updated_at` now |
| RST-02-D02 | — (partial) | `{ "restaurantName": "Only Name" }` | Saved | only `restaurantName` changed; everything else as before |
| RST-02-D03 | Input validation | (a) `{}` (b) a body that is not JSON (`Content-Type: application/json`, body `{{{`) | Saved (nothing to do) | unchanged |
| RST-02-D04 | Business rule (accepted RST-D2) | (a) `{ "enableKot": false }` (b) `{ "enableKOT": false }` | Saved both | `enableKOT` still 1 |
| RST-02-D05 | Business rule (accepted RST-D2) | (a) `{ "invoicePrefix": "INV" }` (b) `{ "invocePrefix": "INV" }` | Saved both | `invoicePrefix` still NULL |
| RST-02-D06 | Business-rule bypass (accepted RST-D1) | `{ "enableTakeAway": false, "enableDelivery": false, "enableRoomService": false }`; then `POST /api/orders/getOrderChannels` | Saved; channel list still has all four (`Dine In`, `Takeaway`, `Room Service`, `Delivery`) | config: the three flags 0; `order_channels` for REST-A all `isActive = 1` |
| RST-02-D07 | Business rule (accepted RST-D2) | Set `serviceChargePercent = 10.00`, `companyAddress = 'HQ'` in the database; send app-full | Saved | `serviceChargePercent` NULL, `companyAddress` NULL |
| RST-02-D08 | Input validation | `{ "restaurantName": "New", "enableTakeAway": null }` | Failed | unchanged — the name is **not** saved either (rollback) |
| RST-02-D09 | Input validation (accepted RST-D4) | (a) `{ "restaurantName": null }` (b) `{ "restaurantName": "" }` (c) `{ "restaurantName": "   " }` (d) `{ "restaurantOwnerName": "" }` | Saved each | `restaurantName` NULL (a–c); `restaurantOwnerName` NULL (d) |
| RST-02-D10 | Input validation (accepted RST-D4) | `taxPercent` = (a) `-5` (b) `"abc"` (c) `1000` (d) `12.345` (e) `999.99` (f) `[1]` | (a) Saved (b) Failed (c) Failed (d) Saved (e) Saved (f) Failed | (a) `-5.00` (b)(c)(f) unchanged (d) `12.35` (e) `999.99` |
| RST-02-D11 | Input validation (accepted RST-D4) | `serviceChargePercent` = (a) `"5"` (b) `5.5` (c) `"5.00"` (d) `"-1.00"` | (a)(b) 422 `The service charge percent field must have 2 decimal places.` (c)(d) Saved | (c) `5.00` (d) `-1.00` |
| RST-02-D12 | Input validation | `restaurantEmail` = (a) `"owner-at-cafe"` (b) `"a@b"` (c) `null` | (a) 422 `The restaurant email field must be a valid email address.` (b)(c) Saved | (b) `a@b` (c) NULL |
| RST-02-D13 | Input validation | `restaurantLogo` = (a) `"not a url"` (b) `"https://example.com/l.png"` | (a) 422 `The restaurant logo field must be a valid URL.` (b) Saved | (b) stored |
| RST-02-D14 | Input validation | `enableDiscount` = (a) `"yes"` (b) `"1"` (c) `2` | (a)(c) 422 `The enable discount field must be true or false.` (b) Saved | (b) `enableDiscount` 1 |
| RST-02-D15 | Input validation | `restaurantName` = (a) `123` (b) `["x"]` | 422 `The restaurant name field must be a string.` | unchanged |
| RST-02-D16 | Input validation / Error hygiene | (a) `restaurantName` = TEXT-256 (b) `kotDisplayName` = 101 × `a` (c) `restaurantAddress` = 70 000 characters | (a)(b) Failed (c) Saved (`text` column) | (a)(b) unchanged |
| RST-02-D17 | Over-posting / Restaurant isolation | app-full plus `restaurantId: "<REST-B hex>"`, `isApproved: 1`, `status: 0`, `isOnboardingDone: 0`, `restaurantCode: "HACKED"`, `enableKotGrouping: 1`, `createdBy: "x"` | Saved | REST-A: only app-full fields changed; `isApproved`, `status`, `isOnboardingDone`, `restaurantCode`, `enableKotGrouping`, `createdBy` unchanged. REST-B untouched. |
| RST-02-D18 | Role (accepted RST-D3) | (a) WTR-A: `{ "qr": "UPI-X" }` (b) CHF-A: `{ "taxPercent": "0" }` | Saved both | `qr` `UPI-X`; `taxPercent` `0.00` |
| RST-02-D19 | Authentication / Account state | (a) no token (b) `Bearer garbage` (c) OWN-P (d) WTR-A-OFF token from before the switch-off (e) NOREST (f) `GET` | (a)(b) 401 `{ "message": "Unauthenticated." }` (c)(d) Saved (AUTH-D2) (e) 403 `Restaurant context missing` (f) 405 | (c)(d) changed; others unchanged |
| RST-02-D20 | Concurrency | Two app-full requests in parallel: names `Par 1` / `Par 2`, taxes `1` / `2` | Both Saved | `restaurantName` ∈ {`Par 1`, `Par 2`}, `taxPercent` ∈ {`1.00`, `2.00`} (they may come from different requests); no error, no partial row |
| RST-02-D21 | Injection | `restaurantName` = `'); DROP TABLE restaurants; --`; `restaurantAddress` = `<script>alert(1)</script>`; `kotDisplayName` = `{{7*7}}` | Saved | stored exactly as sent; all tables intact; the panel page shows the text, nothing runs |
| RST-02-D22 | Error hygiene | The 500 responses of D08, D10, D16 | Body exactly `{ status: false, message: "Failed to update restaurant details", data: null }` — no `error` key, no SQL, no stack trace | — |
| RST-02-D23 | Rate limiting | 60 saves in one minute | All Saved; no 429 | last values |
| RST-02-D24 | Error hygiene (finding F2) | (a) `{ "restaurantName": "Café Délice 12" }` (b) then app-full with `restaurantName: "436166c3a92044c3a96c696365203132"` (what the app would send back) | (a) Saved; `data.restaurant.restaurantName` = `"436166c3a92044c3a96c696365203132"` (b) Saved | after (a): `Café Délice 12`; after (b): the hex string |
| RST-02-D25 | Input validation | `restaurantGstId` = `"NOT-A-GSTIN"` | Saved (the GSTIN check is only in the app) | stored |
| RST-02-D26 | Super-admin boundary | No bearer; ADMIN-1's panel session cookie and CSRF header | 401 `{ "message": "Unauthenticated." }` | unchanged |
| RST-02-D27 | — (effect on orders) | Open an order on a table (ORD `createOrder`) at tax 5; send `{ "taxPercent": "7" }`; create a second order | Saved | first order `taxPercent` 5.00; second 7.00 |
