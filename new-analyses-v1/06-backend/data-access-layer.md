# Data Access Layer

## Overview

YemenMart uses Prisma ORM with PostgreSQL. The data access layer follows the Repository + Unit of Work patterns to keep domain logic decoupled from persistence concerns.

---

## Prisma ORM Configuration

### Schema Location

```
packages/prisma/prisma/schema.prisma
```

### Connection Configuration

```typescript
// packages/prisma/src/client.ts
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient({
  log: process.env.NODE_ENV === 'development'
    ? ['query', 'error', 'warn']
    : ['error'],
  datasources: {
    db: {
      url: process.env.DATABASE_URL,
      // Connection pool: 5-20 connections
      // Configured via connection string params
      // postgresql://user:pass@host:5432/db?connection_limit=20&pool_timeout=10
    },
  },
});

export default prisma;
```

### Environment Variables

```env
# packages/prisma/.env
DATABASE_URL="postgresql://yemenmart:password@localhost:5432/yemenmart_dev?schema=public&connection_limit=20&pool_timeout=10"
SHADOW_DATABASE_URL="postgresql://yemenmart:password@localhost:5432/yemenmart_shadow"
DIRECT_URL="postgresql://yemenmart:password@localhost:5432/yemenmart_dev?schema=public"
```

### Key Models (Partial)

