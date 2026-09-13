# YemenMart UI/UX — Missing Information Register

> **Project:** YemenMart Multi-Vendor E-Commerce Marketplace  
> **Generated:** 2026-09-13  
> **Status:** Open Items  
> **Total Items:** 48

---

## 1. Missing Page Specifications

| Missing ID | Category | Description | Impact | Required From | Priority |
|---|---|---|---|---|---|
| MP-001 | Page Spec | Delivery Provider registration/onboarding screens (profile setup, vehicle info, document upload) | High — delivery partner cannot onboard without UI | Delivery Ops Lead | P0 |
| MP-002 | Page Spec | Delivery Provider dashboard — active deliveries, earnings summary, route view | High — core delivery workflow undefined | Delivery Ops Lead | P0 |
| MP-003 | Page Spec | Delivery Provider order detail — pickup/dropoff, navigation handoff, status updates | High — delivery completion flow breaks | Delivery Ops Lead | P0 |
| MP-004 | Page Spec | Admin dashboard sub-pages — order detail, refund approval, dispute resolution | Medium — admin workflow gaps | Product Owner | P1 |
| MP-005 | Page Spec | Admin user management — vendor/customer detail views, account actions | Medium — user ops incomplete | Product Owner | P1 |
| MP-006 | Page Spec | Vendor analytics detail pages — sales trends, product performance, customer insights | Medium — vendor self-service limited | Product Owner | P1 |
| MP-007 | Page Spec | Vendor product variant management — SKU-level stock, pricing, images | High — variant products cannot be managed | Product Owner | P0 |
| MP-008 | Page Spec | Customer order tracking — real-time delivery status, map integration | High — customer expectation unmet | UX Lead | P0 |

## 2. Missing API Endpoints

| Missing ID | Category | Description | Impact | Required From | Priority |
|---|---|---|---|---|---|
| MA-001 | API | Delivery provider CRUD endpoints — register, profile update, availability toggle, location update | High — delivery module non-functional | Backend Lead | P0 |
| MA-002 | API | Delivery assignment algorithm endpoints — matching, reassignment, batch assign | High — order fulfillment breaks | Backend Lead | P0 |
| MA-003 | API | Delivery tracking endpoints — real-time location, ETA calculation, route optimization | High — tracking feature undefined | Backend Lead | P0 |
| MA-004 | API | Notification preferences CRUD — channel opt-in/out, quiet hours, frequency | Medium — personalization blocked | Backend Lead | P1 |
| MA-005 | API | Support ticket CRUD — create, assign, escalate, resolve, reopen, SLA tracking | High — customer support workflow broken | Backend Lead | P0 |
| MA-006 | API | Analytics/reporting endpoints — sales reports, traffic analytics, conversion funnels | Medium — vendor/admin insights missing | Backend Lead | P1 |
| MA-007 | API | Banner/promotion management — CRUD, scheduling, targeting, A/B variants | Medium — marketing capabilities blocked | Backend Lead | P1 |
| MA-008 | API | Loyalty points endpoints — earn, redeem, balance, history, tier transitions | Medium — loyalty program undefined | Backend Lead | P1 |

## 3. Missing Validation Rules

| Missing ID | Category | Description | Impact | Required From | Priority |
|---|---|---|---|---|---|
| MV-001 | Validation | Product name validation — min/max length, allowed characters (Arabic/English mix), profanity filter | Medium — data quality risk | Product Owner | P1 |
| MV-002 | Validation | Store description validation — max length, HTML stripping, language detection | Low — cosmetic but needed | Product Owner | P2 |
| MV-003 | Validation | Review text validation — min length (meaningful review), max length, spam detection | Medium — review quality risk | Product Owner | P1 |
| MV-004 | Validation | Coupon code format — allowed charset, length range, uniqueness constraints, case sensitivity | Medium — coupon system integrity | Product Owner | P1 |
| MV-005 | Validation | Delivery zone validation — polygon complexity, overlap rules, min/max area, coordinate bounds (Yemen) | High — delivery routing integrity | Delivery Ops Lead | P0 |
| MV-006 | Validation | Wallet balance validation — min top-up, max balance, decimal precision, currency constraints | High — payment integrity | Backend Lead | P0 |
| MV-007 | Validation | Phone number validation — Yemen country code (+967), format, carrier detection | High — auth flow breaks | Backend Lead | P0 |

