# 03 — View a restaurant's overview — developer cases

Use case: [03-view-restaurant-overview.md](../03-view-restaurant-overview.md). Contract: [api-contract.md](../api-contract.md#restaurant-pages). Fixtures: [README.md](./README.md#fixtures) (SESS).

`GET /admin/analytics/<id>?filter=<f>` with SESS; `<id>` = REST-A hex unless stated. Read the figures from the HTML.

| ID | Category | Request / manipulation | Expected |
|---|---|---|---|
| ADM-03-D01 | — (happy path) — *Fails until ADM-D2 is fixed* | ORDER-LIVE + ORDER-PAID; `filter=today` | Figures as ADM-03-T01; the `Recent Orders` IDs = `HEX(orderId)` (upper case); the embedded `revenueLabels` = `["<today IST>"]`, `revenueData` = `["525.00"]` (today: the UTC date; figures as today's T01) |
| ADM-03-D02 | Business rule — *Fails until ADM-D2 is fixed* | PAID-AT(today − 6 d 14:00 IST), PAID-AT(today − 7 d 14:00 IST), PAID-AT(today − 29 d 14:00 IST), PAID-AT(today − 30 d 14:00 IST), PAID-AT(today 00:30 IST); each filter | `today` 1; `yesterday` 0; `7days` 2; `15days` 3; `30days` 4 (today: `today` 0, `yesterday` 1, `7days` 3, `15days` 3, `30days` 5) |
| ADM-03-D03 | Restaurant isolation / Input validation / Injection / Error hygiene | `<id>` = (a) REST-B hex (b) REST-A as a dashed GUID (c) `zz` (d) `' OR 1=1 --` (URL-encoded) (e) 32 `0`s (f) 31 hex characters | (a) REST-B's own figures (any admin, any restaurant — ADM-D1) (b) same as D01 (c)–(f) 200 with all zeros (no 404, no SQL error) |
| ADM-03-D04 | Business-rule bypass — *Fails until ADM-D2 is fixed* | D01 data plus ORDER-OLD; `filter=all`, `filter=`, `filter=7DAYS`, no filter | Each the same as `filter=today` (ORDER-OLD not counted), with the dropdown on `Today` (today: `all`, `filter=` and `7DAYS` give all-time figures including ORDER-OLD) |
| ADM-03-D05 | Rate limiting | 100 × D01 in a minute | all 200; no 429 |
| ADM-03-D06 | Business rule — *Fails until ADM-D2 is fixed* | ITEMS-12; plus one unpaid order of a 13th item with quantity 50; `filter=today` | `Popular Items` = 10 rows, quantities 12 … 3; the unpaid item absent (today: 13 rows, the unpaid item first) |
