# Risk Register — YemenMart

## 1. Overview

This register catalogs all identified risks to the YemenMart platform across 9 domains. Each risk is scored using Probability × Impact methodology and assigned an owner for mitigation tracking.

---

## 2. Risk Scoring

### 2.1 Probability Scale
| Score | Rating | Description |
|---|---|---|
| 1 | Rare | < 5% likelihood in next 12 months |
| 2 | Unlikely | 5-20% likelihood |
| 3 | Possible | 20-50% likelihood |
| 4 | Likely | 50-80% likelihood |
| 5 | Almost Certain | > 80% likelihood |

### 2.2 Impact Scale
| Score | Rating | Description |
|---|---|---|
| 1 | Negligible | Minimal disruption, < $1K loss |
| 2 | Minor | Limited disruption, $1K-$10K loss |
| 3 | Moderate | Partial disruption, $10K-$50K loss |
| 4 | Major | Significant disruption, $50K-$200K loss |
| 5 | Catastrophic | Full disruption, > $200K loss |

### 2.3 Risk Rating Matrix
| Probability × Impact | 1 | 2 | 3 | 4 | 5 |
|---|---|---|---|---|---|
| **5** | 5-Medium | 10-High | 15-Critical | 20-Critical | 25-Critical |
| **4** | 4-Low | 8-Medium | 12-High | 16-Critical | 20-Critical |
| **3** | 3-Low | 6-Medium | 9-Medium | 12-High | 15-Critical |
| **2** | 2-Low | 4-Low | 6-Medium | 8-Medium | 10-High |
| **1** | 1-Low | 2-Low | 3-Low | 4-Low | 5-Medium |

---

## 3. Risk Categories

---

## 4. Technical Risks (T-001 to T-020)

| ID | Risk Description | Probability | Impact | Score | Rating | Owner | Status |
|---|---|---|---|---|---|---|---|
| T-001 | Server infrastructure failure causing full platform outage | 3 | 5 | 15 | Critical | CTO | Open |
| T-002 | Database corruption or loss leading to data unavailability | 2 | 5 | 10 | High | DBA Lead | Open |
| T-003 | DDoS attack overwhelming platform capacity | 4 | 4 | 16 | Critical | Security Lead | Open |
| T-004 | API rate limiting insufficient preventing abuse | 3 | 3 | 9 | Medium | Backend Lead | Open |
| T-005 | Third-party API dependency failure (payment, SMS, maps) | 4 | 3 | 12 | High | Integration Lead | Open |
| T-006 | Mobile app critical bug preventing checkout | 3 | 5 | 15 | Critical | Mobile Lead | Open |
| T-007 | Search engine degradation affecting product discovery | 3 | 3 | 9 | Medium | Search Lead | Open |
| T-008 | CDN failure causing slow asset delivery | 3 | 2 | 6 | Medium | DevOps | Open |
| T-009 | SSL certificate expiry causing trust issues | 2 | 4 | 8 | Medium | DevOps | Mitigated |
| T-010 | Cache stampede causing database overload | 3 | 4 | 12 | High | Backend Lead | Open |
| T-011 | Memory leak causing gradual performance degradation | 3 | 3 | 9 | Medium | Backend Lead | Open |
| T-012 | Message queue backlog causing order processing delays | 3 | 4 | 12 | High | Backend Lead | Open |
| T-013 | Image storage failure preventing product display | 2 | 3 | 6 | Medium | DevOps | Open |
| T-014 | Email delivery failure affecting notifications | 3 | 2 | 6 | Medium | DevOps | Open |
| T-015 | Mobile app crash rate exceeding 1% threshold | 3 | 3 | 9 | Medium | Mobile Lead | Open |
| T-016 | Microservice communication failure | 2 | 4 | 8 | Medium | Backend Lead | Open |
| T-017 | Insufficient load testing before major releases | 3 | 3 | 9 | Medium | QA Lead | Open |
| T-018 | Configuration drift across environments | 3 | 2 | 6 | Medium | DevOps | Open |
| T-019 | Backup failure discovered only during recovery | 2 | 5 | 10 | High | DBA Lead | Open |
| T-020 | Deployment pipeline failure blocking releases | 3 | 2 | 6 | Medium | DevOps | Open |

---

## 5. Security Risks (S-001 to S-015)

