# System Context Diagram — YemenMart Platform

## 1. Overview

C4 Level 1 context diagram showing YemenMart as the central system, all human actors, and external system integrations.

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                              EXTERNAL ACTORS                                       │
│                                                                                     │
│   ┌───────────┐   ┌───────────┐   ┌───────────┐   ┌───────────┐   ┌───────────┐  │
│   │ Customer  │   │  Vendor   │   │   Admin   │   │ Delivery  │   │  Support  │  │
│   │           │   │           │   │           │   │ Provider  │   │  Agent    │  │
│   └─────┬─────┘   └─────┬─────┘   └─────┬─────┘   └─────┬─────┘   └─────┬─────┘  │
│         │               │               │               │               │          │
│         │  Browse/Order │  Manage Store │  Oversee Ops  │  Deliver/POD │  Resolve │
│         │  Pay/Review   │  KYC/Products │  Approve/Flag │  Track Zones │  Tickets │
└─────────┼───────────────┼───────────────┼───────────────┼───────────────┼──────────┘
          │               │               │               │               │
          ▼               ▼               ▼               ▼               ▼
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                                                                                     │
│                      ╔═══════════════════════════════╗                              │
│                      ║     YEMENMART PLATFORM        ║                              │
│                      ║                               ║                              │
│                      ║  ┌─────────┐  ┌─────────┐    ║                              │
│                      ║  │  B01    │  │  B02    │    ║                              │
│                      ║  │Identity │←→│Marketpl.│    ║                              │
│                      ║  └────┬────┘  └────┬────┘    ║                              │
│                      ║       │            │         ║                              │
│                      ║  ┌────▼────┐  ┌────▼────┐    ║                              │
│                      ║  │  B03    │  │  B04    │    ║                              │
│                      ║  │Catalog  │←→│Commerce │    ║                              │
│                      ║  └────┬────┘  └────┬────┘    ║                              │
│                      ║       │            │         ║                              │
│                      ║  ┌────▼────┐  ┌────▼────┐    ║                              │
│                      ║  │  B05    │  │  B06    │    ║                              │
│                      ║  │ Payment │←→│ Finance │    ║                              │
│                      ║  └────┬────┘  └────┬────┘    ║                              │
│                      ║       │            │         ║                              │
│                      ║  ┌────▼────┐  ┌────▼────┐    ║                              │
│                      ║  │  B07    │  │  B08    │    ║                              │
│                      ║  │Logistic │←→│Inventory│    ║                              │
│                      ║  └────┬────┘  └────┬────┘    ║                              │
│                      ║       │            │         ║                              │
│                      ║  ┌────▼────┐  ┌────▼────┐    ║                              │
│                      ║  │  B09    │  │  B10    │    ║                              │
│                      ║  │Storefrt.│←→│Engagemnt│    ║                              │
│                      ║  └────┬────┘  └────┬────┘    ║                              │
│                      ║       │            │         ║                              │
│                      ║  ┌────▼────┐  ┌────▼────┐    ║                              │
│                      ║  │  B11    │  │  B12    │    ║                              │
│                      ║  │Content  │←→│Support  │    ║                              │
│                      ║  └────┬────┘  └────┬────┘    ║                              │
│                      ║       │            │         ║                              │
│                      ║  ┌────▼────┐                 ║                              │
│                      ║  │  B13    │                 ║                              │
│                      ║  │Promotion│                 ║                              │
│                      ║  └─────────┘                 ║                              │
│                      ╚═══════════════════════════════╝                              │
│                                  │                                                 │
└──────────────────────────────────┼─────────────────────────────────────────────────┘
                                   │
          ┌────────────────────────┼────────────────────────┐
          │                        │                        │
          ▼                        ▼                        ▼
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│   PAYMENTS      │  │   MESSAGING     │  │   COMPLIANCE    │
│                 │  │                 │  │                 │
│ ┌─────────────┐ │  │ ┌─────────────┐ │  │ ┌─────────────┐ │
│ │  m-Floos    │ │  │ │ Telesom SMS │ │  │ │   ZATCA     │ │
│ │  (Wallet)   │ │  │ │             │ │  │ │ (E-Invoice) │ │
│ └─────────────┘ │  │ └─────────────┘ │  │ └─────────────┘ │
│ ┌─────────────┐ │  │ ┌─────────────┐ │  │ ┌─────────────┐ │
│ │  OneCash    │ │  │ │ Sabafon SMS │ │  │ │Elasticsearch│ │
│ │  (Wallet)   │ │  │ │             │ │  │ │  (Search)   │ │
│ └─────────────┘ │  │ └─────────────┘ │  │ └─────────────┘ │
│                 │  │ ┌─────────────┐ │  │                 │
│                 │  │ │ WhatsApp    │ │  │                 │
│                 │  │ │ Business    │ │  │                 │
│                 │  │ └─────────────┘ │  │                 │
└─────────────────┘  └─────────────────┘  └─────────────────┘
```

## 2. Actor Descriptions

### 2.1 Customer
- **Role**: End consumer who browses, purchases, and reviews products
- **Interactions**: Browse catalog, place orders, pay via wallet or cash, track deliveries, leave reviews, manage wishlist/cart, contact support, follow stores
- **Channels**: Web app, mobile app (PWA/native)

### 2.2 Vendor
- **Role**: Merchant who sells products through the marketplace
- **Interactions**: Register stores, complete KYC, manage products/inventory, view orders, process fulfillments, receive payouts, view analytics, respond to reviews
- **Channels**: Vendor dashboard (web)

### 2.3 Admin
- **Role**: Platform operator who oversees marketplace operations
- **Interactions**: Approve vendor KYC, manage categories, handle disputes, configure promotions, view financial reports, manage content/banners, handle support escalations, manage zones
- **Channels**: Admin dashboard (web)

### 2.4 Delivery Provider
- **Role**: Third-party or in-house courier who fulfills last-mile delivery
- **Interactions**: Receive assignments, update delivery status, capture proof of delivery (POD), manage delivery zones
- **Channels**: Delivery portal/app (web/mobile)

### 2.5 Support Agent
- **Role**: Customer service representative who resolves buyer/seller issues
- **Interactions**: Manage tickets, escalate issues, process returns/refunds, communicate with customers via WhatsApp
- **Channels**: Support dashboard (web)

## 3. External System Integrations

| System | Purpose | Protocol | Direction |
|--------|---------|----------|-----------|
| **m-Floos** | Mobile wallet payments, balance inquiry, transfer | REST API (mTLS) | Bidirectional |
| **OneCash** | Alternative mobile wallet, payments, refunds | REST API (mTLS) | Bidirectional |
| **Telesom SMS** | OTP delivery, transactional SMS (Telesom subscribers) | REST API | Outbound |
| **Sabafon SMS** | OTP delivery, transactional SMS (Sabafon subscribers) | REST API | Outbound |
| **WhatsApp Business** | Order notifications, support chat, delivery updates | Cloud API | Bidirectional |
| **ZATCA** | Saudi e-invoicing compliance (Phase 2 FATOORA) | REST API (mTLS) | Outbound |
| **Elasticsearch** | Full-text product search, faceted filtering, autocomplete | REST API | Bidirectional |

## 4. Data Flows

### 4.1 Customer Data Flows
```
Customer ──[Browse/Search]──► YemenMart ──[Query]──► Elasticsearch
Customer ──[Place Order]────► YemenMart
Customer ──[Pay via Wallet]─► YemenMart ──[Charge]──► m-Floos / OneCash
Customer ──[Track Delivery]─► YemenMart ──[Status]──► Delivery Provider
Customer ──[OTP Verify]─────► YemenMart ──[Send OTP]─► Telesom SMS / Sabafon SMS
Customer ──[Get Updates]────► YemenMart ──[Notify]──► WhatsApp Business
```

### 4.2 Vendor Data Flows
```
Vendor ──[Submit KYC]───────► YemenMart ──[Store]──► B02 Marketplace
Vendor ──[Add Products]─────► YemenMart ──[Index]──► Elasticsearch
Vendor ──[View Orders]──────► YemenMart ──[Read]───► B04 Commerce
Vendor ──[Receive Payout]───► YemenMart ──[Notify]─► WhatsApp Business
```

### 4.3 Admin Data Flows
```
Admin ──[Approve KYC]───────► YemenMart ──[Update]──► B02 Marketplace
Admin ──[Create Promotion]──► YemenMart ──[Store]───► B13 Promotion
Admin ──[Generate Report]───► YemenMart ──[Read]────► B06 Finance
Admin ──[Flag Content]──────► YemenMart ──[Update]──► B11 Content
```

### 4.4 Delivery Provider Data Flows
```
YemenMart ──[Assign Order]───► Delivery Provider
Delivery Provider ──[Update Status]──► YemenMart ──[Update]──► B04 Commerce
Delivery Provider ──[POD]────────────► YemenMart ──[Store]───► B07 Logistics
YemenMart ──[Notify Customer]─────────► WhatsApp Business
```

### 4.5 External System Flows (Outbound)
```
YemenMart ──[Invoice]──────► ZATCA (e-invoicing)
YemenMart ──[Reindex]──────► Elasticsearch (product sync)
YemenMart ──[OTP]──────────► Telesom SMS / Sabafon SMS
YemenMart ──[Notification]─► WhatsApp Business
YemenMart ──[Charge/Credit]─► m-Floos / OneCash
```

## 5. Security & Trust Boundaries

| Boundary | Description |
|----------|-------------|
| **YemenMart Platform** | Internal trust zone; all 13 bounded contexts operate here |
| **Payment Gateways** | m-Floos/OneCash: mTLS required; PCI-DSS scope; tokenized card storage |
| **SMS Providers** | Telesom/Sabafon: API key authentication; rate-limited OTP delivery |
| **WhatsApp Business** | Cloud API with Meta token; webhook verification for inbound messages |
| **ZATCA** | Government endpoint; mTLS with client certificates; immutable invoice hash chain |
| **Elasticsearch** | Internal cluster; no public access; index-level ACL |

## 6. Deployment Context

```
┌────────────────────────────────────────────────────┐
│                   Cloud Region                     │
│                                                    │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐         │
│  │ API      │  │ Worker   │  │ Scheduler│         │
│  │ Gateway  │  │ Processes│  │ (Cron)   │         │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘         │
│       │              │              │               │
│  ┌────▼──────────────▼──────────────▼─────┐        │
│  │            Message Bus (Redis)         │        │
│  └────┬──────────────┬──────────────┬─────┘        │
│       │              │              │               │
│  ┌────▼─────┐  ┌─────▼────┐  ┌─────▼────┐         │
│  │PostgreSQL│  │  Redis   │  │  S3/MinIO│         │
│  │  (Main)  │  │ (Cache)  │  │ (Files)  │         │
│  └──────────┘  └──────────┘  └──────────┘         │
│                                                    │
└────────────────────────────────────────────────────┘
```
