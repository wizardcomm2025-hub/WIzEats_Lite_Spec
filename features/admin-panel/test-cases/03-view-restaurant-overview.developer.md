# 03 — View a restaurant's overview — developer cases

Use case: [03-view-restaurant-overview.md](../03-view-restaurant-overview.md). Contract: [api-contract.md](../api-contract.md#restaurant-pages). Fixtures: [README.md](./README.md#fixtures) (SESS).

`GET /admin/analytics/<id>?filter=<f>` with SESS; `<id>` = REST-A hex unless stated. Read the figures from the HTML.

| ID | Category | Request / manipulation | Expected |
|---|---|---|---|
| ADM-03-D01 | — (happy path; accepted ADM-D2) | ORDER-LIVE + ORDER-PAID; `filter=today` | Figures as ADM-03-T01; the `Recent Orders` IDs = `HEX(orderId)` (upper case); the embedded `revenueLabels` = `["<today UTC>"]`, `revenueData` = `["525.00"]` |
| ADM-03-D02 | Business rule (accepted ADM-D2) | PAID-AT(today − 7 d 14:00 IST), PAID-AT(today − 8 d), PAID-AT(today − 30 d), PAID-AT(today − 31 d); each filter | `7days` 1; `15days` 2; `30days` 3; `today` 0 |
| ADM-03-D03 | Restaurant isolation / Input validation / Injection / Error hygiene | `<id>` = (a) REST-B hex (b) REST-A as a dashed GUID (c) `zz` (d) `' OR 1=1 --` (URL-encoded) (e) 32 `0`s (f) 31 hex characters | (a) REST-B's own figures (any admin, any restaurant — ADM-D1) (b) same as D01 (c)–(f) 200 with all zeros (no 404, no SQL error) |
| ADM-03-D04 | Business-rule bypass (accepted ADM-D2) | D01 data plus ORDER-OLD; `filter=all`, `filter=`, no filter | `all` and `filter=` (empty becomes null): all-time figures (ORDER-OLD included), while the dropdown shows `Today`. No `filter` parameter: today only |
| ADM-03-D05 | Rate limiting | 100 × D01 in a minute | all 200; no 429 |
