# Service Architecture

## Overview

YemenMart backend follows a modular monolith architecture with 13 bounded-context services (B01–B13). Each service owns its domain logic, data access, and public interface. Services communicate through typed interfaces — no shared database tables across module boundaries.

---

## Module Services

| ID   | Service                 | Domain                              |
|------|-------------------------|-------------------------------------|
| B01  | AuthService             | OTP, JWT, sessions, password mgmt  |
| B02  | UserService             | Profiles, addresses, preferences    |
| B03  | ProductService          | Catalog, categories, variants       |
| B04  | StoreService            | Seller stores, branding, settings   |
| B05  | OrderService            | Cart, checkout, order lifecycle     |
| B06  | PaymentService          | Wallet, escrow, refunds             |
| B07  | DeliveryService         | Shipping zones, carriers, tracking  |
| B08  | NotificationService     | Multi-channel notifications         |
| B09  | CommissionService       | Fee calculation, tiered rates       |
| B10  | CouponService           | Promo codes, stacking rules         |
| B11  | LoyaltyService          | Points, tiers, rewards              |
| B12  | AnalyticsService        | Dashboards, reports, metrics        |
| B13  | AdminService            | RBAC, moderation, system config     |

---

## Service Interface Contracts

Every service exposes a public interface defined in a shared types package. No service imports another service's implementation — only its contract.

```typescript
// packages/contracts/src/order.service.ts
export interface IOrderService {
  createOrder(input: CreateOrderInput): Promise<Order>;
  getOrder(orderId: string, userId: string): Promise<OrderDetail>;
  cancelOrder(orderId: string, userId: string): Promise<Order>;
  requestReturn(orderId: string, input: ReturnInput): Promise<ReturnRequest>;
  getOrdersByUser(userId: string, pagination: PaginationInput): Promise<PaginatedOrders>;
  getOrdersByStore(storeId: string, filters: OrderFilters): Promise<PaginatedOrders>;
  updateOrderStatus(orderId: string, status: OrderStatus, actor: ActorInfo): Promise<Order>;
}
```

```typescript
// packages/contracts/src/payment.service.ts
export interface IPaymentService {
  debitWallet(userId: string, input: WalletDebitInput): Promise<WalletTransaction>;
  creditWallet(userId: string, input: WalletCreditInput): Promise<WalletTransaction>;
  holdEscrow(orderId: string, amount: Money, duration: Duration): Promise<EscrowHold>;
  releaseEscrow(orderId: string): Promise<EscrowRelease>;
  processRefund(orderId: string, input: RefundInput): Promise<RefundResult>;
  getBalance(userId: string): Promise<WalletBalance>;
}
```

```typescript
// packages/contracts/src/notification.service.ts
export interface INotificationService {
  send(input: NotificationInput): Promise<NotificationResult>;
  sendBatch(inputs: NotificationInput[]): Promise<NotificationResult[]>;
  getPreferences(userId: string): Promise<NotificationPreferences>;
  updatePreferences(userId: string, prefs: NotificationPreferences): Promise<void>;
}
```

---

## Dependency Injection

Services are wired through a lightweight DI container. Each module registers its implementation against its contract.

```typescript
// packages/di/src/container.ts
import { Container } from 'tsyringe';
import { IOrderService } from '@yemenmart/contracts';
import { OrderService } from '@yemenmart/order-module';
import { IPaymentService } from '@yemenmart/contracts';
import { PaymentService } from '@yemenmart/payment-module';

const container = container = new Container();

// Registration
container.register<IOrderService>('IOrderService', { useClass: OrderService });
container.register<IPaymentService>('IPaymentService', { useClass: PaymentService });
container.register<INotificationService>('INotificationService', { useClass: NotificationService });
container.register<IAuthService>('IAuthService', { useClass: AuthService });
container.register<IProductService>('IProductService', { useClass: ProductService });
container.register<IStoreService>('IStoreService', { useClass: StoreService });
container.register<IDeliveryService>('IDeliveryService', { useClass: DeliveryService });
container.register<ICommissionService>('ICommissionService', { useClass: CommissionService });
container.register<ICouponService>('ICouponService', { useClass: CouponService });
container.register<ILoyaltyService>('ILoyaltyService', { useClass: LoyaltyService });
container.register<IUserService>('IUserService', { useClass: UserService });
container.register<IAnalyticsService>('IAnalyticsService', { useClass: AnalyticsService });
container.register<IAdminService>('IAdminService', { useClass: AdminService });
```

