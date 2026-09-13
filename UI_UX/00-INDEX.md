# YemenMart UI/UX Specification Suite — Master Index

---

| Field            | Value                                                                 |
|------------------|-----------------------------------------------------------------------|
| **Version**      | 1.1.0                                                                 |
| **Status**       | Draft — Pending Review                                                |
| **Last Updated** | 2026-09-13                                                            |
| **Author**       | YemenMart Architecture Team                                           |
| **Scope**        | Complete UI/UX specification for all 5 actor portals                 |
| **Review Due**   | 2026-09-26                                                            |

---

## Brand Colors

| Color | Name | Hex | Usage |
|-------|------|-----|-------|
| Navy Blue | Primary | **#1B2A4A** | Primary buttons, headers, navigation, links |
| Orange | Accent/CTA | **#F57C20** | Call-to-action buttons, highlights, badges, accents |

---

## 1. Project Overview

YemenMart is a multi-vendor e-commerce marketplace built for the Yemeni market. The platform is **Arabic-first** with right-to-left (RTL) design as the default, supporting a secondary English language toggle.

### 1.1 Core Platform Characteristics

| Attribute | Detail |
|---|---|
| Primary Language | Arabic (RTL) |
| Secondary Language | English (LTR toggle) |
| Design Approach | Mobile-first responsive |
| Payment Model | Wallet-only (no cards, no BNPL) |
| Authentication | SMS-only (OTP) — no email login |
| Order Lifecycle | 17 distinct states |
| Actors | 5: Customer, Vendor, Admin, Delivery Provider, System |
| Modules | 23 (M01–M23) across 13 building blocks (B01–B13) |
| Functional Requirements | 17 (FR-001 to FR-017) |
| Test Points | 974 |
| Non-negotiable Constraints | 26 |

### 1.2 Technology Stack

| Layer | Technology |
|---|---|
| Customer Storefront | Next.js 15 (App Router) |
| Vendor Portal | React 18 + Vite |
| Mobile Apps | React Native |
| Styling | Tailwind CSS 4 |
| Primary Database | PostgreSQL 15 |
| Cache / Sessions | Redis 7 |
| Search Engine | Elasticsearch 8 |
| Object Storage | MinIO (S3-compatible) |

---