```prisma
// packages/prisma/prisma/schema.prisma

generator client {
  provider        = "prisma-client-js"
  previewFeatures = ["fullTextSearch", "postgresqlExtensions", "metrics"]
}

datasource db {
  provider   = "postgresql"
  url        = env("DATABASE_URL")
  directUrl  = env("DIRECT_URL")
}

// ─── User & Auth ───────────────────────────────

model User {
  id            String    @id @default(uuid())
  phone         String    @unique
  email         String?   @unique
  firstName     String
  lastName      String
  role          UserRole  @default(BUYER)
  status        UserStatus @default(ACTIVE)
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt

  // Relations
  profile       UserProfile?
  addresses     Address[]
  wallet        Wallet?
  sessions      Session[]
  orders        Order[]
  store         Store?      @relation("StoreOwner")
  loyaltyAccount LoyaltyAccount?

  @@index([phone])
  @@index([email])
  @@index([role])
}

model UserProfile {
  id        String  @id @default(uuid())
  userId    String  @unique
  avatar    String?
  bio       String?
  user      User    @relation(fields: [userId], references: [id])
}

model Session {
  id            String   @id @default(uuid())
  userId        String
  token         String   @unique
  refreshToken  String   @unique
  ipAddress     String?
  userAgent     String?
  expiresAt     DateTime
  createdAt     DateTime @default(now())
  user          User     @relation(fields: [userId], references: [id])

  @@index([userId])
  @@index([token])
  @@index([expiresAt])
}

model Address {
  id          String  @id @default(uuid())
  userId      String
  label       String  // "Home", "Work", etc.
  fullName    String
  phone       String
  city        String
  district    String
  street      String
  building    String?
  floor       String?
  apartment   String?
  latitude    Float?
  longitude   Float?
  isDefault   Boolean @default(false)
  user        User    @relation(fields: [userId], references: [id])

  @@index([userId])
}

// ─── Product ───────────────────────────────────

model Product {
  id            String        @id @default(uuid())
  storeId       String
  name          String
  slug          String        @unique
  description   String?
  status        ProductStatus @default(DRAFT)
  basePrice     Decimal       @db.Decimal(10, 2)
  currency      String        @default("YER")
  stockQuantity Int           @default(0)
  softHoldQty   Int           @default(0)
  availableQty  Int           @default(0) // stockQuantity - softHoldQty
  images        ProductImage[]
  variants      ProductVariant[]
  category      Category?     @relation(fields: [categoryId], references: [id])
  categoryId    String?
  store         Store         @relation(fields: [storeId], references: [id])
  ratings       Rating[]
  createdAt     DateTime      @default(now())
  updatedAt     DateTime      @updatedAt

  @@index([storeId])
  @@index([categoryId])
  @@index([status])
  @@index([name])
}

model ProductVariant {
  id           String   @id @default(uuid())
  productId    String
  name         String
  sku          String   @unique
  price        Decimal  @db.Decimal(10, 2)
  stockQuantity Int     @default(0)
  softHoldQty  Int      @default(0)
  attributes   Json     // {"color": "Red", "size": "XL"}
  product      Product  @relation(fields: [productId], references: [id])

  @@index([productId])
  @@index([sku])
}

model Category {
  id        String     @id @default(uuid())
  name      String
  slug      String     @unique
  parentId  String?
  parent    Category?  @relation("CategoryTree", fields: [parentId], references: [id])
  children  Category[] @relation("CategoryTree")
  products  Product[]
  icon      String?
  sortOrder Int        @default(0)

  @@index([parentId])
}

// ─── Store ─────────────────────────────────────

model Store {
  id          String      @id @default(uuid())
  ownerId     String      @unique
  name        String
  slug        String      @unique
  description String?
  logo        String?
  banner      String?
  status      StoreStatus @default(PENDING)
  rating      Float       @default(0)
  commission  Json        // Commission config
  owner       User        @relation("StoreOwner", fields: [ownerId], references: [id])
  products    Product[]
  orders      Order[]     @relation("StoreOrders")
  createdAt   DateTime    @default(now())

  @@index([ownerId])
  @@index([slug])
}

// ─── Order ─────────────────────────────────────

model Order {
  id            String      @id @default(uuid())
  userId        String
  parentId      String?     // For sub-orders
  storeId       String?     // For sub-orders
  type          OrderType   @default(MASTER)
  status        OrderStatus @default(PENDING)
  total         Decimal     @db.Decimal(10, 2)
  currency      String      @default("YER")
  subtotal      Decimal     @db.Decimal(10, 2)
  tax           Decimal     @db.Decimal(10, 2) @default(0)
  deliveryFee   Decimal     @db.Decimal(10, 2) @default(0)
  discount      Decimal     @db.Decimal(10, 2) @default(0)
  commission    Decimal     @db.Decimal(10, 2) @default(0)
  sellerPayout  Decimal     @db.Decimal(10, 2) @default(0)
  couponCode    String?
  pointsUsed    Int         @default(0)
  addressId     String?
  notes         String?
  createdAt     DateTime    @default(now())
  updatedAt     DateTime    @updatedAt

  // Relations
  user          User         @relation(fields: [userId], references: [id])
  store         Store?       @relation("StoreOrders", fields: [storeId], references: [id])
  parent        Order?       @relation("SubOrders", fields: [parentId], references: [id])
  subOrders     Order[]      @relation("SubOrders")
  items         OrderItem[]
  statusHistory OrderStatusHistory[]
  deliveryCode  DeliveryCode?
  returns       ReturnRequest[]
  escrow        EscrowHold?
  payments      WalletTransaction[]

  @@index([userId])
  @@index([parentId])
  @@index([storeId])
  @@index([status])
  @@index([type])
  @@index([createdAt])
}

model OrderItem {
  id          String   @id @default(uuid())
  orderId     String
  productId   String
  variantId   String?
  name        String
  price       Decimal  @db.Decimal(10, 2)
  quantity    Int
  total       Decimal  @db.Decimal(10, 2)
  order       Order    @relation(fields: [orderId], references: [id])

  @@index([orderId])
}

model OrderStatusHistory {
  id        String      @id @default(uuid())
  orderId   String
  from      OrderStatus?
  to        OrderStatus
  actorId   String?
  actorRole String?
  notes     String?
  createdAt DateTime    @default(now())
  order     Order       @relation(fields: [orderId], references: [id])

  @@index([orderId])
}

// ─── Delivery ──────────────────────────────────

model DeliveryCode {
  id          String   @id @default(uuid())
  orderId     String   @unique
  codeHash    String
  attempts    Int      @default(0)
  maxAttempts Int      @default(3)
  expiresAt   DateTime
  used        Boolean  @default(false)
  usedAt      DateTime?
  order       Order    @relation(fields: [orderId], references: [id])

  @@index([orderId])
}

// ─── Payment ───────────────────────────────────

model Wallet {
  id        String   @id @default(uuid())
  userId    String   @unique
  balance   Decimal  @db.Decimal(12, 2) @default(0)
  currency  String   @default("YER")
  version   Int      @default(0) // Optimistic concurrency
  user      User     @relation(fields: [userId], references: [id])
  transactions WalletTransaction[]

  @@index([userId])
}

model WalletTransaction {
  id              String   @id @default(uuid())
  walletId        String
  type            WalletTxType
  amount          Decimal  @db.Decimal(12, 2)
  currency        String   @default("YER")
  balanceAfter    Decimal  @db.Decimal(12, 2)
  orderId         String?
  idempotencyKey  String   @unique
  description     String?
  metadata        Json?
  createdAt       DateTime @default(now())
  wallet          Wallet   @relation(fields: [walletId], references: [id])
  order           Order?   @relation(fields: [orderId], references: [id])

  @@index([walletId])
  @@index([orderId])
  @@index([idempotencyKey])
  @@index([createdAt])
}

model EscrowHold {
  id        String    @id @default(uuid())
  orderId   String    @unique
  amount    Decimal   @db.Decimal(12, 2)
  currency  String    @default("YER")
  status    EscrowStatus @default(HELD)
  heldAt    DateTime  @default(now())
  releaseAt DateTime
  releasedAt DateTime?
  order     Order     @relation(fields: [orderId], references: [id])

  @@index([orderId])
  @@index([status])
  @@index([releaseAt])
}

// ─── Coupon ────────────────────────────────────

model Coupon {
  id                String         @id @default(uuid())
  code              String         @unique
  type              CouponType
  value             Decimal        @db.Decimal(10, 2)
  minOrderAmount    Decimal        @db.Decimal(10, 2) @default(0)
  maxDiscount       Decimal?       @db.Decimal(10, 2)
  usageLimit        Int?
  usageCount        Int            @default(0)
  perUserLimit      Int            @default(1)
  stackable         Boolean        @default(false)
  validFrom         DateTime
  validUntil        DateTime
  active            Boolean        @default(true)
  applicableStoreIds String[]
  applicableCategoryIds String[]
  usages            CouponUsage[]

  @@index([code])
  @@index([validFrom, validUntil])
}

model CouponUsage {
  id        String   @id @default(uuid())
  couponId  String
  userId    String
  orderId   String
  usedAt    DateTime @default(now())
  coupon    Coupon   @relation(fields: [couponId], references: [id])

  @@index([couponId])
  @@index([userId])
  @@unique([couponId, userId, orderId])
}

// ─── Loyalty ───────────────────────────────────

model LoyaltyAccount {
  id             String   @id @default(uuid())
  userId         String   @unique
  totalPoints    Int      @default(0)
  lifetimePoints Int      @default(0)
  tier           String   @default("BRONZE")
  user           User     @relation(fields: [userId], references: [id])
  points         LoyaltyPoints[]

  @@index([userId])
}

model LoyaltyPoints {
  id        String          @id @default(uuid())
  userId    String
  type      LoyaltyTxType
  points    Int
  source    String
  orderId   String?
  expiresAt DateTime?
  createdAt DateTime        @default(now())
  account   LoyaltyAccount  @relation(fields: [userId], references: [userId])

  @@index([userId])
  @@index([expiresAt])
}

// ─── Notification ──────────────────────────────

model Notification {
  id        String             @id @default(uuid())
  userId    String
  type      NotificationType
  channel   NotificationChannel
  title     String
  body      String
  data      Json?
  status    NotificationStatus @default(PENDING)
  sentAt    DateTime?
  readAt    DateTime?
  error     String?
  retries   Int                @default(0)
  createdAt DateTime           @default(now())

  @@index([userId])
  @@index([status])
  @@index([createdAt])
  @@index([userId, readAt])
}

model NotificationPreference {
  id              String  @id @default(uuid())
  userId          String  @unique
  email           Boolean @default(true)
  push            Boolean @default(true)
  sms             Boolean @default(true)
  whatsapp        Boolean @default(false)
  orderUpdates    Boolean @default(true)
  promotions      Boolean @default(false)
  priceAlerts     Boolean @default(true)
  user            User    @relation(fields: [userId], references: [id])

  @@index([userId])
}

// ─── Enums ─────────────────────────────────────

enum UserRole { BUYER SELLER ADMIN }
enum UserStatus { ACTIVE SUSPENDED DELETED }
enum ProductStatus { DRAFT ACTIVE OUT_OF_STOCK DISCONTINUED }
enum StoreStatus { PENDING ACTIVE SUSPENDED REJECTED }
enum OrderType { MASTER SUB }
enum OrderStatus {
  PENDING ON_HOLD CONFIRMED PROCESSING SHIPPED OUT_FOR_DELIVERY
  DELIVERED DELIVERY_FAILED COMPLETED CANCELLED
  RETURN_REQUESTED RETURN_APPROVED RETURN_REJECTED RETURNED
  REFUNDING REFUNDED DISPUTE_OPENED DISPUTE_RESOLVED
}
enum WalletTxType { DEBIT CREDIT TRANSFER ESCROW_HOLD ESCROW_RELEASE REFUND }
enum EscrowStatus { HELD RELEASED REFUNDED EXPIRED }
enum CouponType { PERCENTAGE FIXED_AMOUNT FREE_SHIPPING }
enum LoyaltyTxType { EARNED REDEEMED REVOKED EXPIRED }
enum NotificationType {
  ORDER_CONFIRMED ORDER_SHIPPED ORDER_DELIVERED ORDER_CANCELLED
  DELIVERY_CODE WALLET_TOPUP WALLET_DEBIT
  PROMOTION PRICE_ALERT SYSTEM
}
enum NotificationChannel { EMAIL SMS WHATSAPP PUSH IN_APP }
enum NotificationStatus { PENDING SENT FAILED READ }
```

