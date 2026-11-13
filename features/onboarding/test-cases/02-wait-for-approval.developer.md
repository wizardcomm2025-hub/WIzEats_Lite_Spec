# 02 — Wait for approval — developer cases

Use case: [02-wait-for-approval.md](../02-wait-for-approval.md). Fixtures: [README.md](./README.md#fixtures).

These cases show what the API does for restaurants that are not Accepted. The app's screens are the only gate (AUTH-D2), so most cases are **accepted risks**.

| ID | Category | Caller | Request / manipulation | Expected | Database after |
|---|---|---|---|---|---|
| ONB-02-D01 | Account state | OWN-P token | `POST /api/restaurants/showDetails` | 200 · `status: true` · `data.isApproved` = 0, `data.isOnboardingDone` true — this is what the app's check reads | unchanged |
| ONB-02-D02 | Account state / Business-rule bypass (**accepted risk** AUTH-D2) | OWN-P token | `POST /api/tables/ZoneSave` `{ "zoneName": "Pending Zone" }` | 200 · `message: "Zone created successfully"` — a Pending restaurant can use the POS API | zone created for REST-P |
| ONB-02-D03 | Account state / Business-rule bypass (**accepted risk** AUTH-D2) | OWN-R token | the same two calls as D01 and D02 | 200 for both; `data.isApproved` = 2 | zone created for REST-R |
| ONB-02-D04 | Account state (ONB-D4) | OWN-LEG token | `showDetails` | 200 · `data.isOnboardingDone` false, `data.isApproved` 0 (the app never makes this call for OWN-LEG). REST-LEG is not in the panel list. | unchanged |
| ONB-02-D05 | Account state (ONB-D2) | none | set REST-P `isApproved = 7` in the database; loginPin as OWN-P; then `showDetails` with the token | loginPin `data.isApproved: "Rejected"`; `showDetails` `data.isApproved` = 7 | restore REST-P afterwards |
