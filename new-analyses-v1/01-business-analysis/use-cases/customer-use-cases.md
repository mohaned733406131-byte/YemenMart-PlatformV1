# Customer Use Cases — YemenMart

## Overview

128 use cases for customer-facing functionality organized by category. Each use case includes ID, name, description, actor, preconditions, main flow, and alternatives.

---

## 1. Account Management (UC-C-001 to UC-C-015)

### UC-C-001: Register with Phone Number
- **Actor**: New Customer
- **Precondition**: No existing account with this phone number
- **Main Flow**: Enter phone number → Receive OTP via SMS/WhatsApp → Enter OTP → Set name and password → Account created
- **Alternative**: OTP delivery fails → Resend OTP (max 3 attempts) → Switch channel (SMS ↔ WhatsApp)

### UC-C-002: Login with Phone + Password (Primary)
- **Actor**: Existing Customer
- **Precondition**: Registered account with password set
- **Main Flow**: Enter phone number → Enter password → Access granted
- **Alternative**: 5 failed attempts → 15-minute lockout → Contact support

### UC-C-003: Forgot Password (OTP Verification)
- **Actor**: Customer who forgot password
- **Precondition**: Registered account with phone number
- **Main Flow**: Enter phone number → Receive OTP via SMS/WhatsApp → Enter OTP → Set new password → All sessions invalidated
- **Alternative**: OTP delivery fails → Resend OTP (max 3 attempts) → Switch channel (SMS ↔ WhatsApp)

### UC-C-003: Update Profile Information
- **Actor**: Logged-in Customer
- **Precondition**: Active session
- **Main Flow**: Navigate to profile → Edit name/email/phone → Save changes
- **Alternative**: Phone number change → Re-verification required

### UC-C-004: Change Password
- **Actor**: Logged-in Customer
- **Precondition**: Active session
- **Main Flow**: Navigate to security → Enter current password → Enter new password → Confirm → Password updated
- **Alternative**: Forgot password → OTP-based reset flow

### UC-C-005: Add Delivery Address
- **Actor**: Logged-in Customer
- **Precondition**: Active session
- **Main Flow**: Navigate to addresses → Add new address → Enter details (governorate, city, street, landmarks) → Save
- **Alternative**: GPS auto-fill → Manual correction → Save

### UC-C-006: Edit Delivery Address
- **Actor**: Logged-in Customer
- **Precondition**: Existing saved address
- **Main Flow**: Select address → Edit details → Save changes
- **Alternative**: Address used in active order → Cannot edit until order completes

### UC-C-007: Delete Delivery Address
- **Actor**: Logged-in Customer
- **Precondition**: Existing saved address
- **Main Flow**: Select address → Confirm deletion → Address removed
- **Alternative**: Address is default → Prompt to set new default first

### UC-C-008: Set Default Delivery Address
- **Actor**: Logged-in Customer
- **Precondition**: Multiple saved addresses
- **Main Flow**: Select address → Mark as default → Confirmation
- **Alternative**: Only one address → Automatically default

### UC-C-009: View Order History
- **Actor**: Logged-in Customer
- **Precondition**: Has previous orders
- **Main Flow**: Navigate to orders → View list with filters (status, date, vendor) → Select order for details
- **Alternative**: No orders → Empty state with "Start Shopping" CTA

### UC-C-010: Delete Account
- **Actor**: Logged-in Customer
- **Precondition**: No active orders or pending disputes
- **Main Flow**: Navigate to settings → Request account deletion → Confirm via OTP → Account deactivated → Data retained 5 years
- **Alternative**: Active orders exist → Cannot delete until resolved

### UC-C-011: Manage Notification Preferences
- **Actor**: Logged-in Customer
- **Precondition**: Active session
- **Main Flow**: Navigate to settings → Notifications → Toggle SMS/WhatsApp/Push/Email per notification type → Save
- **Alternative**: Critical notifications (security) cannot be disabled

### UC-C-012: Add Profile Photo
- **Actor**: Logged-in Customer
- **Precondition**: Active session
- **Main Flow**: Tap profile photo → Take photo or select from gallery → Crop → Upload → Photo updated
- **Alternative**: Image too large → Auto-compress → Upload

### UC-C-013: View Wallet Balance
- **Actor**: Logged-in Customer
- **Precondition**: Active session
- **Main Flow**: Navigate to wallet → View current balance and recent transactions
- **Alternative**: No wallet → Prompt to top up

