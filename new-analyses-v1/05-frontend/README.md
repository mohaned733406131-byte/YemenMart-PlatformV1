# 05 - Frontend

**Category:** Frontend  
**Purpose:** UI/UX implementation for all user-facing applications

---

## Contents

- `customer-storefront.md` - Next.js customer-facing web application
- `vendor-panel.md` - React vendor management dashboard
- `admin-panel.md` - React admin control panel
- `mobile-apps.md` - React Native iOS/Android apps
- `component-library.md` - Shared UI component catalog
- `state-management.md` - Redux/Zustand patterns
- `responsive-design.md` - Mobile-first responsive approach

---

## Applications

### Customer Storefront (Port 3000)
- **Framework:** Next.js 15 + React 19
- **Styling:** Tailwind CSS
- **Features:** Product browsing, cart, checkout, account, reviews
- **Language:** Arabic RTL primary, English LTR secondary
- **Auth:** SMS OTP + JWT

### Vendor Panel (Port 7002)
- **Framework:** React 18 + Vite
- **Styling:** Tailwind CSS
- **Features:** Store management, products, orders, inventory, analytics
- **Auth:** SMS OTP + JWT

### Admin Panel (Port 7001)
- **Framework:** React 18 + Vite
- **Styling:** Tailwind CSS
- **Features:** Platform oversight, vendor approval, finance, support, CMS
- **Auth:** SMS OTP + JWT + MFA

### Mobile Apps
- **Framework:** React Native
- **Platforms:** iOS 14+, Android 9+
- **Features:** Full customer and vendor experiences on mobile

---

## Design Principles

1. **Arabic-First** - RTL layout, Arabic typography
2. **Mobile-First** - Responsive design from smallest screens
3. **Accessibility** - WCAG 2.1 AA compliance
4. **Performance** - Page load < 2 seconds, lazy loading
5. **Offline-Ready** - Progressive Web App (PWA) capabilities

---

## Related Categories
- `11-ui-ux` - Design system and wireframes
- `07-api` - API integration
- `13-testing` - Frontend testing strategy

---

*Source: Frontend specifications from design documents and architecture*
