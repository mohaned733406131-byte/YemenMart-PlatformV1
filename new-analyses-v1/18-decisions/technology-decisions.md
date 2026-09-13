# Technology Selection Rationale — YemenMart

**Document ID:** YM-TECH-001
**Version:** 1.0
**Status:** Active
**Date:** 2026-09-13
**Language:** English

---

## Table of Contents

1. [Decision Framework](#1-decision-framework)
2. [Runtime & Language](#2-runtime--language)
3. [Backend Framework](#3-backend-framework)
4. [Database & ORM](#4-database--orm)
5. [Cache & Queue](#5-cache--queue)
6. [Search Engine](#6-search-engine)
7. [Object Storage](#7-object-storage)
8. [Frontend Stack](#8-frontend-stack)
9. [Mobile](#9-mobile)
10. [Testing Tools](#10-testing-tools)
11. [DevOps & Infrastructure](#11-devops--infrastructure)
12. [Third-Party Services](#12-third-party-services)
13. [Technology Risk Assessment](#13-technology-risk-assessment)

---

## 1. Decision Framework

### 1.1 Evaluation Criteria

| Criterion | Weight | Description |
|-----------|--------|-------------|
| Team familiarity | 25% | Team already knows or can learn quickly |
| Community & ecosystem | 20% | Active community, plugins, documentation |
| Long-term viability | 20% | Backing company/community, release cadence |
| Performance | 15% | Meets latency/throughput requirements |
| Cost | 10% | Licensing, hosting, operational costs |
| Yemen market fit | 10% | RTL support, low-bandwidth, mobile-first |

### 1.2 Scoring Method

```
Score (1-5) × Weight = Weighted Score
Total = Sum of weighted scores
Threshold: ≥ 3.5 to accept
```

---

## 2. Runtime & Language

### 2.1 Node.js 20 LTS

| Criterion | Score | Rationale |
|-----------|-------|-----------|
| Team familiarity | 5 | Team is TypeScript-first |
| Community | 5 | Largest JS ecosystem |
| Performance | 4 | Async I/O ideal for API servers |
| Viability | 5 | LTS release cycle, OpenJS Foundation |
| Cost | 5 | Free, open source |
| Market fit | 4 | Great for mobile API backends |

**Total: 4.8/5** — ACCEPTED

**Alternatives Considered:**

| Option | Score | Why Rejected |
|--------|-------|-------------|
| Deno | 3.2 | Smaller ecosystem, less mature |
| Bun | 3.0 | Not LTS, immature production tooling |
| Go | 3.8 | Team would need reskilling |
| Python | 3.5 | Slower for I/O-heavy workloads |

### 2.2 TypeScript 5.x

| Criterion | Score | Rationale |
|-----------|-------|-----------|
| Team familiarity | 5 | Primary language for the team |
| Performance | 4 | Compile-time type checking catches bugs early |
| Viability | 5 | Backed by Microsoft, rapid evolution |
| Market fit | 5 | Essential for React/Next.js integration |

**Total: 4.8/5** — ACCEPTED

### Configuration

```json
{
  "compilerOptions": {
    "strict": true,
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "skipLibCheck": true,
    "incremental": true,
    "declaration": true,
    "sourceMap": true
  }
}
```

---

## 3. Backend Framework

### 3.1 Express.js 4.x (Admin & Webhooks)

| Criterion | Score | Rationale |
|-----------|-------|-----------|
| Team familiarity | 5 | Most widely used Node.js framework |
| Community | 5 | Massive middleware ecosystem |
| Performance | 3 | Moderate throughput |
| Market fit | 5 | Industry standard |

**Total: 4.6/5** — ACCEPTED for admin APIs and webhook endpoints

### 3.2 Fastify 4.x (High-Throughput APIs)

| Criterion | Score | Rationale |
|-----------|-------|-----------|
| Team familiarity | 4 | Requires learning, but familiar API |
| Community | 4 | Growing, strong plugin system |
| Performance | 5 | 2-3x faster than Express |
| Market fit | 4 | Ideal for customer-facing APIs |

**Total: 4.4/5** — ACCEPTED for customer/vendor-facing APIs

### Framework Selection Logic

| API Type | Framework | Rationale |
|----------|-----------|-----------|
| Customer API (high traffic) | Fastify | Performance critical |
| Vendor API (medium traffic) | Fastify | Consistent with customer API |
| Admin API (low traffic) | Express | Mature ecosystem, easier middleware |
| Webhooks (external) | Express | Simplicity for callback handlers |

---

## 4. Database & ORM

### 4.1 PostgreSQL 15

| Criterion | Score | Rationale |
|-----------|-------|-----------|
| Team familiarity | 4 | Team has relational DB experience |
| Community | 5 | Largest open-source RDBMS community |
| Performance | 4 | Handles YemenMart scale requirements |
| Viability | 5 | 30+ year track record |
| Cost | 5 | Free, managed options available |
| Market fit | 5 | ACID for financial, JSONB for flexibility |

**Total: 4.8/5** — ACCEPTED

**Key Features Used:**

| Feature | Use Case |
|---------|----------|
| JSONB | Product attributes, metadata |
| Full-text search | Arabic + English product search |
| Table partitioning | order_items, audit_logs by month |
| Materialized views | Analytics dashboards |
| Row-level security | Multi-tenant vendor data isolation |
| Logical replication | Read replicas for reporting |

### 4.2 Prisma 5.x

| Criterion | Score | Rationale |
|-----------|-------|-----------|
| Team familiarity | 4 | Learning curve but strong TypeScript DX |
| Community | 5 | Most popular TypeScript ORM |
| Performance | 3 | Overhead vs raw SQL, but acceptable |
| Viability | 5 | Active development, strong funding |
| Cost | 5 | Free for open source use |
| Market fit | 4 | Excellent for PostgreSQL |

**Total: 4.5/5** — ACCEPTED

**Alternatives Considered:**

| Option | Score | Why Rejected |
|--------|-------|-------------|
| TypeORM | 3.8 | Weaker TypeScript, less active maintenance |
| Sequelize | 3.2 | Poor TypeScript support |
| Knex.js | 3.5 | Query builder only, no schema management |
| Drizzle ORM | 3.8 | Newer, smaller ecosystem |

---

## 5. Cache & Queue

### 5.1 Redis 7+

| Criterion | Score | Rationale |
|-----------|-------|-----------|
| Team familiarity | 4 | Widely known, simple API |
| Community | 5 | Industry standard cache |
| Performance | 5 | Sub-millisecond latency |
| Viability | 5 | 15+ year track record |
| Cost | 5 | Free, managed options available |

**Total: 4.8/5** — ACCEPTED for caching, sessions, rate limiting

### 5.2 BullMQ 5.x

| Criterion | Score | Rationale |
|-----------|-------|-----------|
| Team familiarity | 3 | New for team, but well-documented |
| Community | 4 | Active development, good docs |
| Performance | 5 | Built on Redis, high throughput |
| Viability | 4 | Active maintenance |
| Cost | 5 | Free, open source |

**Total: 4.4/5** — ACCEPTED for job queues and event bus

**Alternatives Considered:**

| Option | Score | Why Rejected |
|--------|-------|-------------|
| RabbitMQ | 3.5 | Separate infrastructure, more ops overhead |
| Apache Kafka | 3.0 | Overkill for current scale |
| Redis Streams | 3.8 | Lower-level API, less job management |

---

## 6. Search Engine

### 6.1 Elasticsearch 8

| Criterion | Score | Rationale |
|-----------|-------|-----------|
| Team familiarity | 3 | Requires learning DSL |
| Community | 5 | Industry standard search |
| Performance | 5 | Sub-100ms for complex queries |
| Viability | 5 | Elastic company backing |
| Cost | 4 | Free self-managed, paid cloud |
| Market fit | 5 | Arabic analyzer, faceted search |

**Total: 4.6/5** — ACCEPTED

**Key Capabilities:**

| Capability | YemenMart Use |
|------------|--------------|
| Arabic analyzer | Product search in Arabic |
| Fuzzy matching | Typo tolerance |
| Faceted search | Category, price, rating filters |
| Geolocation | Nearby vendor/store search |
| Autocomplete | Search suggestions |

**Alternatives Considered:**

| Option | Score | Why Rejected |
|--------|-------|-------------|
| PostgreSQL FTS | 3.5 | Lacks faceted search, fuzzy matching |
| Meilisearch | 3.8 | Newer, less production maturity |
| Algolia | 3.0 | Vendor lock-in, cost |

---

## 7. Object Storage

### 7.1 MinIO

| Criterion | Score | Rationale |
|-----------|-------|-----------|
| Team familiarity | 3 | New, but S3-compatible API |
| Community | 4 | Active open-source community |
| Performance | 4 | Good for image serving |
| Viability | 4 | Strong Kubernetes integration |
| Cost | 5 | Free, self-hosted |

**Total: 4.2/5** — ACCEPTED

**Alternatives Considered:**

| Option | Score | Why Rejected |
|--------|-------|-------------|
| AWS S3 | 4.0 | Vendor lock-in, egress costs |
| Cloudflare R2 | 3.8 | Newer, less control |
| Local filesystem | 2.5 | No replication, not scalable |

---

## 8. Frontend Stack

### 8.1 Next.js 15 (Customer Storefront)

| Criterion | Score | Rationale |
|-----------|-------|-----------|
| Team familiarity | 4 | React experience transfers |
| Community | 5 | Largest React meta-framework |
| Performance | 5 | SSR/SSG for SEO, automatic code splitting |
| Viability | 5 | Vercel backing, rapid development |
| Market fit | 5 | SEO critical for e-commerce |

**Total: 4.8/5** — ACCEPTED

**Key Features Used:**

| Feature | Use Case |
|---------|----------|
| SSR | Product pages for SEO |
| SSG | Category pages, CMS content |
| Image optimization | Product images |
| API routes | Lightweight BFF endpoints |
| Middleware | Auth checks, redirects |

### 8.2 React 18 + Vite (Vendor/Admin Portals)

| Criterion | Score | Rationale |
|-----------|-------|-----------|
| Team familiarity | 5 | Primary frontend expertise |
| Community | 5 | Largest frontend ecosystem |
| Performance | 5 | Vite instant HMR, optimized builds |
| Viability | 5 | Industry standard |
| Market fit | 4 | SPA appropriate for admin/vendor |

**Total: 4.8/5** — ACCEPTED

### 8.3 Tailwind CSS

| Criterion | Score | Rationale |
|-----------|-------|-----------|
| Team familiarity | 4 | Utility-first learning curve |
| Community | 5 | Fastest-growing CSS framework |
| Performance | 5 | Purged CSS, minimal bundle |
| Market fit | 4 | RTL support via plugin |

**Total: 4.6/5** — ACCEPTED

**Alternatives Considered:**

| Option | Score | Why Rejected |
|--------|-------|-------------|
| Bootstrap | 3.5 | Heavy, less customizable |
| Material UI | 3.8 | Opinionated design, harder to customize |
| Chakra UI | 3.5 | Smaller ecosystem |

---

## 9. Mobile

### 9.1 React Native 0.74+

| Criterion | Score | Rationale |
|-----------|-------|-----------|
| Team familiarity | 4 | React experience transfers |
| Community | 5 | Largest cross-platform mobile framework |
| Performance | 4 | Near-native with Hermes engine |
| Viability | 5 | Meta backing, massive adoption |
| Cost | 5 | Single codebase for iOS + Android |
| Market fit | 5 | Essential for Yemen mobile-first market |

**Total: 4.6/5** — ACCEPTED

**Key Features:**

| Feature | Use Case |
|---------|----------|
| Hermes engine | Faster startup on low-end devices |
| React Navigation | Screen navigation |
| Async Storage | Local data persistence |
| Push notifications | Order updates, promotions |
| Camera integration | Product image upload |

**Alternatives Considered:**

| Option | Score | Why Rejected |
|--------|-------|-------------|
| Flutter | 3.8 | Dart learning curve, smaller team familiarity |
| Native (Swift/Kotlin) | 3.0 | Double development cost |
| Ionic | 2.5 | Web-based, poor performance |

---

## 10. Testing Tools

### 10.1 Jest (Unit Testing)

| Criterion | Score | Rationale |
|-----------|-------|-----------|
| Team familiarity | 5 | Industry standard |
| Community | 5 | Largest test framework ecosystem |
| Performance | 4 | Parallel execution, fast |
| Market fit | 5 | TypeScript native |

**Total: 4.8/5** — ACCEPTED

### 10.2 Playwright (E2E Testing)

| Criterion | Score | Rationale |
|-----------|-------|-----------|
| Team familiarity | 3 | Learning curve for team |
| Community | 4 | Growing rapidly |
| Performance | 5 | Multi-browser parallel execution |
| Market fit | 5 | Full browser automation |

**Total: 4.4/5** — ACCEPTED

### 10.3 k6 (Performance Testing)

| Criterion | Score | Rationale |
|-----------|-------|-----------|
| Team familiarity | 3 | Requires learning |
| Community | 4 | Grafana-backed |
| Performance | 5 | Designed for load testing |
| Market fit | 4 | API load testing |

**Total: 4.2/5** — ACCEPTED

**Alternatives Considered:**

| Option | Score | Why Rejected |
|--------|-------|-------------|
| Mocha | 3.5 | Less batteries-included |
| Cypress | 3.8 | Single-browser limitation |
| JMeter | 3.0 | Java-based, heavier |

---

## 11. DevOps & Infrastructure

### 11.1 Docker + Docker Compose

| Criterion | Score | Rationale |
|-----------|-------|-----------|
| Team familiarity | 4 | Widely used |
| Community | 5 | Industry standard containers |
| Performance | 4 | Lightweight virtualization |
| Cost | 5 | Free |

**Total: 4.6/5** — ACCEPTED for local development and CI

### 11.2 Kubernetes (Production)

| Criterion | Score | Rationale |
|-----------|-------|-----------|
| Team familiarity | 3 | Learning curve |
| Community | 5 | Cloud-native standard |
| Performance | 5 | Auto-scaling, self-healing |
| Viability | 5 | CNCF backed |
| Cost | 4 | Managed options available |

**Total: 4.4/5** — ACCEPTED for production orchestration

### 11.3 GitHub Actions (CI/CD)

| Criterion | Score | Rationale |
|-----------|-------|-----------|
| Team familiarity | 4 | GitHub-based workflow |
| Community | 5 | Largest marketplace |
| Performance | 4 | Adequate for project size |
| Cost | 5 | Free for public, generous free tier |

**Total: 4.6/5** — ACCEPTED

### 11.4 Prometheus + Grafana (Monitoring)

| Criterion | Score | Rationale |
|-----------|-------|-----------|
| Team familiarity | 3 | Learning curve |
| Community | 5 | Industry standard monitoring |
| Performance | 5 | Handles high cardinality metrics |
| Cost | 5 | Free, open source |

**Total: 4.6/5** — ACCEPTED

---

## 12. Third-Party Services

### 12.1 SMS Providers

| Provider | Use Case | Rationale |
|----------|----------|-----------|
| Twilio | Primary SMS | Global reliability, API quality |
| Telesom | Local fallback | Yemen local carrier, lower cost |
| Sabafon | Local fallback | Yemen local carrier, wider coverage |

### 12.2 Payment Wallets

| Provider | Use Case | Rationale |
|----------|----------|-----------|
| m-Floos | Primary wallet | Largest Yemen mobile wallet |
| OneCash | Secondary wallet | Growing market share |

### 12.3 Communication

| Provider | Use Case | Rationale |
|----------|----------|-----------|
| WhatsApp Business | OTP + notifications | High penetration in Yemen |
| SendGrid | Transactional email | Reliable delivery |

### 12.4 Infrastructure

| Service | Use Case | Rationale |
|---------|----------|-----------|
| Cloudflare | CDN + DDoS protection | Performance + security |
| GitHub | Source control + CI | Team already uses |
| Let's Encrypt | SSL certificates | Free, automated |

---

## 13. Technology Risk Assessment

| Technology | Risk Level | Risk Description | Mitigation |
|-----------|------------|-----------------|------------|
| Prisma | Medium | Newer ORM, potential bugs | Stay on stable versions, monitor issues |
| BullMQ | Low | Redis dependency | Redis is battle-tested |
| React Native | Medium | Mobile-specific bugs | Thorough device testing |
| Elasticsearch | Low | Complex operations | Use managed service in production |
| MinIO | Medium | Self-managed storage | Regular backups, monitoring |
| Kubernetes | High | Operational complexity | Start with managed K8s (EKS/GKE) |
| Fastify | Low | Smaller community than Express | Well-documented, growing fast |

---

## Technology Decision Log

| Date | Decision | Rationale | ADR |
|------|----------|-----------|-----|
| 2026-09-01 | Node.js 20 LTS | Team expertise, async I/O | ADR-001 |
| 2026-09-01 | TypeScript 5.x | Type safety, team familiarity | ADR-001 |
| 2026-09-01 | PostgreSQL 15 | ACID, JSONB, FTS | ADR-002 |
| 2026-09-01 | Prisma 5.x | TypeScript-first ORM | ADR-003 |
| 2026-09-01 | Redis 7+ + BullMQ | Cache + queue | ADR-004, ADR-005 |
| 2026-09-01 | Elasticsearch 8 | Product search | ADR-006 |
| 2026-09-01 | MinIO | Self-hosted object storage | ADR-007 |
| 2026-09-01 | Next.js 15 + React 18 | Frontend stack | ADR-006 |
| 2026-09-01 | React Native | Cross-platform mobile | ADR-006 |
| 2026-09-01 | Docker + K8s | Container orchestration | ADR-001 |

---

## Related Categories

- `04-architecture/technology-stack.md` - Stack configuration details
- `04-architecture/architecture-overview.md` - Architecture decisions
- `18-decisions/architecture-decisions.md` - Architecture ADRs

---

*Source: Technology evaluation from market research, team capabilities, and project requirements*
