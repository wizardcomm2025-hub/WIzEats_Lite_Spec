# Restaurant profile & settings — API contract

Shared shapes: [project-overview.md → API conventions](../../project-overview.md#api-conventions). Facts from the code on 2026-11-06.

## Endpoints

| Endpoint | Middleware | Rate limit | Used by | Use case |
|---|---|---|---|---|
| `POST /api/restaurants/showDetails` | `auth:sanctum`, `restaurant.tenant` | none | App (Home, Orders, Profile, approval check) | [01](./01-view-restaurant-profile.md) |
| `POST /api/restaurants/updateDetails` | `auth:sanctum`, `restaurant.tenant` | none | App (Profile) | [02](./02-edit-profile-and-settings.md) |
| `GET /admin/restaurant-management/{restaurantId}` (web) | `web`, `auth:admin` | none | Panel | [03](./03-edit-in-panel.md) |
| `POST /admin/restaurant/update` (web) | `web` (CSRF), `auth:admin` | none | Panel | [03](./03-edit-in-panel.md) |
| `POST /admin/restaurant/config/update` (web) | `web` (CSRF), `auth:admin` | none | Panel | [03](./03-edit-in-panel.md) |

No role check on any API endpoint (RST-D3). Common failures for both API endpoints:

| Situation | HTTP | Body |
|---|---|---|
| No token / invalid token | 401 | `{ "message": "Unauthenticated." }` |
| User has no `restaurantId` | 403 | `Restaurant context missing` (tenant middleware shape — see project overview) |
| `GET` instead of `POST` | 405 | Method Not Allowed |

A switched-off user, or a Pending / Rejected / switched-off restaurant, still gets 200 (AUTH-D2).

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
| `enableRestaurant`, `enableDelivery`, `enableTakeAway`, `enableRoomService`, `enableDiscount`, `showKotInBill` | `nullable|boolean` | same name in `restaurant_configs` (`null` → 500, NOT NULL) |
| `enableKot` | `nullable|boolean` | **none** — validated, never saved (RST-D2) |
| `taxPercent` | `nullable` (no type) | `restaurant_configs.taxPercent` decimal(5,2) |
| `serviceChargePercent` | `nullable|decimal:2` | `restaurant_configs.serviceChargePercent` |
| `invocePrefix` | `nullable|string` | **none** — misspelt, never saved (RST-D2) |
| `qr`, `kotDisplayName` | `nullable|string` | same name in `restaurant_configs` |

`enableKOT` and `invoicePrefix` (correct spellings) are not in the rules, so they are dropped too.

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
| Service charge not exactly 2 decimals | `The service charge percent field must have 2 decimal places.` |

Booleans accept `true`, `false`, `1`, `0`, `"1"`, `"0"`.

### Side effects

Order channel updates are attempted and match nothing (RST-D1). No e-mail is sent.

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

The values are the form's. `enableKot` is the held KOT setting, or `false` when the record has none. Empty optional texts go as `""` or `null`.

---

## Panel routes (web)

All three run behind the `web` group and `auth:admin`. The two POSTs need the session's CSRF token (the page sends it as the `X-CSRF-TOKEN` header). None validates input.

### `GET /admin/restaurant-management/{restaurantId}`

`{restaurantId}` is 32-char hex or a dashed UUID. The response is the HTML page ([ui-ux.md §3](./ui-ux.md#3-panel--restaurant-management-page)). An unknown ID gives a 500 error page (the view reads properties of a missing row).

### `POST /admin/restaurant/update`

JSON body: `restaurantId`, `restaurantName`, `restaurantContact`, `restaurantOwnerName`, `restaurantGstId`, `restaurantFssaiId`, `restaurantAddress`.

| Situation | Result |
|---|---|
| Any values | All six columns set to what was sent; a field left out is set to `NULL`. `updated_at` = now. `{ "success": true }` |
| Unknown `restaurantId` | Nothing changes; still `{ "success": true }` |
| `restaurantId` missing | 500 (type error) |
| Text too long for the column | 500, nothing saved |
| Other fields (`isApproved`, `status`, `restaurantEmail`, …) | Ignored |

### `POST /admin/restaurant/config/update`

JSON body: `restaurantId`, `enableRestaurant`, `enableDelivery`, `enableKOT`, `enableRoomService` (1 / 0), `taxPercent`, `serviceChargePercent`, `invoicePrefix`.

| Situation | Result |
|---|---|
| Valid values | Config row updated with all seven values; `{ "success": true }` |
| A switch missing / `null` | 500 (NOT NULL), nothing saved |
| Tax / service charge not a number or ≥ 1000 | 500, nothing saved |
| Tax / service charge empty | Stored `NULL` |
| Negative tax / service charge | Stored (RST-D4) |
| `restaurantId` unknown, or the restaurant has no config row | Insert attempted without `configId` → 500 |
| `enableTakeAway`, `enableDiscount`, `qr`, … | Ignored |

Common failures for both POSTs:

| Situation | Result |
|---|---|
| No panel session | 302 to `/`; with `Accept: application/json`: 401 `{ "message": "Unauthenticated." }` |
| No or wrong CSRF token | 419 `Page Expired`; JSON `{ "message": "CSRF token mismatch." }` |
| POS bearer token instead of a session | Same as no session |
