# Admin Use Cases — YemenMart

## Overview

120 use cases for admin-facing functionality organized by category. Each use case includes ID, name, description, actor, preconditions, main flow, and alternatives.

---

## 1. Dashboard & Monitoring (UC-A-001 to UC-A-015)

### UC-A-001: View Platform Dashboard
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Login to view dashboard overview (total orders, revenue, users, vendors, delivery stats)
- **Alternative**: Dashboard slow, use cache fallback

### UC-A-002: View Real-Time Metrics
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to dashboard to view live metrics (active users, orders in progress, revenue today)
- **Alternative**: Data delay shows last-updated timestamp

### UC-A-003: View Revenue Summary
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to finances to view total revenue, commission earned, subscription revenue, ad revenue
- **Alternative**: No data, show no-revenue message

### UC-A-004: View Order Statistics
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to orders to view stats (total, completed, cancelled, returned, average value)
- **Alternative**: No orders, show empty state

### UC-A-005: View User Growth
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to analytics to view registration trends, active users, retention rates
- **Alternative**: No users, show empty state

### UC-A-006: View Vendor Metrics
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to merchants to view vendor count, active vendors, top vendors, new registrations
- **Alternative**: No vendors, show empty state

### UC-A-007: View Delivery Performance
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to delivery to view stats (success rate, average time, failed deliveries)
- **Alternative**: No delivery data, show empty state

### UC-A-008: View Geographic Distribution
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to analytics to view orders/users/vendors by governorate and city
- **Alternative**: No geographic data, show empty state

### UC-A-009: Export Dashboard Data
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to dashboard, tap export, select data set, download CSV or PDF
- **Alternative**: Export fails, retry

### UC-A-010: Set Dashboard Alerts
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to settings to set alert thresholds (low stock, high cancellations, revenue targets)
- **Alternative**: None

### UC-A-011: View System Health
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to system to view server health, response times, error rates
- **Alternative**: System healthy shows all-systems-operational message

### UC-A-012: View Audit Log
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to system to view audit log, filter by user action date, search
- **Alternative**: No logs, show no-audit-entries message

### UC-A-013: View Platform Activity Feed
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to dashboard to view real-time activity feed (new orders, registrations, disputes)
- **Alternative**: No activity, show no-recent-activity message

### UC-A-014: Compare Period Metrics
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to analytics, select two periods, compare metrics side by side
- **Alternative**: Insufficient historical data, comparison requires 30+ days

### UC-A-015: View Custom Report
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to reports, select report type, configure parameters, generate, view or download
- **Alternative**: Report generation fails, retry

---

## 2. Customer Management (UC-A-016 to UC-A-030)

### UC-A-016: View Customer List
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to customers to view all customers with search and filter by status, tier, registration date
- **Alternative**: No customers, show empty state

### UC-A-017: View Customer Details
- **Actor**: Admin
- **Precondition**: Customer exists
- **Main Flow**: Select customer to view profile (name, phone, email, tier, orders, wallet balance, addresses)
- **Alternative**: None

### UC-A-018: Search Customer by Phone
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to customers, search by phone number, view matching customers
- **Alternative**: No match, show no-customer-found message

### UC-A-019: Block Customer Account
- **Actor**: Admin
- **Precondition**: Customer with violations
- **Main Flow**: Select customer, tap block, enter reason, confirm, account blocked, customer notified
- **Alternative**: Active orders exist, cannot block until resolved

### UC-A-020: Unblock Customer Account
- **Actor**: Admin
- **Precondition**: Blocked customer
- **Main Flow**: Select customer, tap unblock, confirm, account restored
- **Alternative**: None

### UC-A-021: Edit Customer Profile
- **Actor**: Admin
- **Precondition**: Customer exists
- **Main Flow**: Select customer, edit details (name, email, tier override), save
- **Alternative**: Phone number change requires re-verification

### UC-A-022: View Customer Orders
- **Actor**: Admin
- **Precondition**: Customer exists
- **Main Flow**: Select customer to view order history, select order for details
- **Alternative**: No orders, show empty state

