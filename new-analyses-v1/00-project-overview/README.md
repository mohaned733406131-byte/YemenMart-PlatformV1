# 00 - Project Overview

**Category:** Project Overview  
**Purpose:** High-level project definition, objectives, scope, and constraints

---

## Contents

This directory contains foundational project documentation:

### Core Documents
- `project-charter.md` - Official project authorization and high-level definition
- `project-objectives.md` - Measurable business and technical objectives
- `project-scope.md` - What is included and excluded from the project
- `stakeholders.md` - All project stakeholders and their roles
- `actors-and-roles.md` - System actors (Customer, Vendor, Admin, Delivery Provider, System)
- `project-constraints.md` - 26 non-negotiable platform constraints
- `success-criteria.md` - Metrics for measuring project success

---

## Key Information

### Platform Type
Multi-vendor e-commerce marketplace for Yemen and MENA region

### Critical Constraints
1. **Wallet-only payments** - NO credit/debit cards, NO BNPL, NO COD
2. **SMS + WhatsApp authentication** - NO email verification, NO social login
3. **100% custom build** - NO Medusa.js or third-party frameworks
4. **17-state order system** - Master/Sub-order architecture
5. **7-day escrow** - Payment holding period after delivery

### Primary Actors
1. **Customer** - Browses, orders, pays, reviews (128 use cases)
2. **Vendor** - Manages store, products, orders, inventory (112 use cases)
3. **Admin** - Platform governance, oversight, support (120 use cases)
4. **Delivery Provider** - Pickup, delivery, confirmation (47 use cases)
5. **System** - Automated jobs, notifications, health checks (70 use cases)

---

## Related Categories
- `01-business-analysis` - Business model and rules
- `02-requirements` - Detailed requirements
- `22-glossary` - Terminology definitions

---

*Source: Extracted from existing YemenMart requirements and analayesev2 documentation*