## 4. Missing Business Rules

| Missing ID | Category | Description | Impact | Required From | Priority |
|---|---|---|---|---|---|
| MB-001 | Business Rule | Product moderation criteria — auto-approve thresholds, image requirements, category-specific rules | High — vendor onboarding bottleneck | Product Owner | P0 |
| MB-002 | Business Rule | Review moderation rules — automated flagging, manual review triggers, response SLA | Medium — trust & safety risk | Product Owner | P1 |
| MB-003 | Business Rule | Coupon usage rules — stacking limits, per-user caps, minimum order value, expiry logic, vendor-specific coupons | High — promotional strategy undefined | Product Owner | P0 |
| MB-004 | Business Rule | Loyalty points earning rules — earn rate per category, tier multipliers, points expiry, referral bonuses | Medium — loyalty program design gap | Product Owner | P1 |
| MB-005 | Business Rule | Delivery pricing rules — base fee, distance calc, weight surcharge, time-of-day pricing, free delivery thresholds | High — delivery pricing undefined | Delivery Ops Lead | P0 |
| MB-006 | Business Rule | Refund processing rules — eligible states, partial refund logic, wallet vs original payment, vendor split reversal | High — financial integrity risk | Product Owner | P0 |
| MB-007 | Business Rule | Vendor commission rules — category-based rates, tiered pricing, payout schedule, minimum payout threshold | High — revenue model undefined | Product Owner | P0 |

## 5. Missing Design Decisions

| Missing ID | Category | Description | Impact | Required From | Priority |
|---|---|---|---|---|---|
| MD-001 | Design Decision | Dark mode implementation scope — full app or partial, per-actor support, system preference detection | Medium — affects component library | UX Lead | P1 |
| MD-002 | Design Decision | Animation/transition preferences — duration standards, easing curves, reduced-motion alternatives, page transitions | Low — polish phase, but needed early | UX Lead | P2 |
| MD-003 | Design Decision | Loading skeleton exact shapes — per-component skeleton definitions, shimmer direction, responsive scaling | Medium — perceived performance impact | UX Lead | P1 |
| MD-004 | Design Decision | Empty state illustrations — per-module custom art or generic, tone of copy, CTA placement | Medium — user experience quality | UX Lead | P1 |
| MD-005 | Design Decision | Error state illustrations — error hierarchy (inline, modal, page-level), illustration style, retry affordance | Medium — error recovery UX | UX Lead | P1 |
| MD-006 | Design Decision | Image placeholder strategy — blur-up, solid color, SVG pattern, lazy load behavior | Low — visual polish | UX Lead | P2 |
| MD-007 | Design Decision | Toast/notification styling — position, duration, stacking, dismiss behavior, RTL mirroring | Medium — consistent feedback UX | UX Lead | P1 |

## 6. Missing Responsive Behaviors

| Missing ID | Category | Description | Impact | Required From | Priority |
|---|---|---|---|---|---|
| MR-001 | Responsive | Tablet-specific layouts — product grid columns, sidebar collapse, touch targets, dashboard split-view | Medium — tablet users underserved | UX Lead | P1 |
| MR-002 | Responsive | Mobile vendor panel — bottom nav vs hamburger, quick actions, order management touch UX | High — vendors manage on mobile | UX Lead | P0 |
| MR-003 | Responsive | Admin panel mobile considerations — whether mobile access is supported, key workflows on small screens | Low — admin typically desktop | UX Lead | P3 |
| MR-004 | Responsive | Delivery provider app — native or PWA, offline support, background location, low-bandwidth mode | High — delivery app is mobile-first | Delivery Ops Lead | P0 |
| MR-005 | Responsive | Cart/checkout flow on small screens — step indicators, payment sheet, address selection UX | High — conversion-critical path | UX Lead | P0 |

## 7. Missing Accessibility Details

