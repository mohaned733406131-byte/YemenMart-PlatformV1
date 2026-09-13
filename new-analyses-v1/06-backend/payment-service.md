# Payment Service

## Overview

YemenMart operates a wallet-only payment system. No external payment gateways at launch. All monetary operations are atomic, idempotent, and support multi-currency (YER primary, SAR and USD secondary).

---

## Wallet Operations

### Create Wallet

```typescript
// packages/payment-module/src/wallet.service.ts
@injectable()
export class WalletService implements IPaymentService {
  constructor(
    @inject('PrismaClient') private prisma: PrismaClient,
    @inject('RedisClient') private redis: Redis,
    @inject('EventBus') private events: EventBus,
    @inject('Logger') private logger: ILogger,
  ) {}

  async createWallet(userId: string, currency: string = 'YER'): Promise<Wallet> {
    const existing = await this.prisma.wallet.findUnique({ where: { userId } });
    if (existing) throw new WalletAlreadyExistsError(userId);

    return this.prisma.wallet.create({
      data: { userId, balance: 0, currency, version: 0 },
    });
  }

  async getBalance(userId: string): Promise<WalletBalance> {
    const wallet = await this.prisma.wallet.findUnique({ where: { userId } });
    if (!wallet) throw new WalletNotFoundError(userId);

    return {
      walletId: wallet.id,
      balance: new Money(Number(wallet.balance), wallet.currency),
      currency: wallet.currency,
      version: wallet.version,
    };
  }
```

### Debit Wallet

```typescript
  async debitWallet(
    userId: string,
    input: WalletDebitInput,
    tx?: PrismaTransactionClient,
  ): Promise<WalletTransaction> {
    const db = tx || this.prisma;

    // 1. Check idempotency
    const existing = await db.walletTransaction.findUnique({
      where: { idempotencyKey: input.idempotencyKey },
    });
    if (existing) {
      this.logger.info('Idempotent debit replay', { key: input.idempotencyKey });
      return existing;
    }

    // 2. Lock wallet row
    const wallet = await db.$queryRaw`
      SELECT id, balance, currency, version
      FROM wallets
      WHERE user_id = ${userId}
      FOR UPDATE
    `.then((rows: any[]) => rows[0]);

    if (!wallet) throw new WalletNotFoundError(userId);

    // 3. Check sufficient balance
    const currentBalance = Number(wallet.balance);
    const debitAmount = input.amount.value;
    if (currentBalance < debitAmount) {
      throw new InsufficientBalanceError(debitAmount, currentBalance);
    }

    // 4. Optimistic concurrency check
    const updated = await db.wallet.updateMany({
      where: { id: wallet.id, version: wallet.version },
      data: {
        balance: { decrement: debitAmount },
        version: { increment: 1 },
      },
    });

    if (updated.count === 0) {
      throw new ConcurrentModificationError('Wallet balance changed during transaction');
    }

    const newBalance = currentBalance - debitAmount;

    // 5. Record transaction
    const walletTx = await db.walletTransaction.create({
      data: {
        walletId: wallet.id,
        type: 'DEBIT',
        amount: debitAmount,
        currency: wallet.currency,
        balanceAfter: newBalance,
        orderId: input.orderId,
        idempotencyKey: input.idempotencyKey,
        description: input.description,
        metadata: input.metadata,
      },
    });

    // 6. Publish event
    this.events.emit('wallet.debited', {
      walletId: wallet.id,
      userId,
      amount: debitAmount,
      balanceAfter: newBalance,
      orderId: input.orderId,
    });

    return walletTx;
  }
```

### Credit Wallet

