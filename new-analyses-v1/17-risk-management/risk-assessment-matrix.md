# Risk Assessment Matrix — YemenMart

## 1. Overview

This document defines the probability-impact framework used to evaluate and prioritize all risks identified in the YemenMart Risk Register. It provides a systematic methodology for risk scoring, categorization, and response planning.

---

## 2. Probability Assessment

### 2.1 Probability Scale
| Score | Rating | Likelihood | Historical Basis | Timeframe |
|---|---|---|---|---|
| 1 | Rare | < 5% | No similar incidents in industry | > 5 years |
| 2 | Unlikely | 5-20% | Few incidents in similar platforms | 2-5 years |
| 3 | Possible | 20-50% | Some incidents in industry | 1-2 years |
| 4 | Likely | 50-80% | Regular incidents observed | 6-12 months |
| 5 | Almost Certain | > 80% | Expected to occur | < 6 months |

### 2.2 Probability Assessment Factors
| Factor | Weight | Assessment Criteria |
|---|---|---|
| Historical frequency | 30% | How often has this occurred? |
| Industry prevalence | 25% | How common in e-commerce? |
| Control effectiveness | 20% | Current mitigation strength |
| Threat landscape | 15% | Current environment trends |
| Organizational exposure | 10% | Our specific vulnerability |

### 2.3 Probability Calibration Benchmarks
| Rating | Technical Example | Business Example |
|---|---|---|
| 1 (Rare) | Zero-day exploit on our stack | Complete market collapse |
| 2 (Unlikely) | Database corruption | Key supplier bankruptcy |
| 3 (Possible) | DDoS attack (moderate) | Customer churn spike |
| 4 (Likely) | Third-party API outage | Seasonal demand surge |
| 5 (Almost Certain) | Minor API rate limit hit | Price competition pressure |

---

## 3. Impact Assessment

### 3.1 Impact Scale
| Score | Rating | Financial | Operational | Reputational | Legal/Compliance |
|---|---|---|---|---|---|
| 1 | Negligible | < $1K | < 1 hour | No coverage | No impact |
| 2 | Minor | $1K-$10K | 1-4 hours | Local coverage | Minor finding |
| 3 | Moderate | $10K-$50K | 4-24 hours | Regional coverage | Significant finding |
| 4 | Major | $50K-$200K | 1-7 days | National coverage | Regulatory action |
| 5 | Catastrophic | > $200K | > 7 days | International coverage | Severe penalties |

### 3.2 Impact Dimensions
| Dimension | Weight | Measurement |
|---|---|---|
| Financial Loss | 30% | Direct + indirect costs |
| Operational Disruption | 25% | Downtime, service degradation |
| Reputational Damage | 20% | Brand perception, trust |
| Legal/Compliance | 15% | Regulatory, contractual |
| Strategic Impact | 10% | Long-term competitive position |

### 3.3 Impact Assessment by Category
#### Financial Impact
| Score | Description | Example |
|---|---|---|
| 1 | < $1K loss | Minor billing error |
| 2 | $1K-$10K loss | Small fraud incident |
| 3 | $10K-$50K loss | Payment processing failure |
| 4 | $50K-$200K loss | Major data breach remediation |
| 5 | > $200K loss | Extended platform outage |

#### Operational Impact
| Score | Description | Example |
|---|---|---|
| 1 | < 1 hour disruption | Brief API latency spike |
| 2 | 1-4 hours disruption | Single service outage |
| 3 | 4-24 hours disruption | Multiple service degradation |
| 4 | 1-7 days disruption | Major system failure |
| 5 | > 7 days disruption | Complete platform failure |

#### Reputational Impact
| Score | Description | Example |
|---|---|---|
| 1 | No external visibility | Internal incident only |
| 2 | Limited social media mention | A few customer complaints |
| 3 | Regional news coverage | Local media report |
| 4 | National news coverage | Major news outlet story |
| 5 | International coverage | Global tech media story |

---

## 4. Risk Matrix

### 4.1 Full 5×5 Matrix
```
                        IMPACT
                 1      2      3      4      5
              Neglig  Minor  Mod    Major  Cata
         ┌──────┬──────┬──────┬──────┬──────┐
    5    │  5   │  10  │  15  │  20  │  25  │
    P    │ MED  │ HIGH │ CRIT │ CRIT │ CRIT │
    r    ├──────┼──────┼──────┼──────┼──────┤
    o 4  │  4   │   8  │  12  │  16  │  20  │
    b    │ LOW  │ MED  │ HIGH │ CRIT │ CRIT │
    a    ├──────┼──────┼──────┼──────┼──────┤
    b 3  │  3   │   6  │   9  │  12  │  15  │
    i    │ LOW  │ MED  │ MED  │ HIGH │ CRIT │
    l    ├──────┼──────┼──────┼──────┼──────┤
    i 2  │  2   │   4  │   6  │   8  │  10  │
    t    │ LOW  │ LOW  │ MED  │ MED  │ HIGH │
    y    ├──────┼──────┼──────┼──────┼──────┤
    1    │  1   │   2  │   3  │   4  │   5  │
         │ LOW  │ LOW  │ LOW  │ LOW  │ MED  │
         └──────┴──────┴──────┴──────┴──────┘
```

