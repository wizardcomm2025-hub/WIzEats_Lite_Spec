# {{FEATURE_NAME}} — API contract

Shared shapes (success, 422 validation, 401, 403, 429, 500) are defined once in [project-overview.md → API conventions](../../project-overview.md#api-conventions). This file lists only what is specific to each endpoint.

## Endpoints

| Endpoint | Middleware | Roles allowed | Used by use case |
|---|---|---|---|
| `POST /api/<!-- module/action -->` | `auth:sanctum`, `restaurant.tenant` | <!-- owner --> | [01](./01-use-case.md) |

---

## `POST /api/<!-- module/action -->`

**Purpose:** <!-- one line -->

### Request

| Field | Type | Required | Rule |
|---|---|---|---|
| <!-- --> | | | |

Fields that must be **ignored** if sent (over-posting): `restaurantId`, `userId`, `createdBy`<!-- , … -->.

### Response — success

HTTP 200

```json
{ "status": true, "message": "<!-- exact text -->", "data": { } }
```

### Messages

Every message this endpoint can return. Tests quote these verbatim.

| Situation | HTTP | `status` | `message` |
|---|---|---|---|
| Success | 200 | true | <!-- --> |
| <!-- Missing field --> | 422 | false | <!-- --> |
| <!-- Duplicate --> | 422 | false | <!-- --> |
| <!-- Not found / other restaurant's ID --> | 404 | false | <!-- --> |
| <!-- Wrong role --> | 403 | false | <!-- --> |
| No / invalid token | 401 | — | `Unauthenticated.` |
| Unexpected error | 500 | false | <!-- no exception text --> |