---

## Repository Pattern

### Interface Definition

```typescript
// packages/contracts/src/repository.ts
export interface IRepository<T, ID> {
  findById(id: ID): Promise<T | null>;
  findAll(filters?: FindFilters): Promise<T[]>;
  create(data: CreateInput<T>): Promise<T>;
  update(id: ID, data: Partial<T>): Promise<T>;
  delete(id: ID): Promise<void>;
  count(filters?: FindFilters): Promise<number>;
}

export interface IOrderRepository extends IRepository<Order, string> {
  findByUserId(userId: string, pagination: PaginationInput): Promise<PaginatedResult<Order>>;
  findByStoreId(storeId: string, filters: OrderFilters): Promise<PaginatedResult<Order>>;
  findSubOrders(parentId: string): Promise<Order[]>;
  findByStatus(status: OrderStatus): Promise<Order[]>;
  updateStatus(id: string, status: OrderStatus, metadata?: StatusMetadata): Promise<Order>;
}
```

### Prisma Repository Implementation

```typescript
// packages/prisma/src/repositories/order.repository.ts
import { PrismaClient, Prisma } from '@prisma/client';
import { IOrderRepository, Order, OrderStatus, PaginationInput } from '@yemenmart/contracts';

export class PrismaOrderRepository implements IOrderRepository {
  constructor(private prisma: PrismaClient) {}

  async findById(id: string): Promise<Order | null> {
    return this.prisma.order.findUnique({
      where: { id },
      include: {
        items: true,
        statusHistory: { orderBy: { createdAt: 'desc' }, take: 10 },
        store: true,
        user: { select: { id: true, firstName: true, lastName: true, phone: true } },
      },
    });
  }

  async findByUserId(userId: string, pagination: PaginationInput): Promise<PaginatedResult<Order>> {
    const [data, total] = await Promise.all([
      this.prisma.order.findMany({
        where: { userId, type: 'MASTER' },
        include: { items: true, subOrders: true },
        orderBy: { createdAt: 'desc' },
        skip: (pagination.page - 1) * pagination.limit,
        take: pagination.limit,
      }),
      this.prisma.order.count({
        where: { userId, type: 'MASTER' },
      }),
    ]);

    return { data, total, page: pagination.page, limit: pagination.limit };
  }

  async findSubOrders(parentId: string): Promise<Order[]> {
    return this.prisma.order.findMany({
      where: { parentId },
      include: { items: true },
    });
  }

  async findByStatus(status: OrderStatus): Promise<Order[]> {
    return this.prisma.order.findMany({
      where: { status, type: 'MASTER' },
      include: { items: true },
    });
  }

  async create(data: CreateOrderInput): Promise<Order> {
    return this.prisma.order.create({
      data: {
        userId: data.userId,
        type: data.type ?? 'MASTER',
        total: data.total,
        currency: data.currency ?? 'YER',
        subtotal: data.subtotal,
        tax: data.tax ?? 0,
        deliveryFee: data.deliveryFee ?? 0,
        discount: data.discount ?? 0,
        addressId: data.addressId,
        notes: data.notes,
        items: {
          create: data.items.map(item => ({
            productId: item.productId,
            variantId: item.variantId,
            name: item.name,
            price: item.price,
            quantity: item.quantity,
            total: item.price * item.quantity,
          })),
        },
      },
      include: { items: true },
    });
  }

  async updateStatus(
    id: string,
    status: OrderStatus,
    metadata?: StatusMetadata,
  ): Promise<Order> {
    return this.prisma.$transaction(async (tx) => {
      const order = await tx.order.findUnique({ where: { id } });
      if (!order) throw new OrderNotFoundError(id);

      const [updated] = await Promise.all([
        tx.order.update({
          where: { id },
          data: { status, updatedAt: new Date() },
          include: { items: true },
        }),
        tx.orderStatusHistory.create({
          data: {
            orderId: id,
            from: order.status,
            to: status,
            actorId: metadata?.actorId,
            actorRole: metadata?.actorRole,
            notes: metadata?.notes,
          },
        }),
      ]);

      return updated;
    });
  }

  async count(filters?: OrderFilters): Promise<number> {
    return this.prisma.order.count({
      where: {
        type: 'MASTER',
        ...(filters?.status && { status: filters.status }),
        ...(filters?.storeId && { storeId: filters.storeId }),
        ...(filters?.userId && { userId: filters.userId }),
      },
    });
  }
}
```

