# Integration Points — YemenMart Platform

## 1. Overview

Complete catalog of all integration points: synchronous API calls, asynchronous events, external system integrations, and anti-corruption layers.

---

## 2. Synchronous API Calls (Inter-Block)

| Caller | Callee | Endpoint | Method | Purpose |
|--------|--------|----------|--------|---------|
| B09 | B03 | `/api/v1/products/{id}` | GET | Fetch product details for cart display |
| B09 | B03 | `/api/v1/products/{id}/variants/{vid}` | GET | Fetch variant price and availability |
| B09 | B03 | `/api/v1/categories/tree` | GET | Display category navigation |
| B09 | B08 | `/api/v1/stock/{variantId}/available` | GET | Check stock before add-to-cart |
| B09 | B13 | `/api/v1/coupons/validate` | POST | Validate coupon at checkout |
| B09 | B04 | `/api/v1/orders` | POST | Create master order at checkout |
| B04 | B02 | `/api/v1/stores/{id}` | GET | Fetch store info for sub-order splitting |
| B04 | B03 | `/api/v1/variants/{id}` | GET | Fetch variant details for order items |
| B04 | B08 | `/api/v1/stock/reserve` | POST | Reserve stock for order |
| B04 | B13 | `/api/v1/coupons/apply` | POST | Apply coupon discount to order |
| B04 | B05 | `/api/v1/escrow/create` | POST | Hold escrow for order payment |
| B05 | B04 | `/api/v1/orders/{id}/status` | PATCH | Update order status on payment |
| B05 | B06 | `/api/v1/commissions/calculate` | POST | Trigger commission on payment release |
| B06 | B02 | `/api/v1/stores/{id}/commission-rate` | GET | Fetch commission rate for calculations |
| B06 | B04 | `/api/v1/sub-orders/{id}` | GET | Fetch sub-order for payout calculation |
| B06 | B05 | `/api/v1/wallets/{id}/credit` | POST | Credit vendor wallet on payout |
| B07 | B04 | `/api/v1/sub-orders/{id}` | GET | Fetch delivery details |
| B07 | B02 | `/api/v1/stores/{id}/address` | GET | Fetch pickup address |
| B10 | B04 | `/api/v1/orders/{id}/items` | GET | Verify purchase for review eligibility |
| B10 | B03 | `/api/v1/variants/{id}` | GET | Fetch product info for review |
| B12 | B04 | `/api/v1/orders/{id}` | GET | Fetch order context for ticket |
| B12 | B01 | `/api/v1/users/{id}` | GET | Fetch user info for ticket assignment |
| B13 | B04 | `/api/v1/orders/{id}/total` | GET | Fetch order total for discount eligibility |
| B13 | B03 | `/api/v1/products/{id}/category` | GET | Fetch category for category-based rules |

---

## 3. Asynchronous Events (BullMQ)

### 3.1 Event Queues

| Queue | Consumer Blocks | Purpose |
|-------|----------------|---------|
| `yemenmart:orders` | B04, B05, B07, B08, B10, B11 | Order lifecycle events |
| `yemenmart:payments` | B04, B06, B11 | Payment/escrow events |
| `yemenmart:inventory` | B03, B04, B09, B11 | Stock change events |
| `yemenmart:logistics` | B04, B08, B11 | Delivery status events |
| `yemenmart:notifications` | B11 | Notification dispatch |
| `yemenmart:finance` | B11 | Payout/invoice events |
| `yemenmart:support` | B11 | Ticket escalation events |
| `yemenmart:engagement` | B11 | Review/loyalty events |

### 3.2 Complete Event Catalog

#### B01: Identity & Access
| Event | Payload | Consumers | Trigger |
|-------|---------|-----------|---------|
| `user.registered` | `{ userId, phone, role }` | B02, B05, B10 | New user signup |
| `user.phone_verified` | `{ userId, phone }` | B02 | OTP verified |
| `user.login` | `{ userId, sessionId, ip }` | AuditLog | Successful login |
| `user.logout` | `{ userId, sessionId }` | AuditLog | User logout |
| `user.role_changed` | `{ userId, oldRole, newRole }` | B02 | Admin role update |

