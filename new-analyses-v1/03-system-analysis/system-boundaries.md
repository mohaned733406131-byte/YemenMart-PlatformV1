# System Boundaries — YemenMart Platform

## 1. Overview

Each bounded context owns its data, exposes query/command interfaces, and communicates through defined integration patterns. This document defines ownership, reads, publishes, and anti-corruption layers.

## 2. Boundary Matrix

| Block | Owns | Reads From | Publishes Events To | Anti-Corruption Layer |
|-------|------|-----------|---------------------|----------------------|
| B01 Identity | Users, Sessions, OTP, AuditLogs | — | B02, B04, B09, B12 | None (root context) |
| B02 Marketplace | Vendors, Stores, KYC, Badges | B01 (User) | B03, B04, B06, B07, B09 | ACL-B02 |
| B03 Catalog | Products, Variants, Categories, Offers | B02 (Store), B08 (Stock) | B04, B09, B10, B13 | ACL-B03 |
| B04 Commerce | MasterOrders, SubOrders, OrderItems, Returns | B02, B03, B08, B13 | B05, B06, B07, B08, B10, B11 | ACL-B04 |
| B05 Payment | Wallets, Escrow, Transactions, Refunds | B04 (Order), B06 (Rules) | B04, B06, B11 | ACL-B05 |
| B06 Finance | Commissions, Payouts, Invoices, Ledger | B02 (CommissionRate), B04 (SubOrder), B05 (Wallet) | B11 | ACL-B06 |
| B07 Logistics | Providers, Zones, Assignments, POD | B04 (SubOrder), B02 (Store) | B04, B08, B11 | ACL-B07 |
| B08 Inventory | Stock, Reservations, Warehouses, Movements | B03 (Variant) | B03, B04 | ACL-B08 |
| B09 Storefront | Cart, Wishlist, StoreFollow, RecentlyViewed | B03 (Product/Variant/Offer), B08 (Stock) | B01, B04, B10 | None (UI layer) |
| B10 Engagement | Reviews, Loyalty, Tiers | B04 (Order), B03 (Variant) | B11 | None (read-heavy) |
| B11 Content | Pages, Banners, Notifications, Push | B04 (Order), B02 (Store) | — | None (publish-only) |
| B12 Support | Tickets, Messages, Bookings | B04 (Order), B01 (User) | B11 | ACL-B12 |
| B13 Promotion | Coupons, DiscountRules, Usage, Promotions | B04 (Order), B03 (Product) | B04 | None (read by B04) |

## 3. Detailed Block Boundaries

---

### B01: Identity & Access

**Owns**:
- `User` entity and all profile data
- `Session` management and token lifecycle
- `OTP` generation, validation, and rate limiting
- `AuditLog` (append-only)
- `RefreshToken` lifecycle
- Password hashing and verification
- Role-based access control (RBAC) definitions

**Does NOT Own**:
- Vendor-specific profile (→ B02)
- Wallet balance (→ B05)
- Loyalty points (→ B10)

**Reads From Other Blocks**:
- None (root context; other blocks depend on B01)

**Publishes To**:
- `user.registered` → B02 (create vendor profile), B05 (create wallet), B10 (create loyalty account)
- `user.phone_verified` → B02 (allow KYC submission)
- `user.login` → AuditLog
- `user.role_changed` → B02 (update vendor permissions)

**Anti-Corruption Layer**: None — B01 is the foundational context. All other blocks reference `UserId` as a value object.

---

### B02: Marketplace

**Owns**:
- `Vendor` entity and lifecycle
- `Store` entity, slug uniqueness, commission rate
- `KYC` submission, review, approval/rejection
- `Badge` assignment and expiration
- `VendorAnalytics` aggregation
- Vendor status transitions (pending → approved → suspended)

**Does NOT Own**:
- User credentials (→ B01)
- Products (→ B03)
- Orders (→ B04)
- Payouts (→ B06)

**Reads From Other Blocks**:
- B01 `User` → userId, phone, email for vendor account creation
- B01 `AuditLog` → track who approved/rejected KYC

**Publishes To**:
- `vendor.kyc_approved` → B03 (allow product publishing), B06 (enable payout)
- `vendor.suspended` → B03 (archive products), B04 (flag active orders)
- `store.created` → B03 (enable catalog setup), B11 (store landing page)
- `store.commission_changed` → B06 (update commission rules)

