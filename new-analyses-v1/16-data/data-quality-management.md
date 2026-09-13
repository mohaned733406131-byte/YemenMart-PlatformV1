# Data Quality Management — YemenMart

## 1. Overview

Data quality directly impacts business outcomes: order accuracy, customer satisfaction, inventory optimization, and financial reporting. This framework ensures data meets quality standards across all domains through validation, cleansing, monitoring, and continuous improvement.

---

## 2. Quality Dimensions

### 2.1 Core Dimensions
| Dimension | Definition | Target | Measurement Method |
|---|---|---|---|
| **Completeness** | Required fields populated | > 98% | Null check on required fields |
| **Accuracy** | Data matches real-world values | > 99% | Validation against source/reference |
| **Consistency** | Same data across systems | > 99.5% | Cross-system reconciliation |
| **Timeliness** | Data available within expected time | < 5 min latency | Timestamp delta measurement |
| **Uniqueness** | No unwanted duplicates | > 99.9% | Deduplication detection |
| **Validity** | Data conforms to format/rules | > 99% | Schema + business rule validation |
| **Integrity** | Referential relationships intact | 100% | FK constraint checks |

### 2.2 Domain-Specific Quality Rules
| Domain | Key Quality Rule | Validation |
|---|---|---|
| Customers | Valid phone format (+967-XXXXXXXXX) | Regex validation |
| Customers | Unique email per account | Uniqueness check |
| Orders | Order total = sum of line items | Arithmetic check |
| Orders | Valid status transitions | State machine validation |
| Products | Price > 0, stock >= 0 | Range check |
| Products | SKU format: CAT-XXXX-XX | Format validation |
| Payments | Amount matches order total | Cross-reference check |
| Inventory | Physical count = system count | Reconciliation |

---

## 3. Validation Framework

### 3.1 Validation Layers
```
┌─────────────────────────────────────────────┐
│            Layer 1: Input Validation         │
│   (API boundary, form validation)            │
├─────────────────────────────────────────────┤
│            Layer 2: Business Rules           │
│   (Domain logic, constraints)                │
├─────────────────────────────────────────────┤
│            Layer 3: Cross-system             │
│   (Reconciliation, consistency)              │
├─────────────────────────────────────────────┤
│            Layer 4: Analytics                │
│   (Anomaly detection, statistical)           │
└─────────────────────────────────────────────┘
```

### 3.2 Validation Rules Engine
| Rule Type | Example | Action on Failure |
|---|---|---|
| Format | Phone: `+967-\d{8,9}` | Reject input |
| Range | Price: 0 < price < 1,000,000 | Reject input |
| Uniqueness | Email unique per user | Reject input |
| Referential | Product ID exists in catalog | Reject input |
| Business logic | Order status follows allowed transitions | Reject state change |
| Cross-field | Delivery date > order date | Warning + confirm |
| Statistical | Order amount within 3σ of mean | Flag for review |
| Temporal | Created timestamp ≤ Updated timestamp | Auto-correct |

### 3.3 Validation Implementation
```python
# Validation Pipeline (per record)
class DataValidator:
    def validate(self, record, schema):
        errors = []
        
        # Layer 1: Schema validation
        errors += self.validate_schema(record, schema)
        
        # Layer 2: Business rules
        errors += self.validate_business_rules(record)
        
        # Layer 3: Cross-reference
        errors += self.validate_references(record)
        
        # Layer 4: Statistical
        errors += self.validate_anomalies(record)
        
        return ValidationResult(
            valid=len(errors) == 0,
            errors=errors,
            warnings=self.get_warnings(record)
        )
```

---

## 4. Data Cleansing

### 4.1 Cleansing Rules
| Issue | Detection | Resolution | Automation |
|---|---|---|---|
| Duplicate records | Fuzzy match (name+phone) | Merge, keep latest | Semi-auto |
| Invalid phone format | Regex check | Normalize format | Auto |
| Invalid email format | Regex check | Flag for user update | Auto-flag |
| Missing required fields | Null check | Prompt user / default | Context-dependent |
| Outdated addresses | Last updated > 1 year | Prompt user to verify | Auto-prompt |
| Orphaned records | FK violation | Link or remove | Manual review |
| Inconsistent casing | Pattern detection | Normalize (Title Case) | Auto |
| Extra whitespace | Pattern detection | Trim | Auto |
| Invalid dates | Range check | Flag for review | Auto-flag |
| Negative quantities | Range check | Flag for review | Auto-flag |