#### B02: Marketplace
| Event | Payload | Consumers | Trigger |
|-------|---------|-----------|---------|
| `vendor.registered` | `{ vendorId, userId, businessName }` | B06, B11 | Vendor creation |
| `vendor.kyc_submitted` | `{ vendorId, kycId }` | B11 (notify admin) | KYC documents submitted |
| `vendor.kyc_approved` | `{ vendorId, kycId }` | B03, B06, B11 | KYC approved by admin |
| `vendor.kyc_rejected` | `{ vendorId, kycId, reason }` | B11 | KYC rejected |
| `vendor.suspended` | `{ vendorId, reason }` | B03, B04, B11 | Vendor suspended |
| `store.created` | `{ storeId, vendorId, name }` | B03, B11 | Store created |
| `store.activated` | `{ storeId }` | B03, B09 | Store activated |
| `store.deactivated` | `{ storeId }` | B03, B09 | Store deactivated |
| `store.commission_changed` | `{ storeId, oldRate, newRate }` | B06 | Commission rate updated |

#### B03: Catalog
| Event | Payload | Consumers | Trigger |
|-------|---------|-----------|---------|
| `product.created` | `{ productId, storeId }` | B09, B13 | Product created |
| `product.updated` | `{ productId, changes[] }` | B09 | Product updated |
| `product.published` | `{ productId, storeId }` | B09, B13, Elasticsearch | Product goes live |
| `product.archived` | `{ productId }` | B09, B13, Elasticsearch | Product archived |
| `variant.created` | `{ variantId, productId }` | B08 | New variant added |
| `variant.stock_changed` | `{ variantId, delta }` | B09 | Stock change propagated |
| `category.created` | `{ categoryId, parentId }` | Elasticsearch | New category |
| `category.updated` | `{ categoryId }` | Elasticsearch | Category updated |

#### B04: Commerce
| Event | Payload | Consumers | Trigger |
|-------|---------|-----------|---------|
| `order.placed` | `{ masterOrderId, userId, total }` | B05, B07, B08, B11 | Checkout complete |
| `order.confirmed` | `{ masterOrderId }` | B07, B11 | Payment confirmed |
| `order.paid` | `{ masterOrderId, amount }` | B06, B11 | Payment processed |
| `order.shipped` | `{ masterOrderId, subOrderId }` | B11 | Order shipped |
| `order.delivered` | `{ masterOrderId, subOrderId }` | B05, B06, B10, B11 | Delivery confirmed |
| `order.completed` | `{ masterOrderId }` | B10, B11 | Order finalized |
| `order.cancelled` | `{ masterOrderId, reason }` | B05, B08, B11 | Order cancelled |
| `return.requested` | `{ returnId, orderId, reason }` | B05, B11 | Return initiated |
| `return.approved` | `{ returnId }` | B05, B11 | Return approved |
| `return.rejected` | `{ returnId, reason }` | B11 | Return rejected |
| `return.completed` | `{ returnId }` | B05, B11 | Return processed |

#### B05: Payment
| Event | Payload | Consumers | Trigger |
|-------|---------|-----------|---------|
| `wallet.topup_initiated` | `{ walletId, amount, method }` | B11 | Top-up started |
| `wallet.topup_completed` | `{ walletId, amount, transactionId }` | B11 | Top-up successful |
| `wallet.debit` | `{ walletId, amount, reference }` | B06 | Wallet debited |
| `escrow.held` | `{ escrowId, orderId, amount }` | B04 | Funds held |
| `escrow.released` | `{ escrowId, orderId }` | B04, B06 | Funds released |
| `escrow.refunded` | `{ escrowId, orderId }` | B04, B06 | Funds refunded |
| `refund.initiated` | `{ refundId, orderId, amount }` | B11 | Refund started |
| `refund.completed` | `{ refundId, orderId }` | B04, B11 | Refund successful |
| `refund.failed` | `{ refundId, orderId, reason }` | B04, B11, B12 | Refund failed |

