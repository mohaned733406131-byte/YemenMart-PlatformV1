# Data Dictionary — YemenMart v2

**Document Version:** 1.0
**Date:** 2026-09-13
**Database:** PostgreSQL 15+
**Character Set:** UTF-8
**Collation:** `en_US.UTF-8`

---

## Conventions

- All tables use UUID primary keys (`uuid` type, default `gen_random_uuid()`)
- Timestamps are stored in UTC with `timestamptz`
- Soft deletes use `deleted_at` timestamp (NULL = active)
- Monetary values stored as `bigint` in smallest currency unit (YER has no decimals)
- All tables include `created_at` and `updated_at` columns
- Audit columns: `created_by`, `updated_by` where applicable
- Index naming: `idx_{table}_{column(s)}`; unique: `uniq_{table}_{column}`

---

## 1. Users & Authentication

### `users`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `user_id` | `uuid` | PK, DEFAULT gen_random_uuid() | Unique user identifier |
| `email` | `varchar(255)` | UNIQUE, NOT NULL | User email address |
| `phone` | `varchar(20)` | UNIQUE | Phone number (E.164 format) |
| `phone_country_code` | `varchar(5)` | | Country code (+967) |
| `password_hash` | `varchar(255)` | NOT NULL | Bcrypt password hash |
| `first_name` | `varchar(100)` | NOT NULL | Given name |
| `last_name` | `varchar(100)` | NOT NULL | Family name |
| `avatar_url` | `text` | | Profile picture URL |
| `role` | `varchar(20)` | NOT NULL, DEFAULT 'customer' | `customer`, `vendor`, `rider`, `admin`, `super_admin` |
| `status` | `varchar(20)` | NOT NULL, DEFAULT 'active' | `active`, `suspended`, `banned`, `pending_verification` |
| `locale` | `varchar(5)` | DEFAULT 'ar' | Preferred locale (ar, en) |
| `currency` | `varchar(3)` | DEFAULT 'YER' | Preferred currency |
| `email_verified` | `boolean` | DEFAULT false | Email verification status |
| `phone_verified` | `boolean` | DEFAULT false | Phone verification status |
| `two_factor_enabled` | `boolean` | DEFAULT false | 2FA enabled |
| `two_factor_secret` | `varchar(255)` | | TOTP secret (encrypted) |
| `last_login_at` | `timestamptz` | | Last login timestamp |
| `last_login_ip` | `inet` | | Last login IP address |
| `failed_login_attempts` | `integer` | DEFAULT 0 | Consecutive failed logins |
| `locked_until` | `timestamptz` | | Account lock expiration |
| `password_changed_at` | `timestamptz` | | Last password change |
| `deletion_requested_at` | `timestamptz` | | GDPR deletion request timestamp |
| `metadata` | `jsonb` | DEFAULT '{}' | Additional user data |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT now() | Record creation time |
| `updated_at` | `timestamptz` | NOT NULL, DEFAULT now() | Last update time |
| `deleted_at` | `timestamptz` | | Soft delete timestamp |

**Indexes:**
- `uniq_users_email` UNIQUE on (`email`)
- `uniq_users_phone` UNIQUE on (`phone`)
- `idx_users_role` on (`role`)
- `idx_users_status` on (`status`)
- `idx_users_created_at` on (`created_at`)
- `idx_users_deleted_at` on (`deleted_at`) WHERE deleted_at IS NULL

---

### `user_sessions`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `session_id` | `uuid` | PK, DEFAULT gen_random_uuid() | Session identifier |
| `user_id` | `uuid` | NOT NULL, FK → users(user_id) | Session owner |
| `refresh_token_hash` | `varchar(255)` | NOT NULL | Hashed refresh token |
| `device_id` | `varchar(255)` | | Device fingerprint |
| `device_type` | `varchar(20)` | | `web`, `ios`, `android` |
| `device_name` | `varchar(255)` | | Human-readable device name |
| `ip_address` | `inet` | | Client IP address |
| `user_agent` | `text` | | Client user agent string |
| `expires_at` | `timestamptz` | NOT NULL | Session expiration |
| `last_active_at` | `timestamptz` | DEFAULT now() | Last activity timestamp |
| `is_revoked` | `boolean` | DEFAULT false | Revoked flag |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT now() | Session creation |

**Indexes:**
- `idx_user_sessions_user_id` on (`user_id`)
- `idx_user_sessions_refresh_token_hash` on (`refresh_token_hash`)
- `idx_user_sessions_expires_at` on (`expires_at`)

---

### `user_addresses`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `address_id` | `uuid` | PK, DEFAULT gen_random_uuid() | Address identifier |
| `user_id` | `uuid` | NOT NULL, FK → users(user_id) | Address owner |
| `label` | `varchar(50)` | NOT NULL | Address label (Home, Work) |
| `full_name` | `varchar(200)` | NOT NULL | Recipient full name |
| `phone` | `varchar(20)` | NOT NULL | Recipient phone |
| `governorate` | `varchar(100)` | NOT NULL | Governorate name |
| `district` | `varchar(100)` | NOT NULL | District name |
| `street` | `varchar(255)` | NOT NULL | Street address |
| `building` | `varchar(50)` | | Building number |
| `floor` | `varchar(10)` | | Floor number |
| `apartment` | `varchar(10)` | | Apartment number |
| `landmark` | `varchar(255)` | | Nearby landmark |
| `latitude` | `decimal(10,7)` | | GPS latitude |
| `longitude` | `decimal(10,7)` | | GPS longitude |
| `is_default` | `boolean` | DEFAULT false | Default delivery address |
| `delivery_instructions` | `text` | | Special delivery notes |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT now() | Record creation |
| `updated_at` | `timestamptz` | NOT NULL, DEFAULT now() | Last update |
| `deleted_at` | `timestamptz` | | Soft delete |

**Indexes:**
- `idx_user_addresses_user_id` on (`user_id`)
- `idx_user_addresses_governorate` on (`governorate`)

---

### `verification_tokens`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `token_id` | `uuid` | PK, DEFAULT gen_random_uuid() | Token identifier |
| `user_id` | `uuid` | NOT NULL, FK → users(user_id) | Token owner |
| `token_type` | `varchar(20)` | NOT NULL | `email`, `phone`, `password_reset`, `2fa_backup` |
| `token_hash` | `varchar(255)` | NOT NULL | Hashed token value |
| `expires_at` | `timestamptz` | NOT NULL | Token expiration |
| `used_at` | `timestamptz` | | When token was used |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT now() | Token creation |

**Indexes:**
- `idx_verification_tokens_token_hash` on (`token_hash`)
- `idx_verification_tokens_user_id` on (`user_id`)

---

## 2. Vendors

