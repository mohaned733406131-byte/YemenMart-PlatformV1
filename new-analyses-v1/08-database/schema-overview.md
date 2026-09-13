# YemenMart Database Schema Overview

## Overview

This document provides a comprehensive overview of the YemenMart database schema across 13 blocks. The system uses PostgreSQL with UUID primary keys, soft deletes, and audit timestamps.

**Design Principles:**
- UUID primary keys for all entities
- Soft deletes via `deleted_at` column
- `created_at` / `updated_at` timestamps on all tables
- Foreign key constraints for referential integrity
- Composite indexes for common query patterns
- JSONB columns for flexible attribute storage

---

## B01 - System Core

### users

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK, DEFAULT gen_random_uuid() | Unique identifier |
| phone | VARCHAR(20) | UNIQUE, NOT NULL | Phone number (primary contact) |
| email | VARCHAR(255) | UNIQUE | Email address |
| password_hash | VARCHAR(255) | NOT NULL | bcrypt hashed password |
| role | ENUM | NOT NULL, DEFAULT 'customer' | customer, vendor, admin, super_admin |
| status | ENUM | NOT NULL, DEFAULT 'active' | active, suspended, banned |
| first_name | VARCHAR(100) | | First name |
| last_name | VARCHAR(100) | | Last name |
| avatar_url | VARCHAR(500) | | Profile picture URL |
| locale | VARCHAR(5) | DEFAULT 'ar' | Preferred language |
| last_login_at | TIMESTAMP | | Last login timestamp |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Record creation |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Last update |
| deleted_at | TIMESTAMP | | Soft delete marker |

**Indexes:**
```sql
CREATE UNIQUE INDEX idx_users_phone ON users(phone) WHERE deleted_at IS NULL;
CREATE UNIQUE INDEX idx_users_email ON users(email) WHERE deleted_at IS NULL;
CREATE INDEX idx_users_role_status ON users(role, status);
CREATE INDEX idx_users_created_at ON users(created_at);
```

### sessions

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Session identifier |
| user_id | UUID | FK -> users(id), NOT NULL | Associated user |
| token | VARCHAR(500) | UNIQUE, NOT NULL | JWT or session token |
| ip_address | INET | | Client IP address |
| user_agent | TEXT | | User agent string |
| expires_at | TIMESTAMP | NOT NULL | Token expiration |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Session creation |

**Indexes:**
```sql
CREATE INDEX idx_sessions_user_id ON sessions(user_id);
CREATE INDEX idx_sessions_token ON sessions(token);
CREATE INDEX idx_sessions_expires_at ON sessions(expires_at);
```

### otp_codes

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | OTP record identifier |
| phone | VARCHAR(20) | NOT NULL | Target phone number |
| code | VARCHAR(6) | NOT NULL | OTP code |
| purpose | ENUM | NOT NULL | login, registration, password_reset, phone_verify |
| attempts | INTEGER | DEFAULT 0, NOT NULL | Verification attempts |
| max_attempts | INTEGER | DEFAULT 3, NOT NULL | Maximum allowed attempts |
| expires_at | TIMESTAMP | NOT NULL | OTP expiration |
| verified_at | TIMESTAMP | | Verification timestamp |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Record creation |

**Indexes:**
```sql
CREATE INDEX idx_otp_phone_purpose ON otp_codes(phone, purpose);
CREATE INDEX idx_otp_expires_at ON otp_codes(expires_at);
```

### email_verifications

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Verification identifier |
| user_id | UUID | FK -> users(id), NOT NULL | Associated user |
| email | VARCHAR(255) | NOT NULL | Email to verify |
| token | VARCHAR(255) | UNIQUE, NOT NULL | Verification token |
| expires_at | TIMESTAMP | NOT NULL | Token expiration |
| verified_at | TIMESTAMP | | Verification timestamp |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Record creation |

**Indexes:**
```sql
CREATE INDEX idx_email_verify_user ON email_verifications(user_id);
CREATE INDEX idx_email_verify_token ON email_verifications(token);
```

### audit_logs

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Log identifier |
| user_id | UUID | FK -> users(id) | Acting user (NULL for system) |
| action | VARCHAR(100) | NOT NULL | Action performed |
| entity_type | VARCHAR(50) | | Entity type affected |
| entity_id | UUID | | Entity ID affected |
| details | JSONB | | Action details |
| ip_address | INET | | Client IP |
| user_agent | TEXT | | User agent |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Log timestamp |

**Indexes:**
```sql
CREATE INDEX idx_audit_user ON audit_logs(user_id);
CREATE INDEX idx_audit_entity ON audit_logs(entity_type, entity_id);
CREATE INDEX idx_audit_action ON audit_logs(action);
CREATE INDEX idx_audit_created ON audit_logs(created_at);
```

---

## B02 - Marketplace

### vendors

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Vendor identifier |
| user_id | UUID | FK -> users(id), UNIQUE, NOT NULL | Owner user |
| store_name | VARCHAR(200) | NOT NULL | Store display name |
| store_name_ar | VARCHAR(200) | NOT NULL | Arabic store name |
| store_slug | VARCHAR(200) | UNIQUE, NOT NULL | URL-friendly slug |
| description | TEXT | | Store description |
| logo_url | VARCHAR(500) | | Store logo |
| banner_url | VARCHAR(500) | | Store banner |
| status | ENUM | NOT NULL, DEFAULT 'pending' | pending, active, suspended, banned |
| kyc_status | ENUM | DEFAULT 'none' | none, pending, verified, rejected |
| commission_rate | DECIMAL(5,2) | DEFAULT 10.00 | Commission percentage |
| rating | DECIMAL(3,2) | DEFAULT 0 | Average rating |
| total_sales | BIGINT | DEFAULT 0 | Total sales count |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Registration date |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Last update |
| deleted_at | TIMESTAMP | | Soft delete marker |

**Indexes:**
```sql
CREATE UNIQUE INDEX idx_vendors_slug ON vendors(store_slug) WHERE deleted_at IS NULL;
CREATE INDEX idx_vendors_user ON vendors(user_id);
CREATE INDEX idx_vendors_status ON vendors(status);
CREATE INDEX idx_vendors_kyc ON vendors(kyc_status);
```

### stores

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Store identifier |
| vendor_id | UUID | FK -> vendors(id), NOT NULL | Associated vendor |
| template_id | UUID | FK -> store_templates(id) | Selected template |
| settings | JSONB | DEFAULT '{}' | Store configuration |
| status | ENUM | NOT NULL, DEFAULT 'active' | active, maintenance |
| custom_domain | VARCHAR(255) | UNIQUE | Custom domain |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Record creation |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Last update |

**Indexes:**
```sql
CREATE INDEX idx_stores_vendor ON stores(vendor_id);
CREATE INDEX idx_stores_domain ON stores(custom_domain) WHERE custom_domain IS NOT NULL;
```

