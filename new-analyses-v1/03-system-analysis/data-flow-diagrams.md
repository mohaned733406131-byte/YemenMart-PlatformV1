# Data Flow Diagrams — YemenMart Platform

## 1. Overview

This document defines the critical data flows through the YemenMart bounded context architecture. Each flow shows the sequence of block interactions, events published, and data mutations.

---

## 2. Flow 1: Order Lifecycle (End-to-End)

**Trigger**: Customer initiates checkout from storefront

```
┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐
│   B09   │   │   B04   │   │   B08   │   │   B13   │   │   B05   │
│Storefrt.│   │Commerce │   │Inventry │   │Promotion│   │ Payment │
└────┬────┘   └────┬────┘   └────┬────┘   └────┬────┘   └────┬────┘
     │             │             │             │             │
     │ 1.checkout  │             │             │             │
     │────────────►│             │             │             │
     │             │             │             │             │
     │             │ 2.reserve   │             │             │
     │             │────────────►│             │             │
     │             │             │             │             │
     │             │ ◄──stock.reserved          │             │
     │             │             │             │             │
     │             │ 3.validate coupon          │             │
     │             │──────────────────────────►│             │
     │             │             │             │             │
     │             │ ◄──discount.applied        │             │
     │             │             │             │             │
     │             │ 4.hold escrow              │             │
     │             │─────────────────────────────────────────►
     │             │             │             │             │
     │             │ ◄──escrow.held             │             │
     │             │             │             │             │
     │ ◄──order.placed (confirmation)          │             │
     │             │             │             │             │
     │             │             │             │             │
     │ ─ ─ ─ ─ ─  PAYMENT PROCESSING  ─ ─ ─ ─ ─ ─ ─ ─ ─ ─│
     │             │             │             │             │
     │             │ 5.payment success           │             │
     │             │─────────────────────────────────────────►
     │             │             │             │             │
     │             │ ◄──escrow.confirmed         │             │
     │             │             │             │             │
     │             │ 6.commit stock              │             │
     │             │────────────►│             │             │
     │             │             │             │             │
     │             │ ◄──stock.committed          │             │
     │             │             │             │             │
     └─────────────┴─────────────┴─────────────┴─────────────┘
```

### Detailed Sequence

| Step | Block | Action | Event Published | Data Mutated |
|------|-------|--------|-----------------|--------------|
| 1 | B09 | `Cart.checkout()` | `cart.checkout_initiated` | Cart status → checkout |
| 2 | B04 | `MasterOrder.create()` | `order.placed` | MasterOrder, SubOrders created |
| 3 | B08 | `Stock.reserve()` | `stock.reserved` | Reservation created, availableQty decremented |
| 4 | B13 | `Coupon.validate()` | `discount.applied` | Usage recorded |
| 5 | B05 | `Escrow.hold()` | `escrow.held` | Escrow created, wallet debited |
| 6 | B04 | `Order.confirm()` | `order.confirmed` | MasterOrder status → confirmed |
| 7 | B05 | `Payment.process()` | `payment.completed` | Transaction recorded |
| 8 | B08 | `Stock.commit()` | `stock.committed` | Reservation → committed, reservedQty decremented |
| 9 | B07 | `Assignment.create()` | `delivery.assigned` | Assignment created |
| 10 | B07 | `Assignment.deliver()` | `delivery.delivered` | POD recorded |
| 11 | B05 | `Escrow.release()` | `escrow.released` | Escrow status → released, vendor wallet credited |
| 12 | B06 | `Commission.calculate()` | `commission.calculated` | Commission entry created |
| 13 | B06 | `Payout.settle()` | `commission.settled` | LedgerEntry created |
| 14 | B10 | `Review.allow()` | — | Review eligibility enabled |
| 15 | B04 | `Order.complete()` | `order.completed` | MasterOrder status → completed |
| 16 | B11 | `Notification.send()` | `notification.sent` | Customer notified |

### Status Flow
```
B09 Cart ──► B04 MasterOrder ──► B08 Reservation ──► B05 Escrow ──► B07 Assignment
│checkout     │placed              │reserved            │held          │assigned
│             │confirmed           │committed           │              │picked_up
│             │paid                │                    │              │in_transit
│             │shipped             │                    │              │delivered
│             │delivered           │                    │released      │
│             │completed           │                    │              │
```

---

## 3. Flow 2: Vendor Onboarding

