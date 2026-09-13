# DESIGN COORDINATION GUIDE

**Project:** YemenMart Multi-Vendor E-Commerce Marketplace  
**Version:** 1.0.0  
**Last Updated:** 2026-09-13  
**Status:** Active  
**Applies To:** All Designers (A, B, C, D)

---

## 1. Overview

This guide coordinates the work of multiple designers across YemenMart's four portals. It ensures consistency in design tokens, components, layouts, and user experience across all customer-facing and internal surfaces.

**Core Principles:**
- Arabic-first, RTL-native design
- Mobile-first responsive approach
- Shared design system compliance
- Cross-portal visual consistency

**Brand Colors:**
- Navy Blue: `#1B2A4A` (Primary)
- Orange: `#F57C20` (Accent)

---

## 2. Design Team Structure

### Portal Assignments

| Portal | Designer | Tech Stack | Scope | Pages |
|--------|----------|------------|-------|-------|
| Customer Storefront | Designer A | React Native (Mobile) + Next.js 15 (Web) | Full customer experience | 13+ |
| Vendor Panel | Designer B | React 18 + Vite (Web) | Vendor management dashboard | 17+ |
| Admin Panel | Designer C | React 18 + Vite (Web) | Platform administration | 27+ |
| Delivery Provider | Designer D | React Native (Mobile) | Delivery management | 13+ |
| Auth System | Shared (All) | Shared across all portals | Authentication flows | 14+ |

### Designer Responsibilities

| Responsibility | Designer A | Designer B | Designer C | Designer D |
|----------------|:----------:|:----------:|:----------:|:----------:|
| Portal page layouts | ✓ | ✓ | ✓ | ✓ |
| Portal components | ✓ | ✓ | ✓ | ✓ |
| Responsive variants | ✓ | ✓ | ✓ | ✓ |
| RTL/LTR variants | ✓ | ✓ | ✓ | ✓ |
| Shared token compliance | ✓ | ✓ | ✓ | ✓ |
| Shared component usage | ✓ | ✓ | ✓ | ✓ |
| Auth system integration | ✓ | ✓ | ✓ | ✓ |
| Cross-portal alignment | ✓ | ✓ | ✓ | ✓ |

---

## 3. Shared Design Tokens

All designers MUST reference and use the tokens defined in `DESIGN-SYSTEM.md`. No custom tokens are allowed without team approval.

### 3.1 Color Tokens

| Token | Hex | Usage |
|-------|-----|-------|
| `--color-primary` | `#1B2A4A` | Navy Blue - headers, navigation, primary actions |
| `--color-accent` | `#F57C20` | Orange - CTAs, highlights, badges |
| `--color-success` | `#28A745` | Positive states, confirmations |
| `--color-warning` | `#FFC107` | Caution states, pending |
| `--color-danger` | `#DC3545` | Errors, destructive actions |
| `--color-info` | `#17A2B8` | Informational states |
| `--color-bg-primary` | `#FFFFFF` | Page backgrounds |
| `--color-bg-secondary` | `#F8F9FA` | Card backgrounds, sections |
| `--color-bg-tertiary` | `#E9ECEF` | Disabled states, separators |
| `--color-text-primary` | `#1B2A4A` | Headings, primary text |
| `--color-text-secondary` | `#6C757D` | Subtitles, descriptions |
| `--color-text-muted` | `#ADB5BD` | Placeholders, hints |
| `--color-border` | `#DEE2E6` | Borders, dividers |

### 3.2 Typography Tokens

| Token | Font Family | Weight | Size | Usage |
|-------|-------------|--------|------|-------|
| `--font-heading` | Tajawal | Bold (700) | 24-32px | Page titles |
| `--font-subheading` | Tajawal | SemiBold (600) | 18-20px | Section headers |
| `--font-body` | Tajawal | Regular (400) | 14-16px | Body text |
| `--font-caption` | Tajawal | Regular (400) | 12px | Captions, hints |
| `--font-ui` | Inter | Medium (500) | 14px | Buttons, labels |
| `--font-mono` | Inter | Regular (400) | 13px | Code, numbers |

### 3.3 Spacing Scale

| Token | Value | Usage |
|-------|-------|-------|
| `--space-xxs` | 4px | Tight spacing (icon gaps) |
| `--space-xs` | 8px | Small spacing (inline elements) |
| `--space-sm` | 12px | Compact spacing |
| `--space-md` | 16px | Default spacing |
| `--space-lg` | 24px | Section spacing |
| `--space-xl` | 32px | Large section spacing |
| `--space-xxl` | 48px | Page section spacing |

### 3.4 Border Radius

| Token | Value | Usage |
|-------|-------|-------|
| `--radius-sm` | 4px | Small elements (badges, chips) |
| `--radius-md` | 8px | Medium elements (cards, inputs) |
| `--radius-lg` | 12px | Large elements (modals, panels) |
| `--radius-xl` | 16px | XL elements (bottom sheets) |
| `--radius-full` | 9999px | Circular (avatars, pills) |