### UC-A-023: View Customer Wallet Activity
- **Actor**: Admin
- **Precondition**: Customer exists
- **Main Flow**: Select customer to view wallet balance and transaction history
- **Alternative**: No transactions, show empty state

### UC-A-024: Adjust Customer Wallet
- **Actor**: Admin
- **Precondition**: Customer exists with valid reason
- **Main Flow**: Select customer, navigate to wallet, adjust balance, enter amount plus or minus, enter reason, confirm, balance adjusted
- **Alternative**: Adjustment exceeds limit requires senior admin approval

### UC-A-025: View Customer Disputes
- **Actor**: Admin
- **Precondition**: Customer has disputes
- **Main Flow**: Select customer to view dispute history, select dispute for details
- **Alternative**: No disputes, show empty state

### UC-A-026: View Customer Loyalty Status
- **Actor**: Admin
- **Precondition**: Customer exists
- **Main Flow**: Select customer to view loyalty tier, points balance, referral history
- **Alternative**: No loyalty data, show empty state

### UC-A-027: Manually Upgrade Customer Tier
- **Actor**: Admin
- **Precondition**: Customer exists
- **Main Flow**: Select customer, navigate to loyalty, override tier, confirm, tier updated
- **Alternative**: Tier downgrade requires justification

### UC-A-028: Export Customer Data
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to customers, tap export, select filters, download CSV
- **Alternative**: No customers matching filter, show empty state

### UC-A-029: View Customer Feedback
- **Actor**: Admin
- **Precondition**: Customer has feedback
- **Main Flow**: Select customer to view feedback submissions including reviews, support tickets, ratings
- **Alternative**: No feedback, show empty state

### UC-A-030: Send Customer Notification
- **Actor**: Admin
- **Precondition**: Customer exists
- **Main Flow**: Select customer, tap send notification, compose message, select channel (SMS, WhatsApp, push), send
- **Alternative**: Customer opted out of channel, show cannot-send message

---

## 3. Merchant Management (UC-A-031 to UC-A-050)

### UC-A-031: View Vendor List
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to merchants to view all vendors with status filter (Pending, Active, Suspended, Deactivated)
- **Alternative**: No vendors, show empty state

### UC-A-032: View Vendor Details
- **Actor**: Admin
- **Precondition**: Vendor exists
- **Main Flow**: Select vendor to view profile, store, products, orders, KYC status, rating, financials
- **Alternative**: None

### UC-A-033: Approve Vendor KYC
- **Actor**: Admin
- **Precondition**: Vendor KYC submitted
- **Main Flow**: Navigate to KYC queue, select vendor, review documents, approve, vendor notified, store goes live
- **Alternative**: Documents invalid, reject with reason, vendor re-submits

### UC-A-034: Reject Vendor KYC
- **Actor**: Admin
- **Precondition**: Vendor KYC submitted
- **Main Flow**: Navigate to KYC queue, select vendor, review documents, reject, enter rejection reason, vendor notified
- **Alternative**: None

### UC-A-035: Suspend Vendor
- **Actor**: Admin
- **Precondition**: Vendor with violations
- **Main Flow**: Select vendor, tap suspend, enter reason, confirm, store deactivated, active orders handled, vendor notified
- **Alternative**: Active orders exist, complete or cancel before suspension

### UC-A-036: Reactivate Vendor
- **Actor**: Admin
- **Precondition**: Suspended vendor
- **Main Flow**: Select vendor, tap reactivate, confirm, store restored, vendor notified
- **Alternative**: KYC expired, re-verification required

### UC-A-037: Edit Vendor Details
- **Actor**: Admin
- **Precondition**: Vendor exists
- **Main Flow**: Select vendor, edit details (business name, commission rate, subscription), save
- **Alternative**: Commission rate change requires justification

### UC-A-038: View Vendor Products
- **Actor**: Admin
- **Precondition**: Vendor exists
- **Main Flow**: Select vendor to view product list, select product for details
- **Alternative**: No products, show empty state

### UC-A-039: Moderate Vendor Products
- **Actor**: Admin
- **Precondition**: Vendor has products pending moderation
- **Main Flow**: Navigate to moderation queue, review product listings, approve or reject, enter notes
- **Alternative**: Products violate policy, reject and notify vendor

