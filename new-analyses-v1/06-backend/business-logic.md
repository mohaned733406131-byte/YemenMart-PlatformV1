# Business Logic Implementation

## Overview

Core business rules that govern YemenMart operations. Every rule is enforced at the service layer with unit tests. Database constraints serve as a safety net, not the primary enforcement mechanism.

---

## 1. Payment Processing

### Wallet-Only Payments

YemenMart accepts wallet payments exclusively. No external payment gateways at launch.

```typescript
// Wallet debit - atomic and idempotent
async debitWallet(userId: string, input: WalletDebitInput): Promise<WalletTransaction> {
  return this.db.$transaction(async (tx) => {
    // Check idempotency
    const existing = await tx.walletTransaction.findUnique({
      where: { idempotencyKey: input.idempotencyKey },
    });
    if (existing) return existing;

    // Lock wallet row for update
    const wallet = await tx.wallet.findFirst({
      where: { userId },
      select: { id: true, balance: true, version: true },
    });

    if (!wallet) throw new WalletNotFoundError(userId);

    const newBalance = wallet.balance - input.amount.value;
    if (newBalance < 0) {
      throw new InsufficientBalanceError(input.amount.value, wallet.balance);
    }

    // Optimistic concurrency check
    const updated = await tx.wallet.updateMany({
      where: { id: wallet.id, version: wallet.version },
      data: {
        balance: newBalance,
        version: { increment: 1 },
      },
    });

    if (updated.count === 0) {
      throw new ConcurrentModificationError('Wallet was modified by another request');
    }

    // Record transaction
    return tx.walletTransaction.create({
      data: {
        walletId: wallet.id,
        type: 'DEBIT',
        amount: input.amount.value,
        currency: input.amount.currency,
        orderId: input.orderId,
        idempotencyKey: input.idempotencyKey,
        description: input.description,
        balanceAfter: newBalance,
      },
    });
  });
}
```

### Idempotency

All payment operations require an idempotency key (UUIDv4). Duplicate requests return the original result without re-executing.

```typescript
// Idempotency middleware
async function withIdempotency<T>(
  key: string,
  fn: () => Promise<T>,
  ttl = 86400, // 24 hours
): Promise<T> {
  const cached = await redis.get(`idempotency:${key}`);
  if (cached) return JSON.parse(cached);

  const result = await fn();
  await redis.setex(`idempotency:${key}`, ttl, JSON.stringify(result));
  return result;
}
```

### Atomic Operations

Wallet balance changes always happen within a database transaction. The wallet row is locked with `SELECT ... FOR UPDATE` or optimistic versioning.

```typescript
// Atomic debit + transaction record
const [walletTx] = await this.db.$transaction([
  // Debit wallet
  this.db.wallet.update({
    where: { id: walletId },
    data: { balance: { decrement: amount } },
  }),
  // Record transaction
  this.db.walletTransaction.create({
    data: { walletId, type: 'DEBIT', amount, ... },
  }),
]);
```

---

## 2. Order Management

### Order States (17-State Machine)

```
┌─────────────────────────────────────────────────────────────────┐
│                         ORDER LIFECYCLE                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐                  │
│  │  PENDING │───▶│CONFIRMED │───▶│PROCESSING│                  │
│  └──────────┘    └──────────┘    └──────────┘                  │
│       │                              │                          │
│       ▼                              ▼                          │
│  ┌──────────┐                  ┌──────────┐                     │
│  │CANCELLED │                  │ SHIPPED  │                     │
│  └──────────┘                  └──────────┘                     │
│                                    │                             │
│                    ┌───────────────┼───────────────┐            │
│                    ▼               ▼               ▼            │
│              ┌──────────┐   ┌──────────┐   ┌──────────┐       │
│              │OUT_FOR_  │   │ DELIVERED│   │ RETURNED │       │
│              │DELIVERY  │   └──────────┘   └──────────┘       │
│              └──────────┘        │               │              │
│                    │             ▼               ▼              │
│                    │      ┌──────────┐   ┌──────────┐          │
│                    │      │COMPLETED │   │REFUNDING │          │
│                    │      └──────────┘   └──────────┘          │
│                    │                          │                  │
│                    ▼                          ▼                  │
│              ┌──────────┐              ┌──────────┐             │
│              │DELIVERY_ │              │ REFUNDED │             │
│              │FAILED    │              └──────────┘             │
│              └──────────┘                                       │
│                                                                 │
│  Additional states:                                             │
│  - ON_HOLD         (payment verification pending)               │
│  - RETURN_REQUESTED (customer initiated return)                 │
│  - RETURN_APPROVED  (seller/admin approved)                     │
│  - RETURN_REJECTED  (return denied)                             │
│  - DISPUTE_OPENED   (customer dispute)                          │
│  - DISPUTE_RESOLVED (dispute closed)                            │
└─────────────────────────────────────────────────────────────────┘
```