#### B06: Finance
| Event | Payload | Consumers | Trigger |
|-------|---------|-----------|---------|
| `commission.calculated` | `{ commissionId, vendorId, amount }` | B11 | Commission computed |
| `commission.settled` | `{ commissionId, vendorId }` | Ledger | Commission settled |
| `payout.created` | `{ payoutId, vendorId, amount }` | B11 | Payout batch created |
| `payout.processing` | `{ payoutId }` | B11 | Payout being processed |
| `payout.completed` | `{ payoutId, vendorId, amount }` | B11 | Payout sent |
| `payout.failed` | `{ payoutId, reason }` | B11, B12 | Payout failed |
| `invoice.generated` | `{ invoiceId, vendorId }` | B11 | Invoice ready |

#### B07: Logistics
| Event | Payload | Consumers | Trigger |
|-------|---------|-----------|---------|
| `delivery.assigned` | `{ assignmentId, subOrderId, providerId }` | B04, B11 | Delivery assigned |
| `delivery.picked_up` | `{ assignmentId }` | B04, B11 | Package picked up |
| `delivery.in_transit` | `{ assignmentId, lat, lng }` | B04, B11 | In transit |
| `delivery.delivered` | `{ assignmentId, podId }` | B04, B05, B08, B10, B11 | Delivery complete |
| `delivery.failed` | `{ assignmentId, reason }` | B04, B11, B12 | Delivery failed |
| `delivery.returned` | `{ assignmentId }` | B04, B05, B11 | Package returned |

#### B08: Inventory
| Event | Payload | Consumers | Trigger |
|-------|---------|-----------|---------|
| `stock.reserved` | `{ reservationId, variantId, qty }` | B04 | Stock reserved |
| `stock.committed` | `{ reservationId, variantId, qty }` | B04 | Stock committed |
| `stock.released` | `{ reservationId, variantId, qty }` | B04, B03 | Stock released |
| `stock.low` | `{ variantId, remaining }` | B02, B11 | Low stock threshold |
| `stock.out_of_stock` | `{ variantId }` | B03, B09, B11 | Out of stock |
| `stock.adjusted` | `{ stockId, delta, reason }` | AuditLog | Manual adjustment |

#### B09: Storefront
| Event | Payload | Consumers | Trigger |
|-------|---------|-----------|---------|
| `cart.item_added` | `{ cartId, variantId, qty }` | B08 (analytics) | Item added to cart |
| `cart.item_removed` | `{ cartId, variantId }` | — | Item removed |
| `cart.checkout_initiated` | `{ cartId, userId }` | B04, B08, B05 | Checkout started |
| `wishlist.item_added` | `{ wishlistId, variantId }` | — | Item wishlisted |
| `wishlist.item_removed` | `{ wishlistId, variantId }` | — | Item removed from wishlist |
| `store.followed` | `{ userId, storeId }` | B10, B11 | Store followed |
| `store.unfollowed` | `{ userId, storeId }` | B10 | Store unfollowed |

#### B10: Engagement
| Event | Payload | Consumers | Trigger |
|-------|---------|-----------|---------|
| `review.created` | `{ reviewId, userId, variantId, rating }` | B02, B11 | Review submitted |
| `review.flagged` | `{ reviewId, reason }` | B11 | Review flagged |
| `review.helpful` | `{ reviewId, voteType }` | — | Review voted |
| `loyalty.points_earned` | `{ accountId, points, reference }` | B11 | Points earned |
| `loyalty.points_redeemed` | `{ accountId, points, reference }` | B11 | Points redeemed |
| `loyalty.tier_upgraded` | `{ accountId, oldTier, newTier }` | B11 | Tier upgraded |
| `loyalty.tier_downgraded` | `{ accountId, oldTier, newTier }` | B11 | Tier downgraded |

#### B11: Content
| Event | Payload | Consumers | Trigger |
|-------|---------|-----------|---------|
| `notification.sent` | `{ notificationId, userId, channel }` | — | Notification dispatched |
| `notification.read` | `{ notificationId }` | — | Notification read |
| `banner.impression` | `{ bannerId, userId }` | B02 (analytics) | Banner viewed |
| `banner.clicked` | `{ bannerId, userId }` | B02 (analytics) | Banner clicked |

