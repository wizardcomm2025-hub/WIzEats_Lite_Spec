# 02 — Mark a KOT delivered

## Summary

When the food reaches the customer, the waiter taps `Delivered to Customer` on the ready card. The server marks the KOT picked up and the card moves to the `Delivered` tab. **Today no card is ever shown (WAIT-D1); through the API the update matches no row unless the raw ID is sent, and then any KOT anywhere can be marked (WAIT-D2) — both To fix.**

## Prerequisites

- **Backend:** `POST /api/waiter/updatePickedUpStatus/{kotId}`, `POST /api/waiter/ready-kots` (`pickedUp: true`, target) — [api-contract.md](./api-contract.md#post-apiwaiterupdatepickedupstatuskotid)
- **Data:** `kots`

## Actors & preconditions

- **Actor:** waiter. Today any role through the API; target waiter only (WAIT-D2 / X-D1).
- **Preconditions:** a ready card on screen — unreachable in the app until KITCH-D1 and WAIT-D1 are fixed.

## Main flow

1. The waiter taps `Delivered to Customer`. That button shows a spinner; the other cards' buttons are disabled.
2. The app posts `/waiter/updatePickedUpStatus/<kotId>` with `{}`.
3. **Target:** the server checks the role, finds the KOT by hex ID in the caller's restaurant, checks it is Ready and not yet picked up, sets `isPickedUp` 1 and `pickedUp_at` now, and answers 200 with that KOT (`kotId`, `kotNumber`, `pickedUp_at`). **Today:** `UPDATE kots SET isPickedUp = 1, pickedUp_at = now WHERE kotId = '<path text>'`, then every `kots` row with `isPickedUp` 1 (all restaurants) is returned.
4. On success the app moves the card to `Delivered` (blue `DELIVERED` pill) and removes it from `Pickup`.
5. **Target:** the `Delivered` tab loads `ready-kots` with `{ "pickedUp": true }` — today's picked-up KOTs of the restaurant, from every device.

## Alternative flows

- **Hex `kotId`.** Today no row changes; the reply is 200 with `data: []` only if no picked-up KOT exists in the whole database, otherwise 500 (WAIT-D1/D2). *Target:* picked up.
- **Raw ID (percent-encoded bytes).** Today the KOT is marked picked up whatever its restaurant or stage, and the reply is a 500. *Target:* 404 `KOT not found`.
- **KOT not Ready, or already picked up.** Today marked again. *Target:* 409 `KOT is not ready for pickup` / `KOT already picked up`; nothing written.
- **Failure in the app.** The spinner stops; the card stays in `Pickup`; no message.
- **App restarted, signed out, or another device.** Today the `Delivered` tab is empty (memory only). *Target:* it shows today's picked-up KOTs.

## Acceptance criteria

1. *(Target, WAIT-D1)* Given KOT-R (Ready, REST-A), when WTR-A sends its hex ID, then 200 with `kotNumber` and `pickedUp_at`, and KOT-R has `isPickedUp` 1. (Today: unchanged.)
2. *(Target, WAIT-D2)* Given KOT-R is already picked up, then 409 `KOT already picked up`; given KOT-P (Pending), then 409 `KOT is not ready for pickup`.
3. *(Target, WAIT-D2)* Given KOT-BR (REST-B), then 404 `KOT not found` and it is unchanged.
4. *(Target, WAIT-D2)* Given WTR-A delivered a KOT on DEV-1, when the app is restarted or Ready Orders is opened on DEV-2, then `Delivered` lists it.

## Platform notes

### POS API

`StaffController::updateKOTPickedUpStatus` → `StaffService::updateKOTPickedUpStatus`. No form request; the body is ignored. Today the JSON encoding fails in the controller, after the service has returned, so the controller's own catch answers.

### Data — MySQL

`kots.isPickedUp`, `kots.pickedUp_at`. The KOT's `status` isn't changed.

### POS app

`confirmKot` thunk; `waiter.slice` keeps `deliveredKots` in memory only today (target: loaded from `ready-kots` with `pickedUp: true`). `PickupKotCard`, `DeliveredKotCard`.

### Super-admin panel

None.

### Integrations

None.

## Configuration

None.

## Out of scope

- Closing the order or the table — BILL / TBL.

## Open questions

- None open. Related decisions: WAIT-D1, WAIT-D2.
