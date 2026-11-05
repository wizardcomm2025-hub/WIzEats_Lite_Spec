# 06 — Resume or end a session

## Summary

When the app starts, it resumes the stored session without asking for the PIN again, even offline. Sessions never expire on the server (AUTH-D3). If the server ever refuses the session on a normal call (HTTP 401) — because the token was logged out or removed — the app signs out on the device and tells the user why. This keeps a POS tablet ready all day while still recovering cleanly from a dead session.

## Prerequisites

- **Backend:** every protected endpoint behind `auth:sanctum` + `restaurant.tenant`; shared 401 / 403 shapes in [project-overview.md](../../project-overview.md#response-shapes)
- **Data:** `personal_access_tokens` — [data-model.md](./data-model.md)
- **Depends on:** [02](./02-sign-in-with-pin.md), [05](./05-log-out.md); ONB for the approval refresh

## Actors & preconditions

- **Actor:** system.
- **Trigger:** the app starts or comes back to the foreground; or any protected call returns 401.
- **Preconditions:** a token and user summary are stored on the device.

## Main flow

1. On start, the app reads the stored token and user summary.
2. If both are there, the app opens signed in, using the stored role, approval and `mustChangePin` to pick the screen — no PIN, no network needed.
3. Each protected call sends the token.
4. On start and on each return to the foreground, the app re-reads the restaurant's approval, so a newly approved or rejected restaurant is shown without a new sign-in (ONB).
5. If a protected call (not under `/auth/`) returns 401, the app cancels other requests, wipes storage and resets data.
6. The Login phone step shows `Your session has ended. Please log in again.`

## Alternative flows

- **Nothing stored.** The Login screen shows.
- **Token valid, user switched off or restaurant switched off / rejected.** The session keeps working on the server (AUTH-D2). A rejected restaurant is shown the Rejected screen through the approval refresh (ONB); a switched-off user or restaurant keeps using the app.
- **401 from an `/auth/` call.** Treated as that call's own error (for example `Invalid PIN`), not as a dead session.
- **403 `Restaurant context missing`.** Shown as the calling screen's error; the session is not ended.
- **Token of a user row that was removed.** Every protected call returns 403 `Restaurant context missing` (see [data-model.md → Rules](./data-model.md#rules)).
- **Several 401s at once.** The app ends the session once.

## Acceptance criteria

1. Given a signed-in user, when the app is closed and reopened (even offline), then the same screen family opens without asking for the PIN.
2. Given the session was logged out on the server (for example the token was deleted), when the user does anything that loads data, then the Login phone step shows `Your session has ended. Please log in again.`
3. Given (2), then no data of the previous session remains on the device.
4. Given a wrong PIN on Change PIN (401 from `/auth/changePin`), then the user is **not** signed out and sees `Current PIN is incorrect`.
5. Given a user is switched off by the owner while signed in, then that user can keep working until they log out (AUTH-D2).
6. Given a token that has not been used for months, then it still works (AUTH-D3).

## Platform notes

### POS API

No endpoint of its own. 401 body `{ "message": "Unauthenticated." }` from `bootstrap/app.php`; tenant 403 from `RestaurantTenantMiddleware`. No expiry, no status or approval check on protected calls (AUTH-D2, AUTH-D3).

### Data — MySQL

Reads `personal_access_tokens` (Sanctum updates `last_used_at`). Writes nothing else.

### POS app

`appInitializer` / `setInitialstate` restore; `axiosInstance` response interceptor calls the unauthorized handler on 401 for non-`/auth/` URLs; thunk `expireSession` (runs once while signed in); `RootNavigator` re-reads approval on start and foreground.

### Super-admin panel

Not touched.

### Integrations

Not touched.

## Configuration

`sanctum.expiration` (null).

## Out of scope

- Approval refresh screens — ONB.
- Idle auto-lock on the device — not offered.

## Open questions

- None; behaviour fixed by AUTH-D2 and AUTH-D3.
