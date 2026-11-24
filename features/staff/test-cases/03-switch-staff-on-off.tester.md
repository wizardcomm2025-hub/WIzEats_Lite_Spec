# 03 — Switch a staff member on or off — tester cases

Use case: [03-switch-staff-on-off.md](../03-switch-staff-on-off.md). Screen text: [ui-ux.md §1](../ui-ux.md#switch-confirmation). Fixtures: [README.md](./README.md#fixtures).

Unless stated: signed in as OWN-A with User Management open.

| ID | Covers | Steps | Expected |
|---|---|---|---|
| STAFF-03-T01 | AC1 (STAFF-D1 fixed 2026-11-17) | Tap WTR-A's switch. Read the dialog. Tap `Deactivate`. Reopen the screen. Then try to sign in as WTR-A on DEV-2. | Dialog `Deactivate staff member?` / `Are you sure you want to deactivate "<WTR-A name>"?` with a red `Deactivate`. After confirming: a spinner on the row, then the toast `Staff member "<WTR-A name>" deactivated`; the switch is off, `Inactive`, row faded. After reopening: still `Inactive`. WTR-A's sign-in is refused. |
| STAFF-03-T02 | AC6 (STAFF-D1 fixed) | Panel: set CHF-A `Inactive`. App: reopen the screen; tap CHF-A's switch; `Activate`. | Dialog `Activate staff member?` / `Are you sure you want to activate "<CHF-A name>"?`. Toast `Staff member "<CHF-A name>" activated`; CHF-A `Active`; CHF-A can sign in again. |
| STAFF-03-T03 | AC4 | Tap WTR-A's switch; cancel the dialog. | Dialog closes; no spinner, no toast; nothing changes. |
| STAFF-03-T04 | Waiter view | Signed in as WTR-A, open User Management. | No switch on the own row. |
| STAFF-03-T05 | AC2 — *Fails until AUTH-D2 is fixed* | WTR-A signed in on DEV-2. On DEV-1 OWN-A switches WTR-A off (switch + `Deactivate`). On DEV-2 open any screen that loads data (for example pull to refresh on Home); then try to sign in again. | DEV-2 returns to Login with `Your account is inactive.` in red; signing in again is refused. (Today: DEV-2 keeps working until logout; only the new sign-in is refused.) |
| STAFF-03-T06 | AC5 — *Fails until STAFF-D5 is fixed* | REST-A has CHF-A active and CHF-A2 inactive. Tap CHF-A2's switch. Then switch CHF-A off (`Deactivate`) and tap CHF-A2's switch again. | First tap: toast `You can have 1 active chef and 1 active waiter.`, no dialog, CHF-A2 stays `Inactive` (today: the dialog shows and CHF-A2 becomes active — two active chefs). After CHF-A is off: the `Activate` dialog, then `Staff member "<CHF-A2 name>" activated`. |
