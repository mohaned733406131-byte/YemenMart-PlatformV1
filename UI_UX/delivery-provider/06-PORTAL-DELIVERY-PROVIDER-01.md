# YemenMart — Delivery Provider Portal (DP) — Page Specifications

| Field | Value |
|---|---|
| **Document** | Delivery Provider Portal — Page Specifications |
| **Version** | 1.0.0 |
| **Status** | Draft |
| **Date** | 2026-09-13 |
| **Portal** | Delivery Provider (DP) |
| **Actor** | Delivery Provider (DR) |
| **Technology** | React Native (Mobile), React 18 + Vite (Web) |
| **Pages** | 13 |
| **Priority** | P0: 5 · P1: 6 · P2: 2 |

---

## Table of Contents

1. [Portal Overview](#1-portal-overview)
2. [ID Convention](#2-id-convention)
3. [Page Summary](#3-page-summary)
4. [DP-DB-001 — Delivery Dashboard](#4-dp-db-001--delivery-dashboard)
5. [DP-AS-001 — Available Deliveries / Bids](#5-dp-as-001--available-deliveries--bids)
6. [DP-AS-002 — Delivery Detail (Accept)](#6-dp-as-002--delivery-detail-accept)
7. [DP-AC-001 — Active Deliveries](#7-dp-ac-001--active-deliveries)
8. [DP-AC-002 — Delivery In-Progress](#8-dp-ac-002--delivery-in-progress)
9. [DP-CD-001 — Delivery Code Verification](#9-dp-cd-001--delivery-code-verification)
10. [DP-HS-001 — Delivery History](#10-dp-hs-001--delivery-history)
11. [DP-FN-001 — Earnings Dashboard](#11-dp-fn-001--earnings-dashboard)
12. [DP-FN-002 — Transaction History](#12-dp-fn-002--transaction-history)
13. [DP-PR-001 — Profile Settings](#13-dp-pr-001--profile-settings)
14. [DP-ZN-001 — Delivery Zone Management](#14-dp-zn-001--delivery-zone-management)
15. [DP-NT-001 — Notifications](#15-dp-nt-001--notifications)
16. [DP-RT-001 — Rating/Performance](#16-dp-rt-001--ratingperformance)
17. [Cross-Cutting Concerns](#17-cross-cutting-concerns)

---

## 1. Portal Overview

### 1.1 Portal Summary

| Attribute | Detail |
|---|---|
| Portal Name | Delivery Provider Portal |
| Portal Code | DP |
| Primary Actor | Delivery Provider (DR) |
| Primary Device | Mobile App (React Native) |
| Secondary Device | Web (React 18 + Vite) |
| Authentication | SMS OTP only (no email, no password) |
| Default Language | Arabic (RTL) |
| Layout Direction | RTL-first, mobile-first |
| Payment Model | Wallet-only (earnings received to delivery wallet) |

### 1.2 Platform Matrix

| Platform | Delivery Provider |
|---|---|
| Web (Desktop) | ✅ Responsive |
| Web (Tablet) | ✅ Responsive |
| Web (Mobile) | ✅ Responsive |
| Native Mobile App | ✅ Primary |
| PWA | ❌ |

### 1.3 Portal-Specific Constraints

| ID | Constraint | Description |
|---|---|---|
| DP-001 | No GPS Tracking | Delivery updates via delivery codes sent to customer via SMS/WhatsApp. No live location tracking. |
| DP-002 | 3-Attempt Lockout | After 3 failed delivery attempts, account is frozen for 24 hours. |
| DP-003 | Wallet-Only Earnings | All earnings credited to delivery provider's platform wallet. |
| DP-004 | SMS-Only Auth | Login via SMS OTP only. No email login. |
| DP-005 | 17 Order States | Full order lifecycle with 17 distinct states. |
| DP-006 | Delivery Code Verification | Customer provides delivery code to confirm receipt. |
| DP-007 | Arabic-First | All UI copy, labels, notifications in Arabic by default. |

### 1.4 Order States (Delivery-Relevant)

| State | Arabic | Description |
|---|---|---|
| PENDING_ASSIGNMENT | بانتظار التعيين | Order waiting for delivery provider assignment |
| ASSIGNED | تم التعيين | Assigned to delivery provider, pending acceptance |
| ACCEPTED | تم القبول | Delivery provider accepted the order |
| PICKUP_IN_PROGRESS | جاري التوصيل من البائع | In progress picking up from vendor |
| PICKED_UP | تم الاستلام من البائع | Items picked up from vendor |
| IN_TRANSIT | في الطريق | In transit to customer |
| ARRIVED_AT_CUSTOMER | وصل إلى العميل | Arrived at customer location |
| DELIVERY_ATTEMPT_FAILED | فشلت محاولة التوصيل | Delivery attempt failed |
| DELIVERED | تم التوصيل | Successfully delivered (code verified) |
| RETURNED_TO_VENDOR | عاد إلى البائع | Returned to vendor |
| CANCELLED_BY_CUSTOMER | ملغى من العميل | Cancelled by customer |
| CANCELLED_BY_VENDOR | ملغى من البائع | Cancelled by vendor |
| CANCELLED_BY_ADMIN | ملغى من الإدارة | Cancelled by admin |
| REFUNDED | مسترد | Order refunded |
| DISPUTE_OPENED | تم فتح نزاع | Dispute opened |
| DISPUTE_RESOLVED | تم حسم النزاع | Dispute resolved |
| EXPIRED | منتهي الصلاحية | Order expired |

### 1.5 Navigation Structure

**Mobile Bottom Navigation (RTL):**

```
+-----------------------------------------------+
|  🏠      📦      ✅     💰      👤          |
| الرئيسية المتاح  النشطة  الأرباح  حسابي    |
+-----------------------------------------------+
```

| Tab | Label (AR) | Label (EN) | Icon | Screen |
|---|---|---|---|---|
| Home | الرئيسية | Home | Home | DP-DB-001 |
| Available | المتاح | Available | Package | DP-AS-001 |
| Active | النشطة | Active | CheckCircle | DP-AC-001 |
| Earnings | الأرباح | Earnings | Wallet | DP-FN-001 |
| Profile | حسابي | Profile | User | DP-PR-001 |

---

## 2. ID Convention

All page IDs follow the pattern:

```
{Portal}-{Module}-{Type}-{Sequence}
```

| Component | Meaning |
|---|---|
| Portal | `DP` = Delivery Provider |
| Module | Two-letter module code (e.g., `DB` = Dashboard, `AS` = Available/Store) |
| Type | Page type: `PG` = Page, `MD` = Modal, `DR` = Drawer, `FL` = Flow |
| Sequence | Three-digit sequential number |

### Module Codes

| Code | Module Name | Description |
|---|---|---|
| DB | Dashboard | Portal dashboard |
| AS | Available/Store | Available deliveries / bids |
| AC | Active | Active deliveries in progress |
| CD | Code | Delivery code verification |
| HS | History | Delivery history |
| FN | Finance | Earnings and transactions |
| PR | Profile | Profile settings |
| ZN | Zone | Delivery zone management |
| NT | Notifications | Notification center |
| RT | Rating | Rating and performance |

---

## 3. Page Summary

| ID | Page Name | Module | Type | Priority | FR Ref | Status |
|---|---|---|---|---|---|---|
| DP-DB-001 | Delivery Dashboard | DB | PG | P0 | FR-015 | 📝 Draft |
| DP-AS-001 | Available Deliveries / Bids | AS | PG | P0 | FR-015 | 📝 Draft |
| DP-AS-002 | Delivery Detail (Accept) | AS | PG | P0 | FR-015 | 📝 Draft |
| DP-AC-001 | Active Deliveries | AC | PG | P0 | FR-015 | 📝 Draft |
| DP-AC-002 | Delivery In-Progress | AC | PG | P0 | FR-015 | 📝 Draft |
| DP-CD-001 | Delivery Code Verification | CD | FL | P1 | FR-015 | 📝 Draft |
| DP-HS-001 | Delivery History | HS | PG | P1 | FR-015 | 📝 Draft |
| DP-FN-001 | Earnings Dashboard | FN | PG | P1 | FR-015 | 📝 Draft |
| DP-FN-002 | Transaction History | FN | PG | P1 | FR-015 | 📝 Draft |
| DP-PR-001 | Profile Settings | PR | PG | P2 | FR-008 | 📝 Draft |
| DP-ZN-001 | Delivery Zone Management | ZN | PG | P2 | FR-015 | 📝 Draft |
| DP-NT-001 | Notifications | NT | PG | P1 | FR-012 | 📝 Draft |
| DP-RT-001 | Rating/Performance | RT | PG | P1 | FR-015 | 📝 Draft |

---

## 4. DP-DB-001 — Delivery Dashboard

### 4.1 Page Metadata

| Field | Value |
|---|---|
| Page ID | DP-DB-001 |
| Module | M10 — Dashboard |
| Type | Page (PG) |
| Priority | P0 |
| FR Ref | FR-015 |
| Building Blocks | B01, B08, B11, B13 |
| Constraints | DP-001, DP-002, DP-005, DP-007 |

### 4.2 Purpose

Primary landing screen for delivery providers. Provides at-a-glance summary of today's activity, active deliveries, earnings, and quick access to key workflows.

### 4.3 Layout

**Mobile (Primary — 360px):**

```
+-----------------------------------------------+
| ☰  [Logo]  الرئيسية              🔔  👤    |
+-----------------------------------------------+
|                                               |
|  مرحباً، [اسم المزود] 👋                    |
|  اليوم: ١٣ سبتمبر ٢٠٢٦                      |
|                                               |
+-----------------------------------------------+
|  📦 طلبات متاحة            ⚡ طلبات نشطة    |
|  ┌─────────────┐  ┌─────────────┐           |
|  │     ٥       │  │     ٢       │           |
|  │  طلب ينتظر │  │  قيد التوصيل│           |
|  └─────────────┘  └─────────────┘           |
+-----------------------------------------------+
|  💰 أرباح اليوم                               |
|  ┌─────────────────────────────┐             |
|  │  ٢٬٥٠٠ ر.ي                  │             |
|  │  ↑ ١٥٪ عن الأمس             │             |
|  └─────────────────────────────┘             |
+-----------------------------------------------+
|  📊 إحصائيات هذا الأسبوع                     |
|  ┌─────┐  ┌─────┐  ┌─────┐  ┌─────┐        |
|  │ ١٢  │  │ ١٠  │  │ ٩٥٪ │  │ ٤٫٨ │        |
|  │إجمالي│  │مسلم│  │نسبة │  │تقييم│        |
|  │طلبات│  │    │  │النجز│  │    │        |
|  └─────┘  └─────┘  └─────┘  └─────┘        |
+-----------------------------------------------+
|  ⏱ أحدث الطلبات النشطة                       |
|  ┌─────────────────────────────────┐         |
|  │ #١٢٣٤  قيد التوصيل  ٢٫٥ كم     │         |
|  │ #١٢٣٥  تم الاستلام  ٤٫١ كم     │         |
|  └─────────────────────────────────┘         |
|                                               |
+-----------------------------------------------+
|  🏠     📦      ✅     💰      👤          |
| الرئيسية المتاح  النشطة  الأرباح  حسابي    |
+-----------------------------------------------+
```

**Desktop (≥1024px):**

```
+----------+----------------------------------+
|          |                                  |
| Sidebar  |  Main Content                    |
| (RTL)    |  - Welcome header                |
|          |  - Stats grid (4 cards)          |
| الرئيسية |  - Earnings summary              |
| ---------|  - Active deliveries table        |
| المتاح   |  - Weekly chart                   |
| ---------|                                  |
| النشطة   |                                  |
| ---------|                                  |
| الأرباح  |                                  |
| ---------|                                  |
| حسابي    |                                  |
|          |                                  |
+----------+----------------------------------+
```

### 4.4 Components

#### 4.4.1 Welcome Header

| Property | Value |
|---|---|
| Component | WelcomeHeader |
| Content | Greeting + date |
| Greeting | `مرحبًا، {firstName} 👋` |
| Date | Arabic format: `اليوم: ١٣ سبتمبر ٢٠٢٦` |
| Location | Top of content area |
| Background | White or light gray |

#### 4.4.2 Quick Stats Cards

| Stat | Arabic Label | Value Source | Color |
|---|---|---|---|
| Available Deliveries | طلبات متاحة | `available_count` | Blue (`navy-800`) |
| Active Deliveries | طلبات نشطة | `active_count` | Green (`green-500`) |
| Today's Earnings | أرباح اليوم | `today_earnings` | Amber (`amber-500`) |
| Rating | تقييم | `rating` | Purple (`purple-500`) |

| Property | Value |
|---|---|
| Component | StatCard |
| Layout | 2×2 grid (mobile), 4-column (desktop) |
| Tap Action | Navigate to relevant screen |
| Card Style | Rounded-lg, white bg, shadow-sm, icon + value + label |

#### 4.4.3 Earnings Summary Card

| Property | Value |
|---|---|
| Component | EarningsSummaryCard |
| Content | Today's earnings, comparison to yesterday |
| Value | `٢٬٥٠٠ ر.ي` |
| Trend | `↑ ١٥٪ عن الأمس` (green if positive, red if negative) |
| Tap Action | Navigate to DP-FN-001 |
| Background | Gradient (navy-800 to navy-900) |
| Text Color | White |

#### 4.4.4 Weekly Stats Row

| Property | Value |
|---|---|
| Component | WeeklyStatsRow |
| Stats | Total deliveries, Successful, Success rate (%), Average rating |
| Layout | 4 equal-width cards, horizontal scroll on mobile |
| Card Style | White bg, rounded-lg, shadow-sm |

#### 4.4.5 Recent Active Deliveries List

| Property | Value |
|---|---|
| Component | ActiveDeliveryList |
| Items | Last 3–5 active deliveries |
| Per Item | Order ID, status badge, distance (estimated), customer name |
| Status Badge | Colored by state (see status colors below) |
| Tap Action | Navigate to DP-AC-002 (Delivery In-Progress) |
| Empty State | `لا توجد طلبات نشطة` with icon |
| Max Items | 5 (with "عرض الكل" link to DP-AC-001) |

### 4.5 Status Colors

| State | Background | Text | Badge |
|---|---|---|---|
| PENDING_ASSIGNMENT | `yellow-50` | `yellow-800` | `yellow-100` |
| ASSIGNED | `blue-50` | `blue-800` | `blue-100` |
| ACCEPTED | `indigo-50` | `indigo-800` | `indigo-100` |
| IN_TRANSIT | `orange-50` | `orange-800` | `orange-100` |
| DELIVERED | `green-50` | `green-800` | `green-100` |
| DELIVERY_ATTEMPT_FAILED | `red-50` | `red-800` | `red-100` |
| RETURNED_TO_VENDOR | `gray-50` | `gray-800` | `gray-100` |

### 4.6 Interactions

| Trigger | Action | Notes |
|---|---|---|
| Tap stat card | Navigate to related screen | Available → DP-AS-001, Active → DP-AC-001 |
| Tap earnings card | Navigate to DP-FN-001 | — |
| Tap active delivery item | Navigate to DP-AC-002 | — |
| Pull-to-refresh | Refresh all dashboard data | Spinner overlay |
| Swipe down (gesture) | Refresh | — |

### 4.7 Loading States

| Section | Loading Pattern |
|---|---|
| Welcome header | Skeleton: 2 lines |
| Stat cards | Skeleton: 4 cards with circles |
| Earnings card | Skeleton: 1 card with gradient placeholder |
| Weekly stats | Skeleton: 4 small cards |
| Recent deliveries | Skeleton: 3 list items |

### 4.8 Error States

| Error | Display | Action |
|---|---|---|
| Network error | Toast: `تحقق من اتصالك بالإنترنت` | Auto-retry, pull-to-refresh |
| Auth expired | Redirect to login | Auto-redirect |
| Account frozen (lockout) | Banner: `حسابك مقفل لمدة ٢٤ ساعة بسبب ٣ محاولات توصيل فاشلة` | No action (read-only) |

### 4.9 Empty States

| Scenario | Display |
|---|---|
| No available deliveries | Icon (package) + `لا توجد طلبات متاحة حاليًا` |
| No active deliveries | Icon (check circle) + `لا توجد طلبات نشطة` |
| Zero earnings today | `لم تسجل أي أرباح اليوم` |

### 4.10 Accessibility

| Requirement | Implementation |
|---|---|
| Screen reader | All stat values announced with labels |
| Keyboard | Tab order: stat cards → earnings → weekly → recent |
| Focus | Visible ring on interactive elements |
| Contrast | All text meets WCAG AA (4.5:1) |
| Touch target | 44px minimum on all cards |

### 4.11 RTL/LTR Behavior

| Element | RTL | LTR |
|---|---|---|
| Welcome text | Right-aligned | Left-aligned |
| Stat cards | Grid flows right-to-left | Grid flows left-to-right |
| Trend arrow (↑/↓) | No flip | No flip |
| Recent list | Swipe right for details | Swipe left for details |

---

## 5. DP-AS-001 — Available Deliveries / Bids

### 5.1 Page Metadata

| Field | Value |
|---|---|
| Page ID | DP-AS-001 |
| Module | M02 — Available/Store |
| Type | Page (PG) |
| Priority | P0 |
| FR Ref | FR-015 |
| Building Blocks | B01, B05, B08, B11 |
| Constraints | DP-001, DP-003, DP-007 |

### 5.2 Purpose

Displays all delivery orders available for acceptance. Delivery providers can view order details, estimated earnings, distance, and choose to accept or decline.

### 5.3 Layout

**Mobile (360px):**

```
+-----------------------------------------------+
|  [Back]   الطلبات المتاحة           🔍  ⚙️  |
+-----------------------------------------------+
|  ┌─────────────────────────────────┐         |
|  │  🔍 ابحث عن طلب...              │         |
|  └─────────────────────────────────┘         |
|                                               |
|  [الكل] [قريبة] [أرباح عالية] [جديدة]      |
|                                               |
|  ٥ طلبات متاحة                               |
|                                               |
|  ┌─────────────────────────────────┐         |
|  │  #١٢٣٤                          │         |
|  │  📍 صنعاء، حي đổi mới           │         |
|  │  → صنعاء، حي التvil              │         |
│  │  ─────────────────────────────── │         |
|  │  📦 ٣ منتجات    💰 ٥٠٠ ر.ي      │         |
│  │  📏 ٢٫٥ كم      ⏱ ١٥ دقيقة     │         |
│  │  ─────────────────────────────── │         |
│  │  [عرض التفاصيل]  [قبول الطلب]  │         |
│  └─────────────────────────────────┘         |
|                                               |
|  ┌─────────────────────────────────┐         |
|  │  #١٢٣٥                          │         |
│  │  📍 عدن، كريتر                   │         |
│  │  → عدن، صيرة                     │         |
│  │  ─────────────────────────────── │         |
│  │  📦 ٥ منتجات    💰 ٧٥٠ ر.ي      │         |
│  │  📏 ٤٫١ كم      ⏱ ٢٥ دقيقة     │         |
│  │  ─────────────────────────────── │         |
│  │  [عرض التفاصيل]  [قبول الطلب]  │         |
│  └─────────────────────────────────┘         |
|                                               |
|  ┌─────────────────────────────────┐         |
|  │  📭 لا مزيد من الطلبات          │         |
│  └─────────────────────────────────┘         |
|                                               |
+-----------------------------------------------+
|  🏠     📦      ✅     💰      👤          |
| الرئيسية المتاح  النشطة  الأرباح  حسابي    |
+-----------------------------------------------+
```

### 5.4 Components

#### 5.4.1 Search Bar

| Property | Value |
|---|---|
| Component | SearchBar |
| Placeholder | `ابحث عن طلب...` |
| Input Type | `search`, `dir="auto"` |
| Position | Below header, full width |
| Behavior | Debounced search (300ms) |

#### 5.4.2 Filter Tabs

| Property | Value |
|---|---|
| Component | FilterTabs |
| Options | `الكل` (All), `قريبة` (Nearby), `أرباح عالية` (High Earnings), `جديدة` (New) |
| Default | `الكل` |
| Style | Horizontal scroll, pill-shaped tabs |
| Active State | Filled background (navy-800), white text |

#### 5.4.3 Delivery Card

| Property | Value |
|---|---|
| Component | DeliveryCard |
| Content | Order ID, pickup address, delivery address, item count, earnings, distance, estimated time |
| Actions | "عرض التفاصيل" (View Details) → DP-AS-002, "قبول الطلب" (Accept) → Confirmation modal |
| Card Style | White bg, rounded-lg, shadow-sm, border-gray-200 |
| Swipe | Swipe left to dismiss/decline |

#### 5.4.4 Empty State

| Property | Value |
|---|---|
| Component | EmptyState |
| Icon | Package (outline) |
| Title | `لا توجد طلبات متاحة` |
| Description | `سيظهر الطلبات الجديدة هنا` |
| Action | Pull-to-refresh indicator |

### 5.5 Delivery Card Data Fields

| Field | Arabic Label | Source | Display |
|---|---|---|---|
| Order ID | رقم الطلب | `order.id` | `#١٢٣٤` |
| Pickup Address | عنوان الاستلام | `order.pickup_address` | `📍 صنعاء، حي التvil` |
| Delivery Address | عنوان التوصيل | `order.delivery_address` | `→ صنعاء، حي النزهة` |
| Item Count | عدد المنتجات | `order.items.length` | `📦 ٣ منتجات` |
| Earnings | الأرباح | `delivery.earnings` | `💰 ٥٠٠ ر.ي` |
| Distance | المسافة | `delivery.distance_km` | `📏 ٢٫٥ كم` |
| Est. Time | الوقت المقدر | `delivery.estimated_minutes` | `⏱ ١٥ دقيقة` |
| Created At | أُنشئ | `order.created_at` | `منذ ٥ دقائق` |

### 5.6 Interactions

| Trigger | Action | Notes |
|---|---|---|
| Tap "عرض التفاصيل" | Navigate to DP-AS-002 | — |
| Tap "قبول الطلب" | Show confirmation modal | — |
| Swipe left on card | Quick decline | Animated dismiss |
| Tap filter tab | Filter list | Active state update |
| Pull-to-refresh | Refresh available list | Spinner |
| Search input | Filter by order ID or address | Debounced |

### 5.7 Confirmation Modal (Accept)

```
+-----------------------------------------------+
|  تأكيد قبول الطلب                             |
|  ───────────────────────────────────────       |
|  هل تريد قبول الطلب #١٢٣٤؟                  |
|                                               |
|  من: صنعاء، حي التvil                          |
|  إلى: صنعاء، حي النزهة                        |
|  الأرباح: ٥٠٠ ر.ي                              |
|                                               |
|  [إلغاء]              [قبول الطلب]            |
+-----------------------------------------------+
```

| Property | Value |
|---|---|
| Destructive | No |
| Primary Button | "قبول الطلب" (navy-800) |
| Secondary Button | "إلغاء" (gray) |
| Confirm Action | POST `/api/delivery/accept/{orderId}` |
| Loading | Button shows spinner during API call |
| Success | Navigate to DP-AC-002 (Delivery In-Progress) |
| Error | Toast: `حدث خطأ أثناء قبول الطلب` |

### 5.8 Loading States

| Section | Loading Pattern |
|---|---|
| Search bar | Skeleton: 1 line |
| Filter tabs | Skeleton: 4 pills |
| Delivery cards | Skeleton: 3 cards |

### 5.9 Error States

| Error | Display | Action |
|---|---|---|
| Network error | Toast: `تحقق من اتصالك بالإنترنت` | Auto-retry |
| Accept failed | Toast: `لم يتم قبول الطلب، حاول مجددًا` | Retry button |
| No deliveries available | Empty state | Pull-to-refresh |

### 5.10 Accessibility

| Requirement | Implementation |
|---|---|
| Screen reader | Card announces: `طلب #{id}، من {pickup} إلى {delivery}، {items} منتجات، {earnings} ر.ي` |
| Keyboard | Tab: search → filters → cards → actions |
| Focus | Visible ring on card actions |
| Touch target | 44px on "قبول" and "عرض التفاصيل" buttons |

### 5.11 RTL/LTR Behavior

| Element | RTL | LTR |
|---|---|---|
| Pickup → Delivery arrow | `→` flips to `←` | `→` |
| Card layout | Pickup on right, delivery on left | Pickup on left, delivery on right |
| Filter tabs | Flow right-to-left | Flow left-to-right |

---

## 6. DP-AS-002 — Delivery Detail (Accept)

### 6.1 Page Metadata

| Field | Value |
|---|---|
| Page ID | DP-AS-002 |
| Module | M02 — Available/Store |
| Type | Page (PG) |
| Priority | P0 |
| FR Ref | FR-015 |
| Building Blocks | B01, B05, B08, B11 |
| Constraints | DP-001, DP-003, DP-007 |

### 6.2 Purpose

Full detail view of a delivery order before acceptance. Shows complete pickup and delivery addresses, customer info, item details, earnings breakdown, and estimated route information (no GPS — estimated distance/time only).

### 6.3 Layout

**Mobile (360px):**

```
+-----------------------------------------------+
|  [Back]   تفاصيل الطلب                       |
+-----------------------------------------------+
|                                               |
|  الطلب #١٢٣٤                    قيد الانتظار |
|  أُنشئ: منذ ٥ دقائق                          |
|                                               |
+-----------------------------------------------+
|  📍 نقاط التوصيل                              |
|  ┌─────────────────────────────────┐         |
|  │  🟢 الاستلام                    │         |
│  │  صنعاء، حي التvil، شارع ٤٠      │         |
│  │  ═══════════════════════════     │         |
│  │  🔴 التوصيل                      │         |
│  │  صنعاء، حي النزهة، مجمع ١٢     │         |
│  └─────────────────────────────────┘         |
|                                               |
+-----------------------------------------------+
|  👤 معلومات العميل                           |
|  ┌─────────────────────────────────┐         |
│  │  الاسم: أحمد محمد                │         |
│  │  الهاتف: 77X XXX XXXX           │         |
│  │  [اتصال] [رسالة]                 │         |
│  └─────────────────────────────────┘         |
|                                               |
+-----------------------------------------------+
|  📦 المنتجات (٣)                             |
|  ┌─────────────────────────────────┐         |
│  │  📱 آيفون 15 — ×١              │         |
│  │  🎧 سماعات — ×٢                │         |
│  │  📱 غطاء آيفون — ×١            │         |
│  └─────────────────────────────────┘         |
|                                               |
+-----------------------------------------------+
|  💰 ملخص الأرباح                             |
|  ┌─────────────────────────────────┐         |
│  │  أجرة التوصيل      ٥٠٠ ر.ي     │         |
│  │  المكافأة           ٠ ر.ي       │         |
│  │  ═══════════════════════════     │         |
│  │  الإجمالي          ٥٠٠ ر.ي     │         |
│  └─────────────────────────────────┘         |
|                                               |
+-----------------------------------------------+
|  📏 معلومات المسار                            |
|  ┌─────────────────────────────────┐         |
│  │  المسافة: ٢٫٥ كم                │         |
│  │  الوقت المقدر: ١٥ دقيقة          │         |
│  │  ⚠️ تقدير فقط (بدون GPS)         │         |
│  └─────────────────────────────────┘         |
|                                               |
+-----------------------------------------------+
|  ┌─────────────────────────────────┐         |
|  │  [رفض الطلب]    [قبول الطلب]   │         |
|  └─────────────────────────────────┘         |
+-----------------------------------------------+
```

### 6.4 Components

#### 6.4.1 Order Header

| Property | Value |
|---|---|
| Component | OrderHeader |
| Content | Order ID, status badge, created time |
| Status Badge | `بانتظار القبول` (yellow-100) |

#### 6.4.2 Delivery Points Card

| Property | Value |
|---|---|
| Component | DeliveryPointsCard |
| Content | Pickup address, delivery address, visual route indicator |
| Visual | Green dot (pickup) → dashed line → Red dot (delivery) |
| Direction | RTL: Right (pickup) to Left (delivery) |

#### 6.4.3 Customer Info Card

| Property | Value |
|---|---|
| Component | CustomerInfoCard |
| Content | Customer name, phone (partially masked) |
| Actions | Call button (tel:), WhatsApp button (if available) |
| Privacy | Phone shows `77X XXX XXXX` until order accepted |

#### 6.4.4 Items List

| Property | Value |
|---|---|
| Component | ItemsList |
| Content | Product name, quantity, optional thumbnail |
| Style | Compact list with icons |

#### 6.4.5 Earnings Breakdown

| Property | Value |
|---|---|
| Component | EarningsBreakdown |
| Content | Base delivery fee, bonus (if any), total |
| Style | Table-like rows, total highlighted |

#### 6.4.6 Route Info Card

| Property | Value |
|---|---|
| Component | RouteInfoCard |
| Content | Distance (km), estimated time |
| Note | `⚠️ تقدير فقط (بدون GPS)` — important constraint indicator |
| Constraint Ref | DP-001 (No GPS Tracking) |

#### 6.4.7 Action Buttons

| Button | Style | Action |
|---|---|---|
| رفض الطلب (Decline) | Secondary, red text | Confirmation modal → decline |
| قبول الطلب (Accept) | Primary, navy-800 | Confirmation modal → accept |

### 6.5 Interactions

| Trigger | Action | Notes |
|---|---|---|
| Tap "قبول الطلب" | Show confirm modal | — |
| Tap "رفض الطلب" | Show decline modal | — |
| Tap phone number | Open dialer (native) | `tel:` link |
| Tap WhatsApp | Open WhatsApp (native) | `whatsapp://` deep link |
| Scroll | Full page scroll | — |
| Back button | Navigate to DP-AS-001 | — |

### 6.6 Accept Confirmation Modal

| Property | Value |
|---|---|
| Title | `تأكيد قبول الطلب` |
| Message | `هل تريد قبول الطلب #١٢٢٤؟` |
| Primary Button | `قبول` (navy-800) |
| Secondary Button | `إلغاء` (gray) |
| Success | Navigate to DP-AC-002 |
| Error | Toast: `حدث خطأ، حاول مجددًا` |

### 6.7 Decline Confirmation Modal

| Property | Value |
|---|---|
| Title | `تأكيد رفض الطلب` |
| Message | `هل أنت متأكد من رفض هذا الطلب؟` |
| Reason Required | Yes — optional dropdown: `مسافة بعيدة`، `أرباح منخفضة`، `لا أستطيع` |
| Primary Button | `رفض` (red-500) |
| Secondary Button | `إلغاء` (gray) |
| Success | Navigate to DP-AS-001 with toast: `تم رفض الطلب` |

### 6.8 Loading/Error States

| State | Display |
|---|---|
| Loading | Skeleton for all sections |
| Network error | Toast: `تحقق من اتصالك بالإنترنت` |
| Accept error | Toast: `لم يتم قبول الطلب` |
| Decline error | Toast: `لم يتم رفض الطلب` |

### 6.9 Accessibility

| Requirement | Implementation |
|---|---|
| Screen reader | All sections announced with headings |
| Keyboard | Tab: back → header → points → customer → items → earnings → route → buttons |
| Focus | Visible ring on all interactive elements |

---

## 7. DP-AC-001 — Active Deliveries

### 7.1 Page Metadata

| Field | Value |
|---|---|
| Page ID | DP-AC-001 |
| Module | M04 — Active |
| Type | Page (PG) |
| Priority | P0 |
| FR Ref | FR-015 |
| Building Blocks | B01, B05, B08, B11 |
| Constraints | DP-001, DP-002, DP-005, DP-006, DP-007 |

### 7.2 Purpose

Lists all currently active deliveries — orders accepted and in progress. Shows real-time status for each, with quick actions.

### 7.3 Layout

**Mobile (360px):**

```
+-----------------------------------------------+
|  [Back]   الطلبات النشطة              ⚙️     |
+-----------------------------------------------+
|                                               |
|  ٢ طلب نشط                                   |
|                                               |
|  ┌─────────────────────────────────┐         |
│  │  #١٢٣٤  ● قيد التوصيل          │         |
│  │  من: صنعاء، حي التvil           │         |
│  │  إلى: صنعاء، حي النزهة         │         |
│  │  📏 ٢٫٥ كم  ⏱ ١٥ دقيقة        │         |
│  │  ─────────────────────────────── │         |
│  │  [تحديث الحالة]  [التفاصيل]    │         |
│  └─────────────────────────────────┘         |
|                                               |
|  ┌─────────────────────────────────┐         |
│  │  #١٢٣٥  ● تم الاستلام         │         |
│  │  من: عدن، كريتر                 │         |
│  │  إلى: عدن، صيرة                 │         |
│  │  📏 ٤٫١ كم  ⏱ ٢٥ دقيقة        │         |
│  │  ─────────────────────────────── │         |
│  │  [تحديث الحالة]  [التفاصيل]    │         |
│  └─────────────────────────────────┘         |
|                                               |
|  ┌─────────────────────────────────┐         |
│  │  ⚠️ ملاحظة: لا يمكن تتبع       │         |
│  │  الموقع المباشر (بدون GPS)      │         |
│  └─────────────────────────────────┘         |
|                                               |
+-----------------------------------------------+
|  🏠     📦      ✅     💰      👤          |
| الرئيسية المتاح  النشطة  الأرباح  حسابي    |
+-----------------------------------------------+
```

### 7.4 Components

#### 7.4.1 Active Delivery Card

| Property | Value |
|---|---|
| Component | ActiveDeliveryCard |
| Content | Order ID, status, pickup/delivery addresses, distance, time |
| Status Indicator | Colored dot + status text |
| Actions | "تحديث الحالة" (Update Status), "التفاصيل" (Details) |
| Card Style | White bg, rounded-lg, shadow, left border color = status color |

#### 7.4.2 GPS Warning Banner

| Property | Value |
|---|---|
| Component | InfoBanner |
| Content | `⚠️ لا يمكن تتبع الموقع المباشر (بدون GPS)` |
| Style | Yellow-50 bg, yellow-700 text |
| Dismissible | No (persistent constraint reminder) |

#### 7.4.3 Update Status Button

| Property | Value |
|---|---|
| Component | StatusUpdateButton |
| Content | "تحديث الحالة" |
| Action | Opens status update flow (DP-AC-002) |
| Style | Primary, navy-800 |

### 7.5 Status Flow (Delivery Provider View)

```
                    ┌──────────────────┐
                    │   ASSIGNED       │
                    │  (تم التعيين)     │
                    └────────┬─────────┘
                             │ قبول (Accept)
                             ▼
                    ┌──────────────────┐
                    │   ACCEPTED       │
                    │  (تم القبول)      │
                    └────────┬─────────┘
                             │ بدء الاستلام (Start Pickup)
                             ▼
                    ┌──────────────────┐
                    │ PICKUP_IN_PROGRESS│
                    │(جاري الاستلام)    │
                    └────────┬─────────┘
                             │ تم الاستلام (Picked Up)
                             ▼
                    ┌──────────────────┐
                    │   PICKED_UP      │
                    │  (تم الاستلام)    │
                    └────────┬─────────┘
                             │ بدء التوصيل (Start Transit)
                             ▼
                    ┌──────────────────┐
                    │   IN_TRANSIT     │
                    │  (في الطريق)     │
                    └────────┬─────────┘
                             │ الوصول (Arrived)
                             ▼
                    ┌──────────────────┐
                    │ARRIVED_AT_CUSTOMER│
                    │ (وصل إلى العميل) │
                    └────────┬─────────┘
                             │ إدخال رمز التوصيل (Enter Code)
                             ▼
                    ┌──────────────────┐
                    │    DELIVERED     │
                    │  (تم التوصيل)    │
                    └──────────────────┘

     ╔══════════════════════════════════════════╗
     ║  On Failure (at ARRIVED_AT_CUSTOMER):    ║
     ║  → DELIVERY_ATTEMPT_FAILED               ║
     ║  → After 3 failures: ACCOUNT LOCKED 24h  ║
     ╚══════════════════════════════════════════╝
```

### 7.6 Interactions

| Trigger | Action | Notes |
|---|---|---|
| Tap "تحديث الحالة" | Navigate to DP-AC-002 | Status update flow |
| Tap "التفاصيل" | Navigate to DP-AS-002 (detail view) | Read-only of accepted order |
| Pull-to-refresh | Refresh active list | Spinner |
| Swipe left on card | Quick action: View details | — |

### 7.7 Empty State

| Property | Value |
|---|---|
| Icon | CheckCircle (outline) |
| Title | `لا توجد طلبات نشطة` |
| Description | `ستظهر الطلبات المقبولة هنا` |

### 7.8 Loading/Error States

| State | Display |
|---|---|
| Loading | Skeleton: 3 list items |
| Network error | Toast: `تحقق من اتصالك بالإنترنت` |

### 7.9 Accessibility

| Requirement | Implementation |
|---|---|
| Screen reader | Card announces: `طلب #{id}، {status}، من {pickup} إلى {delivery}` |
| Keyboard | Tab: cards → actions |
| Touch target | 44px on buttons |

---

## 8. DP-AC-002 — Delivery In-Progress

### 8.1 Page Metadata

| Field | Value |
|---|---|
| Page ID | DP-AC-002 |
| Module | M04 — Active |
| Type | Page (PG) |
| Priority | P0 |
| FR Ref | FR-015 |
| Building Blocks | B01, B05, B07, B08, B11 |
| Constraints | DP-001, DP-002, DP-005, DP-006, DP-007 |

### 8.2 Purpose

Detailed view of a single active delivery. Provides status update actions, delivery code verification (final step), and customer contact options. This is the primary working screen during a delivery.

### 8.3 Layout

**Mobile (360px):**

```
+-----------------------------------------------+
|  [Back]   توصيل الطلب #١٢٤  ● قيد التوصيل   |
+-----------------------------------------------+
|                                               |
|  ────── ○ ─── ○ ──── ◉ ────── ○ ───         |
|  استلام  │  │   في الطريق  │  الوصول  │       |
|                                               |
+-----------------------------------------------+
|  📍 مسارات التوصيل                            |
|  ┌─────────────────────────────────┐         |
│  │  🟢 صنعاء، حي التvil (الاستلام) │         |
│  │  ═══════════════════════════     │         |
│  │  🔴 صنعاء، حي النزهة (التوصيل) │         |
│  └─────────────────────────────────┘         |
|                                               |
+-----------------------------------------------+
|  👤 العميل: أحمد محمد                        |
|  📞 77X XXX XXXX                             |
|  [📞 اتصال] [💬 واتساب]                      |
|                                               |
+-----------------------------------------------+
|  📦 المنتجات (٣ منتجات)                      |
|  ┌─────────────────────────────────┐         |
│  │  📱 آيفون 15 — ×١              │         |
│  │  🎧 سماعات — ×٢                │         |
│  └─────────────────────────────────┘         |
|                                               |
+-----------------------------------------------+
|  ⚡ تحديث الحالة                              |
|                                               |
|  الحالة الحالية: في الطريق                    |
|                                               |
|  ما هي خطوتك التالية؟                        |
|                                               |
|  [وصلت إلى العميل]                           |
|                                               |
|  [محҫّلت في التوصيل — أدخل الرمز]            |
|                                               |
+-----------------------------------------------+
|  💰 أجرة التوصيل: ٥٠٠ ر.ي                    |
+-----------------------------------------------+
```

### 8.4 Components

#### 8.4.1 Progress Stepper

| Property | Value |
|---|---|
| Component | DeliveryProgressStepper |
| Steps | استلام → في الطريق → الوصول → التوصيل |
| Current Step | Highlighted (navy-800), filled circle |
| Completed Steps | Green checkmark |
| Upcoming Steps | Gray, outline circle |
| RTL | Steps flow right-to-left |
| Visual | Horizontal line connecting circles |

#### 8.4.2 Delivery Points Card

| Property | Value |
|---|---|
| Component | DeliveryPointsCard |
| Content | Pickup and delivery addresses |
| Visual | Same as DP-AS-002 |

#### 8.4.3 Customer Contact Card

| Property | Value |
|---|---|
| Component | CustomerContactCard |
| Content | Name, phone (masked until accepted), action buttons |
| Actions | Call (`tel:`), WhatsApp (`whatsapp://`) |
| Privacy | Phone fully visible only after order is in transit |

#### 8.4.4 Items List

| Property | Value |
|---|---|
| Component | ItemsList |
| Content | Product name, quantity |
| Style | Compact list |

#### 8.4.5 Status Update Actions

| Current State | Available Actions |
|---|---|
| ACCEPTED | `جاري الاستلام` (Start Pickup) |
| PICKUP_IN_PROGRESS | `تم الاستلام` (Picked Up) |
| PICKED_UP | `بدء التوصيل` (Start Transit) |
| IN_TRANSIT | `وصلت إلى العميل` (Arrived) |
| ARRIVED_AT_CUSTOMER | `إدخال رمز التوصيل` (Enter Code) → DP-CD-001 |

| Property | Value |
|---|---|
| Component | StatusActionButton |
| Style | Primary button, full width |
| Loading | Spinner during API call |
| Success | Status updates, stepper advances |
| Error | Toast with retry |

#### 8.4.6 Failed Attempt Action

| Property | Value |
|---|---|
| Component | FailedAttemptButton |
| Content | `محҫّلت في التوصيل — فشل التوصيل` |
| Style | Red text, outline button |
| Action | Opens failed attempt modal |
| Constraint | DP-002 (3-Attempt Lockout) |

### 8.5 Failed Attempt Modal

```
+-----------------------------------------------+
|  تسجيل فشل التوصيل                           |
|  ───────────────────────────────────────       |
|  سبب الفشل:                                   |
|  ┌─────────────────────────────────┐         |
|  │  اختر السبب...                   ▼         |
|  └─────────────────────────────────┘         |
|                                               |
|  ملاحظات (اختياري):                           |
|  ┌─────────────────────────────────┐         |
│  │                                   │         |
│  └─────────────────────────────────┘         |
|                                               |
|  ⚠️ محاولة ١ من ٣ — بعد ٣ محاولات فاشلة     |
|  سيتم تعليق حسابك لمدة ٢٤ ساعة              |
|                                               |
|  [إلغاء]          [تسجيل الفشل]              |
+-----------------------------------------------+
```

| Property | Value |
|---|---|
| Title | `تسجيل فشل التوصيل` |
| Reason Options | `العميل غير متواجد`، `العنوان خاطئ`، `رفض الاستلام`، `لا يمكن الوصول`، `أخرى` |
| Notes Field | Optional textarea |
| Warning | Shows current attempt count and lockout warning |
| Constraint | DP-002: After 3 failures → 24h freeze |
| Destructive | Yes (red-500 button) |
| Confirm Action | POST `/api/delivery/failed-attempt` |
| Success | Return to DP-AC-001, toast: `تم تسجيل الفشل` |
| Lockout Trigger | If `failed_attempts >= 3`: redirect to DP-DB-001 with frozen banner |

### 8.6 Lockout Banner (After 3 Failed Attempts)

```
┌─────────────────────────────────────────┐
│  🔒 حسابك مقفل مؤقتًا                   │
│  ─────────────────────────────────────   │
│  بسبب ٣ محاولات توصيل فاشلة متتالية،    │
│  تم تعليق حسابك لمدة ٢٤ ساعة.          │
│                                           │
│  سيتم إعادة التفعيل في:                   │
│  ٢٣ سبتمبر ٢٠٢٦، ٢:٣٠ م                 │
│                                           │
│  لا يمكنك قبول طلبات جديدة أثناء القفل.  │
└─────────────────────────────────────────┘
```

| Property | Value |
|---|---|
| Component | LockoutBanner |
| Style | Red-50 bg, red-800 text, red-200 border |
| Duration | 24 hours from last failure |
| Behavior | Read-only mode, no accept/reject actions |
| Timer | Countdown displayed |

### 8.7 Interactions

| Trigger | Action | Notes |
|---|---|---|
| Tap status action | Update delivery status | API call, UI update |
| Tap "فشل التوصيل" | Open failed attempt modal | — |
| Tap call button | Open dialer | Native |
| Tap WhatsApp | Open WhatsApp | Native |
| Tap back | Confirm if in-progress | `هل تريد المغادرة؟ الحالة لن تُحفظ` |

### 8.8 Loading/Error States

| State | Display |
|---|---|
| Loading | Skeleton for all sections |
| Status update loading | Button spinner, disable other actions |
| Status update error | Toast: `لم يتم التحديث` with retry |
| Network error | Toast: `تحقق من اتصالك بالإنترنت` |

### 8.9 Accessibility

| Requirement | Implementation |
|---|---|
| Screen reader | Stepper announces current step and progress |
| Keyboard | Tab: stepper → addresses → customer → items → actions |
| Focus | Visible ring on buttons |
| Touch target | 44px minimum on all buttons |

---

## 9. DP-CD-001 — Delivery Code Verification

### 9.1 Page Metadata

| Field | Value |
|---|---|
| Page ID | DP-CD-001 |
| Module | M05 — Code |
| Type | Flow (FL) |
| Priority | P1 |
| FR Ref | FR-015 |
| Building Blocks | B01, B07, B09 |
| Constraints | DP-001, DP-006, DP-007 |

### 9.2 Purpose

Final verification step for delivery completion. The customer receives a delivery code via SMS/WhatsApp and provides it to the delivery provider. The provider enters the code to confirm successful delivery. **This is the substitute for GPS-based proof of delivery.**

### 9.3 Layout

**Mobile (360px):**

```
+-----------------------------------------------+
|  [Back]   رمز التوصيل                         |
+-----------------------------------------------+
|                                               |
|  ┌─────────────────────────────────┐         |
│  │  ✅ الخطوة الأخيرة!             │         |
│  │                                  │         |
│  │  اطلب من العميل رمز التوصيل     │         |
│  │  المرسل عبر الرسائل القصيرة     │         |
│  │  أو واتساب.                     │         |
│  └─────────────────────────────────┘         |
|                                               |
+-----------------------------------------------+
|  أدخل رمز التوصيل                            |
|                                               |
|  ┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐      |
|  │   │ │   │ │   │ │   │ │   │ │   │      |
|  └───┘ └───┘ └───┘ └───┘ └───┘ └───┘      |
|                                               |
|  أعد الإرسال خلال ٣٠ ثانية                    |
|                                               |
+-----------------------------------------------+
|                                               |
|  ┌─────────────────────────────────┐         |
|  │  [تأكيد التوصيل]                │         |
│  └─────────────────────────────────┘         |
|                                               |
|  ┌─────────────────────────────────┐         |
│  │  ❌ رمز غير صحيح — حاول مجددًا  │         |
│  └─────────────────────────────────┘         |
|                                               |
+-----------------------------------------------+
```

### 9.4 Components

#### 9.4.1 Instruction Card

| Property | Value |
|---|---|
| Component | InstructionCard |
| Content | Instructions to ask customer for the delivery code |
| Icon | CheckCircle (green) |
| Style | Green-50 bg, rounded-lg |

#### 9.4.2 Code Input

| Property | Value |
|---|---|
| Component | CodeInput |
| Type | 6-digit OTP-style input |
| Input Mode | `numeric` |
| Dir | `ltr` (always LTR for codes) |
| Auto-advance | Yes (move to next box on digit entry) |
| Auto-submit | Yes (submit when all 6 digits entered) |
| Style | Individual boxes, 48px × 48px each |
| Max Length | 6 digits |
| Keyboard | Numeric pad |

#### 9.4.3 Resend Timer

| Property | Value |
|---|---|
| Component | ResendTimer |
| Initial Countdown | 30 seconds |
| Display | `أعد الإرسال خلال {seconds} ثانية` |
| After Expiry | `إعادة إرسال الرمز` (clickable) |
| Style | Gray-500 text |

#### 9.4.4 Submit Button

| Property | Value |
|---|---|
| Component | SubmitButton |
| Content | `تأكيد التوصيل` |
| Style | Primary, navy-800, full width |
| Disabled | Until 6 digits entered |
| Loading | Spinner during verification |

#### 9.4.5 Error Message

| Property | Value |
|---|---|
| Component | ErrorMessage |
| Content | `رمز غير صحيح — حاول مجددًا` |
| Style | Red-600 text, icon (alert circle) |
| Attempts | Max 5 attempts per code |
| After 5 attempts | `لقد تجاوزت الحد الأقصى من المحاولات` |

### 9.5 Verification Flow

```
Delivery Provider at ARRIVED_AT_CUSTOMER
    │
    ▼
Ask customer for code (verbal/phone)
    │
    ▼
Customer checks SMS/WhatsApp for 6-digit code
    │
    ▼
Customer provides code verbally
    │
    ▼
Delivery Provider enters code in DP-CD-001
    │
    ├─ Code correct → DELIVERED ✅
    │   └─ Earnings credited to wallet
    │   └─ Customer prompted to rate
    │
    └─ Code incorrect → Error message
        └─ Retry (max 5 attempts)
        └─ After 5: Contact support
```

### 9.6 API Interaction

| Action | Endpoint | Method | Payload |
|---|---|---|---|
| Verify code | `/api/delivery/verify-code` | POST | `{ orderId, code }` |
| Resend code | `/api/delivery/resend-code` | POST | `{ orderId }` |

### 9.7 Success State

```
+-----------------------------------------------+
|                                               |
|              ✅ تم التوصيل بنجاح!             |
|                                               |
|  تم إضافة ٥٠٠ ر.ي إلى محفظتك                 |
|                                               |
|  الرصيد الحالي: ٣٬٥٠٠ ر.ي                    |
|                                               |
|  ┌─────────────────────────────────┐         |
│  │  [العودة إلى الطلبات النشطة]   │         |
│  └─────────────────────────────────┘         |
|                                               |
|  ┌─────────────────────────────────┐         |
│  │  [تقييم العميل]                 │         |
│  └─────────────────────────────────┘         |
|                                               |
+-----------------------------------------------+
```

| Property | Value |
|---|---|
| Component | SuccessScreen |
| Content | Success message, earnings added, wallet balance |
| Actions | "العودة إلى الطلبات النشطة" → DP-AC-001, "تقييم العميل" → DP-RT-001 |
| Auto-redirect | After 5 seconds → DP-AC-001 |
| Haptic | Short vibration (success) |

### 9.8 Error States

| Error | Display | Action |
|---|---|---|
| Wrong code | `رمز غير صحيح — حاول مجددًا` | Clear input, allow retry |
| Max attempts exceeded | `لقد تجاوزت الحد الأقصى من المحاولات` | Redirect to support |
| Network error | `تحقق من اتصالك بالإنترنت` | Retry button |
| Code expired | `انتهت صلاحية الرمز — اطلب رمزًا جديدًا` | Resend button |

### 9.9 Accessibility

| Requirement | Implementation |
|---|---|
| Screen reader | Each digit box announced: `رقم {position} من ٦` |
| Keyboard | Numeric input, auto-advance |
| Focus | Auto-focus first empty box |
| Touch target | 48px × 48px per digit box |
| Error announcement | `role="alert"` on error message |

### 9.10 RTL/LTR Behavior

| Element | RTL | LTR |
|---|---|---|
| Code input boxes | Same (always LTR for codes) | Same |
| Instruction text | RTL | LTR |

---

## 10. DP-HS-001 — Delivery History

### 10.1 Page Metadata

| Field | Value |
|---|---|
| Page ID | DP-HS-001 |
| Module | M06 — History |
| Type | Page (PG) |
| Priority | P1 |
| FR Ref | FR-015 |
| Building Blocks | B01, B05, B08, B10, B11 |
| Constraints | DP-003, DP-007 |

### 10.2 Purpose

Complete history of all past deliveries — successful, failed, and cancelled. Provides filtering, search, and detailed view of each historical delivery.

### 10.3 Layout

**Mobile (360px):**

```
+-----------------------------------------------+
|  [Back]   سجل التوصيلات              🔍  📊  |
+-----------------------------------------------+
|  ┌─────────────────────────────────┐         |
|  │  🔍 ابحث في السجل...             │         |
|  └─────────────────────────────────┘         |
|                                               |
|  [الكل] [مكتملة] [فاشلة] [ملغاة]           |
|                                               |
|  ┌─────────────────────────────────┐         |
|  │  من ١ سبتمبر إلى ١٣ سبتمبر     │         |
│  │  [اختيار الفترة]                 │         |
│  └─────────────────────────────────┘         |
|                                               |
|  ١٢ طلب — ١٠ مكتمل — ١ فاشل — ١ ملغي     |
|                                               |
|  ┌─────────────────────────────────┐         |
│  │  #١٢٣٤  ✅ تم التوصيل           │         |
│  │  ١٣ سبتمبر ٢٠٢٦  ٢:٣٠ م       │         |
│  │  صنعاء → صنعاء  ٥٠٠ ر.ي        │         |
│  │  ⭐ ٤٫٨ تقييم                   │         |
│  └─────────────────────────────────┘         |
|                                               |
|  ┌─────────────────────────────────┐         |
│  │  #١٢٣٥  ❌ فشل التوصيل          │         |
│  │  ١٢ سبتمبر ٢٠٢٦  ١٠:١٥ ص      │         |
│  │  عدن → عدن  ٠ ر.ي               │         |
│  │  السبب: العميل غير متواجد       │         |
│  └─────────────────────────────────┘         |
|                                               |
|  ┌─────────────────────────────────┐         |
│  │  #١٢٣٦  ✅ تم التوصيل           │         |
│  │  ١١ سبتمبر ٢٠٢٦  ٤:٤٥ م       │         |
│  │  صنعاء → صنعاء  ٤٥٠ ر.ي        │         |
│  │  ⭐ ٤٫٥ تقييم                   │         |
│  └─────────────────────────────────┘         |
|                                               |
|  صفحات: ١  ٢  ٣  [التالي]                  |
|                                               |
+-----------------------------------------------+
|  🏠     📦      ✅     💰      👤          |
| الرئيسية المتاح  النشطة  الأرباح  حسابي    |
+-----------------------------------------------+
```

### 10.4 Components

#### 10.4.1 Search Bar

| Property | Value |
|---|---|
| Component | SearchBar |
| Placeholder | `ابحث في السجل...` |
| Search By | Order ID, address, customer name |

#### 10.4.2 Filter Tabs

| Property | Value |
|---|---|
| Component | FilterTabs |
| Options | `الكل` (All), `مكتملة` (Completed), `فاشلة` (Failed), `ملغاة` (Cancelled) |
| Default | `الكل` |

#### 10.4.3 Date Range Picker

| Property | Value |
|---|---|
| Component | DateRangePicker |
| Default Range | Last 30 days |
| Format | `من {start} إلى {end}` |
| Style | Compact, collapsible |

#### 10.4.4 Summary Stats

| Property | Value |
|---|---|
| Component | SummaryStats |
| Content | Total deliveries, completed, failed, cancelled |
| Style | Inline text: `١٢ طلب — ١٠ مكتمل — ١ فاشل — ١ ملغي` |

#### 10.4.5 History Card

| Property | Value |
|---|---|
| Component | HistoryCard |
| Content | Order ID, status, date/time, route, earnings, rating |
| Status | Colored badge (green for completed, red for failed, gray for cancelled) |
| Tap Action | Navigate to delivery detail view |

#### 10.4.6 Pagination

| Property | Value |
|---|---|
| Component | Pagination |
| Items Per Page | 20 |
| Style | Page numbers + prev/next |

### 10.5 History Card Data Fields

| Field | Source | Display |
|---|---|---|
| Order ID | `order.id` | `#١٢٣٤` |
| Status | `delivery.status` | Badge |
| Date | `delivery.completed_at` | `١٣ سبتمبر ٢٠٢٦` |
| Time | `delivery.completed_at` | `٢:٣٠ م` |
| Pickup | `order.pickup_address.city` | `صنعاء` |
| Delivery | `order.delivery_address.city` | `صنعاء` |
| Earnings | `delivery.earnings` | `٥٠٠ ر.ي` |
| Rating | `delivery.rating` | `⭐ ٤٫٨` |
| Failure Reason | `delivery.failure_reason` | `العميل غير متواجد` |

### 10.6 Interactions

| Trigger | Action | Notes |
|---|---|---|
| Tap history card | Navigate to delivery detail | Read-only view |
| Tap filter tab | Filter list | Active state update |
| Tap date range | Open date picker | Bottom sheet |
| Search input | Filter by ID/address | Debounced |
| Pull-to-refresh | Refresh history | Spinner |
| Page change | Load next page | Pagination |

### 10.7 Empty States

| Scenario | Display |
|---|---|
| No history | Icon (clock) + `لا يوجد سجل توصيلات` |
| No results for filter | `لا توجد نتائج لهذا الفلتر` |
| No results for search | `لا توجد نتائج لـ "{query}"` |

### 10.8 Loading/Error States

| State | Display |
|---|---|
| Loading | Skeleton: 5 list items |
| Network error | Toast: `تحقق من اتصالك بالإنترنت` |

### 10.9 Accessibility

| Requirement | Implementation |
|---|---|
| Screen reader | Card announces: `طلب #{id}، {status}، {date}، {earnings} ر.ي` |
| Keyboard | Tab: search → filters → date → cards → pagination |
| Touch target | 44px on cards and buttons |

---

## 11. DP-FN-001 — Earnings Dashboard

### 11.1 Page Metadata

| Field | Value |
|---|---|
| Page ID | DP-FN-001 |
| Module | M07 — Finance |
| Type | Page (PG) |
| Priority | P1 |
| FR Ref | FR-015 |
| Building Blocks | B01, B06, B08, B11 |
| Constraints | DP-003, DP-007 |

### 11.2 Purpose

Overview of earnings, wallet balance, and financial summary. Delivery providers can see their current balance, earnings breakdown by period, and access withdrawal options.

### 11.3 Layout

**Mobile (360px):**

```
+-----------------------------------------------+
|  [Back]   الأرباح والمحفظة           📊  ⚙️  |
+-----------------------------------------------+
|                                               |
|  ┌─────────────────────────────────┐         |
│  │  💰 رصيد المحفظة                │         |
│  │  ٣٬٥٠٠ ر.ي                      │         |
│  │                                  │         |
│  │  [سحب الأرباح]                  │         |
│  └─────────────────────────────────┘         |
|                                               |
+-----------------------------------------------+
|  📊 ملخص الأرباح                             |
|                                               |
|  [اليوم] [هذا الأسبوع] [هذا الشهر]          |
|                                               |
|  ┌──────────┐  ┌──────────┐                  |
│  │ ٢٬٥٠٠   │  │ ١٢٬٠٠٠  │                  |
│  │ ر.ي      │  │ ر.ي      │                  |
│  │ أرباح    │  │ أرباح    │                  |
│  │ اليوم    │  │ الأسبوع  │                  |
│  └──────────┘  └──────────┘                  |
|                                               |
+-----------------------------------------------+
|  📈 رسم بياني — أرباح آخر ٧ أيام            |
|  ┌─────────────────────────────────┐         |
│  │  ███████                        │         |
│  │  ████████████                   │         |
│  │  ████████████████               │         |
│  │  ██████████████████████         │         |
│  │  ████                           │         |
│  │  ████████████                   │         |
│  │  ██████████████                 │         |
│  │  ─────────────────────────────── │         |
│  │  سبت أحد اثنين ثلاثاء أربعاء خمس │         |
│  └─────────────────────────────────┘         |
|                                               |
+-----------------------------------------------+
|  📋 تفاصيل الأرباح                           |
|  ┌─────────────────────────────────┐         |
│  │  إجمالي التوصيلات    ١٥ طلب    │         |
│  │  متوسط الأرباح/طلب   ٥٠٠ ر.ي  │         |
│  │  أعلى أرباح/طلب      ٧٥٠ ر.ي  │         |
│  │  إجمالي الأرباح      ٧٬٥٠٠ ر.ي│         |
│  └─────────────────────────────────┘         |
|                                               |
+-----------------------------------------------+
|  🏠     📦      ✅     💰      👤          |
| الرئيسية المتاح  النشطة  الأرباح  حسابي    |
+-----------------------------------------------+
```

### 11.4 Components

#### 11.4.1 Wallet Balance Card

| Property | Value |
|---|---|
| Component | WalletBalanceCard |
| Content | Current balance, withdraw button |
| Style | Gradient bg (navy-800 to navy-900), white text |
| Value | `٣٬٥٠٠ ر.ي` |
| Button | "سحب الأرباح" → withdrawal flow |

#### 11.4.2 Period Tabs

| Property | Value |
|---|---|
| Component | PeriodTabs |
| Options | `اليوم` (Today), `هذا الأسبوع` (This Week), `هذا الشهر` (This Month) |
| Default | `اليوم` |
| Style | Pill-shaped tabs |

#### 11.4.3 Earnings Summary Cards

| Property | Value |
|---|---|
| Component | EarningsSummaryCards |
| Layout | 2-column grid |
| Cards | Today's earnings, This week's earnings |
| Style | White bg, rounded-lg, shadow-sm |

#### 11.4.4 Earnings Chart

| Property | Value |
|---|---|
| Component | EarningsChart |
| Type | Bar chart (last 7 days) |
| Data | Daily earnings |
| X-Axis | Days (Arabic abbreviations) |
| Y-Axis | Earnings (YER) |
| Color | Blue-500 bars |
| Responsive | Full width, scrollable on mobile |

#### 11.4.5 Earnings Detail Table

| Property | Value |
|---|---|
| Component | EarningsDetailTable |
| Rows | Total deliveries, Average per delivery, Highest single delivery, Total earnings |
| Style | List items with labels and values |

### 11.5 Interactions

| Trigger | Action | Notes |
|---|---|---|
| Tap period tab | Update stats and chart | API call with date range |
| Tap "سحب الأرباح" | Navigate to withdrawal flow | — |
| Pull-to-refresh | Refresh earnings data | Spinner |
| Tap chart bar | Show tooltip with exact value | — |

### 11.6 Loading/Error States

| State | Display |
|---|---|
| Loading | Skeleton: balance card, stats, chart placeholder |
| Network error | Toast: `تحقق من اتصالك بالإنترنت` |
| Zero earnings | `لم تسجل أي أرباح في هذه الفترة` |

### 11.7 Accessibility

| Requirement | Implementation |
|---|---|
| Screen reader | Chart has accessible data table fallback |
| Keyboard | Tab: balance → tabs → stats → chart → details |
| Touch target | 44px on all interactive elements |

---

## 12. DP-FN-002 — Transaction History

### 12.1 Page Metadata

| Field | Value |
|---|---|
| Page ID | DP-FN-002 |
| Module | M07 — Finance |
| Type | Page (PG) |
| Priority | P1 |
| FR Ref | FR-015 |
| Building Blocks | B01, B06, B08, B10, B11 |
| Constraints | DP-003, DP-007 |

### 12.2 Purpose

Detailed log of all wallet transactions — earnings credited, withdrawals, and adjustments. Each transaction shows date, type, amount, and balance after.

### 12.3 Layout

**Mobile (360px):**

```
+-----------------------------------------------+
|  [Back]   سجل المعاملات              🔍  📅  |
+-----------------------------------------------+
|  ┌─────────────────────────────────┐         |
|  │  🔍 ابحث في المعاملات...        │         |
|  └─────────────────────────────────┘         |
|                                               |
|  [الكل] [أرباح] [سحوبات] [تعديلات]         |
|                                               |
|  ┌─────────────────────────────────┐         |
│  │  ١٣ سبتمبر ٢٠٢٦                │         |
│  │  ✅ أرباح توصيل #١٢٣٤   +٥٠٠ ر.ي│       |
│  │  الرصيد: ٣٬٥٠٠ ر.ي             │         |
│  └─────────────────────────────────┘         |
|                                               |
|  ┌─────────────────────────────────┐         |
│  │  ١٣ سبتمبر �٢٠٢٦                │         |
│  │  ✅ أرباح توصيل #١٢٣٥   +٤٥٠ ر.ي│       |
│  │  الرصيد: ٣٬٠٠٠ ر.ي             │         |
│  └─────────────────────────────────┘         |
|                                               |
|  ┌─────────────────────────────────┐         |
│  │  ١٢ سبتمبر ٢٠٢٦                │         |
│  │  💳 سحب إلى محفظة خارجية -٢٬٠٠٠ ر.ي│   |
│  │  الرصيد: ٢٬٥٥٠ ر.ي             │         |
│  └─────────────────────────────────┘         |
|                                               |
|  صفحات: ١  ٢  ٣  [التالي]                  |
|                                               |
+-----------------------------------------------+
|  🏠     📦      ✅     💰      👤          |
| الرئيسية المتاح  النشطة  الأرباح  حسابي    |
+-----------------------------------------------+
```

### 12.4 Components

#### 12.4.1 Search Bar

| Property | Value |
|---|---|
| Component | SearchBar |
| Placeholder | `ابحث في المعاملات...` |

#### 12.4.2 Filter Tabs

| Property | Value |
|---|---|
| Component | FilterTabs |
| Options | `الكل`, `أرباح` (Earnings), `سحوبات` (Withdrawals), `تعديلات` (Adjustments) |

#### 12.4.3 Transaction Group (by Date)

| Property | Value |
|---|---|
| Component | TransactionGroup |
| Content | Date header, list of transactions |
| Date Format | `١٣ سبتمبر ٢٠٢٦` |
| Grouping | By day, newest first |

#### 12.4.4 Transaction Item

| Property | Value |
|---|---|
| Component | TransactionItem |
| Content | Type icon, description, amount (+/-), running balance |
| Amount Color | Green for credits (`+٥٠٠ ر.ي`), Red for debits (`-٢٬٠٠٠ ر.ي`) |
| Balance | `الرصيد: {balance} ر.ي` |

#### 12.4.5 Pagination

| Property | Value |
|---|---|
| Component | Pagination |
| Items Per Page | 20 |

### 12.5 Transaction Types

| Type | Icon | Color | Description |
|---|---|---|---|
| Earning | CheckCircle | Green | Earnings from completed delivery |
| Withdrawal | CreditCard | Red | Withdrawal to external wallet |
| Adjustment | AlertCircle | Amber | Admin adjustment (bonus/penalty) |
| Refund | RotateCcw | Blue | Refunded delivery fee |

### 12.6 Interactions

| Trigger | Action | Notes |
|---|---|---|
| Tap transaction | Show detail bottom sheet | — |
| Tap filter tab | Filter by type | — |
| Search | Filter by description or order ID | Debounced |
| Pull-to-refresh | Refresh transactions | Spinner |
| Page change | Load next page | — |

### 12.7 Transaction Detail Bottom Sheet

```
+-----------------------------------------------+
|  تفاصيل المعاملة                              |
|  ───────────────────────────────────────       |
|  النوع: أرباح توصيل                           |
|  الطلب: #١٢٣٤                                |
|  التاريخ: ١٣ سبتمبر ٢٠٢٦، ٢:٣٠ م            |
|  المبلغ: +٥٠٠ ر.ي                             |
|  الرصيد بعد المعاملة: ٣٬٥٠٠ ر.ي              |
|                                               |
|  [إغلاق]                                      |
+-----------------------------------------------+
```

### 12.8 Loading/Error States

| State | Display |
|---|---|
| Loading | Skeleton: 5 transaction items |
| Network error | Toast: `تحقق من اتصالك بالإنترنت` |
| Empty state | `لا توجد معاملات` |

### 12.9 Accessibility

| Requirement | Implementation |
|---|---|
| Screen reader | Transaction announces: `{type}، {description}، {amount}، الرصيد {balance} ر.ي` |
| Keyboard | Tab: search → filters → transactions → pagination |
| Touch target | 44px on items |

---

## 13. DP-PR-001 — Profile Settings

### 13.1 Page Metadata

| Field | Value |
|---|---|
| Page ID | DP-PR-001 |
| Module | M08 — Profile |
| Type | Page (PG) |
| Priority | P2 |
| FR Ref | FR-008 |
| Building Blocks | B01, B07, B09, B11 |
| Constraints | DP-004, DP-007 |

### 13.2 Purpose

Delivery provider profile management — view and edit personal information, change phone number, manage notification preferences, language settings, and account actions.

### 13.3 Layout

**Mobile (360px):**

```
+-----------------------------------------------+
|  [Back]   إعدادات الحساب                      |
+-----------------------------------------------+
|                                               |
|  ┌─────────────────────────────────┐         |
│  │         [صورة الملف الشخصي]      │         |
│  │              أحمد محمد           │         |
│  │         77X XXX XXXX             │         |
│  │         📍 صنعاء                 │         |
│  └─────────────────────────────────┘         |
|                                               |
+-----------------------------------------------+
|  المعلومات الشخصية                           |
│  ├─  الاسم الكامل        أحمد محمد     [تعديل]│
│  ├─  رقم الهاتف          77X XXX XXXX  [تعديل]│
│  ├─  المدينة              صنعاء        [تعديل]│
│  └─  المنطقة              حي التvil     [تعديل]│
+-----------------------------------------------+
|  إعدادات التطبيق                             │
│  ├─  اللغة               العربية       [▶]   │
│  ├─  الإشعارات           مفعّلة        [▶]   │
│  └─  الوضع الليلي        معطّل        [▶]   │
+-----------------------------------------------+
|  الحساب                                       │
│  ├─  سجل التوصيلات                   [▶]    │
│  ├─  التقييم والأداء                  [▶]    │
│  ├─  سياسة الخصوصية                  [▶]    │
│  └─  شروط الاستخدام                   [▶]    │
+-----------------------------------------------+
|                                               |
|  ┌─────────────────────────────────┐         |
|  │  🚪 تسجيل الخروج                │         |
│  └─────────────────────────────────┘         |
|                                               |
|  ┌─────────────────────────────────┐         |
│  │  🗑️ حذف الحساب                  │         |
│  └─────────────────────────────────┘         |
|                                               |
|  الإصدار: ١٫٠٫٠                              |
|                                               |
+-----------------------------------------------+
|  🏠     📦      ✅     💰      👤          |
| الرئيسية المتاح  النشطة  الأرباح  حسابي    |
+-----------------------------------------------+
```

### 13.4 Components

#### 13.4.1 Profile Header

| Property | Value |
|---|---|
| Component | ProfileHeader |
| Content | Avatar (circle), name, phone, city |
| Avatar | Camera icon overlay for upload |
| Tap Avatar | Open image picker |

#### 13.4.2 Personal Info Section

| Property | Value |
|---|---|
| Component | SettingsSection |
| Title | `المعلومات الشخصية` |
| Items | Full name, phone number, city, zone |
| Each Item | Label + value + edit button |
| Edit Action | Opens inline edit or bottom sheet |

#### 13.4.3 App Settings Section

| Property | Value |
|---|---|
| Component | SettingsSection |
| Title | `إعدادات التطبيق` |
| Items | Language, Notifications toggle, Dark mode |

#### 13.4.4 Account Section

| Property | Value |
|---|---|
| Component | SettingsSection |
| Title | `الحساب` |
| Items | Delivery history, Rating/Performance, Privacy policy, Terms of use |

#### 13.4.5 Logout Button

| Property | Value |
|---|---|
| Component | LogoutButton |
| Content | `تسجيل الخروج` |
| Style | Red text, outline |
| Action | Confirmation modal |

#### 13.4.6 Delete Account Button

| Property | Value |
|---|---|
| Component | DeleteAccountButton |
| Content | `حذف الحساب` |
| Style | Red text, outline |
| Action | Confirmation modal (destructive) |

### 13.5 Edit Profile Bottom Sheet

```
+-----------------------------------------------+
|  تعديل الملف الشخصي                           |
|  ───────────────────────────────────────       |
|  الاسم الكامل *                                |
│  ┌─────────────────────────────────┐         │
│  │  أحمد محمد                       │         │
│  └─────────────────────────────────┘         │
|                                               |
|  المدينة *                                    |
│  ┌─────────────────────────────────┐         │
│  │  صنعاء                          │         │
│  └─────────────────────────────────┘         │
|                                               |
|  المنطقة *                                   |
│  ┌─────────────────────────────────┐         │
│  │  حي التvil                       │         │
│  └─────────────────────────────────┘         │
|                                               |
|  [إلغاء]              [حفظ التعديلات]       |
+-----------------------------------------------+
```

### 13.6 Interactions

| Trigger | Action | Notes |
|---|---|---|
| Tap edit button | Open edit bottom sheet | Pre-fill current values |
| Tap phone number | Open change phone flow | OTP verification required |
| Tap language | Open language picker | Arabic/English toggle |
| Tap notifications | Toggle switch | Immediate save |
| Tap logout | Show confirmation modal | `هل تريد تسجيل الخروج؟` |
| Tap delete account | Show destructive confirmation | Requires OTP |
| Tap delivery history | Navigate to DP-HS-001 | — |
| Tap rating/performance | Navigate to DP-RT-001 | — |

### 13.7 Change Phone Flow

```
1. Enter new phone number (阿拉伯)
2. Receive OTP on new number
3. Enter OTP (6 digits)
4. Phone updated
5. Session invalidated, re-login required
```

### 13.8 Logout Confirmation Modal

| Property | Value |
|---|---|
| Title | `تسجيل الخروج` |
| Message | `هل تريد تسجيل الخروج من حسابك؟` |
| Primary Button | `تسجيل الخروج` (red-500) |
| Secondary Button | `إلغاء` (gray) |
| Action | Clear session, navigate to login |

### 13.9 Delete Account Modal

| Property | Value |
|---|---|
| Title | `حذف الحساب` |
| Message | `هل أنت متأكد من حذف حسابك؟ هذا الإجراء لا يمكن التراجع عنه.` |
| Warning | `سيتم حذف جميع بياناتك وسجل توصيلاتك` |
| Primary Button | `حذف الحساب` (red-500) |
| Secondary Button | `إلغاء` (gray) |
| Verification | Requires OTP confirmation |

### 13.10 Loading/Error States

| State | Display |
|---|---|
| Loading | Skeleton: profile header, sections |
| Save success | Toast: `تم حفظ التعديلات` |
| Save error | Toast: `لم يتم الحفظ` |
| Phone change error | Toast: `لم يتم تغيير الرقم` |

### 13.11 Accessibility

| Requirement | Implementation |
|---|---|
| Screen reader | All sections and items announced |
| Keyboard | Tab through all settings items |
| Focus | Visible ring on interactive elements |
| Touch target | 44px on all buttons and list items |

---

## 14. DP-ZN-001 — Delivery Zone Management

### 14.1 Page Metadata

| Field | Value |
|---|---|
| Page ID | DP-ZN-001 |
| Module | M09 — Zone |
| Type | Page (PG) |
| Priority | P2 |
| FR Ref | FR-015 |
| Building Blocks | B01, B07, B08, B11 |
| Constraints | DP-001, DP-007 |

### 14.2 Purpose

Delivery providers can select and manage their preferred delivery zones. This affects which available deliveries they see. Zones are defined by the platform admin; providers choose which zones to serve.

### 14.3 Layout

**Mobile (360px):**

```
+-----------------------------------------------+
|  [Back]   مناطق التوصيل              🗺️     |
+-----------------------------------------------+
|                                               |
|  اختر المناطق التي تريد التوصيل فيها:       |
|  يمكنك اختيار منطقة واحدة أو أكثر.          |
|                                               |
+-----------------------------------------------+
|  🟢 صنعاء                                     |
│  ┌─────────────────────────────────┐         │
│  │  ☑️ حي التvil                     │         │
│  │  ☑️ حي النزهة                     │         │
│  │  ☐ حي الحRA                       │         │
│  │  ☐ حي السبعين                     │         │
│  └─────────────────────────────────┘         │
|                                               |
|  🟢 عدن                                       │
│  ┌─────────────────────────────────┐         │
│  │  ☑️ كريتر                        │         │
│  │  ☐ صيرة                         │         │
│  │  ☐ الشيخ عثمان                   │         │
│  └─────────────────────────────────┘         │
|                                               |
|  🟡 تعز                                      │
│  ┌─────────────────────────────────┐         │
│  │  ☐ تعز (وسط المدينة)            │         │
│  │  ☐ المعافر                      │         │
│  └─────────────────────────────────┘         │
|                                               |
| _selected_count من _total_count مناطق محددة  |
|                                               |
|  ┌─────────────────────────────────┐         │
│  │  [حفظ التغييرات]                │         │
│  └─────────────────────────────────┘         |
|                                               |
+-----------------------------------------------+
|  🏠     📦      ✅     💰      👤          |
| الرئيسية المتاح  النشطة  الأرباح  حسابي    |
+-----------------------------------------------+
```

### 14.4 Components

#### 14.4.1 Instruction Text

| Property | Value |
|---|---|
| Component | InstructionText |
| Content | `اختر المناطق التي تريد التوصيل فيها:` |

#### 14.4.2 Zone Group

| Property | Value |
|---|---|
| Component | ZoneGroup |
| Content | City name, list of sub-zones with checkboxes |
| City Header | Green icon + city name |
| Sub-Zones | Checkbox + zone name |

#### 14.4.3 Zone Checkbox

| Property | Value |
|---|---|
| Component | ZoneCheckbox |
| States | Checked (navy-800), Unchecked (gray-300) |
| Style | 24px × 24px checkbox |
| Label | Zone name (Arabic) |

#### 14.4.4 Selection Counter

| Property | Value |
|---|---|
| Content | `٣ من ١٢ منطقة محددة` |
| Position | Below zones, above save button |

#### 14.4.5 Save Button

| Property | Value |
|---|---|
| Component | SaveButton |
| Content | `حفظ التغييرات` |
| Style | Primary, navy-800, full width |
| Disabled | No changes made |
| Loading | Spinner during save |

### 14.5 Interactions

| Trigger | Action | Notes |
|---|---|---|
| Tap checkbox | Toggle zone selection | Update counter |
| Tap city header | Expand/collapse zone list | Accordion |
| Tap "حفظ التغييرات" | Save zone preferences | API call |
| Pull-to-refresh | Refresh available zones | Spinner |

### 14.6 API Interaction

| Action | Endpoint | Method | Payload |
|---|---|---|---|
| Get zones | `/api/delivery/zones` | GET | — |
| Update zones | `/api/delivery/zones` | PUT | `{ zone_ids: [1, 3, 5] }` |

### 14.7 Success/Error States

| State | Display |
|---|---|
| Save success | Toast: `تم حفظ مناطق التوصيل` |
| Save error | Toast: `لم يتم الحفظ` |
| No zones available | `لا توجد مناطق متاحة حاليًا` |
| Loading | Skeleton: 3 zone groups |

### 14.8 Accessibility

| Requirement | Implementation |
|---|---|
| Screen reader | Checkbox announces: `{zone name}، {checked/unchecked}` |
| Keyboard | Tab through checkboxes, space to toggle |
| Focus | Visible ring on checkboxes |
| Touch target | 44px on checkboxes (wrap with padding) |

---

## 15. DP-NT-001 — Notifications

### 15.1 Page Metadata

| Field | Value |
|---|---|
| Page ID | DP-NT-001 |
| Module | M10 — Notifications |
| Type | Page (PG) |
| Priority | P1 |
| FR Ref | FR-012 |
| Building Blocks | B01, B05, B08, B11 |
| Constraints | DP-007 |

### 15.2 Purpose

Notification center for delivery providers. Shows all push/in-app notifications including new delivery assignments, status updates, earnings credits, and system announcements.

### 15.3 Layout

**Mobile (360px):**

```
+-----------------------------------------------+
|  [Back]   الإشعارات                📬  ⚙️    |
+-----------------------------------------------+
|                                               |
|  [الكل] [طلبات] [أرباح] [نظام]              |
|                                               |
| 今天                                          |
|  ┌─────────────────────────────────┐         |
│  │  🔵 طلب جديد متاح               │         │
│  │  طلب #١٢٣٤ متاح للقبول          │         │
│  │  صنعاء → صنعاء  ٥٠٠ ر.ي         │         │
│  │  منذ ٥ دقائق                     │         │
│  └─────────────────────────────────┘         │
|                                               |
|  ┌─────────────────────────────────┐         │
│  │  🟢 تم إضافة أرباح              │         │
│  │  تمت إضافة ٥٠٠ ر.ي إلى محفظتك  │         │
│  │  للطلب #١٢٣٤                     │         │
│  │  منذ ٣٠ دقيقة                    │         │
│  └─────────────────────────────────┘         │
|                                               |
|  أمس                                          |
|  ┌─────────────────────────────────┐         │
│  │  🟡 تنبيه: حسابك سيقفل          │         │
│  │  محاولة فشل التوصيل الثانية     │         │
│  │  محاولة ٢ من ٣                   │         │
│  │  منذ يوم                         │         │
│  └─────────────────────────────────┘         │
|                                               |
|  ┌─────────────────────────────────┐         │
│  │  🔵 تحديث النظام                 │         │
│  │  تم تحديث شروط التوصيل          │         │
│  │  راجع التفاصيل                  │         │
│  │  منذ ٣ أيام                      │         │
│  └─────────────────────────────────┘         │
|                                               |
+-----------------------------------------------+
|  🏠     📦      ✅     💰      👤          |
| الرئيسية المتاح  النشطة  الأرباح  حسابي    |
+-----------------------------------------------+
```

### 15.4 Components

#### 15.4.1 Filter Tabs

| Property | Value |
|---|---|
| Component | FilterTabs |
| Options | `الكل` (All), `طلبات` (Deliveries), `أرباح` (Earnings), `نظام` (System) |
| Default | `الكل` |

#### 15.4.2 Notification Group

| Property | Value |
|---|---|
| Component | NotificationGroup |
| Content | Date header, list of notifications |
| Groups | `اليوم` (Today), `أمس` (Yesterday), `هذا الأسبوع` (This Week), `أقدم` (Older) |

#### 15.4.3 Notification Item

| Property | Value |
|---|---|
| Component | NotificationItem |
| Content | Type icon, title, description, timestamp |
| Icon Colors | Blue (new delivery), Green (earnings), Yellow (warning), Gray (system) |
| Unread | Bold title, blue dot indicator |
| Tap Action | Navigate to relevant screen or show detail |
| Swipe | Swipe right to mark as read |

#### 15.4.4 Empty State

| Property | Value |
|---|---|
| Component | EmptyState |
| Icon | Bell (outline) |
| Title | `لا توجد إشعارات` |
| Description | `ستظهر الإشعارات الجديدة هنا` |

### 15.5 Notification Types

| Type | Icon | Color | Title | Action |
|---|---|---|---|---|
| New Delivery | Package | Blue | `طلب جديد متاح` | Navigate to DP-AS-001 |
| Earnings Credit | CheckCircle | Green | `تم إضافة أرباح` | Navigate to DP-FN-001 |
| Warning | AlertTriangle | Yellow | `تنبيه` | Show detail |
| Lockout | Lock | Red | `تم تعليق الحساب` | Show lockout info |
| System | Info | Gray | `تحديث النظام` | Show detail |

### 15.6 Interactions

| Trigger | Action | Notes |
|---|---|---|
| Tap notification | Navigate to related screen | — |
| Swipe right | Mark as read | Animated |
| Tap filter | Filter by type | Active state update |
| Tap "设置" | Notification preferences | — |
| Pull-to-refresh | Refresh notifications | Spinner |
| Mark all read | Mark all as read | Button in header |

### 15.7 Badge Counter

| Property | Value |
|---|---|
| Component | NotificationBadge |
| Location | Header bell icon |
| Content | Unread count |
| Max Display | `٩٩+` |
| Color | Red-500 circle, white text |

### 15.8 Loading/Error States

| State | Display |
|---|---|
| Loading | Skeleton: 3 notification items |
| Network error | Toast: `تحقق من اتصالك بالإنترنت` |

### 15.9 Accessibility

| Requirement | Implementation |
|---|---|
| Screen reader | Notification announces: `{title}، {description}، {timestamp}` |
| Keyboard | Tab through notifications |
| Focus | Visible ring on items |
| Unread indicator | `aria-label="غير مقروء"` |

---

## 16. DP-RT-001 — Rating/Performance

### 16.1 Page Metadata

| Field | Value |
|---|---|
| Page ID | DP-RT-001 |
| Module | M11 — Rating |
| Type | Page (PG) |
| Priority | P1 |
| FR Ref | FR-015 |
| Building Blocks | B01, B08, B11, B12 |
| Constraints | DP-002, DP-007 |

### 16.2 Purpose

Displays the delivery provider's overall rating, performance metrics, and breakdown of ratings received from customers. Helps providers understand their performance and areas for improvement.

### 16.3 Layout

**Mobile (360px):**

```
+-----------------------------------------------+
|  [Back]   التقييم والأداء            📊  ⚙️  |
+-----------------------------------------------+
|                                               |
|  ┌─────────────────────────────────┐         |
│  │         ⭐ ٤٫٨                   │         │
│  │     من ٥ — ممتاز                │         │
│  │     بناءً على ١٢٠ تقييم         │         │
│  └─────────────────────────────────┘         |
|                                               |
+-----------------------------------------------+
|  📊 مقاييس الأداء                            |
│                                               |
│  ┌──────────┐  ┌──────────┐                  │
│  │  نسبة    │  │  متوسط   │                  │
│  │  النجاح  │  │  الوقت   │                  │
│  │  ٩٥٪    │  │  ٢٥ دقيقة│                  │
│  └──────────┘  └──────────┘                  │
│                                               │
│  ┌──────────┐  ┌──────────┐                  │
│  │  طلبات  │  │  محاولات │                  │
│  │  مكتملة │  │  فاشلة   │                  │
│  │  ١٢٠    │  │  ٦       │                  │
│  └──────────┘  └──────────┘                  │
|                                               |
+-----------------------------------------------+
|  ⭐ توزيع التقييمات                          |
│                                               |
│  ٥ نجوم  ████████████████████  ٨٥  (٧١٪)   |
│  ٤ نجوم  ████                  ٢٠  (١٧٪)   |
│  ٣ نجوم  ██                     ٨  (٧٪)    │
│  ٢ نجوم  █                       ٤  (٣٪)    │
│  ١ نجم   █                       ٣  (٢٪)    │
|                                               |
+-----------------------------------------------+
|  💬 آخر التقييمات                             |
|                                               |
│  ┌─────────────────────────────────┐         │
│  │  ⭐⭐⭐⭐⭐ أحمد م.               │         │
│  │  "توصيل سريع ودقيق"            │         │
│  │  الطلب #١٢٣٤ — ١٣ سبتمبر      │         │
│  └─────────────────────────────────┘         │
│                                               │
│  ┌─────────────────────────────────┐         │
│  │  ⭐⭐⭐⭐⭐ سارة ح.               │         │
│  │  "محترم ومضمون"                 │         │
│  │  الطلب #١٢٣٥ — ١٢ سبتمبر      │         │
│  └─────────────────────────────────┘         │
│                                               │
│  ┌─────────────────────────────────┐         │
│  │  ⭐⭐⭐⭐ محمد ع.                 │         │
│  │  "جيد لكن التأخير قليل"         │         │
│  │  الطلب #١٢٣٦ — ١١ سبتمبر      │         │
│  └─────────────────────────────────┘         │
|                                               |
+-----------------------------------------------+
|  🏠     📦      ✅     💰      👤          |
| الرئيسية المتاح  النشطة  الأرباح  حسابي    |
+-----------------------------------------------+
```

### 16.4 Components

#### 16.4.1 Overall Rating Card

| Property | Value |
|---|---|
| Component | OverallRatingCard |
| Content | Star rating (large), label, review count |
| Stars | Visual stars (filled/unfilled), 48px size |
| Rating Value | `٤٫٨ من ٥` |
| Label | `ممتاز` (Excellent) |
| Reviews Count | `بناءً على ١٢٠ تقييم` |
| Style | Centered, gradient bg |

#### 16.4.2 Performance Metrics Grid

| Property | Value |
|---|---|
| Component | MetricsGrid |
| Layout | 2×2 grid |
| Metrics | Success rate, Average time, Completed deliveries, Failed attempts |
| Card Style | White bg, rounded-lg, shadow-sm |

#### 16.4.3 Rating Distribution

| Property | Value |
|---|---|
| Component | RatingDistribution |
| Content | 5-star to 1-star breakdown |
| Visual | Horizontal bar chart per star level |
| Data | Count + percentage |

#### 16.4.4 Recent Reviews List

| Property | Value |
|---|---|
| Component | RecentReviewsList |
| Content | Customer name (partial), star rating, comment, order ID, date |
| Max Items | 10 (with "عرض المزيد" link) |
| Style | Cards with star rating, comment text |

### 16.5 Rating Labels (Arabic)

| Rating | Label | Color |
|---|---|---|
| 4.5–5.0 | ممتاز (Excellent) | Green |
| 4.0–4.4 | جيد جداً (Very Good) | Green |
| 3.5–3.9 | جيد (Good) | Blue |
| 3.0–3.4 | مقبول (Acceptable) | Yellow |
| 2.0–2.9 | ضعيف (Poor) | Orange |
| 1.0–1.9 | سيء (Bad) | Red |

### 16.6 Interactions

| Trigger | Action | Notes |
|---|---|---|
| Tap star distribution | Show detailed stats | Expand section |
| Tap review | Show full review | Bottom sheet |
| Tap "عرض المزيد" | Load more reviews | Pagination |
| Pull-to-refresh | Refresh rating data | Spinner |

### 16.7 Performance Warning

If success rate drops below 80%:

```
┌─────────────────────────────────────────┐
│  ⚠️ تنبيه: نسبة نجاحك منخفضة          │
│  ─────────────────────────────────────   │
│  نسبة النجاح الحالية: ٧٥٪              │
│  الحد الأدنى المطلوب: ٨٠٪              │
│                                           │
│  يُرجى تحسين أداء التوصيل لتجنب        │
│  تعليق الحساب.                           │
└─────────────────────────────────────────┘
```

| Property | Value |
|---|---|
| Component | PerformanceWarning |
| Trigger | `success_rate < 80%` |
| Style | Yellow-50 bg, yellow-700 text |
| Dismissible | No |
| Action | None (informational) |

### 16.8 Loading/Error States

| State | Display |
|---|---|
| Loading | Skeleton: rating card, metrics, bars, reviews |
| Network error | Toast: `تحقق من اتصالك بالإنترنت` |
| No reviews | `لم تتلقَّ أي تقييمات بعد` |

### 16.9 Accessibility

| Requirement | Implementation |
|---|---|
| Screen reader | Rating announces: `تقييم {value} من ٥، بناءً على {count} تقييم` |
| Keyboard | Tab through sections |
| Focus | Visible ring on interactive elements |
| Stars | `aria-label="٤٫٨ من ٥ نجوم"` |

---

## 17. Cross-Cutting Concerns

### 17.1 Authentication Flow

All Delivery Provider pages require SMS OTP authentication.

| Step | Screen | Description |
|---|---|---|
| 1 | Phone Entry | Enter phone number (阿拉伯) |
| 2 | OTP Verification | Enter 6-digit code |
| 3 | Dashboard | DP-DB-001 (if verified) |

| Property | Value |
|---|---|
| OTP Length | 6 digits |
| OTP Expiry | 5 minutes |
| Resend Cooldown | 60 seconds |
| Session Duration | 24 hours |
| Max OTP Attempts | 5 |

### 17.2 Push Notifications

| Event | Notification Title (AR) | Notification Body (AR) | Screen |
|---|---|---|---|
| New delivery assigned | طلب جديد متاح | طلب #{id} متاح للقبول | DP-AS-001 |
| Delivery accepted | تم قبول الطلب | قمت بقبول الطلب #{id} | DP-AC-001 |
| Customer sent code | رمز التوصيل جاهز | العميل أرسل رمز التوصيل للطلب #{id} | DP-CD-001 |
| Earnings credited | أرباح جديدة | تمت إضافة {amount} ر.ي لمحفظتك | DP-FN-001 |
| Account warning | تنبيه: محاولة فشل | محاولة فشل {n} من ٣ | DP-AC-001 |
| Account locked | حسابك مقفل | تم تعليق حسابك لمدة ٢٤ ساعة | DP-DB-001 |
| Rating received | تقييم جديد | العميل #{name} قيمك بـ {stars} نجوم | DP-RT-001 |

### 17.3 Offline Behavior

| Scenario | Behavior |
|---|---|
| No internet | Show cached data, queue status updates |
| Connection restored | Sync pending updates, refresh data |
| Status update offline | Queue locally, sync when online, show pending indicator |
| Code verification offline | **Not possible** — requires server validation |

### 17.4 Performance Budgets

| Metric | Target |
|---|---|
| First Contentful Paint | < 1.5s |
| Largest Contentful Paint | < 2.5s |
| Cumulative Layout Shift | < 0.1 |
| First Input Delay | < 100ms |
| Time to Interactive | < 3.5s |
| Total page weight | < 500KB |
| JavaScript bundle | < 200KB (compressed) |

### 17.5 Error Handling

| Error Type | Display | Action |
|---|---|---|
| Network error | Toast: `تحقق من اتصالك بالإنترنت` | Auto-retry, pull-to-refresh |
| Auth expired | Redirect to login | Auto-redirect |
| Server error | Toast: `حدث خطأ في الخادم` | Retry button |
| Rate limit | Toast: `لقد تجاوزت الحد المسموح` | Wait and retry |
| Account frozen | Banner: `حسابك مقفل مؤقتًا` | Read-only mode |

### 17.6 Localization Keys

```typescript
const deliveryTranslations = {
  dashboard: {
    welcome: { ar: 'مرحبًا، {{name}} 👋', en: 'Hello, {{name}} 👋' },
    today: { ar: 'اليوم', en: 'Today' },
    availableDeliveries: { ar: 'طلبات متاحة', en: 'Available Deliveries' },
    activeDeliveries: { ar: 'طلبات نشطة', en: 'Active Deliveries' },
    todayEarnings: { ar: 'أرباح اليوم', en: "Today's Earnings" },
    rating: { ar: 'تقييم', en: 'Rating' },
    weeklyStats: { ar: 'إحصائيات هذا الأسبوع', en: 'This Week Stats' },
    recentActive: { ar: 'أحدث الطلبات النشطة', en: 'Recent Active Deliveries' },
    noAvailable: { ar: 'لا توجد طلبات متاحة', en: 'No available deliveries' },
    noActive: { ar: 'لا توجد طلبات نشطة', en: 'No active deliveries' },
  },
  available: {
    title: { ar: 'الطلبات المتاحة', en: 'Available Deliveries' },
    search: { ar: 'ابحث عن طلب...', en: 'Search deliveries...' },
    filterAll: { ar: 'الكل', en: 'All' },
    filterNearby: { ar: 'قريبة', en: 'Nearby' },
    filterHighEarnings: { ar: 'أرباح عالية', en: 'High Earnings' },
    filterNew: { ar: 'جديدة', en: 'New' },
    viewDetails: { ar: 'عرض التفاصيل', en: 'View Details' },
    accept: { ar: 'قبول الطلب', en: 'Accept Delivery' },
    decline: { ar: 'رفض الطلب', en: 'Decline' },
    confirmAccept: { ar: 'تأكيد قبول الطلب', en: 'Confirm Accept' },
    confirmDecline: { ar: 'تأكيد رفض الطلب', en: 'Confirm Decline' },
    items: { ar: '{{count}} منتجات', en: '{{count}} items' },
    distance: { ar: '{{km}} كم', en: '{{km}} km' },
    estimatedTime: { ar: '{{min}} دقيقة', en: '{{min}} min' },
  },
  active: {
    title: { ar: 'الطلبات النشطة', en: 'Active Deliveries' },
    updateStatus: { ar: 'تحديث الحالة', en: 'Update Status' },
    details: { ar: 'التفاصيل', en: 'Details' },
    gpsWarning: { ar: '⚠️ لا يمكن تتبع الموقع المباشر (بدون GPS)', en: '⚠️ Live location tracking unavailable (No GPS)' },
  },
  inProgress: {
    title: { ar: 'توصيل الطلب', en: 'Delivering Order' },
    steps: {
      pickup: { ar: 'الاستلام', en: 'Pickup' },
      inTransit: { ar: 'في الطريق', en: 'In Transit' },
      arrived: { ar: 'الوصول', en: 'Arrived' },
      delivered: { ar: 'التوصيل', en: 'Delivered' },
    },
    callCustomer: { ar: 'اتصال', en: 'Call' },
    whatsappCustomer: { ar: 'واتساب', en: 'WhatsApp' },
    startPickup: { ar: 'جاري الاستلام', en: 'Start Pickup' },
    pickedUp: { ar: 'تم الاستلام', en: 'Picked Up' },
    startTransit: { ar: 'بدء التوصيل', en: 'Start Transit' },
    arrivedAtCustomer: { ar: 'وصلت إلى العميل', en: 'Arrived at Customer' },
    failedAttempt: { ar: 'فشل التوصيل', en: 'Delivery Failed' },
  },
  code: {
    title: { ar: 'رمز التوصيل', en: 'Delivery Code' },
    instruction: { ar: 'اطلب من العميل رمز التوصيل المرسل عبر الرسائل القصيرة أو واتساب.', en: 'Ask the customer for the delivery code sent via SMS or WhatsApp.' },
    enterCode: { ar: 'أدخل رمز التوصيل', en: 'Enter delivery code' },
    verify: { ar: 'تأكيد التوصيل', en: 'Confirm Delivery' },
    wrongCode: { ar: 'رمز غير صحيح — حاول مجددًا', en: 'Wrong code — try again' },
    maxAttempts: { ar: 'لقد تجاوزت الحد الأقصى من المحاولات', en: 'Maximum attempts exceeded' },
    resend: { ar: 'إعادة إرسال الرمز', en: 'Resend Code' },
    resendIn: { ar: 'أعد الإرسال خلال {{seconds}} ثانية', en: 'Resend in {{seconds}} seconds' },
    success: { ar: 'تم التوصيل بنجاح!', en: 'Delivery Successful!' },
    earningsAdded: { ar: 'تم إضافة {{amount}} ر.ي إلى محفظتك', en: '{{amount}} YER added to your wallet' },
  },
  history: {
    title: { ar: 'سجل التوصيلات', en: 'Delivery History' },
    search: { ar: 'ابحث في السجل...', en: 'Search history...' },
    filterCompleted: { ar: 'مكتملة', en: 'Completed' },
    filterFailed: { ar: 'فاشلة', en: 'Failed' },
    filterCancelled: { ar: 'ملغاة', en: 'Cancelled' },
    summary: { ar: '{{total}} طلب — {{completed}} مكتمل — {{failed}} فاشل — {{cancelled}} ملغي', en: '{{total}} deliveries — {{completed}} completed — {{failed}} failed — {{cancelled}} cancelled' },
  },
  earnings: {
    title: { ar: 'الأرباح والمحفظة', en: 'Earnings & Wallet' },
    walletBalance: { ar: 'رصيد المحفظة', en: 'Wallet Balance' },
    withdraw: { ar: 'سحب الأرباح', en: 'Withdraw Earnings' },
    today: { ar: 'اليوم', en: 'Today' },
    thisWeek: { ar: 'هذا الأسبوع', en: 'This Week' },
    thisMonth: { ar: 'هذا الشهر', en: 'This Month' },
    totalDeliveries: { ar: 'إجمالي التوصيلات', en: 'Total Deliveries' },
    avgPerDelivery: { ar: 'متوسط الأرباح/طلب', en: 'Avg per Delivery' },
    highestDelivery: { ar: 'أعلى أرباح/طلب', en: 'Highest Delivery' },
    totalEarnings: { ar: 'إجمالي الأرباح', en: 'Total Earnings' },
  },
  profile: {
    title: { ar: 'إعدادات الحساب', en: 'Account Settings' },
    personalInfo: { ar: 'المعلومات الشخصية', en: 'Personal Information' },
    fullName: { ar: 'الاسم الكامل', en: 'Full Name' },
    phone: { ar: 'رقم الهاتف', en: 'Phone Number' },
    city: { ar: 'المدينة', en: 'City' },
    zone: { ar: 'المنطقة', en: 'Zone' },
    appSettings: { ar: 'إعدادات التطبيق', en: 'App Settings' },
    language: { ar: 'اللغة', en: 'Language' },
    notifications: { ar: 'الإشعارات', en: 'Notifications' },
    darkMode: { ar: 'الوضع الليلي', en: 'Dark Mode' },
    logout: { ar: 'تسجيل الخروج', en: 'Logout' },
    deleteAccount: { ar: 'حذف الحساب', en: 'Delete Account' },
    version: { ar: 'الإصدار {{version}}', en: 'Version {{version}}' },
  },
  zones: {
    title: { ar: 'مناطق التوصيل', en: 'Delivery Zones' },
    instruction: { ar: 'اختر المناطق التي تريد التوصيل فيها:', en: 'Select the zones you want to deliver to:' },
    selected: { ar: '{{selected}} من {{total}} مناطق محددة', en: '{{selected}} of {{total}} zones selected' },
    save: { ar: 'حفظ التغييرات', en: 'Save Changes' },
    saved: { ar: 'تم حفظ مناطق التوصيل', en: 'Delivery zones saved' },
  },
  notifications: {
    title: { ar: 'الإشعارات', en: 'Notifications' },
    filterAll: { ar: 'الكل', en: 'All' },
    filterDeliveries: { ar: 'طلبات', en: 'Deliveries' },
    filterEarnings: { ar: 'أرباح', en: 'Earnings' },
    filterSystem: { ar: 'نظام', en: 'System' },
    today: { ar: 'اليوم', en: 'Today' },
    yesterday: { ar: 'أمس', en: 'Yesterday' },
    thisWeek: { ar: 'هذا الأسبوع', en: 'This Week' },
    older: { ar: 'أقدم', en: 'Older' },
    markAllRead: { ar: 'تحديد الكل كمقروء', en: 'Mark All Read' },
    empty: { ar: 'لا توجد إشعارات', en: 'No notifications' },
    newDelivery: { ar: 'طلب جديد متاح', en: 'New Delivery Available' },
    earningsCredited: { ar: 'تم إضافة أرباح', en: 'Earnings Credited' },
    warning: { ar: 'تنبيه', en: 'Warning' },
    accountLocked: { ar: 'تم تعليق الحساب', en: 'Account Locked' },
    systemUpdate: { ar: 'تحديث النظام', en: 'System Update' },
  },
  rating: {
    title: { ar: 'التقييم والأداء', en: 'Rating & Performance' },
    overall: { ar: 'من ٥', en: 'out of 5' },
    reviews: { ar: 'بناءً على {{count}} تقييم', en: 'Based on {{count}} reviews' },
    successRate: { ar: 'نسبة النجاح', en: 'Success Rate' },
    avgTime: { ar: 'متوسط الوقت', en: 'Average Time' },
    completed: { ar: 'طلبات مكتملة', en: 'Completed Deliveries' },
    failed: { ar: 'محاولات فاشلة', en: 'Failed Attempts' },
    distribution: { ar: 'توزيع التقييمات', en: 'Rating Distribution' },
    recentReviews: { ar: 'آخر التقييمات', en: 'Recent Reviews' },
    excellent: { ar: 'ممتاز', en: 'Excellent' },
    veryGood: { ar: 'جيد جداً', en: 'Very Good' },
    good: { ar: 'جيد', en: 'Good' },
    acceptable: { ar: 'مقبول', en: 'Acceptable' },
    poor: { ar: 'ضعيف', en: 'Poor' },
    bad: { ar: 'سيء', en: 'Bad' },
    performanceWarning: { ar: 'تنبيه: نسبة نجاحك منخفضة', en: 'Warning: Your success rate is low' },
  },
  lockout: {
    title: { ar: 'حسابك مقفل مؤقتًا', en: 'Your Account is Temporarily Locked' },
    message: { ar: 'بسبب ٣ محاولات توصيل فاشلة متتالية، تم تعليق حسابك لمدة ٢٤ ساعة.', en: 'Due to 3 consecutive failed delivery attempts, your account has been locked for 24 hours.' },
    unlockAt: { ar: 'سيتم إعادة التفعيل في:', en: 'Account will be reactivated at:' },
    noAccept: { ar: 'لا يمكنك قبول طلبات جديدة أثناء القفل.', en: 'You cannot accept new deliveries while locked.' },
  },
  failedAttempt: {
    title: { ar: 'تسجيل فشل التوصيل', en: 'Record Delivery Failure' },
    reasonLabel: { ar: 'سبب الفشل:', en: 'Failure Reason:' },
    reasonOptions: {
      notAvailable: { ar: 'العميل غير متواجد', en: 'Customer not available' },
      wrongAddress: { ar: 'العنوان خاطئ', en: 'Wrong address' },
      refused: { ar: 'رفض الاستلام', en: 'Refused to receive' },
      unreachable: { ar: 'لا يمكن الوصول', en: 'Cannot reach' },
      other: { ar: 'أخرى', en: 'Other' },
    },
    notesLabel: { ar: 'ملاحظات (اختياري):', en: 'Notes (optional):' },
    attemptWarning: { ar: 'محاولة {{current}} من ٣ — بعد ٣ محاولات فاشلة سيتم تعليق حسابك لمدة ٢٤ ساعة', en: 'Attempt {{current}} of 3 — after 3 failed attempts your account will be locked for 24 hours' },
    record: { ar: 'تسجيل الفشل', en: 'Record Failure' },
  },
} as const;
```

### 17.7 Screen Navigation Map

```
                        ┌──────────────┐
                        │   Login      │
                        │  (SMS OTP)   │
                        └──────┬───────┘
                               │
                    ┌──────────┴──────────┐
                    │                     │
                    ▼                     ▼
            ┌──────────────┐      ┌──────────────┐
            │  Dashboard   │      │ Notifications │
            │  DP-DB-001   │      │  DP-NT-001   │
            └──────┬───────┘      └──────────────┘
                   │
       ┌───────────┼───────────┬───────────┐
       │           │           │           │
       ▼           ▼           ▼           ▼
  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐
  │Available │ │  Active  │ │ Earnings │ │ Profile  │
  │DP-AS-001 │ │DP-AC-001 │ │DP-FN-001 │ │DP-PR-001 │
  └────┬─────┘ └────┬─────┘ └────┬─────┘ └────┬─────┘
       │            │            │            │
       ▼            ▼            │            │
  ┌──────────┐ ┌──────────┐     │     ┌──────────┐
  │ Delivery │ │Delivery  │     │     │  Zones   │
  │ Detail   │ │In-Progress│    │     │DP-ZN-001 │
  │DP-AS-002 │ │DP-AC-002 │     │     └──────────┘
  └────┬─────┘ └────┬─────┘     │
       │            │            │
       │            ▼            │
       │     ┌──────────┐       │
       │     │   Code   │       │
       │     │Verify    │       │
       │     │DP-CD-001 │       │
       │     └────┬─────┘       │
       │          │             │
       ▼          ▼             ▼
  ┌─────────────────────────────────┐
  │         History                 │
  │         DP-HS-001               │
  │         (accessible from        │
  │          Profile or Dashboard)  │
  └─────────────────────────────────┘

  ┌─────────────────────────────────┐
  │       Rating/Performance        │
  │         DP-RT-001               │
  │    (accessible from Profile     │
  │     or after delivery)          │
  └─────────────────────────────────┘
```

---

*This document provides the complete UI/UX specification for the YemenMart Delivery Provider Portal (DP). All 13 pages are defined with full layout, components, interactions, states, accessibility, and localization details. The specification follows the established conventions from the Master Plan and Design Guidelines.*
