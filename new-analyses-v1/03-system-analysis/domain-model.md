# Domain Model — YemenMart Platform

## 1. Overview

13 bounded contexts organized by domain cohesion. Each context owns its data, enforces invariants, and communicates through published events or query APIs.

## 2. Bounded Context Map

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                           YEMENMART DOMAIN MODEL                                │
│                                                                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │
│  │  B01         │  │  B02         │  │  B03         │  │  B04         │       │
│  │  Identity &  │←→│  Marketplace │←→│  Catalog     │←→│  Commerce    │       │
│  │  Access      │  │              │  │              │  │              │       │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘       │
│         │                 │                 │                 │                │
│         │    ┌────────────┴─────────────────┴─────────────────┤                │
│         │    │                                                │                │
│  ┌──────▼────▼──┐  ┌──────────────┐  ┌──────────────┐  ┌────▼──────────┐     │
│  │  B05         │  │  B06         │  │  B07         │  │  B08          │     │
│  │  Payment     │←→│  Finance     │←→│  Logistics   │←→│  Inventory    │     │
│  │              │  │              │  │              │  │               │     │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘  └──────┬────────┘     │
│         │                 │                 │                 │               │
│         │    ┌────────────┴─────────────────┴─────────────────┤               │
│         │    │                                                │               │
│  ┌──────▼────▼──┐  ┌──────────────┐  ┌──────────────┐  ┌────▼──────────┐     │
│  │  B09         │  │  B10         │  │  B11         │  │  B12          │     │
│  │  Storefront  │←→│  Engagement  │←→│  Content     │←→│  Support      │     │
│  │              │  │              │  │              │  │               │     │
│  └──────────────┘  └──────────────┘  └──────────────┘  └───────────────┘     │
│                                    │                                          │
│                              ┌─────▼─────┐                                   │
│                              │  B13      │                                   │
│                              │ Promotion │                                   │
│                              └───────────┘                                   │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Bounded Context Definitions

### B01: Identity & Access

**Purpose**: Manages authentication, authorization, user identity, sessions, and audit logging.

| Entity | Description | Key Fields |
|--------|-------------|------------|
| `User` | Registered platform user (customer, vendor, admin, agent) | id, phone, email, name, role, status, locale, createdAt |
| `Session` | Active authenticated session | id, userId, token, deviceInfo, ip, expiresAt |
| `OTP` | One-time password for phone verification | id, phone, code, channel (sms/whatsapp), expiresAt, attempts, status |
| `AuditLog` | Immutable audit trail of all state changes | id, actorId, action, entity, entityId, payload, ip, timestamp |
| `RefreshToken` | Long-lived token for session renewal | id, userId, token, expiresAt, revokedAt |

**Invariants**:
- One active session per device; old session revoked on new login
- OTP expires after 5 minutes; max 5 attempts
- Audit logs are append-only; no updates or deletes
- Passwords hashed with bcrypt (cost 12)
- JWT access tokens expire after 15 minutes

**Domain Events Published**:
- `user.registered`
- `user.phone_verified`
- `user.login`
- `user.logout`
- `user.role_changed`

---

### B02: Marketplace

**Purpose**: Manages vendor registration, store creation, KYC verification, and vendor reputation.

| Entity | Description | Key Fields |
|--------|-------------|------------|
| `Vendor` | Seller account tied to a user | id, userId, businessName, tradeLicense, status (pending/approved/suspended), rating |
| `Store` | Virtual storefront within the marketplace | id, vendorId, name, slug, description, logo, banner, isActive, commissionRate |
| `KYC` | Know Your Customer submission for vendor verification | id, vendorId, documents[], status (pending/rejected/approved), reviewedBy, reviewedAt |
| `Badge` | Achievement or trust indicator for vendors | id, vendorId, type (top_seller/fast_shipper/verified), awardedAt, expiresAt |
| `VendorAnalytics` | Aggregated vendor performance metrics | vendorId, totalOrders, avgDeliveryTime, cancellationRate, period |

**Invariants**:
- Vendor must have approved KYC before publishing products
- Store slug must be unique across the platform
- KYC documents must be PDF/JPEG/PNG, max 5MB each
- Vendor suspended → all stores deactivated automatically
- Commission rate per store: 5%–25% range

