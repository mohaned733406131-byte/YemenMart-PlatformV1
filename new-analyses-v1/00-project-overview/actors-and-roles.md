# YemenMart Actors & Roles Analysis

## Overview

YemenMart defines **7 actors** with distinct responsibilities and permissions. This document provides a complete RBAC (Role-Based Access Control) matrix, actor descriptions, and permission mappings.

---

## Actor Registry

### A-01: Customer (BUYER)

| Field | Details |
|-------|---------|
| **Actor ID** | A-01 |
| **Role Code** | BUYER |
| **Description** | End user who purchases products from merchants |
| **Registration** | Phone-first (SMS/WhatsApp OTP) |
| **KYC Required** | No |
| **Default Locale** | ar (Arabic) |

**Core Responsibilities:**
- Register and authenticate via phone OTP
- Fund wallet and manage balance
- Browse products, add to cart, checkout
- Receive deliveries and enter delivery codes
- Request returns within merchant policy window
- Rate and review products and merchants

**Key Permissions:**
- View public product catalog
- Manage own cart (max 50 items, max 10 per product)
- Place orders (500–5,000,000 YER range)
- View own order history
- Initiate returns
- View own wallet transactions

---

### A-02: Merchant (SELLER)

| Field | Details |
|-------|---------|
| **Actor ID** | A-02 |
| **Role Code** | SELLER |
| **Description** | Vendor who lists and sells products on the platform |
| **Registration** | Phone-first + KYC mandatory (Constraint 19) |
| **KYC Required** | Yes |
| **Default Locale** | ar (Arabic) |

**Core Responsibilities:**
- Complete KYC verification
- Set up storefront (choose from 10+ templates)
- List products with pricing, images, descriptions
- Manage inventory and stock levels
- Process orders and update status
- Define return policies (`isReturnable`, `returnPeriodDays`)
- Respond to customer inquiries
- View sales analytics and financial reports

**Key Permissions:**
- Full CRUD on own products
- Manage own store settings
- View and process own orders
- View own sales analytics
- Manage own return policies
- View own wallet balance and transactions
- Respond to customer reviews

---

### A-03: Delivery Agent (COURIER)

| Field | Details |
|-------|---------|
| **Actor ID** | A-03 |
| **Role Code** | COURIER |
| **Description** | Agent who picks up and delivers orders to customers |
| **Registration** | Phone-first + Admin approval |
| **KYC Required** | Yes (simplified) |
| **Default Locale** | ar (Arabic) |

**Core Responsibilities:**
- Accept delivery assignments
- Pick up packages from merchants
- Deliver to customer addresses
- Verify delivery codes (3 attempts max)
- Update delivery status
- Handle failed delivery scenarios

**Key Permissions:**
- View assigned deliveries
- Update delivery status (PICKED_UP, IN_TRANSIT, OUT_FOR_DELIVERY, DELIVERED)
- Verify delivery codes
- Report delivery issues
- View own delivery history and earnings

---

### A-04: Platform Administrator (ADMIN)

| Field | Details |
|-------|---------|
| **Actor ID** | A-04 |
| **Role Code** | ADMIN |
| **Description** | Platform operator with full system access |
| **Registration** | Admin-only provisioning |
| **KYC Required** | N/A (internal) |
| **Default Locale** | ar (Arabic) |

**Core Responsibilities:**
- Manage platform configuration
- Approve merchant KYC applications
- Monitor system health and performance
- Handle escalated support tickets
- Manage content (banners, pages, promotions)
- Generate compliance reports (ZATCA, VAT)
- Manage 40+ system service categories
- Oversee 17-order-state lifecycle

**Key Permissions:**
- Full system read/write access
- User management (all actors)
- Product moderation
- Order intervention (override, cancel)
- Financial reports and exports
- System configuration
- Content management
- Audit log access

---

### A-05: Super Administrator (SUPER_ADMIN)

