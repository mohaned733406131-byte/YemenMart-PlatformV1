# 22 - Glossary

**Category:** Glossary  
**Purpose:** Terminology, business terms, technical terms, naming conventions

---

## Contents

- `terminology.md` - Core platform entities and concepts
- `business-terms.md` - Business and domain terms
- `technical-terms.md` - Technical and architectural terms
- `naming-conventions.md` - Code, database, and API naming standards
- `abbreviations.md` - Common abbreviations and acronyms

---

## Core Entities (Ubiquitous Language)

### User Entities
- **Customer** - End user who browses and purchases products
- **Vendor** - Seller who lists and fulfills products
- **Admin** - Platform administrator with oversight privileges
- **Delivery Provider** - Third-party logistics provider for shipments
- **System** - Automated processes and scheduled jobs

### Product Entities
- **Product** - Item for sale with attributes and variants
- **Product Variant** - Specific SKU (size, color, etc.)
- **Category** - Hierarchical product classification
- **Attribute** - Product property (e.g., brand, material, weight)
- **Offer** - Promotion or discount applied to products

### Order Entities
- **Master Order** - Customer's complete order (may span multiple vendors)
- **Sub-Order** - Vendor-specific portion of master order
- **Order Item** - Individual product line in sub-order
- **Order State** - Current status in 17-state lifecycle
- **Delivery Code** - 3-digit verification code for delivery confirmation

### Payment Entities
- **Wallet** - Digital account holding customer/vendor funds
- **Wallet Transaction** - Ledger entry for wallet activity
- **Escrow Hold** - 7-day payment hold after delivery
- **Commission** - Platform fee on vendor sales

### Marketplace Entities
- **Store** - Vendor's branded presence on marketplace
- **Store Template** - Pre-designed store layout (10+ options)
- **KYC (Know Your Customer)** - Vendor verification documents
- **Product Trial** - Customer try-before-buy feature
- **System Service** - Service booking (40+ categories)

---

## Business Terms

### Payment Terms
- **Bank Transfer** - Only method for wallet top-up (NO cards)
- **m-Floos** - Yemeni mobile wallet provider
- **OneCash** - Alternative Yemeni mobile wallet provider
- **Escrow** - Secure holding of payment until delivery confirmed
- **Payout** - Transfer of funds from escrow to vendor wallet

### Order Terms
- **Partial Cancellation** - Cancel individual items in multi-item order
- **Vendor Confirmation Window** - 4-hour period for vendor to accept order
- **Stock Reservation** - 15-minute hold on inventory during checkout
- **Delivery Marketplace** - Competitive bidding by delivery providers
- **Delivery Zone** - Geographic area for delivery pricing

### Loyalty Terms
- **Loyalty Points** - Reward points for all wallet payments
- **Points Redemption** - Using points for discounts
- **Tier System** - Customer tiers based on lifetime value
- **Electronic Payment** - Wallet or bank transfer (NOT COD)

---

## Technical Terms

### Architecture Terms
- **Bounded Context** - Domain-driven design module boundary
- **Master/Sub-Order** - Order decomposition pattern
- **Event Sourcing** - Audit trail via event log
- **State Machine** - Order/payment state transition logic
- **Adapter Pattern** - Abstraction for payment/delivery providers

### Database Terms
- **Soft Delete** - Mark as deleted without removing from database
- **Audit Trail** - Immutable log of changes
- **Read Replica** - Database copy for analytics queries
- **Partitioning** - Time-based data splitting for performance
- **Composite Key** - Multi-column primary key

### API Terms
- **JWT (JSON Web Token)** - Authentication token format
- **OTP (One-Time Password)** - SMS verification code
- **Rate Limiting** - Request throttling (100 req/min authenticated)
- **Idempotency** - Safe retry behavior for duplicate requests
- **Pagination** - Cursor-based result paging

---

## Naming Conventions

### Code Naming
- **Files:** `lowercase-kebab-case.ts` (e.g., `payment-service.ts`)
- **Classes:** `PascalCase` (e.g., `PaymentService`)
- **Functions:** `camelCase` (e.g., `createOrder()`)
- **Constants:** `UPPER_SNAKE_CASE` (e.g., `MAX_RETRIES`)
- **Types/Interfaces:** `PascalCase` with `I` prefix for interfaces (e.g., `IPaymentRequest`)