**Trigger**: Vendor signs up to sell on YemenMart

```
┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐
│   B09   │   │   B01   │   │   B02   │   │   B03   │
│Storefrt.│   │Identity │   │Marketpl.│   │Catalog  │
└────┬────┘   └────┬────┘   └────┬────┘   └────┬────┘
     │             │             │             │
     │ 1.register  │             │             │
     │────────────►│             │             │
     │             │             │             │
     │             │ 2.user.registered           │
     │             │────────────►│             │
     │             │             │             │
     │             │ 3.create vendor             │
     │             │             │             │
     │ ◄──OTP sent (B01 → Telesom/Sabafon)     │
     │             │             │             │
     │ 4.verify OTP│             │             │
     │────────────►│             │             │
     │             │             │             │
     │             │ 5.user.phone_verified       │
     │             │────────────►│             │
     │             │             │             │
     │             │ 6.vendor.kyc_unlocked       │
     │             │             │             │
     │ 7.submit KYC│             │             │
     │──────────────────────────►│             │
     │             │             │             │
     │             │ 8.vendor.kyc_submitted      │
     │             │             │             │
     │ ◄──KYC under review        │             │
     │             │             │             │
     │             │  [Admin reviews KYC]        │
     │             │             │             │
     │             │ 9.vendor.kyc_approved       │
     │             │────────────►│             │
     │             │             │             │
     │             │ 10.store.created            │
     │             │────────────►│             │
     │             │             │             │
     │             │ 11.catalog enabled          │
     │             │──────────────────────────►│
     │             │             │             │
     │             │             │ 12.vendor ready to sell│
     │             │             │             │
     └─────────────┴─────────────┴─────────────┘
```

### Detailed Sequence

| Step | Block | Action | Event Published | Data Mutated |
|------|-------|--------|-----------------|--------------|
| 1 | B09 | `VendorRegistration.submit()` | — | Registration form submitted |
| 2 | B01 | `User.register()` | `user.registered` | User created, OTP sent |
| 3 | B02 | `Vendor.create()` (on event) | `vendor.registered` | Vendor entity created (pending) |
| 4 | B01 | `OTP.send()` | — | OTP via Telesom/Sabafon SMS |
| 5 | B01 | `OTP.verify()` | `user.phone_verified` | User phone verified |
| 6 | B02 | `KYC.unlock()` | — | KYC submission enabled |
| 7 | B09 | `KYC.submit()` | — | Documents uploaded |
| 8 | B02 | `KYC.process()` | `vendor.kyc_submitted` | KYC status → pending |
| 9 | B02 | `KYC.approve()` (admin) | `vendor.kyc_approved` | Vendor status → approved |
| 10 | B02 | `Store.create()` | `store.created` | Store entity created |
| 11 | B03 | `Catalog.enable()` | — | Product publishing enabled |

---

## 4. Flow 3: Wallet Top-Up

**Trigger**: Customer adds funds to wallet via m-Floos or OneCash

```
┌─────────┐   ┌─────────┐   ┌─────────┐
│   B09   │   │   B05   │   │   B11   │
│Storefrt.│   │ Payment │   │ Content │
└────┬────┘   └────┬────┘   └────┬────┘
     │             │             │
     │ 1.topup.initiate          │
     │────────────►│             │
     │             │             │
     │             │ 2.charge external gateway
     │             │ ────────────► m-Floos / OneCash
     │             │ ◄──────────── gateway response
     │             │             │
     │             │ 3.wallet.topup_initiated
     │             │             │
     │             │ [gateway confirms payment]
     │             │             │
     │             │ 4.wallet.topup_completed
     │             │────────────►│
     │             │             │
     │ ◄──topup.success│ 5.notification.sent
     │             │────────────►│
     │             │             │
     │             │             │ 6.push/email to customer
     │             │             │
     └─────────────┴─────────────┘
```

### Detailed Sequence

| Step | Block | Action | Event Published | Data Mutated |
|------|-------|--------|-----------------|--------------|
| 1 | B09 | `Wallet.topup(amount, method)` | `wallet.topup_initiated` | Top-up request created |
| 2 | B05 | `Gateway.charge(amount, method)` | — | API call to m-Floos/OneCash |
| 3 | B05 | `Transaction.record()` | — | Pending transaction logged |
| 4 | B05 | `Wallet.credit(amount)` | `wallet.topup_completed` | Wallet balance incremented |
| 5 | B05 | `Transaction.finalize()` | — | Transaction status → completed |
| 6 | B11 | `Notification.send()` | `notification.sent` | Push/email confirmation |

