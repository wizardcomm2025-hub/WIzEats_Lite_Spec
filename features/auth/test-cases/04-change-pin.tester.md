# 04 — Change PIN — tester cases

Use case: [04-change-pin.md](../04-change-pin.md). Screen text: [ui-ux.md](../ui-ux.md). Fixtures: [README.md](./README.md#fixtures).

Each case starts by signing in as MCP-A (or a user just reset in AUTH-03-T01), which opens `Set a new PIN`. Reset the fixture between cases.

| ID | Covers | Steps | Expected |
|---|---|---|---|
| AUTH-04-T01 | AC1 | Sign in as MCP-A. Look for a menu, back button or header; press the Android back button. | Only the Change PIN card is shown; no way into the app. Back either does nothing or closes the app — it never shows another app screen. |
| AUTH-04-T02 | AC2 | Enter the current PIN, a new PIN, the new PIN again. Tap `Change PIN`. | Home opens (KOT for a chef user). |
| AUTH-04-T03 | AC3 | After T02: log out, sign in with the old PIN, then with the new PIN. | Old: `Invalid PIN`. New: Home opens directly, no Change PIN screen. |
| AUTH-04-T04 | AC4 | Enter a wrong current PIN, a valid new PIN twice. Tap `Change PIN`. | `Current PIN is incorrect`; still on the screen and still signed in (not sent back to login). |
| AUTH-04-T05 | AC5 | Enter current; new `111111`; confirm `222222`. | `New PIN and confirmation do not match` as soon as confirm has 6 digits; `Change PIN` disabled. |
| AUTH-04-T06 | AC6 | Enter current PIN; new = the same PIN. | `New PIN must be different from the current PIN`; `Change PIN` disabled. |
| AUTH-04-T07 | AC7 | Tap `Logout` on this screen. Then sign in again as MCP-A. | Login screen right away (no confirmation dialog). Sign-in shows Change PIN again. |
| AUTH-04-T08 | AC8 | Close the app fully from the recent-apps list; reopen. | `Set a new PIN` shows again. |
| AUTH-04-T09 | Alt: other sessions | MCP-A also signed in on DEV-2 (still on Change PIN). Change the PIN on DEV-1. | DEV-2 is not signed out (AUTH-D4); it still shows Change PIN until changed or logged out there. |
| AUTH-04-T10 | Alt: no network | Fill a valid form; airplane mode on; tap `Change PIN`. | Red message (`Could not change PIN` or a network message); stays on the screen. Airplane off, tap again → app opens. |
| AUTH-04-T11 | Button state | Fill fields one by one. | `Change PIN` stays disabled until all three have 6 digits, confirm matches, and new differs from current. |
| AUTH-04-T12 | Sizes | Tablet portrait and landscape. | All three PIN rows and both buttons reachable; the card scrolls when the keyboard is open. |