### `vendors`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `vendor_id` | `uuid` | PK, DEFAULT gen_random_uuid() | Vendor identifier |
| `user_id` | `uuid` | UNIQUE, NOT NULL, FK → users(user_id) | Owner user account |
| `business_name` | `varchar(200)` | NOT NULL | Legal business name |
| `business_name_ar` | `varchar(200)` | | Arabic business name |
| `slug` | `varchar(200)` | UNIQUE, NOT NULL | URL-friendly identifier |
| `description` | `text` | | Business description |
| `description_ar` | `text` | | Arabic business description |
| `avatar_url` | `text` | | Business logo |
| `banner_url` | `text` | | Store banner image |
| `business_type` | `varchar(50)` | NOT NULL | Business category |
| `commercial_registration` | `varchar(100)` | | CR number |
| `tax_id` | `varchar(50)` | | Tax identification number |
| `governorate` | `varchar(100)` | NOT NULL | Primary location |
| `district` | `varchar(100)` | NOT NULL | District |
| `phone` | `varchar(20)` | NOT NULL | Business phone |
| `email` | `varchar(255)` | NOT NULL | Business email |
| `status` | `varchar(20)` | NOT NULL, DEFAULT 'pending_review' | `pending_review`, `approved`, `rejected`, `suspended` |
| `verified` | `boolean` | DEFAULT false | Verification badge |
| `rating_average` | `decimal(3,2)` | DEFAULT 0.00 | Average rating (0.00-5.00) |
| `rating_count` | `integer` | DEFAULT 0 | Total rating count |
| `product_count` | `integer` | DEFAULT 0 | Active product count |
| `commission_rate` | `decimal(5,2)` | DEFAULT 10.00 | Platform commission % |
| `established_year` | `integer` | | Year business started |
| `return_days` | `integer` | DEFAULT 14 | Return window in days |
| `warranty_days` | `integer` | DEFAULT 365 | Warranty period in days |
| `min_order_amount` | `bigint` | DEFAULT 0 | Minimum order (YER) |
| `free_shipping_min` | `bigint` | DEFAULT 0 | Free shipping threshold |
| `shipping_cost` | `bigint` | DEFAULT 5000 | Default shipping cost |
| `response_time_hours` | `integer` | DEFAULT 24 | Avg response time |
| `metadata` | `jsonb` | DEFAULT '{}' | Additional vendor data |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT now() | Registration date |
| `updated_at` | `timestamptz` | NOT NULL, DEFAULT now() | Last update |
| `approved_at` | `timestamptz` | | Approval timestamp |
| `suspended_at` | `timestamptz` | | Suspension timestamp |
| `deleted_at` | `timestamptz` | | Soft delete |

**Indexes:**
- `uniq_vendors_slug` UNIQUE on (`slug`)
- `uniq_vendors_user_id` UNIQUE on (`user_id`)
- `idx_vendors_status` on (`status`)
- `idx_vendors_governorate` on (`governorate`)
- `idx_vendors_rating` on (`rating_average` DESC)
- `idx_vendors_created_at` on (`created_at`)

---

### `vendor_documents`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `document_id` | `uuid` | PK | Document identifier |
| `vendor_id` | `uuid` | NOT NULL, FK → vendors(vendor_id) | Vendor |
| `document_type` | `varchar(50)` | NOT NULL | `commercial_registration`, `tax_cert`, `national_id`, `bank_statement` |
| `file_url` | `text` | NOT NULL | S3 file URL |
| `file_name` | `varchar(255)` | NOT NULL | Original file name |
| `file_size` | `integer` | | File size in bytes |
| `status` | `varchar(20)` | DEFAULT 'pending' | `pending`, `approved`, `rejected` |
| `rejection_reason` | `text` | | Why document was rejected |
| `verified_by` | `uuid` | FK → users(user_id) | Admin who verified |
| `verified_at` | `timestamptz` | | Verification timestamp |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT now() | Upload time |

**Indexes:**
- `idx_vendor_documents_vendor_id` on (`vendor_id`)
- `idx_vendor_documents_status` on (`status`)

---

### `vendor_bank_accounts`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `bank_account_id` | `uuid` | PK | Account identifier |
| `vendor_id` | `uuid` | NOT NULL, FK → vendors(vendor_id) | Vendor |
| `bank_name` | `varchar(200)` | NOT NULL | Bank name |
| `account_number` | `varchar(50)` | NOT NULL | Account number (encrypted) |
| `account_name` | `varchar(200)` | NOT NULL | Account holder name |
| `iban` | `varchar(34)` | | IBAN (if available) |
| `is_default` | `boolean` | DEFAULT false | Default payout account |
| `verified` | `boolean` | DEFAULT false | Bank verification status |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT now() | Record creation |
| `deleted_at` | `timestamptz` | | Soft delete |

**Indexes:**
- `idx_vendor_bank_accounts_vendor_id` on (`vendor_id`)

---

## 3. Products & Catalog

### `categories`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `category_id` | `uuid` | PK | Category identifier |
| `parent_id` | `uuid` | FK → categories(category_id) | Parent category (NULL for root) |
| `name` | `varchar(200)` | NOT NULL | Category name |
| `name_ar` | `varchar(200)` | NOT NULL | Arabic category name |
| `slug` | `varchar(200)` | UNIQUE, NOT NULL | URL-friendly identifier |
| `description` | `text` | | Category description |
| `description_ar` | `text` | | Arabic description |
| `icon_url` | `text` | | Category icon |
| `image_url` | `text` | | Category banner image |
| `sort_order` | `integer` | DEFAULT 0 | Display order |
| `is_active` | `boolean` | DEFAULT true | Active status |
| `product_count` | `integer` | DEFAULT 0 | Denormalized product count |
| `level` | `integer` | NOT NULL, DEFAULT 0 | Hierarchy depth (0 = root) |
| `path` | `ltree` | | Materialized path for tree queries |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT now() | Record creation |
| `updated_at` | `timestamptz` | NOT NULL, DEFAULT now() | Last update |

**Indexes:**
- `uniq_categories_slug` UNIQUE on (`slug`)
- `idx_categories_parent_id` on (`parent_id`)
- `idx_categories_path` GIST on (`path`)
- `idx_categories_sort_order` on (`sort_order`)

---

### `products`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `product_id` | `uuid` | PK | Product identifier |
| `vendor_id` | `uuid` | NOT NULL, FK → vendors(vendor_id) | Product vendor |
| `category_id` | `uuid` | NOT NULL, FK → categories(category_id) | Product category |
| `name` | `varchar(300)` | NOT NULL | Product name |
| `name_ar` | `varchar(300)` | NOT NULL | Arabic product name |
| `slug` | `varchar(350)` | UNIQUE, NOT NULL | URL-friendly identifier |
| `description` | `text` | | Product description (HTML) |
| `description_ar` | `text` | | Arabic product description |
| `sku` | `varchar(100)` | | Vendor SKU |
| `barcode` | `varchar(50)` | | EAN/UPC barcode |
| `price` | `bigint` | NOT NULL | Current price (smallest currency unit) |
| `original_price` | `bigint` | | Price before discount |
| `currency` | `varchar(3)` | NOT NULL, DEFAULT 'YER' | Price currency |
| `cost_price` | `bigint` | | Vendor cost (internal) |
| `stock_quantity` | `integer` | NOT NULL, DEFAULT 0 | Available stock |
| `low_stock_threshold` | `integer` | DEFAULT 10 | Low stock alert level |
| `track_inventory` | `boolean` | DEFAULT true | Enable inventory tracking |
| `weight_grams` | `integer` | | Product weight for shipping |
| `length_cm` | `integer` | | Package dimensions |
| `width_cm` | `integer` | | Package dimensions |
| `height_cm` | `integer` | | Package dimensions |
| `status` | `varchar(20)` | NOT NULL, DEFAULT 'pending_review' | `active`, `pending_review`, `rejected`, `archived` |
| `is_featured` | `boolean` | DEFAULT false | Featured product flag |
| `rating_average` | `decimal(3,2)` | DEFAULT 0.00 | Average rating |
| `rating_count` | `integer` | DEFAULT 0 | Total reviews count |
| `sales_count` | `integer` | DEFAULT 0 | Total units sold |
| `view_count` | `integer` | DEFAULT 0 | Total views |
| `has_variants` | `boolean` | DEFAULT false | Product has variants |
| `is_digital` | `boolean` | DEFAULT false | Digital product flag |
| `tags` | `text[]` | | Array of tags |
| `attributes` | `jsonb` | DEFAULT '{}' | Key-value attributes (brand, color, etc.) |
| `metadata` | `jsonb` | DEFAULT '{}' | Additional product data |
| `seo_title` | `varchar(200)` | | SEO title override |
| `seo_description` | `text` | | SEO meta description |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT now() | Record creation |
| `updated_at` | `timestamptz` | NOT NULL, DEFAULT now() | Last update |
| `published_at` | `timestamptz` | | When product went live |
| `deleted_at` | `timestamptz` | | Soft delete |