### UC-C-014: Top Up Wallet
- **Actor**: Logged-in Customer
- **Precondition**: Active session
- **Main Flow**: Navigate to wallet → Select top-up amount → Choose payment method → Confirm → Wallet credited
- **Alternative**: Payment fails → Retry → Contact support

### UC-C-015: View Wallet Transaction History
- **Actor**: Logged-in Customer
- **Precondition**: Has wallet transactions
- **Main Flow**: Navigate to wallet → View transaction list with filters → Select for details
- **Alternative**: No transactions → Empty state

---

## 2. Browse & Search (UC-C-016 to UC-C-030)

### UC-C-016: Browse Categories
- **Actor**: Customer
- **Precondition**: None
- **Main Flow**: Navigate to home → Select category → View subcategories → Browse products
- **Alternative**: Deep category nesting → Maximum 3 levels

### UC-C-017: Search Products by Name
- **Actor**: Customer
- **Precondition**: None
- **Main Flow**: Tap search bar → Enter Arabic/English product name → View results → Sort/filter
- **Alternative**: No results → Suggest similar products

### UC-C-018: Search Products by Barcode
- **Actor**: Customer
- **Precondition**: None
- **Main Flow**: Tap search → Scan barcode → View matching product
- **Alternative**: No match → "Product not found" with suggestion to search by name

### UC-C-019: Filter Products by Price
- **Actor**: Customer
- **Precondition**: Viewing product list
- **Main Flow**: Apply price filter → Set min/max → View filtered results
- **Alternative**: No products in range → Expand filter suggestion

### UC-C-020: Filter Products by Rating
- **Actor**: Customer
- **Precondition**: Viewing product list
- **Main Flow**: Apply rating filter → Select minimum rating → View filtered results
- **Alternative**: No products match → Show all with note

### UC-C-021: Filter Products by Vendor
- **Actor**: Customer
- **Precondition**: Viewing product list
- **Main Flow**: Apply vendor filter → Select vendor → View vendor's products
- **Alternative**: Vendor not found → Search for vendor

### UC-C-022: Sort Products by Price
- **Actor**: Customer
- **Precondition**: Viewing product list
- **Main Flow**: Select sort option → Price low-to-high or high-to-low → View sorted results
- **Alternative**: Multiple products same price → Secondary sort by rating

### UC-C-023: Sort Products by Rating
- **Actor**: Customer
- **Precondition**: Viewing product list
- **Main Flow**: Select sort option → Highest rated first → View sorted results
- **Alternative**: No rated products → Sort by popularity

### UC-C-024: Sort Products by Newest
- **Actor**: Customer
- **Precondition**: Viewing product list
- **Main Flow**: Select sort option → Newest first → View sorted results
- **Alternative**: No new products → Show all

### UC-C-025: View Product Details
- **Actor**: Customer
- **Precondition**: None
- **Main Flow**: Tap product card → View images, name (Arabic), price, description, vendor, reviews, rating → Add to cart or buy now
- **Alternative**: Out of stock → "Notify when available" option

### UC-C-026: View Product Images
- **Actor**: Customer
- **Precondition**: Viewing product details
- **Main Flow**: Tap image → Full-screen gallery → Swipe between images → Zoom
- **Alternative**: No images → Placeholder displayed

### UC-C-027: View Product Reviews
- **Actor**: Customer
- **Precondition**: Viewing product details
- **Main Flow**: Scroll to reviews section → View rating summary → Read individual reviews
- **Alternative**: No reviews → "Be the first to review"

### UC-C-028: View Vendor Store
- **Actor**: Customer
- **Precondition**: Viewing product details
- **Main Flow**: Tap vendor name → View vendor store page → Browse vendor's products → View vendor rating
- **Alternative**: Vendor store not found → Product orphaned → Report to admin

### UC-C-029: Add to Wishlist
- **Actor**: Customer
- **Precondition**: Logged in
- **Main Flow**: Tap heart icon on product → Product added to wishlist → View wishlist anytime
- **Alternative**: Not logged in → Prompt to login first

### UC-C-030: View Wishlist
- **Actor**: Customer
- **Precondition**: Has wishlist items
- **Main Flow**: Navigate to wishlist → View saved products → Remove items or add to cart
- **Alternative**: Empty wishlist → "Start exploring" CTA

---

## 3. Product Interaction (UC-C-031 to UC-C-040)

