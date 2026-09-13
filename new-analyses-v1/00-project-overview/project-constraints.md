# YemenMart Project Constraints

## Overview

These 26 constraints are **non-negotiable**. They represent architectural decisions, business rules, and compliance requirements that must be honored in every module and use case.

---

## Payment Constraints

### Constraint 1: Wallet-Only Payments
**Category:** Financial | **Priority:** Critical

All transactions must use the platform's built-in wallet system. The following payment methods are **explicitly prohibited**:
- Credit/debit cards
- Buy Now Pay Later (BNPL)
- Installment plans
- Bank transfers (direct)
- Cash payments

**Enforcement:** Payment gateway rejects all non-wallet payment types at API level.

### Constraint 2: No Cash on Delivery (COD)
**Category:** Financial | **Priority:** Critical

Orders cannot be paid upon delivery. All payments must be processed through the wallet before order confirmation. This eliminates cash handling risks and simplifies accounting.

**Enforcement:** Order creation requires wallet balance verification; COD option removed from checkout UI.

---

## Authentication Constraints

### Constraint 3: Phone-First Authentication
**Category:** Identity | **Priority:** Critical

Authentication is phone-number-based using SMS or WhatsApp OTP. The phone number serves as the primary identifier for all user types (customers, merchants, admins, delivery agents).

**Enforcement:** Registration flow requires phone number verification before account activation. OTP delivered via SMS/WhatsApp with 5-minute expiry.

### Constraint 4: Email Optional
**Category:** Identity | **Priority:** High

Email addresses are optional. If a user provides an email, it must be verified via confirmation link before being used for notifications or account recovery.

**Enforcement:** Email verification flag required before email-based notifications are sent. Profile accepts unverified emails without restriction.

### Constraint 5: Forgot Password via Phone OTP
**Category:** Identity | **Priority:** Critical

Password recovery is exclusively through phone OTP. No email-based password reset, no security questions, no backup codes.

**Enforcement:** Password reset flow generates OTP sent to registered phone number. No alternative recovery paths.

---

## Build Constraints

### Constraint 6: 100% Custom Build
**Category:** Architecture | **Priority:** Critical

The platform must be built from scratch. The following are **prohibited**:
- Medusa.js or any e-commerce framework
- Pre-built marketplace templates
- White-label solutions

**Enforcement:** Code review gates; dependency scanning rejects e-commerce frameworks.

### Constraint 7: Arabic-First Bilingual (RTL)
**Category:** UX | **Priority:** Critical

Arabic is the primary language with full RTL layout support. English is the secondary language. All content, UI elements, error messages, and documentation must be available in both languages.

**Enforcement:** i18n framework enforces dual-language content; RTL CSS applied by default.

---

## Order Management Constraints

### Constraint 8: 17 Order States
**Category:** Orders | **Priority:** Critical

The order lifecycle consists of exactly 17 states. No additional states may be added. The states are:

1. `PENDING` - Order created, awaiting payment
2. `PAID` - Payment confirmed
3. `CONFIRMED` - Merchant acknowledged
4. `PROCESSING` - Merchant preparing order
5. `READY_FOR_PICKUP` - Package ready
6. `ASSIGNED` - Delivery agent assigned
7. `PICKED_UP` - Agent collected package
8. `IN_TRANSIT` - Package en route
9. `OUT_FOR_DELIVERY` - Agent near destination
10. `DELIVERED` - Package delivered
11. `COMPLETED` - Customer confirmed receipt
12. `RETURN_REQUESTED` - Customer initiated return
13. `RETURN_APPROVED` - Merchant approved return
14. `RETURN_IN_TRANSIT` - Return package in transit
15. `RETURN_RECEIVED` - Merchant received return
16. `REFUNDED` - Wallet credited back
17. `CANCELLED` - Order cancelled

**Enforcement:** State machine enforces valid transitions only; new states rejected at design review.

### Constraint 9: Master/Sub-Order Architecture
**Category:** Orders | **Priority:** Critical

Each customer order (master order) can contain items from multiple merchants. Each merchant's items form a sub-order with independent lifecycle tracking.

**Enforcement:** Order creation splits items by merchant; sub-order ID links to master order ID.

---

## Delivery Constraints

### Constraint 10: No GPS Tracking
**Category:** Delivery | **Priority:** High

Real-time GPS tracking of delivery agents is **prohibited**. Delivery status is communicated through delivery codes only.

**Enforcement:** No location APIs called during delivery; status updates require delivery code verification.

### Constraint 11: 3-Attempt Delivery Code Lockout
**Category:** Delivery | **Priority:** Critical

Customers have exactly 3 attempts to enter the correct delivery code. After 3 failed attempts:
- Account is locked for 24 hours
- Delivery agent is notified
- Support ticket is auto-generated

**Enforcement:** Delivery code input tracks attempts; lockout enforced at service layer.

---

## Financial Constraints

### Constraint 12: 7-Day Escrow Hold
**Category:** Payments | **Priority:** Critical

Customer payments are held in escrow for 7 days after delivery confirmation. Funds are released to merchant wallets after the hold period, minus platform fees.

**Enforcement:** Escrow engine holds funds; release triggered by timer after delivery confirmation.

### Constraint 13: Merchant Return Policies
**Category:** Returns | **Priority:** Critical

Each merchant defines their own return policy:
- `isReturnable` (boolean): Whether the product can be returned
- `returnPeriodDays` (integer): Number of days after delivery for return eligibility

**Enforcement:** Return requests validated against merchant's policy before approval.

### Constraint 19: KYC Mandatory for Vendors
**Category:** Compliance | **Priority:** Critical

All vendors must complete Know Your Customer (KYC) verification before:
- Listing products
- Receiving payments
- Accessing analytics

