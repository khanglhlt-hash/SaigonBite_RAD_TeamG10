# CRC Cards – UC #14: Manage Brands and Menu Items
- **Author:** Nguyễn Phan Anh Tuấn (`524H0135`)

| Class Name (Layer) | Responsibilities | Collaborators |
| :--- | :--- | :--- |
| **CatalogManagementView** *(Boundary)* | Render bilingual Brand & Menu Item CRUD forms, hub assignment checkboxes, archive confirmation dialogs, and validation errors. | `CatalogController` |
| **CatalogController** *(Control)* | Enforce Admin RBAC, validate bilingual fields & `unitPrice > 0`, coordinate Brand/Item CRUD and cascading soft-archive to `HubMenuAvailability`. | `CatalogManagementView`, `VirtualBrand`, `MenuItem`, `HubMenuAvailability`, `CatalogDA` |
| **VirtualBrand** *(Domain)* | Represent virtual brand bilingual metadata and archive status (`isArchived`). | `MenuItem` |
| **MenuItem** *(Domain)* | Represent bilingual dish info, unit price (`> 0`), category, and archive state. | `VirtualBrand`, `HubMenuAvailability` |
| **HubMenuAvailability** *(Domain)* | Track real-time per-hub dish availability and stock count; disable on archive. | `MenuItem`, `CloudKitchenHub` |
| **CatalogDA** *(Data Access)* | Execute SQL CRUD and soft-archive transactions across `VirtualBrand`, `MenuItem`, and `HubMenuAvailability`. | `VirtualBrand`, `MenuItem`, `HubMenuAvailability` |