| Field | Details |
|-------|---------|
| **Actor ID** | A-05 |
| **Role Code** | SUPER_ADMIN |
| **Description** | Platform owner with unrestricted access |
| **Registration** | System bootstrap only |
| **KYC Required** | N/A (system) |
| **Default Locale** | ar (Arabic) |

**Core Responsibilities:**
- Manage all administrators
- Access all system data
- Configure platform-wide settings
- Manage payment gateway connections
- Access financial ledgers (double-entry bookkeeping)
- Configure security policies
- Manage notification templates

**Key Permissions:**
- All ADMIN permissions
- Manage ADMIN accounts
- Access raw financial data
- Configure payment systems
- Access all audit logs
- System backup and restore
- Emergency lockdown capabilities

---

### A-06: Content Moderator (MODERATOR)

| Field | Details |
|-------|---------|
| **Actor ID** | A-06 |
| **Role Code** | MODERATOR |
| **Description** | Reviews and moderates platform content |
| **Registration** | Admin provisioning |
| **KYC Required** | N/A (internal) |
| **Default Locale** | ar (Arabic) |

**Core Responsibilities:**
- Review product listings for policy compliance
- Check for third-party branding violations (Constraint 18)
- Moderate customer reviews
- Handle content-related complaints
- Flag suspicious merchant activity
- Maintain content quality standards

**Key Permissions:**
- Read all product listings
- Approve/reject product listings
- Edit product content (with merchant notification)
- Remove policy-violating content
- View merchant activity logs
- Flag accounts for review

---

### A-07: Support Agent (SUPPORT)

| Field | Details |
|-------|---------|
| **Actor ID** | A-07 |
| **Role Code** | SUPPORT |
| **Description** | Handles customer and merchant support requests |
| **Registration** | Admin provisioning |
| **KYC Required** | N/A (internal) |
| **Default Locale** | ar (Arabic) |

**Core Responsibilities:**
- Handle customer support tickets
- Assist with delivery code lockout issues
- Process manual refund requests (with approval)
- Help merchants with KYC issues
- Escalate issues to ADMIN
- Maintain knowledge base

**Key Permissions:**
- Read customer/merchant profiles
- View order details
- Create and update support tickets
- Initiate manual actions (with approval workflow)
- View delivery agent status
- Access knowledge base

---

## RBAC Permission Matrix

### Legend
- **CRUD** = Create, Read, Update, Delete
- **R** = Read only
- **RW** = Read + Write
- **X** = Execute
- **-** = No access

### Module-Level Permissions

| Module | BUYER | SELLER | COURIER | ADMIN | SUPER_ADMIN | MODERATOR | SUPPORT |
|--------|-------|--------|---------|-------|-------------|-----------|---------|
| **M01: Registration & Login** | CRUD | CRUD | CRUD | CRUD | CRUD | - | - |
| **M02: Profile Management** | CRUD | CRUD | CRUD | RW | RW | - | R |
| **M03: KYC & Verification** | - | CRUD | CRUD | RW | RW | - | R |
| **M04: Product CRUD** | R | CRUD | - | RW | RW | RW | R |
| **M05: Category Management** | R | R | - | CRUD | CRUD | R | - |
| **M06: Inventory Management** | R | CRUD | - | R | RW | - | R |
| **M07: Storefront Setup** | - | CRUD | - | R | RW | - | R |
| **M08: Store Templates** | R | R | - | CRUD | CRUD | - | - |
| **M09: Search Engine** | R | R | R | R | R | R | R |
| **M10: Recommendations** | R | R | - | R | RW | - | - |
| **M11: Cart Service** | CRUD | - | - | R | RW | - | - |
| **M12: Checkout Flow** | X | - | - | R | RW | - | - |
| **M13: Order Lifecycle** | RW | RW | RW | RW | RW | R | RW |
| **M14: Sub-Order Mgmt** | R | RW | R | RW | RW | R | R |
| **M15: Wallet Service** | RW | RW | RW | R | RW | - | R |
| **M16: Escrow Engine** | R | R | - | RW | RW | - | R |
| **M17: Delivery Mgmt** | R | R | RW | RW | RW | - | R |
| **M18: Delivery Code System** | X | - | X | R | RW | - | X |
| **M19: Return Processing** | X | X | - | RW | RW | R | X |
| **M20: Refund Engine** | R | R | - | RW | RW | - | X |
| **M21: Notification Hub** | R | R | R | RW | RW | - | R |
| **M22: Analytics Engine** | R | R | R | RW | RW | R | R |
| **M23: Admin Panel** | - | - | - | CRUD | CRUD | R | R |