### State Transition Rules

```typescript
const ORDER_TRANSITIONS: Record<OrderStatus, OrderStatus[]> = {
  PENDING:           ['CONFIRMED', 'CANCELLED', 'ON_HOLD'],
  ON_HOLD:           ['CONFIRMED', 'CANCELLED'],
  CONFIRMED:         ['PROCESSING', 'CANCELLED'],
  PROCESSING:        ['SHIPPED', 'CANCELLED'],
  SHIPPED:           ['OUT_FOR_DELIVERY', 'RETURNED'],
  OUT_FOR_DELIVERY:  ['DELIVERED', 'DELIVERY_FAILED'],
  DELIVERED:         ['COMPLETED', 'RETURN_REQUESTED', 'DISPUTE_OPENED'],
  DELIVERY_FAILED:   ['SHIPPED', 'CANCELLED'],
  COMPLETED:         ['RETURN_REQUESTED'],
  RETURN_REQUESTED:  ['RETURN_APPROVED', 'RETURN_REJECTED'],
  RETURN_APPROVED:   ['RETURNED'],
  RETURN_REJECTED:   ['COMPLETED'],
  RETURNED:          ['REFUNDING'],
  REFUNDING:         ['REFUNDED'],
  REFUNDED:          [],
  CANCELLED:         [],
  DISPUTE_OPENED:    ['DISPUTE_RESOLVED'],
  DISPUTE_RESOLVED:  [],
};

function canTransition(from: OrderStatus, to: OrderStatus): boolean {
  return ORDER_TRANSITIONS[from]?.includes(to) ?? false;
}
```

### Master / Sub-Order Splitting

Each customer order (master) is split into sub-orders per seller. Each sub-order has its own delivery tracking, commission calculation, and status.

```typescript
// Master order
{
  id: "ord_master_001",
  userId: "user_001",
  status: "CONFIRMED",
  total: 15000,
  currency: "YER",
  subOrders: [
    {
      id: "ord_sub_001",
      storeId: "store_A",
      items: [...],
      subtotal: 8000,
      commission: 800,
      sellerPayout: 7200,
      status: "CONFIRMED",
    },
    {
      id: "ord_sub_002",
      storeId: "store_B",
      items: [...],
      subtotal: 7000,
      commission: 700,
      sellerPayout: 6300,
      status: "CONFIRMED",
    }
  ]
}
```

```typescript
async create(input: CreateOrderInput): Promise<MasterOrder> {
  return this.db.$transaction(async (tx) => {
    // Group items by seller
    const itemsBySeller = groupBy(input.items, 'storeId');

    // Create master order
    const masterOrder = await tx.order.create({
      data: {
        userId: input.userId,
        type: 'MASTER',
        total: input.total,
        currency: 'YER',
        status: 'PENDING',
        addressId: input.addressId,
      },
    });

    // Create sub-orders per seller
    const subOrders = [];
    for (const [storeId, items] of Object.entries(itemsBySeller)) {
      const subtotal = items.reduce((sum, i) => sum + i.price * i.quantity, 0);
      const commission = await this.commissionService.calculate(storeId, subtotal);
      const sellerPayout = subtotal - commission.totalFee;

      const subOrder = await tx.order.create({
        data: {
          userId: input.userId,
          type: 'SUB',
          parentId: masterOrder.id,
          storeId,
          items: { create: items.map(i => ({ ...i })) },
          subtotal,
          commission: commission.totalFee,
          sellerPayout,
          status: 'PENDING',
        },
      });
      subOrders.push(subOrder);
    }

    return { ...masterOrder, subOrders };
  });
}
```

---

## 3. Delivery Code Generation

### Flow

