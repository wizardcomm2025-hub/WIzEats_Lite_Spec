# 04 — Edit users in the panel — tester cases

Use case: [04-edit-users-in-panel.md](../04-edit-users-in-panel.md). Screen text: [ui-ux.md §3](../ui-ux.md#3-panel--user-management-page). Fixtures: [README.md](./README.md#fixtures).

Run in a desktop browser, signed in to the panel as ADMIN-1, on REST-A's user management page. A second tester holds the app where needed. Reset REST-A's users after each case.

| ID | Covers | Steps | Expected |
|---|---|---|---|
| STAFF-04-T01 | AC1 | Open the page; give REST-A a fourth user first (create one in the app as OWN-A after freeing a slot, or ask a developer); use the page links. | Tab `Analytics`; title `User Overview` / `User management`. Columns `SL No`, `Name`, `Phone`, `Role`, `Active`, `Actions`. OWN-A, WTR-A, CHF-A are listed, with the owner included; 3 rows per page; page 2 shows the fourth user with `SL No` 1. |
| STAFF-04-T02 | AC2 | Edit icon on WTR-A; Name `Waiter Two`; `Update`. App (OWN-A): reopen User Management. | Alert `User Updated Successfully`; page reloads with `Waiter Two`; the app lists `Waiter Two`. |
| STAFF-04-T03 | AC3 | Edit WTR-A; Status `Inactive`; `Update`. App (OWN-A): reopen User Management; tap `Add Staff`. | Alert; app row `Inactive` and faded; the `Waiter` chip is free; a new waiter can be created. WTR-A's next sign-in is refused. |
| STAFF-04-T04 | AC4 | Click `+ Add User`. | Nothing happens. |
| STAFF-04-T05 | AC5 | Click CHF-A's Active switch so it turns off; reload the page. | After reload the switch is on again; the app still shows CHF-A `Active`. |
| STAFF-04-T06 | AC6 (accepted STAFF-D3) | Edit WTR-A; Role `owner`; `Update`. WTR-A signs in on the app. | Alert; after sign-in WTR-A sees the owner's menus (Setup, Profile, Dashboard…). |
| STAFF-04-T07 | Alt: unknown role | Edit WTR-A; Role `manager`; `Update`. WTR-A signs in. OWN-A opens User Management. | WTR-A gets no menus; WTR-A is missing from OWN-A's staff list. |
| STAFF-04-T08 | AC7 | Open the dialog for WTR-A; in another tab log out of the panel; back in the first tab click `Update`. Sign in again and look. | No alert, no reload; WTR-A unchanged. |
| STAFF-04-T09 | Encoding / HTML | Edit WTR-A; Name `<b>Bold</b> রাহুল`; `Update`. | The list shows the text literally, not bold; the app shows the same text. |
