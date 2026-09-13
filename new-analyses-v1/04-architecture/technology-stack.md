# Technology Stack — YemenMart

## 1. Stack Overview

| Layer | Technology | Version | Purpose |
|---|---|---|---|
| Runtime | Node.js | 20 LTS | Server-side JavaScript runtime |
| Language | TypeScript | 5.x | Type-safe development |
| Backend Framework | Express.js / Fastify | 4.x / 4.x | HTTP API server |
| Frontend (Vendor) | React + Vite | 18.x / 5.x | Vendor portal SPA |
| Frontend (Customer) | Next.js | 15.x | Customer-facing SSR/SSG |
| Mobile | React Native | 0.74+ | iOS and Android apps |
| Database | PostgreSQL | 15 | Primary relational store |
| Cache | Redis | 7+ | Caching, sessions, pub/sub |
| Queue | BullMQ | 5.x | Job queues, event bus |
| Search | Elasticsearch | 8.x | Full-text and faceted search |
| Object Storage | MinIO | Latest | S3-compatible file storage |
| ORM | Prisma | 5.x | Database access and migrations |
| Validation | Zod | 3.x | Runtime schema validation |
| Auth | JWT + Refresh Tokens | — | Authentication and sessions |
| Payment | Stripe / Moyasar | — | Online payment processing |
| SMS | Twilio / Local Provider | — | OTP and notifications |
| Email | Nodemailer / SendGrid | — | Transactional email |
| Monitoring | Prometheus + Grafana | — | Metrics and dashboards |
| Logging | Pino | 8.x | Structured JSON logging |
| Tracing | OpenTelemetry | 1.x | Distributed tracing |
| Testing | Jest + Playwright + k6 | — | Unit, E2E, performance |
| CI/CD | GitHub Actions | — | Build, test, deploy |
| Containers | Docker + Docker Compose | — | Local dev and deployment |
| Orchestration | Kubernetes (prod) | 1.28+ | Container orchestration |

## 2. Backend Stack Details

### 2.1 Runtime & Language

```json
{
  "engines": {
    "node": ">=20.0.0",
    "typescript": "5.x"
  }
}
```

- **Node.js 20 LTS**: Long-term support, stable performance, native ESM support
- **TypeScript 5.x**: Strict mode enabled, path aliases, incremental compilation

### 2.2 Framework Selection

| Option | When Used | Rationale |
|---|---|---|
| **Express.js** | Admin APIs, webhooks | Mature ecosystem, middleware availability |
| **Fastify** | High-throughput internal APIs | 2-3x faster than Express, schema-based validation |

### 2.3 Database

```
PostgreSQL 15
├── Primary (Read/Write)
├── Replica 1 (Read-only, reporting)
├── Replica 2 (Read-only, search sync)
└── Connection Pool: pgBouncer (max 100 connections)
```

**Key Features Used:**
- JSONB columns for flexible metadata
- Full-text search with `tsvector` (Arabic + English)
- Partial indexes for soft-deleted records
- Partitioning for `order_items` by month
- Materialized views for analytics dashboards

### 2.4 Cache Strategy (Redis 7+)

| Cache Layer | TTL | Key Pattern | Invalidation |
|---|---|---|---|
| Session store | 24h | `session:{userId}` | Logout / expiry |
| Product cache | 1h | `product:{id}` | Write-through on update |
| Cart cache | 30m | `cart:{userId}` | Checkout / manual clear |
| Rate limiter | 1m | `rate:{ip}:{endpoint}` | Sliding window |
| Search results | 5m | `search:{queryHash}` | TTL-based |
| Inventory locks | 10m | `lock:stock:{sku}` | Reserve / release |

### 2.5 Queue System (BullMQ)

```typescript
// Queue definitions
const queues = {
  'order.events':       { concurrency: 10, priority: 'high' },
  'payment.process':    { concurrency: 5,  priority: 'critical' },
  'notification.send':  { concurrency: 20, priority: 'normal' },
  'inventory.sync':     { concurrency: 15, priority: 'high' },
  'search.reindex':     { concurrency: 5,  priority: 'low' },
  'analytics.track':    { concurrency: 50, priority: 'low' },
  'delivery.assign':    { concurrency: 10, priority: 'high' },
  'payout.process':     { concurrency: 3,  priority: 'critical' },
};
```

