# Vendor Use Cases — YemenMart

## Overview

107 use cases for vendor-facing functionality organized by category. Each use case includes ID, name, description, actor, preconditions, main flow, and alternatives.

---

## 1. Account Management (UC-V-001 to UC-V-010)

### UC-V-001: Register as Vendor
- **Actor**: New Vendor
- **Precondition**: No existing vendor account
- **Main Flow**: Tap "Sell on YemenMart" → Enter business details (name, phone, email) → Set password → Account created → Pending KYC
- **Alternative**: Already registered → Login instead

### UC-V-002: Complete KYC Verification
- **Actor**: Registered Vendor
- **Precondition**: Account pending verification
- **Main Flow**: Navigate to KYC section → Upload commercial registration → Upload ID proof → Upload bank/wallet details → Submit → Await review (24–48 hours)
- **Alternative**: Documents rejected → Re-upload → Resubmit

### UC-V-003: Login to Vendor Dashboard
- **Actor**: Verified Vendor
- **Precondition**: KYC approved
- **Main Flow**: Enter credentials → OTP verification → Access vendor dashboard
- **Alternative**: 5 failed attempts → 15-minute lockout

### UC-V-004: Update Business Profile
- **Actor**: Verified Vendor
- **Precondition**: Active vendor account
- **Main Flow**: Navigate to settings → Update business name, description, logo, contact info → Save
- **Alternative**: Major changes (business name) → Re-verification required

### UC-V-005: Change Password
- **Actor**: Verified Vendor
- **Precondition**: Active session
- **Main Flow**: Navigate to security → Enter current password → Enter new password → Confirm → Updated
- **Alternative**: Forgot password → OTP reset flow

### UC-V-006: Manage Store Settings
- **Actor**: Verified Vendor
- **Precondition**: Active vendor account
- **Main Flow**: Navigate to store settings → Configure shipping, return policy, store hours → Save
- **Alternative**: None

### UC-V-007: View Vendor Dashboard
- **Actor**: Verified Vendor
- **Precondition**: Active vendor account
- **Main Flow**: Login → View dashboard overview (orders, revenue, products, ratings)
- **Alternative**: New vendor → Onboarding tour

### UC-V-008: Deactivate Store
- **Actor**: Verified Vendor
- **Precondition**: No active orders
- **Main Flow**: Navigate to settings → Request deactivation → Confirm → Store deactivated
- **Alternative**: Active orders → Cannot deactivate until completed

### UC-V-009: Reactivate Store
- **Actor**: Deactivated Vendor
- **Precondition**: Previously deactivated store
- **Main Flow**: Login → Request reactivation → Admin review → Store reactivated
- **Alternative**: KYC expired → Re-verification required

### UC-V-010: View Subscription Status
- **Actor**: Verified Vendor
- **Precondition**: Active vendor account
- **Main Flow**: Navigate to subscription → View current plan, features, expiry date, renewal options
- **Alternative**: No subscription → "Choose a plan" prompt

---

## 2. Store Management (UC-V-011 to UC-V-020)

### UC-V-011: Select Store Template
- **Actor**: Verified Vendor
- **Precondition**: Active account
- **Main Flow**: Navigate to store customization → Browse templates → Select template → Apply → Storefront updated
- **Alternative**: Premium templates → Require subscription upgrade

### UC-V-012: Customize Store Banner
- **Actor**: Verified Vendor
- **Precondition**: Store template selected
- **Main Flow**: Navigate to store customization → Upload banner image → Crop/resize → Save
- **Alternative**: Image too large → Auto-compress → Upload

### UC-V-013: Add Store Logo
- **Actor**: Verified Vendor
- **Precondition**: Active account
- **Main Flow**: Navigate to store settings → Upload logo → Crop → Save → Logo displayed on store page
- **Alternative**: No logo → Default placeholder used

### UC-V-014: Set Store Description
- **Actor**: Verified Vendor
- **Precondition**: Active account
- **Main Flow**: Navigate to store settings → Enter store description (Arabic) → Save
- **Alternative**: Description too long → Truncated with warning

### UC-V-015: Configure Store Hours
- **Actor**: Verified Vendor
- **Precondition**: Active account
- **Main Flow**: Navigate to store settings → Set operating hours per day → Save
- **Alternative**: 24/7 operation → Select "Always Open"

