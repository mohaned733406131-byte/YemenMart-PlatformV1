# YemenMart Functional Requirements

## Overview

This document defines **17 functional requirements** (FR-001 to FR-017) covering all 13 building blocks and 23 modules of the YemenMart platform. Each requirement includes acceptance criteria aligned with the 974 test points.

---

## FR-001: User Registration & Authentication

**Module:** M01: Registration & Login
**Priority:** Critical
**Blocks:** Identity & Access

### Description
Users (customers, merchants, delivery agents) register using phone numbers and set a password during registration. Phone + password is the **primary login method**. Phone + OTP (SMS/WhatsApp) is available as a **secondary method for forgot password scenarios only**.

### Acceptance Criteria

| ID | Criterion | Test Points |
|----|-----------|-------------|
| AC-001-01 | Registration requires valid Yemeni phone number (+967) | TP-001 |
| AC-001-02 | Registration requires password with minimum 8 characters | TP-002 |
| AC-001-03 | OTP delivered via SMS within 30 seconds (forgot password only) | TP-003 |
| AC-001-04 | OTP delivered via WhatsApp within 30 seconds (forgot password only) | TP-004 |
| AC-001-05 | OTP expires after 5 minutes | TP-005 |
| AC-001-06 | Maximum 3 OTP resend attempts per 10-minute window | TP-006 |
| AC-001-07 | Account created only after phone verification via OTP | TP-007 |
| AC-001-08 | Duplicate phone number rejected with clear error message | TP-008 |
| AC-001-09 | Password login is primary method (phone + password) | TP-009 |
| AC-001-10 | Forgot password uses phone + OTP verification code only | TP-010 |
| AC-001-11 | Session token issued upon successful authentication | TP-011 |
| AC-001-12 | Session expires after 24 hours of inactivity | TP-012 |

### Business Rules
- BR-001-01: Phone number is primary identifier (Constraint 3)
- BR-001-02: Email is optional (Constraint 4)
- BR-001-03: Primary login is phone + password
- BR-001-04: Forgot password uses phone + OTP verification code only
- BR-001-05: Password must be at least 8 characters with mixed case and numbers

---

## FR-002: Profile Management

**Module:** M02: Profile Management
**Priority:** High
**Blocks:** Identity & Access

### Description
Users manage their profile information including name, phone, email (optional), addresses, and preferences. All content supports Arabic and English.

### Acceptance Criteria

| ID | Criterion | Test Points |
|----|-----------|-------------|
| AC-002-01 | Profile displays in Arabic (default) or English | TP-011 |
| AC-002-02 | Phone number cannot be changed after verification | TP-012 |
| AC-002-03 | Email can be added/changed and requires verification | TP-013 |
| AC-002-04 | Multiple delivery addresses supported (max 10) | TP-014 |
| AC-002-05 | Profile changes logged in audit trail | TP-015 |
| AC-002-06 | Profile image upload supports JPEG/PNG, max 5MB | TP-016 |

### Business Rules
- BR-002-01: RTL layout enforced (Constraint 7)
- BR-002-02: Profile completeness affects search ranking

---

## FR-003: KYC & Vendor Verification

**Module:** M03: KYC & Verification
**Priority:** Critical
**Blocks:** Identity & Access

### Description
Merchants must complete KYC verification before listing products or receiving payments. Verification includes identity documents and business information.

### Acceptance Criteria

| ID | Criterion | Test Points |
|----|-----------|-------------|
| AC-003-01 | KYC submission requires national ID or passport | TP-017 |
| AC-003-02 | Business registration document required | TP-018 |
| AC-003-03 | Bank account information for wallet settlement | TP-019 |
| AC-003-04 | KYC status tracked: PENDING, UNDER_REVIEW, APPROVED, REJECTED | TP-020 |
| AC-003-05 | Merchant actions blocked until KYC APPROVED | TP-021 |
| AC-003-06 | KYC rejection includes reason and re-submission guidance | TP-022 |
| AC-003-07 | KYC documents encrypted at rest and in transit | TP-023 |
| AC-003-08 | KYC review SLA: 48 hours | TP-024 |

### Business Rules
- BR-003-01: KYC mandatory (Constraint 19)
- BR-003-02: Rejection allows re-submission after 7 days

---

## FR-004: Product Catalog Management

**Module:** M04: Product CRUD, M05: Category Management
**Priority:** Critical
**Blocks:** Product Catalog

### Description
Merchants create, read, update, and delete product listings. Products belong to system-defined categories with full Arabic/English support.

