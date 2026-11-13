# Onboarding & approval — API contract

Shared shapes: [project-overview.md → API conventions](../../project-overview.md#api-conventions). Facts from the code on 2026-11-06.

## Endpoints

| Endpoint | Middleware | Rate limit | Used by | Use case |
|---|---|---|---|---|
| `POST /api/restaurants/setup` | public; a Sanctum token is **optional** (no tenant middleware) | 10 / min per IP | App (no token); old apps (token) | [01](./01-register-restaurant.md), [04](./04-legacy-setup-with-token.md) |
| `POST /api/restaurants/showDetails` | `auth:sanctum`, `restaurant.tenant` | none | App (approval refresh) | [02](./02-wait-for-approval.md) — contract owned by RST |
| `POST /admin/update-status/{restaurantId}` (web) | `web` group (session, CSRF), `auth:admin` | none | Super-admin panel | [03](./03-approve-or-reject.md) |

---

## `POST /api/restaurants/setup`

### Request

| Field | Type | Required | Rule |
|---|---|---|---|
| `phone` | string | **yes without a token**; ignored with one | `^[6-9][0-9]{9}$`, not already in `users` |
| `restaurantName` | string | yes | string; ≤ 255 characters or the database refuses it (500) |
| `restaurantAddress` | string | yes | string |
| `restaurantOwnerName` | string | yes | string |
| `restaurantEmail` | string | yes | Laravel `email` rule (the app is stricter — see [ui-ux.md](./ui-ux.md)) |
| `enableRestaurant` | boolean | yes | `true` / `false` / `1` / `0` / `"1"` / `"0"` |
| `enableDelivery` | boolean | yes | as above |
| `enableKot` | boolean | yes | as above |
| `enableRoomService` | boolean | yes | as above |
| `enableTakeAway` | boolean | yes | as above |

Any other field (`restaurantId`, `isApproved`, `status`, `role`, `pin`…) is ignored.

### Responses

| Situation | HTTP | Body |
|---|---|---|
| Onboarded | 200 | `{ "status": true, "message": "Restaurant onboarded successfully", "data": { …restaurant }, "token": "<id>|<secret>", "mustChangePin": true }` |
| A required field missing | 422 | `{ "status": false, "message": "The <field> field is required.", "errors": {…} }` — `<field>` is the name in words: `restaurant name`, `restaurant address`, `restaurant owner name`, `enable restaurant`, `enable delivery`, `enable kot`, `enable room service`, `enable take away`, `restaurant email` |
| A text field not a string | 422 | `message: "The <field> field must be a string."` |
| A flag not boolean | 422 | `message: "The <field> field must be true or false."` |
| E-mail invalid | 422 | `message: "The restaurant email field must be a valid email address."` |
| `phone` missing (no token) | 422 | `message: "Phone number is required"` |
| `phone` invalid (no token) | 422 | `message: "Please enter a valid mobile number"` |
| `phone` already registered (no token) | 422 | `message: "This phone number is already registered. Please log in."` |
| Token user's restaurant not found | 404 | `{ "status": false, "message": "Restaurant not found", "data": null, "token": null, "mustChangePin": null }` |
| Database refuses a value / unexpected error | 500 | `{ "status": false, "message": "Failed to onboard restaurant", "data": null, "token": null, "mustChangePin": null }` — no exception text |
| Over the limit | 429 | `{ "message": "Too Many Attempts." }` |

When several fields fail, `message` is the first failing field in this order: `restaurantName`, `restaurantAddress`, `restaurantOwnerName`, `enableRestaurant`, `enableDelivery`, `enableKot`, `enableRoomService`, `enableTakeAway`, `restaurantEmail`, `phone`. `errors` lists all.

**`data` on success** — the restaurant row (IDs as 32-char hex) plus:

| Key | Content |
|---|---|
| `isApproved` | `"Pending"` / `"Accepted"` / `"Rejected"` / `"Unknown"` |
| `isOnboardingDone` | `true` |
| `zone` | `{ zoneId, zoneName: "Main Hall", isActive }` |
| `tables` | the zone's tables |
| `restaurantConfig` | the config row incl. `restaurantCode` |
| `orderChannels` | the four channels |

`mustChangePin` is the caller's flag: `true` for a new owner; the caller's own value with a token.

### Side effects (after the data is committed)

1. Plain-text e-mail **"Restaurant Onboarding Completed"** — body `Restaurant <name> has completed onboarding successfully.` — to `ADMIN_EMAIL` (from `.env`). Sent on every successful call, with or without a token. Failure is logged only.
2. **New owner only:** `PinMail` (reason `onboarding`) to `restaurantEmail` with the PIN. Failure is logged only; the call still answers 200 (ONB-D3).

---

## `POST /admin/update-status/{restaurantId}` (web)

| Input | Rule |
|---|---|
| `{restaurantId}` (path) | 32-char hex or dashed UUID; anything else is used as is (matches nothing) |
| `isApproved` (form field) | No validation (ONB-D2). Stored as posted. |
| `_token` (form field) | CSRF token of the panel session |

| Situation | Result |
|---|---|
| Signed-in super admin, valid CSRF, any numeric `isApproved` | `isApproved` set to the value; 302 redirect back, session flash `success = Status updated successfully` (not displayed by the panel) |
| Unknown `restaurantId` | Nothing changes; same 302 redirect and flash |
| `isApproved` missing or non-numeric | Database error (strict mode): HTTP 500 error page; nothing changes |
| Value changed **and** new value is 1 or 2 **and** restaurant has an e-mail | `RestaurantStatusMail` to `restaurantEmail` — subject `Your restaurant has been approved` (1) or `Your restaurant registration was not approved` (2). Mail failure is logged only. |
| Same value posted again, or new value 0 / other | No e-mail |
| No CSRF token / wrong token | 419 (`Page Expired`; JSON `{ "message": "CSRF token mismatch." }` with `Accept: application/json`) — nothing changes |
| Not signed in to the panel (valid CSRF) | Redirect to `/` (panel login); with `Accept: application/json`: 401 `{ "message": "Unauthenticated." }` — nothing changes |
| A POS app bearer token instead of a panel session | Same as not signed in |
