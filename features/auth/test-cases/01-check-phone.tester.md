# 01 — Check a phone number — tester cases

Use case: [01-check-phone.md](../01-check-phone.md). Screen text: [ui-ux.md](../ui-ux.md). Fixtures: [README.md](./README.md#fixtures).

Run in the POS app on a phone and on a tablet, signed out. No endpoints, request bodies, HTTP codes or SQL.

| ID | Covers | Precondition | Steps | Expected |
|---|---|---|---|---|
| AUTH-01-T01 | AC1 | Login screen shows `Welcome` | 1. Enter OWN-A's 10-digit number. 2. Tap `Continue`. | Title changes to `Enter your PIN`; subtitle `Enter the 6-digit PIN for +91 <number>`; 6 empty PIN boxes; keyboard open. |
| AUTH-01-T02 | AC2 | Login screen | 1. Enter PH-NEW. 2. Tap `Continue`. | The restaurant onboarding screen opens with PH-NEW filled in. |
| AUTH-01-T03 | AC3 | Login screen, field empty | Tap `Continue`. | `Mobile number is required` in red under the field; the screen does not change. |
| AUTH-01-T04 | AC4 | Login screen | 1. Enter 9 digits. 2. Tap `Continue`. | `Enter a valid 10-digit number`; `Continue` is disabled until the number is edited. |
| AUTH-01-T05 | AC5 | Login screen | 1. Enter `5123456789`. 2. Tap `Continue`. | `Please enter a valid mobile number`; stays on the phone step. |
| AUTH-01-T06 | AC6 | WTR-A-OFF exists | 1. Enter WTR-A-OFF's number. 2. Tap `Continue`. | PIN step opens (the refusal comes at sign-in — see AUTH-02-T08). |
| AUTH-01-T07 | AC9 | T05 just failed | Change one digit. | The red message disappears. |
| AUTH-01-T08 | Alt: double tap | Slow network (throttle the device's connection) | Enter OWN-A's number and tap `Continue` several times quickly. | Button shows a spinner and ignores extra taps; the PIN step opens once. |
| AUTH-01-T09 | Alt: no network | Airplane mode on | Enter OWN-A's number, tap `Continue`. | Red message under the field (`Could not check this number` or a network message); stays on the phone step. Turn airplane mode off and tap again → PIN step. |
| AUTH-01-T10 | Field rules | Login screen | Try to type letters, spaces or an 11th digit. | Only digits can be entered (number pad), at most 10; `+91` cannot be edited. |
| AUTH-01-T11 | Legal links | Login screen | Tap `Terms and Conditions`, then `Privacy Policy`. | Each opens the matching page in the browser; returning shows the login screen unchanged. |
| AUTH-01-T12 | Sizes | Tablet, landscape and portrait | Open the login screen. | Card is centred and phone-width; nothing is cut off; keyboard does not hide `Continue`. |
