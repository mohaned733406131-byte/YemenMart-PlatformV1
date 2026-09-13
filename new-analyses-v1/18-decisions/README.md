# 18 - Decisions

**Category:** Decisions  
**Purpose:** Architecture Decision Records (ADRs) and key project decisions

---

## Contents

- `decision-log.md` - Chronological list of all decisions
- `adr/` - Architecture Decision Records directory
  - `adr-001-custom-build-no-medusa.md`
  - `adr-002-wallet-only-payments.md`
  - `adr-003-sms-only-authentication.md`
  - `adr-004-master-suborder-architecture.md`
  - `adr-005-orm-selection-prisma.md`
  - `adr-006-react-frontend-stack.md`
  - `adr-007-postgresql-database.md`
  - ... (more ADRs)

---

## Decision Record Format

Each ADR follows this template:

```markdown
# ADR-XXX: [Decision Title]

**Status:** Accepted | Rejected | Deprecated | Superseded  
**Date:** YYYY-MM-DD  
**Deciders:** [Names/Roles]  
**Related:** [Related ADRs]

## Context
What is the issue we're trying to address? What factors led to this decision?

## Decision
What decision did we make?

## Consequences
What are the positive and negative consequences of this decision?

## Alternatives Considered
What other options did we evaluate?

## References
Links to related documents, discussions, or research.
```

---

## Key Decisions (ADR Summary)

### ADR-001: 100% Custom Build (NO Medusa.js)
- **Status:** ACCEPTED
- **Decision:** Build custom e-commerce platform from scratch
- **Rationale:** 
  - Full control over features and constraints
  - No framework lock-in
  - Specific Yemen market requirements (wallet, COD, SMS)
- **Alternatives:** Medusa.js, Shopify, WooCommerce
- **Consequences:** 
  - ✅ Full flexibility
  - ✅ Optimized for specific needs
  - ❌ Longer development time
  - ❌ More maintenance burden

### ADR-002: Wallet-Only Payments (NO Cards)
- **Status:** ACCEPTED
- **Decision:** All payments through YemenMart wallet system
- **Rationale:**
  - Limited credit card penetration in Yemen
  - Trust issues with online card payments
  - Escrow mechanism requires wallet control
- **Alternatives:** Card payments, BNPL, installments
- **Consequences:**
  - ✅ Full control over payment flow
  - ✅ Escrow implementation easier
  - ❌ Barrier to entry (must fund wallet)
  - ❌ May limit customer adoption

### ADR-003: SMS-Only Authentication (NO Email/Social)
- **Status:** ACCEPTED
- **Decision:** SMS OTP as sole verification method
- **Rationale:**
  - Phone numbers more reliable than email in Yemen
  - High mobile penetration
  - SMS delivery infrastructure established
- **Alternatives:** Email verification, social login, email+SMS
- **Consequences:**
  - ✅ Simpler user flow
  - ✅ Better reach in target market
  - ❌ SMS gateway dependency
  - ❌ Cost per verification

### ADR-004: Master/Sub-Order Architecture
- **Status:** ACCEPTED
- **Decision:** Split multi-vendor orders into master + sub-orders
- **Rationale:**
  - Independent vendor fulfillment
  - Separate tracking per vendor
  - Partial cancellation support
- **Alternatives:** Single order with vendor items, separate orders per vendor
- **Consequences:**
  - ✅ Flexible fulfillment
  - ✅ Clear vendor separation
  - ❌ More complex order management
  - ❌ Customer sees multiple orders

### ADR-005: Prisma ORM Selection
- **Status:** ACCEPTED
- **Decision:** Use Prisma as ORM for database access
- **Rationale:**
  - TypeScript-first design
  - Type-safe queries
  - Excellent migration tooling
  - Good PostgreSQL support
- **Alternatives:** TypeORM, Sequelize, Knex.js
- **Consequences:**
  - ✅ Type safety
  - ✅ Great developer experience
  - ❌ Learning curve for team
  - ❌ Performance overhead vs raw SQL

### ADR-006: React Frontend Stack
- **Status:** ACCEPTED
- **Decision:** React 18+ for admin/vendor, Next.js 15 for customer storefront
- **Rationale:**
  - Large ecosystem and community
  - Component reusability
  - Next.js for SSR/SEO on storefront
- **Alternatives:** Vue.js, Angular, Svelte
- **Consequences:**
  - ✅ Proven technology
  - ✅ Large talent pool
  - ❌ Bundle size concerns
  - ❌ Hydration complexity with Next.js

### ADR-007: PostgreSQL Database
- **Status:** ACCEPTED
- **Decision:** PostgreSQL 16+ as primary database
- **Rationale:**
  - ACID compliance for financial data
  - JSON support for flexible fields
  - Full-text search capabilities
  - Excellent scaling story
- **Alternatives:** MySQL, MongoDB, SQL Server
- **Consequences:**
  - ✅ Data integrity
  - ✅ Rich feature set
  - ❌ Complex setup for HA
  - ❌ Vertical scaling limits

### ADR-008: 17-State Order System
- **Status:** ACCEPTED
- **Decision:** Implement 17 distinct order states
- **Rationale:**
  - Granular tracking of order lifecycle
  - Clear state transitions
  - Support for complex scenarios (partial cancel, COD approval)
- **Alternatives:** Simpler state machine (10-12 states)
- **Consequences:**
  - ✅ Detailed tracking
  - ✅ Flexible workflows
  - ❌ UI complexity
  - ❌ More edge cases to test

### ADR-009: No GPS Order Tracking
- **Status:** ACCEPTED
- **Decision:** Remove real-time GPS tracking feature
- **Rationale:**
  - Infrastructure limitations in Yemen
  - High implementation cost vs value
  - Delivery provider marketplace handles tracking
- **Alternatives:** GPS tracking, estimated delivery windows
- **Consequences:**
  - ✅ Simplified scope
  - ✅ Lower cost
  - ❌ Reduced customer visibility
  - ❌ Competitive disadvantage

### ADR-010: 7-Day Escrow Hold
- **Status:** ACCEPTED
- **Decision:** Hold payments in escrow for 7 days after delivery
- **Rationale:**
  - Customer protection period
  - Dispute resolution window
  - Trust building mechanism
- **Alternatives:** 3 days, 14 days, instant release
- **Consequences:**
  - ✅ Customer trust
  - ✅ Dispute buffer
  - ❌ Vendor cash flow delay
  - ❌ Platform capital requirements

---

## Decision Categories

### Architecture Decisions
- Technology stack choices
- System design patterns
- Integration approaches

### Business Decisions
- Payment model (wallet-only)
- Commission structure
- Feature prioritization

### Constraint Decisions
- SMS-only authentication
- No GPS tracking
- No subscriptions

### Process Decisions
- Development methodology
- Testing strategy
- Deployment approach

---

## Decision Review Process

1. **Proposal:** Team member proposes decision with ADR draft
2. **Discussion:** Team reviews and discusses alternatives
3. **Consensus:** Team reaches agreement or escalates to leadership
4. **Documentation:** ADR finalized and merged
5. **Review:** Quarterly review of all accepted ADRs

---

## Related Categories
- `04-architecture` - Architecture decisions
- `00-project-overview` - Project constraints
- `18-decisions` - Decision tracking

---

*Source: Architecture decisions from design process and constraint analysis*
