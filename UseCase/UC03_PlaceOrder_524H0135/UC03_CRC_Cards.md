# CRC Cards – UC #3: Place an Order
- **Author:** Nguyễn Phan Anh Tuấn (`524H0135`)

| Class Name (Layer) | Responsibilities | Collaborators |
| :--- | :--- | :--- |
| **OrderPlacementView** *(Boundary)* | Display cart, checkout summary, busy-hub warning & quiet-hub offer, payment selector; capture voucher code & order confirmation within <= 4 screens. | `OrderController` |
| **OrderController** *(Control)* | Orchestrate distance check (<= 7.0 km), busy/quiet hub rule, fee calculation, single-voucher validation, tourist COD cap, atomic order creation, and gateway call. | `OrderPlacementView`, `CloudKitchenHub`, `Order`, `Voucher`, `PaymentTransaction`, `OrderDA` |
| **Order** *(Domain)* | Maintain order attributes, line items, pricing breakdown, and enforce valid initial state `Placed`. | `OrderLineItem`, `Voucher`, `CloudKitchenHub`, `PaymentTransaction` |
| **CloudKitchenHub** *(Domain)* | Check busy threshold (`>= 25`), quiet threshold (`< 50%`), calculate distance, and increment active order count. | `HubMenuAvailability`, `Order` |
| **Voucher** *(Domain)* | Validate active window, usage limit, minimum spend, and compute capped discount (`<= foodSubtotal`). | `Order` |
| **OrderDA** *(Data Access)* | Execute atomic SQL transaction (`BEGIN IMMEDIATE`) to persist `Order`, `OrderLineItem`, `PaymentTransaction`, and update hub load / voucher count. | `Order`, `CloudKitchenHub`, `Voucher`, `PaymentTransaction` |