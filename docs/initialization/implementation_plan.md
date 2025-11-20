# Inventory Management Dashboard - Project Plan

## Goal Description
Create a comprehensive, self-hosted Inventory Management Dashboard for an organization. The system will track products, stock levels, suppliers, and orders, ensuring data integrity and optimal performance using a modern tech stack.

## User Review Required
> [!IMPORTANT]
> **Tech Stack Confirmation**:
> - **Frontend**: Angular (Latest Version) - Excellent for enterprise-grade dashboards.
> - **Database**: PostgreSQL - Robust, relational, perfect for structured inventory data.
> - **Backend**: *Suggestion*: **NestJS** (Node.js framework). It pairs perfectly with Angular (both use TypeScript, decorators, modules) and is enterprise-ready. Alternatively, we could use Python (FastAPI/Django) or Go. **Please confirm if NestJS is acceptable.**
> - **Deployment**: Docker & Docker Compose (for easy self-hosting).

## Proposed Features (The "Bucket")

### 1. Dashboard & Analytics
- **Overview Cards**: Total Inventory Value, Low Stock Items, Total Products, Pending Orders.
- **Charts**: Monthly Sales/Stock Movement, Top Selling Products.
- **Recent Activity Log**: Who did what and when.

### 2. Product Management
- **CRUD Operations**: Add, Edit, Delete, View products.
- **Categorization**: Categories and Sub-categories.
- **Attributes**: SKU, Barcode/QR Code generation, Unit of Measure (kg, pcs, ltr), Dimensions.
- **Variants**: Support for sizes, colors, etc.
- **Image Upload**: Product images.

### 3. Inventory Control
- **Stock Adjustments**: Manually increase/decrease stock (with reason codes like "Damaged", "Audit", "New Stock").
- **Low Stock Alerts**: Configurable threshold per product.
- **Warehouses/Locations**: Support for multiple storage locations (optional, but good for scale).
- **Stock History**: Audit trail of every stock movement.

### 4. Supplier & Procurement
- **Supplier Database**: Contact info, lead times.
- **Purchase Orders (PO)**: Create POs to send to suppliers.
- **Goods Received Note (GRN)**: Receive stock against a PO.

### 5. Order Management (Outbound)
- **Sales Orders**: Record customer orders.
- **Pick, Pack, Ship**: Workflow for fulfilling orders.
- **Invoicing**: Generate simple invoices/packing slips.

### 6. User Management & Security
- **Role-Based Access Control (RBAC)**: Admin, Manager, Warehouse Staff.
- **Audit Logs**: Track user actions for security.

### 7. System Settings
- **General Config**: Currency, Timezone, Company Details.
- **Backup/Restore**: Database backup options.

## Database Structure Strategy
We will use a normalized relational schema (3NF) to ensure data integrity.
- **Tables**: Users, Roles, Products, Categories, Suppliers, PurchaseOrders, SalesOrders, InventoryTransactions, Locations.
- **Indexes**: On frequently searched columns (SKU, Name).
- **Foreign Keys**: To enforce referential integrity.

## Verification Plan
### Automated Tests
- Unit tests for backend logic (especially stock calculations).
- E2E tests for critical flows (Create Product -> Add Stock -> Sell Product).

### Manual Verification
- Verify "Self-Hosted" setup using `docker-compose up`.
- Test concurrent stock updates to ensure no race conditions.