### 4.2 Cleansing Pipeline
```
Raw Data → Profiling → Rule Matching → Cleansing → Validation → Clean Data
              ↓              ↓              ↓            ↓
         Quality Report  Issue Log    Change Log   Quality Score
```

### 4.3 Cleansing Schedule
| Data Type | Frequency | Method | Responsible |
|---|---|---|---|
| Customer profiles | Weekly | Automated + manual | Data team |
| Product catalog | Daily | Automated | System |
| Order data | Real-time | Validation at entry | System |
| Inventory | Daily | Reconciliation with physical | Warehouse |
| Addresses | On user interaction | Validation + enrichment | System |
| Phone numbers | Monthly | Format normalization | System |

---

## 5. Duplicate Detection

### 5.1 Matching Rules
| Record Type | Match Fields | Threshold | Action |
|---|---|---|---|
| Customers | Phone (exact) | 100% | Auto-merge |
| Customers | Email (exact) | 100% | Auto-merge |
| Customers | Name + City (fuzzy) | 85% | Flag for review |
| Products | SKU (exact) | 100% | Auto-merge |
| Products | Name + Category (fuzzy) | 90% | Flag for review |
| Addresses | Full address (fuzzy) | 90% | Flag for review |
| Orders | Unique ID | 100% | N/A (unique by design) |

### 5.2 Merge Strategy
```
Duplicate Detected
      ↓
Score ≥ 95% → Auto-merge (keep most complete record)
Score 85-94% → Queue for manual review
Score < 85% → Ignore (likely different entities)
```

### 5.3 Merge Rules
- Preserve all unique data from both records
- Keep most recent update for conflicting fields
- Maintain audit trail of merge operation
- Preserve relationships (orders, support tickets)
- Notify affected users of profile consolidation

---

## 6. Monitoring & Alerting

### 6.1 Quality Monitoring Dashboard
| Metric | Visual | Refresh | Alert Threshold |
|---|---|---|---|
| Overall Quality Score | Gauge | Hourly | < 95% |
| Completeness Rate | Trend line | Hourly | < 98% |
| Duplicate Rate | Bar chart | Daily | > 0.1% |
| Validation Errors | Counter | Real-time | > 10/hour |
| Cleansing Actions | Log stream | Real-time | Anomaly |
| Stale Data | Heat map | Daily | > 7 days old |

### 6.2 Alert Rules
| Alert | Condition | Severity | Channel | SLA |
|---|---|---|---|---|
| Quality drop | Score < 95% | High | PagerDuty | 1 hour |
| Spike in errors | > 50 errors/hour | Medium | Slack | 4 hours |
| Duplicate surge | > 0.5% duplicates | Medium | Slack | 24 hours |
| Stale data | > 24 hours old | Low | Email | 48 hours |
| Constraint violation | Any FK violation | Critical | PagerDuty | 30 minutes |

### 6.3 Quality Scorecard
```json
{
  "date": "2024-01-15",
  "overall_score": 97.8,
  "dimensions": {
    "completeness": 98.2,
    "accuracy": 99.1,
    "consistency": 99.5,
    "timeliness": 98.8,
    "uniqueness": 99.9,
    "validity": 99.3
  },
  "domains": {
    "customers": 97.5,
    "products": 98.9,
    "orders": 99.2,
    "inventory": 96.8,
    "payments": 99.7
  },
  "issues_found": 145,
  "issues_resolved": 138,
  "trend": "improving"
}
```

---

## 7. Data Profiling

### 7.1 Profiling Dimensions
| Profile Aspect | Metrics | Frequency |
|---|---|---|
| Column statistics | Min, max, mean, median, std dev | Weekly |
| Value distribution | Histogram, frequency table | Weekly |
| Pattern analysis | Regex patterns, formats | Monthly |
| Relationship analysis | FK compliance, orphan rates | Weekly |
| Temporal analysis | Creation/update patterns | Weekly |
| Completeness analysis | Null rates per field | Daily |

### 7.2 Profiling Report
| Table | Rows | Completeness | Uniqueness | Validity | Quality Score |
|---|---|---|---|---|---|
| customers | 125,000 | 98.5% | 99.9% | 99.2% | 97.9% |
| products | 45,000 | 99.1% | 100% | 99.8% | 99.6% |
| orders | 890,000 | 99.8% | 100% | 99.9% | 99.9% |
| inventory | 45,000 | 97.2% | 100% | 98.5% | 98.2% |
| payments | 890,000 | 100% | 100% | 100% | 100% |