### UC-A-040: View Vendor Orders
- **Actor**: Admin
- **Precondition**: Vendor exists
- **Main Flow**: Select vendor to view order history, select order for details
- **Alternative**: No orders, show empty state

### UC-A-041: View Vendor Financials
- **Actor**: Admin
- **Precondition**: Vendor exists
- **Main Flow**: Select vendor to view earnings, commissions, payouts, refunds
- **Alternative**: No financial data, show empty state

### UC-A-042: Adjust Vendor Commission Rate
- **Actor**: Admin
- **Precondition**: Vendor exists
- **Main Flow**: Select vendor, navigate to commission settings, enter new rate, save
- **Alternative**: Rate below minimum requires justification

### UC-A-043: Set Vendor Subscription Plan
- **Actor**: Admin
- **Precondition**: Vendor exists
- **Main Flow**: Select vendor, navigate to subscription, assign plan, confirm
- **Alternative**: Plan downgrade, prorate refund

### UC-A-044: View Vendor Rating
- **Actor**: Admin
- **Precondition**: Vendor exists
- **Main Flow**: Select vendor to view overall rating, breakdown by factor (fulfillment, reviews, response time)
- **Alternative**: No ratings, show empty state

### UC-A-045: Feature Vendor Store
- **Actor**: Admin
- **Precondition**: Vendor with high rating
- **Main Flow**: Select vendor, tap feature, confirm, store appears in featured section
- **Alternative**: Vendor does not meet criteria, show message

### UC-A-046: Send Vendor Notification
- **Actor**: Admin
- **Precondition**: Vendor exists
- **Main Flow**: Select vendor, tap send notification, compose message, send
- **Alternative**: Vendor opted out, show message

### UC-A-047: View Vendor Disputes
- **Actor**: Admin
- **Precondition**: Vendor has disputes
- **Main Flow**: Select vendor to view dispute history, select dispute for details
- **Alternative**: No disputes, show empty state

### UC-A-048: Export Vendor Data
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to merchants, tap export, select filters, download CSV
- **Alternative**: No data matching filter

### UC-A-049: Bulk Vendor Operations
- **Actor**: Admin
- **Precondition**: Multiple vendors selected
- **Main Flow**: Select multiple vendors, apply bulk action (suspend, activate, notify), confirm
- **Alternative**: Conflict detected, manual resolution required

### UC-A-050: View Vendor Analytics
- **Actor**: Admin
- **Precondition**: Vendor exists
- **Main Flow**: Select vendor to view analytics (traffic, conversion, revenue trends)
- **Alternative**: Insufficient data, show message

---

## 4. Delivery Management (UC-A-051 to UC-A-060)

### UC-A-051: View Delivery Providers List
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to delivery to view all providers with status and performance metrics
- **Alternative**: No providers, show empty state

### UC-A-052: Approve Delivery Provider
- **Actor**: Admin
- **Precondition**: Provider KYC submitted
- **Main Flow**: Navigate to delivery queue, review provider documents, approve, provider activated
- **Alternative**: Documents invalid, reject with reason

### UC-A-053: Suspend Delivery Provider
- **Actor**: Admin
- **Precondition**: Provider with low rating or violations
- **Main Flow**: Select provider, tap suspend, enter reason, confirm, provider deactivated
- **Alternative**: Active deliveries exist, handle before suspension

### UC-A-054: View Delivery Zones
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to delivery zones to view active zones, coverage map, provider assignments
- **Alternative**: None

### UC-A-055: Create Delivery Zone
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to delivery zones, tap create, define zone boundaries, set base fee, assign providers, save
- **Alternative**: Zone overlaps existing zone, resolve conflicts

### UC-A-056: Edit Delivery Zone
- **Actor**: Admin
- **Precondition**: Delivery zone exists
- **Main Flow**: Select zone, edit boundaries, fees, or providers, save
- **Alternative**: Active deliveries in zone, notify affected users

