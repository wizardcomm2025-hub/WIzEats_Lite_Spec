# Bill & payment — data model

Sources:

- `db_script/tables/tables.sql` (`bills`, `bill_items`, `payments`, `orders`)
- `app/Services/Bill/BillService.php`, `app/Services/Payment/PaymentService.php`
- `app/Models/Bill.php`, `BillItem.php`, `Payment.php`

## `bills`

| Column | Value |
|---|---|
| `id` | new GUID |
| `restaurantId`, `orderId`, `orderNumber`, `orderChannelId` | from the order |
| `financialYear` | `YYYY-YY`, April–March, from the server date |
| `billNumber` | int; `max(billNumber) + 1` for the restaurant and year; no unique key (BILL-D3) |
| `subTotal` … `grandTotal` (8 totals) | **as posted** (BILL-D1) |
| `billStatus` | 0 on create; 1 on payment |
| `customerName`, `customerPhone` | from the order (always NULL for app orders — ORD-D3) |
| `createdBy` | caller's role |

## `bill_items`

One row per distinct (`menuItemId`, `menuItemName`, `price`, `taxPercent`) of the order's lines: `quantity`, `total`, `taxAmount` summed. `menuItemName` NOT NULL (BILL-D1). No `restaurantId`.

## `payments`

| Column | Value |
|---|---|
| `paymentId` | new GUID |
| `restaurantId`, `orderId` | caller's restaurant; the order |
| `transactionId` | `TXN-` + 12 random upper-case letters / digits |
| `paymentType` | as posted, any text |
| `receivedAmount` | `create`: `amount`; `create2`: `receivedAmount` (nullable) |
| `changeAmount` | `create2` only |
| `paymentStatus` | `COMPLETED` |
| `paymentNote`, `createdBy` | as posted; caller's role |

`payments.*.amount` (create2) is not stored anywhere (BILL-D1).

## `orders` changes

| Event | Columns |
|---|---|
| Bill generated | the 8 totals (overwriting `subTotal`), `orderStatus` 1 |
| Paid | `orderStatus` 2, `paymentStatus` 1 |

## Other side effects of payment

`tables.status` / `rooms.status` = 0 for the order's table / room, if any (BILL-D2).

## Locks

- Generate: `orders` row and the restaurant's `restaurant_configs` row, `FOR UPDATE`.
- Pay: `orders` row `FOR UPDATE`; pay-and-bill runs the generate step inside the same transaction.
