# 06 — Resume or end a session — developer cases

Use case: [06-resume-or-end-session.md](../06-resume-or-end-session.md). Shared shapes: [project-overview.md → Response shapes](../../../project-overview.md#response-shapes). Fixtures and standard headers: [README.md](./README.md).

Probe = `POST /api/restaurants/showDetails` (any protected endpoint behaves the same). Some cases change rows directly in the database to simulate time or admin actions — restore them afterwards.

| ID | Category | Caller | Manipulation | Expected | Database after |
|---|---|---|---|---|---|
| AUTH-06-D01 | Authentication (expiry) | OWN-A token | set the token row's `created_at` and `last_used_at` to 400 days ago | probe → 200 (tokens never expire — AUTH-D3) | `last_used_at` updated to now |
| AUTH-06-D02 | Account state | WTR-A token | set WTR-A `status = 0` | probe → 200 (AUTH-D2); a **new** loginPin → 403 `Your account is inactive.` | token unchanged |
| AUTH-06-D03 | Account state | OWN-A token | set REST-A `isApproved = 2` | probe → 200 with `isApproved` 2 in the data (AUTH-D2; the app shows Rejected via the approval refresh — ONB) | token unchanged |
| AUTH-06-D04 | Account state | OWN-A token | set REST-A `status = 0` | probe → 200 (AUTH-D2) | token unchanged |
| AUTH-06-D05 | Account state | token of a throw-away user | delete that user's `users` row, keep the token row | probe → 403 · `{ "status": false, "message": "Restaurant context missing" }` (not 401) | token row remains |
| AUTH-06-D06 | Authentication | OWN-A token | (a) change one character of the secret (b) OWN-A's token id with OWN-B's secret (c) `Bearer` missing, raw token only (d) empty `Bearer ` (e) the secret alone, without `<id>\|` | (a)–(d): probe → 401 · `{ "message": "Unauthenticated." }` · (e): 200 — Sanctum also finds a token by the hash of the secret alone, so the `<id>\|` part is optional | unchanged |
| AUTH-06-D07 | Authentication | OWN-A token | send it as `?token=…` in the URL, or as body field `token`, with no `Authorization` header | probe → 401 `Unauthenticated.` | unchanged |
| AUTH-06-D08 | Restaurant isolation / Over-posting | OWN-A token | probe body `{ "restaurantId": "<REST-B id>" }` and header `X-Restaurant-Id: <REST-B id>` | 200 with **REST-A's** details; nothing of REST-B | unchanged |