### 2.6 Search Engine (Elasticsearch 8)

**Indices:**

| Index | Purpose | Refresh Interval |
|---|---|---|
| `products` | Product catalog search | 1s |
| `vendors` | Vendor/store search | 5s |
| `orders` | Order lookup (admin) | 5s |
| `reviews` | Review search | 10s |
| `knowledge_base` | FAQ/support articles | 30s |

**Analyzer Configuration:**
- Arabic analyzer with stemmer and stop words
- Edge n-gram for partial matching
- Custom synonyms for product categories

## 3. Frontend Stack Details

### 3.1 Customer Storefront — Next.js 15

| Concern | Technology |
|---|---|
| Rendering | SSR (dynamic), ISR (catalog), SSG (landing) |
| State Management | Zustand (client), React Query (server) |
| Styling | Tailwind CSS 3.x + Headless UI |
| Forms | React Hook Form + Zod |
| i18n | next-intl (Arabic RTL, English LTR) |
| Analytics | PostHog / Plausible |
| Payments | Stripe Elements / Moyasar SDK |

### 3.2 Vendor Portal — React + Vite

| Concern | Technology |
|---|---|
| Build Tool | Vite 5.x |
| Routing | React Router 6.x |
| State Management | Zustand + React Query |
| UI Library | shadcn/ui + Radix primitives |
| Charts | Recharts / Tremor |
| Tables | TanStack Table |
| Forms | React Hook Form + Zod |

### 3.3 Mobile — React Native

| Concern | Technology |
|---|---|
| Framework | React Native 0.74+ |
| Navigation | React Navigation 6 |
| State | Zustand + React Query |
| UI | React Native Paper |
| Push | Firebase Cloud Messaging |
| Deep Links | Universal Links / App Links |
| OTA Updates | Expo Updates (if Expo) |

## 4. Infrastructure Stack

### 4.1 Container & Orchestration

```yaml
# Docker Compose (Development)
services:
  app:          # Node.js application
  postgres:     # PostgreSQL 15
  redis:        # Redis 7
  elasticsearch: # Elasticsearch 8
  minio:        # MinIO object storage
  bullmq:       # BullMQ Dashboard (Bull Board)
```

### 4.2 CI/CD Pipeline

```
┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐
│  Push    │──▶│  Lint &  │──▶│  Unit    │──▶│  Build   │──▶│  Deploy  │
│  to main │   │  TypeCheck│  │  Tests   │   │  Docker  │   │  to K8s  │
└──────────┘   └──────────┘   └──────────┘   └──────────┘   └──────────┘
                                  │
                                  ▼
                           ┌──────────┐
                           │  E2E     │
                           │  Tests   │
                           │(Playwright)│
                           └──────────┘
                                  │
                                  ▼
                           ┌──────────┐
                           │  k6      │
                           │  Perf.   │
                           │  Tests   │
                           └──────────┘
```

### 4.3 Monitoring Stack

| Tool | Purpose | Retention |
|---|---|---|
| Prometheus | Metrics collection | 30 days |
| Grafana | Dashboards & alerting | — |
| Loki | Log aggregation | 14 days |
| Tempo | Distributed tracing | 7 days |
| PagerDuty | Alert escalation | — |

## 5. Testing Stack

| Level | Tool | Coverage Target |
|---|---|---|
| Unit | Jest + Vitest | 80%+ |
| Integration | Jest + Supertest | Critical paths |
| E2E | Playwright | User journeys |
| Performance | k6 | < 200ms p95 API |
| Mobile | Detox / Maestro | Core flows |
| Security | OWASP ZAP | Quarterly scans |

## 6. Development Tools

| Tool | Purpose |
|---|---|
| ESLint | Code linting |
| Prettier | Code formatting |
| Husky | Git hooks |
| lint-staged | Pre-commit checks |
| Prisma Studio | Database GUI |
| Bull Board | Queue dashboard |
| TypeDoc | API documentation |
| Storybook | Component library |