**Anti-Corruption Layer: ACL-B02**
```
┌─────────────────────────────────────────────────────┐
│ ACL-B02: Marketplace Anti-Corruption Layer          │
│                                                     │
│ Exposes:                                            │
│   VendorQueryService.getVendorById(id)              │
│   VendorQueryService.getVendorByUserId(userId)      │
│   StoreQueryService.getStoreById(id)                │
│   StoreQueryService.getCommissionRate(storeId)      │
│   StoreQueryService.isStoreActive(storeId)          │
│   KYCQueryService.getKycStatus(vendorId)            │
│                                                     │
│ Accepts:                                            │
│   UserRegisteredEvent → create vendor stub          │
│   PhoneVerifiedEvent → unlock KYC submission        │
│                                                     │
│ Rejects:                                            │
│   Direct DB access from other blocks                │
│   User entity modification                          │
└─────────────────────────────────────────────────────┘
```

---

### B03: Catalog

**Owns**:
- `Product` entity and lifecycle (draft → active → archived)
- `Variant` entity, SKU management, pricing
- `Category` hierarchy and tree operations
- `Offer` time-bound pricing
- `ProductImage` management
- `Brand` entity
- Product search indexing (Elasticsearch sync)

**Does NOT Own**:
- Store ownership (→ B02)
- Stock quantities (→ B08)
- Cart/order context (→ B04, B09)

**Reads From Other Blocks**:
- B02 `Store` → storeId, isActive, for product ownership validation
- B08 `Stock` → availableQty, for "in stock" badge on product pages

**Publishes To**:
- `product.published` → B09 (available in storefront), B13 (eligible for promotions)
- `product.archived` → B09 (remove from storefront), B04 (flag in active orders)
- `variant.stock_changed` → B09 (update availability badge)

**Anti-Corruption Layer: ACL-B03**
```
┌─────────────────────────────────────────────────────┐
│ ACL-B03: Catalog Anti-Corruption Layer              │
│                                                     │
│ Exposes:                                            │
│   ProductQueryService.getProductById(id)             │
│   ProductQueryService.getVariantById(id)             │
│   ProductQueryService.getPrice(variantId)            │
│   ProductQueryService.getProductsByStore(storeId)    │
│   CategoryQueryService.getCategoryTree()             │
│   CategoryQueryService.getCategoryById(id)           │
│   SearchService.searchProducts(query, filters)       │
│   SearchService.autocomplete(prefix)                 │
│                                                     │
│ Accepts:                                            │
│   StoreActivatedEvent → enable product publishing    │
│   StockChangedEvent → update availability badge      │
│                                                     │
│ Rejects:                                            │
│   Direct variant price modification from B04         │
│   Category hierarchy changes from non-admin          │
└─────────────────────────────────────────────────────┘
```

---

### B04: Commerce

**Owns**:
- `MasterOrder` lifecycle and state machine
- `SubOrder` per-vendor splitting
- `OrderItem` line items
- `ReturnRequest` workflow
- `OrderTimeline` audit trail
- Order status transitions
- Total calculation logic

**Does NOT Own**:
- Payment processing (→ B05)
- Commission calculation (→ B06)
- Delivery assignment (→ B07)
- Stock reservation (→ B08)
- Coupon validation (→ B13)

**Reads From Other Blocks**:
- B02 `Store` → vendorId, storeId for sub-order splitting
- B03 `Product/Variant` → productName, price, weight for order items
- B08 `Stock` → availableQty for order validation
- B13 `Coupon` → discount calculation for order total

**Publishes To**:
- `order.placed` → B05 (hold escrow), B08 (commit reservation), B07 (assign delivery), B11 (notification)
- `order.paid` → B06 (calculate commission), B11 (confirmation)
- `order.delivered` → B05 (release escrow), B10 (enable review), B06 (settle commission)
- `return.requested` → B05 (initiate refund), B11 (notification)

**Anti-Corruption Layer: ACL-B04**
```
┌─────────────────────────────────────────────────────┐
│ ACL-B04: Commerce Anti-Corruption Layer             │
│                                                     │
│ Exposes:                                            │
│   OrderQueryService.getMasterOrderById(id)           │
│   OrderQueryService.getSubOrderById(id)              │
│   OrderQueryService.getOrdersByUser(userId)          │
│   OrderQueryService.getOrdersByVendor(vendorId)      │
│   OrderQueryService.getOrderTotal(orderId)           │
│   ReturnQueryService.getReturnRequestById(id)        │
│                                                     │
│ Accepts:                                            │
│   StockCommittedEvent → confirm order items          │
│   PaymentReleasedEvent → mark order paid             │
│   DeliveryDeliveredEvent → update order status       │
│                                                     │
│ Rejects:                                            │
│   Direct order status manipulation from B07          │
│   Price modification after payment                   │
└─────────────────────────────────────────────────────┘
```