1. Seller ships → system generates 6-character code
2. Code hashed with bcrypt before storage
3. Customer provides code to delivery agent on delivery
4. Agent submits code → system verifies against hash
5. 3 verification attempts allowed; code expires after 24 hours

```typescript
// Generate delivery code
async generateDeliveryCode(orderId: string): Promise<DeliveryCodeResult> {
  const code = this.generateRandomCode(6); // e.g., "A7X9K2"
  const hash = await bcrypt.hash(code, 12);

  await this.db.deliveryCode.create({
    data: {
      orderId,
      codeHash: hash,
      attempts: 0,
      maxAttempts: 3,
      expiresAt: new Date(Date.now() + 24 * 60 * 60 * 1000), // 24h
      used: false,
    },
  });

  // Send code to customer via secure channel (NOT SMS in plaintext)
  await this.notificationService.send({
    userId: await this.getOrderUserId(orderId),
    type: 'DELIVERY_CODE',
    data: { orderId, code }, // Encrypted in transit
  });

  return { orderId, expiresIn: 86400 };
}

// Verify delivery code
async verifyDeliveryCode(orderId: string, code: string): Promise<boolean> {
  const record = await this.db.deliveryCode.findFirst({
    where: { orderId, used: false },
    orderBy: { createdAt: 'desc' },
  });

  if (!record) throw new DeliveryCodeNotFoundError(orderId);
  if (record.expiresAt < new Date()) throw new DeliveryCodeExpiredError();
  if (record.attempts >= record.maxAttempts) throw new DeliveryCodeMaxAttemptsError();

  // Increment attempts
  await this.db.deliveryCode.update({
    where: { id: record.id },
    data: { attempts: { increment: 1 } },
  });

  const valid = await bcrypt.compare(code, record.codeHash);
  if (!valid) return false;

  // Mark as used
  await this.db.deliveryCode.update({
    where: { id: record.id },
    data: { used: true, usedAt: new Date() },
  });

  // Update order status
  await this.orderService.updateOrderStatus(orderId, 'DELIVERED', {
    actor: 'DELIVERY_AGENT',
    deliveryCodeUsed: true,
  });

  return true;
}

private generateRandomCode(length: number): string {
  const chars = 'ABCDEFGHJKLMNPQRSTUVWXYZ23456789'; // No ambiguous chars (0/O, 1/I)
  return Array.from({ length }, () => chars[Math.floor(Math.random() * chars.length)]).join('');
}
```

---

## 4. Commission Calculation

### Three Calculation Models

```typescript
enum CommissionType {
  FIXED = 'FIXED',           // Flat percentage
  TIERED = 'TIERED',         // Volume-based tiers
  CATEGORY = 'CATEGORY',     // Per-category rates
}

// Commission tier structure
interface CommissionTier {
  minAmount: number;
  maxAmount: number | null;
  rate: number; // percentage, e.g., 10 = 10%
}

// Store commission config
interface StoreCommission {
  storeId: string;
  type: CommissionType;
  tiers: CommissionTier[];      // For TIERED type
  fixedRate: number;            // For FIXED type
  categoryRates: CategoryRate[]; // For CATEGORY type
}
```