### Acceptance Criteria

| ID | Criterion | Test Points |
|----|-----------|-------------|
| AC-004-01 | Product creation requires: title, description, price, images, category | TP-025 |
| AC-004-02 | Minimum 1, maximum 10 images per product | TP-026 |
| AC-004-03 | Product title max 200 characters, Arabic and English | TP-027 |
| AC-004-04 | Price must be > 0 YER (no trial products, Constraint 16) | TP-028 |
| AC-004-05 | No subscription product type available (Constraint 17) | TP-029 |
| AC-004-06 | 40+ system service categories available (Constraint 15) | TP-030 |
| AC-004-07 | Category changes require admin approval | TP-031 |
| AC-004-08 | Product search returns results within 200ms | TP-032 |
| AC-004-09 | Product listing moderation checks for branding violations | TP-033 |
| AC-004-10 | Inventory quantity tracked per product variant | TP-034 |

### Business Rules
- BR-004-01: No trial products (Constraint 16)
- BR-004-02: No subscription products (Constraint 17)
- BR-004-03: No third-party branding (Constraint 18)

---

## FR-005: Inventory Management

**Module:** M06: Inventory Management
**Priority:** High
**Blocks:** Product Catalog

### Description
Merchants manage stock levels, track inventory movements, and receive low-stock alerts.

### Acceptance Criteria

| ID | Criterion | Test Points |
|----|-----------|-------------|
| AC-005-01 | Real-time stock level tracking | TP-035 |
| AC-005-02 | Low-stock alert threshold configurable per product | TP-036 |
| AC-005-03 | Stock reserved when order placed, released on cancel | TP-037 |
| AC-005-04 | Out-of-stock products hidden from search results | TP-038 |
| AC-005-05 | Inventory history log maintained for 12 months | TP-039 |
| AC-005-06 | Bulk inventory update via CSV upload | TP-040 |

---

## FR-006: Store Management & Templates

**Module:** M07: Storefront Setup, M08: Store Templates
**Priority:** High
**Blocks:** Store Management

### Description
Merchants set up their storefront using one of 10+ available templates. Store customization includes branding, layout, and featured products.

### Acceptance Criteria

| ID | Criterion | Test Points |
|----|-----------|-------------|
| AC-006-01 | Minimum 10 store templates available (Constraint 14) | TP-041 |
| AC-006-02 | Template selection during merchant onboarding | TP-042 |
| AC-006-03 | Store name, logo, description configurable | TP-043 |
| AC-006-04 | Store page displays in Arabic (default) and English | TP-044 |
| AC-006-05 | Store URL follows format: yemenmart.com/store/{slug} | TP-045 |
| AC-006-06 | Store policies (return policy) configurable | TP-046 |
| AC-006-07 | Store preview before publish | TP-047 |
| AC-006-08 | Store analytics dashboard (views, orders, revenue) | TP-048 |

### Business Rules
- BR-006-01: Store template must be selected (Constraint 14)
- BR-006-02: Store slug must be unique

---

## FR-007: Search & Discovery

**Module:** M09: Search Engine, M10: Recommendations
**Priority:** High
**Blocks:** Search & Discovery

### Description
Customers search products by keyword, category, price range, and merchant. Recommendations are generated based on browsing and purchase history.

### Acceptance Criteria

| ID | Criterion | Test Points |
|----|-----------|-------------|
| AC-007-01 | Full-text search in Arabic and English | TP-049 |
| AC-007-02 | Search results returned within 200ms (p95) | TP-050 |
| AC-007-03 | Filters: category, price range, merchant, rating | TP-051 |
| AC-007-04 | Sort by: relevance, price (low/high), rating, newest | TP-052 |
| AC-007-05 | Search suggestions auto-complete | TP-053 |
| AC-007-06 | "Related products" displayed on product detail page | TP-054 |
| AC-007-07 | Search analytics tracked for business intelligence | TP-055 |
| AC-007-08 | Zero-result queries logged for catalog improvement | TP-056 |

---

## FR-008: Cart Management

**Module:** M11: Cart Service
**Priority:** Critical
**Blocks:** Cart & Checkout

### Description
Customers add products to cart, adjust quantities, and proceed to checkout. Cart enforces platform constraints on item count and quantities.

### Acceptance Criteria

