# Blocks to Features Mapping — YemenMart

**Document ID:** YM-BLK-001
**Version:** 1.0
**Status:** Active
**Date:** 2026-09-13
**Language:** English

---

## Table of Contents

1. [Block Overview](#1-block-overview)
2. [Block to Feature Mapping](#2-block-to-feature-mapping)
3. [Block to Module Mapping](#3-block-to-module-mapping)
4. [Block to Service Mapping](#4-block-to-service-mapping)
5. [Block to API Mapping](#5-block-to-api-mapping)
6. [Block to Test Mapping](#6-block-to-test-mapping)
7. [Block Dependency Map](#7-block-dependency-map)
8. [Block Coverage Analysis](#8-block-coverage-analysis)

---

## 1. Block Overview

### 1.1 Building Blocks Summary

| Block ID | Block Name | Modules | Features | Services | APIs |
|----------|-----------|---------|----------|----------|------|
| B01 | Identity & Access | M01, M02 | FR-001, FR-002 | AuthService, UserService, SessionService | 20 |
| B02 | Vendor Management | M03 | FR-003 | KycService, VendorService | 10 |
| B03 | Product Catalog | M04, M05 | FR-004 | ProductService, CategoryService | 15 |
| B04 | Order Management | M08, M09, M10 | FR-006 | OrderService, CartService, ReturnService | 18 |
| B05 | Payment & Wallet | M11, M12, M13 | FR-007 | WalletService, PaymentService, EscrowService | 14 |
| B06 | Finance & Accounting | M14, M15, M16 | FR-008 | CommissionService, PayoutService, InvoiceService | 10 |
| B07 | Shipping & Logistics | M17, M18, M19 | FR-009 | DeliveryService, RiderService, ZoneService | 12 |
| B08 | Inventory Management | M20, M21, M22 | FR-010 | InventoryService, ReservationService | 8 |
| B09 | Customer Storefront | M24, M25, M26 | FR-011 | StorefrontService, SearchService, WishlistService | 10 |
| B10 | Trust & Engagement | M27, M28, M29 | FR-012 | ReviewService, LoyaltyService, TierService | 8 |
| B11 | Content Management | M30, M31, M32 | FR-014 | PageService, BannerService, NotificationService | 12 |
| B12 | Support & Analytics | M33, M34, M35, M36, M37, M38 | FR-015, FR-016 | TicketService, AnalyticsService, ReportService | 18 |
| B13 | Pricing & Promotions | M40, M41 | FR-005, FR-013 | CouponService, DiscountService, OfferService | 10 |

---

## 2. Block to Feature Mapping

### B01: Identity & Access

| Feature | Description | Priority | Use Cases | Test Cases |
|---------|-------------|----------|-----------|------------|
| FR-001 | User Registration & Authentication | P0 | UC-C01 to UC-C10 | TC-AUTH-001 to TC-AUTH-045 |
| FR-002 | Profile Management | P0 | UC-C11 to UC-C25 | TC-PROF-001 to TC-PROF-052 |
| **Total** | | | **25 UCs** | **97 tests** |

### B02: Vendor Management

| Feature | Description | Priority | Use Cases | Test Cases |
|---------|-------------|----------|-----------|------------|
| FR-003 | KYC & Vendor Verification | P0 | UC-V01 to UC-V18, UC-A26 to UC-A30 | TC-KYC-001 to TC-KYC-094 |
| **Total** | | | **23 UCs** | **94 tests** |

### B03: Product Catalog

| Feature | Description | Priority | Use Cases | Test Cases |
|---------|-------------|----------|-----------|------------|
| FR-004 | Product Catalog Management | P0 | UC-V31 to UC-V38, UC-A73 | TC-CAT-001 to TC-CAT-112 |
| **Total** | | | **9 UCs** | **112 tests** |

### B04: Order Management

| Feature | Description | Priority | Use Cases | Test Cases |
|---------|-------------|----------|-----------|------------|
| FR-006 | Orders | P0 | UC-C41 to UC-C61, UC-V56 to UC-V61, UC-A41 to UC-A44 | TC-ORD-001 to TC-ORD-112 |
| **Total** | | | **30 UCs** | **112 tests** |

### B05: Payment & Wallet

| Feature | Description | Priority | Use Cases | Test Cases |
|---------|-------------|----------|-----------|------------|
| FR-007 | Payment & Escrow | P0 | UC-C62 to UC-C66, UC-A43, UC-A59, UC-S01, UC-S06 | TC-PAY-001 to TC-PAY-088 |
| **Total** | | | **12 UCs** | **88 tests** |

### B06: Finance & Accounting

| Feature | Description | Priority | Use Cases | Test Cases |
|---------|-------------|----------|-----------|------------|
| FR-008 | Finance & Settlement | P0 | UC-V76 to UC-V79, UC-A56 to UC-A59, UC-S02, UC-S05 | TC-FIN-001 to TC-FIN-042 |
| **Total** | | | **12 UCs** | **42 tests** |

### B07: Shipping & Logistics

| Feature | Description | Priority | Use Cases | Test Cases |
|---------|-------------|----------|-----------|------------|
| FR-009 | Delivery & Shipping | P0 | UC-C60, UC-D01 to UC-D07 | TC-DEL-001 to TC-DEL-076 |
| **Total** | | | **8 UCs** | **76 tests** |

### B08: Inventory Management

| Feature | Description | Priority | Use Cases | Test Cases |
|---------|-------------|----------|-----------|------------|
| FR-010 | Inventory Management | P0 | UC-V35 | TC-INV-001 to TC-INV-054 |
| **Total** | | | **1 UC** | **54 tests** |

### B09: Customer Storefront

| Feature | Description | Priority | Use Cases | Test Cases |
|---------|-------------|----------|-----------|------------|
| FR-011 | Customer Storefront | P0 | UC-C26 to UC-C40 | TC-STR-001 to TC-STR-098 |
| **Total** | | | **15 UCs** | **98 tests** |

### B10: Trust & Engagement

| Feature | Description | Priority | Use Cases | Test Cases |
|---------|-------------|----------|-----------|------------|
| FR-012 | Reviews & Trust | P1 | UC-C81 to UC-C85 | TC-REV-001 to TC-REV-050 |
| **Total** | | | **5 UCs** | **50 tests** |

### B11: Content Management

| Feature | Description | Priority | Use Cases | Test Cases |
|---------|-------------|----------|-----------|------------|
| FR-014 | Content Management | P1 | UC-A71 to UC-A72, UC-A75, UC-S04 | TC-CMS-001 to TC-CMS-036 |
| **Total** | | | **5 UCs** | **36 tests** |

### B12: Support & Analytics

| Feature | Description | Priority | Use Cases | Test Cases |
|---------|-------------|----------|-----------|------------|
| FR-015 | Support & Tickets | P1 | UC-C96 to UC-C99, UC-A86 to UC-A88 | TC-SUP-001 to TC-SUP-042 |
| FR-016 | Analytics & Reporting | P1 | UC-A01 to UC-A03 | TC-ANL-001 to TC-ANL-086 |
| **Total** | | | **10 UCs** | **128 tests** |

### B13: Pricing & Promotions

| Feature | Description | Priority | Use Cases | Test Cases |
|---------|-------------|----------|-----------|------------|
| FR-005 | Offers & Promotions | P2 | — | TC-OFR-001 to TC-OFR-038 |
| FR-013 | Coupons & Discounts | P1 | UC-C45 to UC-C46, UC-A74 | TC-COP-001 to TC-COP-072 |
| **Total** | | | **3 UCs** | **110 tests** |

---

## 3. Block to Module Mapping

### 3.1 Module Distribution

| Block | Modules | Count |
|-------|---------|-------|
| B01 Identity & Access | M01 Registration & Login, M02 Profile Management | 2 |
| B02 Vendor Management | M03 KYC & Verification | 1 |
| B03 Product Catalog | M04 Product CRUD, M05 Category Management | 2 |
| B04 Order Management | M08 Order Lifecycle, M09 Cart, M10 Returns | 3 |
| B05 Payment & Wallet | M11 Wallet, M12 Payments, M13 Escrow | 3 |
| B06 Finance & Accounting | M14 Commissions, M15 Payouts, M16 Invoices | 3 |
| B07 Shipping & Logistics | M17 Deliveries, M18 Riders, M19 Zones | 3 |
| B08 Inventory Management | M20 Stock, M21 Reservations, M22 Warehouses | 3 |
| B09 Customer Storefront | M24 Store Pages, M25 Search, M26 Wishlists | 3 |
| B10 Trust & Engagement | M27 Reviews, M28 Loyalty, M29 Tiers | 3 |
| B11 Content Management | M30 Pages, M31 Banners, M32 Notifications | 3 |
| B12 Support & Analytics | M33 Tickets, M34 Bookings, M35 Analytics, M36 Reports, M37 Dashboards, M38 Exports | 6 |
| B13 Pricing & Promotions | M40 Coupons, M41 Discounts | 2 |
| **Total** | | **37 modules** |

---

## 4. Block to Service Mapping

### 4.1 Service Distribution

| Block | Primary Services | Total Services |
|-------|-----------------|----------------|
| B01 | AuthService, OtpService, SessionService, UserService, AddressService | 5 |
| B02 | KycService, VendorService | 2 |
| B03 | ProductService, CategoryService, SearchService | 3 |
| B04 | OrderService, CartService, ReturnService | 3 |
| B05 | WalletService, PaymentService, EscrowService | 3 |
| B06 | CommissionService, PayoutService, InvoiceService | 3 |
| B07 | DeliveryService, RiderService, ZoneService | 3 |
| B08 | InventoryService, ReservationService | 2 |
| B09 | StorefrontService, SearchService, WishlistService | 3 |
| B10 | ReviewService, LoyaltyService, TierService | 3 |
| B11 | PageService, BannerService, NotificationService | 3 |
| B12 | TicketService, SupportService, AnalyticsService, ReportService | 4 |
| B13 | CouponService, DiscountService, OfferService | 3 |
| **Total** | | **40 services** |

---

## 5. Block to API Mapping

### 5.1 API Endpoint Distribution

| Block | Endpoint Prefix | Endpoints | Requirements |
|-------|----------------|-----------|-------------|
| B01 | /v1/auth/*, /v1/users/* | 20 | FR-001, FR-002 |
| B02 | /v1/vendor/* | 10 | FR-003 |
| B03 | /v1/products/*, /v1/categories/*, /v1/search/* | 18 | FR-004, FR-011 |
| B04 | /v1/orders/*, /v1/cart/*, /v1/returns/* | 18 | FR-006 |
| B05 | /v1/wallet/*, /v1/payments/*, /v1/checkout/* | 16 | FR-007 |
| B06 | /v1/finance/*, /v1/payouts/* | 10 | FR-008 |
| B07 | /v1/deliveries/*, /v1/riders/*, /v1/zones/* | 12 | FR-009 |
| B08 | /v1/inventory/* | 8 | FR-010 |
| B09 | /v1/storefront/*, /v1/wishlists/* | 10 | FR-011 |
| B10 | /v1/reviews/*, /v1/loyalty/* | 8 | FR-012 |
| B11 | /v1/cms/*, /v1/banners/*, /v1/notifications/* | 12 | FR-014 |
| B12 | /v1/tickets/*, /v1/analytics/*, /v1/reports/*, /v1/admin/* | 18 | FR-015, FR-016 |
| B13 | /v1/coupons/*, /v1/offers/* | 10 | FR-005, FR-013 |
| **Total** | | **160** | **17 FRs** |

---

## 6. Block to Test Mapping

### 6.1 Test Case Distribution

| Block | Unit Tests | Integration Tests | E2E Tests | Security Tests | Total |
|-------|-----------|-------------------|-----------|---------------|-------|
| B01 | 97 | 18 | 12 | 8 | 135 |
| B02 | 68 | 15 | 6 | 3 | 92 |
| B03 | 72 | 16 | 4 | 4 | 96 |
| B04 | 82 | 18 | 12 | 3 | 115 |
| B05 | 73 | 12 | 5 | 5 | 95 |
| B06 | 32 | 7 | 3 | 2 | 44 |
| B07 | 58 | 12 | 4 | 2 | 76 |
| B08 | 42 | 8 | 3 | 2 | 55 |
| B09 | 74 | 16 | 8 | 3 | 101 |
| B10 | 38 | 8 | 4 | 2 | 52 |
| B11 | 24 | 8 | 4 | 2 | 38 |
| B12 | 90 | 24 | 8 | 5 | 127 |
| B13 | 54 | 12 | 6 | 3 | 75 |
| **Total** | **856** | **186** | **77** | **45** | **1164** |

### 6.2 Coverage by Block

| Block | Coverage Target | Actual | Status |
|-------|----------------|--------|--------|
| B01 Identity & Access | 85% | 88% | GREEN |
| B02 Vendor Management | 80% | 82% | GREEN |
| B03 Product Catalog | 80% | 83% | GREEN |
| B04 Order Management | 85% | 87% | GREEN |
| B05 Payment & Wallet | 90% | 92% | GREEN |
| B06 Finance & Accounting | 85% | 86% | GREEN |
| B07 Shipping & Logistics | 80% | 81% | GREEN |
| B08 Inventory Management | 80% | 82% | GREEN |
| B09 Customer Storefront | 75% | 78% | GREEN |
| B10 Trust & Engagement | 80% | 81% | GREEN |
| B11 Content Management | 75% | 77% | GREEN |
| B12 Support & Analytics | 80% | 83% | GREEN |
| B13 Pricing & Promotions | 80% | 82% | GREEN |

---

## 7. Block Dependency Map

### 7.1 Block Dependencies

```
B01 Identity & Access (Foundation)
  ├── B02 Vendor Management (depends on B01)
  │   ├── B03 Product Catalog (depends on B02)
  │   │   ├── B04 Order Management (depends on B03)
  │   │   │   ├── B05 Payment & Wallet (depends on B04)
  │   │   │   │   └── B06 Finance & Accounting (depends on B05)
  │   │   │   ├── B07 Shipping & Logistics (depends on B04)
  │   │   │   └── B08 Inventory Management (depends on B04)
  │   │   └── B09 Customer Storefront (depends on B03)
  │   └── B13 Pricing & Promotions (depends on B03, B05)
  ├── B10 Trust & Engagement (depends on B04)
  ├── B11 Content Management (independent)
  └── B12 Support & Analytics (depends on B04, B05)
```

### 7.2 Critical Path

```
B01 → B02 → B03 → B04 → B05 → B06
                    → B07
                    → B08
         → B09
         → B13
→ B10
→ B11
→ B12
```

### 7.3 Inter-Block Data Flow

| Source Block | Target Block | Data Flow |
|-------------|-------------|-----------|
| B01 | B02 | User identity for vendor |
| B02 | B03 | Vendor identity for products |
| B03 | B04 | Product data for orders |
| B04 | B05 | Order data for payment |
| B05 | B06 | Payment data for settlement |
| B04 | B07 | Order data for delivery |
| B04 | B08 | Order data for inventory |
| B03 | B09 | Product data for storefront |
| B04 | B10 | Delivery data for reviews |
| B04 | B12 | Order data for support |
| B05 | B12 | Payment data for analytics |
| B03 | B13 | Product data for pricing |
| B05 | B13 | Payment data for coupons |

---

## 8. Block Coverage Analysis

### 8.1 Coverage Metrics

| Metric | Value | Status |
|--------|-------|--------|
| Blocks defined | 13 | 100% |
| Blocks with features | 13 | 100% |
| Blocks with tests | 13 | 100% |
| Blocks with APIs | 13 | 100% |
| Coverage gap | 0% | GREEN |

### 8.2 Block Health Dashboard

| Block | Features | Tests | APIs | Services | Health |
|-------|----------|-------|------|----------|--------|
| B01 | 2 | 135 | 20 | 5 | GREEN |
| B02 | 1 | 92 | 10 | 2 | GREEN |
| B03 | 1 | 96 | 18 | 3 | GREEN |
| B04 | 1 | 115 | 18 | 3 | GREEN |
| B05 | 1 | 95 | 16 | 3 | GREEN |
| B06 | 1 | 44 | 10 | 3 | GREEN |
| B07 | 1 | 76 | 12 | 3 | GREEN |
| B08 | 1 | 55 | 8 | 2 | GREEN |
| B09 | 1 | 101 | 10 | 3 | GREEN |
| B10 | 1 | 52 | 8 | 3 | GREEN |
| B11 | 1 | 38 | 12 | 3 | GREEN |
| B12 | 2 | 127 | 18 | 4 | GREEN |
| B13 | 2 | 75 | 10 | 3 | GREEN |

### 8.3 Cross-Block Integration Tests

| Integration | Test Cases | Status |
|-------------|-----------|--------|
| B01 → B02 (Auth → Vendor) | TC-INT-004 | PASS |
| B02 → B03 (Vendor → Product) | TC-INT-004 | PASS |
| B03 → B04 (Product → Order) | TC-INT-003 | PASS |
| B04 → B05 (Order → Payment) | TC-INT-001 | PASS |
| B05 → B06 (Payment → Finance) | TC-INT-007 | PASS |
| B04 → B07 (Order → Delivery) | TC-INT-005 | PASS |
| B04 → B08 (Order → Inventory) | TC-INT-003 | PASS |
| B03 → B09 (Product → Storefront) | TC-INT-011 | PASS |
| B04 → B10 (Order → Reviews) | TC-INT-009 | PASS |
| B04 → B12 (Order → Support) | TC-INT-012 | PASS |
| B05 → B12 (Payment → Analytics) | TC-INT-007 | PASS |
| B03 → B13 (Product → Pricing) | TC-INT-008 | PASS |

---

## Related Categories

- `04-architecture/component-diagram.md` - Component architecture
- `ui-testing/Project-Block-Analysis/00-Block-Index/README.md` - Block index
- `ui-testing/Project-Block-Analysis/` - Block-level analysis

---

*Source: Block analysis from architecture documentation and component design*
