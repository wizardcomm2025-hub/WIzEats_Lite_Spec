# 01 — Sign in and out — developer cases

Use case: [01-sign-in-and-out.md](../01-sign-in-and-out.md). Contract: [api-contract.md](../api-contract.md#sign-in-and-out). Fixtures: [README.md](./README.md#fixtures).

`POST /admin/login` with `Content-Type: application/json`, `Accept: application/json`, a cookie jar from `GET /`, and `X-CSRF-TOKEN` from its meta tag, unless stated.

| ID | Category | Request / manipulation | Expected |
|---|---|---|---|
| ADM-01-D01 | — (happy path) | `{ "username": "qa.admin", "password": "<ADM-1>", "remember": false }` | 200 `{ "message": "Login successful", "redirect": "<APP_URL>/admin" }`; the `laravel_session` cookie value changes (session regenerated); `super_admins.last_login_at` = now; `GET /admin` with the jar → 200 |
| ADM-01-D02 | Account state / Business rule / Error hygiene (accepted ADM-D1) | (a) wrong password (b) unknown username (c) ADM-OFF with its correct password (d) username `QA.ADMIN` | (a)–(c) 422 `{ "message": "Invalid credentials" }`, with no hint of which part was wrong. (d) 200: the `utf8mb4_unicode_ci` collation compares usernames case-insensitively |
| ADM-01-D03 | Rate limiting (accepted ADM-D1) | 6 wrong attempts in a minute from one IP; then the correct password; then repeat the 6 attempts from a second IP against the same username | Attempt 6 → 429; the correct password within the minute → 429; the second IP gets its own 5 attempts (no account lockout) |
| ADM-01-D04 | Authentication / Error hygiene | (a) `POST /admin/logout` with the CSRF token (b) then `GET /admin` with the same jar (c) `GET /admin/logout` (d) `POST /admin/logout` without the token | (a) 302 → `/` (b) 302 → `/` (c) 405 (d) 419 |
| ADM-01-D05 | Super-admin boundary | (a) `GET /admin` with `Authorization: Bearer <OWN-A API token>` and no cookie (b) `POST /api/reports/dashboard-summary` with only the panel session cookie | (a) 302 → `/` (b) 401 `{ "message": "Unauthenticated." }` |
| ADM-01-D06 | Input validation / Injection / Over-posting | (a) `{}` (b) `{ "username": "' OR 1=1 -- ", "password": "x" }` (c) `{ "username": ["qa.admin"], "password": "x" }` (d) D01 plus `"status": 0, "id": 1` (e) D01 without the CSRF header | (a) 422 `{ "message": "The username field is required. (and 1 more error)", "errors": { username, password } }` (b) 422 `Invalid credentials` (c) 422 `The username field must be a string.` (d) 200 as D01 (extra fields ignored) (e) 419 |
| ADM-01-D07 | Authentication (accepted ADM-D1) | D01 with `"remember": true`; inspect `Set-Cookie` | A `remember_admin_<hash>` cookie with an expiry about 400 days ahead (not 30); after deleting `laravel_session`, `GET /admin` with only the remember cookie → 200 |
| ADM-01-D08 | Account state (accepted ADM-D1) | Sign in as ADM-1 (D07 jar). Then by SQL set ADM-1 `status` = 0 and change its password. `GET /admin`; then drop `laravel_session` and `GET /admin` with the remember cookie | Both 200: the open session and the remember cookie keep working. A new sign-in → 422 `Invalid credentials` |
