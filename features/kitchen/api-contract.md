# Kitchen (chef) — API contract

Shared shapes: [project-overview.md → API conventions](../../project-overview.md#api-conventions). Facts from the code on 2026-11-11, re-checked 2026-11-24 (API commit d54d6c8 — no change). Rows marked **Target** describe agreed fixes (KITCH-D1 to KITCH-D3) and are not in the code yet.

| Endpoint | Middleware | Rate limit | Roles — target (X-D1) | Use case |
|---|---|---|---|---|
| `POST /api/chef/pending-kots` | `auth:sanctum`, `restaurant.tenant` | none | chef | [01](./01-see-kots-by-stage.md) |
| `POST /api/chef/update-status/{kotId}` | same | none | chef | [02](./02-move-kot-through-stages.md) |
| `POST /api/chef/dashboard` | same | none | chef | [03](./03-chef-dashboard.md) |

Today there is no role check (KITCH-D2). **Target:** others → 403 `{ "status": false, "message": "You do not have permission to perform this action", "data": null }`. The common failures: no or invalid token → 401 `{ "message": "Unauthenticated." }`; no `restaurantId` → 403 `Restaurant context missing`; `GET` → 405. Today AUTH-D2 applies; **target (X-D2)** 403 `Your account is inactive.` / `Your restaurant is not approved.` / `Your restaurant is inactive.`

Validation failures (both `pending-kots` and `update-status` use `updateKOTStatusRequest`): 422 `{ "status": false, "message": "<first error>", "errors": { … } }`, e.g. `The status field is required.`, `The status field must be an integer.` **Target:** also `The selected status is invalid.` outside 0–4.

---

## `POST /api/chef/pending-kots`

| Field | Rule |
|---|---|
| `status` | required integer (the stage to list); target 0–4 |

| Situation | HTTP | Body |
|---|---|---|
| No KOT of the restaurant in that stage | 200 | `{ "status": true, "message": "No pending KOTs found", "data": [] }` |
| At least one KOT (KITCH-D1) | 500 | `{ "status": false, "message": "An error occurred while fetching pending kots", "error": "Malformed UTF-8 characters, possibly incorrectly encoded" }`. **Target:** 200 as the next row |
| **Target** KOTs found | 200 | `{ "status": true, "message": "Pending KOTs fetched successfully", "data": [ { kotId: "<hex>", kotNumber, orderNumber, kotNote, status: "<label>", tableName, zoneName, roomName, floorName, created_at, items: [ { menuItemName, quantity } ] } ] }` — newest first; no binary IDs |
| Rows with a status that has no label (e.g. 7) | 500 | `{ "status": false, "message": "Something went wrong", "data": null }` (target: such rows can't exist; the request is 422) |
| **Target** unexpected error | 500 | `{ "status": false, "message": "Something went wrong", "data": null }` — no `error` key |

The message says "pending" for every stage.

## `POST /api/chef/update-status/{kotId}`

| Field | Rule |
|---|---|
| `{kotId}` (path) | today used as is: hex text never matches (KITCH-D1). **Target:** 32-char hex, dashes allowed, a KOT of the caller's restaurant |
| `status` | required integer; today no range or order check (KITCH-D2). **Target:** 0–4 and an allowed change |

| Situation | HTTP | Body / effect |
|---|---|---|
| Status 0–4, row matched or not | 200 | `{ "status": true, "message": "KOT status updated successfully", "data": { "kotStatus": "<label>" } }`; matched row: `status`, `chefId`, stage times, `updated_at` set. **Target:** only for a found KOT and an allowed change |
| Status outside 0–4, within −128…127 | 500 | `{ "status": false, "message": "Something went wrong", "data": null }`; a matched row **is** written first. **Target:** 422, nothing written |
| Status outside −128…127 | 500 | same body; nothing written. **Target:** 422 |
| **Target (KITCH-D1)** unknown / other restaurant's / malformed `kotId` | 404 | `{ "status": false, "message": "KOT not found", "data": null }` |
| **Target (KITCH-D2)** change not in Pending→Preparing, Pending→Rejected, Preparing→Completed, Completed→Ready | 409 | `{ "status": false, "message": "Invalid status change", "data": null }`; nothing written |

**Target effect of an allowed change:** `status`, `updated_at`; the new stage's time (`preparing_at`, `rejected_at` or `completed_at` — Ready has none) set to now, earlier times kept; `chefId` = the caller on Pending→Preparing and Pending→Rejected only.

## `POST /api/chef/dashboard`

No body.

| Situation | HTTP | Body |
|---|---|---|
| OK | 200 | `{ "status": true, "message": "Chef KOT stats fetched successfully", "data": { "total_received": <int>, "total_accepted": <int>, "total_rejected": <int> } }` |
| Error | 500 | `{ "status": false, "message": "Something went wrong", "data": null }` |

Counts: today by `chefId` (KITCH-D3); **target** by restaurant — see [data-model.md](./data-model.md#dashboard-counts).
