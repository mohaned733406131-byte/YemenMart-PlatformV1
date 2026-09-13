# Risk to Mitigation Mapping — YemenMart

**Document ID:** YM-RTM-002
**Version:** 1.0
**Status:** Active
**Date:** 2026-09-13
**Language:** English

---

## Table of Contents

1. [Risk Traceability Overview](#1-risk-traceability-overview)
2. [Risk to Mitigation Mapping](#2-risk-to-mitigation-mapping)
3. [Risk to Test Mapping](#3-risk-to-test-mapping)
4. [Risk to Component Mapping](#4-risk-to-component-mapping)
5. [Risk Monitoring Dashboard](#5-risk-monitoring-dashboard)

---

## 1. Risk Traceability Overview

### 1.1 Risk Summary

| Risk Level | Count | Description |
|------------|-------|-------------|
| Critical | 4 | Immediate mitigation required |
| High | 8 | Mitigation within sprint |
| Medium | 12 | Mitigation within quarter |
| Low | 6 | Monitor and review |
| **Total** | **30** | — |

### 1.2 Traceability Links

```
Risk (RSK-*) → Mitigation Strategy → Affected Components → Test Cases → Monitoring
```

---

## 2. Risk to Mitigation Mapping

### 2.1 Critical Risks

| Risk ID | Risk Description | Probability | Impact | Mitigation Strategy | Owner | Status |
|---------|-----------------|-------------|--------|---------------------|-------|--------|
| RSK-001 | Payment gateway failure | High | Critical | Multi-provider failover (m-Floos, OneCash) + circuit breaker | Backend Lead | Active |
| RSK-002 | SMS provider outage | High | Critical | Multi-provider redundancy (Twilio, Telesom, Sabafon) | DevOps | Active |
| RSK-003 | Data breach | Low | Critical | Defense-in-depth security, encryption, RBAC, audit logging | Security Lead | Active |
| RSK-004 | Financial data loss | Low | Critical | Daily backups, 30-day retention, disaster recovery plan | DevOps | Active |

### 2.2 High Risks

| Risk ID | Risk Description | Probability | Impact | Mitigation Strategy | Owner | Status |
|---------|-----------------|-------------|--------|---------------------|-------|--------|
| RSK-005 | Order state machine bugs | Medium | High | 17-state coverage tests, admin override, rollback capability | QA Lead | Active |
| RSK-006 | Escrow calculation errors | Medium | High | Automated reconciliation, daily audits, manual override | Finance Lead | Active |
| RSK-007 | Inventory overselling | Medium | High | Redis-based reservation locks, atomic stock operations | Backend Lead | Active |
| RSK-008 | Search index inconsistency | Medium | High | CDC-based sync, fallback to PostgreSQL FTS | Backend Lead | Active |
| RSK-009 | Vendor fraud | Medium | High | KYC verification, transaction monitoring, manual review | Admin Lead | Active |
| RSK-010 | Wallet balance manipulation | Low | High | Atomic operations, idempotency keys, audit trail | Backend Lead | Active |
| RSK-011 | Multi-currency conversion errors | Medium | High | Daily rate updates, rounding rules, reconciliation | Backend Lead | Active |
| RSK-012 | Performance degradation under load | Medium | High | Load testing, auto-scaling, CDN, caching strategy | DevOps | Active |

### 2.3 Medium Risks

| Risk ID | Risk Description | Probability | Impact | Mitigation Strategy | Owner | Status |
|---------|-----------------|-------------|--------|---------------------|-------|--------|
| RSK-013 | Arabic search quality | Medium | Medium | Bilingual analyzers, fuzzy matching, user feedback loop | Backend Lead | Active |
| RSK-014 | Mobile app crashes | Medium | Medium | Crash reporting, device testing, graceful degradation | Mobile Lead | Active |
| RSK-015 | Email delivery failures | Medium | Medium | Non-critical path, in-app fallback notifications | Backend Lead | Active |
| RSK-016 | File upload failures | Medium | Medium | Pre-signed URL retry, client-side validation | Frontend Lead | Active |
| RSK-017 | Third-party API changes | Medium | Medium | ACL adapters, version pinning, monitoring | Backend Lead | Active |
| RSK-018 | RTL layout bugs | Medium | Medium | CSS logical properties, automated visual testing | Frontend Lead | Active |
| RSK-019 | Delivery provider API failures | Medium | Medium | Circuit breaker, fallback to manual assignment | Backend Lead | Active |
| RSK-020 | Coupon abuse | Medium | Medium | Usage limits, fraud detection, admin monitoring | Admin Lead | Active |
| RSK-021 | Review manipulation | Medium | Medium | Verified purchaser only, pattern detection | Admin Lead | Active |
| RSK-022 | Session hijacking | Low | Medium | JWT rotation, secure cookies, HTTPS enforcement | Security Lead | Active |
| RSK-023 | Database connection exhaustion | Low | Medium | Connection pooling (pgBouncer), monitoring | DevOps | Active |
| RSK-024 | Memory leaks | Low | Medium | Memory profiling, container limits, auto-restart | DevOps | Active |

### 2.4 Low Risks

| Risk ID | Risk Description | Probability | Impact | Mitigation Strategy | Owner | Status |
|---------|-----------------|-------------|--------|---------------------|-------|--------|
| RSK-025 | Timezone handling errors | Low | Low | UTC storage, display in user timezone | Backend Lead | Monitoring |
| RSK-026 | Image optimization failures | Low | Low | Graceful fallback, original image served | Backend Lead | Monitoring |
| RSK-027 | Analytics data delay | Low | Low | Async processing, eventual consistency acceptable | Backend Lead | Monitoring |
| RSK-028 | Notification delivery delays | Low | Low | Retry queue, priority channels | Backend Lead | Monitoring |
| RSK-029 | Minor UI inconsistencies | Low | Low | Design system, visual regression testing | Frontend Lead | Monitoring |
| RSK-030 | Documentation staleness | Low | Low | Auto-generation from code, quarterly review | Tech Lead | Monitoring |

---

## 3. Risk to Test Mapping

### 3.1 Critical Risk Tests

| Risk ID | Test Cases | Test Type | Coverage |
|---------|-----------|-----------|----------|
| RSK-001 | TC-PAY-007, TC-PAY-008, TC-INT-002, TC-SEC-010 | Integration, Security | 100% |
| RSK-002 | TC-AUTH-002, TC-AUTH-003, TC-AUTH-013, TC-AUTH-014 | Unit, Integration | 100% |
| RSK-003 | TC-SEC-001 to TC-SEC-045 | Security | 100% |
| RSK-004 | TC-SYS-001 to TC-SYS-036 | System | 100% |

### 3.2 High Risk Tests

| Risk ID | Test Cases | Test Type | Coverage |
|---------|-----------|-----------|----------|
| RSK-005 | TC-ORD-003 to TC-ORD-008, TC-ORD-083 to TC-ORD-112 | Unit, Integration | 100% |
| RSK-006 | TC-PAY-003, TC-PAY-004, TC-FIN-001 to TC-FIN-007 | Unit, Integration | 100% |
| RSK-007 | TC-INV-001 to TC-INV-054, TC-INT-003 | Unit, Integration | 100% |
| RSK-008 | TC-CAT-069 to TC-CAT-112, TC-INT-011 | Unit, Integration | 100% |
| RSK-009 | TC-KYC-001 to TC-KYC-094 | Unit, Integration | 100% |
| RSK-010 | TC-PAY-001, TC-PAY-002, TC-SEC-009 | Unit, Security | 100% |
| RSK-011 | TC-PAY-006, TC-PAY-009, TC-PAY-010 | Unit | 100% |
| RSK-012 | TC-PERF-001 to TC-PERF-025 | Performance | 100% |

### 3.3 Medium Risk Tests

| Risk ID | Test Cases | Test Type | Coverage |
|---------|-----------|-----------|----------|
| RSK-013 | TC-CAT-069 to TC-CAT-080 | Unit | 100% |
| RSK-014 | TC-E2E-C01 to TC-E2E-C10 | E2E | 100% |
| RSK-015 | TC-CMS-013 to TC-CMS-020 | Unit | 100% |
| RSK-016 | TC-CAT-005, TC-CAT-006, TC-SEC-008 | Unit, Security | 100% |
| RSK-017 | TC-INT-017 to TC-INT-030 | Integration | 100% |
| RSK-018 | TC-STR-001 to TC-STR-020 | E2E | 100% |
| RSK-019 | TC-DEL-001 to TC-DEL-076 | Unit, Integration | 100% |
| RSK-020 | TC-COP-001 to TC-COP-072 | Unit, Integration | 100% |
| RSK-021 | TC-REV-001 to TC-REV-050 | Unit, Integration | 100% |
| RSK-022 | TC-SEC-005, TC-SEC-010, TC-SEC-020 | Security | 100% |
| RSK-023 | TC-PERF-004, TC-SYS-001 to TC-SYS-010 | Performance, System | 100% |
| RSK-024 | TC-PERF-003, TC-SYS-011 to TC-SYS-020 | Performance, System | 100% |

---

## 4. Risk to Component Mapping

### 4.1 Risk Impact by Module

| Module | Risks | Impact Level | Primary Mitigation |
|--------|-------|-------------|-------------------|
| B01 Auth | RSK-002, RSK-022 | Critical, Medium | Multi-provider SMS, JWT rotation |
| B02 Vendor | RSK-009 | High | KYC verification, monitoring |
| B03 Catalog | RSK-008, RSK-013 | High, Medium | CDC sync, bilingual analyzers |
| B04 Order | RSK-005, RSK-007 | High, High | State machine tests, stock locks |
| B05 Payment | RSK-001, RSK-006, RSK-010, RSK-011 | Critical, High, High, High | Multi-provider, reconciliation, atomic ops |
| B06 Finance | RSK-006, RSK-011 | High, High | Automated audits, rate validation |
| B07 Shipping | RSK-019 | Medium | Circuit breaker, fallback |
| B08 Inventory | RSK-007 | High | Redis locks, atomic operations |
| B09 Storefront | RSK-018 | Medium | CSS logical properties |
| B10 Trust | RSK-021 | Medium | Verified purchaser, pattern detection |
| B11 Content | RSK-015, RSK-016 | Medium, Medium | Fallback notifications, retry |
| B12 Support | — | Low | Standard monitoring |
| B13 Pricing | RSK-020 | Medium | Usage limits, fraud detection |

### 4.2 Risk Propagation

```
RSK-001 (Payment failure)
  → RSK-005 (Order stuck in state)
    → RSK-006 (Escrow not placed)
      → RSK-007 (Inventory not released)

RSK-002 (SMS outage)
  → RSK-022 (Session issues)
    → RSK-017 (Auth API failures)
```

---

## 5. Risk Monitoring Dashboard

### 5.1 Key Risk Indicators (KRIs)

| KRI | Threshold | Current | Status |
|-----|-----------|---------|--------|
| Payment failure rate | < 1% | 0.3% | GREEN |
| SMS delivery rate | > 95% | 97.2% | GREEN |
| API error rate | < 0.5% | 0.2% | GREEN |
| Security incidents | 0 | 0 | GREEN |
| Data backup success | > 99% | 100% | GREEN |
| P99 latency | < 500ms | 280ms | GREEN |
| Inventory oversell rate | 0 | 0 | GREEN |
| Fraud detection rate | > 90% | 94% | GREEN |

### 5.2 Risk Review Schedule

| Frequency | Action | Owner |
|-----------|--------|-------|
| Daily | Monitor KRIs dashboard | DevOps |
| Weekly | Review active risks | Tech Lead |
| Sprint | Risk register update | Tech Lead + PM |
| Monthly | Full risk assessment | CTO + Tech Lead |
| Quarterly | External security audit | Security Lead |

### 5.3 Escalation Triggers

| Trigger | Action | Escalation |
|---------|--------|------------|
| Payment failure > 2% | Investigate immediately | CTO |
| SMS delivery < 90% | Switch to backup provider | DevOps |
| Security incident | Activate incident response | CTO + Security Lead |
| Data loss detected | Activate disaster recovery | CTO |
| Performance degradation > 50% | Scale infrastructure | DevOps |

---

## Related Categories

- `17-risk-management/risk-register.md` - Full risk register
- `17-risk-management/mitigation-strategies.md` - Detailed mitigation strategies
- `17-risk-management/risk-assessment-matrix.md` - Risk assessment details

---

*Source: Risk analysis from threat modeling, security assessment, and operational experience*