| ID | Risk Description | Probability | Impact | Score | Rating | Owner | Status |
|---|---|---|---|---|---|---|---|
| S-001 | Data breach exposing customer PII | 2 | 5 | 10 | High | CISO | Open |
| S-002 | SQL injection vulnerability in critical endpoints | 2 | 5 | 10 | High | Security Lead | Open |
| S-003 | Cross-site scripting (XSS) attacks on web frontend | 3 | 3 | 9 | Medium | Frontend Lead | Open |
| S-004 | Man-in-the-middle attack on API communications | 2 | 4 | 8 | Medium | Security Lead | Mitigated |
| S-005 | Brute force attack on admin panel | 3 | 4 | 12 | High | Security Lead | Open |
| S-006 | Payment data compromise (PCI-DSS violation) | 2 | 5 | 10 | High | CISO | Open |
| S-007 | Unauthorized access to admin functions | 2 | 4 | 8 | Medium | Security Lead | Open |
| S-008 | API key exposure in client-side code | 3 | 3 | 9 | Medium | Security Lead | Open |
| S-009 | Session hijacking via insecure cookies | 2 | 4 | 8 | Medium | Security Lead | Mitigated |
| S-010 | Supply chain attack via third-party dependency | 2 | 4 | 8 | Medium | Security Lead | Open |
| S-011 | Insider threat (employee data theft) | 2 | 4 | 8 | Medium | HR + CISO | Open |
| S-012 | Ransomware attack encrypting production data | 2 | 5 | 10 | High | CISO | Open |
| S-013 | API abuse leading to data scraping | 4 | 2 | 8 | Medium | Security Lead | Open |
| S-014 | Credential stuffing attack on customer accounts | 4 | 3 | 12 | High | Security Lead | Open |
| S-015 | Inadequate encryption key rotation | 2 | 3 | 6 | Medium | Security Lead | Mitigated |

---

## 6. Business Risks (B-001 to B-020)

| ID | Risk Description | Probability | Impact | Score | Rating | Owner | Status |
|---|---|---|---|---|---|---|---|
| B-001 | Customer churn exceeding 15% monthly | 3 | 4 | 12 | High | CMO | Open |
| B-002 | Competitive pressure from new market entrants | 4 | 3 | 12 | High | CEO | Open |
| B-003 | Supplier reliability issues affecting inventory | 4 | 3 | 12 | High | Supply Chain Lead | Open |
| B-004 | Revenue decline due to economic factors in Yemen | 4 | 4 | 16 | Critical | CFO | Open |
| B-005 | Vendor/seller dissatisfaction leading to platform abandonment | 3 | 4 | 12 | High | Vendor Relations | Open |
| B-006 | Price war eroding profit margins | 4 | 3 | 12 | High | CFO | Open |
| B-007 | Failed product launch or feature rollout | 3 | 3 | 9 | Medium | Product Lead | Open |
| B-008 | Customer acquisition cost exceeding LTV | 3 | 4 | 12 | High | CMO | Open |
| B-009 | Negative brand reputation event (viral complaint) | 3 | 4 | 12 | High | CMO | Open |
| B-010 | Loss of key strategic partnership | 3 | 3 | 9 | Medium | CEO | Open |
| B-011 | Inability to scale for peak demand (Ramadan, sales) | 3 | 4 | 12 | High | CTO | Open |
| B-012 | Regulatory changes increasing operational costs | 3 | 3 | 9 | Medium | Legal | Open |
| B-013 | Currency fluctuation impacting pricing/profitability | 4 | 3 | 12 | High | CFO | Open |
| B-014 | Delivery network expansion failure | 3 | 3 | 9 | Medium | Logistics Lead | Open |
| B-015 | Failed international expansion | 2 | 3 | 6 | Medium | CEO | Open |
| B-016 | Customer payment defaults (COD orders) | 3 | 3 | 9 | Medium | Finance | Open |
| B-017 | Vendor fraud (counterfeit products) | 3 | 3 | 9 | Medium | Vendor Relations | Open |
| B-018 | Loss of market share to mobile-first competitors | 4 | 3 | 12 | High | CTO | Open |
| B-019 | Failed loyalty/rewards program | 3 | 2 | 6 | Medium | Marketing | Open |
| B-020 | Adverse media coverage | 3 | 3 | 9 | Medium | CMO | Open |

---

## 7. Operational Risks (O-001 to O-015)

