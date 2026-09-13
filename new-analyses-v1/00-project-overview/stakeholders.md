# YemenMart Stakeholder Analysis

## Overview

YemenMart has **9 identified stakeholders** with varying levels of influence, interest, and impact. This document analyzes each stakeholder's role, expectations, concerns, and communication requirements.

---

## Stakeholder Register

### S-01: Platform Owner / Investor

| Field | Details |
|-------|---------|
| **Role** | Executive sponsor, financial backer |
| **Category** | Internal - High Influence, High Interest |
| **Phase** | All phases |
| **Communication** | Weekly executive summary, monthly financial report |

**Responsibilities:**
- Approve budget and resource allocation
- Define strategic direction and market positioning
- Approve major architectural decisions
- Sign off on go/no-go gates

**Expectations:**
- ROI within 24 months
- 10,000+ merchants within 12 months of launch
- 100,000+ registered customers within 12 months
- 50M YER monthly GMV by month 18

**Concerns:**
- Regulatory compliance (ZATCA, local laws)
- Market penetration speed
- Competition from regional platforms
- Security and fraud prevention

**Success Criteria:**
- Positive unit economics within 18 months
- 99.99% platform uptime
- Zero regulatory fines

---

### S-02: Technical Lead / Architect

| Field | Details |
|-------|---------|
| **Role** | Architecture owner, technical decision-maker |
| **Category** | Internal - High Influence, High Interest |
| **Phase** | All phases |
| **Communication** | Daily standups, weekly architecture review |

**Responsibilities:**
- Design system architecture (13 blocks, 23 modules)
- Define technical standards and coding guidelines
- Conduct code reviews and approve PRs
- Ensure non-functional requirements (p95 < 200ms, 99.99% uptime)
- Manage technical debt

**Expectations:**
- 100% custom build (no e-commerce frameworks)
- Complete architectural documentation
- 974 test points with 100% pass rate
- Comprehensive threat model (55 threats identified)

**Concerns:**
- Scalability under load
- Arabic-first RTL implementation complexity
- Wallet system security
- Integration reliability with SMS/WhatsApp providers

**Success Criteria:**
- Architecture passes all 26 constraint validations
- Zero critical security vulnerabilities
- All modules meet performance benchmarks

---

### S-03: QA Lead / Test Engineer

| Field | Details |
|-------|---------|
| **Role** | Quality assurance, test strategy owner |
| **Category** | Internal - Medium Influence, High Interest |
| **Phase** | All phases |
| **Communication** | Daily test reports, weekly quality review |

**Responsibilities:**
- Execute 974 test points across 13 blocks
- Maintain test automation (80% unit, 15% integration, 5% E2E)
- Report quality gates and defect metrics
- Conduct security testing (26 security test points)
- Validate 62 acceptance criteria

**Expectations:**
- Complete test coverage for all 477 use cases
- Automated regression suite
- Clear defect classification and prioritization
- Performance testing for p95 < 200ms

**Concerns:**
- Test environment availability
- Arabic content validation
- Wallet transaction edge cases
- Delivery code lockout scenarios

**Success Criteria:**
- 100% test point pass rate for production release
- Zero P1/P2 defects in production
- 95%+ automation coverage for unit tests

---

### S-04: Product Manager

| Field | Details |
|-------|---------|
| **Role** | Product vision, feature prioritization |
| **Category** | Internal - High Influence, High Interest |
| **Phase** | All phases |
| **Communication** | Daily standups, bi-weekly sprint review |

**Responsibilities:**
- Define and prioritize product backlog
- Write user stories and acceptance criteria
- Validate feature completeness against 477 use cases
- Coordinate cross-module dependencies
- Manage stakeholder expectations

**Expectations:**
- All 26 non-negotiable constraints honored
- Arabic-first UX with RTL support
- Intuitive mobile-first experience
- Merchant-friendly onboarding

**Concerns:**
- Feature scope creep
- User adoption and retention
- Competitive differentiation
- Regulatory compliance delays

**Success Criteria:**
- 100% use case coverage
- User satisfaction score > 4.5/5
- Time-to-market < 12 months

---

### S-05: Security Officer

| Field | Details |
|-------|---------|
| **Role** | Security governance, threat management |
| **Category** | Internal - High Influence, Medium Interest |
| **Phase** | All phases |
| **Communication** | Weekly security review, incident reports |

**Responsibilities:**
- Manage threat model (55 identified threats)
- Conduct security audits and penetration testing
- Enforce 26 security test points
- Validate 62 security acceptance criteria
- Review code for vulnerabilities
- Monitor security incidents

**Expectations:**
- Zero critical/high vulnerabilities in production
- Complete OWASP Top 10 coverage
- Secure wallet implementation
- PCI-DSS-like practices for wallet data

**Concerns:**
- Wallet fraud and theft
- OTP interception attacks
- SQL injection and XSS
- Privilege escalation
- Data leakage

**Success Criteria:**
- Zero security breaches
- All 26 security test points passing
- Quarterly security audit pass
- Incident response time < 1 hour

---

### S-06: Merchant Success Manager

| Field | Details |
|-------|---------|
| **Role** | Merchant onboarding, support, retention |
| **Category** | Internal - Medium Influence, High Interest |
| **Phase** | Launch and post-launch |
| **Communication** | Weekly merchant metrics, monthly retention report |

