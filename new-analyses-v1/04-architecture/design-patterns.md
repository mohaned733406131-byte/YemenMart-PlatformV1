# Design Patterns — YemenMart

## 1. Pattern Summary

| # | Pattern | Category | Primary Use |
|---|---|---|---|
| 1 | Repository Pattern | Data Access | Abstract database operations behind interfaces |
| 2 | Unit of Work | Transaction Management | Group multiple operations into single transactions |
| 3 | Domain Events | Integration | Cross-module communication without direct coupling |
| 4 | CQRS | Data Architecture | Separate read and write models for performance |
| 5 | Saga | Distributed Transactions | Coordinate multi-step business processes |
| 6 | Circuit Breaker | Resilience | Prevent cascading failures in external calls |
| 7 | Anti-Corruption Layer | Integration | Isolate external system models from domain |
| 8 | Strategy | Behavioral | Swap payment/shipping algorithms at runtime |
| 9 | Observer | Behavioral | Decouple event producers from consumers |
| 10 | Factory | Creational | Encapsulate complex object creation logic |

---

## 2. Repository Pattern

**Category:** Data Access

**Purpose:** Decouple domain entities from persistence mechanics. Every module's domain defines repository interfaces; Infrastructure provides Prisma implementations.

### Implementation

```typescript
// domain/interfaces/repositories/OrderRepository.ts
export interface OrderRepository {
  findById(id: OrderId): Promise<Order | null>;
  findByCustomer(customerId: UserId, pagination: Pagination): Promise<PaginatedResult<Order>>;
  save(order: Order): Promise<void>;
  saveMany(orders: Order[]): Promise<void>;
  countByStatus(status: OrderStatus): Promise<number>;
}

// domain/interfaces/repositories/ProductRepository.ts
export interface ProductRepository {
  findById(id: ProductId): Promise<Product | null>;
  findByVendor(vendorId: VendorId): Promise<Product[]>;
  findBySku(sku: SKU): Promise<Product | null>;
  save(product: Product): Promise<void>;
  delete(id: ProductId): Promise<void>;
  search(query: string, filters: ProductFilters): Promise<PaginatedResult<Product>>;
}
```

```typescript
// infrastructure/database/repositories/PrismaOrderRepository.ts
export class PrismaOrderRepository implements OrderRepository {
  constructor(private prisma: PrismaClient) {}

  async findById(id: OrderId): Promise<Order | null> {
    const record = await this.prisma.order.findUnique({
      where: { id: id.value },
      include: {
        items: true,
        customer: { select: { id: true, name: true, email: true } },
      },
    });
    return record ? OrderMapper.toDomain(record) : null;
  }

  async save(order: Order): Promise<void> {
    const data = OrderMapper.toPersistence(order);
    await this.prisma.order.upsert({
      where: { id: order.id.value },
      create: data,
      update: data,
    });
  }
}
```

### Where Applied

| Module | Repositories |
|---|---|
| B01 Auth | `UserRepository`, `SessionRepository`, `AuditRepository` |
| B02 Vendor | `VendorRepository`, `StoreRepository`, `KYCRepository` |
| B03 Catalog | `ProductRepository`, `CategoryRepository`, `OfferRepository` |
| B04 Order | `OrderRepository`, `ReturnRepository` |
| B05 Payment | `PaymentRepository`, `WalletRepository`, `EscrowRepository` |
| B06 Finance | `CommissionRepository`, `PayoutRepository`, `InvoiceRepository` |
| B07 Shipping | `ShipmentRepository`, `RiderRepository`, `ZoneRepository` |
| B08 Inventory | `StockRepository`, `ReservationRepository`, `WarehouseRepository` |
| B09 Storefront | `CartRepository`, `WishlistRepository` |
| B10 Trust | `ReviewRepository`, `LoyaltyRepository`, `TierRepository` |
| B11 Content | `PageRepository`, `NotificationRepository` |
| B12 Support | `TicketRepository`, `BookingRepository`, `AnalyticsRepository` |
| B13 Pricing | `CouponRepository`, `DiscountRepository` |

