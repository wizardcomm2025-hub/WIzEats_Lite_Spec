# 05 — Log out — tester cases

Use case: [05-log-out.md](../05-log-out.md). Screen text: [ui-ux.md](../ui-ux.md). Fixtures: [README.md](./README.md#fixtures).

Logout lives in the side drawer on a phone and in the nav rail on a tablet. Run each case on both.

| ID | Covers | Precondition | Steps | Expected |
|---|---|---|---|---|
| AUTH-05-T01 | AC1 | Signed in as OWN-A | Open the drawer / nav rail, tap Logout, then `Logout` in the dialog. | Dialog `Logout Confirmation` / `Are you sure you want to logout from your account?`; then `Logging out...`; then `Welcome` with an empty number field. |
| AUTH-05-T02 | AC2 | Signed in as OWN-A | Tap Logout, then `Cancel`. | Dialog closes; same screen, still signed in. |
| AUTH-05-T03 | AC3 | T01 done | Close and reopen the app. | Login screen; nothing from OWN-A's session. |
| AUTH-05-T04 | AC5 | OWN-A signed in on DEV-1 and DEV-2 | Log out on DEV-1. Use DEV-2. | DEV-2 keeps working. |
| AUTH-05-T05 | AC6 | Signed in as WTR-A | Airplane mode on; log out. | Still reaches the Login screen. |
| AUTH-05-T06 | AC7 | OWN-A signed in, with a cart or open order in progress | Log out; sign in as OWN-B on the same device. | Nothing of REST-A (menu, tables, cart, orders, restaurant name) is visible to OWN-B. |
| AUTH-05-T07 | Roles | Signed in as CHF-A, then as WTR-A | Find and use Logout. | Available for every role; same dialog and result. |
| AUTH-05-T08 | Double tap | Signed in, slow network | Tap `Logout` in the dialog twice quickly. | One spinner; ends on the Login screen once; no error. |
