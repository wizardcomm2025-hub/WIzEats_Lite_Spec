# Super-admin panel — HTTP contract

Web routes (`routes/web.php`), `web` middleware group: session, CSRF, `SetLocale`. Facts from the code on 2026-11-12.

Common rules:
- a guest on any `auth:admin` route gets 302 → `/`;
- a POST without a valid CSRF token gets 419 (`Page Expired`);
- the wrong method gets 405;
- with `APP_DEBUG=false` a server error is the standard 500 page (`Server Error`);
- an API bearer token is ignored here (the session guard only).

## Sign in and out

| Route | Middleware | Request | Responses |
|---|---|---|---|
| `GET /` | — | — | 200 sign-in page; signed in → 302 `/admin` |
| `POST /admin/login` | `throttle:5,1` (per IP) | JSON `{ username, password, remember }` + `X-CSRF-TOKEN` | 200 `{ "message": "Login successful", "redirect": "<APP_URL>/admin" }`; 422 `{ "message": "Invalid credentials" }` (wrong username / password, or `status` 0); 422 validation `{ "message": "The username field is required. (and 1 more error)", "errors": { … } }` (missing fields, with `Accept: application/json`); 429 after 5 attempts in a minute; 419 without CSRF |
| `POST /admin/logout` | — | form + CSRF | 302 → `/`; the session is invalidated and the token regenerated |

## Restaurant list

| Route | Middleware | Query | Response |
|---|---|---|---|
| `GET /admin` | `auth:admin` | `page` (non-number → 1; beyond the last page → an empty table, with the footer `Showing  to  of <n> results`) | 200 HTML |
| `GET /dashboard` | `auth:admin` | — | 200 static HTML (ADM-D3) |

## Restaurant pages

| Route | Query | Response |
|---|---|---|
| `GET /admin/analytics/{restaurantId}` | `filter` = `today` / `yesterday` / `7days` / `15days` / `30days`; other values = all time (ADM-D2) | 200 HTML; an unknown ID → 200 with zeros (ADM-D3) |
| `GET /admin/orders/{restaurantId}` | — | 200 HTML |
| `GET /admin/items/{restaurantId}` | `sort_by` = `menuItemName` / `quantity` / `total` / `last_id` (others → `last_id`); `sort_order` = `asc` / `desc` | 200 HTML; any other `sort_order` → 500 (`InvalidArgumentException: Order direction must be "asc" or "desc".`) |

All three require `auth:admin`.

## Language

| Route | Middleware | Response |
|---|---|---|
| `GET /set-locale/{locale}` | none (no sign-in) | 302 → the `Referer`, or the session's previous URL (or `/`) when there is none; `en`, `es`, `de`, `bn` are stored in the session, anything else is ignored (ADM-D1) |