| ID | Risk Description | Probability | Impact | Score | Rating | Owner | Status |
|---|---|---|---|---|---|---|---|
| O-001 | Warehouse fire or natural disaster destroying inventory | 1 | 5 | 5 | Medium | Operations Lead | Open |
| O-002 | Delivery fleet breakdown causing service disruption | 3 | 3 | 9 | Medium | Logistics Lead | Open |
| O-003 | Staff shortage during peak periods | 4 | 3 | 12 | High | HR Lead | Open |
| O-004 | Process failure causing order fulfillment errors | 3 | 3 | 9 | Medium | Operations Lead | Open |
| O-005 | Vendor onboarding bottleneck limiting growth | 3 | 3 | 9 | Medium | Vendor Relations | Open |
| O-006 | Customer support overwhelmed during incidents | 3 | 3 | 9 | Medium | Support Lead | Open |
| O-007 | Inventory management errors causing stockouts | 3 | 3 | 9 | Medium | Warehouse Lead | Open |
| O-008 | Quality control failure (damaged goods shipped) | 3 | 3 | 9 | Medium | QA Lead | Open |
| O-009 | Vendor dispute resolution delays | 3 | 2 | 6 | Medium | Legal | Open |
| O-010 | Returns/refunds process failure | 3 | 3 | 9 | Medium | Operations Lead | Open |
| O-011 | Driver/rider safety incidents | 3 | 3 | 9 | Medium | Logistics Lead | Open |
| O-012 | Cash handling errors (COD operations) | 3 | 2 | 6 | Medium | Finance | Open |
| O-013 | Communication breakdown between departments | 3 | 2 | 6 | Medium | COO | Open |
| O-014 | Knowledge loss due to key employee departure | 3 | 3 | 9 | Medium | HR Lead | Open |
| O-015 | Vendor payment processing delays | 3 | 3 | 9 | Medium | Finance | Open |

---

## 8. Financial Risks (F-001 to F-012)

| ID | Risk Description | Probability | Impact | Score | Rating | Owner | Status |
|---|---|---|---|---|---|---|---|
| F-001 | Cash flow crisis due to delayed payments | 3 | 4 | 12 | High | CFO | Open |
| F-002 | Payment gateway failure blocking transactions | 2 | 5 | 10 | High | Finance Lead | Open |
| F-003 | Fraudulent transactions (stolen cards, identity theft) | 3 | 4 | 12 | High | Security Lead | Open |
| F-004 | Tax compliance failure resulting in penalties | 2 | 4 | 8 | Medium | Finance | Open |
| F-005 | Vendor payment disputes causing legal costs | 3 | 2 | 6 | Medium | Legal | Open |
| F-006 | Currency exchange losses on international transactions | 3 | 2 | 6 | Medium | Finance | Open |
| F-007 | Refund abuse (return fraud) | 4 | 2 | 8 | Medium | Fraud Team | Open |
| F-008 | Commission calculation errors | 2 | 3 | 6 | Medium | Finance | Open |
| F-009 | Insurance coverage gaps | 2 | 3 | 6 | Medium | CFO | Open |
| F-010 | Revenue leakage from未计价 orders | 3 | 3 | 9 | Medium | Finance | Open |
| F-011 | Failed funding/investment round | 3 | 4 | 12 | High | CEO | Open |
| F-012 | Audit findings requiring remediation costs | 2 | 3 | 6 | Medium | CFO | Open |

---

## 9. Compliance & Legal Risks (C-001 to C-010)

| ID | Risk Description | Probability | Impact | Score | Rating | Owner | Status |
|---|---|---|---|---|---|---|---|
| C-001 | Non-compliance with Yemen e-commerce regulations | 3 | 4 | 12 | High | Legal | Open |
| C-002 | GDPR violation for EU-based customers | 2 | 4 | 8 | Medium | DPO | Open |
| C-003 | PCI-DSS non-compliance | 2 | 5 | 10 | High | CISO | Open |
| C-004 | Tax audit resulting in penalties | 3 | 3 | 9 | Medium | Finance | Open |
| C-005 | Intellectual property infringement claim | 2 | 3 | 6 | Medium | Legal | Open |
| C-006 | Consumer protection law violation | 2 | 3 | 6 | Medium | Legal | Open |
| C-007 | Anti-money laundering compliance failure | 2 | 4 | 8 | Medium | Compliance | Open |
| C-008 | Data localization requirement violation | 2 | 3 | 6 | Medium | CTO | Open |
| C-009 | License/permit renewal failure | 1 | 4 | 4 | Low | Legal | Open |
| C-010 | Employment law violation | 2 | 3 | 6 | Medium | HR Lead | Open |

---

## 10. Human Resource Risks (H-001 to H-008)

