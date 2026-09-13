# 08 - Database

**Category:** Database  
**Purpose:** Schema design, entity relationships, migrations

---

## Contents

- `schema-overview.md` - Database architecture overview
- `entity-relationships.md` - ER diagrams and relationships
- `table-specifications.md` - Detailed table definitions (200+ tables)
- `indexes-optimization.md` - Index strategy
- `partitioning-strategy.md` - Data partitioning for scale
- `migration-strategy.md` - Schema versioning and migrations
- `backup-recovery.md` - Backup and disaster recovery

---

## Database Technology

- **DBMS:** PostgreSQL 16+
- **ORM:** Prisma
- **Migrations:** Prisma Migrate
- **Search:** Elasticsearch 8+ (full-text search)
- **Cache:** Redis 7+ (query caching, sessions)

---

## Core Entities

### Identity & Access
- `users` - All system users (customers, vendors, admins)
- `roles` - Role definitions
- `permissions` - Permission definitions
- `user_roles` - User-role assignments
- `sessions` - Active sessions
- `otp_codes` - SMS OTP verification codes

### Marketplace
- `vendors` - Vendor accounts
- `vendor_kyc` - KYC verification documents
- `vendor_stores` - Vendor store configurations
- `store_templates` - 10+ store templates

### Catalog
- `products` - Product master table
- `product_variants` - Product variations (size, color, etc.)
- `categories` - Product categories (tree structure)
- `attributes` - Product attributes
- `product_attributes` - Product-attribute values
- `offers` - Promotions and discounts

### Orders
- `orders` - Master orders
- `order_items` - Sub-orders per vendor
- `order_states` - 17-state order tracking
- `order_state_history` - State transition audit

### Payments
- `wallets` - User wallet accounts
- `wallet_transactions` - Transaction ledger
- `escrow_holds` - 7-day escrow tracking
- `cod_approvals` - COD vendor approvals

### Shipping
- `delivery_providers` - Registered providers
- `delivery_bids` - Competitive bidding
- `shipments` - Shipment tracking
- `delivery_codes` - 3-attempt verification

### Content
- `reviews` - Product reviews
- `ratings` - Product ratings
- `loyalty_points` - Customer loyalty tracking
- `support_tickets` - Customer support

---

## Data Constraints

### Non-Negotiable Rules
1. **Wallet-only payments** - NO card data storage
2. **SMS OTP storage** - 5-minute expiry
3. **Escrow hold** - 7-day minimum
4. **Order states** - Exactly 17 states
5. **Multi-currency** - YER, SAR, USD support
6. **Audit trails** - Immutable financial records

---

## Performance

- **Indexing:** All foreign keys, search fields, date ranges
- **Partitioning:** Orders and transactions by date
- **Caching:** Redis for hot queries (<100ms)
- **Read Replicas:** For analytics and reporting
- **Connection Pooling:** Max 100 connections per service

---

## Related Categories
- `06-backend` - Data access layer
- `16-data` - Data management and analytics
- `03-system-analysis` - Domain model

---

*Source: Database schema from analayesev2/05-DATABASE and entity analysis*