| ID | Criterion | Test Points |
|----|-----------|-------------|
| AC-008-01 | Cart max 50 items total (Constraint 24) | TP-057 |
| AC-008-02 | Max 10 units per product (Constraint 25) | TP-058 |
| AC-008-03 | Cart persists across sessions (logged-in users) | TP-059 |
| AC-008-04 | Cart groups items by merchant for sub-order preview | TP-060 |
| AC-008-05 | Real-time price calculation (subtotal, VAT, total) | TP-061 |
| AC-008-06 | Out-of-stock items flagged in cart | TP-062 |
| AC-008-07 | Cart timeout after 30 days of inactivity | TP-063 |
| AC-008-08 | Empty cart state displays recommendations | TP-064 |

### Business Rules
- BR-008-01: Cart validates 500–5,000,000 YER range (Constraint 26)
- BR-008-02: Stock reservation at checkout, not cart addition

---

## FR-009: Checkout & Order Placement

**Module:** M12: Checkout Flow
**Priority:** Critical
**Blocks:** Cart & Checkout

### Description
Customers review cart, select delivery address, confirm wallet balance, and place order. System creates master order and sub-orders per merchant.

### Acceptance Criteria

| ID | Criterion | Test Points |
|----|-----------|-------------|
| AC-009-01 | Checkout requires wallet balance >= order total | TP-065 |
| AC-009-02 | Delivery address selection/entry required | TP-066 |
| AC-009-03 | Order summary shows items, VAT, total before confirm | TP-067 |
| AC-009-04 | Master order created with unique ID | TP-068 |
| AC-009-05 | Sub-orders created per merchant | TP-069 |
| AC-009-06 | Wallet debited upon order confirmation | TP-070 |
| AC-009-07 | Order status set to PAID after wallet debit | TP-071 |
| AC-009-08 | Order confirmation SMS/WhatsApp sent | TP-072 |
| AC-009-09 | Order value validated: 500–5,000,000 YER (Constraint 26) | TP-073 |
| AC-009-10 | 15% VAT calculated on discounted price (Constraint 20) | TP-074 |

### Business Rules
- BR-009-01: No COD (Constraint 2)
- BR-009-02: Master/sub-order architecture (Constraint 9)
- BR-009-03: Wallet-only payments (Constraint 1)

---

## FR-010: Order Lifecycle Management

**Module:** M13: Order Lifecycle, M14: Sub-Order Management
**Priority:** Critical
**Blocks:** Order Management

### Description
Orders progress through 17 defined states. Each state transition is validated and logged. Sub-orders follow independent lifecycles within the master order.

### Acceptance Criteria

| ID | Criterion | Test Points |
|----|-----------|-------------|
| AC-010-01 | All 17 order states implemented (Constraint 8) | TP-075 |
| AC-010-02 | State transitions follow defined state machine | TP-076 |
| AC-010-03 | Invalid transitions rejected with clear error | TP-077 |
| AC-010-04 | Sub-order status aggregated for master order view | TP-078 |
| AC-010-05 | Order status changes trigger notifications | TP-079 |
| AC-010-06 | Order history maintained for 5 years | TP-080 |
| AC-010-07 | Customer can view order status in real-time | TP-081 |
| AC-010-08 | Merchant can update order status (up to DELIVERED) | TP-082 |
| AC-010-09 | Admin can override order status in emergency | TP-083 |
| AC-010-10 | Order cancellation allowed before ASSIGNED state | TP-084 |

### Order State Machine

```
PENDING → PAID → CONFIRMED → PROCESSING → READY_FOR_PICKUP
    ↓                                              ↓
CANCELLED                                    ASSIGNED
                                                  ↓
                                          PICKED_UP → IN_TRANSIT
                                                        ↓
                                              OUT_FOR_DELIVERY → DELIVERED
                                                                       ↓
                                                                  COMPLETED
                                                                       ↓
                                                    RETURN_REQUESTED → RETURN_APPROVED
                                                                              ↓
                                                          RETURN_IN_TRANSIT → RETURN_RECEIVED
                                                                                    ↓
                                                                                  REFUNDED
```

---

## FR-011: Wallet & Payment Management

**Module:** M15: Wallet Service
**Priority:** Critical
**Blocks:** Payment & Wallet

### Description
Users fund wallets via bank transfer or other approved methods. Wallet balance used for all transactions. No card or BNPL payments.

### Acceptance Criteria

| ID | Criterion | Test Points |
|----|-----------|-------------|
| AC-011-01 | Wallet funding via approved methods only | TP-085 |
| AC-011-02 | No card payments (Constraint 1) | TP-086 |
| AC-011-03 | No BNPL or installments (Constraint 1) | TP-087 |
| AC-011-04 | Wallet balance displayed in real-time | TP-088 |
| AC-011-05 | Transaction history with 12-month retention | TP-089 |
| AC-011-06 | Wallet-to-wallet transfers supported | TP-090 |
| AC-011-07 | Transaction receipts generated for all operations | TP-091 |
| AC-011-08 | Daily transaction limit configurable per user type | TP-092 |