### store_templates

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Template identifier |
| name | VARCHAR(100) | NOT NULL | Template name |
| description | TEXT | | Template description |
| preview_url | VARCHAR(500) | | Preview image |
| config | JSONB | NOT NULL | Template configuration |
| status | ENUM | DEFAULT 'active' | active, deprecated |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Record creation |

### kyc_documents

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Document identifier |
| vendor_id | UUID | FK -> vendors(id), NOT NULL | Associated vendor |
| type | ENUM | NOT NULL | national_id, commercial_register, tax_cert, bank_statement |
| file_url | VARCHAR(500) | NOT NULL | Document file URL |
| file_hash | VARCHAR(64) | NOT NULL | SHA-256 hash |
| status | ENUM | NOT NULL, DEFAULT 'pending' | pending, approved, rejected |
| reviewed_by | UUID | FK -> users(id) | Reviewer |
| review_notes | TEXT | | Review comments |
| reviewed_at | TIMESTAMP | | Review timestamp |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Upload date |

**Indexes:**
```sql
CREATE INDEX idx_kyc_vendor ON kyc_documents(vendor_id);
CREATE INDEX idx_kyc_status ON kyc_documents(status);
```

### vendor_badges

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Badge identifier |
| vendor_id | UUID | FK -> vendors(id), NOT NULL | Associated vendor |
| badge_type | ENUM | NOT NULL | top_seller, fast_shipper, verified, new_arrival |
| description | TEXT | | Badge description |
| awarded_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Award date |
| expires_at | TIMESTAMP | | Expiration date |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Record creation |

**Indexes:**
```sql
CREATE INDEX idx_badges_vendor ON vendor_badges(vendor_id);
CREATE INDEX idx_badges_type ON vendor_badges(badge_type);
```

---

## B03 - Products

### products

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Product identifier |
| vendor_id | UUID | FK -> vendors(id), NOT NULL | Owner vendor |
| category_id | UUID | FK -> categories(id), NOT NULL | Product category |
| name_ar | VARCHAR(300) | NOT NULL | Arabic name |
| name_en | VARCHAR(300) | NOT NULL | English name |
| slug | VARCHAR(350) | UNIQUE, NOT NULL | URL-friendly slug |
| description_ar | TEXT | | Arabic description |
| description_en | TEXT | | English description |
| status | ENUM | NOT NULL, DEFAULT 'draft' | draft, active, archived, banned |
| score | INTEGER | DEFAULT 0 | Search ranking score |
| views_count | BIGINT | DEFAULT 0 | View counter |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Creation date |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Last update |
| deleted_at | TIMESTAMP | | Soft delete marker |

**Indexes:**
```sql
CREATE UNIQUE INDEX idx_products_slug ON products(slug) WHERE deleted_at IS NULL;
CREATE INDEX idx_products_vendor ON products(vendor_id);
CREATE INDEX idx_products_category ON products(category_id);
CREATE INDEX idx_products_status ON products(status);
CREATE INDEX idx_products_score ON products(score DESC);
CREATE INDEX idx_products_created ON products(created_at DESC);
```

### product_variants

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Variant identifier |
| product_id | UUID | FK -> products(id), NOT NULL | Parent product |
| sku | VARCHAR(50) | UNIQUE, NOT NULL | Stock keeping unit |
| name | VARCHAR(200) | | Variant display name |
| price | DECIMAL(12,2) | NOT NULL | Selling price |
| compare_at_price | DECIMAL(12,2) | | Original price (for discounts) |
| cost_price | DECIMAL(12,2) | | Vendor cost price |
| stock_quantity | INTEGER | DEFAULT 0 | Available stock |
| weight | DECIMAL(8,2) | | Weight in grams |
| attributes | JSONB | DEFAULT '{}' | Variant attributes (color, size, etc.) |
| barcode | VARCHAR(100) | | Barcode |
| status | ENUM | DEFAULT 'active' | active, inactive, out_of_stock |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Creation date |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Last update |

**Indexes:**
```sql
CREATE INDEX idx_variants_product ON product_variants(product_id);
CREATE INDEX idx_variants_sku ON product_variants(sku);
CREATE INDEX idx_variants_status ON product_variants(status);
```

### categories

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Category identifier |
| parent_id | UUID | FK -> categories(id) | Parent category (NULL for root) |
| name_ar | VARCHAR(200) | NOT NULL | Arabic name |
| name_en | VARCHAR(200) | NOT NULL | English name |
| slug | VARCHAR(250) | UNIQUE, NOT NULL | URL-friendly slug |
| icon_url | VARCHAR(500) | | Category icon |
| image_url | VARCHAR(500) | | Category image |
| level | INTEGER | DEFAULT 0 | Hierarchy level |
| sort_order | INTEGER | DEFAULT 0 | Display order |
| status | ENUM | DEFAULT 'active' | active, hidden |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Creation date |

**Indexes:**
```sql
CREATE UNIQUE INDEX idx_categories_slug ON categories(slug);
CREATE INDEX idx_categories_parent ON categories(parent_id);
CREATE INDEX idx_categories_level ON categories(level, sort_order);
```

### product_media

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Media identifier |
| product_id | UUID | FK -> products(id), NOT NULL | Associated product |
| type | ENUM | NOT NULL | image, video, 3d_model |
| url | VARCHAR(500) | NOT NULL | Media URL |
| alt_text | VARCHAR(200) | | Alt text for accessibility |
| file_size | INTEGER | | File size in bytes |
| sort_order | INTEGER | DEFAULT 0 | Display order |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Upload date |

**Indexes:**
```sql
CREATE INDEX idx_media_product ON product_media(product_id);
CREATE INDEX idx_media_sort ON product_media(product_id, sort_order);
```

### offers

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Offer identifier |
| vendor_id | UUID | FK -> vendors(id), NOT NULL | Creating vendor |
| product_id | UUID | FK -> products(id) | Specific product (NULL for store-wide) |
| type | ENUM | NOT NULL | flash_sale, seasonal, clearance, bundle |
| discount_percent | DECIMAL(5,2) | NOT NULL | Discount percentage |
| start_date | TIMESTAMP | NOT NULL | Offer start |
| end_date | TIMESTAMP | NOT NULL | Offer end |
| status | ENUM | DEFAULT 'draft' | draft, active, expired, cancelled |
| max_quantity | INTEGER | | Maximum units per customer |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Creation date |

**Indexes:**
```sql
CREATE INDEX idx_offers_vendor ON offers(vendor_id);
CREATE INDEX idx_offers_dates ON offers(start_date, end_date);
CREATE INDEX idx_offers_status ON offers(status);
```

---

## B04 - Orders

