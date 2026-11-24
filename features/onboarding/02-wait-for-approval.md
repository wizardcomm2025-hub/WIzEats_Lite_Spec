# 02 — Wait for approval

## Summary

Until a super admin accepts a restaurant, anyone from it who signs in sees a "Verification in Progress" screen instead of the app, and a rejected restaurant sees "Account Access Denied". The app re-reads the approval on its own and on request, and lets the user in as soon as it is accepted. Today this screen is the only thing that holds back an unapproved restaurant. Target (AUTH-D2 / X-D2): the API also refuses every call of an unapproved restaurant except this screen's check and logout.

## Prerequisites

- **Backend:** `POST /api/restaurants/showDetails` (RST) — returns `isApproved`
- **Data:** `restaurants.isApproved`, `restaurants.isOnboardingDone`
- **Depends on:** [AUTH 02](../auth/02-sign-in-with-pin.md), [03](./03-approve-or-reject.md)

## Actors & preconditions

- **Actor:** owner, waiter or chef of a restaurant that is not Accepted.
- **Trigger:** signs in, or reopens the app while signed in.
- **Preconditions:** signed in; `isApproved` is Pending or Rejected.

## Main flow

1. After sign-in, the app sees approval `Pending` and shows `Verification\nin Progress`.
2. The app asks the server for the restaurant's approval when the screen opens, when the app starts and each time it returns to the foreground.
3. The user taps `Check Approval Status` (or pulls down); the app asks again and shows `Checked just now — still under review.`
4. When a super admin accepts the restaurant ([03](./03-approve-or-reject.md)), the next check returns Accepted.
5. The app leaves this screen by itself: to Change PIN if a change is due (always for a new owner), otherwise to Home or KOT.

## Alternative flows

- **Rejected.** The screen shows `Account Access\nDenied` and the decline text; no check button; only `LOGOUT FROM SYSTEM` (and `Contact Support` if configured).
- **Rejected while waiting.** The next check switches the screen to the Rejected version.
- **No network.** `Could not reach the server. Check the connection and try again.`
- **Onboarding not done (legacy account).** The check never runs; the button always shows `Could not reach the server. Check the connection and try again.` and the restaurant never appears in the panel (ONB-D4).
- **Logout.** `LOGOUT FROM SYSTEM` logs out at once ([AUTH 05](../auth/05-log-out.md)).
- **Direct API use.** Today the session token works on every endpoint while Pending or Rejected. *Target (AUTH-D2):* every protected call except `restaurants/showDetails` and `auth/logout` → 403 `Your restaurant is not approved.`; nothing is written.
- **Restaurant or user switched off while waiting.** Today the check still says `Checked just now — still under review.` *Target (AUTH-D2):* the check gets 403 `Your restaurant is inactive.` / `Your account is inactive.` and the app signs out to Login with that message in red ([AUTH 06](../auth/06-resume-or-end-session.md)).
- **Accepted, then later rejected.** Only possible by a crafted panel request (ONB-D2); the next check shows the Rejected screen.

## Acceptance criteria

1. Given a Pending restaurant, when its owner signs in, then `Verification\nin Progress` shows with `Check Approval Status` and `LOGOUT FROM SYSTEM`, and nothing else of the app is reachable.
2. Given a Pending restaurant, when `Check Approval Status` is tapped, then `Checked just now — still under review.` shows.
3. Given the restaurant is accepted while the owner waits, when they tap `Check Approval Status` (or bring the app to the foreground), then the app moves on without signing in again.
4. Given a Rejected restaurant, when anyone from it signs in, then `Account Access\nDenied` shows with the decline text and no check button.
5. Given no network, when `Check Approval Status` is tapped, then `Could not reach the server. Check the connection and try again.` shows.
6. Given `LOGOUT FROM SYSTEM`, then Login shows immediately, with no confirmation.
7. Waiters and chefs of a Pending restaurant see the same screen as the owner.
8. No support number configured → no `Contact Support` button.
9. *(Target, AUTH-D2)* Given a Pending or Rejected restaurant, when its token calls any protected endpoint other than `showDetails` or `logout`, then 403 `Your restaurant is not approved.` and nothing is written; `showDetails` still answers 200 with the approval.
10. *(Target, AUTH-D2)* Given REST-P is switched off while its owner is on this screen, when the approval is checked, then the app shows Login with `Your restaurant is inactive.` in red.

## Platform notes

### POS API

No endpoint of its own; uses `showDetails` (RST). Today no API endpoint checks approval. Target (X-D2): the tenant middleware checks it on every call; `showDetails` is exempt from the approval check only, `logout` from all checks.

### Data — MySQL

Reads `restaurants.isApproved`.

### POS app

`PendingApprovalScreen`; thunk `refreshApprovalStatus` (runs only when signed in and `isOnboardingDone` is true); `RootNavigator` decides the screen. See [ui-ux.md §4](./ui-ux.md#4-pending-approval).

### Super-admin panel

Decision made there — [03](./03-approve-or-reject.md).

### Integrations

`Contact Support` opens WhatsApp, else the phone dialler, for `SUPPORT_PHONE`.

## Configuration

`SUPPORT_PHONE` in the app's `constants.ts` (empty today → button hidden).

## Out of scope

- Building the API approval check — owned by the X-D2 work ([project-overview.md](../../project-overview.md#known-gaps-and-targets)); this use case only states how the screen behaves with it.

## Open questions

- None open. Related decisions: ONB-D2, ONB-D4, AUTH-D2 (To fix).
