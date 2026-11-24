# 01 — See the staff list — tester cases

Use case: [01-see-staff-list.md](../01-see-staff-list.md). Screen text: [ui-ux.md §1](../ui-ux.md#1-user-management-screen). Fixtures: [README.md](./README.md#fixtures).

"Open User Management" means: on a phone, menu → `Account` → `User Management`; on a tablet, Setup (owner) or the rail's users button (waiter / chef).

| ID | Covers | Precondition | Steps | Expected |
|---|---|---|---|---|
| STAFF-01-T01 | AC1 | Signed in as OWN-A on DEVICE-TAB | Open User Management. | Header `USER MANAGEMENT` / `Rest A` / `Manage your staff members`. Card `Staff Members` lists WTR-A and CHF-A, each with name, phone, role chip (`Waiter` / `Chef`), switch on and `Active`. |
| STAFF-01-T02 | AC1 | T01 | Look for OWN-A in the list. | The owner is not listed. |
| STAFF-01-T03 | AC2 | Signed in as WTR-A on DEVICE-PHONE | Open User Management. | Card `Your Profile` with only WTR-A's row; no switch, no `Add Staff`, no limit line. |
| STAFF-01-T04 | AC2 | Signed in as CHF-A on DEVICE-TAB | Tap the users button in the rail. | `Your Profile` with only CHF-A's row. |
| STAFF-01-T05 | AC3 | T01 (one active waiter, one active chef) | Look at the card. | `Add Staff` is greyed and can't be tapped; line `You have 1 active chef and 1 active waiter. Disable one to add another.` |
| STAFF-01-T06 | AC4 | Signed in as OWN-S | Open User Management. | `No users added yet`; `Add Staff` active; line `You can have 1 active chef and 1 active waiter.` |
| STAFF-01-T07 | Alt: role changed | A developer sets WTR-A's role to `manager` in the database (the panel refuses it once STAFF-D6 is fixed) | As OWN-A, reopen User Management. | Only CHF-A is listed; the `Waiter` slot is free (`Add Staff` active). |
| STAFF-01-T08 | Alt: inactive | Panel: set CHF-A to `Inactive` | As OWN-A, reopen User Management. | CHF-A's row is faded, switch off, `Inactive`; `Add Staff` active. |