### UC-C-031: Select Product Variant
- **Actor**: Customer
- **Precondition**: Product has variants
- **Main Flow**: View product → Select size/color/variant → Price and stock update → Add to cart
- **Alternative**: Variant out of stock → "Out of stock" label shown

### UC-C-032: Check Product Availability
- **Actor**: Customer
- **Precondition**: Viewing product details
- **Main Flow**: View product → Stock status displayed (In Stock / Low Stock / Out of Stock)
- **Alternative**: Stock status unknown → "Check availability" with vendor

### UC-C-033: View Product Specifications
- **Actor**: Customer
- **Precondition**: Viewing product details
- **Main Flow**: Scroll to specifications tab → View technical details, dimensions, materials
- **Alternative**: No specifications → "No specifications available"

### UC-C-034: Share Product
- **Actor**: Customer
- **Precondition**: Viewing product details
- **Main Flow**: Tap share icon → Select sharing method (WhatsApp, copy link) → Share
- **Alternative**: Share fails → Copy link fallback

### UC-C-035: Compare Products
- **Actor**: Customer
- **Precondition**: Multiple products selected
- **Main Flow**: Add products to comparison → View side-by-side comparison → Make decision
- **Alternative**: Same product from different vendors → Price/vendor comparison

### UC-C-036: Report Product
- **Actor**: Customer
- **Precondition**: Viewing product details
- **Main Flow**: Tap report → Select reason (wrong info, prohibited, counterfeit) → Submit report
- **Alternative**: Already reported → Show "Report submitted" status

### UC-C-037: View Similar Products
- **Actor**: Customer
- **Precondition**: Viewing product details
- **Main Flow**: Scroll to "Similar Products" section → Browse recommendations → Select product
- **Alternative**: No similar products → Section hidden

### UC-C-038: View Frequently Bought Together
- **Actor**: Customer
- **Precondition**: Viewing product details
- **Main Flow**: View "Frequently Bought Together" section → Add bundle to cart
- **Alternative**: No data → Section hidden

### UC-C-039: Check Delivery Estimate
- **Actor**: Customer
- **Precondition**: Viewing product details
- **Main Flow**: Enter delivery location → View estimated delivery time and cost
- **Alternative**: Not deliverable to location → "Not available in your area"

### UC-C-040: Notify When Available
- **Actor**: Customer
- **Precondition**: Product is out of stock
- **Main Flow**: Tap "Notify Me" → Enter notification preference → Receive notification when restocked
- **Alternative**: Vendor doesn't restock → Notification never sent → Auto-cleanup after 90 days

---

## 4. Cart Management (UC-C-041 to UC-C-055)

### UC-C-041: Add Product to Cart
- **Actor**: Customer
- **Precondition**: Product in stock
- **Main Flow**: View product → Select variant → Tap "Add to Cart" → Cart updated → View cart badge
- **Alternative**: Variant out of stock → Cannot add

### UC-C-042: View Cart
- **Actor**: Customer
- **Precondition**: Has items in cart
- **Main Flow**: Tap cart icon → View items grouped by vendor → See subtotals and total
- **Alternative**: Empty cart → "Your cart is empty" message

### UC-C-043: Update Cart Quantity
- **Actor**: Customer
- **Precondition**: Item in cart
- **Main Flow**: In cart → Tap quantity selector → Increase/decrease → Cart totals update
- **Alternative**: Max quantity reached → "Maximum limit reached" message

### UC-C-044: Remove Item from Cart
- **Actor**: Customer
- **Precondition**: Item in cart
- **Main Flow**: In cart → Tap remove icon → Confirm removal → Item removed, totals update
- **Alternative**: Item was last from vendor → Vendor sub-order removed

### UC-C-045: Apply Coupon to Cart
- **Actor**: Customer
- **Precondition**: Has valid coupon code
- **Main Flow**: In cart → Enter coupon code → Apply → Discount shown → Updated total
- **Alternative**: Invalid coupon → "Invalid coupon" error → Expired coupon → "Coupon expired" error

### UC-C-046: Remove Coupon from Cart
- **Actor**: Customer
- **Precondition**: Coupon applied to cart
- **Main Flow**: In cart → Tap remove coupon → Discount removed → Original total restored
- **Alternative**: None

### UC-C-047: Move to Wishlist
- **Actor**: Customer
- **Precondition**: Item in cart
- **Main Flow**: In cart → Tap "Move to Wishlist" → Item removed from cart, added to wishlist
- **Alternative**: Wishlist full → "Wishlist limit reached" message

