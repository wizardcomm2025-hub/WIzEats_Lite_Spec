# 02 — Wait for approval — tester cases

Use case: [02-wait-for-approval.md](../02-wait-for-approval.md). Screen text: [ui-ux.md §4](../ui-ux.md#4-pending-approval). Fixtures: [README.md](./README.md#fixtures).

Approval changes are made by a second tester in the super-admin panel ([03](./03-approve-or-reject.tester.md)). Reset fixtures after each case.

| ID | Covers | Precondition | Steps | Expected |
|---|---|---|---|---|
| ONB-02-T01 | AC1 | REST-P Pending | Sign in as OWN-P. | `Verification\nin Progress` with the review text, `Check Approval Status`, `LOGOUT FROM SYSTEM`, footer `WhizEats Lite v1.0`. No menu, header or other screen can be reached. |
| ONB-02-T02 | AC2 | T01 | Tap `Check Approval Status`. | Spinner, then `Checked just now — still under review.` |
| ONB-02-T03 | AC2 | T01 | Pull the screen down. | Same as T02. |
| ONB-02-T04 | AC3 | T01, OWN-P has never changed the PIN | Panel: Accept REST-P. App: tap `Check Approval Status`. | The app leaves the screen by itself and shows `Set a new PIN`; after changing, Home. |
| ONB-02-T05 | AC3 | T01 | Send the app to the background. Panel: Accept REST-P. Bring the app back. | The app moves on without a tap. |
| ONB-02-T06 | AC4 | REST-R Rejected | Sign in as OWN-R. | `Account Access\nDenied` with the decline text; red cross; no `Check Approval Status`; dark `LOGOUT FROM SYSTEM`. |
| ONB-02-T07 | AC4 | T01 | Panel: Reject REST-P. App: tap `Check Approval Status`. | Screen switches to `Account Access\nDenied`. |
| ONB-02-T08 | AC5 | T01 | Airplane mode on; tap `Check Approval Status`. | `Could not reach the server. Check the connection and try again.` |
| ONB-02-T09 | AC6 | T01 | Tap `LOGOUT FROM SYSTEM`. | Login screen right away, no confirmation dialog. |
| ONB-02-T10 | AC7 | REST-P has a waiter (created by a developer) | Sign in as that waiter. | Same Pending screen as the owner. |
| ONB-02-T11 | AC8 | T01 | Look for `Contact Support`. | Not shown (no support number configured). |
| ONB-02-T12 | Alt: legacy owner (ONB-D4) | OWN-LEG | Sign in; tap `Check Approval Status`. | Pending screen; the check always says `Could not reach the server. Check the connection and try again.`, even with a good connection. |
| ONB-02-T13 | AC10 — *Fails until AUTH-D2 is fixed* | T01 | Ask a super admin to switch REST-P off. App: tap `Check Approval Status`. | Login screen with `Your restaurant is inactive.` in red; nothing of the session remains. (Today: `Checked just now — still under review.`) Restore REST-P afterwards. |
