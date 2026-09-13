# API Endpoints to Services Mapping — YemenMart

**Document ID:** YM-APT-001
**Version:** 1.0
**Status:** Active
**Date:** 2026-09-13
**Language:** English

---

## Table of Contents

1. [API Traceability Overview](#1-api-traceability-overview)
2. [Authentication API Mapping](#2-authentication-api-mapping)
3. [Vendor API Mapping](#3-vendor-api-mapping)
4. [Product API Mapping](#4-product-api-mapping)
5. [Order API Mapping](#5-order-api-mapping)
6. [Payment API Mapping](#6-payment-api-mapping)
7. [Delivery API Mapping](#7-delivery-api-mapping)
8. [Admin API Mapping](#8-admin-api-mapping)
9. [API Coverage Analysis](#9-api-coverage-analysis)

---

## 1. API Traceability Overview

### 1.1 API Summary

| Category | Endpoints | Services | Requirements |
|----------|-----------|----------|-------------|
| Authentication | 12 | AuthService, OtpService, SessionService | FR-001 |
| Profile | 8 | UserService, AddressService | FR-002 |
| KYC/Vendor | 10 | KycService, VendorService | FR-003 |
| Products | 15 | ProductService, CategoryService | FR-004 |
| Orders | 18 | OrderService, CartService, ReturnService | FR-006 |
| Payments | 14 | WalletService, PaymentService, EscrowService | FR-007 |
| Finance | 10 | CommissionService, PayoutService, InvoiceService | FR-008 |
| Delivery | 12 | DeliveryService, RiderService, ZoneService | FR-009 |
| Inventory | 8 | InventoryService, ReservationService | FR-010 |
| Storefront | 10 | StorefrontService, SearchService, WishlistService | FR-011 |
| Reviews | 8 | ReviewService, LoyaltyService | FR-012 |
| Coupons | 10 | CouponService, DiscountService | FR-013 |
| CMS | 12 | PageService, BannerService, NotificationService | FR-014 |
| Support | 8 | TicketService, SupportService | FR-015 |
| Analytics | 10 | AnalyticsService, ReportService | FR-016 |
| System | 5 | AuditService, ConfigService, HealthService | FR-017 |
| **Total** | **160** | — | **17 FRs** |

### 1.2 Traceability Format

```
API Endpoint → Service Layer → Database Tables → Requirements → Tests
```

---

## 2. Authentication API Mapping

### 2.1 Auth Endpoints

| Method | Endpoint | Service | Database | Requirement | Tests |
|--------|----------|---------|----------|-------------|-------|
| POST | /api/v1/auth/register | AuthService | users | FR-001 | TC-AUTH-001 to TC-AUTH-006 |
| POST | /api/v1/auth/login | AuthService, OtpService | users, sessions | FR-001 | TC-AUTH-007 to TC-AUTH-010 |
| POST | /api/v1/auth/otp/request | OtpService | otp_codes | FR-001 | TC-AUTH-002 to TC-AUTH-005 |
| POST | /api/v1/auth/otp/verify | OtpService | otp_codes, sessions | FR-001 | TC-AUTH-006 |
| POST | /api/v1/auth/refresh | SessionService | sessions | FR-001 | TC-AUTH-026 to TC-AUTH-030 |
| POST | /api/v1/auth/logout | SessionService | sessions | FR-001 | TC-AUTH-021 to TC-AUTH-025 |
| POST | /api/v1/auth/forgot-password | OtpService, AuthService | users, otp_codes | FR-001 | TC-AUTH-011 to TC-AUTH-015 |
| POST | /api/v1/auth/reset-password | AuthService | users | FR-001 | TC-AUTH-016 to TC-AUTH-020 |
| POST | /api/v1/auth/change-password | AuthService | users | FR-001 | TC-AUTH-016 to TC-AUTH-020 |
| GET | /api/v1/auth/me | UserService | users | FR-001 | TC-AUTH-008 |
| PUT | /api/v1/auth/preferences | UserService | users | FR-002 | TC-PROF-046 to TC-PROF-050 |
| POST | /api/v1/auth/verify-email | AuthService | users | FR-002 | TC-PROF-011 to TC-PROF-015 |

### 2.2 Auth Middleware Chain

```
Request → RateLimiter → CORS → RequestValidator → AuthService.validateToken()
       → RBACMiddleware → Controller
```

---

## 3. Vendor API Mapping

### 3.1 KYC Endpoints

| Method | Endpoint | Service | Database | Requirement | Tests |
|--------|----------|---------|----------|-------------|-------|
| POST | /api/v1/vendor/kyc | KycService | vendors, kyc_documents | FR-003 | TC-KYC-001 to TC-KYC-025 |
| GET | /api/v1/vendor/kyc/status | KycService | vendors | FR-003 | TC-KYC-026 to TC-KYC-030 |
| PUT | /api/v1/vendor/kyc/resubmit | KycService | vendors, kyc_documents | FR-003 | TC-KYC-031 to TC-KYC-035 |
| GET | /api/v1/vendor/profile | VendorService | vendors | FR-003 | TC-KYC-036 to TC-KYC-045 |
| PUT | /api/v1/vendor/profile | VendorService | vendors | FR-003 | TC-KYC-046 to TC-KYC-050 |
| POST | /api/v1/vendor/logo | VendorService | vendors | FR-003 | TC-KYC-051 to TC-KYC-055 |
| GET | /api/v1/vendor/dashboard | VendorService, AnalyticsService | vendors, orders | FR-003 | TC-KYC-086 to TC-KYC-090 |
| GET | /api/v1/vendor/settings | VendorService | vendors | FR-003 | TC-KYC-046 to TC-KYC-050 |
| PUT | /api/v1/vendor/settings | VendorService | vendors | FR-003 | TC-KYC-046 to TC-KYC-050 |
| GET | /api/v1/vendor/:id/store | VendorService | vendors | FR-011 | TC-STR-056 to TC-STR-060 |

---

## 4. Product API Mapping

### 4.1 Product Endpoints

| Method | Endpoint | Service | Database | Requirement | Tests |
|--------|----------|---------|----------|-------------|-------|
| POST | /api/v1/products | ProductService | products | FR-004 | TC-CAT-001 to TC-CAT-015 |
| GET | /api/v1/products | ProductService | products | FR-004 | TC-CAT-066 to TC-CAT-070 |
| GET | /api/v1/products/:id | ProductService | products | FR-004 | TC-CAT-003 |
| PUT | /api/v1/products/:id | ProductService | products | FR-004 | TC-CAT-016 to TC-CAT-025 |
| DELETE | /api/v1/products/:id | ProductService | products | FR-004 | TC-CAT-056 to TC-CAT-060 |
| POST | /api/v1/products/:id/images | ProductService, FileService | products, product_images | FR-004 | TC-CAT-026 to TC-CAT-035 |
| DELETE | /api/v1/products/:id/images/:imageId | ProductService | product_images | FR-004 | TC-CAT-026 to TC-CAT-035 |
| GET | /api/v1/products/:id/variants | ProductService | product_variants | FR-004 | TC-CAT-040 to TC-CAT-050 |
| POST | /api/v1/products/:id/variants | ProductService | product_variants | FR-004 | TC-CAT-040 to TC-CAT-050 |
| GET | /api/v1/categories | CategoryService | categories | FR-004 | TC-CAT-004 |
| GET | /api/v1/categories/:id | CategoryService | categories | FR-004 | TC-CAT-004 |
| GET | /api/v1/vendor/products | ProductService | products | FR-004 | TC-CAT-066 to TC-CAT-070 |
| POST | /api/v1/products/bulk | ProductService | products | FR-004 | TC-CAT-061 to TC-CAT-065 |
| GET | /api/v1/products/:id/reviews | ReviewService | reviews | FR-012 | TC-REV-001 to TC-REV-005 |
| GET | /api/v1/search/products | SearchService | Elasticsearch | FR-011 | TC-STR-006 to TC-STR-020 |

### 4.2 Search API

| Method | Endpoint | Service | Database | Requirement | Tests |
|--------|----------|---------|----------|-------------|-------|
| GET | /api/v1/search | SearchService | Elasticsearch | FR-011 | TC-STR-006 to TC-STR-050 |
| GET | /api/v1/search/suggest | SearchService | Elasticsearch | FR-011 | TC-STR-051 to TC-STR-055 |
| GET | /api/v1/search/filters | SearchService | Elasticsearch | FR-011 | TC-STR-021 to TC-STR-035 |

---

## 5. Order API Mapping

### 5.1 Cart Endpoints

| Method | Endpoint | Service | Database | Requirement | Tests |
|--------|----------|---------|----------|-------------|-------|
| GET | /api/v1/cart | CartService | carts, cart_items | FR-006 | TC-ORD-029 to TC-ORD-033 |
| POST | /api/v1/cart/items | CartService | carts, cart_items | FR-006 | TC-ORD-013 to TC-ORD-018 |
| PUT | /api/v1/cart/items/:id | CartService | cart_items | FR-006 | TC-ORD-019 to TC-ORD-023 |
| DELETE | /api/v1/cart/items/:id | CartService | cart_items | FR-006 | TC-ORD-024 to TC-ORD-028 |
| POST | /api/v1/cart/coupon | CartService, CouponService | carts, coupons | FR-013 | TC-COP-001 to TC-COP-010 |
| DELETE | /api/v1/cart/coupon | CartService | carts | FR-013 | TC-COP-011 to TC-COP-015 |
| POST | /api/v1/cart/validate | CartService | carts, products | FR-006 | TC-ORD-034 to TC-ORD-038 |

### 5.2 Order Endpoints

| Method | Endpoint | Service | Database | Requirement | Tests |
|--------|----------|---------|----------|-------------|-------|
| POST | /api/v1/orders | OrderService | orders, order_items | FR-006 | TC-ORD-044 to TC-ORD-050 |
| GET | /api/v1/orders | OrderService | orders | FR-006 | TC-ORD-056 to TC-ORD-060 |
| GET | /api/v1/orders/:id | OrderService | orders, order_items | FR-006 | TC-ORD-061 to TC-ORD-065 |
| POST | /api/v1/orders/:id/cancel | OrderService | orders | FR-006 | TC-ORD-071 to TC-ORD-075 |
| GET | /api/v1/orders/:id/suborders | OrderService | orders, sub_orders | FR-006 | TC-ORD-081 to TC-ORD-085 |
| POST | /api/v1/orders/:id/confirm-delivery | OrderService, DeliveryService | orders, deliveries | FR-006 | TC-DEL-020 to TC-DEL-025 |
| POST | /api/v1/returns | ReturnService | returns | FR-006 | TC-ORD-076 to TC-ORD-080 |
| GET | /api/v1/returns/:id | ReturnService | returns | FR-006 | TC-ORD-076 to TC-ORD-080 |
| PUT | /api/v1/returns/:id/approve | ReturnService | returns | FR-006 | TC-ORD-076 to TC-ORD-080 |
| PUT | /api/v1/returns/:id/reject | ReturnService | returns | FR-006 | TC-ORD-076 to TC-ORD-080 |

### 5.3 Vendor Order Endpoints

| Method | Endpoint | Service | Database | Requirement | Tests |
|--------|----------|---------|----------|-------------|-------|
| GET | /api/v1/vendor/orders | OrderService | orders, sub_orders | FR-006 | TC-ORD-086 to TC-ORD-090 |
| GET | /api/v1/vendor/orders/:id | OrderService | orders, sub_orders | FR-006 | TC-ORD-086 to TC-ORD-090 |
| POST | /api/v1/vendor/orders/:id/accept | OrderService | orders | FR-006 | TC-ORD-091 to TC-ORD-095 |
| POST | /api/v1/vendor/orders/:id/reject | OrderService | orders | FR-006 | TC-ORD-096 to TC-ORD-100 |
| PUT | /api/v1/vendor/orders/:id/status | OrderService | orders | FR-006 | TC-ORD-101 to TC-ORD-110 |

---

## 6. Payment API Mapping

### 6.1 Wallet Endpoints

| Method | Endpoint | Service | Database | Requirement | Tests |
|--------|----------|---------|----------|-------------|-------|
| GET | /api/v1/wallet/balance | WalletService | wallets | FR-007 | TC-PAY-036 to TC-PAY-040 |
| POST | /api/v1/wallet/topup | WalletService, PaymentService | wallets, wallet_transactions | FR-007 | TC-PAY-041 to TC-PAY-060 |
| GET | /api/v1/wallet/transactions | WalletService | wallet_transactions | FR-007 | TC-PAY-061 to TC-PAY-065 |
| POST | /api/v1/wallet/transfer | WalletService | wallets, wallet_transactions | FR-007 | TC-PAY-066 to TC-PAY-070 |
| GET | /api/v1/wallet/currencies | WalletService | wallets | FR-007 | TC-PAY-006 |

### 6.2 Payment Endpoints

| Method | Endpoint | Service | Database | Requirement | Tests |
|--------|----------|---------|----------|-------------|-------|
| POST | /api/v1/payments/charge | PaymentService | wallet_transactions, orders | FR-007 | TC-PAY-021 to TC-PAY-030 |
| POST | /api/v1/payments/refund | PaymentService | wallet_transactions, refunds | FR-007 | TC-PAY-071 to TC-PAY-075 |
| GET | /api/v1/payments/:id | PaymentService | wallet_transactions | FR-007 | TC-PAY-061 to TC-PAY-065 |
| POST | /api/v1/payments/escrow/hold | EscrowService | escrow_holds | FR-007 | TC-PAY-003 |
| POST | /api/v1/payments/escrow/release | EscrowService | escrow_holds | FR-007 | TC-PAY-004, TC-PAY-005 |
| GET | /api/v1/payments/escrow/status | EscrowService | escrow_holds | FR-007 | TC-PAY-081 to TC-PAY-085 |
| POST | /api/v1/payments/webhook | PaymentService | wallet_transactions | FR-007 | TC-PAY-076 to TC-PAY-080 |

### 6.3 Checkout Endpoints

| Method | Endpoint | Service | Database | Requirement | Tests |
|--------|----------|---------|----------|-------------|-------|
| POST | /api/v1/checkout | OrderService, PaymentService | orders, wallets | FR-006, FR-007 | TC-INT-001 |
| GET | /api/v1/checkout/summary | CartService, PaymentService | carts, wallets | FR-006, FR-007 | TC-ORD-039 to TC-ORD-043 |

---

## 7. Delivery API Mapping

| Method | Endpoint | Service | Database | Requirement | Tests |
|--------|----------|---------|----------|-------------|-------|
| GET | /api/v1/deliveries | DeliveryService | deliveries | FR-009 | TC-DEL-001 to TC-DEL-005 |
| GET | /api/v1/deliveries/:id | DeliveryService | deliveries | FR-009 | TC-DEL-006 to TC-DEL-010 |
| POST | /api/v1/deliveries/:id/assign | DeliveryService | deliveries, riders | FR-009 | TC-DEL-011 to TC-DEL-015 |
| PUT | /api/v1/deliveries/:id/status | DeliveryService | deliveries | FR-009 | TC-DEL-036 to TC-DEL-045 |
| POST | /api/v1/deliveries/:id/confirm | DeliveryService | deliveries | FR-009 | TC-DEL-046 to TC-DEL-055 |
| POST | /api/v1/deliveries/:id/failed | DeliveryService | deliveries | FR-009 | TC-DEL-056 to TC-DEL-065 |
| GET | /api/v1/deliveries/track/:orderId | DeliveryService | deliveries | FR-009 | TC-DEL-006 to TC-DEL-010 |
| GET | /api/v1/zones | ZoneService | delivery_zones | FR-009 | TC-DEL-001 to TC-DEL-005 |
| GET | /api/v1/zones/:id/rates | ZoneService | delivery_zones | FR-009 | TC-DEL-001 to TC-DEL-005 |
| GET | /api/v1/riders | RiderService | riders | FR-009 | TC-DEL-026 to TC-DEL-030 |
| POST | /api/v1/riders/:id/availability | RiderService | riders | FR-009 | TC-DEL-026 to TC-DEL-030 |
| GET | /api/v1/riders/:id/deliveries | RiderService, DeliveryService | riders, deliveries | FR-009 | TC-DEL-066 to TC-DEL-070 |

---

## 8. Admin API Mapping

### 8.1 User Management

| Method | Endpoint | Service | Database | Requirement | Tests |
|--------|----------|---------|----------|-------------|-------|
| GET | /api/v1/admin/users | UserService | users | FR-016 | TC-ANL-026 to TC-ANL-030 |
| GET | /api/v1/admin/users/:id | UserService | users | FR-016 | TC-ANL-031 to TC-ANL-035 |
| PUT | /api/v1/admin/users/:id/ban | UserService | users | FR-017 | TC-SYS-001 to TC-SYS-005 |
| PUT | /api/v1/admin/users/:id/unban | UserService | users | FR-017 | TC-SYS-006 to TC-SYS-010 |

### 8.2 Vendor Management

| Method | Endpoint | Service | Database | Requirement | Tests |
|--------|----------|---------|----------|-------------|-------|
| GET | /api/v1/admin/vendors | VendorService | vendors | FR-003 | TC-KYC-056 to TC-KYC-070 |
| GET | /api/v1/admin/vendors/:id/kyc | KycService | vendors, kyc_documents | FR-003 | TC-KYC-056 to TC-KYC-070 |
| PUT | /api/v1/admin/vendors/:id/approve | VendorService | vendors | FR-003 | TC-KYC-071 to TC-KYC-075 |
| PUT | /api/v1/admin/vendors/:id/reject | VendorService | vendors | FR-003 | TC-KYC-076 to TC-KYC-080 |
| PUT | /api/v1/admin/vendors/:id/suspend | VendorService | vendors | FR-003 | TC-KYC-081 to TC-KYC-085 |
| PUT | /api/v1/admin/vendors/:id/tier | VendorService | vendors, vendor_tiers | FR-003 | TC-KYC-086 to TC-KYC-090 |

### 8.3 Order Management

| Method | Endpoint | Service | Database | Requirement | Tests |
|--------|----------|---------|----------|-------------|-------|
| GET | /api/v1/admin/orders | OrderService | orders | FR-006 | TC-ORD-113 to TC-ORD-115 |
| PUT | /api/v1/admin/orders/:id/override | OrderService | orders | FR-006 | TC-ORD-116 to TC-ORD-118 |
| POST | /api/v1/admin/orders/:id/refund | PaymentService | refunds | FR-007 | TC-PAY-071 to TC-PAY-075 |

### 8.4 Finance Management

| Method | Endpoint | Service | Database | Requirement | Tests |
|--------|----------|---------|----------|-------------|-------|
| GET | /api/v1/admin/finance/dashboard | AnalyticsService | materialized views | FR-008 | TC-FIN-031 to TC-FIN-035 |
| POST | /api/v1/admin/payouts/process | PayoutService | payouts | FR-008 | TC-FIN-036 to TC-FIN-040 |
| GET | /api/v1/admin/payouts | PayoutService | payouts | FR-008 | TC-FIN-021 to TC-FIN-025 |
| GET | /api/v1/admin/commissions | CommissionService | commissions | FR-008 | TC-FIN-016 to TC-FIN-020 |

### 8.5 Content Management

| Method | Endpoint | Service | Database | Requirement | Tests |
|--------|----------|---------|----------|-------------|-------|
| GET | /api/v1/admin/pages | PageService | pages | FR-014 | TC-CMS-001 to TC-CMS-010 |
| POST | /api/v1/admin/pages | PageService | pages | FR-014 | TC-CMS-001 to TC-CMS-010 |
| PUT | /api/v1/admin/pages/:id | PageService | pages | FR-014 | TC-CMS-001 to TC-CMS-010 |
| GET | /api/v1/admin/banners | BannerService | banners | FR-014 | TC-CMS-011 to TC-CMS-015 |
| POST | /api/v1/admin/banners | BannerService | banners | FR-014 | TC-CMS-011 to TC-CMS-015 |
| GET | /api/v1/admin/coupons | CouponService | coupons | FR-013 | TC-COP-046 to TC-COP-055 |
| POST | /api/v1/admin/coupons | CouponService | coupons | FR-013 | TC-COP-046 to TC-COP-055 |

### 8.6 System Endpoints

| Method | Endpoint | Service | Database | Requirement | Tests |
|--------|----------|---------|----------|-------------|-------|
| GET | /api/v1/health | HealthService | — | FR-017 | TC-SYS-021 to TC-SYS-025 |
| GET | /api/v1/admin/audit | AuditService | audit_logs | FR-017 | TC-SYS-011 to TC-SYS-015 |
| GET | /api/v1/admin/config | ConfigService | system_config | FR-017 | TC-SYS-026 to TC-SYS-030 |
| PUT | /api/v1/admin/config | ConfigService | system_config | FR-017 | TC-SYS-026 to TC-SYS-030 |
| POST | /api/v1/admin/backup | AuditService | — | FR-017 | TC-SYS-016 to TC-SYS-020 |

---

## 9. API Coverage Analysis

### 9.1 Coverage by Requirement

| Requirement | API Endpoints | Services | Tests |
|-------------|--------------|----------|-------|
| FR-001 (Auth) | 12 | 3 | 59 |
| FR-002 (Profile) | 8 | 2 | 68 |
| FR-003 (KYC) | 10 | 2 | 94 |
| FR-004 (Catalog) | 15 | 3 | 99 |
| FR-006 (Orders) | 18 | 4 | 112 |
| FR-007 (Payments) | 14 | 4 | 88 |
| FR-008 (Finance) | 10 | 4 | 42 |
| FR-009 (Delivery) | 12 | 4 | 76 |
| FR-010 (Inventory) | 8 | 3 | 54 |
| FR-011 (Storefront) | 10 | 4 | 98 |
| FR-012 (Reviews) | 8 | 3 | 50 |
| FR-013 (Coupons) | 10 | 3 | 72 |
| FR-014 (CMS) | 12 | 4 | 36 |
| FR-015 (Support) | 8 | 3 | 42 |
| FR-016 (Analytics) | 10 | 3 | 86 |
| FR-017 (System) | 5 | 4 | 36 |
| **Total** | **160** | **53** | **1150** |

### 9.2 API Health Metrics

| Metric | Value | Status |
|--------|-------|--------|
| Endpoint coverage | 100% | GREEN |
| Service mapping | 100% | GREEN |
| Test coverage | 100% | GREEN |
| Documentation | 100% | GREEN |
| Versioning | v1 prefix | GREEN |

---

## Related Categories

- `07-api/api-overview.md` - Complete API specification
- `06-backend/` - Service implementations
- `19-traceability/requirements-traceability.md` - Requirements traceability

---

*Source: API traceability from endpoint documentation and service architecture*