### UC-C-048: Save for Later
- **Actor**: Customer
- **Precondition**: Item in cart
- **Main Flow**: In cart → Tap "Save for Later" → Item moved to saved items section
- **Alternative**: None

### UC-C-049: View Saved Items
- **Actor**: Customer
- **Precondition**: Has saved items
- **Main Flow**: In cart → View "Saved for Later" section → Move to cart or remove
- **Alternative**: No saved items → Section hidden

### UC-C-050: Clear Cart
- **Actor**: Customer
- **Precondition**: Has items in cart
- **Main Flow**: In cart → Tap "Clear Cart" → Confirm → All items removed
- **Alternative**: Cancel → Cart unchanged

### UC-C-051: View Vendor Sub-Totals
- **Actor**: Customer
- **Precondition**: Multiple vendor items in cart
- **Main Flow**: In cart → View items grouped by vendor → See vendor sub-totals
- **Alternative**: Single vendor → No grouping needed

### UC-C-052: View Delivery Fees
- **Actor**: Customer
- **Precondition**: Items in cart
- **Main Flow**: In cart → View delivery fee per vendor → View total delivery fee
- **Alternative**: Free delivery threshold met → "Free delivery" message

### UC-C-053: View Cart Summary
- **Actor**: Customer
- **Precondition**: Items in cart
- **Main Flow**: In cart → View summary: items total, delivery fees, coupon discount, grand total
- **Alternative**: None

### UC-C-054: Proceed to Checkout
- **Actor**: Customer
- **Precondition**: Items in cart
- **Main Flow**: In cart → Tap "Checkout" → Proceed to checkout flow
- **Alternative**: Cart empty → Cannot proceed → Login required → Prompt to login

### UC-C-055: Share Cart
- **Actor**: Customer
- **Precondition**: Items in cart
- **Main Flow**: In cart → Tap share → Share cart contents with friend
- **Alternative**: Share fails → Copy link fallback

---

## 5. Checkout & Payment (UC-C-056 to UC-C-070)

### UC-C-056: Select Delivery Address
- **Actor**: Customer
- **Precondition**: In checkout flow
- **Main Flow**: View saved addresses → Select address or add new → Confirm selection
- **Alternative**: No addresses → Prompt to add address first

### UC-C-057: Select Delivery Speed
- **Actor**: Customer
- **Precondition**: In checkout flow
- **Main Flow**: View delivery options (standard, express) → Select → Delivery fee updates
- **Alternative**: Only one option → Pre-selected

### UC-C-058: Select Payment Method
- **Actor**: Customer
- **Precondition**: In checkout flow
- **Main Flow**: View wallet balance → Confirm wallet payment → Proceed
- **Alternative**: Insufficient balance → Prompt to top up first

### UC-C-059: Apply Coupon at Checkout
- **Actor**: Customer
- **Precondition**: Has coupon code
- **Main Flow**: Enter coupon code → Apply → Discount applied to total → Proceed
- **Alternative**: Invalid/expired coupon → Error message

### UC-C-060: Use Loyalty Points
- **Actor**: Customer
- **Precondition**: Has redeemable points
- **Main Flow**: Toggle "Use Points" → Enter points amount (max 30% of order) → Discount applied → Proceed
- **Alternative**: No points → Option hidden

### UC-C-061: Review Order Summary
- **Actor**: Customer
- **Precondition**: In checkout flow
- **Main Flow**: View items, delivery address, delivery fee, payment method, coupon discount, points, grand total → Confirm
- **Alternative**: Discrepancy → Edit before confirming

### UC-C-062: Place Order
- **Actor**: Customer
- **Precondition**: All checkout details confirmed
- **Main Flow**: Tap "Place Order" → Payment processed → Order confirmed → Confirmation screen → Order details
- **Alternative**: Payment fails → "Payment failed" → Retry or contact support

### UC-C-063: View Order Confirmation
- **Actor**: Customer
- **Precondition**: Order placed successfully
- **Main Flow**: View confirmation screen → Order number, estimated delivery, items summary → Continue shopping or view order
- **Alternative**: None

### UC-C-064: Pay with Multiple Wallets
- **Actor**: Customer
- **Precondition**: In checkout, has multiple wallet accounts
- **Main Flow**: Select primary wallet → Add secondary wallet → Split payment → Confirm
- **Alternative**: Single wallet → Standard flow