**Domain Events Published**:
- `vendor.registered`
- `vendor.kyc_submitted`
- `vendor.kyc_approved`
- `vendor.kyc_rejected`
- `vendor.suspended`
- `store.created`
- `store.activated`
- `store.deactivated`

---

### B03: Catalog

**Purpose**: Manages product listings, variants, categories, pricing, and search indexing.

| Entity | Description | Key Fields |
|--------|-------------|------------|
| `Product` | Base product listing (non-purchasable without variants) | id, storeId, name, slug, description, brand, status (draft/active/archived), createdAt |
| `Variant` | Purchasable unit with specific attributes | id, productId, sku, price, compareAtPrice, weight, dimensions, attributes{} |
| `Category` | Hierarchical product classification | id, parentId, name, slug, icon, isActive, level, path |
| `Offer` | Time-limited pricing or promotional offer on a variant | id, variantId, offerPrice, startsAt, endsAt, maxQuantity |
| `ProductImage` | Media attachment for products | id, productId, variantId?, url, alt, sortOrder, isPrimary |
| `Brand` | Product brand entity | id, name, slug, logo, isActive |

**Invariants**:
- Product must belong to exactly one store
- At least one variant required to make product purchasable
- SKU must be unique per store
- Category nesting max depth: 5 levels
- Product images max: 10 per product, max 5MB each
- Price must be ≥ 0; compareAtPrice must be > price

**Domain Events Published**:
- `product.created`
- `product.updated`
- `product.published`
- `product.archived`
- `variant.created`
- `variant.stock_changed`
- `category.created`
- `category.updated`

**Reads From**: B02 (Store), B08 (Stock)

---

### B04: Commerce

**Purpose**: Manages the full order lifecycle — from cart checkout to delivery completion, including returns.

| Entity | Description | Key Fields |
|--------|-------------|------------|
| `MasterOrder` | Top-level order aggregating all items from a single checkout | id, userId, status, totalAmount, discountAmount, shippingFee, grandTotal, createdAt |
| `SubOrder` | Per-vendor split of a master order | id, masterOrderId, vendorId, storeId, status, subtotal, commission, netAmount |
| `OrderItem` | Individual line item in a sub-order | id, subOrderId, variantId, productName, quantity, unitPrice, totalPrice |
| `ReturnRequest` | Request to return/refund an order item | id, orderItemId, reason, status (pending/approved/rejected/completed), evidence[] |
| `OrderTimeline` | Chronological status change log | id, orderId, fromStatus, toStatus, actorId, timestamp, note |

**Invariants**:
- MasterOrder status flow: `pending → confirmed → processing → shipped → delivered → completed`
- SubOrder must all be completed for MasterOrder to be completed
- ReturnRequest allowed within 14 days of delivery
- Order items cannot be modified after payment confirmation
- Grand total = sum(subOrder.subtotals) + shippingFee - discountAmount

**Domain Events Published**:
- `order.placed`
- `order.confirmed`
- `order.paid`
- `order.shipped`
- `order.delivered`
- `order.completed`
- `order.cancelled`
- `return.requested`
- `return.approved`
- `return.rejected`
- `return.completed`

**Reads From**: B02 (Store/Vendor), B03 (Product/Variant), B08 (Stock), B13 (Promotion/Coupon)

---

### B05: Payment

**Purpose**: Manages wallet balances, escrow for order funds, payment processing, and refunds.

| Entity | Description | Key Fields |
|--------|-------------|------------|
| `Wallet` | User or vendor wallet holding balance | id, userId, balance, currency (YER), status (active/frozen), createdAt |
| `Escrow` | Funds held during order fulfillment | id, orderId, amount, status (held/released/refunded), createdAt, releasedAt |
| `Transaction` | Ledger entry for all money movements | id, walletId, type (credit/debit), amount, balanceAfter, referenceType, referenceId, createdAt |
| `PaymentMethod` | Saved payment instrument | id, userId, type (mfloos/onecash), identifier, isDefault |
| `Refund` | Refund transaction linked to a return | id, escrowId, walletId, amount, status (pending/completed/failed), createdAt |

**Invariants**:
- Wallet balance must never go negative
- Escrow funds released only on delivery confirmation
- Refund amount ≤ escrow amount
- Transaction ledger is append-only
- Payment method identifier must be encrypted at rest
- Currency: YER (Yemeni Rial) only

