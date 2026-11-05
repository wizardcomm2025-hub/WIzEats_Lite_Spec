# Sign-in & PIN

Every restaurant user — owner, waiter or chef — signs in to the POS app with their mobile number and a 6-digit PIN. The app first checks whether the number has an account: a known number goes to the PIN step, a new number goes to restaurant onboarding. Users who forget their PIN get a new one by e-mail and must replace it at the next sign-in. Sessions last until the user logs out. The spec records today's behaviour (decisions AUTH-D1 to D10, several of them accepted risks); AUTH-D11 is the only change to build. An older phone + OTP sign-in and a phone-only register endpoint are still live on the API for old app versions; this spec documents them as they are today.

**Feature code:** `AUTH` (test IDs `AUTH-01-T01`, `AUTH-02-D05`, …)

## Use cases

| # | Use case | Actor | Tester file | Developer file |
|---|---|---|---|---|
| 01 | [Check a phone number](./01-check-phone.md) | Anyone (not signed in) | [tester](./test-cases/01-check-phone.tester.md) | [developer](./test-cases/01-check-phone.developer.md) |
| 02 | [Sign in with PIN](./02-sign-in-with-pin.md) | Owner, waiter, chef | [tester](./test-cases/02-sign-in-with-pin.tester.md) | [developer](./test-cases/02-sign-in-with-pin.developer.md) |
| 03 | [Reset a forgotten PIN](./03-reset-forgotten-pin.md) | Owner, waiter, chef (not signed in) | [tester](./test-cases/03-reset-forgotten-pin.tester.md) | [developer](./test-cases/03-reset-forgotten-pin.developer.md) |
| 04 | [Change PIN](./04-change-pin.md) | Signed-in user | [tester](./test-cases/04-change-pin.tester.md) | [developer](./test-cases/04-change-pin.developer.md) |
| 05 | [Log out](./05-log-out.md) | Signed-in user | [tester](./test-cases/05-log-out.tester.md) | [developer](./test-cases/05-log-out.developer.md) |
| 06 | [Resume or end a session](./06-resume-or-end-session.md) | System (app start, any rejected call) | [tester](./test-cases/06-resume-or-end-session.tester.md) | [developer](./test-cases/06-resume-or-end-session.developer.md) |
| 07 | [Legacy OTP sign-in and register](./07-legacy-otp-and-register.md) | Old app versions / any caller | Tester: n/a — no UI | [developer](./test-cases/07-legacy-otp-and-register.developer.md) |

## Key concepts