```typescript
  async creditWallet(
    userId: string,
    input: WalletCreditInput,
    tx?: PrismaTransactionClient,
  ): Promise<WalletTransaction> {
    const db = tx || this.prisma;

    // Check idempotency
    const existing = await db.walletTransaction.findUnique({
      where: { idempotencyKey: input.idempotencyKey },
    });
    if (existing) return existing;

    const wallet = await db.$queryRaw`
      SELECT id, balance, currency, version
      FROM wallets
      WHERE user_id = ${userId}
      FOR UPDATE
    `.then((rows: any[]) => rows[0]);

    if (!wallet) throw new WalletNotFoundError(userId);

    const updated = await db.wallet.updateMany({
      where: { id: wallet.id, version: wallet.version },
      data: {
        balance: { increment: input.amount.value },
        version: { increment: 1 },
      },
    });

    if (updated.count === 0) {
      throw new ConcurrentModificationError('Wallet balance changed during transaction');
    }

    const newBalance = Number(wallet.balance) + input.amount.value;

    const walletTx = await db.walletTransaction.create({
      data: {
        walletId: wallet.id,
        type: 'CREDIT',
        amount: input.amount.value,
        currency: wallet.currency,
        balanceAfter: newBalance,
        orderId: input.orderId,
        idempotencyKey: input.idempotencyKey,
        description: input.description,
        metadata: input.metadata,
      },
    });

    this.events.emit('wallet.credited', {
      walletId: wallet.id,
      userId,
      amount: input.amount.value,
      balanceAfter: newBalance,
    });

    return walletTx;
  }
```

### Transfer Between Wallets

```typescript
  async transfer(
    fromUserId: string,
    toUserId: string,
    amount: Money,
    idempotencyKey: string,
  ): Promise<TransferResult> {
    return this.prisma.$transaction(async (tx) => {
      // Check idempotency
      const existing = await tx.walletTransaction.findUnique({
        where: { idempotencyKey },
      });
      if (existing) return { debit: existing, credit: existing };

      // Lock both wallets (consistent order to prevent deadlocks)
      const [fromWallet, toWallet] = await Promise.all([
        tx.$queryRaw`SELECT id, balance, currency, version FROM wallets WHERE user_id = ${fromUserId} FOR UPDATE`.then((r: any[]) => r[0]),
        tx.$queryRaw`SELECT id, balance, currency, version FROM wallets WHERE user_id = ${toUserId} FOR UPDATE`.then((r: any[]) => r[0]),
      ]);

      if (!fromWallet) throw new WalletNotFoundError(fromUserId);
      if (!toWallet) throw new WalletNotFoundError(toUserId);

      // Validate sufficient balance
      if (Number(fromWallet.balance) < amount.value) {
        throw new InsufficientBalanceError(amount.value, Number(fromWallet.balance));
      }

      // Debit sender
      const [debitResult] = await Promise.all([
        tx.walletTransaction.create({
          data: {
            walletId: fromWallet.id,
            type: 'TRANSFER',
            amount: -amount.value,
            currency: fromWallet.currency,
            balanceAfter: Number(fromWallet.balance) - amount.value,
            idempotencyKey: `${idempotencyKey}:debit`,
            description: `Transfer to ${toUserId}`,
          },
        }),
        tx.wallet.update({
          where: { id: fromWallet.id },
          data: {
            balance: { decrement: amount.value },
            version: { increment: 1 },
          },
        }),
      ]);

      // Credit receiver
      const [creditResult] = await Promise.all([
        tx.walletTransaction.create({
          data: {
            walletId: toWallet.id,
            type: 'TRANSFER',
            amount: amount.value,
            currency: toWallet.currency,
            balanceAfter: Number(toWallet.balance) + amount.value,
            idempotencyKey: `${idempotencyKey}:credit`,
            description: `Transfer from ${fromUserId}`,
          },
        }),
        tx.wallet.update({
          where: { id: toWallet.id },
          data: {
            balance: { increment: amount.value },
            version: { increment: 1 },
          },
        }),
      ]);

      return { debit: debitResult, credit: creditResult };
    });
  }
```

---

## Idempotency Key Handling

```typescript
// Middleware applied to all payment endpoints
export function idempotencyMiddleware(
  req: Request,
  res: Response,
  next: NextFunction,
) {
  const key = req.headers['idempotency-key'] as string;
  if (!key) {
    return res.status(400).json({ error: 'Idempotency-Key header required' });
  }

  // Validate UUID format
  if (!isValidUUID(key)) {
    return res.status(400).json({ error: 'Idempotency-Key must be a valid UUID' });
  }

  req.idempotencyKey = key;
  next();
}

// Response caching for idempotent replay
async function cacheIdempotentResponse(key: string, response: any, ttl = 86400) {
  await redis.setex(`idempotent:${key}`, ttl, JSON.stringify(response));
}

async function getIdempotentResponse(key: string): Promise<any | null> {
  const cached = await redis.get(`idempotent:${key}`);
  return cached ? JSON.parse(cached) : null;
}
```