### UC-V-016: Set Shipping Policy
- **Actor**: Verified Vendor
- **Precondition**: Active account
- **Main Flow**: Navigate to store settings → Enter shipping policy (Arabic) → Set delivery zones → Save
- **Alternative**: Use platform default policy

### UC-V-017: Set Return Policy
- **Actor**: Verified Vendor
- **Precondition**: Active account
- **Main Flow**: Navigate to store settings → Enter return policy (Arabic) → Set return window → Save
- **Alternative**: Use platform default (7 days)

### UC-V-018: Preview Storefront
- **Actor**: Verified Vendor
- **Precondition**: Store configured
- **Main Flow**: Navigate to store → Tap "Preview" → View store as customer would see it
- **Alternative**: None

### UC-V-019: View Store Performance
- **Actor**: Verified Vendor
- **Precondition**: Active store
- **Main Flow**: Navigate to analytics → View store metrics (views, conversion, revenue, ratings)
- **Alternative**: No data yet → "Insufficient data" message

### UC-V-020: Share Store Link
- **Actor**: Verified Vendor
- **Precondition**: Active store
- **Main Flow**: Navigate to store → Tap "Share" → Copy link or share via WhatsApp/social
- **Alternative**: Share fails → Copy link fallback

---

## 3. KYC & Verification (UC-V-021 to UC-V-028)

### UC-V-021: Upload Commercial Registration
- **Actor**: Registered Vendor
- **Precondition**: KYC submission in progress
- **Main Flow**: Navigate to KYC → Upload CR document → System validates format → Document stored
- **Alternative**: Invalid format → "Please upload PDF/JPG" → Re-upload

### UC-V-022: Upload ID Proof
- **Actor**: Registered Vendor
- **Precondition**: KYC submission in progress
- **Main Flow**: Navigate to KYC → Upload national ID or passport → System validates → Document stored
- **Alternative**: Expired ID → "ID must be valid" → Re-upload

### UC-V-023: Upload Bank/Wallet Details
- **Actor**: Registered Vendor
- **Precondition**: KYC submission in progress
- **Main Flow**: Navigate to KYC → Enter wallet account details → Verify via micro-deposit or OTP → Confirmed
- **Alternative**: Verification fails → Re-enter details → Contact support

### UC-V-024: Submit KYC for Review
- **Actor**: Registered Vendor
- **Precondition**: All KYC documents uploaded
- **Main Flow**: Review uploaded documents → Submit KYC → Status changes to "Under Review" → Admin reviews within 48 hours
- **Alternative**: Missing documents → Cannot submit → Prompt to complete

### UC-V-025: View KYC Status
- **Actor**: Registered Vendor
- **Precondition**: KYC submitted
- **Main Flow**: Navigate to KYC → View status (Pending → Under Review → Approved/Rejected)
- **Alternative**: Rejected → See rejection reason → Re-upload and resubmit

### UC-V-026: Re-upload Rejected Documents
- **Actor**: Registered Vendor
- **Precondition**: KYC rejected
- **Main Flow**: View rejection reason → Re-upload corrected document → Resubmit → Await review
- **Alternative**: Repeated rejection → Contact support

### UC-V-027: Update KYC Information
- **Actor**: Verified Vendor
- **Precondition**: KYC previously approved
- **Main Flow**: Navigate to KYC → Update information (address, bank details) → Submit changes → Re-verification if major change
- **Alternative**: Minor changes → Auto-approved

### UC-V-028: View KYC Expiry
- **Actor**: Verified Vendor
- **Precondition**: KYC approved
- **Main Flow**: Navigate to KYC → View expiry date → Renew before expiry
- **Alternative**: Expired → Store deactivated → Re-verification required

---

## 4. Product Management (UC-V-029 to UC-V-050)

### UC-V-029: Add New Product
- **Actor**: Verified Vendor
- **Precondition**: Active store + within product limit
- **Main Flow**: Navigate to products → "Add Product" → Enter details (Arabic name, description, price, stock, category) → Upload images → Save → Product pending moderation
- **Alternative**: Product limit reached → "Upgrade plan" prompt

### UC-V-030: Enter Arabic Product Name
- **Actor**: Verified Vendor
- **Precondition**: Adding/editing product
- **Main Flow**: Enter product name in Arabic (min 3 characters) → System validates → Name saved
- **Alternative**: Name too short → "Name must be at least 3 characters"

