# Architecture Overview — YemenMart

## 1. Architectural Style

YemenMart adopts a **Modular Monolith** architecture. A single deployable unit houses all business modules, communicating through well-defined internal interfaces and domain events. This provides the simplicity of a monolith while preserving the option to extract services later.

```
┌─────────────────────────────────────────────────────────────────────┐
│                        YEMENMART PLATFORM                          │
│                                                                     │
│  ┌───────────┐ ┌───────────┐ ┌───────────┐ ┌───────────┐          │
│  │  B01 Auth │ │B02 Vendor │ │B03 Catalog│ │B04 Order  │          │
│  └───────────┘ └───────────┘ └───────────┘ └───────────┘          │
│  ┌───────────┐ ┌───────────┐ ┌───────────┐ ┌───────────┐          │
│  │B05 Payment│ │B06 Finance│ │B07 Ship   │ │B08 Inv.   │          │
│  └───────────┘ └───────────┘ └───────────┘ └───────────┘          │
│  ┌───────────┐ ┌───────────┐ ┌───────────┐ ┌───────────┐          │
│  │B09 Store  │ │B10 Trust  │ │B11 Content│ │B12 Support│          │
│  └───────────┘ └───────────┘ └───────────┘ └───────────┘          │
│  ┌───────────┐                                                      │
│  │B13 Pricing│                                                      │
│  └───────────┘                                                      │
│                                                                     │
│  ═══════════════ Domain Event Bus (BullMQ) ════════════════        │
└─────────────────────────────────────────────────────────────────────┘
```

## 2. Core Principles

| Principle | Description |
|---|---|
| API-First | Every capability is exposed through a versioned REST/GraphQL contract before implementation |
| Event-Driven | Cross-module communication uses domain events via BullMQ; modules never call each other's repositories |
| Single Responsibility | Each module owns one bounded context and its data |
| Dependency Inversion | Domain layer depends on nothing; infrastructure implements interfaces defined in domain |
| Explicit Boundaries | Inter-module data access goes through service interfaces, not shared tables |

## 3. Layered Architecture

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

## 4. C4 Container Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          YemenMart Platform                                  │
│                                                                             │
│  External Users                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐                     │
│  │   Customer   │  │    Vendor    │  │    Admin     │                     │
│  │   (Browser)  │  │   (Browser)  │  │   (Browser)  │                     │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘                     │
│         │                  │                  │                              │
│         ▼                  ▼                  ▼                              │
│  ┌─────────────────────────────────────────────────────┐                   │
│  │              Next.js 15 (Customer SPA)               │                   │
│  │         React 18 + Vite (Vendor Portal)              │                   │
│  │         React Native (Mobile Apps)                   │                   │
│  └──────────────────────┬──────────────────────────────┘                   │
│                          │ HTTPS / WSS                                      │
│                          ▼                                                  │
│  ┌─────────────────────────────────────────────────────┐                   │
│  │          API Gateway (Express/Fastify)               │                   │
│  │    Auth · Rate Limit · CORS · Request Validation     │                   │
│  └──────────────────────┬──────────────────────────────┘                   │
│                          │                                                  │
│  ┌──────────────────────┼──────────────────────────────┐                   │
│  │              YemenMart Modular Monolith              │                   │
│  │                                                      │                   │
│  │  ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ │                   │
│  │  │ B01 │ │ B02 │ │ B03 │ │ B04 │ │ B05 │ │ B06 │ │                   │
│  │  │Auth │ │Vend.│ │Cat. │ │Ord. │ │Pay. │ │Fin. │ │                   │
│  │  └─────┘ └─────┘ └─────┘ └─────┘ └─────┘ └─────┘ │                   │
│  │  ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ │                   │
│  │  │ B07 │ │ B08 │ │ B09 │ │ B10 │ │ B11 │ │ B12 │ │                   │
│  │  │Ship │ │Inv. │ │Store│ │Trust│ │CMS  │ │Supp.│ │                   │
│  │  └─────┘ └─────┘ └─────┘ └─────┘ └─────┘ └─────┘ │                   │
│  │  ┌─────┐                                            │                   │
│  │  │ B13 │                                            │                   │
│  │  │Price│                                            │                   │
│  │  └─────┘                                            │                   │
│  └─────────────────────────────────────────────────────┘                   │
│         │            │            │            │                             │
│         ▼            ▼            ▼            ▼                             │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐                      │
│  │PostgreSQL│ │  Redis 7 │ │ Elastic- │ │  MinIO   │                      │
│  │   15     │ │ (Cache + │ │ search 8 │ │  (S3)   │                      │
│  │          │ │  Queue)  │ │          │ │          │                      │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘                      │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 5. Module Communication

| Pattern | Mechanism | Use Case |
|---|---|---|
| Synchronous | In-process service method calls | Same-request reads within a module |
| Asynchronous | Domain events via BullMQ | Cross-module state changes (e.g., order.placed → inventory.reserve) |
| Query | Repository read models | Fetching data across module boundaries |
| Shared Events | Event store + projection | Analytics, reporting, audit trails |

## 6. Module Dependency Rules

1. **Domain** never imports from Infrastructure, Service, or API layers.
2. **Service** imports only from Domain; uses Infrastructure through interfaces.
3. **API** imports from Service and Domain; never from Infrastructure directly.
4. **Presentation** communicates exclusively via HTTP/WS to the API layer.
5. **Cross-module** access: Module A calls `ModuleBService.method()` — never `ModuleBRepository.find()`.

## 7. Deployment Topology

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   CDN Edge   │────▶│  Load Bal.   │────▶│  App Server  │
│  (CloudFlare)│     │  (Nginx/ALB) │     │  (Node.js)   │
└──────────────┘     └──────────────┘     └──────┬───────┘
                                                  │
                     ┌────────────────────────────┼──────────────┐
                     │                            │              │
                ┌────▼─────┐  ┌───────────┐  ┌───▼────┐  ┌─────▼────┐
                │PostgreSQL│  │   Redis   │  │Elastic │  │  MinIO   │
                │ Primary  │  │  Cluster  │  │ search │  │   S3     │
                │ + Replica│  │           │  │ Cluster│  │          │
                └──────────┘  └───────────┘  └────────┘  └──────────┘
```

## 8. Key Architectural Decisions

| ID | Decision | Rationale |
|---|---|---|
| ADR-01 | Modular monolith over microservices | Team size < 10; operational simplicity; same deploy benefits |
| ADR-02 | BullMQ over RabbitMQ | Redis-based; simpler ops; Lua scripting for reliable jobs |
| ADR-03 | Prisma over TypeORM | Type safety; migration management; first-class Next.js support |
| ADR-04 | Elasticsearch over PostgreSQL full-text | Arabic text support; faceted search; relevance tuning |
| ADR-05 | MinIO over AWS S3 | Self-hosted; S3-compatible; cost control |
| ADR-06 | CQRS for B04 Orders | Read/write separation for high-throughput order queries |