### UC-C-065: Schedule Delivery
- **Actor**: Customer
- **Precondition**: In checkout flow
- **Main Flow**: Select "Schedule" → Choose date and time window → Confirm
- **Alternative**: Schedule not available for area → Standard delivery only

### UC-C-066: Add Order Note
- **Actor**: Customer
- **Precondition**: In checkout flow
- **Main Flow**: Tap "Add Note" → Enter delivery instructions → Proceed
- **Alternative**: Note exceeds character limit → Truncated with warning

### UC-C-067: Request Gift Wrapping
- **Actor**: Customer
- **Precondition**: In checkout flow, gift wrapping available
- **Main Flow**: Toggle "Gift Wrapping" → Additional fee added → Proceed
- **Alternative**: Not available for vendor → Option hidden

### UC-C-068: View Delivery ETA Before Payment
- **Actor**: Customer
- **Precondition**: In checkout flow
- **Main Flow**: View estimated delivery date range before confirming payment
- **Alternative**: ETA unavailable → "Delivery within X days" estimate

### UC-C-069: Cancel Checkout
- **Actor**: Customer
- **Precondition**: In checkout flow
- **Main Flow**: Tap back/cancel → Return to cart → Cart preserved
- **Alternative**: Session timeout → Cart preserved, redirect to cart

### UC-C-070: Receive Order Confirmation Notification
- **Actor**: Customer
- **Precondition**: Order placed successfully
- **Main Flow**: Receive SMS/WhatsApp confirmation → Contains order number, summary, estimated delivery
- **Alternative**: Notification delivery fails → Confirmation visible in app

---

## 6. Order Management (UC-C-071 to UC-C-085)

### UC-C-071: Track Order Status
- **Actor**: Customer
- **Precondition**: Has active order
- **Main Flow**: Navigate to orders → Select order → View current status and timeline
- **Alternative**: No active orders → "No active orders" message

### UC-C-072: Track Delivery in Real-Time
- **Actor**: Customer
- **Precondition**: Order in "In Transit" or "Out for Delivery"
- **Main Flow**: Select order → Tap "Track Delivery" → View live map with delivery person location
- **Alternative**: GPS unavailable → Show last known location

### UC-C-073: Cancel Order
- **Actor**: Customer
- **Precondition**: Order in Pending state (within 15 minutes)
- **Main Flow**: Select order → Tap "Cancel" → Confirm → Order cancelled → Refund initiated
- **Alternative**: Order already confirmed → Cannot cancel; contact support

### UC-C-074: Contact Vendor About Order
- **Actor**: Customer
- **Precondition**: Has active order
- **Main Flow**: Select order → Tap "Contact Vendor" → In-app chat or WhatsApp → Communicate
- **Alternative**: Vendor not responding → Escalate to support after 24 hours

### UC-C-075: Request Order Modification
- **Actor**: Customer
- **Precondition**: Order in Pending state
- **Main Flow**: Select order → Tap "Modify" → Change quantity/address → Submit modification → Vendor approves/rejects
- **Alternative**: Vendor rejects → Original order maintained

### UC-C-076: View Invoice
- **Actor**: Customer
- **Precondition**: Has order
- **Main Flow**: Select order → Tap "View Invoice" → Download/share invoice PDF
- **Alternative**: Invoice not generated → Generate on demand

### UC-C-077: Confirm Delivery Receipt
- **Actor**: Customer
- **Precondition**: Order delivered
- **Main Flow**: Receive delivery → Enter OTP (if required) → Sign (if required) → Confirm receipt → Escrow timer starts
- **Alternative**: Refuse delivery → Return process initiated

### UC-C-078: Rate Order After Delivery
- **Actor**: Customer
- **Precondition**: Order delivered + receipt confirmed
- **Main Flow**: Receive rating prompt → Rate vendor (1-5 stars) → Rate delivery (1-5 stars) → Write optional review → Submit
- **Alternative**: Skip rating → Prompt again in 24 hours → Auto-close after 7 days

### UC-C-079: Report Delivery Issue
- **Actor**: Customer
- **Precondition**: Delivery attempted or completed
- **Main Flow**: Select order → Tap "Report Issue" → Select issue type → Describe → Submit → Support ticket created
- **Alternative**: No active order → Cannot report

### UC-C-080: View Delivery Attempts
- **Actor**: Customer
- **Precondition**: Delivery attempted
- **Main Flow**: Select order → View delivery attempt history → See timestamps and outcomes
- **Alternative**: No attempts → "Awaiting delivery"