### UC-V-031: Upload Product Images
- **Actor**: Verified Vendor
- **Precondition**: Adding/editing product
- **Main Flow**: Tap "Add Images" → Select up to 10 images → Upload → Crop/arrange → Save
- **Alternative**: No images → Cannot save → Image too large → Auto-compress

### UC-V-032: Set Product Price
- **Actor**: Verified Vendor
- **Precondition**: Adding/editing product
- **Main Flow**: Enter price in YER → System validates range (0–10,000,000 YER) → Price saved
- **Alternative**: Price outside range → Error message → Adjust price

### UC-V-033: Set Stock Quantity
- **Actor**: Verified Vendor
- **Precondition**: Adding/editing product
- **Main Flow**: Enter stock quantity → Stock tracked per variant → Saved
- **Alternative**: Zero stock → Product marked "Out of Stock"

### UC-V-034: Create Product Variants
- **Actor**: Verified Vendor
- **Precondition**: Adding/editing product
- **Main Flow**: Enable variants → Add dimensions (size, color) → Add options → Set price/stock per variant → Save
- **Alternative**: Max variants reached (50 combinations) → Cannot add more

### UC-V-035: Edit Product Details
- **Actor**: Verified Vendor
- **Precondition**: Product exists in store
- **Main Flow**: Select product → Edit details → Save → Changes reflected after moderation
- **Alternative**: Product in active order → Cannot edit until order completes

### UC-V-036: Delete Product
- **Actor**: Verified Vendor
- **Precondition**: Product exists, no active orders
- **Main Flow**: Select product → Tap "Delete" → Confirm → Product removed
- **Alternative**: Active orders exist → Cannot delete → "Deactivate" instead

### UC-V-037: Deactivate Product
- **Actor**: Verified Vendor
- **Precondition**: Product exists
- **Main Flow**: Select product → Tap "Deactivate" → Product hidden from marketplace
- **Alternative**: Reactivate anytime → Product restored

### UC-V-038: Bulk Edit Products
- **Actor**: Verified Vendor
- **Precondition**: Multiple products
- **Main Flow**: Select multiple products → Apply bulk action (price change, stock update, deactivate) → Confirm → Applied
- **Alternative**: Conflicts detected → Manual resolution required

### UC-V-039: Import Products via CSV
- **Actor**: Verified Vendor
- **Precondition**: Has CSV file with product data
- **Main Flow**: Navigate to products → "Import" → Upload CSV → Map columns → Preview → Import → Products created
- **Alternative**: CSV errors → Error report shown → Fix and re-upload

### UC-V-040: Export Products
- **Actor**: Verified Vendor
- **Precondition**: Has products
- **Main Flow**: Navigate to products → "Export" → Select format (CSV, Excel) → Download
- **Alternative**: No products → Export hidden

### UC-V-041: View Product Analytics
- **Actor**: Verified Vendor
- **Precondition**: Product exists
- **Main Flow**: Select product → View analytics (views, clicks, conversion, revenue)
- **Alternative**: Insufficient data → "Analytics available after 7 days"

### UC-V-042: Set Product Category
- **Actor**: Verified Vendor
- **Precondition**: Adding/editing product
- **Main Flow**: Select primary category → Optionally select up to 2 secondary categories → Save
- **Alternative**: Category not found → Request new category → Admin creates

### UC-V-043: Add Product Specifications
- **Actor**: Verified Vendor
- **Precondition**: Adding/editing product
- **Main Flow**: Navigate to specifications → Enter key-value pairs (weight, dimensions, material) → Save
- **Alternative**: No specifications → Optional

### UC-V-044: Mark Product as Fragile
- **Actor**: Verified Vendor
- **Precondition**: Adding/editing product
- **Main Flow**: Toggle "Fragile" flag → Special handling enabled for delivery
- **Alternative**: Not fragile → Default handling

### UC-V-045: Set Product Tags
- **Actor**: Verified Vendor
- **Precondition**: Adding/editing product
- **Main Flow**: Enter tags (keywords) → Tags used for search improvement → Save
- **Alternative**: No tags → Optional

### UC-V-046: View Product Reviews
- **Actor**: Verified Vendor
- **Precondition**: Product has reviews
- **Main Flow**: Select product → View customer reviews → See ratings and comments
- **Alternative**: No reviews → "No reviews yet"

