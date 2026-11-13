# Kitchen (chef) — API contract

Shared shapes: [project-overview.md → API conventions](../../project-overview.md#api-conventions). Facts from the code on 2026-11-11.

| Endpoint | Middleware | Rate limit | Use case |
|---|---|---|---|
| `POST /api/chef/pending-kots` | `auth:sanctum`, `restaurant.tenant` | none | [01](./01-see-kots-by-stage.md) |
| `POST /api/chef/update-status/{kotId}` | same | none | [02](./02-move-kot-through-stages.md) |
| `POST /api/chef/dashboard` | same | none | [03](./03-chef-dashboard.md) |

There is no role check (KITCH-D2). The common failures: no or invalid token → 401 `{ "message": "Unauthenticated." }`; no `restaurantId` → 403 `Restaurant context missing`; `GET` → 405. AUTH-D2 applies.

Validation failures (both `pending-kots` and `update-status` use `updateKOTStatusRequest`): 422 `{ "status": false, "message": "<first error>", "errors": { … } }`, e.g. `The status field is required.`, `The status field must be an integer.`

---

## `POST /api/chef/pending-kots`

| Field | Rule |
|---|---|
| `status` | required integer (the stage to list) |

| Situation | HTTP | Body |
|---|---|---|
| No KOT of the restaurant in that stage | 200 | `{ "status": true, "message": "No pending KOTs found", "data": [] }` |
| At least one KOT (KITCH-D1) | 500 | `{ "status": false, "message": "An error occurred while fetching pending kots", "error": "Malformed UTF-8 characters, possibly incorrectly encoded" }` |
| Every listed ID happens to be valid UTF-8 (very rare) | 200 | `{ "status": true, "message": "Pending KOTs fetched successfully", "data": [ …KOTs ] }` — see [data-model.md](./data-model.md#list-row-shape-per-kot) |
| Rows with a status that has no label (e.g. 7) | 500 | `{ "status": false, "message": "Something went wrong", "data": null }` |

The message says "pending" for every stage.

## `POST /api/chef/update-status/{kotId}`

| Field | Rule |
|---|---|
| `{kotId}` (path) | used as is: hex text never matches (KITCH-D1); the 16 raw bytes, percent-encoded, do match |
| `status` | required integer; no range or order check (KITCH-D2) |

| Situation | HTTP | Body / effect |
|---|---|---|
| Status 0–4, row matched or not | 200 | `{ "status": true, "message": "KOT status updated successfully", "data": { "kotStatus": "<label>" } }`; matched row: `status`, `chefId`, stage times, `updated_at` set |
| Status outside 0–4, within −128…127 | 500 | `{ "status": false, "message": "Something went wrong", "data": null }`; a matched row **is** written first |
| Status outside −128…127 | 500 | same body; nothing written |

## `POST /api/chef/dashboard`

No body.

| Situation | HTTP | Body |
|---|---|---|
| OK | 200 | `{ "status": true, "message": "Chef KOT stats fetched successfully", "data": { "total_received": <int>, "total_accepted": <int>, "total_rejected": <int> } }` |
| Error | 500 | `{ "status": false, "message": "Something went wrong", "data": null }` |

Counts as in [data-model.md](./data-model.md#dashboard-counts) (KITCH-D3).
