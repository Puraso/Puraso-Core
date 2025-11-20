# API Architecture & Tech Stack (NestJS)

## Overview
This document defines the architectural standards for the NestJS backend. We will follow a **Modular Monolith** approach, keeping domains isolated but running in a single application for simplicity and performance.

## Tech Stack Details
-   **Framework**: NestJS (v10+)
-   **Language**: TypeScript
-   **Database ORM**: TypeORM (Great integration with NestJS & Postgres)
-   **Validation**: `class-validator` & `class-transformer`
-   **Documentation**: Swagger (OpenAPI)
-   **Authentication**: Passport (JWT Strategy)

## Global Standards
-   **API Prefix**: `/api/v1`
-   **Response Format**:
    ```json
    {
      "data": { ... }, // Or [...]
      "meta": {        // Optional, for pagination
        "total": 100,
        "page": 1,
        "limit": 10
      }
    }
    ```
-   **Error Format**:
    ```json
    {
      "statusCode": 400,
      "message": ["email must be an email"],
      "error": "Bad Request"
    }
    ```

## Module Structure

### 1. Auth Module (`/auth`)
Handles authentication and token generation.
-   `POST /auth/login`: Returns JWT access token.
-   `POST /auth/register`: Register new user (Admin only or public depending on config).
-   `GET /auth/profile`: Get current user info.

### 2. Products Module (`/products`)
Manages product catalog.
-   `GET /products`: List products (Pagination, Filter by Category/Name).
-   `GET /products/:id`: Get single product details.
-   `POST /products`: Create product (DTO: `CreateProductDto`).
-   `PATCH /products/:id`: Update product.
-   `DELETE /products/:id`: Soft delete product.

### 3. Inventory Module (`/inventory`)
Manages stock levels and movements.
-   `GET /inventory/stock`: Get current stock for all products (can filter by location).
-   `POST /inventory/adjust`: Manual stock adjustment (DTO: `AdjustStockDto` -> reason, qty, type).
    -   *Logic*: Creates a transaction record AND updates inventory balance.
-   `GET /inventory/transactions`: Audit log of movements.

### 4. Suppliers Module (`/suppliers`)
-   `GET /suppliers`: List suppliers.
-   `POST /suppliers`: Add supplier.

### 5. Orders Module (`/orders`)
Handles both Purchase Orders (Inbound) and Sales Orders (Outbound).
-   `POST /orders/purchase`: Create PO.
-   `POST /orders/purchase/:id/receive`: Receive goods (Updates Inventory).
-   `POST /orders/sales`: Create SO.
-   `POST /orders/sales/:id/fulfill`: Deduct stock and mark shipped.

## Key Design Patterns
1.  **DTOs (Data Transfer Objects)**:
    -   Strictly typed classes for all inputs.
    -   Example: `CreateProductDto` will use `@IsString()`, `@IsPositive()` decorators.
2.  **Services**:
    -   Business logic lives here, NOT in controllers.
    -   Example: `InventoryService.adjustStock()` handles the transaction creation logic.
3.  **Guards**:
    -   `JwtAuthGuard`: Protects private routes.
    -   `RolesGuard`: Enforces RBAC (e.g., `@Roles('ADMIN')`).

## Next Steps
1.  Initialize NestJS project.
2.  Install dependencies (`typeorm`, `pg`, `passport`, `swagger`).
3.  Generate modules using Nest CLI.
