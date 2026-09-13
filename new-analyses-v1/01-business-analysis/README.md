# 01 - Business Analysis

**Category:** Business Analysis  
**Purpose:** Business model, rules, processes, and use cases

---

## Contents

### Core Documents
- `business-model.md` - Multi-vendor marketplace revenue model
- `business-rules.md` - All BR-* business rules organized by domain
- `value-proposition.md` - Value propositions for each actor
- `competitive-analysis.md` - Market positioning and competitors

### Use Cases Directory (`use-cases/`)
477 total use cases organized by actor:
- `customer-use-cases.md` - 128 customer workflows
- `vendor-use-cases.md` - 112 vendor workflows
- `admin-use-cases.md` - 120 admin workflows
- `delivery-provider-use-cases.md` - 47 delivery workflows
- `system-use-cases.md` - 70 automated system processes

---

## Business Model Summary

### Revenue Streams
1. **Commission** - Percentage of vendor sales (default: 10%)
2. **Featured Listings** - Promoted product placement
3. **Delivery Marketplace** - Commission on delivery bids
4. **System Services** - Commission on 40+ service categories
5. **Advertising** - Banner ads and promotions

### Key Business Rules (BR-*)
- **BR-PAY-10** - Wallet-only payments (NO cards, NO COD)
- **BR-PAY-12** - Bank transfer only for wallet top-up
- **BR-PAY-01** - 7-day escrow hold after delivery
- **BR-SYS-07** - SMS + WhatsApp verification
- **BR-ORD-01** - 4-hour vendor confirmation window
- **BR-VEND-01** - Dual identity (Customer → Vendor separate registration)

### Target Market
- **Primary:** Yemen (~34M population, 27% internet penetration)
- **Secondary:** GCC countries (Saudi Arabia, UAE, Oman, etc.)
- **Payment:** Wallet transfers, m-Floos, OneCash

---

## Use Case Summary

| Actor | Use Cases | Primary Blocks |
|-------|-----------|----------------|
| Customer | 128 | B01, B03, B04, B05, B09, B10, B13 |
| Vendor | 112 | B02, B03, B05, B06, B08, B09 |
| Admin | 120 | B01, B02, B04, B05, B06, B07, B11, B12 |
| Delivery Provider | 47 | B07, B04, B05 |
| System | 70 | B01, B05, B06, B07, B08, B11, B12 |

---

## Related Categories
- `00-project-overview` - Project foundation
- `02-requirements` - Functional requirements derived from use cases
- `03-system-analysis` - System design based on business processes

---

*Source: Business rules and use cases extracted from analayesev2 and Project-Block-Analysis*