## 2. Specification Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│                      00-INDEX.md (this file)                    │
│                  Master navigation & status dashboard            │
├──────────────────────────────────────────────────────────────────┤
│  STRATEGIC LAYER                                                 │
│  ├── DESIGN-SYSTEM.md               — Brand tokens, colors       │
│  ├── DESIGN-COORDINATION-GUIDE.md   — Multi-designer workflow    │
│  ├── 01-UI-UX-MASTER-PLAN.md        — Full SDLC UI/UX plan     │
│  └── 02-DESIGN-GUIDELINES.md        — Design tokens, RTL rules  │
├──────────────────────────────────────────────────────────────────┤
│  PORTAL LAYER (per-actor specifications)                        │
│  ├── customer-storefront/                                        │
│  │   └── 03-PORTAL-CUSTOMER-STOREFRONT-01.md                   │
│  ├── vendor-panel/                                               │
│  │   └── 04-PORTAL-VENDOR-PANEL-01.md                          │
│  ├── admin-panel/                                                │
│  │   └── 05-PORTAL-ADMIN-PANEL-01.md                           │
│  ├── delivery-provider/                                          │
│  │   └── 06-PORTAL-DELIVERY-PROVIDER-01.md                     │
│  └── auth-system/                                                │
│      └── 07-PORTAL-AUTH-SYSTEM-01.md                            │
├──────────────────────────────────────────────────────────────────┤
│  INFRASTRUCTURE LAYER                                           │
│  ├── 08-COMPONENT-LIBRARY-SPEC.md   — Widget/component catalog  │
│  ├── 09-DATA-FLOW-MAPPING-00.md     — API ↔ UI data contracts  │
│  └── 10-ERROR-STATES-AND-EDGE-CASES-00.md — Error/empty/loading │
├──────────────────────────────────────────────────────────────────┤
│  QUALITY LAYER                                                  │
│  ├── 11-HCI-METRICS-AND-CRITERIA.md — Usability test criteria   │
│  └── 12-VISUAL-PREVIEW-DESCRIPTIONS.md — Screen-by-screen mockups│
├──────────────────────────────────────────────────────────────────┤
│  GOVERNANCE LAYER                                               │
│  ├── CONFLICTS-AND-DECISIONS.md      — Open decisions log       │
│  ├── UI-UX-MISSING-INFORMATION.md    — Gaps & unknowns          │
│  └── UI-UX-COVERAGE-REPORT.md       — Coverage matrix           │
└──────────────────────────────────────────────────────────────────┘
```

---

## 3. File Tree

```
UI_UX/
│
├── 00-INDEX.md                                  ← YOU ARE HERE
├── DESIGN-SYSTEM.md                             ← NEW: Brand tokens, colors, typography
├── DESIGN-COORDINATION-GUIDE.md                 ← NEW: Multi-designer coordination
├── 01-UI-UX-MASTER-PLAN.md
├── 02-DESIGN-GUIDELINES.md
│
├── customer-storefront/
│   └── 03-PORTAL-CUSTOMER-STOREFRONT-01.md      — Home, Search, PDP, Cart, Checkout
│
├── vendor-panel/
│   └── 04-PORTAL-VENDOR-PANEL-01.md             — Dashboard, Products, Orders
│
├── admin-panel/
│   └── 05-PORTAL-ADMIN-PANEL-01.md              — Dashboard, Users, Finance
│
├── delivery-provider/
│   └── 06-PORTAL-DELIVERY-PROVIDER-01.md        — Dashboard, Assignments, Proof
│
├── auth-system/
│   └── 07-PORTAL-AUTH-SYSTEM-01.md              — Registration, Login, Recovery
│
├── 08-COMPONENT-LIBRARY-SPEC.md                 — Shared widget catalog
│
├── 09-DATA-FLOW-MAPPING-00.md                   — Overview & glossary
│
├── 10-ERROR-STATES-AND-EDGE-CASES-00.md         — Catalog & conventions
│
├── 11-HCI-METRICS-AND-CRITERIA.md               — Usability heuristics & KPIs
├── 12-VISUAL-PREVIEW-DESCRIPTIONS.md            — Screen mockup descriptions
│
├── CONFLICTS-AND-DECISIONS.md                   — Decision log
├── UI-UX-MISSING-INFORMATION.md                 — Open questions
└── UI-UX-COVERAGE-REPORT.md                     — Traceability matrix
```

**Total files: 16**

---

## 4. Portal Reference

| Portal | Files | Pages | Primary Actor |
|--------|-------|-------|---------------|
| Customer Storefront | customer-storefront/03-* | ~42 | Customer |
| Vendor Panel | vendor-panel/04-* | ~28 | Vendor |
| Admin Panel | admin-panel/05-* | ~35 | Admin |
| Delivery Provider | delivery-provider/06-* | ~12 | Delivery Provider |
| Auth System | auth-system/07-* | ~8 | All Actors |
| **Total** | **5 portal folders** | **~125** | — |

---

## 5. Aggregate Metrics

| Metric | Count |
|--------|-------|
| Specification Files | 16 |
| Portals | 5 |
| Total Pages (estimated) | ~125 |
| Shared Widgets (Component Library) | ~65 |
| Data Flow Diagrams | ~38 |
| Error/Edge Case Scenarios | ~120 |
| Modules Covered | 23 (M01–M23) |
| Building Blocks | 13 (B01–B13) |
| Functional Requirements | 17 (FR-001–FR-017) |
| Test Points | 974 |
| Non-negotiable Constraints | 26 |
| Order States | 17 |
| Actors | 5 |

---

## 6. Cross-Reference Conventions

### 6.1 Module References

Modules are referenced as `M##` (e.g., M01, M23). The full module registry:

| Module | Name | Primary Portal(s) |
|--------|------|--------------------|
| M01 | Authentication & Identity | auth-system/07-AUTH |
| M02 | Product Catalog | customer-storefront/03-CUSTOMER, vendor-panel/04-VENDOR |
| M03 | Search & Discovery | customer-storefront/03-CUSTOMER |
| M04 | Shopping Cart | customer-storefront/03-CUSTOMER |
| M05 | Checkout & Payment | customer-storefront/03-CUSTOMER |
| M06 | Wallet Management | customer-storefront/03-CUSTOMER, admin-panel/05-ADMIN |
| M07 | Order Management | customer-storefront/03-CUSTOMER, vendor-panel/04-VENDOR, admin-panel/05-ADMIN |
| M08 | Vendor Management | vendor-panel/04-VENDOR, admin-panel/05-ADMIN |
| M09 | Product Management | vendor-panel/04-VENDOR |
| M10 | Inventory Management | vendor-panel/04-VENDOR |
| M11 | Pricing & Promotions | vendor-panel/04-VENDOR, admin-panel/05-ADMIN |
| M12 | Delivery & Logistics | delivery-provider/06-DELIVERY, admin-panel/05-ADMIN |
| M13 | Notifications | auth-system/07-AUTH (cross-cutting) |
| M14 | Reviews & Ratings | customer-storefront/03-CUSTOMER |
| M15 | Customer Support | customer-storefront/03-CUSTOMER, admin-panel/05-ADMIN |
| M16 | Analytics & Reporting | vendor-panel/04-VENDOR, admin-panel/05-ADMIN |
| M17 | Content Management | admin-panel/05-ADMIN |
| M18 | User Profile & Settings | All Portals |
| M19 | Address Management | customer-storefront/03-CUSTOMER |
| M20 | Category Management | admin-panel/05-ADMIN |
| M21 | Commission & Finance | admin-panel/05-ADMIN |
| M22 | Fraud & Compliance | admin-panel/05-ADMIN |
| M23 | System Administration | admin-panel/05-ADMIN |

### 6.2 Functional Requirement References

Requirements are referenced as `FR-###` (e.g., FR-001, FR-017).

### 6.3 Building Block References

Building blocks are referenced as `B##` (e.g., B01, B13).

### 6.4 Page ID Convention

Pages follow the pattern: `[PORTAL]-[MODULE]-[SEQUENCE]`
- Example: `CUST-M02-003` = Customer Portal, Module 02 (Product Catalog), Page 3

### 6.5 Widget ID Convention

Widgets follow the pattern: `WGT-[CATEGORY]-[NAME]-[VERSION]`
- Example: `WGT-CARD-PRODUCT-V2` = Product Card widget, version 2

### 6.6 Data Flow Convention

Data flows are referenced as: `DF-[SOURCE]-[TARGET]-[NAME]`
- Example: `DF-CUST-API-CART-SYNC` = Customer → API: Cart Sync

---

## 7. Status Legend

| Symbol | Status | Meaning |
|--------|--------|---------|
| ✅ | Complete | Spec written, reviewed, and frozen |
| 🔄 | In Progress | Actively being drafted |
| 📝 | Draft | Initial draft exists, not yet reviewed |
| ⏳ | Pending | Not yet started |
| ⚠️ | Blocked | Dependent on unresolved decision |
| ❌ | Removed | Scope cut or deprecated |
| 🔁 | Revised | Updated after initial freeze |
| ❓ | Unknown | Information gap — see MISSING-INFORMATION |

---

## 8. Coverage Status Summary

| Document | Status | Pages | Notes |
|----------|--------|-------|-------|
| 00-INDEX.md | ✅ Complete | — | This file |
| DESIGN-SYSTEM.md | 📝 Draft | — | Brand tokens: Navy Blue #1B2A4A, Orange #F57C20 |
| DESIGN-COORDINATION-GUIDE.md | 📝 Draft | — | Multi-designer coordination workflow |
| 01-UI-UX-MASTER-PLAN.md | 📝 Draft | — | Awaiting tech stack confirmation |
| 02-DESIGN-GUIDELINES.md | 📝 Draft | — | RTL rules drafted; tokens pending |
| customer-storefront/03-PORTAL-CUSTOMER-STOREFRONT-01.md | 🔄 In Progress | ~42 | Home, Search, PDP in progress |
| vendor-panel/04-PORTAL-VENDOR-PANEL-01.md | 📝 Draft | ~28 | Dashboard, Products drafted |
| admin-panel/05-PORTAL-ADMIN-PANEL-01.md | ⏳ Pending | ~35 | Dashboard, Users, Vendors |
| delivery-provider/06-PORTAL-DELIVERY-PROVIDER-01.md | ⏳ Pending | ~12 | All delivery pages |
| auth-system/07-PORTAL-AUTH-SYSTEM-01.md | 📝 Draft | ~8 | SMS OTP flow drafted |
| 08-COMPONENT-LIBRARY-SPEC.md | 📝 Draft | — | ~65 widgets cataloged |
| 09-DATA-FLOW-MAPPING-00.md | 📝 Draft | — | Overview written |
| 10-ERROR-STATES-AND-EDGE-CASES-00.md | 📝 Draft | — | Conventions defined |
| 11-HCI-METRICS-AND-CRITERIA.md | ⏳ Pending | — | Heuristics TBD |
| 12-VISUAL-PREVIEW-DESCRIPTIONS.md | ⏳ Pending | — | Screen descriptions TBD |
| CONFLICTS-AND-DECISIONS.md | 📝 Draft | — | 8 open decisions |
| UI-UX-MISSING-INFORMATION.md | 📝 Draft | — | 14 open questions |
| UI-UX-COVERAGE-REPORT.md | ⏳ Pending | — | Traceability matrix |