### master_orders

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Master order identifier |
| customer_id | UUID | FK -> users(id), NOT NULL | Customer |
| order_number | VARCHAR(20) | UNIQUE, NOT NULL | Human-readable order number |
| status | ENUM | NOT NULL, DEFAULT 'pending' | pending, confirmed, processing, shipped, delivered, cancelled, refunded |
| subtotal | DECIMAL(12,2) | NOT NULL | Order subtotal |
| shipping_total | DECIMAL(12,2) | DEFAULT 0 | Total shipping cost |
| discount_total | DECIMAL(12,2) | DEFAULT 0 | Total discount |
| tax_total | DECIMAL(12,2) | DEFAULT 0 | Total tax |
| total | DECIMAL(12,2) | NOT NULL | Grand total |
| currency | VARCHAR(3) | DEFAULT 'YER' | Currency code |
| payment_method | VARCHAR(50) | | Payment method used |
| shipping_address | JSONB | NOT NULL | Delivery address |
| billing_address | JSONB | | Billing address |
| notes | TEXT | | Customer notes |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Order date |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Last update |

**Indexes:**
```sql
CREATE UNIQUE INDEX idx_master_orders_number ON master_orders(order_number);
CREATE INDEX idx_master_orders_customer ON master_orders(customer_id);
CREATE INDEX idx_master_orders_status ON master_orders(status);
CREATE INDEX idx_master_orders_created ON master_orders(created_at DESC);
```

### sub_orders

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Sub-order identifier |
| master_order_id | UUID | FK -> master_orders(id), NOT NULL | Parent order |
| vendor_id | UUID | FK -> vendors(id), NOT NULL | Vendor |
| status | ENUM | NOT NULL, DEFAULT 'pending' | pending, confirmed, processing, shipped, delivered, cancelled |
| subtotal | DECIMAL(12,2) | NOT NULL | Vendor subtotal |
| shipping_total | DECIMAL(12,2) | DEFAULT 0 | Vendor shipping |
| discount_total | DECIMAL(12,2) | DEFAULT 0 | Vendor discount |
| total | DECIMAL(12,2) | NOT NULL | Vendor total |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Creation date |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Last update |

**Indexes:**
```sql
CREATE INDEX idx_sub_orders_master ON sub_orders(master_order_id);
CREATE INDEX idx_sub_orders_vendor ON sub_orders(vendor_id);
CREATE INDEX idx_sub_orders_status ON sub_orders(status);
```

### order_items

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Order item identifier |
| sub_order_id | UUID | FK -> sub_orders(id), NOT NULL | Parent sub-order |
| product_variant_id | UUID | FK -> product_variants(id), NOT NULL | Product variant |
| product_name | VARCHAR(300) | NOT NULL | Snapshot of product name |
| variant_snapshot | JSONB | NOT NULL | Variant details snapshot |
| quantity | INTEGER | NOT NULL, CHECK > 0 | Quantity ordered |
| unit_price | DECIMAL(12,2) | NOT NULL | Price at time of order |
| discount_amount | DECIMAL(12,2) | DEFAULT 0 | Item discount |
| total | DECIMAL(12,2) | NOT NULL | Line total |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Creation date |

**Indexes:**
```sql
CREATE INDEX idx_order_items_sub ON order_items(sub_order_id);
CREATE INDEX idx_order_items_variant ON order_items(product_variant_id);
```

### return_requests

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Return request identifier |
| order_item_id | UUID | FK -> order_items(id), NOT NULL | Order item to return |
| customer_id | UUID | FK -> users(id), NOT NULL | Requesting customer |
| reason | TEXT | NOT NULL | Return reason |
| reason_type | ENUM | NOT NULL | defective, wrong_item, not_as_described, changed_mind |
| status | ENUM | NOT NULL, DEFAULT 'pending' | pending, approved, rejected, received, refunded |
| resolution_notes | TEXT | | Admin/vendor notes |
| refund_amount | DECIMAL(12,2) | | Refund amount if approved |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Request date |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Last update |

**Indexes:**
```sql
CREATE INDEX idx_returns_item ON return_requests(order_item_id);
CREATE INDEX idx_returns_customer ON return_requests(customer_id);
CREATE INDEX idx_returns_status ON return_requests(status);
```

---

## B05 - Payments

### wallets

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Wallet identifier |
| user_id | UUID | FK -> users(id), NOT NULL | Wallet owner |
| currency | VARCHAR(3) | NOT NULL, DEFAULT 'YER' | Currency code |
| balance | DECIMAL(14,2) | DEFAULT 0, NOT NULL | Current balance |
| locked_balance | DECIMAL(14,2) | DEFAULT 0 | Balance in escrow |
| status | ENUM | DEFAULT 'active' | active, frozen, closed |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Creation date |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Last update |

**Constraints:**
```sql
ALTER TABLE wallets ADD CONSTRAINT chk_balance_non_negative CHECK (balance >= 0);
ALTER TABLE wallets ADD CONSTRAINT chk_locked_non_negative CHECK (locked_balance >= 0);
```

**Indexes:**
```sql
CREATE UNIQUE INDEX idx_wallets_user_currency ON wallets(user_id, currency);
```

### wallet_transactions

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Transaction identifier |
| wallet_id | UUID | FK -> wallets(id), NOT NULL | Associated wallet |
| type | ENUM | NOT NULL | credit, debit, hold, release |
| amount | DECIMAL(14,2) | NOT NULL | Transaction amount |
| balance_after | DECIMAL(14,2) | NOT NULL | Balance after transaction |
| reference_type | VARCHAR(50) | | Reference entity type |
| reference_id | UUID | | Reference entity ID |
| description | TEXT | | Transaction description |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Transaction time |

**Indexes:**
```sql
CREATE INDEX idx_wallet_tx_wallet ON wallet_transactions(wallet_id);
CREATE INDEX idx_wallet_tx_type ON wallet_transactions(type);
CREATE INDEX idx_wallet_tx_created ON wallet_transactions(created_at DESC);
```

### wallet_topups

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Topup identifier |
| wallet_id | UUID | FK -> wallets(id), NOT NULL | Target wallet |
| amount | DECIMAL(14,2) | NOT NULL, CHECK > 0 | Topup amount |
| provider | VARCHAR(50) | NOT NULL | Payment provider (e.g., jamiepay, qcash) |
| provider_ref | VARCHAR(200) | | Provider transaction reference |
| status | ENUM | NOT NULL, DEFAULT 'pending' | pending, completed, failed, reversed |
| callback_data | JSONB | | Provider callback payload |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Request date |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Last update |

**Indexes:**
```sql
CREATE INDEX idx_topup_wallet ON wallet_topups(wallet_id);
CREATE INDEX idx_topup_status ON wallet_topups(status);
CREATE INDEX idx_topup_provider_ref ON wallet_topups(provider_ref);
```

### escrow_transactions

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Escrow identifier |
| sub_order_id | UUID | FK -> sub_orders(id), NOT NULL | Associated sub-order |
| amount | DECIMAL(14,2) | NOT NULL | Escrow amount |
| status | ENUM | NOT NULL, DEFAULT 'held' | held, released, disputed, refunded |
| held_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Funds held timestamp |
| released_at | TIMESTAMP | | Funds released timestamp |
| release_reason | TEXT | | Reason for release |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Creation date |

