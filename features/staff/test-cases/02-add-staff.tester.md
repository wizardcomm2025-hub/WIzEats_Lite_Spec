# 02 — Add a staff member — tester cases

Use case: [02-add-staff.md](../02-add-staff.md). Screen text: [ui-ux.md §2](../ui-ux.md#2-add-staff-sheet). Fixtures: [README.md](./README.md#fixtures).

Unless stated: signed in as OWN-S (no staff) with User Management open. Delete the users created after each case.

| ID | Covers | Steps | Expected |
|---|---|---|---|
| STAFF-02-T01 | AC1 | `Add Staff`; Name `Test Waiter`, Phone PH-NEW, Role `Waiter`; `Create Staff`. | Spinner; toast `Staff created successfully`; the sheet closes; the list shows `Test Waiter`, PH-NEW, `Waiter`, `Active`. |
| STAFF-02-T02 | AC2 (accepted risk STAFF-D4) | After T01, on DEV-2: Login with PH-NEW, PIN `123456`. | `Set a new PIN` screen; after setting one, the waiter Home. Signing in again needs the new PIN. |
| STAFF-02-T03 | AC3 | After T01, tap `Add Staff`. | `Waiter` chip greyed with `Already added` and can't be picked; `Chef` can. |
| STAFF-02-T04 | AC4 | After T01, add a chef the same way. | Toast; both listed; `Add Staff` greyed; line `You have 1 active chef and 1 active waiter. Disable one to add another.` |
| STAFF-02-T05 | AC5 | Open the sheet; tap `Create Staff` with everything empty. Then Phone `5123456789`; then `98765`. | Empty: `Name is required`, `Phone number is required`, `Please select a role` in red. `5123456789` and `98765`: `Enter a valid 10-digit number starting with 6-9`. Nothing created. |
| STAFF-02-T06 | AC6 (accepted risk STAFF-D3) | Name `Dup`, Phone PH-USED, `Waiter`; `Create Staff`. | Toast `Failed to create staff member`; the sheet stays open with the values; no new row. |
| STAFF-02-T07 | Alt: spaces | Name of three spaces, valid phone, `Chef`; create. | Toast `The name field is required.`; nothing created. |
| STAFF-02-T08 | Alt: no network | Valid values; airplane mode on; create. | Toast `Failed to create staff`; values kept. |
| STAFF-02-T09 | Alt: double tap | Slow network; tap `Create Staff` several times quickly. | One spinner; one toast; one new row. |
| STAFF-02-T10 | Alt: close | Type a name and phone; close the sheet; open it again. | Fields empty, no role picked. |
| STAFF-02-T11 | AC2 (accepted risk STAFF-D4) | Create a waiter with PH-NEW2. Before the waiter signs in, a second tester signs in on DEV-2 with PH-NEW2 and `123456` and sets PIN `111111`. Then the waiter tries `123456`. | The second tester gets in. The waiter's `123456` is refused (wrong PIN — AUTH 02). |
| STAFF-02-T12 | Encoding | Name `রাহুল 🍳`; create. | Shown exactly in the list and in the panel. |