**Responsibilities:**
- Onboard merchants to the platform
- Provide training on store setup (10+ templates)
- Support merchant KYC completion
- Monitor merchant satisfaction and churn
- Gather merchant feedback for product improvements

**Expectations:**
- Intuitive merchant dashboard
- Clear KYC process (mandatory per Constraint 19)
- 40+ system service categories available
- Responsive support (< 4 hour response time)

**Concerns:**
- Merchant onboarding friction
- KYC completion rates
- Store template usability
- Payment settlement delays (7-day escrow)

**Success Criteria:**
- 80% KYC completion within 7 days of registration
- Merchant satisfaction score > 4.0/5
- < 5% monthly merchant churn

---

### S-07: Delivery Operations Manager

| Field | Details |
|-------|---------|
| **Role** | Delivery fleet management, logistics |
| **Category** | Internal - Medium Influence, Medium Interest |
| **Phase** | Launch and post-launch |
| **Communication** | Daily delivery metrics, weekly operations review |

**Responsibilities:**
- Manage delivery agent fleet
- Ensure delivery code system reliability
- Monitor 3-attempt lockout compliance
- Track delivery performance metrics
- Coordinate across 17 governates

**Expectations:**
- No GPS tracking (Constraint 10)
- Reliable SMS/WhatsApp delivery codes
- Clear agent assignment workflow
- Delivery confirmation within SLA

**Concerns:**
- Delivery code delivery delays
- Agent availability in rural areas
- Customer lockout handling
- Multi-merchant order coordination

**Success Criteria:**
- 95%+ on-time delivery rate
- < 1% delivery code lockout rate
- Customer delivery satisfaction > 4.0/5

---

### S-08: Finance Officer

| Field | Details |
|-------|---------|
| **Role** | Financial operations, compliance, reporting |
| **Category** | Internal - Medium Influence, Medium Interest |
| **Phase** | All phases |
| **Communication** | Monthly financial reports, quarterly audit prep |

**Responsibilities:**
- Manage double-entry bookkeeping system
- Ensure ZATCA-compliant invoice generation
- Maintain 5-year invoice retention
- Process wallet transactions and escrow releases
- Calculate 15% VAT on discounted prices
- Prepare financial reports for stakeholders

**Expectations:**
- Automated invoice generation
- Accurate VAT calculation
- Immutable financial records
- Audit-ready documentation

**Concerns:**
- Invoice compliance errors
- VAT calculation accuracy
- Escrow release timing
- Reconciliation discrepancies

**Success Criteria:**
- Zero VAT calculation errors
- 100% ZATCA invoice compliance
- Monthly reconciliation balance
- Audit pass rate 100%

---

### S-09: End Customer

| Field | Details |
|-------|---------|
| **Role** | Platform user, buyer |
| **Category** | External - Low Influence, High Interest |
| **Phase** | Post-launch |
| **Communication** | In-app notifications, email (optional), SMS |

**Responsibilities:**
- Create account with phone number
- Fund wallet
- Browse and purchase products
- Enter delivery codes (max 3 attempts)
- Request returns within merchant policy window

**Expectations:**
- Easy phone-based registration
- Fast product search and discovery
- Secure wallet transactions
- Clear order status updates
- Fair return policies
- Arabic-first interface

**Concerns:**
- Wallet security
- Delivery code complexity
- Return/refund delays
- Product quality from merchants

**Success Criteria:**
- Registration completion > 70%
- Repeat purchase rate > 40%
- Customer satisfaction > 4.0/5
- Support ticket resolution < 24 hours

---

## Stakeholder Power/Interest Matrix

```
                    HIGH INTEREST
                         │
    ┌────────────────────┼────────────────────┐
    │                    │                    │
    │   S-01 (Owner)     │   S-02 (Tech Lead) │
    │   S-04 (PM)        │   S-03 (QA Lead)   │
    │                    │   S-05 (Security)   │
    │   KEEP SATISFIED   │   MANAGE CLOSELY   │
    │                    │                    │
HIGH├────────────────────┼────────────────────┤LOW
INFL│                    │                    │INFL
UEN │   S-06 (Merchant)  │   S-09 (Customer)  │UEN
CE  │   S-07 (Delivery)  │                    │CE
    │   S-08 (Finance)   │   MONITOR          │
    │                    │                    │
    │   KEEP INFORMED    │                    │
    │                    │                    │
    └────────────────────┼────────────────────┘
                         │
                    LOW INTEREST
```

## Communication Plan

| Stakeholder | Frequency | Channel | Content |
|-------------|-----------|---------|---------|
| S-01 Owner | Weekly | Executive Summary | KPIs, budget, risks |
| S-02 Tech Lead | Daily | Standup + Wiki | Architecture, blockers |
| S-03 QA Lead | Daily | Test Dashboard | Test results, defects |
| S-04 PM | Daily | Standup + Backlog | Stories, priorities |
| S-05 Security | Weekly | Security Report | Threats, vulnerabilities |
| S-06 Merchant SM | Weekly | Metrics Report | Onboarding, retention |
| S-07 Delivery | Daily | Operations Dashboard | Delivery metrics |
| S-08 Finance | Monthly | Financial Report | VAT, invoices, escrow |
| S-09 Customer | Event-driven | In-app + SMS | Order updates, alerts |

---

*Document Version: 1.0 | Last Updated: 2026-09-13 | Classification: Confidential*