### UC-V-047: Respond to Product Review
- **Actor**: Verified Vendor
- **Precondition**: Product has reviews
- **Main Flow**: Select review → Write response → Submit → Response displayed
- **Alternative**: Review period expired → Cannot respond after 30 days

### UC-V-048: View Product Listings
- **Actor**: Verified Vendor
- **Precondition**: Has products
- **Main Flow**: Navigate to products → View all products with status (Active, Inactive, Pending, Out of Stock) → Filter/sort
- **Alternative**: No products → "Add your first product" CTA

### UC-V-049: Duplicate Product
- **Actor**: Verified Vendor
- **Precondition**: Product exists
- **Main Flow**: Select product → Tap "Duplicate" → Edit copy → Save as new product
- **Alternative**: Product limit reached → Cannot duplicate

### UC-V-050: Set Minimum Order Quantity
- **Actor**: Verified Vendor
- **Precondition**: Product exists
- **Main Flow**: Select product → Set minimum order quantity → Save
- **Alternative**: Default is 1

---

## 5. Offers & Promotions (UC-V-051 to UC-V-060)

### UC-V-051: Create Flash Sale
- **Actor**: Verified Vendor
- **Precondition**: Active store
- **Main Flow**: Navigate to promotions → "Create Flash Sale" → Select products → Set discount percentage → Set start/end time → Save → Flash sale live
- **Alternative**: Discount >90% → Requires admin approval

### UC-V-052: Schedule Promotion
- **Actor**: Verified Vendor
- **Precondition**: Active store
- **Main Flow**: Navigate to promotions → "Schedule" → Select products → Set dates → Set discount → Save → Active during scheduled period
- **Alternative**: Conflict with existing promotion → "Product already on sale" warning

### UC-V-053: Create Bundle Offer
- **Actor**: Verified Vendor
- **Precondition**: Active store
- **Main Flow**: Navigate to promotions → "Create Bundle" → Select products → Set bundle price → Save → Bundle available
- **Alternative**: Bundle price higher than individual → Warning shown

### UC-V-054: View Promotion Performance
- **Actor**: Verified Vendor
- **Precondition**: Active promotion
- **Main Flow**: Navigate to promotions → Select promotion → View metrics (sales, revenue, views)
- **Alternative**: Insufficient data → "Data available after promotion ends"

### UC-V-055: Edit Active Promotion
- **Actor**: Verified Vendor
- **Precondition**: Active promotion
- **Main Flow**: Select promotion → Edit details → Save → Changes applied
- **Alternative**: Orders already placed at old price → Old price honored

### UC-V-056: Cancel Promotion
- **Actor**: Verified Vendor
- **Precondition**: Active promotion
- **Main Flow**: Select promotion → Tap "Cancel" → Confirm → Promotion ended immediately
- **Alternative**: Orders in progress → Old price honored for those orders

### UC-V-057: View Promotion History
- **Actor**: Verified Vendor
- **Precondition**: Has past promotions
- **Main Flow**: Navigate to promotions → View all past promotions with performance
- **Alternative**: No history → "No promotions yet"

### UC-V-058: Participate in Platform Campaign
- **Actor**: Verified Vendor
- **Precondition**: Platform campaign active
- **Main Flow**: Receive campaign invitation → Review terms → Opt-in → Select products → Campaign live
- **Alternative**: Decline → No impact on store

### UC-V-059: Set Volume Discount
- **Actor**: Verified Vendor
- **Precondition**: Active store
- **Main Flow**: Select product → Set tiered pricing (buy 2+, get X% off) → Save
- **Alternative**: No volume discount → Default single price

### UC-V-060: Create Buy-One-Get-One Offer
- **Actor**: Verified Vendor
- **Precondition**: Active store
- **Main Flow**: Navigate to promotions → "BOGO" → Select product → Set get-one product → Save → BOGO live
- **Alternative**: Get-one product out of stock → Offer paused

---

## 6. Coupon Management (UC-V-061 to UC-V-070)

### UC-V-061: Create Coupon
- **Actor**: Verified Vendor
- **Precondition**: Active store
- **Main Flow**: Navigate to coupons → "Create Coupon" → Set code, discount (1–90%), min order, expiry, usage limit → Save → Coupon active
- **Alternative**: Discount outside range → Error → Adjust

