# 05 - PORTAL: ADMIN PANEL — UI/UX Specification

**Version:** 1.0.0
**Date:** 2026-09-13
**Status:** Active
**Portal:** Admin Panel (Web — Desktop-First)
**Tech Stack:** React 18, Vite, Tailwind CSS 4, TypeScript
**Locale:** Arabic-first (RTL), English (LTR) secondary
**Target Viewport:** 1280px — 2560px (Desktop-first)

---

## Table of Contents

1. [Global Admin Panel Conventions](#1-global-admin-panel-conventions)
2. [AP-DB-001 — Admin Dashboard](#2-ap-db-001--admin-dashboard)
3. [AP-US-001 — User Management](#3-ap-us-001--user-management)
4. [AP-US-002 — User Detail](#4-ap-us-002--user-detail)
5. [AP-VN-001 — Vendor Management](#5-ap-vn-001--vendor-management)
6. [AP-VN-002 — KYC Review Queue](#6-ap-vn-002--kyc-review-queue)
7. [AP-VN-003 — KYC Detail Review](#7-ap-vn-003--kyc-detail-review)
8. [AP-PR-001 — Product Management](#8-ap-pr-001--product-management)
9. [AP-PR-002 — Product Moderation](#9-ap-pr-002--product-moderation)
10. [AP-OR-001 — Order Management](#10-ap-or-001--order-management)
11. [AP-OR-002 — Order Detail](#11-ap-or-002--order-detail)
12. [AP-FN-001 — Financial Dashboard](#12-ap-fn-001--financial-dashboard)
13. [AP-FN-002 — Commission Management](#13-ap-fn-002--commission-management)
14. [AP-FN-003 — Escrow Management](#14-ap-fn-003--escrow-management)
15. [AP-FN-004 — Payout Management](#15-ap-fn-004--payout-management)
16. [AP-FN-005 — VAT Management](#16-ap-fn-005--vat-management)
17. [AP-CM-001 — Content Management](#17-ap-cm-001--content-management)
18. [AP-CM-002 — Category Management](#18-ap-cm-002--category-management)
19. [AP-CM-003 — Coupon Management](#19-ap-cm-003--coupon-management)
20. [AP-DL-001 — Delivery Provider Management](#20-ap-dl-001--delivery-provider-management)
21. [AP-AN-001 — Analytics Dashboard](#21-ap-an-001--analytics-dashboard)
22. [AP-AN-002 — Reports](#22-ap-an-002--reports)
23. [AP-SY-001 — System Configuration](#23-ap-sy-001--system-configuration)
24. [AP-SY-002 — Audit Log Viewer](#24-ap-sy-002--audit-log-viewer)
25. [AP-SU-001 — Support Ticket Management](#25-ap-su-001--support-ticket-management)
26. [AP-SU-002 — Ticket Detail](#26-ap-su-002--ticket-detail)
27. [AP-NT-001 — Notification Management](#27-ap-nt-001--notification-management)
28. [AP-PR-003 — Admin Profile](#28-ap-pr-003--admin-profile)

---

## 1. Global Admin Panel Conventions

### 1.1 Navigation Architecture

```
+------------------------------------------------------------------+
|  Top Bar                                                         |
|  [Logo/Brand] [Search (Global)]        [Notifications] [Avatar]  |
+------------+-----------------------------------------------------+
|            |                                                     |
|  Sidebar   |  Main Content Area                                 |
|            |                                                     |
|  Dashboard |  +-----------------------------------------------+  |
|  Users     |  |  Breadcrumb                                   |  |
|  Vendors   |  +-----------------------------------------------+  |
|  Products  |  |                                               |  |
|  Orders    |  |  Page Content                                 |  |
|  Finance   |  |                                               |  |
|  Content   |  |                                               |  |
|  Delivery  |  |                                               |  |
|  Analytics |  +-----------------------------------------------+  |
|  System    |  |  Pagination / Footer                          |  |
|  Support   |  +-----------------------------------------------+  |
|  Settings  |                                                     |
+------------+-----------------------------------------------------+
```

### 1.2 Sidebar Navigation Items

| # | Label (AR) | Label (EN) | Route | Icon | Badge | Permission Gate |
|---|-----------|------------|-------|------|-------|-----------------|
| 1 | لوحة التحكم | Dashboard | `/admin/dashboard` | home | — | any authenticated |
| 2 | إدارة المستخدمين | Users | `/admin/users` | users | count:pending | `users:read` |
| 3 | إدارة البائعين | Vendors | `/admin/vendors` | store | count:pending KYC | `vendors:read` |
| 4 | إدارة المنتجات | Products | `/admin/products` | box | count:moderation | `products:read` |
| 5 | الطلبات | Orders | `/admin/orders` | cart | count:today | `orders:read` |
| 6 | المالية | Finance | `/admin/finance` | dollar-sign | — | `finance:read` |
| 7 | إدارة المحتوى | Content | `/admin/content` | file-text | — | super_admin only |
| 8 | إدارة التوصيل | Delivery | `/admin/delivery` | truck | — | super_admin only |
| 9 | التحليلات | Analytics | `/admin/analytics` | bar-chart-2 | — | `reports:read` |
| 10 | النظام | System | `/admin/system` | settings | — | super_admin only |
| 11 | الدعم الفني | Support | `/admin/support` | life-buoy | count:open | `support:read` |
| 12 | الإشعارات | Notifications | `/admin/notifications` | bell | count:unread | any authenticated |
| 13 | الملف الشخصي | Profile | `/admin/profile` | user | — | any authenticated |

### 1.3 Permission Matrix

| Role | users | vendors | products | orders | finance | reports | support | system | content | delivery |
|------|-------|---------|----------|--------|---------|---------|---------|--------|---------|----------|
| Super Admin | CRUD | CRUD+KYC | CRUD | RU | CRUD | R | CRUD | CRUD | CRUD | CRUD |
| Admin (users:read, users:update) | RU | — | — | — | — | — | — | — | — | — |
| Admin (vendors:read, vendors:update, vendors:approve_kyc) | — | RU+KYC | — | — | — | — | — | — | — | — |
| Admin (products:read, products:update, products:delete) | — | — | RUD | — | — | — | — | — | — | — |
| Admin (orders:read, orders:update) | — | — | — | RU | — | — | — | — | — | — |
| Admin (finance:read) | — | — | — | — | R | — | — | — | — | — |
| Admin (reports:read) | — | — | — | — | — | R | — | — | — | — |
| Admin (support:read, support:update) | — | — | — | — | — | — | RU | — | — | — |

### 1.4 Global Design Tokens (Admin Panel)

```css
/* Tailwind CSS 4 — Admin Panel Custom Theme */
@theme {
  --color-primary: #1B5E20;
  --color-primary-light: #4CAF50;
  --color-primary-dark: #0D3B12;
  --color-secondary: #FF8F00;
  --color-danger: #D32F2F;
  --color-warning: #F57C00;
  --color-success: #388E3C;
  --color-info: #1976D2;
  --color-surface: #FFFFFF;
  --color-surface-alt: #F5F5F5;
  --color-sidebar: #1A1A2E;
  --color-sidebar-text: #E0E0E0;
  --color-sidebar-active: #4CAF50;
  --color-text-primary: #212121;
  --color-text-secondary: #757575;
  --color-border: #E0E0E0;
  --color-border-light: #EEEEEE;
  --font-family-ar: 'Noto Kufi Arabic', 'Cairo', sans-serif;
  --font-family-en: 'Inter', sans-serif;
  --font-family-mono: 'JetBrains Mono', monospace;
  --radius-sm: 4px;
  --radius-md: 8px;
  --radius-lg: 12px;
  --radius-xl: 16px;
  --radius-full: 9999px;
  --shadow-sm: 0 1px 2px rgba(0,0,0,0.05);
  --shadow-md: 0 4px 6px rgba(0,0,0,0.07);
  --shadow-lg: 0 10px 15px rgba(0,0,0,0.10);
  --shadow-xl: 0 20px 25px rgba(0,0,0,0.12);
  --z-sidebar: 40;
  --z-topbar: 50;
  --z-dropdown: 60;
  --z-modal: 70;
  --z-toast: 80;
  --z-tooltip: 90;
}
```

### 1.5 RTL Layout Rules

- All layout uses CSS logical properties (`ms-`/`me-` instead of `ml-`/`mr-`)
- Sidebar always on the **right** side in RTL mode
- Text alignment: `text-start` / `text-end` (not `text-left` / `text-right`)
- Icons that imply direction (arrows, chevrons) are mirrored in RTL
- Data tables: first column anchored to the **right** in RTL
- All numerical values displayed in Western Arabic numerals (1,2,3)
- Date format: `YYYY/MM/DD` (e.g., 2026/09/13)

### 1.6 Component Library (Shared)

| Component | Description | Variants |
|-----------|-------------|----------|
| DataTable | Sortable, filterable table with pagination | default, compact, striped |
| StatCard | KPI display with icon, value, delta | default, gradient, outlined |
| FilterBar | Horizontal filter controls with chips | inline, collapsible |
| Modal | Dialog overlay | default, confirm, form |
| Drawer | Slide-in panel (right in RTL) | default, wide, full-height |
| Toast | Non-blocking notification | success, error, warning, info |
| Badge | Status indicator pill | colored, dot, count |
| Breadcrumb | Navigation trail | default, collapsed |
| Tabs | Content tab switcher | underline, enclosed, pills |
| Dropdown | Menu overlay | default, multi-select |
| DatePicker | Calendar input | single, range, datetime |
| SearchInput | Text input with icon + clear | default, with-filters |
| Toggle | Boolean switch | default, labeled |
| Pagination | Page navigation | default, compact, numbered |
| EmptyState | Placeholder when no data | illustration + CTA |
| SkeletonLoader | Loading placeholder | text, card, table, chart |
| StatusBadge | Colored badge mapped to enum states | per-entity color scheme |
| ConfirmDialog | Destructive action confirmation | default, destructive |
| FileUpload | Drag-and-drop or click to upload | image, document, multi |

### 1.7 17 Order States — Status Badge Colors

| # | State (AR) | State (EN) | Color | Hex |
|---|-----------|------------|-------|-----|
| 1 | بانتظار الدفع | Pending Payment | gray | #9E9E9E |
| 2 | بانتظار التأكيد | Pending Confirmation | amber | #FF8F00 |
| 3 | قيد المعالجة | Processing | blue | #1976D2 |
| 4 | قيد التجهيز | Preparing | indigo | #3F51B5 |
| 5 | جاهز للتوصيل | Ready for Pickup | teal | #00897B |
| 6 | قيد التوصيل | Out for Delivery | purple | #7B1FA2 |
| 7 | تم التوصيل | Delivered | green | #388E3C |
| 8 | تم التسليم | Completed | green-600 | #2E7D32 |
| 9 | بانتظار الاستلام | Awaiting Pickup | cyan | #00ACC1 |
| 10 | ملغي من البائع | Cancelled by Vendor | red-300 | #EF9A9A |
| 11 | ملغي من العميل | Cancelled by Customer | red-400 | #EF5350 |
| 12 | ملغي من الإدارة | Cancelled by Admin | red-600 | #D32F2F |
| 13 | قيد الاسترجاع | Refund Pending | orange | #F57C00 |
| 14 | تمت الاسترجاع | Refunded | green-400 | #66BB6A |
| 15 | مرتجع جزئي | Partially Refunded | lime | #9CCC65 |
| 16 | نزاع مفتوح | Dispute Open | red-700 | #C62828 |
| 17 | مكتمل مع ملاحظات | Completed with Notes | green-800 | #1B5E20 |

---

## 2. AP-DB-001 — Admin Dashboard

### 2.1 Metadata

| Field | Value |
|-------|-------|
| Page ID | `AP-DB-001` |
| Page Title (AR) | لوحة التحكم |
| Page Title (EN) | Admin Dashboard |
| Route | `/admin/dashboard` |
| Required Role | Any authenticated admin |
| Required Permission | — (all authenticated admins) |
| Frequency | Primary entry — visited on every login |
| Last Updated | 2026-09-13 |

### 2.2 Purpose

Provide administrators with an at-a-glance overview of platform health including key business metrics, operational alerts, pending actions, and real-time activity. Serve as the operational nerve center.

### 2.3 Entry / Exit Points

| Direction | Source / Destination | Trigger |
|-----------|---------------------|---------|
| Entry | Login redirect | Post-authentication |
| Entry | Sidebar "Dashboard" click | Navigation |
| Entry | Any admin page "Logo" click | Brand logo in top bar |
| Exit | Click stat card drill-down | Navigate to relevant list/detail |
| Exit | Click alert/notification | Navigate to context page |
| Exit | Sidebar navigation | Navigate to other pages |

### 2.4 Information Architecture

```
AP-DB-001: Admin Dashboard
+-- Top Stats Row (4 cards)
|   +-- Revenue Today (YER + USD)
|   +-- Orders Today (count + delta)
|   +-- Active Vendors (count + delta)
|   +-- Active Users (count + delta)
+-- Quick Actions Bar
|   +-- Pending KYC Reviews (badge + link)
|   +-- Pending Payouts (badge + link)
|   +-- Open Disputes (badge + link)
|   +-- Product Moderation Queue (badge + link)
+-- Charts Section (2-column)
|   +-- Revenue Trend (7-day / 30-day line chart)
|   +-- Order Volume by Status (donut chart)
+-- Recent Activity Feed (scrollable list)
+-- Top Vendors Table (top 5 by revenue)
+-- Top Products Table (top 5 by sales)
+-- System Health Mini-Panel
    +-- API Response Time
    +-- Error Rate (24h)
    +-- Active Sessions
```

### 2.5 Layout

```
+------------------------------------------------------------------+
|  Breadcrumb: الرئيسية / لوحة التحكم                               |
+------------------------------------------------------------------+
|                                                                  |
|  +----------+  +----------+  +----------+  +----------+          |
|  | إيرادات  |  |  طلبات   |  | بائعون    |  | مستخدمون |          |
|  | اليوم    |  |  اليوم   |  | نشطون    |  | نشطون    |          |
|  | 125,000  |  |    87    |  |   234    |  |  1,245   |          |
|  | ^ 12.5%  |  | ^ 8.3%   |  | ^ 3.1%   |  | v 1.2%   |          |
|  +----------+  +----------+  +----------+  +----------+          |
|                                                                  |
|  +-------------------------------------------------------------+ |
|  | إجراءات سريعة: [KYC معلق: 5] [مدفوعات: 3] [نزاعات: 2]    | |
|  +-------------------------------------------------------------+ |
|                                                                  |
|  +---------------------------+  +---------------------------+   |
|  |  نمو الإيرادات            |  |  توزيع الطلبات بالحالات   |   |
|  |  (line chart)             |  |  (donut chart)            |   |
|  +---------------------------+  +---------------------------+   |
|                                                                  |
|  +---------------------------+  +---------------------------+   |
|  |  آخر النشاطات             |  |  أفضل البائعين            |   |
|  |  (feed list)              |  |  (table: top 5)           |   |
|  +---------------------------+  +---------------------------+   |
|                                                                  |
|  +---------------------------+  +---------------------------+   |
|  |  أفضل المنتجات            |  |  صحة النظام                |   |
|  |  (table: top 5)           |  |  (mini metrics)           |   |
|  +---------------------------+  +---------------------------+   |
|                                                                  |
+------------------------------------------------------------------+
```

### 2.6 Widgets

#### 2.6.1 Stat Card (reusable)

| Property | Specification |
|----------|---------------|
| Icon | Top-left, 24x24, muted bg circle |
| Label | Top-right, 14px, text-secondary |
| Value | Center, 28px bold, text-primary |
| Delta | Bottom, 12px, green=up (good) / red=down (bad) |
| Background | White, shadow-sm, radius-lg |
| Interaction | Clickable — navigates to detail page |
| Hover | Shadow-md lift + subtle border color change |
| Loading | Skeleton: 3 lines, pulsing |

#### 2.6.2 Quick Action Badge

| Property | Specification |
|----------|---------------|
| Display | Pill badge with count + label text |
| Background | Contextual color (amber for pending, red for urgent) |
| Click | Navigates to filtered queue page |
| Max visible | 4 items; overflow → "المزيد" link |

#### 2.6.3 Revenue Trend Chart

| Property | Specification |
|----------|---------------|
| Type | Line chart (smooth curves) |
| X-axis | Time (days/weeks depending on range) |
| Y-axis | Revenue (YER primary, USD secondary toggle) |
| Controls | Toggle: 7 أيام / 30 يوم / 90 يوم |
| Tooltip | On hover: date + exact value |
| Grid | Horizontal dashed lines, light gray |
| Library | Recharts or Chart.js |

#### 2.6.4 Order Status Donut Chart

| Property | Specification |
|----------|---------------|
| Type | Donut chart |
| Segments | Each of 17 order states as a segment |
| Center | Total orders count + label |
| Legend | Scrollable list below chart, color + label + count |
| Click segment | Navigate to Order Management filtered by that state |

#### 2.6.5 Recent Activity Feed

| Property | Specification |
|----------|---------------|
| Items | Last 20 activities |
| Format | [Icon] [Description text] [Relative time] |
| Icon color | Green=order, Blue=user, Amber=vendor, Red=dispute |
| Click | Navigate to relevant entity detail |
| Auto-refresh | Every 60 seconds via WebSocket polling |
| Empty state | "لا توجد نشاطات حديثة" with clock illustration |

### 2.7 Data Requirements

| Data Point | Source API | Refresh | Cache |
|------------|-----------|---------|-------|
| Revenue Today | `GET /admin/stats/revenue` | 60s | 30s |
| Orders Today | `GET /admin/stats/orders` | 60s | 30s |
| Active Vendors | `GET /admin/stats/vendors` | 300s | 120s |
| Active Users | `GET /admin/stats/users` | 300s | 120s |
| Pending KYC Count | `GET /admin/stats/pending-kyc` | 120s | 60s |
| Pending Payouts Count | `GET /admin/stats/pending-payouts` | 120s | 60s |
| Open Disputes Count | `GET /admin/stats/disputes` | 120s | 60s |
| Moderation Queue Count | `GET /admin/stats/moderation` | 120s | 60s |
| Revenue Trend | `GET /admin/charts/revenue` | 300s | 120s |
| Order Status Distribution | `GET /admin/charts/orders` | 300s | 120s |
| Recent Activity | `GET /admin/activity` | 60s (WS) | 0 |
| Top Vendors | `GET /admin/stats/top-vendors` | 600s | 300s |
| Top Products | `GET /admin/stats/top-products` | 600s | 300s |
| System Health | `GET /admin/system/health` | 30s | 0 |

### 2.8 Actions

| Action | Trigger | Behavior | Permission |
|--------|---------|----------|------------|
| Refresh Dashboard | Button / pull | Re-fetch all dashboard data | any auth |
| Toggle Currency | Toggle control | Switch YER ↔ USD on revenue cards | any auth |
| Change Time Range | Dropdown | Update charts to 7d/30d/90d | any auth |
| Click Stat Card | Click | Navigate to filtered list page | any auth |
| Click Quick Action | Click | Navigate to relevant queue page | any auth |
| Click Activity Item | Click | Navigate to entity detail | any auth |
| Dismiss Alert | Click X | Mark alert as read, remove from feed | any auth |

### 2.9 States

| State | Description | UI |
|-------|-------------|-----|
| Loading | Initial page load | Skeleton grid (12 blocks) |
| Loaded | Data successfully fetched | Full dashboard |
| Partial Error | One or more API calls failed | Show available data, error toast per failed widget |
| Empty | Platform has no data yet | Empty state illustration + setup CTA |
| Offline | Network disconnected | Banner + last-cached data |

### 2.10 Error Handling

| Error Type | Display | Action |
|------------|---------|--------|
| API 500 | Toast: "خطأ في الخادم" | Retry button in toast |
| API 401 | Redirect to login | Session expired |
| API 403 | Toast: "ليس لديك صلاحية" | Hide restricted widgets |
| Network error | Top banner: "لا يوجد اتصال" | Auto-retry with backoff |
| Chart data error | Widget-level fallback: "تعذر تحميل البيانات" | Retry per-widget |

### 2.11 Empty States

| Scenario | Illustration | Title (AR) | Action Button |
|----------|-------------|------------|---------------|
| No data at all | Chart illustration | "مرحباً بك في لوحة التحكم" | "إضافة أول بائع" |
| No recent activity | Clock/empty list | "لا توجد نشاطات حديثة" | — |
| No revenue today | Wallet illustration | "لا توجد إيرادات اليوم" | — |

### 2.12 Loading States

- **Full page skeleton**: 12-block grid matching dashboard layout, pulsing animation
- **Chart skeleton**: Gray rectangle with wave animation
- **Table skeleton**: 5 rows × 4 columns shimmer
- **Feed skeleton**: 5 items × icon + 2 lines shimmer

### 2.13 Permissions & Visibility

- All stat cards visible to any authenticated admin
- Quick actions respect permission gates (e.g., KYC badge only visible if `vendors:approve_kyc`)
- System Health panel visible only to Super Admin
- Revenue details hidden from admins without `finance:read`
- Activity feed filtered by admin's permission scope

### 2.14 RTL Considerations

- All stat cards flow RTL: icon on right, label on left
- Delta arrows: ↑ on left, percentage on right
- Quick actions bar: items flow RTL
- Charts: X-axis labels right-to-left
- Activity feed: icon on right, text on left, time on far left
- Tables: name column on rightmost position

### 2.15 Accessibility

| Requirement | Implementation |
|-------------|----------------|
| Keyboard navigation | Tab order: stat cards → quick actions → charts → feed → tables |
| Screen reader | All cards have `aria-label` with full context |
| Charts | Data provided as accessible table fallback |
| Color contrast | WCAG AA (4.5:1 for text, 3:1 for large text) |
| Focus indicators | Visible ring on all interactive elements |
| Reduced motion | Disable chart animations, skeleton pulse |
| Live regions | Stat updates announced via `aria-live="polite"` |
| Language | `lang="ar"` on document, RTL `dir="rtl"` |

---

## 3. AP-US-001 — User Management

### 3.1 Metadata

| Field | Value |
|-------|-------|
| Page ID | `AP-US-001` |
| Page Title (AR) | إدارة المستخدمين |
| Page Title (EN) | User Management |
| Route | `/admin/users` |
| Required Role | Admin |
| Required Permission | `users:read` |
| Parent | Sidebar → إدارة المستخدمين |
| Children | AP-US-002 (User Detail) |
| Last Updated | 2026-09-13 |

### 3.2 Purpose

Allow administrators to view, search, filter, and manage all registered users (buyers) on the platform. Provide quick access to user profiles, order history, and wallet status.

### 3.3 Entry / Exit Points

| Direction | Source / Destination | Trigger |
|-----------|---------------------|---------|
| Entry | Sidebar "إدارة المستخدمين" | Navigation |
| Entry | AP-DB-001 stat card click | Active Users drill-down |
| Entry | Global search result | Click user result |
| Exit | Click user row | Navigate to AP-US-002 |
| Exit | Export button | Download CSV/Excel |
| Exit | Sidebar navigation | Navigate to other pages |

### 3.4 Information Architecture

```
AP-US-001: User Management
+-- Page Header
|   +-- Title: إدارة المستخدمين
|   +-- Stats Bar (inline): Total | Active | Suspended | Banned
|   +-- Actions: [Export] [Bulk Actions]
+-- Filter Bar
|   +-- Search (name, email, phone)
|   +-- Status filter (multi-select)
|   +-- Registration date range
|   +-- Has orders (toggle)
|   +-- Has wallet balance > 0 (toggle)
|   +-- Saved filters dropdown
+-- Data Table
|   +-- Columns: Avatar | Name | Phone | Email | Orders | Wallet | Status | Joined | Actions
|   +-- Sortable columns: Name, Orders, Wallet, Joined
|   +-- Row click → AP-US-002
|   +-- Bulk selection checkboxes
+-- Bulk Action Bar (appears on selection)
|   +-- Export Selected
|   +-- Send Notification
|   +-- Suspend
|   +-- Ban
+-- Pagination
    +-- Page size: 25 / 50 / 100
    +-- Total count + page navigation
```

### 3.5 Data Table Columns

| # | Column (AR) | Column (EN) | Field | Width | Sortable | Filterable | Format |
|---|------------|-------------|-------|-------|----------|------------|--------|
| 1 | — | checkbox | _select | 40px | — | — | checkbox |
| 2 | الصورة | Avatar | avatar | 48px | — | — | circular image |
| 3 | الاسم | Name | name | flex | yes | no | text, clickable |
| 4 | الهاتف | Phone | phone | 140px | yes | no | +967 XXX XXX XXX |
| 5 | البريد | Email | email | 180px | yes | no | text |
| 6 | الطلبات | Orders | orderCount | 80px | yes | yes | integer |
| 7 | المحفظة | Wallet Balance | walletBalance | 100px | yes | yes | YER formatted |
| 8 | الحالة | Status | status | 100px | yes | yes (multi) | StatusBadge |
| 9 | تاريخ الانضمام | Joined | createdAt | 120px | yes | yes (range) | YYYY/MM/DD |
| 10 | إجراءات | Actions | _actions | 80px | — | — | icon buttons |

### 3.6 Status Badges

| Status (AR) | Status (EN) | Color | Hex |
|-------------|-------------|-------|-----|
| نشط | Active | green | #388E3C |
| معلّق | Suspended | amber | #FF8F00 |
| محظور | Banned | red | #D32F2F |
| غير مؤكد | Unverified | gray | #9E9E9E |

### 3.7 Filters

| Filter | Type | Options / Behavior |
|--------|------|-------------------|
| Search | Text input | Debounced 300ms, searches name + email + phone |
| Status | Multi-select | checkboxes: Active, Suspended, Banned, Unverified |
| Registration Date | Date range | From/To date pickers |
| Has Orders | Toggle | Yes / No / Any |
| Wallet Balance > 0 | Toggle | Yes / No / Any |
| Saved Filters | Dropdown | Pre-saved filter combos by admin |

### 3.8 Actions

| Action | Trigger | Behavior | Permission |
|--------|---------|----------|------------|
| View User | Row click / icon | Navigate to AP-US-002 | `users:read` |
| Edit User | Row action icon | Open edit drawer | `users:update` |
| Suspend User | Row action icon | Confirm dialog → suspend | `users:update` |
| Reinstate User | Row action icon | Confirm dialog → reinstate | `users:update` |
| Ban User | Row action icon | Confirm dialog (destructive) → ban | `users:update` |
| Send Notification | Row action / bulk | Compose notification modal | `users:update` |
| Export | Header button | Download filtered CSV | `users:read` |
| Bulk Suspend | Bulk action bar | Confirm dialog → suspend selected | `users:update` |
| Bulk Ban | Bulk action bar | Confirm dialog (destructive) → ban | `users:update` |

### 3.9 Forms — Edit User Drawer

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| Name | Text input | yes | 2-100 chars, no special chars |
| Email | Email input | yes | Valid email format |
| Phone | Tel input | yes | +967XXXXXXXXX format |
| Status | Select | yes | Active / Suspended / Banned |
| Notes | Textarea | no | Max 500 chars (admin internal note) |

### 3.10 Validation Rules

| Field | Rule | Error Message (AR) |
|-------|------|-------------------|
| Name | Required, 2-100 chars | "الاسم مطلوب (2-100 حرف)" |
| Email | Required, valid email format | "البريد الإلكتروني غير صحيح" |
| Phone | Required, +967XXXXXXXXX | "رقم الهاتف غير صحيح" |
| Status | Required, valid enum value | "الحالة مطلوبة" |

### 3.11 States

| State | Description | UI |
|-------|-------------|-----|
| Loading | Fetching user list | Table skeleton (10 rows) |
| Loaded | Data present | Full table |
| Empty | No users match filters | EmptyState: "لا يوجد مستخدمون يطابقون الفلاتر" |
| Error | API failure | ErrorState + retry button |
| No Results | Search returns nothing | EmptyState: "لا توجد نتائج للبحث" |

### 3.12 Empty States

| Scenario | Title (AR) | Action |
|----------|-----------|--------|
| No users at all | "لا يوجد مستخدمون بعد" | — |
| Filter returns empty | "لا يوجد مستخدمون يطابقون الفلاتر" | "مسح الفلاتر" button |
| Search returns empty | "لا توجد نتائج للبحث" | Clear search button |

### 3.13 Permissions & Visibility

- Table visible only with `users:read`
- Edit/Suspend/Ban buttons hidden without `users:update`
- Wallet column visible only to admins with `finance:read`
- Bulk actions require `users:update`
- Super Admin sees all users without scope restriction

### 3.14 RTL Considerations

- Table columns flow RTL: avatar on rightmost, actions on leftmost
- Name column right-aligned
- Phone numbers displayed LTR (numbers are LTR even in RTL context)
- Filter bar flows RTL
- Pagination: "السابق" on right, "التالي" on left

### 3.15 Accessibility

- Table has proper `<th>` with `scope="col"`
- Row clicks use `<a>` or `<button>` (not div click)
- Status badges have `aria-label` with full status text
- Filter controls have associated `<label>` elements
- Pagination has `aria-label="تنقل بين الصفحات"`
- Keyboard: Enter on row opens detail, Space toggles checkbox

---

## 4. AP-US-002 — User Detail

### 4.1 Metadata

| Field | Value |
|-------|-------|
| Page ID | `AP-US-002` |
| Page Title (AR) | تفاصيل المستخدم |
| Page Title (EN) | User Detail |
| Route | `/admin/users/:userId` |
| Required Role | Admin |
| Required Permission | `users:read` |
| Parent | AP-US-001 |
| Last Updated | 2026-09-13 |

### 4.2 Purpose

Display comprehensive user profile information, order history, wallet transactions, and support interactions for a single user.

### 4.3 Entry / Exit Points

| Direction | Source / Destination | Trigger |
|-----------|---------------------|---------|
| Entry | AP-US-001 row click | Navigate to user detail |
| Entry | Global search result | Click user result |
| Entry | AP-OR-002 customer link | Click customer name |
| Exit | Back button / breadcrumb | Return to AP-US-001 |
| Exit | Sidebar navigation | Navigate to other pages |

### 4.4 Information Architecture

```
AP-US-002: User Detail
+-- Header Section
|   +-- Avatar (large)
|   +-- Name + Status Badge
|   +-- Contact: Phone, Email
|   +-- Registration date
|   +-- Actions: [Edit] [Suspend] [Ban] [Send Notification]
+-- Tabs
|   +-- نبذة (Overview)
|   |   +-- Stats: Total Orders | Total Spent | Avg Order | Wallet Balance
|   |   +-- Recent Orders (last 5, mini table)
|   |   +-- Wallet Summary (balance, last 5 transactions)
|   |   +-- Addresses (list)
|   +-- الطلبات (Orders) — Full order history table
|   +-- المحفظة (Wallet) — Full wallet transaction history
|   +-- الدعم (Support) — Support tickets related to this user
|   +-- السجل (Activity Log) — Admin actions on this user
+-- Sidebar (sticky)
    +-- Quick Info Card
    +-- Wallet Balance Card (large)
    +-- Risk Score (if implemented)
```

### 4.5 Widgets

#### User Header Card

| Property | Specification |
|----------|---------------|
| Avatar | 80×80 circular, initials fallback |
| Name | 24px bold |
| Status badge | Inline right of name |
| Contact info | 14px, text-secondary, icons: phone, email |
| Registration | "انضم: YYYY/MM/DD" |
| Actions | Button row: Edit, Suspend/Ban, Send Notification |

#### Stats Row (3 cards)

| Stat | Value Source | Format |
|------|-------------|--------|
| إجمالي الطلبات | orderCount | integer |
| إجمالي المصروفات | totalSpent | YER formatted |
| متوسط الطلب | avgOrderValue | YER formatted |

### 4.6 Data Requirements

| Data Point | Source API | Refresh |
|------------|-----------|---------|
| User Profile | `GET /admin/users/:id` | on load |
| Order History | `GET /admin/users/:id/orders` | on tab |
| Wallet Transactions | `GET /admin/users/:id/wallet` | on tab |
| Support Tickets | `GET /admin/users/:id/tickets` | on tab |
| Activity Log | `GET /admin/users/:id/activity` | on tab |

### 4.7 Actions

| Action | Trigger | Behavior | Permission |
|--------|---------|----------|------------|
| Edit Profile | Button click | Open edit drawer | `users:update` |
| Suspend User | Button click | Confirm dialog → suspend | `users:update` |
| Reinstate User | Button click | Confirm dialog → reinstate | `users:update` |
| Ban User | Button click | Confirm dialog (destructive) → ban | `users:update` |
| Send Notification | Button click | Compose notification modal | `users:update` |
| View Order | Order row click | Navigate to AP-OR-002 | `orders:read` |
| View Ticket | Ticket row click | Navigate to AP-SU-002 | `support:read` |

### 4.8 Forms — Edit User Modal

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| Name | Text input | yes | 2-100 chars |
| Email | Email input | yes | Valid email, unique |
| Phone | Tel input | yes | +967XXXXXXXXX format |
| Status | Select | yes | Active / Suspended / Banned |
| Admin Notes | Textarea | no | Max 1000 chars |

### 4.9 States

| State | Description | UI |
|-------|-------------|-----|
| Loading | Fetching user data | Skeleton: header + tabs + content |
| Loaded | Data present | Full detail page |
| Not Found | User ID doesn't exist | EmptyState: "المستخدم غير موجود" |
| Error | API failure | ErrorState + retry |

### 4.10 Empty States

| Scenario | Title (AR) | Action |
|----------|-----------|--------|
| No orders | "لا توجد طلبات بعد" | — |
| No wallet transactions | "لا توجد معاملات محفظة" | — |
| No support tickets | "لا توجد تذاكر دعم" | — |
| No activity log | "لا توجد سجلات نشاط" | — |

### 4.11 RTL Considerations

- Header: avatar on right, name/contact on left
- Tabs flow RTL: first tab on right
- Stats cards: RTL layout
- Order tables: RTL column order

### 4.12 Accessibility

- Tabs use `role="tablist"`, `role="tab"`, `role="tabpanel"`
- Tabs keyboard: Arrow keys to navigate, Enter/Space to activate
- All action buttons have `aria-label`
- Status badge has `aria-label` with full status text
- Heading hierarchy: h1 (name), h2 (tabs), h3 (sections)

---

## 5. AP-VN-001 — Vendor Management

### 5.1 Metadata

| Field | Value |
|-------|-------|
| Page ID | `AP-VN-001` |
| Page Title (AR) | إدارة البائعين |
| Page Title (EN) | Vendor Management |
| Route | `/admin/vendors` |
| Required Role | Admin |
| Required Permission | `vendors:read` |
| Parent | Sidebar → إدارة البائعين |
| Children | AP-VN-002, AP-VN-003 |
| Last Updated | 2026-09-13 |

### 5.2 Purpose

Allow administrators to manage all registered vendors, review their profiles, approve/suspend vendors, and monitor vendor performance.

### 5.3 Entry / Exit Points

| Direction | Source / Destination | Trigger |
|-----------|---------------------|---------|
| Entry | Sidebar "إدارة البائعين" | Navigation |
| Entry | AP-DB-001 stat card click | Active Vendors drill-down |
| Exit | Click vendor row | Navigate to vendor detail |
| Exit | KYC pending badge click | Navigate to AP-VN-002 |

### 5.4 Information Architecture

```
AP-VN-001: Vendor Management
+-- Page Header
|   +-- Title: إدارة البائعين
|   +-- Stats Bar: Total | Active | Pending KYC | Suspended
|   +-- Actions: [Export] [Add Vendor]
+-- Filter Bar
|   +-- Search (name, business name, phone)
|   +-- Status filter (multi-select)
|   +-- KYC Status (Pending / Approved / Rejected)
|   +-- Category filter
|   +-- Registration date range
|   +-- Saved filters
+-- Data Table
|   +-- Columns: Logo | Business Name | Owner | Phone | Products | Revenue | KYC | Status | Actions
|   +-- Sortable: Business Name, Products, Revenue, Joined
|   +-- Row click → Vendor Detail
+-- Bulk Action Bar
+-- Pagination
```

### 5.5 Data Table Columns

| # | Column (AR) | Column (EN) | Field | Width | Sortable | Format |
|---|------------|-------------|-------|-------|----------|--------|
| 1 | — | checkbox | _select | 40px | — | checkbox |
| 2 | الشعار | Logo | logo | 48px | — | circular image |
| 3 | اسم المتجر | Business Name | businessName | flex | yes | text, clickable |
| 4 | المالك | Owner | ownerName | 120px | yes | text |
| 5 | الهاتف | Phone | phone | 140px | — | +967 format |
| 6 | المنتجات | Products | productCount | 80px | yes | integer |
| 7 | الإيرادات | Revenue | revenue | 120px | yes | YER formatted |
| 8 | KYC | KYC Status | kycStatus | 80px | yes | StatusBadge |
| 9 | الحالة | Status | status | 100px | yes | StatusBadge |
| 10 | إجراءات | Actions | _actions | 100px | — | icon buttons |

### 5.6 Status Badges

| Status (AR) | Status (EN) | Color |
|-------------|-------------|-------|
| نشط | Active | green |
| معلّق | Suspended | amber |
| محظور | Banned | red |
| بانتظار KYC | Pending KYC | blue |
| KYC مرفوض | KYC Rejected | red-300 |

### 5.7 KYC Status Badges

| Status (AR) | Status (EN) | Color |
|-------------|-------------|-------|
| مقبول | Approved | green |
| مرفوض | Rejected | red |
| بانتظار المراجعة | Pending Review | amber |
| لم يتم الإرسال | Not Submitted | gray |

### 5.8 Filters

| Filter | Type | Options / Behavior |
|--------|------|-------------------|
| Search | Text input | Debounced 300ms, searches name + business + phone |
| Status | Multi-select | Active, Suspended, Banned |
| KYC Status | Multi-select | Approved, Rejected, Pending, Not Submitted |
| Category | Multi-select | Platform categories |
| Registration Date | Date range | From/To |

### 5.9 Actions

| Action | Trigger | Behavior | Permission |
|--------|---------|----------|------------|
| View Vendor | Row click | Navigate to vendor detail | `vendors:read` |
| Approve KYC | Row action icon | Confirm → approve KYC | `vendors:approve_kyc` |
| Reject KYC | Row action icon | Confirm + reason → reject KYC | `vendors:approve_kyc` |
| Suspend Vendor | Row action icon | Confirm dialog → suspend | `vendors:update` |
| Reinstate Vendor | Row action icon | Confirm dialog → reinstate | `vendors:update` |
| Ban Vendor | Row action icon | Confirm dialog (destructive) → ban | `vendors:update` |
| Export | Header button | Download filtered CSV | `vendors:read` |

### 5.10 States

| State | Description | UI |
|-------|-------------|-----|
| Loading | Fetching vendor list | Table skeleton |
| Loaded | Data present | Full table |
| Empty | No vendors | EmptyState: "لا يوجد بائعون بعد" |
| Error | API failure | ErrorState + retry |

### 5.11 RTL Considerations

- Table: logo rightmost, actions leftmost
- Business name right-aligned
- Revenue numbers LTR (formatted with commas)
- Filter bar RTL flow

### 5.12 Accessibility

- Same as AP-US-001 with vendor-specific labels
- KYC badges have `aria-label` with full status
- Action buttons have descriptive `aria-label`

---

## 6. AP-VN-002 — KYC Review Queue

### 6.1 Metadata

| Field | Value |
|-------|-------|
| Page ID | `AP-VN-002` |
| Page Title (AR) | قائمة مراجعة KYC |
| Page Title (EN) | KYC Review Queue |
| Route | `/admin/vendors/kyc` |
| Required Role | Admin |
| Required Permission | `vendors:approve_kyc` |
| Parent | Sidebar → إدارة البائعين |
| Sibling | AP-VN-001, AP-VN-003 |
| Last Updated | 2026-09-13 |

### 6.2 Purpose

Provide a dedicated queue for administrators to review and process vendor KYC (Know Your Customer) submissions. Prioritize by submission date.

### 6.3 Entry / Exit Points

| Direction | Source / Destination | Trigger |
|-----------|---------------------|---------|
| Entry | Sidebar (badge: pending KYC) | Navigation |
| Entry | AP-VN-001 KYC filter | Click pending KYC count |
| Entry | AP-DB-001 quick action | Click "KYC معلق" badge |
| Exit | Click vendor row | Navigate to AP-VN-003 |

### 6.4 Information Architecture

```
AP-VN-002: KYC Review Queue
+-- Page Header
|   +-- Title: قائمة مراجعة KYC
|   +-- Pending Count badge
|   +-- Actions: [Bulk Approve] [Export]
+-- Filter Bar
|   +-- Search (vendor name)
|   +-- Submission date range
|   +-- Document type
|   +-- Priority (oldest first toggle)
+-- Queue Table
|   +-- Columns: Vendor | Documents | Submitted | Age (days) | Priority | Actions
|   +-- Color coding: > 7 days = amber, > 14 days = red
|   +-- Row click → AP-VN-003
+-- Bulk Action Bar
|   +-- Approve Selected
|   +-- Reject Selected (requires reason)
|   +-- Export
+-- Pagination
```

### 6.5 Queue Table Columns

| # | Column (AR) | Field | Width | Sortable | Format |
|---|------------|-------|-------|----------|--------|
| 1 | — | _select | 40px | — | checkbox |
| 2 | البائع | vendor | flex | yes | logo + name, clickable |
| 3 | المستندات | documents | 120px | — | document icon count |
| 4 | تاريخ الإرسال | submittedAt | 120px | yes | YYYY/MM/DD |
| 5 | العمر (أيام) | ageDays | 80px | yes | integer + color coding |
| 6 | الأولوية | priority | 80px | yes | High / Medium / Low badge |
| 7 | إجراءات | _actions | 120px | — | buttons |

### 6.6 Priority Rules

| Age (Days) | Priority | Color | Badge Text (AR) |
|-----------|----------|-------|-----------------|
| 0-3 | Low | green | عادي |
| 4-7 | Medium | amber | متوسط |
| 8-14 | High | red | عالي |
| 15+ | Critical | red-700 | عاجل |

### 6.7 Actions

| Action | Trigger | Behavior |
|--------|---------|----------|
| Review KYC | Row click / button | Navigate to AP-VN-003 |
| Quick Approve | Button | Confirm dialog → approve |
| Quick Reject | Button | Confirm + reason modal → reject |
| Bulk Approve | Header button | Confirm → approve all selected |
| Bulk Reject | Header button | Confirm + reason → reject all selected |

### 6.8 States

| State | Description | UI |
|-------|-------------|-----|
| Loading | Fetching queue | Table skeleton |
| Loaded | Data present | Full queue table |
| Empty | No pending KYC | EmptyState: "لا توجد طلبات KYC معلقة" + checkmark illustration |
| Error | API failure | ErrorState + retry |

### 6.9 Empty States

| Scenario | Title (AR) | Illustration |
|----------|-----------|-------------|
| No pending KYC | "لا توجد طلبات KYC معلقة" | Checkmark |
| Queue cleared | "تمت مراجعة جميع الطلبات" | Celebration |

### 6.10 RTL Considerations

- Table flows RTL
- Document icons flow RTL
- Age coloring: visual only, no directional dependency

### 6.11 Accessibility

- Priority badges have `aria-label` with priority level
- Age warnings have visual + text indicators (not color-only)
- Row clicks use semantic elements

---

## 7. AP-VN-003 — KYC Detail Review

### 7.1 Metadata

| Field | Value |
|-------|-------|
| Page ID | `AP-VN-003` |
| Page Title (AR) | مراجعة KYC التفصيلية |
| Page Title (EN) | KYC Detail Review |
| Route | `/admin/vendors/kyc/:vendorId` |
| Required Role | Admin |
| Required Permission | `vendors:approve_kyc` |
| Parent | AP-VN-002 |
| Last Updated | 2026-09-13 |

### 7.2 Purpose

Allow administrators to review vendor KYC documents in detail, view submitted documents, and make approve/reject decisions with reasons.

### 7.3 Entry / Exit Points

| Direction | Source / Destination | Trigger |
|-----------|---------------------|---------|
| Entry | AP-VN-002 row click / button | Navigate to detail |
| Exit | Back / breadcrumb | Return to AP-VN-002 |
| Exit | Approve/Reject action | Return to AP-VN-002 + toast |

### 7.4 Information Architecture

```
AP-VN-003: KYC Detail Review
+-- Header
|   +-- Vendor Name + Logo
|   +-- KYC Status Badge
|   +-- Actions: [Approve] [Reject] [Back]
+-- Vendor Info Section
|   +-- Business Name, Owner Name
|   +-- Phone, Email
|   +-- Commercial Registration Number
|   +-- Tax ID
|   +-- Business Address
+-- Documents Section
|   +-- Document 1 (preview + download)
|   +-- Document 2 (preview + download)
|   +-- Document 3 (preview + download)
+-- Review Notes (admin notes)
|   +-- Previous review notes (if any)
|   +-- Add new note
+-- Decision Panel
    +-- Approve Button → Confirm
    +-- Reject Button → Confirm + Reason Modal
    +-- Request More Info Button
```

### 7.5 Widgets

#### Document Preview Card

| Property | Specification |
|----------|---------------|
| Thumbnail | 200×150, click to enlarge (lightbox) |
| Document Type | Label below thumbnail |
| File Size | Small text below type |
| Actions | "معاينة" (preview) + "تحميل" (download) |

#### Decision Panel

| Property | Specification |
|----------|---------------|
| Position | Sticky at bottom of page |
| Approve | Green button, full-width on mobile |
| Reject | Red outline button |
| Request Info | Amber outline button |
| Status | Shows current decision status |

### 7.6 Data Requirements

| Data Point | Source API | Refresh |
|------------|-----------|---------|
| Vendor KYC Info | `GET /admin/vendors/:id/kyc` | on load |
| Documents | `GET /admin/vendors/:id/kyc/documents` | on load |
| Review Notes | `GET /admin/vendors/:id/kyc/notes` | on load |
| Previous Decisions | `GET /admin/vendors/:id/kyc/history` | on load |

### 7.7 Actions

| Action | Trigger | Behavior |
|--------|---------|----------|
| Approve KYC | Button click | Confirm dialog → approve → redirect |
| Reject KYC | Button click | Reason modal → confirm → redirect |
| Request More Info | Button click | Message modal → send to vendor |
| Preview Document | Click thumbnail | Lightbox modal with full document |
| Download Document | Click download | Download file |
| Add Note | Type + save | Add note to review notes |
| Back to Queue | Back button | Navigate to AP-VN-002 |

### 7.8 Forms

#### Reject KYC Modal

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| Rejection Reason | Select | yes | مخالفات في المستندات / بيانات غير صحيحة / مستندات منتهية / أخرى |
| Details | Textarea | yes | Min 10 chars, max 500 chars |

#### Request More Info Modal

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| Message | Textarea | yes | Min 10 chars, max 1000 chars |

### 7.9 States

| State | Description | UI |
|-------|-------------|-----|
| Loading | Fetching KYC data | Skeleton layout |
| Loaded | Data present | Full review page |
| Already Reviewed | KYC already approved/rejected | Read-only view + decision history |
| Not Found | Vendor KYC not found | EmptyState: "طلب KYC غير موجود" |

### 7.10 Empty States

| Scenario | Title (AR) |
|----------|-----------|
| No documents uploaded | "لم يتم رفع مستندات بعد" |
| No review notes | "لا توجد ملاحظات مراجعة" |

### 7.11 RTL Considerations

- Document grid: RTL flow (first document on right)
- Decision panel: buttons flow RTL
- Review notes: RTL text alignment

### 7.12 Accessibility

- Document previews have `aria-label` with document type
- Lightbox has focus trap
- Decision buttons have `aria-label` with full action description
- Confirm dialogs have focus management

---

## 8. AP-PR-001 — Product Management

### 8.1 Metadata

| Field | Value |
|-------|-------|
| Page ID | `AP-PR-001` |
| Page Title (AR) | إدارة المنتجات |
| Page Title (EN) | Product Management |
| Route | `/admin/products` |
| Required Role | Admin |
| Required Permission | `products:read` |
| Parent | Sidebar → إدارة المنتجات |
| Children | AP-PR-002 (Product Moderation) |
| Last Updated | 2026-09-13 |

### 8.2 Purpose

Allow administrators to view, search, filter, and manage all products across all vendors on the platform.

### 8.3 Entry / Exit Points

| Direction | Source / Destination | Trigger |
|-----------|---------------------|---------|
| Entry | Sidebar "إدارة المنتجات" | Navigation |
| Entry | AP-DB-001 stat card | Product count drill-down |
| Exit | Click product row | Navigate to product detail |
| Exit | Moderation badge click | Navigate to AP-PR-002 |

### 8.4 Information Architecture

```
AP-PR-001: Product Management
+-- Page Header
|   +-- Title: إدارة المنتجات
|   +-- Stats: Total | Active | Pending Moderation | Rejected
|   +-- Actions: [Export]
+-- Filter Bar
|   +-- Search (product name, SKU, vendor)
|   +-- Status (Active / Pending / Rejected / Archived)
|   +-- Category (multi-select tree)
|   +-- Vendor (search + select)
|   +-- Price range (min/max YER)
|   +-- Stock status (In Stock / Out of Stock / Low Stock)
|   +-- Date range
+-- Data Table
|   +-- Columns: Image | Name | Vendor | Category | Price | Stock | Status | Actions
|   +-- Sortable: Name, Price, Stock, Created
|   +-- Row click → Product Detail (modal or drawer)
+-- Bulk Action Bar
|   +-- Activate Selected | Deactivate Selected | Delete Selected | Export
+-- Pagination
```

### 8.5 Data Table Columns

| # | Column (AR) | Field | Width | Sortable | Format |
|---|------------|-------|-------|----------|--------|
| 1 | — | _select | 40px | — | checkbox |
| 2 | الصورة | image | 56px | — | 48×48 thumbnail |
| 3 | الاسم | name | flex | yes | text, clickable |
| 4 | البائع | vendorName | 120px | yes | text |
| 5 | الفئة | category | 120px | yes | text |
| 6 | السعر | price | 100px | yes | YER formatted |
| 7 | المخزون | stock | 80px | yes | integer + color |
| 8 | الحالة | status | 100px | yes | StatusBadge |
| 9 | إجراءات | _actions | 100px | — | icon buttons |

### 8.6 Stock Color Coding

| Stock Level | Color | Badge Text (AR) |
|-------------|-------|-----------------|
| 0 | red | نفد المخزون |
| 1-5 | amber | مخزون منخفض |
| 6-20 | green | متوفر |
| 21+ | green | متوفر بكثرة |

### 8.7 Filters

| Filter | Type | Behavior |
|--------|------|----------|
| Search | Text input | Searches name, SKU, vendor name |
| Status | Multi-select | Active, Pending Moderation, Rejected, Archived |
| Category | Tree select | Hierarchical category selection |
| Vendor | Search select | Search by vendor name |
| Price Range | Dual input | Min/Max in YER |
| Stock Status | Multi-select | In Stock, Out of Stock, Low Stock |
| Date Range | Date range | Created date filter |

### 8.8 Actions

| Action | Trigger | Behavior | Permission |
|--------|---------|----------|------------|
| View Product | Row click | Open product detail drawer | `products:read` |
| Edit Product | Row action icon | Open edit drawer | `products:update` |
| Activate Product | Row action icon | Confirm → activate | `products:update` |
| Deactivate Product | Row action icon | Confirm → deactivate | `products:update` |
| Delete Product | Row action icon | Confirm (destructive) → delete | `products:delete` |
| View Vendor | Vendor name click | Navigate to vendor detail | `vendors:read` |
| Export | Header button | Download filtered CSV | `products:read` |

### 8.9 States

| State | Description | UI |
|-------|-------------|-----|
| Loading | Fetching products | Table skeleton |
| Loaded | Data present | Full table |
| Empty | No products | EmptyState: "لا توجد منتجات بعد" |
| Error | API failure | ErrorState + retry |

### 8.10 RTL Considerations

- Table: image rightmost, actions leftmost
- Product name right-aligned
- Price numbers LTR (formatted)
- Category tree: RTL indentation

### 8.11 Accessibility

- Product images have `aria-label` with product name
- Stock color coding supplemented with text badges
- Table headers with `scope="col"`
- Keyboard navigation for tree category selector

---

## 9. AP-PR-002 — Product Moderation

### 9.1 Metadata

| Field | Value |
|-------|-------|
| Page ID | `AP-PR-002` |
| Page Title (AR) | مراجعة المنتجات |
| Page Title (EN) | Product Moderation |
| Route | `/admin/products/moderation` |
| Required Role | Admin |
| Required Permission | `products:update` |
| Parent | Sidebar → إدارة المنتجات |
| Sibling | AP-PR-001 |
| Last Updated | 2026-09-13 |

### 9.2 Purpose

Provide a dedicated moderation queue for reviewing and approving/rejecting newly submitted or edited products before they go live on the platform.

### 9.3 Entry / Exit Points

| Direction | Source / Destination | Trigger |
|-----------|---------------------|---------|
| Entry | Sidebar (moderation badge) | Navigation |
| Entry | AP-PR-001 moderation filter | Click pending count |
| Entry | AP-DB-001 quick action | Click moderation queue badge |
| Exit | Click product row | Navigate to moderation detail |

### 9.4 Information Architecture

```
AP-PR-002: Product Moderation
+-- Page Header
|   +-- Title: مراجعة المنتجات
|   +-- Pending Count
|   +-- Actions: [Bulk Approve] [Bulk Reject]
+-- Filter Bar
|   +-- Search (product name, vendor)
|   +-- Submission date range
|   +-- Category
|   +-- Vendor
+-- Moderation Table
|   +-- Columns: Image | Product | Vendor | Category | Submitted | Actions
|   +-- Row click → Moderation detail modal
|   +-- Quick actions: Approve / Reject per row
+-- Bulk Action Bar
+-- Pagination
```

### 9.5 Moderation Detail Modal

When clicking a product row or review button, a modal opens showing:

```
+------------------------------------------------------------------+
|  مراجعة المنتج: حقيبة ظهر سياحية                           [X]  |
+------------------------------------------------------------------+
|  [product image]  [img 2] [img 3] [img 4]                       |
|                                                                   |
|  اسم المنتج: حقيبة ظهر سياحية                                    |
|  الوصف: حقيبة ظهر مثالية للمواسم ...                             |
|  السعر: 25,000 YER                                               |
|  الفئة: إلكترونات > أكسسوارات                                    |
|  البائع: متجر أحمد علي                                           |
|  المخزون: 15 وحدة                                                |
|                                                                   |
|  +-------------------------------------------------------------+ |
|  | ملاحظات المراجعة                                             | |
|  | [أضف ملاحظة...]                                              | |
|  +-------------------------------------------------------------+ |
|                                                                   |
|  [قبول المنتج]  [رفض المنتج]  [طلب تعديل]                      |
+------------------------------------------------------------------+
```

### 9.6 Actions

| Action | Trigger | Behavior |
|--------|---------|----------|
| Review Product | Row click | Open moderation detail modal |
| Quick Approve | Button | Confirm → approve → remove from queue |
| Quick Reject | Button | Reason modal → reject → remove |
| Approve (in modal) | Button click | Confirm → approve → close modal |
| Reject (in modal) | Button click | Reason modal → reject → close |
| Request Edit | Button click | Message modal → send to vendor |
| Bulk Approve | Header button | Confirm → approve selected |
| Bulk Reject | Header button | Confirm + reason → reject selected |

### 9.7 Forms — Reject Product Modal

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| Rejection Reason | Select | yes | محتوى مخالف / صورة غير واضحة / سعر غير مناسب / وصف غير كافٍ / أخرى |
| Details | Textarea | yes | Min 10 chars, max 500 chars |

### 9.8 States

| State | Description | UI |
|-------|-------------|-----|
| Loading | Fetching moderation queue | Table skeleton |
| Loaded | Data present | Full queue |
| Empty | No products pending moderation | EmptyState: "لا توجد منتجات بانتظار المراجعة" + checkmark |
| Error | API failure | ErrorState + retry |

### 9.9 Empty States

| Scenario | Title (AR) |
|----------|-----------|
| Queue empty | "تمت مراجعة جميع المنتجات" |
| No products from vendor | "لا توجد منتجات من هذا البائع" |

### 9.10 RTL Considerations

- Product images: RTL grid flow
- Modal content: RTL alignment
- Decision buttons: RTL button order

### 9.11 Accessibility

- Modal has focus trap
- Image gallery has keyboard navigation
- Reject reason select has proper labeling
- Confirmation dialogs announced via `aria-live`

---

## 10. AP-OR-001 — Order Management

### 10.1 Metadata

| Field | Value |
|-------|-------|
| Page ID | `AP-OR-001` |
| Page Title (AR) | إدارة الطلبات |
| Page Title (EN) | Order Management |
| Route | `/admin/orders` |
| Required Role | Admin |
| Required Permission | `orders:read` |
| Parent | Sidebar → الطلبات |
| Children | AP-OR-002 (Order Detail) |
| Last Updated | 2026-09-13 |

### 10.2 Purpose

Allow administrators to view, search, filter, and manage all orders on the platform. Monitor order status, handle disputes, and assist with order issues.

### 10.3 Entry / Exit Points

| Direction | Source / Destination | Trigger |
|-----------|---------------------|---------|
| Entry | Sidebar "الطلبات" | Navigation |
| Entry | AP-DB-001 stat card | Orders drill-down |
| Exit | Click order row | Navigate to AP-OR-002 |
| Exit | Status filter click | Navigate with pre-applied filter |

### 10.4 Information Architecture

```
AP-OR-001: Order Management
+-- Page Header
|   +-- Title: إدارة الطلبات
|   +-- Stats by Status (horizontal scroll chips)
|   +-- Actions: [Export] [Advanced Search]
+-- Filter Bar
|   +-- Search (order ID, customer name, vendor name)
|   +-- Status (multi-select with all 17 states)
|   +-- Date range
|   +-- Payment method (Wallet)
|   +-- Amount range (min/max YER)
|   +-- Vendor
|   +-- Delivery provider
+-- Data Table
|   +-- Columns: Order ID | Customer | Vendor | Items | Total | Status | Payment | Created | Actions
|   +-- Sortable: Order ID, Total, Created
|   +-- Row color: status-dependent subtle background
|   +-- Row click → AP-OR-002
+-- Bulk Action Bar
+-- Pagination
```

### 10.5 Data Table Columns

| # | Column (AR) | Field | Width | Sortable | Format |
|---|------------|-------|-------|----------|--------|
| 1 | — | _select | 40px | — | checkbox |
| 2 | رقم الطلب | orderId | 120px | yes | #ORD-XXXX, clickable |
| 3 | العميل | customerName | 120px | yes | text, clickable |
| 4 | البائع | vendorName | 120px | yes | text, clickable |
| 5 | المنتجات | itemCount | 80px | — | "3 منتجات" |
| 6 | المبلغ الإجمالي | totalAmount | 120px | yes | YER formatted |
| 7 | حالة الطلب | status | 140px | yes | StatusBadge (17) |
| 8 | الدفع | paymentStatus | 100px | — | StatusBadge |
| 9 | تاريخ الإنشاء | createdAt | 120px | yes | YYYY/MM/DD HH:mm |
| 10 | إجراءات | _actions | 80px | — | icon buttons |

### 10.6 Status Filter Chips (17 states)

All 17 order states displayed as horizontally scrollable chips at the top. Each chip shows: status color dot, status label (AR), count badge, click to filter.

### 10.7 Filters

| Filter | Type | Behavior |
|--------|------|----------|
| Search | Text input | Searches orderId, customer name, vendor name |
| Status | Multi-select | All 17 order states |
| Date Range | Date range | Order creation date |
| Payment Status | Multi-select | Paid, Pending, Refunded |
| Amount Range | Dual input | Min/Max in YER |
| Vendor | Search select | Search by vendor name |
| Delivery Provider | Select | Platform delivery providers |

### 10.8 Row Color Coding

| Status Group | Background Tint |
|-------------|----------------|
| Pending Payment/Confirmation | Light gray |
| Processing/Preparing | Light blue |
| Ready/Out for Delivery | Light purple |
| Delivered/Completed | Light green |
| Cancelled (any) | Light red |
| Refund states | Light orange |
| Dispute Open | Light red (stronger) |

### 10.9 Status Update Flow

Admin can only update to valid next states based on current status:

```
Pending Payment → Pending Confirmation / Cancelled by Admin
Pending Confirmation → Processing / Cancelled by Admin
Processing → Preparing / Cancelled by Admin
Preparing → Ready for Pickup / Cancelled by Admin
Ready for Pickup → Out for Delivery / Cancelled by Admin
Out for Delivery → Delivered / Dispute Open
Delivered → Completed / Refund Pending
Completed → (terminal, can add notes)
Cancelled by * → (terminal)
Refund Pending → Refunded / Partially Refunded
Refunded → (terminal)
Partially Refunded → (terminal)
Dispute Open → (resolves to other states)
```

### 10.10 Actions

| Action | Trigger | Behavior | Permission |
|--------|---------|----------|------------|
| View Order | Row click | Navigate to AP-OR-002 | `orders:read` |
| Update Status | Row action icon | Status update modal (if allowed) | `orders:update` |
| Cancel Order | Row action icon | Confirm + reason → cancel | `orders:update` |
| Refund Order | Row action icon | Confirm + reason → refund | `orders:update` |
| Contact Customer | Row action icon | Open notification compose | `orders:read` |
| Contact Vendor | Row action icon | Open notification compose | `orders:read` |
| Export | Header button | Download filtered CSV | `orders:read` |

### 10.11 States

| State | Description | UI |
|-------|-------------|-----|
| Loading | Fetching orders | Table skeleton |
| Loaded | Data present | Full table + status chips |
| Empty | No orders match filters | EmptyState: "لا توجد طلبات" |
| Error | API failure | ErrorState + retry |

### 10.12 Empty States

| Scenario | Title (AR) |
|----------|-----------|
| No orders at all | "لا توجد طلبات بعد" |
| No orders for filter | "لا توجد طلبات تطابق الفلاتر" |
| No orders today | "لا توجد طلبات اليوم" |

### 10.13 RTL Considerations

- Table: Order ID rightmost, actions leftmost
- Amounts LTR formatted
- Status chips: RTL flow
- Date format: YYYY/MM/DD

### 10.14 Accessibility

- Status chips have `aria-label` with count
- Row clicks use semantic elements
- Status update modal has proper focus management
- Table sortable columns have `aria-sort`

---

## 11. AP-OR-002 — Order Detail

### 11.1 Metadata

| Field | Value |
|-------|-------|
| Page ID | `AP-OR-002` |
| Page Title (AR) | تفاصيل الطلب |
| Page Title (EN) | Order Detail |
| Route | `/admin/orders/:orderId` |
| Required Role | Admin |
| Required Permission | `orders:read` |
| Parent | AP-OR-001 |
| Last Updated | 2026-09-13 |

### 11.2 Purpose

Display comprehensive order information including items, customer details, vendor details, payment, delivery, and timeline. Allow administrators to take actions on the order.

### 11.3 Entry / Exit Points

| Direction | Source / Destination | Trigger |
|-----------|---------------------|---------|
| Entry | AP-OR-001 row click | Navigate to order detail |
| Entry | Global search (order ID) | Click order result |
| Exit | Back / breadcrumb | Return to AP-OR-001 |
| Exit | Customer name link | Navigate to AP-US-002 |
| Exit | Vendor name link | Navigate to vendor detail |

### 11.4 Information Architecture

```
AP-OR-002: Order Detail
+-- Header
|   +-- Order ID (#ORD-XXXX)
|   +-- Status Badge (large)
|   +-- Created date
|   +-- Actions: [Update Status] [Cancel] [Refund] [Print]
+-- Order Timeline (vertical)
|   +-- Status history with timestamps
|   +-- Current status highlighted
+-- Two-Column Layout
|   +-- Left Column
|   |   +-- Order Items (table: product image + name, quantity, unit price, subtotal)
|   |   +-- Order Summary (subtotal, delivery fee, VAT, discount, total)
|   |   +-- Payment Info (method, transaction ID, status)
|   +-- Right Column
|       +-- Customer Info (name, phone, email, link to AP-US-002, delivery address)
|       +-- Vendor Info (business name, logo, link to vendor detail)
|       +-- Delivery Info (provider, tracking number, estimated delivery, address)
+-- Admin Notes Section (previous notes + add new)
+-- Activity Log (all admin actions on this order)
```

### 11.5 Order Timeline Component

| Property | Specification |
|----------|---------------|
| Orientation | Vertical, RTL (right-to-left flow) |
| Completed nodes | Green circle + checkmark + timestamp |
| Current node | Blue pulsing circle + "الحالي" label |
| Future nodes | Gray circle + dashed line |
| Animation | Subtle fade-in on load |

### 11.6 Data Requirements

| Data Point | Source API | Refresh |
|------------|-----------|---------|
| Order Details | `GET /admin/orders/:id` | on load |
| Order Timeline | `GET /admin/orders/:id/timeline` | on load |
| Customer Info | Nested in order response | — |
| Vendor Info | Nested in order response | — |
| Delivery Info | Nested in order response | — |
| Admin Notes | `GET /admin/orders/:id/notes` | on load |
| Activity Log | `GET /admin/orders/:id/activity` | on load |

### 11.7 Actions

| Action | Trigger | Behavior | Permission |
|--------|---------|----------|------------|
| Update Status | Button click | Status update modal | `orders:update` |
| Cancel Order | Button click | Confirm + reason modal | `orders:update` |
| Process Refund | Button click | Confirm + reason → refund modal | `orders:update` |
| Print Order | Button click | Print-friendly view / PDF | `orders:read` |
| View Customer | Link click | Navigate to AP-US-002 | `users:read` |
| View Vendor | Link click | Navigate to vendor detail | `vendors:read` |
| Add Note | Type + save | Add admin note | `orders:update` |
| Send Notification | Button click | Compose notification to customer/vendor | `orders:update` |

### 11.8 Forms

#### Update Status Modal

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| New Status | Select | yes | Only valid next states shown |
| Admin Notes | Textarea | no | Max 500 chars |
| Notify Customer | Checkbox | — | Default: checked |
| Notify Vendor | Checkbox | — | Default: unchecked |

#### Cancel Order Modal

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| Cancellation Reason | Select | yes | طلب العميل / مشكلة في المنتج / عدم توفر المخزون / أخرى |
| Details | Textarea | yes | Min 10 chars, max 500 chars |
| Notify Customer | Checkbox | — | Default: checked |

#### Refund Order Modal

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| Refund Type | Radio | yes | كامل / جزئي |
| Refund Amount | Number | conditional | Required if partial, max = order total |
| Refund Reason | Select | yes | إلغاء الطلب / مشكلة في المنتج / خطأ في الطلب / أخرى |
| Details | Textarea | yes | Min 10 chars, max 500 chars |

### 11.9 States

| State | Description | UI |
|-------|-------------|-----|
| Loading | Fetching order data | Skeleton layout |
| Loaded | Data present | Full detail page |
| Not Found | Order ID invalid | EmptyState: "الطلب غير موجود" |
| Error | API failure | ErrorState + retry |

### 11.10 Empty States

| Scenario | Title (AR) |
|----------|-----------|
| No admin notes | "لا توجد ملاحظات بعد" |
| No activity log | "لا توجد سجلات نشاط" |
| No timeline events | "لا توجد أحداث في الجدول الزمني" |

### 11.11 RTL Considerations

- Timeline: vertical flow, RTL alignment
- Two-column layout: RTL (customer info on left, items on right)
- Amounts: LTR formatted
- Addresses: RTL text

### 11.12 Accessibility

- Timeline has `role="list"` with `role="listitem"` for each event
- Status update modal has focus trap
- Print view has proper heading structure
- All action buttons have `aria-label`

---

## 12. AP-FN-001 — Financial Dashboard

### 12.1 Metadata

| Field | Value |
|-------|-------|
| Page ID | `AP-FN-001` |
| Page Title (AR) | لوحة المالية |
| Page Title (EN) | Financial Dashboard |
| Route | `/admin/finance` |
| Required Role | Admin |
| Required Permission | `finance:read` |
| Parent | Sidebar → المالية |
| Children | AP-FN-002, AP-FN-003, AP-FN-004, AP-FN-005 |
| Last Updated | 2026-09-13 |

### 12.2 Purpose

Provide a comprehensive financial overview of the platform including revenue, commissions, escrow balances, pending payouts, and VAT collection.

### 12.3 Entry / Exit Points

| Direction | Source / Destination | Trigger |
|-----------|---------------------|---------|
| Entry | Sidebar "المالية" | Navigation |
| Entry | AP-DB-001 stat card | Revenue drill-down |
| Exit | Click metric card | Navigate to sub-page |
| Exit | Sidebar sub-navigation | Navigate to sub-pages |

### 12.4 Information Architecture

```
AP-FN-001: Financial Dashboard
+-- Page Header
|   +-- Title: لوحة المالية
|   +-- Date Range Picker (Today / 7d / 30d / Custom)
|   +-- Currency Toggle (YER / USD)
+-- Top Stats Row (5 cards)
|   +-- Total Revenue | Platform Commission | Escrow Balance | Pending Payouts | VAT Collected
+-- Charts Section (2-column)
|   +-- Revenue vs Commission Trend (line chart)
|   +-- Revenue by Category (bar chart)
+-- Quick Links → AP-FN-002/003/004/005
+-- Recent Transactions (table, last 10)
+-- Financial Alerts (low escrow, pending payouts, VAT deadline)
```

### 12.5 Data Requirements

| Data Point | Source API | Refresh |
|------------|-----------|---------|
| Revenue Summary | `GET /admin/finance/revenue` | 300s |
| Commission Summary | `GET /admin/finance/commission` | 300s |
| Escrow Balance | `GET /admin/finance/escrow` | 120s |
| Pending Payouts | `GET /admin/finance/pending-payouts` | 120s |
| VAT Collected | `GET /admin/finance/vat` | 600s |
| Revenue Trend | `GET /admin/finance/trend` | 300s |
| Revenue by Category | `GET /admin/finance/by-category` | 600s |
| Recent Transactions | `GET /admin/finance/transactions` | 120s |

### 12.6 Actions

| Action | Trigger | Behavior |
|--------|---------|----------|
| Change Date Range | Dropdown | Update all metrics for selected range |
| Toggle Currency | Toggle | Switch YER ↔ USD display |
| View Commission | Card/link click | Navigate to AP-FN-002 |
| View Escrow | Card/link click | Navigate to AP-FN-003 |
| View Payouts | Card/link click | Navigate to AP-FN-004 |
| View VAT | Card/link click | Navigate to AP-FN-005 |
| View Transaction | Table row click | Open transaction detail modal |

### 12.7 States

| State | Description | UI |
|-------|-------------|-----|
| Loading | Fetching financial data | Skeleton cards + chart placeholders |
| Loaded | Data present | Full dashboard |
| Error | API failure | ErrorState + retry |

### 12.8 Empty States

| Scenario | Title (AR) |
|----------|-----------|
| No transactions | "لا توجد معاملات مالية بعد" |
| No revenue data | "لا توجد بيانات إيرادات" |

### 12.9 RTL Considerations

- Stats cards: RTL layout
- Charts: X-axis RTL
- Transaction table: RTL column order
- Amounts: LTR formatted within RTL context

### 12.10 Accessibility

- Financial data announced with `aria-live` on updates
- Charts have accessible table fallback
- All links have descriptive `aria-label`
- Color coding supplemented with icons/text

---

## 13. AP-FN-002 — Commission Management

### 13.1 Metadata

| Field | Value |
|-------|-------|
| Page ID | `AP-FN-002` |
| Page Title (AR) | إدارة العمولات |
| Page Title (EN) | Commission Management |
| Route | `/admin/finance/commissions` |
| Required Role | Admin |
| Required Permission | `finance:read` |
| Parent | AP-FN-001 |
| Last Updated | 2026-09-13 |

### 13.2 Purpose

Manage platform commission rates, view commission history, and configure commission rules per category or vendor.

### 13.3 Information Architecture

```
AP-FN-002: Commission Management
+-- Page Header
|   +-- Title: إدارة العمولات
|   +-- Current Commission Rate (display)
|   +-- Actions: [Edit Rate] [View History]
+-- Commission Rules
|   +-- Default Rate Card
|   +-- Category-specific Rates
|   +-- Vendor-specific Rates
+-- Commission History Table
|   +-- Columns: Date | Order | Vendor | Sale Amount | Commission | Status
|   +-- Filters: Date range, vendor, status
+-- Commission Summary
|   +-- Total Commission (period) | Average Commission Rate | Commission by Category (chart)
+-- Export Options: CSV, PDF
```

### 13.4 Commission Rule Types

| Rule Type | Description |
|-----------|-------------|
| Default Rate | Platform-wide commission percentage |
| Category Rate | Override rate per category |
| Vendor Rate | Override rate per vendor (negotiated) |
| Minimum Commission | Floor amount per transaction |
| Maximum Commission | Cap amount per transaction |

### 13.5 Data Table Columns

| # | Column (AR) | Field | Sortable | Format |
|---|------------|-------|----------|--------|
| 1 | التاريخ | date | yes | YYYY/MM/DD |
| 2 | رقم الطلب | orderId | yes | #ORD-XXXX |
| 3 | البائع | vendorName | yes | text |
| 4 | مبلغ البيع | saleAmount | yes | YER formatted |
| 5 | العمولة (%) | commissionPct | — | percentage |
| 6 | مبلغ العمولة | commissionAmt | yes | YER formatted |
| 7 | الحالة | status | yes | StatusBadge |

### 13.6 Actions

| Action | Trigger | Behavior | Permission |
|--------|---------|----------|------------|
| Edit Default Rate | Button click | Edit modal | super_admin |
| Edit Category Rate | Row action icon | Edit modal | super_admin |
| Edit Vendor Rate | Row action icon | Edit modal | super_admin |
| View Commission Detail | Row click | Open detail modal | `finance:read` |
| Export | Button click | Download CSV/PDF | `finance:read` |

### 13.7 Forms — Edit Commission Rate Modal

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| Rate Type | Radio | yes | Default / Category / Vendor |
| Category | Select | conditional | Required if Category type |
| Vendor | Search select | conditional | Required if Vendor type |
| Commission Rate (%) | Number | yes | 0-50%, step 0.5% |
| Min Commission | Number | no | ≥ 0 YER |
| Max Commission | Number | no | ≥ min commission |
| Effective Date | Date picker | yes | Must be today or future |

### 13.8 States

| State | Description | UI |
|-------|-------------|-----|
| Loading | Fetching commission data | Table skeleton |
| Loaded | Data present | Full page |
| Empty | No commission history | EmptyState: "لا توجد سجلات عمولات" |

### 13.9 RTL Considerations

- Commission rates: LTR percentage display
- Table: RTL column order
- Charts: RTL axes

### 13.10 Accessibility

- Rate editing modal has focus trap
- Commission percentages have `aria-label`
- Table sortable columns properly labeled

---

## 14. AP-FN-003 — Escrow Management

### 14.1 Metadata

| Field | Value |
|-------|-------|
| Page ID | `AP-FN-003` |
| Page Title (AR) | إدارة الوصائل (الضمان المالي) |
| Page Title (EN) | Escrow Management |
| Route | `/admin/finance/escrow` |
| Required Role | Admin |
| Required Permission | `finance:read` |
| Parent | AP-FN-001 |
| Last Updated | 2026-09-13 |

### 14.2 Purpose

Monitor and manage the escrow system that holds customer payments until order completion. View escrow balances, release payments, and handle refund escrows.

### 14.3 Information Architecture

```
AP-FN-003: Escrow Management
+-- Page Header
|   +-- Title: إدارة الوصائل
|   +-- Total Escrow Balance
|   +-- Actions: [Export] [Manual Release]
+-- Escrow Stats
|   +-- Total Held in Escrow | Released Today | Pending Release | Refund Escrows
+-- Escrow Transactions Table
|   +-- Columns: TXN ID | Order | Vendor | Amount | Status | Created | Released
|   +-- Status: Held / Released / Refunded / Disputed
|   +-- Filters: Status, date range, vendor
+-- Escrow by Vendor (table: Vendor | Held Amount | Released Amount | Pending)
+-- Escrow Alerts (high balance, release failures, dispute escalations)
```

### 14.4 Escrow States

| State (AR) | State (EN) | Color | Description |
|-----------|-----------|-------|-------------|
| محتجز | Held | amber | Payment held pending delivery |
| محرر | Released | green | Released to vendor after completion |
| مسترجع | Refunded | blue | Refunded to customer |
| متنازع عليه | Disputed | red | Under dispute review |

### 14.5 Data Table Columns

| # | Column (AR) | Field | Sortable | Format |
|---|------------|-------|----------|--------|
| 1 | رقم المعاملة | txnId | yes | TXN-XXXX |
| 2 | رقم الطلب | orderId | yes | #ORD-XXXX |
| 3 | البائع | vendorName | yes | text |
| 4 | المبلغ | amount | yes | YER formatted |
| 5 | الحالة | status | yes | StatusBadge |
| 6 | تاريخ الإنشاء | createdAt | yes | YYYY/MM/DD |
| 7 | تاريخ التحرير | releasedAt | yes | YYYY/MM/DD |
| 8 | إجراءات | _actions | — | icon buttons |

### 14.6 Actions

| Action | Trigger | Behavior | Permission |
|--------|---------|----------|------------|
| View Escrow Detail | Row click | Open detail modal | `finance:read` |
| Manual Release | Button click | Confirm → release escrow to vendor | super_admin |
| Hold Escrow | Row action icon | Confirm + reason → hold | super_admin |
| Release Escrow | Row action icon | Confirm → release | super_admin |
| Export | Button click | Download CSV | `finance:read` |

### 14.7 States

| State | Description | UI |
|-------|-------------|-----|
| Loading | Fetching escrow data | Table skeleton |
| Loaded | Data present | Full page |
| Empty | No escrow transactions | EmptyState: "لا توجد معاملات وصيلة" |

### 14.8 RTL Considerations

- Amounts: LTR formatted
- Table: RTL column order
- Status badges: RTL alignment

### 14.9 Accessibility

- Manual release has confirmation dialog with focus trap
- Escrow amounts have `aria-label` with full context
- Status badges have text alternatives

---

## 15. AP-FN-004 — Payout Management

### 15.1 Metadata

| Field | Value |
|-------|-------|
| Page ID | `AP-FN-004` |
| Page Title (AR) | إدارة المدفوعات للبائعين |
| Page Title (EN) | Payout Management |
| Route | `/admin/finance/payouts` |
| Required Role | Admin |
| Required Permission | `finance:read` |
| Parent | AP-FN-001 |
| Last Updated | 2026-09-13 |

### 15.2 Purpose

Manage vendor payouts including scheduling, processing, and tracking payments to vendors for completed orders.

### 15.3 Information Architecture

```
AP-FN-004: Payout Management
+-- Page Header
|   +-- Title: إدارة المدفوعات
|   +-- Pending Payouts Count + Total Amount
|   +-- Actions: [Process All] [Export] [Schedule]
+-- Payout Stats
|   +-- Total Paid (period) | Pending Payouts | Failed Payouts | Average Payout Time
+-- Payout Queue (primary table)
|   +-- Columns: Vendor | Amount | Orders | Payment Method | Status | Actions
|   +-- Status: Pending / Processing / Completed / Failed
|   +-- Row actions: Process / Cancel / View
+-- Payout History (secondary table)
|   +-- Columns: Date | Vendor | Amount | Method | Status | Reference
|   +-- Filters: Date range, vendor, status
+-- Payout Configuration
    +-- Minimum payout threshold | Payout schedule | Payment methods
```

### 15.4 Payout States

| State (AR) | State (EN) | Color |
|-----------|-----------|-------|
| بانتظار المعالجة | Pending | amber |
| قيد المعالجة | Processing | blue |
| مكتمل | Completed | green |
| فاشل | Failed | red |
| ملغي | Cancelled | gray |

### 15.5 Data Table Columns

| # | Column (AR) | Field | Sortable | Format |
|---|------------|-------|----------|--------|
| 1 | البائع | vendorName | yes | logo + name |
| 2 | المبلغ | amount | yes | YER formatted |
| 3 | عدد الطلبات | orderCount | yes | integer |
| 4 | طريقة الدفع | paymentMethod | yes | icon + text |
| 5 | الحالة | status | yes | StatusBadge |
| 6 | تاريخ الاستحقاق | dueDate | yes | YYYY/MM/DD |
| 7 | إجراءات | _actions | — | buttons |

### 15.6 Actions

| Action | Trigger | Behavior | Permission |
|--------|---------|----------|------------|
| Process Payout | Row action icon | Confirm → process payout | super_admin |
| Process All Pending | Header button | Confirm → batch process | super_admin |
| Cancel Payout | Row action icon | Confirm + reason → cancel | super_admin |
| View Payout Detail | Row click | Open detail modal | `finance:read` |
| Export | Button click | Download CSV/PDF | `finance:read` |
| Edit Schedule | Button click | Edit configuration modal | super_admin |

### 15.7 Forms

#### Process Payout Modal

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| Vendor | Display only | — | — |
| Amount | Display only | — | — |
| Payment Method | Select | yes | Bank Transfer / Mobile Wallet |
| Account Details | Display only | — | Pre-filled from vendor profile |
| Reference Number | Text input | yes | Required, alphanumeric |
| Notes | Textarea | no | Max 500 chars |

#### Payout Configuration Modal

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| Minimum Threshold | Number | yes | ≥ 1,000 YER |
| Schedule | Select | yes | Weekly / Bi-weekly / Monthly |
| Payment Methods | Multi-checkbox | yes | At least one selected |

### 15.8 States

| State | Description | UI |
|-------|-------------|-----|
| Loading | Fetching payout data | Table skeleton |
| Loaded | Data present | Full page |
| Empty | No pending payouts | EmptyState: "لا توجد مدفوعات معلقة" |

### 15.9 Empty States

| Scenario | Title (AR) |
|----------|-----------|
| No pending payouts | "لا توجد مدفوعات معلقة" |
| No payout history | "لا توجد سجلات مدفوعات" |
| All payouts processed | "تمت معالجة جميع المدفوعات" |

### 15.10 RTL Considerations

- Amounts: LTR formatted
- Table: RTL column order
- Payment method icons: no directional dependency

### 15.11 Accessibility

- Process payout modal has focus trap
- Batch process has confirmation dialog
- Amount values have `aria-label`
- Status badges have text alternatives

---

## 16. AP-FN-005 — VAT Management

### 16.1 Metadata

| Field | Value |
|-------|-------|
| Page ID | `AP-FN-005` |
| Page Title (AR) | إدارة الضريبة (VAT) |
| Page Title (EN) | VAT Management |
| Route | `/admin/finance/vat` |
| Required Role | Admin |
| Required Permission | `finance:read` |
| Parent | AP-FN-001 |
| Last Updated | 2026-09-13 |

### 16.2 Purpose

Manage VAT (Value Added Tax) configuration, track VAT collection, and generate VAT reports for tax authority filing.

### 16.3 Information Architecture

```
AP-FN-005: VAT Management
+-- Page Header
|   +-- Title: إدارة الضريبة (VAT)
|   +-- Current VAT Rate
|   +-- Actions: [Edit Rate] [Generate Report]
+-- VAT Stats
|   +-- Total VAT Collected (period) | VAT by Category | VAT by Vendor | Next Filing Deadline
+-- VAT Configuration
|   +-- VAT Rate (%) | Exempt Categories | Exempt Vendors (below threshold) | Tax Registration Number
+-- VAT Collection Table
|   +-- Columns: Period | Gross Sales | VAT Collected | Status | Filed
|   +-- Filters: Period, status
+-- VAT by Vendor Table (Vendor | Sales | VAT Collected | Filed)
+-- Filing History (past VAT filings with status)
```

### 16.4 VAT Configuration Fields

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| VAT Rate | Number | 0% | Current VAT percentage |
| Tax Registration Number | Text | — | Platform tax ID |
| Exempt Categories | Multi-select | — | Categories exempt from VAT |
| Exempt Vendor Threshold | Number | — | Vendors below this annual sales exempt |
| Filing Frequency | Select | Monthly | Monthly / Quarterly |

### 16.5 Data Requirements

| Data Point | Source API | Refresh |
|------------|-----------|---------|
| VAT Configuration | `GET /admin/finance/vat/config` | on load |
| VAT Collection Summary | `GET /admin/finance/vat/summary` | 600s |
| VAT by Vendor | `GET /admin/finance/vat/by-vendor` | 600s |
| Filing History | `GET /admin/finance/vat/filings` | on load |
| Filing Deadline | `GET /admin/finance/vat/deadline` | 3600s |

### 16.6 Actions

| Action | Trigger | Behavior | Permission |
|--------|---------|----------|------------|
| Edit VAT Rate | Button click | Edit modal | super_admin |
| Generate Report | Button click | Generate VAT report for period | `finance:read` |
| File VAT | Button click | Mark period as filed | super_admin |
| Export | Button click | Download CSV/PDF | `finance:read` |
| View Vendor VAT | Row click | Open vendor VAT detail modal | `finance:read` |

### 16.7 Forms — Edit VAT Configuration Modal

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| VAT Rate (%) | Number | yes | 0-30%, step 0.5% |
| Tax Registration | Text | yes | Required, alphanumeric |
| Exempt Categories | Multi-select | no | — |
| Exempt Threshold | Number | no | ≥ 0 YER |
| Filing Frequency | Select | yes | Monthly / Quarterly |

### 16.8 States

| State | Description | UI |
|-------|-------------|-----|
| Loading | Fetching VAT data | Table skeleton |
| Loaded | Data present | Full page |
| Empty | No VAT collection data | EmptyState: "لا توجد بيانات ضريبة" |

### 16.9 Empty States

| Scenario | Title (AR) |
|----------|-----------|
| No VAT data | "لا توجد بيانات ضريبية" |
| No filing history | "لا توجد سجلات إقرار ضريبي" |

### 16.10 RTL Considerations

- VAT rates: LTR percentage
- Table: RTL column order
- Filing dates: RTL format

### 16.11 Accessibility

- VAT rate editing has focus trap
- Report generation has loading indicator
- Filing status has text alternatives

---

## 17. AP-CM-001 — Content Management (Banners, Pages)

### 17.1 Metadata

| Field | Value |
|-------|-------|
| Page ID | `AP-CM-001` |
| Page Title (AR) | إدارة المحتوى |
| Page Title (EN) | Content Management |
| Route | `/admin/content` |
| Required Role | Super Admin |
| Required Permission | super_admin only |
| Parent | Sidebar → إدارة المحتوى |
| Last Updated | 2026-09-13 |

### 17.2 Purpose

Manage platform content including homepage banners, promotional slides, static pages (About, Terms, Privacy Policy), and announcement banners.

### 17.3 Information Architecture

```
AP-CM-001: Content Management
+-- Page Header
|   +-- Title: إدارة المحتوى
|   +-- Tabs: [البانرات] [الصفحات] [الإعلانات]
+-- Banners Tab
|   +-- Banner List (drag-to-reorder)
|   |   +-- Columns: Preview | Title | Position | Link | Status | Actions
|   +-- Actions: [Add Banner] [Preview All]
+-- Pages Tab
|   +-- Pages List
|   |   +-- Columns: Title | Slug | Last Updated | Status | Actions
|   +-- Actions: [Add Page]
+-- Announcements Tab
|   +-- Active Announcements
|   +-- Scheduled Announcements
|   +-- Actions: [Add Announcement]
```

### 17.4 Banner Data Table Columns

| # | Column (AR) | Field | Format |
|---|------------|-------|--------|
| 1 | المعاينة | preview | 120×60 thumbnail |
| 2 | العنوان | title | text |
| 3 | الموضع | position | homepage / category / checkout |
| 4 | الرابط | linkUrl | URL |
| 5 | الحالة | status | StatusBadge (Active/Draft/Scheduled) |
| 6 | إجراءات | _actions | edit/delete/preview |

### 17.5 Page Data Table Columns

| # | Column (AR) | Field | Format |
|---|------------|-------|--------|
| 1 | العنوان | title | text |
| 2 | الرابط | slug | /pages/slug |
| 3 | آخر تحديث | updatedAt | YYYY/MM/DD |
| 4 | الحالة | status | StatusBadge |
| 5 | إجراءات | _actions | edit/delete/preview |

### 17.6 Actions

| Action | Trigger | Behavior |
|--------|---------|----------|
| Add Banner | Button click | Open banner create modal |
| Edit Banner | Row action | Open banner edit modal |
| Delete Banner | Row action | Confirm (destructive) → delete |
| Reorder Banners | Drag and drop | Update sort order |
| Add Page | Button click | Open page editor (rich text) |
| Edit Page | Row action | Open page editor |
| Delete Page | Row action | Confirm (destructive) → delete |
| Preview | Row action | Open preview in new tab |
| Add Announcement | Button click | Open announcement create modal |

### 17.7 Forms — Banner Create/Edit Modal

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| Title | Text input | yes | 2-100 chars |
| Image | FileUpload | yes | JPG/PNG/WebP, max 2MB, 1920×600 recommended |
| Mobile Image | FileUpload | no | JPG/PNG/WebP, max 1MB, 750×400 recommended |
| Link URL | Text input | no | Valid URL |
| Position | Select | yes | homepage / category / checkout |
| Status | Select | yes | Active / Draft / Scheduled |
| Start Date | DatePicker | conditional | Required if Scheduled |
| End Date | DatePicker | conditional | Must be after start date |
| Alt Text | Text input | yes | Max 125 chars (accessibility) |

### 17.8 States

| State | Description | UI |
|-------|-------------|-----|
| Loading | Fetching content | Table skeleton |
| Loaded | Data present | Full page with tabs |
| Empty | No content | EmptyState: "لا يوجد محتوى بعد" |

### 17.9 RTL Considerations

- Banner preview: RTL text overlay
- Drag-to-reorder: RTL handle position
- Page editor: RTL content direction

### 17.10 Accessibility

- Banner images require alt text
- Drag-and-drop has keyboard alternative
- Rich text editor supports RTL content
- Preview opens with proper heading structure

---

## 18. AP-CM-002 — Category Management

### 18.1 Metadata

| Field | Value |
|-------|-------|
| Page ID | `AP-CM-002` |
| Page Title (AR) | إدارة الفئات |
| Page Title (EN) | Category Management |
| Route | `/admin/content/categories` |
| Required Role | Super Admin |
| Required Permission | super_admin only |
| Parent | AP-CM-001 |
| Last Updated | 2026-09-13 |

### 18.2 Purpose

Manage the product category hierarchy including parent/child relationships, category images, sorting, and visibility.

### 18.3 Information Architecture

```
AP-CM-002: Category Management
+-- Page Header
|   +-- Title: إدارة الفئات
|   +-- Actions: [Add Category] [Import] [Export]
+-- Category Tree View (left panel)
|   +-- Draggable tree structure
|   +-- Expand/collapse children
|   +-- Inline rename
+-- Category Detail (right panel)
|   +-- Name (AR), Name (EN)
|   +-- Image
|   +-- Parent Category
|   +-- Sort Order
|   +-- Status (Active/Inactive)
|   +-- Product Count
|   +-- Actions: [Edit] [Delete] [Add Subcategory]
```

### 18.4 Actions

| Action | Trigger | Behavior |
|--------|---------|----------|
| Add Category | Button click | Open create modal |
| Edit Category | Tree node click | Show detail panel |
| Delete Category | Tree action | Confirm → delete (only if no products) |
| Reorder Categories | Drag and drop | Update sort order |
| Add Subcategory | Button click | Open create modal with parent pre-selected |
| Toggle Status | Toggle | Activate/Deactivate category |

### 18.5 Forms — Category Create/Edit

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| Name (AR) | Text input | yes | 2-100 chars |
| Name (EN) | Text input | yes | 2-100 chars |
| Parent Category | Tree select | no | Optional (top-level if empty) |
| Image | FileUpload | no | JPG/PNG, max 1MB, 512×512 recommended |
| Icon | Icon picker | no | Platform icon set |
| Sort Order | Number | yes | Integer, ascending |
| Status | Select | yes | Active / Inactive |
| Description | Textarea | no | Max 500 chars |
| Meta Title (SEO) | Text input | no | Max 60 chars |
| Meta Description (SEO) | Textarea | no | Max 160 chars |

### 18.6 States

| State | Description | UI |
|-------|-------------|-----|
| Loading | Fetching categories | Tree skeleton |
| Loaded | Data present | Split view (tree + detail) |
| Empty | No categories | EmptyState: "لا توجد فئات بعد" |

### 18.7 RTL Considerations

- Tree indentation: RTL (children indented to left)
- Drag handle: RTL position
- Split view: tree on right, detail on left

### 18.10 Accessibility

- Tree has `role="tree"` with `role="treeitem"` for nodes
- Keyboard: Arrow keys navigate tree, Enter expands/collapses
- Drag-and-drop has keyboard alternative (cut/paste)
- Image uploads have alt text field

---

## 19. AP-CM-003 — Coupon Management

### 19.1 Metadata

| Field | Value |
|-------|-------|
| Page ID | `AP-CM-003` |
| Page Title (AR) | إدارة الكوبونات |
| Page Title (EN) | Coupon Management |
| Route | `/admin/content/coupons` |
| Required Role | Super Admin |
| Required Permission | super_admin only |
| Parent | AP-CM-001 |
| Last Updated | 2026-09-13 |

### 19.2 Purpose

Create, manage, and track promotional coupons/discount codes for the platform.

### 19.3 Information Architecture

```
AP-CM-003: Coupon Management
+-- Page Header
|   +-- Title: إدارة الكوبونات
|   +-- Stats: Active | Expired | Scheduled
|   +-- Actions: [Add Coupon] [Export]
+-- Filter Bar
|   +-- Search (code, name)
|   +-- Status (Active/Expired/Scheduled/Disabled)
|   +-- Type (Percentage/Fixed/Free Shipping)
|   +-- Date range
+-- Data Table
|   +-- Columns: Code | Type | Value | Usage | Limit | Validity | Status | Actions
|   +-- Sortable: Code, Value, Usage, Validity
+-- Pagination
```

### 19.4 Data Table Columns

| # | Column (AR) | Field | Format |
|---|------------|-------|--------|
| 1 | الكود | code | monospace, copyable |
| 2 | النوع | type | badge: نسبة / مبلغ / شحن مجاني |
| 3 | القيمة | value | percentage or YER |
| 4 | الاستخدام | usageCount | "45 / 100" (used/limit) |
| 5 | الحد الأقصى | usageLimit | integer or "غير محدود" |
| 6 | الصلاحية | validity | YYYY/MM/DD - YYYY/MM/DD |
| 7 | الحالة | status | StatusBadge |
| 8 | إجراءات | _actions | edit/delete/disable |

### 19.5 Actions

| Action | Trigger | Behavior |
|--------|---------|----------|
| Add Coupon | Button click | Open create modal |
| Edit Coupon | Row action | Open edit modal |
| Delete Coupon | Row action | Confirm (destructive) → delete |
| Disable/Enable | Row action | Toggle status |
| Copy Code | Click icon | Copy to clipboard |
| View Usage | Row click | Open usage history modal |
| Export | Header button | Download CSV |

### 19.6 Forms — Create/Edit Coupon

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| Code | Text input | yes | 4-20 chars, alphanumeric + hyphens, unique |
| Type | Radio | yes | Percentage / Fixed Amount / Free Shipping |
| Value | Number | yes | > 0, max depends on type |
| Minimum Order | Number | no | ≥ 0 YER |
| Maximum Discount | Number | conditional | Required for percentage type |
| Usage Limit | Number | no | ≥ 0 (0 = unlimited) |
| Per User Limit | Number | no | ≥ 0 (0 = unlimited) |
| Start Date | DatePicker | yes | Must be today or future |
| End Date | DatePicker | yes | Must be after start date |
| Applicable To | Radio | yes | All / Specific Categories / Specific Products / Specific Vendors |
| Target Selection | Multi-select | conditional | Required based on "Applicable To" |
| Status | Select | yes | Active / Scheduled / Disabled |

### 19.7 States

| State | Description | UI |
|-------|-------------|-----|
| Loading | Fetching coupons | Table skeleton |
| Loaded | Data present | Full table |
| Empty | No coupons | EmptyState: "لا توجد كوبونات بعد" |

### 19.8 RTL Considerations

- Coupon code: LTR (codes are always Latin characters)
- Table: RTL column order
- Value display: LTR formatted

### 19.9 Accessibility

- Copy code button has `aria-label` with code value
- Status toggles have `aria-label`
- Date pickers have proper labels
- Table sortable columns have `aria-sort`

---

## 20. AP-DL-001 — Delivery Provider Management

### 20.1 Metadata

| Field | Value |
|-------|-------|
| Page ID | `AP-DL-001` |
| Page Title (AR) | إدارة مزودي التوصيل |
| Page Title (EN) | Delivery Provider Management |
| Route | `/admin/delivery` |
| Required Role | Super Admin |
| Required Permission | super_admin only |
| Parent | Sidebar → إدارة التوصيل |
| Last Updated | 2026-09-13 |

### 20.2 Purpose

Manage delivery service providers including configuration, pricing, coverage areas, and API integration settings.

### 20.3 Information Architecture

```
AP-DL-001: Delivery Provider Management
+-- Page Header
|   +-- Title: إدارة مزودي التوصيل
|   +-- Actions: [Add Provider]
+-- Provider List (card grid)
|   +-- Per Provider Card:
|       +-- Logo/Name
|       +-- Status (Active/Inactive)
|       +-- Coverage Areas
|       +-- Average Delivery Time
|       +-- Rating
|       +-- Active Orders Count
|       +-- Actions: [Configure] [Toggle] [View Stats]
+-- Provider Configuration Drawer
|   +-- General Info (name, logo, contact)
|   +-- API Settings (endpoint, API key, webhook URL)
|   +-- Pricing Rules (base fee, per km, per kg)
|   +-- Coverage Areas (governorate selection)
|   +-- Operating Hours
|   +-- Status Toggle
```

### 20.4 Provider Card Data

| Property | Source | Display |
|----------|--------|---------|
| Logo | provider.logo | 64×64 image |
| Name | provider.name | Bold text |
| Status | provider.status | StatusBadge |
| Coverage | provider.coverageAreas | "3 محافظات" |
| Avg Delivery | provider.avgDeliveryTime | "2.5 ساعة" |
| Rating | provider.rating | Star rating + "4.5/5" |
| Active Orders | provider.activeOrders | Count |

### 20.5 Actions

| Action | Trigger | Behavior |
|--------|---------|----------|
| Add Provider | Button click | Open create modal |
| Configure Provider | Card button | Open configuration drawer |
| Toggle Status | Toggle | Activate/Deactivate provider |
| View Stats | Card button | Open stats modal (delivery time, success rate, etc.) |
| Test API | Drawer button | Send test request to provider API |
| Delete Provider | Drawer action | Confirm (destructive) → delete |

### 20.6 Forms — Provider Configuration

| Section | Fields |
|---------|--------|
| General | Name (AR/EN), Logo, Contact Person, Phone, Email |
| API | Endpoint URL, API Key, Webhook URL, Test Button |
| Pricing | Base Fee (YER), Per KM Rate, Per KG Rate, Minimum Fee |
| Coverage | Governorate multi-select, District level (optional) |
| Schedule | Operating hours (day/time ranges), Holiday dates |
| Status | Active/Inactive toggle |

### 20.7 States

| State | Description | UI |
|-------|-------------|-----|
| Loading | Fetching providers | Card skeleton grid |
| Loaded | Data present | Card grid |
| Empty | No providers | EmptyState: "لا يوجد مزودو توصيل بعد" |

### 20.8 RTL Considerations

- Card grid: RTL flow
- Configuration drawer: slides from left in RTL
- Pricing inputs: LTR formatted

### 20.9 Accessibility

- Provider cards have `role="article"` with heading
- Status toggles have `aria-label`
- API test button has loading state announcement
- Coverage selection has proper grouping

---

## 21. AP-AN-001 — Analytics Dashboard

### 21.1 Metadata

| Field | Value |
|-------|-------|
| Page ID | `AP-AN-001` |
| Page Title (AR) | لوحة التحليلات |
| Page Title (EN) | Analytics Dashboard |
| Route | `/admin/analytics` |
| Required Role | Admin |
| Required Permission | `reports:read` |
| Parent | Sidebar → التحليلات |
| Children | AP-AN-002 (Reports) |
| Last Updated | 2026-09-13 |

### 21.2 Purpose

Provide advanced analytics and insights beyond the basic dashboard, including user behavior, conversion funnels, vendor performance, and product analytics.

### 21.3 Information Architecture

```
AP-AN-001: Analytics Dashboard
+-- Page Header
|   +-- Title: لوحة التحليلات
|   +-- Date Range Picker
|   +-- Export Report Button
+-- KPI Row (6 cards)
|   +-- GMV (Gross Merchandise Value)
|   +-- Conversion Rate
|   +-- Average Order Value
|   +-- Customer Acquisition Cost
|   +-- Customer Lifetime Value
|   +-- Return Rate
+-- Charts Grid (3-column)
|   +-- Revenue Trend (area chart, 30/60/90 days)
|   +-- Orders by Governorate (map visualization)
|   +-- User Growth (line chart)
|   +-- Top Categories by Revenue (bar chart)
|   +-- Conversion Funnel (funnel chart)
|   +-- Peak Hours Heatmap (hourly order distribution)
+-- Vendor Performance Section
|   +-- Top Vendors by Revenue
|   +-- Vendor Growth Trend
|   +-- Vendor Rating Distribution
+-- Product Insights
|   +-- Best Sellers
|   +-- Trending Products
|   +-- Low Stock Alert Products
```

### 21.4 KPI Cards

| KPI | Value Source | Format | Delta |
|-----|-------------|--------|-------|
| GMV | totalGMV | YER formatted | vs previous period |
| Conversion Rate | conversionRate | percentage | vs previous period |
| Average Order Value | aov | YER formatted | vs previous period |
| Customer Acquisition Cost | cac | YER formatted | vs previous period |
| Customer Lifetime Value | clv | YER formatted | vs previous period |
| Return Rate | returnRate | percentage | vs previous period |

### 21.5 Charts

| Chart | Type | Data | Controls |
|-------|------|------|----------|
| Revenue Trend | Area chart | Daily revenue | 30/60/90 day toggle |
| Orders by Governorate | Map | Order count per region | Click region for detail |
| User Growth | Line chart | New users per day | Cumulative toggle |
| Top Categories | Horizontal bar | Revenue per category | Top 10 |
| Conversion Funnel | Funnel | Visit → Cart → Checkout → Order | Percentage labels |
| Peak Hours Heatmap | Heatmap | Orders by hour × day | Color intensity |

### 21.6 Data Requirements

| Data Point | Source API | Refresh |
|------------|-----------|---------|
| KPI Summary | `GET /admin/analytics/kpi` | 600s |
| Revenue Trend | `GET /admin/analytics/revenue-trend` | 600s |
| Orders by Region | `GET /admin/analytics/by-region` | 900s |
| User Growth | `GET /admin/analytics/user-growth` | 600s |
| Category Performance | `GET /admin/analytics/categories` | 900s |
| Conversion Funnel | `GET /admin/analytics/funnel` | 900s |
| Peak Hours | `GET /admin/analytics/peak-hours` | 3600s |
| Vendor Performance | `GET /admin/analytics/vendors` | 900s |
| Product Insights | `GET /admin/analytics/products` | 900s |

### 21.7 Actions

| Action | Trigger | Behavior |
|--------|---------|----------|
| Change Date Range | Picker | Update all analytics for period |
| Export Report | Button | Generate PDF/Excel report |
| Click Chart Segment | Click | Drill-down to detail view |
| Click Vendor | Table link | Navigate to vendor detail |
| Click Product | Table link | Navigate to product detail |
| Click Region | Map click | Filter by governorate |

### 21.8 States

| State | Description | UI |
|-------|-------------|-----|
| Loading | Fetching analytics | Skeleton grid |
| Loaded | Data present | Full dashboard |
| Error | API failure | ErrorState + retry |
| No Data | Period has no data | EmptyState with suggestion to change range |

### 21.9 Empty States

| Scenario | Title (AR) |
|----------|-----------|
| No data for period | "لا توجد بيانات لهذه الفترة" |
| No conversion data | "لا توجد بيانات تحويل" |

### 21.10 RTL Considerations

- Charts: X-axis RTL
- Map: SVG mirrored if needed (geographic accuracy may override)
- Heatmap: RTL hour labels
- Tables: RTL column order

### 21.11 Accessibility

- All charts have accessible table fallback
- KPI cards have `aria-label` with full context
- Map has text alternative for screen readers
- Funnel chart labels are accessible
- Color intensity in heatmap supplemented with text

---

## 22. AP-AN-002 — Reports

### 22.1 Metadata

| Field | Value |
|-------|-------|
| Page ID | `AP-AN-002` |
| Page Title (AR) | التقارير |
| Page Title (EN) | Reports |
| Route | `/admin/analytics/reports` |
| Required Role | Admin |
| Required Permission | `reports:read` |
| Parent | AP-AN-001 |
| Last Updated | 2026-09-13 |

### 22.2 Purpose

Generate, schedule, and download various platform reports including sales, financial, vendor, and operational reports.

### 22.3 Information Architecture

```
AP-AN-002: Reports
+-- Page Header
|   +-- Title: التقارير
|   +-- Actions: [Schedule Report]
+-- Report Categories (tabs)
|   +-- مبيعات (Sales)
|   |   +-- Daily Sales Report
|   |   +-- Sales by Category
|   |   +-- Sales by Vendor
|   |   +-- Sales by Region
|   +-- مالية (Financial)
|   |   +-- Revenue Report
|   |   +-- Commission Report
|   |   +-- Tax Report
|   |   +-- Payout Report
|   +-- عمليات (Operations)
|   |   +-- Order Fulfillment Report
|   |   +-- Delivery Performance Report
|   |   +-- Return/Refund Report
|   +-- مستخدمين (Users)
|       +-- User Registration Report
|       +-- User Activity Report
|       +-- Vendor Performance Report
+-- Report Builder (per report)
|   +-- Parameters (date range, filters)
|   +-- Preview
|   +-- Generate / Schedule / Download
+-- Generated Reports History
|   +-- Table: Report Name | Generated | Format | Size | Actions
```

### 22.4 Report Parameters

| Parameter | Type | Applies To |
|-----------|------|-----------|
| Date Range | Date range picker | All reports |
| Category | Multi-select | Sales, Product reports |
| Vendor | Search select | Sales, Vendor reports |
| Governorate | Multi-select | Sales, Delivery reports |
| Order Status | Multi-select | Order reports |
| Format | Radio | All (CSV, Excel, PDF) |

### 22.5 Actions

| Action | Trigger | Behavior |
|--------|---------|----------|
| Select Report | Tab + click | Show report builder |
| Set Parameters | Form interaction | Update preview |
| Generate Report | Button click | Generate and download |
| Schedule Report | Button click | Open schedule modal |
| Download Previous | Table row click | Download generated file |
| Delete Previous | Row action | Confirm → delete |

### 22.6 Forms — Schedule Report Modal

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| Frequency | Select | yes | Daily / Weekly / Monthly |
| Day/Time | DatePicker/TimePicker | yes | Valid future date/time |
| Format | Radio | yes | CSV / Excel / PDF |
| Recipients | Email input | yes | Comma-separated emails |
| Parameters | Hidden | — | Inherited from current report builder |

### 22.7 States

| State | Description | UI |
|-------|-------------|-----|
| Loading | Fetching report list | Table skeleton |
| Loaded | Data present | Full page with tabs |
| Empty | No generated reports | EmptyState: "لا توجد تقارير مولدة" |
| Generating | Report being generated | Progress indicator |

### 22.8 RTL Considerations

- Report preview: RTL text
- Table: RTL column order
- Tabs: RTL flow

### 22.9 Accessibility

- Report builder has proper form labels
- Download buttons have `aria-label` with report name and format
- Generated report table has proper headers
- Schedule modal has focus trap

---

## 23. AP-SY-001 — System Configuration

### 23.1 Metadata

| Field | Value |
|-------|-------|
| Page ID | `AP-SY-001` |
| Page Title (AR) | إعدادات النظام |
| Page Title (EN) | System Configuration |
| Route | `/admin/system` |
| Required Role | Super Admin |
| Required Permission | super_admin only |
| Parent | Sidebar → النظام |
| Children | AP-SY-002 (Audit Log) |
| Last Updated | 2026-09-13 |

### 23.2 Purpose

Configure platform-wide settings including general settings, payment configuration, notification settings, email templates, and maintenance mode.

### 23.3 Information Architecture

```
AP-SY-001: System Configuration
+-- Page Header
|   +-- Title: إعدادات النظام
|   +-- Actions: [Save All] [Reset to Defaults]
+-- Settings Sections (vertical tabs or accordion)
|   +-- عام (General)
|   |   +-- Platform Name (AR/EN)
|   |   +-- Platform Logo
|   |   +-- Contact Email
|   |   +-- Support Phone
|   |   +-- Default Language
|   |   +-- Timezone
|   |   +-- Currency (YER/USD)
|   +-- الدفع (Payment)
|   |   +-- Wallet System Enabled (toggle)
|   |   +-- Minimum Top-up Amount
|   |   +── Maximum Wallet Balance
|   |   +-- Auto-deduct Commission (toggle)
|   +-- الإشعارات (Notifications)
|   |   +-- Email Notifications Enabled (toggle)
|   |   +-- SMS Notifications Enabled (toggle)
|   |   +-- Push Notifications Enabled (toggle)
|   |   +-- Admin Alert Emails
|   +-- التوصيل (Delivery)
|   |   +-- Default Delivery Fee
|   |   +-- Free Delivery Threshold
|   |   +-- Delivery Radius (km)
|   +-- الصيانة (Maintenance)
|       +-- Maintenance Mode (toggle)
|       +-- Maintenance Message (AR/EN)
|       +-- Allowed IPs (for admin access during maintenance)
```

### 23.4 Actions

| Action | Trigger | Behavior |
|--------|---------|----------|
| Save Section | Button click | Save current section settings |
| Save All | Header button | Save all modified settings |
| Reset to Defaults | Header button | Confirm → reset all settings |
| Toggle Maintenance | Toggle | Confirm (destructive) → enable/disable |
| Upload Logo | FileUpload | Upload new platform logo |
| Test Email | Button | Send test email to verify configuration |
| Test SMS | Button | Send test SMS to verify configuration |

### 23.5 Forms — General Settings

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| Platform Name (AR) | Text input | yes | 2-100 chars |
| Platform Name (EN) | Text input | yes | 2-100 chars |
| Logo | FileUpload | no | SVG/PNG, max 2MB |
| Contact Email | Email input | yes | Valid email |
| Support Phone | Tel input | yes | +967XXXXXXXXX |
| Default Language | Select | yes | Arabic / English |
| Timezone | Select | yes | Asia/Aden |
| Currency | Select | yes | YER / USD |

### 23.6 States

| State | Description | UI |
|-------|-------------|-----|
| Loading | Fetching settings | Form skeleton |
| Loaded | Data present | Full settings form |
| Saving | Saving changes | Button loading state |
| Saved | Changes saved | Success toast |
| Error | Save failed | Error toast + retry |

### 23.7 RTL Considerations

- Settings form: RTL field layout
- Toggle labels: RTL alignment
- Upload preview: RTL context

### 23.8 Accessibility

- All form fields have associated `<label>` elements
- Toggle switches have `aria-label`
- Save buttons have loading state announcements
- Maintenance mode toggle has confirmation dialog
- Form validation errors linked to fields via `aria-describedby`

---

## 24. AP-SY-002 — Audit Log Viewer

### 24.1 Metadata

| Field | Value |
|-------|-------|
| Page ID | `AP-SY-002` |
| Page Title (AR) | سجل التدقيق |
| Page Title (EN) | Audit Log Viewer |
| Route | `/admin/system/audit` |
| Required Role | Super Admin |
| Required Permission | super_admin only |
| Parent | AP-SY-001 |
| Last Updated | 2026-09-13 |

### 24.2 Purpose

View a comprehensive audit trail of all admin actions on the platform for security, compliance, and troubleshooting purposes.

### 24.3 Information Architecture

```
AP-SY-002: Audit Log Viewer
+-- Page Header
|   +-- Title: سجل التدقيق
|   +-- Actions: [Export] [Purge Old Logs]
+-- Filter Bar
|   +-- Search (admin name, action, entity)
|   +-- Admin (multi-select)
|   +-- Action Type (Create/Update/Delete/Login/Logout)
|   +-- Entity Type (User/Vendor/Product/Order/System)
|   +-- Date range
|   +-- IP Address
+-- Log Table
|   +-- Columns: Timestamp | Admin | Action | Entity | Details | IP | User Agent
|   +-- Row click → Detail modal
+-- Pagination (50 per page default)
```

### 24.4 Data Table Columns

| # | Column (AR) | Field | Format |
|---|------------|-------|--------|
| 1 | الوقت | timestamp | YYYY/MM/DD HH:mm:ss |
| 2 | المدير | adminName | text + avatar |
| 3 | الإجراء | action | StatusBadge (color-coded) |
| 4 | الكيان | entityType | Badge + entity name |
| 5 | التفاصيل | details | Truncated text, expandable |
| 6 | عنوان IP | ipAddress | Monospace |
| 7 | متصفح الويب | userAgent | Truncated, tooltip on hover |

### 24.5 Action Type Badges

| Action (AR) | Action (EN) | Color |
|-------------|-------------|-------|
| إنشاء | Create | green |
| تعديل | Update | blue |
| حذف | Delete | red |
| دخول | Login | teal |
| خروج | Logout | gray |
| محاولة فاشلة | Failed Attempt | red-700 |

### 24.6 Actions

| Action | Trigger | Behavior |
|--------|---------|----------|
| View Detail | Row click | Open detail modal with full info |
| Export | Header button | Download CSV of filtered logs |
| Purge Old Logs | Header button | Confirm + date → delete logs older than |
| Filter by Admin | Click admin name | Apply admin filter |
| Filter by Action | Click action badge | Apply action filter |

### 24.7 Log Detail Modal

```
+------------------------------------------------------------------+
|  تفاصيل العملية                                            [X]  |
+------------------------------------------------------------------+
|  الوقت: 2026/09/13 14:30:25                                      |
|  المدير: أحمد محمد (admin@yemenmart.com)                          |
|  الإجراء: تعديل                                                   |
|  الكيان: مستخدم #1234                                             |
|  عنوان IP: 192.168.1.100                                          |
|  المتصفح: Chrome 120 / Windows 11                                 |
|                                                                   |
|  +-------------------------------------------------------------+ |
|  | تغييرات                                                      | |
|  | الحقل          | من                 | إلى                   | |
|  | الاسم          | أحمد علي          | أحمد محمد علي          | |
|  | الحالة         | نشط               | معلّق                  | |
|  +-------------------------------------------------------------+ |
+------------------------------------------------------------------+
```

### 24.8 States

| State | Description | UI |
|-------|-------------|-----|
| Loading | Fetching logs | Table skeleton |
| Loaded | Data present | Full table |
| Empty | No logs match filters | EmptyState: "لا توجد سجلات تطابق الفلاتر" |
| Error | API failure | ErrorState + retry |

### 24.9 RTL Considerations

- Log table: RTL column order
- Timestamp: LTR (standard format)
- IP addresses: LTR (standard format)
- User agent: LTR (standard format)

### 24.10 Accessibility

- Log table has proper `<th>` with `scope="col"`
- Detail modal has focus trap
- Timestamp has `aria-label` with relative time
- Action badges have `aria-label` with full action text

---

## 25. AP-SU-001 — Support Ticket Management

### 25.1 Metadata

| Field | Value |
|-------|-------|
| Page ID | `AP-SU-001` |
| Page Title (AR) | إدارة تذاكر الدعم |
| Page Title (EN) | Support Ticket Management |
| Route | `/admin/support` |
| Required Role | Admin |
| Required Permission | `support:read` |
| Parent | Sidebar → الدعم الفني |
| Children | AP-SU-002 (Ticket Detail) |
| Last Updated | 2026-09-13 |

### 25.2 Purpose

Manage customer and vendor support tickets, assign to agents, track resolution, and monitor support performance.

### 25.3 Entry / Exit Points

| Direction | Source / Destination | Trigger |
|-----------|---------------------|---------|
| Entry | Sidebar "الدعم الفني" | Navigation |
| Entry | AP-DB-001 open tickets badge | Click open count |
| Exit | Click ticket row | Navigate to AP-SU-002 |
| Exit | Sidebar navigation | Navigate to other pages |

### 25.4 Information Architecture

```
AP-SU-001: Support Ticket Management
+-- Page Header
|   +-- Title: إدارة تذاكر الدعم
|   +-- Stats: Open | In Progress | Escalated | Resolved Today
|   +-- Actions: [Export] [Assign Agent]
+-- Filter Bar
|   +-- Search (ticket ID, customer name, subject)
|   +-- Status (Open/In Progress/Escalated/Resolved/Closed)
|   +-- Priority (Low/Medium/High/Urgent)
|   +-- Category (Order Issue/Product Issue/Account/Payment/Other)
|   +-- Assigned Agent
|   +-- Date range
+-- Data Table
|   +-- Columns: ID | Subject | Customer | Priority | Status | Agent | Created | Last Reply | Actions
|   +-- Sortable: ID, Priority, Created, Last Reply
|   +-- Row color: priority-based (red for Urgent)
|   +-- Row click → AP-SU-002
+-- Bulk Actions
|   +-- Assign Selected
|   +-- Change Status
|   +-- Export
+-- Pagination
```

### 25.5 Data Table Columns

| # | Column (AR) | Field | Format |
|---|------------|-------|--------|
| 1 | رقم التذكرة | ticketId | #TKT-XXXX, clickable |
| 2 | الموضوع | subject | text, truncated |
| 3 | العميل | customerName | text + type badge (user/vendor) |
| 4 | الأولوية | priority | StatusBadge (color-coded) |
| 5 | الحالة | status | StatusBadge |
| 6 | الوكيل | agentName | text or "غير مسند" |
| 7 | تاريخ الإنشاء | createdAt | YYYY/MM/DD HH:mm |
| 8 | آخر رد | lastReplyAt | Relative time (e.g., "منذ 2 ساعة") |
| 9 | إجراءات | _actions | view/assign |

### 25.6 Priority Badges

| Priority (AR) | Priority (EN) | Color | Description |
|--------------|---------------|-------|-------------|
| منخفضة | Low | gray | — |
| متوسطة | Medium | blue | — |
| عالية | High | amber | — |
| عاجلة | Urgent | red | Pulsing indicator |

### 25.7 Filters

| Filter | Type | Behavior |
|--------|------|----------|
| Search | Text input | Searches ticket ID, customer name, subject |
| Status | Multi-select | Open, In Progress, Escalated, Resolved, Closed |
| Priority | Multi-select | Low, Medium, High, Urgent |
| Category | Multi-select | Order Issue, Product Issue, Account, Payment, Other |
| Assigned Agent | Search select | Filter by assigned support agent |
| Date Range | Date range | Ticket creation date |

### 25.8 Actions

| Action | Trigger | Behavior | Permission |
|--------|---------|----------|------------|
| View Ticket | Row click | Navigate to AP-SU-002 | `support:read` |
| Assign Agent | Row action / bulk | Assign modal | `support:update` |
| Change Priority | Row action | Priority select | `support:update` |
| Change Status | Row action | Status select | `support:update` |
| Escalate | Row action | Confirm → escalate to supervisor | `support:update` |
| Export | Header button | Download filtered CSV | `support:read` |

### 25.9 States

| State | Description | UI |
|-------|-------------|-----|
| Loading | Fetching tickets | Table skeleton |
| Loaded | Data present | Full table |
| Empty | No tickets match | EmptyState: "لا توجد تذاكر" |
| Error | API failure | ErrorState + retry |

### 25.10 Empty States

| Scenario | Title (AR) |
|----------|-----------|
| No tickets | "لا توجد تذاكر دعم" |
| No tickets for filter | "لا توجد تذاكر تطابق الفلاتر" |
| All tickets resolved | "تم حل جميع التذاكر" |

### 25.11 RTL Considerations

- Table: ID rightmost, actions leftmost
- Subject text: RTL alignment
- Relative time: RTL context

### 25.12 Accessibility

- Ticket ID is a link with `aria-label`
- Priority badges have `aria-label` with priority level
- Status badges have `aria-label`
- Table sortable columns have `aria-sort`
- Keyboard: Enter on row opens ticket

---

## 26. AP-SU-002 — Ticket Detail

### 26.1 Metadata

| Field | Value |
|-------|-------|
| Page ID | `AP-SU-002` |
| Page Title (AR) | تفاصيل التذكرة |
| Page Title (EN) | Ticket Detail |
| Route | `/admin/support/:ticketId` |
| Required Role | Admin |
| Required Permission | `support:read` |
| Parent | AP-SU-001 |
| Last Updated | 2026-09-13 |

### 26.2 Purpose

View and respond to support tickets, manage ticket status, assign agents, and track resolution history.

### 26.3 Entry / Exit Points

| Direction | Source / Destination | Trigger |
|-----------|---------------------|---------|
| Entry | AP-SU-001 row click | Navigate to ticket detail |
| Exit | Back / breadcrumb | Return to AP-SU-001 |
| Exit | Customer link | Navigate to AP-US-002 |
| Exit | Vendor link | Navigate to vendor detail |
| Exit | Order link | Navigate to AP-OR-002 |

### 26.4 Information Architecture

```
AP-SU-002: Ticket Detail
+-- Header
|   +-- Ticket ID + Subject
|   +-- Status Badge + Priority Badge
|   +-- Actions: [Assign] [Change Status] [Escalate] [Close]
+-- Ticket Info Panel
|   +-- Customer info (name, type, contact)
|   +-- Category
|   +-- Related Order (if any, clickable)
|   +-- Assigned Agent
|   +-- Created / Last Updated
+-- Conversation Thread
|   +-- Customer messages (right-aligned in RTL)
|   +-- Agent responses (left-aligned in RTL)
|   +-- System messages (centered)
|   +-- Each message: sender, timestamp, content, attachments
+-- Reply Section
|   +-- Text editor (rich text)
|   +-- Attachments
|   +-- Internal Note toggle
|   +-- Send Button
+-- Ticket Metadata
|   +-- Resolution time (if resolved)
|   +-- Customer satisfaction rating (if provided)
|   +-- Related tickets
```

### 26.5 Conversation Thread Component

| Property | Specification |
|----------|---------------|
| Customer messages | Right-aligned in RTL, green bubble |
| Agent responses | Left-aligned in RTL, blue bubble |
| System messages | Centered, gray, smaller font |
| Timestamps | Relative (e.g., "منذ ساعتين") + absolute on hover |
| Attachments | Clickable thumbnails or file icons |
| Auto-scroll | Scroll to newest message on load |

### 26.6 Data Requirements

| Data Point | Source API | Refresh |
|------------|-----------|---------|
| Ticket Details | `GET /admin/support/:id` | on load |
| Messages | `GET /admin/support/:id/messages` | on load + WebSocket |
| Related Entity | Nested in ticket response | — |
| Customer Info | Nested in ticket response | — |

### 26.7 Actions

| Action | Trigger | Behavior | Permission |
|--------|---------|----------|------------|
| Reply | Send button | Post agent response | `support:update` |
| Add Internal Note | Toggle + send | Post internal note (not visible to customer) | `support:update` |
| Change Status | Button click | Status select | `support:update` |
| Change Priority | Button click | Priority select | `support:update` |
| Assign Agent | Button click | Agent select modal | `support:update` |
| Escalate | Button click | Confirm → escalate | `support:update` |
| Close Ticket | Button click | Confirm → close | `support:update` |
| View Order | Link click | Navigate to AP-OR-002 | `orders:read` |
| View Customer | Link click | Navigate to AP-US-002 | `users:read` |
| Attach File | Button click | File upload | `support:update` |

### 26.8 Forms — Reply

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| Message | Rich text editor | yes | Min 1 char, max 5000 chars |
| Attachments | FileUpload | no | Max 5 files, 10MB each |
| Internal Note | Toggle | — | Default: off |

### 26.9 States

| State | Description | UI |
|-------|-------------|-----|
| Loading | Fetching ticket | Skeleton layout |
| Loaded | Data present | Full conversation view |
| Sending | Sending reply | Button loading state |
| Not Found | Ticket doesn't exist | EmptyState: "التذكرة غير موجودة" |

### 26.10 Empty States

| Scenario | Title (AR) |
|----------|-----------|
| No messages | "لا توجد رسائل بعد" |
| No attachments | "لا توجد مرفقات" |

### 26.11 RTL Considerations

- Conversation: customer messages on right, agent on left
- Text editor: RTL content direction
- File attachments: RTL layout
- Metadata panel: RTL alignment

### 26.12 Accessibility

- Conversation has `role="log"` with `aria-live="polite"`
- Each message has proper heading/label
- Rich text editor has `aria-label`
- File upload has proper labeling
- Reply button has loading state announcement
- Close ticket has confirmation dialog

---

## 27. AP-NT-001 — Notification Management

### 27.1 Metadata

| Field | Value |
|-------|-------|
| Page ID | `AP-NT-001` |
| Page Title (AR) | إدارة الإشعارات |
| Page Title (EN) | Notification Management |
| Route | `/admin/notifications` |
| Required Role | Any authenticated admin |
| Required Permission | — (all authenticated admins) |
| Parent | Sidebar → الإشعارات |
| Last Updated | 2026-09-13 |

### 27.2 Purpose

View, manage, and send platform notifications to users and vendors. Manage notification templates and settings.

### 27.3 Information Architecture

```
AP-NT-001: Notification Management
+-- Page Header
|   +-- Title: إدارة الإشعارات
|   +-- Tabs: [الإشعارات الواردة] [إرسال إشعار] [القوالب] [الإعدادات]
+-- Inbox Tab
|   +-- Notification List
|   |   +-- Columns: Icon | Title | Body | Read | Time | Actions
|   +-- Filters: Read/Unread, Type, Date
|   +-- Actions: [Mark All Read] [Delete Selected]
+-- Compose Tab
|   +-- Send Notification Form
|   |   +-- Recipients (All / Specific Users / Specific Vendors / Segment)
|   |   +-- Title (AR/EN)
|   |   +-- Body (AR/EN)
|   |   +-- Type (Info/Warning/Promotional/System)
|   |   +-- Link (optional)
|   |   +-- Schedule (Now / Later)
|   +-- Preview
|   +-- Send Button
+-- Templates Tab
|   +-- Template List
|   +-- Actions: [Add Template] [Edit] [Delete]
+-- Settings Tab
|   +-- Email notification templates
|   +-- SMS notification templates
|   +-- Push notification settings
```

### 27.4 Notification List Columns

| # | Column (AR) | Field | Format |
|---|------------|-------|--------|
| 1 | الأيقونة | icon | Type-based icon |
| 2 | العنوان | title | Bold if unread |
| 3 | المحتوى | body | Truncated text |
| 4 | مقروء | isRead | Dot indicator (unread = filled) |
| 5 | الوقت | timestamp | Relative time |
| 6 | إجراءات | _actions | mark read/delete |

### 27.5 Actions

| Action | Trigger | Behavior |
|--------|---------|----------|
| Mark as Read | Row action / click | Mark notification as read |
| Mark All Read | Header button | Mark all as read |
| Delete Notification | Row action | Confirm → delete |
| Delete Selected | Bulk action | Confirm → delete selected |
| Compose Notification | Tab click | Switch to compose tab |
| Send Notification | Button click | Confirm → send |
| Schedule Notification | Toggle + date picker | Schedule for later |

### 27.6 Forms — Compose Notification

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| Recipients | Radio + conditional select | yes | All / Specific Users / Specific Vendors / Segment |
| User/Vendor Selection | Multi-select | conditional | Required if "Specific" selected |
| Title (AR) | Text input | yes | 1-100 chars |
| Title (EN) | Text input | yes | 1-100 chars |
| Body (AR) | Textarea | yes | 1-500 chars |
| Body (EN) | Textarea | yes | 1-500 chars |
| Type | Select | yes | Info / Warning / Promotional / System |
| Link URL | URL input | no | Valid URL |
| Schedule | Radio | yes | Now / Later |
| Schedule Date | DatePicker | conditional | Required if "Later" |
| Schedule Time | TimePicker | conditional | Required if "Later" |

### 27.7 States

| State | Description | UI |
|-------|-------------|-----|
| Loading | Fetching notifications | List skeleton |
| Loaded | Data present | Full page with tabs |
| Empty | No notifications | EmptyState: "لا توجد إشعارات" |
| Sending | Sending notification | Button loading state |
| Sent | Notification sent | Success toast |

### 27.8 Empty States

| Scenario | Title (AR) |
|----------|-----------|
| No notifications | "لا توجد إشعارات" |
| No unread notifications | "لا توجد إشعارات غير مقروءة" |
| No templates | "لا توجد قوالب إشعارات" |

### 27.9 RTL Considerations

- Notification list: icon on right, text on left
- Compose form: RTL field layout
- Preview: RTL rendering
- Tabs: RTL flow

### 27.10 Accessibility

- Notification list items have `role="article"`
- Unread notifications have `aria-label` with "غير مقروء" prefix
- Compose form has proper labels
- Send button has loading state announcement
- Mark all read has confirmation feedback

---

## 28. AP-PR-003 — Admin Profile

### 28.1 Metadata

| Field | Value |
|-------|-------|
| Page ID | `AP-PR-003` |
| Page Title (AR) | الملف الشخصي للمدير |
| Page Title (EN) | Admin Profile |
| Route | `/admin/profile` |
| Required Role | Any authenticated admin |
| Required Permission | — (all authenticated admins) |
| Parent | Sidebar → الملف الشخصي |
| Last Updated | 2026-09-13 |

### 28.2 Purpose

Allow administrators to view and manage their own profile, change password, manage 2FA, and view their activity history.

### 28.3 Entry / Exit Points

| Direction | Source / Destination | Trigger |
|-----------|---------------------|---------|
| Entry | Sidebar "الملف الشخصي" | Navigation |
| Entry | Top bar avatar click → Profile | Navigation |
| Exit | Sidebar navigation | Navigate to other pages |

### 28.4 Information Architecture

```
AP-PR-003: Admin Profile
+-- Profile Header
|   +-- Avatar (large, editable)
|   +-- Name + Role Badge
|   +-- Email + Phone
|   +-- Last Login info
+-- Tabs
|   +-- الملف الشخصي (Profile)
|   |   +-- Personal Info Form
|   |   |   +-- Name (AR/EN)
|   |   |   +-- Email
|   |   |   +-- Phone
|   |   |   +-- Avatar upload
|   |   +-- Save Button
|   +-- الأمان (Security)
|   |   +-- Change Password Form
|   |   |   +-- Current Password
|   |   |   +-- New Password
|   |   |   +-- Confirm New Password
|   |   +-- Two-Factor Authentication
|   |   |   +-- Enable/Disable 2FA toggle
|   |   |   +-- QR Code for authenticator app
|   |   |   +-- Backup codes
|   |   +-- Active Sessions
|   |       +-- List of active sessions (device, IP, last active)
|   |       +-- Revoke session button
|   +-- النشاط (Activity)
|       +-- Recent admin actions log
|       +-- Login history
|       +-- Actions performed
```

### 28.5 Forms — Personal Info

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| Name (AR) | Text input | yes | 2-100 chars |
| Name (EN) | Text input | yes | 2-100 chars |
| Email | Email input | yes | Valid email, unique |
| Phone | Tel input | yes | +967XXXXXXXXX format |
| Avatar | FileUpload | no | JPG/PNG, max 2MB, 512×512 recommended |

### 28.6 Forms — Change Password

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| Current Password | Password input | yes | Must match current |
| New Password | Password input | yes | Min 8 chars, 1 uppercase, 1 lowercase, 1 number, 1 special |
| Confirm Password | Password input | yes | Must match new password |

### 28.7 Forms — Two-Factor Authentication

| Step | UI |
|------|-----|
| 1. Enable | Toggle switch |
| 2. QR Code | Display QR for authenticator app scan |
| 3. Verification | Enter 6-digit code from app |
| 4. Backup Codes | Display one-time backup codes (copyable) |
| 5. Confirmation | Success message + 2FA enabled status |

### 28.8 Sessions Table

| # | Column | Format |
|---|--------|--------|
| 1 | الجهاز | Device type + browser |
| 2 | عنوان IP | IP address |
| 3 | آخر نشاط | Relative time |
| 4 | إجراءات | [إلغاء] button |

### 28.9 Actions

| Action | Trigger | Behavior |
|--------|---------|----------|
| Update Profile | Save button | Validate → save → success toast |
| Change Password | Save button | Validate → change → success toast → logout other sessions |
| Enable 2FA | Toggle + QR + code | Verify → enable → show backup codes |
| Disable 2FA | Toggle | Confirm → disable |
| Revoke Session | Button click | Confirm → revoke session |
| Upload Avatar | FileUpload | Crop → upload → update preview |
| Regenerate Backup Codes | Button click | Confirm → new codes displayed |

### 28.10 States

| State | Description | UI |
|-------|-------------|-----|
| Loading | Fetching profile | Skeleton layout |
| Loaded | Data present | Full profile page |
| Saving | Saving changes | Button loading state |
| Saved | Changes saved | Success toast |
| Error | Save failed | Error toast + retry |

### 28.11 Empty States

| Scenario | Title (AR) |
|----------|-----------|
| No activity log | "لا توجد سجلات نشاط" |
| No active sessions | "لا توجد جلسات نشطة" |
| No 2FA backup codes | "لا توجد أكواد احتياطية" |

### 28.12 RTL Considerations

- Profile header: avatar on right
- Forms: RTL field layout
- Tabs: RTL flow
- Session table: RTL column order

### 28.13 Accessibility

- All form fields have associated `<label>` elements
- Password fields have `aria-describedby` for requirements
- 2FA QR code has `aria-label`
- Backup codes are copyable with `aria-label`
- Session revoke has confirmation dialog
- Avatar upload has file input with proper labeling
- Save buttons have loading state announcements

---

**END OF FILE — 05-PORTAL-ADMIN-PANEL-01.md**
