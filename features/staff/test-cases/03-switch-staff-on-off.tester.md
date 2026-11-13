# 03 — Switch a staff member on or off — tester cases

Use case: [03-switch-staff-on-off.md](../03-switch-staff-on-off.md). Screen text: [ui-ux.md §1](../ui-ux.md#switch-confirmation). Fixtures: [README.md](./README.md#fixtures).

Unless stated: signed in as OWN-A with User Management open.

| ID | Covers | Steps | Expected |
|---|---|---|---|
| STAFF-03-T01 | AC1 (accepted STAFF-D1) | Tap WTR-A's switch. Read the dialog. Tap `Deactivate`. Reopen the screen. | Dialog `Deactivate staff member?` / `Are you sure you want to deactivate "<WTR-A name>"?` with a red `Deactivate`. After confirming: a spinner on the row, then the toast `The status field is required.`; the switch stays on, `Active`. After reopening: still `Active`. WTR-A can still sign in. |
| STAFF-03-T02 | AC1 (accepted STAFF-D1) | Panel: set CHF-A `Inactive`. App: reopen the screen; tap CHF-A's switch; `Activate`. | Dialog `Activate staff member?`. Toast `The status field is required.`; CHF-A stays `Inactive`. |
| STAFF-03-T03 | AC4 | Tap WTR-A's switch; cancel the dialog. | Dialog closes; no spinner, no toast; nothing changes. |
| STAFF-03-T04 | Waiter view | Signed in as WTR-A, open User Management. | No switch on the own row. |
| STAFF-03-T05 | AC2 (AUTH-D2) | WTR-A signed in on DEV-2. Panel: set WTR-A `Inactive` (the only working way). On DEV-2 keep using the app (open orders); then log out and sign in again. | Before logging out: the app keeps working. Signing in again is refused with the switched-off message (AUTH 02). |
