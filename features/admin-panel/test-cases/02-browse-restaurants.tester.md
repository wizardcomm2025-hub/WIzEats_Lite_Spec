# 02 — Browse restaurants — tester cases

Use case: [02-browse-restaurants.md](../02-browse-restaurants.md). Screen text: [ui-ux.md §2](../ui-ux.md#2-restaurant-list-admin). Fixtures: [README.md](./README.md#fixtures) (REST-MANY).

Signed in as ADM-1 on the restaurant list.

| ID | Covers | Steps | Expected |
|---|---|---|---|
| ADM-02-T01 | AC1 | Look at the header and page 1. | `Total Restaurants` 12; `Active` + `Inactive` = 12 (REST-OFF counted as Inactive). Ten rows, newest first, numbered `01`–`10`. Footer `Showing 1 to 10 of 12 results`. REST-LEG is not listed. |
| ADM-02-T02 | AC1 (ADM-D3) | Go to page 2. | Two rows numbered `01`, `02` (not 11, 12); footer `Showing 11 to 12 of 12 results`. |
| ADM-02-T03 | AC2, AC3 | Find REST-A, REST-P, REST-R, REST-OFF. | REST-A `Active` / `Approved`; REST-P buttons `Accept` and `Reject`; REST-R `Rejected`; REST-OFF `Inactive`. |
| ADM-02-T04 | AC4 | Click REST-A's name; then REST-B's name. | REST-A's details open: `Restaurant ID` (32 lower-case hex characters, the same as `ID:` in the row), `GST ID` or `No gst id Given`, `FSSAI License` or `No fssai id Given`, `View Analytics`. Opening REST-B closes REST-A. `View Analytics` opens REST-A's Overview. |
| ADM-02-T05 | AC5 (ADM-D3) | On page 1, type part of the name of a restaurant that is on page 2. Then clear the box and type part of REST-A's owner name. | First: no rows shown. Then: only rows whose text contains it. The footer and pagination don't change. |
| ADM-02-T06 | Alt (ADM-D3) | Open `APP_URL/dashboard`. | A plain page `Restaurant Management` with four fixed rows: `Spice Hub` / `Rahul Sharma` / `Kolkata` `Active`, `Tandoori Flames`, `Urban Bites` `Inactive`, `Food Junction`; none of them is a real restaurant. |