### UC-A-057: Delete Delivery Zone
- **Actor**: Admin
- **Precondition**: Delivery zone exists
- **Main Flow**: Select zone, tap delete, confirm, zone removed
- **Alternative**: Active orders in zone, cannot delete

### UC-A-058: View Delivery Performance
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to delivery analytics to view success rates, average times, failure reasons
- **Alternative**: No data, show empty state

### UC-A-059: Set Delivery Fee Structure
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to delivery settings to configure fee calculation rules (base, per-km, weight-based)
- **Alternative**: None

### UC-A-060: View Delivery Disputes
- **Actor**: Admin
- **Precondition**: Delivery disputes exist
- **Main Flow**: Navigate to delivery to view all delivery-related disputes, select for details
- **Alternative**: No disputes, show empty state

---

## 5. Product Management (UC-A-061 to UC-A-070)

### UC-A-061: View All Products
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to products to view all marketplace products with filters (status, category, vendor)
- **Alternative**: No products, show empty state

### UC-A-062: View Product Details
- **Actor**: Admin
- **Precondition**: Product exists
- **Main Flow**: Select product to view full details, images, vendor, reviews, sales data
- **Alternative**: None

### UC-A-063: Moderate Product Listing
- **Actor**: Admin
- **Precondition**: Product pending moderation
- **Main Flow**: Navigate to moderation queue, review product, approve or reject, enter notes
- **Alternative**: Product violates policy, reject and notify vendor

### UC-A-064: Remove Product
- **Actor**: Admin
- **Precondition**: Product violates policy
- **Main Flow**: Select product, tap remove, enter reason, confirm, product removed, vendor notified
- **Alternative**: Active orders exist, handle before removal

### UC-A-065: Edit Product Details
- **Actor**: Admin
- **Precondition**: Product exists
- **Main Flow**: Select product, edit details (price, description, category, stock), save
- **Alternative**: Major changes, notify vendor

### UC-A-066: Manage Product Categories
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to categories to add, edit, delete, or reorder categories and subcategories
- **Alternative**: Category has products, confirm reassignment

### UC-A-067: Set Product Scoring Rules
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to settings to configure auto-scoring weights (images, description, reviews, sales)
- **Alternative**: None

### UC-A-068: View Product Reports
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to products to view reports (flagged products, low-stock, high-return)
- **Alternative**: No flagged products, show empty state

### UC-A-069: Bulk Product Operations
- **Actor**: Admin
- **Precondition**: Multiple products selected
- **Main Flow**: Select multiple products, apply bulk action (approve, reject, remove, feature), confirm
- **Alternative**: Conflict detected, manual resolution required

### UC-A-070: View Product Analytics
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to analytics to view product metrics (total listings, average price, category distribution)
- **Alternative**: No data, show empty state

---

## 6. Order Management (UC-A-071 to UC-A-080)

### UC-A-071: View All Orders
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to orders to view all orders with filters (status, date, vendor, value)
- **Alternative**: No orders, show empty state

### UC-A-072: View Order Details
- **Actor**: Admin
- **Precondition**: Order exists
- **Main Flow**: Select order to view full details (customer, vendor, items, payment, delivery, timeline)
- **Alternative**: None

### UC-A-073: Cancel Order
- **Actor**: Admin
- **Precondition**: Order can be cancelled
- **Main Flow**: Select order, tap cancel, enter reason, confirm, order cancelled, refund initiated
- **Alternative**: Order already delivered, cannot cancel

### UC-A-074: Override Order Status
- **Actor**: Admin
- **Precondition**: Order stuck or requires manual intervention
- **Main Flow**: Select order, tap override status, select new status, enter reason, confirm
- **Alternative**: Status change invalid, show error

### UC-A-075: View Order Disputes
- **Actor**: Admin
- **Precondition**: Order has dispute
- **Main Flow**: Select order to view dispute details, evidence from both parties
- **Alternative**: No disputes, show empty state

### UC-A-076: Resolve Order Dispute
- **Actor**: Admin
- **Precondition**: Active dispute
- **Main Flow**: Select dispute, review evidence, decide resolution (full refund, partial refund, no refund), notify both parties
- **Alternative**: Insufficient evidence, request more information