### Business Rules
- BR-011-01: Wallet-only payments (Constraint 1)
- BR-011-02: Double-entry bookkeeping (Constraint 22)

---

## FR-012: Escrow Engine

**Module:** M16: Escrow Engine
**Priority:** Critical
**Blocks:** Payment & Wallet

### Description
Customer payments held in escrow for 7 days after delivery. Funds released to merchant minus platform fees. Handles refund scenarios.

### Acceptance Criteria

| ID | Criterion | Test Points |
|----|-----------|-------------|
| AC-012-01 | Escrow hold starts at DELIVERED status | TP-093 |
| AC-012-02 | 7-day hold period (Constraint 12) | TP-094 |
| AC-012-03 | Funds released to merchant after hold period | TP-095 |
| AC-012-04 | Platform fee deducted before release | TP-096 |
| AC-012-05 | Return request during hold pauses release | TP-097 |
| AC-012-06 | Escrow status visible to merchant | TP-098 |
| AC-012-07 | Escrow ledger maintains double-entry records | TP-099 |
| AC-012-08 | Automated reconciliation daily | TP-100 |

### Business Rules
- BR-012-01: 7-day escrow hold (Constraint 12)
- BR-012-02: Double-entry bookkeeping (Constraint 22)
- BR-012-03: 15% VAT on discounted price (Constraint 20)

---

## FR-013: Shipping & Delivery Management

**Module:** M17: Delivery Management, M18: Delivery Code System
**Priority:** Critical
**Blocks:** Shipping & Delivery

### Description
Delivery agents assigned to orders. Customers receive delivery codes via SMS/WhatsApp. No GPS tracking; status updates via delivery codes.

### Acceptance Criteria

| ID | Criterion | Test Points |
|----|-----------|-------------|
| AC-013-01 | Delivery agent assignment (manual or auto) | TP-101 |
| AC-013-02 | No GPS tracking (Constraint 10) | TP-102 |
| AC-013-03 | Delivery code sent via SMS/WhatsApp | TP-103 |
| AC-013-04 | 3-attempt lockout (Constraint 11) | TP-104 |
| AC-013-05 | Lockout triggers 24-hour account freeze | TP-105 |
| AC-013-06 | Lockout generates support ticket | TP-106 |
| AC-013-07 | Delivery status updates logged with timestamp | TP-107 |
| AC-013-08 | Agent can report delivery failure | TP-108 |
| AC-013-09 | Re-delivery scheduled for failed attempts | TP-109 |
| AC-013-10 | Delivery performance metrics tracked | TP-110 |

### Business Rules
- BR-013-01: No GPS tracking (Constraint 10)
- BR-013-02: 3-attempt lockout (Constraint 11)

---

## FR-014: Returns & Refunds

**Module:** M19: Return Processing, M20: Refund Engine
**Priority:** Critical
**Blocks:** Returns & Refunds

### Description
Customers request returns within merchant-defined policy windows. Returns validated against `isReturnable` and `returnPeriodDays`. Refunds processed to wallet.

### Acceptance Criteria

| ID | Criterion | Test Points |
|----|-----------|-------------|
| AC-014-01 | Return request validates `isReturnable` flag (Constraint 13) | TP-111 |
| AC-014-02 | Return request validates `returnPeriodDays` (Constraint 13) | TP-112 |
| AC-014-03 | Return request requires reason selection | TP-113 |
| AC-014-04 | Merchant approves/rejects return request | TP-114 |
| AC-014-05 | Return shipping instructions provided | TP-115 |
| AC-014-06 | Return status tracked through 17-state lifecycle | TP-116 |
| AC-014-07 | Refund processed to customer wallet | TP-117 |
| AC-014-08 | Refund amount = original payment amount | TP-118 |
| AC-014-09 | Escrow hold paused during return process | TP-119 |
| AC-014-10 | Return/refund history maintained for 5 years | TP-120 |

### Business Rules
- BR-014-01: Merchant return policies enforced (Constraint 13)
- BR-014-02: Refund to wallet only (Constraint 1)

---

## FR-015: Notification System

**Module:** M21: Notification Hub
**Priority:** High
**Blocks:** Notifications