---

## Unit of Work Pattern

### Interface

```typescript
// packages/core/src/unit-of-work.ts
export interface IUnitOfWork {
  execute<T>(fn: (repositories: RepositorySet) => Promise<T>): Promise<T>;
  getRepository<T>(token: string): T;
}

export interface RepositorySet {
  orders: IOrderRepository;
  products: IProductRepository;
  users: IUserRepository;
  wallets: IWalletRepository;
  // ... other repositories
}
```

### Implementation

```typescript
// packages/prisma/src/unit-of-work.ts
import { PrismaClient } from '@prisma/client';
import { IUnitOfWork, RepositorySet } from '@yemenmart/contracts';
import { PrismaOrderRepository } from './repositories/order.repository';
import { PrismaProductRepository } from './repositories/product.repository';
import { PrismaWalletRepository } from './repositories/wallet.repository';

export class PrismaUnitOfWork implements IUnitOfWork {
  private prisma: PrismaClient;
  private repositories: RepositorySet | null = null;

  constructor(private container: Container) {
    this.prisma = this.container.resolve(PrismaClient);
  }

  async execute<T>(fn: (repos: RepositorySet) => Promise<T>): Promise<T> {
    return this.prisma.$transaction(async (tx) => {
      // Create repositories bound to this transaction
      this.repositories = {
        orders: new PrismaOrderRepository(tx as any),
        products: new PrismaProductRepository(tx as any),
        wallets: new PrismaWalletRepository(tx as any),
        // ...
      };

      try {
        const result = await fn(this.repositories);
        return result;
      } finally {
        this.repositories = null;
      }
    }, {
      maxWait: 5000,
      timeout: 30000,
    });
  }

  getRepository<T>(token: string): T {
    if (!this.repositories) {
      throw new Error('UnitOfWork not in active transaction');
    }
    return (this.repositories as any)[token];
  }
}
```