### Data-Level Permissions

| Data Entity | BUYER | SELLER | COURIER | ADMIN | SUPER_ADMIN | MODERATOR | SUPPORT |
|-------------|-------|--------|---------|-------|-------------|-----------|---------|
| **User Profile (Own)** | CRUD | CRUD | CRUD | - | - | - | - |
| **User Profile (Others)** | - | - | - | RW | RW | - | R |
| **Products (Own)** | - | CRUD | - | RW | RW | - | R |
| **Products (All)** | R | R | - | RW | RW | RW | R |
| **Orders (Own)** | RW | - | - | - | - | - | - |
| **Orders (Merchant)** | - | RW | - | RW | RW | - | R |
| **Orders (All)** | - | - | - | RW | RW | - | R |
| **Wallet (Own)** | RW | RW | RW | - | - | - | - |
| **Wallet (All)** | - | - | - | R | RW | - | - |
| **Financial Ledger** | - | - | - | R | RW | - | - |
| **Audit Logs** | - | - | - | R | RW | - | - |
| **System Config** | - | - | - | RW | CRUD | - | - |
| **Content (CMS)** | R | R | - | RW | RW | RW | R |
| **Support Tickets** | CRUD | CRUD | - | RW | RW | R | CRUD |

### Action Permissions

| Action | BUYER | SELLER | COURIER | ADMIN | SUPER_ADMIN | MODERATOR | SUPPORT |
|--------|-------|--------|---------|-------|-------------|-----------|---------|
| Place Order | ✅ | - | - | ✅ | ✅ | - | - |
| Cancel Order | ✅* | ✅* | - | ✅ | ✅ | - | ✅* |
| Request Return | ✅ | - | - | ✅ | ✅ | - | ✅* |
| Approve Return | - | ✅ | - | ✅ | ✅ | - | - |
| Process Refund | - | - | - | ✅ | ✅ | - | ✅* |
| Enter Delivery Code | ✅ | - | ✅ | ✅ | ✅ | - | ✅ |
| Approve KYC | - | - | - | ✅ | ✅ | - | - |
| Suspend User | - | - | - | ✅ | ✅ | - | - |
| View Reports | ✅** | ✅** | ✅** | ✅ | ✅ | ✅** | ✅** |

*With approval workflow*
**Limited to own data*

---

## Role Hierarchy

```
SUPER_ADMIN
    │
    ├── ADMIN
    │     ├── MODERATOR
    │     └── SUPPORT
    │
    ├── SELLER (Merchant)
    │
    ├── COURIER (Delivery Agent)
    │
    └── BUYER (Customer)
```

**Inheritance Rules:**
- SUPER_ADMIN inherits all ADMIN permissions
- ADMIN inherits all MODERATOR and SUPPORT permissions
- Lower roles do NOT inherit from higher roles
- Each role has independent, scoped permissions

---

## Actor Lifecycle

| Event | BUYER | SELLER | COURIER | ADMIN | SUPER_ADMIN |
|-------|-------|--------|---------|-------|-------------|
| Registration | Self-service | Self-service + KYC | Admin invite | System only | Bootstrap |
| Activation | OTP verify | KYC approve | Admin approve | System only | System only |
| Suspension | Admin action | Admin action | Admin action | Super Admin | Emergency only |
| Deactivation | Self or Admin | Admin action | Admin action | Super Admin | Emergency only |
| Reactivation | Admin action | Re-KYC | Admin action | Super Admin | Emergency only |

---

*Document Version: 1.0 | Last Updated: 2026-09-13 | Classification: Confidential*
