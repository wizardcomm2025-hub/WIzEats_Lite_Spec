# 02 — Edit profile and settings in the app — tester cases

Use case: [02-edit-profile-and-settings.md](../02-edit-profile-and-settings.md). Screen text: [ui-ux.md](../ui-ux.md). Fixtures: [README.md](./README.md#fixtures).

Reset REST-A to REST-A-BASE before each case. Unless stated, the tester is signed in as OWN-A on DEVICE-TAB with Profile open. A second tester uses the panel ([03](./03-edit-in-panel.tester.md)) where needed.

| ID | Covers | Steps | Expected |
|---|---|---|---|
| RST-02-T01 | AC1 | Change Restaurant Name to `Rest A2`; tap `Save Changes`. | `Save Changes` turns active after typing; spinner; toast `Profile updated successfully`; the header shows `Rest A2`; button grey again. Panel page shows `Rest A2`. |
| RST-02-T02 | Alt: change back | Add `X` to Address, then delete it. | `Save Changes` goes active, then grey again. |
| RST-02-T03 | AC2 | Clear Restaurant Name; tap Save. Then enter only spaces; tap Save. | Both times toast `Restaurant name is required`; reopening Profile shows `Rest A` (nothing saved). |
| RST-02-T04 | AC2 | Clear Owner Name; tap Save. | Toast `Owner name is required`; nothing saved. |
| RST-02-T05 | AC3 | GSTIN `22AAAAA0000A1Z` (14 characters); tap Save. Then `22AAAAA0000A1Z5`; Save. Then clear it; Save. | First: field red with `Enter a valid 15-character GSTIN, e.g. 22AAAAA0000A1Z5`, toast `Please fix the highlighted fields`, nothing saved. Second: saved. Third: saved; GSTIN empty after reopening. |
| RST-02-T06 | AC4 | Restaurant Email `owner-at-cafe`; Save. | Toast `The restaurant email field must be a valid email address.`; the typed value stays; reopening Profile shows the old e-mail. |
| RST-02-T07 | AC5 (accepted risk RST-D4) | Tax Rate `7` → Save; take a new Dine In order and open its bill. Then Tax Rate `-5` → Save; take another order. Then `abc` → Save. Then `1000` → Save. | `7`: saved, shows `7.00`; the new order is taxed 7 %; an order opened before keeps 5 %. `-5`: saved as `-5.00`; the new order's tax is negative. `abc` and `1000`: toast `Failed to update restaurant details`; tax stays `-5.00`. |
| RST-02-T08 | AC9 (accepted risk RST-D1) | Switch Enable Takeaway off; Save; go Home. | Saved (switch off after reopening). Home still shows the `TAKEAWAY` tile and it opens as before. |
| RST-02-T09 | AC6 | Switch Enable Discount off; Save; open Bill Payment for a running order. Switch it on; Save; open Bill Payment again. | Off: no discount section. On: discount section shown. |
| RST-02-T10 | AC7 | Show KOT Number on Bill on, KOT Name `Token`; Save; open a bill. Clear the name; Save; open the bill. Switch Show KOT Number on Bill off. | `Token : <numbers>` in the bill header; then `KOT : <numbers>`; when switched off the `KOT Name on Bill` field disappears at once and, after Save, the bill header shows no KOT numbers. |
| RST-02-T11 | AC8 | QR ID `UPI-X`; Save; open Bill Payment; scan the QR with a UPI app (don't pay). | The UPI app shows UPI-X as the payee. |
| RST-02-T12 | AC10 (accepted risk RST-D2) | Panel: set Service Charge % `10`, `Save Configuration`. App: refresh (background → foreground), change Alternate Contact, Save. Panel: reload. | Service Charge % is empty in the panel. |
| RST-02-T13 | AC11 (accepted risk RST-D3) | Sign in as WTR-A on DEVICE-PHONE; open Profile; QR ID `UPI-X`; Save. | Toast `Profile updated successfully`; OWN-A's Bill Payment QR now pays UPI-X. |
| RST-02-T14 | AC12 | Change Address; airplane mode on; Save. Airplane off; Save again. | First: toast `Failed to update restaurant profile`; the typed address stays. Second: saved. |
| RST-02-T15 | Alt: double tap | Slow network; change the name; tap Save several times quickly. | One spinner; one toast; the name is saved once. |
| RST-02-T16 | Alt: long text | Paste TEXT-256 into Restaurant Name; Save. | Toast `Failed to update restaurant details`; nothing saved (also the other fields changed in the same save). |
| RST-02-T17 | Alt: e-mail cleared | Clear Restaurant Email; Save. | Saved; e-mail empty after reopening; panel list shows no e-mail. (PIN e-mails now have no address — AUTH-D6.) |
| RST-02-T18 | Finding F2 | A developer sets the name to NAME-HEX in the database. Open Profile (it shows hex — RST-01-T07); change only the Tax Rate; Save. Look at the panel. | Saved; from now on the panel and the app both show `436166c3a92044c3a96c696365203132` as the name — the real name is lost. |
| RST-02-T19 | Encoding | Restaurant Name NAME-UNI; Save. | Saved; Profile header, panel and bill show `রেস্তোরাঁ 🍛 Café` exactly. |
| RST-02-T20 | Layout | DEVICE-TAB landscape and portrait, and DEVICE-PHONE: tap QR ID (near the bottom) to open the keyboard. | The field scrolls above the keyboard; the `Save Changes` bar stays visible; no content hidden under it. |
| RST-02-T21 | Alt: decimals | Tax Rate `12.345`; Save. | Saved; Profile shows `12.35`. |