---

## 3. Unit of Work

**Category:** Transaction Management

**Purpose:** Ensure multiple repository operations commit atomically. Prevents partial writes in complex business operations.

### Implementation

```typescript
// infrastructure/database/UnitOfWork.ts
export interface UnitOfWork {
  execute<T>(fn: () => Promise<T>): Promise<T>;
  getRepository<T>(symbol: symbol): T;
}

export class PrismaUnitOfWork implements UnitOfWork {
  constructor(private prisma: PrismaClient) {}

  async execute<T>(fn: () => Promise<T>): Promise<T> {
    return this.prisma.$transaction(async (tx) => {
      // Set transaction-scoped repository instances
      this.setTransaction(tx);
      const result = await fn();
      return result;
    });
  }

  getRepository<T>(symbol: symbol): T {
    const repo = this.repositories.get(symbol);
    if (!repo) throw new Error(`Repository not registered: ${symbol.toString()}`);
    return repo as T;
  }
}
```

### Usage Example

```typescript
// services/order/checkout.service.ts
export class CheckoutService {
  constructor(private uow: UnitOfWork) {}

  async completeCheckout(dto: CompleteCheckoutDTO): Promise<Order> {
    return this.uow.execute(async () => {
      const orderRepo = this.uow.getRepository<OrderRepository>(ORDER_REPO);
      const stockRepo = this.uow.getRepository<StockRepository>(STOCK_REPO);
      const paymentRepo = this.uow.getRepository<PaymentRepository>(PAYMENT_REPO);

      // All three operations commit atomically
      const order = await orderRepo.save(OrderMapper.toDomain(dto));
      await stockRepo.reserve(dto.items);
      await paymentRepo.create(PaymentMapper.fromOrder(order));

      return order;
    });
  }
}
```

### Where Applied

| Use Case | Operations Wrapped |
|---|---|
| Order placement | Order create + Stock reserve + Payment initiate |
| Order cancellation | Order cancel + Stock release + Payment refund |
| Return processing | Return approve + Stock restore + Refund initiate |
| Vendor payout | Payout create + Wallet debit + Invoice generate |
| Product publish | Product update + Search reindex + Cache invalidate |

---

## 4. Domain Events

**Category:** Integration / Communication

**Purpose:** Decouple modules by emitting events when state changes. Consumers react independently without the producer knowing about them.

### Event Bus Interface

```typescript
// domain/interfaces/services/EventBus.ts
export interface DomainEvent {
  readonly eventId: string;
  readonly eventType: string;
  readonly aggregateId: string;
  readonly timestamp: Date;
  readonly version: number;
  readonly payload: Record<string, unknown>;
}

export interface EventBus {
  publish(event: DomainEvent): Promise<void>;
  publishMany(events: DomainEvent[]): Promise<void>;
}
```

### Event Definitions

```typescript
// domain/events/order/OrderPlaced.ts
export class OrderPlaced implements DomainEvent {
  readonly eventType = 'order.placed';

  constructor(
    public readonly eventId: string,
    public readonly aggregateId: string,
    public readonly timestamp: Date,
    public readonly version: number,
    public readonly payload: {
      orderId: string;
      customerId: string;
      vendorId: string;
      items: Array<{ productId: string; quantity: number; price: number }>;
      totalAmount: number;
      currency: string;
    },
  ) {}
}
```

### BullMQ Implementation