---

### B05: Payment

**Owns**:
- `Wallet` balance and lifecycle
- `Escrow` hold/release/refund
- `Transaction` ledger (append-only)
- `PaymentMethod` management
- `Refund` processing
- External gateway integration (m-Floos, OneCash)

**Does NOT Own**:
- Commission calculation (→ B06)
- Order status (→ B04)
- Loyalty points deduction (→ B10)

**Reads From Other Blocks**:
- B04 `MasterOrder` → grandTotal for escrow amount
- B06 `Finance Rules` → minimum payout threshold

**Publishes To**:
- `escrow.held` → B04 (confirm payment)
- `escrow.released` → B06 (trigger settlement)
- `wallet.topup_completed` → B11 (confirmation notification)
- `refund.completed` → B04 (close return request), B11 (refund notification)

**Anti-Corruption Layer: ACL-B05**
```
┌─────────────────────────────────────────────────────┐
│ ACL-B05: Payment Anti-Corruption Layer              │
│                                                     │
│ Exposes:                                            │
│   WalletQueryService.getBalance(walletId)            │
│   WalletQueryService.getWalletByUserId(userId)       │
│   EscrowQueryService.getEscrowByOrderId(orderId)     │
│   TransactionQueryService.getTransactions(walletId)  │
│   RefundQueryService.getRefundById(id)               │
│                                                     │
│ Accepts:                                            │
│   OrderPlacedEvent → create escrow                   │
│   OrderDeliveredEvent → release escrow               │
│   ReturnApprovedEvent → initiate refund              │
│                                                     │
│ Rejects:                                            │
│   Direct balance manipulation                        │
│   Bypass of escrow lifecycle                         │
│   Transaction deletion or modification               │
└─────────────────────────────────────────────────────┘
```

---

### B06: Finance

**Owns**:
- `Commission` calculation and settlement
- `Payout` batch generation and processing
- `Invoice` generation
- `LedgerEntry` double-entry bookkeeping
- Payout scheduling (weekly/biweekly)
- Minimum payout threshold enforcement

**Does NOT Own**:
- Wallet balance (→ B05)
- Vendor profile (→ B02)
- Order details (→ B04)

**Reads From Other Blocks**:
- B02 `Store` → commissionRate per store
- B04 `SubOrder` → subtotal per vendor for commission calculation
- B05 `Wallet` → vendor wallet ID for payout target

**Publishes To**:
- `payout.completed` → B11 (notification to vendor)
- `invoice.generated` → B11 (email attachment)
- `commission.settled` → B10 (loyalty points calculation)

**Anti-Corruption Layer: ACL-B06**
```
┌─────────────────────────────────────────────────────┐
│ ACL-B06: Finance Anti-Corruption Layer              │
│                                                     │
│ Exposes:                                            │
│   CommissionQueryService.getCommissions(vendorId)    │
│   CommissionQueryService.getPendingAmount(vendorId)  │
│   PayoutQueryService.getPayoutById(id)               │
│   PayoutQueryService.getNextPayoutDate(vendorId)     │
│   InvoiceQueryService.getInvoiceById(id)             │
│   LedgerQueryService.getBalance(account)             │
│                                                     │
│ Accepts:                                            │
│   EscrowReleasedEvent → calculate commission         │
│   CommissionSettledEvent → update ledger             │
│                                                     │
│ Rejects:                                            │
│   Direct commission rate modification                │
│   Ledger entry deletion or modification              │
│   Payout amount override                             │
└─────────────────────────────────────────────────────┘
```

---

### B07: Logistics

**Owns**:
- `DeliveryProvider` registration and status
- `Zone` geographic boundaries and pricing
- `Assignment` algorithm (nearest provider, zone matching)
- `ProofOfDelivery` capture and storage
- `DeliveryTracking` real-time updates
- Delivery status transitions

**Does NOT Own**:
- Order status (→ B04)
- Vendor address (→ B02)
- Warehouse locations (→ B08)

**Reads From Other Blocks**:
- B04 `SubOrder` → delivery address, items for assignment
- B02 `Store` → pickup address for delivery routing

