# 06 - Backend

**Category:** Backend  
**Purpose:** Server-side business logic, services, and data access

---

## Contents

- `service-architecture.md` - Service layer design
- `business-logic.md` - Core business rules implementation
- `data-access-layer.md` - Repository pattern, ORM usage
- `authentication-service.md` - SMS OTP verification flow
- `payment-service.md` - Wallet and escrow logic
- `order-service.md` - Master/Sub-order management
- `notification-service.md` - SMS and system notifications
- `job-scheduling.md` - Cron jobs and background tasks

---

## Backend Architecture

### Technology
- **Runtime:** Node.js 20+ LTS
- **Language:** TypeScript 5+
- **Framework:** Express/Fastify (custom REST API)
- **ORM:** Prisma
- **Database:** PostgreSQL 16+
- **Cache:** Redis 7+
- **Queue:** Bull/BullMQ for job processing

### Core Services
1. **Auth Service** - SMS OTP, JWT, session management
2. **User Service** - Customer/Vendor/Admin management
3. **Vendor Service** - KYC, store setup, approval workflow
4. **Product Service** - Catalog, attributes, inventory
5. **Order Service** - Master/Sub-orders, 17-state machine
6. **Payment Service** - Wallet, escrow, COD approval
7. **Finance Service** - Accounting, ledger, commissions
8. **Shipping Service** - Delivery marketplace, tracking
9. **Notification Service** - SMS, email, push notifications
10. **Analytics Service** - Reporting, dashboards, metrics

---

## Patterns

- **Repository Pattern** - Data access abstraction
- **Service Layer** - Business logic encapsulation
- **Factory Pattern** - Payment provider abstraction
- **Strategy Pattern** - Delivery provider selection
- **State Machine** - Order and payment state transitions
- **Event Sourcing** - Audit trail for financial transactions

---

## Related Categories
- `07-api` - API endpoints
- `08-database` - Database schema
- `09-security` - Security implementation

---

*Source: Backend design from analayesev2/03-FUNCTIONAL and architecture documents*