**Indexes:**
- `uniq_products_slug` UNIQUE on (`slug`)
- `idx_products_vendor_id` on (`vendor_id`)
- `idx_products_category_id` on (`category_id`)
- `idx_products_status` on (`status`)
- `idx_products_price` on (`price`)
- `idx_products_rating` on (`rating_average` DESC)
- `idx_products_sales` on (`sales_count` DESC)
- `idx_products_created_at` on (`created_at`)
- `idx_products_tags` GIN on (`tags`)
- `idx_products_attributes` GIN on (`attributes`)
- `idx_products_fulltext` GIN on (to_tsvector('english', name || ' ' || COALESCE(description, '')))

---

### `product_variants`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `variant_id` | `uuid` | PK | Variant identifier |
| `product_id` | `uuid` | NOT NULL, FK → products(product_id) | Parent product |
| `name` | `varchar(300)` | NOT NULL | Variant display name |
| `sku` | `varchar(100)` | | Variant-specific SKU |
| `barcode` | `varchar(50)` | | Variant barcode |
| `price` | `bigint` | NOT NULL | Variant price |
| `original_price` | `bigint` | | Price before discount |
| `stock_quantity` | `integer` | NOT NULL, DEFAULT 0 | Available stock |
| `attributes` | `jsonb` | NOT NULL | Variant attributes (color, size, etc.) |
| `image_url` | `text` | | Variant-specific image |
| `is_active` | `boolean` | DEFAULT true | Active status |
| `sort_order` | `integer` | DEFAULT 0 | Display order |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT now() | Record creation |
| `updated_at` | `timestamptz` | NOT NULL, DEFAULT now() | Last update |
| `deleted_at` | `timestamptz` | | Soft delete |

**Indexes:**
- `idx_product_variants_product_id` on (`product_id`)
- `idx_product_variants_sku` on (`sku`)

---

### `product_images`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `image_id` | `uuid` | PK | Image identifier |
| `product_id` | `uuid` | NOT NULL, FK → products(product_id) | Parent product |
| `variant_id` | `uuid` | FK → product_variants(variant_id) | Associated variant (nullable) |
| `url` | `text` | NOT NULL | Image URL (CDN) |
| `alt_text` | `varchar(255)` | | Alt text for accessibility |
| `is_primary` | `boolean` | DEFAULT false | Primary product image |
| `sort_order` | `integer` | DEFAULT 0 | Display order |
| `width` | `integer` | | Image width in pixels |
| `height` | `integer` | | Image height in pixels |
| `file_size` | `integer` | | File size in bytes |
| `format` | `varchar(10)` | DEFAULT 'webp' | Image format |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT now() | Upload time |

**Indexes:**
- `idx_product_images_product_id` on (`product_id`)
- `idx_product_images_variant_id` on (`variant_id`)

---

### `product_inventory_log`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `log_id` | `uuid` | PK | Log identifier |
| `product_id` | `uuid` | NOT NULL, FK → products(product_id) | Product |
| `variant_id` | `uuid` | FK → product_variants(variant_id) | Variant (nullable) |
| `change_type` | `varchar(20)` | NOT NULL | `sale`, `restock`, `adjustment`, `return`, `damage` |
| `quantity_change` | `integer` | NOT NULL | Stock delta (+/-) |
| `quantity_after` | `integer` | NOT NULL | Stock after change |
| `reference_type` | `varchar(20)` | | `order`, `return`, `manual` |
| `reference_id` | `uuid` | | Related entity ID |
| `notes` | `text` | | Adjustment notes |
| `performed_by` | `uuid` | FK → users(user_id) | Who made the change |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT now() | Change timestamp |

**Indexes:**
- `idx_product_inventory_log_product_id` on (`product_id`)
- `idx_product_inventory_log_created_at` on (`created_at`)

---

## 4. Orders & Transactions

### `orders`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `order_id` | `uuid` | PK | Order identifier |
| `order_number` | `varchar(20)` | UNIQUE, NOT NULL | Human-readable order number |
| `user_id` | `uuid` | NOT NULL, FK → users(user_id) | Customer |
| `status` | `varchar(30)` | NOT NULL, DEFAULT 'pending_payment' | Order status (see status machine) |
| `subtotal` | `bigint` | NOT NULL | Sum of item prices |
| `discount_amount` | `bigint` | DEFAULT 0 | Coupon discount |
| `shipping_amount` | `bigint` | DEFAULT 0 | Shipping cost |
| `tax_amount` | `bigint` | DEFAULT 0 | Tax amount |
| `total_amount` | `bigint` | NOT NULL | Grand total |
| `currency` | `varchar(3)` | NOT NULL, DEFAULT 'YER' | Order currency |
| `payment_method` | `varchar(30)` | | `card`, `wallet`, `cod`, `bank_transfer` |
| `payment_status` | `varchar(20)` | DEFAULT 'pending' | `pending`, `paid`, `failed`, `refunded`, `partially_refunded` |
| `coupon_id` | `uuid` | FK → coupons(coupon_id) | Applied coupon |
| `coupon_code` | `varchar(50)` | | Coupon code used |
| `shipping_address_id` | `uuid` | FK → user_addresses(address_id) | Delivery address |
| `shipping_address_snapshot` | `jsonb` | | Address at time of order |
| `billing_address_id` | `uuid` | | Billing address (nullable) |
| `notes` | `text` | | Customer order notes |
| `vendor_notes` | `text` | | Vendor internal notes |
| `metadata` | `jsonb` | DEFAULT '{}' | Additional order data |
| `placed_at` | `timestamptz` | | When order was placed |
| `confirmed_at` | `timestamptz` | | When vendor confirmed |
| `shipped_at` | `timestamptz` | | When shipped |
| `delivered_at` | `timestamptz` | | When delivered |
| `cancelled_at` | `timestamptz` | | When cancelled |
| `cancel_reason` | `text` | | Cancellation reason |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT now() | Record creation |
| `updated_at` | `timestamptz` | NOT NULL, DEFAULT now() | Last update |
| `deleted_at` | `timestamptz` | | Soft delete |