### 3.5 Shadows

| Token | Value | Usage |
|-------|-------|-------|
| `--shadow-sm` | `0 1px 2px rgba(0,0,0,0.05)` | Subtle elevation |
| `--shadow-md` | `0 4px 6px rgba(0,0,0,0.07)` | Card elevation |
| `--shadow-lg` | `0 10px 15px rgba(0,0,0,0.1)` | Modal elevation |
| `--shadow-xl` | `0 20px 25px rgba(0,0,0,0.15)` | Dropdown elevation |

### 3.6 Breakpoints

| Token | Value | Target |
|-------|-------|--------|
| `--bp-mobile` | 0-576px | Mobile phones |
| `--bp-tablet` | 577-768px | Tablets (portrait) |
| `--bp-desktop` | 769-1024px | Tablets (landscape) / Small desktop |
| `--bp-wide` | 1025-1440px | Desktop |
| `--bp-ultra` | 1441px+ | Large desktop |

### 3.7 Z-Index Scale

| Token | Value | Usage |
|-------|-------|-------|
| `--z-base` | 0 | Default layer |
| `--z-dropdown` | 1000 | Dropdowns, tooltips |
| `--z-sticky` | 1100 | Sticky headers |
| `--z-overlay` | 1200 | Overlays, backdrops |
| `--z-modal` | 1300 | Modals, dialogs |
| `--z-toast` | 1400 | Toast notifications |
| `--z-tooltip` | 1500 | Tooltips |

---

## 4. Shared Component Library

All designers MUST reference the component library defined in `08-COMPONENT-LIBRARY-SPEC.md`. Portal-specific components must extend shared components, not replace them.

### 4.1 Button Variants

| Variant | Style | Usage |
|---------|-------|-------|
| `btn-primary` | Navy Blue background, white text | Primary actions |
| `btn-accent` | Orange background, white text | CTAs, highlights |
| `btn-outline` | Navy Blue border, transparent | Secondary actions |
| `btn-ghost` | Transparent, no border | Tertiary actions |
| `btn-danger` | Red background, white text | Destructive actions |
| `btn-disabled` | Gray background, muted text | Disabled state |

### 4.2 Input Styles

| Variant | Style | Usage |
|---------|-------|-------|
| `input-default` | White bg, gray border | Standard inputs |
| `input-focused` | White bg, Orange border | Active focus state |
| `input-error` | White bg, red border | Validation errors |
| `input-disabled` | Gray bg, muted text | Disabled state |
| `input-rtl` | Mirrored layout | RTL direction |

### 4.3 Card Layouts

| Variant | Usage |
|---------|-------|
| `card-product` | Product display cards |
| `card-order` | Order summary cards |
| `card-user` | User/vendor profile cards |
| `card-stat` | Statistics/KPI cards |
| `card-action` | Cards with primary action |

### 4.4 Modal Patterns

| Variant | Usage |
|---------|-------|
| `modal-alert` | Confirmation dialogs |
| `modal-form` | Forms requiring input |
| `modal-info` | Information display |
| `modal-fullscreen` | Full-page modals (mobile) |
| `modal-bottom-sheet` | Mobile bottom sheets |

### 4.5 Table Designs

| Variant | Usage |
|---------|-------|
| `table-default` | Standard data tables |
| `table-sortable` | Tables with column sorting |
| `table-selectable` | Tables with row selection |
| `table-responsive` | Tables that collapse on mobile |

### 4.6 Badge/Status Chips

| Variant | Color | Usage |
|---------|-------|-------|
| `badge-success` | Green | Completed, Active, Approved |
| `badge-warning` | Yellow | Pending, Processing |
| `badge-danger` | Red | Cancelled, Rejected, Error |
| `badge-info` | Blue | Informational |
| `badge-neutral` | Gray | Default, Draft |

### 4.7 Navigation Patterns

| Variant | Usage |
|---------|-------|
| `nav-sidebar` | Desktop sidebar navigation |
| `nav-topbar` | Desktop top navigation |
| `nav-tabbar` | Mobile bottom tab bar |
| `nav-drawer` | Mobile slide-out drawer |
| `nav-breadcrumb` | Breadcrumb navigation |

---

## 5. Cross-Portal Consistency Rules

All designers MUST follow these rules to ensure consistency across all portals.

### CR-NAV: Navigation Patterns

| Rule | Description | Applies To |
|------|-------------|------------|
| CR-NAV-01 | All portals use same icon set for navigation | All |
| CR-NAV-02 | Active state uses Orange accent color | All |
| CR-NAV-03 | Navigation items have consistent spacing (16px vertical) | All |
| CR-NAV-04 | Mobile navigation uses bottom tab bar (max 5 items) | All |
| CR-NAV-05 | Desktop navigation uses sidebar (240px width) | Vendor, Admin |
| CR-NAV-06 | Navigation labels use Tajawal SemiBold 14px | All |
| CR-NAV-07 | RTL navigation mirrors layout direction | All |