### Coverage by Portal

| Portal | Drafted | In Progress | Pending | Total |
|--------|---------|-------------|---------|-------|
| Customer Storefront | 1 | 1 | 0 | 2 |
| Vendor Panel | 1 | 0 | 0 | 1 |
| Admin Panel | 0 | 0 | 1 | 1 |
| Delivery Provider | 0 | 0 | 1 | 1 |
| Auth System | 1 | 0 | 0 | 1 |

### Coverage by Building Block

| Block | Name | Document(s) | Status |
|-------|------|-------------|--------|
| B01 | Identity & Access | auth-system/07-AUTH-01 | 📝 Draft |
| B02 | Product Domain | customer-storefront/03-CUSTOMER-01, vendor-panel/04-VENDOR-01 | 🔄 / 📝 |
| B03 | Search & Discovery | customer-storefront/03-CUSTOMER-01 | 🔄 In Progress |
| B04 | Cart & Checkout | customer-storefront/03-CUSTOMER-01 | 🔄 In Progress |
| B05 | Payment & Wallet | customer-storefront/03-CUSTOMER-01 | 🔄 In Progress |
| B06 | Order Lifecycle | customer-storefront/03-CUSTOMER-01, vendor-panel/04-VENDOR-01, admin-panel/05-ADMIN-01 | 📝 / ⏳ / ⏳ |
| B07 | Vendor Operations | vendor-panel/04-VENDOR-01 | 📝 Draft |
| B08 | Delivery & Fulfillment | delivery-provider/06-DELIVERY-01 | ⏳ Pending |
| B09 | Admin Governance | admin-panel/05-ADMIN-01 | ⏳ Pending |
| B10 | Notifications | auth-system/07-AUTH-01 (cross-cutting) | 📝 Draft |
| B11 | Reviews & Trust | customer-storefront/03-CUSTOMER-01 | 🔄 In Progress |
| B12 | Analytics & Reporting | vendor-panel/04-VENDOR-01, admin-panel/05-ADMIN-01 | 📝 / ⏳ |
| B13 | Content & CMS | admin-panel/05-ADMIN-01 | ⏳ Pending |

---

## 9. Quick Navigation by Role

### For Product Managers
1. Start with `01-UI-UX-MASTER-PLAN.md` for the full roadmap
2. Review `CONFLICTS-AND-DECISIONS.md` for open decisions requiring input
3. Check `UI-UX-MISSING-INFORMATION.md` for gaps needing stakeholder answers
4. Use `UI-UX-COVERAGE-REPORT.md` to track completion