### UC-C-081: Reschedule Delivery
- **Actor**: Customer
- **Precondition**: Order in "Out for Delivery" state
- **Main Flow**: Select order → Tap "Reschedule" → Choose new delivery window → Confirm
- **Alternative**: Reschedule not available → Contact support

### UC-C-082: View Order Timeline
- **Actor**: Customer
- **Precondition**: Has order
- **Main Flow**: Select order → View timeline showing all status changes with timestamps
- **Alternative**: None

### UC-C-083: Reorder Previous Order
- **Actor**: Customer
- **Precondition**: Has completed order
- **Main Flow**: Select completed order → Tap "Reorder" → All items added to cart → Proceed to checkout
- **Alternative**: Some items unavailable → Show which items can be reordered

### UC-C-084: View All Orders
- **Actor**: Customer
- **Precondition**: Has orders
- **Main Flow**: Navigate to orders → View all orders with filters (active, completed, cancelled, returned)
- **Alternative**: No orders → Empty state

### UC-C-085: Export Order History
- **Actor**: Customer
- **Precondition**: Has orders
- **Main Flow**: Navigate to orders → Tap "Export" → Download order history as CSV/PDF
- **Alternative**: No orders → Export option hidden

---

## 7. Returns & Disputes (UC-C-086 to UC-C-098)

### UC-C-086: Initiate Return
- **Actor**: Customer
- **Precondition**: Order delivered within 7 days + eligible product
- **Main Flow**: Select order → Tap "Return" → Select reason → Upload photos → Submit return request
- **Alternative**: Return window expired → "Return period ended" → Product marked "No Return" → Cannot return

### UC-C-087: Select Return Reason
- **Actor**: Customer
- **Precondition**: Initiated return
- **Main Flow**: View return reasons (defective, wrong item, not as described, changed mind, etc.) → Select → Add description
- **Alternative**: Reason not listed → Select "Other" → Describe

### UC-C-088: Upload Return Evidence
- **Actor**: Customer
- **Precondition**: Initiated return
- **Main Flow**: Upload photos of product issue → Add description → Submit
- **Alternative**: No photos → Description only → Lower priority processing

### UC-C-089: Track Return Status
- **Actor**: Customer
- **Precondition**: Return initiated
- **Main Flow**: Select order → View return status (Pending Review → Approved/Rejected → Pickup Scheduled → Completed)
- **Alternative**: Return rejected → See reason → Appeal option

### UC-C-090: Schedule Return Pickup
- **Actor**: Customer
- **Precondition**: Return approved
- **Main Flow**: Select return → Choose pickup window → Confirm → Delivery provider assigned
- **Alternative**: No pickup slots → Contact support for manual scheduling

### UC-C-091: Receive Refund
- **Actor**: Customer
- **Precondition**: Return completed
- **Main Flow**: Return confirmed → Refund credited to wallet → Notification received → Balance updated
- **Alternative**: Refund delayed → Contact support → Escalation

### UC-C-092: Open Dispute
- **Actor**: Customer
- **Precondition**: Order issue unresolved with vendor
- **Main Flow**: Select order → Tap "Open Dispute" → Describe issue → Upload evidence → Submit → Admin reviews
- **Alternative**: Vendor resolves before admin review → Dispute closed

### UC-C-093: Respond to Dispute
- **Actor**: Customer
- **Precondition**: Active dispute with admin request for info
- **Main Flow**: Receive notification → Open dispute → Add response/evidence → Submit
- **Alternative**: No response in 48 hours → Dispute closed with original evidence

### UC-C-094: View Dispute Resolution
- **Actor**: Customer
- **Precondition**: Dispute resolved
- **Main Flow**: Select order → View resolution (refund, partial refund, no refund) → Accept or appeal
- **Alternative**: Appeal within 7 days → escalated to senior admin

### UC-C-095: Cancel Return Request
- **Actor**: Customer
- **Precondition**: Return in Pending Review status
- **Main Flow**: Select return → Tap "Cancel Return" → Confirm → Return cancelled
- **Alternative**: Return already approved → Cannot cancel

### UC-C-096: Dispute Partial Delivery
- **Actor**: Customer
- **Precondition**: Order partially delivered
- **Main Flow**: Select order → Report missing items → Describe → Submit → Admin reviews
- **Alternative**: Vendor confirms partial shipment → Refund for missing items