**Publishes To**:
- `delivery.assigned` → B04 (update order status)
- `delivery.delivered` → B04 (mark delivered), B05 (release escrow), B08 (update stock)
- `delivery.failed` → B04 (flag for re-attempt), B11 (notify customer)

**Anti-Corruption Layer: ACL-B07**
```
┌─────────────────────────────────────────────────────┐
│ ACL-B07: Logistics Anti-Corruption Layer            │
│                                                     │
│ Exposes:                                            │
│   AssignmentQueryService.getAssignment(id)           │
│   AssignmentQueryService.getAssignments(orderId)     │
│   ZoneQueryService.getZoneForAddress(address)        │
│   ZoneQueryService.calculateFee(zoneId, distance)    │
│   DeliveryQueryService.getTracking(assignmentId)     │
│   PODQueryService.getPOD(assignmentId)               │
│                                                     │
│ Accepts:                                            │
│   OrderPaidEvent → trigger delivery assignment       │
│   SubOrderConfirmedEvent → queue for pickup          │
│                                                     │
│ Rejects:                                            │
│   Direct order status changes from delivery providers│
│   Zone fee modification without admin approval       │
└─────────────────────────────────────────────────────┘
```

---

### B08: Inventory

**Owns**:
- `Stock` levels per variant per warehouse
- `Reservation` lifecycle (hold → commit/release)
- `Warehouse` entity
- `StockAdjustment` with authorization
- `StockMovement` immutable log
- Low-stock threshold monitoring

**Does NOT Own**:
- Product details (→ B03)
- Order context (→ B04)
- Warehouse physical operations (→ B07)

**Reads From Other Blocks**:
- B03 `Variant` → variantId for stock association

**Publishes To**:
- `stock.committed` → B04 (confirm order)
- `stock.released` → B03 (update availability)
- `stock.low` → B11 (admin alert), B02 (vendor alert)
- `stock.out_of_stock` → B03 (mark unavailable), B09 (remove from cart)

**Anti-Corruption Layer: ACL-B08**
```
┌─────────────────────────────────────────────────────┐
│ ACL-B08: Inventory Anti-Corruption Layer            │
│                                                     │
│ Exposes:                                            │
│   StockQueryService.getAvailable(variantId)           │
│   StockQueryService.getTotalStock(variantId)          │
│   StockQueryService.isInStock(variantId)              │
│   StockQueryService.getStockByWarehouse(variantId)    │
│   ReservationQueryService.getActive(orderId)          │
│                                                     │
│ Accepts:                                            │
│   OrderPlacedEvent → create reservation              │
│   PaymentConfirmedEvent → commit reservation         │
│   PaymentFailedEvent → release reservation           │
│   DeliveryDeliveredEvent → decrement stock           │
│                                                     │
│ Rejects:                                            │
│   Direct stock quantity modification from B04        │
│   Reservation extension beyond 15 minutes             │
│   Stock adjustment without admin role                 │
└─────────────────────────────────────────────────────┘
```

---

### B09: Storefront

**Owns**:
- `Cart` and `CartItem` per user
- `Wishlist` and `WishlistItem`
- `StoreFollow` relationships
- `RecentlyViewed` browsing history
- Cart total calculation
- Checkout orchestration (assembles data from B03, B08, B13)

**Does NOT Own**:
- Order creation (→ B04)
- Product data (→ B03)
- Stock data (→ B08)
- Payment processing (→ B05)

**Reads From Other Blocks**:
- B03 `Product/Variant/Offer` → product details, pricing, offers for display
- B08 `Stock` → availableQty for cart validation

**Publishes To**:
- `cart.checkout_initiated` → B04 (create order), B08 (reserve stock), B05 (initiate payment)
- `store.followed` → B10 (loyalty points), B11 (follow notification)

**Anti-Corruption Layer**: None — B09 is the UI/orchestration layer that reads from other blocks. It does not expose query interfaces consumed by other blocks.

---

### B10: Engagement

**Owns**:
- `Review` and `ReviewVote` for product feedback
- `LoyaltyAccount` and `LoyaltyTransaction` for points
- `Tier` definitions and progression rules
- Review moderation rules
- Loyalty point accrual rules

**Does NOT Own**:
- Order verification (→ B04)
- Product data (→ B03)
- Notification delivery (→ B11)

**Reads From Other Blocks**:
- B04 `Order` → verify purchase eligibility for reviews
- B03 `Variant` → product association for reviews

**Publishes To**:
- `review.created` → B11 (notify vendor)
- `loyalty.tier_upgraded` → B11 (celebration notification)