### UC-A-077: View Order Timeline
- **Actor**: Admin
- **Precondition**: Order exists
- **Main Flow**: Select order to view complete timeline with all status changes and timestamps
- **Alternative**: None

### UC-A-078: Export Order Data
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to orders, tap export, select filters, download CSV or Excel
- **Alternative**: No orders matching filter

### UC-A-079: View Order Metrics
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to analytics to view order metrics (volume, value, completion rate, cancellation rate)
- **Alternative**: No data, show empty state

### UC-A-080: Bulk Order Operations
- **Actor**: Admin
- **Precondition**: Multiple orders selected
- **Main Flow**: Select multiple orders, apply bulk action (cancel, refund), confirm
- **Alternative**: Conflicts detected, manual resolution required

---

## 7. Financial Management (UC-A-081 to UC-A-090)

### UC-A-081: View Platform Revenue
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to finances to view total revenue, commission, subscriptions, advertising
- **Alternative**: No data, show empty state

### UC-A-082: View Vendor Settlements
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to settlements to view all vendor payouts (pending, processed, failed)
- **Alternative**: No settlements, show empty state

### UC-A-083: Process Vendor Settlement
- **Actor**: Admin
- **Precondition**: Pending settlement exists
- **Main Flow**: Select settlement, review details, process payout, mark as completed
- **Alternative**: Settlement fails, retry or mark as failed

### UC-A-084: View Commission Report
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to finances to view commission breakdown by vendor, category, period
- **Alternative**: No data, show empty state

### UC-A-085: View Refund Report
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to finances to view all refunds (pending, processed), total refund value
- **Alternative**: No refunds, show empty state

### UC-A-086: Process Refund
- **Actor**: Admin
- **Precondition**: Pending refund exists
- **Main Flow**: Select refund, review details, process refund to customer wallet, confirm
- **Alternative**: Refund fails, retry

### UC-A-087: View Transaction Log
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to finances to view complete transaction log with filters
- **Alternative**: No transactions, show empty state

### UC-A-088: Download Financial Reports
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to finances, tap download, select report type and period, download PDF or CSV
- **Alternative**: Report generation fails, retry

### UC-A-089: View Subscription Revenue
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to finances to view vendor subscription revenue, active subscriptions, churn rate
- **Alternative**: No data, show empty state

### UC-A-090: View Advertising Revenue
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to finances to view ad revenue, active campaigns, performance metrics
- **Alternative**: No data, show empty state

---

## 8. Coupon Management (UC-A-091 to UC-A-095)

### UC-A-091: Create Platform Coupon
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to coupons, tap create, set code, discount, min order, expiry, usage limit, target vendors, save
- **Alternative**: Discount outside allowed range, show error

### UC-A-092: View All Coupons
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to coupons to view all coupons (platform and vendor-created) with status
- **Alternative**: No coupons, show empty state

### UC-A-093: Deactivate Coupon
- **Actor**: Admin
- **Precondition**: Active coupon exists
- **Main Flow**: Select coupon, tap deactivate, confirm, coupon immediately deactivated
- **Alternative**: None

### UC-A-094: View Coupon Performance
- **Actor**: Admin
- **Precondition**: Coupon has usage data
- **Main Flow**: Select coupon to view usage count, revenue generated, discount given, conversion rate
- **Alternative**: No usage data, show empty state

### UC-A-095: Delete Coupon
- **Actor**: Admin
- **Precondition**: Coupon with zero usage
- **Main Flow**: Select coupon, tap delete, confirm, coupon removed
- **Alternative**: Coupon has usage, cannot delete, deactivate instead

---

## 9. Content Management (UC-A-096 to UC-A-100)

### UC-A-096: Manage Homepage Banner
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to content to add, edit, or remove homepage banners, set display schedule, save
- **Alternative**: Banner dimensions incorrect, show guidelines

### UC-A-097: Manage Promotions Page
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to content to create, edit, or remove promotion entries, set featured status, save
- **Alternative**: None

### UC-A-098: Manage Static Pages
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to content to edit static pages (About, Terms, Privacy Policy, FAQ), save changes
- **Alternative**: None