**Indexes:**
```sql
CREATE INDEX idx_escrow_sub_order ON escrow_transactions(sub_order_id);
CREATE INDEX idx_escrow_status ON escrow_transactions(status);
```

### refunds

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Refund identifier |
| escrow_id | UUID | FK -> escrow_transactions(id) | Related escrow |
| wallet_id | UUID | FK -> wallets(id) | Target wallet for refund |
| amount | DECIMAL(14,2) | NOT NULL, CHECK > 0 | Refund amount |
| reason | TEXT | NOT NULL | Refund reason |
| status | ENUM | NOT NULL, DEFAULT 'pending' | pending, approved, processed, rejected |
| approved_by | UUID | FK -> users(id) | Admin who approved |
| processed_at | TIMESTAMP | | Processing timestamp |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Request date |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Last update |

**Indexes:**
```sql
CREATE INDEX idx_refunds_escrow ON refunds(escrow_id);
CREATE INDEX idx_refunds_wallet ON refunds(wallet_id);
CREATE INDEX idx_refunds_status ON refunds(status);
```

---

## B06 - Finance

### commissions

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Commission identifier |
| vendor_id | UUID | FK -> vendors(id), NOT NULL | Vendor |
| sub_order_id | UUID | FK -> sub_orders(id), NOT NULL | Related sub-order |
| amount | DECIMAL(14,2) | NOT NULL | Commission amount |
| rate | DECIMAL(5,2) | NOT NULL | Commission rate (%) |
| status | ENUM | DEFAULT 'pending' | pending, charged, waived |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Calculation date |

**Indexes:**
```sql
CREATE INDEX idx_commissions_vendor ON commissions(vendor_id);
CREATE INDEX idx_commissions_sub_order ON commissions(sub_order_id);
CREATE INDEX idx_commissions_status ON commissions(status);
```

### payouts

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Payout identifier |
| vendor_id | UUID | FK -> vendors(id), NOT NULL | Vendor |
| amount | DECIMAL(14,2) | NOT NULL, CHECK > 0 | Payout amount |
| method | VARCHAR(50) | NOT NULL | bank_transfer, wallet |
| reference | VARCHAR(200) | | Payment reference |
| status | ENUM | NOT NULL, DEFAULT 'pending' | pending, processing, completed, failed |
| period_start | TIMESTAMP | NOT NULL | Payout period start |
| period_end | TIMESTAMP | NOT NULL | Payout period end |
| processed_at | TIMESTAMP | | Processing timestamp |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Creation date |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Last update |

**Indexes:**
```sql
CREATE INDEX idx_payouts_vendor ON payouts(vendor_id);
CREATE INDEX idx_payouts_status ON payouts(status);
CREATE INDEX idx_payouts_period ON payouts(period_start, period_end);
```

### invoices

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Invoice identifier |
| sub_order_id | UUID | FK -> sub_orders(id), NOT NULL | Related sub-order |
| invoice_number | VARCHAR(30) | UNIQUE, NOT NULL | Sequential invoice number |
| qr_code | TEXT | | QR code data |
| signature | TEXT | | Digital signature |
| issued_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Issue date |
| due_at | TIMESTAMP | | Due date |
| status | ENUM | DEFAULT 'issued' | issued, paid, void |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Creation date |

**Indexes:**
```sql
CREATE UNIQUE INDEX idx_invoices_number ON invoices(invoice_number);
CREATE INDEX idx_invoices_sub_order ON invoices(sub_order_id);
```

### ledger_entries

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Entry identifier |
| account_id | VARCHAR(50) | NOT NULL | Chart of accounts code |
| debit | DECIMAL(14,2) | DEFAULT 0 | Debit amount |
| credit | DECIMAL(14,2) | DEFAULT 0 | Credit amount |
| reference_type | VARCHAR(50) | NOT NULL | Source entity type |
| reference_id | UUID | NOT NULL | Source entity ID |
| description | TEXT | | Entry description |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Entry date |

**Constraints:**
```sql
ALTER TABLE ledger_entries ADD CONSTRAINT chk_ledger_balance CHECK (
    (debit > 0 AND credit = 0) OR (credit > 0 AND debit = 0)
);
```

**Indexes:**
```sql
CREATE INDEX idx_ledger_account ON ledger_entries(account_id);
CREATE INDEX idx_ledger_reference ON ledger_entries(reference_type, reference_id);
CREATE INDEX idx_ledger_created ON ledger_entries(created_at);
```

### tax_records

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Tax record identifier |
| order_id | UUID | FK -> master_orders(id), NOT NULL | Related order |
| tax_type | VARCHAR(50) | NOT NULL | VAT, sales_tax, import_duty |
| amount | DECIMAL(14,2) | NOT NULL | Tax amount |
| rate | DECIMAL(5,2) | NOT NULL | Tax rate (%) |
| jurisdiction | VARCHAR(100) | | Tax jurisdiction |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Record date |

**Indexes:**
```sql
CREATE INDEX idx_tax_order ON tax_records(order_id);
CREATE INDEX idx_tax_type ON tax_records(tax_type);
```

---

## B07 - Shipping

### delivery_providers

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Provider identifier |
| name | VARCHAR(100) | NOT NULL | Provider name |
| api_key | VARCHAR(500) | | API credentials |
| api_endpoint | VARCHAR(500) | | API base URL |
| status | ENUM | DEFAULT 'active' | active, inactive |
| rating | DECIMAL(3,2) | DEFAULT 0 | Average rating |
| config | JSONB | DEFAULT '{}' | Provider-specific configuration |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Registration date |

### delivery_zones

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Zone identifier |
| provider_id | UUID | FK -> delivery_providers(id), NOT NULL | Provider |
| name | VARCHAR(100) | NOT NULL | Zone name |
| name_ar | VARCHAR(100) | NOT NULL | Arabic zone name |
| coordinates | JSONB | | GeoJSON polygon |
| base_price | DECIMAL(10,2) | NOT NULL | Base delivery price |
| per_km_price | DECIMAL(10,2) | | Price per km |
| estimated_hours | INTEGER | | Estimated delivery time |
| status | ENUM | DEFAULT 'active' | active, inactive |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Creation date |

**Indexes:**
```sql
CREATE INDEX idx_zones_provider ON delivery_zones(provider_id);
```

### delivery_assignments

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Assignment identifier |
| sub_order_id | UUID | FK -> sub_orders(id), NOT NULL | Related sub-order |
| provider_id | UUID | FK -> delivery_providers(id), NOT NULL | Delivery provider |
| driver_name | VARCHAR(100) | | Assigned driver |
| driver_phone | VARCHAR(20) | | Driver phone |
| tracking_number | VARCHAR(100) | UNIQUE | Tracking number |
| status | ENUM | NOT NULL, DEFAULT 'assigned' | assigned, picked_up, in_transit, out_for_delivery, delivered, failed |
| estimated_delivery | TIMESTAMP | | Estimated delivery time |
| actual_delivery | TIMESTAMP | | Actual delivery time |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Assignment date |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Last update |

