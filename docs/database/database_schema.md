# Database Schema Design (v2 - Multi-Tenant)

## Overview
This document outlines the database structure for the Inventory Management System. We are using **PostgreSQL**. The design follows 3NF and supports **Multi-Tenancy** via an `organizations` table.

## ER Diagram
```mermaid
erDiagram
    ORGANIZATIONS ||--o{ ORGANIZATION_MEMBERS : "has members"
    USERS ||--o{ ORGANIZATION_MEMBERS : "belongs to"
    ROLES ||--o{ ORGANIZATION_MEMBERS : "assigned role"

    ORGANIZATIONS ||--o{ PRODUCTS : "owns"
    ORGANIZATIONS ||--o{ LOCATIONS : "owns"
    ORGANIZATIONS ||--o{ SUPPLIERS : "manages"
    ORGANIZATIONS ||--o{ CUSTOMERS : "serves"
    
    CATEGORIES ||--o{ PRODUCTS : "classifies"
    SUPPLIERS ||--o{ PRODUCTS : "supplies"
    
    PRODUCTS ||--o{ INVENTORY_ITEMS : "has stock in"
    LOCATIONS ||--o{ INVENTORY_ITEMS : "stores"
    
    SUPPLIERS ||--o{ PURCHASE_ORDERS : "receives"
    PURCHASE_ORDERS ||--o{ PURCHASE_ORDER_ITEMS : "contains"
    PRODUCTS ||--o{ PURCHASE_ORDER_ITEMS : "ordered as"
    
    CUSTOMERS ||--o{ SALES_ORDERS : "places"
    SALES_ORDERS ||--o{ SALES_ORDER_ITEMS : "contains"
    PRODUCTS ||--o{ SALES_ORDER_ITEMS : "sold as"
    
    INVENTORY_TRANSACTIONS }o--|| PRODUCTS : "affects"
    INVENTORY_TRANSACTIONS }o--|| LOCATIONS : "at"
    INVENTORY_TRANSACTIONS }o--|| USERS : "performed by"
    
    USERS {
        uuid id PK
        string email UK
        string password_hash
        string full_name
        boolean is_active
        timestamp created_at
    }

    ORGANIZATIONS {
        uuid id PK
        string name
        string slug UK "unique-url-friendly-id"
        uuid owner_id FK
        timestamp created_at
    }

    ORGANIZATION_MEMBERS {
        uuid id PK
        uuid organization_id FK
        uuid user_id FK
        uuid role_id FK
        timestamp joined_at
    }

    ROLES {
        uuid id PK
        string name "Admin, Manager, Staff"
        jsonb permissions
    }

    PRODUCTS {
        uuid id PK
        uuid organization_id FK
        string sku
        string name
        text description
        uuid category_id FK
        decimal price
        decimal cost_price
        string unit_of_measure
        int low_stock_threshold
        boolean is_active
        timestamp created_at
        timestamp updated_at
    }

    CATEGORIES {
        uuid id PK
        uuid organization_id FK
        string name
        uuid parent_id FK
    }

    LOCATIONS {
        uuid id PK
        uuid organization_id FK
        string name
        string address
        string type
    }

    INVENTORY_ITEMS {
        uuid id PK
        uuid organization_id FK
        uuid product_id FK
        uuid location_id FK
        int quantity
        timestamp last_counted_at
    }

    INVENTORY_TRANSACTIONS {
        uuid id PK
        uuid organization_id FK
        uuid product_id FK
        uuid location_id FK
        string transaction_type
        int quantity_change
        string reference_id
        string reason
        uuid performed_by_user_id FK
        timestamp created_at
    }

    SUPPLIERS {
        uuid id PK
        uuid organization_id FK
        string name
        string contact_email
        string phone
        string address
    }

    PURCHASE_ORDERS {
        uuid id PK
        uuid organization_id FK
        string po_number
        uuid supplier_id FK
        string status
        decimal total_amount
        timestamp created_at
        timestamp expected_delivery_date
    }

    SALES_ORDERS {
        uuid id PK
        uuid organization_id FK
        string so_number
        uuid customer_id FK
        string status
        decimal total_amount
        timestamp created_at
    }
    
    CUSTOMERS {
        uuid id PK
        uuid organization_id FK
        string name
        string email
        string phone
    }
```

## Key Changes for Multi-Tenancy

### 1. `organizations` Table
The root entity. All data belongs to an organization.
-   `slug`: Allows for URLs like `app.com/org-slug/dashboard`.

### 2. `organization_members` Table
Links Users to Organizations.
-   Allows a single User (email) to belong to multiple Organizations (e.g., a freelancer working for two companies).
-   Roles are now assigned *per organization*. You can be an Admin in Org A but a Viewer in Org B.

### 3. Row-Level Security (RLS) Preparation
Every single data table (`products`, `orders`, etc.) now has an `organization_id` column.
-   **Constraint**: `unique(organization_id, sku)` for products. SKUs only need to be unique *within* an organization.
-   **Constraint**: `unique(organization_id, po_number)` for orders.

## Detailed Table Specifications (Updates)

#### `users`
*Global user identity.*
| Column | Type | Constraints | Description |
| :--- | :--- | :--- | :--- |
| `id` | UUID | PK | |
| `email` | VARCHAR | UNIQUE | |
| `password_hash` | VARCHAR | | |

#### `organization_members`
| Column | Type | Constraints | Description |
| :--- | :--- | :--- | :--- |
| `id` | UUID | PK | |
| `organization_id` | UUID | FK | |
| `user_id` | UUID | FK | |
| `role_id` | UUID | FK | Role within this org |

#### `products`
| Column | Type | Constraints | Description |
| :--- | :--- | :--- | :--- |
| `id` | UUID | PK | |
| `organization_id` | UUID | FK, Index | **CRITICAL** |
| `sku` | VARCHAR | | Unique per Org |
| ... | ... | ... | |

## Design Decisions
1.  **Shared Database, Separate Rows**: We are using a single database where all tenants share tables, distinguished by `organization_id`. This is the most cost-effective and easiest to manage for self-hosting.
2.  **Global Users**: Users are global entities. This allows a user to switch organizations without logging out and back in.