```typescript
// infrastructure/queue/BullMQEventBus.ts
export class BullMQEventBus implements EventBus {
  private queues: Map<string, Queue> = new Map();

  constructor(private redisConnection: RedisConnection) {}

  async publish(event: DomainEvent): Promise<void> {
    const queue = this.getQueue(event.eventType);
    await queue.add(event.eventType, event, {
      jobId: event.eventId,
      attempts: 3,
      backoff: { type: 'exponential', delay: 1000 },
      removeOnComplete: { age: 86400 },
    });
  }

  private getQueue(eventType: string): Queue {
    if (!this.queues.has(eventType)) {
      this.queues.set(eventType, new Queue(eventType, {
        connection: this.redisConnection,
      }));
    }
    return this.queues.get(eventType)!;
  }
}
```

### Event Map

| Event | Producer | Consumers |
|---|---|---|
| `order.placed` | B04 Order | B08 Inventory, B11 Notification |
| `order.confirmed` | B04 Order | B07 Shipping, B11 Notification |
| `order.completed` | B04 Order | B06 Finance, B10 Loyalty |
| `payment.processed` | B05 Payment | B04 Order, B11 Notification |
| `payment.failed` | B05 Payment | B04 Order, B11 Notification |
| `shipment.delivered` | B07 Shipping | B04 Order, B10 Trust |
| `stock.low` | B08 Inventory | B11 Notification, B02 Vendor |
| `vendor.verified` | B02 Vendor | B01 Auth, B11 Notification |
| `review.submitted` | B10 Trust | B11 Notification, B02 Vendor |
| `payout.processed` | B06 Finance | B05 Payment, B11 Notification |

---

## 5. CQRS (Command Query Responsibility Segregation)

**Category:** Data Architecture

**Purpose:** Separate read and write models for high-throughput modules (Orders, Products, Analytics). Write side uses normalized relational data; read side uses denormalized projections optimized for queries.

### Architecture

```
┌─────────────────────────────────────────────────────┐
│                    CQRS Pattern                      │
│                                                      │
│  Commands (Write)          Queries (Read)            │
│  ┌──────────────┐         ┌──────────────┐          │
│  │  CreateOrder │         │  GetOrder    │          │
│  │  UpdateOrder │         │  ListOrders  │          │
│  │  CancelOrder │         │  SearchOrders│          │
│  └──────┬───────┘         └──────┬───────┘          │
│         │                        │                    │
│         ▼                        ▼                    │
│  ┌──────────────┐         ┌──────────────┐          │
│  │  PostgreSQL  │────────▶│ Elasticsearch│          │
│  │  (Write DB)  │  Sync   │  (Read DB)   │          │
│  └──────────────┘         └──────────────┘          │
│                                                      │
└─────────────────────────────────────────────────────┘
```

### Implementation

```typescript
// Command Side
export class OrderCommandService {
  constructor(
    private orderRepo: OrderRepository,
    private eventBus: EventBus,
  ) {}

  async placeOrder(dto: PlaceOrderCommand): Promise<Order> {
    const order = Order.create(dto);
    await this.orderRepo.save(order);
    await this.eventBus.publish(new OrderPlaced(/* ... */));
    return order;
  }
}

// Query Side
export class OrderQueryService {
  constructor(
    private searchClient: ElasticsearchClient,
  ) {}

  async getOrder(orderId: string): Promise<OrderView> {
    return this.searchClient.get<OrderView>('orders', orderId);
  }

  async searchOrders(filters: OrderFilters): Promise<PaginatedResult<OrderView>> {
    return this.searchClient.search<OrderView>('orders', {
      bool: {
        must: this.buildFilters(filters),
      },
    }, { from: filters.offset, size: filters.limit });
  }
}
```

### Read Models

| Module | Write Model (PostgreSQL) | Read Model (Elasticsearch) |
|---|---|---|
| B03 Catalog | `products` table | `products` index (faceted search) |
| B04 Order | `orders` + `order_items` tables | `orders` index (admin search) |
| B09 Storefront | `products` + `stores` tables | `storefront` index (customer search) |
| B12 Analytics | Event store | Materialized aggregations |

---

## 6. Saga Pattern

**Category:** Distributed Transactions

**Purpose:** Coordinate multi-step business processes that span multiple modules, with compensating actions for failure recovery.