### UC-A-099: Manage Push Notifications
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to notifications to create, schedule, or cancel push notifications, target audience, send
- **Alternative**: Notification content violates policy, review required

### UC-A-100: View Content Reports
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to content to view content performance (banner clicks, page views, notification open rates)
- **Alternative**: No data, show empty state

---

## 10. Settings Management (UC-A-101 to UC-A-110)

### UC-A-101: Configure Platform Settings
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to settings to configure platform-wide settings (name, logo, currency, language), save
- **Alternative**: None

### UC-A-102: Configure Payment Settings
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to settings to configure payment providers, escrow settings, commission rates, save
- **Alternative**: Payment provider integration fails, troubleshoot

### UC-A-103: Configure Delivery Settings
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to settings to configure delivery zones, fee structures, attempt limits, save
- **Alternative**: None

### UC-A-104: Configure Notification Settings
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to settings to configure notification channels (SMS, WhatsApp, push), templates, save
- **Alternative**: None

### UC-A-105: Configure Loyalty Settings
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to settings to configure loyalty tiers, points rates, redemption rules, save
- **Alternative**: None

### UC-A-106: Configure Tax Settings
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to settings to configure tax rates, ZATCA integration, invoice templates, save
- **Alternative**: ZATCA integration fails, troubleshoot

### UC-A-107: Configure Exchange Rate Settings
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to settings to configure exchange rate source, update frequency, save
- **Alternative**: Rate source unavailable, manual update

### UC-A-108: Manage Admin Roles
- **Actor**: Super Admin
- **Precondition**: Super Admin logged in
- **Main Flow**: Navigate to settings to create, edit, or delete admin roles with permissions, save
- **Alternative**: Cannot modify own role

### UC-A-109: View System Configuration
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to settings to view current system configuration, export config
- **Alternative**: None

### UC-A-110: Backup System Configuration
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to settings to export system configuration as backup file, download
- **Alternative**: Backup fails, retry

---

## 11. Security Management (UC-A-111 to UC-A-115)

### UC-A-111: View Security Alerts
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to security to view security alerts (failed logins, suspicious activity, breaches)
- **Alternative**: No alerts, show all-clear message

### UC-A-112: View Failed Login Attempts
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to security to view failed login attempts with user, IP, timestamp
- **Alternative**: No failed attempts, show empty state

### UC-A-113: Block IP Address
- **Actor**: Admin
- **Precondition**: Suspicious IP activity
- **Main Flow**: Navigate to security, select IP address, tap block, enter reason, confirm, IP blocked
- **Alternative**: IP is whitelisted, cannot block

### UC-A-114: View Security Audit Log
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to security to view security-specific audit log (permission changes, data access, config changes)
- **Alternative**: No entries, show empty state

### UC-A-115: Manage Two-Factor Authentication
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to security to enable or disable 2FA for admin accounts, configure settings
- **Alternative**: None

---

## 12. Support Management (UC-A-116 to UC-A-120)

### UC-A-116: View Support Tickets
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to support to view all tickets with status (Open, In Progress, Resolved, Closed)
- **Alternative**: No tickets, show empty state

### UC-A-117: Assign Support Ticket
- **Actor**: Admin
- **Precondition**: Ticket exists
- **Main Flow**: Select ticket, assign to support agent, set priority, save
- **Alternative**: Agent unavailable, reassign

### UC-A-118: Respond to Support Ticket
- **Actor**: Admin
- **Precondition**: Ticket assigned
- **Main Flow**: Select ticket, review customer message, compose response, send
- **Alternative**: Issue requires escalation, escalate to senior admin

### UC-A-119: Close Support Ticket
- **Actor**: Admin
- **Precondition**: Ticket resolved
- **Main Flow**: Select ticket, mark as resolved, enter resolution notes, close
- **Alternative**: Customer not satisfied, reopen ticket

### UC-A-120: View Support Metrics
- **Actor**: Admin
- **Precondition**: Admin logged in
- **Main Flow**: Navigate to support to view metrics (tickets per day, average resolution time, satisfaction rate)
- **Alternative**: No data, show empty state