**Order Status Machine:**
```
pending_payment → confirmed → processing → shipped → delivered
                ↓                                    ↓
            cancelled                          returned
pending_payment → cancelled
```

**Indexes:**
- `uniq_orders_order_number` UNIQUE on (`order_number`)
- `idx_orders_user_id` on (`user_id`)
- `idx_orders_status` on (`status`)
- `idx_orders_payment_status` on (`payment_status`)
- `idx_orders_created_at` on (`created_at`)
- `idx_orders_placed_at` on (`placed_at`)
- `idx_orders_coupon_id` on (`coupon_id`)

---

### `order_items`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `item_id` | `uuid` | PK | Order item identifier |
| `order_id` | `uuid` | NOT NULL, FK → orders(order_id) | Parent order |
| `product_id` | `uuid` | NOT NULL, FK → products(product_id) | Product |
| `variant_id` | `uuid` | FK → product_variants(variant_id) | Variant (nullable) |
| `vendor_id` | `uuid` | NOT NULL, FK → vendors(vendor_id) | Vendor (denormalized) |
| `product_name` | `varchar(300)` | NOT NULL | Product name at time of order |
| `variant_name` | `varchar(300)` | | Variant name at time of order |
| `sku` | `varchar(100)` | | SKU at time of order |
| `price` | `bigint` | NOT NULL | Unit price at time of order |
| `original_price` | `bigint` | | Original price before discount |
| `quantity` | `integer` | NOT NULL, CHECK > 0 | Quantity ordered |
| `subtotal` | `bigint` | NOT NULL | price × quantity |
| `discount_amount` | `bigint` | DEFAULT 0 | Item-level discount |
| `tax_amount` | `bigint` | DEFAULT 0 | Item tax |
| `image_url` | `text` | | Product image at time of order |
| `is_gift` | `boolean` | DEFAULT false | Gift flag |
| `gift_message` | `text` | | Gift message |
| `status` | `varchar(20)` | DEFAULT 'pending' | `pending`, `confirmed`, `shipped`, `delivered`, `returned`, `cancelled` |
| `return_status` | `varchar(20)` | | `none`, `requested`, `approved`, `received`, `refunded` |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT now() | Record creation |
| `updated_at` | `timestamptz` | NOT NULL, DEFAULT now() | Last update |

**Indexes:**
- `idx_order_items_order_id` on (`order_id`)
- `idx_order_items_product_id` on (`product_id`)
- `idx_order_items_vendor_id` on (`vendor_id`)

---

### `order_status_history`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `history_id` | `uuid` | PK | History identifier |
| `order_id` | `uuid` | NOT NULL, FK → orders(order_id) | Order |
| `from_status` | `varchar(30)` | | Previous status |
| `to_status` | `varchar(30)` | NOT NULL | New status |
| `note` | `text` | | Status change note |
| `performed_by` | `uuid` | FK → users(user_id) | Who changed status |
| `actor_type` | `varchar(20)` | NOT NULL | `customer`, `vendor`, `admin`, `system`, `rider` |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT now() | Change timestamp |

**Indexes:**
- `idx_order_status_history_order_id` on (`order_id`)
- `idx_order_status_history_created_at` on (`created_at`)

---

### `payments`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `payment_id` | `uuid` | PK | Payment identifier |
| `order_id` | `uuid` | NOT NULL, FK → orders(order_id) | Related order |
| `user_id` | `uuid` | NOT NULL, FK → users(user_id) | Payer |
| `method` | `varchar(30)` | NOT NULL | Payment method |
| `status` | `varchar(20)` | NOT NULL, DEFAULT 'pending' | `pending`, `processing`, `completed`, `failed`, `cancelled`, `refunded` |
| `amount` | `bigint` | NOT NULL | Payment amount |
| `currency` | `varchar(3)` | NOT NULL | Payment currency |
| `gateway` | `varchar(50)` | | Payment gateway name |
| `gateway_payment_id` | `varchar(255)` | | Gateway transaction ID |
| `gateway_response` | `jsonb` | | Raw gateway response |
| `card_last_four` | `varchar(4)` | | Last 4 digits (for card) |
| `card_brand` | `varchar(20)` | | Card brand (visa, mastercard) |
| `failure_reason` | `text` | | Payment failure reason |
| `refund_amount` | `bigint` | DEFAULT 0 | Total refunded amount |
| `refund_reason` | `text` | | Refund reason |
| `metadata` | `jsonb` | DEFAULT '{}' | Additional payment data |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT now() | Payment initiation |
| `updated_at` | `timestamptz` | NOT NULL, DEFAULT now() | Last update |
| `completed_at` | `timestamptz` | | Payment completion |

**Indexes:**
- `idx_payments_order_id` on (`order_id`)
- `idx_payments_user_id` on (`user_id`)
- `idx_payments_status` on (`status`)
- `idx_payments_gateway_payment_id` on (`gateway_payment_id`)
- `idx_payments_created_at` on (`created_at`)

---

### `refunds`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `refund_id` | `uuid` | PK | Refund identifier |
| `payment_id` | `uuid` | NOT NULL, FK → payments(payment_id) | Original payment |
| `order_id` | `uuid` | NOT NULL, FK → orders(order_id) | Related order |
| `user_id` | `uuid` | NOT NULL, FK → users(user_id) | Refund recipient |
| `amount` | `bigint` | NOT NULL | Refund amount |
| `currency` | `varchar(3)` | NOT NULL | Refund currency |
| `reason` | `varchar(50)` | NOT NULL | `item_return`, `cancelled_order`, `item_not_received`, `duplicate_charge`, `other` |
| `description` | `text` | | Detailed reason |
| `status` | `varchar(20)` | NOT NULL, DEFAULT 'pending' | `pending`, `approved`, `processed`, `rejected` |
| `processed_by` | `uuid` | FK → users(user_id) | Admin who processed |
| `gateway_refund_id` | `varchar(255)` | | Gateway refund transaction ID |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT now() | Request timestamp |
| `processed_at` | `timestamptz` | | Processing timestamp |

**Indexes:**
- `idx_refunds_payment_id` on (`payment_id`)
- `idx_refunds_order_id` on (`order_id`)
- `idx_refunds_user_id` on (`user_id`)
- `idx_refunds_status` on (`status`)

---

## 5. Wallet System

### `wallets`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `wallet_id` | `uuid` | PK | Wallet identifier |
| `user_id` | `uuid` | UNIQUE, NOT NULL, FK → users(user_id) | Wallet owner |
| `balance` | `bigint` | NOT NULL, DEFAULT 0 | Available balance (YER) |
| `pending_balance` | `bigint` | NOT NULL, DEFAULT 0 | Pending (unconfirmed) balance |
| `currency` | `varchar(3)` | NOT NULL, DEFAULT 'YER' | Wallet currency |
| `is_frozen` | `boolean` | DEFAULT false | Frozen (admin action) |
| `freeze_reason` | `text` | | Freeze reason |
| `total_earned` | `bigint` | DEFAULT 0 | Lifetime earnings |
| `total_spent` | `bigint` | DEFAULT 0 | Lifetime spending |
| `total_withdrawn` | `bigint` | DEFAULT 0 | Lifetime withdrawals |
| `total_deposited` | `bigint` | DEFAULT 0 | Lifetime deposits |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT now() | Wallet creation |
| `updated_at` | `timestamptz` | NOT NULL, DEFAULT now() | Last update |

