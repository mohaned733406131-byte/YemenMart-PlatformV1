# Data Retention Policy — YemenMart

## 1. Overview

This policy defines how long YemenMart retains data, when it is archived, and when it is permanently deleted. It balances operational needs, legal requirements, user rights, and storage efficiency.

---

## 2. Regulatory Framework

### 2.1 Applicable Regulations
| Regulation | Jurisdiction | Key Requirement |
|---|---|---|
| Yemen E-Commerce Law | Yemen | Transaction records 7 years |
| Yemen Tax Law | Yemen | Financial records 7 years |
| PCI-DSS | Global | Card data 1 year (minimum) |
| GDPR | EU Users | Minimization, right to deletion |
| CCPA | California Users | Right to delete, disclosure |
| Yemen Data Protection Draft | Yemen | Emerging requirements |

---

## 3. Retention Schedule

### 3.1 Customer Data
| Data Type | Retention | Archive After | Delete After | Legal Basis |
|---|---|---|---|---|
| Profile (active) | While account active | N/A | On deletion request | Consent |
| Profile (inactive) | 3 years from last login | 3 years | 7 years | Legitimate interest |
| Phone numbers | While account active | N/A | On deletion request | Contract |
| Email addresses | While account active | N/A | On deletion request | Contract |
| Addresses | While account active | N/A | On deletion request | Contract |
| Order history | 7 years | 3 years | 7 years | Legal obligation |
| Support tickets | 3 years | 1 year | 3 years | Legitimate interest |
| Feedback/Reviews | Indefinite (anonymized) | N/A | Anonymize at 3 years | Consent |
| Marketing consent | While active + 2 years | N/A | On withdrawal | Consent |
| Session data | 30 days | N/A | 30 days | Legitimate interest |

### 3.2 Transaction Data
| Data Type | Retention | Archive After | Delete After | Legal Basis |
|---|---|---|---|---|
| Orders | 7 years | 2 years | 7 years | Legal obligation |
| Invoices | 7 years | 3 years | 7 years | Tax regulation |
| Payments | 7 years | 2 years | 7 years | PCI-DSS + Tax |
| Refunds | 7 years | 2 years | 7 years | Legal obligation |
| Tax records | 7 years | 3 years | 7 years | Tax regulation |
| Commission records | 7 years | 3 years | 7 years | Contract |
| Payment tokens | Until expired | N/A | On expiry | PCI-DSS |

### 3.3 Product Data
| Data Type | Retention | Archive After | Delete After | Legal Basis |
|---|---|---|---|---|
| Product listings | While active | 1 year after delist | Indefinite archive | Business need |
| Product images | While active | 1 year after delist | 3 years after delist | Business need |
| Price history | 7 years | 3 years | 7 years | Legal obligation |
| Inventory records | 7 years | 3 years | 7 years | Legal obligation |
| Supplier data | While active + 5 years | 3 years after end | 7 years after end | Contract |
| Product reviews | Indefinite (anonymized) | N/A | Anonymize at 3 years | Consent |

### 3.4 Operational Data
| Data Type | Retention | Archive After | Delete After | Legal Basis |
|---|---|---|---|---|
| Audit logs | 7 years | 2 years | 7 years | Compliance |
| Security logs | 2 years | 6 months | 2 years | Security |
| API access logs | 90 days | N/A | 90 days | Legitimate interest |
| Error logs | 90 days | N/A | 90 days | Legitimate interest |
| Application logs | 30 days | N/A | 30 days | Legitimate interest |
| System metrics | 90 days | 30 days | 90 days | Performance |

### 3.5 Analytics Data
| Data Type | Retention | Archive After | Delete After | Legal Basis |
|---|---|---|---|---|
| Aggregated reports | Indefinite | N/A | N/A | Business need |
| Raw analytics | 2 years | 1 year | 2 years | Legitimate interest |
| Clickstream data | 1 year | 6 months | 1 year | Legitimate interest |
| Search queries | 90 days | N/A | 90 days | Legitimate interest |
| A/B test results | 3 years | 1 year | 3 years | Business need |

---

## 4. Retention Enforcement

### 4.1 Automated Enforcement
```python
# Retention Enforcement Pipeline (Daily)
1. Run retention query per data type
2. Identify records exceeding retention period
3. Apply archival (move to cold storage)
4. Apply deletion (soft delete, then hard delete after grace)
5. Log actions for audit trail
6. Update retention dashboard
```

### 4.2 Enforcement Schedule
| Data Category | Frequency | Execution Window | Responsible |
|---|---|---|---|
| Session data | Daily | 03:00 AST | System |
| Application logs | Daily | 03:00 AST | System |
| API logs | Weekly (Sunday) | 03:00 AST | System |
| Inactive profiles | Monthly (1st) | 02:00 AST | System |
| Archived transactions | Quarterly | 01:00 AST | Data Team |
| Full audit cleanup | Annually (Jan 1) | 01:00 AST | Data Team |

### 4.3 Deletion Process
```
Retention Period Exceeded
        ↓
Soft Delete (flag as deleted, retain 30 days)
        ↓
Grace Period (user notification if applicable)
        ↓
Hard Delete (permanent removal)
        ↓
Audit Log Entry
```

---

## 5. Archival Strategy

### 5.1 Archive Tiers
| Tier | Storage | Cost/GB/Month | Access Time | Use Case |
|---|---|---|---|---|
| Hot | SSD (primary) | $0.10 | < 100ms | Active data |
| Warm | HDD (secondary) | $0.02 | < 1s | Recent archive |
| Cold | Object storage | $0.004 | < 5 min | Long-term archive |
| Deep Archive | Glacier-equivalent | $0.001 | < 24 hours | Compliance-only |