### UC-C-097: Dispute Damaged Item
- **Actor**: Customer
- **Precondition**: Received damaged item
- **Main Flow**: Select order → Report damage → Upload photos → Submit → Return initiated
- **Alternative**: Vendor offers replacement → Accept or decline

### UC-C-098: View Dispute History
- **Actor**: Customer
- **Precondition**: Has dispute history
- **Main Flow**: Navigate to disputes → View all past and current disputes with statuses
- **Alternative**: No disputes → "No disputes" message

---

## 8. Wallet & Payments (UC-C-099 to UC-C-105)

### UC-C-099: View Wallet Balance
- **Actor**: Customer
- **Precondition**: Has wallet
- **Main Flow**: Navigate to wallet → View current balance in YER/SAR/USD
- **Alternative**: No wallet → Prompt to create

### UC-C-100: Top Up Wallet via Agent
- **Actor**: Customer
- **Precondition**: Has wallet
- **Main Flow**: Navigate to wallet → Select "Top Up" → Choose "Agent" → Enter agent code and amount → Visit agent → Pay cash → Wallet credited
- **Alternative**: Agent unavailable → Try another agent or method

### UC-C-101: Top Up Wallet via Mobile
- **Actor**: Customer
- **Precondition**: Has wallet
- **Main Flow**: Navigate to wallet → Select "Top Up" → Choose "Mobile" → Enter amount → Confirm → Wallet credited
- **Alternative**: Mobile payment fails → Retry → Contact support

### UC-C-102: Transfer to Another User
- **Actor**: Customer
- **Precondition**: Has sufficient balance
- **Main Flow**: Navigate to wallet → Select "Transfer" → Enter recipient phone → Enter amount → Confirm → Funds transferred
- **Alternative**: Insufficient balance → "Insufficient funds" → Recipient not found → "User not found"

### UC-C-103: View Transaction History
- **Actor**: Customer
- **Precondition**: Has transactions
- **Main Flow**: Navigate to wallet → View transactions list → Filter by type (top-up, payment, refund, transfer) → Select for details
- **Alternative**: No transactions → Empty state

### UC-C-104: Download Wallet Statement
- **Actor**: Customer
- **Precondition**: Has transactions
- **Main Flow**: Navigate to wallet → Tap "Download Statement" → Select date range → Download PDF/CSV
- **Alternative**: No transactions in range → "No data" message

### UC-C-105: Set Wallet PIN
- **Actor**: Customer
- **Precondition**: Has wallet
- **Main Flow**: Navigate to wallet settings → Set 4-digit PIN → Confirm PIN → PIN enabled for transactions
- **Alternative**: PIN forgotten → OTP-based reset

---

## 9. Reviews & Ratings (UC-C-106 to UC-C-115)

### UC-C-106: Write Product Review
- **Actor**: Customer
- **Precondition**: Purchased product + delivered
- **Main Flow**: Select order → Tap "Review" → Rate (1-5 stars) → Write text review → Upload photos → Submit
- **Alternative**: Already reviewed → Edit review option

### UC-C-107: Edit Product Review
- **Actor**: Customer
- **Precondition**: Has written review
- **Main Flow**: Select product → View my review → Edit → Save changes
- **Alternative**: Review period expired (30 days) → Cannot edit

### UC-C-108: Delete Product Review
- **Actor**: Customer
- **Precondition**: Has written review
- **Main Flow**: Select product → View my review → Delete → Confirm → Review removed
- **Alternative**: None

### UC-C-109: Rate Vendor
- **Actor**: Customer
- **Precondition**: Completed order with vendor
- **Main Flow**: After delivery → Rate vendor (1-5 stars) → Write optional review → Submit
- **Alternative**: Skip → Prompt again later

### UC-C-110: Rate Delivery Person
- **Actor**: Customer
- **Precondition**: Order delivered
- **Main Flow**: After delivery → Rate delivery (1-5 stars) → Write optional comment → Submit
- **Alternative**: Skip → No further prompts

### UC-C-111: View My Reviews
- **Actor**: Customer
- **Precondition**: Has written reviews
- **Main Flow**: Navigate to profile → View "My Reviews" → See all reviews with ratings
- **Alternative**: No reviews → "No reviews yet" message

### UC-C-112: Report Review
- **Actor**: Customer
- **Precondition**: Viewing someone else's review
- **Main Flow**: Select review → Tap "Report" → Select reason → Submit → Admin moderation
- **Alternative**: Already reported → "Report submitted" status