### 4.2 Risk Distribution
| Rating | Score Range | Color | Response Required |
|---|---|---|---|
| Critical | 15-25 | Red | Immediate action, executive oversight |
| High | 10-14 | Orange | Priority action, management review |
| Medium | 5-9 | Yellow | Planned action, team-level |
| Low | 1-4 | Green | Monitor, accept or minimal action |

---

## 5. Current Risk Distribution

### 5.1 By Rating
| Rating | Count | % of Total | Action Required |
|---|---|---|---|
| Critical | 5 | 4.3% | Immediate mitigation plan |
| High | 32 | 27.6% | Active mitigation within 30 days |
| Medium | 56 | 48.3% | Planned mitigation within 90 days |
| Low | 23 | 19.8% | Monitor quarterly |
| **Total** | **116** | **100%** | |

### 5.2 By Category × Rating
| Category | Critical | High | Medium | Low | Total |
|---|---|---|---|---|---|
| Technical | 3 | 6 | 9 | 2 | 20 |
| Security | 0 | 7 | 6 | 2 | 15 |
| Business | 2 | 8 | 8 | 2 | 20 |
| Operational | 0 | 4 | 9 | 2 | 15 |
| Financial | 0 | 5 | 5 | 2 | 12 |
| Compliance | 0 | 3 | 5 | 2 | 10 |
| HR | 0 | 3 | 4 | 1 | 8 |
| Infrastructure | 1 | 3 | 3 | 1 | 8 |
| **Total** | **5** | **32** | **56** | **23** | **116** |

### 5.3 Risk Heat Map
```
        CRITICAL (5)  ████████████████████
        HIGH (32)     ████████████████████████████████
        MEDIUM (56)   ████████████████████████████████████████████████████████
        LOW (23)      ███████████████████████
                       ─────────────────────────────────────────────────────
                       0    10    20    30    40    50    60
```

---

## 6. Response Strategies

### 6.1 Response Types
| Strategy | Description | When to Use |
|---|---|---|
| **Avoid** | Eliminate the risk entirely | High-impact risk with alternative approach |
| **Mitigate** | Reduce probability or impact | Most risks - primary strategy |
| **Transfer** | Shift risk to third party | Financial, legal, insurance |
| **Accept** | Acknowledge and monitor | Low-impact risks, cost-benefit |
| **Escalate** | Move to higher authority | Critical risks, executive decisions |

### 6.2 Response by Rating
| Rating | Primary Strategy | Response Time | Approver | Review Frequency |
|---|---|---|---|---|
| Critical | Avoid/Mitigate | 24 hours | Executive | Weekly |
| High | Mitigate | 7 days | Director | Bi-weekly |
| Medium | Mitigate/Transfer | 30 days | Manager | Monthly |
| Low | Accept/Monitor | 90 days | Team Lead | Quarterly |

### 6.3 Response Plan Template
```
Risk ID: [ID]
Risk Description: [Description]
Current Rating: [P×I = Score]

Response Strategy: [Avoid/Mitigate/Transfer/Accept]
Action Plan:
  1. [Action item] - Owner: [Name] - Due: [Date]
  2. [Action item] - Owner: [Name] - Due: [Date]
  3. [Action item] - Owner: [Name] - Due: [Date]

Expected Rating After Response: [P×I = Score]
Cost of Response: [$]
Residual Risk: [Description]
Monitoring Plan: [How/When to monitor]
Escalation Path: [When to escalate]
```

---

## 7. Critical Risk Deep Dive

### 7.1 Critical Risks (Score ≥ 15)
| ID | Risk | P | I | Score | Response Owner | Deadline |
|---|---|---|---|---|---|---|
| T-003 | DDoS Attack | 4 | 4 | 16 | Security Lead | Immediate |
| B-004 | Economic Factors | 4 | 4 | 16 | CFO | 30 days |
| I-001 | Internet Disruption | 4 | 4 | 16 | CTO | Immediate |
| T-001 | Server Infrastructure Failure | 3 | 5 | 15 | CTO | 7 days |
| T-006 | Mobile App Checkout Bug | 3 | 5 | 15 | Mobile Lead | 7 days |

### 7.2 Critical Risk Response Plans

#### T-003: DDoS Attack (Score: 16)
**Strategy**: Mitigate
**Actions**:
1. Deploy CDN with DDoS protection (Cloudflare/AWS Shield) - Security Lead - Week 1
2. Implement rate limiting at API gateway - Backend Lead - Week 2
3. Configure auto-scaling for traffic spikes - DevOps - Week 2
4. Establish DDoS response playbook - Security Lead - Week 3
5. Conduct DDoS simulation drill - Security Lead - Week 4