**Indexes:**
- `uniq_wallets_user_id` UNIQUE on (`user_id`)

**Constraints:**
- `chk_wallets_balance_positive` CHECK (balance >= 0)

---

### `wallet_transactions`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `transaction_id` | `uuid` | PK | Transaction identifier |
| `wallet_id` | `uuid` | NOT NULL, FK → wallets(wallet_id) | Source wallet |
| `type` | `varchar(20)` | NOT NULL | `credit`, `debit`, `refund`, `cashback`, `withdrawal`, `topup`, `transfer_in`, `transfer_out` |
| `amount` | `bigint` | NOT NULL, CHECK > 0 | Transaction amount |
| `balance_before` | `bigint` | NOT NULL | Balance before transaction |
| `balance_after` | `bigint` | NOT NULL | Balance after transaction |
| `description` | `text` | NOT NULL | Human-readable description |
| `reference_type` | `varchar(20)` | | `order`, `payment`, `refund`, `coupon`, `transfer`, `manual` |
| `reference_id` | `uuid` | | Related entity ID |
| `status` | `varchar(20)` | NOT NULL, DEFAULT 'completed' | `pending`, `completed`, `failed`, `reversed` |
| `performed_by` | `uuid` | FK → users(user_id) | Who initiated |
| `ip_address` | `inet` | | Client IP |
| `metadata` | `jsonb` | DEFAULT '{}' | Additional transaction data |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT now() | Transaction time |

**Indexes:**
- `idx_wallet_transactions_wallet_id` on (`wallet_id`)
- `idx_wallet_transactions_type` on (`type`)
- `idx_wallet_transactions_reference` on (`reference_type`, `reference_id`)
- `idx_wallet_transactions_created_at` on (`created_at`)
- `idx_wallet_transactions_status` on (`status`)

---

### `wallet_topups`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `topup_id` | `uuid` | PK | Top-up identifier |
| `wallet_id` | `uuid` | NOT NULL, FK → wallets(wallet_id) | Target wallet |
| `user_id` | `uuid` | NOT NULL, FK → users(user_id) | User performing top-up |
| `amount` | `bigint` | NOT NULL | Top-up amount |
| `currency` | `varchar(3)` | NOT NULL | Payment currency |
| `payment_method` | `varchar(30)` | NOT NULL | `card`, `bank_transfer` |
| `payment_id` | `uuid` | FK → payments(payment_id) | Related payment |
| `status` | `varchar(20)` | NOT NULL, DEFAULT 'pending' | `pending`, `completed`, `failed` |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT now() | Request time |
| `completed_at` | `timestamptz` | | Completion time |

**Indexes:**
- `idx_wallet_topups_wallet_id` on (`wallet_id`)
- `idx_wallet_topups_user_id` on (`user_id`)
- `idx_wallet_topups_status` on (`status`)

---

### `wallet_withdrawals`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `withdrawal_id` | `uuid` | PK | Withdrawal identifier |
| `wallet_id` | `uuid` | NOT NULL, FK → wallets(wallet_id) | Source wallet |
| `user_id` | `uuid` | NOT NULL, FK → users(user_id) | User requesting withdrawal |
| `bank_account_id` | `uuid` | NOT NULL, FK → vendor_bank_accounts(bank_account_id) | Destination bank |
| `amount` | `bigint` | NOT NULL | Withdrawal amount |
| `currency` | `varchar(3)` | NOT NULL | Withdrawal currency |
| `status` | `varchar(20)` | NOT NULL, DEFAULT 'pending' | `pending`, `processing`, `completed`, `failed`, `cancelled` |
| `processed_by` | `uuid` | FK → users(user_id) | Admin who processed |
| `bank_reference` | `varchar(100)` | | Bank transfer reference |
| `failure_reason` | `text` | | Failure reason |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT now() | Request time |
| `processed_at` | `timestamptz` | | Processing time |
| `completed_at` | `timestamptz` | | Completion time |

**Indexes:**
- `idx_wallet_withdrawals_wallet_id` on (`wallet_id`)
- `idx_wallet_withdrawals_user_id` on (`user_id`)
- `idx_wallet_withdrawals_status` on (`status`)

---

### `wallet_transfers`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `transfer_id` | `uuid` | PK | Transfer identifier |
| `sender_wallet_id` | `uuid` | NOT NULL, FK → wallets(wallet_id) | Sender wallet |
| `receiver_wallet_id` | `uuid` | NOT NULL, FK → wallets(wallet_id) | Receiver wallet |
| `sender_user_id` | `uuid` | NOT NULL, FK → users(user_id) | Sender |
| `receiver_user_id` | `uuid` | NOT NULL, FK → users(user_id) | Receiver |
| `amount` | `bigint` | NOT NULL, CHECK > 0 | Transfer amount |
| `currency` | `varchar(3)` | NOT NULL | Transfer currency |
| `note` | `text` | | Transfer note |
| `status` | `varchar(20)` | NOT NULL, DEFAULT 'completed' | `pending`, `completed`, `failed`, `reversed` |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT now() | Transfer time |

**Indexes:**
- `idx_wallet_transfers_sender` on (`sender_wallet_id`)
- `idx_wallet_transfers_receiver` on (`receiver_wallet_id`)
- `idx_wallet_transfers_created_at` on (`created_at`)

---

## 6. Deliveries

### `deliveries`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `delivery_id` | `uuid` | PK | Delivery identifier |
| `order_id` | `uuid` | UNIQUE, NOT NULL, FK → orders(order_id) | Related order |
| `rider_id` | `uuid` | FK → users(user_id) | Assigned rider |
| `vendor_id` | `uuid` | NOT NULL, FK → vendors(vendor_id) | Pickup vendor |
| `status` | `varchar(20)` | NOT NULL, DEFAULT 'pending' | `pending`, `assigned`, `picked_up`, `in_transit`, `delivered`, `failed`, `returned` |
| `tracking_number` | `varchar(50)` | | Tracking number |
| `carrier` | `varchar(50)` | | Delivery carrier name |
| `pickup_address` | `jsonb` | NOT NULL | Vendor pickup address |
| `delivery_address` | `jsonb` | NOT NULL | Customer delivery address |
| `estimated_delivery` | `timestamptz` | | Estimated delivery time |
| `actual_delivery` | `timestamptz` | | Actual delivery time |
| `delivery_fee` | `bigint` | DEFAULT 0 | Delivery fee charged |
| `delivery_proof` | `jsonb` | | Proof of delivery (photo, signature, location) |
| `failure_reason` | `varchar(50)` | | `customer_unavailable`, `wrong_address`, `refused`, `damaged` |
| `failure_notes` | `text` | | Failure description |
| `attempt_count` | `integer` | DEFAULT 0 | Delivery attempts |
| `max_attempts` | `integer` | DEFAULT 3 | Maximum attempts |
| `metadata` | `jsonb` | DEFAULT '{}' | Additional delivery data |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT now() | Record creation |
| `updated_at` | `timestamptz` | NOT NULL, DEFAULT now() | Last update |
| `assigned_at` | `timestamptz` | | When rider assigned |
| `picked_up_at` | `timestamptz` | | Pickup time |
| `delivered_at` | `timestamptz` | | Delivery completion |

