# Restaurant profile & settings — API contract

Shared shapes: [project-overview.md → API conventions](../../project-overview.md#api-conventions). Facts from the code on 2026-11-06, re-checked 2026-11-24 (API commit d54d6c8). Rows marked **Target** describe agreed fixes (RST-D2 to RST-D6, X-D1, X-D2) and are not in the code yet.

## Endpoints

| Endpoint | Middleware | Rate limit | Used by | Use case |
|---|---|---|---|---|
| `POST /api/restaurants/showDetails` | `auth:sanctum`, `restaurant.tenant` | none | App (Home, Orders, Profile, approval check) | [01](./01-view-restaurant-profile.md) |
| `POST /api/restaurants/updateDetails` | `auth:sanctum`, `restaurant.tenant` | none | App (Profile) | [02](./02-edit-profile-and-settings.md) |
| `GET /admin/restaurant-management/{restaurantId}` (web) | `web`, `auth:admin` | none | Panel | [03](./03-edit-in-panel.md) |
| `POST /admin/restaurant/update` (web) | `web` (CSRF), `auth:admin` | none | Panel | [03](./03-edit-in-panel.md) |
| `POST /admin/restaurant/config/update` (web) | `web` (CSRF), `auth:admin` | none | Panel | [03](./03-edit-in-panel.md) |

Roles: today none checked (RST-D3). **Target (X-D1):** `showDetails` — owner, waiter, chef; `updateDetails` — owner only, else 403 `{ "status": false, "message": "You do not have permission to perform this action", "data": null }`. Common failures for both API endpoints:

| Situation | HTTP | Body |
|---|---|---|
| No token / invalid token | 401 | `{ "message": "Unauthenticated." }` |
| User has no `restaurantId` | 403 | `Restaurant context missing` (tenant middleware shape — see project overview) |
| `GET` instead of `POST` | 405 | Method Not Allowed |

Today a switched-off user, or a Pending / Rejected / switched-off restaurant, still gets 200 (AUTH-D2). **Target (X-D2):** 403 `Your account is inactive.` / `Your restaurant is not approved.` / `Your restaurant is inactive.`; `showDetails` skips only the approval check.

---

## `POST /api/restaurants/showDetails`

**Request:** no fields. Any body is ignored, including a `restaurantId`.

| Situation | HTTP | Body |
|---|---|---|
| Found | 200 | `{ "status": true, "message": "Restaurant details fetched successfully", "data": { … } }` |
| Tenant restaurant row missing | 404 | `{ "status": false, "message": "Restaurant not found", "data": null }` |
| Database error | 500 | `{ "status": false, "message": "Failed to fetch restaurant details", "data": null }` |

**`data`** — every column of the `restaurants` row (IDs as 32-char hex), plus:

| Key | Content |
|---|---|
| `restaurantConfig` | the whole `restaurant_configs` row: switches as `true` / `false`, `taxPercent` / `serviceChargePercent` as `"5.00"` strings or `null`, `kotDisplayName`, `qr`, `invoicePrefix`, `restaurantCode`, IDs as hex |
| `zone` | array of active zones; each has `tables` — active tables with `orderId` of the latest running order or `null` |

Values are **not** hidden. There is no `pin` or user data in this response. Hex quirk: see [data-model.md → Rules](./data-model.md#rules) (F2).

---

## `POST /api/restaurants/updateDetails`

### Request

Every field is optional. Only the fields below are used; anything else is dropped. A field sent as `null` (or `""`, which becomes `null`) is written as `NULL`.

| Field | Rule | Column |
|---|---|---|
| `restaurantName`, `restaurantAddress`, `restaurantOwnerName`, `restaurantGstId`, `restaurantFssaiId`, `companyAddress`, `restaurantContact`, `alternateMobileNo` | `nullable|string` | same name in `restaurants` |
| `restaurantEmail` | `nullable|email` | `restaurants.restaurantEmail` |
| `restaurantLogo` | `nullable|url` | `restaurants.restaurantLogo` |
| `enableRestaurant`, `enableDelivery`, `enableTakeAway`, `enableRoomService`, `enableDiscount`, `showKotInBill` | `nullable|boolean` (**target** `sometimes|boolean`) | same name in `restaurant_configs` (today `null` → 500, NOT NULL; target 422) |
| `enableKot` | `nullable|boolean` | today **none** — validated, never saved (RST-D2). **Target:** saved to `enableKOT` |
| `enableKOT` | today not a rule (dropped). **Target:** `sometimes|boolean` | `restaurant_configs.enableKOT` |
| `taxPercent` | `nullable` (no type) (**target** `nullable|numeric|min:0|max:100`) | `restaurant_configs.taxPercent` decimal(5,2) |
| `serviceChargePercent` | `nullable|decimal:2` (**target** `nullable|numeric|min:0|max:100`) | `restaurant_configs.serviceChargePercent` |
| `invocePrefix` | `nullable|string` | **none** — misspelt, never saved (RST-D2). **Target:** no longer a rule |
| `invoicePrefix` | today not a rule (dropped). **Target:** `nullable|string|max:255` | `restaurant_configs.invoicePrefix` |
| `qr`, `kotDisplayName` | `nullable|string` | same name in `restaurant_configs` |

Today `enableKOT` and `invoicePrefix` (correct spellings) are not in the rules, so they are dropped too.

**Target text rules (RST-D4):** `restaurantName`, `restaurantOwnerName` `sometimes|filled|string|max:255` (blank or `null` refused); other text fields add `max:255`, except `restaurantAddress` `max:65535`, `restaurantGstId` `max:15`, `kotDisplayName` `max:100`.

### Responses

| Situation | HTTP | Body |
|---|---|---|
| Saved | 200 | `{ "status": true, "message": "Restaurant details updated successfully", "data": { "restaurant": { …row }, "restaurantConfig": { …row } } }` |
| Validation | 422 | `{ "status": false, "message": "<first error>", "errors": { … } }` — messages below |
| Tenant restaurant row missing | 404 | `{ "status": false, "message": "Restaurant not found", "data": null }` |
| Database refuses a value (tax text or ≥ 1000, text too long, `null` switch) / any error | 500 | `{ "status": false, "message": "Failed to update restaurant details", "data": null }` — no exception text; nothing saved |

Validation messages (Laravel defaults):

| Failure | `message` |
|---|---|
| A text field not a string | `The <field> field must be a string.` (e.g. `The restaurant name field must be a string.`) |
| A switch not boolean | `The <field> field must be true or false.` (e.g. `The enable discount field must be true or false.`) |
| E-mail | `The restaurant email field must be a valid email address.` |
| Logo | `The restaurant logo field must be a valid URL.` |
| Service charge not exactly 2 decimals | `The service charge percent field must have 2 decimal places.` (today only; the target drops this rule) |
| **Target** blank name / owner | `The restaurant name field must have a value.` / `The restaurant owner name field must have a value.` |
| **Target** too long | `The <field> field must not be greater than <n> characters.` |
| **Target** tax / service charge | `The tax percent field must be a number.` · `… must be at least 0.` · `… must not be greater than 100.` (same with `service charge percent`) |
| **Target** `null` switch | `The <field> field must be true or false.` |

Booleans accept `true`, `false`, `1`, `0`, `"1"`, `"0"`.

### Side effects

When `enableTakeAway`, `enableDelivery` or `enableRoomService` is sent, the channel with `channelCode` `TAKEAWAY`, `DELIVERY` or `ROOM_SERVICE` gets `isActive` = that value, in the same transaction (since 2026-11-16, RST-D1 fixed). `orders/getOrderChannels` then lists only active channels. No e-mail is sent.

### What the app sends

```json
{
  "restaurantName": "…", "restaurantAddress": "…", "restaurantOwnerName": "…",
  "restaurantGstId": "…", "restaurantFssaiId": "…", "companyAddress": null,
  "restaurantContact": "…", "alternateMobileNo": "…" ,
  "enableRestaurant": true, "enableKot": false, "enableTakeAway": false,
  "enableDelivery": false, "enableRoomService": false,
  "taxPercent": "0", "serviceChargePercent": null, "restaurantEmail": "…",
  "qr": "", "enableDiscount": false, "kotDisplayName": "KOT", "showKotInBill": false
}
```

The values are the form's. `enableKot` is the held KOT setting, or `false` when the record has none. Empty optional texts go as `""` or `null`. **Target (RST-D2):** the same body **without** `serviceChargePercent` and `companyAddress`.

---

## Panel routes (web)

All three run behind the `web` group and `auth:admin`. The two POSTs need the session's CSRF token (the page sends it as the `X-CSRF-TOKEN` header). Today none validates input. **Target (RST-D4):** validated; failures answer 422 JSON `{ "message": "<first error>", "errors": { … } }` (Laravel default; the page's script still shows nothing).

### `GET /admin/restaurant-management/{restaurantId}`

`{restaurantId}` is 32-char hex or a dashed UUID. The response is the HTML page ([ui-ux.md §3](./ui-ux.md#3-panel--restaurant-management-page)). An unknown ID gives a 500 error page (the view reads properties of a missing row). **Target (RST-D4 / X-D4):** 404.

### `POST /admin/restaurant/update`

JSON body: `restaurantId`, `restaurantName`, `restaurantContact`, `restaurantOwnerName`, `restaurantGstId`, `restaurantFssaiId`, `restaurantAddress`.

| Situation | Result |
|---|---|
| Any values | All six columns set to what was sent; a field left out is set to `NULL`. `updated_at` = now. `{ "success": true }` |
| Unknown `restaurantId` | Nothing changes; still `{ "success": true }`. **Target:** 404 |
| `restaurantId` missing | 500 (type error). **Target:** 422 `The restaurant id field is required.` |
| Text too long for the column | 500, nothing saved. **Target:** 422 |
| **Target (RST-D4)** name or owner name empty | 422 `The restaurant name field is required.` / `The restaurant owner name field is required.`; nothing saved (today: saved as `NULL`) |
| Other fields (`isApproved`, `status`, `restaurantEmail`, …) | Ignored |

### `POST /admin/restaurant/config/update`

JSON body: `restaurantId`, `enableRestaurant`, `enableDelivery`, `enableKOT`, `enableRoomService` (1 / 0), `taxPercent`, `serviceChargePercent`, `invoicePrefix`.

| Situation | Result |
|---|---|
| Valid values | Config row updated with all seven values; `{ "success": true }`. **Target (RST-D5):** the `DELIVERY` and `ROOM_SERVICE` channels' `isActive` set to `enableDelivery` / `enableRoomService` in the same transaction (today: channels untouched) |
| A switch missing / `null` | 500 (NOT NULL), nothing saved. **Target:** 422 `The <field> field is required.` |
| Tax / service charge not a number or ≥ 1000 | 500, nothing saved. **Target:** 422 (numeric 0–100) |
| Tax / service charge empty | Stored `NULL` |
| Negative tax / service charge | Stored (RST-D4). **Target:** 422 `… must be at least 0.` |
| `restaurantId` unknown, or the restaurant has no config row | Insert attempted without `configId` → 500. **Target:** unknown → 404; no config row → one is created, 200 |
| `enableTakeAway`, `enableDiscount`, `qr`, … | Ignored |

Common failures for both POSTs:

| Situation | Result |
|---|---|
| No panel session | 302 to `/`; with `Accept: application/json`: 401 `{ "message": "Unauthenticated." }` |
| No or wrong CSRF token | 419 `Page Expired`; JSON `{ "message": "CSRF token mismatch." }` |
| POS bearer token instead of a session | Same as no session |
