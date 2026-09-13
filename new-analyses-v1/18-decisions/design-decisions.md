# Design Pattern and Approach Decisions — YemenMart

**Document ID:** YM-DES-001
**Version:** 1.0
**Status:** Active
**Date:** 2026-09-13
**Language:** English

---

## Table of Contents

1. [Design Pattern Selection](#1-design-pattern-selection)
2. [Data Access Pattern](#2-data-access-pattern)
3. [Transaction Management](#3-transaction-management)
4. [Cross-Module Communication](#4-cross-module-communication)
5. [API Design Approach](#5-api-design-approach)
6. [Error Handling Strategy](#6-error-handling-strategy)
7. [Caching Strategy](#7-caching-strategy)
8. [Search Design](#8-search-design)
9. [File Upload Design](#9-file-upload-design)
10. [Notification Design](#10-notification-design)
11. [Multi-Currency Design](#11-multi-currency-design)
12. [Internationalization Design](#12-internationalization-design)
13. [Mobile-First Design](#13-mobile-first-design)
14. [Security Design](#14-security-design)

---

## 1. Design Pattern Selection

### Pattern Selection Matrix

| Pattern | Category | YemenMart Application | Rationale |
|---------|----------|----------------------|-----------|
| Repository | Data Access | All 13 modules | Decouple domain from persistence |
| Unit of Work | Transaction | Complex operations | Atomic multi-entity updates |
| Domain Events | Integration | Cross-module sync | Decoupled communication |
| CQRS | Architecture | Product search, storefront | Separate read/write models |
| Saga | Distributed Tx | Order workflow | Multi-step compensation |
| Circuit Breaker | Resilience | External APIs | Prevent cascading failure |
| Anti-Corruption Layer | Integration | Third-party adapters | Isolate external models |
| Strategy | Behavioral | Payment, shipping | Runtime algorithm swap |
| Observer | Behavioral | Event subscriptions | Decouple event handling |
| Factory | Creational | Entity creation | Complex object construction |

### Pattern Application by Module

| Module | Primary Patterns | Secondary Patterns |
|--------|-----------------|-------------------|
| B01 Auth | Repository, Factory | Strategy (OTP provider) |
| B02 Vendor | Repository, Saga | Circuit Breaker (KYC) |
| B03 Catalog | Repository, CQRS | Observer (search sync) |
| B04 Order | Saga, State Machine | Domain Events, UoW |
| B05 Payment | Repository, Strategy | Circuit Breaker (wallets) |
| B06 Finance | Repository, Saga | UoW (settlements) |
| B07 Shipping | Repository, Strategy | ACL (delivery providers) |
| B08 Inventory | Repository, UoW | Domain Events |
| B09 Storefront | Repository, CQRS | Observer (events) |
| B10 Trust | Repository, Observer | Domain Events |
| B11 Content | Repository | — |
| B12 Support | Repository, Saga | Domain Events |
| B13 Pricing | Repository, Strategy | Observer (discount calc) |

---

## 2. Data Access Pattern

### 2.1 Repository Implementation

```typescript
// Domain Interface
export interface ProductRepository {
  findById(id: ProductId): Promise<Product | null>;
  findByVendor(vendorId: VendorId): Promise<Product[]>;
  findBySku(sku: SKU): Promise<Product | null>;
  save(product: Product): Promise<void>;
  delete(id: ProductId): Promise<void>;
  search(query: string, filters: ProductFilters): Promise<PaginatedResult<Product>>;
}

// Infrastructure Implementation
export class PrismaProductRepository implements ProductRepository {
  constructor(private prisma: PrismaClient) {}

  async findById(id: ProductId): Promise<Product | null> {
    const record = await this.prisma.product.findUnique({
      where: { id: id.value },
      include: { vendor: true, category: true, images: true },
    });
    return record ? ProductMapper.toDomain(record) : null;
  }
}
```

### 2.2 Mapper Pattern

```typescript
export class ProductMapper {
  static toDomain(record: ProductRecord): Product {
    return new Product(
      new ProductId(record.id),
      new VendorId(record.vendorId),
      ProductName.create(record.nameAr, record.nameEn),
      Money.create(record.price, record.currency),
      record.status as ProductStatus,
      new Date(record.createdAt),
    );
  }

  static toPersistence(product: Product): ProductRecord {
    return {
      id: product.id.value,
      vendorId: product.vendorId.value,
      nameAr: product.name.arabic,
      nameEn: product.name.english,
      price: product.price.amount,
      currency: product.price.currency,
      status: product.status,
      createdAt: product.createdAt,
    };
  }
}
```

### Decision

**Repository + Mapper** pattern for all modules. Domain models are pure TypeScript classes; persistence models are Prisma-generated types. Mappers handle translation.

---

## 3. Transaction Management

### 3.1 Unit of Work Pattern

```typescript
export class UnitOfWork {
  constructor(private prisma: PrismaClient) {}

  async execute<T>(workFn: () => Promise<T>): Promise<T> {
    return this.prisma.$transaction(async (tx) => {
      // All repositories within workFn use the same transaction
      return workFn();
    });
  }
}
```

### 3.2 Transaction Scope Decision

| Operation | Transaction Scope | Rationale |
|-----------|------------------|-----------|
| Order creation | Single transaction (order + items) | Atomic order creation |
| Payment + escrow | Single transaction | Payment must be atomic |
| Multi-vendor split | Per-vendor sub-transactions | Independent vendor fulfillment |
| Inventory reservation | Per-item reservation | Prevent over-reservation |
| Wallet operations | Single transaction | Financial integrity |

### Decision

**Explicit Unit of Work** with Prisma `$transaction`. No implicit global transactions. Each service method that needs atomicity explicitly uses the UoW.

---

## 4. Cross-Module Communication

### 4.1 Event-Driven Architecture

```typescript
// Event Publisher
export class OrderService {
  constructor(
    private eventBus: EventBus,
    private orderRepo: OrderRepository,
  ) {}

  async createOrder(command: CreateOrderCommand): Promise<Order> {
    const order = Order.create(command);
    await this.orderRepo.save(order);

    // Publish event for other modules
    await this.eventBus.publish(new OrderCreatedEvent({
      orderId: order.id,
      customerId: order.customerId,
      totalAmount: order.totalAmount,
      items: order.items.map(i => ({
        productId: i.productId,
        quantity: i.quantity,
        vendorId: i.vendorId,
      })),
    }));

    return order;
  }
}

// Event Subscriber
export class InventoryEventHandler {
  constructor(private inventoryService: InventoryService) {}

  @SubscribeTo('order.created')
  async handleOrderCreated(event: OrderCreatedEvent): Promise<void> {
    for (const item of event.items) {
      await this.inventoryService.reserveStock(item.productId, item.quantity);
    }
  }
}
```

### 4.2 Event Schema Versioning

```typescript
export interface DomainEvent<T = unknown> {
  eventId: string;
  eventType: string;
  eventVersion: string;  // SemVer: "1.0.0"
  aggregateId: string;
  timestamp: Date;
  metadata: {
    correlationId: string;
    causationId: string;
    userId?: string;
  };
  payload: T;
}
```

### Decision

**Async domain events via BullMQ** for all cross-module communication. No synchronous module-to-module calls. Event schema versioning follows SemVer. Dead letter queue for failed event processing.

---

## 5. API Design Approach

### 5.1 REST API Standards

```typescript
// Standard endpoint structure
router.get('/v1/products',           validate(paginationSchema),  listProducts);
router.get('/v1/products/:id',       validate(idSchema),         getProduct);
router.post('/v1/products',          validate(createSchema),     createProduct);
router.put('/v1/products/:id',       validate(updateSchema),     updateProduct);
router.delete('/v1/products/:id',    validate(idSchema),         deleteProduct);
```

### 5.2 Request Validation

```typescript
import { z } from 'zod';

const createProductSchema = z.object({
  nameAr: z.string().min(1).max(200),
  nameEn: z.string().min(1).max(200),
  descriptionAr: z.string().max(5000).optional(),
  descriptionEn: z.string().max(5000).optional(),
  price: z.number().positive().max(5_000_000),
  currency: z.enum(['YER', 'SAR', 'USD']),
  categoryId: z.string().uuid(),
  sku: z.string().min(1).max(50),
  stock: z.number().int().min(0),
});
```

### 5.3 Response Envelope

```typescript
// Success
{
  "success": true,
  "message": "Product created successfully",
  "data": { ... },
  "request_id": "req_abc123"
}

// Error
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input data",
    "details": [...]
  },
  "request_id": "req_abc123"
}
```

### Decision

**RESTful API v1** with Zod validation, standardized response envelope, and consistent error codes. No GraphQL at launch (future consideration).

---

## 6. Error Handling Strategy

### 6.1 Error Classification

```typescript
// Base application error
export abstract class ApplicationError extends Error {
  abstract readonly code: string;
  abstract readonly statusCode: number;
  abstract readonly isOperational: boolean;

  constructor(message: string, metadata?: ErrorMetadata) {
    super(message);
    this.metadata = metadata;
  }
}

// Specific error types
export class InsufficientFundsError extends ApplicationError {
  code = 'INSUFFICIENT_FUNDS';
  statusCode = 422;
  isOperational = true;
}

export class DatabaseConnectionError extends ApplicationError {
  code = 'DATABASE_ERROR';
  statusCode = 500;
  isOperational = false;  // System error, needs alerting
}
```

### 6.2 Error Handling Layers

| Layer | Responsibility |
|-------|---------------|
| API Layer | Catch errors, map to HTTP status, log |
| Service Layer | Throw domain-specific errors |
| Domain Layer | Validate invariants, throw domain errors |
| Infrastructure | Catch infrastructure errors, wrap in application errors |

### 6.3 Error Response Mapping

| Error Type | HTTP Status | Example |
|-----------|-------------|---------|
| ValidationError | 400 | Invalid input data |
| UnauthorizedError | 401 | Missing/invalid token |
| ForbiddenError | 403 | Insufficient permissions |
| NotFoundError | 404 | Resource not found |
| ConflictError | 409 | Duplicate phone number |
| BusinessError | 422 | Insufficient funds |
| RateLimitError | 429 | Too many requests |
| SystemError | 500 | Database failure |

### Decision

**Typed error hierarchy** with operational vs system error distinction. Operational errors return user-friendly messages; system errors trigger alerting. All errors logged with correlation ID.

---

## 7. Caching Strategy

### 7.1 Cache-Aside Pattern

```typescript
export class CachedProductService {
  constructor(
    private productRepo: ProductRepository,
    private cache: RedisCache,
  ) {}

  async getProduct(id: ProductId): Promise<Product | null> {
    const cacheKey = `product:${id.value}`;

    // Try cache first
    const cached = await this.cache.get<Product>(cacheKey);
    if (cached) return cached;

    // Fall back to database
    const product = await this.productRepo.findById(id);
    if (product) {
      await this.cache.set(cacheKey, product, { ttl: 3600 }); // 1 hour
    }
    return product;
  }

  async updateProduct(product: Product): Promise<void> {
    await this.productRepo.save(product);
    // Invalidate cache
    await this.cache.delete(`product:${product.id.value}`);
  }
}
```

### 7.2 Cache Invalidation Strategy

| Data Type | Strategy | TTL |
|-----------|----------|-----|
| Product details | Write-through invalidation | 1 hour |
| Category tree | TTL-based (daily refresh) | 24 hours |
| User sessions | Automatic expiry | 24 hours |
| Cart data | Write-through invalidation | 30 minutes |
| Search results | TTL-based | 5 minutes |
| Rate limits | Sliding window | 1 minute |
| Inventory locks | Write-through | 10 minutes |

### Decision

**Cache-aside with write-through invalidation** for mutable data. TTL-based for immutable/semi-immutable data. Cache warming for high-traffic products during off-peak hours.

---

## 8. Search Design

### 8.1 Index Structure

```json
{
  "mappings": {
    "properties": {
      "id": { "type": "keyword" },
      "nameAr": {
        "type": "text",
        "analyzer": "arabic",
        "fields": { "keyword": { "type": "keyword" } }
      },
      "nameEn": {
        "type": "text",
        "analyzer": "english",
        "fields": { "keyword": { "type": "keyword" } }
      },
      "descriptionAr": { "type": "text", "analyzer": "arabic" },
      "descriptionEn": { "type": "text", "analyzer": "english" },
      "price": { "type": "scaled_float", "scaling_factor": 100 },
      "currency": { "type": "keyword" },
      "categoryId": { "type": "keyword" },
      "vendorId": { "type": "keyword" },
      "rating": { "type": "float" },
      "reviewCount": { "type": "integer" },
      "status": { "type": "keyword" },
      "createdAt": { "type": "date" },
      "suggest": { "type": "completion" }
    }
  }
}
```

### 8.2 Search Query Pattern

```typescript
export class ProductSearchService {
  async search(query: string, filters: SearchFilters): Promise<SearchResult> {
    const must = [];
    const filter = [];

    if (query) {
      must.push({
        multi_match: {
          query,
          fields: ['nameAr^3', 'nameEn^3', 'descriptionAr', 'descriptionEn'],
          fuzziness: 'AUTO',
        },
      });
    }

    if (filters.categoryId) {
      filter.push({ term: { categoryId: filters.categoryId } });
    }

    if (filters.priceMin || filters.priceMax) {
      filter.push({
        range: {
          price: {
            gte: filters.priceMin,
            lte: filters.priceMax,
          },
        },
      });
    }

    return this.elasticsearch.search({
      index: 'products',
      body: { query: { bool: { must, filter } } },
    });
  }
}
```

### Decision

**Elasticsearch as primary search engine** with CDC-based sync from PostgreSQL. Bilingual Arabic/English analyzers. Fuzzy matching for typo tolerance. Faceted filtering for category, price, rating.

---

## 9. File Upload Design

### 9.1 Upload Flow

```
Client → API (pre-signed URL) → Client uploads to MinIO → API confirms → Metadata saved to PostgreSQL
```

### 9.2 File Constraints

| File Type | Allowed Extensions | Max Size | Validation |
|-----------|-------------------|----------|------------|
| Product images | JPEG, PNG, WebP | 5MB each | Dimensions, content type |
| Profile images | JPEG, PNG | 2MB | Face detection optional |
| KYC documents | JPEG, PNG, PDF | 10MB | Document type verification |
| CMS content | JPEG, PNG, WebP | 5MB | Content moderation |

### 9.3 Image Processing Pipeline

```typescript
export class ImageProcessor {
  async processProductImage(buffer: Buffer): Promise<ProcessedImage> {
    const resized = await sharp(buffer)
      .resize(800, 800, { fit: 'inside', withoutEnlargement: true })
      .jpeg({ quality: 85 })
      .toBuffer();

    const thumbnail = await sharp(buffer)
      .resize(200, 200, { fit: 'cover' })
      .jpeg({ quality: 70 })
      .toBuffer();

    return { original: buffer, resized, thumbnail };
  }
}
```

### Decision

**Pre-signed URL upload** directly to MinIO (no server proxy). Server-side image processing with Sharp. Automatic thumbnail generation. Content type validation. Metadata stored in PostgreSQL with MinIO object keys.

---

## 10. Notification Design

### 10.1 Notification Channels

| Channel | Use Cases | Delivery SLA |
|---------|-----------|-------------|
| SMS | OTP, order updates | < 30 seconds |
| WhatsApp | OTP, promotions | < 60 seconds |
| Push (FCM) | Order status, offers | < 5 seconds |
| In-app | All notifications | Real-time |
| Email | Receipts, reports | < 5 minutes |

### 10.2 Notification Template System

```typescript
export interface NotificationTemplate {
  id: string;
  channel: 'sms' | 'whatsapp' | 'push' | 'email';
  event: string;  // e.g., 'order.confirmed'
  subject?: string;  // For email
  bodyTemplate: string;  // Supports {{variable}} interpolation
  variables: string[];
}
```

### 10.3 Notification Priority

| Priority | Channel | Retries | Backoff |
|----------|---------|---------|---------|
| Critical | SMS, Push | 5 | Exponential |
| High | SMS, WhatsApp | 3 | Linear |
| Normal | Push, In-app | 2 | Linear |
| Low | Email, In-app | 1 | None |

### Decision

**Multi-channel notification system** with template-based messages. SMS for OTP and critical alerts. WhatsApp for secondary delivery. Push for real-time updates. In-app for persistent notifications. Retry with exponential backoff.

---

## 11. Multi-Currency Design

### 11.1 Currency Configuration

```typescript
export const CURRENCIES = {
  YER: { code: 'YER', symbol: '﷼', name: 'Yemeni Rial', decimals: 0 },
  SAR: { code: 'SAR', symbol: '﷼', name: 'Saudi Riyal', decimals: 2 },
  USD: { code: 'USD', symbol: '$', name: 'US Dollar', decimals: 2 },
} as const;
```

### 11.2 Exchange Rate Management

```typescript
export interface ExchangeRate {
  from: CurrencyCode;
  to: CurrencyCode;
  rate: number;
  fetchedAt: Date;
  source: string;
}

export class CurrencyService {
  async convert(amount: number, from: CurrencyCode, to: CurrencyCode): Promise<number> {
    if (from === to) return amount;
    const rate = await this.exchangeRateRepo.getRate(from, to);
    return Math.round(amount * rate.rate * 100) / 100;
  }
}
```

### 11.3 Display Rules

| Context | Currency Display |
|---------|-----------------|
| Product listing | Vendor's currency |
| Cart/checkout | Customer's preferred currency |
| Wallet balance | Wallet currency |
| Admin dashboard | USD equivalent |
| Vendor settlement | Settlement currency (configurable) |

### Decision

**Multi-currency support** with daily exchange rate updates from central bank APIs. Products listed in vendor currency; displayed in customer preferred currency. Wallet supports YER, SAR, USD. Settlement configurable per vendor.

---

## 12. Internationalization Design

### 12.1 Bilingual Content Model

```typescript
export interface BilingualText {
  ar: string;  // Arabic (primary)
  en: string;  // English (secondary)
}

export interface BilingualContent {
  title: BilingualText;
  description: BilingualText;
  slug: BilingualText;
}
```

### 12.2 RTL/LTR Handling

| Element | Direction | Implementation |
|---------|-----------|---------------|
| UI Layout | RTL (default), LTR (toggle) | CSS `dir` attribute |
| Text alignment | Auto | CSS `text-align: start` |
| Navigation | RTL flow | CSS logical properties |
| Forms | RTL labels, LTR inputs | Per-element direction |
| Numbers | LTR within RTL | Unicode algorithm |

### 12.3 Content Storage

| Field | Column | Default |
|-------|--------|---------|
| Product name | `name_ar`, `name_en` | Both required |
| Category name | `name_ar`, `name_en` | Both required |
| CMS page content | `content_ar`, `content_en` | Both required |
| Error messages | i18n JSON files | All languages |

### Decision

**Bilingual (Arabic/English) content** with Arabic as default. All user-facing content stored in separate `ar` and `en` columns. RTL-first CSS design. Language toggle in UI. Admin content entry in both languages required.

---

## 13. Mobile-First Design

### 13.1 Responsive Breakpoints

```css
/* Mobile-first breakpoints */
:root {
  --bp-mobile: 375px;    /* iPhone SE */
  --bp-tablet: 768px;    /* iPad */
  --bp-desktop: 1024px;  /* Desktop */
  --bp-wide: 1440px;     /* Wide desktop */
}
```

### 13.2 Performance Budgets

| Metric | Mobile | Desktop |
|--------|--------|---------|
| First Contentful Paint | < 1.5s | < 1.0s |
| Largest Contentful Paint | < 2.5s | < 2.0s |
| First Input Delay | < 100ms | < 50ms |
| Cumulative Layout Shift | < 0.1 | < 0.1 |
| Total Bundle Size | < 200KB | < 350KB |

### 13.3 Touch Target Sizes

| Element | Minimum Size | YemenMart Target |
|---------|-------------|------------------|
| Button | 44×44px | 48×48px |
| Link | 44×44px | 48×48px |
| Input | 44px height | 48px height |
| Icon | 24×24px | 32×32px |

### Decision

**Mobile-first responsive design** with 4 breakpoints. Performance budgets enforced via Lighthouse CI. Touch targets minimum 48×48px. Lazy loading for images below the fold. Skeleton screens for perceived performance.

---

## 14. Security Design

### 14.1 Authentication Flow

```
1. User enters phone number
2. System sends OTP via SMS/WhatsApp
3. User enters OTP
4. System validates OTP
5. JWT access token (15 min) + refresh token (7 days) issued
6. Subsequent requests use Bearer token
```

### 14.2 Authorization Model (RBAC)

```typescript
export const ROLES = {
  CUSTOMER: {
    permissions: ['order:create', 'order:read:own', 'review:create'],
  },
  VENDOR: {
    permissions: [
      'product:create', 'product:read:own', 'product:update:own',
      'order:read:own', 'order:update:own',
      'store:read:own', 'store:update:own',
    ],
  },
  ADMIN: {
    permissions: [
      'user:read', 'user:update', 'user:ban',
      'vendor:approve', 'vendor:reject',
      'order:read:all', 'order:update:all',
      'finance:read', 'finance:payout',
      'content:manage', 'coupon:manage',
    ],
  },
  SUPPORT: {
    permissions: [
      'ticket:read', 'ticket:update', 'ticket:assign',
      'order:read:all', 'user:read',
    ],
  },
  DELIVERY: {
    permissions: [
      'shipment:read:assigned', 'shipment:update:status',
      'delivery:confirm',
    ],
  },
} as const;
```

### 14.3 Rate Limiting

| Endpoint Category | Limit | Window |
|------------------|-------|--------|
| OTP request | 3 per phone | 10 minutes |
| Login | 5 per IP | 15 minutes |
| API (authenticated) | 100 per user | 1 minute |
| API (unauthenticated) | 20 per IP | 1 minute |
| File upload | 10 per user | 1 minute |

### Decision

**JWT-based authentication** with RS256 signing. 15-minute access tokens, 7-day refresh tokens. RBAC with 5 predefined roles. Rate limiting per endpoint category. All API endpoints require authentication except public product browsing.

---

## Design Decision Log

| Date | Decision | Rationale |
|------|----------|-----------|
| 2026-09-01 | Repository + Mapper | Clean domain, testable |
| 2026-09-01 | Explicit Unit of Work | Transaction control |
| 2026-09-01 | Async domain events | Module decoupling |
| 2026-09-01 | REST API with Zod | Type-safe validation |
| 2026-09-01 | Typed error hierarchy | Consistent error handling |
| 2026-09-01 | Cache-aside + write-through | Performance + consistency |
| 2026-09-01 | Elasticsearch CDC | Bilingual search |
| 2026-09-01 | Pre-signed URL upload | No server bottleneck |
| 2026-09-01 | Multi-channel notifications | Redundancy |
| 2026-09-01 | Multi-currency with daily rates | Market requirement |
| 2026-09-01 | Bilingual content model | Arabic-first market |
| 2026-09-01 | Mobile-first responsive | Yemen mobile usage |
| 2026-09-01 | JWT + RBAC security | Industry standard |

---

## Related Categories

- `04-architecture/design-patterns.md` - Pattern implementation details
- `04-architecture/layered-architecture.md` - Layer responsibilities
- `18-decisions/architecture-decisions.md` - Architecture ADRs

---

*Source: Design pattern evaluation from architectural analysis and best practices*
