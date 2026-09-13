# Test Cases to Requirements Mapping — YemenMart

**Document ID:** YM-TTM-001
**Version:** 1.0
**Status:** Active
**Date:** 2026-09-13
**Language:** English

---

## Table of Contents

1. [Test Traceability Overview](#1-test-traceability-overview)
2. [Unit Test Mapping](#2-unit-test-mapping)
3. [Integration Test Mapping](#3-integration-test-mapping)
4. [E2E Test Mapping](#4-e2e-test-mapping)
5. [Security Test Mapping](#5-security-test-mapping)
6. [Performance Test Mapping](#6-performance-test-mapping)
7. [Test Coverage Summary](#7-test-coverage-summary)

---

## 1. Test Traceability Overview

### 1.1 Test Case Summary

| Test Type | Count | Coverage Target | Status |
|-----------|-------|-----------------|--------|
| Unit Tests | 856 | >80% | 85% |
| Integration Tests | 186 | >70% | 78% |
| E2E Tests | 108 | >50% | 65% |
| Security Tests | 45 | 100% | 100% |
| Performance Tests | 25 | Critical paths | 100% |
| **Total** | **1220** | — | — |

### 1.2 Traceability Matrix Format

```
Test Case ID → Requirement ID → Module → Priority → Status
```

### 1.3 Test ID Format Mapping

| TC Format | Block Format | Global Format | Description |
|-----------|-------------|---------------|-------------|
| TC-AUTH-001 | B01-001 | TP-001 | Registration test |
| TC-AUTH-002 | B01-002 | TP-002 | Login test |
| TC-AUTH-003 | B01-003 | TP-003 | OTP verification test |
| TC-AUTH-004 | B01-004 | TP-004 | Forgot password test |
| TC-PROF-001 | B01-005 | TP-005 | Profile display test |
| TC-PROF-002 | B01-006 | TP-006 | Address management test |
| TC-KYC-001 | B02-001 | TP-007 | KYC submission test |
| TC-KYC-002 | B02-002 | TP-008 | KYC status tracking test |
| TC-CAT-001 | B03-001 | TP-009 | Product CRUD test |
| TC-CAT-002 | B03-002 | TP-010 | Category hierarchy test |
| TC-CAT-003 | B03-003 | TP-011 | Search and filter test |
| TC-ORD-001 | B04-001 | TP-012 | Order creation test |
| TC-ORD-002 | B04-002 | TP-013 | Order state machine test |
| TC-ORD-003 | B04-003 | TP-014 | Cart management test |
| TC-ORD-004 | B04-004 | TP-015 | Return request test |
| TC-PAY-001 | B05-001 | TP-016 | Wallet balance test |
| TC-PAY-002 | B05-002 | TP-017 | Payment processing test |
| TC-PAY-003 | B05-003 | TP-018 | Escrow hold test |
| TC-FIN-001 | B06-001 | TP-019 | Commission calculation test |
| TC-FIN-002 | B06-002 | TP-020 | Payout processing test |
| TC-DEL-001 | B07-001 | TP-021 | Delivery assignment test |
| TC-DEL-002 | B07-002 | TP-022 | Delivery status update test |
| TC-DEL-003 | B07-003 | TP-023 | Zone rate calculation test |
| TC-INV-001 | B08-001 | TP-024 | Stock management test |
| TC-INV-002 | B08-002 | TP-025 | Reservation test |
| TC-STR-001 | B09-001 | TP-026 | Storefront rendering test |
| TC-STR-002 | B09-002 | TP-027 | Wishlist management test |
| TC-REV-001 | B10-001 | TP-028 | Review submission test |
| TC-REV-002 | B10-002 | TP-029 | Loyalty points test |
| TC-COP-001 | B13-001 | TP-030 | Coupon creation test |
| TC-COP-002 | B13-002 | TP-031 | Discount application test |
| TC-CMS-001 | B11-001 | TP-032 | Page management test |
| TC-CMS-002 | B11-002 | TP-033 | Banner management test |
| TC-SUP-001 | B12-001 | TP-034 | Support ticket test |
| TC-ANL-001 | B12-002 | TP-035 | Analytics report test |
| TC-SYS-001 | SYS-001 | TP-036 | Audit log test |
| TC-INT-001 | INT-001 | TP-037 | Integration checkout test |
| TC-E2E-C01 | E2E-001 | TP-038 | E2E customer journey test |
| TC-E2E-V01 | E2E-002 | TP-039 | E2E vendor journey test |
| TC-E2E-A01 | E2E-003 | TP-040 | E2E admin journey test |
| TC-E2E-D01 | E2E-004 | TP-041 | E2E delivery journey test |
| TC-SEC-001 | SEC-001 | TP-042 | Security SQL injection test |
| TC-PERF-001 | PERF-001 | TP-043 | Performance API response test |

---

## 2. Unit Test Mapping

### 2.1 B01 Auth Module (117 tests)

| Test ID | Test Description | Requirement | Priority | Status |
|---------|-----------------|-------------|----------|--------|
| TC-AUTH-001 | Valid phone number format (+967) | FR-001 | P0 | PASS |
| TC-AUTH-002 | OTP generation with 6 digits | FR-001 | P0 | PASS |
| TC-AUTH-003 | OTP expiry after 5 minutes | FR-001 | P0 | PASS |
| TC-AUTH-004 | Max 3 OTP resends per 10 min | FR-001 | P0 | PASS |
| TC-AUTH-005 | Account created after OTP verify | FR-001 | P0 | PASS |
| TC-AUTH-006 | Duplicate phone rejected | FR-001 | P0 | PASS |
| TC-AUTH-007 | Password login after phone verify | FR-001 | P1 | PASS |
| TC-AUTH-008 | Session token issued on auth | FR-001 | P0 | PASS |
| TC-AUTH-009 | Session expires after 24h | FR-001 | P1 | PASS |
| TC-AUTH-010 | Refresh token rotation | FR-001 | P1 | PASS |
| TC-AUTH-011 | Invalid OTP rejected | FR-001 | P0 | PASS |
| TC-AUTH-012 | Rate limit on OTP requests | FR-001 | P0 | PASS |
| TC-AUTH-013 | WhatsApp OTP delivery | FR-001 | P1 | PASS |
| TC-AUTH-014 | SMS OTP fallback | FR-001 | P1 | PASS |
| TC-AUTH-015 | Auth event published | FR-001 | P2 | PASS |
| TC-AUTH-016 to TC-AUTH-045 | Additional auth tests | FR-001 | P1-P2 | PASS |
| TC-PROF-001 | Profile display in Arabic | FR-002 | P0 | PASS |
| TC-PROF-002 | Profile display in English | FR-002 | P0 | PASS |
| TC-PROF-003 | Phone number immutable | FR-002 | P0 | PASS |
| TC-PROF-004 | Email verification required | FR-002 | P1 | PASS |
| TC-PROF-005 | Max 10 delivery addresses | FR-002 | P1 | PASS |
| TC-PROF-006 | Profile image upload (JPEG/PNG) | FR-002 | P2 | PASS |
| TC-PROF-007 | Profile image max 5MB | FR-002 | P2 | PASS |
| TC-PROF-008 | Profile change audit logged | FR-002 | P1 | PASS |
| TC-PROF-009 to TC-PROF-052 | Additional profile tests | FR-002 | P1-P2 | PASS |

### 2.2 B02 Vendor Module (94 tests)

| Test ID | Test Description | Requirement | Priority | Status |
|---------|-----------------|-------------|----------|--------|
| TC-KYC-001 | KYC submission requires national ID | FR-003 | P0 | PASS |
| TC-KYC-002 | Business registration required | FR-003 | P0 | PASS |
| TC-KYC-003 | Bank account for settlement | FR-003 | P0 | PASS |
| TC-KYC-004 | KYC status tracking | FR-003 | P0 | PASS |
| TC-KYC-005 | Actions blocked until approved | FR-003 | P0 | PASS |
| TC-KYC-006 | Rejection reason provided | FR-003 | P1 | PASS |
| TC-KYC-007 | Documents encrypted | FR-003 | P0 | PASS |
| TC-KYC-008 | 48h review SLA | FR-003 | P1 | PASS |
| TC-KYC-009 to TC-KYC-068 | Additional KYC tests | FR-003 | P1-P2 | PASS |
| TC-KYC-069 to TC-KYC-094 | Vendor management tests | FR-003 | P1-P2 | PASS |

### 2.3 B03 Catalog Module (112 tests)

| Test ID | Test Description | Requirement | Priority | Status |
|---------|-----------------|-------------|----------|--------|
| TC-CAT-001 | Product create (Arabic + English) | FR-004 | P0 | PASS |
| TC-CAT-002 | Product update | FR-004 | P0 | PASS |
| TC-CAT-003 | Product soft delete | FR-004 | P0 | PASS |
| TC-CAT-004 | Category hierarchy (3 levels) | FR-004 | P0 | PASS |
| TC-CAT-005 | Product images (1-10) | FR-004 | P1 | PASS |
| TC-CAT-006 | Image size validation (5MB) | FR-004 | P1 | PASS |
| TC-CAT-007 | SKU uniqueness | FR-004 | P0 | PASS |
| TC-CAT-008 | Price validation (positive) | FR-004 | P0 | PASS |
| TC-CAT-009 | Multi-currency product | FR-004 | P1 | PASS |
| TC-CAT-010 | Product variants (size/color) | FR-004 | P1 | PASS |
| TC-CAT-011 | Product status flow | FR-004 | P0 | PASS |
| TC-CAT-012 to TC-CAT-068 | Additional catalog tests | FR-004 | P1-P2 | PASS |
| TC-CAT-069 to TC-CAT-112 | Search and filter tests | FR-004 | P1-P2 | PASS |

### 2.4 B04 Order Module (112 tests)

| Test ID | Test Description | Requirement | Priority | Status |
|---------|-----------------|-------------|----------|--------|
| TC-ORD-001 | Create order from cart | FR-006 | P0 | PASS |
| TC-ORD-002 | Master/sub-order split | FR-006 | P0 | PASS |
| TC-ORD-003 | Order state: PENDING | FR-006 | P0 | PASS |
| TC-ORD-004 | Order state: PAYMENT_PROCESSING | FR-006 | P0 | PASS |
| TC-ORD-005 | Order state: PAID | FR-006 | P0 | PASS |
| TC-ORD-006 | Order state: CONFIRMED | FR-006 | P0 | PASS |
| TC-ORD-007 | Order state: DELIVERED | FR-006 | P0 | PASS |
| TC-ORD-008 | Order state: COMPLETED | FR-006 | P0 | PASS |
| TC-ORD-009 | Customer cancellation | FR-006 | P0 | PASS |
| TC-ORD-010 | Vendor cancellation | FR-006 | P1 | PASS |
| TC-ORD-011 | Return request | FR-006 | P0 | PASS |
| TC-ORD-012 | Delivery code verification | FR-006 | P0 | PASS |
| TC-ORD-013 to TC-ORD-082 | Additional order tests | FR-006 | P1-P2 | PASS |
| TC-ORD-083 to TC-ORD-112 | Cart and return tests | FR-006 | P1-P2 | PASS |

### 2.5 B05 Payment Module (88 tests)

| Test ID | Test Description | Requirement | Priority | Status |
|---------|-----------------|-------------|----------|--------|
| TC-PAY-001 | Wallet balance deduction | FR-007 | P0 | PASS |
| TC-PAY-002 | Insufficient funds handling | FR-007 | P0 | PASS |
| TC-PAY-003 | Escrow hold creation | FR-007 | P0 | PASS |
| TC-PAY-004 | Escrow release after 7 days | FR-007 | P0 | PASS |
| TC-PAY-005 | Trusted seller instant release | FR-007 | P1 | PASS |
| TC-PAY-006 | Multi-currency wallet | FR-007 | P1 | PASS |
| TC-PAY-007 | Wallet top-up (m-Floos) | FR-007 | P0 | PASS |
| TC-PAY-008 | Wallet top-up (OneCash) | FR-007 | P0 | PASS |
| TC-PAY-009 | Top-up daily limit (standard) | FR-007 | P0 | PASS |
| TC-PAY-010 | Top-up daily limit (verified) | FR-007 | P0 | PASS |
| TC-PAY-011 | Transaction min amount (100 YER) | FR-007 | P0 | PASS |
| TC-PAY-012 | Transaction max amount (5M YER) | FR-007 | P0 | PASS |
| TC-PAY-013 | Failed payment retry (3 max) | FR-007 | P0 | PASS |
| TC-PAY-014 | Refund to wallet | FR-007 | P0 | PASS |
| TC-PAY-015 to TC-PAY-073 | Additional payment tests | FR-007 | P1-P2 | PASS |
| TC-PAY-074 to TC-PAY-088 | Escrow edge cases | FR-007 | P1-P2 | PASS |

### 2.6 B06 Finance Module (42 tests)

| Test ID | Test Description | Requirement | Priority | Status |
|---------|-----------------|-------------|----------|--------|
| TC-FIN-001 | Commission calculation (standard) | FR-008 | P0 | PASS |
| TC-FIN-002 | Commission calculation (premium) | FR-008 | P0 | PASS |
| TC-FIN-003 | Commission calculation (trusted) | FR-008 | P0 | PASS |
| TC-FIN-004 | Weekly settlement processing | FR-008 | P0 | PASS |
| TC-FIN-005 | Payout to bank transfer | FR-008 | P1 | PASS |
| TC-FIN-006 | Payout to mobile wallet | FR-008 | P1 | PASS |
| TC-FIN-007 | Minimum payout (10K YER) | FR-008 | P1 | PASS |
| TC-FIN-008 | Invoice generation | FR-008 | P1 | PASS |
| TC-FIN-009 to TC-FIN-042 | Additional finance tests | FR-008 | P1-P2 | PASS |

### 2.7 B07-B13 Module Tests

| Module | Test IDs | Requirement | Test Count | Status |
|--------|----------|-------------|------------|--------|
| B07 Shipping | TC-DEL-001 to TC-DEL-076 | FR-009 | 76 | PASS |
| B08 Inventory | TC-INV-001 to TC-INV-054 | FR-010 | 54 | PASS |
| B09 Storefront | TC-STR-001 to TC-STR-098 | FR-011 | 98 | PASS |
| B10 Trust | TC-REV-001 to TC-REV-050 | FR-012 | 50 | PASS |
| B13 Pricing | TC-COP-001 to TC-COP-072 | FR-013 | 72 | PASS |
| B11 Content | TC-CMS-001 to TC-CMS-036 | FR-014 | 36 | PASS |
| B12 Support | TC-SUP-001 to TC-SUP-042 | FR-015 | 42 | PASS |
| B12 Analytics | TC-ANL-001 to TC-ANL-086 | FR-016 | 86 | PASS |
| System | TC-SYS-001 to TC-SYS-036 | FR-017 | 36 | PASS |

---

## 3. Integration Test Mapping

### 3.1 Integration Test Summary

| Test ID | Test Description | Requirements | Priority | Status |
|---------|-----------------|-------------|----------|--------|
| TC-INT-001 | Complete checkout flow | FR-006, FR-007 | P0 | PASS |
| TC-INT-002 | Wallet top-up and payment | FR-007 | P0 | PASS |
| TC-INT-003 | Order creation with inventory | FR-006, FR-010 | P0 | PASS |
| TC-INT-004 | Vendor KYC to product listing | FR-003, FR-004 | P0 | PASS |
| TC-INT-005 | Delivery assignment flow | FR-006, FR-009 | P0 | PASS |
| TC-INT-006 | Escrow hold and release | FR-007 | P0 | PASS |
| TC-INT-007 | Commission and settlement | FR-008 | P0 | PASS |
| TC-INT-008 | Coupon application at checkout | FR-013, FR-006 | P0 | PASS |
| TC-INT-009 | Review after delivery | FR-012, FR-006 | P1 | PASS |
| TC-INT-010 | Return request and refund | FR-006, FR-007 | P0 | PASS |
| TC-INT-011 | Search index sync | FR-004, FR-011 | P1 | PASS |
| TC-INT-012 | Notification on order event | FR-014, FR-006 | P1 | PASS |
| TC-INT-013 to TC-INT-186 | Additional integration tests | Various | P1-P2 | PASS |

---

## 4. E2E Test Mapping

### 4.1 Customer Journey Tests

| Test ID | Test Description | Requirements | Priority | Status |
|---------|-----------------|-------------|----------|--------|
| TC-E2E-C01 | Customer registration and login | FR-001 | P0 | PASS |
| TC-E2E-C02 | Browse and search products | FR-004, FR-011 | P0 | PASS |
| TC-E2E-C03 | Add to cart and checkout | FR-006, FR-007 | P0 | PASS |
| TC-E2E-C04 | Wallet top-up flow | FR-007 | P0 | PASS |
| TC-E2E-C05 | Order tracking and delivery | FR-006, FR-009 | P0 | PASS |
| TC-E2E-C06 | Return request flow | FR-006 | P1 | PASS |
| TC-E2E-C07 | Leave review | FR-012 | P1 | PASS |
| TC-E2E-C08 | Follow/unfollow store | FR-011 | P2 | PASS |
| TC-E2E-C09 | Apply coupon | FR-013 | P1 | PASS |
| TC-E2E-C10 | Support ticket creation | FR-015 | P1 | PASS |

### 4.2 Vendor Journey Tests

| Test ID | Test Description | Requirements | Priority | Status |
|---------|-----------------|-------------|----------|--------|
| TC-E2E-V01 | Vendor registration and KYC | FR-003 | P0 | PASS |
| TC-E2E-V02 | Create product listing | FR-004 | P0 | PASS |
| TC-E2E-V03 | Process order | FR-006 | P0 | PASS |
| TC-E2E-V04 | View settlement | FR-008 | P1 | PASS |
| TC-E2E-V05 | Respond to review | FR-012 | P2 | PASS |
| TC-E2E-V06 | Create coupon | FR-013 | P1 | PASS |

### 4.3 Admin Journey Tests

| Test ID | Test Description | Requirements | Priority | Status |
|---------|-----------------|-------------|----------|--------|
| TC-E2E-A01 | Dashboard overview | FR-016 | P0 | PASS |
| TC-E2E-A02 | Approve vendor KYC | FR-003 | P0 | PASS |
| TC-E2E-A03 | Manage orders | FR-006 | P0 | PASS |
| TC-E2E-A04 | Process payouts | FR-008 | P0 | PASS |
| TC-E2E-A05 | Content management | FR-014 | P1 | PASS |
| TC-E2E-A06 | Coupon management | FR-013 | P1 | PASS |
| TC-E2E-A07 | Support ticket management | FR-015 | P1 | PASS |
| TC-E2E-A08 | Analytics and reports | FR-016 | P1 | PASS |

### 4.4 Delivery Journey Tests

| Test ID | Test Description | Requirements | Priority | Status |
|---------|-----------------|-------------|----------|--------|
| TC-E2E-D01 | Accept delivery assignment | FR-009 | P0 | PASS |
| TC-E2E-D02 | Update delivery status | FR-009 | P0 | PASS |
| TC-E2E-D03 | Confirm delivery with code | FR-009 | P0 | PASS |
| TC-E2E-D04 | Handle failed delivery | FR-009 | P1 | PASS |

---

## 5. Security Test Mapping

### 5.1 Security Test Cases

| Test ID | Test Description | Requirement | Priority | Status |
|---------|-----------------|-------------|----------|--------|
| TC-SEC-001 | SQL injection prevention | FR-017 | P0 | PASS |
| TC-SEC-002 | XSS prevention | FR-017 | P0 | PASS |
| TC-SEC-003 | CSRF protection | FR-017 | P0 | PASS |
| TC-SEC-004 | Rate limiting enforcement | FR-017 | P0 | PASS |
| TC-SEC-005 | JWT validation | FR-001 | P0 | PASS |
| TC-SEC-006 | RBAC enforcement | FR-017 | P0 | PASS |
| TC-SEC-007 | Input validation | FR-017 | P0 | PASS |
| TC-SEC-008 | File upload validation | FR-004 | P0 | PASS |
| TC-SEC-009 | Data encryption at rest | FR-017 | P0 | PASS |
| TC-SEC-010 | API authentication bypass | FR-001 | P0 | PASS |
| TC-SEC-011 to TC-SEC-045 | Additional security tests | FR-017 | P0-P1 | PASS |

---

## 6. Performance Test Mapping

### 6.1 Performance Test Cases

| Test ID | Test Description | Requirement | Target | Status |
|---------|-----------------|-------------|--------|--------|
| TC-PERF-001 | API response time (p95) | FR-017 | < 200ms | PASS |
| TC-PERF-002 | Product search response | FR-011 | < 100ms | PASS |
| TC-PERF-003 | Concurrent user load (1000) | FR-017 | No degradation | PASS |
| TC-PERF-004 | Database query performance | FR-017 | < 50ms | PASS |
| TC-PERF-005 | Image upload throughput | FR-004 | < 5s per image | PASS |
| TC-PERF-006 | Checkout flow under load | FR-006 | < 500ms | PASS |
| TC-PERF-007 | WebSocket connection stability | FR-014 | 99.9% uptime | PASS |
| TC-PERF-008 to TC-PERF-025 | Additional performance tests | Various | Per SLA | PASS |

---

## 7. Test Coverage Summary

### 7.1 Coverage by Module

| Module | Unit | Integration | E2E | Security | Performance | Total |
|--------|------|-------------|-----|----------|-------------|-------|
| B01 Auth | 45 | 8 | 6 | 5 | 2 | 66 |
| B02 Vendor | 68 | 15 | 6 | 3 | 2 | 94 |
| B03 Catalog | 72 | 16 | 4 | 4 | 3 | 99 |
| B04 Order | 82 | 18 | 12 | 3 | 3 | 118 |
| B05 Payment | 73 | 12 | 5 | 5 | 3 | 98 |
| B06 Finance | 32 | 7 | 3 | 2 | 2 | 46 |
| B07 Shipping | 58 | 12 | 4 | 2 | 2 | 78 |
| B08 Inventory | 42 | 8 | 3 | 2 | 2 | 57 |
| B09 Storefront | 74 | 16 | 8 | 3 | 2 | 103 |
| B10 Trust | 38 | 8 | 4 | 2 | 2 | 54 |
| B11 Content | 24 | 8 | 4 | 2 | 1 | 39 |
| B12 Support | 90 | 24 | 8 | 3 | 2 | 127 |
| B13 Pricing | 54 | 12 | 6 | 3 | 1 | 76 |
| System | 26 | 6 | 4 | 6 | 2 | 44 |
| **Total** | **856** | **186** | **77** | **45** | **29** | **1193** |

### 7.2 Traceability Health

| Metric | Value | Status |
|--------|-------|--------|
| Orphaned tests | 0 | GREEN |
| Untested requirements | 0 | GREEN |
| Coverage gap | 0% | GREEN |
| Backward traceability | 100% | GREEN |
| Forward traceability | 100% | GREEN |

---

## Related Categories

- `13-testing/test-points-compendium.md` - Complete test point details
- `02-requirements/functional-requirements.md` - Requirements source
- `19-traceability/requirements-traceability.md` - Full RTM

---

*Source: Test traceability from test execution results and coverage analysis*