**Domain Events Published**:
- `wallet.topup_initiated`
- `wallet.topup_completed`
- `wallet.debit`
- `escrow.held`
- `escrow.released`
- `escrow.refunded`
- `refund.initiated`
- `refund.completed`
- `refund.failed`

**Reads From**: B04 (Order), B06 (Finance rules)

---

### B06: Finance

**Purpose**: Manages commission calculations, vendor payouts, invoicing, and the general ledger.

| Entity | Description | Key Fields |
|--------|-------------|------------|
| `Commission` | Fee earned by the platform per completed sub-order | id, subOrderId, vendorId, rate, amount, status (pending/settled), settledAt |
| `Payout` | Batch disbursement to a vendor | id, vendorId, amount, period (weekly/biweekly), status (pending/processing/completed), paidAt |
| `PayoutItem` | Individual commission entry included in a payout | id, payoutId, commissionId, amount |
| `Invoice` | Vendor-facing invoice for completed sales | id, vendorId, payoutId, items[], subtotal, commission, netPayable, issuedAt |
| `LedgerEntry` | Double-entry bookkeeping record | id, account, debit, credit, referenceType, referenceId, createdAt |

**Invariants**:
- Commission rate set per store in B02
- Payout only generated for vendors with settled commissions ≥ minimum threshold (500 YER)
- Ledger must always balance (total debits = total credits)
- Invoice generated automatically on payout creation
- Commission settled 7 days after order delivery (dispute window)

**Domain Events Published**:
- `commission.calculated`
- `commission.settled`
- `payout.created`
- `payout.processing`
- `payout.completed`
- `payout.failed`
- `invoice.generated`

**Reads From**: B02 (Store commission rate), B04 (SubOrder), B05 (Wallet/Transaction)

---

### B07: Logistics

**Purpose**: Manages delivery providers, zone coverage, assignment algorithms, and proof of delivery.

| Entity | Description | Key Fields |
|--------|-------------|------------|
| `DeliveryProvider` | External or in-house courier service | id, name, type (api/courier), status (active/inactive), apiEndpoint |
| `Zone` | Geographic delivery zone | id, name, polygon (GeoJSON), baseFee, perKmFee, estimatedDays |
| `Assignment` | Link between a sub-order and delivery provider | id, subOrderId, providerId, zoneId, status, assignedAt, pickedUpAt, deliveredAt |
| `ProofOfDelivery` | Evidence of successful delivery | id, assignmentId, signatureUrl, photoUrl, recipientName, lat, lng, timestamp |
| `DeliveryTracking` | Real-time tracking updates | id, assignmentId, status, lat, lng, timestamp |

**Invariants**:
- Assignment requires approved sub-order (paid status)
- POD required for delivery confirmation
- Zone fee calculated: baseFee + (perKmFee × distance)
- Provider must be active to receive assignments
- Tracking updates stored every 30 seconds during active delivery

**Domain Events Published**:
- `delivery.assigned`
- `delivery.picked_up`
- `delivery.in_transit`
- `delivery.delivered`
- `delivery.failed`
- `delivery.returned`

**Reads From**: B04 (SubOrder), B02 (Store address)

---

### B08: Inventory

**Purpose**: Manages stock levels, reservations during checkout, and warehouse operations.

| Entity | Description | Key Fields |
|--------|-------------|------------|
| `Stock` | Current available quantity per variant per warehouse | id, variantId, warehouseId, quantity, reservedQty, availableQty |
| `Reservation` | Temporary hold on stock during checkout | id, stockId, orderId, quantity, expiresAt, status (active/released/committed) |
| `Warehouse` | Physical storage location | id, name, address, zone, isActive |
| `StockAdjustment` | Manual or system-initiated inventory change | id, stockId, adjustment, reason, actorId, createdAt |
| `StockMovement` | Immutable log of all stock changes | id, stockId, type (in/out/transfer), quantity, referenceType, referenceId, createdAt |

**Invariants**:
- `availableQty = quantity - reservedQty`
- Reservation expires after 15 minutes if checkout not completed
- Stock cannot go below 0
- Reservation → Commit on payment success
- Reservation → Release on payment failure or timeout
- Stock adjustments require admin authorization

**Domain Events Published**:
- `stock.reserved`
- `stock.committed`
- `stock.released`
- `stock.low` (threshold per variant)
- `stock.out_of_stock`
- `stock.adjusted`