```typescript
// Calculate commission for an order
async calculate(storeId: string, subtotal: Money): Promise<CommissionResult> {
  const config = await this.getStoreCommission(storeId);

  switch (config.type) {
    case CommissionType.FIXED:
      return this.calculateFixed(config.fixedRate, subtotal);

    case CommissionType.TIERED:
      return this.calculateTiered(config.tiers, subtotal);

    case CommissionType.CATEGORY:
      throw new Error('Use calculateCategory for category-based commissions');

    default:
      throw new Error(`Unknown commission type: ${config.type}`);
  }
}

// Fixed rate
private calculateFixed(rate: number, subtotal: Money): CommissionResult {
  const fee = subtotal.value * (rate / 100);
  return {
    totalFee: new Money(fee, subtotal.currency),
    rate,
    type: CommissionType.FIXED,
  };
}

// Tiered by volume (monthly sales)
private calculateTiered(tiers: CommissionTier[], subtotal: Money): CommissionResult {
  const monthlySales = await this.getMonthlySales(subtotal.currency);
  const cumulative = monthlySales + subtotal.value;

  let remaining = subtotal.value;
  let totalFee = 0;

  for (const tier of tiers) {
    if (remaining <= 0) break;
    if (cumulative < tier.minAmount) continue;

    const tierUpper = tier.maxAmount ?? Infinity;
    const tierMin = Math.max(tier.minAmount, monthlySales);
    const tierAvailable = Math.min(tierUpper, cumulative) - tierMin;
    const applicable = Math.min(remaining, Math.max(0, tierAvailable));

    totalFee += applicable * (tier.rate / 100);
    remaining -= applicable;
  }

  return {
    totalFee: new Money(totalFee, subtotal.currency),
    rate: totalFee / subtotal.value * 100,
    type: CommissionType.TIERED,
    monthlySales: cumulative,
  };
}

// Per-category rates
async calculateCategory(items: OrderItem[]): Promise<CommissionResult> {
  let totalFee = 0;

  for (const item of items) {
    const categoryRate = await this.getCategoryRate(item.categoryId);
    const itemCommission = item.price * item.quantity * (categoryRate / 100);
    totalFee += itemCommission;
  }

  return {
    totalFee: new Money(totalFee, items[0].price.currency),
    rate: 0, // Blended rate not meaningful for category
    type: CommissionType.CATEGORY,
  };
}
```

### Default Tier Configuration

| Tier  | Monthly Volume (YER) | Rate |
|-------|---------------------|------|
| Base  | 0 – 500,000         | 10%  |
| Growth| 500,001 – 2,000,000 | 8%   |
| Scale | 2,000,001+          | 6%   |

---

## 5. Coupon Validation

### Validation Rules

```typescript
interface Coupon {
  id: string;
  code: string;
  type: 'PERCENTAGE' | 'FIXED_AMOUNT' | 'FREE_SHIPPING';
  value: number;
  minOrderAmount: Money;
  maxDiscount: Money | null;
  usageLimit: number | null;
  usageCount: number;
  perUserLimit: number;
  stackable: boolean;
  validFrom: Date;
  validUntil: Date;
  applicableStoreIds: string[] | null;
  applicableCategoryIds: string[] | null;
}
```

```typescript
async validate(couponCode: string, items: OrderItem[], userId: string): Promise<CouponValidation> {
  const coupon = await this.db.coupon.findFirst({
    where: { code: couponCode.toUpperCase(), active: true },
  });

  if (!coupon) throw new CouponNotFoundError(couponCode);
  if (coupon.validFrom > new Date()) throw new CouponNotYetValidError();
  if (coupon.validUntil < new Date()) throw new CouponExpiredError();
  if (coupon.usageLimit && coupon.usageCount >= coupon.usageLimit) {
    throw new CouponUsageLimitReachedError();
  }

  // Per-user usage check
  const userUsage = await this.db.couponUsage.count({
    where: { couponId: coupon.id, userId },
  });
  if (userUsage >= coupon.perUserLimit) {
    throw new CouponPerUserLimitError();
  }

  // Minimum order amount
  const orderTotal = items.reduce((sum, i) => sum + i.price * i.quantity, 0);
  if (orderTotal < coupon.minOrderAmount.value) {
    throw new CouponMinOrderNotMetError(coupon.minOrderAmount.value, orderTotal);
  }

  // Store/category restrictions
  if (coupon.applicableStoreIds?.length) {
    const hasApplicable = items.some(i => coupon.applicableStoreIds.includes(i.storeId));
    if (!hasApplicable) throw new CouponNotApplicableError();
  }

  // Calculate discount
  let discount: Money;
  switch (coupon.type) {
    case 'PERCENTAGE':
      discount = new Money(orderTotal * (coupon.value / 100), 'YER');
      if (coupon.maxDiscount && discount.value > coupon.maxDiscount.value) {
        discount = coupon.maxDiscount;
      }
      break;
    case 'FIXED_AMOUNT':
      discount = new Money(Math.min(coupon.value, orderTotal), 'YER');
      break;
    case 'FREE_SHIPPING':
      discount = new Money(0, 'YER'); // Handled at checkout
      break;
  }

  return { coupon, discount, freeShipping: coupon.type === 'FREE_SHIPPING' };
}

// Record usage atomically
async recordUsage(couponId: string, userId: string, orderId: string): Promise<void> {
  await this.db.$transaction([
    this.db.couponUsage.create({
      data: { couponId, userId, orderId },
    }),
    this.db.coupon.update({
      where: { id: couponId },
      data: { usageCount: { increment: 1 } },
    }),
  ]);
}
```