### Usage in Service

```typescript
@injectable()
export class OrderService implements IOrderService {
  constructor(
    @inject('UnitOfWork') private uow: IUnitOfWork,
    @inject('EventBus') private events: EventBus,
  ) {}

  async createOrder(input: CreateOrderInput): Promise<Order> {
    return this.uow.execute(async (repos) => {
      // All operations within single transaction
      const order = await repos.orders.create(input);

      for (const item of input.items) {
        await repos.products.decrementStock(item.productId, item.quantity);
      }

      if (input.couponCode) {
        await repos.coupons.recordUsage(input.couponCode, input.userId, order.id);
      }

      this.events.emit('order.created', { orderId: order.id });
      return order;
    });
  }
}
```

---

## Query Optimization

### Indexing Strategy

```sql
-- Composite indexes for common queries
CREATE INDEX idx_orders_user_status ON orders(user_id, status, created_at DESC);
CREATE INDEX idx_orders_store_status ON orders(store_id, status, created_at DESC);
CREATE INDEX idx_orders_created_status ON orders(created_at DESC, status);

-- Partial indexes for active records
CREATE INDEX idx_products_active ON products(status, category_id) WHERE status = 'ACTIVE';
CREATE INDEX idx_orders_pending ON orders(created_at) WHERE status IN ('PENDING', 'ON_HOLD');

-- GIN index for product search
CREATE INDEX idx_products_search ON products USING gin(to_tsvector('simple', name || ' ' || COALESCE(description, '')));

-- Covering index for balance checks
CREATE INDEX idx_wallet_balance ON wallets(user_id) INCLUDE (balance, version);
```

### Pagination