**Indexes:**
```sql
CREATE INDEX idx_delivery_sub_order ON delivery_assignments(sub_order_id);
CREATE INDEX idx_delivery_tracking ON delivery_assignments(tracking_number);
CREATE INDEX idx_delivery_status ON delivery_assignments(status);
```

### delivery_status_logs

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Log identifier |
| assignment_id | UUID | FK -> delivery_assignments(id), NOT NULL | Related assignment |
| status | VARCHAR(50) | NOT NULL | Status update |
| location | JSONB | | GPS coordinates {lat, lng} |
| notes | TEXT | | Status notes |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Log timestamp |

**Indexes:**
```sql
CREATE INDEX idx_delivery_logs_assignment ON delivery_status_logs(assignment_id);
```

### proof_of_delivery

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Proof identifier |
| assignment_id | UUID | FK -> delivery_assignments(id), NOT NULL | Related assignment |
| type | ENUM | NOT NULL | photo, signature, pin_code |
| file_url | VARCHAR(500) | | File URL (for photo) |
| signature_data | TEXT | | Base64 signature |
| pin_code | VARCHAR(10) | | PIN verification code |
| recipient_name | VARCHAR(200) | | Name of person who received |
| delivered_at | TIMESTAMP | NOT NULL | Delivery timestamp |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Record creation |

**Indexes:**
```sql
CREATE INDEX idx_pod_assignment ON proof_of_delivery(assignment_id);
```

---

## B08 - Inventory

### stock

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Stock record identifier |
| product_variant_id | UUID | FK -> product_variants(id), NOT NULL | Product variant |
| warehouse_id | UUID | FK -> warehouses(id), NOT NULL | Warehouse |
| quantity | INTEGER | NOT NULL, DEFAULT 0, CHECK >= 0 | Available quantity |
| reserved | INTEGER | NOT NULL, DEFAULT 0, CHECK >= 0 | Reserved quantity |
| reorder_level | INTEGER | | Minimum stock threshold |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Creation date |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Last update |

**Constraints:**
```sql
ALTER TABLE stock ADD CONSTRAINT chk_reserved_lte_quantity CHECK (reserved <= quantity);
```

**Indexes:**
```sql
CREATE UNIQUE INDEX idx_stock_variant_warehouse ON stock(product_variant_id, warehouse_id);
CREATE INDEX idx_stock_warehouse ON stock(warehouse_id);
```

### stock_reservations

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Reservation identifier |
| stock_id | UUID | FK -> stock(id), NOT NULL | Stock record |
| order_item_id | UUID | FK -> order_items(id) | Reserved for order item |
| quantity | INTEGER | NOT NULL, CHECK > 0 | Reserved quantity |
| expires_at | TIMESTAMP | NOT NULL | Reservation expiry |
| status | ENUM | DEFAULT 'active' | active, fulfilled, expired, cancelled |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Reservation date |

**Indexes:**
```sql
CREATE INDEX idx_stock_res_stock ON stock_reservations(stock_id);
CREATE INDEX idx_stock_res_expires ON stock_reservations(expires_at) WHERE status = 'active';
```

### warehouses

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Warehouse identifier |
| name | VARCHAR(100) | NOT NULL | Warehouse name |
| location | JSONB | | Location details {address, lat, lng} |
| capacity | INTEGER | | Total capacity |
| status | ENUM | DEFAULT 'active' | active, maintenance, closed |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Creation date |

### purchase_orders

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Purchase order identifier |
| vendor_id | UUID | FK -> vendors(id), NOT NULL | Vendor placing order |
| supplier_name | VARCHAR(200) | | Supplier name |
| status | ENUM | NOT NULL, DEFAULT 'draft' | draft, submitted, confirmed, received, cancelled |
| total | DECIMAL(14,2) | | Order total |
| items | JSONB | NOT NULL | Order line items |
| expected_date | TIMESTAMP | | Expected delivery date |
| received_at | TIMESTAMP | | Actual receipt timestamp |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Creation date |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Last update |

**Indexes:**
```sql
CREATE INDEX idx_po_vendor ON purchase_orders(vendor_id);
CREATE INDEX idx_po_status ON purchase_orders(status);
```

---

## B09 - Storefront

### cart_items

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Cart item identifier |
| user_id | UUID | FK -> users(id), NOT NULL | Cart owner |
| product_variant_id | UUID | FK -> product_variants(id), NOT NULL | Product variant |
| quantity | INTEGER | NOT NULL, CHECK > 0 | Quantity |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Added date |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Last update |

**Indexes:**
```sql
CREATE UNIQUE INDEX idx_cart_user_variant ON cart_items(user_id, product_variant_id);
CREATE INDEX idx_cart_user ON cart_items(user_id);
```

### wishlists

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Wishlist entry identifier |
| user_id | UUID | FK -> users(id), NOT NULL | User |
| product_id | UUID | FK -> products(id), NOT NULL | Product |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Added date |

**Indexes:**
```sql
CREATE UNIQUE INDEX idx_wishlist_user_product ON wishlists(user_id, product_id);
CREATE INDEX idx_wishlist_user ON wishlists(user_id);
```

### store_follows

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Follow identifier |
| user_id | UUID | FK -> users(id), NOT NULL | Follower |
| vendor_id | UUID | FK -> vendors(id), NOT NULL | Followed vendor |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Follow date |

**Indexes:**
```sql
CREATE UNIQUE INDEX idx_follow_user_vendor ON store_follows(user_id, vendor_id);
CREATE INDEX idx_follow_vendor ON store_follows(vendor_id);
```

---

## B10 - Reviews

### reviews

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Review identifier |
| customer_id | UUID | FK -> users(id), NOT NULL | Reviewer |
| product_id | UUID | FK -> products(id), NOT NULL | Product reviewed |
| order_item_id | UUID | FK -> order_items(id), NOT NULL | Verified purchase |
| rating | INTEGER | NOT NULL, CHECK (1-5) | Star rating |
| title | VARCHAR(200) | | Review title |
| text | TEXT | | Review body |
| images | JSONB | DEFAULT '[]' | Review image URLs |
| vendor_reply | TEXT | | Vendor response |
| status | ENUM | DEFAULT 'pending' | pending, approved, rejected, flagged |
| helpful_count | INTEGER | DEFAULT 0 | Helpful votes |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Review date |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Last update |

**Constraints:**
```sql
ALTER TABLE reviews ADD CONSTRAINT chk_rating_range CHECK (rating >= 1 AND rating <= 5);
```