### UC-C-113: Vote Review as Helpful
- **Actor**: Customer
- **Precondition**: Viewing someone else's review
- **Main Flow**: Select review → Tap "Helpful" → Vote recorded
- **Alternative**: Already voted → Unvote option

### UC-C-114: View Vendor Reviews
- **Actor**: Customer
- **Precondition**: Viewing vendor store
- **Main Flow**: Navigate to vendor store → View reviews section → Read all vendor reviews
- **Alternative**: No reviews → "No reviews yet"

### UC-C-115: Sort Reviews
- **Actor**: Customer
- **Precondition**: Viewing reviews
- **Main Flow**: Select sort option (newest, highest rated, most helpful) → View sorted reviews
- **Alternative**: No reviews → Empty state

---

## 10. Loyalty & Rewards (UC-C-116 to UC-C-121)

### UC-C-116: View Loyalty Points Balance
- **Actor**: Customer
- **Precondition**: Has loyalty points
- **Main Flow**: Navigate to loyalty section → View current points balance and tier status
- **Alternative**: No points → "Start earning" CTA

### UC-C-117: View Tier Status
- **Actor**: Customer
- **Precondition**: Has loyalty account
- **Main Flow**: Navigate to loyalty → View current tier (Bronze/Silver/Gold/Platinum) → View progress to next tier
- **Alternative**: At maximum tier → "Top member" status

### UC-C-118: Redeem Points for Discount
- **Actor**: Customer
- **Precondition**: Has redeemable points
- **Main Flow**: At checkout → Toggle "Use Points" → Enter amount (max 30% of order) → Discount applied → Points deducted
- **Alternative**: Insufficient points → "Not enough points"

### UC-C-119: View Points History
- **Actor**: Customer
- **Precondition**: Has point transactions
- **Main Flow**: Navigate to loyalty → View points history → See earnings and redemptions
- **Alternative**: No history → Empty state

### UC-C-120: Use Referral Code
- **Actor**: New Customer
- **Precondition**: Has referral code from friend
- **Main Flow**: Register → Enter referral code → Account linked → Both users receive bonus
- **Alternative**: Invalid code → "Invalid referral code" → Expired code → "Code expired"

### UC-C-121: Share Referral Code
- **Actor**: Customer
- **Precondition**: Has referral code
- **Main Flow**: Navigate to referral section → View referral code → Share via WhatsApp/SMS → Friend registers → Both earn bonus
- **Alternative**: Monthly limit reached (5) → "Referral limit reached"

---

## 11. Customer Support (UC-C-122 to UC-C-126)

### UC-C-122: Contact Support
- **Actor**: Customer
- **Precondition**: Has issue
- **Main Flow**: Navigate to support → Select issue category → Describe issue → Submit → Ticket created
- **Alternative**: No internet → SMS support available

### UC-C-123: View Support Tickets
- **Actor**: Customer
- **Precondition**: Has support tickets
- **Main Flow**: Navigate to support → View ticket list → Select for details and updates
- **Alternative**: No tickets → "No tickets" message

### UC-C-124: Respond to Support Ticket
- **Actor**: Customer
- **Precondition**: Active ticket with admin response
- **Main Flow**: Receive notification → Open ticket → Add response → Submit
- **Alternative**: Ticket closed → Cannot respond; open new ticket

### UC-C-125: Rate Support Experience
- **Actor**: Customer
- **Precondition**: Support ticket resolved
- **Main Flow**: Receive rating prompt → Rate experience (1-5 stars) → Write optional feedback → Submit
- **Alternative**: Skip rating → No further prompts

### UC-C-126: View FAQ
- **Actor**: Customer
- **Precondition**: None
- **Main Flow**: Navigate to support → View FAQ → Search or browse topics
- **Alternative**: FAQ not helpful → Contact support

---

## 12. Notifications (UC-C-127 to UC-C-128)

### UC-C-127: View Notifications
- **Actor**: Customer
- **Precondition**: Has notifications
- **Main Flow**: Tap notification bell → View notification list → Mark as read → Tap to navigate to relevant screen
- **Alternative**: No notifications → "No notifications" message

### UC-C-128: Clear All Notifications
- **Actor**: Customer
- **Precondition**: Has unread notifications
- **Main Flow**: Tap notification bell → Tap "Clear All" → All notifications marked as read
- **Alternative**: No unread notifications → "Clear All" hidden