### Description
Multi-channel notification system supporting SMS, WhatsApp, push notifications, and in-app messages. Notifications triggered by order events, wallet transactions, and system alerts.

### Acceptance Criteria

| ID | Criterion | Test Points |
|----|-----------|-------------|
| AC-015-01 | SMS notifications for critical events (OTP, order status) | TP-121 |
| AC-015-02 | WhatsApp notifications for order updates | TP-122 |
| AC-015-03 | Push notifications for mobile app users | TP-123 |
| AC-015-04 | In-app notification center | TP-124 |
| AC-015-05 | Notification preferences configurable per user | TP-125 |
| AC-015-06 | Notification delivery status tracked | TP-126 |
| AC-015-07 | Failed notifications retried (max 3 attempts) | TP-127 |
| AC-015-08 | Notification templates in Arabic and English | TP-128 |

---

## FR-016: Analytics & Reporting

**Module:** M22: Analytics Engine
**Priority:** High
**Blocks:** Analytics & Reporting

### Description
Merchants access sales analytics, customer insights, and financial reports. Admins access platform-wide analytics. All reports support Arabic/English export.

### Acceptance Criteria

| ID | Criterion | Test Points |
|----|-----------|-------------|
| AC-016-01 | Merchant dashboard: sales, orders, revenue, top products | TP-129 |
| AC-016-02 | Real-time analytics updated within 5 minutes | TP-130 |
| AC-016-03 | Date range filtering (daily, weekly, monthly, custom) | TP-131 |
| AC-016-04 | Export to PDF and Excel (Arabic and English) | TP-132 |
| AC-016-05 | Admin platform-wide analytics | TP-133 |
| AC-016-06 | Financial reports: VAT collected, escrow status, settlements | TP-134 |
| AC-016-07 | Delivery performance metrics | TP-135 |
| AC-016-08 | Search analytics (top queries, zero-result queries) | TP-136 |

---

## FR-017: Platform Administration

**Module:** M23: Admin Panel
**Priority:** Critical
**Blocks:** Platform Administration

### Description
Administrators manage platform configuration, user accounts, content, and compliance. Super admins have unrestricted access.

### Acceptance Criteria

| ID | Criterion | Test Points |
|----|-----------|-------------|
| AC-017-01 | Admin dashboard with system health overview | TP-137 |
| AC-017-02 | User management (view, suspend, deactivate) | TP-138 |
| AC-017-03 | Merchant KYC review queue | TP-139 |
| AC-017-04 | Content management (banners, pages, promotions) | TP-140 |
| AC-017-05 | System configuration (40+ categories, 17 states) | TP-141 |
| AC-017-06 | Audit log viewer with filtering | TP-142 |
| AC-017-07 | ZATCA compliance report generation | TP-143 |
| AC-017-08 | Invoice management (5-year retention, Constraint 23) | TP-144 |
| AC-017-09 | Escalation management for support tickets | TP-145 |
| AC-017-10 | System backup and restore (Super Admin only) | TP-146 |

---

## Requirements Traceability Matrix

| Requirement | Module(s) | Test Points | Constraints |
|-------------|-----------|-------------|-------------|
| FR-001 | M01 | TP-001 to TP-010 | C3, C4, C5 |
| FR-002 | M02 | TP-011 to TP-016 | C7 |
| FR-003 | M03 | TP-017 to TP-024 | C19 |
| FR-004 | M04, M05 | TP-025 to TP-034 | C15, C16, C17, C18 |
| FR-005 | M06 | TP-035 to TP-040 | - |
| FR-006 | M07, M08 | TP-041 to TP-048 | C14 |
| FR-007 | M09, M10 | TP-049 to TP-056 | - |
| FR-008 | M11 | TP-057 to TP-064 | C24, C25, C26 |
| FR-009 | M12 | TP-065 to TP-074 | C1, C2, C9, C20, C26 |
| FR-010 | M13, M14 | TP-075 to TP-084 | C8, C9 |
| FR-011 | M15 | TP-085 to TP-092 | C1, C22 |
| FR-012 | M16 | TP-093 to TP-100 | C12, C20, C22 |
| FR-013 | M17, M18 | TP-101 to TP-110 | C10, C11 |
| FR-014 | M19, M20 | TP-111 to TP-120 | C1, C13 |
| FR-015 | M21 | TP-121 to TP-128 | - |
| FR-016 | M22 | TP-129 to TP-136 | - |
| FR-017 | M23 | TP-137 to TP-146 | C21, C23 |

---

*Document Version: 1.0 | Last Updated: 2026-09-13 | Classification: Confidential*
