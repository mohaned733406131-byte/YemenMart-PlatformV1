# Layered Architecture — YemenMart

## 1. Layer Overview

YemenMart follows a strict five-layer architecture. Each layer has a single responsibility and depends only on the layer directly below it.

```
┌─────────────────────────────────────────────────────────────┐
│                   PRESENTATION LAYER                         │
│  User interfaces that render data and capture user input     │
├─────────────────────────────────────────────────────────────┤
│                       API LAYER                              │
│  HTTP routing, request validation, authentication            │
├─────────────────────────────────────────────────────────────┤
│                    SERVICE LAYER                             │
│  Business logic, orchestration, transaction management       │
├─────────────────────────────────────────────────────────────┤
│                    DOMAIN LAYER                              │
│  Entities, value objects, domain events, repo interfaces     │
├─────────────────────────────────────────────────────────────┤
│                 INFRASTRUCTURE LAYER                         │
│  Database, cache, queues, external APIs, file storage        │
└─────────────────────────────────────────────────────────────┘
```

## 2. Presentation Layer

**Responsibility**: Render UI and capture user interaction. No business logic.

### 2.1 Customer Storefront (Next.js 15)

```
src/
├── app/                          # Next.js App Router
│   ├── (shop)/
│   │   ├── page.tsx              # Homepage
│   │   ├── products/
│   │   │   ├── page.tsx          # Product listing
│   │   │   └── [slug]/page.tsx   # Product detail
│   │   ├── cart/page.tsx         # Shopping cart
│   │   └── checkout/page.tsx     # Checkout flow
│   ├── (account)/
│   │   ├── orders/page.tsx       # Order history
│   │   ├── profile/page.tsx      # User profile
│   │   └── addresses/page.tsx    # Address management
│   └── api/                      # API routes (BFF pattern)
├── components/
│   ├── ui/                       # Base UI components
│   ├── product/                  # Product-specific components
│   ├── cart/                     # Cart components
│   └── checkout/                 # Checkout components
├── hooks/                        # Custom React hooks
├── lib/                          # Utilities, API client
└── stores/                       # Zustand state stores
```

### 2.2 Vendor Portal (React + Vite)

```
src/
├── pages/
│   ├── Dashboard.tsx
│   ├── Products/
│   │   ├── ProductList.tsx
│   │   ├── ProductCreate.tsx
│   │   └── ProductEdit.tsx
│   ├── Orders/
│   │   ├── OrderList.tsx
│   │   └── OrderDetail.tsx
│   ├── Store/
│   │   └── StoreSettings.tsx
│   ├── Finance/
│   │   ├── Wallet.tsx
│   │   └── Payouts.tsx
│   └── Analytics/
│       └── Dashboard.tsx
├── components/
│   ├── ui/                       # shadcn/ui components
│   ├── forms/                    # Form components
│   └── charts/                   # Chart components
├── hooks/                        # Custom hooks
├── services/                     # API client functions
└── stores/                       # Zustand stores
```

### 2.3 Mobile (React Native)

```
src/
├── screens/
│   ├── Home/
│   ├── Product/
│   ├── Cart/
│   ├── Checkout/
│   ├── Orders/
│   └── Profile/
├── components/
│   ├── common/
│   └── product/
├── navigation/
│   └── AppNavigator.tsx
├── services/
└── stores/
```

### 2.4 Presentation Layer Rules

1. **Never** import from Service, Domain, or Infrastructure layers directly
2. All data fetching goes through the API client (`services/api.ts`)
3. State management uses Zustand stores that call API client functions
4. Forms use React Hook Form + Zod for client-side validation
5. All UI text goes through i18n (Arabic default, English secondary)

---

## 3. API Layer

**Responsibility**: HTTP routing, request/response handling, cross-cutting concerns.

```
src/api/
├── middleware/
│   ├── auth.ts               # JWT verification, role extraction
│   ├── rateLimiter.ts        # Rate limiting (Redis sliding window)
│   ├── validator.ts          # Zod schema request validation
│   ├── cors.ts               # CORS configuration
│   ├── errorHandler.ts       # Global error handler
│   ├── requestId.ts          # Request ID propagation
│   └── logger.ts             # Request/response logging
├── routes/
│   ├── v1/
│   │   ├── auth.routes.ts
│   │   ├── vendor.routes.ts
│   │   ├── product.routes.ts
│   │   ├── order.routes.ts
│   │   ├── payment.routes.ts
│   │   ├── wallet.routes.ts
│   │   ├── delivery.routes.ts
│   │   └── ...
│   └── index.ts              # Route aggregation
├── schemas/
│   ├── auth.schema.ts        # Request/response schemas
│   ├── product.schema.ts
│   └── ...
├── controllers/
│   ├── auth.controller.ts
│   ├── vendor.controller.ts
│   └── ...
└── app.ts                    # Express/Fastify app setup
```

### 3.1 Request Lifecycle

