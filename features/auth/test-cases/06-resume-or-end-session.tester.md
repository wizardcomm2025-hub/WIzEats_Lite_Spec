# 06 — Resume or end a session — tester cases

Use case: [06-resume-or-end-session.md](../06-resume-or-end-session.md). Screen text: [ui-ux.md](../ui-ux.md). Fixtures: [README.md](./README.md#fixtures).

Some cases need a developer to change data on the server; the step says so.

| ID | Covers | Precondition | Steps | Expected |
|---|---|---|---|---|
| AUTH-06-T01 | AC1 | Signed in as OWN-A | 1. Close the app fully. 2. Airplane mode on. 3. Reopen. | The app opens signed in (Home) without asking for the number or PIN. |
| AUTH-06-T02 | AC2, AC3 | Signed in as WTR-A on DEV-1 | Ask a developer to end DEV-1's session on the server (delete its token). Then open any screen that loads data (for example pull to refresh on Home). | The app returns to the Login screen with `Your session has ended. Please log in again.`; after signing in again, nothing from the old session is left. |
| AUTH-06-T03 | AC4 | Signed in as MCP-A on Change PIN | Enter a wrong current PIN and submit. | `Current PIN is incorrect`; **not** sent back to login. |
| AUTH-06-T04 | AC5 | WTR-A signed in on DEV-1 | Owner switches WTR-A off (Staff screen). Keep using DEV-1; then log out and try to sign in again. | DEV-1 keeps working until logout (AUTH-D2). The new sign-in shows `Your account is inactive.` |
| AUTH-06-T05 | Alt: rejected restaurant | OWN-A signed in | Ask a super admin to reject REST-A. Send the app to the background and bring it back. | The Rejected screen is shown (ONB). Restore REST-A afterwards. |
