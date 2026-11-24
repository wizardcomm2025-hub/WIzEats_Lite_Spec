# 02 — Move a delivery order on

## Summary

On a delivery card staff tap `Accept` or `Reject`, then `Out For Delivery`, then `Complete Payment`. Each tap sets the order's KOT stage, and accepting, sending out and completing message the customer on WhatsApp. **Today no card is shown (DLV-D1), and the endpoint never updates the KOT it is given (DLV-D2).** Target: the stage rules hold, the message reaches the customer, and `Complete Payment` takes a real payment (DLV-D3).

## Prerequisites

- **Backend:** `POST /api/orders/update-status/{kotId}` — [api-contract.md](./api-contract.md#post-apiordersupdate-statuskotid); BILL (payment, target)
- **Data:** `kots`, `orders`
- **Integration:** WhatsApp Cloud API (`WHATSAPP_API_URL`, `WHATSAPP_ACCESS_TOKEN`)

## Actors & preconditions

- **Actor:** owner or waiter. Today any role through the API; target owner and waiter (X-D1).
- **Preconditions:** a delivery card on screen — unreachable until DLV-D1 is fixed.

## Main flow

1. `Pending` card: `Reject` (status 2) or `Accept` (1). `Accepted` card: `Out For Delivery` (4). `Out for Delivery` card: `Complete Payment`. One action at a time; the other cards are dimmed.
2. For Accept, Reject and Out For Delivery the app posts `{ "status": <n> }` to `/orders/update-status/<kotId>`. A card with no KOT ID shows the toast `Order #<orderId> has no KOT to update` instead.
3. **Target:** the server finds the KOT on a delivery order of the restaurant, checks the role, the value and that the change is allowed, updates `status` and `updated_at`, sends the WhatsApp text for 1 / 4 to the order's customer phone, and answers 200 `KOT status updated successfully` with `{ kotId, status }`. The app shows `Order #<orderId> accepted` / `rejected` / `is out for delivery` and changes the card's stage. **Today:** see the alternative flows; nothing is updated and nothing is sent (DLV-D2).
4. **`Complete Payment` — target (DLV-D3):** opens the normal Bill Payment page for the order (BILL). When the bill is fully paid, the server sets the KOT to Completed (3), sends the completed message, and the order leaves the board. **Today:** the app posts status 3 like the other buttons and shows `Order #<orderId> payment completed` on success; no payment or bill is recorded.

## Alternative flows

- **ID cast to a non-zero number** (today; hex starting with digits, e.g. `42ab…` → 42). Normally no KOT matches: 400 `{ status: false, message: "KOT not found" }`. *Target:* the hex ID is found.
- **ID cast to 0** (today; hex starting with a letter or `0`, or `NaN`). The lookup finds some KOT of any restaurant; the update then fails in strict mode: 400 with the SQL error text, or 500 `Server Error`. Nothing changed, no message sent. *Target:* 404 `KOT not found` for a malformed ID.
- **A stage jump** (e.g. Pending → Out for Delivery, or back from Completed). Today no check. *Target:* 409 `Invalid status change`.
- **Order without a customer phone.** *Target:* the stage changes; no WhatsApp message (logged).
- **`status` missing or not an integer.** 422 `{ status: false, message: "Validation error", errors }`.
- **Failure in the app.** Red toast with the server's message or `Failed to update delivery status`; the card stays as it was.

## Acceptance criteria

1. *(Target, DLV-D2)* Given DLV-ORD (Pending, REST-A), when `Accept` is tapped, then the toast `Order #… accepted` shows, the card is `Accepted`, and KOT status is 1. (Today: no card; through the API `KOT not found` or an SQL error.)
2. *(Target, DLV-D2)* Given a Pending delivery KOT, when status 4 is posted, then 409 `Invalid status change`.
3. *(Target, DLV-D3)* Given DLV-ORD has `customerPhone` 9876543210, when it is accepted, then the WhatsApp request goes to 9876543210 with `Your order has been accepted.`
4. *(Target, DLV-D3)* Given an `Out for Delivery` card, when `Complete Payment` is tapped and the bill paid in full, then a payment and bill exist for the order and the card leaves the board.
5. *(Target, DLV-D2)* Given a Dine In order's KOT or REST-B's delivery KOT, then 404 `KOT not found`.

## Platform notes

### POS API

`OrderController::updateOrderKOTStatus` (today `(int) $kotId`; any exception → 400 with its message) → `KotService::updateOrderKOTStatus` → `sendKOTStatusWhatsAppMessage` (today a fixed recipient; errors logged only).

### Data — MySQL

`kots.status`, `kots.updated_at`. Strict mode is on (`config/database.php`).

### POS app

`useDelivery.updateStatus`, `updateDeliveryStatusThunk`, `DeliveryOrderCard`. The toast's `<orderId>` is the order's 32-char hex ID today.

### Super-admin panel

None.

### Integrations

WhatsApp Cloud API — today a text message to a fixed number; target the order's customer (DLV-D3).

## Configuration

`WHATSAPP_API_URL`, `WHATSAPP_ACCESS_TOKEN` in `.env`.

## Out of scope

- The Bill Payment page itself — BILL.

## Open questions

- None open. Related decisions: DLV-D2, DLV-D3.