### Data Flow
```
Customer ──[amount, method]──► B09 ──[topup request]──► B05
B05 ──[charge API]──► m-Floos/OneCash ──[response]──► B05
B05 ──[balance update]──► Wallet
B05 ──[event]──► B11 ──[notification]──► Customer
```

---

## 5. Flow 4: Refund

**Trigger**: Customer requests return; admin approves; refund processed

```
┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐
│   B09   │   │   B04   │   │   B05   │   │   B11   │   │   B06   │
│Storefrt.│   │Commerce │   │ Payment │   │ Content │   │ Finance │
└────┬────┘   └────┬────┘   └────┬────┘   └────┬────┘   └────┬────┘
     │             │             │             │             │
     │ 1.request return           │             │             │
     │────────────►│             │             │             │
     │             │             │             │             │
     │             │ 2.return.requested          │             │
     │             │──────────────────────────►│             │
     │             │             │             │             │
     │ ◄──return acknowledged     │ 3.notification.sent       │
     │             │             │────────────►│             │
     │             │             │             │             │
     │             │ [admin approves return]    │             │
     │             │             │             │             │
     │             │ 4.return.approved           │             │
     │             │──────────────────────────►│             │
     │             │             │             │             │
     │             │ 5.initiate refund           │             │
     │             │────────────►│             │             │
     │             │             │             │             │
     │             │             │ 6.refund.initiated         │
     │             │             │────────────►│             │
     │             │             │             │             │
     │ ◄──refund processing         7.notification.sent       │
     │             │             │────────────►│             │
     │             │             │             │             │
     │             │             │ [gateway processes refund] │
     │             │             │             │             │
     │             │             │ 8.refund.completed         │
     │             │             │────────────►│             │
     │             │             │             │             │
     │ ◄──refund complete   9.notification.sent│             │
     │             │             │────────────►│             │
     │             │             │             │             │
     │             │ 10.return.completed          │             │
     │             │             │             │             │
     │             │ 11.update commission          │             │
     │             │───────────────────────────────────────►│
     │             │             │             │             │
     └─────────────┴─────────────┴─────────────┴─────────────┘
```

### Detailed Sequence

| Step | Block | Action | Event Published | Data Mutated |
|------|-------|--------|-----------------|--------------|
| 1 | B09 | `ReturnRequest.create()` | `return.requested` | ReturnRequest created |
| 2 | B04 | `ReturnRequest.process()` | — | Status → pending |
| 3 | B11 | `Notification.send()` | `notification.sent` | Customer notified |
| 4 | B04 | `ReturnRequest.approve()` (admin) | `return.approved` | Status → approved |
| 5 | B05 | `Refund.initiate()` | `refund.initiated` | Refund record created |
| 6 | B05 | `Gateway.refund()` | — | API call to m-Floos/OneCash |
| 7 | B05 | `Wallet.credit()` | `refund.completed` | Customer wallet credited |
| 8 | B04 | `ReturnRequest.complete()` | `return.completed` | Status → completed |
| 9 | B11 | `Notification.send()` | `notification.sent` | Refund confirmation sent |
| 10 | B06 | `Commission.adjust()` | `commission.adjusted` | Commission reversed |

---

## 6. Flow 5: Coupon Redemption

**Trigger**: Customer applies coupon code at checkout

```
┌─────────┐   ┌─────────┐   ┌─────────┐
│   B09   │   │   B13   │   │   B05   │
│Storefrt.│   │Promotion│   │ Payment │
└────┬────┘   └────┬────┘   └────┬────┘
     │             │             │
     │ 1.apply coupon             │
     │────────────►│             │
     │             │             │
     │             │ 2.validate: │
     │             │  - code exists               │
     │             │  - is active                 │
     │             │  - not expired               │
     │             │  - usage < maxUses            │
     │             │  - user < maxUsesPerUser      │
     │             │  - order ≥ minOrderAmount     │
     │             │  - eligible items in cart     │
     │             │             │
     │ ◄──discount.applied         │
     │             │             │
     │             │ 3.record usage               │
     │             │             │
     │ 4.display discounted total │
     │             │             │
     │ 5.proceed to payment        │
     │────────────────────────────►│
     │             │             │
     │             │ 6.apply discount to escrow   │
     │             │             │
     └─────────────┴─────────────┘
```

