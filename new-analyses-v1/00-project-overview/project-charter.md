# YemenMart Project Charter

## 1. Project Identification

| Field | Value |
|-------|-------|
| **Project Name** | YemenMart |
| **Version** | 1.0 |
| **Date** | 2026-09-13 |
| **Classification** | Confidential |
| **Document Owner** | YemenMart Architecture Team |

## 2. Project Purpose

YemenMart is a **multi-vendor e-commerce marketplace** purpose-built for Yemen. It addresses the unique challenges of the Yemeni market including limited banking infrastructure, mobile-first internet usage, Arabic-first communication, and complex regional logistics across 17 governates.

The platform enables merchants to list products, customers to discover and purchase goods, and delivery agents to fulfill orders—all within a wallet-only payment ecosystem that respects local financial constraints.

## 3. Project Objectives

| # | Objective | Success Metric |
|---|-----------|----------------|
| O-01 | Wallet-only payments | Zero card/BNPL integrations; 100% wallet-funded transactions |
| O-02 | Phone-first authentication | 100% auth flows via SMS/WhatsApp OTP; <30s delivery |
| O-03 | Arabic-first bilingual UX | RTL as default; 100% Arabic content parity |
| O-04 | 17-state order management | All order lifecycle states implemented; zero state additions |
| O-05 | 99.99% uptime | Measured monthly; max 4.32min downtime/month |
| O-06 | p95 response time <200ms | API response times at 95th percentile |
| O-07 | 477 use cases implemented | Full coverage across 13 blocks, 23 modules |
| O-08 | 974 test points | 100% pass rate for production deployment |

## 4. Scope

### 4.1 Architecture Scale

| Metric | Count |
|--------|-------|
| **Building Blocks** | 13 |
| **Modules** | 23 |
| **Use Cases** | 477 |
| **Actors** | 7 |
| **Threats** | 55 |
| **Test Points** | 974 |
| **Non-negotiable Rules** | 26 |

### 4.2 Building Blocks

1. **Identity & Access** - Phone-first auth, RBAC, KYC
2. **Product Catalog** - Multi-vendor products, categories, attributes
3. **Store Management** - Vendor storefronts, templates, branding
4. **Search & Discovery** - Product search, filtering, recommendations
5. **Cart & Checkout** - Cart management, order placement, validation
6. **Order Management** - 17-state lifecycle, master/sub-orders
7. **Payment & Wallet** - Wallet funding, transactions, escrow
8. **Shipping & Delivery** - Delivery codes, agent assignment, tracking
9. **Returns & Refunds** - Return requests, refund processing, merchant policies
10. **Notifications** - SMS, WhatsApp, push, in-app
11. **Analytics & Reporting** - Merchant dashboards, admin reports
12. **Content & CMS** - Pages, banners, promotions, governance
13. **Platform Administration** - System config, moderation, compliance

### 4.3 Modules (23)

| Block | Module |
|-------|--------|
| Identity & Access | M01: Registration & Login |
| Identity & Access | M02: Profile Management |
| Identity & Access | M03: KYC & Verification |
| Product Catalog | M04: Product CRUD |
| Product Catalog | M05: Category Management |
| Product Catalog | M06: Inventory Management |
| Store Management | M07: Storefront Setup |
| Store Management | M08: Store Templates |
| Search & Discovery | M09: Search Engine |
| Search & Discovery | M10: Recommendations |
| Cart & Checkout | M11: Cart Service |
| Cart & Checkout | M12: Checkout Flow |
| Order Management | M13: Order Lifecycle |
| Order Management | M14: Sub-Order Management |
| Payment & Wallet | M15: Wallet Service |
| Payment & Wallet | M16: Escrow Engine |
| Shipping & Delivery | M17: Delivery Management |
| Shipping & Delivery | M18: Delivery Code System |
| Returns & Refunds | M19: Return Processing |
| Returns & Refunds | M20: Refund Engine |
| Notifications | M21: Notification Hub |
| Analytics & Reporting | M22: Analytics Engine |
| Platform Administration | M23: Admin Panel |

## 5. Constraints

- **26 non-negotiable platform rules** (see `project-constraints.md`)
- Zero card/BNPL payment integration
- 100% custom build—no e-commerce frameworks (Medusa.js, etc.)
- Arabic-first with full RTL support
- ZATCA-compliant invoicing
- 5-year invoice retention

## 6. Success Criteria

| Criterion | Target | Measurement |
|-----------|--------|-------------|
| Test Coverage | 974 test points, 100% pass | Automated test suite |
| Availability | 99.99% uptime | Monthly monitoring |
| Performance | p95 < 200ms | API response time monitoring |
| Security | 0 critical vulnerabilities | Penetration testing |
| Compliance | ZATCA audit pass | Quarterly audit |
| User Adoption | 10,000 merchants, 100,000 customers | Monthly reports |
| Transaction Volume | 50M YER monthly GMV | Financial reports |

## 7. Assumptions

1. SMS/WhatsApp OTP delivery is reliable in Yemen
2. Internet connectivity supports real-time wallet transactions
3. Merchants have basic smartphone/computer literacy
4. Yemeni banking infrastructure supports wallet-to-wallet transfers
5. ZATCA regulations remain stable for 24 months

## 8. Risks

| Risk | Impact | Mitigation |
|------|--------|------------|
| SMS delivery delays | High | Multi-provider SMS failover |
| Low merchant adoption | High | Onboarding support, Arabic UI |
| Wallet fraud | High | Transaction limits, velocity checks |
| Regulatory changes | Medium | Modular compliance layer |
| Infrastructure outages | High | Multi-region deployment |

## 9. Approvals

| Role | Name | Signature | Date |
|------|------|-----------|------|
| Project Sponsor | _____________ | _____________ | ____/____/____ |
| Technical Lead | _____________ | _____________ | ____/____/____ |
| Security Officer | _____________ | _____________ | ____/____/____ |
| QA Lead | _____________ | _____________ | ____/____/____ |

---

*Document Version: 1.0 | Last Updated: 2026-09-13 | Classification: Confidential*