**Indexes:**
- `uniq_deliveries_order_id` UNIQUE on (`order_id`)
- `idx_deliveries_rider_id` on (`rider_id`)
- `idx_deliveries_vendor_id` on (`vendor_id`)
- `idx_deliveries_status` on (`status`)
- `idx_deliveries_tracking_number` on (`tracking_number`)

---

### `delivery_tracking`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `tracking_id` | `uuid` | PK | Tracking event identifier |
| `delivery_id` | `uuid` | NOT NULL, FK → deliveries(delivery_id) | Delivery |
| `status` | `varchar(20)` | NOT NULL | Status at this event |
| `latitude` | `decimal(10,7)` | | Rider GPS latitude |
| `longitude` | `decimal(10,7)` | | Rider GPS longitude |
| `location_name` | `varchar(255)` | | Human-readable location |
| `notes` | `text` | | Event notes |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT now() | Event timestamp |

**Indexes:**
- `idx_delivery_tracking_delivery_id` on (`delivery_id`)
- `idx_delivery_tracking_created_at` on (`created_at`)

---

### `rider_locations`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `rider_id` | `uuid` | PK, FK → users(user_id) | Rider |
| `latitude` | `decimal(10,7)` | NOT NULL | Current latitude |
| `longitude` | `decimal(10,7)` | NOT NULL | Current longitude |
| `accuracy_meters` | `integer` | | GPS accuracy |
| `speed_kmh` | `decimal(5,2)` | | Current speed |
| `heading` | `decimal(5,2)` | | Direction (0-360°) |
| `is_online` | `boolean` | DEFAULT false | Rider online status |
| `updated_at` | `timestamptz` | NOT NULL, DEFAULT now() | Last location update |

**Indexes:**
- `idx_rider_locations_updated_at` on (`updated_at`)
- `idx_rider_locations_is_online` on (`is_online`) WHERE is_online = true

---

### `rider_earnings`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `earning_id` | `uuid` | PK | Earning identifier |
| `rider_id` | `uuid` | NOT NULL, FK → users(user_id) | Rider |
| `delivery_id` | `uuid` | NOT NULL, FK → deliveries(delivery_id) | Related delivery |
| `amount` | `bigint` | NOT NULL | Earning amount |
| `currency` | `varchar(3)` | NOT NULL, DEFAULT 'YER' | Currency |
| `type` | `varchar(20)` | NOT NULL | `delivery_fee`, `tip`, `bonus`, `penalty` |
| `status` | `varchar(20)` | NOT NULL, DEFAULT 'pending' | `pending`, `paid`, `withheld` |
| `payout_id` | `uuid` | | Related payout batch |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT now() | Earning time |

**Indexes:**
- `idx_rider_earnings_rider_id` on (`rider_id`)
- `idx_rider_earnings_status` on (`status`)
- `idx_rider_earnings_created_at` on (`created_at`)

---

## 7. Reviews & Ratings

### `reviews`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `review_id` | `uuid` | PK | Review identifier |
| `product_id` | `uuid` | NOT NULL, FK → products(product_id) | Reviewed product |
| `user_id` | `uuid` | NOT NULL, FK → users(user_id) | Reviewer |
| `order_id` | `uuid` | NOT NULL, FK → orders(order_id) | Related order (verified purchase) |
| `rating` | `smallint` | NOT NULL, CHECK (1-5) | Star rating |
| `title` | `varchar(200)` | | Review title |
| `comment` | `text` | | Review text |
| `images` | `text[]` | | Array of image URLs |
| `is_verified_purchase` | `boolean` | DEFAULT true | Verified purchase flag |
| `is_anonymous` | `boolean` | DEFAULT false | Post anonymously |
| `helpful_count` | `integer` | DEFAULT 0 | Helpful votes count |
| `vendor_reply` | `text` | | Vendor response |
| `vendor_replied_at` | `timestamptz` | | Vendor reply timestamp |
| `status` | `varchar(20)` | DEFAULT 'published' | `published`, `hidden`, `flagged`, `deleted` |
| `flag_reason` | `text` | | Why review was flagged |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT now() | Review time |
| `updated_at` | `timestamptz` | NOT NULL, DEFAULT now() | Last update |
| `deleted_at` | `timestamptz` | | Soft delete |

**Indexes:**
- `uniq_reviews_user_product` UNIQUE on (`user_id`, `product_id`)
- `idx_reviews_product_id` on (`product_id`)
- `idx_reviews_user_id` on (`user_id`)
- `idx_reviews_rating` on (`rating`)
- `idx_reviews_created_at` on (`created_at`)
- `idx_reviews_status` on (`status`)

---

### `review_helpful`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `review_id` | `uuid` | NOT NULL, FK → reviews(review_id) | Review |
| `user_id` | `uuid` | NOT NULL, FK → users(user_id) | Voter |
| `is_helpful` | `boolean` | NOT NULL | Helpful or not |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT now() | Vote time |

**Constraints:**
- PK on (`review_id`, `user_id`)

---

### `vendor_reviews`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `review_id` | `uuid` | PK | Vendor review identifier |
| `vendor_id` | `uuid` | NOT NULL, FK → vendors(vendor_id) | Reviewed vendor |
| `user_id` | `uuid` | NOT NULL, FK → users(user_id) | Reviewer |
| `order_id` | `uuid` | NOT NULL, FK → orders(order_id) | Related order |
| `overall_rating` | `smallint` | NOT NULL, CHECK (1-5) | Overall rating |
| `quality_rating` | `smallint` | CHECK (1-5) | Quality rating |
| `delivery_rating` | `smallint` | CHECK (1-5) | Delivery speed rating |
| `service_rating` | `smallint` | CHECK (1-5) | Service rating |
| `comment` | `text` | | Review text |
| `vendor_reply` | `text` | | Vendor response |
| `vendor_replied_at` | `timestamptz` | | Reply timestamp |
| `status` | `varchar(20)` | DEFAULT 'published' | `published`, `hidden`, `flagged` |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT now() | Review time |

**Indexes:**
- `idx_vendor_reviews_vendor_id` on (`vendor_id`)
- `idx_vendor_reviews_user_id` on (`user_id`)

---

## 8. Coupons