| Missing ID | Category | Description | Impact | Required From | Priority |
|---|---|---|---|---|---|
| MA-A01 | Accessibility | Screen reader announcements for dynamic content — cart updates, order status changes, live region definitions | Medium — legal compliance risk (WCAG 2.1 AA) | Accessibility Lead | P1 |
| MA-A02 | Accessibility | Focus management for complex workflows — modals, multi-step forms, drawer navigation, RTL focus order | Medium — keyboard navigation broken | Accessibility Lead | P1 |
| MA-A03 | Accessibility | Reduced motion preferences — `prefers-reduced-motion` support, disable parallax, carousel auto-play | Low — compliance item | Accessibility Lead | P2 |
| MA-A04 | Accessibility | Color contrast for all interactive states — hover, focus, active, disabled in both themes | Medium — WCAG 2.1 AA requirement | Accessibility Lead | P1 |
| MA-A05 | Accessibility | Touch target sizing — minimum 44×44px for all interactive elements on mobile | Medium — mobile accessibility | Accessibility Lead | P1 |

## 8. Missing Localization

| Missing ID | Category | Description | Impact | Required From | Priority |
|---|---|---|---|---|---|
| ML-001 | Localization | Translation keys for all UI strings — complete key registry, nesting strategy, namespace organization | High — entire UI depends on i18n | Backend Lead | P0 |
| ML-002 | Localization | Pluralization rules — Arabic has 6 plural forms (zero, one, two, few, many, other), CLDR compliance | Medium — grammatical correctness | Backend Lead | P1 |
| ML-003 | Localization | Date format edge cases — Hijri/Gregorian calendar toggle, time zones (AST), relative time strings in Arabic | Medium — date display consistency | Backend Lead | P1 |
| ML-004 | Localization | Number format edge cases — Arabic-Indic numerals option, currency formatting (YER), decimal separator, thousand separator | Medium — number display consistency | Backend Lead | P1 |
| ML-005 | Localization | RTL layout validation — mirrored icons, bidirectional text handling, mixed LTR content (phone numbers, codes) | High — RTL is non-negotiable | UX Lead | P0 |
| ML-006 | Localization | Voice input support — Arabic speech-to-text for search, product descriptions | Low — enhancement | Product Owner | P3 |

## 9. Missing Performance Targets

| Missing ID | Category | Description | Impact | Required From | Priority |
|---|---|---|---|---|---|
| MP-F01 | Performance | Specific bundle size limits — initial JS < 200KB gzipped, per-route chunk limits, vendor bundle isolation | Medium — performance budget needed | Frontend Lead | P1 |
| MP-F02 | Performance | Image optimization requirements — WebP/AVIF support, responsive srcset breakpoints, CDN strategy, lazy loading thresholds | High — page load on slow networks | Frontend Lead | P0 |
| MP-F03 | Performance | Cache invalidation strategies — stale-while-revalidate for product data, cart cache policy, API response caching | Medium — data freshness vs speed | Backend Lead | P1 |
| MP-F04 | Performance | Time to Interactive targets — homepage < 3s on 3G, search results < 2s, checkout < 4s | High — conversion impact on slow networks | Frontend Lead | P0 |
| MP-F05 | Performance | Offline/PWA strategy — service worker scope, offline cart, background sync, cache-first for static assets | Medium — offline experience undefined | Frontend Lead | P1 |

## 10. Missing Security Details

| Missing ID | Category | Description | Impact | Required From | Priority |
|---|---|---|---|---|---|
| MS-001 | Security | 2FA implementation scope — which actors require 2FA, SMS vs app-based, backup codes, recovery flow | High — admin/vendor account security | Security Lead | P0 |
| MS-002 | Security | Device fingerprinting details — what signals collected, consent requirements, storage, rotation policy | Medium — fraud prevention | Security Lead | P1 |
| MS-003 | Security | Session management specifics — token lifetime, refresh strategy, concurrent session limits, forced logout | High — session hijacking risk | Security Lead | P0 |
| MS-004 | Security | Rate limiting rules — per-endpoint limits, SMS throttle, login attempt caps, API key rotation | High — abuse prevention | Security Lead | P0 |
| MS-005 | Security | Data encryption requirements — at-rest for PII, in-transit enforcement, key management, wallet data encryption | High — compliance requirement | Security Lead | P0 |

---

## Priority Assessment

| Priority | Count | Items | Definition |
|---|---|---|---|
| **P0** | 22 | Core workflow blockers, security gaps, financial integrity | Must resolve before development begins |
| **P1** | 20 | Quality, compliance, polish, business model gaps | Must resolve before beta release |
| **P2** | 5 | Visual polish, edge cases, nice-to-haves | Can resolve post-launch |
| **P3** | 3 | Future enhancements, optional features | Backlog items |

---

## Recommended Approach for Gathering Missing Information

