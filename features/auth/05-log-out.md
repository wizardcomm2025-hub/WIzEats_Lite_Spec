# 05 — Log out

## Summary

A signed-in user logs out from the drawer (phone) or the nav rail (tablet) after confirming. The server ends that device's session and the app wipes everything it stored, so the next person on a shared tablet starts from a clean Login screen. Logout works on the device even when the server can't be reached.

## Prerequisites

- **Backend:** `POST /api/auth/logout` — [api-contract.md](./api-contract.md#post-apiauthlogout)
- **Data:** `personal_access_tokens` — [data-model.md](./data-model.md)
- **Depends on:** [02](./02-sign-in-with-pin.md)

## Actors & preconditions

- **Actor:** signed-in owner, waiter or chef.
- **Trigger:** taps Logout in the drawer / nav rail; or `Logout` on Change PIN or Pending-approval (no confirmation there).
- **Preconditions:** signed in.

## Main flow

1. The app asks `Are you sure you want to logout from your account?`
2. The user taps `Logout`; the app shows `Logging out...`.
3. The app stops sending any other request.
4. The app sends logout with the session token.
5. The server deletes that token.
6. The app removes every stored key and resets all screens' data.
7. The Login phone step shows.

## Alternative flows

- **Cancel.** The dialog closes; nothing changes.
- **No network / server error.** The device signs out anyway; the server token stays valid (AUTH-D3).
- **Token already invalid.** Server answers 401; the device signs out anyway.
- **Other devices.** Stay signed in — logout ends only this device's session.
- **Requests in flight.** Any request other than logout is cancelled while signing out.

## Acceptance criteria

1. Given a signed-in user, when Logout is confirmed, then the Login phone step shows with an empty number field.
2. Given Cancel is tapped, then the user stays on the same screen, still signed in.
3. Given a logout, when the app is reopened, then the Login screen shows (no data from the previous user).
4. Given a logout, then the token used no longer works on the server.
5. Given the user is signed in on two devices and logs out on one, then the other keeps working.
6. Given airplane mode, when Logout is confirmed, then the device still returns to Login.
7. Given a different user signs in on the same device after a logout, then no restaurant data of the previous user is shown.

## Platform notes

### POS API

`POST /api/auth/logout`, `auth:sanctum` + `restaurant.tenant`. Deletes the current token only. Envelope `status`. Messages in [api-contract.md](./api-contract.md#post-apiauthlogout).

### Data — MySQL

Deletes one `personal_access_tokens` row.

### POS app

`useLogout` / `LogoutDialogs`, thunk `logoutUser`: `cancelAllRequests()`, call API, then `AsyncStorage.multiRemove(ALL_STORAGE_KEYS)` and `RESET_APP_STATE` regardless of the API result. See [ui-ux.md §3](./ui-ux.md#3-bottom-sheets-and-dialogs).

### Super-admin panel

Not touched.

### Integrations

Not touched.

## Configuration

None.

## Out of scope

- "Log out of all devices" — not offered (AUTH-D4).
- Owner signing staff out remotely — not offered (AUTH-D2).

## Open questions

- None.
