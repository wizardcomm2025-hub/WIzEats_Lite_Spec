# Sign-in & PIN — UI / UX

App conventions are in `POS_APP-master/CLAUDE.md`. Text below is **verbatim** from the app (2026-11-05) or, where marked *(server)*, the API message the app shows as is. Tester cases quote it.

## 1. Screens and entry points

| Screen | Shown when | Roles |
|---|---|---|
| Login — phone step (`Login`, `PhoneLoginScreen`) | Not signed in | Anyone |
| Login — PIN step (same screen) | The number entered has an account | Anyone |
| Change PIN (`ChangePinScreen`) | Signed in, restaurant **Accepted**, `mustChangePin` true. Replaces the whole app until done. | Owner, waiter, chef |
| Logout confirmation (`LogoutDialogs`) | Logout tapped in the drawer (phone) or nav rail (tablet) | Signed-in users |

Routing after sign-in (`RootNavigator`):

| State | Goes to |
|---|---|
| Restaurant not Accepted (Pending / Rejected) | Pending-approval screen — ONB |
| Accepted and `mustChangePin` | Change PIN |
| Accepted, PIN fine, role chef | KOT screen |
| Accepted, PIN fine, owner / waiter | Home |

A new number on the phone step goes to the Restaurant profile (onboarding) screen — ONB.

## 2. Layout and fields

### Login — phone step

| Element | Text / behaviour |
|---|---|
| Title | `Welcome` |
| Subtitle | `Enter your number to manage your restaurant` |
| Label | `Mobile Number` |
| Prefix | `+91` (fixed, not editable) |
| Input | Placeholder `00000 00000`; number pad; max 10 characters |
| Button | `Continue` — disabled while a field error shows or while loading |
| Footer | `By continuing, you agree to our Terms and Conditions & Privacy Policy` — both links open in the browser |

Client-side validation (on Continue):

| Input | Message |
|---|---|
| Empty | `Mobile number is required` |
| Not exactly 10 digits | `Enter a valid 10-digit number` |
| 10 digits starting 0–5 | *(server)* `Please enter a valid mobile number` |

### Login — PIN step

| Element | Text / behaviour |
|---|---|
| Title | `Enter your PIN` |
| Subtitle | `Enter the 6-digit PIN for +91 <phone>` |
| PIN boxes | 6 boxes; digits show as `•`; non-digits are dropped; keyboard opens automatically |
| Button | `Login` — disabled until 6 digits are entered, and while loading |
| Link | `Forgot PIN?` — disabled while loading |
| Link | `Change Phone Number` — back to an empty phone step |

### Change PIN

| Element | Text / behaviour |
|---|---|
| Title | `Set a new PIN` |
| Subtitle | `Replace the PIN we emailed you with one only you know.` |
| Fields | `Current PIN`, `New PIN`, `Confirm new PIN` — 6 boxes each |
| Button | `Change PIN` — disabled until all three have 6 digits, confirm equals new, and new differs from current |
| Link | `Logout` — logs out without a confirmation dialog |

Inline messages (checked as the user types):

| Condition | Message |
|---|---|
| Confirm has 6 digits and differs from New | `New PIN and confirmation do not match` |
| New has 6 digits and equals Current | `New PIN must be different from the current PIN` |

## 3. Bottom sheets and dialogs

### Logout confirmation

| Element | Text |
|---|---|
| Title | `Logout Confirmation` |
| Body | `Are you sure you want to logout from your account?` |
| Buttons | `Cancel` · `Logout` |
| While signing out | Full-screen spinner with `Logging out...` |

## 4. Feedback states

| Where | State | What the user sees |
|---|---|---|
| Phone step | Loading | `Continue` shows a spinner and is disabled |
| Phone step | Server / network error | Red text under the field: *(server message)*, else `Could not check this number` |
| Phone step | Session ended by the server | Red text: `Your session has ended. Please log in again.` |
| PIN step | Loading | `Login` shows a spinner |
| PIN step | Wrong PIN | *(server)* `Invalid PIN` |
| PIN step | Switched-off user | *(server)* `Your account is inactive.` |
| PIN step | Too many tries | *(server)* `Too Many Attempts.` |
| PIN step | Other failure / no network | *(server message)*, else `Login failed` |
| PIN step | Reset sent | Green text: *(server)* `A new PIN has been sent to <masked e-mail>` |
| PIN step | Reset failed | *(server)* `No email is registered for this account. Please contact support.` / `Could not send the PIN email. Please try again later.` / `Too Many Attempts.`, else `Could not send a new PIN` |
| Change PIN | Wrong current PIN | *(server)* `Current PIN is incorrect` |
| Change PIN | Other failure / no network | *(server message)*, else `Could not change PIN` |
| Change PIN | Success | No message; the app opens (Home, or KOT for chefs) |
| Error text | Cleared | When the user edits the phone, edits the PIN, or taps `Change Phone Number` |

## 5. Confirmations

Logout from the drawer / nav rail asks for confirmation (section 3). Logout from Change PIN does not.

## 6. Role-based visibility

All AUTH screens look the same for every role. Only the landing screen after sign-in differs (chef → KOT).

## 7. Sizes

Login and Change PIN cards are capped at 460 px wide and centred, so they stay phone-sized on tablets. Logout dialogs work in portrait and landscape.

## 8. Printing / sharing

None.

## 9. Shared components reused

`AppBackground`, `Input`, `PinInput`, `ErrorMessage`, `SubmitButton`, `LogoutDialogs`.

## 10. Super-admin panel pages

Not covered — the panel's own sign-in belongs to ADM.

## Device storage (for testers' awareness)

The token and the user summary stay on the device until logout or a 401 from the server. Logout clears every stored key, so the next person on a shared tablet starts clean — including when the server can't be reached.
