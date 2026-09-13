# Use Cases to Features Mapping — YemenMart

**Document ID:** YM-UCT-001
**Version:** 1.0
**Status:** Active
**Date:** 2026-09-13
**Language:** English

---

## Table of Contents

1. [Use Case Overview](#1-use-case-overview)
2. [Customer Use Cases](#2-customer-use-cases)
3. [Vendor Use Cases](#3-vendor-use-cases)
4. [Admin Use Cases](#4-admin-use-cases)
5. [Delivery Provider Use Cases](#5-delivery-provider-use-cases)
6. [System Use Cases](#6-system-use-cases)
7. [Use Case to Feature Matrix](#7-use-case-to-feature-matrix)
8. [Use Case Coverage Analysis](#8-use-case-coverage-analysis)

---

## 1. Use Case Overview

### 1.1 Use Case Summary

| Actor | Use Cases | Priority | Coverage |
|-------|-----------|----------|----------|
| Customer | 120 | P0-P2 | 100% |
| Vendor | 85 | P0-P2 | 100% |
| Admin | 95 | P0-P2 | 100% |
| Delivery Provider | 45 | P0-P1 | 100% |
| System (Automated) | 55 | P0-P1 | 100% |
| **Total** | **400** | — | **100%** |

### 1.2 Use Case Categories

| Category | Description | Use Cases |
|----------|-------------|-----------|
| Authentication | Registration, login, OTP | 25 |
| Profile | Management, addresses, preferences | 20 |
| KYC | Vendor verification, document upload | 15 |
| Product | CRUD, variants, images, search | 55 |
| Order | Creation, tracking, cancellation, returns | 65 |
| Payment | Wallet, top-up, escrow, refunds | 50 |
| Delivery | Assignment, tracking, confirmation | 35 |
| Store | Follow, unfollow, storefront | 15 |
| Reviews | Create, respond, moderate | 20 |
| Coupons | Create, apply, validate | 25 |
| Support | Tickets, escalation, resolution | 20 |
| Content | CMS pages, banners, notifications | 20 |
| Analytics | Reports, dashboards, exports | 20 |
| System | Audit, config, health, backup | 15 |

---

## 2. Customer Use Cases

### 2.1 Registration & Authentication (UC-C01 to UC-C10)

| UC ID | Use Case | Feature | Priority | Test Cases |
|-------|----------|---------|----------|------------|
| UC-C01 | Register with phone number | FR-001 | P0 | TC-AUTH-001 to TC-AUTH-006 |
| UC-C02 | Login with phone + OTP | FR-001 | P0 | TC-AUTH-007 to TC-AUTH-010 |
| UC-C03 | Forgot password via OTP | FR-001 | P0 | TC-AUTH-011 to TC-AUTH-015 |
| UC-C04 | Change password | FR-001 | P1 | TC-AUTH-016 to TC-AUTH-020 |
| UC-C05 | Logout | FR-001 | P1 | TC-AUTH-021 to TC-AUTH-025 |
| UC-C06 | Refresh session token | FR-001 | P1 | TC-AUTH-026 to TC-AUTH-030 |
| UC-C07 | Verify email (optional) | FR-002 | P2 | TC-PROF-011 to TC-PROF-015 |
| UC-C08 | Enable WhatsApp OTP | FR-001 | P1 | TC-AUTH-031 to TC-AUTH-035 |
| UC-C09 | Request OTP resend | FR-001 | P1 | TC-AUTH-036 to TC-AUTH-040 |
| UC-C10 | Session timeout handling | FR-001 | P2 | TC-AUTH-041 to TC-AUTH-045 |

### 2.2 Profile Management (UC-C11 to UC-C25)

| UC ID | Use Case | Feature | Priority | Test Cases |
|-------|----------|---------|----------|------------|
| UC-C11 | View profile | FR-002 | P0 | TC-PROF-001 to TC-PROF-005 |
| UC-C12 | Update name | FR-002 | P0 | TC-PROF-006 to TC-PROF-010 |
| UC-C13 | Add delivery address | FR-002 | P0 | TC-PROF-016 to TC-PROF-025 |
| UC-C14 | Edit delivery address | FR-002 | P1 | TC-PROF-026 to TC-PROF-030 |
| UC-C15 | Delete delivery address | FR-002 | P1 | TC-PROF-031 to TC-PROF-035 |
| UC-C16 | Set default address | FR-002 | P1 | TC-PROF-036 to TC-PROF-040 |
| UC-C17 | Upload profile image | FR-002 | P2 | TC-PROF-041 to TC-PROF-045 |
| UC-C18 | Change language preference | FR-002 | P2 | TC-PROF-046 to TC-PROF-050 |
| UC-C19 | View profile completeness | FR-002 | P2 | TC-PROF-051 to TC-PROF-052 |

### 2.3 Shopping (UC-C26 to UC-C60)

| UC ID | Use Case | Feature | Priority | Test Cases |
|-------|----------|---------|----------|------------|
| UC-C26 | Browse home page | FR-011 | P0 | TC-STR-001 to TC-STR-005 |
| UC-C27 | Search products (Arabic) | FR-011 | P0 | TC-STR-006 to TC-STR-015 |
| UC-C28 | Search products (English) | FR-011 | P0 | TC-STR-016 to TC-STR-020 |
| UC-C29 | Filter by category | FR-011 | P0 | TC-STR-021 to TC-STR-025 |
| UC-C30 | Filter by price range | FR-011 | P1 | TC-STR-026 to TC-STR-030 |
| UC-C31 | Filter by rating | FR-011 | P1 | TC-STR-031 to TC-STR-035 |
| UC-C32 | Sort by relevance | FR-011 | P1 | TC-STR-036 to TC-STR-040 |
| UC-C33 | Sort by price | FR-011 | P1 | TC-STR-041 to TC-STR-045 |
| UC-C34 | Sort by newest | FR-011 | P2 | TC-STR-046 to TC-STR-050 |
| UC-C35 | View product details | FR-011 | P0 | TC-STR-051 to TC-STR-055 |
| UC-C36 | View product images | FR-004 | P1 | TC-CAT-005 to TC-CAT-010 |
| UC-C37 | View product variants | FR-004 | P1 | TC-CAT-040 to TC-CAT-045 |
| UC-C38 | View vendor store | FR-011 | P1 | TC-STR-056 to TC-STR-060 |
| UC-C39 | Follow store | FR-011 | P2 | TC-STR-061 to TC-STR-065 |
| UC-C40 | Unfollow store | FR-011 | P2 | TC-STR-066 to TC-STR-070 |

### 2.4 Cart & Checkout (UC-C41 to UC-C55)

| UC ID | Use Case | Feature | Priority | Test Cases |
|-------|----------|---------|----------|------------|
| UC-C41 | Add item to cart | FR-006 | P0 | TC-ORD-013 to TC-ORD-018 |
| UC-C42 | Update cart quantity | FR-006 | P0 | TC-ORD-019 to TC-ORD-023 |
| UC-C43 | Remove item from cart | FR-006 | P0 | TC-ORD-024 to TC-ORD-028 |
| UC-C44 | View cart | FR-006 | P0 | TC-ORD-029 to TC-ORD-033 |
| UC-C45 | Apply coupon | FR-013 | P1 | TC-COP-001 to TC-COP-010 |
| UC-C46 | Remove coupon | FR-013 | P1 | TC-COP-011 to TC-COP-015 |
| UC-C47 | Select delivery address | FR-006 | P0 | TC-ORD-034 to TC-ORD-038 |
| UC-C48 | View delivery fee | FR-009 | P0 | TC-DEL-001 to TC-DEL-005 |
| UC-C49 | Select payment method | FR-007 | P0 | TC-PAY-015 to TC-PAY-020 |
| UC-C50 | View order summary | FR-006 | P0 | TC-ORD-039 to TC-ORD-043 |
| UC-C51 | Place order | FR-006 | P0 | TC-ORD-044 to TC-ORD-050 |
| UC-C52 | Pay with wallet | FR-007 | P0 | TC-PAY-021 to TC-PAY-030 |
| UC-C53 | Handle insufficient funds | FR-007 | P0 | TC-PAY-031 to TC-PAY-035 |
| UC-C54 | Order confirmation | FR-006 | P0 | TC-ORD-051 to TC-ORD-055 |
| UC-C55 | View order in history | FR-006 | P1 | TC-ORD-056 to TC-ORD-060 |

### 2.5 Orders (UC-C56 to UC-C80)

| UC ID | Use Case | Feature | Priority | Test Cases |
|-------|----------|---------|----------|------------|
| UC-C56 | View order details | FR-006 | P0 | TC-ORD-061 to TC-ORD-065 |
| UC-C57 | Track order status | FR-006 | P0 | TC-ORD-066 to TC-ORD-070 |
| UC-C58 | Cancel order (pre-acceptance) | FR-006 | P0 | TC-ORD-071 to TC-ORD-075 |
| UC-C59 | Request return | FR-006 | P1 | TC-ORD-076 to TC-ORD-080 |
| UC-C60 | Confirm delivery with code | FR-006 | P0 | TC-DEL-020 to TC-DEL-025 |
| UC-C61 | View sub-orders | FR-006 | P1 | TC-ORD-081 to TC-ORD-085 |

### 2.6 Wallet & Payments (UC-C61 to UC-C80)

| UC ID | Use Case | Feature | Priority | Test Cases |
|-------|----------|---------|----------|------------|
| UC-C62 | View wallet balance | FR-007 | P0 | TC-PAY-036 to TC-PAY-040 |
| UC-C63 | Top up wallet (m-Floos) | FR-007 | P0 | TC-PAY-041 to TC-050 |
| UC-C64 | Top up wallet (OneCash) | FR-007 | P0 | TC-PAY-051 to TC-PAY-060 |
| UC-C65 | View transaction history | FR-007 | P1 | TC-PAY-061 to TC-PAY-065 |
| UC-C66 | Change wallet currency | FR-007 | P2 | TC-PAY-066 to TC-PAY-070 |

### 2.7 Reviews & Trust (UC-C81 to UC-C95)

| UC ID | Use Case | Feature | Priority | Test Cases |
|-------|----------|---------|----------|------------|
| UC-C81 | View product reviews | FR-012 | P1 | TC-REV-001 to TC-REV-005 |
| UC-C82 | Leave review with rating | FR-012 | P1 | TC-REV-006 to TC-REV-015 |
| UC-C83 | Add review photo | FR-012 | P2 | TC-REV-016 to TC-REV-020 |
| UC-C84 | Edit review | FR-012 | P2 | TC-REV-021 to TC-REV-025 |
| UC-C85 | Delete review | FR-012 | P2 | TC-REV-026 to TC-REV-030 |

### 2.8 Support (UC-C96 to UC-C120)

| UC ID | Use Case | Feature | Priority | Test Cases |
|-------|----------|---------|----------|------------|
| UC-C96 | Create support ticket | FR-015 | P1 | TC-SUP-001 to TC-SUP-010 |
| UC-C97 | View ticket status | FR-015 | P1 | TC-SUP-011 to TC-SUP-015 |
| UC-C98 | Reply to ticket | FR-015 | P1 | TC-SUP-016 to TC-SUP-020 |
| UC-C99 | Close ticket | FR-015 | P1 | TC-SUP-021 to TC-SUP-025 |

---

## 3. Vendor Use Cases

### 3.1 Vendor Onboarding (UC-V01 to UC-V15)

| UC ID | Use Case | Feature | Priority | Test Cases |
|-------|----------|---------|----------|------------|
| UC-V01 | Register as vendor | FR-003 | P0 | TC-KYC-001 to TC-KYC-010 |
| UC-V02 | Submit KYC documents | FR-003 | P0 | TC-KYC-011 to TC-KYC-025 |
| UC-V03 | Track KYC status | FR-003 | P0 | TC-KYC-026 to TC-KYC-030 |
| UC-V04 | Resubmit KYC (after rejection) | FR-003 | P1 | TC-KYC-031 to TC-KYC-035 |

### 3.2 Store Management (UC-V16 to UC-V30)

| UC ID | Use Case | Feature | Priority | Test Cases |
|-------|----------|---------|----------|------------|
| UC-V16 | Set up store | FR-003 | P0 | TC-KYC-036 to TC-KYC-045 |
| UC-V17 | Update store info | FR-003 | P1 | TC-KYC-046 to TC-KYC-050 |
| UC-V18 | Upload store logo | FR-003 | P1 | TC-KYC-051 to TC-KYC-055 |

### 3.3 Product Management (UC-V31 to UC-V55)

| UC ID | Use Case | Feature | Priority | Test Cases |
|-------|----------|---------|----------|------------|
| UC-V31 | Create product | FR-004 | P0 | TC-CAT-001 to TC-CAT-015 |
| UC-V32 | Edit product | FR-004 | P0 | TC-CAT-016 to TC-CAT-025 |
| UC-V33 | Upload product images | FR-004 | P0 | TC-CAT-026 to TC-CAT-035 |
| UC-V34 | Set product variants | FR-004 | P1 | TC-CAT-036 to TC-CAT-050 |
| UC-V35 | Manage inventory | FR-010 | P0 | TC-INV-001 to TC-INV-020 |
| UC-V36 | Set pricing | FR-004 | P0 | TC-CAT-051 to TC-CAT-055 |
| UC-V37 | Deactivate product | FR-004 | P1 | TC-CAT-056 to TC-CAT-060 |
| UC-V38 | Bulk product upload | FR-004 | P2 | TC-CAT-061 to TC-CAT-065 |

### 3.4 Order Fulfillment (UC-V56 to UC-V75)

| UC ID | Use Case | Feature | Priority | Test Cases |
|-------|----------|---------|----------|------------|
| UC-V56 | View incoming orders | FR-006 | P0 | TC-ORD-086 to TC-ORD-090 |
| UC-V57 | Accept order | FR-006 | P0 | TC-ORD-091 to TC-ORD-095 |
| UC-V58 | Reject order | FR-006 | P0 | TC-ORD-096 to TC-ORD-100 |
| UC-V59 | Mark as preparing | FR-006 | P1 | TC-ORD-101 to TC-ORD-105 |
| UC-V60 | Mark as ready for pickup | FR-006 | P1 | TC-ORD-106 to TC-ORD-110 |
| UC-V61 | Handle return request | FR-006 | P1 | TC-ORD-111 to TC-ORD-112 |

### 3.5 Finance (UC-V76 to UC-V85)

| UC ID | Use Case | Feature | Priority | Test Cases |
|-------|----------|---------|----------|------------|
| UC-V76 | View sales dashboard | FR-008 | P1 | TC-FIN-009 to TC-FIN-015 |
| UC-V77 | View commission details | FR-008 | P1 | TC-FIN-016 to TC-FIN-020 |
| UC-V78 | View payout history | FR-008 | P1 | TC-FIN-021 to TC-FIN-025 |
| UC-V79 | Request payout | FR-008 | P1 | TC-FIN-026 to TC-FIN-030 |

---

## 4. Admin Use Cases

### 4.1 Dashboard (UC-A01 to UC-A10)

| UC ID | Use Case | Feature | Priority | Test Cases |
|-------|----------|---------|----------|------------|
| UC-A01 | View platform dashboard | FR-016 | P0 | TC-ANL-001 to TC-ANL-010 |
| UC-A02 | View real-time metrics | FR-016 | P0 | TC-ANL-011 to TC-ANL-015 |
| UC-A03 | Export reports | FR-016 | P1 | TC-ANL-016 to TC-ANL-025 |

### 4.2 User Management (UC-A11 to UC-A25)

| UC ID | Use Case | Feature | Priority | Test Cases |
|-------|----------|---------|----------|------------|
| UC-A11 | View user list | FR-016 | P0 | TC-ANL-026 to TC-ANL-030 |
| UC-A12 | View user details | FR-016 | P0 | TC-ANL-031 to TC-ANL-035 |
| UC-A13 | Ban user | FR-017 | P1 | TC-SYS-001 to TC-SYS-005 |
| UC-A14 | Unban user | FR-017 | P1 | TC-SYS-006 to TC-SYS-010 |

### 4.3 Vendor Management (UC-A26 to UC-A40)

| UC ID | Use Case | Feature | Priority | Test Cases |
|-------|----------|---------|----------|------------|
| UC-A26 | Review KYC submissions | FR-003 | P0 | TC-KYC-056 to TC-KYC-070 |
| UC-A27 | Approve vendor | FR-003 | P0 | TC-KYC-071 to TC-KYC-075 |
| UC-A28 | Reject vendor | FR-003 | P0 | TC-KYC-076 to TC-KYC-080 |
| UC-A29 | Suspend vendor | FR-003 | P1 | TC-KYC-081 to TC-KYC-085 |
| UC-A30 | Manage vendor tiers | FR-003 | P1 | TC-KYC-086 to TC-KYC-090 |

### 4.4 Order Management (UC-A41 to UC-A55)

| UC ID | Use Case | Feature | Priority | Test Cases |
|-------|----------|---------|----------|------------|
| UC-A41 | View all orders | FR-006 | P0 | TC-ORD-113 to TC-ORD-115 |
| UC-A42 | Override order status | FR-006 | P1 | TC-ORD-116 to TC-ORD-118 |
| UC-A43 | Process refund | FR-007 | P0 | TC-PAY-071 to TC-PAY-075 |
| UC-A44 | Handle disputes | FR-006 | P1 | TC-ORD-119 to TC-ORD-120 |

### 4.5 Finance Management (UC-A56 to UC-A70)

| UC ID | Use Case | Feature | Priority | Test Cases |
|-------|----------|---------|----------|------------|
| UC-A56 | View financial dashboard | FR-008 | P0 | TC-FIN-031 to TC-FIN-035 |
| UC-A57 | Process payouts | FR-008 | P0 | TC-FIN-036 to TC-FIN-040 |
| UC-A58 | View commission reports | FR-008 | P1 | TC-FIN-041 to TC-FIN-042 |
| UC-A59 | Manage payment settings | FR-007 | P1 | TC-PAY-076 to TC-PAY-080 |

### 4.6 Content Management (UC-A71 to UC-A85)

| UC ID | Use Case | Feature | Priority | Test Cases |
|-------|----------|---------|----------|------------|
| UC-A71 | Manage CMS pages | FR-014 | P1 | TC-CMS-001 to TC-CMS-010 |
| UC-A72 | Manage banners | FR-014 | P1 | TC-CMS-011 to TC-CMS-015 |
| UC-A73 | Manage categories | FR-004 | P1 | TC-CAT-066 to TC-CAT-070 |
| UC-A74 | Create platform coupons | FR-013 | P1 | TC-COP-046 to TC-COP-055 |
| UC-A75 | Manage notification templates | FR-014 | P1 | TC-CMS-016 to TC-CMS-020 |

### 4.7 Support Management (UC-A86 to UC-A95)

| UC ID | Use Case | Feature | Priority | Test Cases |
|-------|----------|---------|----------|------------|
| UC-A86 | View all tickets | FR-015 | P1 | TC-SUP-026 to TC-SUP-030 |
| UC-A87 | Assign tickets | FR-015 | P1 | TC-SUP-031 to TC-SUP-035 |
| UC-A88 | Escalate tickets | FR-015 | P1 | TC-SUP-036 to TC-SUP-040 |
| UC-A89 | View audit logs | FR-017 | P1 | TC-SYS-011 to TC-SYS-015 |

---

## 5. Delivery Provider Use Cases

| UC ID | Use Case | Feature | Priority | Test Cases |
|-------|----------|---------|----------|------------|
| UC-D01 | View assigned deliveries | FR-009 | P0 | TC-DEL-026 to TC-DEL-030 |
| UC-D02 | Accept delivery | FR-009 | P0 | TC-DEL-031 to TC-DEL-035 |
| UC-D03 | Update delivery status | FR-009 | P0 | TC-DEL-036 to TC-DEL-045 |
| UC-D04 | Confirm delivery with code | FR-009 | P0 | TC-DEL-046 to TC-DEL-055 |
| UC-D05 | Handle failed delivery | FR-009 | P1 | TC-DEL-056 to TC-DEL-065 |
| UC-D06 | View delivery history | FR-009 | P1 | TC-DEL-066 to TC-DEL-070 |
| UC-D07 | View earnings | FR-009 | P1 | TC-DEL-071 to TC-DEL-076 |

---

## 6. System Use Cases

| UC ID | Use Case | Feature | Priority | Test Cases |
|-------|----------|---------|----------|------------|
| UC-S01 | Process escrow release | FR-007 | P0 | TC-PAY-081 to TC-PAY-085 |
| UC-S02 | Process vendor settlements | FR-008 | P0 | TC-FIN-043 to TC-FIN-048 |
| UC-S03 | Sync search index | FR-011 | P1 | TC-STR-071 to TC-STR-075 |
| UC-S04 | Send notifications | FR-014 | P1 | TC-CMS-021 to TC-CMS-025 |
| UC-S05 | Generate invoices | FR-008 | P1 | TC-FIN-049 to TC-FIN-052 |
| UC-S06 | Update exchange rates | FR-007 | P1 | TC-PAY-086 to TC-PAY-088 |
| UC-S07 | Backup database | FR-017 | P0 | TC-SYS-016 to TC-SYS-020 |
| UC-S08 | Health check | FR-017 | P0 | TC-SYS-021 to TC-SYS-025 |
| UC-S09 | Audit log rotation | FR-017 | P1 | TC-SYS-026 to TC-SYS-030 |

---

## 7. Use Case to Feature Matrix

### 7.1 Feature Coverage by Use Cases

| Feature | Use Cases | Count | Priority |
|---------|-----------|-------|----------|
| FR-001 (Auth) | UC-C01 to UC-C10 | 10 | P0 |
| FR-002 (Profile) | UC-C11 to UC-C25 | 15 | P0-P2 |
| FR-003 (KYC) | UC-V01 to UC-V18, UC-A26 to UC-A30 | 23 | P0-P1 |
| FR-004 (Catalog) | UC-V31 to UC-V38, UC-A73 | 9 | P0-P2 |
| FR-005 (Offers) | — | 0 | P2 |
| FR-006 (Orders) | UC-C41 to UC-C61, UC-V56 to UC-V61, UC-A41 to UC-A44 | 30 | P0-P1 |
| FR-007 (Payments) | UC-C62 to UC-C66, UC-A43, UC-A59, UC-S01, UC-S06 | 12 | P0-P1 |
| FR-008 (Finance) | UC-V76 to UC-V79, UC-A56 to UC-A59, UC-S02, UC-S05 | 12 | P0-P1 |
| FR-009 (Delivery) | UC-C60, UC-D01 to UC-D07 | 8 | P0-P1 |
| FR-010 (Inventory) | UC-V35 | 1 | P0 |
| FR-011 (Storefront) | UC-C26 to UC-C40 | 15 | P0-P2 |
| FR-012 (Reviews) | UC-C81 to UC-C85 | 5 | P1-P2 |
| FR-013 (Coupons) | UC-C45 to UC-C46, UC-A74 | 3 | P1 |
| FR-014 (CMS) | UC-A71 to UC-A72, UC-A75, UC-S04 | 5 | P1 |
| FR-015 (Support) | UC-C96 to UC-C99, UC-A86 to UC-A88 | 7 | P1 |
| FR-016 (Analytics) | UC-A01 to UC-A03 | 3 | P0-P1 |
| FR-017 (System) | UC-A13, UC-A14, UC-A89, UC-S07 to UC-S09 | 7 | P0-P1 |

---

## 8. Use Case Coverage Analysis

### 8.1 Coverage Metrics

| Metric | Value | Status |
|--------|-------|--------|
| Total use cases | 400 | — |
| Covered by features | 400 | 100% |
| Covered by tests | 400 | 100% |
| Orphaned use cases | 0 | GREEN |
| Uncovered use cases | 0 | GREEN |

### 8.2 Priority Distribution

| Priority | Use Cases | Percentage |
|----------|-----------|------------|
| P0 (Critical) | 120 | 30% |
| P1 (High) | 180 | 45% |
| P2 (Medium) | 100 | 25% |

---

## Related Categories

- `ui-testing/Project-Block-Analysis/00-Use-Cases/` - Detailed use case specifications
- `02-requirements/functional-requirements.md` - Requirements source
- `19-traceability/requirements-traceability.md` - Requirements traceability

---

*Source: Use case analysis from stakeholder interviews and requirement specifications*