### Order Placement Saga

```
┌──────────────────────────────────────────────────────┐
│                ORDER PLACEMENT SAGA                    │
│                                                       │
│  Step 1: Reserve Stock                               │
│  ┌─────────┐     ┌─────────┐                         │
│  │ B08     │────▶│ Success │                         │
│  │ Reserve │     └────┬────┘                         │
│  └─────────┘          │                              │
│                       ▼                              │
│  Step 2: Process Payment                             │
│  ┌─────────┐     ┌─────────┐                         │
│  │ B05     │────▶│ Success │                         │
│  │ Pay     │     └────┬────┘                         │
│  └─────────┘          │                              │
│                       ▼                              │
│  Step 3: Confirm Order                               │
│  ┌─────────┐     ┌─────────┐                         │
│  │ B04     │────▶│ Success │                         │
│  │ Confirm │     └─────────┘                         │
│  └─────────┘                                         │
│                                                       │
│  Compensating Actions:                               │
│  • Payment fails → Release stock (Step 1 compensate) │
│  • Confirm fails → Refund payment + Release stock    │
└──────────────────────────────────────────────────────┘
```

### Implementation

```typescript
// services/order/sagas/OrderPlacementSaga.ts
export class OrderPlacementSaga {
  constructor(
    private stockService: StockService,
    private paymentService: PaymentService,
    private orderService: OrderService,
    private eventBus: EventBus,
  ) {}

  async execute(command: PlaceOrderSagaCommand): Promise<Order> {
    let reservationId: string | null = null;
    let paymentId: string | null = null;

    try {
      // Step 1: Reserve stock
      reservationId = await this.stockService.reserve({
        items: command.items,
        orderId: command.orderId,
      });

      // Step 2: Process payment
      paymentId = await this.paymentService.processPayment({
        customerId: command.customerId,
        amount: command.totalAmount,
        currency: command.currency,
        orderId: command.orderId,
      });

      // Step 3: Confirm order
      const order = await this.orderService.confirmOrder({
        orderId: command.orderId,
        reservationId,
        paymentId,
      });

      return order;
    } catch (error) {
      // Compensating actions
      if (paymentId) {
        await this.paymentService.refundPayment(paymentId);
      }
      if (reservationId) {
        await this.stockService.releaseReservation(reservationId);
      }
      throw error;
    }
  }
}
```

---

## 7. Circuit Breaker

**Category:** Resilience

**Purpose:** Prevent cascading failures when external services (payment gateways, SMS providers) are unavailable.

### Implementation

```typescript
// infrastructure/external/CircuitBreaker.ts
export class CircuitBreaker {
  private failures = 0;
  private state: 'closed' | 'open' | 'half-open' = 'closed';
  private lastFailureTime = 0;

  constructor(
    private options: {
      threshold: number;        // Failures before opening
      timeout: number;          // Time before half-open (ms)
      monitoringPeriod: number; // Reset failure count (ms)
    },
  ) {}

  async execute<T>(fn: () => Promise<T>): Promise<T> {
    if (this.state === 'open') {
      if (Date.now() - this.lastFailureTime > this.options.timeout) {
        this.state = 'half-open';
      } else {
        throw new CircuitBreakerOpenError('Circuit breaker is open');
      }
    }

    try {
      const result = await fn();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }

  private onSuccess(): void {
    this.failures = 0;
    this.state = 'closed';
  }

  private onFailure(): void {
    this.failures++;
    this.lastFailureTime = Date.now();
    if (this.failures >= this.options.threshold) {
      this.state = 'open';
    }
  }
}
```

### Where Applied

| External Service | Threshold | Timeout |
|---|---|---|
| Stripe API | 5 failures | 30s |
| Moyasar API | 5 failures | 30s |
| Twilio SMS | 3 failures | 60s |
| SendGrid Email | 3 failures | 60s |
| Elasticsearch | 5 failures | 10s |
| MinIO/S3 | 3 failures | 15s |

