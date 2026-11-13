# 01 — Sign in and out — tester cases

Use case: [01-sign-in-and-out.md](../01-sign-in-and-out.md). Screen text: [ui-ux.md §1](../ui-ux.md#1-sign-in-page-). Fixtures: [README.md](./README.md#fixtures).

Fresh browser profile (no panel cookies) unless stated; open `APP_URL`.

| ID | Covers | Steps | Expected |
|---|---|---|---|
| ADM-01-T01 | AC1 | Enter ADM-1's username and password; tap `Sign In`. | The button reads `Signing in...`, then `Success!` in green; the `Restaurant Management` page opens at `/admin`. |
| ADM-01-T02 | AC2 | Enter ADM-1's username with a wrong password; `Sign In`. | Red box `Invalid credentials`; the form shakes; the button returns to `Sign In`. Typing in a field hides the box. |
| ADM-01-T03 | AC3 (ADM-D1) | Sign in as ADM-OFF with its correct password. | `Invalid credentials`. |
| ADM-01-T04 | AC4 | Submit a wrong password six times within one minute. | Attempts 1–5: `Invalid credentials`. Attempt 6: `Too many attempts. Please try again in a minute.` After a minute, the correct password works. |
| ADM-01-T05 | AC5 | Sign in (T01). Tap `Logout`. Then press the browser's Back button and reload. | The sign-in page opens. After Back + reload you are on the sign-in page again, not the restaurant list. |
| ADM-01-T06 | Alt: guest | Without signing in, type `APP_URL/admin`, then `APP_URL/admin/analytics/<REST-A hex>`, then `APP_URL/dashboard`. | Each time the sign-in page opens. |
| ADM-01-T07 | Alt: empty fields | Tap `Sign In` with both fields empty; then with only the username. | The empty field(s) get a red border and the form shakes; no error box; no sign-in. |
| ADM-01-T08 | Alt: expired page | Open the sign-in page and leave it for more than 2 hours; then sign in with ADM-1. | `Session expired. Please refresh the page.` After a refresh, sign-in works. |
| ADM-01-T09 | Alt: dead links (ADM-D1, D3) | Click `Forgot password?`, `Privacy Policy`, `Terms of Service`. Then, signed in, open `APP_URL/`. | The links do nothing (the URL gets `#`). Signed in, `/` goes straight to the restaurant list. |
