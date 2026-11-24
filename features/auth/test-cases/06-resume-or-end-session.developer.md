# 06 — Resume or end a session — developer cases

Use case: [06-resume-or-end-session.md](../06-resume-or-end-session.md). Shared shapes: [project-overview.md → Response shapes](../../../project-overview.md#response-shapes). Fixtures and standard headers: [README.md](./README.md).

Probe = `POST /api/restaurants/showDetails`. Probe-2 = `POST /api/tables/getTablesDetails` (any protected endpoint other than `showDetails` and `logout` behaves the same). Some cases change rows directly in the database to simulate time or admin actions — restore them afterwards.

| ID | Category | Caller | Manipulation | Expected | Database after |
|---|---|---|---|---|---|
| AUTH-06-D01 | Authentication (expiry; accepted AUTH-D3) | OWN-A token | set the token row's `created_at` and `last_used_at` to 400 days ago | probe → 200 (tokens never expire) | `last_used_at` updated to now |
| AUTH-06-D02 | Account state — *Fails until AUTH-D2 is fixed* | WTR-A token | set WTR-A `status = 0` | probe and probe-2 → 403 · `{ "status": false, "message": "Your account is inactive." }` (today: 200); a **new** loginPin → 403 `Your account is inactive.` | token unchanged; nothing written by the probes |
| AUTH-06-D03 | Account state — *Fails until AUTH-D2 is fixed* | OWN-A token | set REST-A `isApproved = 2`; then `isApproved = 0` | probe → 200 with `isApproved` 2 (then 0) in the data — `showDetails` is exempt from the approval check · probe-2 → 403 · `{ "status": false, "message": "Your restaurant is not approved." }` (today: 200) | token unchanged |
| AUTH-06-D04 | Account state — *Fails until AUTH-D2 is fixed* | OWN-A token | set REST-A `status = 0` | probe and probe-2 → 403 · `{ "status": false, "message": "Your restaurant is inactive." }` (today: 200) | token unchanged |
| AUTH-06-D05 | Account state | token of a throw-away user | delete that user's `users` row, keep the token row | probe → 403 · `{ "status": false, "message": "Restaurant context missing" }` (not 401) | token row remains |
| AUTH-06-D06 | Authentication | OWN-A token | (a) change one character of the secret (b) OWN-A's token id with OWN-B's secret (c) `Bearer` missing, raw token only (d) empty `Bearer ` (e) the secret alone, without `<id>\|` | (a)–(d): probe → 401 · `{ "message": "Unauthenticated." }` · (e): 200 — Sanctum also finds a token by the hash of the secret alone, so the `<id>\|` part is optional | unchanged |
| AUTH-06-D07 | Authentication | OWN-A token | send it as `?token=…` in the URL, or as body field `token`, with no `Authorization` header | probe → 401 `Unauthenticated.` | unchanged |
| AUTH-06-D08 | Restaurant isolation / Over-posting | OWN-A token | probe body `{ "restaurantId": "<REST-B id>" }` and header `X-Restaurant-Id: <REST-B id>` | 200 with **REST-A's** details; nothing of REST-B | unchanged |
| AUTH-06-D09 | Account state — order of checks — *Fails until AUTH-D2 is fixed* | WTR-A token | set WTR-A `status = 0` **and** REST-A `isApproved = 0` **and** REST-A `status = 0` | probe-2 → 403 `Your account is inactive.` (user checked first); restore WTR-A → `Your restaurant is not approved.`; restore approval → `Your restaurant is inactive.` | unchanged |
| AUTH-06-D10 | Business-rule bypass — *Fails until AUTH-D2 is fixed* | WTR-A token, WTR-A `status = 0` | a write call, e.g. `POST /api/tables/ZoneSave` with a valid new zone | 403 `Your account is inactive.` | **no** `zones` row created |