### Detailed Sequence

| Step | Block | Action | Event Published | Data Mutated |
|------|-------|--------|-----------------|--------------|
| 1 | B09 | `Coupon.apply(code)` | — | Coupon code submitted |
| 2 | B13 | `Coupon.validate(code, order)` | — | Validation checks passed |
| 3 | B13 | `Usage.record()` | `discount.applied` | Usage entry created |
| 4 | B09 | `Cart.applyDiscount(amount)` | — | Cart total updated |
| 5 | B05 | `Escrow.applyDiscount()` | — | Escrow amount reduced |

### Validation Rules
```
Coupon.validate(code, order):
  1. coupon.isActive = true
  2. coupon.startsAt ≤ now ≤ coupon.endsAt
  3. coupon.usedCount < coupon.maxUses
  4. userUsage < coupon.maxUsesPerUser
  5. order.subtotal ≥ coupon.minOrderAmount
  6. coupon.allowedStores contains order.storeId (if specified)
  7. coupon.allowedCategories ∩ order.categories (if specified)
  
  if type = "fixed":
    discount = min(coupon.value, order.subtotal)
  if type = "percent":
    discount = order.subtotal × (coupon.value / 100)
    discount = min(discount, coupon.maxDiscount ?? ∞)
  if type = "free_shipping":
    discount = min(order.shippingFee, coupon.maxDiscount ?? 5000)
```

---

## 7. Flow Summary Matrix

| Flow | Start Block | End Block | Events | Critical Invariant |
|------|------------|-----------|--------|-------------------|
| Order Lifecycle | B09 | B11 | 12+ events | Stock reservation → Payment → Commit |
| Vendor Onboarding | B09 | B03 | 6 events | KYC approved → Store created |
| Wallet Top-Up | B09 | B11 | 3 events | Gateway confirms → Balance credited |
| Refund | B09 | B06 | 8 events | Return approved → Refund processed |
| Coupon Redemption | B09 | B05 | 2 events | Usage recorded → Discount applied |

## 8. Event Bus Topology

```
┌─────────────────────────────────────────────────────────────┐
│                     Redis / BullMQ                          │
│                                                             │
│  Queue: orders          Queue: payments    Queue: logistics │
│  ┌──────────────┐      ┌──────────────┐   ┌──────────────┐ │
│  │ order.placed │      │ escrow.held  │   │ delivery.    │ │
│  │ order.paid   │      │ escrow.      │   │   assigned   │ │
│  │ order.       │      │   released   │   │ delivery.    │ │
│  │   delivered  │      │ refund.      │   │   delivered  │ │
│  │ order.       │      │   initiated  │   │ delivery.    │ │
│  │   completed  │      │ wallet.      │   │   failed     │ │
│  │ return.      │      │   topup_*    │   │              │ │
│  │   requested  │      │              │   │              │ │
│  └──────────────┘      └──────────────┘   └──────────────┘ │
│                                                             │
│  Queue: inventory       Queue: notifications               │
│  ┌──────────────┐      ┌──────────────┐                    │
│  │ stock.       │      │ notification.│                    │
│  │   reserved   │      │   sent       │                    │
│  │ stock.       │      │ banner.      │                    │
│  │   committed  │      │   impression │                    │
│  │ stock.       │      │              │                    │
│  │   low        │      │              │                    │
│  │ stock.       │      │              │                    │
│  │   out_of_    │      │              │                    │
│  │   stock      │      │              │                    │
│  └──────────────┘      └──────────────┘                    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## 9. Failure & Compensation Flows

### 9.1 Payment Failure After Stock Reservation
```
B08: stock.reserved ──► B05: payment fails
  │
  ├──► B08: stock.released (compensation)
  ├──► B04: order.cancelled
  └──► B11: notification.sent (payment failed)
```

### 9.2 Delivery Failure
```
B07: delivery.failed
  │
  ├──► B04: order.flagged_for_retry
  ├──► B07: assignment.retry (after 24h)
  └──► B11: notification.sent (delivery issue)
```

### 9.3 Refund Gateway Timeout
```
B05: refund.initiated ──► gateway timeout
  │
  ├──► B05: refund.pending (retry in 1h)
  ├──► B05: refund.failed (after 3 retries)
  ├──► B11: notification.sent (manual intervention needed)
  └──► B12: ticket.created (escalation)
```