### `coupons`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `coupon_id` | `uuid` | PK | Coupon identifier |
| `code` | `varchar(50)` | UNIQUE, NOT NULL | Coupon code |
| `description` | `text` | | Coupon description |
| `description_ar` | `text` | | Arabic description |
| `discount_type` | `varchar(20)` | NOT NULL | `percentage`, `fixed_amount`, `free_shipping` |
| `discount_value` | `bigint` | NOT NULL | Discount amount or percentage |
| `max_discount` | `bigint` | | Maximum discount cap (for percentage) |
| `min_order_amount` | `bigint` | DEFAULT 0 | Minimum order value |
| `usage_limit` | `integer` | | Total usage limit (NULL = unlimited) |
| `per_user_limit` | `integer` | DEFAULT 1 | Max uses per user |
| `usage_count` | `integer` | DEFAULT 0 | Current usage count |
| `applicable_categories` | `uuid[]` | | Applicable category IDs (empty = all) |
| `applicable_vendors` | `uuid[]` | | Applicable vendor IDs (empty = all) |
| `excluded_products` | `uuid[]` | | Excluded product IDs |
| `valid_from` | `timestamptz` | NOT NULL | Start date |
| `valid_until` | `timestamptz` | NOT NULL | End date |
| `is_active` | `boolean` | DEFAULT true | Active status |
| `created_by` | `uuid` | FK → users(user_id) | Admin who created |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT now() | Creation time |
| `updated_at` | `timestamptz` | NOT NULL, DEFAULT now() | Last update |

**Indexes:**
- `uniq_coupons_code` UNIQUE on (`code`)
- `idx_coupons_valid` on (`valid_from`, `valid_until`)
- `idx_coupons_is_active` on (`is_active`)

---

### `coupon_usage`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `usage_id` | `uuid` | PK | Usage identifier |
| `coupon_id` | `uuid` | NOT NULL, FK → coupons(coupon_id) | Coupon used |
| `user_id` | `uuid` | NOT NULL, FK → users(user_id) | User who used it |
| `order_id` | `uuid` | NOT NULL, FK → orders(order_id) | Related order |
| `discount_amount` | `bigint` | NOT NULL | Actual discount applied |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT now() | Usage time |

**Indexes:**
- `idx_coupon_usage_coupon_id` on (`coupon_id`)
- `idx_coupon_usage_user_id` on (`user_id`)
- `idx_coupon_usage_order_id` on (`order_id`)

---

## 9. Notifications

### `notifications`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `notification_id` | `uuid` | PK | Notification identifier |
| `user_id` | `uuid` | NOT NULL, FK → users(user_id) | Recipient |
| `type` | `varchar(30)` | NOT NULL | `order_update`, `payment`, `promotion`, `system`, `review`, `delivery`, `wallet` |
| `title` | `varchar(200)` | NOT NULL | Notification title |
| `title_ar` | `varchar(200)` | NOT NULL | Arabic title |
| `body` | `text` | NOT NULL | Notification body |
| `body_ar` | `text` | NOT NULL | Arabic body |
| `data` | `jsonb` | | Deep link / payload data |
| `channel` | `varchar(10)` | NOT NULL | `push`, `email`, `sms`, `in_app` |
| `is_read` | `boolean` | DEFAULT false | Read status |
| `read_at` | `timestamptz` | | When marked read |
| `sent_at` | `timestamptz` | | When sent to channel |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT now() | Creation time |

**Indexes:**
- `idx_notifications_user_id` on (`user_id`)
- `idx_notifications_type` on (`type`)
- `idx_notifications_is_read` on (`user_id`, `is_read`)
- `idx_notifications_created_at` on (`created_at`)

---

### `notification_preferences`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `user_id` | `uuid` | PK, FK → users(user_id) | User |
| `email_order_updates` | `boolean` | DEFAULT true | Email: order updates |
| `email_promotions` | `boolean` | DEFAULT false | Email: promotions |
| `email_newsletter` | `boolean` | DEFAULT true | Email: newsletter |
| `push_order_updates` | `boolean` | DEFAULT true | Push: order updates |
| `push_promotions` | `boolean` | DEFAULT true | Push: promotions |
| `push_price_drops` | `boolean` | DEFAULT true | Push: price drops |
| `sms_order_updates` | `boolean` | DEFAULT true | SMS: order updates |
| `sms_promotions` | `boolean` | DEFAULT false | SMS: promotions |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT now() | Record creation |
| `updated_at` | `timestamptz` | NOT NULL, DEFAULT now() | Last update |

---

### `device_tokens`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `token_id` | `uuid` | PK | Token identifier |
| `user_id` | `uuid` | NOT NULL, FK → users(user_id) | Device owner |
| `token` | `text` | UNIQUE, NOT NULL | FCM/APNs device token |
| `platform` | `varchar(10)` | NOT NULL | `android`, `ios`, `web` |
| `device_name` | `varchar(255)` | | Device description |
| `is_active` | `boolean` | DEFAULT true | Active status |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT now() | Registration time |
| `last_used_at` | `timestamptz` | | Last push delivery |

**Indexes:**
- `uniq_device_tokens_token` UNIQUE on (`token`)
- `idx_device_tokens_user_id` on (`user_id`)

---

## 10. Search & Analytics

### `search_queries`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `query_id` | `uuid` | PK | Query identifier |
| `user_id` | `uuid` | FK → users(user_id) | Searching user (nullable) |
| `query_text` | `text` | NOT NULL | Search query |
| `results_count` | `integer` | | Number of results returned |
| `clicked_product_id` | `uuid` | FK → products(product_id) | Product clicked (if any) |
| `session_id` | `varchar(255)` | | Anonymous session ID |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT now() | Query timestamp |

**Indexes:**
- `idx_search_queries_query_text` on (`query_text`)
- `idx_search_queries_created_at` on (`created_at`)

---

### `page_views`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `view_id` | `uuid` | PK | View identifier |
| `user_id` | `uuid` | FK → users(user_id) | Viewer (nullable) |
| `session_id` | `varchar(255)` | | Anonymous session |
| `page_type` | `varchar(20)` | NOT NULL | `home`, `product`, `category`, `search`, `cart`, `checkout` |
| `page_id` | `uuid` | | Entity ID (product_id, category_id, etc.) |
| `referrer` | `text` | | Referrer URL |
| `user_agent` | `text` | | Client user agent |
| `ip_address` | `inet` | | Client IP |
| `country` | `varchar(2)` | | ISO country code |
| `device_type` | `varchar(10)` | | `desktop`, `mobile`, `tablet` |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT now() | View timestamp |

**Indexes:**
- `idx_page_views_user_id` on (`user_id`)
- `idx_page_views_page_type` on (`page_type`)
- `idx_page_views_created_at` on (`created_at`)

---

## 11. Admin & Audit

### `audit_logs`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `log_id` | `uuid` | PK | Log identifier |
| `user_id` | `uuid` | FK → users(user_id) | Actor (nullable for system) |
| `action` | `varchar(50)` | NOT NULL | Action performed |
| `resource_type` | `varchar(50)` | NOT NULL | Entity type |
| `resource_id` | `uuid` | | Entity ID |
| `old_values` | `jsonb` | | Previous state |
| `new_values` | `jsonb` | | New state |
| `ip_address` | `inet` | | Actor IP |
| `user_agent` | `text` | | Actor user agent |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT now() | Action timestamp |

**Indexes:**
- `idx_audit_logs_user_id` on (`user_id`)
- `idx_audit_logs_resource` on (`resource_type`, `resource_id`)
- `idx_audit_logs_action` on (`action`)
- `idx_audit_logs_created_at` on (`created_at`)

