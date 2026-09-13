# YemenMart UI/UX Specification — Conflicts & Decisions Log

> **Version:** 1.0  
> **Date:** 2026-09-13  
> **Status:** Active  
> **Owner:** UI/UX Analysis Team

---

## Table of Contents

1. [Conflict Register](#1-conflict-register)
2. [Decisions Made](#2-decisions-made)
3. [Open Items Requiring Product Decisions](#3-open-items-requiring-product-decisions)
4. [Resolved Conflicts](#4-resolved-conflicts)

---

## 1. Conflict Register

| Conflict ID | Source A | Source B | Conflict | Impact | Recommended Resolution | Status |
|-------------|----------|----------|----------|--------|------------------------|--------|
| **C-001** | Wireframes (checkout step 2 — "الدفع عند التوصيل" / Cash on Delivery option visible) | Constraint 1 (Wallet-only payments; no COD, no card, no BNPL) | Wireframes explicitly show COD as a payment option in checkout, directly contradicting the wallet-only payment constraint. | **Critical** — Checkout flow and payment UI are built around a method that is not permitted. Affects checkout step 2 layout, order summary, and post-order messaging. | Remove COD from all wireframes. Checkout step 2 must show only: (1) Wallet balance + (2) Wallet top-up prompt if balance is insufficient. Update order confirmation messaging to reflect wallet-only deduction. | 🔴 Open |
| **C-002** | API specification: phone format `+967XXXXXXXXX` (E.164-like, with country code prefix) | Validation logic / some UX references: phone format `7XXXXXXXX` (local format, 9 digits, no prefix) | Inconsistent phone number validation across spec documents. Users may enter numbers in either format; backend and frontend disagree on canonical form. | **High** — Registration, login, and OTP flows may fail silently or reject valid numbers depending on which validation is applied. Impacts SMS delivery reliability. | Standardize on `+967XXXXXXXXX` (E.164) as the canonical format stored in the database. Accept both `7XXXXXXXX` and `+967XXXXXXXXX` at the UI input layer; normalize on the backend. Document this in the API contract. | 🔴 Open |
| **C-003** | Constraint 14: "Minimum 10 store templates" | Some UX wireframe references and copy mention "10+ templates" | Ambiguity on whether 10 is the hard minimum or whether more templates are expected at launch. | **Medium** — Affects vendor onboarding design (template selector UI) and content team deliverables. If more than 10 are expected, the selector UI needs pagination or filtering. | Treat 10 as the hard minimum (Constraint 14 is normative). "10+" in UX copy is aspirational; pin the spec to exactly "minimum 10" and design the template selector to scale (grid with pagination/filter). Update UX copy to say "10+". | 🟡 Open |
| **C-004** | Constraint 3: "Phone number is primary identifier" (implies phone-only auth) | FR-001: "Password login supported after phone verification" (implies password is an additional auth method) | Unclear whether password login is a primary, secondary, or optional authentication path. Affects login UI design and auth flow architecture. | **High** — If passwords are supported, the login screen needs a password field and "forgot password" flow. If not, the UI should be SMS-only. Half-measures create confusion. | **Phone + Password is PRIMARY login method.** All users must set a password during registration. Phone + OTP is SECONDARY, available only for forgot password scenarios. Login UI shows phone + password fields by default. "Forgot password" link triggers OTP verification flow. | ✅ Resolved |
| **C-005** | Constraint 8: "17 order states" | Some internal documentation and state diagrams show 16 states | Off-by-one discrepancy in the order state machine. Affects order tracking UI, status labels, and notification triggers. | **High** — If the UI shows 16 states but the backend enforces 17, users will see unexplained jumps or missing statuses. If the UI shows 17 but the backend has 16, one status label will never display. | Audit the full state machine. The canonical list must be agreed upon and locked. If 17 is correct, update all diagrams and UI labels. If 16 is correct, update Constraint 8. One source of truth in the API spec. | 🔴 Open |
| **C-006** | Constraint 20: "15% VAT on discounted price" (VAT applied after discount) | Some references suggest VAT is calculated on the pre-discount price or handled differently per category | Different VAT calculation methods across documents. Affects price display, order summary, and financial reporting. | **High** — Incorrect VAT calculation leads to legal/compliance issues and customer trust problems. Price shown at checkout must match final charge. | Standardize: VAT = 15% applied to the **discounted price** (post-discount). This is the YemenMart standard per Constraint 20. All price displays must show: Original Price → Discount → Subtotal → VAT (15%) → Total. Update all conflicting references. | 🔴 Open |
| **C-007** | Constraint 24: "Cart max 50 items total" | Some UX copy and wireframe notes reference different limits (e.g., 30 or "unlimited") | Inconsistent cart item limit across specifications. | **Medium** — Affects cart UI (pagination, scroll behavior, "add to cart" validation) and backend cart API limits. | Lock to **50 items** per Constraint 24. Design cart UI to handle up to 50 items with lazy loading / virtual scroll. Update all references to 50. | 🟡 Open |
| **C-008** | Constraint 12: "7-day escrow hold" | Some documentation mentions 5-day or 14-day escrow periods | Conflicting escrow hold durations affect vendor payout schedules and financial projections. | **High** — Vendors need clear, consistent information about when they get paid. Financial planning depends on this number. | Standardize to **7 days** per Constraint 12. This is the post-delivery hold period before funds release to vendor. Update all references and the vendor dashboard payout timeline. | 🟡 Open |
| **C-009** | SMS-only authentication references (Constraint 3, Constraint 5) | Some UX flows and feature references mention WhatsApp OTP as an alternative channel | Unclear whether WhatsApp is a supported OTP delivery channel alongside SMS. | **Medium** — If WhatsApp is supported, the auth UI needs a channel selector and WhatsApp integration. If not, references must be cleaned up. | Per the constraint suite, YemenMart uses **SMS-only** for OTP delivery (no WhatsApp, no email). Remove all WhatsApp OTP references. If WhatsApp is desired in future, it should be a separate feature request. | 🟡 Open |
| **C-010** | FR-003: "KYC review SLA: 48 hours" | API specification / operational docs mention "24–72 hours" for KYC review | Inconsistent KYC review timeframes affect vendor onboarding UX and internal SLA dashboards. | **Medium** — Vendors see conflicting expectations on how long KYC takes. Internal teams may track against the wrong SLA. | Standardize to **48 hours** as the target SLA (per FR-003). The "24–72 hours" range in operational docs reflects realistic variance but should not be the stated SLA. UI should display: "KYC review typically completes within 48 hours." | 🟡 Open |
| **C-011** | RTL design constraint (Constraint 2: Arabic-first) | Some wireframe components (e.g., progress bars, carousels) appear designed with LTR default alignment | UI components may not be fully mirrored for RTL, causing visual inconsistencies for Arabic users. | **Medium** — Arabic users will notice misaligned elements, awkward text flow, or reversed iconography. Impacts perceived quality. | All wireframes and component specs must be reviewed for RTL compliance. Progress bars, carousels, timelines, and icon placements must mirror. Use `direction: rtl` and logical properties (start/end vs left/right) in all CSS. | 🟡 Open |
| **C-012** | Constraint 6: "17 order states" (referenced in C-005) | Notification system references mention only 10–12 notification triggers | Not all order state transitions have corresponding user notifications defined. | **Medium** — Customers and vendors may not receive updates for intermediate states, leading to support inquiries. | Map all 17 order states to notification triggers. Define which states trigger SMS, in-app, or both. Ensure no state transition is "silent" without explicit justification. | 🟡 Open |

---

## 2. Decisions Made

| Decision ID | Date | Decision | Rationale | Approved By | Status |
|-------------|------|----------|-----------|-------------|--------|
| **D-001** | 2026-09-13 | **COD will be removed from all wireframes.** Checkout step 2 will show only wallet-based payment. | Constraint 1 is normative: wallet-only payments. COD contradicts the constraint suite and the Yemen fintech/regulatory environment. | Product Owner | ✅ Resolved |
| **D-002** | 2026-09-13 | **Phone format canonical form is `+967XXXXXXXXX`.** UI accepts both local and international formats; backend normalizes. | Aligns with E.164 international standard and simplifies SMS gateway integration. | Engineering Lead | ✅ Resolved |
| **D-003** | 2026-09-13 | **Phone + Password is PRIMARY login method.** All users set a password during registration. Phone + OTP is SECONDARY, used only for forgot password. | Password-based login provides faster, more reliable authentication. OTP delivery can fail due to network issues. Password is always available. OTP recovery is for edge cases. | Product Owner | ✅ Resolved |
| **D-004** | 2026-09-13 | **VAT is calculated on discounted price at 15%.** Price display format: Original → Discount → Subtotal → VAT → Total. | Constraint 20 is explicit. This is also standard practice in Yemen and aligns with ZATCA guidelines. | Finance / Legal | ✅ Resolved |
| **D-005** | 2026-09-13 | **Cart maximum is 50 items.** Cart UI must support this limit with virtualized rendering for performance. | Constraint 24 is the authoritative source. 50 items covers typical multi-vendor bulk orders. | Product Owner | ✅ Resolved |
| **D-006** | 2026-09-13 | **OTP delivery is SMS-only.** No WhatsApp, no email, no voice call. | Constraint 3 and Constraint 5 define SMS as the sole channel. WhatsApp integration is out of scope for v1. | Product Owner | ✅ Resolved |
| **D-007** | 2026-09-13 | **All wireframes must be audited for RTL compliance.** Arabic-first means every component defaults to RTL. | Constraint 2 requires Arabic-first design. LTR artifacts in RTL wireframes create rework later. | Design Lead | ✅ Resolved |
| **D-008** | 2026-09-13 | **Store template selector UI will support pagination.** Even though minimum is 10, the UI must scale for future growth. | Constraint 14 sets minimum at 10; the selector should not need redesign when more templates are added. | Design Lead | ✅ Resolved |

---

## 3. Open Items Requiring Product Decisions

| Item ID | Topic | Question | Options | Impact | Deadline | Assigned To |
|---------|-------|----------|---------|--------|----------|-------------|
| **O-001** | Password Login Flow | Should users be prompted to set a password after their first SMS login, or only on demand? | (A) Prompt after first login (B) Only on demand via profile settings (C) Never prompt, purely opt-in | Affects onboarding conversion and security posture. Prompting adds friction but improves retention. | Pre-sprint planning | Product Owner |
| **O-002** | KYC Document Types | Which KYC documents are accepted? National ID, passport, commercial registration? | (A) National ID only (B) National ID + Passport (C) National ID + Passport + CR | Affects vendor onboarding UI, document upload flow, and review process. | Pre-sprint planning | Product Owner + Legal |
| **O-003** | Wallet Top-Up Methods | If wallet is the only payment method, what are the top-up channels? | (A) Bank transfer only (B) Bank transfer + mobile money (C) Bank transfer + mobile money + cash agents | Directly affects wallet top-up UX and payment integration scope. | Sprint 0 | Product Owner + Finance |
| **O-004** | Order State 17 | Which specific state is the 17th? The state machine document must list all 17. | Need canonical list from backend team. | Affects order tracking UI, notifications, and state transition logic. | Pre-sprint planning | Engineering Lead |
| **O-005** | Store Template Design | Are the 10+ store templates pre-designed by the design team, or are vendors self-service? | (A) Pre-designed, vendor picks one (B) Vendor builds from components (C) Hybrid | Affects template selector UI, vendor onboarding, and design team workload. | Sprint 0 | Design Lead + Product Owner |
| **O-006** | Escrow Exceptions | Are there cases where escrow is released earlier than 7 days? (e.g., digital goods, vendor tier) | (A) No exceptions, always 7 days (B) Exceptions for digital goods (C) Tiered by vendor rating | Affects payout UI, vendor expectations, and financial modeling. | Pre-sprint planning | Product Owner + Finance |
| **O-007** | Notification Preferences | Can users opt out of certain notification types? Or are all SMS notifications mandatory? | (A) All mandatory (B) Users can mute non-critical (C) Users control all | Affects notification settings UI and compliance with anti-spam regulations. | Sprint 0 | Product Owner |
| **O-008** | Multi-Language Support | Should the UI support English as a secondary language, or Arabic-only? | (A) Arabic-only (B) Arabic + English toggle (C) Arabic primary, English for product descriptions only | Affects entire UI architecture, i18n framework, and design system. | Pre-sprint planning | Product Owner |
| **O-009** | Cart Persistence | Should cart contents persist across devices (synced to account) or be device-local only? | (A) Account-synced (B) Device-local only (C) Hybrid (guest = local, logged-in = synced) | Affects cart data model, offline behavior, and cross-device UX. | Sprint 0 | Product Owner + Engineering |
| **O-010** | Return/Refund Flow | What is the return window? How are refunds processed (wallet credit vs reverse transaction)? | (A) 7-day return, wallet credit (B) 14-day return, wallet credit (C) 7-day return, original payment method | Affects order detail UI, return request flow, and wallet system design. | Pre-sprint planning | Product Owner + Finance |

---

## 4. Resolved Conflicts

The following conflicts were identified, analyzed, and resolved during the specification review:

| Resolved ID | Original Conflict | Resolution | Date Resolved | Notes |
|-------------|-------------------|------------|---------------|-------|
| **R-001** | C-001 (COD in wireframes) | COD removed from all wireframes. Checkout step 2 shows only wallet payment. | 2026-09-13 | See Decision D-001. Wireframes to be updated in next revision. |
| **R-002** | C-002 (Phone format) | Canonical format: `+967XXXXXXXXX`. UI accepts both; backend normalizes. | 2026-09-13 | See Decision D-002. API spec updated. |
| **R-003** | C-004 (Password login) | Phone + Password is PRIMARY login. OTP is SECONDARY for forgot password only. | 2026-09-13 | See Decision D-003. Login UI updated to show password field by default. |
| **R-004** | C-006 (VAT rate) | VAT = 15% on discounted price. Display: Original → Discount → Subtotal → VAT → Total. | 2026-09-13 | See Decision D-004. Price display components updated. |
| **R-005** | C-007 (Cart item limit) | Maximum 50 items. Cart UI uses virtualized rendering. | 2026-09-13 | See Decision D-005. Constraint 24 is authoritative. |
| **R-006** | C-009 (OTP channels) | SMS-only. No WhatsApp, no email. | 2026-09-13 | See Decision D-006. All WhatsApp references removed. |

---

## Appendix: Conflict Severity Definitions

| Severity | Definition | Examples |
|----------|------------|---------|
| **Critical** | Directly contradicts a normative constraint. Affects core user flows. Must be resolved before design begins. | Payment method mismatch (C-001) |
| **High** | Creates ambiguity in a core feature. May cause implementation errors if unresolved. Must be resolved before sprint planning. | Auth method confusion (C-004), order state mismatch (C-005) |
| **Medium** | Creates inconsistency across documents. Unlikely to cause failures but degrades spec quality. Should be resolved before final spec sign-off. | Template count (C-003), cart limits (C-007) |
| **Low** | Minor inconsistency or aspirational copy. Can be resolved during implementation. | UX copy saying "10+" vs "10" (C-003) |

---

## Appendix: Status Definitions

| Status | Icon | Meaning |
|--------|------|---------|
| Open | 🔴 | Conflict identified, not yet resolved |
| In Progress | 🟡 | Resolution being discussed or implemented |
| Resolved | ✅ | Conflict resolved, decision documented |
| Deferred | ⏸️ | Deferred to a future phase/sprint |
| Won't Fix | ❌ | Identified as non-issue after analysis |

---

*This document is maintained alongside the YemenMart UI/UX specification suite. All conflicts must be resolved before design handoff. Updated as new conflicts are discovered during spec review.*