**Indexes:**
```sql
CREATE INDEX idx_reviews_product ON reviews(product_id);
CREATE INDEX idx_reviews_customer ON reviews(customer_id);
CREATE INDEX idx_reviews_status ON reviews(status);
CREATE INDEX idx_reviews_rating ON reviews(product_id, rating);
```

### vendor_ratings

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Rating identifier |
| vendor_id | UUID | FK -> vendors(id), NOT NULL | Vendor |
| customer_id | UUID | FK -> users(id), NOT NULL | Rater |
| order_id | UUID | FK -> master_orders(id), NOT NULL | Related order |
| rating | INTEGER | NOT NULL, CHECK (1-5) | Overall rating |
| communication | INTEGER | CHECK (1-5) | Communication rating |
| shipping_speed | INTEGER | CHECK (1-5) | Shipping speed rating |
| product_quality | INTEGER | CHECK (1-5) | Product quality rating |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Rating date |

**Indexes:**
```sql
CREATE INDEX idx_vendor_ratings_vendor ON vendor_ratings(vendor_id);
CREATE UNIQUE INDEX idx_vendor_ratings_order ON vendor_ratings(customer_id, order_id);
```

### loyalty_points

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Loyalty identifier |
| user_id | UUID | FK -> users(id), UNIQUE, NOT NULL | User |
| balance | INTEGER | DEFAULT 0, NOT NULL, CHECK >= 0 | Available points |
| total_earned | INTEGER | DEFAULT 0 | Lifetime earned |
| total_redeemed | INTEGER | DEFAULT 0 | Lifetime redeemed |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Creation date |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Last update |

### loyalty_transactions

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Transaction identifier |
| user_id | UUID | FK -> users(id), NOT NULL | User |
| type | ENUM | NOT NULL | earned, redeemed, expired, adjusted |
| points | INTEGER | NOT NULL | Points (positive=earn, negative=redeem) |
| balance_after | INTEGER | NOT NULL | Balance after transaction |
| reference_type | VARCHAR(50) | | Source entity type |
| reference_id | UUID | | Source entity ID |
| description | TEXT | | Transaction description |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Transaction time |

**Indexes:**
```sql
CREATE INDEX idx_loyalty_tx_user ON loyalty_transactions(user_id);
CREATE INDEX idx_loyalty_tx_type ON loyalty_transactions(type);
```

### loyalty_tiers

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Tier identifier |
| user_id | UUID | FK -> users(id), NOT NULL | User |
| tier | ENUM | NOT NULL | bronze, silver, gold, platinum |
| upgraded_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Upgrade date |
| points_required | INTEGER | | Points needed for tier |
| benefits | JSONB | | Tier benefits |

**Indexes:**
```sql
CREATE INDEX idx_loyalty_tier_user ON loyalty_tiers(user_id);
CREATE INDEX idx_loyalty_tier_level ON loyalty_tiers(tier);
```

---

## B11 - Content

### pages

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Page identifier |
| title_ar | VARCHAR(200) | NOT NULL | Arabic title |
| title_en | VARCHAR(200) | NOT NULL | English title |
| slug | VARCHAR(250) | UNIQUE, NOT NULL | URL slug |
| content_ar | TEXT | | Arabic content (HTML) |
| content_en | TEXT | | English content (HTML) |
| meta_title | VARCHAR(200) | | SEO meta title |
| meta_description | TEXT | | SEO meta description |
| status | ENUM | DEFAULT 'draft' | draft, published, archived |
| published_at | TIMESTAMP | | Publication date |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Creation date |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Last update |

**Indexes:**
```sql
CREATE UNIQUE INDEX idx_pages_slug ON pages(slug);
CREATE INDEX idx_pages_status ON pages(status);
```

### banners

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Banner identifier |
| title | VARCHAR(200) | NOT NULL | Banner title |
| image_url | VARCHAR(500) | NOT NULL | Banner image |
| mobile_image_url | VARCHAR(500) | | Mobile-specific image |
| link_url | VARCHAR(500) | | Click destination URL |
| position | ENUM | NOT NULL | hero, category, sidebar, footer |
| sort_order | INTEGER | DEFAULT 0 | Display order |
| status | ENUM | DEFAULT 'draft' | draft, active, inactive |
| start_date | TIMESTAMP | NOT NULL | Display start |
| end_date | TIMESTAMP | NOT NULL | Display end |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Creation date |

**Indexes:**
```sql
CREATE INDEX idx_banners_position ON banners(position, sort_order);
CREATE INDEX idx_banners_dates ON banners(start_date, end_date);
```

### notifications

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Notification identifier |
| user_id | UUID | FK -> users(id), NOT NULL | Recipient |
| template_id | UUID | FK -> notification_templates(id) | Template used |
| type | VARCHAR(50) | NOT NULL | Notification type |
| title | VARCHAR(200) | NOT NULL | Notification title |
| body | TEXT | NOT NULL | Notification body |
| data | JSONB | | Additional payload |
| channel | ENUM | NOT NULL | in_app, push, sms, email |
| read_at | TIMESTAMP | | Read timestamp |
| sent_at | TIMESTAMP | | Send timestamp |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Creation date |

**Indexes:**
```sql
CREATE INDEX idx_notif_user ON notifications(user_id);
CREATE INDEX idx_notif_unread ON notifications(user_id, read_at) WHERE read_at IS NULL;
CREATE INDEX idx_notif_type ON notifications(type);
```

### notification_templates

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Template identifier |
| type | VARCHAR(50) | NOT NULL | Template type |
| channel | ENUM | NOT NULL | in_app, push, sms, email |
| subject | VARCHAR(200) | | Template subject |
| body | TEXT | NOT NULL | Template body with placeholders |
| status | ENUM | DEFAULT 'active' | active, inactive |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Creation date |

**Indexes:**
```sql
CREATE UNIQUE INDEX idx_notif_template_type_channel ON notification_templates(type, channel);
```

---

## B12 - Support

### support_tickets

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Ticket identifier |
| user_id | UUID | FK -> users(id), NOT NULL | Ticket creator |
| order_id | UUID | FK -> master_orders(id) | Related order |
| category | VARCHAR(50) | NOT NULL | Ticket category |
| priority | ENUM | DEFAULT 'medium' | low, medium, high, urgent |
| status | ENUM | NOT NULL, DEFAULT 'open' | open, in_progress, waiting, resolved, closed |
| subject | VARCHAR(200) | NOT NULL | Ticket subject |
| assigned_to | UUID | FK -> users(id) | Assigned agent |
| resolved_at | TIMESTAMP | | Resolution timestamp |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Creation date |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Last update |

**Indexes:**
```sql
CREATE INDEX idx_tickets_user ON support_tickets(user_id);
CREATE INDEX idx_tickets_status ON support_tickets(status);
CREATE INDEX idx_tickets_assigned ON support_tickets(assigned_to);
CREATE INDEX idx_tickets_priority ON support_tickets(priority);
```

