# 02 — Wait for approval — developer cases

Use case: [02-wait-for-approval.md](../02-wait-for-approval.md). Fixtures: [README.md](./README.md#fixtures).

These cases show what the API does for restaurants that are not Accepted. Today the app's screens are the only gate; cases tagged *Fails until AUTH-D2 is fixed* assert the X-D2 target, where the API refuses them too. `showDetails` and `logout` stay open to Pending / Rejected restaurants.

| ID | Category | Caller | Request / manipulation | Expected | Database after |
|---|---|---|---|---|---|
| ONB-02-D01 | Account state (X-D2 exemption) | OWN-P token | `POST /api/restaurants/showDetails` | 200 · `status: true` · `data.isApproved` = 0, `data.isOnboardingDone` true — this is what the app's check reads. Passes today and must keep passing after AUTH-D2. | unchanged |
| ONB-02-D02 | Account state / Business-rule bypass — *Fails until AUTH-D2 is fixed* | OWN-P token | `POST /api/tables/ZoneSave` `{ "zoneName": "Pending Zone" }` | 403 · `{ "status": false, "message": "Your restaurant is not approved.", "data": null }` (today: 200 `Zone created successfully`) | **no** zone created |
| ONB-02-D03 | Account state / Business-rule bypass — *Fails until AUTH-D2 is fixed* | OWN-R token | the same two calls as D01 and D02 | `showDetails` 200 with `data.isApproved` = 2 · `ZoneSave` 403 `Your restaurant is not approved.` (today: 200) | no zone created |
| ONB-02-D04 | Account state (ONB-D4) | OWN-LEG token | `showDetails` | 200 · `data.isOnboardingDone` false, `data.isApproved` 0 (the app never makes this call for OWN-LEG). REST-LEG is not in the panel list. | unchanged |
| ONB-02-D05 | Account state | none | set REST-P `isApproved = 7` in the database (after ONB-D2 only a database edit can do this); loginPin as OWN-P; then `showDetails` with the token | loginPin `data.isApproved: "Rejected"`; `showDetails` `data.isApproved` = 7 | restore REST-P afterwards |
| ONB-02-D06 | Account state — *Fails until AUTH-D2 is fixed* | OWN-P token | set REST-P `status = 0`; call `showDetails`; then `auth/logout` | `showDetails` → 403 `Your restaurant is inactive.` (the approval exemption does not cover a switched-off restaurant; today: 200) · `logout` → 200 | token deleted by the logout; restore REST-P |