---

## 8. Anti-Corruption Layer

**Category:** Integration

**Purpose:** Translate external system models into domain models, preventing external changes from corrupting the domain.

### Implementation

```typescript
// infrastructure/external/StripePaymentGateway.ts
export class StripePaymentGateway implements PaymentGateway {
  private acl: AntiCorruptionLayer<Stripe.PaymentIntent, PaymentResult>;

  constructor(private stripe: Stripe, private logger: Logger) {
    this.acl = new AntiCorruptionLayer({
      toExternal: (domain) => this.toStripeIntent(domain),
      fromExternal: (external) => this.fromStripeIntent(external),
      onError: (error, direction) => {
        this.logger.error({ error, direction }, 'Stripe ACL translation error');
        throw new PaymentGatewayError('Payment processing failed');
      },
    });
  }

  async processPayment(command: ProcessPaymentCommand): Promise<PaymentResult> {
    const stripeIntent = this.acl.toExternal(command);
    const result = await this.stripe.paymentIntents.create(stripeIntent);
    return this.acl.fromExternal(result);
  }

  private toStripeIntent(command: ProcessPaymentCommand): Stripe.PaymentIntentCreateParams {
    return {
      amount: command.amount * 100, // Stripe uses cents
      currency: command.currency.toLowerCase(),
      metadata: { orderId: command.orderId },
    };
  }

  private fromStripeIntent(intent: Stripe.PaymentIntent): PaymentResult {
    return {
      transactionId: intent.id,
      status: this.mapStatus(intent.status),
      amount: intent.amount / 100,
      currency: intent.currency.toUpperCase(),
    };
  }
}
```

---

## 9. Strategy Pattern

**Category:** Behavioral

**Purpose:** Swap algorithms at runtime — payment providers, shipping calculators, discount strategies.

### Payment Strategy

```typescript
// domain/interfaces/services/PaymentStrategy.ts
export interface PaymentStrategy {
  process(command: ProcessPaymentCommand): Promise<PaymentResult>;
  refund(command: RefundPaymentCommand): Promise<RefundResult>;
  verify(command: VerifyPaymentCommand): Promise<VerificationResult>;
}

// infrastructure/external/strategies/StripeStrategy.ts
export class StripeStrategy implements PaymentStrategy { /* ... */ }

// infrastructure/external/strategies/MoyasarStrategy.ts
export class MoyasarStrategy implements PaymentStrategy { /* ... */ }

// infrastructure/external/strategies/CODStrategy.ts
export class CODStrategy implements PaymentStrategy { /* ... */ }
```

### Shipping Calculator Strategy

```typescript
export interface ShippingCalculator {
  calculate(command: CalculateShippingCommand): Promise<ShippingQuote>;
}

export class StandardShippingCalculator implements ShippingCalculator { /* ... */ }
export class ExpressShippingCalculator implements ShippingCalculator { /* ... */ }
export class FreeShippingCalculator implements ShippingCalculator { /* ... */ }
```

### Discount Strategy

```typescript
export interface DiscountStrategy {
  calculate(command: CalculateDiscountCommand): Promise<DiscountResult>;
  isApplicable(command: CheckDiscountCommand): Promise<boolean>;
}

export class PercentageDiscount implements DiscountStrategy { /* ... */ }
export class FixedAmountDiscount implements DiscountStrategy { /* ... */ }
export class BuyOneGetOneDiscount implements DiscountStrategy { /* ... */ }
export class TieredDiscount implements DiscountStrategy { /* ... */ }
```

---

## 10. Observer Pattern

**Category:** Behavioral

**Purpose:** Decouple event producers from consumers. Used extensively via domain events and BullMQ workers.

### Implementation

