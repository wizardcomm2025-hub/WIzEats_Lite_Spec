# 01 — Register a new restaurant — tester cases

Use case: [01-register-restaurant.md](../01-register-restaurant.md). Screen text: [ui-ux.md](../ui-ux.md). Fixtures: [README.md](./README.md#fixtures).

Each case starts signed out: on Login enter a fresh PH-NEW number and tap `Continue`, which opens Restaurant Profile. Run on a phone and a tablet.

| ID | Covers | Steps | Expected |
|---|---|---|---|
| ONB-01-T01 | AC1, AC2, AC10 | Fill PROFILE-OK, switch `Takeaway Service` on, tap `Save & Continue`. | Spinner; `Success!` / `Restaurant submitted for approval`; after about 3 seconds `Waiting for\nApproval` with `… We have emailed your login PIN to <your e-mail>. Once you are approved, log in with your mobile number and that PIN.` The restaurant inbox gets `Your login PIN` with a 6-digit PIN and the line `Your restaurant is pending approval.` |
| ONB-01-T02 | AC5 | Leave one field empty (try each of the four in turn). | `Save & Continue` stays grey and can't be tapped. |
| ONB-01-T03 | AC5 | Type in `Restaurant Name`, then delete it; repeat for each field. | `Restaurant Name is required` / `Owner Name is required` / `Email Address is required` / `Business Address is required` under the field, which turns red. |
| ONB-01-T04 | AC6 | Enter e-mail `owner@cafe.co`; then `owner@cafe`; then `owner@cafe.in`. | First two: `Email must end with .com, .net, .org, .in` and the button stays disabled. `.in`: message goes away. |
| ONB-01-T05 | AC5 | Enter only spaces in `Owner Name`, fill the rest. | Button stays disabled. |
| ONB-01-T06 | AC7 | On DEV-1 open Restaurant Profile for PH-NEW. On DEV-2 complete registration with the same number. Back on DEV-1, fill the form and submit. | Alert `Error` / `This phone number is already registered. Please log in.`; tap `OK`; stays on the form. |
| ONB-01-T07 | Alt: long name | Paste a 300-character restaurant name, fill the rest, submit. | Alert `Error` / `Failed to onboard restaurant`. Logging in with the number later: it is still unknown (goes to onboarding again). |
| ONB-01-T08 | Alt: no network | Fill PROFILE-OK, airplane mode on, submit. | Alert `Error` / `Failed to setup restaurant`; the fields keep their values. Airplane off, submit again → success. |
| ONB-01-T09 | AC9 | Slow network; tap `Save & Continue` several times quickly. | One spinner; one success; only one PIN e-mail. |
| ONB-01-T10 | Alt: back | On Restaurant Profile press back. | Login screen; the number has no account (entering it again opens Restaurant Profile). |
| ONB-01-T11 | Registration Submitted | After T01, tap `Go to Login`; then try swipe-back / Android back on Registration Submitted (repeat T01 for this). | `Go to Login` opens an empty Login. Swipe-back does nothing; Android back leaves the app. |
| ONB-01-T12 | While saving | Slow network; while the spinner shows, try to edit a field. | Fields can't be edited. |
| ONB-01-T13 | AC4 | After T01 (not yet approved), sign in with the number and the e-mailed PIN. | `Verification\nin Progress` screen ([02](./02-wait-for-approval.tester.md)). |
| ONB-01-T14 | AC10 | Register with `Takeaway Service` off; after approval (ONB-03-T01) and PIN change, open Profile, then Home. | Profile: `Enable Takeaway` is off. Home still shows the `TAKEAWAY` tile — the switch is only stored (accepted risk [RST-D1](../../restaurant-settings/README.md#decisions)). |
| ONB-01-T15 | Encoding | Register with name `রেস্তোরাঁ 🍛 Café`. | Success; the PIN e-mail and the panel list show the name exactly. |
| ONB-01-T16 | Sizes | Tablet portrait and landscape; open the keyboard in `Business Address`. | Form centred, max width; `Save & Continue` stays visible above the keyboard. |