```typescript
// Usage in controller
@injectable()
export class OrderController {
  constructor(
    @inject('IOrderService') private orderService: IOrderService,
    @inject('IPaymentService') private paymentService: IPaymentService,
  ) {}
}
```

---

## Service Composition

Complex operations compose multiple services through orchestration handlers rather than direct cross-service calls.

```typescript
// packages/core/src/use-cases/place-order.ts
@injectable()
export class PlaceOrderUseCase {
  constructor(
    @inject('IOrderService') private orders: IOrderService,
    @inject('IPaymentService') private payments: IPaymentService,
    @inject('ICouponService') private coupons: ICouponService,
    @inject('ILoyaltyService') private loyalty: ILoyaltyService,
    @inject('IProductService') private products: IProductService,
  ) {}

  async execute(input: PlaceOrderInput): Promise<PlaceOrderResult> {
    return this.db.transaction(async (tx) => {
      // 1. Validate stock availability
      await this.products.holdStock(input.items, tx);

      // 2. Validate and apply coupon
      const couponDiscount = input.couponCode
        ? await this.coupons.validateAndApply(input.couponCode, input.items, tx)
        : Money.zero('YER');

      // 3. Calculate totals
      const subtotal = Money.sum(input.items.map(i => i.price.multiply(i.quantity)));
      const commission = await this.commissions.calculate(subtotal, input.items, tx);
      const deliveryFee = await this.delivery.calculateFee(input.address, input.items, tx);
      const loyaltyDiscount = input.usePoints
        ? await this.loyalty.calculateRedemption(input.userId, subtotal, tx)
        : Money.zero('YER');
      const total = subtotal.add(deliveryFee).subtract(couponDiscount).subtract(loyaltyDiscount);

      // 4. Debit wallet
      await this.payments.debitWallet(input.userId, {
        amount: total,
        idempotencyKey: input.idempotencyKey,
        orderId: tempOrderId,
      }, tx);

      // 5. Create order
      const order = await this.orders.create(input, total, tx);

      // 6. Hold escrow
      await this.payments.holdEscrow(order.id, total, { days: 7 }, tx);

      // 7. Award loyalty points
      await this.loyalty.awardPoints(input.userId, total, tx);

      return { order, totalCharged: total };
    });
  }
}
```

---

## Cross-Cutting Concerns

### Logging

```typescript
// Middleware-style logging for every service method
@injectable()
export class LoggingInterceptor implements IInterceptor {
  constructor(private logger: ILogger) {}

  intercept(context: ServiceContext, next: NextFn): Promise<any> {
    const start = Date.now();
    const { serviceName, methodName, args } = context;

    this.logger.info(`${serviceName}.${methodName} called`, {
      args: sanitizeSensitive(args),
      traceId: context.traceId,
    });

    try {
      const result = next();
      const duration = Date.now() - start;
      this.logger.info(`${serviceName}.${methodName} completed`, { duration });
      return result;
    } catch (error) {
      const duration = Date.now() - start;
      this.logger.error(`${serviceName}.${methodName} failed`, {
        duration,
        error: error.message,
        traceId: context.traceId,
      });
      throw error;
    }
  }
}
```

### Validation

```typescript
// Zod schemas at service boundaries
const CreateOrderSchema = z.object({
  items: z.array(z.object({
    productId: z.string().uuid(),
    variantId: z.string().uuid().optional(),
    quantity: z.number().int().min(1).max(100),
    price: z.number().positive(),
  })).min(1).max(50),
  addressId: z.string().uuid(),
  couponCode: z.string().optional(),
  usePoints: z.boolean().default(false),
  idempotencyKey: z.string().uuid(),
});

export function validate<T>(schema: z.ZodSchema<T>, data: unknown): T {
  const result = schema.safeParse(data);
  if (!result.success) {
    throw new ValidationError(result.error.flatten().fieldErrors);
  }
  return result.data;
}
```

### Authentication Guard