### 5.2 Archive Process
1. **Identify**: Records exceeding hot/warm threshold
2. **Compress**: Gzip compression for text data
3. **Encrypt**: AES-256 encryption for sensitive data
4. **Transfer**: Move to lower-cost storage tier
5. **Index**: Maintain metadata index for retrieval
6. **Verify**: Checksum validation post-transfer
7. **Update**: Mark records as archived in primary DB

### 5.3 Archive Retrieval
- **Hot → Warm**: Instant, no user impact
- **Warm → Cold**: < 5 minutes, admin request required
- **Cold → Deep Archive**: < 24 hours, compliance approval required
- **Retrieval Audit**: All archive retrievals logged

---

## 6. Deletion Procedures

### 6.1 User-Initiated Deletion (GDPR/CCPA)
```
User Request Received
        ↓
Verify Identity (email + phone OTP)
        ↓
Check Legal Holds (pending orders, disputes)
        ↓
Identify Data to Delete
        ↓
Execute Deletion (or anonymization)
        ↓
Confirm to User
        ↓
Log for Compliance
```

### 6.2 Systematic Deletion
| Trigger | Action | Notification | Approval |
|---|---|---|---|
| Retention expired | Auto-delete | None | System |
| Account deletion request | 30-day grace, then delete | User email | DPO |
| Legal hold removed | Resume deletion | Legal team | Legal |
| Data breach | Emergency purge | CISO + DPO | CISO |
| Regulatory order | Immediate compliance | Legal + DPO | Legal |

### 6.3 Deletion Scope
When a user requests deletion:
- **Deleted**: Profile, addresses, preferences, session data
- **Anonymized**: Order history, reviews, support tickets
- **Retained**: Financial records (legal obligation), audit logs
- **Retained**: Aggregated analytics (non-personal)

### 6.4 Deletion Confirmation
```json
{
  "request_id": "del_xyz789",
  "user_id": "usr_abc123",
  "requested_at": "2024-01-15T10:00:00Z",
  "completed_at": "2024-02-14T10:00:00Z",
  "deleted_records": {
    "profile": 1,
    "addresses": 3,
    "sessions": 45,
    "preferences": 12
  },
  "anonymized_records": {
    "orders": 28,
    "reviews": 5,
    "support_tickets": 8
  },
  "retained_records": {
    "invoices": 28,
    "audit_logs": 156
  }
}
```

---

## 7. Legal Holds

### 7.1 Hold Triggers
| Trigger | Source | Duration | Scope |
|---|---|---|---|
| Litigation | Legal team | Until resolved | Related records |
| Regulatory investigation | Legal + Compliance | Until resolved | All relevant data |
| Audit | Internal/External auditor | Audit period | Requested records |
| Law enforcement | Court order | Per order | Specified records |

### 7.2 Hold Process
1. Legal hold notice received and validated
2. Affected data identified and tagged
3. Retention enforcement paused for held data
4. Regular review of hold status
5. Hold release triggers normal retention resumption

---

## 8. Data Minimization

### 8.1 Principles
- Collect only data necessary for stated purpose
- Use data only for original purpose (or compatible)
- Limit access to minimum necessary personnel
- Regularly review and purge unnecessary data

### 8.2 Minimization Rules
| Data Type | Minimization Rule | Implementation |
|---|---|---|
| Full name | Store only if required for order | Optional field |
| Date of birth | Never required | Not collected |
| Government ID | Never collected | Not in forms |
| Location | Approximate (city-level) | IP geolocation only |
| Browsing history | Session only, 30-day max | Auto-delete |
| Device fingerprint | Hashed, 90-day max | Auto-delete |

---

## 9. Reporting & Compliance

### 9.1 Retention Dashboard Metrics
| Metric | Target | Review |
|---|---|---|
| Records deleted on schedule | > 99% | Monthly |
| Deletion request fulfillment | < 30 days | Monthly |
| Archive retrieval success | > 99.9% | Quarterly |
| Legal hold compliance | 100% | Monthly |
| Storage cost reduction | 10% YoY | Annually |

### 9.2 Audit Requirements
- Quarterly review of retention compliance
- Annual third-party audit of deletion processes
- Monthly reporting to DPO on GDPR requests
- Real-time monitoring of retention enforcement failures

---

## 10. Exceptions

### 10.1 Extended Retention
| Scenario | Extension | Approval |
|---|---|---|
| Pending litigation | Indefinite until resolved | Legal |
| Active investigation | Indefinite until resolved | CISO |
| Tax audit | 3 years extension | CFO |
| User dispute | 90 days extension | Support Manager |

### 10.2 Early Deletion
| Scenario | Action | Approval |
|---|---|---|
| Data breach (compromised) | Immediate purge | CISO |
| Regulatory order | Per order | Legal |
| User safety concern | Immediate purge | DPO |
| Storage emergency | Accelerate archival | CTO |

---

## 11. Communication

### 11.1 User Notifications
| Event | Channel | Timing |
|---|---|---|
| Account inactive warning | Email | At 2 years inactivity |
| Scheduled deletion notice | Email | 30 days before |
| Deletion confirmation | Email | On completion |
| Policy updates | Email + In-app | 30 days before effective |

### 11.2 Internal Communications
| Event | Audience | Channel |
|---|---|---|
| Retention job failures | Engineering | PagerDuty |
| Legal hold received | Legal + DPO | Email + Slack |
| Deletion request spike | Support + DPO | Slack |
| Policy review | Governance council | Meeting |