**Enforcement:** KYC status checked at merchant onboarding; incomplete KYC blocks all merchant actions.

---

## Catalog Constraints

### Constraint 14: 10+ Store Templates
**Category:** Storefront | **Priority:** High

The platform must provide a minimum of 10 customizable store templates for merchants to choose from.

**Enforcement:** Template library validated at deployment; minimum count enforced.

### Constraint 15: 40+ System Service Categories
**Category:** Catalog | **Priority:** High

The system must define a minimum of 40 service categories covering product types available in the marketplace.

**Enforcement:** Category count validated during catalog initialization; new categories require admin approval.

### Constraint 16: No Trial Products
**Category:** Catalog | **Priority:** High

Products cannot be offered as free trials, samples, or test items. All listings must have a price and represent real inventory.

**Enforcement:** Product creation rejects zero-price items unless marked as "service" type.

### Constraint 17: No Subscription Products
**Category:** Catalog | **Priority:** High

Recurring subscription products are not supported. All products are one-time purchases.

**Enforcement:** Product type enum excludes subscription; recurring billing logic not implemented.

### Constraint 18: No Third-Party Branding
**Category:** Catalog | **Priority:** Medium

Merchants cannot use third-party brand names, logos, or trademarks in their store names, product titles, or descriptions without authorization.

**Enforcement:** AI-powered content moderation scans for brand names; admin review queue for flagged content.

---

## Compliance Constraints

### Constraint 20: 15% VAT on Discounted Price
**Category:** Tax | **Priority:** Critical**

VAT is calculated at 15% on the **discounted price** (after any promotions or discounts), not the original price.

**Enforcement:** Tax engine applies 15% to final sale price; invoice generation reflects discounted amount.

### Constraint 21: ZATCA-Compliant Invoices
**Category:** Tax | **Priority:** Critical**

All invoices must comply with Saudi Arabia's ZATCA (Zakat, Tax and Customs Authority) requirements:
- Unique invoice numbering
- Timestamp and seller/buyer info
- Itemized tax breakdown
- QR code for verification

**Enforcement:** Invoice generation engine produces ZATCA-compliant XML/JSON; validation before issuance.

### Constraint 22: Double-Entry Bookkeeping
**Category:** Finance | **Priority:** Critical**

All financial transactions follow double-entry bookkeeping principles. Every debit has a corresponding credit.

**Enforcement:** Ledger engine enforces balanced entries; imbalanced transactions rejected.

### Constraint 23: 5-Year Invoice Retention
**Category:** Compliance | **Priority:** Critical**

All invoices must be retained for a minimum of 5 years in an immutable, searchable archive.

**Enforcement:** Archive service retains invoices; deletion blocked for 5-year window; audit trail maintained.

---

## Cart Constraints

### Constraint 24: Max 50 Cart Items
**Category:** Cart | **Priority:** Medium**

A single cart can contain a maximum of 50 items (across all products and merchants).

**Enforcement:** Cart service rejects additions beyond 50 items; UI displays remaining capacity.

### Constraint 25: Max 10 Units Per Product in Cart
**Category:** Cart | **Priority:** Medium**

For any single product in the cart, the maximum quantity is 10 units.

**Enforcement:** Quantity adjustment capped at 10 per product; bulk-add operations validated.

---

## Order Value Constraints

### Constraint 26: 500–5,000,000 YER Order Range
**Category:** Orders | **Priority:** Critical**

Minimum order value: 500 YER (Yemeni Rial)
Maximum order value: 5,000,000 YER (Yemeni Rial)

Orders outside this range are rejected at checkout.

**Enforcement:** Cart total validated before order creation; error message for out-of-range totals.

---

## Constraint Summary Matrix

| # | Constraint | Category | Priority | Module(s) |
|---|-----------|----------|----------|-----------|
| 1 | Wallet-only payments | Financial | Critical | M15, M16 |
| 2 | No COD | Financial | Critical | M12, M15 |
| 3 | Phone-first auth | Identity | Critical | M01, M02 |
| 4 | Email optional | Identity | High | M01, M02 |
| 5 | Forgot password via phone | Identity | Critical | M01 |
| 6 | 100% custom build | Architecture | Critical | All |
| 7 | Arabic-first bilingual | UX | Critical | All |
| 8 | 17 order states | Orders | Critical | M13, M14 |
| 9 | Master/sub-order | Orders | Critical | M13, M14 |
| 10 | No GPS tracking | Delivery | Critical | M17, M18 |
| 11 | 3-attempt lockout | Delivery | Critical | M18 |
| 12 | 7-day escrow | Payments | Critical | M16 |
| 13 | Merchant return policy | Returns | Critical | M19, M20 |
| 14 | 10+ store templates | Storefront | High | M07, M08 |
| 15 | 40+ service categories | Catalog | High | M04, M05 |
| 16 | No trial products | Catalog | High | M04 |
| 17 | No subscriptions | Catalog | High | M04 |
| 18 | No third-party branding | Catalog | Medium | M04, M23 |
| 19 | KYC mandatory | Compliance | Critical | M03 |
| 20 | 15% VAT on discounted | Tax | Critical | M16, M22 |
| 21 | ZATCA invoices | Tax | Critical | M16, M22 |
| 22 | Double-entry bookkeeping | Finance | Critical | M15, M16 |
| 23 | 5-year invoice retention | Compliance | Critical | M22, M23 |
| 24 | Max 50 cart items | Cart | Medium | M11 |
| 25 | Max 10 units/product | Cart | Medium | M11 |
| 26 | 500–5M YER range | Orders | Critical | M12, M13 |

---

*Document Version: 1.0 | Last Updated: 2026-09-13 | Classification: Confidential*
