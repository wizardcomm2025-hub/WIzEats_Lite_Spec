# 04 — View a running order — developer cases

Use case: [04-view-running-order.md](../04-view-running-order.md). Contract: [api-contract.md](../api-contract.md#post-apiordersorderid). Fixtures: [README.md](./README.md#fixtures).

`POST /api/orders/<orderId>`, no body, OWN-A's token.

| ID | Category | Request / manipulation | Expected |
|---|---|---|---|
| ORD-04-D01 | — (happy path) | ORDER-RUN with two KOTs | 200 `data.source` `live`; `kots` = 2 entries with `kotNumber` and items (`totalQuantity`, `total`, `taxAmount`); `menuItemName` = current menu name |
| ORD-04-D02 | — (billed) | ORDER-BILLED | 200 `source` `bill`; `billNumber`, `billStatus`, `grandTotal`, `items` from the bill |
| ORD-04-D03 | Restaurant isolation | ORDER-B | 404 `{ status: false, message: "Order not found" }` |
| ORD-04-D04 | Authentication / Account state / Role / Super-admin | no token → 401; panel cookie → 401; `GET /api/orders/<id>` → 405; CHF-A → 200; NOREST → 403 | as listed |
| ORD-04-D05 | Input validation / Injection / Error hygiene (accepted ORD-D2/D4) | `<orderId>` = `NaN` (what the app sends); `xyz`; `' OR 1=1 --` | 404 with the PHP `hex2bin()` warning text in `message` |
| ORD-04-D06 | Business rule / Rate limit | Rename ITEM-1 in the DB, then view ORDER-RUN; delete ITEM-2's row, view again; 60 calls/min | name shows the **new** name; the ITEM-2 lines disappear (inner join); no 429 |
