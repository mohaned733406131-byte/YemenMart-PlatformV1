# YemenMart Vendor Panel - UI/UX Specification v1.0

> **Portal:** Vendor Panel
> **Tech Stack:** React 18 + Vite, Tailwind CSS 4
> **Design Language:** Arabic-first RTL, Mobile-first (desktop-primary for Vendor Panel)
> **Payment Model:** Wallet-only
> **Last Updated:** 2026-09-13

---

## Table of Contents

1. [Global Conventions](#global-conventions)
2. [Page Catalog](#page-catalog)
3. [VP-DB-001: Vendor Dashboard](#vp-db-001-vendor-dashboard)
4. [VP-PR-001: Product Management (List)](#vp-pr-001-product-management-list)
5. [VP-PR-002: Product Create/Edit](#vp-pr-002-product-createedit)
6. [VP-PR-003: Product Detail](#vp-pr-003-product-detail)
7. [VP-IV-001: Inventory Management](#vp-iv-001-inventory-management)
8. [VP-OR-001: Order Management (List)](#vp-or-001-order-management-list)
9. [VP-OR-002: Order Detail](#vp-or-002-order-detail)
10. [VP-ST-001: Store Settings](#vp-st-001-store-settings)
11. [VP-ST-002: Store Template Selection](#vp-st-002-store-template-selection)
12. [VP-FN-001: Financial Dashboard](#vp-fn-001-financial-dashboard)
13. [VP-FN-002: Transaction History](#vp-fn-002-transaction-history)
14. [VP-FN-003: Payout Management](#vp-fn-003-payout-management)
15. [VP-RV-001: Reviews Management](#vp-rv-001-reviews-management)
16. [VP-KY-001: KYC Verification](#vp-ky-001-kyc-verification)
17. [VP-AN-001: Analytics Dashboard](#vp-an-001-analytics-dashboard)
18. [VP-NT-001: Notifications](#vp-nt-001-notifications)
19. [VP-PR-004: Profile Settings](#vp-pr-004-profile-settings)

---

## Global Conventions

### Navigation Structure (Sidebar)

`
+---------------------------------------------------------------------------+
|  YemenMart Vendor  [AR|EN]                          🔔 3  👤 ▼          |
+------------+--------------------------------------------------------------+
|            |                                                              |
|  📊 Dashboard       |   MAIN CONTENT AREA                                  |
|  📦 Products        |   (scrollable)                                       |
|  📋 Orders          |                                                      |
|  🏪 Store           |                                                      |
|  💰 Finance         |                                                      |
|  ⭐ Reviews         |                                                      |
|  📈 Analytics       |                                                      |
|  🔔 Notifications   |                                                      |
|  ────────────       |                                                      |
|  ⚙️ Settings        |                                                      |
|  🪪 KYC             |                                                      |
|            |                                                              |
+------------+--------------------------------------------------------------+
|  © 2026 YemenMart · version 1.0.0                                          |
+---------------------------------------------------------------------------+
`

### Design Tokens

| Token | Value | Notes |
|-------|-------|-------|
| --color-primary | #1B5E20 (Green 900) | Brand green |
| --color-primary-light | #4CAF50 | CTA buttons, links |
| --color-primary-dark | #0D3B12 | Hover states |
| --color-secondary | #FF8F00 (Amber 800) | Wallet, earnings |
| --color-danger | #D32F2F | Errors, destructive actions |
| --color-warning | #F57F17 | Pending states |
| --color-info | #1565C0 | Info messages |
| --color-success | #2E7D32 | Success states |
| --color-bg | #FAFAFA | Page background |
| --color-surface | #FFFFFF | Card/panel background |
| --color-border | #E0E0E0 | Default borders |
| --font-family-ar | 'Tajawal', 'Noto Sans Arabic', sans-serif | Arabic font |
| --font-family-en | 'Inter', 'Noto Sans', sans-serif | Latin font |
| --sidebar-width | 260px | Collapsed: 72px |
| --header-height | 64px | Top bar |

### RTL Baseline

- All layouts mirror horizontally. margin-right becomes margin-left, order-right becomes order-left, etc.
- Tailwind tl: variants are used throughout. The dir="rtl" attribute is set on <html>.
- LTR mode is available via a language toggle but the default is Arabic/RTL.
- Icons that convey direction (arrows, chevrons) are flipped via tl:rotate-180.
- Scrollbar direction changes to match RTL.
- Arabic text uses 	ext-align: right; English mixed text uses 	ext-align: start (auto).

### Typography Scale

| Class | Size (px) | Weight | Use |
|-------|-----------|--------|-----|
| 	ext-4xl | 36 | 800 | Page title (rare) |
| 	ext-3xl | 30 | 700 | Section headings |
| 	ext-2xl | 24 | 700 | Card headings |
| 	ext-xl | 20 | 600 | Widget titles |
| 	ext-lg | 18 | 600 | Sub-headings |
| 	ext-base | 16 | 400 | Body / labels |
| 	ext-sm | 14 | 400 | Secondary text |
| 	ext-xs | 12 | 400 | Captions, badges |

### Common Components

| Component | Tailwind Classes | Description |
|-----------|-----------------|-------------|
| Card | g-white rounded-xl border border-gray-200 shadow-sm p-6 | Content containers |
| Badge | inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium | Status labels |
| Button (Primary) | g-green-700 hover:bg-green-800 text-white px-4 py-2 rounded-lg font-medium | Primary CTA |
| Button (Secondary) | order border-gray-300 hover:bg-gray-50 text-gray-700 px-4 py-2 rounded-lg | Secondary actions |
| Button (Danger) | g-red-600 hover:bg-red-700 text-white px-4 py-2 rounded-lg | Destructive actions |
| Table Row | order-b border-gray-100 hover:bg-gray-50 | Data grid rows |
| Input | order border-gray-300 rounded-lg px-3 py-2 focus:ring-2 focus:ring-green-500 focus:border-green-500 | Form inputs |
| Skeleton | nimate-pulse bg-gray-200 rounded | Loading placeholders |
| Toast | ixed bottom-4 right-4 bg-gray-800 text-white px-4 py-3 rounded-lg shadow-lg | Feedback toasts |

### Empty State Pattern

`
+------------------------------------------+
|                                          |
|           📦 (icon)                      |
|                                          |
|     No products yet                      |
|     Create your first product to         |
|     start selling on YemenMart.          |
|                                          |
|     [ + Add Product ]                    |
|                                          |
+------------------------------------------+
`

---

## Page Catalog

| ID | Page Name | Module | Category | Priority | Auth | Platform |
|----|-----------|--------|----------|----------|------|----------|
| VP-DB-001 | Vendor Dashboard | Dashboard | Overview | P0 | Yes | Desktop |
| VP-PR-001 | Product Management (List) | Products | Management | P0 | Yes | Desktop |
| VP-PR-002 | Product Create/Edit | Products | Management | P0 | Yes | Desktop |
| VP-PR-003 | Product Detail | Products | Management | P1 | Yes | Desktop |
| VP-IV-001 | Inventory Management | Inventory | Management | P0 | Yes | Desktop |
| VP-OR-001 | Order Management (List) | Orders | Management | P0 | Yes | Desktop |
| VP-OR-002 | Order Detail | Orders | Management | P0 | Yes | Desktop |
| VP-ST-001 | Store Settings | Store | Configuration | P1 | Yes | Desktop |
| VP-ST-002 | Store Template Selection | Store | Configuration | P2 | Yes | Desktop |
| VP-FN-001 | Financial Dashboard | Finance | Overview | P0 | Yes | Desktop |
| VP-FN-002 | Transaction History | Finance | Reporting | P1 | Yes | Desktop |
| VP-FN-003 | Payout Management | Finance | Operations | P0 | Yes | Desktop |
| VP-RV-001 | Reviews Management | Reviews | Engagement | P1 | Yes | Desktop |
| VP-KY-001 | KYC Verification | Compliance | Verification | P0 | Yes | Desktop |
| VP-AN-001 | Analytics Dashboard | Analytics | Reporting | P1 | Yes | Desktop |
| VP-NT-001 | Notifications | System | Communication | P1 | Yes | Desktop |
| VP-PR-004 | Profile Settings | Settings | Configuration | P1 | Yes | Desktop |

### Permission Matrix

| Page | Permission Required |
|------|---------------------|
| VP-DB-001 | Any authenticated vendor |
| VP-PR-001 | products:read |
| VP-PR-002 | products:create / products:update |
| VP-PR-003 | products:read |
| VP-IV-001 | inventory:read / inventory:update |
| VP-OR-001 | orders:read |
| VP-OR-002 | orders:read / orders:update |
| VP-ST-001 | store:read / store:update |
| VP-ST-002 | store:read / store:update |
| VP-FN-001 | inance:read |
| VP-FN-002 | inance:read |
| VP-FN-003 | inance:read |
| VP-RV-001 | eviews:read |
| VP-KY-001 | Any authenticated vendor |
| VP-AN-001 | Any authenticated vendor |
| VP-NT-001 | Any authenticated vendor |
| VP-PR-004 | Any authenticated vendor |

---

## VP-DB-001: Vendor Dashboard

### 1. Page Metadata

| Field | Value |
|-------|-------|
| **ID** | VP-DB-001 |
| **Page Name** | Vendor Dashboard |
| **Portal** | Vendor Panel |
| **Platform** | Desktop |
| **Module** | Dashboard |
| **Category** | Overview |
| **Priority** | P0 |
| **Auth Required** | Yes |
| **Vendor Roles** | All authenticated vendors |
| **Permissions** | None (default after auth) |
| **URL** | /vendor/dashboard |
| **Parent Page** | — (Root) |
| **Requirements References** | REQ-VN-DASH-001, REQ-VN-DASH-002, REQ-VN-ROLE-001 |
| **Status** | Draft |

### 2. Page Purpose

Provide vendors with a real-time overview of their store performance, pending actions, recent orders, financial summary, and key metrics at a glance. This is the landing page after vendor login and the primary hub for daily operations.

### 3. Entry Points

| Source | Trigger |
|--------|---------|
| Vendor Login | Redirect after successful authentication |
| Sidebar Navigation | Click "Dashboard" in sidebar |
| Deep Link | Direct URL navigation |

### 4. Exit Points

| Destination | Trigger |
|-------------|---------|
| VP-PR-001 | Click "View All Products" |
| VP-OR-001 | Click "View All Orders" / click pending order |
| VP-FN-001 | Click "View Financials" / earnings widget |
| VP-RV-001 | Click "View Reviews" / unread review badge |
| VP-NT-001 | Click notification bell |
| VP-KY-001 | Click KYC banner CTA |
| VP-ST-001 | Click "Store Settings" shortcut |
| VP-AN-001 | Click "Analytics" in sidebar |

### 5. Information Architecture

`
Dashboard
├── Header
│   ├── Welcome message (vendor name, store name)
│   ├── Quick actions dropdown
│   └── Notification indicator
├── KPI Summary Strip
│   ├── Today's Sales (YER)
│   ├── Pending Orders (count)
│   ├── Active Products (count)
│   ├── Average Rating (★)
│   └── Wallet Balance (YER)
├── Recent Orders Widget
│   ├── Latest 5 orders
│   ├── Status badges
│   └── Quick action (view details)
├── Low Stock Alert Widget
│   ├── Products with stock < threshold
│   └── Quick action (restock)
├── Revenue Chart Widget
│   ├── Last 7 days line chart
│   └── Period selector
├── Reviews Summary Widget
│   ├── Average rating
│   ├── Pending reviews count
│   └── Rating distribution bar
├── Store Status Banner
│   ├── KYC status
│   ├── Store verification status
│   └── Profile completion %
└── Quick Links
    ├── Add New Product
    ├── Update Store Info
    └── View Payouts
`

### 6. Layout Structure

`
┌─────────────────────────────────────────────────────────────────────────────┐
│  مرحباً، محمد!    [إجراءات سريعة ▼]                  🔔 3  👤 ▼          │
│  متجر إلكترونيات اليمن                                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────┐ ┌──────────┐ │
│  │ مبيعات اليوم│ │طلبات معلقة  │ │منتجات نشطة  │ │متوسط    │ │رصيد      │ │
│  │  45,200 ر.ي │ │     12      │ │     89      │ │تقييم 4.7│ │128,500 ر.ي│ │
│  │  ↑ 12% ↗   │ │  ↓ 3 مقارنة │ │  +2 هذا    │ │ ↑ 0.1   │ │          │ │
│  │             │ │  بالأمس     │ │  الأسبوع   │ │         │ │          │ │
│  └─────────────┘ └─────────────┘ └─────────────┘ └─────────┘ └──────────┘ │
│                                                                             │
│  ┌──────────────────────────────┐ ┌──────────────────────────────────────┐  │
│  │  أحدث الطلبات               │ │  تنبيهات المخزون المنخفض             │  │
│  │  ┌──────────────────────────┐│ │  ┌──────────────────────────────────┐│ │
│  │  │ #ORD-8847  تم التوصيل ✅ ││ │  │ فأرة لاسلكية  ── مخزون: 2 ⚠️   ││ │
│  │  │ #ORD-8843  تم الشحن 🚚   ││ │  │ كابل USB-C     ── مخزون: 5 ⚠️   ││ │
│  │  │ #ORD-8841  قيد المعالجة 📦││ │  │ موزع HDMI      ── مخزون: 1 🔴   ││ │
│  │  │ #ORD-8839  جديد ⏳       ││ │  │                                  ││ │
│  │  │ #ORD-8837  تم التوصيل ✅ ││ │  │  [ عرض كل المنتجات ]            ││ │
│  │  └──────────────────────────┘│ │  └──────────────────────────────────┘│ │
│  │  [ عرض كل الطلبات ]         │ └──────────────────────────────────────┘  │
│  └──────────────────────────────┘                                           │
│                                                                             │
│  ┌──────────────────────────────┐ ┌──────────────────────────────────────┐  │
│  │  الإيرادات (آخر 7 أيام)      │ │  ملخص التقييمات                     │  │
│  │                              │ │                                      │  │
│  │  ╭─────╮                     │ │  ★★★★★  ████████░░  67%             │  │
│  │  │     ╰──╮                  │ │  ★★★★☆  ███░░░░░░░  18%             │  │
│  │  │        ╰──╮               │ │  ★★★☆☆  ██░░░░░░░░   9%             │  │
│  │  │           ╰─              │ │  ★★☆☆☆  █░░░░░░░░░   4%             │  │
│  │  ──────────────────          │ │  ★☆☆☆☆  ░░░░░░░░░░   2%             │  │
│  │  سبت أحد اثنين ثالث...       │ │                                      │  │
│  └──────────────────────────────┘ │  معلق: 3 تقييمات                     │  │
│                                   │  [ عرض كل التقييمات ]                │  │
│                                   └──────────────────────────────────────┘  │
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │  ⚠️  أكمل التحقق من هويتك (KYC) لتمكين سحب الأرباح.               │   │
│  │     [ إكمال التحقق → ]                                               │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
`

### 7. Responsive Design

| Breakpoint | Behavior |
|------------|----------|
| **Desktop (≥1024px)** | Full layout as shown — 5-column KPI strip, 2-column content grid |
| **Tablet (768–1023px)** | Sidebar collapses to icons (72px). KPI strip wraps to 3+2. Content goes single-column. |
| **Mobile (<768px)** | Limited dashboard: KPIs only, simplified list views. Full vendor management not supported on mobile. |

### 8. Widget Inventory

| Widget | ID | Type | Data Source | Refresh |
|--------|----|------|-------------|---------|
| KPI: Today's Sales | kpi-sales | Metric Card | GET /vendors/dashboard | 60s |
| KPI: Pending Orders | kpi-pending | Metric Card | GET /vendors/dashboard | 60s |
| KPI: Active Products | kpi-products | Metric Card | GET /vendors/dashboard | 300s |
| KPI: Average Rating | kpi-rating | Metric Card | GET /vendors/dashboard | 300s |
| KPI: Wallet Balance | kpi-wallet | Metric Card | GET /vendors/dashboard | 60s |
| Recent Orders | widget-recent-orders | List (5 items) | GET /vendors/dashboard/recent-orders | 120s |
| Low Stock Alerts | widget-low-stock | List (5 items) | GET /vendors/dashboard/low-stock | 300s |
| Revenue Chart | widget-revenue | Line Chart | GET /vendors/dashboard/revenue?period=7d | Manual |
| Reviews Summary | widget-reviews | Bar Chart + Stats | GET /vendors/dashboard/reviews-summary | 300s |
| Store Status Banner | widget-store-status | Alert Banner | GET /vendors/me | On Load |
| Quick Links | static-links | Static | — | — |

### 9. Widget Specification — KPI Cards

`	sx
interface KPICardProps {
  title: string;          // Localized: "مبيعات اليوم" | "Today's Sales"
  value: string;          // Formatted: "45,200"
  unit?: string;          // "YER" for financial, "★" for rating
  change?: number;        // Percentage change from previous period
  changeDirection: 'up' | 'down' | 'flat';
  icon: LucideIcon;
  color: 'green' | 'amber' | 'blue' | 'purple' | 'red';
  loading?: boolean;
  onClick?: () => void;
}
`

### 10. Data Requirements

| Endpoint | Method | Response | Cache TTL | Used By |
|----------|--------|----------|-----------|---------|
| /api/v1/vendors/dashboard | GET | KPIs, store status | 60s | KPI cards, store banner |
| /api/v1/vendors/dashboard/recent-orders | GET | OrderSummary[] | 120s | Recent orders widget |
| /api/v1/vendors/dashboard/low-stock | GET | ProductSummary[] | 300s | Low stock widget |
| /api/v1/vendors/dashboard/revenue | GET | RevenuePoint[] | 300s | Revenue chart |
| /api/v1/vendors/dashboard/reviews-summary | GET | ReviewSummary | 300s | Reviews widget |
| /api/v1/vendors/me | GET | VendorProfile | 600s | Store status, welcome |

### 11. Data Loading Strategy

1. **Critical Path:** KPI cards render immediately with skeleton states.
2. **Below Fold:** Revenue chart and reviews load after KPIs are visible (IntersectionObserver).
3. **Deferred:** Low stock alerts load after 2s delay (low priority).
4. **Stale-While-Revalidate:** Show cached data while refreshing in background.
5. **WebSocket:** Order count updates via WS /vendors/orders/live.

### 12. User Actions

| Action | Trigger | Target | Permission |
|--------|---------|--------|------------|
| View all products | Button click | VP-PR-001 | products:read |
| View all orders | Button click | VP-OR-001 | orders:read |
| View order detail | Row click | VP-OR-002 | orders:read |
| View financials | KPI click | VP-FN-001 | inance:read |
| View all reviews | Button click | VP-RV-001 | eviews:read |
| Restock product | Button click | VP-IV-001 | inventory:update |
| Complete KYC | Banner CTA | VP-KY-001 | — |
| View notifications | Bell icon | VP-NT-001 | — |
| Add new product | Quick action | VP-PR-002 | products:create |

### 13. Form Specification

Dashboard has no forms. Quick action dropdown uses a button group.

### 14. Validation Rules

N/A — Read-only dashboard.

### 15. State Machine

`
                    ┌──────────┐
                    │  Initial │
                    └────┬─────┘
                         │
                    ┌────▼─────┐
              ┌─────┤ Loading  ├─────┐
              │     └────┬─────┘     │
              │          │           │
         Error│     Success│     Empty│
              │          │           │
         ┌────▼───┐ ┌───▼────┐ ┌───▼──────┐
         │ Error  │ │  Data  │ │  Empty   │
         │ Retry  │ │ Render │ │  CTA     │
         └────────┘ └────────┘ └──────────┘
`

### 16. Error States

| Error | Display | Action |
|-------|---------|--------|
| Network failure | Toast: "خطأ في الاتصال" + retry | Auto-retry 3x, then manual retry button |
| Auth expired | Redirect to login | — |
| Server 500 | Error card on dashboard | "Try Again" button |
| Partial data load | Show available KPIs, gray out failed ones | Retry individual widget |

### 17. Empty States

| Scenario | Display |
|----------|---------|
| No orders ever | Illustration + "لا توجد طلبات بعد. شارك متجرك للبدء." |
| No low stock | Green checkmark + "جميع منتجاتك متوفرة بمخزون كافٍ." |
| No reviews | "لا توجد تقييمات بعد. سيظهر تقييمك الأول هنا." |
| No revenue data | "ستظهر بيانات الإيرادات بعد تلقي أول طلب." |

### 18. Loading States

- Each KPI card shows a skeleton with pulsing rectangles.
- Chart area shows skeleton axes and curve.
- List widgets show 3–5 skeleton rows.

### 19. Success States

- Toast on successful KYC completion: "تم التحقق بنجاح! ✓"
- Real-time order count update with subtle pulse animation on badge.

### 20. Permission-Based UI

| Permission | UI Impact |
|------------|-----------|
| No products:create | "إضافة منتج" button hidden from quick actions |
| No orders:read | Recent orders widget shows "صلاحيات غير كافية" message |
| No inance:read | Wallet KPI hidden, revenue chart shows lock icon |
| No eviews:read | Reviews widget hidden entirely |
| KYC not verified | Persistent yellow banner at bottom of dashboard |

### 21. Security UX

- Wallet balance is masked by default; click to reveal (eye icon toggle).
- Financial KPIs show approximate values (±5%) until KYC is verified.
- Session timeout warning at 15 min before expiry.

### 22. RTL/LTR Behavior

- All KPI cards, charts, and lists respect RTL layout.
- Chart Y-axis labels move from left to right side.
- Sparkline arrows (↑↓) are positioned correctly for RTL.
- Welcome text reads right-to-left: "!مرحباً محمد"

### 23. Internationalization

| Key | Arabic | English |
|-----|--------|---------|
| dashboard.welcome | مرحباً {name} | Welcome, {name} |
| dashboard.kpi.todaysSales | مبيعات اليوم | Today's Sales |
| dashboard.kpi.pendingOrders | الطلبات المعلقة | Pending Orders |
| dashboard.kpi.activeProducts | المنتجات النشطة | Active Products |
| dashboard.kpi.avgRating | متوسط التقييم | Average Rating |
| dashboard.kpi.walletBalance | رصيد المحفظة | Wallet Balance |
| dashboard.recentOrders | أحدث الطلبات | Recent Orders |
| dashboard.lowStock | تنبيهات المخزون المنخفض | Low Stock Alerts |
| dashboard.revenue | الإيرادات (آخر 7 أيام) | Revenue (Last 7 Days) |
| dashboard.reviews | ملخص التقييمات | Reviews Summary |
| dashboard.kycBanner | أكمل التحقق من هويتك لتمكين سحب الأرباح | Complete KYC to enable payouts |
| dashboard.empty.orders | لا توجد طلبات بعد | No orders yet |

### 24. Accessibility

- All KPI values have ria-label with full text description.
- Chart has ria-label="Revenue chart for the last 7 days" and hidden data table.
- Focus order: KPIs → Recent Orders → Charts → Quick Actions.
- Color is never the only indicator; badges include text + icon.
- Keyboard: Tab navigates widgets, Enter/Space activates links.

### 25. Performance Requirements

| Metric | Target |
|--------|--------|
| Initial Paint (FCP) | < 1.2s |
| Time to Interactive (TTI) | < 2.5s |
| Largest Contentful Paint (LCP) | < 2.0s |
| Bundle Size (dashboard route) | < 80KB gzipped |
| API Response (p95) | < 500ms |
| Chart Render | < 200ms |

### 26. Analytics Events

| Event | Trigger | Properties |
|-------|---------|------------|
| endor_dashboard_view | Page load | { loadTime, dataSources } |
| endor_dashboard_kpi_click | KPI card click | { kpiType, value } |
| endor_dashboard_quick_action | Quick action use | { actionType } |
| endor_dashboard_widget_interact | Widget click | { widgetId, action } |

### 27. Notifications

| Trigger | Type | Channel |
|---------|------|---------|
| New order received | Real-time badge update | WebSocket |
| Low stock alert | Badge + toast | WebSocket |
| New review | Badge update | WebSocket |
| KYC status change | Banner update | WebSocket |
| Payout processed | Toast | WebSocket |

### 28. Cross-Page Relationships

`
VP-DB-001 ──→ VP-PR-001 (Products link)
         ──→ VP-OR-001 (Orders link)
         ──→ VP-OR-002 (Order row click)
         ──→ VP-FN-001 (Wallet click)
         ──→ VP-RV-001 (Reviews link)
         ──→ VP-IV-001 (Restock link)
         ──→ VP-KY-001 (KYC banner)
         ──→ VP-NT-001 (Notification bell)
         ──→ VP-PR-002 (Add product quick action)
         ──→ VP-ST-001 (Store settings shortcut)
`

### 29. Visual Preview

See Layout Structure above for the ASCII wireframe. Key visual characteristics:
- Clean card-based layout with generous whitespace.
- Green accent color on primary metrics.
- Amber accent on financial metrics.
- Red accent on alerts/low stock.
- Subtle shadows (shadow-sm) on cards.
- Rounded corners (ounded-xl) on all containers.

---
## VP-PR-001: Product Management (List)

### 1. Page Metadata

| Field | Value |
|-------|-------|
| **ID** | VP-PR-001 |
| **Page Name** | Product Management (List) |
| **Portal** | Vendor Panel |
| **Platform** | Desktop |
| **Module** | Products |
| **Category** | Management |
| **Priority** | P0 |
| **Auth Required** | Yes |
| **Vendor Roles** | Vendor (all) |
| **Permissions** | products:read, products:create, products:update (own), inventory:read |
| **URL** | /vendor/products |
| **Parent Page** | VP-DB-001 |
| **Requirements References** | REQ-VN-PROD-001, REQ-VN-PROD-002, REQ-VN-PROD-003 |
| **Status** | Draft |

### 2. Page Purpose

Display a searchable, filterable, sortable list of all products owned by the vendor. Enable quick product management actions: create, edit, duplicate, toggle status, view inventory, and bulk operations.

### 3. Entry Points

| Source | Trigger |
|--------|---------|
| Sidebar Navigation | Click "Products" |
| VP-DB-001 | "View All Products" button |
| VP-PR-002 | After product create/edit save |
| VP-PR-003 | Back button from detail |

### 4. Exit Points

| Destination | Trigger |
|-------------|---------|
| VP-PR-002 | "Add Product" button / row edit action |
| VP-PR-003 | Row click |
| VP-IV-001 | "Manage Inventory" action |
| VP-DB-001 | Breadcrumb / sidebar |

### 5. Information Architecture

`
Product List
├── Header
│   ├── Page Title: "المنتجات" / "Products"
│   ├── Product Count Badge
│   └── [ + Add Product ] CTA
├── Toolbar
│   ├── Search Bar (name, SKU, barcode)
│   ├── Filter Panel
│   │   ├── Category Filter (multi-select)
│   │   ├── Status Filter (Active, Draft, Out of Stock, Archived)
│   │   ├── Stock Level (In Stock, Low Stock, Out of Stock)
│   │   └── Price Range
│   ├── Sort Dropdown (Name, Price, Stock, Created, Updated)
│   └── View Toggle (Grid / Table)
├── Bulk Actions Bar (when items selected)
│   ├── Selected Count
│   ├── Bulk Status Change
│   ├── Bulk Price Update
│   ├── Bulk Delete
│   └── Export Selected
├── Product Grid / Table
│   ├── Product Card / Row
│   │   ├── Thumbnail Image
│   │   ├── Product Name (AR/EN)
│   │   ├── SKU
│   │   ├── Price (YER) / Sale Price
│   │   ├── Stock Level (with color indicator)
│   │   ├── Status Badge (Active/Draft/OOS/Archived)
│   │   ├── Rating (★)
│   │   └── Actions Menu (⋯)
│   └── Pagination
└── Empty State
`

### 6. Layout Structure

`
┌─────────────────────────────────────────────────────────────────────────────┐
│  المنتجات (89)                                [+ إضافة منتج]              │
├─────────────────────────────────────────────────────────────────────────────┤
│  🔍 بحث عن منتج...        [التصنيف ▼] [الحالة ▼] [المخزون ▼] [ترتيب ▼]   │
│                             ☐ تم تحديد 5  [تغيير الحالة] [حذف] [تصدير]   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌────────────┐ ┌────────────┐ ┌────────────┐ ┌────────────┐ ┌────────────┐│
│  │ [img]      │ │ [img]      │ │ [img]      │ │ [img]      │ │ [img]      ││
│  │ ☐          │ │ ☐          │ │ ☐          │ │ ☐          │ │ ☐          ││
│  │ فأرة لاسلكية│ │ كابل USB-C │ │ موزع HDMI  │ │ لوحة مفاتيح│ │ سماعات     ││
│  │ SKU: WL-001│ │ SKU: UC-012│ │ SKU: HM-003│ │ SKU: KB-007│ │ SKU: HS-015││
│  │ 3,500 ر.ي  │ │ 1,200 ر.ي  │ │ 4,800 ر.ي  │ │ 8,900 ر.ي  │ │ 6,200 ر.ي  ││
│  │ مخزون: 45  │ │ مخزون: 120 │ │ مخزون: 8   │ │ مخزون: 32  │ │ مخزون: 0   ││
│  │ ● نشط      │ │ ● نشط      │ │ ⚠ منخفض   │ │ ● نشط      │ │ ○ نفد      ││
│  │ ★ 4.7      │ │ ★ 4.3      │ │ ★ 4.9      │ │ ★ 4.5      │ │ ★ 4.1      ││
│  │    [⋯]     │ │    [⋯]     │ │    [⋯]     │ │    [⋯]     │ │    [⋯]     ││
│  └────────────┘ └────────────┘ └────────────┘ └────────────┘ └────────────┘│
│                                                                             │
│  ◀ 1 2 3 4 5 ... 18 ▶                          عرض 1-10 من 89             │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
`

### 7. Responsive Design

| Breakpoint | Behavior |
|------------|----------|
| **Desktop (≥1024px)** | Grid view: 5 columns. Table view available. Full toolbar. |
| **Tablet (768–1023px)** | Grid: 3 columns. Table view only. Sidebar collapsed. |
| **Mobile (<768px)** | Not supported for vendor management. |

### 8. Widget Inventory

| Widget | ID | Type | Data Source |
|--------|----|------|-------------|
| Product Search | prod-search | Search Input | Client-side + API |
| Filter Panel | prod-filters | Dropdown Group | Static options + API |
| Product Grid Card | prod-grid-card | Card | GET /vendors/products |
| Product Table Row | prod-table-row | Row | GET /vendors/products |
| Bulk Actions Bar | prod-bulk-actions | Action Bar | Client state |
| Pagination | prod-pagination | Pagination | Query param |
| View Toggle | prod-view-toggle | Toggle Group | Local state |
| Empty State | prod-empty | Illustration + CTA | — |

### 9. Widget Specification — Product Grid Card

`	sx
interface ProductGridCardProps {
  product: {
    id: string;
    nameAr: string;
    nameEn: string;
    sku: string;
    thumbnail: string;
    price: number;
    salePrice?: number;
    stock: number;
    status: 'active' | 'draft' | 'out_of_stock' | 'archived';
    rating: number;
    reviewCount: number;
  };
  selected: boolean;
  onSelect: (id: string) => void;
  onEdit: (id: string) => void;
  onDuplicate: (id: string) => void;
  onToggleStatus: (id: string) => void;
  onDelete: (id: string) => void;
}
`

### 10. Data Requirements

| Endpoint | Method | Query Params | Response | Cache TTL |
|----------|--------|-------------|----------|-----------|
| /api/v1/vendors/products | GET | page, limit, search, category, status, stock, sort, order | Paginated<Product> | 30s |
| /api/v1/vendors/products/bulk-status | PATCH | — | { ids, status } | — |
| /api/v1/vendors/products/bulk-delete | DELETE | — | { ids } | — |
| /api/v1/vendors/products/export | GET | ormat, filters | CSV/XLSX file | — |
| /api/v1/vendors/categories | GET | — | Category[] | 3600s |

### 11. Data Loading Strategy

1. **Initial:** Skeleton grid (10 placeholders) renders immediately.
2. **API Response:** Replace skeletons with product cards.
3. **Search:** Debounced 300ms after last keystroke.
4. **Filters:** Apply instantly with loading spinner overlay on grid.
5. **Pagination:** Client-side route change, no full reload.

### 12. User Actions

| Action | Trigger | Target | Permission |
|--------|---------|--------|------------|
| Add Product | Button click | VP-PR-002 | products:create |
| Edit Product | Row menu → Edit | VP-PR-002 | products:update |
| View Product | Row click | VP-PR-003 | products:read |
| Duplicate Product | Row menu → Duplicate | VP-PR-002 (prefilled) | products:create |
| Toggle Status | Row menu → Toggle | Modal confirm | products:update |
| Manage Inventory | Row menu → Inventory | VP-IV-001 | inventory:update |
| Bulk Status Change | Select → Bulk action | Confirmation modal | products:update |
| Bulk Delete | Select → Bulk action | Confirmation modal | products:update |
| Export | Bulk action → Export | Download file | products:read |

### 13. Form Specification

**Bulk Status Change Modal:**
`	sx
interface BulkStatusForm {
  ids: string[];
  status: 'active' | 'draft' | 'archived';
  reason?: string;  // Required if archiving
}
`

### 14. Validation Rules

| Rule | Field | Message |
|------|-------|---------|
| Archiving requires reason | eason | "يرجى إدخال سبب الأرشفة" / "Reason is required for archiving" |
| Cannot delete active products with pending orders | ids | "لا يمكن حذف المنتجات النشطة ذات الطلبات المعلقة" |
| Max bulk selection | 100 | "يمكنك تحديد 100 منتج كحد أقصى" / "Maximum 100 products" |

### 15. State Machine — Product Lifecycle

`
  ┌─────────┐    Publish    ┌─────────┐
  │  Draft  │──────────────→│  Active  │
  └────┬────┘               └────┬────┘
       │                         │
       │          Unpublish       │
       │←────────────────────────┤
       │                         │
       │              Stock=0     │
       │                    ┌────▼──────┐
       │                    │ Out of    │
       │                    │ Stock     │
       │                    └────┬──────┘
       │                         │
       │          Restock         │
       │←────────────────────────┤
       │                         │
       │    Archive              │ Archive
       │←─────────────┐   ┌─────▼──────┐
  ┌────▼─────┐        │   │ Archived   │
  │ Archived │←───────┘   └────────────┘
  └──────────┘
`

### 16. Error States

| Error | Display |
|-------|---------|
| API failure | Error banner with retry button |
| Search timeout | "لا توجد نتائج" (even if query entered) |
| Bulk action failure | Toast with error details, list of failed items |
| Image load failure | Placeholder image with retry |

### 17. Empty States

| Scenario | Display |
|----------|---------|
| No products | "لم تقم بإضافة أي منتج بعد" + "Add Your First Product" CTA |
| No search results | "لا توجد نتائج لـ '{query}'" + suggested filters |
| No filtered results | "لا توجد منتجات تطابق الفلتر المحدد" + "Clear Filters" |

### 18. Loading States

- Grid: 10 skeleton cards with pulsing image + text blocks.
- Table: 10 skeleton rows.
- Search: Spinner inside search input.
- Bulk action: Progress bar modal.

### 19. Success States

- Product created: Toast "تم إضافة المنتج بنجاح" + auto-navigate to list.
- Status toggled: Row status badge animates to new state.
- Bulk delete: Toast "{n} products deleted" with undo option (5s).

### 20. Permission-Based UI

| Permission | UI Impact |
|------------|-----------|
| No products:create | "إضافة منتج" button hidden |
| No products:update | Edit/toggle/archive actions hidden in row menu |
| Read-only vendor | Table becomes read-only, no checkboxes for selection |

### 21. Security UX

- Product prices show full values only to vendor owner.
- SKU generation is server-side only (vendor cannot forge).
- Duplicate products get new IDs and "Copy of" prefix.

### 22. RTL/LTR Behavior

- Grid cards flow right-to-left (first card on right).
- Table columns mirror: actions column on left in RTL.
- Search input has RTL text alignment.
- Price displayed as "3,500 ر.ي" (right-aligned).

### 23. Internationalization

| Key | Arabic | English |
|-----|--------|---------|
| products.title | المنتجات | Products |
| products.addProduct | إضافة منتج | Add Product |
| products.search | بحث عن منتج... | Search products... |
| products.filter.category | التصنيف | Category |
| products.filter.status | الحالة | Status |
| products.filter.stock | المخزون | Stock |
| products.status.active | نشط | Active |
| products.status.draft | مسودة | Draft |
| products.status.oos | نفد المخزون | Out of Stock |
| products.status.archived | مؤرشف | Archived |
| products.stock.low | منخفض | Low |
| products.stock.out | نفد | Out |
| products.empty | لم تقم بإضافة أي منتج بعد | No products yet |
| products.duplicate | تكرار | Duplicate |
| products.edit | تعديل | Edit |
| products.archive | أرشفة | Archive |
| products.delete | حذف | Delete |
| products.bulk.changeStatus | تغيير الحالة | Change Status |
| products.bulk.delete | حذف المحدد | Delete Selected |
| products.bulk.export | تصدير | Export |

### 24. Accessibility

- Table uses ole="grid" with ria-rowcount, ria-colcount.
- Grid cards have ole="article" with ria-label for product name.
- Checkbox selection: ria-selected state managed.
- Status badges have ole="status" with full text description.
- Keyboard: Arrow keys navigate grid, Enter opens detail, Space selects.

### 25. Performance Requirements

| Metric | Target |
|--------|--------|
| List render (10 items) | < 300ms |
| Search debounce | 300ms |
| Filter apply | < 200ms |
| Image lazy loading | Below fold cards |
| Infinite scroll option | For > 100 products |

### 26. Analytics Events

| Event | Properties |
|-------|------------|
| product_list_view | { viewMode, filters } |
| product_search | { query, resultCount } |
| product_filter_apply | { filterType, filterValue } |
| product_action_click | { action, productId } |
| product_bulk_action | { action, count } |
| product_export | { format, count } |

### 27. Notifications

| Trigger | Type |
|---------|------|
| Product approved by admin | Toast on list |
| Product rejected by admin | Toast + badge on affected product |
| Bulk action complete | Toast with summary |

### 28. Cross-Page Relationships

`
VP-PR-001 ←→ VP-PR-002 (Create/Edit)
VP-PR-001 ──→ VP-PR-003 (Detail)
VP-PR-001 ──→ VP-IV-001 (Inventory)
VP-PR-001 ←── VP-DB-001 (Dashboard link)
VP-PR-001 ←── VP-PR-002 (After save redirect)
VP-PR-001 ←── VP-PR-003 (Back button)
`

### 29. Visual Preview

Key characteristics:
- Cards have subtle hover effect (hover:shadow-md).
- Status badges use semantic colors: green=active, gray=draft, red=oos, blue=archived.
- Grid toggle icon switches between LayoutGrid and List Lucide icons.
- Pagination shows Arabic numerals in RTL mode.

---

## VP-PR-002: Product Create/Edit

### 1. Page Metadata

| Field | Value |
|-------|-------|
| **ID** | VP-PR-002 |
| **Page Name** | Product Create/Edit |
| **Portal** | Vendor Panel |
| **Platform** | Desktop |
| **Module** | Products |
| **Category** | Management |
| **Priority** | P0 |
| **Auth Required** | Yes |
| **Vendor Roles** | Vendor (all) |
| **Permissions** | products:create (create mode), products:update (edit mode) |
| **URL** | /vendor/products/new, /vendor/products/:id/edit |
| **Parent Page** | VP-PR-001 |
| **Requirements References** | REQ-VN-PROD-004, REQ-VN-PROD-005, REQ-VN-PROD-006 |
| **Status** | Draft |

### 2. Page Purpose

Multi-step form for creating or editing a product listing. Supports rich product information entry including multilingual names/descriptions, pricing, images, variants, SEO metadata, and shipping configuration.

### 3. Entry Points

| Source | Trigger |
|--------|---------|
| VP-PR-001 | "Add Product" button |
| VP-PR-001 | Row menu → Edit |
| VP-PR-001 | Row menu → Duplicate |
| Deep Link | Direct URL with product ID |

### 4. Exit Points

| Destination | Trigger |
|-------------|---------|
| VP-PR-001 | Save & Exit / Back button |
| VP-PR-003 | Save & View |
| VP-PR-001 | Discard confirmation → Cancel |

### 5. Information Architecture

`
Product Form
├── Progress Stepper
│   ├── Step 1: Basic Info
│   ├── Step 2: Pricing & Inventory
│   ├── Step 3: Images & Media
│   ├── Step 4: Variants (optional)
│   ├── Step 5: Shipping
│   └── Step 6: SEO & Publish
├── Step Content (dynamic)
├── Navigation
│   ├── [ رجوع / Back ]
│   ├── [ حفظ كمسودة / Save as Draft ]
│   └── [ نشر / Publish ] (or [ تحديث / Update ])
├── Sidebar
│   ├── Publish Settings
│   ├── Category Selector
│   └── Product Status
└── Validation Summary (on error)
`

### 6. Layout Structure

`
┌─────────────────────────────────────────────────────────────────────────────┐
│  📦 إضافة منتج جديد                                          [ رجوع ]   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌──────────────────────────────────────────────┐ ┌───────────────────────┐ │
│  │  ●───●───●───○───○───○                       │ │  الشريط الجانبي       │ │
│  │  1    2    3    4    5    6                   │ │  ┌─────────────────┐ │ │
│  │                                               │ │  │ الحالة: مسودة   │ │ │
│  │  ── المعلومات الأساسية ───────────────────    │ │  └─────────────────┘ │ │
│  │                                               │ │                      │ │
│  │  اسم المنتج (عربي) *                          │ │  التصنيف             │ │
│  │  ┌──────────────────────────────────────────┐ │ │  ┌─────────────────┐ │ │
│  │  │ فأرة لاسلكية بلوتوث 5.0                  │ │ │  │ > إلكترونيات    │ │ │
│  │  └──────────────────────────────────────────┘ │ │  │   > أجهزة إدخال │ │ │
│  │                                               │ │  │     ☑ فأرة      │ │ │
│  │  اسم المنتج (إنجليزي) *                       │ │  └─────────────────┘ │ │
│  │  ┌──────────────────────────────────────────┐ │ │                      │ │
│  │  │ Bluetooth Wireless Mouse 5.0             │ │ │  الصور              │ │ │
│  │  └──────────────────────────────────────────┘ │ │  ┌─────────────────┐ │ │
│  │                                               │ │  │ 1/6 صور مرفوعة  │ │ │
│  │  الوصف (عربي) *                               │ │  └─────────────────┘ │ │
│  │  ┌──────────────────────────────────────────┐ │ │                      │ │
│  │  │ 📎 B  I  U  🔗  📷                      │ │ │  ┌─────────────────┐ │ │
│  │  │ فأرة لاسلكية بتقنية البلوتوث 5.0...     │ │ │  │  حفظ كمسودة     │ │ │
│  │  │                                          │ │ │  │  نشر المنتج     │ │ │
│  │  │                                          │ │ │  └─────────────────┘ │ │
│  │  └──────────────────────────────────────────┘ │ └───────────────────────┘ │
│  │                                               │                           │
│  │  ── التسعير والمخزون ────────────────────     │                           │
│  │                                               │                           │
│  │  السعر (ر.ي) *                                │                           │
│  │  ┌──────────────────────────────────────────┐ │                           │
│  │  │ 3,500                                   │ │                           │
│  │  └──────────────────────────────────────────┘ │                           │
│  │                                               │                           │
│  │  سعر الخصم (ر.ي)                              │                           │
│  │  ┌──────────────────────────────────────────┐ │                           │
│  │  │ 2,800                                   │ │                           │
│  │  └──────────────────────────────────────────┘ │                           │
│  │                                               │                           │
│  │  المخزون *    │  رقم المنتج (SKU) *           │                           │
│  │  ┌──────────┐│  ┌──────────────────────────┐ │                           │
│  │  │ 45       ││  │ WL-MOUSE-001             │ │                           │
│  │  └──────────┘│  └──────────────────────────┘ │                           │
│  │                                               │                           │
│  │  [ رجوع ]                    [ حفظ مسودة ] [ نشر ]                       │
│  └──────────────────────────────────────────────┘                           │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
`

### 7. Responsive Design

| Breakpoint | Behavior |
|------------|----------|
| **Desktop (≥1024px)** | 2-column: form left (70%), sidebar right (30%). Stepper horizontal. |
| **Tablet (768–1023px)** | Single column, sidebar moves below form. Stepper horizontal. |
| **Mobile (<768px)** | Not supported. |

### 8. Widget Inventory

| Widget | ID | Type |
|--------|----|------|
| Progress Stepper | prod-stepper | Stepper |
| Text Input (AR) | prod-name-ar | Text Input |
| Text Input (EN) | prod-name-en | Text Input |
| Rich Text Editor (AR) | prod-desc-ar | Textarea + toolbar |
| Rich Text Editor (EN) | prod-desc-en | Textarea + toolbar |
| Number Input | prod-price | Number Input |
| Image Uploader | prod-images | Drag & Drop Upload |
| Category Tree Selector | prod-category | Tree Select |
| Variant Builder | prod-variants | Dynamic Form |
| Shipping Config | prod-shipping | Form Group |
| SEO Fields | prod-seo | Collapsible Section |
| Publish Settings | prod-publish | Sidebar Widget |

### 9. Widget Specification — Image Uploader

`	sx
interface ImageUploaderProps {
  images: ProductImage[];
  maxImages: number;  // 6
  maxFileSize: number; // 5MB
  acceptedTypes: string[]; // ['image/jpeg', 'image/png', 'image/webp']
  onUpload: (files: File[]) => Promise<ImageUploadResult[]>;
  onReorder: (orderedIds: string[]) => void;
  onRemove: (id: string) => void;
  onSetPrimary: (id: string) => void;
}
`

Features:
- Drag & drop zone + file picker button.
- Image preview with thumbnail grid.
- Drag to reorder (dnd-kit library).
- Primary image indicator (star icon).
- Crop/rotate tools in modal.
- Upload progress bar per image.
- RTL-aware: drag handles flip sides.

### 10. Data Requirements

| Endpoint | Method | Used When |
|----------|--------|-----------|
| GET /api/v1/vendors/products/:id | GET | Edit mode: load existing product |
| POST /api/v1/vendors/products | POST | Create new product |
| PUT /api/v1/vendors/products/:id | PUT | Update existing product |
| POST /api/v1/vendors/products/:id/images | POST | Upload images |
| DELETE /api/v1/vendors/products/:id/images/:imageId | DELETE | Remove image |
| PUT /api/v1/vendors/products/:id/images/reorder | PUT | Reorder images |
| GET /api/v1/vendors/categories | GET | Category tree |
| GET /api/v1/vendors/products/:id/duplicate | POST | Duplicate product |

### 11. Data Loading Strategy

- **Create mode:** Form loads empty, categories fetched in parallel.
- **Edit mode:** Product data fetched on mount, form pre-filled.
- **Image uploads:** Chunked upload with progress per image.
- **Auto-save:** Draft auto-saved every 60s if form has unsaved changes.
- **Dirty form:** Unsaved changes detected via eact-hook-form dirty state.

### 12. User Actions

| Action | Target |
|--------|--------|
| Next Step | Stepper next |
| Previous Step | Stepper back |
| Save as Draft | POST /vendors/products with status: draft |
| Publish | POST /vendors/products with status: active |
| Update | PUT /vendors/products/:id |
| Save & View | Save + redirect to VP-PR-003 |
| Discard | Confirm modal → VP-PR-001 |

### 13. Form Specification

**Step 1 — Basic Info:**
`	sx
interface BasicInfoForm {
  nameAr: string;          // Required, max 200 chars
  nameEn: string;          // Required, max 200 chars
  descriptionAr: string;   // Required, max 5000 chars (rich text)
  descriptionEn: string;   // Optional, max 5000 chars (rich text)
  shortDescriptionAr: string; // Optional, max 300 chars
  shortDescriptionEn: string; // Optional, max 300 chars
  barcode?: string;        // Optional, max 50 chars
}
`

**Step 2 — Pricing & Inventory:**
`	sx
interface PricingForm {
  price: number;           // Required, > 0
  salePrice?: number;      // Optional, < price
  costPrice?: number;      // Optional, >= 0
  currency: 'YER';
  stock: number;           // Required, >= 0
  sku: string;             // Required, unique, max 50 chars
  trackInventory: boolean; // Default: true
  allowBackorder: boolean; // Default: false
  lowStockThreshold: number; // Default: 10
}
`

**Step 3 — Images:**
`	sx
interface ImagesForm {
  images: ProductImage[];  // Min 1, Max 6
  primaryImageIndex: number;
}
`

**Step 4 — Variants (Optional):**
`	sx
interface VariantsForm {
  hasVariants: boolean;
  variantAttributes: VariantAttribute[];
  variants: Variant[];
}
`

**Step 5 — Shipping:**
`	sx
interface ShippingForm {
  weight?: number;         // kg
  dimensions?: {
    length: number;
    width: number;
    height: number;
  };
  shippingClass: 'standard' | 'heavy' | 'fragile' | 'none';
  freeShipping: boolean;
  estimatedDays: { min: number; max: number };
}
`

**Step 6 — SEO & Publish:**
`	sx
interface SEOForm {
  metaTitleAr?: string;    // max 60 chars
  metaTitleEn?: string;    // max 60 chars
  metaDescriptionAr?: string; // max 160 chars
  metaDescriptionEn?: string; // max 160 chars
  slug?: string;           // auto-generated, editable
  tags: string[];
  publishNow: boolean;
  scheduledAt?: string;    // ISO date, if scheduled publish
}
`

### 14. Validation Rules

| Rule | Field | Message |
|------|-------|---------|
| Required field | 
ameAr | "اسم المنتج مطلوب بالعربي" |
| Required field | 
ameEn | "Product name is required in English" |
| Max length | 
ameAr | "الاسم يجب ألا يتجاوز 200 حرف" |
| Required field | descriptionAr | "الوصف مطلوب بالعربي" |
| Min length | descriptionAr | "الوصف يجب أن يكون 20 حرف على الأقل" |
| Required field | price | "السعر مطلوب" |
| Min value | price | "السعر يجب أن يكون أكبر من 0" |
| salePrice < price | salePrice | "سعر الخصم يجب أن يكون أقل من السعر الأصلي" |
| Required field | sku | "رقم المنتج (SKU) مطلوب" |
| Unique SKU | sku | "رقم SKU مستخدم بالفعل" |
| Min images | images | "يجب إضافة صورة واحدة على الأقل" |
| Max images | images | "لا يمكن إضافة أكثر من 6 صور" |
| Image size | images | "حجم الصورة يجب أن لا يتجاوز 5 ميجابايت" |
| Image type | images | "يُسمح فقط بصور JPEG, PNG, WEBP" |
| Required stock | stock | "المخزون مطلوب" |
| Non-negative stock | stock | "المخزون لا يمكن أن يكون سالباً" |

### 15. State Machine — Form State

`
┌──────────┐  Start    ┌──────────┐
│  Pristine │─────────→│  Dirty   │
└──────────┘           └────┬─────┘
                            │
              ┌─────────────┼─────────────┐
              │             │             │
         Auto-save     Save Draft    Publish
              │             │             │
         ┌────▼────┐  ┌────▼────┐  ┌────▼────┐
         │ Saving  │  │ Saving  │  │ Saving  │
         └────┬────┘  └────┬────┘  └────┬────┘
              │             │             │
         ┌────▼────┐  ┌────▼────┐  ┌────▼────┐
         │ Saved   │  │ Draft   │  │Active/  │
         │ (dirty) │  │ Saved   │  │ Draft   │
         └─────────┘  └─────────┘  └─────────┘
`

### 16. Error States

| Error | Display |
|-------|---------|
| Validation failure | Inline errors below each field, summary at top |
| Image upload failure | Per-image error icon with retry |
| SKU conflict | Inline error + suggestion for alternative SKU |
| Network error during save | Toast with retry + auto-save retry |
| Session expired during form | Modal: "Session expired, save draft?" → redirect |

### 17. Empty States

| Scenario | Display |
|----------|---------|
| No categories loaded | "جاري تحميل التصنيفات..." or retry |
| No images uploaded | Drag & drop zone with illustration |

### 18. Loading States

- Initial load (edit mode): Full-page skeleton matching form layout.
- Image upload: Per-image progress bar.
- Form submission: Button spinner, disable all inputs.
- Category tree: Skeleton tree nodes.

### 19. Success States

- Draft saved: Toast "تم الحفظ كمسودة ✓"
- Published: Toast "تم نشر المنتج بنجاح ✓" + redirect to VP-PR-003
- Updated: Toast "تم تحديث المنتج ✓"

### 20. Permission-Based UI

| Permission | UI Impact |
|------------|-----------|
| products:create only | Create mode only, no edit |
| products:update only | Edit mode only, pre-filled |
| Both | Create + Edit modes available |

### 21. Security UX

- SKU generation has uniqueness check on blur.
- Price cannot be negative (client + server validation).
- Image EXIF data stripped on upload.
- XSS prevention: Rich text editor sanitizes HTML.

### 22. RTL/LTR Behavior

- Form labels always right-aligned.
- Text inputs always RTL-aligned for Arabic fields.
- English name/description fields are LTR-aligned within RTL form.
- Price input: currency symbol on right side in RTL.
- Stepper progress flows right-to-left.

### 23. Internationalization

| Key | Arabic | English |
|-----|--------|---------|
| productForm.title.new | إضافة منتج جديد | Add New Product |
| productForm.title.edit | تعديل المنتج | Edit Product |
| productForm.step.basic | المعلومات الأساسية | Basic Info |
| productForm.step.pricing | التسعير والمخزون | Pricing & Inventory |
| productForm.step.images | الصور والوسائط | Images & Media |
| productForm.step.variants | المتغيرات | Variants |
| productForm.step.shipping | الشحن | Shipping |
| productForm.step.seo | SEO والنشر | SEO & Publish |
| productForm.saveDraft | حفظ كمسودة | Save as Draft |
| productForm.publish | نشر المنتج | Publish Product |
| productForm.update | تحديث المنتج | Update Product |

### 24. Accessibility

- All form fields have associated <label> elements.
- Error messages linked via ria-describedby.
- Required fields marked with ria-required="true".
- Stepper uses ole="tablist" with ole="tab" per step.
- Image uploader has ria-label for each image with drag handle instructions.
- Form navigation: Tab moves through fields, Enter submits active step.

### 25. Performance Requirements

| Metric | Target |
|--------|--------|
| Form mount (create) | < 200ms |
| Form mount (edit, load data) | < 800ms |
| Image upload (per image) | < 3s (5MB) |
| Auto-save | < 1s |
| Validation feedback | < 100ms (client-side) |

### 26. Analytics Events

| Event | Properties |
|-------|------------|
| product_form_start | { mode: 'create' | 'edit' } |
| product_form_step_complete | { step, timeSpent } |
| product_form_image_upload | { imageCount, totalSize } |
| product_form_submit | { mode, variantCount, imageCount } |
| product_form_auto_save | { step } |
| product_form_abandon | { lastStep, timeSpent } |

### 27. Notifications

| Trigger | Type |
|---------|------|
| Auto-save success | Subtle toast (bottom-right, 3s) |
| SKU conflict detected | Inline warning |
| Image upload complete | Inline success checkmark |

### 28. Cross-Page Relationships

`
VP-PR-002 ←── VP-PR-001 (Create/Edit trigger)
VP-PR-002 ──→ VP-PR-001 (Save & Exit)
VP-PR-002 ──→ VP-PR-003 (Save & View)
VP-PR-002 ──→ VP-PR-001 (Discard)
VP-PR-002 ←── VP-PR-001 (Duplicate prefill)
`

### 29. Visual Preview

Key characteristics:
- Green progress indicator on active/completed steps.
- Rich text editor uses minimal toolbar (bold, italic, link, image).
- Image upload zone has dashed border, green accent on hover.
- Sidebar is sticky (position: sticky, top: 80px).
- Form validation errors appear in red with ⚠ icon.

---

## VP-PR-003: Product Detail

### 1. Page Metadata

| Field | Value |
|-------|-------|
| **ID** | VP-PR-003 |
| **Page Name** | Product Detail |
| **Portal** | Vendor Panel |
| **Platform** | Desktop |
| **Module** | Products |
| **Category** | Management |
| **Priority** | P1 |
| **Auth Required** | Yes |
| **Vendor Roles** | Vendor (all) |
| **Permissions** | products:read |
| **URL** | /vendor/products/:id |
| **Parent Page** | VP-PR-001 |
| **Requirements References** | REQ-VN-PROD-007 |
| **Status** | Draft |

### 2. Page Purpose

Display complete product information in read-only mode with contextual actions. Show product preview as it appears to customers, along with vendor-specific management data.

### 3. Entry Points

| Source | Trigger |
|--------|---------|
| VP-PR-001 | Row click |
| VP-PR-002 | Save & View |
| Deep Link | Direct URL |

### 4. Exit Points

| Destination | Trigger |
|-------------|---------|
| VP-PR-001 | Back button / breadcrumb |
| VP-PR-002 | Edit button |
| VP-IV-001 | Manage Inventory button |

### 5. Information Architecture

`
Product Detail
├── Breadcrumb: Products > Product Name
├── Header
│   ├── Product Name (AR + EN)
│   ├── Status Badge
│   ├── Created / Updated dates
│   └── Action Buttons (Edit, Duplicate, Archive)
├── Image Gallery
│   ├── Primary Image (large)
│   └── Thumbnail strip
├── Product Info
│   ├── Description (AR)
│   ├── Description (EN)
│   └── Barcode
├── Pricing Card
│   ├── Current Price
│   ├── Sale Price (if applicable)
│   └── Price History chart
├── Inventory Card
│   ├── Stock Level
│   ├── SKU
│   ├── Low Stock Threshold
│   └── Stock History chart
├── Category & Tags
├── Shipping Info
├── SEO Metadata
├── Reviews Section (read-only)
└── Activity Log
`

### 6. Layout Structure

`
┌─────────────────────────────────────────────────────────────────────────────┐
│  ← المنتجات / فأرة لاسلكية بلوتوث 5.0                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌──────────────────────────┐ ┌──────────────────────────────────────────┐  │
│  │  [صورة رئيسية كبيرة]     │ │  فأرة لاسلكية بلوتوث 5.0                │  │
│  │                          │ │  Bluetooth Wireless Mouse 5.0            │  │
│  │                          │ │                                          │  │
│  │                          │ │  ● نشط    SKU: WL-MOUSE-001             │  │
│  │                          │ │  تم الإنشاء: 2026/09/01                 │  │
│  │                          │ │  آخر تحديث: 2026/09/12                  │  │
│  │                          │ │                                          │  │
│  │  [صورة] [صورة] [صورة]   │ │  ┌──────────────────────────────────┐   │  │
│  │  [صورة] [صورة]           │ │  │  السعر: 3,500 ر.ي               │   │  │
│  │                          │ │  │  سعر الخصم: 2,800 ر.ي           │   │  │
│  └──────────────────────────┘ │  │  التخفيض: 20%                    │   │  │
│                               │  └──────────────────────────────────┘   │  │
│                               │                                          │  │
│                               │  ┌──────────────────────────────────┐   │  │
│                               │  │  المخزون: 45 وحدة              │   │  │
│                               │  │  الحد الأدنى: 10                │   │  │
│                               │  │  الحالة: متوفر                  │   │  │
│                               │  └──────────────────────────────────┘   │  │
│                               │                                          │  │
│                               │  [ تعديل ✏️ ]  [ تكرار 📋 ]  [ أرشفة ]  │  │
│                               └──────────────────────────────────────────┘  │
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │  الوصف (عربي)                                                       │   │
│  │  فأرة لاسلكية بتقنية البلوتوث 5.0 مع دقة عالية 1600 DPI...        │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  ┌──────────────────────────┐ ┌──────────────────────────────────────────┐  │
│  │  التصنيف                 │ │  الشحن                                    │  │
│  │  إلكترونيات > أجهزة     │ │  الوزن: 0.15 كجم                        │  │
│  │  إدخال > فأرة           │ │  الأبعاد: 12×8×4 سم                     │  │
│  │                          │ │  فئة الشحن: قياسي                       │  │
│  │  الوسوم: فأرة, لاسلكي,  │ │  شحن مجاني: لا                          │  │
│  │  بلوتوث                  │ │  مدة التوصيل: 3-5 أيام                  │  │
│  └──────────────────────────┘ └──────────────────────────────────────────┘  │
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │  التقييمات (23 تقييم)     ★ 4.7                                     │   │
│  │  ★★★★★  ████████░░  67%                                              │   │
│  │  ★★★★☆  ███░░░░░░░  18%                                              │   │
│  │  ★★★☆☆  ██░░░░░░░░   9%                                              │   │
│  │  ★★☆☆☆  █░░░░░░░░░   4%                                              │   │
│  │  ★☆☆☆☆  ░░░░░░░░░░   2%                                              │   │
│  │  [ عرض جميع التقييمات ]                                              │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
`

### 7. Responsive Design

| Breakpoint | Behavior |
|------------|----------|
| **Desktop (≥1024px)** | 2-column top, full-width below. Gallery left, info right. |
| **Tablet (768–1023px)** | Single column, gallery on top, info below. |
| **Mobile (<768px)** | Not supported. |

### 8. Widget Inventory

| Widget | ID | Type |
|--------|----|------|
| Image Gallery | prod-gallery | Gallery |
| Status Badge | prod-status-badge | Badge |
| Price Card | prod-price-card | Card |
| Inventory Card | prod-inventory-card | Card |
| Description Panel | prod-description | Text Block |
| Category Display | prod-category | Breadcrumb |
| Shipping Info | prod-shipping | Info Grid |
| Reviews Summary | prod-reviews-summary | Chart + Stats |
| Activity Log | prod-activity | Timeline |

### 9. Widget Specification — Image Gallery

`	sx
interface ImageGalleryProps {
  images: ProductImage[];
  primaryIndex: number;
  onImageClick: (index: number) => void; // Opens lightbox
}
`

### 10. Data Requirements

| Endpoint | Method | Response |
|----------|--------|----------|
| GET /api/v1/vendors/products/:id | GET | Product |
| GET /api/v1/vendors/products/:id/reviews | GET | Review[] |
| GET /api/v1/vendors/products/:id/activity | GET | ActivityLog[] |

### 11. Data Loading Strategy

- Product data: Critical path, skeleton on load.
- Reviews: Deferred below fold.
- Activity log: Deferred, lazy loaded on scroll.

### 12. User Actions

| Action | Target | Permission |
|--------|--------|------------|
| Edit | VP-PR-002 | products:update |
| Duplicate | VP-PR-002 (prefilled) | products:create |
| Archive | Confirmation modal | products:update |
| Manage Inventory | VP-IV-001 | inventory:update |
| View Reviews | VP-RV-001 | eviews:read |

### 13. Form Specification

Read-only page. Archive modal has optional reason field:
`	sx
interface ArchiveForm {
  reason: string; // Required, max 500 chars
}
`

### 14. Validation Rules

N/A (read-only). Archive form requires reason.

### 15. State Machine

Product lifecycle as defined in VP-PR-001 Section 15.

### 16. Error States

| Error | Display |
|-------|---------|
| Product not found | 404 page with "المنتج غير موجود" message |
| Permission denied | "ليس لديك صلاحية لعرض هذا المنتج" |
| API failure | Error card with retry |

### 17. Empty States

| Scenario | Display |
|----------|---------|
| No reviews | "لا توجد تقييمات بعد" |
| No activity | "لا يوجد سجل نشاط" |

### 18. Loading States

Full-page skeleton matching layout structure.

### 19. Success States

- Archive success: Redirect to VP-PR-001 with toast.
- Duplicate success: Redirect to VP-PR-002 with prefilled data.

### 20. Permission-Based UI

| Permission | UI Impact |
|------------|-----------|
| products:read only | No Edit/Duplicate/Archive buttons |
| products:update | Edit + Archive buttons visible |
| products:create | Duplicate button visible |

### 21. Security UX

- Price history shows only to vendor owner.
- Activity log shows IP addresses of admin changes.

### 22. RTL/LTR Behavior

- Gallery thumbnails flow right-to-left.
- Description text respects language direction.
- Price history chart X-axis flows right-to-left.

### 23. Internationalization

| Key | Arabic | English |
|-----|--------|---------|
| productDetail.title | تفاصيل المنتج | Product Detail |
| productDetail.edit | تعديل | Edit |
| productDetail.duplicate | تكرار | Duplicate |
| productDetail.archive | أرشفة | Archive |
| productDetail.price | السعر | Price |
| productDetail.salePrice | سعر الخصم | Sale Price |
| productDetail.discount | التخفيض | Discount |
| productDetail.stock | المخزون | Stock |
| productDetail.sku | رقم المنتج | SKU |
| productDetail.description | الوصف | Description |
| productDetail.reviews | التقييمات | Reviews |
| productDetail.activity | سجل النشاط | Activity Log |

### 24. Accessibility

- Image gallery has ria-label on each image.
- Lightbox has ole="dialog" with focus trap.
- Status badge has ria-label with full status text.
- Price history chart has accessible data table alternative.

### 25. Performance Requirements

| Metric | Target |
|--------|--------|
| Page load | < 1.0s |
| Image gallery render | < 300ms |
| Lightbox open | < 150ms |

### 26. Analytics Events

| Event | Properties |
|-------|------------|
| product_detail_view | { productId } |
| product_detail_action | { action, productId } |
| product_detail_image_view | { productId, imageIndex } |

### 27. Notifications

| Trigger | Type |
|---------|------|
| Product status changed by admin | Real-time update on page |
| New review received | Badge update on reviews section |

### 28. Cross-Page Relationships

`
VP-PR-003 ←── VP-PR-001 (Row click)
VP-PR-003 ←── VP-PR-002 (Save & View)
VP-PR-003 ──→ VP-PR-002 (Edit)
VP-PR-003 ──→ VP-PR-001 (Back)
VP-PR-003 ──→ VP-IV-001 (Manage Inventory)
VP-PR-003 ──→ VP-RV-001 (View Reviews)
`

### 29. Visual Preview

Key characteristics:
- Large hero image with smooth zoom on hover.
- Status badge with subtle shadow.
- Price card has green border-left accent.
- Inventory card has amber border-left when stock is low.
- Reviews section uses horizontal bar chart.

---

## VP-IV-001: Inventory Management

### 1. Page Metadata

| Field | Value |
|-------|-------|
| **ID** | VP-IV-001 |
| **Page Name** | Inventory Management |
| **Portal** | Vendor Panel |
| **Platform** | Desktop |
| **Module** | Inventory |
| **Category** | Management |
| **Priority** | P0 |
| **Auth Required** | Yes |
| **Vendor Roles** | Vendor (all) |
| **Permissions** | inventory:read, inventory:update (own) |
| **URL** | /vendor/inventory |
| **Parent Page** | VP-DB-001 |
| **Requirements References** | REQ-VN-INV-001, REQ-VN-INV-002 |
| **Status** | Draft |

### 2. Page Purpose

Centralized inventory management view. Allow vendors to update stock levels, set low-stock thresholds, track inventory changes, and perform bulk stock updates. Unlike the product list which focuses on product details, this page focuses purely on stock levels and inventory operations.

### 3. Entry Points

| Source | Trigger |
|--------|---------|
| Sidebar Navigation | Click "Inventory" |
| VP-DB-001 | "Low Stock Alerts" widget |
| VP-PR-001 | Row menu → Manage Inventory |
| VP-PR-003 | Manage Inventory button |

### 4. Exit Points

| Destination | Trigger |
|-------------|---------|
| VP-PR-003 | Product name click |
| VP-PR-001 | Back to products |
| VP-DB-001 | Dashboard link |

### 5. Information Architecture

`
Inventory Management
├── Header
│   ├── Title: "إدارة المخزون"
│   ├── Summary Strip
│   │   ├── Total SKUs
│   │   ├── Total Stock Value (YER)
│   │   ├── Low Stock Count
│   │   └── Out of Stock Count
│   └── Bulk Actions (Export, Bulk Update)
├── Toolbar
│   ├── Search (name, SKU)
│   ├── Stock Level Filter (All, In Stock, Low Stock, Out of Stock)
│   ├── Category Filter
│   └── Sort (Name, Stock ASC, Stock DESC, Last Updated)
├── Inventory Table
│   ├── Thumbnail
│   ├── Product Name
│   ├── SKU
│   ├── Current Stock (editable inline)
│   ├── Low Stock Threshold (editable inline)
│   ├── Stock Status (color indicator)
│   ├── Last Restocked (date)
│   ├── Stock Value (calculated)
│   └── Actions (History, Adjust)
├── Bulk Update Panel (slide-over)
│   ├── Upload CSV template
│   ├── Review changes
│   └── Confirm bulk update
└── Stock History Modal
    └── Timeline of stock changes for a product
`

### 6. Layout Structure

`
┌─────────────────────────────────────────────────────────────────────────────┐
│  إدارة المخزون                [تصدير] [تحديث جماعي]                        │
├─────────────────────────────────────────────────────────────────────────────┤
│  ┌────────────┐ ┌────────────┐ ┌────────────┐ ┌────────────┐              │
│  │ إجمالي     │ │ قيمة       │ │ منخفض      │ │ نفد        │              │
│  │ المنتجات   │ │ المخزون    │ │ المخزون    │ │ المخزون    │              │
│  │    89      │ │ 2,450,000  │ │     12     │ │     3      │              │
│  └────────────┘ └────────────┘ └────────────┘ └────────────┘              │
├─────────────────────────────────────────────────────────────────────────────┤
│  🔍 بحث...          [مستوى المخزون ▼] [التصنيف ▼] [ترتيب ▼]              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ☐  [img] فأرة لاسلكية    WL-001   [45] / [10]  ● متوفر  157,500 ر.ي  ⋯ │
│  ☐  [img] كابل USB-C      UC-012   [120] / [20] ● متوفر  144,000 ر.ي  ⋯ │
│  ☐  [img] موزع HDMI       HM-003   [8] / [10]   ⚠ منخفض  38,400 ر.ي   ⋯ │
│  ☐  [img] لوحة مفاتيح     KB-007   [32] / [15]  ● متوفر  284,800 ر.ي  ⋯ │
│  ☐  [img] سماعات          HS-015   [0] / [10]   ○ نفد    0 ر.ي        ⋯ │
│  ☐  [img] ماسح ضوئي       SC-009   [15] / [5]   ● متوفر  180,000 ر.ي  ⋯ │
│  ☐  [img] قلم رقمي        SP-021   [200] / [10] ● متوفر  560,000 ر.ي  ⋯ │
│  ☐  [img] حامل هاتف       PH-004   [89] / [20]  ● متوفر  133,500 ر.ي  ⋯ │
│  ☐  [img] شاحن لاسلكي     CH-018   [56] / [15]  ● متوفر  414,400 ر.ي  ⋯ │
│  ☐  [img] سلك شحن         CC-025   [3] / [10]   ○ نفد    4,500 ر.ي    ⋯ │
│                                                                             │
│  ◀ 1 2 3 ... 9 ▶                              عرض 1-10 من 89              │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
`

### 7. Responsive Design

| Breakpoint | Behavior |
|------------|----------|
| **Desktop (≥1024px)** | Full table with all columns, inline editing enabled. |
| **Tablet (768–1023px)** | Table with fewer columns (hide Stock Value, Last Restocked). Inline editing via modal. |
| **Mobile (<768px)** | Not supported. |

### 8. Widget Inventory

| Widget | ID | Type |
|--------|----|------|
| Summary KPIs | inv-kpi-strip | Metric Cards |
| Inventory Table | inv-table | Data Table |
| Inline Stock Editor | inv-inline-edit | Number Input |
| Stock Status Indicator | inv-status | Color Dot + Text |
| Bulk Update Panel | inv-bulk-panel | Slide-over |
| Stock History Modal | inv-history-modal | Modal |
| CSV Upload | inv-csv-upload | File Upload |

### 9. Widget Specification — Inline Stock Editor

`	sx
interface InlineStockEditorProps {
  productId: string;
  currentValue: number;
  threshold: number;
  onUpdate: (productId: string, newValue: number) => Promise<void>;
  permission: boolean;
}
`

Features:
- Click to edit mode (double-click or pencil icon).
- Number input with +/- buttons.
- Auto-save on blur or Enter key.
- Optimistic update with rollback on error.
- Visual feedback: green flash on success, red flash on error.

### 10. Data Requirements

| Endpoint | Method | Response |
|----------|--------|----------|
| GET /api/v1/vendors/inventory | GET | InventoryItem[] with pagination |
| GET /api/v1/vendors/inventory/summary | GET | KPI summary |
| PUT /api/v1/vendors/inventory/:productId | PUT | Update stock/threshold |
| PATCH /api/v1/vendors/inventory/bulk | PATCH | Bulk stock update |
| GET /api/v1/vendors/inventory/:productId/history | GET | Stock change history |
| POST /api/v1/vendors/inventory/import | POST | CSV import |
| GET /api/v1/vendors/inventory/export | GET | CSV export |

### 11. Data Loading Strategy

1. Summary KPIs load first.
2. Table loads with skeleton rows.
3. Inline edits use optimistic updates.
4. History modal loads on demand.
5. CSV import is async with progress polling.

### 12. User Actions

| Action | Trigger | Permission |
|--------|---------|------------|
| Inline edit stock | Double-click cell | inventory:update |
| View stock history | Row menu → History | inventory:read |
| Adjust stock | Row menu → Adjust | inventory:update |
| Bulk update | Select → Bulk action | inventory:update |
| Export CSV | Export button | inventory:read |
| Import CSV | Import button | inventory:update |

### 13. Form Specification

**Stock Adjustment Modal:**
`	sx
interface StockAdjustmentForm {
  productId: string;
  adjustmentType: 'set' | 'add' | 'subtract';
  quantity: number;
  reason: string; // Required
}
`

**Bulk Update Panel:**
`	sx
interface BulkUpdateForm {
  file: File; // CSV file
  dryRun: boolean; // Preview changes before applying
}
`

### 14. Validation Rules

| Rule | Field | Message |
|------|-------|---------|
| Non-negative stock | quantity | "المخزون لا يمكن أن يكون سالباً" |
| Reason required | eason | "السبب مطلوب" |
| Max adjustment | quantity | "التعديل يتجاوز الحد المسموح" |
| CSV format | ile | "الملف يجب أن يكون بصيغة CSV" |
| CSV column required | — | "الملف يجب أن يحتوي على SKU والكمية" |

### 15. State Machine — Inventory States

`
  ┌──────────────┐
  │   In Stock   │ (stock > threshold)
  └──────┬───────┘
         │
    Stock decreases
         │
  ┌──────▼───────┐
  │  Low Stock   │ (0 < stock ≤ threshold)
  └──────┬───────┘
         │
    Stock = 0
         │
  ┌──────▼───────┐
  │  Out of Stock│ (stock = 0)
  └──────┬───────┘
         │
    Restock
         │
  ┌──────▼───────┐
  │  In Stock    │
  └──────────────┘
`

### 16. Error States

| Error | Display |
|-------|---------|
| Inline edit failure | Red flash, revert to previous value, toast error |
| Bulk import failure | Error report with line-by-line failures |
| API failure | Error banner with retry |
| CSV parse error | "الملف غير صالح" with specific error detail |

### 17. Empty States

| Scenario | Display |
|----------|---------|
| No inventory items | "لا توجد منتجات في المخزون" + link to add products |
| No search results | "لا توجد نتائج" |
| Import file empty | "الملف فارغ" |

### 18. Loading States

- Table: Skeleton rows with pulsing cells.
- Summary: Skeleton KPI cards.
- Bulk import: Progress bar with percentage.

### 19. Success States

- Inline edit: Green flash + toast "تم تحديث المخزون ✓"
- Bulk update: Toast "{n} products updated" + refresh table
- Export: Download starts + toast "جاري التصدير"

### 20. Permission-Based UI

| Permission | UI Impact |
|------------|-----------|
| inventory:read only | Inline editing disabled, read-only table |
| inventory:update | Full editing capabilities |
| No permission | Page shows "Access denied" message |

### 21. Security UX

- Bulk import CSV is validated server-side for format and values.
- Stock cannot go below 0 (client + server validation).
- Audit trail maintained for all stock changes.

### 22. RTL/LTR Behavior

- Table columns mirror for RTL.
- Inline edit input is RTL-aligned for Arabic product names.
- Stock value displays as "2,450,000 ر.ي" (right-aligned).

### 23. Internationalization

| Key | Arabic | English |
|-----|--------|---------|
| inventory.title | إدارة المخزون | Inventory Management |
| inventory.totalSKUs | إجمالي المنتجات | Total SKUs |
| inventory.stockValue | قيمة المخزون | Stock Value |
| inventory.lowStock | منخفض المخزون | Low Stock |
| inventory.outOfStock | نفد المخزون | Out of Stock |
| inventory.bulkUpdate | تحديث جماعي | Bulk Update |
| inventory.export | تصدير | Export |
| inventory.import | استيراد | Import |
| inventory.history | سجل التغييرات | Change History |
| inventory.adjust | تعديل المخزون | Adjust Stock |
| inventory.threshold | الحد الأدنى | Threshold |

### 24. Accessibility

- Table uses ole="grid" with keyboard navigation.
- Inline edit cells have ole="gridcell" with ria-label.
- Stock status indicators have ole="status".
- Bulk update panel has ole="dialog" with focus management.
- Keyboard: Enter to start editing, Escape to cancel, Tab to next cell.

### 25. Performance Requirements

| Metric | Target |
|--------|--------|
| Table render (100 rows) | < 500ms |
| Inline edit save | < 300ms |
| CSV import (1000 rows) | < 10s |
| Stock history load | < 500ms |

### 26. Analytics Events

| Event | Properties |
|-------|------------|
| inventory_view | { filterApplied } |
| inventory_inline_edit | { productId, oldValue, newValue } |
| inventory_bulk_update | { method, count } |
| inventory_export | { format, count } |
| inventory_import | { rowCount, successCount, errorCount } |
| inventory_history_view | { productId } |

### 27. Notifications

| Trigger | Type |
|---------|------|
| Stock reaches threshold | Real-time toast |
| Stock reaches zero | Real-time toast + badge |
| Bulk import complete | Toast with summary |
| Import errors found | Toast with error count |

### 28. Cross-Page Relationships

`
VP-IV-001 ←── VP-DB-001 (Low stock alerts)
VP-IV-001 ←── VP-PR-001 (Manage inventory)
VP-IV-001 ←── VP-PR-003 (Manage inventory)
VP-IV-001 ──→ VP-PR-003 (Product name click)
VP-IV-001 ──→ VP-PR-001 (Back to products)
`

### 29. Visual Preview

Key characteristics:
- Table rows with alternating subtle backgrounds.
- Stock status dots: green (>threshold), amber (≤threshold), red (0).
- Inline edit cell has green border on focus.
- Bulk update slide-over enters from right (RTL: from left).

---

## VP-OR-001: Order Management (List)

### 1. Page Metadata

| Field | Value |
|-------|-------|
| **ID** | VP-OR-001 |
| **Page Name** | Order Management (List) |
| **Portal** | Vendor Panel |
| **Platform** | Desktop |
| **Module** | Orders |
| **Category** | Management |
| **Priority** | P0 |
| **Auth Required** | Yes |
| **Vendor Roles** | Vendor (all) |
| **Permissions** | orders:read (own), orders:update (own) |
| **URL** | /vendor/orders |
| **Parent Page** | VP-DB-001 |
| **Requirements References** | REQ-VN-ORD-001, REQ-VN-ORD-002, REQ-VN-ORD-003 |
| **Status** | Draft |

### 2. Page Purpose

Display all orders containing the vendor's products. Enable order status tracking, filtering, and quick status updates. Show order summaries with customer info, amounts, and fulfillment status.

### 3. Entry Points

| Source | Trigger |
|--------|---------|
| Sidebar Navigation | Click "Orders" |
| VP-DB-001 | "View All Orders" / pending orders widget |
| VP-OR-002 | Back button |

### 4. Exit Points

| Destination | Trigger |
|-------------|---------|
| VP-OR-002 | Row click |
| VP-DB-001 | Sidebar Dashboard |
| VP-FN-001 | Financial summary link |

### 5. Information Architecture

`
Order List
├── Header
│   ├── Title: "الطلبات"
│   ├── Order Count Badge
│   └── Quick Filters (tabs)
├── Quick Filter Tabs
│   ├── All
│   ├── Pending (new)
│   ├── Processing
│   ├── Shipped
│   ├── Delivered
│   └── Issues
├── Toolbar
│   ├── Search (order ID, customer name)
│   ├── Date Range Picker
│   ├── Status Filter (multi-select)
│   ├── Amount Range
│   └── Sort (Date, Amount, Status)
├── Order Table
│   ├── Order ID
│   ├── Customer Name
│   ├── Date/Time
│   ├── Items Count
│   ├── Total Amount (YER)
│   ├── Payment Status
│   ├── Fulfillment Status
│   └── Actions
├── Bulk Actions (when selected)
│   ├── Mark as Processing
│   ├── Mark as Shipped
│   └── Print Labels
└── Pagination
`

### 6. Layout Structure

`
┌─────────────────────────────────────────────────────────────────────────────┐
│  الطلبات (234)                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│  [الكل (234)] [جديد (12)] [قيد المعالجة (8)] [تم الشحن (15)] [تم التوصيل (189)] [مشاكل (10)]│
├─────────────────────────────────────────────────────────────────────────────┤
│  🔍 بحث...      [التاريخ ▼] [الحالة ▼] [المبلغ ▼]                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ☐  #ORD-8847  │  أحمد محمد      │ 12/09  │ 3 منتجات │ 12,500 ر.ي │ ✅ تم التوصيل │  ⋯ │
│  ☐  #ORD-8843  │  سعيد علي       │ 12/09  │ 1 منتج   │ 3,500 ر.ي  │ 🚚 تم الشحن   │  ⋯ │
│  ☐  #ORD-8841  │  فاطمة أحمد     │ 11/09  │ 5 منتجات │ 28,900 ر.ي │ 📦 قيد المعالجة│  ⋯ │
│  ☐  #ORD-8839  │  خالد يوسف     │ 11/09  │ 2 منتجات │ 8,200 ر.ي  │ ⏳ جديد       │  ⋯ │
│  ☐  #ORD-8837  │  نورة حسن      │ 11/09  │ 1 منتج   │ 1,200 ر.ي  │ ✅ تم التوصيل │  ⋯ │
│  ☐  #ORD-8835  │  عمر عبدالله    │ 10/09  │ 4 منتجات │ 15,800 ر.ي │ ✅ تم التوصيل │  ⋯ │
│  ☐  #ORD-8833  │  مريم سعيد     │ 10/09  │ 2 منتجات │ 6,700 ر.ي  │ ⚠ مشكلة      │  ⋯ │
│  ☐  #ORD-8831  │  ياسر محمود    │ 10/09  │ 1 منتج   │ 4,800 ر.ي  │ 📦 قيد المعالجة│  ⋯ │
│  ☐  #ORD-8829  │  هدى عبدالرحمن │ 09/09  │ 6 منتجات │ 35,200 ر.ي │ ✅ تم التوصيل │  ⋯ │
│  ☐  #ORD-8827  │  عاصم نبيل     │ 09/09  │ 3 منتجات │ 9,400 ر.ي  │ 🚚 تم الشحن   │  ⋯ │
│                                                                             │
│  ◀ 1 2 3 4 ... 24 ▶                              عرض 1-10 من 234          │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
`

### 7. Responsive Design

| Breakpoint | Behavior |
|------------|----------|
| **Desktop (≥1024px)** | Full table with all columns. Quick filter tabs horizontal. |
| **Tablet (768–1023px)** | Table with fewer columns (hide customer name). Tabs wrap. |
| **Mobile (<768px)** | Not supported. |

### 8. Widget Inventory

| Widget | ID | Type |
|--------|----|------|
| Quick Filter Tabs | order-tabs | Tab Group |
| Search Input | order-search | Search Input |
| Date Range Picker | order-date-range | Date Picker |
| Order Table | order-table | Data Table |
| Status Badge | order-status-badge | Badge |
| Payment Status Badge | order-payment-badge | Badge |
| Bulk Actions Bar | order-bulk | Action Bar |
| Pagination | order-pagination | Pagination |

### 9. Widget Specification — Quick Filter Tabs

`	sx
interface OrderFilterTabsProps {
  activeTab: string;
  counts: Record<string, number>;
  onTabChange: (tab: string) => void;
}
`

Tabs: All, Pending, Processing, Shipped, Delivered, Issues.
Each tab shows count badge (e.g., "جديد (12)").

### 10. Data Requirements

| Endpoint | Method | Response |
|----------|--------|----------|
| GET /api/v1/vendors/orders | GET | Paginated<OrderSummary> |
| GET /api/v1/vendors/orders/counts | GET | Record<OrderStatus, number> |
| PATCH /api/v1/vendors/orders/:id/status | PATCH | Update order status |

### 11. Data Loading Strategy

1. Tab counts load immediately.
2. Default tab (Pending) loads first.
3. Tab switch: optimistic loading with skeleton.
4. Status updates: optimistic with rollback.

### 12. User Actions

| Action | Trigger | Permission |
|--------|---------|------------|
| View order | Row click | orders:read |
| Quick status update | Row menu → Status | orders:update |
| Bulk status update | Select → Bulk action | orders:update |
| Print label | Row menu → Print | orders:read |
| Export orders | Export button | orders:read |

### 13. Form Specification

**Quick Status Update (inline dropdown):**
`	sx
interface QuickStatusUpdate {
  orderId: string;
  status: OrderStatus;
  trackingNumber?: string; // Required when marking as shipped
  note?: string;
}
`

### 14. Validation Rules

| Rule | Field | Message |
|------|-------|---------|
| Tracking required for shipping | 	rackingNumber | "رقم التتبع مطلوب عند الشحن" |
| Valid status transition | status | "لا يمكن التحويل لهذا الحالة" |
| Cannot skip states | status | "يجب اتباع تسلسل الحالات" |

### 15. State Machine — Order Processing

`
  ┌──────────┐   Vendor    ┌────────────┐   Ship    ┌──────────┐
  │  Pending  │──Confirms──→│ Processing │─────────→│ Shipped  │
  └────┬─────┘            └─────┬──────┘          └────┬─────┘
       │                        │                       │
       │ Cancel                 │ Cancel                │ Deliver
       │                        │                       │
  ┌────▼─────┐            ┌────▼──────┐          ┌────▼──────┐
  │ Cancelled│            │ Cancelled │          │ Delivered │
  └──────────┘            └───────────┘          └─────┬─────┘
                                                       │
                                                  Complete
                                                       │
                                                  ┌────▼─────┐
                                                  │ Completed│
                                                  └──────────┘

  Error Path:
  ┌──────────┐   Issue    ┌──────────┐
  │ Any State │─────────→│ In Issue │
  └──────────┘           └────┬─────┘
                              │ Resolve
                         ┌────▼─────┐
                         │ Previous │
                         │ State    │
                         └──────────┘
`

### 16. Error States

| Error | Display |
|-------|---------|
| Invalid status transition | Toast with explanation |
| API failure | Row reverts, toast error |
| Network error | "انقطع الاتصال. سيتم إعادة المحاولة." |

### 17. Empty States

| Scenario | Display |
|----------|---------|
| No orders | "لا توجد طلبات بعد. ستظهر الطلبات هنا عندما يطلب الزبائن منتجاتك." |
| No filtered results | "لا توجد طلبات تطابق الفلتر المحدد" |

### 18. Loading States

- Table: 10 skeleton rows with pulsing cells.
- Tab counts: Skeleton badges.
- Status update: Row spinner.

### 19. Success States

- Status updated: Row status badge animates to new state.
- Bulk update: Toast "{n} orders updated" + table refresh.

### 20. Permission-Based UI

| Permission | UI Impact |
|------------|-----------|
| orders:read only | No status update actions |
| orders:update | Status update actions visible |
| Read-only vendor | Table is read-only |

### 21. Security UX

- Customer PII (phone, address) hidden until order expanded.
- Order amounts shown only to vendor owner.
- Tracking numbers masked except last 4 characters.

### 22. RTL/LTR Behavior

- Table columns mirror for RTL.
- Status badges flow right-to-left.
- Date format: "12/09/2026" in RTL layout.

### 23. Internationalization

| Key | Arabic | English |
|-----|--------|---------|
| orders.title | الطلبات | Orders |
| orders.filter.all | الكل | All |
| orders.filter.pending | جديد | Pending |
| orders.filter.processing | قيد المعالجة | Processing |
| orders.filter.shipped | تم الشحن | Shipped |
| orders.filter.delivered | تم التوصيل | Delivered |
| orders.filter.issues | مشاكل | Issues |
| orders.status.pending | جديد | New |
| orders.status.processing | قيد المعالجة | Processing |
| orders.status.shipped | تم الشحن | Shipped |
| orders.status.delivered | تم التوصيل | Delivered |
| orders.status.completed | مكتمل | Completed |
| orders.status.cancelled | ملغي | Cancelled |
| orders.status.issue | مشكلة | Issue |
| orders.table.orderId | رقم الطلب | Order ID |
| orders.table.customer | الزبون | Customer |
| orders.table.date | التاريخ | Date |
| orders.table.items | المنتجات | Items |
| orders.table.total | المبلغ | Total |
| orders.table.status | الحالة | Status |

### 24. Accessibility

- Table uses ole="grid" with full keyboard navigation.
- Status badges have ole="status" with text description.
- Quick filter tabs use ole="tablist" / ole="tab".
- Keyboard: Arrow keys navigate table, Enter opens order.

### 25. Performance Requirements

| Metric | Target |
|--------|--------|
| Table render (100 rows) | < 500ms |
| Tab switch | < 200ms |
| Status update | < 300ms |
| Search (debounced) | 300ms |

### 26. Analytics Events

| Event | Properties |
|-------|------------|
| order_list_view | { filter, tab } |
| order_list_search | { query, resultCount } |
| order_status_update | { orderId, oldStatus, newStatus } |
| order_list_export | { format, count } |

### 27. Notifications

| Trigger | Type |
|---------|------|
| New order received | Real-time badge update on tab + row animation |
| Order status changed by admin | Real-time row update |
| Order cancelled | Toast notification |

### 28. Cross-Page Relationships

`
VP-OR-001 ←── VP-DB-001 (Orders widget)
VP-OR-001 ──→ VP-OR-002 (Row click)
VP-OR-001 ──→ VP-DB-001 (Dashboard link)
VP-OR-001 ←── VP-OR-002 (Back button)
VP-OR-001 ──→ VP-FN-001 (Financial summary)
`

### 29. Visual Preview

Key characteristics:
- Active tab has green underline indicator.
- Status badges use semantic colors: green=delivered, blue=shipped, amber=processing, gray=pending, red=issue.
- Row hover highlights with subtle green tint.
- Order ID formatted as monospace font for readability.

---

## VP-OR-002: Order Detail

### 1. Page Metadata

| Field | Value |
|-------|-------|
| **ID** | VP-OR-002 |
| **Page Name** | Order Detail |
| **Portal** | Vendor Panel |
| **Platform** | Desktop |
| **Module** | Orders |
| **Category** | Management |
| **Priority** | P0 |
| **Auth Required** | Yes |
| **Vendor Roles** | Vendor (all) |
| **Permissions** | orders:read (own), orders:update (own) |
| **URL** | /vendor/orders/:id |
| **Parent Page** | VP-OR-001 |
| **Requirements References** | REQ-VN-ORD-004, REQ-VN-ORD-005 |
| **Status** | Draft |

### 2. Page Purpose

Display complete order details including customer information, ordered items (vendor's products only), pricing breakdown, shipping details, order timeline, and status management actions.

### 3. Entry Points

| Source | Trigger |
|--------|---------|
| VP-OR-001 | Row click |
| Deep Link | Direct URL |
| VP-DB-001 | Recent order click |

### 4. Exit Points

| Destination | Trigger |
|-------------|---------|
| VP-OR-001 | Back button / breadcrumb |
| VP-PR-003 | Product name link |
| VP-FN-001 | Financial summary link |

### 5. Information Architecture

`
Order Detail
├── Breadcrumb: Orders > #ORD-8847
├── Header
│   ├── Order ID
│   ├── Order Date/Time
│   ├── Status Badge (large)
│   └── Status Update Actions
├── Status Timeline
│   ├── Pending → Processing → Shipped → Delivered
│   └── Current position indicator
├── Order Info Grid
│   ├── Customer Info (name, phone, email)
│   ├── Shipping Address
│   ├── Payment Method (Wallet)
│   └── Order Notes
├── Order Items Table
│   ├── Product Image
│   ├── Product Name (linked to VP-PR-003)
│   ├── SKU
│   ├── Quantity
│   ├── Unit Price
│   └── Subtotal
├── Order Summary
│   ├── Subtotal
│   ├── Shipping Fee
│   ├── Discount
│   ├── Total
│   └── Wallet Payment Confirmation
├── Shipping Details
│   ├── Carrier
│   ├── Tracking Number
│   └── Estimated Delivery
├── Order Timeline
│   ├── Status changes with timestamps
│   └── Notes added
└── Vendor Actions
    ├── Update Status
    ├── Add Note
    └── Print Order
`

### 6. Layout Structure

`
┌─────────────────────────────────────────────────────────────────────────────┐
│  ← الطلبات / طلب #ORD-8847                           [ طباعة ] [ ملاحظة ] │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  طلب #ORD-8847                    12 سبتمبر 2026, 14:30                    │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  ●────●────●────○────○    قيد المعالجة                             │   │
│  │  جديد  معالجة  شحن  توصيل  مكتمل                                   │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  ┌──────────────────────────┐ ┌──────────────────────────────────────────┐  │
│  │  معلومات الزبون          │ │  معلومات الدفع والشحن                   │  │
│  │                          │ │                                          │  │
│  │  الاسم: أحمد محمد        │ │  طريقة الدفع: محفظة إلكترونية           │  │
│  │  الهاتف: 77xxxxxxx       │ │  حالة الدفع: مدفوع ✓                    │  │
│  │  البريد: a***@gmail.com  │ │                                          │  │
│  │                          │ │  شركة الشحن: Yemen Express              │  │
│  │  عنوان الشحن:            │ │  رقم التتبع: YE123456789               │  │
│  │  صنعاء، شارع الظهران    │ │  التوصيل المتوقع: 15 سبتمبر             │  │
│  │  nhà 25، الطابق 3       │ │                                          │  │
│  └──────────────────────────┘ └──────────────────────────────────────────┘  │
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │  المنتجات (3 منتجات)                                                │   │
│  │                                                                      │   │
│  │  [img] فأرة لاسلكية     WL-001  ×1    3,500 ر.ي    3,500 ر.ي     │   │
│  │  [img] كابل USB-C       UC-012  ×2    1,200 ر.ي    2,400 ر.ي     │   │
│  │  [img] حامل هاتف        PH-004  ×1    1,500 ر.ي    1,500 ر.ي     │   │
│  │                                                                      │   │
│  │  المجموع الفرعي:              7,400 ر.ي                             │   │
│  │  رسوم الشحن:                   500 ر.ي                              │   │
│  │  الخصم:                       -200 ر.ي                              │   │
│  │  الإجمالي:                     7,700 ر.ي                             │   │
│  │  الدفع: محفظة إلكترونية ✓                                            │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  ┌──────────────────────────┐ ┌──────────────────────────────────────────┐  │
│  │  تحديث الحالة             │ │  سجل الطلب                             │  │
│  │                          │ │                                          │  │
│  │  الحالة الجديدة:          │ │  12/09 14:30 — تم استلام الطلب          │  │
│  │  ┌──────────────────┐   │ │  12/09 14:35 — تم الدفع بالمحفظة        │  │
│  │  │ تم الشحن    ▼   │   │ │  12/09 15:00 — قيد المعالجة              │  │
│  │  └──────────────────┘   │ │  ملاحظة: يرجى التغليف بإحكام             │  │
│  │                          │ │                                          │  │
│  │  رقم التتبع:             │ │                                          │  │
│  │  ┌──────────────────────┐│ │                                          │  │
│  │  │ YE123456789          ││ │                                          │  │
│  │  └──────────────────────┘│ │                                          │  │
│  │                          │ │                                          │  │
│  │  ملاحظة (اختياري):       │ │                                          │  │
│  │  ┌──────────────────────┐│ │                                          │  │
│  │  │                      ││ │                                          │  │
│  │  └──────────────────────┘│ │                                          │  │
│  │                          │ │                                          │  │
│  │  [ تحديث الحالة ]        │ │                                          │  │
│  └──────────────────────────┘ └──────────────────────────────────────────┘  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
`

### 7. Responsive Design

| Breakpoint | Behavior |
|------------|----------|
| **Desktop (≥1024px)** | 2-column layout for info panels. Full timeline. |
| **Tablet (768–1023px)** | Single column, panels stack vertically. |
| **Mobile (<768px)** | Not supported. |

### 8. Widget Inventory

| Widget | ID | Type |
|--------|----|------|
| Order Header | order-header | Header |
| Status Timeline | order-timeline | Stepper |
| Customer Info Panel | order-customer | Card |
| Payment/Shipping Panel | order-payment | Card |
| Order Items Table | order-items | Table |
| Order Summary | order-summary | Summary Card |
| Status Update Form | order-status-form | Form |
| Order Activity Log | order-activity | Timeline |

### 9. Widget Specification — Status Timeline

`	sx
interface StatusTimelineProps {
  statuses: OrderStatus[];
  currentStatus: OrderStatus;
  completedAt?: Record<OrderStatus, string>;
}
`

### 10. Data Requirements

| Endpoint | Method | Response |
|----------|--------|----------|
| GET /api/v1/vendors/orders/:id | GET | OrderDetail |
| GET /api/v1/vendors/orders/:id/timeline | GET | OrderTimeline |
| PATCH /api/v1/vendors/orders/:id/status | PATCH | Status update |
| POST /api/v1/vendors/orders/:id/notes | POST | Add vendor note |

### 11. Data Loading Strategy

- Order detail: Critical path, skeleton on load.
- Timeline: Loaded in parallel.
- Customer PII: Loaded after order detail (separate API call for privacy).

### 12. User Actions

| Action | Trigger | Permission |
|--------|---------|------------|
| Update status | Status form submit | orders:update |
| Add note | Note form submit | orders:update |
| Print order | Print button | orders:read |
| View product | Product link | products:read |
| Copy tracking | Copy button | orders:read |

### 13. Form Specification

**Status Update Form:**
`	sx
interface StatusUpdateForm {
  status: OrderStatus;
  trackingNumber?: string; // Required if status = 'shipped'
  carrier?: string;        // Required if status = 'shipped'
  note?: string;           // Optional
}
`

### 14. Validation Rules

| Rule | Field | Message |
|------|-------|---------|
| Valid transition | status | "لا يمكن التحويل لهذا الحالة من الحالة الحالية" |
| Tracking required | 	rackingNumber | "رقم التتبع مطلوب عند الشحن" |
| Carrier required | carrier | "شركة الشحن مطلوبة" |
| Note max length | 
ote | "الملاحظة يجب ألا تتجاوز 500 حرف" |

### 15. State Machine

Same as VP-OR-001 Section 15.

### 16. Error States

| Error | Display |
|-------|---------|
| Invalid transition | Toast with allowed transitions |
| API failure | Form reverts, toast error |
| Order not found | 404 page |

### 17. Empty States

| Scenario | Display |
|----------|---------|
| No timeline entries | "لا يوجد سجل بعد" |
| No vendor notes | "لا توجد ملاحظات" |

### 18. Loading States

- Full page skeleton matching layout.
- Status timeline: Skeleton stepper.

### 19. Success States

- Status updated: Timeline animates, toast "تم تحديث حالة الطلب ✓"
- Note added: Note appears in timeline with animation.

### 20. Permission-Based UI

| Permission | UI Impact |
|------------|-----------|
| orders:read only | No status update form, read-only view |
| orders:update | Full status management |
| orders:read + orders:update | Full view + edit |

### 21. Security UX

- Customer PII partially masked (phone: 77***xxx, email: ***@gmail.com).
- Full PII visible only after vendor confirms "View Customer Details" (audit logged).
- Tracking number copy action is audit-logged.

### 22. RTL/LTR Behavior

- All text RTL-aligned.
- Status timeline flows right-to-left.
- Order items table columns mirror.
- Price alignment: right-aligned in RTL.

### 23. Internationalization

| Key | Arabic | English |
|-----|--------|---------|
| orderDetail.title | طلب | Order |
| orderDetail.customer | معلومات الزبون | Customer Info |
| orderDetail.payment | معلومات الدفع | Payment Info |
| orderDetail.shipping | معلومات الشحن | Shipping Info |
| orderDetail.items | المنتجات | Items |
| orderDetail.summary | ملخص الطلب | Order Summary |
| orderDetail.timeline | سجل الطلب | Order Timeline |
| orderDetail.updateStatus | تحديث الحالة | Update Status |
| orderDetail.addNote | إضافة ملاحظة | Add Note |
| orderDetail.tracking | رقم التتبع | Tracking Number |
| orderDetail.carrier | شركة الشحن | Carrier |
| orderDetail.print | طباعة | Print |

### 24. Accessibility

- Status timeline uses ole="list" with ole="listitem" per step.
- Order items table uses ole="grid".
- Status update form has proper ria-live for error messages.
- Print action: window.print() with print-specific CSS.

### 25. Performance Requirements

| Metric | Target |
|--------|--------|
| Page load | < 1.0s |
| Status update | < 500ms |
| Note add | < 300ms |

### 26. Analytics Events

| Event | Properties |
|-------|------------|
| order_detail_view | { orderId } |
| order_status_update | { orderId, oldStatus, newStatus } |
| order_note_add | { orderId } |
| order_print | { orderId } |
| order_tracking_copy | { orderId } |

### 27. Notifications

| Trigger | Type |
|---------|------|
| Status changed by admin | Real-time update on page |
| Customer dispute | Toast + alert banner |

### 28. Cross-Page Relationships

`
VP-OR-002 ←── VP-OR-001 (Row click)
VP-OR-002 ←── VP-DB-001 (Recent order click)
VP-OR-002 ──→ VP-OR-001 (Back)
VP-OR-002 ──→ VP-PR-003 (Product link)
VP-OR-002 ──→ VP-FN-001 (Financial summary)
`

### 29. Visual Preview

Key characteristics:
- Order ID in large monospace font.
- Status timeline with green completed steps, gray pending.
- Customer info card has subtle blue border.
- Order summary card has green border for payment confirmation.

---

## VP-ST-001: Store Settings

### 1. Page Metadata

| Field | Value |
|-------|-------|
| **ID** | VP-ST-001 |
| **Page Name** | Store Settings |
| **Portal** | Vendor Panel |
| **Platform** | Desktop |
| **Module** | Store |
| **Category** | Configuration |
| **Priority** | P1 |
| **Auth Required** | Yes |
| **Vendor Roles** | Vendor (all) |
| **Permissions** | store:read, store:update (own) |
| **URL** | /vendor/store/settings |
| **Parent Page** | VP-DB-001 |
| **Requirements References** | REQ-VN-STORE-001, REQ-VN-STORE-002 |
| **Status** | Draft |

### 2. Page Purpose

Manage store configuration including store name, description, logo, banner, contact information, business hours, return policy, and store display settings.

### 3. Entry Points

| Source | Trigger |
|--------|---------|
| Sidebar Navigation | Click "Store" |
| VP-DB-001 | Store settings shortcut |
| VP-ST-002 | Back from template selection |

### 4. Exit Points

| Destination | Trigger |
|-------------|---------|
| VP-ST-002 | "Change Template" button |
| VP-DB-001 | Save & Back |
| Public Store Preview | "Preview Store" button |

### 5. Information Architecture

`
Store Settings
├── Store Info Section
│   ├── Store Name (AR/EN)
│   ├── Store Description (AR/EN)
│   ├── Store Logo (upload)
│   ├── Store Banner (upload)
│   ├── Store Category
│   └── Store Tags
├── Contact Section
│   ├── Phone Number
│   ├── WhatsApp Number
│   ├── Email
│   ├── Website (optional)
│   └── Social Media Links
├── Business Info Section
│   ├── Business Hours
│   ├── Holiday Schedule
│   └── Return/Refund Policy
├── Display Settings
│   ├── Store Template
│   ├── Color Theme
│   └── Featured Products
├── Location Section
│   ├── City/Region
│   ├── Address
│   └── Map Pin
└── Danger Zone
    ├── Deactivate Store
    └── Delete Store
`

### 6. Layout Structure

`
┌─────────────────────────────────────────────────────────────────────────────┐
│  إعدادات المتجر                     [ معاينة المتجر ]                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │  معلومات المتجر                                                      │   │
│  │                                                                      │   │
│  │  اسم المتجر (عربي) *                                                │   │
│  │  ┌──────────────────────────────────────────────────────────────┐   │   │
│  │  │ متجر إلكترونيات اليمن                                         │   │   │
│  │  └──────────────────────────────────────────────────────────────┘   │   │
│  │                                                                      │   │
│  │  اسم المتجر (إنجليزي) *                                            │   │
│  │  ┌──────────────────────────────────────────────────────────────┐   │   │
│  │  │ Yemen Electronics Store                                       │   │   │
│  │  └──────────────────────────────────────────────────────────────┘   │   │
│  │                                                                      │   │
│  │  شعار المتجر            │  بانر المتجر                              │   │
│  │  ┌─────────────────┐    │  ┌────────────────────────────────────┐   │   │
│  │  │                 │    │  │                                    │   │   │
│  │  │   [رفع شعار]    │    │  │          [رفع بانر]                │   │   │
│  │  │                 │    │  │                                    │   │   │
│  │  └─────────────────┘    │  └────────────────────────────────────┘   │   │
│  │                                                                      │   │
│  │  وصف المتجر (عربي)                                                  │   │
│  │  ┌──────────────────────────────────────────────────────────────┐   │   │
│  │  │ نقدم أفضل المنتجات الإلكترونية بأفضل الأسعار في اليمن...    │   │   │
│  │  └──────────────────────────────────────────────────────────────┘   │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │  معلومات الاتصال                                                     │   │
│  │                                                                      │   │
│  │  الهاتف *          │  واتساب            │  البريد الإلكتروني         │   │
│  │  ┌──────────────┐  │  ┌──────────────┐  │  ┌──────────────────────┐ │   │
│  │  │ 77xxxxxxx    │  │  │ 77xxxxxxx    │  │  │ store@yemen.com      │ │   │
│  │  └──────────────┘  │  └──────────────┘  │  └──────────────────────┘ │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │  ساعات العمل                                                         │   │
│  │                                                                      │   │
│  │  السبت     [09:00] إلى [21:00]  ☑                                 │   │
│  │  الأحد     [09:00] إلى [21:00]  ☑                                 │   │
│  │  الاثنين   [09:00] إلى [21:00]  ☑                                 │   │
│  │  الثلاثاء  [09:00] إلى [21:00]  ☑                                 │   │
│  │  الأربعاء  [09:00] إلى [21:00]  ☑                                 │   │
│  │  الخميس    [09:00] إلى [21:00]  ☑                                 │   │
│  │  الجمعة    [إجازة]                ☐                                 │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│                    [ حفظ التغييرات ]                                        │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
`

### 7. Responsive Design

| Breakpoint | Behavior |
|------------|----------|
| **Desktop (≥1024px)** | Full layout with 2-column sections where applicable. |
| **Tablet (768–1023px)** | Single column, sections stack. |
| **Mobile (<768px)** | Not supported. |

### 8. Widget Inventory

| Widget | ID | Type |
|--------|----|------|
| Store Name Input (AR) | store-name-ar | Text Input |
| Store Name Input (EN) | store-name-en | Text Input |
| Logo Upload | store-logo | Image Upload |
| Banner Upload | store-banner | Image Upload |
| Description Editor | store-desc | Rich Textarea |
| Contact Fields | store-contact | Form Group |
| Business Hours | store-hours | Day/Time Picker |
| Color Theme Picker | store-theme | Color Picker |
| Location Section | store-location | Address Form |

### 9. Widget Specification — Business Hours

`	sx
interface BusinessHoursProps {
  schedule: DaySchedule[];
  onChange: (schedule: DaySchedule[]) => void;
}

interface DaySchedule {
  day: 'saturday' | 'sunday' | 'monday' | 'tuesday' | 'wednesday' | 'thursday' | 'friday';
  isOpen: boolean;
  openTime?: string; // "09:00"
  closeTime?: string; // "21:00"
}
`

### 10. Data Requirements

| Endpoint | Method | Response |
|----------|--------|----------|
| GET /api/v1/vendors/store | GET | StoreSettings |
| PUT /api/v1/vendors/store | PUT | Update store settings |
| POST /api/v1/vendors/store/logo | POST | Upload logo |
| POST /api/v1/vendors/store/banner | POST | Upload banner |
| GET /api/v1/vendors/store/preview | GET | Preview URL |

### 11. Data Loading Strategy

- Store settings: Loaded on mount, form pre-filled.
- Image uploads: Chunked with progress.
- Form auto-save: Draft saved every 60s if dirty.

### 12. User Actions

| Action | Target | Permission |
|--------|--------|------------|
| Save settings | Form submit | store:update |
| Upload logo | File picker | store:update |
| Upload banner | File picker | store:update |
| Preview store | Preview button | store:read |
| Change template | Button click | VP-ST-002 |
| Deactivate store | Danger zone | store:update |

### 13. Form Specification

`	sx
interface StoreSettingsForm {
  nameAr: string;          // Required, max 100 chars
  nameEn: string;          // Required, max 100 chars
  descriptionAr: string;   // Required, max 2000 chars
  descriptionEn: string;   // Optional, max 2000 chars
  logo?: File;             // Image, max 2MB
  banner?: File;           // Image, max 5MB
  category: string;        // Required
  tags: string[];
  phone: string;           // Required, valid format
  whatsapp?: string;       // Optional, valid format
  email: string;           // Required, valid email
  website?: string;        // Optional, valid URL
  socialLinks: SocialLink[];
  businessHours: DaySchedule[];
  returnPolicy?: string;   // Max 5000 chars
  city: string;            // Required
  address: string;         // Required
  mapPin?: { lat: number; lng: number };
}
`

### 14. Validation Rules

| Rule | Field | Message |
|------|-------|---------|
| Required field | 
ameAr | "اسم المتجر مطلوب بالعربي" |
| Required field | 
ameEn | "Store name is required in English" |
| Max length | 
ameAr | "الاسم يجب ألا يتجاوز 100 حرف" |
| Required field | descriptionAr | "وصف المتجر مطلوب" |
| Image size | logo | "الشعار يجب ألا يتجاوز 2 ميجابايت" |
| Image size | anner | "البانر يجب ألا يتجاوز 5 ميجابايت" |
| Required field | phone | "رقم الهاتف مطلوب" |
| Phone format | phone | "صيغة رقم الهاتف غير صالحة" |
| Required field | email | "البريد الإلكتروني مطلوب" |
| Email format | email | "صيغة البريد الإلكتروني غير صالحة" |
| URL format | website | "صيغة الرابط غير صالحة" |
| Required field | city | "المدينة مطلوبة" |
| Required field | ddress | "العنوان مطلوب" |
| Business hours | usinessHours | "يجب أن يكون يوم واحد مفتوحاً على الأقل" |

### 15. State Machine

`
  +----------+  Edit    +----------+  Save    +----------+
  |  Pristine |--------->|  Dirty   |--------->| Saving  |
  +----------+           +----+-----+          +----+----+
                            |                       |
                       Auto-save              Success/Error
                            |                       |
                       +----v----+            +----v----+
                       | Saving  |            | Saved/  |
                       +----+----+            | Error   |
                            |                 +---------+
                       +----v----+
                       | Saved   |
                       +---------+
`

### 16. Error States

| Error | Display |
|-------|---------|
| Validation failure | Inline errors below each field |
| Image upload failure | Per-image error with retry |
| API failure | Toast with retry |
| Logo upload too large | Inline error "الشعار يجب ألا يتجاوز 2 ميجابايت" |

### 17. Empty States

| Scenario | Display |
|----------|---------|
| No logo uploaded | Placeholder logo with upload CTA |
| No banner uploaded | Placeholder banner with upload CTA |
| No social links | "إضافة روابط التواصل الاجتماعي" button |

### 18. Loading States

- Initial load: Full-page skeleton.
- Image upload: Progress bar.
- Save: Button spinner.

### 19. Success States

- Settings saved: Toast "تم حفظ إعدادات المتجر"
- Logo uploaded: Image preview replaces placeholder
- Banner uploaded: Banner preview replaces placeholder

### 20. Permission-Based UI

| Permission | UI Impact |
|------------|-----------|
| store:read only | All fields read-only, no save button |
| store:update | Full editing capabilities |

### 21. Security UX

- Phone numbers not displayed in full until verified.
- Social media links validated for URL format.
- Business hours changes are audit-logged.

### 22. RTL/LTR Behavior

- Form labels right-aligned in RTL.
- Business hours days listed right-to-left (Saturday first).
- Logo/banner uploads centered regardless of direction.

### 23. Internationalization

| Key | Arabic | English |
|-----|--------|---------|
| storeSettings.title | إعدادات المتجر | Store Settings |
| storeSettings.preview | معاينة المتجر | Preview Store |
| storeSettings.storeInfo | معلومات المتجر | Store Info |
| storeSettings.storeNameAr | اسم المتجر (عربي) | Store Name (Arabic) |
| storeSettings.storeNameEn | اسم المتجر (إنجليزي) | Store Name (English) |
| storeSettings.logo | شعار المتجر | Store Logo |
| storeSettings.banner | بانر المتجر | Store Banner |
| storeSettings.description | وصف المتجر | Store Description |
| storeSettings.contact | معلومات الاتصال | Contact Info |
| storeSettings.businessHours | ساعات العمل | Business Hours |
| storeSettings.save | حفظ التغييرات | Save Changes |

### 24. Accessibility

- All form fields have associated <label> elements.
- Image uploads have ria-label with file type/size instructions.
- Business hours checkboxes have ria-label for each day.
- Save button has ria-busy during submission.

### 25. Performance Requirements

| Metric | Target |
|--------|--------|
| Page load | < 1.0s |
| Logo upload | < 2s (2MB) |
| Banner upload | < 3s (5MB) |
| Form save | < 1s |

### 26. Analytics Events

| Event | Properties |
|-------|------------|
| store_settings_view | { section } |
| store_settings_save | { fieldsChanged } |
| store_logo_upload | { fileSize } |
| store_banner_upload | { fileSize } |
| store_preview_click | { } |
| store_template_change | { templateId } |

### 27. Notifications

| Trigger | Type |
|---------|------|
| Settings saved | Toast |
| Logo upload complete | Inline success |
| Banner upload complete | Inline success |

### 28. Cross-Page Relationships

`
VP-ST-001 ←── VP-DB-001 (Store settings shortcut)
VP-ST-001 ←── VP-ST-002 (Back from template)
VP-ST-001 ──→ VP-ST-002 (Change template)
VP-ST-001 ──→ VP-DB-001 (Save & Back)
VP-ST-001 ──→ Public Store (Preview)
`

### 29. Visual Preview

Key characteristics:
- Clean form layout with generous spacing.
- Logo preview shows current logo with hover overlay for replacement.
- Banner preview shows full-width image with overlay.
- Business hours use consistent time picker styling.
- Save button is sticky at bottom of form.

---

## VP-ST-002: Store Template Selection

### 1. Page Metadata

| Field | Value |
|-------|-------|
| **ID** | VP-ST-002 |
| **Page Name** | Store Template Selection |
| **Portal** | Vendor Panel |
| **Platform** | Desktop |
| **Module** | Store |
| **Category** | Configuration |
| **Priority** | P2 |
| **Auth Required** | Yes |
| **Vendor Roles** | Vendor (all) |
| **Permissions** | store:read, store:update (own) |
| **URL** | /vendor/store/template |
| **Parent Page** | VP-ST-001 |
| **Requirements References** | REQ-VN-STORE-003 |
| **Status** | Draft |

### 2. Page Purpose

Allow vendors to browse and select from available store templates. Each template provides a different layout and visual style for their public storefront.

### 3. Entry Points

| Source | Trigger |
|--------|---------|
| VP-ST-001 | "Change Template" button |

### 4. Exit Points

| Destination | Trigger |
|-------------|---------|
| VP-ST-001 | Back button / Save selection |
| Public Store Preview | Preview template |

### 5. Information Architecture

`
Template Selection
├── Header
│   ├── Title: "اختر قالب المتجر"
│   └── Current Template badge
├── Template Grid
│   ├── Template Card
│   │   ├── Preview Image
│   │   ├── Template Name
│   │   ├── Description
│   │   ├── Features list
│   │   └── [Select] / [Current] button
│   └── ... more templates
├── Preview Modal (on template click)
│   ├── Full preview of template
│   └── [Apply Template] button
└── Save/Cancel Actions
`

### 6. Layout Structure

`
┌─────────────────────────────────────────────────────────────────────────────┐
│  ← إعدادات المتجر / اختر قالب المتجر                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  القالب الحالي: الافتراضي          [تغيير]                               │
│                                                                             │
│  ┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐           │
│  │  [معاينة]        │ │  [معاينة]        │ │  [معاينة]        │           │
│  │                  │ │                  │ │                  │           │
│  │  كلاسيكي         │ │  عصري           │ │  بسيط            │           │
│  │  تصميم تقليدي   │ │  تصميم نظيف     │ │  يركز على        │           │
│  │  مع شريط جانبي   │ │  مع صور كبيرة   │ │  المنتجات فقط    │           │
│  │                  │ │                  │ │                  │           │
│  │  - تنقل جانبي   │ │  - تخطيط شبكة   │ │  - شبكة فقط     │           │
│  │  - بانر رئيسي   │ │  - بانر رئيسي   │ │  - بدون شريط    │           │
│  │  - تصنيفات      │ │  - تصنيفات      │ │  جانبي           │           │
│  │                  │ │                  │ │                  │           │
│  │  [اختيار]        │ │  [حالي]          │ │  [اختيار]        │           │
│  └──────────────────┘ └──────────────────┘ └──────────────────┘           │
│                                                                             │
│  ┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐           │
│  │  [معاينة]        │ │  [معاينة]        │ │  [معاينة]        │           │
│  │                  │ │                  │ │                  │           │
│  │  شبكة            │ │  مجلة           │ │  فاخر            │           │
│  │  تخطيط يركز     │ │  محتوى غني      │ │  إحساس راقي     │           │
│  │  على المنتجات    │ │  مع قصص         │ │  مع سمة داكنة   │           │
│  │                  │ │                  │ │                  │           │
│  │  [اختيار]        │ │  [اختيار]        │ │  [اختيار]        │           │
│  └──────────────────┘ └──────────────────┘ └──────────────────┘           │
│                                                                             │
│                      [ حفظ الاختيار ]                                       │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
`

### 7. Responsive Design

| Breakpoint | Behavior |
|------------|----------|
| **Desktop (≥1024px)** | 3-column grid of template cards. |
| **Tablet (768–1023px)** | 2-column grid. |
| **Mobile (<768px)** | Not supported. |

### 8. Widget Inventory

| Widget | ID | Type |
|--------|----|------|
| Template Card | 	emplate-card | Card |
| Template Preview Modal | 	emplate-preview | Modal |
| Current Template Badge | 	emplate-current | Badge |
| Select Button | 	emplate-select | Button |

### 9. Widget Specification — Template Card

`	sx
interface TemplateCardProps {
  template: {
    id: string;
    name: string;
    description: string;
    previewImage: string;
    features: string[];
    isPremium: boolean;
  };
  isCurrent: boolean;
  onSelect: (id: string) => void;
  onPreview: (id: string) => void;
}
`

### 10. Data Requirements

| Endpoint | Method | Response |
|----------|--------|----------|
| GET /api/v1/vendors/templates | GET | Template[] |
| GET /api/v1/vendors/templates/:id/preview | GET | Preview HTML/URL |
| PUT /api/v1/vendors/store/template | PUT | Apply template |

### 11. Data Loading Strategy

- Templates list: Loaded on mount.
- Preview: Lazy loaded on hover or click.
- Apply: Optimistic with rollback.

### 12. User Actions

| Action | Target | Permission |
|--------|--------|------------|
| Select template | Select button | store:update |
| Preview template | Preview button | store:read |
| Apply template | Apply button in modal | store:update |
| Save selection | Save button | store:update |

### 13. Form Specification

No complex forms. Single action: template selection.

### 14. Validation Rules

| Rule | Message |
|------|---------|
| Template already applied | "هذا القالب مفعل بالفعل" |
| Premium template without subscription | "القالب المميز يتطلب اشتراك" |

### 15. State Machine

`
  +----------+  Select  +----------+  Apply   +----------+
  |  Browse  |--------->| Selected |--------->| Applied |
  +----------+          +----------+          +----------+
       |                     |
       | Preview             | Cancel
       |                     |
  +----v-----+          +---v--------+
  | Preview  |          | Back to    |
  | Modal    |          | Browse     |
  +----------+          +------------+
`

### 16. Error States

| Error | Display |
|-------|---------|
| Templates fail to load | Error card with retry |
| Apply fails | Toast with error, revert to previous template |
| Premium template locked | Modal: "قم بالترقية للوصول لهذا القالب" |

### 17. Empty States

| Scenario | Display |
|----------|---------|
| No templates available | "لا توجد قوالب متاحة بعد" |

### 18. Loading States

- Templates grid: Skeleton cards.
- Preview modal: Loading spinner.

### 19. Success States

- Template applied: Toast "تم تحديث القالب" + redirect to VP-ST-001.

### 20. Permission-Based UI

| Permission | UI Impact |
|------------|-----------|
| store:read only | Select buttons disabled |
| store:update | Full interaction |

### 21. Security UX

- Template previews are sandboxed iframes.
- No external scripts allowed in template previews.

### 22. RTL/LTR Behavior

- Template cards flow right-to-left.
- Preview images are direction-agnostic.

### 23. Internationalization

| Key | Arabic | English |
|-----|--------|---------|
| 	emplate.title | اختر قالب المتجر | Choose Store Template |
| 	emplate.current | الحالي | Current |
| 	emplate.select | اختيار | Select |
| 	emplate.preview | معاينة | Preview |
| 	emplate.apply | تطبيق | Apply |
| 	emplate.features | المميزات | Features |
| 	emplate.premium | مميز | Premium |

### 24. Accessibility

- Template cards have ole="article" with ria-label.
- Preview modal has ole="dialog" with focus trap.
- Keyboard: Arrow keys navigate grid, Enter selects.

### 25. Performance Requirements

| Metric | Target |
|--------|--------|
| Templates load | < 1.0s |
| Preview load | < 2.0s |
| Apply template | < 1.0s |

### 26. Analytics Events

| Event | Properties |
|-------|------------|
| 	emplate_browse | { templateCount } |
| 	emplate_preview | { templateId } |
| 	emplate_select | { templateId } |
| 	emplate_apply | { templateId, previousTemplateId } |

### 27. Notifications

| Trigger | Type |
|---------|------|
| Template applied | Toast |

### 28. Cross-Page Relationships

`
VP-ST-002 ←── VP-ST-001 (Change template)
VP-ST-002 ──→ VP-ST-001 (Back / Save)
`

### 29. Visual Preview

Key characteristics:
- Template preview images are high-quality screenshots.
- Current template has green border and "Current" badge.
- Premium templates have gold "Premium" badge.
- Select button changes to checkmark when selected.

---

## VP-FN-001: Financial Dashboard

### 1. Page Metadata

| Field | Value |
|-------|-------|
| **ID** | VP-FN-001 |
| **Page Name** | Financial Dashboard |
| **Portal** | Vendor Panel |
| **Platform** | Desktop |
| **Module** | Finance |
| **Category** | Overview |
| **Priority** | P0 |
| **Auth Required** | Yes |
| **Vendor Roles** | Vendor (all) |
| **Permissions** | inance:read (own) |
| **URL** | /vendor/finance |
| **Parent Page** | VP-DB-001 |
| **Requirements References** | REQ-VN-FIN-001, REQ-VN-FIN-002 |
| **Status** | Draft |

### 2. Page Purpose

Provide vendors with a comprehensive financial overview including earnings, commissions, pending payouts, transaction history, and financial analytics. Wallet-only payment model means all transactions flow through the vendor wallet.

### 3. Entry Points

| Source | Trigger |
|--------|---------|
| Sidebar Navigation | Click "Finance" |
| VP-DB-001 | Wallet balance KPI click |
| VP-OR-001 | Financial summary link |

### 4. Exit Points

| Destination | Trigger |
|-------------|---------|
| VP-FN-002 | "View Transactions" button |
| VP-FN-003 | "Request Payout" button |
| VP-DB-001 | Back to dashboard |

### 5. Information Architecture

`
Financial Dashboard
├── Header
│   ├── Title: "المالية"
│   └── Period Selector (This Month, Last 30 Days, This Year, Custom)
├── KPI Summary Strip
│   ├── Total Earnings (YER)
│   ├── Pending Balance (YER)
│   ├── Total Commissions Paid (YER)
│   ├── Net Profit (YER)
│   └── Wallet Balance (YER)
├── Earnings Chart
│   ├── Daily/Weekly/Monthly toggle
│   └── Line chart of earnings over time
├── Commission Breakdown
│   ├── Platform commission (percentage)
│   ├── Shipping commission
│   └── Tax (if applicable)
├── Recent Transactions Widget
│   ├── Latest 5 transactions
│   ├── Type, amount, date
│   └── View all link
├── Payout Summary
│   ├── Last payout date/amount
│   ├── Next payout date
│   ├── Available for payout
│   └── Request payout button
└── Export Section
    ├── Export monthly report
    └── Export annual report
`

### 6. Layout Structure

`
┌─────────────────────────────────────────────────────────────────────────────┐
│  المالية                             [هذا الشهر ▼] [تصدير التقرير]        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────┐ ┌──────────┐ │
│  │ إجمالي      │ │ رصيد       │ │ عمولات      │ │ صافي    │ │ رصيد     │ │
│  │ الأرباح     │ │ معلق       │ │ مدفوعة      │ │ الربح   │ │ المحفظة  │ │
│  │ 450,000 ر.ي │ │ 125,000 ر.ي│ │ 45,000 ر.ي  │ │ 380,000 │ │ 280,000  │ │
│  │ +15%        │ │            │ │            │ │ ر.ي     │ │ ر.ي      │ │
│  └─────────────┘ └─────────────┘ └─────────────┘ └─────────┘ └──────────┘ │
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │  الأرباح عبر الوقت                                                   │   │
│  │                                                                      │   │
│  │  (رسم بياني خطي يظهر الأرباح حسب اليوم/الأسبوع/الشهر)             │   │
│  │                                                                      │   │
│  │  [يومي] [أسبوعي] [شهري]                                             │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  ┌──────────────────────────────┐ ┌──────────────────────────────────────┐  │
│  │  تفصيل العمولات              │ │  أحدث المعاملات                     │  │
│  │                              │ │                                      │  │
│  │  المنصة: 10% = 45,000 ر.ي   │ │  طلب #8847  +12,500 ر.ي  12/09     │  │
│  │  الشحن: 5% = 22,500 ر.ي     │ │  طلب #8843  +3,500 ر.ي   12/09     │  │
│  │  الضريبة: 0%                 │ │  طلب #8841  +28,900 ر.ي  11/09     │  │
│  │                              │ │  طلب #8839  +8,200 ر.ي   11/09     │  │
│  │  صافي الأرباح: 382,500 ر.ي  │ │  طلب #8837  +1,200 ر.ي   11/09     │  │
│  │                              │ │                                      │  │
│  │                              │ │  [ عرض كل المعاملات ]               │  │
│  └──────────────────────────────┘ └──────────────────────────────────────┘  │
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │  ملخص السحب                                                          │   │
│  │                                                                      │   │
│  │  آخر سحب: 1 سبتمبر 2026         السحب القادم: 1 أكتوبر 2026       │   │
│  │  المبلغ: 320,000 ر.ي              المتاح: 125,000 ر.ي              │   │
│  │                                                                      │   │
│  │  [ طلب سحب ]                                                         │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
`

### 7. Responsive Design

| Breakpoint | Behavior |
|------------|----------|
| **Desktop (≥1024px)** | Full layout with 5 KPI cards, 2-column content. |
| **Tablet (768–1023px)** | KPI strip wraps to 3+2. Content single-column. |
| **Mobile (<768px)** | Not supported. |

### 8. Widget Inventory

| Widget | ID | Type |
|--------|----|------|
| KPI: Total Earnings | n-kpi-earnings | Metric Card |
| KPI: Pending Balance | n-kpi-pending | Metric Card |
| KPI: Commissions | n-kpi-commissions | Metric Card |
| KPI: Net Profit | n-kpi-profit | Metric Card |
| KPI: Wallet Balance | n-kpi-wallet | Metric Card |
| Earnings Chart | n-earnings-chart | Line Chart |
| Commission Breakdown | n-commission | Breakdown Card |
| Recent Transactions | n-transactions | List Widget |
| Payout Summary | n-payout-summary | Summary Card |
| Period Selector | n-period | Dropdown |
| Export Buttons | n-export | Button Group |

### 9. Widget Specification — Earnings Chart

`	sx
interface EarningsChartProps {
  data: EarningsDataPoint[];
  period: 'daily' | 'weekly' | 'monthly';
  currency: string;
  onPeriodChange: (period: string) => void;
}

interface EarningsDataPoint {
  date: string;
  earnings: number;
  orders: number;
}
`

### 10. Data Requirements

| Endpoint | Method | Response |
|----------|--------|----------|
| GET /api/v1/vendors/finance/summary | GET | FinanceSummary |
| GET /api/v1/vendors/finance/earnings | GET | EarningsDataPoint[] |
| GET /api/v1/vendors/finance/commissions | GET | CommissionBreakdown |
| GET /api/v1/vendors/finance/transactions | GET | Transaction[] |
| GET /api/v1/vendors/finance/payout-summary | GET | PayoutSummary |
| GET /api/v1/vendors/finance/export | GET | PDF/CSV report |

### 11. Data Loading Strategy

1. KPI summary loads first.
2. Earnings chart loads after KPIs.
3. Transaction list and commission breakdown load in parallel.
4. Payout summary loads last.

### 12. User Actions

| Action | Target | Permission |
|--------|--------|------------|
| View transactions | Button click | VP-FN-002 |
| Request payout | Button click | VP-FN-003 |
| Change period | Period selector | — |
| Export report | Export button | inance:read |

### 13. Form Specification

No forms on this page. Period selector is a dropdown.

### 14. Validation Rules

N/A — Read-only dashboard.

### 15. State Machine

Same pattern as VP-DB-001.

### 16. Error States

| Error | Display |
|-------|---------|
| API failure | Error card with retry |
| Export failure | Toast with retry |

### 17. Empty States

| Scenario | Display |
|----------|---------|
| No transactions | "لا توجد معاملات بعد" |
| No earnings data | "ستظهر بيانات الأرباح بعد أول بيع" |

### 18. Loading States

- KPI cards: Skeleton.
- Chart: Skeleton axes.
- Transactions: 5 skeleton rows.

### 19. Success States

- Export: Toast "جاري التصدير" + download starts.

### 20. Permission-Based UI

| Permission | UI Impact |
|------------|-----------|
| inance:read only | Full read-only view |

### 21. Security UX

- Financial data is only visible to the vendor owner.
- Exported reports include watermark with vendor ID and timestamp.

### 22. RTL/LTR Behavior

- Chart Y-axis labels on right side in RTL.
- Transaction list items aligned right-to-left.

### 23. Internationalization

| Key | Arabic | English |
|-----|--------|---------|
| inance.title | المالية | Finance |
| inance.totalEarnings | إجمالي الأرباح | Total Earnings |
| inance.pendingBalance | رصيد معلق | Pending Balance |
| inance.commissions | العمولات | Commissions |
| inance.netProfit | صافي الربح | Net Profit |
| inance.walletBalance | رصيد المحفظة | Wallet Balance |
| inance.earningsChart | الأرباح عبر الوقت | Earnings Over Time |
| inance.commissionBreakdown | تفصيل العمولات | Commission Breakdown |
| inance.recentTransactions | أحدث المعاملات | Recent Transactions |
| inance.payoutSummary | ملخص السحب | Payout Summary |
| inance.requestPayout | طلب سحب | Request Payout |
| inance.export | تصدير التقرير | Export Report |
| inance.period.thisMonth | هذا الشهر | This Month |
| inance.period.last30 | آخر 30 يوم | Last 30 Days |
| inance.period.thisYear | هذا العام | This Year |
| inance.period.custom | مخصص | Custom |

### 24. Accessibility

- KPI cards have ria-label with full text.
- Chart has ria-label and hidden data table.
- Export buttons have ria-label with format description.

### 25. Performance Requirements

| Metric | Target |
|--------|--------|
| Page load | < 1.5s |
| Chart render | < 500ms |
| Export generation | < 5s |

### 26. Analytics Events

| Event | Properties |
|-------|------------|
| inance_dashboard_view | { period } |
| inance_period_change | { from, to } |
| inance_export_click | { format, period } |
| inance_payout_request | { amount } |

### 27. Notifications

| Trigger | Type |
|---------|------|
| Payout processed | Toast |
| Payout rejected | Toast + badge |

### 28. Cross-Page Relationships

`
VP-FN-001 ←── VP-DB-001 (Wallet click)
VP-FN-001 ←── VP-OR-001 (Financial summary)
VP-FN-001 ──→ VP-FN-002 (View transactions)
VP-FN-001 ──→ VP-FN-003 (Request payout)
VP-FN-001 ──→ VP-DB-001 (Back)
`

### 29. Visual Preview

Key characteristics:
- Financial KPIs use amber accent color.
- Earnings chart has green line with gradient fill.
- Commission breakdown uses horizontal bar chart.
- Payout summary card has green border for available amount.

---

## VP-FN-002: Transaction History

### 1. Page Metadata

| Field | Value |
|-------|-------|
| **ID** | VP-FN-002 |
| **Page Name** | Transaction History |
| **Portal** | Vendor Panel |
| **Platform** | Desktop |
| **Module** | Finance |
| **Category** | Reporting |
| **Priority** | P1 |
| **Auth Required** | Yes |
| **Vendor Roles** | Vendor (all) |
| **Permissions** | inance:read (own) |
| **URL** | /vendor/finance/transactions |
| **Parent Page** | VP-FN-001 |
| **Requirements References** | REQ-VN-FIN-003 |
| **Status** | Draft |

### 2. Page Purpose

Display a detailed, filterable list of all financial transactions including order payments, commissions, refunds, and payout transfers.

### 3. Entry Points

| Source | Trigger |
|--------|---------|
| VP-FN-001 | "View Transactions" button |
| Sidebar Navigation | Finance > Transactions |

### 4. Exit Points

| Destination | Trigger |
|-------------|---------|
| VP-FN-001 | Back button |
| VP-OR-002 | Order reference link |

### 5. Information Architecture

`
Transaction History
├── Header
│   ├── Title: "سجل المعاملات"
│   └── Export Button
├── Toolbar
│   ├── Search (order ID, description)
│   ├── Transaction Type Filter (Income, Commission, Refund, Payout)
│   ├── Date Range Picker
│   ├── Amount Range
│   └── Sort (Date, Amount)
├── Summary Bar
│   ├── Total Income
│   ├── Total Commissions
│   ├── Total Refunds
│   └── Net Amount
├── Transaction Table
│   ├── Transaction ID
│   ├── Date/Time
│   ├── Type (badge)
│   ├── Description
│   ├── Reference (order ID)
│   ├── Amount (YER)
│   ├── Balance After
│   └── Actions
└── Pagination
`

### 6. Layout Structure

`
┌─────────────────────────────────────────────────────────────────────────────┐
│  سجل المعاملات                                  [تصدير CSV] [تصدير PDF]  │
├─────────────────────────────────────────────────────────────────────────────┤
│  🔍 بحث...    [النوع ▼] [التاريخ ▼] [المبلغ ▼]                           │
├─────────────────────────────────────────────────────────────────────────────┤
│  الدخل: 450,000  │  العمولات: -45,000  │  المرتجعات: -5,000  │  صافي: 400,000 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  TXN-001  │ 12/09 14:30 │ 💰 دخل      │ طلب #8847    │ +12,500 │ 280,000 │  ⋯ │
│  TXN-002  │ 12/09 14:30 │ 💸 عمولة    │ طلب #8847    │ -1,250  │ 278,750 │  ⋯ │
│  TXN-003  │ 12/09 12:15 │ 💰 دخل      │ طلب #8843    │ +3,500  │ 280,000 │  ⋯ │
│  TXN-004  │ 12/09 12:15 │ 💸 عمولة    │ طلب #8843    │ -350    │ 279,650 │  ⋯ │
│  TXN-005  │ 11/09 18:00 │ 💰 دخل      │ طلب #8841    │ +28,900 │ 279,300 │  ⋯ │
│  TXN-006  │ 11/09 18:00 │ 💸 عمولة    │ طلب #8841    │ -2,890  │ 250,400 │  ⋯ │
│  TXN-007  │ 11/09 10:30 │ 🔄 مرتجع   │ طلب #8835    │ -1,200  │ 253,290 │  ⋯ │
│  TXN-008  │ 10/09 09:00 │ 💰 دخل      │ طلب #8833    │ +6,700  │ 254,490 │  ⋯ │
│  TXN-009  │ 10/09 09:00 │ 💸 عمولة    │ طلب #8833    │ -670    │ 247,790 │  ⋯ │
│  TXN-010  │ 01/09 00:00 │ 🏦 سحب      │ تحويل بنكي  │ -320,000│ 248,460 │  ⋯ │
│                                                                             │
│  ◀ 1 2 3 ... 15 ▶                              عرض 1-10 من 148            │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
`

### 7. Responsive Design

| Breakpoint | Behavior |
|------------|----------|
| **Desktop (≥1024px)** | Full table with all columns. |
| **Tablet (768–1023px)** | Table with fewer columns (hide Balance After). |
| **Mobile (<768px)** | Not supported. |

### 8. Widget Inventory

| Widget | ID | Type |
|--------|----|------|
| Summary Bar | 	xn-summary | Metric Strip |
| Transaction Table | 	xn-table | Data Table |
| Type Badge | 	xn-type-badge | Badge |
| Date Range Picker | 	xn-date-range | Date Picker |
| Pagination | 	xn-pagination | Pagination |

### 9. Widget Specification — Transaction Type Badge

`	sx
interface TransactionTypeBadgeProps {
  type: 'income' | 'commission' | 'refund' | 'payout';
  amount: number;
}
`

Color coding:
- Income: Green
- Commission: Amber
- Refund: Red
- Payout: Blue

### 10. Data Requirements

| Endpoint | Method | Response |
|----------|--------|----------|
| GET /api/v1/vendors/finance/transactions | GET | Paginated<Transaction> |
| GET /api/v1/vendors/finance/transactions/summary | GET | TransactionSummary |

### 11. Data Loading Strategy

1. Summary bar loads first.
2. Table loads with skeleton rows.
3. Filters apply instantly with loading overlay.

### 12. User Actions

| Action | Target | Permission |
|--------|--------|------------|
| View order | Reference link | VP-OR-002 |
| Export CSV | Export button | inance:read |
| Export PDF | Export button | inance:read |

### 13. Form Specification

No forms. Read-only with filters.

### 14. Validation Rules

N/A — Read-only page.

### 15. State Machine

Same pattern as VP-DB-001.

### 16. Error States

| Error | Display |
|-------|---------|
| API failure | Error banner with retry |
| Export failure | Toast with retry |

### 17. Empty States

| Scenario | Display |
|----------|---------|
| No transactions | "لا توجد معاملات بعد" |
| No filtered results | "لا توجد معاملات تطابق الفلتر" |

### 18. Loading States

- Table: 10 skeleton rows.
- Summary: 4 skeleton metric cards.

### 19. Success States

- Export: Download starts + toast.

### 20. Permission-Based UI

| Permission | UI Impact |
|------------|-----------|
| inance:read only | Full read-only view |

### 21. Security UX

- Transaction details only visible to vendor owner.
- Exported files include vendor watermark.

### 22. RTL/LTR Behavior

- Table columns mirror for RTL.
- Amount alignment: right-aligned.
- Positive amounts green, negative amounts red.

### 23. Internationalization

| Key | Arabic | English |
|-----|--------|---------|
| 	ransactions.title | سجل المعاملات | Transaction History |
| 	ransactions.type.income | دخل | Income |
| 	ransactions.type.commission | عمولة | Commission |
| 	ransactions.type.refund | مرتجع | Refund |
| 	ransactions.type.payout | سحب | Payout |
| 	ransactions.summary.income | إجمالي الدخل | Total Income |
| 	ransactions.summary.commission | إجمالي العمولات | Total Commissions |
| 	ransactions.summary.refund | إجمالي المرتجعات | Total Refunds |
| 	ransactions.summary.net | صافي المبلغ | Net Amount |
| 	ransactions.exportCSV | تصدير CSV | Export CSV |
| 	ransactions.exportPDF | تصدير PDF | Export PDF |

### 24. Accessibility

- Table uses ole="grid" with keyboard navigation.
- Type badges have ole="status" with text description.
- Summary bar has ria-label for each metric.

### 25. Performance Requirements

| Metric | Target |
|--------|--------|
| Table render (100 rows) | < 500ms |
| Export generation | < 5s |

### 26. Analytics Events

| Event | Properties |
|-------|------------|
| 	ransaction_history_view | { filters } |
| 	ransaction_export | { format, count } |

### 27. Notifications

| Trigger | Type |
|---------|------|
| New transaction | Real-time row animation |

### 28. Cross-Page Relationships

`
VP-FN-002 ←── VP-FN-001 (View transactions)
VP-FN-002 ──→ VP-OR-002 (Order reference)
VP-FN-002 ──→ VP-FN-001 (Back)
`

### 29. Visual Preview

Key characteristics:
- Transaction type badges use distinct colors.
- Positive amounts in green, negative in red.
- Balance column shows running total.
- Order references are clickable links.

---

## VP-FN-003: Payout Management

### 1. Page Metadata

| Field | Value |
|-------|-------|
| **ID** | VP-FN-003 |
| **Page Name** | Payout Management |
| **Portal** | Vendor Panel |
| **Platform** | Desktop |
| **Module** | Finance |
| **Category** | Operations |
| **Priority** | P0 |
| **Auth Required** | Yes |
| **Vendor Roles** | Vendor (all) |
| **Permissions** | inance:read (own) |
| **URL** | /vendor/finance/payouts |
| **Parent Page** | VP-FN-001 |
| **Requirements References** | REQ-VN-FIN-004, REQ-VN-FIN-005 |
| **Status** | Draft |

### 2. Page Purpose

Manage vendor payout requests, view payout history, configure payout methods, and track pending/completed payouts. Since YemenMart uses wallet-only payments, payouts are the mechanism for vendors to withdraw earnings.

### 3. Entry Points

| Source | Trigger |
|--------|---------|
| VP-FN-001 | "Request Payout" button |
| Sidebar Navigation | Finance > Payouts |

### 4. Exit Points

| Destination | Trigger |
|-------------|---------|
| VP-FN-001 | Back button |
| VP-FN-002 | View payout details in transactions |

### 5. Information Architecture

`
Payout Management
├── Header
│   ├── Title: "إدارة السحوبات"
│   └── Request Payout CTA
├── Payout Summary Cards
│   ├── Available Balance (YER)
│   ├── Pending Payouts (count + amount)
│   ├── Minimum Payout Threshold
│   └── Last Payout Date
├── Request Payout Form (slide-over)
│   ├── Amount Input
│   ├── Payout Method (Bank Transfer, Mobile Wallet)
│   ├── Account Details (conditional)
│   └── Submit Button
├── Payout History Table
│   ├── Payout ID
│   ├── Date Requested
│   ├── Amount (YER)
│   ├── Method
│   ├── Status (Pending, Processing, Completed, Failed)
│   ├── Date Completed
│   └── Reference Number
└── Payout Settings
    ├── Default Payout Method
    ├── Automatic Payouts toggle
    └── Minimum Payout Amount
`

### 6. Layout Structure

`
┌─────────────────────────────────────────────────────────────────────────────┐
│  إدارة السحوبات                                    [طلب سحب]              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────────────┐  │
│  │ رصيد متاح   │ │ سحوبات     │ │ الحد الأدنى │ │ آخر سحب            │  │
│  │ 125,000 ر.ي │ │ معلقة (2)  │ │ 10,000 ر.ي │ │ 1 سبتمبر 2026      │  │
│  │             │ │ 25,000 ر.ي  │ │             │ │ 320,000 ر.ي         │  │
│  └─────────────┘ └─────────────┘ └─────────────┘ └─────────────────────┘  │
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │  طلب سحب جديد                                                       │   │
│  │                                                                      │   │
│  │  المبلغ (ر.ي) *                                                     │   │
│  │  ┌──────────────────────────────────────────────────────────────┐   │   │
│  │  │ 125,000                                                     │   │   │
│  │  └──────────────────────────────────────────────────────────────┘   │   │
│  │  الحد الأدنى: 10,000 ر.ي   المتاح: 125,000 ر.ي                    │   │
│  │                                                                      │   │
│  │  طريقة السحب *                                                      │   │
│  │  ○ تحويل بنكي  ○ محفظة إلكترونية                                  │   │
│  │                                                                      │   │
│  │  تفاصيل الحساب (تحويل بنكي)                                        │   │
│  │  ┌──────────────────────────────────────────────────────────────┐   │   │
│  │  │ اسم البنك:                                                  │   │   │
│  │  │ رقم الحساب:                                                  │   │   │
│  │  │ اسم الحساب:                                                  │   │   │
│  │  └──────────────────────────────────────────────────────────────┘   │   │
│  │                                                                      │   │
│  │  [ إلغاء ]                                        [ تقديم الطلب ]   │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │  سجل السحوبات                                                        │   │
│  │                                                                      │   │
│  │  PO-001  │ 01/09  │ 320,000 ر.ي │ تحويل بنكي │ ✅ مكتمل  │ 01/09   │   │
│  │  PO-002  │ 01/08  │ 280,000 ر.ي │ تحويل بنكي │ ✅ مكتمل  │ 01/08   │   │
│  │  PO-003  │ 01/07  │ 350,000 ر.ي │ محفظة      │ ✅ مكتمل  │ 01/07   │   │
│  │  PO-004  │ 15/09  │  15,000 ر.ي │ تحويل بنكي │ ⏳ معلق  │         │   │
│  │  PO-005  │ 12/09  │  10,000 ر.ي │ محفظة      │ ⏳ معلق  │         │   │
│  │                                                                      │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
`

### 7. Responsive Design

| Breakpoint | Behavior |
|------------|----------|
| **Desktop (≥1024px)** | Full layout with summary cards, form, and history table. |
| **Tablet (768–1023px)** | Summary cards wrap to 2x2. Form becomes full-width. |
| **Mobile (<768px)** | Not supported. |

### 8. Widget Inventory

| Widget | ID | Type |
|--------|----|------|
| Available Balance Card | payout-available | Metric Card |
| Pending Payouts Card | payout-pending | Metric Card |
| Minimum Payout Card | payout-minimum | Metric Card |
| Last Payout Card | payout-last | Metric Card |
| Request Payout Form | payout-request-form | Form |
| Payout Method Selector | payout-method | Radio Group |
| Account Details Form | payout-account | Conditional Form |
| Payout History Table | payout-history | Data Table |
| Status Badge | payout-status-badge | Badge |

### 9. Widget Specification — Payout Request Form

`	sx
interface PayoutRequestForm {
  amount: number;          // Required, >= minimum, <= available
  method: 'bank_transfer' | 'mobile_wallet';
  bankDetails?: {
    bankName: string;
    accountNumber: string;
    accountHolder: string;
    iban?: string;
  };
  mobileWalletDetails?: {
    provider: string;
    phoneNumber: string;
  };
}
`

### 10. Data Requirements

| Endpoint | Method | Response |
|----------|--------|----------|
| GET /api/v1/vendors/payouts/summary | GET | PayoutSummary |
| POST /api/v1/vendors/payouts/request | POST | PayoutRequest |
| GET /api/v1/vendors/payouts | GET | Payout[] |
| GET /api/v1/vendors/payouts/:id | GET | PayoutDetail |
| GET /api/v1/vendors/payouts/settings | GET | PayoutSettings |
| PUT /api/v1/vendors/payouts/settings | PUT | Update settings |

### 11. Data Loading Strategy

1. Summary cards load first.
2. Payout history table loads after.
3. Form is always available.

### 12. User Actions

| Action | Target | Permission |
|--------|--------|------------|
| Request payout | Form submit | inance:read |
| View payout details | Row click | inance:read |
| Change payout settings | Settings form | inance:read |

### 13. Form Specification

`	sx
interface PayoutRequestForm {
  amount: number;
  method: 'bank_transfer' | 'mobile_wallet';
  bankDetails?: {
    bankName: string;    // Required if bank_transfer
    accountNumber: string; // Required if bank_transfer
    accountHolder: string; // Required if bank_transfer
    iban?: string;
  };
  mobileWalletDetails?: {
    provider: string;    // Required if mobile_wallet
    phoneNumber: string; // Required if mobile_wallet
  };
}
`

### 14. Validation Rules

| Rule | Field | Message |
|------|-------|---------|
| Required field | mount | "المبلغ مطلوب" |
| Min amount | mount | "الحد الأدنى للسحب هو 10,000 ر.ي" |
| Max amount | mount | "المبلغ يتجاوز الرصيد المتاح" |
| Required method | method | "طريقة السحب مطلوبة" |
| Bank details required | ankDetails | "تفاصيل الحساب البنكي مطلوبة" |
| Account number format | ccountNumber | "رقم الحساب غير صالح" |
| Mobile wallet required | mobileWalletDetails | "تفاصيل المحفظة مطلوبة" |
| Phone format | phoneNumber | "رقم الهاتف غير صالح" |

### 15. State Machine — Payout Lifecycle

`
  ┌──────────┐   Submit   ┌──────────┐   Approve   ┌────────────┐
  │  Request  │─────────→│  Pending  │────────────→│ Processing │
  └──────────┘           └────┬─────┘            └─────┬──────┘
                              │                        │
                         Reject│                   Complete
                              │                        │
                         ┌────▼─────┐            ┌────▼─────┐
                         │ Rejected │            │Completed │
                         └──────────┘            └──────────┘
`

### 16. Error States

| Error | Display |
|-------|---------|
| Insufficient balance | "الرصيد المتاح غير كافٍ" |
| Below minimum | "المبلغ أقل من الحد الأدنى" |
| Invalid account details | "تفاصيل الحساب غير صالحة" |
| API failure | Toast with retry |

### 17. Empty States

| Scenario | Display |
|----------|---------|
| No payout history | "لا توجد سحوبات سابقة" |
| No pending payouts | "لا توجد سحوبات معلقة" |

### 18. Loading States

- Summary cards: Skeleton.
- History table: Skeleton rows.
- Form submission: Button spinner.

### 19. Success States

- Payout requested: Toast "تم تقديم طلب السحب بنجاح" + status update.

### 20. Permission-Based UI

| Permission | UI Impact |
|------------|-----------|
| inance:read | Full view + request capability |

### 21. Security UX

- Account details masked after submission (show last 4 digits only).
- Payout requests require confirmation modal.
- Audit trail for all payout activities.

### 22. RTL/LTR Behavior

- Amount input right-aligned in RTL.
- History table columns mirror.
- Status badges consistent across directions.

### 23. Internationalization

| Key | Arabic | English |
|-----|--------|---------|
| payouts.title | إدارة السحوبات | Payout Management |
| payouts.request | طلب سحب | Request Payout |
| payouts.available | رصيد متاح | Available Balance |
| payouts.pending | معلق | Pending |
| payouts.minimum | الحد الأدنى | Minimum |
| payouts.history | سجل السحوبات | Payout History |
| payouts.method.bank | تحويل بنكي | Bank Transfer |
| payouts.method.wallet | محفظة إلكترونية | Mobile Wallet |
| payouts.status.pending | معلق | Pending |
| payouts.status.processing | قيد المعالجة | Processing |
| payouts.status.completed | مكتمل | Completed |
| payouts.status.failed | فاشل | Failed |
| payouts.status.rejected | مرفوض | Rejected |

### 24. Accessibility

- Form fields have associated labels.
- Status badges have ole="status".
- Amount input has ria-label with currency description.
- Confirmation modal has ole="dialog" with focus trap.

### 25. Performance Requirements

| Metric | Target |
|--------|--------|
| Page load | < 1.0s |
| Payout request | < 2.0s |
| History load | < 1.0s |

### 26. Analytics Events

| Event | Properties |
|-------|------------|
| payout_request_start | { amount, method } |
| payout_request_submit | { amount, method, success } |
| payout_settings_change | { field } |
| payout_history_view | { count } |

### 27. Notifications

| Trigger | Type |
|---------|------|
| Payout approved | Toast |
| Payout rejected | Toast + email |
| Payout completed | Toast + email |

### 28. Cross-Page Relationships

`
VP-FN-003 ←── VP-FN-001 (Request payout)
VP-FN-003 ──→ VP-FN-002 (View in transactions)
VP-FN-003 ──→ VP-FN-001 (Back)
`

### 29. Visual Preview

Key characteristics:
- Available balance card uses green accent.
- Pending payouts card uses amber accent.
- Status badges: green=completed, amber=pending/processing, red=failed/rejected.
- Request form has clean, minimal design.

---

## VP-RV-001: Reviews Management

### 1. Page Metadata

| Field | Value |
|-------|-------|
| **ID** | VP-RV-001 |
| **Page Name** | Reviews Management |
| **Portal** | Vendor Panel |
| **Platform** | Desktop |
| **Module** | Reviews |
| **Category** | Engagement |
| **Priority** | P1 |
| **Auth Required** | Yes |
| **Vendor Roles** | Vendor (all) |
| **Permissions** | eviews:read (own products) |
| **URL** | /vendor/reviews |
| **Parent Page** | VP-DB-001 |
| **Requirements References** | REQ-VN-REV-001, REQ-VN-REV-002 |
| **Status** | Draft |

### 2. Page Purpose

Display all reviews for the vendor's products. Allow vendors to view, filter, and respond to customer reviews. Show review analytics and sentiment overview.

### 3. Entry Points

| Source | Trigger |
|--------|---------|
| Sidebar Navigation | Click "Reviews" |
| VP-DB-001 | Reviews widget link |
| VP-PR-003 | Reviews section link |

### 4. Exit Points

| Destination | Trigger |
|-------------|---------|
| VP-PR-003 | Product link |
| VP-DB-001 | Dashboard link |

### 5. Information Architecture

`
Reviews Management
├── Header
│   ├── Title: "التقييمات"
│   └── Summary Stats (avg rating, total reviews, pending)
├── Quick Stats Strip
│   ├── Average Rating
│   ├── Total Reviews
│   ├── 5-Star Reviews
│   ├── Response Rate
│   └── Pending Reviews
├── Filter Toolbar
│   ├── Search (product name, reviewer)
│   ├── Rating Filter (1-5 stars)
│   ├── Product Filter
│   ├── Date Range
│   ├── Response Status (Responded, Pending)
│   └── Sort (Date, Rating)
├── Reviews List
│   ├── Review Card
│   │   ├── Reviewer Name
│   │   ├── Rating (stars)
│   │   ├── Date
│   │   ├── Product Name (linked)
│   │   ├── Review Text
│   │   ├── Review Images
│   │   ├── Vendor Response (if any)
│   │   ├── Response Form (if no response)
│   │   └── Helpful/Report buttons
│   └── Pagination
└── Review Analytics Widget
    ├── Rating Distribution Chart
    ├── Reviews Over Time
    └── Top Products by Rating
`

### 6. Layout Structure

`
┌─────────────────────────────────────────────────────────────────────────────┐
│  التقييمات                                                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐             │
│  │ متوسط   │ │ إجمالي  │ │ 5 نجوم  │ │ نسبة    │ │ معلق    │             │
│  │ التقييم │ │ التقييمات│ │         │ │ الرد    │ │ 3       │             │
│  │  4.7 ★  │ │   234   │ │  67%    │ │  85%    │ │         │             │
│  └─────────┘ └─────────┘ └─────────┘ └─────────┘ └─────────┘             │
│                                                                             │
│  🔍 بحث...    [التقييم ▼] [المنتج ▼] [التاريخ ▼] [الرد ▼]               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │  أحمد محمد  ★★★★★  12 سبتمبر 2026                                  │   │
│  │  المنتج: فأرة لاسلكية بلوتوث 5.0                                    │   │
│  │                                                                      │   │
│  │  فأرة ممتازة! دقة عالية واتصال بلوتوث مستقر. أنصح بها بشدة.       │   │
│  │                                                                      │   │
│  │  [صورة التقييم]                                                      │   │
│  │                                                                      │   │
│  │  رد المлетج:                                                         │   │
│  │  "شكراً لك أحمد! سعدنا بجexpérience الإيجابية. نتطلع لخدمتك مرة   │   │
│  │  أخرى."                                                              │   │
│  │                                                                      │   │
│  │  مفيد (12)  │  إبلاغ  │                                               │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │  سعيد علي  ★★★★☆  11 سبتمبر 2026                                  │   │
│  │  المنتج: كابل USB-C سريع                                             │   │
│  │                                                                      │   │
│  │  كابل جيد وسريع. لكن كانᾱ أتمنى لو كان أطول.                       │   │
│  │                                                                      │   │
│  │  [ رد على التقييم ]                                                  │   │
│  │                                                                      │   │
│  │  مفيد (5)   │  إبلاغ  │                                               │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  ◀ 1 2 3 ... 12 ▶                              عرض 1-10 من 118            │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
`

### 7. Responsive Design

| Breakpoint | Behavior |
|------------|----------|
| **Desktop (≥1024px)** | Full layout with reviews list and analytics sidebar. |
| **Tablet (768–1023px)** | Single column, analytics below reviews. |
| **Mobile (<768px)** | Not supported. |

### 8. Widget Inventory

| Widget | ID | Type |
|--------|----|------|
| Average Rating Card | v-avg-rating | Metric Card |
| Total Reviews Card | v-total | Metric Card |
| 5-Star Reviews Card | v-five-star | Metric Card |
| Response Rate Card | v-response-rate | Metric Card |
| Pending Reviews Card | v-pending | Metric Card |
| Review Card | v-review-card | Card |
| Response Form | v-response-form | Form |
| Rating Distribution | v-distribution | Bar Chart |
| Reviews Over Time | v-timeline | Line Chart |

### 9. Widget Specification — Review Card

`	sx
interface ReviewCardProps {
  review: {
    id: string;
    reviewerName: string;
    rating: number;
    date: string;
    productName: string;
    productId: string;
    text: string;
    images?: string[];
    vendorResponse?: {
      text: string;
      date: string;
    };
    helpfulCount: number;
    isReported: boolean;
  };
  onRespond: (reviewId: string, response: string) => void;
  onHelpful: (reviewId: string) => void;
  onReport: (reviewId: string) => void;
}
`

### 10. Data Requirements

| Endpoint | Method | Response |
|----------|--------|----------|
| GET /api/v1/vendors/reviews | GET | Paginated<Review> |
| GET /api/v1/vendors/reviews/summary | GET | ReviewSummary |
| POST /api/v1/vendors/reviews/:id/respond | POST | Vendor response |
| POST /api/v1/vendors/reviews/:id/helpful | POST | Mark helpful |
| POST /api/v1/vendors/reviews/:id/report | POST | Report review |

### 11. Data Loading Strategy

1. Summary stats load first.
2. Reviews list loads with skeleton.
3. Response form loads on demand.

### 12. User Actions

| Action | Target | Permission |
|--------|--------|------------|
| Respond to review | Form submit | eviews:read |
| Mark as helpful | Button click | eviews:read |
| Report review | Button click | eviews:read |
| View product | Product link | VP-PR-003 |

### 13. Form Specification

**Response Form:**
`	sx
interface ReviewResponseForm {
  reviewId: string;
  response: string; // Required, 10-500 chars
}
`

### 14. Validation Rules

| Rule | Field | Message |
|------|-------|---------|
| Required field | esponse | "الرد مطلوب" |
| Min length | esponse | "الرد يجب أن يكون 10 أحرف على الأقل" |
| Max length | esponse | "الرد يجب ألا يتجاوز 500 حرف" |

### 15. State Machine — Review Response

`
  +----------+  Respond  +----------+
  |  Pending  |--------->| Responded|
  +----------+           +----------+
       |
       | Report
       |
  +----v-----+
  | Reported |
  +----------+
`

### 16. Error States

| Error | Display |
|-------|---------|
| API failure | Toast with retry |
| Response too short | Inline error |
| Already responded | "تم الرد على هذا التقييم بالفعل" |

### 17. Empty States

| Scenario | Display |
|----------|---------|
| No reviews | "لا توجد تقييمات بعد" |
| No filtered results | "لا توجد تقييمات تطابق الفلتر" |

### 18. Loading States

- Stats: Skeleton cards.
- Reviews: 5 skeleton review cards.

### 19. Success States

- Response submitted: Toast "تم إرسال الرد بنجاح" + review updates.
- Helpful marked: Button updates with count.

### 20. Permission-Based UI

| Permission | UI Impact |
|------------|-----------|
| eviews:read | Full view + response capability |

### 21. Security UX

- Reviewer contact info not displayed.
- Reported reviews queued for admin review.

### 22. RTL/LTR Behavior

- Review cards flow right-to-left.
- Star ratings displayed consistently.
- Review text respects language direction.

### 23. Internationalization

| Key | Arabic | English |
|-----|--------|---------|
| eviews.title | التقييمات | Reviews |
| eviews.avgRating | متوسط التقييم | Average Rating |
| eviews.total | إجمالي التقييمات | Total Reviews |
| eviews.respond | رد | Respond |
| eviews.helpful | مفيد | Helpful |
| eviews.report | إبلاغ | Report |
| eviews.response | رد الملتقط | Vendor Response |
| eviews.pending | معلق | Pending |
| eviews.responded | تم الرد | Responded |

### 24. Accessibility

- Review cards have ole="article" with ria-label.
- Star ratings have ria-label="Rating: X out of 5".
- Response form has proper labels and ria-describedby for errors.

### 25. Performance Requirements

| Metric | Target |
|--------|--------|
| Page load | < 1.0s |
| Response submit | < 1.0s |
| List render (10 items) | < 300ms |

### 26. Analytics Events

| Event | Properties |
|-------|------------|
| eviews_view | { filters } |
| eview_respond | { reviewId, responseLength } |
| eview_helpful | { reviewId } |
| eview_report | { reviewId, reason } |

### 27. Notifications

| Trigger | Type |
|---------|------|
| New review received | Real-time badge update |
| Review reported | Toast confirmation |

### 28. Cross-Page Relationships

`
VP-RV-001 ←── VP-DB-001 (Reviews widget)
VP-RV-001 ←── VP-PR-003 (Reviews section)
VP-RV-001 ──→ VP-PR-003 (Product link)
VP-RV-001 ──→ VP-DB-001 (Dashboard link)
`

### 29. Visual Preview

Key characteristics:
- Star ratings use consistent gold color.
- Reviewer names in bold, dates in gray.
- Vendor response has green left border accent.
- Helpful button has subtle hover effect.

---

## VP-FN-003: Payout Management

### 1. Page Metadata

| Field | Value |
|-------|-------|
| **ID** | VP-FN-003 |
| **Page Name** | Payout Management |
| **Portal** | Vendor Panel |
| **Platform** | Desktop |
| **Module** | Finance |
| **Category** | Operations |
| **Priority** | P0 |
| **Auth Required** | Yes |
| **Vendor Roles** | Vendor (all) |
| **Permissions** | inance:read (own) |
| **URL** | /vendor/finance/payouts |
| **Parent Page** | VP-FN-001 |
| **Requirements References** | REQ-VN-FIN-004, REQ-VN-FIN-005 |
| **Status** | Draft |

### 2. Page Purpose

Manage vendor payout requests, view payout history, configure payout methods, and track pending/completed payouts. Since YemenMart uses wallet-only payments, payouts are the mechanism for vendors to withdraw earnings.

### 3. Entry Points

| Source | Trigger |
|--------|---------|
| VP-FN-001 | "Request Payout" button |
| Sidebar Navigation | Finance > Payouts |

### 4. Exit Points

| Destination | Trigger |
|-------------|---------|
| VP-FN-001 | Back button |
| VP-FN-002 | View payout details in transactions |

### 5. Information Architecture

`
Payout Management
+-- Header
|   +-- Title: "إدارة السحوبات"
|   +-- Request Payout CTA
+-- Payout Summary Cards
|   +-- Available Balance (YER)
|   +-- Pending Payouts (count + amount)
|   +-- Minimum Payout Threshold
|   +-- Last Payout Date
+-- Request Payout Form (slide-over)
|   +-- Amount Input
|   +-- Payout Method (Bank Transfer, Mobile Wallet)
|   +-- Account Details (conditional)
|   +-- Submit Button
+-- Payout History Table
|   +-- Payout ID
|   +-- Date Requested
|   +-- Amount (YER)
|   +-- Method
|   +-- Status (Pending, Processing, Completed, Failed)
|   +-- Date Completed
|   +-- Reference Number
+-- Payout Settings
    +-- Default Payout Method
    +-- Automatic Payouts toggle
    +-- Minimum Payout Amount
`

### 6. Layout Structure

`
+---------------------------------------------------------------------------+
|  إدارة السحوبات                                    [طلب سحب]              |
+---------------------------------------------------------------------------+
|                                                                           |
|  +-------------+ +-------------+ +-------------+ +---------------------+  |
|  | رصيد متاح   | | سحوبات     | | الحد الأدنى | | آخر سحب            |  |
|  | 125,000 ر.ي | | معلقة (2)  | | 10,000 ر.ي | | 1 سبتمبر 2026      |  |
|  |             | | 25,000 ر.ي  | |             | | 320,000 ر.ي         |  |
|  +-------------+ +-------------+ +-------------+ +---------------------+  |
|                                                                           |
|  +--------------------------------------------------------------------+  |
|  |  طلب سحب جديد                                                       |  |
|  |                                                                      |  |
|  |  المبلغ (ر.ي) *                                                     |  |
|  |  +--------------------------------------------------------------+  |  |
|  |  | 125,000                                                     |  |  |
|  |  +--------------------------------------------------------------+  |  |
|  |  الحد الأدنى: 10,000 ر.ي   المتاح: 125,000 ر.ي                    |  |
|  |                                                                      |  |
|  |  طريقة السحب *                                                      |  |
|  |  ( ) تحويل بنكي  ( ) محفظة إلكترونية                              |  |
|  |                                                                      |  |
|  |  تفاصيل الحساب (تحويل بنكي)                                        |  |
|  |  +--------------------------------------------------------------+  |  |
|  |  | اسم البنك:                                                  |  |  |
|  |  | رقم الحساب:                                                  |  |  |
|  |  | اسم الحساب:                                                  |  |  |
|  |  +--------------------------------------------------------------+  |  |
|  |                                                                      |  |
|  |  [ إلغاء ]                                        [ تقديم الطلب ]   |  |
|  +--------------------------------------------------------------------+  |
|                                                                           |
|  +--------------------------------------------------------------------+  |
|  |  سجل السحوبات                                                        |  |
|  |                                                                      |  |
|  |  PO-001  | 01/09  | 320,000 ر.ي | تحويل بنكي | مكتمل  | 01/09     |  |
|  |  PO-002  | 01/08  | 280,000 ر.ي | تحويل بنكي | مكتمل  | 01/08     |  |
|  |  PO-003  | 01/07  | 350,000 ر.ي | محفظة      | مكتمل  | 01/07     |  |
|  |  PO-004  | 15/09  |  15,000 ر.ي | تحويل بنكي | معلق  |           |  |
|  |  PO-005  | 12/09  |  10,000 ر.ي | محفظة      | معلق  |           |  |
|  +--------------------------------------------------------------------+  |
|                                                                           |
+---------------------------------------------------------------------------+
`

### 7. Responsive Design

| Breakpoint | Behavior |
|------------|----------|
| **Desktop (>=1024px)** | Full layout with summary cards, form, and history table. |
| **Tablet (768-1023px)** | Summary cards wrap to 2x2. Form becomes full-width. |
| **Mobile (<768px)** | Not supported. |

### 8. Widget Inventory

| Widget | ID | Type |
|--------|----|------|
| Available Balance Card | payout-available | Metric Card |
| Pending Payouts Card | payout-pending | Metric Card |
| Minimum Payout Card | payout-minimum | Metric Card |
| Last Payout Card | payout-last | Metric Card |
| Request Payout Form | payout-request-form | Form |
| Payout Method Selector | payout-method | Radio Group |
| Account Details Form | payout-account | Conditional Form |
| Payout History Table | payout-history | Data Table |
| Status Badge | payout-status-badge | Badge |

### 9. Widget Specification -- Payout Request Form

`	sx
interface PayoutRequestForm {
  amount: number;
  method: 'bank_transfer' | 'mobile_wallet';
  bankDetails?: {
    bankName: string;
    accountNumber: string;
    accountHolder: string;
    iban?: string;
  };
  mobileWalletDetails?: {
    provider: string;
    phoneNumber: string;
  };
}
`

### 10. Data Requirements

| Endpoint | Method | Response |
|----------|--------|----------|
| GET /api/v1/vendors/payouts/summary | GET | PayoutSummary |
| POST /api/v1/vendors/payouts/request | POST | PayoutRequest |
| GET /api/v1/vendors/payouts | GET | Payout[] |
| GET /api/v1/vendors/payouts/settings | GET | PayoutSettings |
| PUT /api/v1/vendors/payouts/settings | PUT | Update settings |

### 11. Data Loading Strategy

1. Summary cards load first.
2. Payout history table loads after.
3. Form is always available.

### 12. User Actions

| Action | Target | Permission |
|--------|--------|------------|
| Request payout | Form submit | inance:read |
| View payout details | Row click | inance:read |
| Change payout settings | Settings form | inance:read |

### 13. Form Specification

`	sx
interface PayoutRequestForm {
  amount: number;
  method: 'bank_transfer' | 'mobile_wallet';
  bankDetails?: {
    bankName: string;
    accountNumber: string;
    accountHolder: string;
    iban?: string;
  };
  mobileWalletDetails?: {
    provider: string;
    phoneNumber: string;
  };
}
`

### 14. Validation Rules

| Rule | Field | Message |
|------|-------|---------|
| Required field | mount | "المبلغ مطلوب" |
| Min amount | mount | "الحد الأدنى للسحب هو 10,000 ر.ي" |
| Max amount | mount | "المبلغ يتجاوز الرصيد المتاح" |
| Required method | method | "طريقة السحب مطلوبة" |
| Bank details required | ankDetails | "تفاصيل الحساب البنكي مطلوبة" |
| Account number format | ccountNumber | "رقم الحساب غير صالح" |
| Mobile wallet required | mobileWalletDetails | "تفاصيل المحفظة مطلوبة" |
| Phone format | phoneNumber | "رقم الهاتف غير صالح" |

### 15. State Machine -- Payout Lifecycle

`
  +----------+   Submit   +----------+   Approve   +------------+
  |  Request  |---------->|  Pending  |----------->| Processing |
  +----------+            +----+-----+            +-----+------+
                           |        |                    |
                      Reject|     Complete          Complete
                           |        |                    |
                      +----v----+ +----v---------+ +----v-----+
                      | Rejected | | Processing   | |Completed |
                      +----------+ +--------------+ +----------+
`

### 16. Error States

| Error | Display |
|-------|---------|
| Insufficient balance | "الرصيد المتاح غير كافٍ" |
| Below minimum | "المبلغ أقل من الحد الأدنى" |
| Invalid account details | "تفاصيل الحساب غير صالحة" |
| API failure | Toast with retry |

### 17. Empty States

| Scenario | Display |
|----------|---------|
| No payout history | "لا توجد سحوبات سابقة" |
| No pending payouts | "لا توجد سحوبات معلقة" |

### 18. Loading States

- Summary cards: Skeleton.
- History table: Skeleton rows.
- Form submission: Button spinner.

### 19. Success States

- Payout requested: Toast "تم تقديم طلب السحب بنجاح" + status update.

### 20. Permission-Based UI

| Permission | UI Impact |
|------------|-----------|
| inance:read | Full view + request capability |

### 21. Security UX

- Account details masked after submission (show last 4 digits only).
- Payout requests require confirmation modal.
- Audit trail for all payout activities.

### 22. RTL/LTR Behavior

- Amount input right-aligned in RTL.
- History table columns mirror.
- Status badges consistent across directions.

### 23. Internationalization

| Key | Arabic | English |
|-----|--------|---------|
| payouts.title | إدارة السحوبات | Payout Management |
| payouts.request | طلب سحب | Request Payout |
| payouts.available | رصيد متاح | Available Balance |
| payouts.pending | معلق | Pending |
| payouts.minimum | الحد الأدنى | Minimum |
| payouts.history | سجل السحوبات | Payout History |
| payouts.method.bank | تحويل بنكي | Bank Transfer |
| payouts.method.wallet | محفظة إلكترونية | Mobile Wallet |
| payouts.status.pending | معلق | Pending |
| payouts.status.processing | قيد المعالجة | Processing |
| payouts.status.completed | مكتمل | Completed |
| payouts.status.failed | فاشل | Failed |
| payouts.status.rejected | مرفوض | Rejected |

### 24. Accessibility

- Form fields have associated labels.
- Status badges have ole="status".
- Amount input has ria-label with currency description.
- Confirmation modal has ole="dialog" with focus trap.

### 25. Performance Requirements

| Metric | Target |
|--------|--------|
| Page load | < 1.0s |
| Payout request | < 2.0s |
| History load | < 1.0s |

### 26. Analytics Events

| Event | Properties |
|-------|------------|
| payout_request_start | { amount, method } |
| payout_request_submit | { amount, method, success } |
| payout_settings_change | { field } |
| payout_history_view | { count } |

### 27. Notifications

| Trigger | Type |
|---------|------|
| Payout approved | Toast |
| Payout rejected | Toast + email |
| Payout completed | Toast + email |

### 28. Cross-Page Relationships

`
VP-FN-003 <-- VP-FN-001 (Request payout)
VP-FN-003 --> VP-FN-002 (View in transactions)
VP-FN-003 --> VP-FN-001 (Back)
`

### 29. Visual Preview

Key characteristics:
- Available balance card uses green accent.
- Pending payouts card uses amber accent.
- Status badges: green=completed, amber=pending/processing, red=failed/rejected.
- Request form has clean, minimal design.

---

## VP-RV-001: Reviews Management

### 1. Page Metadata

| Field | Value |
|-------|-------|
| **ID** | VP-RV-001 |
| **Page Name** | Reviews Management |
| **Portal** | Vendor Panel |
| **Platform** | Desktop |
| **Module** | Reviews |
| **Category** | Engagement |
| **Priority** | P1 |
| **Auth Required** | Yes |
| **Vendor Roles** | Vendor (all) |
| **Permissions** | eviews:read (own products) |
| **URL** | /vendor/reviews |
| **Parent Page** | VP-DB-001 |
| **Requirements References** | REQ-VN-REV-001, REQ-VN-REV-002 |
| **Status** | Draft |

### 2. Page Purpose

Display all reviews for the vendor's products. Allow vendors to view, filter, and respond to customer reviews. Show review analytics and sentiment overview.

### 3. Entry Points

| Source | Trigger |
|--------|---------|
| Sidebar Navigation | Click "Reviews" |
| VP-DB-001 | Reviews widget link |
| VP-PR-003 | Reviews section link |

### 4. Exit Points

| Destination | Trigger |
|-------------|---------|
| VP-PR-003 | Product link |
| VP-DB-001 | Dashboard link |

### 5. Information Architecture

`
Reviews Management
+-- Header
|   +-- Title: "التقييمات"
|   +-- Summary Stats (avg rating, total reviews, pending)
+-- Quick Stats Strip
|   +-- Average Rating
|   +-- Total Reviews
|   +-- 5-Star Reviews
|   +-- Response Rate
|   +-- Pending Reviews
+-- Filter Toolbar
|   +-- Search (product name, reviewer)
|   +-- Rating Filter (1-5 stars)
|   +-- Product Filter
|   +-- Date Range
|   +-- Response Status (Responded, Pending)
|   +-- Sort (Date, Rating)
+-- Reviews List
|   +-- Review Card
|   |   +-- Reviewer Name
|   |   +-- Rating (stars)
|   |   +-- Date
|   |   +-- Product Name (linked)
|   |   +-- Review Text
|   |   +-- Review Images
|   |   +-- Vendor Response (if any)
|   |   +-- Response Form (if no response)
|   |   +-- Helpful/Report buttons
|   +-- Pagination
+-- Review Analytics Widget
    +-- Rating Distribution Chart
    +-- Reviews Over Time
    +-- Top Products by Rating
`

### 6. Layout Structure

`
+---------------------------------------------------------------------------+
|  التقييمات                                                                  |
+---------------------------------------------------------------------------+
|                                                                           |
|  +---------+ +---------+ +---------+ +---------+ +---------+             |
|  | متوسط   | | إجمالي  | | 5 نجوم  | | نسبة    | | معلق    |             |
|  | التقييم | | التقييمات| |         | | الرد    | | 3       |             |
|  |  4.7    | |   234   | |  67%    | |  85%    | |         |             |
|  +---------+ +---------+ +---------+ +---------+ +---------+             |
|                                                                           |
|  Search...    [التقييم] [المنتج] [التاريخ] [الرد]                        |
+---------------------------------------------------------------------------+
|                                                                           |
|  +--------------------------------------------------------------------+  |
|  |  أحمد محمد  ★★★★★  12 سبتمبر 2026                                  |  |
|  |  المنتج: فأرة لاسلكية بلوتوث 5.0                                    |  |
|  |                                                                      |  |
|  |  فأرة ممتازة! دقة عالية واتصال بلوتوث مستقر. أنصح بها بشدة.       |  |
|  |                                                                      |  |
|  |  [صورة التقييم]                                                      |  |
|  |                                                                      |  |
|  |  رد الملتقط:                                                        |  |
|  |  "شكراً لك أحمد! سعدنا بالتجربة الإيجابية."                        |  |
|  |                                                                      |  |
|  |  مفيد (12)  |  إبلاغ                                                  |  |
|  +--------------------------------------------------------------------+  |
|                                                                           |
|  +--------------------------------------------------------------------+  |
|  |  سعيد علي  ★★★★☆  11 سبتمبر 2026                                  |  |
|  |  المنتج: كابل USB-C سريع                                             |  |
|  |                                                                      |  |
|  |  كابل جيد وسريع. لكن كنت أتمنى لو كان أطول.                       |  |
|  |                                                                      |  |
|  |  [ رد على التقييم ]                                                  |  |
|  |                                                                      |  |
|  |  مفيد (5)   |  إبلاغ                                                  |  |
|  +--------------------------------------------------------------------+  |
|                                                                           |
|  < 1 2 3 ... 12 >                              عرض 1-10 من 118            |
|                                                                           |
+---------------------------------------------------------------------------+
`

### 7. Responsive Design

| Breakpoint | Behavior |
|------------|----------|
| **Desktop (>=1024px)** | Full layout with reviews list and analytics sidebar. |
| **Tablet (768-1023px)** | Single column, analytics below reviews. |
| **Mobile (<768px)** | Not supported. |

### 8. Widget Inventory

| Widget | ID | Type |
|--------|----|------|
| Average Rating Card | v-avg-rating | Metric Card |
| Total Reviews Card | v-total | Metric Card |
| 5-Star Reviews Card | v-five-star | Metric Card |
| Response Rate Card | v-response-rate | Metric Card |
| Pending Reviews Card | v-pending | Metric Card |
| Review Card | v-review-card | Card |
| Response Form | v-response-form | Form |
| Rating Distribution | v-distribution | Bar Chart |
| Reviews Over Time | v-timeline | Line Chart |

### 9. Widget Specification -- Review Card

`	sx
interface ReviewCardProps {
  review: {
    id: string;
    reviewerName: string;
    rating: number;
    date: string;
    productName: string;
    productId: string;
    text: string;
    images?: string[];
    vendorResponse?: {
      text: string;
      date: string;
    };
    helpfulCount: number;
    isReported: boolean;
  };
  onRespond: (reviewId: string, response: string) => void;
  onHelpful: (reviewId: string) => void;
  onReport: (reviewId: string) => void;
}
`

### 10. Data Requirements

| Endpoint | Method | Response |
|----------|--------|----------|
| GET /api/v1/vendors/reviews | GET | Paginated<Review> |
| GET /api/v1/vendors/reviews/summary | GET | ReviewSummary |
| POST /api/v1/vendors/reviews/:id/respond | POST | Vendor response |
| POST /api/v1/vendors/reviews/:id/helpful | POST | Mark helpful |
| POST /api/v1/vendors/reviews/:id/report | POST | Report review |

### 11. Data Loading Strategy

1. Summary stats load first.
2. Reviews list loads with skeleton.
3. Response form loads on demand.

### 12. User Actions

| Action | Target | Permission |
|--------|--------|------------|
| Respond to review | Form submit | eviews:read |
| Mark as helpful | Button click | eviews:read |
| Report review | Button click | eviews:read |
| View product | Product link | VP-PR-003 |

### 13. Form Specification

**Response Form:**
`	sx
interface ReviewResponseForm {
  reviewId: string;
  response: string; // Required, 10-500 chars
}
`

### 14. Validation Rules

| Rule | Field | Message |
|------|-------|---------|
| Required field | esponse | "الرد مطلوب" |
| Min length | esponse | "الرد يجب أن يكون 10 أحرف على الأقل" |
| Max length | esponse | "الرد يجب ألا يتجاوز 500 حرف" |

### 15. State Machine -- Review Response

`
  +----------+  Respond  +----------+
  |  Pending  |--------->| Responded|
  +----------+           +----------+
       |
       | Report
       |
  +----v-----+
  | Reported |
  +----------+
`

### 16. Error States

| Error | Display |
|-------|---------|
| API failure | Toast with retry |
| Response too short | Inline error |
| Already responded | "تم الرد على هذا التقييم بالفعل" |

### 17. Empty States

| Scenario | Display |
|----------|---------|
| No reviews | "لا توجد تقييمات بعد" |
| No filtered results | "لا توجد تقييمات تطابق الفلتر" |

### 18. Loading States

- Stats: Skeleton cards.
- Reviews: 5 skeleton review cards.

### 19. Success States

- Response submitted: Toast "تم إرسال الرد بنجاح" + review updates.
- Helpful marked: Button updates with count.

### 20. Permission-Based UI

| Permission | UI Impact |
|------------|-----------|
| eviews:read | Full view + response capability |

### 21. Security UX

- Reviewer contact info not displayed.
- Reported reviews queued for admin review.

### 22. RTL/LTR Behavior

- Review cards flow right-to-left.
- Star ratings displayed consistently.
- Review text respects language direction.

### 23. Internationalization

| Key | Arabic | English |
|-----|--------|---------|
| eviews.title | التقييمات | Reviews |
| eviews.avgRating | متوسط التقييم | Average Rating |
| eviews.total | إجمالي التقييمات | Total Reviews |
| eviews.respond | رد | Respond |
| eviews.helpful | مفيد | Helpful |
| eviews.report | إبلاغ | Report |
| eviews.response | رد الملتقط | Vendor Response |
| eviews.pending | معلق | Pending |
| eviews.responded | تم الرد | Responded |

### 24. Accessibility

- Review cards have ole="article" with ria-label.
- Star ratings have ria-label="Rating: X out of 5".
- Response form has proper labels and ria-describedby for errors.

### 25. Performance Requirements

| Metric | Target |
|--------|--------|
| Page load | < 1.0s |
| Response submit | < 1.0s |
| List render (10 items) | < 300ms |

### 26. Analytics Events

| Event | Properties |
|-------|------------|
| eviews_view | { filters } |
| eview_respond | { reviewId, responseLength } |
| eview_helpful | { reviewId } |
| eview_report | { reviewId, reason } |

### 27. Notifications

| Trigger | Type |
|---------|------|
| New review received | Real-time badge update |
| Review reported | Toast confirmation |

### 28. Cross-Page Relationships

`
VP-RV-001 <-- VP-DB-001 (Reviews widget)
VP-RV-001 <-- VP-PR-003 (Reviews section)
VP-RV-001 --> VP-PR-003 (Product link)
VP-RV-001 --> VP-DB-001 (Dashboard link)
`

### 29. Visual Preview

Key characteristics:
- Star ratings use consistent gold color.
- Reviewer names in bold, dates in gray.
- Vendor response has green left border accent.
- Helpful button has subtle hover effect.

---

## VP-KY-001: KYC Verification

### 1. Page Metadata

| Field | Value |
|-------|-------|
| **ID** | VP-KY-001 |
| **Page Name** | KYC Verification |
| **Portal** | Vendor Panel |
| **Platform** | Desktop |
| **Module** | Compliance |
| **Category** | Verification |
| **Priority** | P0 |
| **Auth Required** | Yes |
| **Vendor Roles** | Vendor (all) |
| **Permissions** | Any authenticated vendor |
| **URL** | /vendor/kyc |
| **Parent Page** | VP-DB-001 |
| **Requirements References** | REQ-VN-KYC-001, REQ-VN-KYC-002 |
| **Status** | Draft |

### 2. Page Purpose

Guide vendors through the Know Your Customer (KYC) verification process. Required for enabling payouts and full platform access. Multi-step form collecting business/personal information and document uploads.

### 3. Entry Points

| Source | Trigger |
|--------|---------|
| Sidebar Navigation | Click "KYC" |
| VP-DB-001 | KYC banner CTA |
| Onboarding flow | After first login |

### 4. Exit Points

| Destination | Trigger |
|-------------|---------|
| VP-DB-001 | After submission / Back button |
| VP-FN-003 | After verification (payout enabled) |

### 5. Information Architecture

`
KYC Verification
+-- Status Banner (if pending/approved/rejected)
+-- Progress Stepper
|   +-- Step 1: Personal Information
|   +-- Step 2: Business Information
|   +-- Step 3: Document Upload
|   +-- Step 4: Review & Submit
+-- Step Content
+-- Navigation
|   +-- [ رجوع / Back ]
|   +-- [ التالي / Next ] (or [ إرسال / Submit ])
+-- Rejection Details (if rejected)
`

### 6. Layout Structure

`
+---------------------------------------------------------------------------+
|  التحقق من الهوية (KYC)                                                    |
+---------------------------------------------------------------------------+
|                                                                           |
|  +--------------------------------------------------------------------+  |
|  |  Step 1  Step 2  Step 3  Step 4                                     |  |
|  |  شخصية   عمل   مستندات  مراجعة                                      |  |
|  +--------------------------------------------------------------------+  |
|                                                                           |
|  +--------------------------------------------------------------------+  |
|  |                                                                      |  |
|  |  معلومات العمل                                                       |  |
|  |                                                                      |  |
|  |  اسم النشاط التجاري *                                                |  |
|  |  +--------------------------------------------------------------+  |  |
|  |  | متجر إلكترونيات اليمن                                         |  |  |
|  |  +--------------------------------------------------------------+  |  |
|  |                                                                      |  |
|  |  رقم السجل التجاري *                                                |  |
|  |  +--------------------------------------------------------------+  |  |
|  |  | CR-123456                                                    |  |  |
|  |  +--------------------------------------------------------------+  |  |
|  |                                                                      |  |
|  |  نوع النشاط *                                                       |  |
|  |  +--------------------------------------------------------------+  |  |
|  |  | بيع بالتجزئة - إلكترونيات                                   |  |  |
|  |  +--------------------------------------------------------------+  |  |
|  |                                                                      |  |
|  |  عنوان النشاط *                                                     |  |
|  |  +--------------------------------------------------------------+  |  |
|  |  | صنعاء، شارع الظهران، nhà 25                                 |  |  |
|  |  +--------------------------------------------------------------+  |  |
|  |                                                                      |  |
|  |  [ رجوع ]                                      [ التالي ]           |  |
|  +--------------------------------------------------------------------+  |
|                                                                           |
+---------------------------------------------------------------------------+
`

### 7. Responsive Design

| Breakpoint | Behavior |
|------------|----------|
| **Desktop (>=1024px)** | Full layout with stepper and form. |
| **Tablet (768-1023px)** | Stepper wraps, form full-width. |
| **Mobile (<768px)** | Not supported. |

### 8. Widget Inventory

| Widget | ID | Type |
|--------|----|------|
| Status Banner | kyc-status-banner | Alert Banner |
| Progress Stepper | kyc-stepper | Stepper |
| Personal Info Form | kyc-personal | Form |
| Business Info Form | kyc-business | Form |
| Document Upload | kyc-documents | File Upload |
| Review Summary | kyc-review | Summary |
| Rejection Details | kyc-rejection | Alert Card |

### 9. Widget Specification -- Document Upload

`	sx
interface DocumentUploadProps {
  documents: KYCDocument[];
  maxFiles: number;
  acceptedTypes: string[];
  onUpload: (files: File[]) => Promise<void>;
  onRemove: (id: string) => void;
}

interface KYCDocument {
  id: string;
  type: 'national_id' | 'commercial_registration' | 'tax_certificate' | 'bank_statement';
  fileName: string;
  uploadDate: string;
  status: 'pending' | 'approved' | 'rejected';
  rejectionReason?: string;
}
`

### 10. Data Requirements

| Endpoint | Method | Response |
|----------|--------|----------|
| GET /api/v1/vendors/kyc | GET | KYCStatus |
| POST /api/v1/vendors/kyc/personal | POST | Save personal info |
| POST /api/v1/vendors/kyc/business | POST | Save business info |
| POST /api/v1/vendors/kyc/documents | POST | Upload documents |
| POST /api/v1/vendors/kyc/submit | POST | Submit for review |
| GET /api/v1/vendors/kyc/status | GET | Current status |

### 11. Data Loading Strategy

1. Current KYC status loads first.
2. Form pre-fills with saved data.
3. Documents load after business info.

### 12. User Actions

| Action | Target | Permission |
|--------|--------|------------|
| Save personal info | Form submit | Any vendor |
| Save business info | Form submit | Any vendor |
| Upload documents | File picker | Any vendor |
| Submit for review | Submit button | Any vendor |

### 13. Form Specification

**Step 1 -- Personal Information:**
`	sx
interface PersonalInfoForm {
  fullName: string;        // Required
  nationalId: string;      // Required
  dateOfBirth: string;     // Required
  phone: string;           // Required
  email: string;           // Required
  address: string;         // Required
}
`

**Step 2 -- Business Information:**
`	sx
interface BusinessInfoForm {
  businessName: string;    // Required
  commercialRegistration: string; // Required
  businessType: string;    // Required
  businessAddress: string; // Required
  taxId?: string;          // Optional
  establishedYear?: number;
}
`

**Step 3 -- Document Upload:**
`	sx
interface DocumentUploadForm {
  nationalIdFront: File;   // Required
  nationalIdBack: File;    // Required
  commercialRegistration: File; // Required
  taxCertificate?: File;   // Optional
  bankStatement?: File;    // Optional
}
`

### 14. Validation Rules

| Rule | Field | Message |
|------|-------|---------|
| Required field | ullName | "الاسم الكامل مطلوب" |
| Required field | 
ationalId | "رقم الهوية مطلوب" |
| Required field | phone | "رقم الهاتف مطلوب" |
| Required field | email | "البريد الإلكتروني مطلوب" |
| Required field | usinessName | "اسم النشاط مطلوب" |
| Required field | commercialRegistration | "رقم السجل التجاري مطلوب" |
| Document size | file | "حجم الملف يجب ألا يتجاوز 10 ميجابايت" |
| Document type | file | "يُسمح فقط بصور JPEG, PNG, PDF" |

### 15. State Machine -- KYC Status

`
  +----------+  Submit   +----------+  Approve  +----------+
  |  Draft   |--------->| Pending  |---------->| Approved |
  +----------+           +----+-----+           +----------+
                          |        |
                     Reject|     Review
                          |        |
                     +----v----+ +----v-----+
                     |Rejected | | Under    |
                     +----+----+ | Review   |
                          |      +----------+
                          |
                     Resubmit
                          |
                     +----v----+
                     |  Draft  |
                     +---------+
`

### 16. Error States

| Error | Display |
|-------|---------|
| Upload failure | Toast with retry |
| Validation failure | Inline errors |
| Submission failure | Toast with error details |

### 17. Empty States

| Scenario | Display |
|----------|---------|
| No documents uploaded | "ارفع المستندات المطلوبة للمتابعة" |

### 18. Loading States

- Status banner: Skeleton.
- Form: Pre-filled or skeleton on first load.

### 19. Success States

- Personal info saved: Toast "تم حفظ المعلومات الشخصية"
- Business info saved: Toast "تم حفظ معلومات العمل"
- Documents uploaded: Checkmark on each uploaded document
- Submission: Toast "تم إرسال طلب التحقق بنجاح"

### 20. Permission-Based UI

| Permission | UI Impact |
|------------|-----------|
| Any authenticated vendor | Full KYC flow |

### 21. Security UX

- Document uploads are encrypted at rest.
- Sensitive data (national ID) masked after submission.
- Audit trail for all KYC activities.

### 22. RTL/LTR Behavior

- Form labels right-aligned in RTL.
- Stepper flows right-to-left.
- Document previews centered.

### 23. Internationalization

| Key | Arabic | English |
|-----|--------|---------|
| kyc.title | التحقق من الهوية | KYC Verification |
| kyc.step.personal | المعلومات الشخصية | Personal Information |
| kyc.step.business | معلومات العمل | Business Information |
| kyc.step.documents | رفع المستندات | Document Upload |
| kyc.step.review | مراجعة وإرسال | Review & Submit |
| kyc.status.pending | قيد المراجعة | Under Review |
| kyc.status.approved | موثق ✓ | Verified ✓ |
| kyc.status.rejected | مرفوض | Rejected |
| kyc.submit | إرسال طلب التحقق | Submit Verification |

### 24. Accessibility

- All form fields have associated labels.
- Document uploads have ria-label with file type instructions.
- Status banner has ole="alert".
- Stepper uses ole="tablist" / ole="tab".

### 25. Performance Requirements

| Metric | Target |
|--------|--------|
| Page load | < 1.0s |
| Document upload (per file) | < 5s (10MB) |
| Form submit | < 2.0s |

### 26. Analytics Events

| Event | Properties |
|-------|------------|
| kyc_start | { step } |
| kyc_step_complete | { step, timeSpent } |
| kyc_document_upload | { documentType, fileSize } |
| kyc_submit | { documentCount } |

### 27. Notifications

| Trigger | Type |
|---------|------|
| KYC approved | Toast + email |
| KYC rejected | Toast + email + banner update |
| Document rejected | Toast with reason |

### 28. Cross-Page Relationships

`
VP-KY-001 <-- VP-DB-001 (KYC banner)
VP-KY-001 --> VP-DB-001 (After submission)
VP-KY-001 --> VP-FN-003 (After approval)
`

### 29. Visual Preview

Key characteristics:
- Stepper uses green for completed, gray for pending.
- Document upload zones have dashed borders.
- Status banner changes color based on status.
- Form has clean, professional appearance.

---

## VP-AN-001: Analytics Dashboard

### 1. Page Metadata

| Field | Value |
|-------|-------|
| **ID** | VP-AN-001 |
| **Page Name** | Analytics Dashboard |
| **Portal** | Vendor Panel |
| **Platform** | Desktop |
| **Module** | Analytics |
| **Category** | Reporting |
| **Priority** | P1 |
| **Auth Required** | Yes |
| **Vendor Roles** | Vendor (all) |
| **Permissions** | Any authenticated vendor |
| **URL** | /vendor/analytics |
| **Parent Page** | VP-DB-001 |
| **Requirements References** | REQ-VN-ANAL-001 |
| **Status** | Draft |

### 2. Page Purpose

Provide vendors with detailed analytics and insights about their store performance, customer behavior, product performance, and sales trends.

### 3. Entry Points

| Source | Trigger |
|--------|---------|
| Sidebar Navigation | Click "Analytics" |
| VP-DB-001 | Dashboard shortcuts |

### 4. Exit Points

| Destination | Trigger |
|-------------|---------|
| VP-DB-001 | Back button |
| VP-PR-001 | Product link |
| VP-OR-001 | Order link |

### 5. Information Architecture

`
Analytics Dashboard
+-- Header
|   +-- Title: "التحليلات"
|   +-- Period Selector (7d, 30d, 90d, Custom)
+-- KPI Summary
|   +-- Revenue Growth
|   +-- Order Growth
|   +-- Conversion Rate
|   +-- Average Order Value
+-- Sales Trends Chart
|   +-- Revenue over time
|   +-- Orders over time
+-- Top Products Widget
|   +-- Products by revenue
|   +-- Products by quantity
|   +-- Products by views
+-- Customer Insights
|   +-- New vs Returning
|   +-- Geographic distribution
|   +-- Peak hours
+-- Conversion Funnel
|   +-- Views -> Cart -> Purchase
|   +-- Drop-off analysis
+-- Export Options
    +-- Export as PDF
    +-- Export as CSV
`

### 6. Layout Structure

`
+---------------------------------------------------------------------------+
|  التحليلات                        [7 أيام] [تصدير PDF] [تصدير CSV]       |
+---------------------------------------------------------------------------+
|                                                                           |
|  +-------------+ +-------------+ +-------------+ +-------------+         |
|  | نمو الإيرادات| | نمو الطلبات| | معدل التحويل| | متوسط       |         |
|  |   +23%      | |   +18%      | |   3.2%      | | قيمة الطلب  |         |
|  |             | |             | |             | | 8,500 ر.ي   |         |
|  +-------------+ +-------------+ +-------------+ +-------------+         |
|                                                                           |
|  +--------------------------------------------------------------------+  |
|  |  اتجاهات المبيعات                                                   |  |
|  |                                                                      |  |
|  |  (line chart showing revenue and orders over time)                  |  |
|  |                                                                      |  |
|  |  ------- Revenue    ------- Orders                                  |  |
|  +--------------------------------------------------------------------+  |
|                                                                           |
|  +------------------------+ +------------------------------------------+  |
|  |  أفضل المنتجات        | |  رؤى الزبائن                            |  |
|  |                        | |                                          |  |
|  |  1. فأرة لاسلكية 45,200| |  جدد: 34%  |  عائدون: 66%              |  |
|  |  2. كابل USB-C   28,900| |                                          |  |
|  |  3. لوحة مفاتيح  22,100| |  الفئة 25-34: 45%                       |  |
|  |  4. موزع HDMI    18,500| |  الفئة 35-44: 28%                       |  |
|  |  5. سماعات       15,200| |  الفئة 18-24: 15%                       |  |
|  |                        | |  الفئة 45+: 12%                         |  |
|  |  [ عرض كل المنتجات ]  | |                                          |  |
|  +------------------------+ +------------------------------------------+  |
|                                                                           |
|  +--------------------------------------------------------------------+  |
|  |  قمع التحويل                                                       |  |
|  |                                                                      |  |
|  |  المشاهدات (100%) --> السلة (32%) --> الشراء (3.2%)                 |  |
|  |                                                                      |  |
|  |  نقاط الانسحاب الرئيسية:                                           |  |
|  |  - صفحة المنتج --> السلة: 68% يتركون                                |  |
|  |  - السلة --> الدفع: 90% يتركون                                     |  |
|  +--------------------------------------------------------------------+  |
|                                                                           |
+---------------------------------------------------------------------------+
`

### 7. Responsive Design

| Breakpoint | Behavior |
|------------|----------|
| **Desktop (>=1024px)** | Full layout with 2-column sections. |
| **Tablet (768-1023px)** | Single column, sections stack. |
| **Mobile (<768px)** | Not supported. |

### 8. Widget Inventory

| Widget | ID | Type |
|--------|----|------|
| Revenue Growth Card | n-revenue-growth | Metric Card |
| Order Growth Card | n-order-growth | Metric Card |
| Conversion Rate Card | n-conversion | Metric Card |
| Average Order Value Card | n-aov | Metric Card |
| Sales Trends Chart | n-sales-trends | Line Chart |
| Top Products | n-top-products | List |
| Customer Insights | n-customer-insights | Chart + Stats |
| Conversion Funnel | n-funnel | Funnel Chart |

### 9. Widget Specification -- Conversion Funnel

`	sx
interface ConversionFunnelProps {
  stages: {
    name: string;
    count: number;
    percentage: number;
    dropOff: number;
  }[];
}
`

### 10. Data Requirements

| Endpoint | Method | Response |
|----------|--------|----------|
| GET /api/v1/vendors/analytics/summary | GET | AnalyticsSummary |
| GET /api/v1/vendors/analytics/sales-trends | GET | SalesTrend[] |
| GET /api/v1/vendors/analytics/top-products | GET | TopProduct[] |
| GET /api/v1/vendors/analytics/customer-insights | GET | CustomerInsights |
| GET /api/v1/vendors/analytics/conversion-funnel | GET | ConversionFunnel |

### 11. Data Loading Strategy

1. Summary KPIs load first.
2. Charts load after KPIs.
3. Detailed lists load last.

### 12. User Actions

| Action | Target | Permission |
|--------|--------|------------|
| Change period | Period selector | Any vendor |
| Export PDF | Export button | Any vendor |
| Export CSV | Export button | Any vendor |
| View product details | Product link | VP-PR-003 |

### 13. Form Specification

No forms. Period selector is a dropdown.

### 14. Validation Rules

N/A -- Read-only analytics.

### 15. State Machine

Same pattern as VP-DB-001.

### 16. Error States

| Error | Display |
|-------|---------|
| API failure | Error card with retry |
| Export failure | Toast with retry |

### 17. Empty States

| Scenario | Display |
|----------|---------|
| No data | "ستظهر التحليلات بعد تلقي أول طلب" |

### 18. Loading States

- KPI cards: Skeleton.
- Charts: Skeleton axes.
- Lists: 5 skeleton rows.

### 19. Success States

- Export: Download starts + toast.

### 20. Permission-Based UI

| Permission | UI Impact |
|------------|-----------|
| Any authenticated vendor | Full analytics view |

### 21. Security UX

- Analytics data only visible to vendor owner.
- Exported reports include watermark.

### 22. RTL/LTR Behavior

- Chart axes respect RTL.
- List items flow right-to-left.
- Funnel chart flows right-to-left.

### 23. Internationalization

| Key | Arabic | English |
|-----|--------|---------|
| nalytics.title | التحليلات | Analytics |
| nalytics.revenueGrowth | نمو الإيرادات | Revenue Growth |
| nalytics.orderGrowth | نمو الطلبات | Order Growth |
| nalytics.conversionRate | معدل التحويل | Conversion Rate |
| nalytics.avgOrderValue | متوسط قيمة الطلب | Average Order Value |
| nalytics.topProducts | أفضل المنتجات | Top Products |
| nalytics.customerInsights | رؤى الزبائن | Customer Insights |
| nalytics.conversionFunnel | قمع التحويل | Conversion Funnel |
| nalytics.newCustomers | جدد | New |
| nalytics.returningCustomers | عائدون | Returning |

### 24. Accessibility

- Charts have ria-label and hidden data tables.
- KPI cards have ria-label with full text.
- Funnel stages have ole="listitem" with progress description.

### 25. Performance Requirements

| Metric | Target |
|--------|--------|
| Page load | < 1.5s |
| Chart render | < 500ms |
| Export generation | < 5s |

### 26. Analytics Events

| Event | Properties |
|-------|------------|
| nalytics_view | { period } |
| nalytics_period_change | { from, to } |
| nalytics_export | { format } |

### 27. Notifications

| Trigger | Type |
|---------|------|
| Weekly report available | Badge update |

### 28. Cross-Page Relationships

`
VP-AN-001 <-- VP-DB-001 (Analytics link)
VP-AN-001 --> VP-PR-001 (Product details)
VP-AN-001 --> VP-OR-001 (Order details)
VP-AN-001 --> VP-DB-001 (Back)
`

### 29. Visual Preview

Key characteristics:
- Growth metrics use green for positive, red for negative.
- Charts use consistent color scheme.
- Conversion funnel uses gradient colors.
- Clean, data-focused design.

---

## VP-NT-001: Notifications

### 1. Page Metadata

| Field | Value |
|-------|-------|
| **ID** | VP-NT-001 |
| **Page Name** | Notifications |
| **Portal** | Vendor Panel |
| **Platform** | Desktop |
| **Module** | System |
| **Category** | Communication |
| **Priority** | P1 |
| **Auth Required** | Yes |
| **Vendor Roles** | Vendor (all) |
| **Permissions** | Any authenticated vendor |
| **URL** | /vendor/notifications |
| **Parent Page** | VP-DB-001 |
| **Requirements References** | REQ-VN-NOTIF-001 |
| **Status** | Draft |

### 2. Page Purpose

Display all vendor notifications including order updates, stock alerts, review notifications, system announcements, and KYC status changes. Allow notification management (read, archive, delete).

### 3. Entry Points

| Source | Trigger |
|--------|---------|
| Sidebar Navigation | Click "Notifications" |
| VP-DB-001 | Notification bell |
| Deep Link | Direct URL |

### 4. Exit Points

| Destination | Trigger |
|-------------|---------|
| VP-OR-002 | Order notification click |
| VP-RV-001 | Review notification click |
| VP-KY-001 | KYC notification click |
| VP-DB-001 | Back button |

### 5. Information Architecture

`
Notifications
+-- Header
|   +-- Title: "الإشعارات"
|   +-- Unread Count Badge
|   +-- Actions (Mark all read, Clear all)
+-- Filter Tabs
|   +-- All
|   +-- Unread
|   +-- Orders
|   +-- Reviews
|   +-- System
|   +-- KYC
+-- Notifications List
|   +-- Notification Item
|   |   +-- Icon (type-specific)
|   |   +-- Title
|   |   +-- Description
|   |   +-- Timestamp
|   |   +-- Read/Unread indicator
|   |   +-- Action link (if applicable)
|   +-- Pagination
+-- Notification Settings (link)
`

### 6. Layout Structure

`
+---------------------------------------------------------------------------+
|  الإشعارات (12 غير مقروءة)              [تعيين الكل كمقروء] [مسح الكل]   |
+---------------------------------------------------------------------------+
|  [الكل (45)] [غير مقروءة (12)] [طلبات (8)] [تقييمات (5)] [نظام (3)] [KYC (1)] |
+---------------------------------------------------------------------------+
|                                                                           |
|  ●  📦 طلب جديد #ORD-8847                     قبل 5 دقائق                |
|     أحمد محمد طلب 3 منتجات بقيمة 12,500 ر.ي                              |
|     [عرض الطلب -->]                                                       |
|                                                                           |
|  ●  ⭐ تقييم جديد على فأرة لاسلكية                قبل 30 دقيقة            |
|     أحمد محمد أضاف تقييم 5 نجوم                                           |
|     [عرض التقييم -->]                                                      |
|                                                                           |
|  ●  ⚠️ تنبيه مخزون منخفض: كابل USB-C             قبل ساعة                 |
|     المخزون وصل إلى 5 وحدات (الحد الأدنى: 10)                            |
|     [عرض المنتج -->]                                                       |
|                                                                           |
|    📧 تم تحديث حالة الطلب #ORD-8843                  قبل 2 ساعة           |
|     تم الشحن - رقم التتبع: YE123456789                                   |
|     [عرض الطلب -->]                                                       |
|                                                                           |
|    📧 تم الدفع للطلب #ORD-8841                      قبل 3 ساعات          |
|     المبلغ: 28,900 ر.ي تم خصمه من محفظة الزبون                          |
|     [عرض الطلب -->]                                                       |
|                                                                           |
|    ✅ تم التحقق من هويتك بنجاح                      قبل يوم               |
|     يمكنك الآن طلب سحوبات الأرباح                                         |
|     [عرض السحوبات -->]                                                     |
|                                                                           |
|    📢 تحديث النظام: ميزات جديدة متاحة                قبل يومين            |
|     تم إضافة تحليلات جديدة للمتجر                                         |
|     [المزيد -->]                                                          |
|                                                                           |
|  < 1 2 3 ... 5 >                                   عرض 1-10 من 45       |
|                                                                           |
+---------------------------------------------------------------------------+
`

### 7. Responsive Design

| Breakpoint | Behavior |
|------------|----------|
| **Desktop (>=1024px)** | Full layout with filter tabs. |
| **Tablet (768-1023px)** | Filter tabs wrap to 2 rows. |
| **Mobile (<768px)** | Not supported. |

### 8. Widget Inventory

| Widget | ID | Type |
|--------|----|------|
| Unread Count Badge | 
t-unread-badge | Badge |
| Filter Tabs | 
t-tabs | Tab Group |
| Notification Item | 
t-item | List Item |
| Mark All Read Button | 
t-mark-all | Button |
| Clear All Button | 
t-clear-all | Button |

### 9. Widget Specification -- Notification Item

`	sx
interface NotificationItemProps {
  notification: {
    id: string;
    type: 'order' | 'review' | 'stock' | 'kyc' | 'system' | 'payout';
    title: string;
    description: string;
    timestamp: string;
    isRead: boolean;
    actionUrl?: string;
    actionLabel?: string;
  };
  onRead: (id: string) => void;
  onArchive: (id: string) => void;
  onAction: (url: string) => void;
}
`

### 10. Data Requirements

| Endpoint | Method | Response |
|----------|--------|----------|
| GET /api/v1/vendors/notifications | GET | Paginated<Notification> |
| GET /api/v1/vendors/notifications/counts | GET | Record<type, number> |
| PATCH /api/v1/vendors/notifications/:id/read | PATCH | Mark as read |
| PATCH /api/v1/vendors/notifications/read-all | PATCH | Mark all as read |
| DELETE /api/v1/vendors/notifications/:id | DELETE | Delete notification |
| DELETE /api/v1/vendors/notifications | DELETE | Clear all |

### 11. Data Loading Strategy

1. Notification counts load first.
2. Default tab (All) loads first.
3. Unread notifications load immediately.
4. Read notifications load on scroll.

### 12. User Actions

| Action | Target | Permission |
|--------|--------|------------|
| View notification | Click item | Any vendor |
| Mark as read | Click item | Any vendor |
| Mark all read | Button click | Any vendor |
| Clear all | Button click (with confirm) | Any vendor |
| Delete single | Swipe/button | Any vendor |

### 13. Form Specification

No forms. Confirmation modal for "Clear All":
`	sx
interface ClearAllConfirm {
  confirm: boolean;
}
`

### 14. Validation Rules

N/A -- Notification management.

### 15. State Machine -- Notification State

`
  +----------+  Read    +----------+
  | Unread   |--------->|  Read    |
  +----------+          +----+-----+
                          |        |
                     Archive   Delete
                          |        |
                     +----v----+ +----v----+
                     | Archived| | Deleted |
                     +---------+ +---------+
`

### 16. Error States

| Error | Display |
|-------|---------|
| API failure | Toast with retry |
| Mark read failure | Silent retry |
| Clear all failure | Toast with error |

### 17. Empty States

| Scenario | Display |
|----------|---------|
| No notifications | "لا توجد إشعارات" |
| No unread notifications | "جميع الإشعارات مقروءة ✓" |
| No filtered results | "لا توجد إشعارات في هذه الفئة" |

### 18. Loading States

- Notification list: 5 skeleton rows.
- Counts: Skeleton badges.

### 19. Success States

- Mark read: Item background changes, count decrements.
- Clear all: Toast "تم مسح كل الإشعارات" + empty state.

### 20. Permission-Based UI

| Permission | UI Impact |
|------------|-----------|
| Any authenticated vendor | Full notification view |

### 21. Security UX

- Notification content does not expose sensitive data.
- Action URLs are validated before navigation.

### 22. RTL/LTR Behavior

- Notification items flow right-to-left.
- Icons on right side in RTL.
- Timestamps on left side in RTL.

### 23. Internationalization

| Key | Arabic | English |
|-----|--------|---------|
| 
otifications.title | الإشعارات | Notifications |
| 
otifications.unread | غير مقروءة | Unread |
| 
otifications.markAllRead | تعيين الكل كمقروء | Mark All Read |
| 
otifications.clearAll | مسح الكل | Clear All |
| 
otifications.type.order | طلب | Order |
| 
otifications.type.review | تقييم | Review |
| 
otifications.type.stock | مخزون | Stock |
| 
otifications.type.kyc | تحقق | KYC |
| 
otifications.type.system | نظام | System |
| 
otifications.type.payout | سحب | Payout |
| 
otifications.empty | لا توجد إشعارات | No notifications |

### 24. Accessibility

- Notification items have ole="article" with ria-label.
- Unread indicator has ria-label="غير مقروء".
- Action links have descriptive ria-label.
- Keyboard: Arrow keys navigate, Enter opens.

### 25. Performance Requirements

| Metric | Target |
|--------|--------|
| Page load | < 1.0s |
| Mark read | < 100ms |
| Clear all | < 1.0s |

### 26. Analytics Events

| Event | Properties |
|-------|------------|
| 
otifications_view | { tab, unreadCount } |
| 
otification_click | { type, actionUrl } |
| 
otification_mark_read | { type } |
| 
otification_clear_all | { count } |

### 27. Notifications

| Trigger | Type |
|---------|------|
| Real-time notification | WebSocket push + badge update |

### 28. Cross-Page Relationships

`
VP-NT-001 <-- VP-DB-001 (Bell icon)
VP-NT-001 --> VP-OR-002 (Order notification)
VP-NT-001 --> VP-RV-001 (Review notification)
VP-NT-001 --> VP-KY-001 (KYC notification)
VP-NT-001 --> VP-DB-001 (Back)
`

### 29. Visual Preview

Key characteristics:
- Unread notifications have white background with green left border.
- Read notifications have gray background.
- Type-specific icons with distinct colors.
- Timestamps in relative format (e.g., "قبل 5 دقائق").

---

## VP-PR-004: Profile Settings

### 1. Page Metadata

| Field | Value |
|-------|-------|
| **ID** | VP-PR-004 |
| **Page Name** | Profile Settings |
| **Portal** | Vendor Panel |
| **Platform** | Desktop |
| **Module** | Settings |
| **Category** | Configuration |
| **Priority** | P1 |
| **Auth Required** | Yes |
| **Vendor Roles** | Vendor (all) |
| **Permissions** | Any authenticated vendor |
| **URL** | /vendor/settings/profile |
| **Parent Page** | VP-DB-001 |
| **Requirements References** | REQ-VN-SET-001 |
| **Status** | Draft |

### 2. Page Purpose

Manage vendor profile settings including personal information, password, notification preferences, language settings, and account security.

### 3. Entry Points

| Source | Trigger |
|--------|---------|
| Sidebar Navigation | Click "Settings" |
| VP-DB-001 | Settings shortcut |
| Header dropdown | Profile menu |

### 4. Exit Points

| Destination | Trigger |
|-------------|---------|
| VP-DB-001 | Back button |
| Login | Logout |

### 5. Information Architecture

`
Profile Settings
+-- Profile Section
|   +-- Profile Photo (upload)
|   +-- Full Name
|   +-- Email (read-only after verification)
|   +-- Phone Number
|   +-- Language Preference (AR/EN)
+-- Security Section
|   +-- Change Password
|   +-- Two-Factor Authentication
|   +-- Active Sessions
+-- Notification Preferences
|   +-- Email Notifications toggle
|   +-- Push Notifications toggle
|   +-- Order Notifications toggle
|   +-- Review Notifications toggle
|   +-- Stock Alert Notifications toggle
+-- Danger Zone
|   +-- Deactivate Account
|   +-- Delete Account
+-- Save Button
`

### 6. Layout Structure

`
+---------------------------------------------------------------------------+
|  إعدادات الملف الشخصي                                                      |
+---------------------------------------------------------------------------+
|                                                                           |
|  +--------------------------------------------------------------------+  |
|  |  الملف الشخصي                                                       |  |
|  |                                                                      |  |
|  |  صورة الملف الشخصي                                                  |  |
|  |  +-----------------+                                                |  |
|  |  |                 |                                                |  |
|  |  |   [رفع صورة]   |                                                |  |
|  |  |                 |                                                |  |
|  |  +-----------------+                                                |  |
|  |                                                                      |  |
|  |  الاسم الكامل *                                                      |  |
|  |  +--------------------------------------------------------------+  |  |
|  |  | محمد أحمد عبدالله                                             |  |  |
|  |  +--------------------------------------------------------------+  |  |
|  |                                                                      |  |
|  |  البريد الإلكتروني                                                   |  |
|  |  +--------------------------------------------------------------+  |  |
|  |  | mohammed@yemen-mart.com  (موثق)                              |  |  |
|  |  +--------------------------------------------------------------+  |  |
|  |                                                                      |  |
|  |  رقم الهاتف *                                                       |  |
|  |  +--------------------------------------------------------------+  |  |
|  |  | 77xxxxxxx                                                    |  |  |
|  |  +--------------------------------------------------------------+  |  |
|  |                                                                      |  |
|  |  اللغة المفضلة *                                                    |  |
|  |  ( ) العربية  ( ) English                                           |  |
|  +--------------------------------------------------------------------+  |
|                                                                           |
|  +--------------------------------------------------------------------+  |
|  |  الأمان                                                               |  |
|  |                                                                      |  |
|  |  تغيير كلمة المرور                                                   |  |
|  |  كلمة المرور الحالية *                                               |  |
|  |  +--------------------------------------------------------------+  |  |
|  |  | ••••••••                                                     |  |  |
|  |  +--------------------------------------------------------------+  |  |
|  |                                                                      |  |
|  |  كلمة المرور الجديدة *                                               |  |
|  |  +--------------------------------------------------------------+  |  |
|  |  | ••••••••                                                     |  |  |
|  |  +--------------------------------------------------------------+  |  |
|  |                                                                      |  |
|  |  تأكيد كلمة المرور الجديدة *                                         |  |
|  |  +--------------------------------------------------------------+  |  |
|  |  | ••••••••                                                     |  |  |
|  |  +--------------------------------------------------------------+  |  |
|  |                                                                      |  |
|  |  المصادقة الثنائية: معطلة  [تفعيل]                                 |  |
|  |                                                                      |  |
|  |  الجلسات النشطة: 2 جلسة نشطة  [إدارة]                            |  |
|  +--------------------------------------------------------------------+  |
|                                                                           |
|  +--------------------------------------------------------------------+  |
|  |  تفضيلات الإشعارات                                                   |  |
|  |                                                                      |  |
|  |  إشعارات البريد الإلكتروني    [═══●]  مفعّل                        |  |
|  |  إشعارات الدفع               [═══●]  مفعّل                        |  |
|  |  إشعارات الطلبات            [═══●]  مفعّل                        |  |
|  |  إشعارات التقييمات           [═══●]  مفعّل                        |  |
|  |  تنبيهات المخزون             [═══●]  مفعّل                        |  |
|  +--------------------------------------------------------------------+  |
|                                                                           |
|                      [ حفظ التغييرات ]                                    |
|                                                                           |
+---------------------------------------------------------------------------+
`

### 7. Responsive Design

| Breakpoint | Behavior |
|------------|----------|
| **Desktop (>=1024px)** | Full layout with sections. |
| **Tablet (768-1023px)** | Sections stack vertically. |
| **Mobile (<768px)** | Not supported. |

### 8. Widget Inventory

| Widget | ID | Type |
|--------|----|------|
| Profile Photo Upload | profile-photo | Image Upload |
| Name Input | profile-name | Text Input |
| Email Display | profile-email | Read-only Input |
| Phone Input | profile-phone | Text Input |
| Language Selector | profile-language | Radio Group |
| Password Change Form | profile-password | Form |
| 2FA Toggle | profile-2fa | Toggle |
| Active Sessions | profile-sessions | List |
| Notification Preferences | profile-notifications | Toggle Group |

### 9. Widget Specification -- Notification Preferences

`	sx
interface NotificationPreferencesProps {
  preferences: {
    email: boolean;
    push: boolean;
    orders: boolean;
    reviews: boolean;
    stockAlerts: boolean;
  };
  onChange: (prefs: NotificationPreferences) => void;
}
`

### 10. Data Requirements

| Endpoint | Method | Response |
|----------|--------|----------|
| GET /api/v1/vendors/me | GET | VendorProfile |
| PUT /api/v1/vendors/me | PUT | Update profile |
| POST /api/v1/vendors/me/photo | POST | Upload photo |
| PUT /api/v1/vendors/me/password | PUT | Change password |
| GET /api/v1/vendors/me/sessions | GET | Active sessions |
| DELETE /api/v1/vendors/me/sessions/:id | DELETE | Revoke session |
| PUT /api/v1/vendors/me/notifications | PUT | Update preferences |
| POST /api/v1/vendors/me/2fa/enable | POST | Enable 2FA |
| POST /api/v1/vendors/me/2fa/disable | POST | Disable 2FA |

### 11. Data Loading Strategy

- Profile data: Loaded on mount, form pre-filled.
- Sessions: Loaded in parallel.
- Photo upload: Immediate with progress.

### 12. User Actions

| Action | Target | Permission |
|--------|--------|------------|
| Update profile | Form submit | Any vendor |
| Change password | Password form | Any vendor |
| Upload photo | File picker | Any vendor |
| Toggle 2FA | Button click | Any vendor |
| Revoke session | Button click | Any vendor |
| Update preferences | Toggle change | Any vendor |
| Logout | Button click | Any vendor |

### 13. Form Specification

**Profile Form:**
`	sx
interface ProfileForm {
  fullName: string;      // Required, max 100 chars
  phone: string;         // Required, valid format
  language: 'ar' | 'en'; // Required
  photo?: File;          // Image, max 2MB
}
`

**Password Form:**
`	sx
interface PasswordForm {
  currentPassword: string;  // Required
  newPassword: string;      // Required, min 8 chars
  confirmPassword: string;  // Must match new password
}
`

### 14. Validation Rules

| Rule | Field | Message |
|------|-------|---------|
| Required field | ullName | "الاسم الكامل مطلوب" |
| Max length | ullName | "الاسم يجب ألا يتجاوز 100 حرف" |
| Required field | phone | "رقم الهاتف مطلوب" |
| Phone format | phone | "صيغة رقم الهاتف غير صالحة" |
| Required field | currentPassword | "كلمة المرور الحالية مطلوبة" |
| Min length | 
ewPassword | "كلمة المرور يجب أن تكون 8 أحرف على الأقل" |
| Password match | confirmPassword | "كلمتا المرور غير متطابقتين" |
| Photo size | photo | "الصورة يجب ألا تتجاوز 2 ميجابايت" |

### 15. State Machine

`
  +----------+  Edit    +----------+  Save    +----------+
  |  Pristine |--------->|  Dirty   |--------->| Saving  |
  +----------+           +----+-----+          +----+----+
                            |                       |
                       Auto-save              Success/Error
                            |                       |
                       +----v----+            +----v----+
                       | Saving  |            | Saved/  |
                       +----+----+            | Error   |
                            |                 +---------+
                       +----v----+
                       | Saved   |
                       +---------+
`

### 16. Error States

| Error | Display |
|-------|---------|
| Validation failure | Inline errors |
| Password mismatch | "كلمتا المرور غير متطابقتين" |
| Current password wrong | "كلمة المرور الحالية غير صحيحة" |
| API failure | Toast with retry |

### 17. Empty States

| Scenario | Display |
|----------|---------|
| No profile photo | Placeholder with upload CTA |
| No active sessions | "لا توجد جلسات نشطة أخرى" |

### 18. Loading States

- Profile: Full-page skeleton.
- Photo upload: Progress bar.
- Password change: Button spinner.

### 19. Success States

- Profile updated: Toast "تم تحديث الملف الشخصي"
- Password changed: Toast "تم تغيير كلمة المرور"
- Photo uploaded: Image preview updates
- 2FA enabled: Toast "تم تفعيل المصادقة الثنائية"

### 20. Permission-Based UI

| Permission | UI Impact |
|------------|-----------|
| Any authenticated vendor | Full profile management |

### 21. Security UX

- Email field is read-only after verification.
- Password form requires current password.
- 2FA toggle requires confirmation modal.
- Active sessions show device/IP info.

### 22. RTL/LTR Behavior

- Form labels right-aligned in RTL.
- Language selector affects entire panel.
- Profile photo centered regardless of direction.

### 23. Internationalization

| Key | Arabic | English |
|-----|--------|---------|
| profile.title | إعدادات الملف الشخصي | Profile Settings |
| profile.photo | صورة الملف الشخصي | Profile Photo |
| profile.fullName | الاسم الكامل | Full Name |
| profile.email | البريد الإلكتروني | Email |
| profile.phone | رقم الهاتف | Phone Number |
| profile.language | اللغة المفضلة | Preferred Language |
| profile.security | الأمان | Security |
| profile.changePassword | تغيير كلمة المرور | Change Password |
| profile.currentPassword | كلمة المرور الحالية | Current Password |
| profile.newPassword | كلمة المرور الجديدة | New Password |
| profile.confirmPassword | تأكيد كلمة المرور الجديدة | Confirm New Password |
| profile.twoFactor | المصادقة الثنائية | Two-Factor Authentication |
| profile.sessions | الجلسات النشطة | Active Sessions |
| profile.notifications | تفضيلات الإشعارات | Notification Preferences |
| profile.save | حفظ التغييرات | Save Changes |

### 24. Accessibility

- All form fields have associated labels.
- Toggle switches have ria-label and ria-checked.
- Password fields have ria-describedby for validation messages.
- Photo upload has ria-label with instructions.

### 25. Performance Requirements

| Metric | Target |
|--------|--------|
| Page load | < 1.0s |
| Photo upload | < 2s (2MB) |
| Password change | < 2.0s |
| Profile save | < 1.0s |

### 26. Analytics Events

| Event | Properties |
|-------|------------|
| profile_view | { section } |
| profile_update | { fieldsChanged } |
| profile_photo_upload | { fileSize } |
| profile_password_change | { success } |
| profile_2fa_toggle | { enabled } |
| profile_session_revoke | { sessionId } |

### 27. Notifications

| Trigger | Type |
|---------|------|
| Profile updated | Toast |
| Password changed | Email confirmation |
| 2FA enabled/disabled | Email confirmation |
| New login from unknown device | Email alert |

### 28. Cross-Page Relationships

`
VP-PR-004 <-- VP-DB-001 (Settings shortcut)
VP-PR-004 --> VP-DB-001 (Back)
VP-PR-004 --> Login (Logout)
`

### 29. Visual Preview

Key characteristics:
- Clean form layout with sections.
- Profile photo has circular preview with hover overlay.
- Toggle switches use green for enabled, gray for disabled.
- Security section has subtle red accent for danger zone.
- Save button is sticky at bottom.

---

## Appendix A: State Machines Summary

### Product Lifecycle

`
Draft --> Active --> Out of Stock --> Active
  |                |
Archived <-- Archived
`

### Order Processing

`
Pending --> Processing --> Shipped --> Delivered --> Completed
   |           |
Cancelled   Cancelled
`

### KYC Verification

`
Draft --> Pending --> Approved
            |
        Rejected --> Draft (resubmit)
`

### Payout Lifecycle

`
Request --> Pending --> Processing --> Completed
             |
         Rejected
`

### Inventory States

`
In Stock --> Low Stock --> Out of Stock --> In Stock
`

---

## Appendix B: Permission Matrix (Detailed)

| Permission | Pages | Actions |
|------------|-------|---------|
| products:create | VP-PR-001, VP-PR-002 | Create products, duplicate |
| products:read | VP-PR-001, VP-PR-003 | View products, list |
| products:update | VP-PR-001, VP-PR-002, VP-PR-003 | Edit, toggle status, archive |
| orders:read | VP-OR-001, VP-OR-002 | View orders, list |
| orders:update | VP-OR-001, VP-OR-002 | Update status, add notes |
| inventory:read | VP-IV-001 | View inventory |
| inventory:update | VP-IV-001 | Edit stock, bulk update |
| store:read | VP-ST-001, VP-ST-002 | View store settings |
| store:update | VP-ST-001, VP-ST-002 | Edit settings, change template |
| inance:read | VP-FN-001, VP-FN-002, VP-FN-003 | View finances, transactions, payouts |
| eviews:read | VP-RV-001 | View reviews, respond |

---

## Appendix C: API Endpoints Summary

| Endpoint | Method | Used By |
|----------|--------|---------|
| /api/v1/vendors/dashboard | GET | VP-DB-001 |
| /api/v1/vendors/dashboard/recent-orders | GET | VP-DB-001 |
| /api/v1/vendors/dashboard/low-stock | GET | VP-DB-001 |
| /api/v1/vendors/dashboard/revenue | GET | VP-DB-001 |
| /api/v1/vendors/dashboard/reviews-summary | GET | VP-DB-001 |
| /api/v1/vendors/me | GET | VP-DB-001, VP-PR-004 |
| /api/v1/vendors/products | GET/POST | VP-PR-001, VP-PR-002 |
| /api/v1/vendors/products/:id | GET/PUT/DELETE | VP-PR-002, VP-PR-003 |
| /api/v1/vendors/products/:id/images | POST/DELETE | VP-PR-002 |
| /api/v1/vendors/categories | GET | VP-PR-001, VP-PR-002 |
| /api/v1/vendors/inventory | GET | VP-IV-001 |
| /api/v1/vendors/inventory/summary | GET | VP-IV-001 |
| /api/v1/vendors/inventory/:productId | PUT | VP-IV-001 |
| /api/v1/vendors/inventory/bulk | PATCH | VP-IV-001 |
| /api/v1/vendors/inventory/:productId/history | GET | VP-IV-001 |
| /api/v1/vendors/inventory/import | POST | VP-IV-001 |
| /api/v1/vendors/inventory/export | GET | VP-IV-001 |
| /api/v1/vendors/orders | GET | VP-OR-001 |
| /api/v1/vendors/orders/counts | GET | VP-OR-001 |
| /api/v1/vendors/orders/:id | GET | VP-OR-002 |
| /api/v1/vendors/orders/:id/status | PATCH | VP-OR-001, VP-OR-002 |
| /api/v1/vendors/orders/:id/notes | POST | VP-OR-002 |
| /api/v1/vendors/store | GET/PUT | VP-ST-001 |
| /api/v1/vendors/store/logo | POST | VP-ST-001 |
| /api/v1/vendors/store/banner | POST | VP-ST-001 |
| /api/v1/vendors/store/template | PUT | VP-ST-002 |
| /api/v1/vendors/templates | GET | VP-ST-002 |
| /api/v1/vendors/finance/summary | GET | VP-FN-001 |
| /api/v1/vendors/finance/earnings | GET | VP-FN-001 |
| /api/v1/vendors/finance/commissions | GET | VP-FN-001 |
| /api/v1/vendors/finance/transactions | GET | VP-FN-002 |
| /api/v1/vendors/finance/payout-summary | GET | VP-FN-001 |
| /api/v1/vendors/finance/export | GET | VP-FN-001 |
| /api/v1/vendors/payouts | GET | VP-FN-003 |
| /api/v1/vendors/payouts/summary | GET | VP-FN-003 |
| /api/v1/vendors/payouts/request | POST | VP-FN-003 |
| /api/v1/vendors/reviews | GET | VP-RV-001 |
| /api/v1/vendors/reviews/summary | GET | VP-RV-001 |
| /api/v1/vendors/reviews/:id/respond | POST | VP-RV-001 |
| /api/v1/vendors/kyc | GET | VP-KY-001 |
| /api/v1/vendors/kyc/personal | POST | VP-KY-001 |
| /api/v1/vendors/kyc/business | POST | VP-KY-001 |
| /api/v1/vendors/kyc/documents | POST | VP-KY-001 |
| /api/v1/vendors/kyc/submit | POST | VP-KY-001 |
| /api/v1/vendors/analytics/summary | GET | VP-AN-001 |
| /api/v1/vendors/analytics/sales-trends | GET | VP-AN-001 |
| /api/v1/vendors/analytics/top-products | GET | VP-AN-001 |
| /api/v1/vendors/analytics/customer-insights | GET | VP-AN-001 |
| /api/v1/vendors/analytics/conversion-funnel | GET | VP-AN-001 |
| /api/v1/vendors/notifications | GET | VP-NT-001 |
| /api/v1/vendors/notifications/counts | GET | VP-NT-001 |
| /api/v1/vendors/notifications/:id/read | PATCH | VP-NT-001 |
| /api/v1/vendors/notifications/read-all | PATCH | VP-NT-001 |
| /api/v1/vendors/me/photo | POST | VP-PR-004 |
| /api/v1/vendors/me/password | PUT | VP-PR-004 |
| /api/v1/vendors/me/sessions | GET | VP-PR-004 |
| /api/v1/vendors/me/notifications | PUT | VP-PR-004 |
| /api/v1/vendors/me/2fa/enable | POST | VP-PR-004 |
| /api/v1/vendors/me/2fa/disable | POST | VP-PR-004 |

---

**End of Document**