### CR-TYP: Typography Hierarchy

| Rule | Description | Applies To |
|------|-------------|------------|
| CR-TYP-01 | Page titles: Tajawal Bold 28px, Navy Blue | All |
| CR-TYP-02 | Section headers: Tajawal SemiBold 20px, Navy Blue | All |
| CR-TYP-03 | Card titles: Tajawal SemiBold 16px, Navy Blue | All |
| CR-TYP-04 | Body text: Tajawal Regular 14px, Gray-700 | All |
| CR-TYP-05 | Captions: Tajawal Regular 12px, Gray-500 | All |
| CR-TYP-06 | All text must have 4.5:1 minimum contrast ratio | All |
| CR-TYP-07 | Line height: 1.5 for body, 1.2 for headings | All |

### CR-CLR: Color Usage

| Rule | Description | Applies To |
|------|-------------|------------|
| CR-CLR-01 | Navy Blue for primary UI elements | All |
| CR-CLR-02 | Orange for CTAs and accent highlights | All |
| CR-CLR-03 | Success states always use Green (#28A745) | All |
| CR-CLR-04 | Error states always use Red (#DC3545) | All |
| CR-CLR-05 | Warning states always use Yellow (#FFC107) | All |
| CR-CLR-06 | Backgrounds alternate: White and Gray-50 | All |
| CR-CLR-07 | Text on colored backgrounds must be White | All |

### CR-SPC: Spacing System

| Rule | Description | Applies To |
|------|-------------|------------|
| CR-SPC-01 | Page padding: 24px desktop, 16px mobile | All |
| CR-SPC-02 | Card padding: 20px desktop, 16px mobile | All |
| CR-SPC-03 | Section spacing: 32px vertical | All |
| CR-SPC-04 | Element spacing: 16px between related items | All |
| CR-SPC-05 | Inline spacing: 8px between adjacent items | All |
| CR-SPC-06 | No spacing values outside the defined scale | All |

### CR-FRM: Form Patterns

| Rule | Description | Applies To |
|------|-------------|------------|
| CR-FRM-01 | Labels above inputs, 8px gap | All |
| CR-FRM-02 | Input height: 48px desktop, 44px mobile | All |
| CR-FRM-03 | Error messages below inputs, Red color | All |
| CR-FRM-04 | Required fields marked with asterisk (Orange) | All |
| CR-FRM-05 | Form sections separated by 24px vertical spacing | All |
| CR-FRM-06 | Submit buttons use btn-primary or btn-accent | All |
| CR-FRM-07 | RTL forms mirror field order | All |

### CR-TBL: Table Layouts

| Rule | Description | Applies To |
|------|-------------|------------|
| CR-TBL-01 | Table header: Navy Blue background, White text | All |
| CR-TBL-02 | Row height: 56px desktop, 48px mobile | All |
| CR-TBL-03 | Alternating row backgrounds: White and Gray-50 | All |
| CR-TBL-04 | Mobile tables collapse to card layout | All |
| CR-TBL-05 | Sortable columns show indicator icon | All |
| CR-TBL-06 | Selected rows highlighted with Orange tint | All |

### CR-MDL: Modal Patterns

| Rule | Description | Applies To |
|------|-------------|------------|
| CR-MDL-01 | Modals centered on desktop, bottom sheet on mobile | All |
| CR-MDL-02 | Modal header: Navy Blue background, White text | All |
| CR-MDL-03 | Close button always top-left (RTL: top-right) | All |
| CR-MDL-04 | Modal width: 480px desktop, 100% mobile | All |
| CR-MDL-05 | Backdrop: Black 50% opacity | All |
| CR-MDL-06 | Focus trap within modal when open | All |

### CR-AUTH: Authentication Flow

| Rule | Description | Applies To |
|------|-------------|------------|
| CR-AUTH-01 | Auth screens use shared components only | All |
| CR-AUTH-02 | Login/registration forms follow CR-FRM rules | All |
| CR-AUTH-03 | OTP input uses 6-digit single-character fields | All |
| CR-AUTH-04 | Password requirements shown below input | All |
| CR-AUTH-05 | Social login buttons match portal button styles | All |
| CR-AUTH-06 | Auth screens are RTL-native | All |
| CR-AUTH-07 | Error handling follows shared error patterns | All |

---

## 6. Deliverables per Portal

### 6.1 Customer Storefront (Designer A)

| # | Deliverable | Description | Status |
|---|-------------|-------------|--------|
| 1 | Design System Integration Guide | How design tokens apply to customer portal | ☐ |
| 2 | Page Layouts (13+ pages) | Complete layouts for all customer pages | ☐ |
| 3 | Component Specifications | Detailed specs for customer-specific components | ☐ |
| 4 | Responsive Breakpoints | Mobile, tablet, desktop variants | ☐ |
| 5 | RTL/LTR Variations | Both direction variants for all pages | ☐ |
| 6 | Mobile App Screens | React Native screen designs | ☐ |
| 7 | Animation/Motion Specs | Transitions, micro-interactions | ☐ |
| 8 | Accessibility Checklist | WCAG 2.1 AA compliance annotations | ☐ |

**Portal Pages:**
1. Home / Landing
2. Product Listing / Search Results
3. Product Detail
4. Shopping Cart
5. Checkout Flow
6. Order Confirmation
7. Order History / Tracking
8. User Profile / Account
9. Wishlist / Favorites
10. Categories / Browse
11. Vendor Store Page
12. Reviews / Ratings
13. Notifications
14. Settings / Preferences

### 6.2 Vendor Panel (Designer B)

| # | Deliverable | Description | Status |
|---|-------------|-------------|--------|
| 1 | Dashboard Layout | Main vendor dashboard with KPIs | ☐ |
| 2 | Product Management Screens | CRUD operations for products | ☐ |
| 3 | Order Management Screens | View, process, fulfill orders | ☐ |
| 4 | Store Settings Screens | Store profile, policies, branding | ☐ |
| 5 | Financial Dashboard | Revenue, payouts, transactions | ☐ |
| 6 | Analytics Dashboard | Sales analytics, visitor metrics | ☐ |
| 7 | Responsive Variants | Desktop and tablet layouts | ☐ |
| 8 | RTL/LTR Variations | Both direction variants | ☐ |

**Portal Pages:**
1. Dashboard (Overview)
2. Products List
3. Product Create/Edit
4. Product Categories
5. Orders List
6. Order Detail
7. Order Processing
8. Returns / Refunds
9. Store Profile
10. Store Settings
11. Payment Settings
12. Shipping Settings
13. Financial Summary
14. Transaction History
15. Payout History
16. Analytics Overview
17. Reports
18. Notifications
19. Support / Help

### 6.3 Admin Panel (Designer C)

| # | Deliverable | Description | Status |
|---|-------------|-------------|--------|
| 1 | Dashboard Layout | Admin overview with platform KPIs | ☐ |
| 2 | User Management Screens | User CRUD, roles, permissions | ☐ |
| 3 | Vendor Management Screens | Vendor approval, monitoring | ☐ |
| 4 | Order Management Screens | Platform-wide order oversight | ☐ |
| 5 | Financial Management Screens | Revenue, commissions, payouts | ☐ |
| 6 | Content Management Screens | CMS for static pages, banners | ☐ |
| 7 | System Configuration Screens | Platform settings, feature flags | ☐ |
| 8 | Responsive Variants | Desktop and tablet layouts | ☐ |
| 9 | RTL/LTR Variations | Both direction variants | ☐ |

**Portal Pages:**
1. Dashboard (Overview)
2. User List
3. User Detail / Profile
4. User Roles / Permissions
5. Vendor List
6. Vendor Approval Queue
7. Vendor Detail
8. Vendor Settings
9. Orders List
10. Order Detail
11. Returns Management
12. Disputes / Complaints
13. Revenue Overview
14. Commission Settings
15. Payout Management
16. Transaction Log
17. Products Moderation
18. Categories Management
19. Banners / Promotions
20. Pages / Content
21. Notifications Settings
22. Email Templates
23. System Settings
24. Feature Flags
25. Audit Logs
26. Reports
27. Support Tickets

### 6.4 Delivery Provider (Designer D)

| # | Deliverable | Description | Status |
|---|-------------|-------------|--------|
| 1 | Dashboard Layout | Delivery overview with active tasks | ☐ |
| 2 | Available Deliveries Screen | Browse and accept delivery requests | ☐ |
| 3 | Active Deliveries Screen | Current delivery with navigation | ☐ |
| 4 | Delivery Code Verification | OTP/code entry for handoff | ☐ |
| 5 | Earnings Dashboard | Daily/weekly/monthly earnings | ☐ |
| 6 | Profile Settings | Delivery provider profile | ☐ |
| 7 | Mobile App Screens | React Native screen designs | ☐ |
| 8 | RTL/LTR Variations | Both direction variants | ☐ |

**Portal Pages:**
1. Dashboard (Overview)
2. Available Deliveries List
3. Delivery Detail / Map View
4. Active Delivery (Navigation)
5. Pickup Confirmation
6. Delivery Confirmation
7. Code Verification Screen
8. Delivery History
9. Earnings Overview
10. Earnings History
11. Profile / Settings
12. Vehicle / Transport Settings
13. Notifications
14. Support / Help

### 6.5 Auth System (Shared)

| # | Deliverable | Description | Assigned To | Status |
|---|-------------|-------------|-------------|--------|
| 1 | Login Flow | Email/phone + password login | All Designers | ☐ |
| 2 | Registration Flow | Customer + Vendor registration | All Designers | ☐ |
| 3 | OTP Verification Flow | Phone/email OTP entry | All Designers | ☐ |
| 4 | Password Reset Flow | Forgot + reset password | All Designers | ☐ |
| 5 | Session Management | Token refresh, logout, session | All Designers | ☐ |
| 6 | Mobile + Desktop Variants | Responsive auth screens | All Designers | ☐ |

**Auth Pages:**
1. Login (Email/Phone)
2. Login (Social - Google, Apple)
3. Registration (Customer)
4. Registration (Vendor)
5. Registration (Delivery)
6. OTP Verification (Phone)
7. OTP Verification (Email)
8. Forgot Password
9. Reset Password
10. Password Success Confirmation
11. Session Expired
12. Account Locked
13. Email Verification
14. Two-Factor Authentication

---

## 7. Handoff Process

### 7.1 Design Handoff Checklist

Before any design work is handed to development, the following must be completed:

- [ ] All pages designed in Figma/Sketch
- [ ] Design tokens exported and documented
- [ ] Component library updated with new components
- [ ] Responsive variants created for all breakpoints
- [ ] RTL variants created for all layouts
- [ ] Accessibility annotations added (WCAG 2.1 AA)
- [ ] Interaction specs documented (hover, focus, active states)
- [ ] Animation specs defined (duration, easing, triggers)
- [ ] Developer handoff notes written
- [ ] Cross-portal consistency verified
- [ ] Token compliance verified
- [ ] Shared component usage verified

### 7.2 Handoff Package Contents

| Item | Format | Required |
|------|--------|----------|
| Design files | Figma/Sketch | ✓ |
| Token JSON | CSS Variables / JSON | ✓ |
| Component specs | Documentation | ✓ |
| Icon assets | SVG | ✓ |
| Image assets | PNG/WebP (multiple densities) | ✓ |
| Animation specs | Lottie / CSS / JSON | ✓ |
| Interaction notes | Markdown | ✓ |
| Accessibility notes | Markdown | ✓ |

### 7.3 Handoff Process Steps

| Step | Action | Owner | Reviewer |
|------|--------|-------|----------|
| 1 | Complete design for portal section | Designer | - |
| 2 | Self-review against checklist | Designer | - |
| 3 | Peer review (cross-portal) | Designer | Another Designer |
| 4 | Token compliance check | Designer | Design Lead |
| 5 | Component consistency check | Designer | Design Lead |
| 6 | Final approval | Design Lead | - |
| 7 | Export handoff package | Designer | - |
| 8 | Developer walkthrough | Designer | Developer |

---

## 8. Review Process

### 8.1 Design Review Gates

All designs must pass these review gates before handoff:

| Gate | Reviewer | Criteria | Blocking |
|------|----------|----------|----------|
| G1: Token Compliance | Design Lead | All tokens from DESIGN-SYSTEM.md used correctly | Yes |
| G2: Component Consistency | Design Lead | All shared components used correctly | Yes |
| G3: Cross-Portal Alignment | All Designers | Visual consistency across portals | Yes |
| G4: Accessibility | Accessibility Lead | WCAG 2.1 AA compliance | Yes |
| G5: RTL Compliance | Arabic Speaker | Correct RTL mirroring and layout | Yes |
| G6: Responsive Design | Design Lead | All breakpoints implemented correctly | Yes |

### 8.2 Review Process Flow

```
Design Complete
    ↓
Self-Review (Designer)
    ↓
Peer Review (Cross-Portal)
    ↓
Gate Review (Design Lead)
    ↓
Accessibility Review
    ↓
RTL Review
    ↓
Approved → Handoff to Development
```

### 8.3 Review Checklist Template

```markdown
## Design Review: [Portal Name] - [Section Name]

**Designer:** [Name]
**Reviewer:** [Name]
**Date:** [Date]

### Token Compliance (G1)
- [ ] All colors use design tokens
- [ ] All typography uses design tokens
- [ ] All spacing uses design tokens
- [ ] All border-radius uses design tokens
- [ ] All shadows use design tokens

### Component Consistency (G2)
- [ ] Shared components used correctly
- [ ] No custom components that duplicate shared ones
- [ ] Portal-specific components extend shared ones
- [ ] Component states documented

### Cross-Portal Alignment (G3)
- [ ] Navigation consistent with other portals
- [ ] Typography hierarchy consistent
- [ ] Color usage consistent
- [ ] Spacing consistent
- [ ] Form patterns consistent
- [ ] Table layouts consistent
- [ ] Modal patterns consistent

### Accessibility (G4)
- [ ] Color contrast meets 4.5:1 minimum
- [ ] Focus states visible
- [ ] Touch targets minimum 44px
- [ ] Form labels associated with inputs
- [ ] Error messages accessible
- [ ] Screen reader annotations added

### RTL Compliance (G5)
- [ ] Layout correctly mirrored
- [ ] Text alignment correct
- [ ] Icons mirrored where appropriate
- [ ] No LTR-specific assumptions

### Responsive Design (G6)
- [ ] Mobile variant (0-576px)
- [ ] Tablet variant (577-768px)
- [ ] Desktop variant (769px+)
- [ ] Content reflows correctly
- [ ] Navigation adapts to viewport

### Notes
[Additional notes or issues]
```

---

## 9. Communication Protocol

### 9.1 Weekly Sync Meeting

**Schedule:** Every [Day] at [Time]  
**Duration:** 30 minutes  
**Attendees:** All Designers + Design Lead

**Agenda:**
1. Portal progress updates (5 min each)
2. Cross-portal alignment issues (10 min)
3. Design token changes (5 min)
4. Component library updates (5 min)
5. Blocking issues (5 min)

### 9.2 Communication Channels

| Channel | Purpose | Frequency |
|---------|---------|-----------|
| Design Sync Meeting | Progress, alignment, blockers | Weekly |
| Design System Slack Channel | Quick questions, updates | Daily |
| Figma Comments | Design feedback, annotations | As needed |
| GitHub Issues | Bug reports, feature requests | As needed |
| Design Review Meetings | Formal design reviews | Per milestone |

### 9.3 Design System Changes

When any designer needs to change shared design elements:

| Step | Action | Owner | Timeline |
|------|--------|-------|----------|
| 1 | Propose change with rationale | Requesting Designer | Day 1 |
| 2 | Impact assessment (affected portals) | Design Lead | Day 2 |
| 3 | Team review and approval | All Designers | Day 3 |
| 4 | Update tokens/components | Requesting Designer | Day 4 |
| 5 | Notify all designers | Design Lead | Day 4 |
| 6 | Update affected portals | Affected Designers | Day 5-7 |

### 9.4 Conflict Resolution

| Conflict Type | Resolution Process |
|---------------|-------------------|
| Token disagreement | Design Lead decides, document in CONFLICTS-AND-DECISIONS.md |
| Component design dispute | Team vote, majority wins |
| Cross-portal alignment issue | Design Lead mediates |
| Accessibility conflict | Accessibility Lead decides |
| RTL conflict | Arabic-speaking designer decides |

---

## 10. File Organization

### 10.1 Directory Structure

```
UI_UX/
├── DESIGN-SYSTEM.md                    # Shared - All Designers
├── DESIGN-COORDINATION-GUIDE.md        # Shared - All Designers
├── 00-INDEX.md                         # Shared - All Designers
├── 01-UI-UX-MASTER-PLAN.md             # Shared - All Designers
├── 02-DESIGN-GUIDELINES.md             # Shared - All Designers
├── 08-COMPONENT-LIBRARY-SPEC.md        # Shared - All Designers
├── 09-DATA-FLOW-MAPPING-00.md          # Shared - All Designers
├── 10-ERROR-STATES-AND-EDGE-CASES-00.md # Shared - All Designers
├── 11-HCI-METRICS-AND-CRITERIA.md      # Shared - All Designers
├── 12-VISUAL-PREVIEW-DESCRIPTIONS.md   # Shared - All Designers
├── CONFLICTS-AND-DECISIONS.md          # Shared - All Designers
├── UI-UX-MISSING-INFORMATION.md        # Shared - All Designers
├── UI-UX-COVERAGE-REPORT.md            # Shared - All Designers
│
├── customer-storefront/                # Designer A
│   ├── 03-PORTAL-CUSTOMER-STOREFRONT-01.md
│   ├── pages/
│   │   ├── home.md
│   │   ├── product-listing.md
│   │   ├── product-detail.md
│   │   ├── cart.md
│   │   ├── checkout.md
│   │   ├── order-confirmation.md
│   │   ├── order-history.md
│   │   ├── profile.md
│   │   ├── wishlist.md
│   │   ├── categories.md
│   │   ├── vendor-store.md
│   │   ├── reviews.md
│   │   └── notifications.md
│   ├── components/
│   │   ├── product-card.md
│   │   ├── cart-item.md
│   │   ├── checkout-step.md
│   │   └── order-card.md
│   ├── assets/
│   │   ├── icons/
│   │   └── images/
│   └── README.md
│
├── vendor-panel/                       # Designer B
│   ├── 04-PORTAL-VENDOR-PANEL-01.md
│   ├── pages/
│   │   ├── dashboard.md
│   │   ├── products-list.md
│   │   ├── product-create.md
│   │   ├── product-edit.md
│   │   ├── product-categories.md
│   │   ├── orders-list.md
│   │   ├── order-detail.md
│   │   ├── order-processing.md
│   │   ├── returns.md
│   │   ├── store-profile.md
│   │   ├── store-settings.md
│   │   ├── payment-settings.md
│   │   ├── shipping-settings.md
│   │   ├── financial-summary.md
│   │   ├── transaction-history.md
│   │   ├── payout-history.md
│   │   ├── analytics.md
│   │   ├── reports.md
│   │   └── notifications.md
│   ├── components/
│   │   ├── product-table.md
│   │   ├── order-card.md
│   │   ├── stat-card.md
│   │   └── financial-chart.md
│   ├── assets/
│   │   ├── icons/
│   │   └── images/
│   └── README.md
│
├── admin-panel/                        # Designer C
│   ├── 05-PORTAL-ADMIN-PANEL-01.md
│   ├── pages/
│   │   ├── dashboard.md
│   │   ├── users-list.md
│   │   ├── user-detail.md
│   │   ├── user-roles.md
│   │   ├── vendors-list.md
│   │   ├── vendor-approval.md
│   │   ├── vendor-detail.md
│   │   ├── orders-list.md
│   │   ├── order-detail.md
│   │   ├── returns-management.md
│   │   ├── disputes.md
│   │   ├── revenue.md
│   │   ├── commissions.md
│   │   ├── payout-management.md
│   │   ├── transactions.md
│   │   ├── products-moderation.md
│   │   ├── categories.md
│   │   ├── banners.md
│   │   ├── content.md
│   │   ├── notifications-settings.md
│   │   ├── email-templates.md
│   │   ├── system-settings.md
│   │   ├── feature-flags.md
│   │   ├── audit-logs.md
│   │   ├── reports.md
│   │   └── support-tickets.md
│   ├── components/
│   │   ├── data-table.md
│   │   ├── stat-card.md
│   │   ├── user-card.md
│   │   └── approval-card.md
│   ├── assets/
│   │   ├── icons/
│   │   └── images/
│   └── README.md
│
├── delivery-provider/                  # Designer D
│   ├── 06-PORTAL-DELIVERY-PROVIDER-01.md
│   ├── pages/
│   │   ├── dashboard.md
│   │   ├── available-deliveries.md
│   │   ├── delivery-detail.md
│   │   ├── active-delivery.md
│   │   ├── pickup-confirmation.md
│   │   ├── delivery-confirmation.md
│   │   ├── code-verification.md
│   │   ├── delivery-history.md
│   │   ├── earnings.md
│   │   ├── earnings-history.md
│   │   ├── profile.md
│   │   ├── vehicle-settings.md
│   │   ├── notifications.md
│   │   └── support.md
│   ├── components/
│   │   ├── delivery-card.md
│   │   ├── map-view.md
│   │   ├── earnings-card.md
│   │   └── code-input.md
│   ├── assets/
│   │   ├── icons/
│   │   └── images/
│   └── README.md
│
└── auth-system/                        # Shared - All Designers
    ├── 07-PORTAL-AUTH-SYSTEM-01.md
    ├── pages/
    │   ├── login-email.md
    │   ├── login-social.md
    │   ├── register-customer.md
    │   ├── register-vendor.md
    │   ├── register-delivery.md
    │   ├── otp-phone.md
    │   ├── otp-email.md
    │   ├── forgot-password.md
    │   ├── reset-password.md
    │   ├── password-success.md
    │   ├── session-expired.md
    │   ├── account-locked.md
    │   ├── email-verification.md
    │   └── two-factor.md
    ├── components/
    │   ├── auth-form.md
    │   ├── otp-input.md
    │   ├── social-login.md
    │   └── password-input.md
    └── README.md
```

### 10.2 File Naming Conventions

| Type | Convention | Example |
|------|-----------|---------|
| Portal spec | `XX-PORTAL-NAME-VERSION.md` | `03-PORTAL-CUSTOMER-STOREFRONT-01.md` |
| Page design | `page-name.md` | `product-detail.md` |
| Component | `component-name.md` | `product-card.md` |
| Asset | `asset-name.format` | `logo.svg` |

### 10.3 File Templates

#### Page Template

```markdown
# [Page Name]

**Portal:** [Portal Name]
**Designer:** [Designer Name]
**Version:** 1.0.0
**Last Updated:** [Date]

## Overview
[Page description]

## Layout
[Layout description]

## Components Used
- [Component 1]
- [Component 2]

## Responsive Variants
### Desktop
[Description]

### Tablet
[Description]

### Mobile
[Description]

## RTL Variant
[Description]

## Interactions
[Interaction specs]

## Accessibility
[Accessibility notes]
```

#### Component Template

```markdown
# [Component Name]

**Type:** [Shared/Portal-Specific]
**Portal:** [Portal Name or "Shared"]

## Variants
### [Variant 1]
[Description]

### [Variant 2]
[Description]

## States
### Default
[Description]

### Hover
[Description]

### Active
[Description]

### Disabled
[Description]

### Error
[Description]

## Responsive Behavior
[Description]

## RTL Behavior
[Description]

## Accessibility
[Description]
```

---

## 11. Timelines and Milestones

### 11.1 Project Phases

| Phase | Duration | Activities |
|-------|----------|------------|
| Phase 1: Foundation | Weeks 1-2 | Design system setup, token definition, shared components |
| Phase 2: Core Pages | Weeks 3-6 | Main page designs for all portals |
| Phase 3: Secondary Pages | Weeks 7-9 | Supporting pages, settings, profiles |
| Phase 4: Responsive | Weeks 10-11 | Responsive variants for all pages |
| Phase 5: RTL | Week 12 | RTL variants for all pages |
| Phase 6: Review | Weeks 13-14 | Cross-portal review, consistency checks |
| Phase 7: Handoff | Weeks 15-16 | Developer handoff, documentation |

### 11.2 Designer Milestones

| Milestone | Designer A | Designer B | Designer C | Designer D |
|-----------|-----------|-----------|-----------|-----------|
| Foundation complete | Week 2 | Week 2 | Week 2 | Week 2 |
| Core pages complete | Week 5 | Week 6 | Week 6 | Week 5 |
| Secondary pages complete | Week 8 | Week 9 | Week 9 | Week 8 |
| Responsive variants complete | Week 10 | Week 11 | Week 11 | Week 10 |
| RTL variants complete | Week 12 | Week 12 | Week 12 | Week 12 |
| Cross-portal review | Week 13 | Week 13 | Week 13 | Week 13 |
| Final handoff | Week 16 | Week 16 | Week 16 | Week 16 |

---

## 12. Quality Assurance

### 12.1 Quality Checklist

| Category | Check | Status |
|----------|-------|--------|
| **Tokens** | All colors from design tokens | ☐ |
| **Tokens** | All typography from design tokens | ☐ |
| **Tokens** | All spacing from design tokens | ☐ |
| **Tokens** | All border-radius from design tokens | ☐ |
| **Tokens** | All shadows from design tokens | ☐ |
| **Components** | All shared components used correctly | ☐ |
| **Components** | No custom components duplicating shared | ☐ |
| **Components** | Component states documented | ☐ |
| **Layout** | Responsive variants for all breakpoints | ☐ |
| **Layout** | Content reflows correctly | ☐ |
| **Layout** | Navigation adapts to viewport | ☐ |
| **RTL** | Layout correctly mirrored | ☐ |
| **RTL** | Text alignment correct | ☐ |
| **RTL** | Icons mirrored where appropriate | ☐ |
| **Accessibility** | Color contrast 4.5:1 minimum | ☐ |
| **Accessibility** | Focus states visible | ☐ |
| **Accessibility** | Touch targets minimum 44px | ☐ |
| **Accessibility** | Form labels associated | ☐ |
| **Accessibility** | Error messages accessible | ☐ |
| **Cross-Portal** | Navigation consistent | ☐ |
| **Cross-Portal** | Typography consistent | ☐ |
| **Cross-Portal** | Color usage consistent | ☐ |
| **Cross-Portal** | Spacing consistent | ☐ |
| **Cross-Portal** | Forms consistent | ☐ |
| **Cross-Portal** | Tables consistent | ☐ |
| **Cross-Portal** | Modals consistent | ☐ |

### 12.2 Common Issues to Watch For

| Issue | Description | Prevention |
|-------|-------------|------------|
| Token drift | Using hardcoded values instead of tokens | Regular token compliance reviews |
| Component duplication | Creating new components that duplicate shared ones | Check shared library first |
| RTL gaps | Forgetting to mirror layouts | RTL review gate |
| Inconsistent spacing | Using custom spacing values | Spacing system enforcement |
| Accessibility gaps | Missing focus states or contrast | Accessibility review gate |
| Cross-portal mismatch | Different designs for similar elements | Cross-portal alignment review |

---

## 13. References

| Document | Description | Audience |
|----------|-------------|----------|
| `DESIGN-SYSTEM.md` | Design tokens and foundations | All Designers |
| `08-COMPONENT-LIBRARY-SPEC.md` | Shared component library | All Designers |
| `02-DESIGN-GUIDELINES.md` | Design principles and guidelines | All Designers |
| `09-DATA-FLOW-MAPPING-00.md` | Data flow between portals | All Designers |
| `10-ERROR-STATES-AND-EDGE-CASES-00.md` | Error handling patterns | All Designers |
| `11-HCI-METRICS-AND-CRITERIA.md` | Usability metrics | All Designers |
| `CONFLICTS-AND-DECISIONS.md` | Design decisions log | All Designers |
| `UI-UX-COVERAGE-REPORT.md` | Coverage tracking | All Designers |

---

## 14. Document Control

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0.0 | 2026-09-13 | Design Lead | Initial version |

---

**End of Design Coordination Guide**
