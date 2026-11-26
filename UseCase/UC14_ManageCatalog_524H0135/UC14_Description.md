# UC #14: Manage Brands and Menu Items
- **Author:** Nguyễn Phan Anh Tuấn (`524H0135`)
- **Group:** (c) CRUD Management
- **Related User Stories / Requirements:** US-03 | FR3, NFR5, NFR6

| Field | Specification |
| :--- | :--- |
| **Use Case Name** | Manage Brands and Menu Items |
| **Primary Actor** | Office Staff / Admin |
| **Secondary Actors** | None |
| **Brief Description** | Admin creates, views, updates, and soft-archives Virtual Brands and bilingual Menu Items (Vietnamese & English), and initializes per-hub availability records (`HubMenuAvailability`) without corrupting historical orders. |
| **Trigger** | Admin opens the **Catalog Management** dashboard to add, edit, or archive a brand or dish. |
| **Pre-conditions** | 1. Actor is authenticated with the `Admin` / `Office Staff` role (RBAC – NFR5). |
| **Post-conditions** | 1. **Create:** New `VirtualBrand` or `MenuItem` (with both VI & EN fields and `unitPrice > 0`) is persisted, and linked `HubMenuAvailability` rows are created for assigned hubs.<br>2. **Update:** Updated bilingual metadata, price, or category is saved and reflected in new customer searches.<br>3. **Archive (Soft Delete):** `isArchived = true` is set and corresponding `HubMenuAvailability.isAvailable` flags are set to `false`; historical `OrderLineItem` rows remain intact. |

## Normal Flow (Main Success Scenario)
1. Admin navigates to the **Catalog Management** page; System retrieves and displays active and archived `VirtualBrand` and `MenuItem` lists.
2. Admin selects an operation: **Create**, **Update**, or **Archive**.
3. **If Create Brand/Menu Item:** Admin inputs Vietnamese name (`nameVi`), English name (`nameEn`), bilingual descriptions, `unitPrice`, category, parent `brandId`, and selects target `CloudKitchenHub`s, then clicks **Save**.
4. System validates mandatory bilingual fields, verifies `unitPrice > 0`, and checks for duplicate item names within the same brand.
5. System persists the `VirtualBrand` or `MenuItem` and creates initial `HubMenuAvailability` rows for each assigned hub.
6. System displays a confirmation banner and refreshes the catalog list.

## Alternative & Exception Flows
- **AF-1 (Update Brand/Item):** At Step 2, Admin chooses **Edit** on an existing entity, modifies bilingual fields, price, or hub assignments, and clicks **Save**. System validates and updates the record (existing past orders retain their snapshot `OrderLineItem.unitPrice`).
- **AF-2 (Soft-Archive Brand/Item):** At Step 2, Admin clicks **Archive** and confirms the warning dialog. System sets `isArchived = true` and disables all linked `HubMenuAvailability` rows (`isAvailable = false`) instead of hard-deleting, preserving referential integrity for historical orders and reports (UC #16).
- **EF-1 (Invalid Input / Missing Bilingual Text / Price <= 0):** At Step 4, if either VI or EN name is blank or `unitPrice <= 0`, System rejects the submission and highlights the invalid fields.
- **EF-2 (Unauthorized Access):** If a non-Admin actor attempts to access catalog endpoints, System returns `403 Forbidden`.