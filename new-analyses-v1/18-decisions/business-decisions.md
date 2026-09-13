# Business Rule and Policy Decisions — YemenMart

**Document ID:** YM-BIZ-001
**Version:** 1.0
**Status:** Active
**Date:** 2026-09-13
**Language:** English

---

## Table of Contents

1. [Payment Business Rules](#1-payment-business-rules)
2. [Authentication Business Rules](#2-authentication-business-rules)
3. [Vendor Business Rules](#3-vendor-business-rules)
4. [Order Business Rules](#4-order-business-rules)
5. [Delivery Business Rules](#5-delivery-business-rules)
6. [Product Catalog Business Rules](#6-product-catalog-business-rules)
7. [Store & Follow Business Rules](#7-store--follow-business-rules)
8. [Review & Trust Business Rules](#8-review--trust-business-rules)
9. [Coupon & Discount Business Rules](#9-coupon--discount-business-rules)
10. [Finance & Settlement Business Rules](#10-finance--settlement-business-rules)
11. [Support Business Rules](#11-support-business-rules)
12. [Content Business Rules](#12-content-business-rules)
13. [System & Compliance Business Rules](#13-system--compliance-business-rules)
14. [Business Rule Decision Log](#14-business-rule-decision-log)

---

## 1. Payment Business Rules

### PAY-001: Wallet-Only Payments
- **Decision:** All payments through integrated mobile wallets
- **Rationale:** Low credit card penetration in Yemen; mobile wallets are dominant
- **Alternatives Rejected:** Credit cards, BNPL, bank transfers, COD
- **Impact:** Customers must fund wallet before purchase

### PAY-002: No Cash on Delivery (COD)
- **Decision:** COD not supported at platform launch
- **Rationale:** Reduces cash handling risk, enables escrow, simplifies settlement
- **Alternatives Rejected:** COD with vendor approval, COD for select categories
- **Future Consideration:** COD for rural areas in Phase 3

### PAY-003: 7-Day Escrow Hold
- **Decision:** Customer funds held in escrow for 7 days after delivery
- **Rationale:** Customer protection period, dispute resolution window
- **Alternatives Rejected:** 3 days (too short), 14 days (too long), instant release
- **Exception:** Instant release for "Trusted Seller" badge (>95% positive, >100 orders)

### PAY-004: Multi-Currency Support
- **Decision:** Support YER, SAR, USD
- **Rationale:** Yemen market uses multiple currencies; Saudi cross-border commerce
- **Exchange Rate:** Updated daily from central bank APIs
- **Conversion:** Real-time at transaction time

### PAY-005: Minimum Transaction Amount
- **Decision:** 100 YER minimum per transaction
- **Rationale:** Transaction costs must be economically viable
- **Exception:** Platform-initiated transactions (refunds, adjustments)

### PAY-006: Maximum Transaction Amount
- **Decision:** 5,000,000 YER maximum per transaction
- **Rationale:** Fraud prevention, regulatory compliance
- **Exception:** Limit increase with KYC upgrade

### PAY-007: Wallet Top-Up Limits
- **Decision:** 500,000 YER/day standard; 2,000,000 YER/day verified
- **Rationale:** Anti-money laundering compliance
- **Enforcement:** Daily cumulative tracking per calendar day

### PAY-008: Refund to Wallet Only
- **Decision:** All refunds credited back to original wallet
- **Rationale:** Maintains wallet ecosystem, simplifies processing
- **Exception:** None — no bank transfer refunds

### PAY-009: Failed Payment Retry Limit
- **Decision:** 3 retries per transaction before re-authentication
- **Rationale:** Prevents brute-force payment attempts
- **Exception:** System errors (timeout, network) don't count

### PAY-010: Platform Commission
- **Decision:** Commission deducted automatically before vendor settlement
- **Rationale:** Primary revenue stream
- **Rate:** Configurable per vendor tier (see Vendor rules)

---

## 2. Authentication Business Rules

### AUTH-001: SMS/WhatsApp OTP
- **Decision:** OTP delivered via SMS or WhatsApp based on user preference
- **Rationale:** Phone numbers more reliable than email in Yemen
- **Delivery SLA:** Within 30 seconds
- **Expiry:** 5 minutes

### AUTH-002: Phone as Primary Identifier
- **Decision:** Phone number is primary identifier for all users
- **Rationale:** High mobile penetration, low email usage in Yemen
- **Constraint:** Phone cannot be changed after verification
- **Format:** +967XXXXXXXXX

### AUTH-003: Email Optional
- **Decision:** Email is optional for all user types
- **Rationale:** Low email penetration in target market
- **Usage:** Password recovery alternative, marketing emails

### AUTH-004: No Social Login
- **Decision:** No Google, Facebook, or Apple login
- **Rationale:** Reduce dependencies, simplify auth flow
- **Future Consideration:** Phase 3 if user demand

### AUTH-005: Session Duration
- **Decision:** 24-hour session with refresh token rotation
- **Rationale:** Balance convenience and security
- **Refresh Token:** 7-day expiry, rotated on use

### AUTH-006: OTP Resend Limit
- **Decision:** Maximum 3 OTP resends per 10-minute window
- **Rationale:** Prevents SMS cost abuse
- **Enforcement:** Rate limit per phone number

---

## 3. Vendor Business Rules

### VEND-001: KYC Mandatory
- **Decision:** KYC verification required before product listing
- **Rationale:** Trust, regulatory compliance
- **Required Documents:** National ID/passport, business registration
- **Review SLA:** 48 hours

### VEND-002: KYC Rejection Handling
- **Decision:** Rejection allows re-submission after 7 days
- **Rationale:** Give vendors time to gather correct documents
- **Communication:** Clear reason and re-submission guidance

### VEND-003: Vendor Tiers
- **Decision:** 3 vendor tiers based on performance
- **Tiers:** Standard, Premium, Trusted Seller
- **Benefits:** Commission rates, settlement frequency, visibility boost

| Tier | Requirements | Commission | Settlement |
|------|-------------|------------|------------|
| Standard | KYC approved | 15% | Weekly |
| Premium | >90% positive, >50 orders | 12% | Weekly |
| Trusted Seller | >95% positive, >100 orders | 10% | Bi-weekly |

### VEND-004: Vendor Suspension
- **Decision:** Automatic suspension for policy violations
- **Triggers:** >5% cancellation rate, >3 complaints/month, fraud
- **Process:** Warning → Suspension → Review → Reactivation/Ban

### VEND-005: Product Limits
- **Decision:** Maximum 1000 active products per vendor
- **Rationale:** Prevent catalog bloat, ensure quality
- **Exception:** Premium vendors get 5000 limit

---

## 4. Order Business Rules

### ORD-001: Master/Sub-Order Architecture
- **Decision:** Multi-vendor orders split into master + sub-orders
- **Rationale:** Independent vendor fulfillment, separate tracking
- **Customer Impact:** Multiple order entries for multi-vendor cart

### ORD-002: 17-State Order System
- **Decision:** 17 distinct order states
- **Rationale:** Granular tracking, complex scenario support
- **States:** PENDING → PAYMENT_PROCESSING → PAID → ... → COMPLETED

### ORD-003: Order Cancellation Windows
- **Decision:** Customer can cancel until VENDOR_ACCEPTED state
- **Rationale:** Vendor hasn't started preparation
- **After Acceptance:** Must request return instead

### ORD-004: Partial Order Cancellation
- **Decision:** Support per-vendor sub-order cancellation
- **Rationale:** Multi-vendor orders may have different readiness
- **Impact:** Master order updated, other sub-orders unaffected

### ORD-005: Return Window
- **Decision:** 7-day return window after delivery
- **Rationale:** Aligns with escrow hold period
- **Conditions:** Item unused, original packaging, non-perishable

### ORD-006: Delivery Code Verification
- **Decision:** OTP-based delivery confirmation
- **Rationale:** Proof of delivery, prevents fraud
- **Flow:** Rider generates code → customer shares at delivery → order confirmed

---

## 5. Delivery Business Rules

### DEL-001: Delivery Provider Marketplace
- **Decision:** Multiple delivery providers, vendor selection
- **Rationale:** Competition drives better pricing and service
- **Providers:** Local delivery companies integrated via API

### DEL-002: Delivery Fee Calculation
- **Decision:** Distance-based + weight-based pricing
- **Rationale:** Fair pricing for different order sizes
- **Minimum Fee:** 100 YER
- **Free Delivery Threshold:** Configurable per vendor/store

### DEL-003: Delivery Zones
- **Decision:** City-based delivery zones
- **Rationale:** Simplified logistics for Yemen market
- **Coverage:** Major cities at launch, expand gradually

### DEL-004: Delivery Time Estimates
- **Decision:** Estimated delivery window (not real-time tracking)
- **Rationale:** Infrastructure limitations in Yemen
- **Estimate:** 1-3 business days (urban), 3-7 days (rural)

### DEL-005: Failed Delivery Attempt
- **Decision:** 2 retry attempts before return to vendor
- **Rationale:** Balance customer convenience with logistics cost
- **Communication:** SMS notification on each attempt

---

## 6. Product Catalog Business Rules

### CAT-001: Bilingual Content Required
- **Decision:** All product content in Arabic and English
- **Rationale:** Arabic-first market with English-speaking segment
- **Enforcement:** Both fields required for product listing

### CAT-002: Product Image Requirements
- **Decision:** Minimum 1, maximum 10 images per product
- **Rationale:** Visual commerce requires good imagery
- **Formats:** JPEG, PNG, WebP
- **Max Size:** 5MB per image
- **Minimum Resolution:** 800×800px

### CAT-003: Category Hierarchy
- **Decision:** 3-level category hierarchy
- **Rationale:** Balance between discoverability and complexity
- **Structure:** Level 1 (Department) → Level 2 (Category) → Level 3 (Subcategory)

### CAT-004: Product Variants
- **Decision:** Support size, color, and custom attribute variants
- **Rationale:** Common e-commerce requirement
- **Implementation:** Variant-level SKU, price, and stock

### CAT-005: Product Status Flow
- **Decision:** DRAFT → PENDING_REVIEW → ACTIVE → INACTIVE → ARCHIVED
- **Rationale:** Quality control before customer visibility
- **Auto-Approval:** Trusted Sellers get auto-approval

### CAT-006: Product Search Ranking
- **Decision:** Multi-factor ranking algorithm
- **Factors:** Relevance, rating, sales, recency, vendor tier
- **Boost:** Premium/Trusted Seller products get visibility boost

---

## 7. Store & Follow Business Rules

### STORE-001: Store Creation
- **Decision:** One store per vendor (KYC-approved only)
- **Rationale:** Quality over quantity
- **Customization:** Name, logo, description, theme color

### STORE-002: Store Follow/Unfollow
- **Decision:** Customers can follow/unfollow stores
- **Rationale:** Build customer-vendor relationships
- **Benefits:** Followed store products prioritized in feed

### STORE-003: Store Ratings
- **Decision:** Aggregate rating from all vendor products
- **Rationale:** Overall vendor quality indicator
- **Calculation:** Weighted average of product ratings

---

## 8. Review & Trust Business Rules

### REV-001: Review Eligibility
- **Decision:** Only verified purchasers can review
- **Rationale:** Authentic reviews build trust
- **Window:** 7 days after delivery confirmation

### REV-002: Review Content
- **Decision:** Star rating (1-5) + text review + optional photo
- **Rationale:** Rich feedback for other customers
- **Moderation:** Auto-filter profanity, manual review for disputes

### REV-003: Vendor Response
- **Decision:** Vendors can respond to reviews
- **Rationale:** Allows vendor to address concerns publicly
- **SLA:** Response within 48 hours recommended

### REV-004: Review Manipulation Prevention
- **Decision:** Anti-fraud measures for reviews
- **Measures:** One review per product per user, IP monitoring, pattern detection
- **Penalty:** Fake reviews result in account suspension

---

## 9. Coupon & Discount Business Rules

### COUP-001: Coupon Types
- **Decision:** 3 coupon types supported
- **Types:** Percentage discount, Fixed amount, Free delivery
- **Stacking:** Only one coupon per order

### COUP-002: Coupon Creation
- **Decision:** Admins and vendors can create coupons
- **Admin:** Platform-wide coupons
- **Vendor:** Store-specific coupons
- **Approval:** Vendor coupons require admin approval

### COUP-003: Coupon Constraints
- **Decision:** Multiple constraint types supported
- **Constraints:** Min order amount, max discount, usage limit, expiry date, applicable categories

### COUP-004: Coupon Abuse Prevention
- **Decision:** Prevent coupon stacking and sharing abuse
- **Measures:** One coupon per order, unique codes, usage tracking
- **Monitoring:** Flag accounts with excessive coupon usage

---

## 10. Finance & Settlement Business Rules

### FIN-001: Vendor Settlement Cycle
- **Decision:** Weekly settlements for Standard/Premium vendors
- **Rationale:** Predictable cash flow, reduced administrative overhead
- **Exception:** Bi-weekly for Trusted Sellers (lower commission offset)

### FIN-002: Commission Calculation
- **Decision:** Commission deducted at settlement time
- **Rate:** Per vendor tier (see VEND-003)
- **Base:** Order total minus delivery fees

### FIN-003: Payout Methods
- **Decision:** Bank transfer or mobile wallet
- **Rationale:** Flexibility for vendors
- **Minimum Payout:** 10,000 YER
- **Currency:** Vendor's configured settlement currency

### FIN-004: Dispute Financial Hold
- **Decision:** Funds held during active disputes
- **Rationale:** Protect customer and vendor during resolution
- **Release:** After dispute resolution (favor vendor or customer)

---

## 11. Support Business Rules

### SUP-001: Ticket Priority
- **Decision:** 4-level priority system
- **Levels:** Critical, High, Medium, Low
- **SLA:** 4h, 24h, 48h, 72h respectively

### SUP-002: Ticket Assignment
- **Decision:** Auto-assignment based on category and agent availability
- **Categories:** Order issues, Payment, Technical, Account, General
- **Escalation:** Auto-escalate if SLA breach imminent

### SUP-003: Escalation Path
- **Decision:** Agent → Team Lead → Manager → CTO
- **Trigger:** SLA breach, customer complaint, complex issue
- **Communication:** Automatic notification at each level

---

## 12. Content Business Rules

### CMS-001: Page Management
- **Decision:** Admin-managed CMS pages
- **Types:** Static pages, FAQ, blog posts
- **Bilingual:** All content in Arabic and English

### CMS-002: Banner Management
- **Decision:** Configurable home page banners
- **Scheduling:** Date-range based activation
- **Targeting:** All users or specific segments

### CMS-003: Notification Templates
- **Decision:** Admin-editable notification templates
- **Variables:** Dynamic content via {{variable}} syntax
- **Channels:** SMS, WhatsApp, push, email

---

## 13. System & Compliance Business Rules

### SYS-001: Data Retention
- **Decision:** 7-year retention for financial data
- **Rationale:** Regulatory compliance
- **Purge:** Non-financial data after 3 years of inactivity

### SYS-002: Audit Logging
- **Decision:** Immutable audit trail for all state changes
- **Rationale:** Security, compliance, debugging
- **Storage:** Append-only table with hash chain

### SYS-003: Rate Limiting
- **Decision:** Per-endpoint rate limits
- **Rationale:** Prevent abuse, ensure availability
- **Enforcement:** Redis-based sliding window

### SYS-004: Data Backup
- **Decision:** Daily automated backups with 30-day retention
- **Rationale:** Disaster recovery
- **Testing:** Monthly backup restoration test

### SYS-005: GDPR Compliance
- **Decision:** Data portability and deletion support
- **Rationale:** Regulatory requirement
- **Implementation:** User data export, account deletion

---

## 14. Business Rule Decision Log

| Date | Rule ID | Decision | Rationale |
|------|---------|----------|-----------|
| 2026-09-01 | PAY-001 | Wallet-only payments | Yemen market fit |
| 2026-09-01 | PAY-002 | No COD | Risk reduction |
| 2026-09-01 | PAY-003 | 7-day escrow | Customer protection |
| 2026-09-01 | AUTH-001 | SMS/WhatsApp OTP | Phone penetration |
| 2026-09-01 | AUTH-002 | Phone as identifier | Market reality |
| 2026-09-01 | VEND-001 | KYC mandatory | Trust + compliance |
| 2026-09-01 | ORD-001 | Master/sub orders | Multi-vendor support |
| 2026-09-01 | DEL-001 | Delivery marketplace | Competition |
| 2026-09-01 | CAT-001 | Bilingual content | Arabic-first market |
| 2026-09-01 | REV-001 | Verified purchaser reviews | Authenticity |
| 2026-09-01 | FIN-001 | Weekly settlements | Vendor cash flow |
| 2026-09-01 | SYS-001 | 7-year data retention | Compliance |

---

## Business Rule Validation Matrix

| Rule ID | Validation Method | Enforcement Point | Error Handling |
|---------|------------------|-------------------|----------------|
| PAY-001 | Pre-checkout validation | PaymentService | Reject with message |
| PAY-003 | Timer-based | EscrowService | Auto-release after 7 days |
| AUTH-001 | OTP verification | AuthService | Lock after 3 failures |
| VEND-001 | Status check | VendorService | Block product listing |
| ORD-003 | State machine | OrderService | Reject cancellation |
| DEL-005 | Retry counter | DeliveryService | Return to vendor |
| CAT-001 | Schema validation | ProductService | Reject listing |
| COUP-001 | Usage tracking | CouponService | Reject coupon |

---

## Related Categories

- `01-business-analysis/business-rules.md` - Full business rules catalog
- `01-business-analysis/business-model.md` - Business model overview
- `18-decisions/architecture-decisions.md` - Architecture implications

---

*Source: Business rules from market analysis, stakeholder interviews, and constraint analysis*
