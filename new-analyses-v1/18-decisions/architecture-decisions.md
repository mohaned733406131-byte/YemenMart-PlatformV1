# Architecture Decision Records — YemenMart

**Document ID:** YM-ADR-001
**Version:** 1.0
**Status:** Active
**Date:** 2026-09-13
**Language:** English

---

## Table of Contents

1. [ADR-001: Modular Monolith over Microservices](#adr-001)
2. [ADR-002: PostgreSQL as Primary Database](#adr-002)
3. [ADR-003: Prisma ORM Selection](#adr-003)
4. [ADR-004: BullMQ Event Bus](#adr-004)
5. [ADR-005: Redis as Cache Layer](#adr-005)
6. [ADR-006: Elasticsearch for Search](#adr-006)
7. [ADR-007: MinIO Object Storage](#adr-007)
8. [ADR-008: Layered Architecture within Modules](#adr-008)
9. [ADR-009: CQRS for High-Traffic Read Paths](#adr-009)
10. [ADR-010: Saga Pattern for Order Workflow](#adr-010)
11. [ADR-011: Repository Pattern Data Access](#adr-011)
12. [ADR-012: Domain Events Cross-Module Communication](#adr-012)
13. [ADR-013: Circuit Breaker for External Integrations](#adr-013)
14. [ADR-014: Anti-Corruption Layer for Third Parties](#adr-014)
15. [ADR-015: 17-State Order State Machine](#adr-015)

---

## ADR-001: Modular Monolith over Microservices

**Status:** ACCEPTED
**Date:** 2026-09-01
**Deciders:** CTO, Tech Lead, Architect
**Related:** ADR-008, ADR-012

### Context

YemenMart requires a platform that can grow from MVP to full enterprise scale. Microservices introduce operational complexity (service discovery, distributed tracing, network partitions) that the team cannot support with current staffing (15-25 agents). A monolith risks becoming unmaintainable without discipline.

### Decision

Adopt a **Modular Monolith** architecture: single deployable unit with 13 isolated business modules communicating through domain events and well-defined internal interfaces.

### Consequences

**Positive:**
- Single deployment artifact simplifies CI/CD
- Module boundaries enforce separation of concerns
- Domain events enable future extraction to microservices
- Lower operational overhead (one database, one cache, one queue)
- Easier debugging and local development

**Negative:**
- All modules share the same process and resource limits
- A bug in one module can crash the entire application
- Scaling requires horizontal replication of the full monolith
- Module extraction to microservices requires significant refactoring

### Alternatives Considered

| Option | Pros | Cons | Verdict |
|--------|------|------|---------|
| Microservices | Independent scaling, fault isolation | Operational complexity, team too small | Rejected |
| Simple Monolith | Fastest to build | Becomes unmaintainable at scale | Rejected |
| **Modular Monolith** | Balance of simplicity and structure | Requires discipline to maintain boundaries | **Accepted** |

### References

- `04-architecture/architecture-overview.md`
- `04-architecture/layered-architecture.md`

---

## ADR-002: PostgreSQL as Primary Database

**Status:** ACCEPTED
**Date:** 2026-09-01
**Deciders:** CTO, Tech Lead
**Related:** ADR-003

### Context

YemenMart handles financial transactions (wallet, escrow, settlements) requiring ACID compliance. The platform stores product catalogs with variable attributes (JSONB), order histories, and multi-currency data. Search capabilities are needed for product discovery.

### Decision

Use **PostgreSQL 15+** as the sole primary relational database with read replicas for reporting and search synchronization.

### Consequences

**Positive:**
- Full ACID compliance for financial data integrity
- JSONB support for flexible product attributes
- Native full-text search with Arabic + English `tsvector`
- Table partitioning for high-volume tables (order_items, audit_logs)
- Materialized views for analytics dashboards
- Mature replication and backup tooling

**Negative:**
- Vertical scaling limits (addressed with read replicas)
- Complex HA setup compared to managed cloud databases
- Requires pgBouncer connection pooling at scale

### Alternatives Considered

| Option | Pros | Cons | Verdict |
|--------|------|------|---------|
| MongoDB | Flexible schema, horizontal scaling | No ACID, weak relational integrity | Rejected |
| MySQL | Simpler setup, lower cost | Weaker JSONB, no native partitioning | Rejected |
| **PostgreSQL** | ACID, JSONB, FTS, partitioning | Vertical scaling limits | **Accepted** |
| SQL Server | Enterprise features | Licensing cost, Yemen market fit | Rejected |

### References

- `04-architecture/technology-stack.md`
- `08-database/schema-overview.md`

---

## ADR-003: Prisma ORM Selection

**Status:** ACCEPTED
**Date:** 2026-09-01
**Deciders:** Tech Lead, Backend Lead
**Related:** ADR-002

### Context

The team needs an ORM that integrates natively with TypeScript, provides type-safe database queries, and supports PostgreSQL features including JSONB, full-text search, and migrations. Team experience is primarily TypeScript.

### Decision

Use **Prisma 5.x** as the primary ORM with auto-generated TypeScript client.

### Consequences

**Positive:**
- 100% type-safe queries with auto-generated client
- Schema-first approach with version-controlled migrations
- Excellent developer experience (autocomplete, refactoring support)
- Built-in connection pooling and query optimization
- Strong community and documentation

**Negative:**
- Learning curve for complex raw SQL scenarios
- Performance overhead vs raw SQL for high-throughput queries
- Limited support for some PostgreSQL-specific features at launch
- Schema file becomes single source of truth (schema drift risk)

### Alternatives Considered

| Option | Pros | Cons | Verdict |
|--------|------|------|---------|
| TypeORM | Decorator-based, familiar patterns | Weaker type safety, less active maintenance | Rejected |
| Sequelize | Mature ecosystem | Poor TypeScript support, verbose API | Rejected |
| Knex.js | SQL-like, flexible | Query builder only, no schema management | Rejected |
| **Prisma** | Type-safe, migrations, DX | Learning curve, raw SQL limitations | **Accepted** |

### References

- `04-architecture/technology-stack.md`
- `06-backend/data-access-layer.md`

---

## ADR-004: BullMQ Event Bus

**Status:** ACCEPTED
**Date:** 2026-09-01
**Deciders:** Tech Lead, Architect
**Related:** ADR-005, ADR-012

### Context

YemenMart modules must communicate asynchronously (order events trigger payment processing, notifications, inventory updates). Synchronous calls create tight coupling and failure cascades. A lightweight in-process event system would not survive monolith restarts.

### Decision

Use **BullMQ 5.x** backed by Redis as the event bus and job queue system for inter-module communication and background processing.

### Consequences

**Positive:**
- Reliable at-least-once delivery with Redis persistence
- Priority queues for critical operations (payment, notifications)
- Built-in rate limiting, retries, and backoff strategies
- Horizontal scaling via multiple worker instances
- Job scheduling for delayed tasks (escrow timers, settlements)

**Negative:**
- Redis dependency adds infrastructure complexity
- Message ordering guaranteed per queue only, not across queues
- Dead letter queue management required for failed jobs

### Queue Definitions

| Queue | Concurrency | Priority | Purpose |
|-------|------------|----------|---------|
| order.events | 10 | High | Order state transitions |
| payment.process | 5 | Critical | Payment execution |
| notification.send | 20 | Normal | SMS, WhatsApp, push |
| inventory.sync | 15 | High | Stock level updates |
| search.reindex | 5 | Low | Elasticsearch sync |
| analytics.track | 50 | Low | Event tracking |
| delivery.assign | 10 | High | Delivery provider assignment |
| payout.process | 3 | Critical | Vendor settlements |

### References

- `04-architecture/technology-stack.md`
- `06-backend/job-scheduling.md`

---

## ADR-005: Redis as Cache Layer

**Status:** ACCEPTED
**Date:** 2026-09-01
**Deciders:** Tech Lead
**Related:** ADR-004

### Context

YemenMart needs sub-second response times for product browsing, cart operations, and session management. Direct database queries for every read operation would not scale for the expected traffic.

### Decision

Use **Redis 7+** as the caching layer with defined TTL and invalidation strategies per data type.

### Consequences

**Positive:**
- Sub-millisecond read latency for cached data
- Built-in pub/sub for cache invalidation signals
- Session store with automatic expiry
- Distributed rate limiting via sliding window
- Inventory lock management with atomic operations

**Negative:**
- Cache invalidation complexity (consistency vs performance trade-off)
- Redis restart causes cache miss storm
- Memory costs for large product catalogs

### Cache Strategy

| Cache Layer | TTL | Key Pattern | Invalidation |
|-------------|-----|-------------|--------------|
| Session store | 24h | `session:{userId}` | Logout / expiry |
| Product cache | 1h | `product:{id}` | Write-through on update |
| Cart cache | 30m | `cart:{userId}` | Checkout / manual clear |
| Rate limiter | 1m | `rate:{ip}:{endpoint}` | Sliding window |
| Search results | 5m | `search:{queryHash}` | TTL-based |
| Inventory locks | 10m | `lock:stock:{sku}` | Reserve / release |

### References

- `04-architecture/technology-stack.md`

---

## ADR-006: Elasticsearch for Search

**Status:** ACCEPTED
**Date:** 2026-09-01
**Deciders:** Tech Lead, Product Lead
**Related:** ADR-002

### Context

PostgreSQL full-text search handles basic queries but lacks faceted search, relevance tuning, typo tolerance, and Arabic language analyzer support needed for product discovery at scale.

### Decision

Use **Elasticsearch 8** as the dedicated search engine for product catalog and content search, synced from PostgreSQL via change data capture.

### Consequences

**Positive:**
- Faceted search (category, price range, vendor, rating)
- Arabic + English bilingual analyzers
- Typo tolerance and fuzzy matching
- Geolocation-based vendor/store search
- Real-time indexing with CDC from PostgreSQL

**Negative:**
- Additional infrastructure component to maintain
- Data consistency lag (eventual consistency between PG and ES)
- Index management overhead for large catalogs

### References

- `04-architecture/technology-stack.md`
- `10-integrations/analytics-integrations.md`

---

## ADR-007: MinIO Object Storage

**Status:** ACCEPTED
**Date:** 2026-09-01
**Deciders:** CTO, Tech Lead
**Related:** ADR-014

### Context

YemenMart stores product images, vendor documents (KYC), user profile images, and CMS assets. Cloud storage (AWS S3) introduces vendor lock-in and egress costs. Self-hosted storage gives full control.

### Decision

Use **MinIO** (S3-compatible) for all object storage with a self-hosted deployment.

### Consequences

**Positive:**
- S3 API compatibility (easy migration if needed later)
- Self-hosted: no vendor lock-in, no egress fees
- Built-in versioning and lifecycle policies
- Erasure coding for data durability
- Kubernetes-native deployment

**Negative:**
- Self-managed infrastructure overhead
- Requires capacity planning and monitoring
- No global CDN (add Cloudflare or similar separately)

### References

- `04-architecture/technology-stack.md`
- `14-devops-infrastructure/infrastructure-overview.md`

---

## ADR-008: Layered Architecture within Modules

**Status:** ACCEPTED
**Date:** 2026-09-01
**Deciders:** Architect, Tech Lead
**Related:** ADR-001

### Context

Without internal layering, modules become spaghetti code. Business logic tangled with database queries makes testing, refactoring, and onboarding difficult.

### Decision

Each module follows **5-layer architecture**: Presentation, API, Service, Domain, Infrastructure. Dependencies flow inward: Domain depends on nothing; Infrastructure implements domain interfaces.

### Consequences

**Positive:**
- Clear separation of concerns within each module
- Domain layer testable without infrastructure mocks
- Consistent structure across all 13 modules
- Easier onboarding (same pattern everywhere)

**Negative:**
- More files and folders per module
- Initial boilerplate overhead
- Risk of "anemic domain" if services absorb domain logic

### Layer Responsibilities

```
┌──────────────────────────────────────────┐
│          PRESENTATION LAYER              │
│  React 18 (Vendor) · Next.js 15 (Cust.) │
│  React Native (Mobile)                   │
├──────────────────────────────────────────┤
│            API LAYER                     │
│  Express/Fastify Routes · Middleware     │
│  Request Validation · Rate Limiting       │
├──────────────────────────────────────────┤
│          SERVICE LAYER                   │
│  Business Logic · Orchestration          │
│  Transaction Boundaries · DTO Mapping    │
├──────────────────────────────────────────┤
│          DOMAIN LAYER                    │
│  Entities · Value Objects · Aggregates   │
│  Domain Events · Repository Interfaces   │
├──────────────────────────────────────────┤
│       INFRASTRUCTURE LAYER               │
│  Prisma ORM · Redis · BullMQ            │
│  Elasticsearch · MinIO · External APIs   │
└──────────────────────────────────────────┘
```

### References

- `04-architecture/layered-architecture.md`
- `04-architecture/design-patterns.md`

---

## ADR-009: CQRS for High-Traffic Read Paths

**Status:** ACCEPTED
**Date:** 2026-09-01
**Deciders:** Architect, Tech Lead
**Related:** ADR-006, ADR-012

### Context

Product browsing (search, category pages, vendor stores) generates 10x more reads than writes. Shared read/write models cause contention and suboptimal query performance.

### Decision

Apply **CQRS (Command Query Responsibility Segregation)** selectively for high-traffic read paths: product catalog, storefront, and search. Write models use PostgreSQL; read models use Elasticsearch or PostgreSQL read replicas with optimized schemas.

### Consequences

**Positive:**
- Read queries optimized without affecting write performance
- Independent scaling of read and write workloads
- Read models can denormalize for query performance
- Search results served from Elasticsearch (not PostgreSQL)

**Negative:**
- Eventual consistency between read and write models
- Additional complexity for data synchronization
- More infrastructure components to manage

### CQRS Boundaries

| Domain | Write Model | Read Model | Sync Mechanism |
|--------|-------------|------------|----------------|
| Product catalog | PostgreSQL | Elasticsearch | CDC + BullMQ |
| Order history | PostgreSQL | PostgreSQL replica | pg replication |
| Search results | PostgreSQL | Elasticsearch | CDC + BullMQ |
| Analytics | PostgreSQL | Materialized views | Scheduled refresh |

### References

- `04-architecture/design-patterns.md`
- `04-architecture/technology-stack.md`

---

## ADR-010: Saga Pattern for Order Workflow

**Status:** ACCEPTED
**Date:** 2026-09-01
**Deciders:** Architect, Tech Lead
**Related:** ADR-015, ADR-004

### Context

Order processing spans multiple modules: inventory reservation, payment charging, escrow hold, delivery assignment, and notifications. A failure at any step must not leave the system in an inconsistent state.

### Decision

Use the **Saga Pattern** (choreography-based) for order workflow. Each step publishes events; downstream modules react and compensate if needed.

### Consequences

**Positive:**
- Each step is atomic within its module
- Compensating actions handle partial failures
- No distributed transactions (2PC) complexity
- Audit trail of each step via event log

**Negative:**
- Compensating logic can be complex (refund if payment charged but delivery fails)
- Debugging multi-step sagas requires good logging
- Eventual consistency across steps

### Order Saga Steps

```
1. OrderCreated → InventoryService.reserveStock()
2. StockReserved → PaymentService.chargeWallet()
3. PaymentCharged → EscrowService.placeHold()
4. EscrowHeld → DeliveryService.assignProvider()
5. DeliveryAssigned → NotificationService.sendConfirmation()
6. DeliveryConfirmed → (wait 7 days) → EscrowService.releaseHold()
7. EscrowReleased → FinanceService.processPayout()
```

### References

- `04-architecture/design-patterns.md`
- `10-SEQUENCING/01-order-flow.md`

---

## ADR-011: Repository Pattern for Data Access

**Status:** ACCEPTED
**Date:** 2026-09-01
**Deciders:** Architect, Backend Lead
**Related:** ADR-003, ADR-008

### Context

Direct Prisma calls scattered throughout service code creates tight coupling to the database. Testing requires database mocks. Swapping persistence strategies (e.g., adding caching layer) requires touching every service.

### Decision

Implement the **Repository Pattern**: domain defines repository interfaces; infrastructure provides Prisma implementations. Services depend only on interfaces.

### Consequences

**Positive:**
- Domain layer has zero database dependencies
- Easy to mock repositories for unit testing
- Can swap persistence implementations (in-memory for tests, Prisma for production)
- Consistent data access pattern across all modules

**Negative:**
- Additional abstraction layer (interface + implementation per entity)
- Potential over-abstraction for simple CRUD operations
- Mapper overhead between domain and persistence models

### Module Repository Map

| Module | Repositories |
|--------|-------------|
| B01 Auth | UserRepository, SessionRepository, AuditRepository |
| B02 Vendor | VendorRepository, StoreRepository, KYCRepository |
| B03 Catalog | ProductRepository, CategoryRepository, OfferRepository |
| B04 Order | OrderRepository, ReturnRepository |
| B05 Payment | PaymentRepository, WalletRepository, EscrowRepository |
| B06 Finance | CommissionRepository, PayoutRepository, InvoiceRepository |
| B07 Shipping | ShipmentRepository, RiderRepository, ZoneRepository |
| B08 Inventory | StockRepository, ReservationRepository, WarehouseRepository |
| B09 Storefront | CartRepository, WishlistRepository |
| B10 Trust | ReviewRepository, LoyaltyRepository, TierRepository |
| B11 Content | PageRepository, NotificationRepository |
| B12 Support | TicketRepository, BookingRepository, AnalyticsRepository |
| B13 Pricing | CouponRepository, DiscountRepository |

### References

- `04-architecture/design-patterns.md`
- `06-backend/data-access-layer.md`

---

## ADR-012: Domain Events for Cross-Module Communication

**Status:** ACCEPTED
**Date:** 2026-09-01
**Deciders:** Architect, Tech Lead
**Related:** ADR-004, ADR-010

### Context

Modules must react to changes in other modules (order placed → payment charged → notification sent → inventory updated). Direct module-to-module calls create tight coupling and circular dependencies.

### Decision

Use **Domain Events** published to BullMQ topics. Modules subscribe to events they care about; publishers have no knowledge of subscribers.

### Consequences

**Positive:**
- Modules are fully decpled at the code level
- New consumers can subscribe without modifying publishers
- Events serve as an audit log of all state changes
- Asynchronous processing improves response times

**Negative:**
- Event ordering not guaranteed across topics
- Debugging requires tracing event chains
- Event schema evolution requires versioning

### Key Domain Events

| Event | Publisher | Subscribers |
|-------|-----------|-------------|
| OrderCreated | B04 Order | B08 Inventory, B11 Notification |
| PaymentCharged | B05 Payment | B04 Order, B11 Notification |
| EscrowReleased | B05 Payment | B06 Finance, B11 Notification |
| StockReserved | B08 Inventory | B04 Order, B05 Payment |
| DeliveryConfirmed | B07 Shipping | B05 Payment, B11 Notification |
| VendorApproved | B02 Vendor | B09 Storefront, B11 Notification |
| ReviewSubmitted | B10 Trust | B02 Vendor, B11 Notification |

### References

- `04-architecture/design-patterns.md`
- `10-SEQUENCING/14-sequencing-interaction-flows.md`

---

## ADR-013: Circuit Breaker for External Integrations

**Status:** ACCEPTED
**Date:** 2026-09-01
**Deciders:** Tech Lead, Architect
**Related:** ADR-014

### Context

YemenMart integrates with external SMS providers, payment wallets, and delivery APIs. An outage in any external service should not cascade into platform-wide failures.

### Decision

Implement **Circuit Breaker** pattern for all external API calls using a configurable failure threshold and half-open recovery.

### Consequences

**Positive:**
- Prevents cascading failures from external outages
- Graceful degradation (SMS fails → queue for retry, don't block registration)
- Faster recovery when external service comes back online
- Metrics on external service health

**Negative:**
- Adds latency on circuit-open responses
- Requires monitoring to detect degraded state
- Half-open recovery logic adds complexity

### Circuit Breaker Configuration

```typescript
const circuitBreakerConfig = {
  failureThreshold: 5,        // Open after 5 consecutive failures
  recoveryTimeout: 30000,     // Try again after 30 seconds
  halfOpenMaxAttempts: 3,     // Test with 3 requests before closing
  monitorInterval: 10000,     // Check state every 10 seconds
};
```

### External Service Map

| Service | Provider | Circuit Breaker |
|---------|----------|----------------|
| SMS OTP | Twilio / Telesom / Sabafon | Yes |
| WhatsApp | WhatsApp Business API | Yes |
| Wallet Top-up | m-Floos, OneCash | Yes |
| Delivery | Local delivery providers | Yes |
| Email | SendGrid / Nodemailer | Yes |

### References

- `04-architecture/design-patterns.md`
- `10-integrations/integration-overview.md`

---

## ADR-014: Anti-Corruption Layer for Third Parties

**Status:** ACCEPTED
**Date:** 2026-09-01
**Deciders:** Architect
**Related:** ADR-013

### Context

External services (m-Floos, OneCash, Twilio) have their own data models and API contracts. Their changes should not propagate into YemenMart domain logic.

### Decision

Implement **Anti-Corruption Layer (ACL)** adapters that translate between external service models and YemenMart domain models.

### Consequences

**Positive:**
- External API changes require only ACL adapter updates
- Domain model remains clean and consistent
- Easy to swap external providers (e.g., Twilio → local SMS)
- External data validation isolated from business logic

**Negative:**
- Additional code per external integration
- Data transformation overhead
- ACL testing requires external service mocks

### ACL Adapter Map

| External Service | ACL Adapter | Domain Model |
|-----------------|-------------|--------------|
| m-Floos API | MFloosWalletAdapter | WalletTransaction |
| OneCash API | OneCashWalletAdapter | WalletTransaction |
| Twilio SMS | TwilioSmsAdapter | OtpDelivery |
| WhatsApp Business | WhatsAppAdapter | OtpDelivery |
| Telesom SMS | TelesomSmsAdapter | OtpDelivery |
| Delivery Provider | DeliveryProviderAdapter | Shipment |

### References

- `04-architecture/design-patterns.md`
- `10-integrations/third-party-apis.md`

---

## ADR-015: 17-State Order State Machine

**Status:** ACCEPTED
**Date:** 2026-09-01
**Deciders:** Tech Lead, Product Lead
**Related:** ADR-010

### Context

YemenMart orders involve multiple vendors, delivery providers, escrow holds, and return windows. A simple 5-6 state model cannot represent all lifecycle stages including partial cancellations, COD approval flows, and return processing.

### Decision

Implement a **17-state order state machine** covering the full order lifecycle from creation to completion (or cancellation/return).

### Consequences

**Positive:**
- Granular tracking of every order stage
- Clear transition rules prevent invalid state changes
- Supports complex scenarios: partial cancel, multi-vendor, returns
- Audit trail of every state transition

**Negative:**
- UI must handle 17 states with clear customer-facing labels
- More edge cases to test (17 states × N transitions)
- State machine complexity in code

### Order States

| # | State | Description |
|---|-------|-------------|
| 1 | PENDING | Order created, awaiting payment |
| 2 | PAYMENT_PROCESSING | Wallet charge in progress |
| 3 | PAID | Payment confirmed, awaiting inventory |
| 4 | STOCK_RESERVED | Inventory allocated |
| 5 | CONFIRMED | Order confirmed by system |
| 6 | VENDOR_ACCEPTED | Vendor accepted the order |
| 7 | PREPARING | Vendor preparing items |
| 8 | READY_FOR_PICKUP | Items ready for delivery |
| 9 | ASSIGNING_DELIVERY |寻找 delivery provider |
| 10 | IN_TRANSIT | Delivery in progress |
| 11 | DELIVERED | Items delivered to customer |
| 12 | ESCROW_HOLDING | 7-day escrow active |
| 13 | COMPLETED | Escrow released, order closed |
| 14 | CANCELLED_BY_CUSTOMER | Customer cancelled |
| 15 | CANCELLED_BY_VENDOR | Vendor cancelled |
| 16 | RETURN_REQUESTED | Customer requested return |
| 17 | RETURN_COMPLETED | Return processed, refund issued |

### References

- `04-architecture/design-patterns.md`
- `10-SEQUENCING/01-order-flow.md`

---

## Decision Review Schedule

| Frequency | Action |
|-----------|--------|
| Monthly | Review new ADRs for consistency |
| Quarterly | Audit all accepted ADRs for relevance |
| On major release | Validate ADRs against production reality |
| On incident | Check if incident relates to an ADR |

---

## Related Categories

- `04-architecture` - Architecture documentation
- `04-architecture/design-patterns.md` - Detailed pattern implementations
- `04-architecture/technology-stack.md` - Technology selections

---

*Source: Architecture decisions from design process, constraint analysis, and team evaluation*