### UC-V-062: Set Coupon Discount Percentage
- **Actor**: Verified Vendor
- **Precondition**: Creating coupon
- **Main Flow**: Enter discount percentage (1–90%) → System validates → Saved
- **Alternative**: Below 1% or above 90% → Error → Adjust

### UC-V-063: Set Coupon Expiry Date
- **Actor**: Verified Vendor
- **Precondition**: Creating coupon
- **Main Flow**: Select expiry date (max 90 days from creation) → Saved
- **Alternative**: Beyond 90 days → "Maximum 90 days" → Adjust

### UC-V-064: Set Coupon Usage Limit
- **Actor**: Verified Vendor
- **Precondition**: Creating coupon
- **Main Flow**: Enter maximum uses per customer and total uses → Saved
- **Alternative**: No limit → Unlimited coupon

### UC-V-065: Set Minimum Order for Coupon
- **Actor**: Verified Vendor
- **Precondition**: Creating coupon
- **Main Flow**: Enter minimum order value → Coupon applicable only when minimum met → Saved
- **Alternative**: No minimum → Coupon applicable to any order

### UC-V-066: View Coupon Performance
- **Actor**: Verified Vendor
- **Precondition**: Has coupons
- **Main Flow**: Navigate to coupons → Select coupon → View usage count, revenue generated, discount given
- **Alternative**: No usage → "No redemptions yet"

### UC-V-067: Deactivate Coupon
- **Actor**: Verified Vendor
- **Precondition**: Active coupon
- **Main Flow**: Select coupon → Tap "Deactivate" → Coupon immediately deactivated → Cannot be used further
- **Alternative**: Reactivate anytime → Coupon restored

### UC-V-068: Edit Coupon
- **Actor**: Verified Vendor
- **Precondition**: Active coupon
- **Main Flow**: Select coupon → Edit details (discount, expiry, limits) → Save → Changes applied
- **Alternative**: Coupon already used → Cannot reduce discount below redeemed value

### UC-V-069: Delete Coupon
- **Actor**: Verified Vendor
- **Precondition**: Coupon with zero usage
- **Main Flow**: Select coupon → Tap "Delete" → Confirm → Coupon removed
- **Alternative**: Coupon has usage → Cannot delete → Deactivate instead

### UC-V-070: View All Coupons
- **Actor**: Verified Vendor
- **Precondition**: Has coupons
- **Main Flow**: Navigate to coupons → View all coupons with status (Active, Expired, Deactivated) → Filter/sort
- **Alternative**: No coupons → "Create your first coupon" CTA

---

## 7. Order Management (UC-V-071 to UC-V-085)

### UC-V-071: View Incoming Orders
- **Actor**: Verified Vendor
- **Precondition**: Has orders
- **Main Flow**: Navigate to orders → View pending orders → See order details, customer info, items
- **Alternative**: No orders → "No pending orders" message

### UC-V-072: Confirm Order
- **Actor**: Verified Vendor
- **Precondition**: Pending order
- **Main Flow**: Select order → Review items → Tap "Confirm" → Order confirmed → Preparation begins
- **Alternative**: Cannot fulfill → "Reject" → Reason required

### UC-V-073: Reject Order
- **Actor**: Verified Vendor
- **Precondition**: Pending order
- **Main Flow**: Select order → Tap "Reject" → Select reason (out of stock, cannot deliver, etc.) → Order rejected → Customer refunded
- **Alternative**: No reason → Cannot reject

### UC-V-074: Process Order
- **Actor**: Verified Vendor
- **Precondition**: Confirmed order
- **Main Flow**: Select order → Mark as "Processing" → Prepare items → Mark as "Ready for Pickup"
- **Alternative**: Items unavailable → Partial processing → Contact customer

### UC-V-075: Mark Order Ready for Delivery
- **Actor**: Verified Vendor
- **Precondition**: Order processed
- **Main Flow**: Select order → Tap "Ready for Delivery" → Delivery provider notified → Pickup scheduled
- **Alternative**: Self-delivery → Mark as "Dispatched" directly

### UC-V-076: Dispatch Order
- **Actor**: Verified Vendor
- **Precondition**: Order ready
- **Main Flow**: Select order → Tap "Dispatch" → Hand to delivery provider → Tracking updated
- **Alternative**: Self-delivery → Enter delivery details

### UC-V-077: View Order Details
- **Actor**: Verified Vendor
- **Precondition**: Has order
- **Main Flow**: Select order → View customer info, items, delivery address, payment status, timeline
- **Alternative**: None

