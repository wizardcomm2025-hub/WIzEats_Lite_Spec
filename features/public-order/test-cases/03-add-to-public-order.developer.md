# 03 — Add to a public order — developer cases

Use case: [03-add-to-public-order.md](../03-add-to-public-order.md). Contract: [api-contract.md](../api-contract.md#post-apipublicorder). Fixtures: [README.md](./README.md#fixtures) (ROW-SNAP for every case).

`POST /api/public/order`, no token. Every 500 below is `{ status: false, message: "Something went wrong", error: <text> }`. "Unchanged" = every value matches ROW-SNAP, including the fixture order's `subTotal`, `grandTotal` and `updated_at`.

| ID | Category | Request / manipulation | Expected | After |
|---|---|---|---|---|
| PUB-03-D01 | — (happy path; accepted PUB-D1) | JSON: PUB-BODY plus `"orderId": "<ORDER-L hex>"` | 500. `error` starts `App\Services\Public\PublicService::addKotToPublicOrder(): Argument #1 ($orderId) must be of type int, string given, called in` and ends `PublicController.php on line 42` | Unchanged |
| PUB-03-D02 | Input validation (accepted PUB-D1) | JSON: PUB-BODY plus `"orderId": "<ORDER-D hex>"` | 500 `error` `A non-numeric value encountered` | Unchanged |
| PUB-03-D03 | Business rule (accepted PUB-D1) | JSON: PUB-BODY (hex REST-A) plus `"orderId"` = (a) `0` (b) `5` (c) `"0"`; ORDER-L exists | 500 `error` `Order not found` each time | Unchanged |
| PUB-03-D04 | Business-rule bypass (accepted PUB-D1) | RAWB(REST-A) with `orderId=0` and `kotNote=more`; ORDER-L is REST-A's only order | 500 `error` starts `SQLSTATE[HY000]: General error: 1364 Field 'kotId' doesn't have a default value (Connection: mysql, SQL: insert into `kots` (`kotNote`, `createdBy`, `created_at`, `updated_at`) values (more, customer,` | Unchanged; ORDER-L's totals and lines as before |
| PUB-03-D05 | Input validation (accepted PUB-D1) | D04 without the `kotNote` field | 500 `error` `Undefined array key "kotNote"` | Unchanged |
| PUB-03-D06 | Account state / Business rule | D04 with ORDER-L-BILLED as REST-A's only order | 500 `error` `Bill already generated, cannot add items to this order` | Unchanged |
| PUB-03-D07 | Restaurant isolation (accepted PUB-D1, PUB-D2) | RAWB(REST-B) with `orderId=0`, `kotNote=more`, REST-A's ITEM-1 as the item; ORDER-B-RUN is REST-B's only order | Same 500 as D04: the lookup found ORDER-B-RUN with no credentials, and the REST-A item was not checked | Unchanged; ORDER-B-RUN as before |
| PUB-03-D08 | Input validation / Over-posting | JSON: PUB-BODY plus `"orderId"` = (a) `""` (b) `null` (c) `["x"]` (d) `true` | (a)(b) Column error from [02](./02-place-public-order.developer.md): treated as a new order (c) 500 `error` contains `must be of type int, array given` (d) 500 `Order not found` (`true` becomes 1) | Unchanged |
| PUB-03-D09 | Input validation | JSON `{ "orderId": 0 }` only; then PUB-BODY with `"orderId": 0` and `"items": []` | 422 `The restaurant id field is required.`; then 422 `The items field is required.` (validation runs before any lookup) | Unchanged |
| PUB-03-D10 | Authentication / Super-admin / Rate limiting / Injection | D03(a) with (a) `Bearer garbage` (b) a panel cookie only; (c) `GET` → 405; (d) 30 × D03(a) in a minute; (e) `"orderId": "1 OR 1=1"` | (a)(b) as D03 (c) 405 (d) all `Order not found`; no 429 (e) 500 `error` `A non-numeric value encountered` (the leading `1` is used) | Unchanged; `orders` intact |
| PUB-03-D11 | Error hygiene (accepted PUB-D3, AUTH-D5) | Inspect D01 and D04; repeat D01 with SHEET-CATCH | D01 `error` exposes the server path of `PublicController.php`; D04 exposes the `kots` table and its column names; SHEET-CATCH receives a row with `function` `App\Http\Controllers\Public\PublicController::order` | Unchanged |