**Anti-Corruption Layer**: None — B10 is primarily a read-heavy, analytics context. It does not expose query interfaces consumed by other blocks.

---

### B11: Content

**Owns**:
- `Page` CMS content
- `Banner` display rules and scheduling
- `Notification` delivery
- `PushSubscription` device registration
- `EmailTemplate` management
- Delivery channel selection (push, email, WhatsApp)

**Does NOT Own**:
- Order data (→ B04)
- User preferences (→ B01)
- Vendor content (→ B02)

**Reads From Other Blocks**:
- B04 `Order` → order details for notification templates
- B02 `Store` → store-specific banners and content

**Publishes To**:
- None — B11 is a terminal context (notifications delivered externally)

**Anti-Corruption Layer**: None — B11 consumes events from other blocks and delivers notifications. It does not expose query interfaces consumed by other blocks.

---

### B12: Support

**Owns**:
- `Ticket` lifecycle and SLA tracking
- `TicketMessage` thread
- `ServiceCategory` classification
- `Booking` scheduling
- `TicketEscalation` workflow
- SLA breach detection

**Does NOT Own**:
- Order details (→ B04)
- User profile (→ B01)
- Refund processing (→ B05)

**Reads From Other Blocks**:
- B04 `Order` → order context for ticket creation
- B01 `User` → user info for ticket assignment

**Publishes To**:
- `ticket.resolved` → B11 (satisfaction survey)
- `ticket.sla_breach` → B11 (admin alert)

**Anti-Corruption Layer: ACL-B12**
```
┌─────────────────────────────────────────────────────┐
│ ACL-B12: Support Anti-Corruption Layer              │
│                                                     │
│ Exposes:                                            │
│   TicketQueryService.getTicketById(id)               │
│   TicketQueryService.getTicketsByUser(userId)        │
│   TicketQueryService.getOpenTicketsCount()           │
│   BookingQueryService.getBookingById(id)             │
│                                                     │
│ Accepts:                                            │
│   OrderDeliveredEvent → auto-create return ticket    │
│   UserRegisteredEvent → create welcome ticket        │
│                                                     │
│ Rejects:                                            │
│   Direct order modification from support agents      │
│   Refund execution (only B05 can execute)            │
└─────────────────────────────────────────────────────┘
```

---

### B13: Promotion

**Owns**:
- `Coupon` creation, validation, and lifecycle
- `DiscountRule` conditions and actions
- `Usage` tracking per coupon/rule
- `Promotion` campaign management
- Coupon stacking rules
- Usage limit enforcement

**Does NOT Own**:
- Order total calculation (→ B04)
- Payment processing (→ B05)
- Product pricing (→ B03)

**Reads From Other Blocks**:
- B04 `Order` → order total, items for discount eligibility
- B03 `Product/Variant` → category-based discount rules

**Publishes To**:
- `discount.applied` → B04 (update order total), B11 (promotion notification)

**Anti-Corruption Layer**: None — B13 is read by B04 at checkout time. It does not expose query interfaces consumed by other blocks for writes.

---

## 4. Anti-Corruption Layer Summary

| ACL | Consumer Block | Provider Block | Pattern | Purpose |
|-----|---------------|----------------|---------|---------|
| ACL-B02 | B03, B04, B06 | B02 | Shared Kernel | Vendor/Store query interface |
| ACL-B03 | B04, B09, B13 | B03 | Shared Kernel | Product/Variant query interface |
| ACL-B04 | B05, B07, B08 | B04 | Shared Kernel | Order query interface |
| ACL-B05 | B04, B06 | B05 | Shared Kernel | Wallet/Escrow query interface |
| ACL-B06 | B05 | B06 | Shared Kernel | Finance rules query interface |
| ACL-B07 | B04 | B07 | Shared Kernel | Delivery assignment query interface |
| ACL-B08 | B04, B09 | B08 | Shared Kernel | Stock query interface |
| ACL-B12 | B04, B01 | B12 | Shared Kernel | Ticket query interface |

## 5. Integration Rules

1. **No direct DB access** between blocks — all reads through query services
2. **No shared entities** — each block defines its own DTOs/projections
3. **Event-driven writes** — blocks publish events, consumers update their own data
4. **ACL validation** — all cross-block calls validated at the ACL boundary
5. **Idempotency** — all event handlers must be idempotent (dedup by eventId)
6. **Circuit breaker** — external integrations (m-Floos, OneCash, SMS) wrapped in circuit breakers