---

### `admin_roles`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `role_id` | `uuid` | PK | Role identifier |
| `name` | `varchar(50)` | UNIQUE, NOT NULL | Role name |
| `description` | `text` | | Role description |
| `permissions` | `text[]` | NOT NULL | Array of permission strings |
| `is_system` | `boolean` | DEFAULT false | System role (cannot delete) |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT now() | Creation time |
| `updated_at` | `timestamptz` | NOT NULL, DEFAULT now() | Last update |

---

### `platform_settings`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `setting_key` | `varchar(100)` | PK | Setting key |
| `setting_value` | `text` | NOT NULL | Setting value (JSON string) |
| `setting_type` | `varchar(20)` | NOT NULL | `string`, `number`, `boolean`, `json` |
| `description` | `text` | | Setting description |
| `is_public` | `boolean` | DEFAULT false | Client-visible setting |
| `updated_by` | `uuid` | FK → users(user_id) | Last editor |
| `updated_at` | `timestamptz` | NOT NULL, DEFAULT now() | Last update |

---

## 12. System Tables

### `rate_limits`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `key` | `varchar(255)` | PK | Rate limit key (user_id:endpoint) |
| `count` | `integer` | NOT NULL | Request count |
| `window_start` | `timestamptz` | NOT NULL | Window start time |
| `expires_at` | `timestamptz` | NOT NULL | Key expiration |

**Indexes:**
- `idx_rate_limits_expires_at` on (`expires_at`)

---

### `scheduled_tasks`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `task_id` | `uuid` | PK | Task identifier |
| `task_type` | `varchar(50)` | NOT NULL | Task type |
| `payload` | `jsonb` | NOT NULL | Task data |
| `scheduled_at` | `timestamptz` | NOT NULL | When to execute |
| `status` | `varchar(20)` | NOT NULL, DEFAULT 'pending' | `pending`, `processing`, `completed`, `failed` |
| `result` | `jsonb` | | Task result |
| `retry_count` | `integer` | DEFAULT 0 | Retries so far |
| `max_retries` | `integer` | DEFAULT 3 | Maximum retries |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT now() | Creation time |
| `processed_at` | `timestamptz` | | Processing time |

**Indexes:**
- `idx_scheduled_tasks_scheduled_at` on (`scheduled_at`) WHERE status = 'pending'
- `idx_scheduled_tasks_status` on (`status`)

---

### `idempotency_keys`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `key` | `varchar(255)` | PK | Idempotency key |
| `user_id` | `uuid` | NOT NULL, FK → users(user_id) | Requesting user |
| `endpoint` | `varchar(100)` | NOT NULL | API endpoint |
| `response_status` | `integer` | | HTTP response status |
| `response_body` | `jsonb` | | Cached response |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT now() | Key creation |
| `expires_at` | `timestamptz` | NOT NULL | Key expiration |

**Indexes:**
- `idx_idempotency_keys_user_endpoint` on (`user_id`, `endpoint`)
- `idx_idempotency_keys_expires_at` on (`expires_at`)

---

### `feature_flags`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `flag_id` | `uuid` | PK | Flag identifier |
| `name` | `varchar(100)` | UNIQUE, NOT NULL | Flag name |
| `description` | `text` | | Flag description |
| `is_enabled` | `boolean` | DEFAULT false | Global enable/disable |
| `rollout_percentage` | `integer` | DEFAULT 100 | % of users who see feature (0-100) |
| `allowed_roles` | `text[]` | | Roles that always see feature |
| `allowed_users` | `uuid[]` | | Specific users who see feature |
| `denied_users` | `uuid[]` | | Users who never see feature |
| `metadata` | `jsonb` | DEFAULT '{}' | A/B test config, etc. |
| `created_at` | `timestamptz` | NOT NULL, DEFAULT now() | Creation time |
| `updated_at` | `timestamptz` | NOT NULL, DEFAULT now() | Last update |

**Indexes:**
- `uniq_feature_flags_name` UNIQUE on (`name`)

---

## Entity Relationship Summary

```
users ─┬─ user_sessions
       ├─ user_addresses
       ├─ verification_tokens
       ├─ wallets ─┬─ wallet_transactions
       │           ├─ wallet_topups
       │           ├─ wallet_withdrawals
       │           └─ wallet_transfers
       ├─ orders ─┬─ order_items ─── products ─┬─ product_variants
       │          │                            ├─ product_images
       │          ├─ order_status_history       └─ product_inventory_log
       │          └─ payments ── refunds
       ├─ vendors ─┬─ vendor_documents
       │           ├─ vendor_bank_accounts
       │           └─ vendor_reviews
       ├─ reviews
       ├─ notifications
       ├─ device_tokens
       ├─ page_views
       └─ search_queries

products ─── categories (self-referential tree)
orders ─── coupons ─── coupon_usage
deliveries ─── delivery_tracking
            ─── rider_locations
            ─── rider_earnings

admin_roles
platform_settings
audit_logs
rate_limits
scheduled_tasks
idempotency_keys
feature_flags
```

---

## Total Table Count: 38

| # | Table | Description |
|---|-------|-------------|
| 1 | users | Core user accounts |
| 2 | user_sessions | Active sessions |
| 3 | user_addresses | Delivery addresses |
| 4 | verification_tokens | Email/phone/password reset tokens |
| 5 | vendors | Vendor profiles |
| 6 | vendor_documents | Vendor verification documents |
| 7 | vendor_bank_accounts | Vendor payout accounts |
| 8 | categories | Product categories (tree) |
| 9 | products | Product listings |
| 10 | product_variants | Product variations |
| 11 | product_images | Product images |
| 12 | product_inventory_log | Stock change history |
| 13 | orders | Customer orders |
| 14 | order_items | Order line items |
| 15 | order_status_history | Order status transitions |
| 16 | payments | Payment transactions |
| 17 | refunds | Refund requests |
| 18 | wallets | User/vendor wallets |
| 19 | wallet_transactions | Wallet ledger |
| 20 | wallet_topups | Wallet deposit requests |
| 21 | wallet_withdrawals | Withdrawal requests |
| 22 | wallet_transfers | Peer-to-peer transfers |
| 23 | deliveries | Delivery assignments |
| 24 | delivery_tracking | Delivery GPS tracking |
| 25 | rider_locations | Real-time rider positions |
| 26 | rider_earnings | Rider payment records |
| 27 | reviews | Product reviews |
| 28 | review_helpful | Helpful vote records |
| 29 | vendor_reviews | Vendor reviews |
| 30 | coupons | Discount coupons |
| 31 | coupon_usage | Coupon redemption history |
| 32 | notifications | User notifications |
| 33 | notification_preferences | Notification settings |
| 34 | device_tokens | FCM/APNs tokens |
| 35 | search_queries | Search analytics |
| 36 | page_views | Page view analytics |
| 37 | audit_logs | System audit trail |
| 38 | admin_roles | Admin role definitions |
| 39 | platform_settings | Platform configuration |
| 40 | rate_limits | API rate limiting |
| 41 | scheduled_tasks | Background task queue |
| 42 | idempotency_keys | Request deduplication |
| 43 | feature_flags | Feature toggle system |
