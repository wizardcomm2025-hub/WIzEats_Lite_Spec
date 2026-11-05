# 02 — Sign in with PIN — tester cases

Use case: [02-sign-in-with-pin.md](../02-sign-in-with-pin.md). Screen text: [ui-ux.md](../ui-ux.md). Fixtures: [README.md](./README.md#fixtures).

Each case starts on the PIN step for the named user (enter their number, tap `Continue`). Run on a phone and a tablet.

| ID | Covers | User | Steps | Expected |
|---|---|---|---|---|
| AUTH-02-T01 | AC1 | OWN-A | Enter the correct PIN, tap `Login`. | Home opens with the owner's menu. |
| AUTH-02-T02 | AC1 | WTR-A | Enter the correct PIN, tap `Login`. | Home opens with the waiter's view. |
| AUTH-02-T03 | AC2 | CHF-A | Enter the correct PIN, tap `Login`. | The KOT screen opens. |
| AUTH-02-T04 | AC3 | MCP-A | Enter the correct PIN, tap `Login`. | Only `Set a new PIN` is shown — no drawer, header or other screens. |
| AUTH-02-T05 | AC4 | OWN-P | Enter the correct PIN, tap `Login`. | The Pending-approval screen is shown (wording per ONB). |
| AUTH-02-T06 | AC4 | OWN-R; then OWN-OFF | Enter the correct PIN, tap `Login`. | OWN-R: the Rejected screen (ONB). OWN-OFF: Home opens (a switched-off restaurant is not blocked — AUTH-D2). |
| AUTH-02-T07 | AC5 | OWN-A | Enter a wrong PIN, tap `Login`. | `Invalid PIN` in red; stays on the PIN step; typing a new digit clears the message. |
| AUTH-02-T08 | AC6 | WTR-A-OFF | Enter the correct PIN, tap `Login`. | `Your account is inactive.`; stays on the PIN step. |
| AUTH-02-T09 | AC7 | CHF-A | Within one minute: 5 wrong PINs, then the correct PIN. | 1–5: `Invalid PIN`. 6th: `Too Many Attempts.` After waiting a minute, the correct PIN signs in. |
| AUTH-02-T10 | AC8 | OWN-A | Type 5 digits; then try letters. | `Login` stays disabled at 5 digits; letters don't appear; becomes active at 6 digits. |
| AUTH-02-T11 | AC9 | OWN-A | Type the PIN and watch the boxes. | Each box shows `•`, never the digit. |
| AUTH-02-T12 | AC10 | OWN-A on DEV-1 and DEV-2 | Sign in on DEV-1, then on DEV-2. Use both. | Both work at the same time; DEV-1 is not signed out. |
| AUTH-02-T13 | Alt: change number | OWN-A | On the PIN step, tap `Change Phone Number`. | Back to `Welcome` with an empty number field; any error is gone. |
| AUTH-02-T14 | Alt: no network | OWN-A | Airplane mode on; enter the correct PIN, tap `Login`. | Red message (`Login failed` or a network message); stays on the PIN step. Airplane off, tap `Login` → signs in. |
| AUTH-02-T15 | Alt: double tap | OWN-A, slow network | Enter the PIN; tap `Login` several times quickly. | Spinner; signs in once; no duplicate screens. |
| AUTH-02-T16 | Sizes | OWN-A on tablet | Sign in in portrait, then rotate. | PIN boxes and buttons fully visible; keyboard does not cover `Login`. |
