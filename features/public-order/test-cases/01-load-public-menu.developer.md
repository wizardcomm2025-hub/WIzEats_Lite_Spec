# 01 — Load the public menu — developer cases

Use case: [01-load-public-menu.md](../01-load-public-menu.md). Contract: [api-contract.md](../api-contract.md#post-apipublicmenu). Fixtures: [README.md](./README.md#fixtures).

`POST /api/public/menu`, no token. "Empty" = 200 `{ status: true, message: "Menu items retrieved successfully", data: { restaurantName: null, restaurantConfig: null, orderChannel: "", items: [] } }`. For RAWB here, send only the `restaurantId` field (plus the fields a case names).

| ID | Category | Request / manipulation | Expected |
|---|---|---|---|
| PUB-01-D01 | — (happy path; accepted PUB-D2) | RAWB(REST-A) | 200 `data.restaurantName` `Rest A`; `restaurantConfig` `{ taxPercent: "5.00" }`; `orderChannel` holds `DINE_IN`, `TAKEAWAY`, `ROOM_SERVICE`, `DELIVERY` once each, comma-separated, any order; `items` = every REST-A item (ITEM-1 to ITEM-4, ITEM-DEL), any order. Each item has 32-character hex `menuItemId`, `restaurantId` (= REST-A hex) and `categoryId`, `price` as a string (`"250.00"`), `isVeg` / `isActive` booleans, `itemCategoryName` (`Starters` / `Drinks`) |
| PUB-01-D02 | Business rule (accepted PUB-D1) | JSON `{ "restaurantId": "<REST-A hex>" }`; again with the hex in upper case; again as a dashed GUID | Empty, each time |
| PUB-01-D03 | Business-rule bypass (accepted PUB-D2) | (a) RAWB(REST-A) and look at ITEM-3 and ITEM-DEL (b) set CAT-D `isActive = 0` by SQL; repeat (c) set REST-A's `ROOM_SERVICE` channel `isActive = 0` by SQL; repeat | (a) ITEM-3 listed with `isActive: false`; ITEM-DEL listed with `isDelete: 1` (b) ITEM-4 `Lime Soda` still listed with `itemCategoryName` `Drinks` (c) `orderChannel` has the other three codes only |
| PUB-01-D04 | Account state (accepted PUB-D2) | RAWB(REST-P); RAWB(REST-R); RAWB(REST-OFF) | 200 with that restaurant's name, config and items each time; no approval or status check |
| PUB-01-D05 | Restaurant isolation (accepted PUB-D2) | RAWB(REST-B) | 200 with REST-B's name and data: ITEM-B1 `B Curry` listed; no REST-A item. Any restaurant's menu is readable by anyone who has its ID |
| PUB-01-D06 | Input validation | (a) `{}` (b) `{ "restaurantId": "" }` (c) `{ "restaurantId": 123 }` (d) `{ "restaurantId": "<REST-A hex>", "tableId": 7 }` (e) `{ "restaurantId": ["x"] }` | 422 `{ status: false, message, errors }` with `message` (a)(b) `The restaurant id field is required.` (c)(e) `The restaurant id field must be a string.` (d) `The table id field must be a string.` |
| PUB-01-D07 | Over-posting | (a) RAWB(REST-A) plus `tableId=<REST-B table hex>&roomId=x&isDelete=0&restaurantName=Hacked` (b) JSON hex REST-A with the same extras | (a) same `data` as D01 (b) Empty. Extra fields are ignored; nothing is written |
| PUB-01-D08 | Authentication / Super-admin | D01 with (a) no `Authorization` header (b) `Authorization: Bearer garbage` (c) OWN-B's token (d) a super-admin panel session cookie only; (e) `GET /api/public/menu` | (a)–(d) the same 200 as D01: no token is read, and REST-B's owner sees REST-A's menu (e) 405 |
| PUB-01-D09 | Rate limiting | 100 × D01 in one minute from one IP | all 200; no 429 (PUB-D3) |
| PUB-01-D10 | Injection | JSON `restaurantId` = (a) `' OR '1'='1` (b) `%` (c) `0` (d) `1; DROP TABLE menu_items` | Empty each; `menu_items` intact (values are bound) |
| PUB-01-D11 | Error hygiene | In a throwaway database, rename `restaurant_configs`; RAWB(REST-A) with SHEET-CATCH; then rename it back | 500 `{ status: false, message: "Failed to retrieve public menu items", data: null }`; no SQL text in the reply; SHEET-CATCH receives one row with `function` `App\Services\Public\PublicService::getPublicMenuItems` |