### ticket_messages

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Message identifier |
| ticket_id | UUID | FK -> support_tickets(id), NOT NULL | Parent ticket |
| sender_id | UUID | FK -> users(id), NOT NULL | Message sender |
| message | TEXT | NOT NULL | Message content |
| attachments | JSONB | DEFAULT '[]' | File attachments |
| is_internal | BOOLEAN | DEFAULT false | Internal note flag |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Message time |

**Indexes:**
```sql
CREATE INDEX idx_ticket_msg_ticket ON ticket_messages(ticket_id);
CREATE INDEX idx_ticket_msg_sender ON ticket_messages(sender_id);
```

### service_categories

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Category identifier |
| name | VARCHAR(100) | NOT NULL | Category name |
| name_ar | VARCHAR(100) | NOT NULL | Arabic name |
| description | TEXT | | Category description |
| icon_url | VARCHAR(500) | | Category icon |
| sort_order | INTEGER | DEFAULT 0 | Display order |
| status | ENUM | DEFAULT 'active' | active, inactive |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Creation date |

### service_providers

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Provider identifier |
| user_id | UUID | FK -> users(id), NOT NULL | Provider user |
| category_id | UUID | FK -> service_categories(id), NOT NULL | Service category |
| name | VARCHAR(200) | NOT NULL | Provider name |
| description | TEXT | | Provider description |
| rating | DECIMAL(3,2) | DEFAULT 0 | Average rating |
| total_jobs | INTEGER | DEFAULT 0 | Completed jobs |
| hourly_rate | DECIMAL(10,2) | | Hourly rate |
| status | ENUM | DEFAULT 'pending' | pending, active, suspended |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Registration date |

**Indexes:**
```sql
CREATE INDEX idx_svc_prov_category ON service_providers(category_id);
CREATE INDEX idx_svc_prov_status ON service_providers(status);
```

### service_bookings

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Booking identifier |
| customer_id | UUID | FK -> users(id), NOT NULL | Customer |
| provider_id | UUID | FK -> service_providers(id), NOT NULL | Service provider |
| category_id | UUID | FK -> service_categories(id), NOT NULL | Service type |
| service_date | TIMESTAMP | NOT NULL | Scheduled date/time |
| address | JSONB | NOT NULL | Service location |
| description | TEXT | | Service description |
| quoted_price | DECIMAL(10,2) | | Quoted price |
| final_price | DECIMAL(10,2) | | Final price |
| status | ENUM | NOT NULL, DEFAULT 'pending' | pending, confirmed, in_progress, completed, cancelled |
| rating | INTEGER | CHECK (1-5) | Customer rating |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Booking date |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Last update |

**Indexes:**
```sql
CREATE INDEX idx_booking_customer ON service_bookings(customer_id);
CREATE INDEX idx_booking_provider ON service_bookings(provider_id);
CREATE INDEX idx_booking_date ON service_bookings(service_date);
CREATE INDEX idx_booking_status ON service_bookings(status);
```

---

## B13 - Coupons

### coupons

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Coupon identifier |
| creator_id | UUID | NOT NULL | Creator (admin or vendor) |
| creator_type | ENUM | NOT NULL | admin, vendor |
| code | VARCHAR(50) | UNIQUE, NOT NULL | Coupon code |
| type | ENUM | NOT NULL | percentage, fixed, free_shipping, bogo |
| value | DECIMAL(12,2) | NOT NULL | Discount value |
| min_order_amount | DECIMAL(12,2) | | Minimum order amount |
| max_discount | DECIMAL(12,2) | | Maximum discount cap |
| max_uses | INTEGER | | Total usage limit |
| max_uses_per_user | INTEGER | DEFAULT 1 | Per-user limit |
| used_count | INTEGER | DEFAULT 0 | Current usage count |
| start_date | TIMESTAMP | NOT NULL | Valid from |
| end_date | TIMESTAMP | NOT NULL | Valid until |
| status | ENUM | DEFAULT 'active' | active, inactive, expired |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Creation date |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Last update |

**Indexes:**
```sql
CREATE UNIQUE INDEX idx_coupons_code ON coupons(code);
CREATE INDEX idx_coupons_creator ON coupons(creator_id, creator_type);
CREATE INDEX idx_coupons_status ON coupons(status);
CREATE INDEX idx_coupons_dates ON coupons(start_date, end_date);
```

### discount_rules

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Rule identifier |
| coupon_id | UUID | FK -> coupons(id), NOT NULL | Parent coupon |
| rule_type | ENUM | NOT NULL | min_items, specific_category, specific_product, bundle |
| rule_value | JSONB | NOT NULL | Rule configuration |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Creation date |

**Indexes:**
```sql
CREATE INDEX idx_discount_rules_coupon ON discount_rules(coupon_id);
```

### coupon_usages

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Usage identifier |
| coupon_id | UUID | FK -> coupons(id), NOT NULL | Used coupon |
| user_id | UUID | FK -> users(id), NOT NULL | User who used it |
| order_id | UUID | FK -> master_orders(id), NOT NULL | Order it was applied to |
| discount_amount | DECIMAL(12,2) | NOT NULL | Discount applied |
| used_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Usage timestamp |

**Indexes:**
```sql
CREATE INDEX idx_coupon_usage_coupon ON coupon_usages(coupon_id);
CREATE INDEX idx_coupon_usage_user ON coupon_usages(user_id);
CREATE INDEX idx_coupon_usage_order ON coupon_usages(order_id);
```

### coupon_restrictions

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Restriction identifier |
| coupon_id | UUID | FK -> coupons(id), NOT NULL | Parent coupon |
| restriction_type | ENUM | NOT NULL | new_users_only, specific_role, specific_vendor, specific_category, excluded_products |
| restriction_value | JSONB | NOT NULL | Restriction configuration |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Creation date |

**Indexes:**
```sql
CREATE INDEX idx_coupon_restrictions_coupon ON coupon_restrictions(coupon_id);
```

---

## Entity Relationship Summary

```
users ──┬── sessions
        ├── otp_codes
        ├── email_verifications
        ├── audit_logs
        ├── wallets ── wallet_transactions
        │              wallet_topups
        ├── cart_items
        ├── wishlists
        ├── store_follows
        ├── reviews
        ├── loyalty_points
        │     loyalty_transactions
        │     loyalty_tiers
        ├── support_tickets ── ticket_messages
        ├── notifications
        └── vendors ──┬── stores ── store_templates
                      ├── kyc_documents
                      ├── vendor_badges
                      ├── products ──┬── product_variants ── stock ── stock_reservations
                      │               ├── product_media
                      │               └── categories
                      ├── offers
                      ├── sub_orders ──┬── order_items
                      │                 ├── escrow_transactions ── refunds
                      │                 └── invoices
                      ├── commissions
                      ├── payouts
                      └── service_providers ── service_bookings

master_orders ── sub_orders
              ── tax_records
              ── coupon_usages

coupons ── discount_rules
        ── coupon_usages
        ── coupon_restrictions

delivery_providers ── delivery_zones
                   ── delivery_assignments ──┬── delivery_status_logs
                                             ── proof_of_delivery
```