**Target Rating**: P:4→2, I:4→3 = Score: 6 (Medium)

#### B-004: Economic Factors (Score: 16)
**Strategy**: Mitigate
**Actions**:
1. Diversify revenue streams (B2B, marketplace) - CEO - Month 1
2. Build 6-month cash reserve - CFO - Month 3
3. Optimize cost structure (15% reduction target) - COO - Month 2
4. Develop economic scenario plans - CFO - Month 1
5. Establish banking relationships for credit lines - CFO - Month 2

**Target Rating**: P:4→3, I:4→3 = Score: 9 (Medium)

#### I-001: Internet Disruption (Score: 16)
**Strategy**: Mitigate + Accept
**Actions**:
1. Implement offline-first mobile architecture - CTO - Month 1
2. Establish redundant ISP connections - DevOps - Week 2
3. Deploy local CDN caches - DevOps - Week 3
4. Create offline mode documentation - Product - Week 2
5. Test failover procedures monthly - DevOps - Ongoing

**Target Rating**: P:4→3, I:4→3 = Score: 9 (Medium)

---

## 8. Risk Trend Analysis

### 8.1 Historical Trends
| Quarter | Critical | High | Medium | Low | Total | Avg Score |
|---|---|---|---|---|---|---|
| Q1 2024 | 7 | 28 | 52 | 29 | 116 | 8.9 |
| Q2 2024 | 6 | 30 | 54 | 26 | 116 | 9.1 |
| Q3 2024 | 5 | 32 | 56 | 23 | 116 | 9.4 |
| Q4 2024 | 5 | 32 | 56 | 23 | 116 | 9.4 |

### 8.2 Trend Indicators
| Metric | Direction | Significance |
|---|---|---|
| Critical risks | ↓ Decreasing | Positive |
| High risks | ↑ Increasing | Requires attention |
| Average score | ↑ Slight increase | Monitor closely |
| Mitigated risks | ↑ Increasing | Program effective |

---

## 9. Risk Appetite Statement

### 9.1 Appetite by Category
| Category | Appetite | Tolerance | Threshold |
|---|---|---|---|
| Technical | Low | Medium score | High score triggers action |
| Security | Very Low | Low score | Medium score triggers action |
| Business | Medium | High score | Critical score triggers action |
| Operational | Low | Medium score | High score triggers action |
| Financial | Low | Medium score | High score triggers action |
| Compliance | Very Low | Low score | Medium score triggers action |

### 9.2 Risk Appetite Boundaries
- **Zero Tolerance**: Data breach, regulatory violation, safety incident
- **Low Tolerance**: Extended outages, financial fraud, compliance gaps
- **Medium Tolerance**: Performance degradation, competitive pressure
- **High Tolerance**: Minor incidents, isolated errors, cost overruns

---

## 10. Assessment Process

### 10.1 Assessment Workflow
```
Risk Identified → Initial Assessment → Scoring → Categorization
        ↓                                              ↓
  Risk Owner Assigned ←──── Review ←──── Response Planning
        ↓                                              ↓
  Implementation ←──── Monitoring ←──── Status Updates
        ↓                                              ↓
  Closure/Retirement ←──── Lessons Learned ←──── Post-Mortem
```

### 10.2 Assessment Criteria
| Factor | Weight | Scoring Method |
|---|---|---|
| Historical data | 30% | Incident database analysis |
| Expert judgment | 25% | Delphi method, workshops |
| Industry benchmarks | 20% | Reports, surveys |
| Control assessment | 15% | Audit findings, maturity model |
| Scenario analysis | 10% | What-if modeling |

### 10.3 Assessment Schedule
| Activity | Frequency | Participants | Output |
|---|---|---|---|
| Quick assessment | As needed | Risk owner | Score update |
| Formal assessment | Quarterly | Risk committee | Full matrix review |
| Annual review | Annually | Executive team | Risk appetite recalibration |
| Post-incident | Per incident | Incident team | Risk score update |
| New risk assessment | Per identification | Risk manager | New risk entry |

---

## 11. Reporting

### 11.1 Risk Dashboard Metrics
| Metric | Current | Target | Status |
|---|---|---|---|
| Total risks | 116 | - | - |
| Critical risks | 5 | < 3 | Behind |
| High risks | 32 | < 25 | Behind |
| Average risk score | 9.4 | < 8.0 | Behind |
| Risks mitigated (YTD) | 45 | 60 | Behind |
| New risks identified | 18 | - | - |
| Risks closed | 12 | - | - |

### 11.2 Report Distribution
| Report | Audience | Frequency | Format |
|---|---|---|---|
| Risk Dashboard | Executive team | Monthly | Visual dashboard |
| Risk Register | Risk committee | Monthly | Excel + Summary |
| Critical Risk Report | CEO, Board | Weekly | PDF |
| Mitigation Status | Risk owners | Bi-weekly | Email + Dashboard |
| Risk Assessment | All stakeholders | Quarterly | Presentation |
