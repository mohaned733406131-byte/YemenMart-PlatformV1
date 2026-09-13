# YemenMart UI/UX Master Plan

**Version:** 1.0.0  
**Date:** 2026-09-13  
**Status:** Draft  
**Classification:** Internal — Engineering & Design

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Scope and Out of Scope](#2-scope-and-out-of-scope)
3. [Portal Architecture](#3-portal-architecture)
4. [Platform Architecture](#4-platform-architecture)
5. [Complete Page Inventory](#5-complete-page-inventory)
6. [Page Counts by Portal and Module](#6-page-counts-by-portal-and-module)
7. [Module Counts](#7-module-counts)
8. [Priority Matrix](#8-priority-matrix)
9. [Deliverables](#9-deliverables)
10. [Cross-Portal Consistency Rules](#10-cross-portal-consistency-rules)
11. [Design Principles](#11-design-principles)
12. [Responsive Strategy](#12-responsive-strategy)
13. [Accessibility Strategy](#13-accessibility-strategy-wcag-21-aa)
14. [RTL/LTR Strategy](#14rtlltr-strategy)
15. [Internationalization Strategy](#15-internationalization-strategy)
16. [Traceability Strategy](#16-traceability-strategy)
17. [Review Gates](#17-review-gates)
18. [Completion Criteria](#18-completion-criteria)

---

## 1. Executive Summary

YemenMart is a **multi-vendor e-commerce marketplace** purpose-built for the Yemeni market. The platform enables customers to browse, purchase, and receive goods through a network of local vendors and delivery providers, with **wallet-only payments** and **SMS-only authentication**.

This master plan establishes the complete UI/UX specification suite governing **four portals** (Customer Storefront, Vendor Panel, Admin Panel, Delivery Provider), **23 modules** (M01–M23) across **13 building blocks** (B01–B13), and **974 test points** validating **17 functional requirements** (FR-001–FR-017).

### Key Metrics

| Metric | Value |
|---|---|
| Total Portals | 4 |
| Total Modules | 23 (M01–M23) |
| Total Building Blocks | 13 (B01–B13) |
| Total Pages | 127 |
| Total Functional Requirements | 17 (FR-001–FR-017) |
| Total Test Points | 974 |
| Platform Constraints | 26 (non-negotiable) |
| Order States | 17 |
| Actors | 5 (Customer, Vendor, Admin, Delivery Provider, System) |

### Design Philosophy

The platform is **Arabic-first, RTL-native, mobile-first, wallet-only, and SMS-only**. Every design decision flows from these five pillars. English is a secondary language accessible via toggle; all layouts mirror cleanly when switching between RTL and LTR.

---

## 2. Scope and Out of Scope

### 2.1 In Scope

| Area | Details |
|---|---|
| Customer Storefront | Web (Next.js 15), Mobile (React Native), responsive layouts |
| Vendor Panel | Web (React 18 + Vite), responsive layouts |
| Admin Panel | Web (React 18 + Vite), responsive layouts |
| Delivery Provider Panel | Web (React 18 + Vite), responsive layouts |
| All 23 Modules (M01–M23) | Full UI/UX specifications, wireframes, interaction patterns |
| All 13 Building Blocks (B01–B13) | Component libraries, design tokens, shared patterns |
| 17 Functional Requirements | FR-001 through FR-017 |
| 974 Test Points | Comprehensive UI/UX validation |
| 26 Platform Constraints | Non-negotiable technical and business rules |
| 17 Order States | Complete state machine visualization and flows |
| Design System | Tailwind CSS 4, design tokens, component library |
| Accessibility | WCAG 2.1 AA compliance |
| Internationalization | Arabic (primary), English (secondary) |
| Responsive Strategy | Mobile-first across all breakpoints |

### 2.2 Out of Scope

| Area | Details |
|---|---|
| Backend API implementation | Covered in separate backend specification |
| Database schema design | Covered in separate data architecture document |
| DevOps / CI/CD pipelines | Covered in infrastructure specification |
| Third-party integrations | Payment gateway (wallet provider), SMS provider — integration specs only |
| Marketing / SEO content strategy | Separate marketing plan |
| Legal / compliance documentation | Separate compliance workstream |
| Physical logistics operations | Delivery partner onboarding process |
| User research / usability testing | Separate UX research plan |
| Performance benchmarking | Separate performance engineering plan |
| Content management system | Admin-side CMS for static pages |

---

## 3. Portal Architecture

### 3.1 Portal Overview

YemenMart operates four distinct portals, each tailored to its actor's workflows and permissions.

```
┌─────────────────────────────────────────────────────────────┐
│                      YemenMart Platform                     │
├──────────────┬──────────────┬──────────────┬───────────────┤
│   Customer   │    Vendor    │    Admin     │   Delivery    │
│  Storefront  │    Panel     │    Panel     │   Provider    │
│  (CS)        │    (VP)      │    (AP)      │   (DP)        │
├──────────────┼──────────────┼──────────────┼───────────────┤
│  Next.js 15  │ React+Vite  │ React+Vite  │ React+Vite   │
│  App Router   │              │              │              │
├──────────────┴──────────────┴──────────────┴───────────────┤
│              Shared Design System (B01–B13)                 │
│              Tailwind CSS 4 · Design Tokens                │
├─────────────────────────────────────────────────────────────┤
│              Unified API Layer (REST / WebSocket)           │
├──────────────┬──────────────┬──────────────┬───────────────┤
│  PostgreSQL  │    Redis     │ Elasticsearch│    MinIO      │
│     15       │      7       │      8       │   (Storage)   │
└──────────────┴──────────────┴──────────────┴───────────────┘
```

### 3.2 Customer Storefront (CS)

- **Technology:** Next.js 15 (App Router), React 18, Tailwind CSS 4
- **Audience:** End consumers browsing and purchasing products
- **Authentication:** SMS OTP only
- **Payment:** Wallet balance only (no cards, no BNPL)
- **Language:** Arabic (default), English (toggle)
- **Layout:** RTL-first, mobile-first responsive

### 3.3 Vendor Panel (VP)

- **Technology:** React 18 + Vite, Tailwind CSS 4
- **Audience:** Vendors managing products, orders, and store settings
- **Authentication:** SMS OTP only
- **Features:** Product CRUD, order management, analytics, wallet, store profile
- **Language:** Arabic (default), English (toggle)
- **Layout:** RTL-first, responsive

### 3.4 Admin Panel (AP)

- **Technology:** React 18 + Vite, Tailwind CSS 4
- **Audience:** Platform administrators managing the entire marketplace
- **Authentication:** SMS OTP only
- **Features:** Vendor approval, order oversight, user management, system config, reports
- **Language:** Arabic (default), English (toggle)
- **Layout:** RTL-first, responsive

### 3.5 Delivery Provider Panel (DP)

- **Technology:** React 18 + Vite, Tailwind CSS 4
- **Audience:** Delivery personnel managing assigned deliveries
- **Authentication:** SMS OTP only
- **Features:** Delivery queue, route management, status updates, proof of delivery
- **Language:** Arabic (default), English (toggle)
- **Layout:** RTL-first, mobile-first responsive (primary use: mobile)

---

## 4. Platform Architecture

### 4.1 Platform Matrix

| Platform | Customer | Vendor | Admin | Delivery |
|---|---|---|---|---|
| Web (Desktop) | ✅ | ✅ | ✅ | ✅ |
| Web (Tablet) | ✅ | ✅ | ✅ | ✅ |
| Web (Mobile) | ✅ | ✅ | ✅ | ✅ |
| Native Mobile App | ✅ | ❌ | ❌ | ✅ |
| PWA | ✅ | ❌ | ❌ | ❌ |

### 4.2 Technology Stack

| Layer | Technology | Version |
|---|---|---|
| Customer Web | Next.js (App Router) | 15.x |
| Vendor/Admin/Delivery Web | React + Vite | 18.x |
| Mobile Apps | React Native | Latest |
| Styling | Tailwind CSS | 4.x |
| Database | PostgreSQL | 15 |
| Cache | Redis | 7 |
| Search | Elasticsearch | 8 |
| Object Storage | MinIO | Latest |
| Language Runtime | Node.js | 20 LTS |

### 4.3 Responsive Breakpoints

| Breakpoint | Width | Target |
|---|---|---|
| `xs` | 0–479px | Small mobile |
| `sm` | 480–767px | Large mobile |
| `md` | 768–1023px | Tablet portrait |
| `lg` | 1024–1279px | Tablet landscape / small desktop |
| `xl` | 1280–1535px | Desktop |
| `2xl` | 1536px+ | Large desktop |

---

## 5. Complete Page Inventory

### 5.1 ID Convention

```
{PORTAL}-{MODULE}-{TYPE}-{SEQUENCE}
```

| Component | Meaning |
|---|---|
| Portal | `CS` = Customer Storefront, `VP` = Vendor Panel, `AP` = Admin Panel, `DP` = Delivery Provider |
| Module | Two-letter module code (e.g., `HM` = Home, `PR` = Products) |
| Type | Page type: `PG` = Page, `MD` = Modal, `DR` = Drawer, `FL` = Flow |
| Sequence | Three-digit sequential number |

### 5.2 Module Codes

| Code | Module Name | Description |
|---|---|---|
| HM | Home | Homepage / landing |
| PR | Products | Product listing / catalog |
| PD | Product Detail | Single product view |
| CT | Cart | Shopping cart |
| CK | Checkout | Checkout flow |
| OR | Orders | Order history / detail |
| WL | Wallet | Wallet / payments |
| AC | Account | Profile / settings |
| ST | Store | Vendor store page |
| DB | Dashboard | Portal dashboard |
| PM | Products Mgmt | Vendor product management |
| OM | Orders Mgmt | Vendor/Admin order management |
| UM | User Mgmt | Admin user management |
| VM | Vendor Mgmt | Admin vendor management |
| DM | Delivery Mgmt | Delivery management |
| RV | Reviews | Reviews / ratings |
| NT | Notifications | Notification center |
| SR | Search | Search / filters |
| CA | Categories | Category browsing |
| SP | Support | Customer support |
| AN | Analytics | Reports / analytics |
| CF | Config | System configuration |
| LG | Legal | Legal / policy pages |

### 5.3 Customer Storefront (CS) — 45 Pages

#### M01 — Home (HM)

| ID | Page Name | Type | FR Ref | Priority |
|---|---|---|---|---|
| CS-HM-001 | Homepage | PG | FR-001 | P0 |
| CS-HM-002 | Homepage — Featured Banner | PG | FR-001 | P0 |
| CS-HM-003 | Homepage — Category Quick Links | PG | FR-001 | P1 |
| CS-HM-004 | Homepage — Flash Deals | PG | FR-001 | P1 |
| CS-HM-005 | Homepage — Recommended For You | PG | FR-001 | P2 |
| CS-HM-006 | Homepage — Trending Products | PG | FR-001 | P2 |

#### M02 — Products (PR)

| ID | Page Name | Type | FR Ref | Priority |
|---|---|---|---|---|
| CS-PR-001 | Product Listing | PG | FR-002 | P0 |
| CS-PR-002 | Product Listing — Filters Drawer | DR | FR-002 | P0 |
| CS-PR-003 | Product Listing — Sort Options | MD | FR-002 | P1 |
| CS-PR-004 | Product Listing — Grid/List Toggle | PG | FR-002 | P2 |

#### M03 — Product Detail (PD)

| ID | Page Name | Type | FR Ref | Priority |
|---|---|---|---|---|
| CS-PD-001 | Product Detail | PG | FR-003 | P0 |
| CS-PD-002 | Product Detail — Image Gallery | PG | FR-003 | P0 |
| CS-PD-003 | Product Detail — Variant Selector | PG | FR-003 | P1 |
| CS-PD-004 | Product Detail — Seller Info | PG | FR-003 | P1 |
| CS-PD-005 | Product Detail — Reviews Section | PG | FR-003 | P2 |
| CS-PD-006 | Product Detail — Related Products | PG | FR-003 | P2 |

#### M04 — Cart (CT)

| ID | Page Name | Type | FR Ref | Priority |
|---|---|---|---|---|
| CS-CT-001 | Cart | PG | FR-004 | P0 |
| CS-CT-002 | Cart — Quantity Adjuster | PG | FR-004 | P0 |
| CS-CT-003 | Cart — Remove Item Confirmation | MD | FR-004 | P1 |
| CS-CT-004 | Cart — Empty State | PG | FR-004 | P1 |
| CS-CT-005 | Cart — Delivery Fee Estimate | PG | FR-004 | P1 |

#### M05 — Checkout (CK)

| ID | Page Name | Type | FR Ref | Priority |
|---|---|---|---|---|
| CS-CK-001 | Checkout — Address Selection | PG | FR-005 | P0 |
| CS-CK-002 | Checkout — Address Form (Add/Edit) | FL | FR-005 | P0 |
| CS-CK-003 | Checkout — Payment (Wallet) | PG | FR-005 | P0 |
| CS-CK-004 | Checkout — Order Summary | PG | FR-005 | P0 |
| CS-CK-005 | Checkout — Order Confirmation | PG | FR-005 | P0 |
| CS-CK-006 | Checkout — Insufficient Balance Warning | MD | FR-005 | P0 |
| CS-CK-007 | Checkout — Wallet Top-Up Redirect | FL | FR-005 | P1 |

#### M06 — Orders (OR)

| ID | Page Name | Type | FR Ref | Priority |
|---|---|---|---|---|
| CS-OR-001 | Order History | PG | FR-006 | P0 |
| CS-OR-002 | Order Detail | PG | FR-006 | P0 |
| CS-OR-003 | Order Tracking (Live) | PG | FR-006 | P0 |
| CS-OR-004 | Order — Cancel Confirmation | MD | FR-006 | P1 |
| CS-OR-005 | Order — Return Request Form | FL | FR-006 | P1 |
| CS-OR-006 | Order — Invoice / Receipt | PG | FR-006 | P2 |
| CS-OR-007 | Order — Rate & Review | FL | FR-006 | P2 |

#### M07 — Wallet (WL)

| ID | Page Name | Type | FR Ref | Priority |
|---|---|---|---|---|
| CS-WL-001 | Wallet Overview | PG | FR-007 | P0 |
| CS-WL-002 | Wallet — Top Up | FL | FR-007 | P0 |
| CS-WL-003 | Wallet — Transaction History | PG | FR-007 | P0 |
| CS-WL-004 | Wallet — Top Up Success | PG | FR-007 | P1 |
| CS-WL-005 | Wallet — Top Up Failed | PG | FR-007 | P1 |

#### M08 — Account (AC)

| ID | Page Name | Type | FR Ref | Priority |
|---|---|---|---|---|
| CS-AC-001 | Account — Profile | PG | FR-008 | P0 |
| CS-AC-002 | Account — Edit Profile | FL | FR-008 | P0 |
| CS-AC-003 | Account — Addresses | PG | FR-008 | P0 |
| CS-AC-004 | Account — Add/Edit Address | FL | FR-008 | P0 |
| CS-AC-005 | Account — Change Phone Number | FL | FR-008 | P1 |
| CS-AC-006 | Account — Language Toggle | MD | FR-008 | P1 |
| CS-AC-007 | Account — Settings | PG | FR-008 | P2 |

#### M09 — Store (ST)

| ID | Page Name | Type | FR Ref | Priority |
|---|---|---|---|---|
| CS-ST-001 | Vendor Store Page | PG | FR-009 | P1 |
| CS-ST-002 | Vendor Store — All Products | PG | FR-009 | P1 |

#### M10 — Search (SR)

| ID | Page Name | Type | FR Ref | Priority |
|---|---|---|---|---|
| CS-SR-001 | Search Results | PG | FR-010 | P0 |
| CS-SR-002 | Search — Autocomplete | PG | FR-010 | P1 |
| CS-SR-003 | Search — No Results | PG | FR-010 | P2 |

#### M11 — Categories (CA)

| ID | Page Name | Type | FR Ref | Priority |
|---|---|---|---|---|
| CS-CA-001 | Categories — Browse | PG | FR-002 | P1 |
| CS-CA-002 | Category — Product Listing | PG | FR-002 | P1 |

#### M12 — Reviews (RV)

| ID | Page Name | Type | FR Ref | Priority |
|---|---|---|---|---|
| CS-RV-001 | Product Reviews List | PG | FR-011 | P2 |
| CS-RV-002 | Write Review Form | FL | FR-011 | P2 |

#### M13 — Notifications (NT)

| ID | Page Name | Type | FR Ref | Priority |
|---|---|---|---|---|
| CS-NT-001 | Notification Center | PG | FR-012 | P1 |
| CS-NT-002 | Notification — Detail | PG | FR-012 | P2 |

#### M14 — Support (SP)

| ID | Page Name | Type | FR Ref | Priority |
|---|---|---|---|---|
| CS-SP-001 | Help Center | PG | FR-013 | P2 |
| CS-SP-002 | FAQ | PG | FR-013 | P2 |
| CS-SP-003 | Contact Support | FL | FR-013 | P2 |

#### M15 — Legal (LG)

| ID | Page Name | Type | FR Ref | Priority |
|---|---|---|---|---|
| CS-LG-001 | Terms & Conditions | PG | FR-017 | P2 |
| CS-LG-002 | Privacy Policy | PG | FR-017 | P2 |
| CS-LG-003 | Return Policy | PG | FR-017 | P2 |

#### M16 — Authentication (Auth)

| ID | Page Name | Type | FR Ref | Priority |
|---|---|---|---|---|
| CS-AU-001 | SMS Login — Phone Entry | PG | FR-008 | P0 |
| CS-AU-002 | SMS Login — OTP Verification | PG | FR-008 | P0 |
| CS-AU-003 | SMS Login — OTP Expired | PG | FR-008 | P1 |
| CS-AU-004 | Logout Confirmation | MD | FR-008 | P1 |

---

### 5.4 Vendor Panel (VP) — 38 Pages

#### M01 — Dashboard (DB)

| ID | Page Name | Type | FR Ref | Priority |
|---|---|---|---|---|
| VP-DB-001 | Vendor Dashboard | PG | FR-009 | P0 |
| VP-DB-002 | Dashboard — Revenue Summary | PG | FR-009 | P0 |
| VP-DB-003 | Dashboard — Order Stats | PG | FR-009 | P1 |
| VP-DB-004 | Dashboard — Recent Activity | PG | FR-009 | P2 |

#### M02 — Products Management (PM)

| ID | Page Name | Type | FR Ref | Priority |
|---|---|---|---|---|
| VP-PM-001 | Product List | PG | FR-009 | P0 |
| VP-PM-002 | Add Product — Basic Info | FL | FR-009 | P0 |
| VP-PM-003 | Add Product — Pricing & Inventory | FL | FR-009 | P0 |
| VP-PM-004 | Add Product — Images | FL | FR-009 | P0 |
| VP-PM-005 | Add Product — Variants | FL | FR-009 | P1 |
| VP-PM-006 | Add Product — SEO & Tags | FL | FR-009 | P2 |
| VP-PM-007 | Edit Product | FL | FR-009 | P0 |
| VP-PM-008 | Product — Bulk Actions | MD | FR-009 | P1 |
| VP-PM-009 | Product — Delete Confirmation | MD | FR-009 | P1 |
| VP-PM-010 | Product — Stock Alert Settings | MD | FR-009 | P2 |

#### M03 — Orders Management (OM)

| ID | Page Name | Type | FR Ref | Priority |
|---|---|---|---|---|
| VP-OM-001 | Vendor Order List | PG | FR-009 | P0 |
| VP-OM-002 | Vendor Order Detail | PG | FR-009 | P0 |
| VP-OM-003 | Order — Confirm Action | MD | FR-009 | P0 |
| VP-OM-004 | Order — Reject Action | MD | FR-009 | P0 |
| VP-OM-005 | Order — Update Status | FL | FR-009 | P1 |
| VP-OM-006 | Order — Print Packing Slip | PG | FR-009 | P2 |

#### M04 — Store Profile (ST)

| ID | Page Name | Type | FR Ref | Priority |
|---|---|---|---|---|
| VP-ST-001 | Store Profile | PG | FR-009 | P1 |
| VP-ST-002 | Edit Store Profile | FL | FR-009 | P1 |
| VP-ST-003 | Store — Logo & Banner Upload | FL | FR-009 | P1 |
| VP-ST-004 | Store — Business Hours | FL | FR-009 | P2 |

#### M05 — Wallet (WL)

| ID | Page Name | Type | FR Ref | Priority |
|---|---|---|---|---|
| VP-WL-001 | Vendor Wallet Overview | PG | FR-007 | P0 |
| VP-WL-002 | Wallet — Withdrawal Request | FL | FR-007 | P0 |
| VP-WL-003 | Wallet — Transaction History | PG | FR-007 | P0 |
| VP-WL-004 | Wallet — Payout History | PG | FR-007 | P1 |

#### M06 — Analytics (AN)

| ID | Page Name | Type | FR Ref | Priority |
|---|---|---|---|---|
| VP-AN-001 | Sales Analytics | PG | FR-009 | P1 |
| VP-AN-002 | Product Performance | PG | FR-009 | P2 |
| VP-AN-003 | Customer Insights | PG | FR-009 | P2 |

#### M07 — Reviews (RV)

| ID | Page Name | Type | FR Ref | Priority |
|---|---|---|---|---|
| VP-RV-001 | Vendor Reviews List | PG | FR-011 | P1 |
| VP-RV-002 | Review — Respond | FL | FR-011 | P2 |

#### M08 — Notifications (NT)

| ID | Page Name | Type | FR Ref | Priority |
|---|---|---|---|---|
| VP-NT-001 | Vendor Notification Center | PG | FR-012 | P1 |

#### M09 — Account (AC)

| ID | Page Name | Type | FR Ref | Priority |
|---|---|---|---|---|
| VP-AC-001 | Vendor Profile | PG | FR-008 | P1 |
| VP-AC-002 | Edit Vendor Profile | FL | FR-008 | P1 |
| VP-AC-003 | Change Phone Number | FL | FR-008 | P1 |
| VP-AC-004 | Language Toggle | MD | FR-008 | P2 |

#### M10 — Authentication (Auth)

| ID | Page Name | Type | FR Ref | Priority |
|---|---|---|---|---|
| VP-AU-001 | SMS Login — Phone Entry | PG | FR-008 | P0 |
| VP-AU-002 | SMS Login — OTP Verification | PG | FR-008 | P0 |
| VP-AU-003 | Vendor Registration | FL | FR-009 | P0 |
| VP-AU-004 | Logout Confirmation | MD | FR-008 | P1 |

---

### 5.5 Admin Panel (AP) — 32 Pages

#### M01 — Dashboard (DB)

| ID | Page Name | Type | FR Ref | Priority |
|---|---|---|---|---|
| AP-DB-001 | Admin Dashboard | PG | FR-014 | P0 |
| AP-DB-002 | Dashboard — Key Metrics | PG | FR-014 | P0 |
| AP-DB-003 | Dashboard — Revenue Overview | PG | FR-014 | P1 |
| AP-DB-004 | Dashboard — Platform Health | PG | FR-014 | P1 |

#### M02 — Vendor Management (VM)

| ID | Page Name | Type | FR Ref | Priority |
|---|---|---|---|---|
| AP-VM-001 | Vendor List | PG | FR-014 | P0 |
| AP-VM-002 | Vendor Detail | PG | FR-014 | P0 |
| AP-VM-003 | Vendor — Approve/Reject | MD | FR-014 | P0 |
| AP-VM-004 | Vendor — Suspend | MD | FR-014 | P1 |
| AP-VM-005 | Vendor — Performance Overview | PG | FR-014 | P2 |

#### M03 — User Management (UM)

| ID | Page Name | Type | FR Ref | Priority |
|---|---|---|---|---|
| AP-UM-001 | User List | PG | FR-014 | P0 |
| AP-UM-002 | User Detail | PG | FR-014 | P0 |
| AP-UM-003 | User — Ban/Unban | MD | FR-014 | P1 |
| AP-UM-004 | User — Activity Log | PG | FR-014 | P2 |

#### M04 — Orders Management (OM)

| ID | Page Name | Type | FR Ref | Priority |
|---|---|---|---|---|
| AP-OM-001 | Platform Order List | PG | FR-014 | P0 |
| AP-OM-002 | Platform Order Detail | PG | FR-014 | P0 |
| AP-OM-003 | Order — Admin Override | MD | FR-014 | P1 |
| AP-OM-004 | Order — Dispute Resolution | FL | FR-014 | P1 |
| AP-OM-005 | Order — Refund Processing | FL | FR-014 | P0 |

#### M05 — Delivery Management (DM)

| ID | Page Name | Type | FR Ref | Priority |
|---|---|---|---|---|
| AP-DM-001 | Delivery Providers List | PG | FR-015 | P0 |
| AP-DM-002 | Delivery Provider Detail | PG | FR-015 | P1 |
| AP-DM-003 | Delivery — Assign Provider | FL | FR-015 | P0 |
| AP-DM-004 | Delivery — Override/Reassign | MD | FR-015 | P1 |
| AP-DM-005 | Delivery — Zone Management | PG | FR-015 | P2 |

#### M06 — Wallet & Finance (WL)

| ID | Page Name | Type | FR Ref | Priority |
|---|---|---|---|---|
| AP-WL-001 | Platform Wallet Overview | PG | FR-014 | P0 |
| AP-WL-002 | Transaction Ledger | PG | FR-014 | P0 |
| AP-WL-003 | Vendor Payouts | PG | FR-014 | P1 |
| AP-WL-004 | Payout — Process | FL | FR-014 | P1 |

#### M07 — Analytics & Reports (AN)

| ID | Page Name | Type | FR Ref | Priority |
|---|---|---|---|---|
| AP-AN-001 | Platform Analytics | PG | FR-014 | P1 |
| AP-AN-002 | Sales Report | PG | FR-014 | P1 |
| AP-AN-003 | Vendor Performance Report | PG | FR-014 | P2 |
| AP-AN-004 | Customer Analytics | PG | FR-014 | P2 |

#### M08 — System Configuration (CF)

| ID | Page Name | Type | FR Ref | Priority |
|---|---|---|---|---|
| AP-CF-001 | System Settings | PG | FR-014 | P1 |
| AP-CF-002 | Platform Parameters | PG | FR-014 | P1 |
| AP-CF-003 | SMS Provider Config | PG | FR-014 | P2 |
| AP-CF-004 | Wallet Provider Config | PG | FR-014 | P2 |
| AP-CF-005 | Delivery Zones Config | PG | FR-015 | P2 |

#### M09 — Notifications (NT)

| ID | Page Name | Type | FR Ref | Priority |
|---|---|---|---|---|
| AP-NT-001 | Admin Notification Center | PG | FR-012 | P1 |
| AP-NT-002 | Broadcast Notification | FL | FR-012 | P2 |

#### M10 — Authentication (Auth)

| ID | Page Name | Type | FR Ref | Priority |
|---|---|---|---|---|
| AP-AU-001 | SMS Login — Phone Entry | PG | FR-008 | P0 |
| AP-AU-002 | SMS Login — OTP Verification | PG | FR-008 | P0 |
| AP-AU-003 | Logout Confirmation | MD | FR-008 | P1 |

---

### 5.6 Delivery Provider Panel (DP) — 12 Pages

#### M01 — Dashboard (DB)

| ID | Page Name | Type | FR Ref | Priority |
|---|---|---|---|---|
| DP-DB-001 | Delivery Dashboard | PG | FR-015 | P0 |
| DP-DB-002 | Dashboard — Today's Deliveries | PG | FR-015 | P0 |
| DP-DB-003 | Dashboard — Earnings Summary | PG | FR-015 | P1 |

#### M02 — Delivery Management (DM)

| ID | Page Name | Type | FR Ref | Priority |
|---|---|---|---|---|
| DP-DM-001 | Delivery Queue | PG | FR-015 | P0 |
| DP-DM-002 | Delivery Detail | PG | FR-015 | P0 |
| DP-DM-003 | Delivery — Accept/Reject | MD | FR-015 | P0 |
| DP-DM-004 | Delivery — Update Status | FL | FR-015 | P0 |
| DP-DM-005 | Delivery — Proof of Delivery | FL | FR-015 | P1 |
| DP-DM-006 | Delivery — Route Map | PG | FR-015 | P1 |
| DP-DM-007 | Delivery — Failed Attempt | FL | FR-015 | P1 |

#### M03 — Wallet (WL)

| ID | Page Name | Type | FR Ref | Priority |
|---|---|---|---|---|
| DP-WL-001 | Delivery Earnings | PG | FR-015 | P1 |
| DP-WL-002 | Earnings — Transaction History | PG | FR-015 | P1 |

#### M04 — Account (AC)

| ID | Page Name | Type | FR Ref | Priority |
|---|---|---|---|---|
| DP-AC-001 | Delivery Profile | PG | FR-008 | P2 |
| DP-AC-002 | Edit Profile | FL | FR-008 | P2 |

#### M05 — Notifications (NT)

| ID | Page Name | Type | FR Ref | Priority |
|---|---|---|---|---|
| DP-NT-001 | Delivery Notification Center | PG | FR-012 | P1 |

#### M06 — Authentication (Auth)

| ID | Page Name | Type | FR Ref | Priority |
|---|---|---|---|---|
| DP-AU-001 | SMS Login — Phone Entry | PG | FR-008 | P0 |
| DP-AU-002 | SMS Login — OTP Verification | PG | FR-008 | P0 |
| DP-AU-003 | Logout Confirmation | MD | FR-008 | P1 |

---

## 6. Page Counts by Portal and Module

### 6.1 By Portal

| Portal | Pages | Percentage |
|---|---|---|
| Customer Storefront (CS) | 45 | 35.4% |
| Vendor Panel (VP) | 38 | 29.9% |
| Admin Panel (AP) | 32 | 25.2% |
| Delivery Provider (DP) | 12 | 9.4% |
| **Total** | **127** | **100%** |

### 6.2 By Module

| Module | CS | VP | AP | DP | Total |
|---|---|---|---|---|---|
| Home (HM) | 6 | — | — | — | 6 |
| Products (PR) | 4 | — | — | — | 4 |
| Product Detail (PD) | 6 | — | — | — | 6 |
| Cart (CT) | 5 | — | — | — | 5 |
| Checkout (CK) | 7 | — | — | — | 7 |
| Orders (OR) | 7 | — | — | — | 7 |
| Orders Mgmt (OM) | — | 6 | 5 | — | 11 |
| Wallet (WL) | 5 | 4 | 4 | 2 | 15 |
| Account (AC) | 7 | 4 | — | 2 | 13 |
| Store (ST) | 2 | 4 | — | — | 6 |
| Dashboard (DB) | — | 4 | 4 | 3 | 11 |
| Products Mgmt (PM) | — | 10 | — | — | 10 |
| User Mgmt (UM) | — | — | 4 | — | 4 |
| Vendor Mgmt (VM) | — | — | 5 | — | 5 |
| Delivery Mgmt (DM) | — | — | 5 | 7 | 12 |
| Reviews (RV) | 2 | 2 | — | — | 4 |
| Notifications (NT) | 2 | 1 | 2 | 1 | 6 |
| Search (SR) | 3 | — | — | — | 3 |
| Categories (CA) | 2 | — | — | — | 2 |
| Support (SP) | 3 | — | — | — | 3 |
| Analytics (AN) | — | 3 | 4 | — | 7 |
| Config (CF) | — | — | 5 | — | 5 |
| Legal (LG) | 3 | — | — | — | 3 |
| Auth (Auth) | 4 | 4 | 3 | 3 | 14 |
| **Total** | **45** | **38** | **32** | **12** | **127** |

---

## 7. Module Counts

### 7.1 Building Blocks (B01–B13)

| Block | Name | Description |
|---|---|---|
| B01 | Navigation | Header, footer, sidebar, bottom nav, breadcrumbs |
| B02 | Authentication | SMS login, OTP input, session management |
| B03 | Product Display | Cards, grids, galleries, variant selectors |
| B04 | Cart & Checkout | Cart items, checkout steps, address forms |
| B05 | Order Management | Order lists, detail views, status tracking |
| B06 | Wallet & Payments | Balance display, top-up, transactions |
| B07 | Forms & Inputs | Text fields, selects, toggles, textareas |
| B08 | Data Display | Tables, charts, stats cards, timelines |
| B09 | Feedback | Toasts, alerts, modals, empty states, loading |
| B10 | Navigation Components | Tabs, pagination, steppers, breadcrumbs |
| B11 | Layout | Page shells, sidebars, grids, containers |
| B12 | Utility | Typography, icons, badges, avatars, dividers |
| B13 | Responsive Utilities | Breakpoint helpers, mobile-specific patterns |

### 7.2 Functional Modules (M01–M23)

| # | Module | Portals | Pages |
|---|---|---|---|
| M01 | Home | CS | 6 |
| M02 | Products | CS | 4 |
| M03 | Product Detail | CS | 6 |
| M04 | Cart | CS | 5 |
| M05 | Checkout | CS | 7 |
| M06 | Orders | CS | 7 |
| M07 | Wallet | CS, VP, AP, DP | 15 |
| M08 | Account | CS, VP, DP | 13 |
| M09 | Store | CS, VP | 6 |
| M10 | Dashboard | VP, AP, DP | 11 |
| M11 | Products Mgmt | VP | 10 |
| M12 | Orders Mgmt | VP, AP | 11 |
| M13 | User Mgmt | AP | 4 |
| M14 | Vendor Mgmt | AP | 5 |
| M15 | Delivery Mgmt | AP, DP | 12 |
| M16 | Reviews | CS, VP | 4 |
| M17 | Notifications | CS, VP, AP, DP | 6 |
| M18 | Search | CS | 3 |
| M19 | Categories | CS | 2 |
| M20 | Support | CS | 3 |
| M21 | Analytics | VP, AP | 7 |
| M22 | Config | AP | 5 |
| M23 | Legal | CS | 3 |

---

## 8. Priority Matrix

### 8.1 Priority Definitions

| Priority | Label | Definition | Delivery Target |
|---|---|---|---|
| **P0** | Critical | Must ship in MVP. Core user journeys. Blocks launch. | Sprint 1–3 |
| **P1** | High | Important for launch quality. Significant UX impact. | Sprint 4–6 |
| **P2** | Medium | Enhances experience. Can launch without but improves retention. | Sprint 7–9 |
| **P3** | Low | Nice to have. Future enhancement. | Post-MVP |

### 8.2 Page Counts by Priority

| Priority | CS | VP | AP | DP | Total | % |
|---|---|---|---|---|---|---|
| P0 | 18 | 14 | 12 | 7 | 51 | 40.2% |
| P1 | 15 | 13 | 12 | 4 | 44 | 34.6% |
| P2 | 12 | 11 | 8 | 1 | 32 | 25.2% |
| P3 | 0 | 0 | 0 | 0 | 0 | 0% |
| **Total** | **45** | **38** | **32** | **12** | **127** | **100%** |

### 8.3 P0 Pages by Portal

#### Customer Storefront — P0 (18 pages)
CS-HM-001, CS-HM-002, CS-PR-001, CS-PR-002, CS-PD-001, CS-PD-002, CS-CT-001, CS-CT-002, CS-CK-001, CS-CK-002, CS-CK-003, CS-CK-004, CS-CK-005, CS-CK-006, CS-OR-001, CS-OR-002, CS-OR-003, CS-WL-001, CS-WL-002, CS-WL-003, CS-AC-001, CS-AC-002, CS-AC-003, CS-AC-004, CS-SR-001, CS-AU-001, CS-AU-002

#### Vendor Panel — P0 (14 pages)
VP-DB-001, VP-DB-002, VP-PM-001, VP-PM-002, VP-PM-003, VP-PM-004, VP-PM-007, VP-OM-001, VP-OM-002, VP-OM-003, VP-OM-004, VP-WL-001, VP-WL-002, VP-WL-003, VP-AU-001, VP-AU-002, VP-AU-003

#### Admin Panel — P0 (12 pages)
AP-DB-001, AP-DB-002, AP-VM-001, AP-VM-002, AP-VM-003, AP-UM-001, AP-UM-002, AP-OM-001, AP-OM-002, AP-OM-005, AP-DM-001, AP-DM-003, AP-WL-001, AP-WL-002, AP-AU-001, AP-AU-002

#### Delivery Provider — P0 (7 pages)
DP-DB-001, DP-DB-002, DP-DM-001, DP-DM-002, DP-DM-003, DP-DM-004, DP-AU-001, DP-AU-002

---

## 9. Deliverables

### 9.1 Specification Documents

| # | Document | File | Status |
|---|---|---|---|
| D01 | UI/UX Master Plan | `01-UI-UX-MASTER-PLAN.md` | This document |
| D02 | Design System Specification | `02-DESIGN-SYSTEM.md` | Pending |
| D03 | Customer Storefront Pages | `03-CUSTOMER-STOREFRONT.md` | Pending |
| D04 | Vendor Panel Pages | `04-VENDOR-PANEL.md` | Pending |
| D05 | Admin Panel Pages | `05-ADMIN-PANEL.md` | Pending |
| D06 | Delivery Provider Pages | `06-DELIVERY-PROVIDER.md` | Pending |
| D07 | Component Library Spec | `07-COMPONENT-LIBRARY.md` | Pending |
| D08 | Interaction Patterns | `08-INTERACTION-PATTERNS.md` | Pending |
| D09 | Responsive Behavior Guide | `09-RESPONSIVE-GUIDE.md` | Pending |
| D10 | Accessibility Checklist | `10-ACCESSIBILITY.md` | Pending |
| D11 | RTL/LTR Implementation Guide | `11-RTL-LTR-GUIDE.md` | Pending |
| D12 | Test Point Registry | `12-TEST-POINTS.md` | Pending |
| D13 | Order State Machine | `13-ORDER-STATE-MACHINE.md` | Pending |
| D14 | Traceability Matrix | `14-TRACEABILITY-MATRIX.md` | Pending |
| D15 | Platform Constraints Reference | `15-PLATFORM-CONSTRAINTS.md` | Pending |

### 9.2 Design Artifacts

| # | Artifact | Format | Status |
|---|---|---|---|
| A01 | Wireframes — All P0 Pages | Figma / SVG | Pending |
| A02 | Wireframes — All P1 Pages | Figma / SVG | Pending |
| A03 | Wireframes — All P2 Pages | Figma / SVG | Pending |
| A04 | Design Token File | JSON / CSS | Pending |
| A05 | Component Library (Storybook) | React | Pending |
| A06 | Icon Set | SVG Sprite | Pending |
| A07 | Typography Scale | CSS / Tailwind Config | Pending |
| A08 | Color Palette | CSS / Tailwind Config | Pending |
| A09 | Spacing & Grid System | CSS / Tailwind Config | Pending |
| A10 | Motion & Animation Specs | CSS / Framer Motion | Pending |

### 9.3 Test Artifacts

| # | Artifact | Description | Status |
|---|---|---|---|
| T01 | Test Point Registry | All 974 test points with mappings | Pending |
| T02 | Accessibility Audit Report | WCAG 2.1 AA compliance | Pending |
| T03 | RTL/LTR Mirror Test Suite | Bidirectional layout validation | Pending |
| T04 | Responsive Test Matrix | All breakpoints × all pages | Pending |
| T05 | Cross-Browser Test Results | Chrome, Firefox, Safari, Edge | Pending |
| T06 | Performance Budget Report | Core Web Vitals compliance | Pending |

---

## 10. Cross-Portal Consistency Rules

### 10.1 Navigation

| Rule | Description |
|---|---|
| CR-NAV-01 | All portals share identical navigation component structure (header, sidebar, footer). |
| CR-NAV-02 | Active state styling is consistent across all portals. |
| CR-NAV-03 | Breadcrumb trail follows `{Portal} > {Section} > {Page}` pattern. |
| CR-NAV-04 | Mobile bottom navigation appears on all portals at `sm` breakpoint and below. |
| CR-NAV-05 | Sidebar collapse/expand behavior is identical across VP, AP, DP. |

### 10.2 Typography

| Rule | Description |
|---|---|
| CR-TYP-01 | All portals use the same type scale (12px, 14px, 16px, 18px, 20px, 24px, 30px, 36px, 48px). |
| CR-TYP-02 | Arabic uses Noto Kufi Arabic / IBM Plex Arabic; English uses Inter. |
| CR-TYP-03 | Heading hierarchy is consistent: H1 = page title, H2 = section, H3 = subsection. |

### 10.3 Color

| Rule | Description |
|---|---|
| CR-CLR-01 | Primary brand color is identical across all portals. |
| CR-CLR-02 | Status colors (success, warning, error, info) are mapped to identical tokens. |
| CR-CLR-03 | Dark mode is not in scope for MVP (single theme only). |

### 10.4 Spacing

| Rule | Description |
|---|---|
| CR-SPC-01 | All spacing uses the 4px base grid. |
| CR-SPC-02 | Page padding: `16px` mobile, `24px` tablet, `32px` desktop. |
| CR-SPC-03 | Card padding: `12px` mobile, `16px` tablet/desktop. |
| CR-SPC-04 | Section gaps: `24px` vertical between major sections. |

### 10.5 Forms

| Rule | Description |
|---|---|
| CR-FRM-01 | All text inputs share identical height (40px desktop, 48px mobile). |
| CR-FRM-02 | Error states display below the field, red text, with icon. |
| CR-FRM-03 | Required fields marked with asterisk `*` in Arabic: `مطلوب`. |
| CR-FRM-04 | Form validation is inline (on blur), not on submit only. |

### 10.6 Data Tables

| Rule | Description |
|---|---|
| CR-TBL-01 | Tables are horizontally scrollable on mobile. |
| CR-TBL-02 | Row height: `48px` default, `56px` with actions. |
| CR-TBL-03 | Pagination shows: `{Start}–{End} of {Total}` in Arabic format. |

### 10.7 Modals & Dialogs

| Rule | Description |
|---|---|
| CR-MDL-01 | Modals max-width: `480px` on mobile, `560px` on tablet, `640px` on desktop. |
| CR-MDL-02 | Destructive actions always require confirmation. |
| CR-MDL-03 | Modal close button is top-left in RTL, top-right in LTR. |

### 10.8 Authentication

| Rule | Description |
|---|---|
| CR-AUTH-01 | All portals use identical SMS OTP login flow. |
| CR-AUTH-02 | OTP input is 6-digit, auto-advance, auto-submit on fill. |
| CR-AUTH-03 | OTP expires after 5 minutes; resend available after 60 seconds. |
| CR-AUTH-04 | Session timeout: 30 minutes of inactivity for VP/AP; 24 hours for CS/DP. |

---

## 11. Design Principles

### 11.1 Arabic-First

The platform is designed for Arabic readers. All UI copy, labels, placeholders, and error messages are written in Arabic first. English is a secondary translation. Design layouts assume RTL reading patterns:

- Primary actions on the **right** side
- Navigation flows **right-to-left**
- Numeric displays (prices, counts) use **Arabic-Indic numerals** (`٠١٢٣٤٥٦٧٨٩`)
- Date formats follow **Hijri** calendar with Gregorian fallback
- Text alignment defaults to **right-aligned**

### 11.2 Mobile-First

Every page is designed for a 360px mobile viewport first, then progressively enhanced for larger screens:

- Touch targets minimum **44px × 44px**
- Thumb-friendly navigation zones
- Bottom sheet patterns for secondary actions
- Swipe gestures for key interactions (cart item removal, order status)
- Offline-aware patterns for flaky mobile networks

### 11.3 Accessible

WCAG 2.1 Level AA compliance is a hard requirement:

- Semantic HTML throughout
- ARIA labels for all interactive elements
- Keyboard navigation for all workflows
- Screen reader support (Arabic and English)
- Minimum contrast ratio 4.5:1 (normal text), 3:1 (large text)
- Focus indicators visible on all interactive elements
- No color-only information conveyance

### 11.4 Consistent

Cross-portal consistency ensures users transitioning between portals (e.g., Customer → Vendor) encounter familiar patterns:

- Shared component library (B01–B13)
- Identical navigation structure
- Consistent status color mapping
- Unified form patterns
- Same authentication flow

### 11.5 Performant

UI performance budgets:

| Metric | Target |
|---|---|
| First Contentful Paint (FCP) | < 1.5s |
| Largest Contentful Paint (LCP) | < 2.5s |
| Cumulative Layout Shift (CLS) | < 0.1 |
| First Input Delay (FID) | < 100ms |
| Time to Interactive (TTI) | < 3.5s |
| Total page weight | < 500KB (initial load) |
| JavaScript bundle | < 200KB (compressed) |

---

## 12. Responsive Strategy

### 12.1 Approach

**Mobile-first, progressive enhancement.** All pages are built for 360px width first, then enhanced at each breakpoint.

### 12.2 Breakpoint Behavior

| Element | Mobile (≤479) | Tablet (480–1023) | Desktop (≥1024) |
|---|---|---|---|
| Navigation | Bottom tab bar | Side rail (collapsed) | Side rail (expanded) |
| Sidebar | Hidden (drawer) | Collapsible | Persistent |
| Product Grid | 1 column | 2 columns | 3–4 columns |
| Cart | Full screen | Side panel | Side panel |
| Checkout | Single column | 2 column | 3 column |
| Tables | Card view | Scrollable table | Full table |
| Modals | Full screen | Centered | Centered |
| Forms | Full width | Constrained width | Constrained width |

### 12.3 Mobile-Specific Patterns

| Pattern | Usage |
|---|---|
| Bottom Sheet | Filters, sort options, quick actions |
| Swipe Actions | Cart items, order list items |
| Pull-to-Refresh | Order list, product listing |
| Sticky CTA | "Add to Cart" / "Checkout" buttons |
| Collapsible Sections | FAQ, product specs, order details |
| Floating Action Button | Primary action (e.g., "Add Product" in VP) |

### 12.4 Tablet-Specific Patterns

| Pattern | Usage |
|---|---|
| Split View | Order list + order detail (VP/AP) |
| Dual Pane | Product grid + filter sidebar |
| Expanded Sidebar | Full navigation rail |

---

## 13. Accessibility Strategy (WCAG 2.1 AA)

### 13.1 Compliance Requirements

| WCAG Criterion | Requirement | Implementation |
|---|---|---|
| 1.1.1 Non-text Content | Alt text for all images | `alt` attributes, decorative images `alt=""` |
| 1.3.1 Info and Relationships | Semantic HTML | `<nav>`, `<main>`, `<article>`, `<aside>`, `<section>` |
| 1.3.4 Orientation | Support both orientations | No lock on orientation |
| 1.4.1 Use of Color | Not sole means | Icons + color, text + color |
| 1.4.3 Contrast Minimum | 4.5:1 normal, 3:1 large | Automated checks in CI |
| 1.4.4 Resize Text | Up to 200% zoom | Responsive typography |
| 1.4.10 Reflow | No horizontal scroll at 320px | Responsive layouts |
| 2.1.1 Keyboard | All functionality via keyboard | Tab order, arrow keys, Enter/Space |
| 2.4.1 Bypass Blocks | Skip navigation links | Skip to main content |
| 2.4.3 Focus Order | Logical tab order | DOM order matches visual order |
| 2.4.6 Headings and Labels | Descriptive headings | Heading hierarchy enforced |
| 2.4.7 Focus Visible | Visible focus ring | 2px focus ring, high contrast |
| 2.5.5 Target Size | 44px minimum | Touch target sizing |
| 3.1.1 Language of Page | `lang` attribute | `lang="ar"` default, toggle to `lang="en"` |
| 3.3.1 Error Identification | Clear error messages | Inline errors with icons |
| 3.3.2 Labels or Instructions | All inputs labeled | `<label>` + `aria-label` |
| 4.1.2 Name, Role, Value | ARIA for custom components | `role`, `aria-*` attributes |

### 13.2 Testing Strategy

| Phase | Tool/Method | Frequency |
|---|---|---|
| Development | ESLint a11y plugin | Every commit |
| Design | Axe DevTools | Every page design |
| QA | Manual screen reader test | Every sprint |
| QA | Keyboard-only navigation test | Every sprint |
| CI/CD | Lighthouse CI | Every PR |
| Release | Third-party audit | Every major release |

### 13.3 Arabic Accessibility Considerations

- Screen readers pronounce Arabic text correctly when `lang="ar"` is set
- RTL reading order must be verified with VoiceOver (iOS) and TalkBack (Android)
- Arabic numerals (٠١٢٣) must be readable by screen readers (provide `aria-label` with Western numerals as fallback)
- Date formats must be announced correctly in Arabic

---

## 14. RTL/LTR Strategy

### 14.1 Implementation Approach

YemenMart uses **bidirectional CSS** with logical properties:

```css
/* Instead of margin-left / margin-right */
margin-inline-start: 16px;
margin-inline-end: 16px;

/* Instead of padding-left / padding-right */
padding-inline-start: 16px;
padding-inline-end: 16px;

/* Instead of text-align: left / right */
text-align: start;

/* Instead of border-left / border-right */
border-inline-start: 1px solid;
border-inline-end: 1px solid;
```

### 14.2 Mirror Rules

| Element | RTL Layout | LTR Layout |
|---|---|---|
| Navigation | Flows right-to-left | Flows left-to-right |
| Sidebar | Right side | Left side |
| Back button | Right side | Left side |
| Progress bar | Fills right-to-left | Fills left-to-right |
| Stepper | Steps flow right-to-left | Steps flow left-to-right |
| Carousels | Scroll right-to-left | Scroll left-to-right |
| Icons with direction | Mirrored (arrows, chevrons) | Original |
| Icons without direction | Not mirrored | Not mirrored |

### 14.3 Icons That Mirror

| Icon | Mirror in RTL |
|---|---|
| Arrow / Chevron | ✅ Yes |
| Back / Forward | ✅ Yes |
| Sort ascending/descending | ✅ Yes |
| Progress indicators | ✅ Yes |
| Step indicators | ✅ Yes |
| Play / Pause | ❌ No |
| Checkmark | ❌ No |
| Close (X) | ❌ No |
| Star / Heart | ❌ No |

### 14.4 Language Toggle

- Toggle is accessible from header on all portals
- Toggle persists preference in `localStorage` and user profile
- Page reloads with new direction on toggle
- URL does not change (single-page app)
- All content re-renders in selected language
- Animations during transition are optional (not required)

---

## 15. Internationalization Strategy

### 15.1 Supported Languages

| Language | Code | Direction | Priority | Status |
|---|---|---|---|---|
| Arabic | `ar` | RTL | Primary | MVP |
| English | `en` | LTR | Secondary | MVP |

### 15.2 Translation Scope

| Content Type | Arabic | English | Notes |
|---|---|---|---|
| UI Labels | ✅ | ✅ | All interactive elements |
| Error Messages | ✅ | ✅ | User-facing errors |
| Placeholder Text | ✅ | ✅ | Input placeholders |
| Tooltips | ✅ | ✅ | Hover/focus tooltips |
| Empty States | ✅ | ✅ | No data / no results |
| System Notifications | ✅ | ✅ | Push / in-app notifications |
| Email Templates | ❌ | ❌ | Out of scope (SMS only) |
| Legal Documents | ✅ | ✅ | Terms, Privacy, Return Policy |
| Product Content | ❌ | ❌ | Vendor-provided, not translated |

### 15.3 Number Formatting

| Format | Arabic | English |
|---|---|---|
| Numbers | ٠١٢٣٤٥٦٧٨٩ | 0123456789 |
| Currency | ١٬٥٠٠ ر.ي | YER 1,500 |
| Percentages | ١٥٪ | 15% |
| Dates (Hijri) | ١٥ محرم ١٤٤٨ | 15 Muharram 1448 |
| Dates (Gregorian) | ١٣ سبتمبر ٢٠٢٦ | September 13, 2026 |
| Time | ٠٢:٣٠ م | 2:30 PM |

### 15.4 Translation Management

- All strings stored in JSON resource files (`/locales/ar.json`, `/locales/en.json`)
- Organized by module: `common.*`, `home.*`, `products.*`, `cart.*`, etc.
- Interpolation: `{{variable}}` syntax
- Pluralization: `count` key with `_one`, `_other` forms
- No hardcoded strings in components
- Translation key coverage tracked in CI

---

## 16. Traceability Strategy

### 16.1 Traceability Chain

Every UI element traces back through:

```
Page → Module → Building Block → Functional Requirement → Test Point
```

### 16.2 ID Mapping

| Layer | ID Format | Example |
|---|---|---|
| Page | `{Portal}-{Module}-{Type}-{Seq}` | CS-HM-001 |
| Module | `M{01-23}` | M01 |
| Building Block | `B{01-13}` | B01 |
| Functional Requirement | `FR-{001-017}` | FR-001 |
| Test Point | `TP-{001-974}` | TP-001 |
| Platform Constraint | `PC-{001-026}` | PC-001 |

### 16.3 Traceability Matrix Structure

| Page ID | Module | Building Blocks | FR | Test Points | Constraints |
|---|---|---|---|---|---|
| CS-HM-001 | M01 | B01, B03, B11, B13 | FR-001 | TP-001–TP-012 | PC-001, PC-005 |

### 16.4 Traceability Queries

The traceability matrix supports the following queries:

1. **Page → Requirements:** What FRs does this page implement?
2. **Requirement → Pages:** Which pages implement this FR?
3. **Module → Test Points:** What test points cover this module?
4. **Test Point → Page:** Which page does this test point validate?
5. **Constraint → Pages:** Which pages are affected by this constraint?
6. **Gap Analysis:** Are there FRs with no page coverage?
7. **Coverage Report:** What % of FRs have full page + test coverage?

### 16.5 Coverage Targets

| Metric | Target |
|---|---|
| FR → Page coverage | 100% |
| Page → Test Point coverage | 100% |
| FR → Test Point coverage | 100% |
| Module → Building Block coverage | 100% |
| Constraint → Page coverage | 100% |

---

## 17. Review Gates

### 17.1 Gate Definitions

| Gate | Name | Trigger | Approver | Criteria |
|---|---|---|---|---|
| G01 | Spec Completeness | All pages specified | Tech Lead | 127 pages documented, 0 gaps |
| G02 | Design System | Token + component spec done | Design Lead | All 13 building blocks defined |
| G03 | RTL Validation | Mirror test pass | QA Lead | All pages render correctly in RTL |
| G04 | Accessibility Audit | WCAG check done | Accessibility Lead | 0 P1/P2 violations |
| G05 | Responsive Validation | All breakpoints tested | QA Lead | All pages functional at all breakpoints |
| G06 | Cross-Portal Consistency | Consistency audit done | Tech Lead | All 34 consistency rules verified |
| G07 | Traceability Complete | Matrix filled | Tech Lead | 100% coverage on all layers |
| G08 | Test Points Defined | All 974 TPs registered | QA Lead | Each TP has page, FR, and expected result |
| G09 | Stakeholder Review | Demo walkthrough | Product Owner | Sign-off on all portals |
| G10 | Final Sign-Off | All gates passed | Engineering Manager | All 9 gates green |

### 17.2 Gate Sequence

```
G01 (Spec) → G02 (Design System) → G03 (RTL) → G04 (A11y) → G05 (Responsive)
                                                                       ↓
G10 (Sign-Off) ← G09 (Stakeholder) ← G08 (Test Points) ← G07 (Traceability) ← G06 (Consistency)
```

### 17.3 Gate Exit Criteria

Each gate requires:
1. All checklist items completed
2. No outstanding P0/P1 issues
3. Documentation updated
4. Review approved by designated approver
5. Gate status recorded in project tracker

---

## 18. Completion Criteria

### 18.1 Specification Completion

| Criteria | Target | Measurement |
|---|---|---|
| All 127 pages specified | 127/127 | Page inventory complete |
| All 23 modules documented | 23/23 | Module spec files exist |
| All 13 building blocks defined | 13/13 | Component specs complete |
| All 17 FRs mapped to pages | 17/17 | Traceability matrix |
| All 974 test points defined | 974/974 | Test point registry |
| All 26 constraints documented | 26/26 | Constraints reference |
| All 17 order states visualized | 17/17 | State machine diagram |

### 18.2 Design System Completion

| Criteria | Target | Measurement |
|---|---|---|
| Design tokens defined | 100% | Token file complete |
| Component library (Storybook) | 100% | All B01–B13 components |
| Icon set | 100% | All required icons |
| Typography scale | 100% | Arabic + English |
| Color palette | 100% | All status/semantic colors |
| Spacing system | 100% | 4px grid |

### 18.3 Quality Completion

| Criteria | Target | Measurement |
|---|---|---|
| WCAG 2.1 AA compliance | 100% | Accessibility audit |
| RTL/LTR mirror correctness | 100% | Mirror test suite |
| Responsive functionality | 100% | All breakpoints × all pages |
| Cross-browser compatibility | 100% | Chrome, Firefox, Safari, Edge |
| Performance budgets met | 100% | Lighthouse CI |
| Cross-portal consistency | 100% | 34 rules verified |
| Translation completeness | 100% | ar.json + en.json coverage |

### 18.4 Process Completion

| Criteria | Target | Measurement |
|---|---|---|
| All 10 gates passed | 10/10 | Gate register |
| Stakeholder sign-off | Obtained | Written approval |
| No open P0 issues | 0 | Issue tracker |
| No open P1 issues | 0 | Issue tracker |
| All review comments addressed | 100% | Review log |

### 18.5 Definition of Done

A page is considered **done** when:

1. ✅ Full specification written (layout, components, interactions, states)
2. ✅ Wireframe created (at least low-fidelity)
3. ✅ RTL layout validated
4. ✅ Responsive behavior documented for all breakpoints
5. ✅ Accessibility requirements listed
6. ✅ FR traceability established
7. ✅ Test points defined
8. ✅ Building block dependencies identified
9. ✅ Edge cases and error states documented
10. ✅ Cross-portal consistency rules checked

---

## Appendix A: Functional Requirements Reference

| ID | Requirement | Description | Pages |
|---|---|---|---|
| FR-001 | Browse Homepage | Customer views homepage with featured content, categories, deals | CS-HM-001–006 |
| FR-002 | Browse Products | Customer browses product listings with filters and sort | CS-PR-001–004, CS-CA-001–002 |
| FR-003 | View Product Detail | Customer views full product information | CS-PD-001–006 |
| FR-004 | Manage Cart | Customer adds/removes/updates cart items | CS-CT-001–005 |
| FR-005 | Checkout & Pay | Customer selects address, pays with wallet, confirms order | CS-CK-001–007 |
| FR-006 | Track Orders | Customer views order history, tracks active orders | CS-OR-001–007 |
| FR-007 | Manage Wallet | Customer tops up wallet, views transactions | CS-WL-001–005, VP-WL-001–004, AP-WL-001–004, DP-WL-001–002 |
| FR-008 | Manage Account | User manages profile, addresses, phone, settings | CS-AC-001–007, VP-AC-001–004, DP-AC-001–002, all Auth pages |
| FR-009 | Vendor Operations | Vendor manages products, orders, store, analytics | VP-DB-001–004, VP-PM-001–010, VP-OM-001–006, VP-ST-001–004, VP-AN-001–003 |
| FR-010 | Search | Customer searches products | CS-SR-001–003 |
| FR-011 | Reviews & Ratings | Customer/vendor manages reviews | CS-RV-001–002, VP-RV-001–002 |
| FR-012 | Notifications | All actors receive and manage notifications | CS-NT-001–002, VP-NT-001, AP-NT-001–002, DP-NT-001 |
| FR-013 | Customer Support | Customer accesses help center and FAQ | CS-SP-001–003 |
| FR-014 | Admin Operations | Admin manages vendors, users, orders, finance, config | AP-DB-001–004, AP-VM-001–005, AP-UM-001–004, AP-OM-001–005, AP-WL-001–004, AP-AN-001–004, AP-CF-001–005 |
| FR-015 | Delivery Operations | Delivery provider manages deliveries | AP-DM-001–005, DP-DB-001–003, DP-DM-001–007 |
| FR-016 | Platform Configuration | Admin configures platform settings | AP-CF-001–005 |
| FR-017 | Legal & Compliance | Legal pages, policies, terms | CS-LG-001–003 |

## Appendix B: Platform Constraints (26 Non-Negotiable)

| ID | Constraint | Category |
|---|---|---|
| PC-001 | Arabic is the primary language | Language |
| PC-002 | RTL is the default layout direction | Layout |
| PC-003 | English is secondary, toggle-available | Language |
| PC-004 | SMS OTP is the only authentication method | Auth |
| PC-005 | No email-based login or registration | Auth |
| PC-006 | Wallet is the only payment method | Payment |
| PC-007 | No credit/debit card payments | Payment |
| PC-008 | No Buy Now Pay Later (BNPL) | Payment |
| PC-009 | Mobile-first responsive design | Design |
| PC-010 | 44px minimum touch targets | Accessibility |
| PC-011 | WCAG 2.1 Level AA compliance | Accessibility |
| PC-012 | 17 order states must be represented | Business |
| PC-013 | 5 actor roles with distinct permissions | Business |
| PC-014 | Vendor approval required before activation | Business |
| PC-015 | Delivery assignment is system-driven | Business |
| PC-016 | Wallet balance cannot go negative | Payment |
| PC-017 | Prices displayed in YER (Yemeni Rial) | Business |
| PC-018 | Hijri calendar primary, Gregorian secondary | Business |
| PC-019 | Arabic-Indic numerals as default display | Design |
| PC-020 | No dark mode in MVP | Design |
| PC-021 | Tailwind CSS 4 for all styling | Tech |
| PC-022 | Next.js 15 App Router for Customer Storefront | Tech |
| PC-023 | React 18 + Vite for VP, AP, DP | Tech |
| PC-024 | React Native for mobile apps | Tech |
| PC-025 | PostgreSQL 15 as primary database | Tech |
| PC-026 | MinIO for object storage (images, documents) | Tech |

## Appendix C: Order States Reference

| # | State | Arabic | Actor Triggered | Description |
|---|---|---|---|---|
| 1 | PENDING | قيد الانتظار | System | Order created, awaiting payment |
| 2 | PAID | مدفوع | System | Payment confirmed via wallet |
| 3 | CONFIRMED | مؤكد | Vendor | Vendor accepts the order |
| 4 | PROCESSING | قيد المعالجة | Vendor | Vendor is preparing the order |
| 5 | READY_FOR_PICKUP | جاهز للاستلام | Vendor | Order ready for delivery pickup |
| 6 | ASSIGNED | مُعيّن | System | Delivery provider assigned |
| 7 | PICKED_UP | تم الاستلام | Delivery | Delivery provider picked up order |
| 8 | IN_TRANSIT | في الطريق | Delivery | Order in transit to customer |
| 9 | OUT_FOR_DELIVERY | خارج للتوصيل | Delivery | Order out for final delivery |
| 10 | DELIVERED | تم التوصيل | Delivery | Order delivered to customer |
| 11 | COMPLETED | مكتمل | System | Delivery confirmed, order closed |
| 12 | RETURN_REQUESTED | طلب إرجاع | Customer | Customer requests return |
| 13 | RETURN_APPROVED | إرجاع مقبول | Admin/Vendor | Return approved |
| 14 | RETURN_IN_TRANSIT | إرجاع في الطريق | Delivery | Return item in transit |
| 15 | RETURN_RECEIVED | تم استلام الإرجاع | Vendor | Vendor receives returned item |
| 16 | REFUNDED | تم استرداد المبلغ | System | Wallet refund processed |
| 17 | CANCELLED | ملغي | Customer/Vendor/Admin | Order cancelled |

---

**End of Master Plan**

*Document version 1.0.0 — YemenMart UI/UX Specification Suite*