---

## Sample Queries

### 1. Get products with pricing and stock

```sql
SELECT
    p.id,
    p.name_ar,
    p.name_en,
    p.slug,
    pv.sku,
    pv.price,
    pv.compare_at_price,
    COALESCE(s.quantity - s.reserved, 0) AS available_stock,
    c.name_ar AS category_name,
    v.store_name
FROM products p
JOIN product_variants pv ON pv.product_id = p.id
JOIN categories c ON c.id = p.category_id
JOIN vendors v ON v.id = p.vendor_id
LEFT JOIN stock s ON s.product_variant_id = pv.id
WHERE p.status = 'active'
  AND pv.status = 'active'
  AND c.status = 'active'
ORDER BY p.score DESC, p.created_at DESC
LIMIT 20;
```

### 2. Get vendor dashboard stats

```sql
SELECT
    v.id,
    v.store_name,
    COUNT(DISTINCT p.id) AS total_products,
    COUNT(DISTINCT so.id) FILTER (WHERE so.status = 'delivered') AS completed_orders,
    COALESCE(SUM(so.total) FILTER (WHERE so.status = 'delivered'), 0) AS total_revenue,
    COALESCE(AVG(r.rating), 0) AS avg_rating,
    COUNT(DISTINCT r.id) AS total_reviews
FROM vendors v
LEFT JOIN products p ON p.vendor_id = v.id AND p.deleted_at IS NULL
LEFT JOIN sub_orders so ON so.vendor_id = v.id
LEFT JOIN reviews r ON r.product_id IN (SELECT id FROM products WHERE vendor_id = v.id)
WHERE v.id = $1
GROUP BY v.id;
```

### 3. Get order with all details

```sql
SELECT
    mo.id AS master_order_id,
    mo.order_number,
    mo.status AS master_status,
    mo.total,
    so.id AS sub_order_id,
    so.status AS vendor_status,
    so.total AS vendor_total,
    oi.product_name,
    oi.quantity,
    oi.unit_price,
    oi.total AS item_total,
    da.tracking_number,
    da.status AS delivery_status
FROM master_orders mo
JOIN sub_orders so ON so.master_order_id = mo.id
JOIN order_items oi ON oi.sub_order_id = so.id
LEFT JOIN delivery_assignments da ON da.sub_order_id = so.id
WHERE mo.id = $1;
```

### 4. Apply coupon validation

```sql
SELECT
    c.id,
    c.type,
    c.value,
    c.min_order_amount,
    c.max_discount,
    c.max_uses,
    c.used_count,
    CASE
        WHEN c.end_date < NOW() THEN 'expired'
        WHEN c.start_date > NOW() THEN 'not_started'
        WHEN c.max_uses IS NOT NULL AND c.used_count >= c.max_uses THEN 'max_used'
        ELSE 'valid'
    END AS validation_status
FROM coupons c
WHERE c.code = $1
  AND c.status = 'active';
```

### 5. Get wallet balance with recent transactions

```sql
SELECT
    w.id,
    w.balance,
    w.locked_balance,
    w.currency,
    (
        SELECT json_agg(tx.* ORDER BY tx.created_at DESC)
        FROM (
            SELECT type, amount, balance_after, description, created_at
            FROM wallet_transactions
            WHERE wallet_id = w.id
            ORDER BY created_at DESC
            LIMIT 10
        ) tx
    ) AS recent_transactions
FROM wallets w
WHERE w.user_id = $1 AND w.currency = 'YER';
```

### 6. Vendor commission calculation

```sql
INSERT INTO commissions (id, vendor_id, sub_order_id, amount, rate, status)
SELECT
    gen_random_uuid(),
    so.vendor_id,
    so.id,
    so.total * (v.commission_rate / 100),
    v.commission_rate,
    'pending'
FROM sub_orders so
JOIN vendors v ON v.id = so.vendor_id
WHERE so.id = $1
  AND so.status = 'delivered';
```

### 7. Search products with full-text

```sql
SELECT
    p.id,
    p.name_ar,
    p.name_en,
    p.slug,
    ts_rank(
        to_tsvector('simple', p.name_ar || ' ' || p.name_en || ' ' || COALESCE(p.description_ar, '')),
        plainto_tsquery('simple', $1)
    ) AS relevance
FROM products p
WHERE p.status = 'active'
  AND (
    to_tsvector('simple', p.name_ar || ' ' || p.name_en || ' ' || COALESCE(p.description_ar, ''))
    @@ plainto_tsquery('simple', $1)
  )
ORDER BY relevance DESC
LIMIT 20;
```

### 8. Get delivery zone pricing

```sql
SELECT
    dz.id,
    dz.name,
    dz.name_ar,
    dz.base_price,
    dz.per_km_price,
    dz.estimated_hours,
    dp.name AS provider_name
FROM delivery_zones dz
JOIN delivery_providers dp ON dp.id = dz.provider_id
WHERE dp.status = 'active'
  AND dz.status = 'active'
  AND dz.coordinates @> ST_Point($1, $2)::geography
ORDER BY dz.base_price ASC
LIMIT 5;
```

### 9. Expire old stock reservations

```sql
WITH expired AS (
    DELETE FROM stock_reservations
    WHERE status = 'active'
      AND expires_at < NOW()
    RETURNING stock_id, quantity
)
UPDATE stock s
SET reserved = s.reserved - e.quantity
FROM expired e
WHERE s.id = e.stock_id;
```

### 10. Generate order summary report

```sql
SELECT
    DATE_TRUNC('day', mo.created_at) AS order_date,
    COUNT(DISTINCT mo.id) AS total_orders,
    SUM(mo.total) AS total_revenue,
    COUNT(DISTINCT so.vendor_id) AS active_vendors,
    AVG(mo.total) AS avg_order_value
FROM master_orders mo
JOIN sub_orders so ON so.master_order_id = mo.id
WHERE mo.created_at >= $1
  AND mo.created_at < $2
  AND mo.status != 'cancelled'
GROUP BY DATE_TRUNC('day', mo.created_at)
ORDER BY order_date DESC;
```

---

## Migration Naming Convention

```
YYYYMMDDHHMMSS_<block>_<description>.sql
```

Examples:
```
20260101120000_b01_create_users.sql
20260101120100_b02_create_vendors.sql
20260101120200_b03_create_products.sql
```

## Index Naming Convention

```
idx_<table>_<columns>
```

Examples:
```
idx_users_phone
idx_products_vendor_status
idx_orders_customer_created
```

## Foreign Key Naming Convention

```
fk_<table>_<referenced_table>
```

Examples:
```
fk_vendors_users
fk_products_categories
fk_sub_orders_master_orders
```