### For UI/UX Designers
1. Start with `DESIGN-SYSTEM.md` for brand tokens, colors (Navy Blue #1B2A4A, Orange #F57C20), and typography
2. Review `DESIGN-COORDINATION-GUIDE.md` for multi-designer workflow and coordination
3. Check `02-DESIGN-GUIDELINES.md` for RTL rules and spacing
4. Review `08-COMPONENT-LIBRARY-SPEC.md` for the shared widget catalog
5. Navigate portal files (`customer-storefront/`, `vendor-panel/`, etc.) for page-by-page specs
6. Reference `12-VISUAL-PREVIEW-DESCRIPTIONS.md` for screen mockups

### For Frontend Developers
1. Start with `DESIGN-SYSTEM.md` for brand tokens and Tailwind CSS integration
2. Review `08-COMPONENT-LIBRARY-SPEC.md` for widget contracts
3. Check `09-DATA-FLOW-MAPPING-00.md` for API ↔ UI data contracts
4. Reference `10-ERROR-STATES-AND-EDGE-CASES-00.md` for loading/error/empty states
5. Navigate portal files for page-specific behavior

### For Backend Developers
1. Start with `09-DATA-FLOW-MAPPING-00.md` for API surface requirements
2. Review `10-ERROR-STATES-AND-EDGE-CASES-00.md` for error codes and messages
3. Check `01-UI-UX-MASTER-PLAN.md` for technical constraints

### For QA Engineers
1. Start with `UI-UX-COVERAGE-REPORT.md` for the 974 test point mapping
2. Review `10-ERROR-STATES-AND-EDGE-CASES-00.md` for edge case coverage
3. Check `11-HCI-METRICS-AND-CRITERIA.md` for usability test criteria
4. Reference `CONFLICTS-AND-DECISIONS.md` for known edge cases

### For Project Managers
1. Start with `00-INDEX.md` (this file) for status overview
2. Review `UI-UX-COVERAGE-REPORT.md` for completion tracking
3. Check `CONFLICTS-AND-DECISIONS.md` for blockers
4. Monitor `UI-UX-MISSING-INFORMATION.md` for outstanding questions

---

## 10. Reading Order (Recommended)

For a first-time reader seeking full context:

1. `00-INDEX.md` — This file (navigation & status)
2. `DESIGN-SYSTEM.md` — Brand tokens, colors, typography (NEW)
3. `DESIGN-COORDINATION-GUIDE.md` — Multi-designer coordination (NEW)
4. `01-UI-UX-MASTER-PLAN.md` — Strategic plan & scope
5. `02-DESIGN-GUIDELINES.md` — Design system foundation
6. `auth-system/07-PORTAL-AUTH-SYSTEM-01.md` — Auth (touches all actors)
7. `customer-storefront/03-PORTAL-CUSTOMER-STOREFRONT-01.md` — Largest portal
8. `vendor-panel/04-PORTAL-VENDOR-PANEL-01.md` — Vendor experience
9. `admin-panel/05-PORTAL-ADMIN-PANEL-01.md` — Admin experience
10. `delivery-provider/06-PORTAL-DELIVERY-PROVIDER-01.md` — Delivery partner
11. `08-COMPONENT-LIBRARY-SPEC.md` — Shared components
12. `09-DATA-FLOW-MAPPING-00.md` — Data contracts overview
13. `10-ERROR-STATES-AND-EDGE-CASES-00.md` — Error conventions
14. `11-HCI-METRICS-AND-CRITERIA.md` — Quality bar
15. `12-VISUAL-PREVIEW-DESCRIPTIONS.md` — Screen descriptions
16. `CONFLICTS-AND-DECISIONS.md` — Open items
17. `UI-UX-MISSING-INFORMATION.md` — Gaps
18. `UI-UX-COVERAGE-REPORT.md` — Full traceability

---

## 11. Related Source Documents

| Document | Location | Description |
|----------|----------|-------------|
| PRD (Product Requirements Document) | `../docs/PRD.md` | Business requirements |
| Technical Architecture | `../docs/ARCHITECTURE.md` | System design |
| API Specification | `../docs/api/` | OpenAPI / Swagger specs |
| Database Schema | `../docs/db/` | PostgreSQL DDL & migrations |
| Test Plan | `../tests/TEST-PLAN.md` | 974 test points detail |
| Security Specification | `../docs/SECURITY.md` | 26 non-negotiable constraints |
| Deployment Guide | `../docs/DEPLOY.md` | Infrastructure & CI/CD |

---

## 12. Changelog

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 0.1.0 | 2026-09-01 | Architecture Team | Initial scaffold |
| 1.0.0 | 2026-09-12 | Architecture Team | Full index with coverage status |
| 1.1.0 | 2026-09-13 | Architecture Team | Restructured folders; added DESIGN-SYSTEM.md and DESIGN-COORDINATION-GUIDE.md; corrected brand colors to Navy Blue #1B2A4A and Orange #F57C20 |

---

*This index is the single source of truth for the YemenMart UI/UX specification suite. Update it whenever a spec file is added, removed, or its status changes.*