```
HTTP Request
    │
    ▼
┌─────────────┐
│   CORS      │
│   Handler   │
└─────┬───────┘
      │
      ▼
┌─────────────┐
│   Request   │
│   ID        │
└─────┬───────┘
      │
      ▼
┌─────────────┐
│   Rate      │
│   Limiter   │
└─────┬───────┘
      │
      ▼
┌─────────────┐
│   Auth      │
│   Middleware │
└─────┬───────┘
      │
      ▼
┌─────────────┐
│   Request   │
│   Validator │
└─────┬───────┘
      │
      ▼
┌─────────────┐
│  Controller │────▶ Service Layer
└─────┬───────┘
      │
      ▼
┌─────────────┐
│   Response  │
│   Serializer│
└─────┬───────┘
      │
      ▼
┌─────────────┐
│   Logger    │
│   (Pino)    │
└─────────────┘
```

### 3.2 API Layer Rules

1. Controllers are thin — they parse input, call services, and format output
2. No business logic in controllers or middleware
3. All request/response types are defined via Zod schemas
4. Errors are mapped to standard HTTP status codes via `errorHandler.ts`
5. Every endpoint has OpenAPI documentation via `@asteasolutions/zod-to-openapi`

---

## 4. Service Layer

**Responsibility**: Business logic, orchestration, transaction boundaries.

```
src/services/
├── auth/
│   ├── auth.service.ts           # Login, register, refresh tokens
│   ├── session.service.ts        # Session management
│   └── audit.service.ts          # Audit logging
├── vendor/
│   ├── vendor.service.ts         # Vendor CRUD
│   ├── store.service.ts          # Store management
│   └── kyc.service.ts            # KYC verification
├── catalog/
│   ├── product.service.ts        # Product management
│   ├── category.service.ts       # Category tree
│   └── offer.service.ts          # Promotional offers
├── order/
│   ├── order.service.ts          # Order lifecycle
│   ├── checkout.service.ts       # Checkout orchestration
│   └── return.service.ts         # Return/refund flow
├── payment/
│   ├── wallet.service.ts         # Vendor wallets
│   ├── escrow.service.ts         # Escrow management
│   └── payment.service.ts        # Payment processing
├── finance/
│   ├── commission.service.ts     # Commission calculation
│   ├── payout.service.ts         # Payout processing
│   └── invoice.service.ts        # Invoice generation
├── shipping/
│   ├── delivery.service.ts       # Delivery management
│   ├── zone.service.ts           # Shipping zones
│   └── assignment.service.ts     # Rider assignment
├── inventory/
│   ├── stock.service.ts          # Stock management
│   ├── reservation.service.ts    # Stock reservation
│   └── warehouse.service.ts      # Warehouse ops
├── storefront/
│   ├── storefront.service.ts     # Store rendering
│   ├── cart.service.ts           # Cart management
│   └── wishlist.service.ts       # Wishlist management
├── trust/
│   ├── review.service.ts         # Review system
│   ├── loyalty.service.ts        # Loyalty points
│   └── tier.service.ts           # Vendor tiers
├── content/
│   ├── cms.service.ts            # CMS management
│   └── notification.service.ts   # Notifications
├── support/
│   ├── ticket.service.ts         # Support tickets
│   ├── analytics.service.ts      # Analytics
│   └── serviceBooking.service.ts # Service bookings
└── pricing/
    ├── coupon.service.ts         # Coupon management
    └── discount.service.ts       # Discount rules
```

### 4.1 Service Pattern

```typescript
// Example: Order Service
export class OrderService {
  constructor(
    private orderRepo: OrderRepository,      // Interface from domain
    private stockService: StockService,      // Cross-module call
    private paymentService: PaymentService,  // Cross-module call
    private eventBus: EventBus,              // Domain events
  ) {}

  async createOrder(dto: CreateOrderDTO): Promise<Order> {
    // 1. Validate business rules (Domain layer)
    // 2. Reserve stock (cross-module)
    // 3. Process payment (cross-module)
    // 4. Persist order (Infrastructure)
    // 5. Emit domain event (BullMQ)
  }
}
```

### 4.2 Service Layer Rules

1. Services depend on Domain interfaces, not Infrastructure implementations
2. Transaction boundaries are defined at the service method level
3. Cross-module communication uses service interfaces (never repositories)
4. Domain events are emitted after transaction commit
5. DTOs are mapped to/from domain entities within the service

---

## 5. Domain Layer

**Responsibility**: Core business concepts, rules, and events. Zero external dependencies.

