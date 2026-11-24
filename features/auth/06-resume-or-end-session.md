# 06 — Resume or end a session

## Summary

When the app starts, it resumes the stored session without asking for the PIN again, even offline. Sessions never expire on the server (AUTH-D3). If the server refuses the session on a normal call — 401 because the token was logged out or removed, or (target, AUTH-D2) 403 because the user or restaurant was switched off or the restaurant is no longer approved — the app reacts: it signs out on the device and tells the user why, or shows the Pending-approval screen. This keeps a POS tablet ready all day while still recovering cleanly from a dead or blocked session.

## Prerequisites

- **Backend:** every protected endpoint behind `auth:sanctum` + `restaurant.tenant`; shared 401 / 403 shapes in [project-overview.md](../../project-overview.md#response-shapes); the X-D2 status checks in [project-overview.md → Known gaps and targets](../../project-overview.md#known-gaps-and-targets) (target)
- **Data:** `personal_access_tokens`, `users.status`, `restaurants.isApproved`, `restaurants.status` — [data-model.md](./data-model.md)
- **Depends on:** [02](./02-sign-in-with-pin.md), [05](./05-log-out.md); ONB for the approval refresh and the Pending-approval / Rejected screens

## Actors & preconditions

- **Actor:** system.
- **Trigger:** the app starts or comes back to the foreground; or any protected call returns 401, or (target) one of the three X-D2 403 messages.
- **Preconditions:** a token and user summary are stored on the device.

## Main flow

1. On start, the app reads the stored token and user summary.
2. If both are there, the app opens signed in, using the stored role, approval and `mustChangePin` to pick the screen — no PIN, no network needed.
3. Each protected call sends the token.
4. On start and on each return to the foreground, the app re-reads the restaurant's approval (`restaurants/showDetails`), so a newly approved or rejected restaurant is shown without a new sign-in (ONB).
5. If a protected call (not under `/auth/`) returns 401, the app cancels other requests, wipes storage and resets data.
6. The Login phone step shows `Your session has ended. Please log in again.`
7. *(Target, AUTH-D2.)* If a protected call returns 403 `Your account is inactive.` or `Your restaurant is inactive.`, the app does the same as step 5, and the Login phone step shows that server message in red.
8. *(Target, AUTH-D2.)* If a protected call returns 403 `Your restaurant is not approved.`, the app keeps the session and opens the Pending-approval screen (ONB), which keeps polling `restaurants/showDetails` (exempt from the approval check).

## Alternative flows

- **Nothing stored.** The Login screen shows.
- **User switched off while signed in.** *Today:* the session keeps working on the server; the user keeps using the app until logout. *Target (AUTH-D2):* the next protected call → 403 `Your account is inactive.` → signed out (step 7).
- **Restaurant switched off while signed in.** *Today:* keeps working. *Target (AUTH-D2):* 403 `Your restaurant is inactive.` → signed out (step 7).
- **Restaurant rejected while signed in.** *Today:* calls keep working; the approval refresh on start / foreground shows the Rejected screen (ONB). *Target (AUTH-D2):* any protected call other than `restaurants/showDetails` → 403 `Your restaurant is not approved.` → Pending-approval screen (step 8), which then shows Rejected once the refresh reads `isApproved` (ONB).
- **401 from an `/auth/` call.** Treated as that call's own error (for example `Current PIN is incorrect`), not as a dead session.
- **403 `Restaurant context missing`.** Shown as the calling screen's error; the session is not ended.
- **Token of a user row that was removed.** Every protected call returns 403 `Restaurant context missing` (see [data-model.md → Rules](./data-model.md#rules)).
- **Several 401s / 403s at once.** The app ends the session (or opens Pending-approval) once.
- **Logout while blocked.** Logout always succeeds (exempt from X-D2) — [05](./05-log-out.md).

## Acceptance criteria

1. Given a signed-in user, when the app is closed and reopened (even offline), then the same screen family opens without asking for the PIN.
2. Given the session was logged out on the server (for example the token was deleted), when the user does anything that loads data, then the Login phone step shows `Your session has ended. Please log in again.`
3. Given (2), then no data of the previous session remains on the device.
4. Given a wrong PIN on Change PIN (401 from `/auth/changePin`), then the user is **not** signed out and sees `Current PIN is incorrect`.
5. *(Target, AUTH-D2 — replaces "keeps working until logout".)* Given a user is switched off by the owner while signed in, when that user does anything that loads data, then the Login phone step shows `Your account is inactive.` and no data of the session remains.
6. Given a token that has not been used for months, then it still works (AUTH-D3).
7. *(Target, AUTH-D2.)* Given the restaurant is switched off by a super admin while a user is signed in, when the user does anything that loads data, then the Login phone step shows `Your restaurant is inactive.`
8. *(Target, AUTH-D2.)* Given the restaurant's approval is withdrawn (pending or rejected) while a user is signed in, when the user does anything that loads data, then the Pending-approval / Rejected screen shows (ONB) and the user is still signed in.

## Platform notes

### POS API

No endpoint of its own. 401 body `{ "message": "Unauthenticated." }` from `bootstrap/app.php`; tenant 403 from `RestaurantTenantMiddleware`. No expiry (AUTH-D3). *Today:* no status or approval check on protected calls. *Target (AUTH-D2 / X-D2):* the middleware checks `users.status`, `restaurants.isApproved`, `restaurants.status` in that order on every protected call; `auth/logout` is exempt from all three, `restaurants/showDetails` from the approval check.

### Data — MySQL

Reads `personal_access_tokens` (Sanctum updates `last_used_at`); in the target also `users.status` and the restaurant row. Writes nothing else.

### POS app

`appInitializer` / `setInitialstate` restore; `axiosInstance` response interceptor calls the unauthorized handler on 401 for non-`/auth/` URLs; thunk `expireSession` (runs once while signed in); `RootNavigator` re-reads approval on start and foreground (`refreshApprovalStatus`). *Target (AUTH-D2):* the interceptor also handles the three X-D2 403 messages — the two "inactive" messages go through `expireSession` with the server message shown on Login; "not approved" sets the stored approval to Pending and lets `RootNavigator` show Pending-approval. See [ui-ux.md → Feedback states](./ui-ux.md#4-feedback-states).

### Super-admin panel

Not touched. (Approving, rejecting and switching restaurants off happens there — ONB / ADM.)

### Integrations

Not touched.

## Configuration

`sanctum.expiration` (null).

## Out of scope

- Approval refresh screens — ONB.
- Idle auto-lock on the device — not offered.

## Open questions

- None; behaviour fixed by AUTH-D2 (target, decided 2026-11-24) and AUTH-D3. _Previously: "behaviour fixed by AUTH-D2 and AUTH-D3" with D2 accepted (2026-11-05)._