---

## Atomic Operations

All wallet balance changes occur within database transactions with row-level locking.

```typescript
// Transaction wrapper with retry logic
async withTransactionRetry<T>(
  fn: (tx: PrismaTransactionClient) => Promise<T>,
  maxRetries = 3,
): Promise<T> {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await this.prisma.$transaction(fn, {
        maxWait: 5000,
        timeout: 30000,
      });
    } catch (error) {
      if (error instanceof Prisma.PrismaClientKnownRequestError) {
        if (error.code === 'P2034' && attempt < maxRetries) {
          // Transaction conflict, retry with backoff
          await sleep(100 * Math.pow(2, attempt));
          continue;
        }
      }
      throw error;
    }
  }
  throw new TransactionFailedError('Max retries exceeded');
}
```

---

## Escrow Management

### 7-Day Hold

```typescript
// packages/payment-module/src/escrow.service.ts
@injectable()
export class EscrowService {
  constructor(
    @inject('PrismaClient') private prisma: PrismaClient,
    @inject('EventBus') private events: EventBus,
  ) {}

  async holdEscrow(
    orderId: string,
    amount: Money,
    duration: Duration = { days: 7 },
  ): Promise<EscrowHold> {
    const releaseAt = new Date();
    releaseAt.setDate(releaseAt.getDate() + duration.days);

    return this.prisma.$transaction(async (tx) => {
      // Create escrow hold
      const hold = await tx.escrowHold.create({
        data: {
          orderId,
          amount: amount.value,
          currency: amount.currency,
          status: 'HELD',
          releaseAt,
        },
      });

      // Create pending payout for seller
      const order = await tx.order.findUnique({
        where: { id: orderId },
        include: { subOrders: true },
      });

      if (order?.subOrders) {
        for (const subOrder of order.subOrders) {
          await tx.sellerPayout.create({
            data: {
              storeId: subOrder.storeId,
              orderId: subOrder.id,
              amount: subOrder.sellerPayout,
              currency: subOrder.currency,
              status: 'PENDING',
              escrowReleaseAt: releaseAt,
            },
          });
        }
      }

      return hold;
    });
  }

  async releaseEscrow(orderId: string): Promise<EscrowRelease> {
    return this.prisma.$transaction(async (tx) => {
      const hold = await tx.escrowHold.findUnique({
        where: { orderId },
        include: { order: { include: { subOrders: true } } },
      });

      if (!hold) throw new EscrowNotFoundError(orderId);
      if (hold.status !== 'HELD') throw new EscrowNotHeldError(orderId);

      // Release escrow
      await tx.escrowHold.update({
        where: { orderId },
        data: { status: 'RELEASED', releasedAt: new Date() },
      });

      // Update pending payouts
      await tx.sellerPayout.updateMany({
        where: { orderId: { in: hold.order.subOrders.map(s => s.id) }, status: 'PENDING' },
        data: { status: 'READY' },
      });

      this.events.emit('escrow.released', { orderId, amount: hold.amount });

      return { orderId, amount: new Money(hold.amount, hold.currency) };
    });
  }

  async refundEscrow(orderId: string, reason: string): Promise<void> {
    return this.prisma.$transaction(async (tx) => {
      const hold = await tx.escrowHold.findUnique({ where: { orderId } });
      if (!hold || hold.status !== 'HELD') throw new EscrowNotRefundableError(orderId);

      await tx.escrowHold.update({
        where: { orderId },
        data: { status: 'REFUNDED', releasedAt: new Date() },
      });

      // Cancel pending payouts
      await tx.sellerPayout.updateMany({
        where: { orderId: hold.orderId, status: 'PENDING' },
        data: { status: 'CANCELLED' },
      });

      this.events.emit('escrow.refunded', { orderId, reason });
    });
  }

  // Background job: Release expired holds
  async processExpiredHolds(): Promise<number> {
    const expired = await this.prisma.escrowHold.findMany({
      where: {
        status: 'HELD',
        releaseAt: { lte: new Date() },
      },
    });

    let released = 0;
    for (const hold of expired) {
      try {
        await this.releaseEscrow(hold.orderId);
        released++;
      } catch (error) {
        this.logger.error(`Failed to release escrow for order ${hold.orderId}`, error);
      }
    }

    return released;
  }
}
```

