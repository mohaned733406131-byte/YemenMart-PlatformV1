# Component Diagram — YemenMart Modules

## 1. Module Dependency Map

```
┌──────────────────────────────────────────────────────────────────────┐
│                     YEMENMART MODULE ECOSYSTEM                        │
│                                                                      │
│  ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐          │
│  │  B01    │───▶│  B02    │───▶│  B03    │───▶│  B04    │          │
│  │  Auth   │    │ Vendor  │    │ Catalog │    │  Order  │          │
│  └─────────┘    └────┬────┘    └─────────┘    └────┬────┘          │
│                      │                              │                │
│                      ▼                              ▼                │
│  ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐          │
│  │  B10    │◀───│  B09    │    │  B08    │◀───│  B05    │          │
│  │  Trust  │    │ Storefr │    │ Inven.  │    │ Payment │          │
│  └─────────┘    └─────────┘    └────┬────┘    └────┬────┘          │
│                                      │              │                │
│                                      ▼              ▼                │
│  ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐          │
│  │  B12    │    │  B11    │    │  B07    │    │  B06    │          │
│  │ Support │    │ Content │    │Shipping │    │ Finance │          │
│  └─────────┘    └─────────┘    └─────────┘    └─────────┘          │
│                                                                      │
│  ┌─────────┐                                                       │
│  │  B13    │◀── Used by B03, B04, B09                               │
│  │ Pricing │                                                       │
│  └─────────┘                                                       │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 2. B01 — Identity & Access

```
┌─────────────────────────────────────────────────────┐
│                    B01 AUTH                          │
│                                                      │
│  ┌──────────────────┐  ┌──────────────────┐         │
│  │   Auth Service   │  │ Session Service  │         │
│  │                  │  │                  │         │
│  │ • register()     │  │ • create()       │         │
│  │ • login()        │  │ • validate()     │         │
│  │ • logout()       │  │ • refresh()      │         │
│  │ • resetPassword()│  │ • revoke()       │         │
│  │ • verifyOTP()    │  │ • revokeAll()    │         │
│  └────────┬─────────┘  └────────┬─────────┘         │
│           │                     │                    │
│           ▼                     ▼                    │
│  ┌──────────────────────────────────────────┐       │
│  │            Audit Service                  │       │
│  │                                           │       │
│  │ • logAction()                             │       │
│  │ • getAuditTrail()                         │       │
│  │ • exportAuditLog()                        │       │
│  └──────────────────────────────────────────┘       │
│                                                      │
│  Domain Events Emitted:                              │
│  • user.registered                                   │
│  • user.loggedIn                                     │
│  • user.passwordReset                                │
│  • session.created / session.revoked                 │
│                                                      │
│  Dependencies: None (root module)                    │
└─────────────────────────────────────────────────────┘
```

**Entities:** `User`, `Session`, `AuditLog`, `Role`, `Permission`
**Value Objects:** `Email`, `PhoneNumber`, `Password`, `OTP`

---

## 3. B02 — Vendor Management

```
┌─────────────────────────────────────────────────────┐
│                  B02 VENDOR                          │
│                                                      │
│  ┌──────────────────┐  ┌──────────────────┐         │
│  │  Vendor Service  │  │  Store Service   │         │
│  │                  │  │                  │         │
│  │ • createVendor() │  │ • createStore()  │         │
│  │ • updateProfile()│  │ • updateStore()  │         │
│  │ • verifyVendor() │  │ • suspendStore() │         │
│  │ • getVendor()    │  │ • getStore()     │         │
│  │ • listVendors()  │  │ • listStores()   │         │
│  └────────┬─────────┘  └────────┬─────────┘         │
│           │                     │                    │
│           ▼                     ▼                    │
│  ┌──────────────────────────────────────────┐       │
│  │            KYC Service                    │       │
│  │                                           │       │
│  │ • submitDocuments()                       │       │
│  │ • reviewKYC()                             │       │
│  │ • getStatus()                             │       │
│  │ • requestResubmission()                   │       │
│  └──────────────────────────────────────────┘       │
│                                                      │
│  Domain Events Emitted:                              │
│  • vendor.created                                    │
│  • vendor.verified / vendor.rejected                 │
│  • store.created / store.suspended                   │
│  • kyc.submitted / kyc.approved / kyc.rejected       │
│                                                      │
│  Dependencies: B01 (Auth for user linking)           │
└─────────────────────────────────────────────────────┘
```

**Entities:** `Vendor`, `Store`, `KYCSubmission`
**Value Objects:** `StoreName`, `BusinessLicense`, `TaxID`

---

## 4. B03 — Product Catalog

```
┌─────────────────────────────────────────────────────┐
│                  B03 CATALOG                         │
│                                                      │
│  ┌──────────────────┐  ┌──────────────────┐         │
│  │ Product Service  │  │Category Service  │         │
│  │                  │  │                  │         │
│  │ • createProduct()│  │ • createCategory()│        │
│  │ • updateProduct()│  │ • updateCategory()│        │
│  │ • archiveProduct │  │ • getCategoryTree()│       │
│  │ • bulkUpdate()   │  │ • moveCategory()  │        │
│  │ • search()       │  │ • listCategories()│        │
│  └────────┬─────────┘  └──────────────────┘         │
│           │                                          │
│           ▼                                          │
│  ┌──────────────────────────────────────────┐       │
│  │           Offer Service                   │       │
│  │                                           │       │
│  │ • createOffer()                           │       │
│  │ • activateOffer()                         │       │
│  │ • deactivateOffer()                       │       │
│  │ • getActiveOffers()                       │       │
│  └──────────────────────────────────────────┘       │
│                                                      │
│  Domain Events Emitted:                              │
│  • product.created / product.updated                 │
│  • product.archived / product.published              │
│  • category.created / category.restructured          │
│  • offer.activated / offer.deactivated               │
│                                                      │
│  Dependencies: B02 (Vendor/Store), B13 (Pricing)     │
└─────────────────────────────────────────────────────┘
```

**Entities:** `Product`, `ProductVariant`, `Category`, `Offer`, `ProductImage`
**Value Objects:** `SKU`, `Money`, `Dimensions`, `Weight`

---

## 5. B04 — Order Management

```
┌─────────────────────────────────────────────────────┐
│                   B04 ORDER                          │
│                                                      │
│  ┌──────────────────┐  ┌──────────────────┐         │
│  │  Order Service   │  │Checkout Service  │         │
│  │                  │  │                  │         │
│  │ • createOrder()  │  │ • initiate()     │         │
│  │ • confirmOrder() │  │ • validate()     │         │
│  │ • cancelOrder()  │  │ • calculate()    │         │
│  │ • getOrder()     │  │ • complete()     │         │
│  │ • listOrders()   │  │                  │         │
│  └────────┬─────────┘  └──────────────────┘         │
│           │                                          │
│           ▼                                          │
│  ┌──────────────────────────────────────────┐       │
│  │          Return Service                   │       │
│  │                                           │       │
│  │ • requestReturn()                         │       │
│  │ • approveReturn()                         │       │
│  │ • processRefund()                         │       │
│  │ • trackReturn()                           │       │
│  └──────────────────────────────────────────┘       │
│                                                      │
│  Domain Events Emitted:                              │
│  • order.placed / order.confirmed                    │
│  • order.cancelled / order.completed                 │
│  • return.requested / return.approved                │
│  • return.completed                                  │
│                                                      │
│  Dependencies: B03 (Catalog), B05 (Payment),         │
│                B08 (Inventory), B07 (Shipping),       │
│                B13 (Pricing)                          │
└─────────────────────────────────────────────────────┘
```

**Entities:** `Order`, `OrderItem`, `Return`, `ReturnItem`
**Value Objects:** `OrderStatus`, `Money`, `Address`

---

## 6. B05 — Payment Processing

```
┌─────────────────────────────────────────────────────┐
│                  B05 PAYMENT                         │
│                                                      │
│  ┌──────────────────┐  ┌──────────────────┐         │
│  │  Wallet Service  │  │ Escrow Service   │         │
│  │                  │  │                  │         │
│  │ • createWallet() │  │ • createEscrow() │         │
│  │ • getBalance()   │  │ • hold()         │         │
│  │ • debit()        │  │ • release()      │         │
│  │ • credit()       │  │ • refund()       │         │
│  │ • getHistory()   │  │ • getStatus()    │         │
│  └──────────────────┘  └────────┬─────────┘         │
│                                  │                    │
│                                  ▼                    │
│  ┌──────────────────────────────────────────┐       │
│  │         Payment Service                   │       │
│  │                                           │       │
│  │ • processPayment()                        │       │
│  │ • verifyPayment()                         │       │
│  │ • handleWebhook()                         │       │
│  │ • refundPayment()                         │       │
│  │ • getPaymentStatus()                      │       │
│  └──────────────────────────────────────────┘       │
│                                                      │
│  Domain Events Emitted:                              │
│  • payment.processed / payment.failed                │
│  • payment.refunded                                  │
│  • escrow.created / escrow.released                  │
│  • wallet.debited / wallet.credited                  │
│                                                      │
│  Dependencies: B01 (Auth), B06 (Finance)             │
└─────────────────────────────────────────────────────┘
```

**Entities:** `Payment`, `Wallet`, `Escrow`, `Transaction`
**Value Objects:** `Money`, `PaymentMethod`, `TransactionRef`

---

## 7. B06 — Financial Operations

```
┌─────────────────────────────────────────────────────┐
│                  B06 FINANCE                         │
│                                                      │
│  ┌──────────────────┐  ┌──────────────────┐         │
│  │Commission Service│  │  Payout Service  │         │
│  │                  │  │                  │         │
│  │ • calculate()    │  │ • requestPayout()│         │
│  │ • getRate()      │  │ • processPayout()│         │
│  │ • updateRate()   │  │ • getPayouts()   │         │
│  │ • getReport()    │  │ • cancelPayout() │         │
│  └──────────────────┘  └──────────────────┘         │
│                                                      │
│  ┌──────────────────────────────────────────┐       │
│  │         Invoice Service                   │       │
│  │                                           │       │
│  │ • generateInvoice()                       │       │
│  │ • sendInvoice()                           │       │
│  │ • getInvoice()                            │       │
│  │ • listInvoices()                          │       │
│  └──────────────────────────────────────────┘       │
│                                                      │
│  Domain Events Emitted:                              │
│  • commission.calculated                             │
│  • payout.requested / payout.processed               │
│  • payout.failed                                     │
│  • invoice.generated / invoice.sent                  │
│                                                      │
│  Dependencies: B05 (Payment), B04 (Order)            │
└─────────────────────────────────────────────────────┘
```

**Entities:** `Commission`, `Payout`, `Invoice`, `CommissionRate`
**Value Objects:** `Money`, `PayoutMethod`, `InvoiceNumber`

---

## 8. B07 — Shipping & Delivery

```
┌─────────────────────────────────────────────────────┐
│                 B07 SHIPPING                         │
│                                                      │
│  ┌──────────────────┐  ┌──────────────────┐         │
│  │Delivery Service  │  │  Zone Service    │         │
│  │                  │  │                  │         │
│  │ • createShipment │  │ • createZone()   │         │
│  │ • trackDelivery()│  │ • updateRates()  │         │
│  │ • updateStatus() │  │ • getZones()     │         │
│  │ • confirmDeliv() │  │ • calculateRate()│         │
│  └────────┬─────────┘  └──────────────────┘         │
│           │                                          │
│           ▼                                          │
│  ┌──────────────────────────────────────────┐       │
│  │       Assignment Service                  │       │
│  │                                           │       │
│  │ • assignRider()                           │       │
│  │ • reassignRider()                         │       │
│  │ • getAvailableRiders()                    │       │
│  │ • optimizeRoute()                         │       │
│  └──────────────────────────────────────────┘       │
│                                                      │
│  Domain Events Emitted:                              │
│  • shipment.created / shipment.dispatched            │
│  • shipment.delivered / shipment.failed              │
│  • rider.assigned / rider.reassigned                 │
│                                                      │
│  Dependencies: B04 (Order), B02 (Store address)      │
└─────────────────────────────────────────────────────┘
```

**Entities:** `Shipment`, `Rider`, `DeliveryZone`, `Route`
**Value Objects:** `Address`, `Coordinates`, `Distance`

---

## 9. B08 — Inventory Management

```
┌─────────────────────────────────────────────────────┐
│                 B08 INVENTORY                        │
│                                                      │
│  ┌──────────────────┐  ┌──────────────────┐         │
│  │  Stock Service   │  │Reservation Svc   │         │
│  │                  │  │                  │         │
│  │ • getStock()     │  │ • reserve()      │         │
│  │ • updateStock()  │  │ • release()      │         │
│  │ • bulkUpdate()   │  │ • confirm()      │         │
│  │ • lowStockAlert()│  │ • getReserved()  │         │
│  └──────────────────┘  └──────────────────┘         │
│                                                      │
│  ┌──────────────────────────────────────────┐       │
│  │       Warehouse Service                   │       │
│  │                                           │       │
│  │ • createWarehouse()                       │       │
│  │ • transferStock()                         │       │
│  │ • getWarehouseStock()                     │       │
│  │ • adjustInventory()                       │       │
│  └──────────────────────────────────────────┘       │
│                                                      │
│  Domain Events Emitted:                              │
│  • stock.updated / stock.low                         │
│  • stock.reserved / stock.released                   │
│  • stock.transferred                                │
│                                                      │
│  Dependencies: B03 (Product), B02 (Vendor/Store)     │
└─────────────────────────────────────────────────────┘
```

**Entities:** `Stock`, `Reservation`, `Warehouse`, `StockMovement`
**Value Objects:** `Quantity`, `SKU`, `Location`

---

## 10. B09 — Storefront

```
┌─────────────────────────────────────────────────────┐
│                 B09 STOREFRONT                       │
│                                                      │
│  ┌──────────────────┐  ┌──────────────────┐         │
│  │Storefront Service│  │  Cart Service    │         │
│  │                  │  │                  │         │
│  │ • getStore()     │  │ • addToCart()    │         │
│  │ • getProducts()  │  │ • removeFromCart()│        │
│  │ • getFeatured()  │  │ • updateQuantity()│        │
│  │ • search()       │  │ • getCart()      │         │
│  │ • getReviews()   │  │ • clearCart()    │         │
│  └──────────────────┘  └──────────────────┘         │
│                                                      │
│  ┌──────────────────────────────────────────┐       │
│  │        Wishlist Service                   │       │
│  │                                           │       │
│  │ • addToWishlist()                         │       │
│  │ • removeFromWishlist()                    │       │
│  │ • getWishlist()                           │       │
│  │ • checkWishlist()                         │       │
│  └──────────────────────────────────────────┘       │
│                                                      │
│  Domain Events Emitted:                              │
│  • cart.itemAdded / cart.itemRemoved                 │
│  • wishlist.itemAdded / wishlist.itemRemoved         │
│                                                      │
│  Dependencies: B03 (Catalog), B08 (Stock),           │
│                B13 (Pricing), B10 (Reviews)           │
└─────────────────────────────────────────────────────┘
```

**Entities:** `Cart`, `CartItem`, `Wishlist`, `WishlistItem`
**Value Objects:** `Quantity`, `Money`

---

## 11. B10 — Trust & Engagement

```
┌─────────────────────────────────────────────────────┐
│                 B10 TRUST                            │
│                                                      │
│  ┌──────────────────┐  ┌──────────────────┐         │
│  │ Review Service   │  │ Loyalty Service  │         │
│  │                  │  │                  │         │
│  │ • submitReview() │  │ • earnPoints()   │         │
│  │ • respondReview()│  │ • redeemPoints() │         │
│  │ • flagReview()   │  │ • getBalance()   │         │
│  │ • getReviews()   │  │ • getHistory()   │         │
│  └──────────────────┘  └────────┬─────────┘         │
│                                  │                    │
│                                  ▼                    │
│  ┌──────────────────────────────────────────┐       │
│  │          Tier Service                     │       │
│  │                                           │       │
│  │ • calculateTier()                         │       │
│  │ • upgradeTier()                           │       │
│  │ • getTierBenefits()                       │       │
│  │ • getTierProgress()                       │       │
│  └──────────────────────────────────────────┘       │
│                                                      │
│  Domain Events Emitted:                              │
│  • review.submitted / review.flagged                 │
│  • loyalty.pointsEarned / loyalty.pointsRedeemed     │
│  • tier.upgraded / tier.downgraded                   │
│                                                      │
│  Dependencies: B01 (User), B04 (Order), B02 (Vendor) │
└─────────────────────────────────────────────────────┘
```

**Entities:** `Review`, `LoyaltyAccount`, `Tier`, `LoyaltyTransaction`
**Value Objects:** `Rating`, `Points`, `TierLevel`

---

## 12. B11 — Content Management

```
┌─────────────────────────────────────────────────────┐
│                 B11 CONTENT                          │
│                                                      │
│  ┌──────────────────┐  ┌──────────────────┐         │
│  │   CMS Service    │  │Notification Svc  │         │
│  │                  │  │                  │         │
│  │ • createPage()   │  │ • sendPush()     │         │
│  │ • updatePage()   │  │ • sendSMS()      │         │
│  │ • publishPage()  │  │ • sendEmail()    │         │
│  │ • getPage()      │  │ • sendInApp()    │         │
│  │ • listPages()    │  │ • getTemplates() │         │
│  └──────────────────┘  └──────────────────┘         │
│                                                      │
│  Domain Events Emitted:                              │
│  • page.published / page.updated                     │
│  • notification.sent / notification.failed           │
│                                                      │
│  Dependencies: B01 (User for targeting)              │
└─────────────────────────────────────────────────────┘
```

**Entities:** `Page`, `Notification`, `NotificationTemplate`
**Value Objects:** `NotificationChannel`, `TemplateData`

---

## 13. B12 — Support & Analytics

```
┌─────────────────────────────────────────────────────┐
│                B12 SUPPORT                           │
│                                                      │
│  ┌──────────────────┐  ┌──────────────────┐         │
│  │  Ticket Service  │  │Analytics Service │         │
│  │                  │  │                  │         │
│  │ • createTicket() │  │ • trackEvent()   │         │
│  │ • assignAgent()  │  │ • getMetrics()   │         │
│  │ • resolveTicket()│  │ • getReport()    │         │
│  │ • escalate()     │  │ • getDashboard() │         │
│  │ • getTickets()   │  │ • exportData()   │         │
│  └──────────────────┘  └──────────────────┘         │
│                                                      │
│  ┌──────────────────────────────────────────┐       │
│  │      ServiceBooking Service               │       │
│  │                                           │       │
│  │ • createBooking()                         │       │
│  │ • confirmBooking()                        │       │
│  │ • cancelBooking()                         │       │
│  │ • getBookings()                           │       │
│  └──────────────────────────────────────────┘       │
│                                                      │
│  Domain Events Emitted:                              │
│  • ticket.created / ticket.resolved                  │
│  • ticket.escalated                                  │
│  • analytics.eventTracked                            │
│  • booking.created / booking.confirmed               │
│                                                      │
│  Dependencies: B01 (User), B04 (Order context)       │
└─────────────────────────────────────────────────────┘
```

**Entities:** `Ticket`, `TicketMessage`, `Booking`, `AnalyticsEvent`
**Value Objects:** `TicketPriority`, `MetricValue`, `DateRange`

---

## 14. B13 — Pricing Rules

```
┌─────────────────────────────────────────────────────┐
│                 B13 PRICING                          │
│                                                      │
│  ┌──────────────────┐  ┌──────────────────┐         │
│  │ Coupon Service   │  │Discount Service  │         │
│  │                  │  │                  │         │
│  │ • createCoupon() │  │ • createDiscount()│        │
│  │ • validateCoupon │  │ • calculateDiscount()│     │
│  │ • applyCoupon()  │  │ • stackDiscounts()│        │
│  │ • expireCoupons()│  │ • getActiveRules()│        │
│  └──────────────────┘  └──────────────────┘         │
│                                                      │
│  Domain Events Emitted:                              │
│  • coupon.created / coupon.redeemed                  │
│  • coupon.expired                                    │
│  • discount.applied                                  │
│                                                      │
│  Dependencies: B04 (Order), B09 (Cart)               │
└─────────────────────────────────────────────────────┘
```

**Entities:** `Coupon`, `DiscountRule`, `DiscountApplication`
**Value Objects:** `Money`, `Percentage`, `CouponCode`

---

## 15. Cross-Module Event Flow

### 15.1 Order Placement Flow

```
Customer places order
        │
        ▼
   ┌─────────┐   order.placed    ┌─────────┐
   │   B04   │──────────────────▶│   B08   │
   │  Order  │                   │ Inventory│
   └────┬────┘                   └─────────┘
        │
        │ order.placed
        ▼
   ┌─────────┐   payment.request ┌─────────┐
   │   B05   │◀─────────────────│   B04   │
   │ Payment │                   │  Order  │
   └────┬────┘                   └─────────┘
        │
        │ payment.processed
        ▼
   ┌─────────┐   shipment.create ┌─────────┐
   │   B07   │◀─────────────────│   B04   │
   │Shipping │                   │  Order  │
   └────┬────┘                   └─────────┘
        │
        │ shipment.dispatched
        ▼
   ┌─────────┐   notification    ┌─────────┐
   │   B11   │◀─────────────────│   B07   │
   │ Content │                   │Shipping │
   └─────────┘                   └─────────┘
```

### 15.2 Commission & Payout Flow

```
Order completed
        │
        ▼
   ┌─────────┐   order.completed ┌─────────┐
   │   B04   │──────────────────▶│   B06   │
   │  Order  │                   │ Finance │
   └─────────┘                   └────┬────┘
                                       │
                                       │ commission.calculated
                                       ▼
                                  ┌─────────┐
                                  │   B06   │
                                  │ Finance │
                                  └────┬────┘
                                       │
                                       │ payout.processed
                                       ▼
                                  ┌─────────┐
                                  │   B05   │
                                  │ Payment │
                                  └─────────┘
```
