# 05 — Log out — developer cases

Use case: [05-log-out.md](../05-log-out.md). Contract: [api-contract.md](../api-contract.md#post-apiauthlogout). Fixtures and standard headers: [README.md](./README.md).

All requests: `POST /api/auth/logout`, no body unless stated. Probe = `POST /api/restaurants/showDetails`.

| ID | Category | Caller | Request / manipulation | Expected | Database after |
|---|---|---|---|---|---|
| AUTH-05-D01 | — (happy path) | OWN-A token T1 | — | 200 · `{ "status": true, "message": "Logged out successfully" }`. Then T1 on the probe → 401 `Unauthenticated.`; T1 on logout again → 401. | T1's `personal_access_tokens` row deleted |
| AUTH-05-D02 | Business-rule bypass | OWN-A tokens T1 and T2 (two sign-ins) | log out with T1 | 200 · T2 on the probe → 200 | only T1 deleted |
| AUTH-05-D03 | Authentication | (a) none (b) `Bearer abc` (c) already logged-out token | — | 401 · `{ "message": "Unauthenticated." }` | no token row deleted |
| AUTH-05-D04 | Account state | NOREST token | — | 403 · `{ "status": false, "message": "Restaurant context missing" }` | NOREST's token **not** deleted |
| AUTH-05-D05 | Over-posting | OWN-A token T1 | body `{ "tokenId": "<id of OWN-B's token>", "userId": "<OWN-B id>", "all": true }` | 200 | only T1 deleted; OWN-B's token and OWN-A's other tokens remain |
| AUTH-05-D06 | Restaurant isolation | OWN-A token | — | 200 | OWN-B's and WTR-A's tokens untouched |
| AUTH-05-D07 | Concurrency | OWN-A token T1 | two logouts with T1 in parallel | each is 200 or 401; never 500 | T1 deleted; nothing else |
| AUTH-05-D08 | Error hygiene | OWN-A token | logout while the database is unreachable | 500 · nothing written. The body may include exception text (an `error` field, or Laravel's error output) — accepted risk (AUTH-D5). | unchanged |
