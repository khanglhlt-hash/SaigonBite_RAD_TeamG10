# UC #3: Place an Order
- **Author:** Nguyễn Phan Anh Tuấn (`524H0135`)
- **Group:** (a) Ordering
- **Related User Stories / Requirements:** US-04, US-05, US-06, US-07 | FR4, FR5, FR6, FR7, NFR1, NFR5, NFR6

| Field | Specification |
| :--- | :--- |
| **Use Case Name** | Place an Order (hub, delivery fee, max 1 voucher, payment) |
| **Primary Actor** | Customer (Local Customer or Foreign Tourist) |
| **Secondary Actors** | Payment Gateway (MoMo / ZaloPay / VietQR / Stripe / VNPay), Hub Kitchen Staff (notified) |
| **Brief Description** | An authenticated customer selects a saved delivery address, chooses available menu items from a hub within 7.0 km, reviews distance-based delivery pricing, applies at most one promotional voucher, completes payment (or COD), and submits the order in `Placed` state. |
| **Trigger** | Customer clicks **Checkout / Place Order** from their shopping cart. |
| **Pre-conditions** | 1. Customer is logged in and has at least one active `DeliveryAddress`.<br>2. Cart contains >= 1 in-stock `MenuItem` from a selected `CloudKitchenHub`. |
| **Post-conditions** | 1. A new `Order` record is persisted in `Placed` state with associated `OrderLineItem` rows.<br>2. `CloudKitchenHub.currentActiveOrders` is incremented by 1 atomically.<br>3. If a voucher was applied, `Voucher.usedCount` is incremented by 1.<br>4. A `PaymentTransaction` record is persisted (`Authorized`/`Paid` for online, `PendingCOD` for COD).<br>5. Digital order ticket notification is pushed to the assigned Hub Kitchen Staff. |

## Normal Flow (Main Success Scenario)
1. Customer selects a saved `DeliveryAddress` (or hotel) and clicks **Proceed to Checkout**.
2. System calculates the distance from the selected `CloudKitchenHub` to the address and verifies `distance <= 7.0 km` (A-02).
3. System checks `CloudKitchenHub.currentActiveOrders < maxActiveOrders (25)` (A-01).
4. System computes delivery fee: `15,000 VND` for the first `2.0 km` + `5,000 VND` per additional km rounded up to `0.5 km` (A-02).
5. System displays the checkout summary (items, subtotal, delivery fee, estimated total).
6. *(Optional)* Customer enters one promotional `Voucher` code; System validates status, validity dates, minimum order value, usage quota, and applies discount (`<= foodSubtotal`) (A-05).
7. Customer chooses a permitted payment method (Local: COD/MoMo/ZaloPay/VietQR; Tourist: International Card or COD `<= 500,000 VND`) (A-03) and clicks **Confirm & Pay**.
8. System opens an atomic transaction, re-verifies hub capacity and item stock, invokes Payment Gateway (if online), persists `Order [Placed]`, `OrderLineItem`, `PaymentTransaction`, increments hub workload & voucher usage, and returns the digital order receipt.

## Alternative & Exception Flows
- **AF-1 (Busy-Kitchen Rule & Quiet-Kitchen Incentive):** At Step 3, if `currentActiveOrders >= 25`, System blocks checkout at the overloaded hub and searches for a nearby hub within `7.0 km` operating below `50%` capacity. If the customer agrees to switch hubs, System re-routes the order and automatically applies a `5%` quiet-kitchen discount (A-01, A-05). If declined, checkout ends.
- **EF-1 (Out of Delivery Radius):** At Step 2, if `distance > 7.0 km`, System declines checkout and prompts the customer to choose another address or closer hub (A-02).
- **EF-2 (Invalid / Second Voucher):** At Step 6, if voucher is expired, inactive, exceeds max usage, below min spend, or customer tries adding a 2nd voucher, System rejects the code with a clear message and preserves the single-voucher rule (A-05).
- **EF-3 (Tourist COD Exceeds Limit):** At Step 7, if a `ForeignTourist` selects COD for `totalAmount > 500,000 VND`, System blocks COD and prompts for international card payment (A-03).
- **EF-4 (Payment Failure or Concurrent Stock-out):** At Step 8, if online authorization fails or an item became out-of-stock / hub filled concurrently, System rolls back the transaction without creating an order and notifies the customer.