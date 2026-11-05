# 03 — Reset a forgotten PIN — tester cases

Use case: [03-reset-forgotten-pin.md](../03-reset-forgotten-pin.md). Screen text: [ui-ux.md](../ui-ux.md). Fixtures: [README.md](./README.md#fixtures).

Testers need access to the restaurant e-mail inbox (MAIL-OK). Each case starts on the PIN step for the named user.

| ID | Covers | User | Steps | Expected |
|---|---|---|---|---|
| AUTH-03-T01 | AC1, AC7, AC8 | WTR-A | Tap `Forgot PIN?`. | Green text `A new PIN has been sent to <masked e-mail>` — first 2 letters of REST-A's e-mail, stars, then `@` and the full domain. The PIN boxes are cleared. REST-A's inbox gets one e-mail with a 6-digit PIN. The PIN is shown nowhere in the app. |
| AUTH-03-T02 | AC2, AC3 | WTR-A after T01 | 1. Enter the **old** PIN, tap `Login`. 2. Enter the e-mailed PIN, tap `Login`. | 1: `Invalid PIN`. 2: `Set a new PIN` screen opens. |
| AUTH-03-T03 | AC4 | OWN-NM | Tap `Forgot PIN?`. Then sign in with the old PIN. | `No email is registered for this account. Please contact support.`; the old PIN still signs in. |
| AUTH-03-T04 | AC5 | OWN-A, mail transport MAIL-DOWN (ask a developer) | Tap `Forgot PIN?`. Then sign in with the old PIN. | `Could not send the PIN email. Please try again later.`; the old PIN still signs in. |
| AUTH-03-T05 | AC6 | CHF-A | Tap `Forgot PIN?` 4 times within 10 minutes. | Taps 1–3: the green message each time and 3 e-mails. Tap 4: `Too Many Attempts.` The PIN from the **3rd** e-mail signs in. |
| AUTH-03-T06 | Alt: switched-off user | WTR-A-OFF | Tap `Forgot PIN?`, then sign in with the e-mailed PIN. | Green message and an e-mail; sign-in then shows `Your account is inactive.` |
| AUTH-03-T07 | Alt: other sessions | OWN-A signed in on DEV-1; PIN step on DEV-2 | On DEV-2 tap `Forgot PIN?`. Keep using DEV-1. | DEV-1 stays signed in and keeps working (AUTH-D4). |
| AUTH-03-T08 | Alt: no network | OWN-A, airplane mode | Tap `Forgot PIN?`. | Red message (`Could not send a new PIN` or a network message); no e-mail; old PIN still works once online. |
| AUTH-03-T09 | Alt: double tap | OWN-A, slow network | Tap `Forgot PIN?` several times quickly. | Link is disabled while sending; one e-mail. |
