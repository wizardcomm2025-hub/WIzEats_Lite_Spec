# 02 — Move a delivery order on

## Summary

On a delivery card staff tap `Accept` or `Reject`, then `Out For Delivery`, then `Complete Payment`. Each tap sets the order's KOT stage; accepting, sending out and completing are meant to message the customer on WhatsApp. **Today no card is shown (DLV-D1), and the endpoint never updates the KOT it is given (DLV-D2).**

## Prerequisites

- **Backend:** `POST /api/orders/update-status/{kotId}` — [api-contract.md](./api-contract.md#post-apiordersupdate-statuskotid)
- **Data:** `kots`
- **Integration:** WhatsApp Cloud API (`WHATSAPP_API_URL`, `WHATSAPP_ACCESS_TOKEN`)

## Actors & preconditions

- **Actor:** owner or waiter. Any role through the API (DLV-D2).
- **Preconditions:** a delivery card on screen — unreachable today, so this use case is exercised through the API.

## Main flow

1. `Pending` card: `Reject` (status 2) or `Accept` (1). `Accepted` card: `Out For Delivery` (4). `Out for Delivery` card: `Complete Payment` (3). One action at a time; the other cards are dimmed.
2. The app posts `{ "status": <n> }` to `/orders/update-status/<kotId>`. A card with no KOT ID shows the toast `Order #<orderId> has no KOT to update` instead.
3. **Intended:** the server updates that KOT's `status` and `updated_at`, sends the WhatsApp text for 1 / 3 / 4, and answers 200 `KOT status updated successfully` with `{ kotId, status }`. The app shows `Order #<n> accepted` / `rejected` / `is out for delivery` / `payment completed` and changes the card's stage.
4. **Today:** see the alternative flows; nothing is updated and nothing is sent (DLV-D2).

## Alternative flows

- **ID cast to a non-zero number** (hex starting with digits, e.g. `42ab…` → 42). Normally no KOT matches: 400 `{ status: false, message: "KOT not found" }`.
- **ID cast to 0** (hex starting with a letter or `0`, or `NaN`). The lookup finds some KOT of any restaurant; the update then fails in strict mode: 400 with the SQL error text, or 500 `Server Error` if that text can't be encoded. Nothing changed, no message sent.
- **`status` missing or not an integer.** 422 `{ status: false, message: "Validation error", errors }`.
- **Failure in the app.** Red toast with the server's message; the card stays as it was.

## Acceptance criteria

1. Given KOT-D (hex starting with a digit 1–9), when its hex ID is posted with status 1, then 400 `KOT not found`, and KOT-D is unchanged.
2. Given KOT-L (hex starting with a letter), when its hex ID is posted with status 1, then an error reply (400 SQL text or 500 `Server Error`), no KOT anywhere changes, and no WhatsApp request is made.
3. Given `{ }`, then 422 `Validation error`.

## Platform notes

### POS API

`OrderController::updateOrderKOTStatus` (`(int) $kotId`; any exception → 400 with its message) → `KotService::updateOrderKOTStatus` → `sendKOTStatusWhatsAppMessage` (fixed recipient; errors logged only).

### Data — MySQL

`kots.status`, `kots.updated_at` (intended). Strict mode is on (`config/database.php`).

### POS app

`useDelivery.updateStatus`, `updateDeliveryStatusThunk`, `DeliveryOrderCard`.

### Super-admin panel

None.

### Integrations

WhatsApp Cloud API — a text message to a fixed number (DLV-D3). Never reached today.

## Configuration

`WHATSAPP_API_URL`, `WHATSAPP_ACCESS_TOKEN` in `.env`.

## Out of scope

- Real payment and the bill — BILL.

## Open questions

- None open.
