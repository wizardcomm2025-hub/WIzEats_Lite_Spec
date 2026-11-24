# Bill & payment — data model

Sources:

- `db_script/pos_stage.sql` (`payments`, `orders`; re-checked 2026-11-24: unchanged — `bills` / `bill_items` are still missing from the dump, X-D7)
- `app/Services/Bill/BillService.php`, `app/Services/Payment/PaymentService.php`
- `app/Models/Bill.php`, `BillItem.php`, `Payment.php`

## `bills`

| Column | Value |
|---|---|
| `id` | new GUID |
| `restaurantId`, `orderId`, `orderNumber`, `orderChannelId` | from the order |
| `financialYear` | `YYYY-YY`, April–March, from the server date |
| `billNumber` | int; `max(billNumber) + 1` for the restaurant and year; no unique key today (BILL-D3); target unique (`restaurantId`, `financialYear`, `billNumber`) |
| `subTotal` … `grandTotal` (8 totals) | **as posted** today (BILL-D1); target worked out by the server from the lines, the order's tax % and the discount |
| `billStatus` | 0 on create; 1 on payment |
| `customerName`, `customerPhone` | from the order (NULL for app orders until ORD-D3 is fixed) |
| `createdBy` | caller's role |

## `bill_items`

One row per distinct (`menuItemId`, `menuItemName`, `price`, `taxPercent`) of the order's lines: `quantity`, `total`, `taxAmount` summed. `menuItemName` NOT NULL (BILL-D1). No `restaurantId`.

## `payments`

| Column | Value |
|---|---|
| `paymentId` | new GUID |
| `restaurantId`, `orderId` | caller's restaurant; the order |
| `transactionId` | `TXN-` + 12 random upper-case letters / digits |
| `paymentType` | as posted, any text (target `CASH` / `UPI`) |
| `amount` | **target only** — new column `decimal(10,2)` (migration needed): the leg's share of the bill |
| `receivedAmount` | `create`: `amount`; `create2`: `receivedAmount` (nullable) |
| `changeAmount` | `create2` only |
| `paymentStatus` | `COMPLETED` |
| `paymentNote`, `createdBy` | as posted; caller's role |

`payments.*.amount` (create2) is not stored anywhere today (BILL-D1); target: in the new `amount` column, and the legs must add up to the bill's `grandTotal`.

## `orders` changes

| Event | Columns |
|---|---|
| Bill generated | the 8 totals (overwriting `subTotal`), `orderStatus` 1 |
| Paid | `orderStatus` 2, `paymentStatus` 1 |

## Other side effects of payment

`tables.status` / `rooms.status` = 0 for the order's table / room, if any (app orders have one since the ORD-D2 fix). Target (DLV-D3): a Delivery-channel order's KOT → status 3.

## Locks

- Generate: `orders` row and the restaurant's `restaurant_configs` row, `FOR UPDATE`.
- Pay: `orders` row `FOR UPDATE`; pay-and-bill runs the generate step inside the same transaction.
