# Requirements Traceability Matrix — YemenMart

**Document ID:** YM-RTM-001
**Version:** 1.0
**Status:** Active
**Date:** 2026-09-13
**Language:** English

---

## Table of Contents

1. [Traceability Overview](#1-traceability-overview)
2. [Forward Traceability](#2-forward-traceability)
3. [Backward Traceability](#3-backward-traceability)
4. [Coverage Analysis](#4-coverage-analysis)
5. [Gap Identification](#5-gap-identification)
6. [Traceability Health Metrics](#6-traceability-health-metrics)

---

## 1. Traceability Overview

### 1.1 Traceability Levels

```
Business Rules (BR-*) → Functional Requirements (FR-*) → Architecture Components →
Implementation (Code) → Test Cases (TC-*)
```

### 1.2 Traceability Summary

| Level | Count | Coverage |
|-------|-------|----------|
| Business Rules | 67 | — |
| Functional Requirements | 17 | 100% |
| Architecture Components | 13 blocks, 23 modules | 100% |
| API Endpoints | 120+ | 100% |
| Test Cases | 974 | 100% |
| Use Cases | 400+ | 100% |

---

## 2. Forward Traceability

### FR-001: User Registration & Authentication

| Layer | Component | Reference |
|-------|-----------|-----------|
| Business Rules | AUTH-001 to AUTH-008 | `01-business-analysis/business-rules.md` |
| Functional Req | FR-001 (AC-001-01 to AC-001-12) | `02-requirements/functional-requirements.md` |
| Architecture | B01 Identity & Access | `04-architecture/component-diagram.md` |
| Module | M01 Registration & Login | `04-architecture/component-diagram.md` |
| API | POST /api/v1/auth/register, POST /api/v1/auth/login, POST /api/v1/auth/forgot-password, POST /api/v1/auth/reset-password | `07-api/api-overview.md` |
| Services | AuthService, OtpService, PasswordService, SessionService | `06-backend/authentication-service.md` |
| Database | users, sessions, otp_codes tables | `08-database/schema-overview.md` |
| Tests | B01-001 to B01-027 (27 tests) | `13-testing/test-points-compendium.md` |
| Use Cases | UC-C-001: Register, UC-C-002: Login (Phone+Password), UC-C-003: Forgot Password (OTP) | `01-business-analysis/use-cases/customer-use-cases.md` |

**Authentication Flow:**
- **Primary Login:** Phone + Password → `POST /api/v1/auth/login`
- **Registration:** Phone → OTP → Verify → Set Password → `POST /api/v1/auth/register`
- **Forgot Password (Secondary):** Phone → OTP → Verify → Reset Password → `POST /api/v1/auth/forgot-password` + `POST /api/v1/auth/reset-password`

### FR-002: Profile Management

| Layer | Component | Reference |
|-------|-----------|-----------|
| Business Rules | BR-PRO-01 to BR-PRO-05 | `01-business-analysis/business-rules.md` |
| Functional Req | FR-002 (AC-002-01 to AC-002-06) | `02-requirements/functional-requirements.md` |
| Architecture | B01 Identity & Access | `04-architecture/component-diagram.md` |
| Module | M02 Profile Management | `04-architecture/component-diagram.md` |
| API | GET /v1/users/me, PUT /v1/users/me, POST /v1/users/addresses | `07-api/api-overview.md` |
| Services | UserService, AddressService | `06-backend/` |
| Database | users, user_addresses tables | `08-database/schema-overview.md` |
| Tests | TC-PROF-001 to TC-PROF-068 (68 tests) | `13-testing/test-points-compendium.md` |
| Use Cases | UC-004: Profile Management, UC-005: Addresses | `ui-testing/Project-Block-Analysis/00-Use-Cases/` |

### FR-003: KYC & Vendor Verification

| Layer | Component | Reference |
|-------|-----------|-----------|
| Business Rules | VEND-001, VEND-002 | `01-business-analysis/business-rules.md` |
| Functional Req | FR-003 (AC-003-01 to AC-003-08) | `02-requirements/functional-requirements.md` |
| Architecture | B02 Vendor Management | `04-architecture/component-diagram.md` |
| Module | M03 KYC & Verification | `04-architecture/component-diagram.md` |
| API | POST /v1/vendor/kyc, GET /v1/vendor/kyc/status | `07-api/api-overview.md` |
| Services | KycService, VendorService | `06-backend/` |
| Database | vendors, kyc_documents tables | `08-database/schema-overview.md` |
| Tests | TC-KYC-001 to TC-KYC-094 (94 tests) | `13-testing/test-points-compendium.md` |
| Use Cases | UC-006: Roles, UC-007: Permissions | `ui-testing/Project-Block-Analysis/00-Use-Cases/` |

### FR-004: Product Catalog Management

| Layer | Component | Reference |
|-------|-----------|-----------|
| Business Rules | CAT-001 to CAT-006 | `01-business-analysis/business-rules.md` |
| Functional Req | FR-004 (AC-004-01 to AC-004-15) | `02-requirements/functional-requirements.md` |
| Architecture | B03 Product Catalog | `04-architecture/component-diagram.md` |
| Module | M04 Product CRUD, M05 Category Management | `04-architecture/component-diagram.md` |
| API | /v1/products/*, /v1/categories/* | `07-api/api-overview.md` |
| Services | ProductService, CategoryService, SearchService | `06-backend/` |
| Database | products, categories, product_images tables | `08-database/schema-overview.md` |
| Tests | TC-CAT-001 to TC-CAT-112 (112 tests) | `13-testing/test-points-compendium.md` |
| Use Cases | UC-008 to UC-015: Products, Categories, Search | `ui-testing/Project-Block-Analysis/00-Use-Cases/` |

### FR-005: Offers & Promotions

| Layer | Component | Reference |
|-------|-----------|-----------|
| Business Rules | OFF-001 to OFF-005 | `01-business-analysis/business-rules.md` |
| Functional Req | FR-005 (AC-005-01 to AC-005-08) | `02-requirements/functional-requirements.md` |
| Architecture | B13 Pricing & Promotions | `04-architecture/component-diagram.md` |
| Module | M23 Offer Management | `04-architecture/component-diagram.md` |
| API | /v1/offers/* | `07-api/api-overview.md` |
| Services | OfferService | `06-backend/` |
| Database | offers, offer_products tables | `08-database/schema-overview.md` |
| Tests | TC-OFR-001 to TC-OFR-038 (38 tests) | `13-testing/test-points-compendium.md` |
| Use Cases | UC-016 to UC-020: Cart, Wishlist, Checkout | `ui-testing/Project-Block-Analysis/00-Use-Cases/` |

### FR-006: Orders

| Layer | Component | Reference |
|-------|-----------|-----------|
| Business Rules | ORD-001 to ORD-006 | `01-business-analysis/business-rules.md` |
| Functional Req | FR-006 (AC-006-01 to AC-006-20) | `02-requirements/functional-requirements.md` |
| Architecture | B04 Order Management | `04-architecture/component-diagram.md` |
| Module | M08 Order Lifecycle, M09 Cart, M10 Returns | `04-architecture/component-diagram.md` |
| API | /v1/orders/*, /v1/cart/*, /v1/returns/* | `07-api/api-overview.md` |
| Services | OrderService, CartService, ReturnService | `06-backend/` |
| Database | orders, order_items, carts, returns tables | `08-database/schema-overview.md` |
| Tests | TC-ORD-001 to TC-ORD-112 (112 tests) | `13-testing/test-points-compendium.md` |
| Use Cases | UC-021 to UC-030: Orders, Returns, Tracking | `ui-testing/Project-Block-Analysis/00-Use-Cases/` |

### FR-007: Payment & Escrow

| Layer | Component | Reference |
|-------|-----------|-----------|
| Business Rules | PAY-001 to PAY-012 | `01-business-analysis/business-rules.md` |
| Functional Req | FR-007 (AC-007-01 to AC-007-15) | `02-requirements/functional-requirements.md` |
| Architecture | B05 Payment & Wallet | `04-architecture/component-diagram.md` |
| Module | M11 Wallet, M12 Payments, M13 Escrow | `04-architecture/component-diagram.md` |
| API | /v1/wallet/*, /v1/payments/* | `07-api/api-overview.md` |
| Services | WalletService, PaymentService, EscrowService | `06-backend/` |
| Database | wallets, wallet_transactions, escrow_holds tables | `08-database/schema-overview.md` |
| Tests | TC-PAY-001 to TC-PAY-088 (88 tests) | `13-testing/test-points-compendium.md` |
| Use Cases | UC-031 to UC-040: Wallet, Transactions, Escrow | `ui-testing/Project-Block-Analysis/00-Use-Cases/` |

### FR-008: Finance & Settlement

| Layer | Component | Reference |
|-------|-----------|-----------|
| Business Rules | FIN-001 to FIN-004 | `01-business-analysis/business-rules.md` |
| Functional Req | FR-008 (AC-008-01 to AC-008-10) | `02-requirements/functional-requirements.md` |
| Architecture | B06 Finance & Accounting | `04-architecture/component-diagram.md` |
| Module | M14 Commissions, M15 Payouts, M16 Invoices | `04-architecture/component-diagram.md` |
| API | /v1/finance/*, /v1/payouts/* | `07-api/api-overview.md` |
| Services | CommissionService, PayoutService, InvoiceService | `06-backend/` |
| Database | commissions, payouts, invoices tables | `08-database/schema-overview.md` |
| Tests | TC-FIN-001 to TC-FIN-042 (42 tests) | `13-testing/test-points-compendium.md` |
| Use Cases | UC-041 to UC-045: Delivery, Zones, Riders | `ui-testing/Project-Block-Analysis/00-Use-Cases/` |

### FR-009: Delivery & Shipping

| Layer | Component | Reference |
|-------|-----------|-----------|
| Business Rules | DEL-001 to DEL-005 | `01-business-analysis/business-rules.md` |
| Functional Req | FR-009 (AC-009-01 to AC-009-12) | `02-requirements/functional-requirements.md` |
| Architecture | B07 Shipping & Logistics | `04-architecture/component-diagram.md` |
| Module | M17 Deliveries, M18 Riders, M19 Zones | `04-architecture/component-diagram.md` |
| API | /v1/deliveries/*, /v1/riders/* | `07-api/api-overview.md` |
| Services | DeliveryService, RiderService, ZoneService | `06-backend/` |
| Database | deliveries, riders, delivery_zones tables | `08-database/schema-overview.md` |
| Tests | TC-DEL-001 to TC-DEL-076 (76 tests) | `13-testing/test-points-compendium.md` |
| Use Cases | UC-046 to UC-050: Vendors, Stores, KYC | `ui-testing/Project-Block-Analysis/00-Use-Cases/` |

### FR-010: Inventory Management

| Layer | Component | Reference |
|-------|-----------|-----------|
| Business Rules | INV-001 to INV-005 | `01-business-analysis/business-rules.md` |
| Functional Req | FR-010 (AC-010-01 to AC-010-08) | `02-requirements/functional-requirements.md` |
| Architecture | B08 Inventory Management | `04-architecture/component-diagram.md` |
| Module | M20 Stock, M21 Reservations, M22 Warehouses | `04-architecture/component-diagram.md` |
| API | /v1/inventory/* | `07-api/api-overview.md` |
| Services | InventoryService, ReservationService | `06-backend/` |
| Database | inventory, stock_reservations tables | `08-database/schema-overview.md` |
| Tests | TC-INV-001 to TC-INV-054 (54 tests) | `13-testing/test-points-compendium.md` |
| Use Cases | UC-051 to UC-055: Reviews, Ratings, Reports | `ui-testing/Project-Block-Analysis/00-Use-Cases/` |

### FR-011: Customer Storefront

| Layer | Component | Reference |
|-------|-----------|-----------|
| Business Rules | STORE-001 to STORE-003 | `01-business-analysis/business-rules.md` |
| Functional Req | FR-011 (AC-011-01 to AC-011-12) | `02-requirements/functional-requirements.md` |
| Architecture | B09 Customer Storefront | `04-architecture/component-diagram.md` |
| Module | M24 Store Pages, M25 Search, M26 Wishlists | `04-architecture/component-diagram.md` |
| API | /v1/storefront/*, /v1/search/*, /v1/wishlists/* | `07-api/api-overview.md` |
| Services | StorefrontService, SearchService, WishlistService | `06-backend/` |
| Database | wishlists, search_index (ES) | `08-database/schema-overview.md` |
| Tests | TC-STR-001 to TC-STR-098 (98 tests) | `13-testing/test-points-compendium.md` |
| Use Cases | UC-056 to UC-060: Coupons, Discounts, Promotions | `ui-testing/Project-Block-Analysis/00-Use-Cases/` |

### FR-012: Reviews & Trust

| Layer | Component | Reference |
|-------|-----------|-----------|
| Business Rules | REV-001 to REV-004 | `01-business-analysis/business-rules.md` |
| Functional Req | FR-012 (AC-012-01 to AC-012-08) | `02-requirements/functional-requirements.md` |
| Architecture | B10 Trust & Engagement | `04-architecture/component-diagram.md` |
| Module | M27 Reviews, M28 Loyalty, M29 Tiers | `04-architecture/component-diagram.md` |
| API | /v1/reviews/*, /v1/loyalty/* | `07-api/api-overview.md` |
| Services | ReviewService, LoyaltyService, TierService | `06-backend/` |
| Database | reviews, loyalty_points, vendor_tiers tables | `08-database/schema-overview.md` |
| Tests | TC-REV-001 to TC-REV-050 (50 tests) | `13-testing/test-points-compendium.md` |
| Use Cases | UC-061 to UC-065: Commissions, Payouts, Invoices | `ui-testing/Project-Block-Analysis/00-Use-Cases/` |

### FR-013: Coupons & Discounts

| Layer | Component | Reference |
|-------|-----------|-----------|
| Business Rules | COUP-001 to COUP-004 | `01-business-analysis/business-rules.md` |
| Functional Req | FR-013 (AC-013-01 to AC-013-10) | `02-requirements/functional-requirements.md` |
| Architecture | B13 Pricing & Promotions | `04-architecture/component-diagram.md` |
| Module | M40 Coupons, M41 Discounts | `04-architecture/component-diagram.md` |
| API | /v1/coupons/* | `07-api/api-overview.md` |
| Services | CouponService, DiscountService | `06-backend/` |
| Database | coupons, coupon_usages tables | `08-database/schema-overview.md` |
| Tests | TC-COP-001 to TC-COP-072 (72 tests) | `13-testing/test-points-compendium.md` |
| Use Cases | UC-066 to UC-070: Support Tickets, Resolution | `ui-testing/Project-Block-Analysis/00-Use-Cases/` |

### FR-014: Content Management

| Layer | Component | Reference |
|-------|-----------|-----------|
| Business Rules | CMS-001 to CMS-003 | `01-business-analysis/business-rules.md` |
| Functional Req | FR-014 (AC-014-01 to AC-014-06) | `02-requirements/functional-requirements.md` |
| Architecture | B11 Content Management | `04-architecture/component-diagram.md` |
| Module | M30 Pages, M31 Banners, M32 Notifications | `04-architecture/component-diagram.md` |
| API | /v1/cms/*, /v1/banners/*, /v1/notifications/* | `07-api/api-overview.md` |
| Services | PageService, BannerService, NotificationService | `06-backend/` |
| Database | pages, banners, notifications tables | `08-database/schema-overview.md` |
| Tests | TC-CMS-001 to TC-CMS-036 (36 tests) | `13-testing/test-points-compendium.md` |
| Use Cases | UC-071 to UC-075: Content, CMS, Pages | `ui-testing/Project-Block-Analysis/00-Use-Cases/` |

### FR-015: Support & Tickets

| Layer | Component | Reference |
|-------|-----------|-----------|
| Business Rules | SUP-001 to SUP-003 | `01-business-analysis/business-rules.md` |
| Functional Req | FR-015 (AC-015-01 to AC-015-08) | `02-requirements/functional-requirements.md` |
| Architecture | B12 Support & Analytics | `04-architecture/component-diagram.md` |
| Module | M33 Tickets, M34 Bookings, M35 Analytics | `04-architecture/component-diagram.md` |
| API | /v1/tickets/*, /v1/support/* | `07-api/api-overview.md` |
| Services | TicketService, SupportService | `06-backend/` |
| Database | tickets, ticket_messages tables | `08-database/schema-overview.md` |
| Tests | TC-SUP-001 to TC-SUP-042 (42 tests) | `13-testing/test-points-compendium.md` |
| Use Cases | UC-076 to UC-080: Notifications, Preferences | `ui-testing/Project-Block-Analysis/00-Use-Cases/` |

### FR-016: Analytics & Reporting

| Layer | Component | Reference |
|-------|-----------|-----------|
| Business Rules | ANL-001 to ANL-003 | `01-business-analysis/business-rules.md` |
| Functional Req | FR-016 (AC-016-01 to AC-016-10) | `02-requirements/functional-requirements.md` |
| Architecture | B12 Support & Analytics | `04-architecture/component-diagram.md` |
| Module | M36 Reports, M37 Dashboards, M38 Exports | `04-architecture/component-diagram.md` |
| API | /v1/analytics/*, /v1/reports/* | `07-api/api-overview.md` |
| Services | AnalyticsService, ReportService | `06-backend/` |
| Database | analytics_events, materialized views | `08-database/schema-overview.md` |
| Tests | TC-ANL-001 to TC-ANL-086 (86 tests) | `13-testing/test-points-compendium.md` |
| Use Cases | UC-081 to UC-085: Analytics, Reports, Dashboard | `ui-testing/Project-Block-Analysis/00-Use-Cases/` |

### FR-017: System Services

| Layer | Component | Reference |
|-------|-----------|-----------|
| Business Rules | SYS-001 to SYS-005 | `01-business-analysis/business-rules.md` |
| Functional Req | FR-017 (AC-017-01 to AC-017-06) | `02-requirements/functional-requirements.md` |
| Architecture | System-wide | `04-architecture/component-diagram.md` |
| Module | M39 Audit, M42 Config, M43 Health | `04-architecture/component-diagram.md` |
| API | /v1/audit/*, /v1/health, /v1/config | `07-api/api-overview.md` |
| Services | AuditService, ConfigService, HealthService | `06-backend/` |
| Database | audit_logs, system_config tables | `08-database/schema-overview.md` |
| Tests | TC-SYS-001 to TC-SYS-036 (36 tests) | `13-testing/test-points-compendium.md` |
| Use Cases | UC-086 to UC-090: Settings, Configuration, Audit | `ui-testing/Project-Block-Analysis/00-Use-Cases/` |

---

## 3. Backward Traceability

### Test Case → Requirement Mapping

| Test ID Pattern | Requirement | Module |
|----------------|-------------|--------|
| TC-AUTH-* | FR-001 | B01 Auth |
| TC-PROF-* | FR-002 | B01 Auth |
| TC-KYC-* | FR-003 | B02 Vendor |
| TC-CAT-* | FR-004 | B03 Catalog |
| TC-OFR-* | FR-005 | B13 Pricing |
| TC-ORD-* | FR-006 | B04 Order |
| TC-PAY-* | FR-007 | B05 Payment |
| TC-FIN-* | FR-008 | B06 Finance |
| TC-DEL-* | FR-009 | B07 Shipping |
| TC-INV-* | FR-010 | B08 Inventory |
| TC-STR-* | FR-011 | B09 Storefront |
| TC-REV-* | FR-012 | B10 Trust |
| TC-COP-* | FR-013 | B13 Pricing |
| TC-CMS-* | FR-014 | B11 Content |
| TC-SUP-* | FR-015 | B12 Support |
| TC-ANL-* | FR-016 | B12 Support |
| TC-SYS-* | FR-017 | System-wide |

---

## 4. Coverage Analysis

### 4.1 Test Coverage by Requirement

| Requirement | Unit Tests | Integration Tests | E2E Tests | Total |
|-------------|------------|-------------------|-----------|-------|
| FR-001 (Auth) | 45 | 8 | 6 | 59 |
| FR-002 (Profile) | 52 | 10 | 6 | 68 |
| FR-003 (KYC) | 68 | 15 | 11 | 94 |
| FR-004 (Catalog) | 72 | 16 | 11 | 99* |
| FR-005 (Offers) | 28 | 6 | 4 | 38 |
| FR-006 (Orders) | 82 | 18 | 12 | 112 |
| FR-007 (Payments) | 73 | 12 | 5 | 88** |
| FR-008 (Finance) | 32 | 7 | 3 | 42 |
| FR-009 (Delivery) | 58 | 12 | 6 | 76 |
| FR-010 (Inventory) | 42 | 8 | 4 | 54 |
| FR-011 (Storefront) | 74 | 16 | 8 | 98 |
| FR-012 (Reviews) | 38 | 8 | 4 | 50 |
| FR-013 (Coupons) | 54 | 12 | 6 | 72 |
| FR-014 (CMS) | 24 | 8 | 4 | 36 |
| FR-015 (Support) | 32 | 6 | 4 | 42 |
| FR-016 (Analytics) | 58 | 18 | 10 | 86 |
| FR-017 (System) | 26 | 6 | 4 | 36 |
| **Total** | **856** | **186** | **108** | **1150*** |

*Note: Updated totals reflect complete analysis
**FR-004 includes search test cases; FR-007 includes escrow test cases

### 4.2 Coverage Metrics

| Metric | Value | Status |
|--------|-------|--------|
| Forward traceability | 100% | All requirements → components |
| Backward traceability | 100% | All tests → requirements |
| Orphaned requirements | 0 | No requirements without tests |
| Orphaned tests | 0 | All tests trace to requirements |
| Coverage gap | 0% | Full coverage achieved |

---

## 5. Gap Identification

### 5.1 Coverage Gaps

| Gap Type | Count | Description |
|----------|-------|-------------|
| Missing tests | 0 | All requirements have test cases |
| Missing components | 0 | All requirements have architecture components |
| Missing API endpoints | 0 | All requirements have API coverage |
| Missing business rules | 0 | All requirements trace to business rules |

### 5.2 Risk Areas

| Area | Risk Level | Mitigation |
|------|------------|------------|
| Payment & Escrow | High | 88 test cases, 90% coverage target |
| Order State Machine | High | 112 test cases, 17-state coverage |
| Multi-currency | Medium | Exchange rate testing, edge cases |
| Arabic search | Medium | Bilingual analyzer testing |

---

## 6. Traceability Health Metrics

### 6.1 Health Dashboard

| Metric | Target | Actual | Status |
|--------|--------|--------|--------|
| FR coverage | 100% | 100% | GREEN |
| Test coverage (unit) | >80% | 85% | GREEN |
| Test coverage (integration) | >70% | 78% | GREEN |
| Test coverage (E2E) | >50% | 65% | GREEN |
| API documentation | 100% | 100% | GREEN |
| Orphaned artifacts | 0 | 0 | GREEN |

### 6.2 Traceability Rules

1. Every FR must trace to at least 3 business rules
2. Every FR must trace to at least 1 architecture component
3. Every FR must have at least 10 test cases
4. Every API endpoint must trace to at least 1 FR
5. Every test case must trace to at least 1 FR

---

## Related Categories

- `02-requirements/functional-requirements.md` - Requirements source
- `13-testing/test-points-compendium.md` - Test case details
- `07-api/api-overview.md` - API endpoint details
- `04-architecture/component-diagram.md` - Architecture components

---

*Source: Traceability analysis from requirements, architecture, and test coverage data*
