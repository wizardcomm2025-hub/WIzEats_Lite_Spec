# 01 — Open the delivery board — developer cases

Use case: [01-open-delivery-board.md](../01-open-delivery-board.md). Contract: [api-contract.md](../api-contract.md#post-apiorderschannelorderchannelid). Fixtures: [README.md](./README.md#fixtures).

`POST /api/orders/channel/<id>`, no body, WTR-A's token unless stated. "Wrapped-fail" = 200 `{ status: true, data: { status: false, message: "Failed to retrieve delivery orders", code: 500 } }`.

| ID | Category | Request / manipulation | Expected |
|---|---|---|---|
| DLV-01-D01 | — (happy path, as the app; accepted DLV-D1) / Error hygiene | `<id>` = `3` | 404 `{ status: false, message: "hex2bin(): Hexadecimal input must have an even length" }` |
| DLV-01-D02 | Business rule (accepted DLV-D1) | `<id>` = CH-DLV; then CH-DLV in dashed UUID form | Wrapped-fail both |
| DLV-01-D03 | Restaurant isolation | `<id>` = CH-B-DLV | Wrapped-fail (no REST-B data) |
| DLV-01-D04 | Input validation / Injection / Error hygiene | `<id>` = `NaN`; `xyzw`; `' OR 1=1 --` (URL-encoded) | 404 with the even-length message; 404 `hex2bin(): Input string must be hexadecimal string`; 404 with one of the two `hex2bin()` messages (by the length after removing dashes) |
| DLV-01-D05 | Role | CHF-A, then OWN-A, with CH-DLV | Wrapped-fail both (no 403) |
| DLV-01-D06 | Authentication / Account state / Super-admin | no token → 401; panel cookie only → 401; `GET` → 405; NOREST → 403 `Restaurant context missing`; WTR-A-OFF old token, CH-DLV → Wrapped-fail (AUTH-D2) | as listed |
| DLV-01-D07 | Over-posting | CH-DLV with body `{ "restaurantId": "<REST-B hex>", "orderChannel": "DELIVERY" }` | Wrapped-fail (body ignored) |
| DLV-01-D08 | Rate limiting | 60 calls in a minute | all answered; no 429 |