### UC-V-078: Contact Customer About Order
- **Actor**: Verified Vendor
- **Precondition**: Has order
- **Main Flow**: Select order → Tap "Contact Customer" → In-app chat or WhatsApp → Communicate
- **Alternative**: Customer not responding → Escalate to support

### UC-V-079: Handle Partial Order
- **Actor**: Verified Vendor
- **Precondition**: Order with some items unavailable
- **Main Flow**: Select order → Mark some items as "Unavailable" → Process available items → Customer notified → Partial refund for unavailable items
- **Alternative**: All items unavailable → Full rejection and refund

### UC-V-080: View Order History
- **Actor**: Verified Vendor
- **Precondition**: Has orders
- **Main Flow**: Navigate to orders → View all orders with filters (status, date, customer) → Select for details
- **Alternative**: No orders → Empty state

### UC-V-081: Export Order Data
- **Actor**: Verified Vendor
- **Precondition**: Has orders
- **Main Flow**: Navigate to orders → "Export" → Select date range → Download CSV/Excel
- **Alternative**: No orders in range → "No data" message

### UC-V-082: Handle Order Cancellation
- **Actor**: Verified Vendor
- **Precondition**: Customer cancelled order
- **Main Flow**: Receive notification → Order auto-cancelled (if within 15 min) → Refund processed → No action needed
- **Alternative**: Order already processing → Vendor approves/rejects cancellation

### UC-V-083: View Order Metrics
- **Actor**: Verified Vendor
- **Precondition**: Has orders
- **Main Flow**: Navigate to analytics → View order metrics (total, average value, fulfillment rate, cancellation rate)
- **Alternative**: No data → "Metrics available after first order"

### UC-V-084: Manage Delivery Provider
- **Actor**: Verified Vendor
- **Precondition**: Self-delivery or provider assigned
- **Main Flow**: Select order → Assign delivery provider → Track delivery → Confirm completion
- **Alternative**: No provider available → Use platform delivery network

### UC-V-085: Handle Return Request
- **Actor**: Verified Vendor
- **Precondition**: Customer return request
- **Main Flow**: Receive return notification → Review request → Approve/reject → If approved → Return pickup scheduled → Refund processed
- **Alternative**: Dispute return → Admin mediates

---

## 8. Financial Management (UC-V-086 to UC-V-095)

### UC-V-086: View Earnings Dashboard
- **Actor**: Verified Vendor
- **Precondition**: Has sales
- **Main Flow**: Navigate to finances → View total earnings, pending settlement, available balance
- **Alternative**: No sales → "Start selling to earn" message

### UC-V-087: View Transaction History
- **Actor**: Verified Vendor
- **Precondition**: Has transactions
- **Main Flow**: Navigate to finances → View transaction list with filters (sale, commission, settlement, refund) → Select for details
- **Alternative**: No transactions → Empty state

### UC-V-088: Request Payout
- **Actor**: Verified Vendor
- **Precondition**: Available balance ≥ 1,000 YER
- **Main Flow**: Navigate to finances → Tap "Request Payout" → Enter amount → Confirm → Payout processed
- **Alternative**: Below minimum → "Minimum payout is 1,000 YER" → Roll over

### UC-V-089: View Commission Statements
- **Actor**: Verified Vendor
- **Precondition**: Has sales
- **Main Flow**: Navigate to finances → View commission breakdown per order → See platform fee deducted
- **Alternative**: No sales → Empty state

### UC-V-090: Download Financial Reports
- **Actor**: Verified Vendor
- **Precondition**: Has transactions
- **Main Flow**: Navigate to finances → "Download Report" → Select period → Download PDF/CSV
- **Alternative**: No transactions in period → "No data"

### UC-V-091: View Payout History
- **Actor**: Verified Vendor
- **Precondition**: Has payouts
- **Main Flow**: Navigate to finances → View all past payouts with dates, amounts, methods
- **Alternative**: No payouts → "No payouts yet"

### UC-V-092: Set Payout Method
- **Actor**: Verified Vendor
- **Precondition**: Active account
- **Main Flow**: Navigate to finances → Set payout method (wallet, bank transfer) → Enter details → Save
- **Alternative**: Bank transfer → Additional verification required