---

## 8. Reference Data Management

### 8.1 Reference Data Domains
| Domain | Examples | Update Frequency | Source |
|---|---|---|---|
| Countries | Country codes, names | Annually | ISO 3166 |
| Cities/Regions | Yemen governorates, cities | Semi-annually | Government |
| Categories | Product category hierarchy | Monthly | Business |
| Currencies | YER, USD exchange rates | Daily | Central Bank |
| Tax Rates | VAT, sales tax | Per regulation | Government |
| Status Codes | Order/payment statuses | As needed | Internal |

### 8.2 Reference Data Quality
- Master data managed in dedicated tables
- Version-controlled with effective dates
- Validated against authoritative sources
- Synchronized across all consuming systems
- Changes audited and reviewed

---

## 9. Data Quality Tools

### 9.1 Tool Stack
| Tool | Purpose | Integration |
|---|---|---|
| Great Expectations | Data validation framework | ETL pipeline |
| dbt Tests | In-database quality checks | Data warehouse |
| Custom validators | Business-specific rules | Application layer |
| Grafana | Quality dashboards | Monitoring |
| Python (pandas) | Profiling scripts | Scheduled jobs |
| SQL queries | Manual validation | Ad-hoc analysis |

### 9.2 Validation Test Suite
```sql
-- Completeness check
SELECT 
  COUNT(*) as total,
  COUNT(phone) as has_phone,
  ROUND(COUNT(phone)::decimal / COUNT(*) * 100, 2) as completeness
FROM customers;

-- Uniqueness check
SELECT email, COUNT(*) 
FROM customers 
GROUP BY email 
HAVING COUNT(*) > 1;

-- Referential integrity
SELECT o.id 
FROM orders o 
LEFT JOIN customers c ON o.customer_id = c.id 
WHERE c.id IS NULL;

-- Range validation
SELECT * FROM products WHERE price <= 0 OR stock < 0;
```

---

## 10. Continuous Improvement

### 10.1 Quality Improvement Cycle
```
Measure → Analyze → Improve → Monitor → Repeat
    ↑                                      │
    └──────────────────────────────────────┘
```

### 10.2 Improvement Actions
| Issue Type | Root Cause | Improvement | Owner |
|---|---|---|---|
| Duplicate customers | Multiple registration paths | Single sign-on | Engineering |
| Invalid addresses | No validation at entry | Address validation API | Product |
| Stale inventory | Delayed sync | Real-time inventory sync | Operations |
| Missing phone numbers | Optional field | Make required for orders | Product |
| Inconsistent product names | Manual entry | Standardized naming rules | Catalog team |

### 10.3 Quality Reviews
| Review | Frequency | Participants | Output |
|---|---|---|---|
| Daily quality check | Daily | Data ops | Automated report |
| Weekly quality review | Weekly | Data team | Issue triage |
| Monthly quality report | Monthly | Stakeholders | Scorecard |
| Quarterly deep dive | Quarterly | Cross-functional | Improvement plan |
| Annual quality audit | Annually | External auditor | Compliance report |

---

## 11. Data Quality SLAs

| Metric | SLA | Measurement | Escalation |
|---|---|---|---|
| Order data accuracy | 99.9% | Per order validation | PagerDuty |
| Payment data accuracy | 100% | Per transaction | PagerDuty |
| Customer data completeness | > 98% | Daily profile check | Email to data team |
| Product data accuracy | > 99% | Daily catalog check | Slack #catalog |
| Inventory accuracy | > 99% | Daily reconciliation | Warehouse manager |
| Delivery address accuracy | > 95% | Per delivery | Operations |
| Duplicate detection rate | < 0.1% | Weekly scan | Data team |

---

## 12. Roles & Responsibilities

| Role | Responsibilities |
|---|---|
| Data Quality Manager | Overall quality strategy, reporting, escalation |
| Data Engineers | Pipeline quality, validation implementation |
| Data Stewards | Domain-specific quality rules, issue resolution |
| Product Managers | Input validation UX, field requirements |
| Customer Support | User-reported data issues triage |
| Engineering Leads | Application-level validation, API contracts |