```typescript
// Notification Observer
export class NotificationObserver {
  constructor(
    private notificationService: NotificationService,
    private templateEngine: TemplateEngine,
  ) {}

  @OnEvent('order.placed')
  async onOrderPlaced(event: OrderPlaced): Promise<void> {
    const template = await this.templateEngine.render('order_confirmation', {
      orderId: event.payload.orderId,
      customerName: event.payload.customerName,
      items: event.payload.items,
      total: event.payload.totalAmount,
    });

    await this.notificationService.send({
      userId: event.payload.customerId,
      channel: 'email',
      subject: 'Order Confirmed',
      body: template,
    });
  }

  @OnEvent('payment.failed')
  async onPaymentFailed(event: PaymentFailed): Promise<void> {
    await this.notificationService.send({
      userId: event.payload.customerId,
      channel: 'push',
      title: 'Payment Failed',
      body: 'Your payment could not be processed. Please try again.',
    });
  }
}
```

### Observer Registry

```typescript
// infrastructure/observers/ObserverRegistry.ts
export class ObserverRegistry {
  private observers: Map<string, Function[]> = new Map();

  register(eventType: string, handler: Function): void {
    const handlers = this.observers.get(eventType) || [];
    handlers.push(handler);
    this.observers.set(eventType, handlers);
  }

  async notify(eventType: string, event: DomainEvent): Promise<void> {
    const handlers = this.observers.get(eventType) || [];
    await Promise.allSettled(handlers.map((h) => h(event)));
  }
}
```

---

## 11. Factory Pattern

**Category:** Creational

**Purpose:** Encapsulate complex object creation with business rule validation.

### Order Factory

```typescript
// domain/factories/OrderFactory.ts
export class OrderFactory {
  static create(command: CreateOrderCommand): Order {
    // Validate minimum items
    if (command.items.length === 0) {
      throw new DomainError('Order must contain at least one item');
    }

    // Validate all items have positive quantity
    const hasInvalidQuantity = command.items.some((item) => item.quantity <= 0);
    if (hasInvalidQuantity) {
      throw new DomainError('All items must have positive quantity');
    }

    // Calculate total
    const total = command.items.reduce(
      (sum, item) => sum + item.price * item.quantity,
      0,
    );

    // Generate order number
    const orderNumber = OrderNumber.generate();

    return new Order(
      OrderId.generate(),
      command.customerId,
      orderNumber,
      OrderStatus.PENDING,
      command.items.map((item) =>
        OrderItem.create({
          productId: item.productId,
          productName: item.productName,
          sku: item.sku,
          quantity: item.quantity,
          unitPrice: Money.create(item.price, item.currency),
        }),
      ),
      Money.create(total, command.currency),
      command.shippingAddress,
      new Date(),
    );
  }
}
```

### Product Factory

```typescript
export class ProductFactory {
  static create(command: CreateProductCommand): Product {
    const slug = SlugGenerator.fromName(command.name);
    const sku = SKUGenerator.generate(command.vendorId, command.categoryId);

    return new Product(
      ProductId.generate(),
      command.vendorId,
      command.name,
      slug,
      sku,
      command.description,
      command.categoryId,
      command.price,
      command.images.map((img) => ProductImage.create(img)),
      command.variants?.map((v) => ProductVariant.create(v)) || [],
      ProductStatus.DRAFT,
      new Date(),
    );
  }
}
```

---

## 12. Pattern Selection Guide

| Scenario | Primary Pattern | Supporting Patterns |
|---|---|---|
| Database abstraction | Repository | Unit of Work |
| Cross-module notification | Domain Events | Observer, Circuit Breaker |
| High-read performance | CQRS | Elasticsearch |
| Multi-step transactions | Saga | Domain Events, Compensation |
| Payment processing | Strategy | Anti-Corruption Layer, Circuit Breaker |
| Complex object creation | Factory | Domain Events |
| External API integration | Anti-Corruption Layer | Circuit Breaker, Strategy |
| Discount calculations | Strategy | Factory |
| Audit logging | Observer | Domain Events |