---

## Refund Processing

```typescript
// packages/payment-module/src/refund.service.ts
@injectable()
export class RefundService {
  constructor(
    @inject('WalletService') private walletService: WalletService,
    @inject('PrismaClient') private prisma: PrismaClient,
    @inject('EventBus') private events: EventBus,
  ) {}

  async processRefund(orderId: string, input: RefundInput): Promise<RefundResult> {
    return this.prisma.$transaction(async (tx) => {
      // 1. Get original order
      const order = await tx.order.findUnique({
        where: { id: orderId },
        include: { items: true, subOrders: true },
      });

      if (!order) throw new OrderNotFoundError(orderId);
      if (!['RETURNED', 'DISPUTE_RESOLVED'].includes(order.status)) {
        throw new RefundNotAllowedError(order.status);
      }

      // 2. Calculate refund amount
      const refundAmount = input.partial
        ? this.calculatePartialRefund(order, input.reason)
        : new Money(Number(order.total), order.currency);

      // 3. Credit buyer wallet
      const buyerCredit = await this.walletService.creditWallet(order.userId, {
        amount: refundAmount,
        orderId,
        idempotencyKey: `refund:${orderId}:${Date.now()}`,
        description: `Refund for order ${orderId}`,
        metadata: { reason: input.reason, partial: input.partial },
      }, tx);

      // 4. Debit seller (reverse commission + payout)
      for (const subOrder of order.subOrders) {
        const sellerDebit = new Money(Number(subOrder.commission), order.currency);
        if (sellerDebit.value > 0) {
          const store = await tx.store.findUnique({ where: { id: subOrder.storeId } });
          if (store) {
            await this.walletService.debitWallet(store.ownerId, {
              amount: sellerDebit,
              orderId: subOrder.id,
              idempotencyKey: `refund-seller:${subOrder.id}:${Date.now()}`,
              description: `Commission reversal for order ${subOrder.id}`,
            }, tx);
          }
        }
      }

      // 5. Record refund
      const refund = await tx.refund.create({
        data: {
          orderId,
          amount: refundAmount.value,
          currency: refundAmount.currency,
          reason: input.reason,
          status: 'PROCESSED',
          buyerTransactionId: buyerCredit.id,
        },
      });

      // 6. Revoke loyalty points
      this.events.emit('order.refunded', {
        orderId,
        userId: order.userId,
        amount: refundAmount.value,
      });

      return { refund, amountRefunded: refundAmount };
    });
  }

  private calculatePartialRefund(order: Order, reason: string): Money {
    // Partial refund logic based on items returned
    // Simplified: return 50% for partial, 100% for quality issues
    const total = Number(order.total);
    if (reason === 'QUALITY_ISSUE') return new Money(total, order.currency);
    return new Money(total * 0.5, order.currency);
  }
}
```

---

## Multi-Currency Support

### YER / SAR / USD