| ID | Risk Description | Probability | Impact | Score | Rating | Owner | Status |
|---|---|---|---|---|---|---|---|
| H-001 | Key technical talent departure | 4 | 3 | 12 | High | CTO | Open |
| H-002 | Inability to recruit specialized talent | 4 | 3 | 12 | High | HR Lead | Open |
| H-003 | Low employee morale affecting productivity | 3 | 3 | 9 | Medium | HR Lead | Open |
| H-004 | Security training gaps leading to vulnerabilities | 3 | 3 | 9 | Medium | CISO | Open |
| H-005 | Succession planning gaps for critical roles | 3 | 3 | 9 | Medium | CEO | Open |
| H-006 | Remote work security risks | 3 | 2 | 6 | Medium | Security Lead | Open |
| H-007 | Contractor/vendor reliability issues | 3 | 2 | 6 | Medium | Procurement | Open |
| H-008 | Knowledge silos in critical systems | 3 | 3 | 9 | Medium | CTO | Open |

---

## 11. Infrastructure Risks (I-001 to I-008)

| ID | Risk Description | Probability | Impact | Score | Rating | Owner | Status |
|---|---|---|---|---|---|---|---|
| I-001 | Internet connectivity disruption in Yemen | 4 | 4 | 16 | Critical | CTO | Open |
| I-002 | Power outage affecting operations | 4 | 3 | 12 | High | Operations Lead | Open |
| I-003 | Cloud provider service degradation | 3 | 3 | 9 | Medium | DevOps | Open |
| I-004 | Hardware failure in on-premise systems | 2 | 3 | 6 | Medium | DevOps | Open |
| I-005 | Network security breach (perimeter) | 2 | 4 | 8 | Medium | Security Lead | Open |
| I-006 | DNS hijacking or manipulation | 2 | 4 | 8 | Medium | Security Lead | Mitigated |
| I-007 | Physical security breach at facilities | 2 | 3 | 6 | Medium | Operations Lead | Open |
| I-008 | Telecommunications provider failure | 3 | 3 | 9 | Medium | CTO | Open |

---

## 12. Risk Summary Dashboard

### 12.1 By Rating
| Rating | Count | Percentage |
|---|---|---|
| Critical (15-25) | 5 | 4.3% |
| High (10-14) | 32 | 27.6% |
| Medium (5-9) | 56 | 48.3% |
| Low (1-4) | 23 | 19.8% |
| **Total** | **116** | **100%** |

### 12.2 By Category
| Category | Risks | Avg Score | Top Risk |
|---|---|---|---|
| Technical | 20 | 9.4 | T-003 DDoS Attack (16) |
| Security | 15 | 9.1 | S-001 Data Breach (10) |
| Business | 20 | 10.5 | B-004 Economic Factors (16) |
| Operational | 15 | 8.5 | O-003 Staff Shortage (12) |
| Financial | 12 | 8.3 | F-001 Cash Flow (12) |
| Compliance | 10 | 8.0 | C-001 Regulation (12) |
| HR | 8 | 9.0 | H-001 Talent Departure (12) |
| Infrastructure | 8 | 9.8 | I-001 Internet Disruption (16) |

### 12.3 Top 10 Risks by Score
| Rank | ID | Risk | Score | Category |
|---|---|---|---|---|
| 1 | T-003 | DDoS Attack | 16 | Technical |
| 2 | B-004 | Economic Factors | 16 | Business |
| 3 | I-001 | Internet Disruption | 16 | Infrastructure |
| 4 | T-001 | Server Infrastructure Failure | 15 | Technical |
| 5 | T-006 | Mobile App Checkout Bug | 15 | Technical |
| 6 | B-005 | Vendor/Supplier Issues | 12 | Business |
| 7 | B-011 | Peak Demand Scaling | 12 | Business |
| 8 | H-001 | Key Talent Departure | 12 | HR |
| 9 | I-002 | Power Outage | 12 | Infrastructure |
| 10 | F-001 | Cash Flow Crisis | 12 | Financial |

---

## 13. Risk Register Maintenance

| Activity | Frequency | Responsible |
|---|---|---|
| Risk review meeting | Monthly | Risk Committee |
| New risk identification | Quarterly (workshop) | All stakeholders |
| Risk score recalculation | Quarterly | Risk Manager |
| Mitigation status update | Bi-weekly | Risk Owners |
| Full register audit | Annually | External auditor |
| Register publication | Monthly | Risk Manager |