#### B12: Support
| Event | Payload | Consumers | Trigger |
|-------|---------|-----------|---------|
| `ticket.created` | `{ ticketId, userId, subject }` | B11 | Ticket created |
| `ticket.assigned` | `{ ticketId, agentId }` | B11 | Ticket assigned |
| `ticket.escalated` | `{ ticketId, reason }` | B11 | Ticket escalated |
| `ticket.resolved` | `{ ticketId }` | B11, B10 | Ticket resolved |
| `ticket.closed` | `{ ticketId }` | B11 | Ticket closed |
| `ticket.sla_breach` | `{ ticketId, sla }` | B11 | SLA breached |
| `booking.created` | `{ bookingId, userId, scheduledAt }` | B11 | Booking created |
| `booking.confirmed` | `{ bookingId }` | B11 | Booking confirmed |
| `booking.completed` | `{ bookingId }` | B11 | Booking completed |

#### B13: Promotion
| Event | Payload | Consumers | Trigger |
|-------|---------|-----------|---------|
| `coupon.created` | `{ couponId, code }` | B09 | Coupon created |
| `coupon.expired` | `{ couponId }` | B09 | Coupon expired |
| `coupon.usage_limit_reached` | `{ couponId }` | B09 | Max uses reached |
| `discount.applied` | `{ couponId, orderId, amount }` | B04 | Discount applied |
| `discount.rejected` | `{ couponId, reason }` | B04 | Discount rejected |
| `promotion.activated` | `{ promotionId }` | B09 | Promotion active |
| `promotion.deactivated` | `{ promotionId }` | B09 | Promotion inactive |

---

## 4. External System Integrations

### 4.1 m-Floos (Mobile Wallet)