```typescript
// Cursor-based pagination for large datasets
async findOrdersCursor(
  storeId: string,
  cursor?: string,
  limit = 20,
): Promise<{ data: Order[]; nextCursor: string | null }> {
  const where = {
    storeId,
    ...(cursor && { createdAt: { lt: new Date(cursor) } }),
  };

  const data = await this.prisma.order.findMany({
    where,
    orderBy: { createdAt: 'desc' },
    take: limit + 1, // Fetch one extra to detect end
    include: { items: true },
  });

  const hasMore = data.length > limit;
  const result = hasMore ? data.slice(0, -1) : data;
  const nextCursor = hasMore ? result[result.length - 1].createdAt.toISOString() : null;

  return { data: result, nextCursor };
}
```

### Read Replicas

```typescript
// packages/prisma/src/read-replica.ts
export class PrismaWithReplicas {
  private readClient: PrismaClient;
  private writeClient: PrismaClient;

  constructor() {
    this.writeClient = new PrismaClient({ datasources: { db: { url: process.env.DATABASE_URL } } });
    this.readClient = new PrismaClient({ datasources: { db: { url: process.env.READ_REPLICA_URL } } });
  }

  // Read operations go to replica
  forRead<T>(fn: (prisma: PrismaClient) => Promise<T>): Promise<T> {
    return fn(this.readClient);
  }

  // Write operations go to primary
  forWrite<T>(fn: (prisma: PrismaClient) => Promise<T>): Promise<T> {
    return fn(this.writeClient);
  }
}
```

---

## Transaction Management

### Nested Transactions (Savepoints)

```typescript
// Prisma supports nested transactions via $transaction
async complexOperation(): Promise<void> {
  await this.prisma.$transaction(async (tx1) => {
    await tx1.order.update({ where: { id: '1' }, data: { status: 'CONFIRMED' } });

    await this.prisma.$transaction(async (tx2) => {
      // Nested transaction creates a savepoint
      await tx2.wallet.update({ where: { id: 'w1' }, data: { balance: { decrement: 100 } } });
    });

    // tx1 continues even if tx2 rolls back (if savepoint behavior)
  });
}
```

### Transaction Timeout Handling

```typescript
// Custom timeout per transaction type
const TRANSACTION_TIMEOUTS: Record<string, number> = {
  'order.create': 30000,     // 30s for complex order creation
  'wallet.debit': 10000,     // 10s for simple wallet ops
  'report.generate': 120000, // 2min for heavy queries
};

async executeWithTimeout<T>(
  key: string,
  fn: (tx: PrismaTransactionClient) => Promise<T>,
): Promise<T> {
  const timeout = TRANSACTION_TIMEOUTS[key] ?? 30000;
  return this.prisma.$transaction(fn, { timeout, maxWait: 5000 });
}
```

---

## Connection Pooling

### Configuration

```
# Connection string parameters
DATABASE_URL="postgresql://user:pass@host:5432/db?connection_limit=20&pool_timeout=10"
```

### Monitoring

```typescript
// packages/prisma/src/metrics.ts
export async function getPoolMetrics() {
  const metrics = await prisma.$metrics.json();
  return {
    pool: {
      active: metrics.gauges.find(g => g.key === 'prisma_pool_connections_active')?.value,
      idle: metrics.gauges.find(g => g.key === 'prisma_pool_connections_idle')?.value,
      total: metrics.gauges.find(g => g.key === 'prisma_pool_connections_total')?.value,
      waiting: metrics.gauges.find(g => g.key === 'prisma_pool_connections_waiting')?.value,
    },
    queries: {
      total: metrics.counters.find(c => c.key === 'prisma_client_queries_total')?.value,
      active: metrics.gauges.find(g => g.key === 'prisma_client_queries_active')?.value,
      duration: metrics.histograms.find(h => h.key === 'prisma_client_queries_duration_ms')?.value,
    },
  };
}
```

### Health Check

```typescript
// packages/prisma/src/health.ts
export async function checkDatabaseHealth(): Promise<HealthStatus> {
  try {
    const start = Date.now();
    await prisma.$queryRaw`SELECT 1`;
    const latency = Date.now() - start;

    return {
      status: latency < 100 ? 'healthy' : 'degraded',
      latency,
      timestamp: new Date().toISOString(),
    };
  } catch (error) {
    return {
      status: 'unhealthy',
      error: error.message,
      timestamp: new Date().toISOString(),
    };
  }
}
```
