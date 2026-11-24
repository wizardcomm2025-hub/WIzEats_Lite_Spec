# 06 — Resume or end a session — tester cases

Use case: [06-resume-or-end-session.md](../06-resume-or-end-session.md). Screen text: [ui-ux.md](../ui-ux.md). Fixtures: [README.md](./README.md#fixtures).

Some cases need a developer or super admin to change data on the server; the step says so.

| ID | Covers | Precondition | Steps | Expected |
|---|---|---|---|---|
| AUTH-06-T01 | AC1 | Signed in as OWN-A | 1. Close the app fully. 2. Airplane mode on. 3. Reopen. | The app opens signed in (Home) without asking for the number or PIN. |
| AUTH-06-T02 | AC2, AC3 | Signed in as WTR-A on DEV-1 | Ask a developer to end DEV-1's session on the server (delete its token). Then open any screen that loads data (for example pull to refresh on Home). | The app returns to the Login screen with `Your session has ended. Please log in again.`; after signing in again, nothing from the old session is left. |
| AUTH-06-T03 | AC4 | Signed in as MCP-A on Change PIN | Enter a wrong current PIN and submit. | `Current PIN is incorrect`; **not** sent back to login. |
| AUTH-06-T04 | AC5 — *Fails until AUTH-D2 is fixed* | WTR-A signed in on DEV-1 | Owner switches WTR-A off (Staff screen). On DEV-1, open any screen that loads data. Then try to sign in again as WTR-A. | DEV-1 returns to Login with `Your account is inactive.` in red; nothing of the session remains. The new sign-in shows `Your account is inactive.` (Today: DEV-1 keeps working until logout.) |
| AUTH-06-T05 | Alt: rejected restaurant | OWN-A signed in | Ask a super admin to reject REST-A. Send the app to the background and bring it back. | The Rejected screen is shown (ONB). Restore REST-A afterwards. |
| AUTH-06-T06 | AC7 — *Fails until AUTH-D2 is fixed* | WTR-A signed in on DEV-1 | Ask a super admin to switch REST-A off. On DEV-1, pull to refresh on Home. | Login screen with `Your restaurant is inactive.` in red. Restore REST-A afterwards. (Today: keeps working.) |
| AUTH-06-T07 | AC8 — *Fails until AUTH-D2 is fixed* | OWN-A signed in, on Home (app kept in the foreground) | Ask a super admin to set REST-A back to Pending. Without leaving the app, pull to refresh on Home. | The Pending-approval screen replaces Home; OWN-A is still signed in (no Login screen). After the super admin approves again, the app returns to Home. (Today: Home keeps working until the next start / foreground.) |