### Stacking Rules

Only one coupon per order. Stacking is disabled at launch.

```typescript
// Future: stacking validation
function validateStacking(appliedCoupons: Coupon[]): boolean {
  const stackable = appliedCoupons.filter(c => c.stackable);
  const nonStackable = appliedCoupons.filter(c => !c.stackable);

  // Only one non-stackable coupon allowed
  if (nonStackable.length > 1) return false;

  // Stackable coupons cannot exceed 50% of order total
  const totalDiscount = stackable.reduce((sum, c) => sum + c.value, 0);
  if (totalDiscount > 50) return false;

  return true;
}
```

---

## 6. Loyalty Points

### Earning Rules

| Payment Method       | Points per YER |
|---------------------|---------------|
| Wallet (all types)  | 1 point       |
| Refunded order      | Points revoked|

### Tier System

| Tier    | Points Required | Benefits                          |
|---------|----------------|-----------------------------------|
| Bronze  | 0              | Base earning rate (1 pt/100 YER)  |
| Silver  | 5,000          | 1.2x earning, free shipping       |
| Gold    | 25,000         | 1.5x earning, priority support    |
| Platinum| 100,000        | 2x earning, exclusive deals       |

```typescript
// Award points for completed order
async awardPoints(userId: string, orderTotal: Money, tx: TransactionClient): Promise<void> {
  // Only wallet payments earn points
  const points = Math.floor(orderTotal.value / 100); // 1 pt per 100 YER
  if (points <= 0) return;

  const tier = await this.getUserTier(userId, tx);
  const multiplier = TIER_MULTIPLIERS[tier];
  const finalPoints = Math.floor(points * multiplier);

  await tx.loyaltyPoints.create({
    data: {
      userId,
      type: 'EARNED',
      points: finalPoints,
      source: 'ORDER_PURCHASE',
      expiresAt: new Date(Date.now() + 365 * 24 * 60 * 60 * 1000), // 1 year
    },
  });

  // Update lifetime points
  await tx.loyaltyAccount.upsert({
    where: { userId },
    create: { userId, totalPoints: finalPoints, lifetimePoints: finalPoints },
    update: {
      totalPoints: { increment: finalPoints },
      lifetimePoints: { increment: finalPoints },
    },
  });
}

// Redeem points at checkout
async calculateRedemption(userId: string, orderTotal: Money): Promise<Money> {
  const account = await this.db.loyaltyAccount.findUnique({ where: { userId } });
  if (!account || account.totalPoints <= 0) return Money.zero('YER');

  // Points to YER: 100 points = 1 YER
  const maxRedeemable = Math.min(account.totalPoints, Math.floor(orderTotal.value * 100));
  return new Money(Math.floor(maxRedeemable / 100), 'YER');
}

// Revoke points on return/refund
async revokePoints(userId: string, orderId: string, tx: TransactionClient): Promise<void> {
  const originalPoints = await tx.loyaltyPoints.findMany({
    where: { userId, source: 'ORDER_PURCHASE', orderId },
  });

  for (const entry of originalPoints) {
    await tx.loyaltyPoints.create({
      data: {
        userId,
        type: 'REVOKED',
        points: -entry.points,
        source: 'ORDER_RETURN',
        orderId,
      },
    });

    await tx.loyaltyAccount.update({
      where: { userId },
      data: { totalPoints: { decrement: Math.abs(entry.points) } },
    });
  }
}

const TIER_MULTIPLIERS: Record<string, number> = {
  BRONZE: 1,
  SILVER: 1.2,
  GOLD: 1.5,
  PLATINUM: 2,
};

async getUserTier(userId: string, tx: TransactionClient): Promise<string> {
  const account = await tx.loyaltyAccount.findUnique({
    where: { userId },
    select: { lifetimePoints: true },
  });

  const points = account?.lifetimePoints ?? 0;
  if (points >= 100_000) return 'PLATINUM';
  if (points >= 25_000) return 'GOLD';
  if (points >= 5_000) return 'SILVER';
  return 'BRONZE';
}
```
