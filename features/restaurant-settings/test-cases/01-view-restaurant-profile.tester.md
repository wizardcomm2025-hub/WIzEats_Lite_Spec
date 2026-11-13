# 01 — View the restaurant profile — tester cases

Use case: [01-view-restaurant-profile.md](../01-view-restaurant-profile.md). Screen text: [ui-ux.md §1](../ui-ux.md#1-profile-screen). Fixtures: [README.md](./README.md#fixtures).

Reset REST-A to REST-A-BASE before each case. "Open Profile" means: on a phone, menu → `Account` → `Profile`; on a tablet, `Profile` at the bottom of the rail.

| ID | Covers | Precondition | Steps | Expected |
|---|---|---|---|---|
| RST-01-T01 | AC1 (and finding F1) | Signed in as OWN-A on DEVICE-TAB | Open Profile. | Header `PROFILE MANAGEMENT` / `Rest A` / `Manage your restaurant information`. Fields show REST-A-BASE: name, address, owner, Role `owner`, Primary Contact = REST-A's phone, e-mail, Tax Rate `5.00`, QR ID `resta@upi`. Enable Takeaway, Enable Discount and Show KOT Number on Bill are on; KOT Name on Bill `KOT`. `Save Changes` is grey. **If the form stays empty or shows a loader, record it and see F1.** |
| RST-01-T02 | AC2 | REST-FRESH; signed in as OWN-FRESH | Open Profile. | Name, owner, address, e-mail as registered; Primary Contact = the registration phone; Tax Rate `0`; QR ID empty; Enable Takeaway off; Enable Discount off; Show KOT Number on Bill off and no `KOT Name on Bill` field. |
| RST-01-T03 | AC3 | T01 | Tap `Role` and `Primary Contact`; try to type. | Both greyed; no keyboard; values unchanged. |
| RST-01-T04 | AC4 (accepted risk RST-D3) | Signed in as WTR-A on DEVICE-PHONE | Open the menu. | `Account` section shows `Profile`; it opens with REST-A's values and Role `waiter`. Repeat as CHF-A: Role `chef`. |
| RST-01-T05 | AC4 | Signed in as WTR-A, then CHF-A, on DEVICE-TAB | Look at the rail. | No `Profile` button for either. |
| RST-01-T06 | AC5 | T01 on the device; a second tester in the panel | Panel: set REST-A's Tax % to `8`, `Save Configuration`. App: send to background, bring back, go Home, then open Profile. | Tax Rate shows `8.00`. |
| RST-01-T07 | Finding F2 | A developer sets REST-A's name to NAME-HEX in the database | Signed in as OWN-A, open Profile. | Header and Restaurant Name show `436166c3a92044c3a96c696365203132`, not `Café Délice 12`. The panel shows `Café Délice 12`. |
| RST-01-T08 | AC6 | Signed in as OWN-A on DEVICE-PHONE | Panel: tick `Enable Room Service`, save. App: background → foreground → Home; open the menu; open Profile. | Menu shows `Floor Management` and `Room Management`. Profile shows the `Modules unlocked` box; there is no Room Service switch. |
| RST-01-T09 | Alt: no network | T01 done once | Airplane mode on; close and reopen Profile. | The same values still show; no error is needed. |
| RST-01-T10 | Alt: unsaved edits | T01 on DEVICE-TAB | Type `X` at the end of Address (don't save). Send the app to the background, bring it back, open Home and return to Profile. | Address still ends with `X`; `Save Changes` is active. |