**Reads From**: B03 (Variant), B04 (OrderItem)

---

### B09: Storefront

**Purpose**: Manages shopping cart, wishlist, and store following for the customer-facing UI.

| Entity | Description | Key Fields |
|--------|-------------|------------|
| `Cart` | Active shopping cart per user | id, userId, items[], totalAmount, currency, updatedAt |
| `CartItem` | Individual item in the cart | id, cartId, variantId, quantity, unitPrice |
| `Wishlist` | Saved items for later | id, userId, items[], createdAt |
| `WishlistItem` | Individual saved item | id, wishlistId, variantId, addedAt |
| `StoreFollow` | Customer following a vendor's store | id, userId, storeId, followedAt |
| `RecentlyViewed` | Browsing history | id, userId, variantId, viewedAt |

**Invariants**:
- One active cart per user
- Cart item quantity cannot exceed variant stock
- Wishlist max 200 items per user
- StoreFollow is toggle (follow/unfollow)
- Cart refreshes prices from B03 on each view

**Domain Events Published**:
- `cart.item_added`
- `cart.item_removed`
- `cart.checkout_initiated`
- `wishlist.item_added`
- `wishlist.item_removed`
- `store.followed`
- `store.unfollowed`

**Reads From**: B03 (Product/Variant/Offer), B08 (Stock)

---

### B10: Engagement

**Purpose**: Manages product reviews, customer loyalty points, and tier progression.

| Entity | Description | Key Fields |
|--------|-------------|------------|
| `Review` | Customer review of a purchased product | id, userId, variantId, orderId, rating (1–5), title, body, images[], isVerified, createdAt |
| `ReviewVote` | Helpful/not helpful vote on a review | id, reviewId, userId, vote (up/down), createdAt |
| `LoyaltyAccount` | Customer loyalty points balance | id, userId, points, lifetimePoints, tier, enrolledAt |
| `LoyaltyTransaction` | Points earned or redeemed | id, accountId, type (earn/redeem/expire), points, referenceType, referenceId, createdAt |
| `Tier` | Loyalty tier definition | id, name, minPoints, benefits[], discountRate |

**Invariants**:
- One review per user per variant per order
- Review only allowed for delivered orders
- Loyalty points earned: 1 point per 100 YER spent
- Points expire after 12 months of inactivity
- Tier reviewed monthly; downgrade only at billing cycle
- Tier thresholds: Bronze (0), Silver (1000), Gold (5000), Platinum (15000)

**Domain Events Published**:
- `review.created`
- `review.flagged`
- `review.helpful`
- `loyalty.points_earned`
- `loyalty.points_redeemed`
- `loyalty.tier_upgraded`
- `loyalty.tier_downgraded`

**Reads From**: B04 (Order for verification), B03 (Variant)

---

### B11: Content

**Purpose**: Manages static pages, banners, push notifications, and system communications.

| Entity | Description | Key Fields |
|--------|-------------|------------|
| `Page` | CMS-managed static page (About, FAQ, T&C) | id, title, slug, body (HTML), isPublished, publishedAt |
| `Banner` | Promotional banner for storefront | id, title, imageUrl, linkUrl, position (home/category), startsAt, endsAt, isActive |
| `Notification` | System notification to a user | id, userId, type (order/promo/system), title, body, isRead, createdAt |
| `PushSubscription` | Device registration for push notifications | id, userId, endpoint, p256dh, auth, platform |
| `EmailTemplate` | Reusable email template | id, name, subject, body, variables[] |

**Invariants**:
- Banner position must be one of: home_hero, home_mid, category_top, checkout
- Notification max body: 255 characters
- Pages support Markdown or HTML (admin choice)
- Banners auto-deactivate after endsAt
- Push subscription must be refreshed every 30 days

**Domain Events Published**:
- `notification.sent`
- `notification.read`
- `banner.impression`
- `banner.clicked`

**Reads From**: B04 (Order status for notifications), B02 (Store for store-specific content)

---

### B12: Support

**Purpose**: Manages customer support tickets, service categories, and vendor service bookings.