```
src/domain/
├── entities/
│   ├── User.ts
│   ├── Vendor.ts
│   ├── Product.ts
│   ├── Order.ts
│   ├── OrderItem.ts
│   ├── Payment.ts
│   ├── Wallet.ts
│   ├── Delivery.ts
│   ├── Review.ts
│   └── ...
├── value-objects/
│   ├── Money.ts
│   ├── Address.ts
│   ├── Email.ts
│   ├── PhoneNumber.ts
│   ├── SKU.ts
│   ├── Money.ts
│   └── ...
├── events/
│   ├── OrderPlaced.ts
│   ├── OrderConfirmed.ts
│   ├── PaymentCompleted.ts
│   ├── StockReserved.ts
│   ├── ReviewSubmitted.ts
│   └── ...
├── enums/
│   ├── OrderStatus.ts
│   ├── PaymentStatus.ts
│   ├── VendorTier.ts
│   └── ...
├── interfaces/
│   ├── repositories/
│   │   ├── OrderRepository.ts
│   │   ├── ProductRepository.ts
│   │   └── ...
│   └── services/
│       ├── EventBus.ts
│       ├── PaymentGateway.ts
│       └── ...
└── errors/
    ├── DomainError.ts
    ├── InsufficientStockError.ts
    ├── PaymentFailedError.ts
    └── ...
```

### 5.1 Entity Example

```typescript
export class Order {
  private constructor(
    public readonly id: OrderId,
    public readonly customerId: UserId,
    private status: OrderStatus,
    private items: OrderItem[],
    private total: Money,
    public readonly createdAt: Date,
  ) {}

  static create(dto: CreateOrderDTO): Order {
    // Factory method with business rule validation
    if (dto.items.length === 0) {
      throw new DomainError('Order must have at least one item');
    }
    // ...
  }

  confirm(): void {
    if (this.status !== OrderStatus.PENDING) {
      throw new DomainError('Only pending orders can be confirmed');
    }
    this.status = OrderStatus.CONFIRMED;
  }

  // Domain events emitted by service, not entity
}
```

### 5.2 Domain Layer Rules

1. **No imports** from any other layer (zero dependencies)
2. Entities contain business rules and state transitions
3. Value objects are immutable and compared by value
4. Repository interfaces are defined here; implementations live in Infrastructure
5. Domain events describe what happened, not what should happen

---

## 6. Infrastructure Layer

**Responsibility**: Technical implementations of domain interfaces, external integrations.

```
src/infrastructure/
├── database/
│   ├── prisma/
│   │   ├── schema.prisma
│   │   └── migrations/
│   ├── repositories/
│   │   ├── PrismaOrderRepository.ts
│   │   ├── PrismaProductRepository.ts
│   │   └── ...
│   └── client.ts                 # Prisma client singleton
├── cache/
│   ├── RedisCacheService.ts
│   └── CacheKeys.ts
├── queue/
│   ├── BullMQEventBus.ts
│   ├── workers/
│   │   ├── orderWorker.ts
│   │   ├── notificationWorker.ts
│   │   └── ...
│   └── queues.ts
├── search/
│   ├── ElasticsearchService.ts
│   └── indices/
│       ├── products.ts
│       └── ...
├── storage/
│   ├── MinIOService.ts
│   └── FileUploadService.ts
├── external/
│   ├── StripePaymentGateway.ts
│   ├── MoyasarPaymentGateway.ts
│   ├── TwilioSMSProvider.ts
│   └── SendGridEmailProvider.ts
├── messaging/
│   └── NotificationService.ts
└── config/
    ├── index.ts
    └── env.ts
```

### 6.1 Repository Pattern (Prisma)

```typescript
// Interface (Domain layer)
export interface OrderRepository {
  findById(id: OrderId): Promise<Order | null>;
  save(order: Order): Promise<void>;
  findByCustomer(customerId: UserId): Promise<Order[]>;
}

// Implementation (Infrastructure layer)
export class PrismaOrderRepository implements OrderRepository {
  constructor(private prisma: PrismaClient) {}

  async findById(id: OrderId): Promise<Order | null> {
    const record = await this.prisma.order.findUnique({
      where: { id: id.value },
      include: { items: true },
    });
    return record ? this.toDomain(record) : null;
  }

  private toDomain(record: any): Order {
    // Map Prisma record to domain entity
  }
}
```

### 6.2 Infrastructure Layer Rules

1. Implements interfaces defined in the Domain layer
2. Contains no business logic — only technical concerns
3. External API calls are wrapped in Anti-Corruption Layers
4. All infrastructure services are registered via dependency injection

---

## 7. Dependency Flow

```
Presentation ──▶ API ──▶ Service ──▶ Domain ◀── Infrastructure
                                          │
                                    (interfaces)
```

| Source | Target | Allowed |
|---|---|---|
| Presentation | API | Yes (HTTP calls) |
| API | Service | Yes (method calls) |
| API | Domain | Yes (DTOs, types) |
| Service | Domain | Yes (entities, events) |
| Service | Infrastructure | No (use interfaces) |
| Infrastructure | Domain | Yes (implements interfaces) |
| Domain | Anything | No (zero dependencies) |

---

## 8. Cross-Cutting Concerns

| Concern | Implementation | Layer |
|---|---|---|
| Logging | Pino | API, Service, Infrastructure |
| Authentication | JWT middleware | API |
| Authorization | Role/permission guards | API, Service |
| Validation | Zod schemas | API |
| Caching | Redis (read-through) | Infrastructure |
| Monitoring | Prometheus metrics | Infrastructure |
| Tracing | OpenTelemetry spans | All layers |
| Error Handling | Domain errors → HTTP errors | Service → API |