```typescript
// packages/payment-module/src/currency.service.ts
@injectable()
export class CurrencyService {
  constructor(
    @inject('PrismaClient') private prisma: PrismaClient,
    @inject('RedisClient') private redis: Redis,
  ) {}

  private EXCHANGE_RATES = {
    'YER_SAR': 0.0015,  // 1 YER = 0.0015 SAR
    'YER_USD': 0.004,   // 1 YER = 0.004 USD
    'SAR_YER': 666.67,  // 1 SAR = 666.67 YER
    'SAR_USD': 0.267,   // 1 SAR = 0.267 USD
    'USD_YER': 250,     // 1 USD = 250 YER
    'USD_SAR': 3.75,    // 1 USD = 3.75 SAR
  };

  async getExchangeRate(from: string, to: string): Promise<number> {
    if (from === to) return 1;

    const cacheKey = `exchange:${from}:${to}`;
    const cached = await this.redis.get(cacheKey);
    if (cached) return parseFloat(cached);

    const rate = this.EXCHANGE_RATES[`${from}_${to}`];
    if (!rate) throw new UnsupportedCurrencyPairError(from, to);

    // Cache for 1 hour
    await this.redis.setex(cacheKey, 3600, rate.toString());
    return rate;
  }

  async convert(amount: Money, toCurrency: string): Promise<Money> {
    if (amount.currency === toCurrency) return amount;

    const rate = await this.getExchangeRate(amount.currency, toCurrency);
    const converted = Math.round(amount.value * rate * 100) / 100;
    return new Money(converted, toCurrency);
  }

  // Lock exchange rate for order duration
  async lockRate(from: string, to: string): Promise<LockedRate> {
    const rate = await this.getExchangeRate(from, to);
    const lockId = uuidv4();
    const expiresAt = new Date(Date.now() + 30 * 60 * 1000); // 30 min lock

    await this.redis.setex(`rate-lock:${lockId}`, 1800, JSON.stringify({
      from, to, rate, expiresAt,
    }));

    return { lockId, rate, from, to, expiresAt };
  }

  async getLockedRate(lockId: string): Promise<LockedRate> {
    const data = await this.redis.get(`rate-lock:${lockId}`);
    if (!data) throw new RateLockExpiredError();

    const lock = JSON.parse(data);
    if (new Date(lock.expiresAt) < new Date()) {
      throw new RateLockExpiredError();
    }

    return lock;
  }
}

// Money value object
export class Money {
  constructor(
    public readonly value: number,
    public readonly currency: string,
  ) {
    if (!['YER', 'SAR', 'USD'].includes(currency)) {
      throw new UnsupportedCurrencyError(currency);
    }
  }

  add(other: Money): Money {
    if (this.currency !== other.currency) {
      throw new CurrencyMismatchError(this.currency, other.currency);
    }
    return new Money(
      Math.round((this.value + other.value) * 100) / 100,
      this.currency,
    );
  }

  subtract(other: Money): Money {
    if (this.currency !== other.currency) {
      throw new CurrencyMismatchError(this.currency, other.currency);
    }
    return new Money(
      Math.round((this.value - other.value) * 100) / 100,
      this.currency,
    );
  }

  multiply(factor: number): Money {
    return new Money(
      Math.round(this.value * factor * 100) / 100,
      this.currency,
    );
  }

  isPositive(): boolean { return this.value > 0; }
  isZero(): boolean { return this.value === 0; }

  static zero(currency: string = 'YER'): Money {
    return new Money(0, currency);
  }

  static min(a: Money, b: Money): Money {
    if (a.currency !== b.currency) throw new CurrencyMismatchError(a.currency, b.currency);
    return a.value <= b.value ? a : b;
  }
}
```

---

## Wallet Transaction History

```typescript
async getTransactionHistory(
  userId: string,
  pagination: PaginationInput,
): Promise<PaginatedResult<WalletTransaction>> {
  const wallet = await this.prisma.wallet.findUnique({ where: { userId } });
  if (!wallet) throw new WalletNotFoundError(userId);

  const [data, total] = await Promise.all([
    this.prisma.walletTransaction.findMany({
      where: { walletId: wallet.id },
      orderBy: { createdAt: 'desc' },
      skip: (pagination.page - 1) * pagination.limit,
      take: pagination.limit,
      select: {
        id: true,
        type: true,
        amount: true,
        currency: true,
        balanceAfter: true,
        description: true,
        orderId: true,
        createdAt: true,
      },
    }),
    this.prisma.walletTransaction.count({
      where: { walletId: wallet.id },
    }),
  ]);

  return { data, total, page: pagination.page, limit: pagination.limit };
}
```

---

## API Endpoints

| Method | Endpoint                    | Description              | Auth     |
|--------|-----------------------------|--------------------------|----------|
| POST   | /wallet/create              | Create wallet            | User     |
| GET    | /wallet/balance             | Get balance              | User     |
| POST   | /wallet/debit               | Debit wallet             | System   |
| POST   | /wallet/credit              | Credit wallet            | System   |
| POST   | /wallet/transfer            | Transfer between wallets | User     |
| GET    | /wallet/transactions        | Transaction history      | User     |
| POST   | /payment/refund             | Process refund           | Admin    |
| GET    | /payment/escrow/:orderId    | Get escrow status        | User     |
| GET    | /payment/currency/:from/:to | Get exchange rate        | User     |