```typescript
// Applied to controller methods
function AuthGuard(): MethodDecorator {
  return (target, propertyKey, descriptor) => {
    const original = descriptor.value;
    descriptor.value = async function (...args: any[]) {
      const req = args[ctxIndex];
      const token = req.headers.authorization?.replace('Bearer ', '');
      if (!token) throw new UnauthorizedError('Missing token');
      const user = await authService.verifyToken(token);
      args[ctxIndex].user = user;
      return original.apply(this, args);
    };
  };
}
```

### Rate Limiting

```typescript
// Per-user and per-endpoint rate limits
const rateLimits = {
  'order.create': { window: '1m', max: 10 },
  'payment.debit': { window: '1m', max: 5 },
  'auth.login': { window: '15m', max: 5 },
  'notification.send': { window: '1h', max: 50 },
};
```

### Transaction Management

```typescript
// Database transaction wrapper
export class DatabaseTransaction {
  async run<T>(fn: (tx: PrismaTransactionClient) => Promise<T>): Promise<T> {
    return this.prisma.$transaction(async (tx) => {
      try {
        const result = await fn(tx);
        return result;
      } catch (error) {
        // Transaction auto-rolls back on throw
        throw error;
      }
    }, { maxWait: 5000, timeout: 30000 });
  }
}
```

### Error Handling

```typescript
// Domain-specific error hierarchy
export class DomainError extends Error {
  constructor(message: string, public code: string, public httpStatus: number) {
    super(message);
  }
}

export class InsufficientBalanceError extends DomainError {
  constructor(required: number, available: number) {
    super(`Insufficient balance: need ${required}, have ${available}`, 'INSUFFICIENT_BALANCE', 400);
  }
}

export class StockHoldExpiredError extends DomainError {
  constructor(productId: string) {
    super(`Stock hold expired for product ${productId}`, 'STOCK_HOLD_EXPIRED', 400);
  }
}

export class DuplicateOrderError extends DomainError {
  constructor(idempotencyKey: string) {
    super(`Order already exists for key ${idempotencyKey}`, 'DUPLICATE_ORDER', 409);
  }
}
```

---

## Service Communication Patterns

| Pattern           | When to Use                          | Example                           |
|-------------------|--------------------------------------|-----------------------------------|
| Direct call       | Synchronous, same request            | Order → Payment debit             |
| Event publish     | Async, decoupled                     | Order placed → Notification       |
| Shared DB view    | Read-only cross-module queries       | Analytics reads order snapshots   |
| Outbox pattern    | Reliable event delivery              | Order events in outbox table      |

### Event Bus

```typescript
// In-process event bus for module monolith
@injectable()
export class EventBus {
  private handlers = new Map<string, Function[]>();

  on(event: string, handler: Function): void {
    const list = this.handlers.get(event) || [];
    list.push(handler);
    this.handlers.set(event, list);
  }

  async emit(event: string, payload: any): Promise<void> {
    const handlers = this.handlers.get(event) || [];
    await Promise.allSettled(handlers.map(h => h(payload)));
  }
}

// Usage
eventBus.on('order.placed', async (event) => {
  await notificationService.send({ userId: event.userId, type: 'ORDER_CONFIRMED', data: event });
});

eventBus.on('order.delivered', async (event) => {
  await commissionService.processCommission(event.orderId);
  await loyaltyService.awardPoints(event.userId, event.total);
});
```

---

## Package Structure

```
packages/
├── contracts/          # Shared interfaces and types
│   └── src/
│       ├── index.service.ts
│       └── events.ts
├── di/                 # Dependency injection container
│   └── src/
│       └── container.ts
├── core/               # Shared use-cases and orchestration
│   └── src/
│       └── use-cases/
├── prisma/             # Schema and migrations
│   └── prisma/
│       └── schema.prisma
├── shared/             # Utilities, errors, helpers
│   └── src/
│       ├── errors/
│       ├── logger/
│       └── validation/
├── auth-module/        # B01
├── user-module/        # B02
├── product-module/     # B03
├── store-module/       # B04
├── order-module/       # B05
├── payment-module/     # B06
├── delivery-module/    # B07
├── notification-module/# B08
├── commission-module/  # B09
├── coupon-module/      # B10
├── loyalty-module/     # B11
├── analytics-module/   # B12
└── admin-module/       # B13
```
