# Onboarding & approval — UI / UX

Text below is **verbatim** from the app and the panel (2026-11-06); *(server)* marks an API message shown as is. `\n` = line break.

## 1. Screens and entry points

| Screen | Shown when | Who |
|---|---|---|
| Restaurant Profile (`RestaurantProfileScreen`) | A new number is entered on Login ([AUTH 01](../auth/01-check-phone.md)) | New owner, signed out |
| Registration Submitted (`RegistrationSubmittedScreen`) | After a successful registration | New owner, signed out |
| Pending approval (`PendingApprovalScreen`) | Signed in and the restaurant is not Accepted | Any role of a Pending / Rejected restaurant |
| Panel — restaurant list (`admin/index`) | Super admin signed in to the web panel | Super admin |

## 2. Restaurant Profile

| Element | Text / behaviour |
|---|---|
| Header | `Onboarding`; steps `Phone` (ticked) and `Profile` (2) |
| Title / subtitle | `Restaurant Profile` / `Setup your business profile for receipts & KOTs.` |
| Field | `Restaurant Name *` — placeholder `e.g. Arsalan Restaurant` |
| Field | `Owner Name *` — placeholder `e.g. Anjan Dutta` |
| Field | `Email Address *` — placeholder `e.g. owner@restaurant.com`; e-mail keyboard; no auto-capitalisation |
| Field | `Business Address *` — placeholder `Street, City, Zip Code`; 3 lines |
| Section | `OPERATIONS` with one switch: `Takeaway Service` — `Enable takeaway / pickup orders` (off by default) |
| Button | `Save & Continue` — grey and disabled until all four fields are filled and the e-mail matches the rule; spinner while saving |
| While saving | Fields can't be edited |

Not shown, sent with fixed values: active restaurant = on, KOT = on, delivery = off, room service = off.

Field messages (as the user types, then clears or mistypes):

| Condition | Message under the field |
|---|---|
| Field emptied | `Restaurant Name is required` / `Owner Name is required` / `Email Address is required` / `Business Address is required` |
| E-mail not `name@domain.com/.net/.org/.in` | `Email must end with .com, .net, .org, .in` |

A field holding only spaces counts as empty: `Save & Continue` stays disabled. (The screen also has submit-time alerts titled `Validation Error` — `Please enter restaurant name` and so on — but they can't be reached, because the button is disabled in exactly those cases.)

Server failure: alert `Error` with *(server message)* — for example `This phone number is already registered. Please log in.` — or `Failed to setup restaurant`; button `OK`.

Success: modal `Success!` / `Restaurant submitted for approval`, closes by itself after 3 seconds (or by its close button), then Registration Submitted opens. Back from Restaurant Profile returns to Login.

## 3. Registration Submitted

| Element | Text |
|---|---|
| Title | `Waiting for\nApproval` |
| Body | `Thank you for registering! Your restaurant is under review. We have emailed your login PIN to <e-mail>. Once you are approved, log in with your mobile number and that PIN.` |
| Button | `Go to Login` — opens Login with an empty history |
| Back | Swipe-back disabled; Android back leaves the app |

## 4. Pending approval

| Element | Pending | Rejected |
|---|---|---|
| Icon | Clock (blue) | Cross (red) |
| Title | `Verification\nin Progress` | `Account Access\nDenied` |
| Body | `Thank you for registering! Your restaurant profile is currently under review. We typically approve accounts within 24–48 hours.` | `Your restaurant account has been declined by the administration. For further details or to appeal this decision, please reach out to our support team.` |
| `Check Approval Status` | Shown; spinner while checking | Hidden |
| Result line | `Checked just now — still under review.` or `Could not reach the server. Check the connection and try again.` | — |
| `Contact Support` | Only if a support number is configured (none today) — opens WhatsApp, else a phone call | same |
| `LOGOUT FROM SYSTEM` | Logs out at once, no confirmation | same (dark button) |
| Footer | `WhizEats Lite v1.0` | same |
| Pull down | Same as `Check Approval Status` | — |

The app also re-checks approval when the screen opens, when the app starts and each time it comes back to the foreground. When the server says Accepted, the app moves on by itself: Change PIN if a PIN change is due, otherwise Home / KOT ([AUTH 02](../auth/02-sign-in-with-pin.md)). When it says Rejected, the screen switches to the Rejected version.

## 5. Panel — restaurant list (approval part)

| Column | Content |
|---|---|
| Restaurant Details | Name and `ID: <32-char hex>`; click to expand details |
| Owner / Contact / Location | Owner name, contact phone, address |
| Status | `Active` / `Inactive` (from `restaurants.status`) |
| Approved | Pending: buttons `Accept` (green) and `Reject` (red). Accepted: badge `Approved`. Rejected: badge `Rejected`. |

Only restaurants with onboarding done are listed, newest first, 10 per page. Accept / Reject submit at once (no confirmation) and reload the list. No success or error message is shown (the server sets a `Status updated successfully` flash, but the page doesn't display it).

## 6. E-mails

| E-mail | To | Subject | Body (essentials) |
|---|---|---|---|
| Onboarding PIN | Restaurant e-mail | `Your login PIN` | `Thank you for registering <name>. Your login PIN is:` the PIN · `Log in with your mobile number and this PIN. You will be asked to set a new PIN after logging in.` · `Your restaurant is pending approval. We will email you once it has been reviewed.` |
| Admin notice | `ADMIN_EMAIL` | `Restaurant Onboarding Completed` | `Restaurant <name> has completed onboarding successfully.` |
| Approved | Restaurant e-mail | `Your restaurant has been approved` | `Good news! <name> has been approved. You can now log in to the app with your mobile number and PIN.` |
| Rejected | Restaurant e-mail | `Your restaurant registration was not approved` | `Unfortunately, the registration for <name> was not approved. Please contact support for more details.` |

Greeting: `Hi <owner name>,` (or `Hi there,`).

## 7. Sizes

Restaurant Profile content is capped at 640 px and centred; the `Save & Continue` bar stays above the keyboard. Registration Submitted is capped at 460 px. Pending approval scrolls on small phones.
