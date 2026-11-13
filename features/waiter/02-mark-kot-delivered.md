# 02 — Mark a KOT delivered

## Summary

When the food reaches the customer, the waiter taps `Delivered to Customer` on the ready card. The server marks the KOT picked up and the card moves to the `Delivered` tab. **Today no card is ever shown (WAIT-D1); through the API the update matches no row unless the raw ID is sent, and then any KOT anywhere can be marked (WAIT-D2).**

## Prerequisites

- **Backend:** `POST /api/waiter/updatePickedUpStatus/{kotId}` — [api-contract.md](./api-contract.md#post-apiwaiterupdatepickedupstatuskotid)
- **Data:** `kots`

## Actors & preconditions

- **Actor:** waiter. Any role through the API (WAIT-D2).
- **Preconditions:** a ready card on screen — unreachable in the app today, so this use case is exercised through the API.

## Main flow

1. The waiter taps `Delivered to Customer`. That button shows a spinner; the other cards' buttons are disabled.
2. The app posts `/waiter/updatePickedUpStatus/<kotId>` with `{}`.
3. The server runs `UPDATE kots SET isPickedUp = 1, pickedUp_at = now WHERE kotId = '<path text>'`, then reads every `kots` row with `isPickedUp` 1 (all restaurants) and returns them.
4. On success the app moves the card to `Delivered` (blue `DELIVERED` pill) and removes it from `Pickup`.

## Alternative flows

- **Hex `kotId`.** No row changes. The reply is 200 with `data: []` only if no picked-up KOT exists in the whole database; otherwise 500 (WAIT-D1/D2).
- **Raw ID (percent-encoded bytes).** The KOT is marked picked up whatever its restaurant or stage. The reply is then a 500, because it lists at least that KOT (WAIT-D2).
- **Failure in the app.** The spinner stops; the card stays in `Pickup`; no message.
- **App restarted or signed out.** The `Delivered` tab is empty again (WAIT-D2).

## Acceptance criteria

1. Given no KOT anywhere has `isPickedUp` 1, when WTR-A sends KOT-R's hex ID, then 200 `KOT picked up status updated successfully`, `data: []`, and KOT-R is unchanged.
2. Given RAW(KOT-R), then KOT-R has `isPickedUp` 1 and `pickedUp_at` set, and the reply is 500 `An error occurred while update kot picked up status `.
3. Given RAW(KOT-B) (REST-B) or RAW(KOT-P) (Pending), then the KOT is marked picked up all the same (WAIT-D2).

## Platform notes

### POS API

`StaffController::updateKOTPickedUpStatus` → `StaffService::updateKOTPickedUpStatus`. No form request; the body is ignored. The JSON encoding fails in the controller, after the service has returned, so the controller's own catch answers.

### Data — MySQL

`kots.isPickedUp`, `kots.pickedUp_at`. The KOT's `status` isn't changed.

### POS app

`confirmKot` thunk; `waiter.slice` keeps `deliveredKots` in memory only. `PickupKotCard`, `DeliveredKotCard`.

### Super-admin panel

None.

### Integrations

None.

## Configuration

None.

## Out of scope

- Closing the order or the table — BILL / TBL.

## Open questions

- None open.
