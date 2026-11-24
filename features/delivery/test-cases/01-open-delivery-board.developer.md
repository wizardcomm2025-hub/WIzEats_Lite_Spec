# 01 — Open the delivery board — developer cases

Use case: [01-open-delivery-board.md](../01-open-delivery-board.md). Contract: [api-contract.md](../api-contract.md#post-apiorderschannelorderchannelid). Fixtures: [README.md](./README.md#fixtures).

`POST /api/orders/channel/<id>`, no body, WTR-A's token unless stated. "Listed" = 200 `{ status: true, message: "Delivery orders retrieved successfully", data: [ … ] }`. "Wrapped-fail" (today) = 200 `{ status: true, data: { status: false, message: "Failed to retrieve delivery orders", code: 500 } }`. Cases tagged *Fails until … is fixed* assert the target; today's result is in brackets.

| ID | Category | Request / manipulation | Expected |
|---|---|---|---|
| DLV-01-D01 | — (happy path, empty) — *Fails until DLV-D1 is fixed* | REST-A has no delivery orders; `<id>` = CH-DLV | Listed with `data: []` (today: Wrapped-fail) |
| DLV-01-D02 | — (happy path) — *Fails until DLV-D1 is fixed* | DLV-ORD exists; `<id>` = CH-DLV; then CH-DLV in dashed form | Listed [DLV-ORD] both: `orderId` / `kotId` hex, `orderNumber`, `customerName` `Ravi`, `customerPhone`, `customerAddress` `12 Park St`, `grandTotal`, `kotStatus` 0, `kotNumber`, items (today: Wrapped-fail) |
| DLV-01-D03 | Restaurant isolation — *Fails until DLV-D1 is fixed* | `<id>` = CH-B-DLV; then CH-DINE (REST-A's Dine In channel) | 404 `{ status: false, message: "Channel not found", data: null }` both (today: Wrapped-fail) |
| DLV-01-D04 | Input validation / Injection / Error hygiene — *Fails until DLV-D1 is fixed* | `<id>` = `3` (the app's fallback); `NaN`; `xyzw`; `' OR 1=1 --` (URL-encoded) | 404 `{ status: false, message: "Channel not found", data: null }` each (today: 404 with a `hex2bin()` message) |
| DLV-01-D05 | Role — *Fails until DLV-D2 is fixed* | CHF-A, then OWN-A, with CH-DLV | CHF-A: 403 `{ status: false, message: "You do not have permission to perform this action", data: null }` (today Wrapped-fail); OWN-A: Listed |
| DLV-01-D06 | Authentication / Account state / Super-admin — switched-off part *Fails until AUTH-D2 is fixed* | no token → 401; panel cookie only → 401; `GET` → 405; NOREST → 403 `Restaurant context missing`; WTR-A-OFF old token → 403 `Your account is inactive.` (today Wrapped-fail) | as listed |
| DLV-01-D07 | Over-posting | CH-DLV with body `{ "restaurantId": "<REST-B hex>", "orderChannel": "DELIVERY" }` | same as D01 / D02 (body ignored) |
| DLV-01-D08 | Rate limiting | 60 calls in a minute | all answered; no 429 |
| DLV-01-D09 | Business rule — *Fails until DLV-D1 is fixed* | Two delivery orders, the second created later; and a third whose KOT is Completed (3) | Listed: the later order first; the Completed one absent (today: Wrapped-fail) |
| DLV-01-D10 | Error hygiene — *Fails until DLV-D1 is fixed* | force a failure (e.g. drop DB read rights) | 500 `{ status: false, message: "Something went wrong", data: null }` (today: the wrapped failure inside a 200) |