### Database Naming
- **Tables:** `lowercase_snake_case` (e.g., `wallet_transactions`)
- **Columns:** `lowercase_snake_case` (e.g., `created_at`)
- **Indexes:** `idx_table_column` (e.g., `idx_orders_created_at`)
- **Foreign Keys:** `fk_table_column` (e.g., `fk_orders_user_id`)
- **Enum Types:** `lowercase_snake_case_enum` (e.g., `order_state_enum`)

### API Naming
- **Endpoints:** `lowercase-kebab-case` (e.g., `/wallet-transactions`)
- **Resources:** Plural nouns (e.g., `/orders`, `/products`)
- **Actions:** HTTP verbs (GET, POST, PUT, DELETE, PATCH)
- **Query Params:** `camelCase` (e.g., `?sortBy=createdAt&order=desc`)
- **Headers:** `Kebab-Case` (e.g., `X-Request-ID`)

### Requirement IDs
- **Functional:** `FR-NNN` (e.g., `FR-001`)
- **Non-Functional:** `NFR-TYPE-NNN` (e.g., `NFR-PERF-001`)
- **Business Rules:** `BR-DOMAIN-NN` (e.g., `BR-PAY-01`)
- **Use Cases:** `UC-ACTOR-NN` (e.g., `UC-C12`)
- **Test Cases:** `TC-BLOCK-NNN` (e.g., `TC-PAY-001`)

---

## Common Abbreviations

### Business Abbreviations
- **AC** - Acceptance Criteria
- **CSAT** - Customer Satisfaction Score
- **GMV** - Gross Merchandise Value
- **KYC** - Know Your Customer
- **NPS** - Net Promoter Score
- **SKU** - Stock Keeping Unit
- **UAT** - User Acceptance Testing

### Technical Abbreviations
- **ADR** - Architecture Decision Record
- **API** - Application Programming Interface
- **CRUD** - Create, Read, Update, Delete
- **DTO** - Data Transfer Object
- **E2E** - End-to-End
- **JWT** - JSON Web Token
- **ORM** - Object-Relational Mapping
- **OTP** - One-Time Password
- **REST** - Representational State Transfer
- **RTL** - Right-to-Left (Arabic text direction)
- **SLA** - Service Level Agreement
- **SMS** - Short Message Service
- **TLS** - Transport Layer Security

### Infrastructure Abbreviations
- **CDN** - Content Delivery Network
- **CI/CD** - Continuous Integration / Continuous Deployment
- **DNS** - Domain Name System
- **ELK** - Elasticsearch, Logstash, Kibana
- **HA** - High Availability
- **HTTPS** - Hypertext Transfer Protocol Secure
- **K8s** - Kubernetes
- **RTO** - Recovery Time Objective
- **RPO** - Recovery Point Objective
- **VPC** - Virtual Private Cloud

### Database Abbreviations
- **ACID** - Atomicity, Consistency, Isolation, Durability
- **DBMS** - Database Management System
- **ERD** - Entity Relationship Diagram
- **FK** - Foreign Key
- **FTS** - Full-Text Search
- **PK** - Primary Key
- **RDBMS** - Relational Database Management System
- **SQL** - Structured Query Language

---

## Yemen-Specific Terms

### Geographic Terms
- **Governorate** - First-level administrative division in Yemen
- **District** - Second-level administrative division
- **Zone** - Delivery zone classification

### Currency Terms
- **YER** - Yemeni Rial (ISO 4217 code)
- **SAR** - Saudi Riyal (for GCC expansion)
- **USD** - US Dollar (for international transactions)

### Cultural Terms
- **Hijri Calendar** - Islamic lunar calendar (optional support)
- **RTL (Right-to-Left)** - Arabic text direction
- **Gregorian Calendar** - Primary calendar format

---

## Related Categories
- `00-project-overview` - Platform overview
- `01-business-analysis` - Business rules
- `23-templates` - Document templates

---

*Source: Terminology extracted from analayesev2/01-PROJECT-IDENTITY/03-ubiquitous-language.md and domain analysis*