### Phase 1 — Immediate (Before Sprint 0)
| Action | Owner | Method | Deadline |
|---|---|---|---|
| Conduct delivery ops workshop — define provider lifecycle, pricing, assignment | Delivery Ops Lead | Workshop + doc review | 1 week |
| Define payment/refund/commission business rules | Product Owner | Stakeholder interviews | 1 week |
| Resolve security architecture — 2FA, sessions, rate limiting | Security Lead | Security review meeting | 1 week |
| Complete API endpoint inventory for delivery & support modules | Backend Lead | API-first design session | 1 week |
| Finalize phone validation rules for Yemen (+967) | Backend Lead | Telecom research | 3 days |

### Phase 2 — Before UI Development
| Action | Owner | Method | Deadline |
|---|---|---|---|
| Complete translation key registry and i18n setup | Backend Lead + UX Lead | Audit existing strings + plan | 2 weeks |
| Define loading skeleton specs per component | UX Lead | Design system session | 1 week |
| Specify accessibility requirements (WCAG 2.1 AA) | Accessibility Lead | Accessibility audit | 1 week |
| Define responsive breakpoints and tablet layouts | UX Lead | Design review | 1 week |
| Set performance budgets and monitoring | Frontend Lead | Technical planning | 1 week |

### Phase 3 — Before Beta
| Action | Owner | Method | Deadline |
|---|---|---|---|
| Finalize dark mode scope | UX Lead | Design decision doc | 2 weeks |
| Complete empty/error state illustrations | UX Lead + Designer | Illustration sprint | 2 weeks |
| Validate all validation rules with edge cases | QA Lead | Test case review | 1 week |
| Document offline/PWA strategy | Frontend Lead | Architecture decision | 1 week |

---

## Dependencies Between Missing Items

```
MA-001 (Delivery API) ──────┐
  ├── depends on ── MB-005 (Delivery Pricing Rules)
  ├── depends on ── MV-005 (Delivery Zone Validation)
  └── blocks ────── MP-001/002/003 (Delivery Pages)

MA-005 (Support Ticket API) ── depends on ── MP-004 (Admin Sub-pages)

MB-003 (Coupon Rules) ── depends on ── MV-004 (Coupon Validation)

MB-007 (Commission Rules) ── depends on ── MA-008 (Loyalty Points API)

MA-F02 (Image Optimization) ── blocks ── MP-F04 (Performance Targets)

MS-001 (2FA Scope) ── blocks ── MP-001 (Delivery Registration)
                          └── blocks ── Vendor/Admin onboarding flows

ML-001 (Translation Keys) ── blocks ── ML-002/003/004 (All Localization)
                        └── blocks ── All UI development

MV-007 (Phone Validation) ── blocks ── SMS Auth Flow
                         └── blocks ── All actor registration

MB-001 (Product Moderation) ── blocks ── MP-007 (Variant Management)
                          └── blocks ── Vendor product listing flow

MD-001 (Dark Mode Scope) ── depends on ── MA-004 (Color Contrast)
```

### Critical Path Dependencies
1. **Phone validation (MV-007)** → SMS auth → All registration flows
2. **Translation keys (ML-001)** → Entire i18n layer → All UI
3. **Delivery API (MA-001)** → Delivery pages → Order fulfillment
4. **Session management (MS-003)** → Auth security → All authenticated flows

---

## Summary

| Category | Total Items | P0 | P1 | P2 | P3 |
|---|---|---|---|---|---|
| Page Specifications | 8 | 4 | 4 | 0 | 0 |
| API Endpoints | 8 | 4 | 4 | 0 | 0 |
| Validation Rules | 7 | 3 | 4 | 0 | 0 |
| Business Rules | 7 | 4 | 3 | 0 | 0 |
| Design Decisions | 7 | 0 | 4 | 3 | 0 |
| Responsive Behaviors | 5 | 3 | 1 | 0 | 1 |
| Accessibility | 5 | 0 | 4 | 1 | 0 |
| Localization | 6 | 2 | 3 | 0 | 1 |
| Performance | 5 | 2 | 3 | 0 | 0 |
| Security | 5 | 4 | 1 | 0 | 0 |
| **Total** | **48** | **22** | **20** | **5** | **3** |

---

*This document should be reviewed weekly and items updated as they are resolved. Assign owners to each item during sprint planning.*
