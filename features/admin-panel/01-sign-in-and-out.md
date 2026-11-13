# 01 — Sign in and out

## Summary

A super admin opens the panel root `/` and signs in with a username and password. They land on the restaurant list, and later sign out with `Logout`. Guests who open any panel page are sent back to the sign-in page.

## Prerequisites

- **Backend:** `GET /`, `POST /admin/login` (throttle 5 / min), `POST /admin/logout`; see [api-contract.md](./api-contract.md#sign-in-and-out)
- **Data:** `super_admins`, `sessions`

## Actors & preconditions

- **Actor:** a super admin with `status` 1. The account was created by SQL (ADM-D1).

## Main flow

1. `/` shows `Admin Panel` / `Please sign in to continue`, with `Username`, `Password` (eye toggle), `Remember me for 30 days` and `Sign In`.
2. The admin fills both fields and taps `Sign In`. The button reads `Signing in...` with a spinner.
3. The page posts JSON `{ username, password, remember }` with the CSRF token. The server checks the username and password, and `status` = 1.
4. On success:
   - the server regenerates the session and stores `last_login_at`;
   - it answers 200 `{ "message": "Login successful", "redirect": "<APP_URL>/admin" }`;
   - the button turns green and reads `Success!`, and half a second later the restaurant list opens.
5. `Logout` (on the list, and in the header of every restaurant page) ends the session and returns to `/`.

## Alternative flows

- **Empty field:** its border turns red and the form shakes. Nothing is sent.
- **Wrong username or password, or the account switched off:** the red box `Invalid credentials` shows and the form shakes.
- **6th attempt within a minute from the same IP:** `Too many attempts. Please try again in a minute.`
- **CSRF token expired** (sign-in page left open longer than the session lifetime, 120 min): `Session expired. Please refresh the page.`
- **Already signed in, opening `/`:** redirected to `/admin`.
- **Guest opening `/admin…` or `/dashboard`:** redirected to `/`.
- **Remember me ticked:** a remember cookie is set that lasts about 400 days, not 30 (ADM-D1).
- **Admin switched off or password changed while signed in:** the open session keeps working (ADM-D1).
- **`Forgot password?`, `Privacy Policy`, `Terms of Service`:** links to `#`; nothing happens (ADM-D1, ADM-D3).

## Acceptance criteria

1. Given ADM-1, when they sign in, then the restaurant list `Restaurant Management` opens and `super_admins.last_login_at` is now.
2. Given a wrong password, then `Invalid credentials` shows and no session is created.
3. Given ADM-OFF's correct password, then `Invalid credentials`.
4. Given six wrong attempts in one minute, then the sixth shows `Too many attempts. Please try again in a minute.`
5. Given a signed-in admin, when they tap `Logout`, then the sign-in page opens, and `/admin` afterwards redirects to `/`.

## Platform notes

### Super-admin panel

`AdminAuthController` (`showLogin`, `login`, `logout`), `resources/views/login.blade.php`. The guard is `admin` (session, provider `super_admins`). Guests are redirected by `redirectGuestsTo` in `bootstrap/app.php`.

### Data — MySQL

`super_admins.last_login_at`, `remember_token`; the `sessions` table (database session driver).

### POS app, POS API

None. The panel session is not accepted by the API, and API tokens are not accepted by the panel.

## Configuration

`SESSION_LIFETIME` (120), `SESSION_DRIVER` (`database`), `APP_URL`.

## Out of scope

- Creating or resetting super admins (SQL only).

## Open questions

- None open.
