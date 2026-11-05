# 01 — <!-- verb phrase --> — tester cases

Use case: [01-use-case.md](../01-use-case.md). Screen text: [ui-ux.md](../ui-ux.md). Fixtures: [README.md](./README.md#fixtures). Worked example: [features/auth/test-cases/](../../auth/test-cases/).

Run in the POS app on a phone and a tablet. No endpoints, request bodies, HTTP codes or SQL.

| ID | Covers | Precondition / user | Steps | Expected |
|---|---|---|---|---|
| {{FEATURE_CODE}}-01-T01 | AC1 | <!-- signed in as OWN-A; … --> | <!-- 1. … 2. … --> | <!-- exact text on screen and the visible outcome --> |

<!-- Typical cases to include:
     happy path; each validation message; duplicate; long text / special characters / emoji;
     double tap on Save; no network then retry; app killed mid-flow;
     same flow signed in as waiter and as chef (screen hidden or refused);
     restaurant pending approval; inactive / deleted reference not offered in pickers;
     phone vs tablet layout; loading, empty and error states. -->
