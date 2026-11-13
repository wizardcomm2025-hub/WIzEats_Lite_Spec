# Staff management — API contract

Shared shapes: [project-overview.md → API conventions](../../project-overview.md#api-conventions). Facts from the code on 2026-11-10.

## Endpoints

| Endpoint | Middleware | Rate limit | Used by | Use case |
|---|---|---|---|---|
| `POST /api/staff/getStaffDetails` | `auth:sanctum`, `restaurant.tenant` | none | App | [01](./01-see-staff-list.md) |
| `POST /api/staff/create` | `auth:sanctum`, `restaurant.tenant` | none | App (owner) | [02](./02-add-staff.md) |
| `POST /api/staff/update` | `auth:sanctum`, `restaurant.tenant` | none | App (owner) — fails today (STAFF-D1) | [03](./03-switch-staff-on-off.md) |
| `GET /admin/user-management/{restaurantId}` (web) | `web`, `auth:admin` | none | Panel | [04](./04-edit-users-in-panel.md) |
| `POST /admin/user/update` (web) | `web` (CSRF), `auth:admin` | none | Panel | [04](./04-edit-users-in-panel.md) |

There is no role check on any endpoint (STAFF-D3). The common failures of the three API endpoints:

| Situation | HTTP | Body |
|---|---|---|
| No token / invalid token | 401 | `{ "message": "Unauthenticated." }` |
| User has no `restaurantId` | 403 | `Restaurant context missing` |
| `GET` instead of `POST` | 405 | Method Not Allowed |
| Validation | 422 | `{ "status": false, "message": "<first error>", "errors": { … } }` |

A switched-off user, or a Pending / Rejected restaurant, still gets through (AUTH-D2).

---

## `POST /api/staff/getStaffDetails`

| Field | Rule |
|---|---|
| `userId` | optional, string. Must be 32-character hex if present; otherwise 500 |

| Situation | HTTP | Body |
|---|---|---|
| OK | 200 | `{ "status": true, "message": "Staff members fetched successfully", "data": [ …users ] }` — the caller's restaurant's users with role waiter / chef (any case), optionally only `userId`. Each user: `userId`, `restaurantId` (hex), `name`, `phone`, `role`, `status` (bool), `mustChangePin` (bool), `createdBy`, `countryCode`, `email`, `phone_verified_at`, `created_at`, `updated_at`. No `pin`. |
| `userId` of another restaurant, of the owner, or unknown | 200 | `data: []` |
| `userId` not hex / odd length | 500 | `{ "status": false, "message": "Failed to fetch staff members", "data": null }` |

---

## `POST /api/staff/create`

| Field | Rule | Message on failure |
|---|---|---|
| `name` | required, string, max 255 | `The name field is required.` / `The name field must be a string.` / `The name field must not be greater than 255 characters.` |
| `phone` | required, regex `^[6-9][0-9]{9}$` (a JSON number is accepted) | `The phone field is required.` / `The phone field format is invalid.` |
| `role` | required, string — **any text** (STAFF-D3) | `The role field is required.` / `The role field must be a string.` |

Other fields (`restaurantId`, `status`, `pin`, `mustChangePin`, `userId`) are ignored.

| Situation | HTTP | Body |
|---|---|---|
| Created | 200 | `{ "status": true, "message": "Staff member created successfully", "data": { userId, restaurantId, name, phone, role, status: true, mustChangePin: true, createdBy, created_at, updated_at } }` — no `pin` |
| Phone already used anywhere | 500 | `{ "status": false, "message": "Failed to create staff member", "data": null }` (STAFF-D3) |

**Writes:** one `users` row — caller's restaurant, `status = 1`, PIN `123456` hashed (STAFF-D4), `mustChangePin = 1`, `createdBy` = caller's role.

---

## `POST /api/staff/update`

| Field | Rule | Message on failure |
|---|---|---|
| `userId` | required, string; used as 32-character hex | `The user id field is required.` |
| `name` | required, string, max 255 | `The name field is required.` … |
| `status` | required, boolean | `The status field is required.` / `The status field must be true or false.` |

The app sends `isActive` instead of `status`, so it always gets the `status` message (STAFF-D1). `role`, `phone` and `restaurantId` are ignored.

| Situation | HTTP | Body |
|---|---|---|
| Updated (**any** restaurant's user — STAFF-D2) | 200 | `{ "status": true, "message": "Staff member updated successfully", "data": null }` — `data` is always null |
| Valid hex, no such user | 404 | `{ "status": false, "message": "Staff member not found", "data": null }` |
| `userId` not hex / odd length | 500 | `{ "status": false, "message": "Failed to update staff member", "data": null }` |

**Writes:** `users.name`, `users.status` and `updated_at`.

---

## Panel routes (web)

### `GET /admin/user-management/{restaurantId}`

32-character hex or a dashed UUID. Lists all users of the restaurant (any role), 3 per page (`?page=n`), with `userId` as hex. An unknown ID gives an empty table.

### `POST /admin/user/update`

JSON `{ id, name, role, status }`, header `X-CSRF-TOKEN`.

| Situation | Result |
|---|---|
| Any values | `name`, `role`, `status` set as sent (empty → NULL; `updated_at` not set); `{ "success": true }` |
| Unknown `id` | Nothing changes; `{ "success": true }` |
| `id` missing | 500 |
| `status` missing / empty / not 0–1 numeric | 500 (NOT NULL / strict mode) — nothing saved |
| `phone`, `pin`, `restaurantId` sent | Ignored |
| No panel session | 302 to `/`; JSON 401 `{ "message": "Unauthenticated." }` |
| No / wrong CSRF | 419 |
| POS bearer token | Same as no session |