See the [glossary](../../project-overview.md#glossary) (User, PIN, OTP, Approval, Restaurant status). Specific to this feature:

- **Session** — one Sanctum personal access token (`personal_access_tokens` row) issued at sign-in. The app keeps it on the device until logout or until a call is refused with 401.
- **Forced PIN change** — `users.mustChangePin = 1`. Set by a PIN reset and by the default PIN of legacy registration. The app shows only the Change PIN screen until it is cleared.
- **Masked e-mail** — the restaurant e-mail shown after a reset: first 2 characters of the name part, then one `*` per remaining character (at least one), then `@domain`. Example: `cafeowner@gmail.com` → `ca*******@gmail.com`; `ab@x.com` → `ab*@x.com`.

## Documents

- [data-model.md](./data-model.md) — `users`, `personal_access_tokens`, `restaurants` (read)
- [api-contract.md](./api-contract.md) — `/api/auth/*`
- [ui-ux.md](./ui-ux.md) — Login, Change PIN, Logout dialog
- [test-cases/README.md](./test-cases/README.md) — fixtures, coverage, endpoint checklist
- [developer-guide.html](./developer-guide.html)
- Implementation: `laravel-pos-api-dev_guid/specs/auth/` (created with the first plan)

## Depends on

- **Onboarding & approval (ONB)** — a new number is handed to onboarding (01), and a signed-in user of a pending or rejected restaurant sees the Pending-approval screen instead of the app (02). Those screens are specified in ONB.
- **Staff (STAFF)** — waiter and chef accounts, their status and their first PIN are created there.

## Decisions

- **AUTH-D1 (2026-11-05):** The legacy endpoints `/api/auth/sendOtp`, `/api/auth/verifyOtp` and `/api/auth/register` stay live and unchanged for now. Accepted risk: `verifyOtp` accepts the fixed OTP `123456` for every phone with no rate limit, so anyone who knows a registered phone number can obtain a token for that account. `register` also returns the name, e-mail, role and restaurant ID of any registered phone to an unauthenticated caller, and creates a restaurant for any new phone without limit. See [07](./07-legacy-otp-and-register.md).
- **AUTH-D2 (2026-11-05):** Switching a user off, or rejecting / switching off a restaurant, only blocks the **next PIN sign-in**. Tokens already issued keep working until the user logs out. Accepted risk.
- **AUTH-D3 (2026-11-05):** Tokens never expire. A session ends only by logout (05) or when the server refuses it (06). Accepted risk: a lost or stolen device stays signed in.
- **AUTH-D4 (2026-11-05):** Changing or resetting a PIN does **not** sign the user out on other devices.
- **AUTH-D5 (2026-11-05, from AUTH-Q1):** Unexpected-error (500) responses stay as they are, including the exception text in `error`. Accepted risk. Error-hygiene cases check only the 500 status and that nothing is written.
- **AUTH-D6 (2026-11-05, from AUTH-Q2):** Forgot PIN stays as is: anyone who knows the number can trigger it, the old PIN stops working at once, and it works for switched-off users. Accepted risk.
- **AUTH-D7 (2026-11-05, from AUTH-Q3):** `changePin` stays without a rate limit. Accepted risk.
- **AUTH-D8 (2026-11-05, from AUTH-Q4):** Revealing whether a phone is registered is accepted; the existing rate limits are enough.
- **AUTH-D9 (2026-11-05, from AUTH-Q5):** No "Change PIN" entry in Profile for now. Future.
- **AUTH-D10 (2026-11-05, from AUTH-Q6):** Keep both envelopes: `success` on PIN / OTP endpoints, `status` on `register` and `logout`.
- **AUTH-D11 (2026-11-05, from AUTH-Q7):** **Behaviour change.** Legacy `register` must never leave a restaurant without an owner. Two parallel calls with the same new phone create exactly one restaurant and one owner; the other call gets 200 `type: "verify"` with that same user. This is the only change to the legacy endpoints (otherwise AUTH-D1 holds). `AUTH-07-D18` fails until it is built.

## Cross-cutting open questions

- **AUTH-Q1.** Unexpected server errors on every AUTH endpoint return the exception text in an `error` field (and in `message` for none). Should AUTH responses drop the `error` field, so no SQL, file paths or class names reach the caller? Developer "Error hygiene" cases are `Blocked` on this. _Resolved 2026-11-05: keep as is — AUTH-D5._
- **AUTH-Q2.** Anyone who knows a registered phone number can trigger Forgot PIN for it (3 per 10 minutes). Each reset replaces the user's PIN at once, so the real user can't sign in until they read the restaurant's e-mail. It also works for switched-off users. Accept as is, or require something more (for example, keep the old PIN valid until the new one is used)? _Resolved 2026-11-05: keep as is — AUTH-D6._
- **AUTH-Q3.** `changePin` has no rate limit, so a holder of a token can try current PINs without limit. Add a limit like `loginPin`'s? _Resolved 2026-11-05: no limit — AUTH-D7._
- **AUTH-Q4.** `checkPhone` (and the 404 of `loginPin` / `forgotPin`) tells any caller whether a phone number is registered. The app needs this to route new numbers to onboarding. Accept? _Resolved 2026-11-05: accepted, rate limits are enough — AUTH-D8._
- **AUTH-Q5.** The app has no way to change a PIN by choice; the Change PIN screen appears only when it is forced. Is a "Change PIN" entry in Profile wanted? (The API already allows it — see [04](./04-change-pin.md).) _Resolved 2026-11-05: not now — AUTH-D9._
- **AUTH-Q6.** PIN endpoints answer with `success`, every other endpoint with `status`. Keep, or align to `status`? (Changing it is breaking for the app.) _Resolved 2026-11-05: keep both — AUTH-D10._
- **AUTH-Q7.** Two parallel `register` calls with the same new phone: the restaurant row is created before the owner, so the losing call can fail on the unique phone and leave a restaurant with no users. What is the expected result? `AUTH-07-D18` is `Blocked` on this. _Resolved 2026-11-05: exactly one restaurant and owner, no leftovers; the other call gets `type: "verify"` — AUTH-D11._

## Feature-wide out of scope

- Restaurant onboarding, the Pending-approval and Rejected screens — ONB.
- Creating staff and setting their first PIN — STAFF.
- Super-admin panel sign-in (`/admin/login`) — ADM.
- Public QR ordering (no sign-in) — PUB.