| Property | Value |
|----------|-------|
| **Type** | Payment Gateway |
| **Protocol** | REST API over HTTPS |
| **Auth** | mTLS (client certificate) + API key |
| **Base URL** | `https://api.m-floos.com/v1` |
| **Integrating Block** | B05 Payment |
| **Timeout** | 30s (charge), 60s (refund) |
| **Retry** | 3 attempts, exponential backoff (1s, 2s, 4s) |
| **Circuit Breaker** | Open after 5 consecutive failures; half-open after 60s |

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/charge` | POST | Charge customer wallet |
| `/refund` | POST | Refund to customer wallet |
| `/balance` | GET | Check wallet balance |
| `/transfer` | POST | Transfer between wallets |
| `/status/{ref}` | GET | Check transaction status |

### 4.2 OneCash (Mobile Wallet)

| Property | Value |
|----------|-------|
| **Type** | Payment Gateway |
| **Protocol** | REST API over HTTPS |
| **Auth** | mTLS + API key |
| **Base URL** | `https://api.onecash.ye/v2` |
| **Integrating Block** | B05 Payment |
| **Timeout** | 30s (charge), 60s (refund) |
| **Retry** | 3 attempts, exponential backoff |
| **Circuit Breaker** | Same as m-Floos |

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/payment/charge` | POST | Charge customer wallet |
| `/payment/refund` | POST | Refund to customer wallet |
| `/wallet/balance` | GET | Check wallet balance |
| `/payment/status/{id}` | GET | Check transaction status |

### 4.3 Telesom SMS

| Property | Value |
|----------|-------|
| **Type** | SMS Provider |
| **Protocol** | REST API over HTTPS |
| **Auth** | API key in header |
| **Base URL** | `https://sms.telesom.com/api/v1` |
| **Integrating Block** | B01 Identity |
| **Timeout** | 10s |
| **Retry** | 2 attempts |
| **Rate Limit** | 10 SMS/second |

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/send` | POST | Send OTP or transactional SMS |
| `/status/{id}` | GET | Check delivery status |

### 4.4 Sabafon SMS

| Property | Value |
|----------|-------|
| **Type** | SMS Provider |
| **Protocol** | REST API over HTTPS |
| **Auth** | API key in header |
| **Base URL** | `https://api.sabafon.com/sms/v1` |
| **Integrating Block** | B01 Identity |
| **Timeout** | 10s |
| **Retry** | 2 attempts |
| **Rate Limit** | 10 SMS/second |

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/send` | POST | Send OTP or transactional SMS |
| `/status/{id}` | GET | Check delivery status |

### 4.5 WhatsApp Business

| Property | Value |
|----------|-------|
| **Type** | Messaging Platform |
| **Protocol** | Cloud API (HTTPS) |
| **Auth** | Bearer token |
| **Base URL** | `https://graph.facebook.com/v17.0/{phone_id}` |
| **Integrating Block** | B11 Content |
| **Timeout** | 15s |
| **Retry** | 3 attempts |
| **Webhook** | Inbound messages verified via X-Hub-Signature |

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/messages` | POST | Send template/free-form message |
| `/media` | POST | Send media (images, documents) |
| Webhook (inbound) | POST | Receive customer replies |

### 4.6 ZATCA (E-Invoicing)

| Property | Value |
|----------|-------|
| **Type** | Government Compliance |
| **Protocol** | REST API over HTTPS |
| **Auth** | mTLS (client certificate) |
| **Base URL** | `https://gw-fatoora.zatca.gov.sa/e-invoicing/shipment` (Production) |
| **Integrating Block** | B06 Finance |
| **Timeout** | 30s |
| **Retry** | 1 attempt (idempotent by UUID) |
| **Compliance** | ZATCA Phase 2 FATOORA |

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/invoice/register` | POST | Register and submit e-invoice |
| `/invoice/status/{uuid}` | GET | Check invoice status |
| `/clearance/report` | POST | Submit clearance invoice |
| `/report/status/{uuid` | GET | Check report status |

### 4.7 Elasticsearch

| Property | Value |
|----------|-------|
| **Type** | Search Engine |
| **Protocol** | REST API (HTTP/HTTPS) |
| **Auth** | API key / Basic auth |
| **Base URL** | `https://es-cluster.yemenmart.com` |
| **Integrating Blocks** | B03 (write), B09 (read), B04 (read) |
| **Timeout** | 5s (search), 10s (index) |
| **Retry** | 2 attempts |

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/products/_search` | POST | Full-text product search |
| `/products/_bulk` | POST | Bulk index/update products |
| `/products/_suggest` | POST | Autocomplete suggestions |
| `/categories/_search` | POST | Category search |
| `/products/{id}` | GET | Get indexed product |

---

## 5. Anti-Corruption Layers (ACL)

### ACL Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    ANTI-CORRUPTION LAYERS                       │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │ ACL-B02: Marketplace ACL                                  │  │
│  │ Consumer: B03, B04, B06 │ Provider: B02                  │  │
│  │ Pattern: Shared Kernel │ Transport: In-process            │  │
│  │ Interface: VendorQueryService, StoreQueryService          │  │
│  │ Validation: Store existence, KYC status, commission rate  │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │ ACL-B03: Catalog ACL                                      │  │
│  │ Consumer: B04, B09, B13 │ Provider: B03                  │  │
│  │ Pattern: Shared Kernel │ Transport: In-process            │  │
│  │ Interface: ProductQueryService, CategoryQueryService      │  │
│  │ Validation: Product existence, price, availability        │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │ ACL-B04: Commerce ACL                                     │  │
│  │ Consumer: B05, B07, B08 │ Provider: B04                  │  │
│  │ Pattern: Shared Kernel │ Transport: In-process            │  │
│  │ Interface: OrderQueryService, ReturnQueryService          │  │
│  │ Validation: Order existence, status, total                │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │ ACL-B05: Payment ACL                                      │  │
│  │ Consumer: B04, B06 │ Provider: B05                       │  │
│  │ Pattern: Shared Kernel │ Transport: In-process            │  │
│  │ Interface: WalletQueryService, EscrowQueryService         │  │
│  │ Validation: Balance sufficiency, escrow status            │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │ ACL-B06: Finance ACL                                      │  │
│  │ Consumer: B05 │ Provider: B06                             │  │
│  │ Pattern: Shared Kernel │ Transport: In-process            │  │
│  │ Interface: CommissionQueryService, PayoutQueryService     │  │
│  │ Validation: Commission rate, payout threshold             │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │ ACL-B08: Inventory ACL                                    │  │
│  │ Consumer: B04, B09 │ Provider: B08                       │  │
│  │ Pattern: Shared Kernel │ Transport: In-process            │  │
│  │ Interface: StockQueryService, ReservationQueryService     │  │
│  │ Validation: Stock availability, reservation expiry        │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │ ACL-B12: Support ACL                                      │  │
│  │ Consumer: B04, B01 │ Provider: B12                       │  │
│  │ Pattern: Shared Kernel │ Transport: In-process            │  │
│  │ Interface: TicketQueryService, BookingQueryService        │  │
│  │ Validation: Ticket SLA, booking availability              │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### ACL Implementation Pattern

```typescript
// Example: ACL-B05 (Payment ACL)
interface PaymentACL {
  // Query interface exposed to other blocks
  getBalance(walletId: string): Promise<Money>;
  getEscrowByOrderId(orderId: string): Promise<EscrowDTO | null>;
  
  // Command interface exposed to other blocks
  holdEscrow(orderId: string, amount: Money): Promise<EscrowDTO>;
  releaseEscrow(escrowId: string): Promise<void>;
  
  // Validation at ACL boundary
  validateSufficientBalance(walletId: string, amount: Money): Promise<boolean>;
}

// Consumer code (B04 Commerce)
class OrderService {
  constructor(private paymentACL: PaymentACL) {}
  
  async placeOrder(order: Order): Promise<void> {
    // Validate through ACL
    const hasBalance = await this.paymentACL.validateSufficientBalance(
      order.walletId, 
      order.total
    );
    if (!hasBalance) throw new InsufficientBalanceError();
    
    // Hold escrow through ACL
    await this.paymentACL.holdEscrow(order.id, order.total);
  }
}
```

### ACL Validation Matrix

| ACL | Validates | Rejects | Fallback |
|-----|-----------|---------|----------|
| ACL-B02 | Store exists, is active, KYC approved | Direct vendor data access | Return 404 |
| ACL-B03 | Product exists, is published, price valid | Direct variant modification | Return 404 |
| ACL-B04 | Order exists, status allows operation | Status manipulation from B07 | Return 409 Conflict |
| ACL-B05 | Balance sufficient, escrow in valid state | Balance bypass, transaction tampering | Return 402 Payment Required |
| ACL-B06 | Commission rate valid, payout threshold met | Rate modification, ledger tampering | Return 400 Bad Request |
| ACL-B08 | Stock available, reservation not expired | Stock bypass, reservation extension | Return 409 Conflict |
| ACL-B12 | Ticket exists, SLA within bounds | Order modification from support | Return 403 Forbidden |

---

## 6. Integration Resilience

### 6.1 Circuit Breaker Configuration

| External System | Failure Threshold | Open Duration | Half-Open Requests |
|----------------|-------------------|---------------|-------------------|
| m-Floos | 5 failures | 60s | 3 |
| OneCash | 5 failures | 60s | 3 |
| Telesom SMS | 3 failures | 30s | 2 |
| Sabafon SMS | 3 failures | 30s | 2 |
| WhatsApp Business | 5 failures | 60s | 3 |
| ZATCA | 3 failures | 120s | 2 |
| Elasticsearch | 5 failures | 30s | 3 |

### 6.2 Retry Policies

| Integration | Max Retries | Strategy | Initial Delay | Max Delay |
|------------|-------------|----------|---------------|-----------|
| m-Floos | 3 | Exponential backoff | 1s | 4s |
| OneCash | 3 | Exponential backoff | 1s | 4s |
| Telesom SMS | 2 | Linear backoff | 2s | 4s |
| Sabafon SMS | 2 | Linear backoff | 2s | 4s |
| WhatsApp Business | 3 | Exponential backoff | 1s | 8s |
| ZATCA | 1 | None (idempotent) | — | — |
| Elasticsearch | 2 | Linear backoff | 1s | 2s |

### 6.3 Idempotency Keys

| Integration | Key Source | TTL |
|------------|-----------|-----|
| m-Floos | `{orderId}:{timestamp}` | 24h |
| OneCash | `{orderId}:{timestamp}` | 24h |
| ZATCA | `{invoiceUUID}` | Permanent |
| Stock Reservation | `{orderId}:{variantId}` | 15min |

### 6.4 Dead Letter Queue (DLQ)

| Queue | DLQ | Retry Policy | Alert After |
|-------|-----|-------------|-------------|
| `yemenmart:orders` | `yemenmart:orders:dlq` | Manual retry | 3 failures |
| `yemenmart:payments` | `yemenmart:payments:dlq` | Auto-retry 3x | 5 failures |
| `yemenmart:inventory` | `yemenmart:inventory:dlq` | Manual retry | 3 failures |
| `yemenmart:logistics` | `yemenmart:logistics:dlq` | Auto-retry 3x | 5 failures |
| `yemenmart:notifications` | `yemenmart:notifications:dlq` | Auto-retry 5x | 10 failures |
| `yemenmart:finance` | `yemenmart:finance:dlq` | Manual retry | 1 failure |
| `yemenmart:support` | `yemenmart:support:dlq` | Auto-retry 3x | 3 failures |
| `yemenmart:engagement` | `yemenmart:engagement:dlq` | Auto-retry 3x | 5 failures |

---

## 7. External System Integration Summary

```
┌─────────────────────────────────────────────────────────────────────┐
│                     EXTERNAL INTEGRATIONS                           │
│                                                                     │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ PAYMENT GATEWAYS                                              │  │
│  │                                                               │  │
│  │  ┌─────────────┐         ┌─────────────┐                     │  │
│  │  │  m-Floos    │◄────────│  B05        │                     │  │
│  │  │  (REST/mTLS)│────────►│  Payment    │                     │  │
│  │  └─────────────┘         │             │                     │  │
│  │                          │             │                     │  │
│  │  ┌─────────────┐         │             │                     │  │
│  │  │  OneCash    │◄────────│             │                     │  │
│  │  │  (REST/mTLS)│────────►│             │                     │  │
│  │  └─────────────┘         └─────────────┘                     │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                     │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ MESSAGING                                                     │  │
│  │                                                               │  │
│  │  ┌─────────────┐         ┌─────────────┐                     │  │
│  │  │  Telesom    │◄────────│  B01        │                     │  │
│  │  │  SMS        │         │  Identity   │                     │  │
│  │  └─────────────┘         └─────────────┘                     │  │
│  │                                                               │  │
│  │  ┌─────────────┐         ┌─────────────┐                     │  │
│  │  │  Sabafon    │◄────────│  B01        │                     │  │
│  │  │  SMS        │         │  Identity   │                     │  │
│  │  └─────────────┘         └─────────────┘                     │  │
│  │                                                               │  │
│  │  ┌─────────────┐         ┌─────────────┐                     │  │
│  │  │  WhatsApp   │◄────────│  B11        │                     │  │
│  │  │  Business   │────────►│  Content    │                     │  │
│  │  │  (Cloud API)│         └─────────────┘                     │  │
│  │  └─────────────┘                                             │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                     │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ COMPLIANCE & SEARCH                                           │  │
│  │                                                               │  │
│  │  ┌─────────────┐         ┌─────────────┐                     │  │
│  │  │  ZATCA      │◄────────│  B06        │                     │  │
│  │  │  (REST/mTLS)│         │  Finance    │                     │  │
│  │  └─────────────┘         └─────────────┘                     │  │
│  │                                                               │  │
│  │  ┌─────────────┐     ┌─────────┐                             │  │
│  │  │Elasticsearch│◄────│  B03    │                             │  │
│  │  │  (REST)     │────►│ Catalog │                             │  │
│  │  │             │◄────│  B09    │                             │  │
│  │  │             │     │Storefrt.│                             │  │
│  │  └─────────────┘     └─────────┘                             │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```