### UC-V-093: View Refund Liability
- **Actor**: Verified Vendor
- **Precondition**: Has orders
- **Main Flow**: Navigate to finances → View pending refunds, completed refunds, refund rate
- **Alternative**: No refunds → "No pending refunds"

### UC-V-094: Dispute Commission Charge
- **Actor**: Verified Vendor
- **Precondition**: Disagrees with commission
- **Main Flow**: Navigate to finances → Select transaction → Tap "Dispute" → Describe issue → Submit → Admin reviews
- **Alternative**: Commission correct → Dispute rejected with explanation

### UC-V-095: View Tax Summary
- **Actor**: Verified Vendor
- **Precondition**: Has sales
- **Main Flow**: Navigate to finances → View tax summary (applicable taxes, collected, remitted)
- **Alternative**: No tax applicable → "No tax data"

---

## 9. Shipping & Delivery (UC-V-096 to UC-V-100)

### UC-V-096: Set Delivery Zones
- **Actor**: Verified Vendor
- **Precondition**: Active store
- **Main Flow**: Navigate to shipping → Select delivery zones (governorates/cities) → Set delivery fees per zone → Save
- **Alternative**: Platform default zones used

### UC-V-097: Set Delivery Fees
- **Actor**: Verified Vendor
- **Precondition**: Delivery zones configured
- **Main Flow**: Navigate to shipping → Set flat rate or zone-based fees → Save
- **Alternative**: Free delivery → Set fee to 0

### UC-V-098: Schedule Delivery Pickup
- **Actor**: Verified Vendor
- **Precondition**: Order ready for delivery
- **Main Flow**: Select order → Schedule pickup → Choose time window → Delivery provider assigned
- **Alternative**: No slots available → Contact support

### UC-V-099: Track Delivery
- **Actor**: Verified Vendor
- **Precondition**: Order dispatched
- **Main Flow**: Select order → View real-time delivery tracking → See delivery person location
- **Alternative**: GPS unavailable → Last known location shown

### UC-V-100: Handle Failed Delivery
- **Actor**: Verified Vendor
- **Precondition**: Delivery attempt failed
- **Main Flow**: Receive notification → Review failure reason → Contact customer → Reschedule or cancel
- **Alternative**: 3 failed attempts → Order returned to vendor

---

## 10. Reviews & Ratings (UC-V-101 to UC-V-103)

### UC-V-101: View Store Rating
- **Actor**: Verified Vendor
- **Precondition**: Active store
- **Main Flow**: Navigate to analytics → View overall store rating → See breakdown by factor (fulfillment, reviews, response time)
- **Alternative**: No ratings → "Ratings available after first order"

### UC-V-102: Respond to Store Review
- **Actor**: Verified Vendor
- **Precondition**: Has store review
- **Main Flow**: Select review → Write response → Submit → Response displayed
- **Alternative**: Review period expired → Cannot respond

### UC-V-103: View Review Trends
- **Actor**: Verified Vendor
- **Precondition**: Has reviews
- **Main Flow**: Navigate to analytics → View rating trends over time → See improvement areas
- **Alternative**: Insufficient data → "Trends available after 10 reviews"

---

## 11. Analytics & Reports (UC-V-104 to UC-V-105)

### UC-V-104: View Sales Analytics
- **Actor**: Verified Vendor
- **Precondition**: Has sales
- **Main Flow**: Navigate to analytics → View sales trends, revenue charts, top products, customer demographics
- **Alternative**: No sales → "Analytics available after first sale"

### UC-V-105: View Product Performance
- **Actor**: Verified Vendor
- **Precondition**: Has products
- **Main Flow**: Navigate to analytics → View product performance (views, conversion, revenue per product) → Sort by metric
- **Alternative**: No products → "Add products to see analytics"

---

## 12. Team Management (UC-V-106)

### UC-V-106: Invite Staff Member
- **Actor**: Store Owner
- **Precondition**: Active store
- **Main Flow**: Navigate to team → "Invite Staff" → Enter email/phone → Select role (Viewer/Editor/Manager) → Send invitation → Staff receives invite → Joins team
- **Alternative**: Staff limit reached → "Upgrade plan" prompt

---

## 13. Support (UC-V-107)

### UC-V-107: Contact Platform Support
- **Actor**: Verified Vendor
- **Precondition**: Has issue
- **Main Flow**: Navigate to support → Select category → Describe issue → Submit → Ticket created → Track resolution
- **Alternative**: No internet → SMS support available