| Entity | Description | Key Fields |
|--------|-------------|------------|
| `Ticket` | Support request from customer or vendor | id, userId, orderId?, subject, status (open/in_progress/resolved/closed), priority, assignedTo |
| `TicketMessage` | Message thread within a ticket | id, ticketId, senderId, body, attachments[], createdAt |
| `ServiceCategory` | Classification for support topics | id, name, parentId, sla (hours), isActive |
| `Booking` | Scheduled service appointment (e.g., pickup, repair) | id, userId, serviceCategoryId, scheduledAt, status (pending/confirmed/completed/cancelled) |
| `TicketEscalation` | Escalation from agent to admin | id, ticketId, fromAgentId, toAdminId, reason, createdAt |

**Invariants**:
- Ticket priority determines SLA: Critical (4h), High (8h), Medium (24h), Low (72h)
- Auto-close tickets after 7 days of inactivity
- Ticket messages capped at 50 per ticket
- Booking cannot be in the past
- Escalation required if SLA breach imminent

**Domain Events Published**:
- `ticket.created`
- `ticket.assigned`
- `ticket.escalated`
- `ticket.resolved`
- `ticket.closed`
- `ticket.sla_breach`
- `booking.created`
- `booking.confirmed`
- `booking.completed`

**Reads From**: B04 (Order details for context), B01 (User info)

---

### B13: Promotion

**Purpose**: Manages coupons, discount rules, and promotion usage tracking.

| Entity | Description | Key Fields |
|--------|-------------|------------|
| `Coupon` | Voucher code for discounts | id, code, type (fixed/percent/free_shipping), value, minOrderAmount, maxUses, usedCount, startsAt, endsAt, isActive |
| `DiscountRule` | Automated discount logic | id, name, type (bundle/buy_x_get_y/tiered), conditions{}, actions{}, priority, isActive |
| `Usage` | Record of coupon/rule application | id, couponId?, ruleId?, orderId, userId, discountAmount, appliedAt |
| `Promotion` | Marketing campaign wrapper | id, name, description, couponIds[], ruleIds[], startsAt, endsAt, isActive |

**Invariants**:
- Coupon code must be unique (case-insensitive)
- Coupon cannot be stacked unless `allowStacking` flag set
- Fixed discount cannot exceed order total
- Percentage discount capped at 50%
- Free shipping coupon max discount: 5000 YER
- Usage per user limited by `maxUsesPerUser`
- Coupon/Rule cannot be used after endsAt

**Domain Events Published**:
- `coupon.created`
- `coupon.expired`
- `coupon.usage_limit_reached`
- `discount.applied`
- `discount.rejected`
- `promotion.activated`
- `promotion.deactivated`

**Reads From**: B04 (Order total for validation), B03 (Products for category-based rules)

---

## 4. Entity Relationship Summary

```
User ──────┬── Session
           ├── Wallet ── Transaction
           ├── LoyaltyAccount ── LoyaltyTransaction
           ├── Cart ── CartItem
           ├── Wishlist ── WishlistItem
           ├── Review ── ReviewVote
           ├── Ticket ── TicketMessage
           └── Notification

Vendor ────┬── Store ── Badge
           ├── VendorAnalytics
           ├── Commission ── PayoutItem ── Payout ── Invoice
           └── KYC

Product ───┬── Variant ── ProductImage
           ├── Offer
           └── Stock ── Reservation ── OrderItem

MasterOrder ── SubOrder ── OrderItem
              └── OrderTimeline

SubOrder ── Assignment ── ProofOfDelivery
           └── DeliveryTracking

Category ── (self-referencing parent)

Coupon ── Usage
DiscountRule ── Usage
Promotion ── Coupon, DiscountRule
```

## 5. Value Objects

| Context | Value Object | Description |
|---------|-------------|-------------|
| B01 | `PhoneNumber` | E.164 format (e.g., +967XXXXXXXXX) |
| B01 | `OTPCode` | 6-digit numeric string |
| B03 | `Money` | { amount: Decimal, currency: "YER" } |
| B03 | `Dimensions` | { length, width, height, unit: "cm" } |
| B03 | `Slug` | URL-safe kebab-case identifier |
| B05 | `PaymentReference` | Unique payment gateway reference |
| B07 | `GeoPoint` | { lat: float, lng: float } |
| B07 | `GeoPolygon` | Array of GeoPoint for zone boundaries |
| B10 | `Rating` | Integer 1–5 |
| B13 | `CouponCode` | Uppercase alphanumeric, 6–20 chars |
