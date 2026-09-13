# Compliance Requirements - YemenMart

## Document Information
| Property | Value |
|----------|-------|
| Document ID | YEM-NFR-CMP-001 |
| Version | 1.0 |
| Status | Active |
| Last Updated | 2026-09-12 |

---

## 1. Regulatory Compliance Overview

### 1.1 Applicable Regulations
| Regulation | Jurisdiction | Priority | Status |
|-----------|--------------|----------|--------|
| ZATCA E-Invoicing | Saudi Arabia | Critical | Active |
| ZATCA FATOORA | Saudi Arabia | Critical | Active |
| Saudi Data Protection Law (PDPL) | Saudi Arabia | Critical | Active |
| Anti-Money Laundering (AML) | Saudi Arabia | High | Active |
| Consumer Protection Law | Saudi Arabia | High | Active |
| Electronic Transactions Law | Saudi Arabia | Medium | Active |
| Payment Services Regulations | Saudi Arabia | High | Active |
| VAT Regulations | Saudi Arabia | Critical | Active |

### 1.2 Compliance Team
| Role | Responsibility | Contact |
|------|---------------|---------|
| Compliance Officer | Overall compliance | compliance@yemenmart.com |
| Data Protection Officer | Data privacy | dpo@yemenmart.com |
| Legal Counsel | Legal interpretation | legal@yemenmart.com |
| Security Officer | Security compliance | security@yemenmart.com |
| Finance Manager | Financial compliance | finance@yemenmart.com |

---

## 2. ZATCA E-Invoicing Compliance

### 2.1 ZATCA Phase 1 (Generation)
| Requirement | Implementation | Status |
|------------|----------------|--------|
| Generate e-invoice for all B2B | Invoice service | Implemented |
| Generate e-invoice for B2C (> 1000 SAR) | Invoice service | Implemented |
| Include mandatory fields | Invoice schema | Implemented |
| Store invoices for 6 years | Database + archive | Implemented |
| Arabic language support | Bilingual invoices | Implemented |

### 2.2 ZATCA Phase 2 (Integration)
| Requirement | Implementation | Status |
|------------|----------------|--------|
| Real-time invoice reporting | API integration | In Progress |
| Cryptographic stamping | UUID + QR code | Implemented |
| Invoice clearance | ZATCA API | In Progress |
| Reporting via API | REST API | In Progress |
| UUID generation | Per invoice | Implemented |

### 2.3 Invoice Required Fields
| Field | Description | Format |
|-------|-------------|--------|
| Invoice Number | Sequential number | YYYYMM-NNNNN |
| Invoice Date | Issue date | ISO 8601 |
| Invoice Time | Issue time | ISO 8601 |
| Seller Name | Business name | Text |
| Seller VAT Number | VAT registration | 15 digits |
| Buyer Name | Customer name | Text |
| Buyer VAT Number | If applicable | 15 digits |
| Invoice Line Items | Products/services | Array |
| VAT Rate | 15% standard | Percentage |
| VAT Amount | Calculated VAT | Currency |
| Total Amount | Including VAT | Currency |
| Currency | SAR | ISO 4217 |
| QR Code | Encoded invoice data | Base64 |

### 2.4 Invoice Data Schema
```json
{
  "invoiceNumber": "string",
  "invoiceDate": "YYYY-MM-DD",
  "invoiceTime": "HH:mm:ss",
  "sellerName": "string",
  "sellerVATNumber": "string(15)",
  "buyerName": "string",
  "buyerVATNumber": "string(15)",
  "items": [
    {
      "name": "string",
      "quantity": "number",
      "unitPrice": "number",
      "vatRate": "15",
      "vatAmount": "number",
      "totalAmount": "number"
    }
  ],
  "subtotal": "number",
  "vatTotal": "number",
  "totalAmount": "number",
  "currency": "SAR",
  "qrCode": "base64"
}
```

---

## 3. Data Protection (PDPL)

### 3.1 Data Classification
| Classification | Examples | Protection Level |
|---------------|---------|-----------------|
| Public | Product catalog, pricing | Basic |
| Internal | Business processes, configs | Standard |
| Confidential | Customer PII, orders | High |
| Restricted | Payment data, health data | Maximum |

### 3.2 Personal Data Processing
| Data Type | Purpose | Legal Basis | Retention |
|-----------|---------|-------------|-----------|
| Name, Email, Phone | Account management | Contract | Account lifetime |
| Address | Delivery | Contract | Order history |
| Payment Info | Transaction processing | Contract | 7 years |
| Order History | Service provision | Contract | 7 years |
| Browsing Data | Analytics | Consent | 1 year |
| Marketing Preferences | Marketing | Consent | Until withdrawn |

### 3.3 Data Subject Rights
| Right | Implementation | Response Time |
|-------|---------------|---------------|
| Right to Access | Self-service + API | 30 days |
| Right to Rectification | Self-service | Immediate |
| Right to Erasure | Account deletion | 30 days |
| Right to Data Portability | Data export | 30 days |
| Right to Object | Opt-out mechanisms | Immediate |
| Right to Restrict Processing | Processing controls | 7 days |

### 3.4 Data Protection Measures
| Measure | Implementation | Verification |
|---------|---------------|-------------|
| Encryption at Rest | AES-256 | Quarterly audit |
| Encryption in Transit | TLS 1.3 | Continuous |
| Access Control | RBAC + MFA | Monthly audit |
| Data Masking | PII masking in logs | Continuous |
| Anonymization | Analytics data | Continuous |
| Backup Encryption | Encrypted backups | Quarterly audit |

---

## 4. Payment Security (PCI DSS)

### 4.1 PCI DSS Requirements
| Requirement | Implementation | Status |
|------------|----------------|--------|
| Install firewall | AWS Security Groups | Implemented |
| Change default passwords | Automated | Implemented |
| Protect stored cardholder data | Tokenization | Implemented |
| Encrypt transmission | TLS 1.3 | Implemented |
| Use antivirus | ECR/Container scanning | Implemented |
| Develop secure systems | Secure SDLC | Implemented |
| Restrict access | RBAC | Implemented |
| Assign unique IDs | MFA | Implemented |
| Track access | Audit logging | Implemented |
| Test security | Penetration testing | Quarterly |
| Maintain policy | Security policies | Annual review |
| Maintain program | Vulnerability management | Continuous |

### 4.2 Payment Data Handling
| Operation | Method | Compliance |
|-----------|--------|------------|
| Card Storage | Tokenization (no raw cards) | PCI DSS |
| Card Transmission | TLS 1.3 encrypted | PCI DSS |
| Card Processing | 3D Secure / SADAD | PCI DSS |
| Card Verification | CVV not stored | PCI DSS |
| Refund Processing | Original token | PCI DSS |

### 4.3 Payment Gateway Integration
| Gateway | Type | PCI Level | Data Handling |
|---------|------|-----------|---------------|
| Tap Payments | Hosted | Level 1 | Token-based |
| SADAD | Direct | Level 1 | Bank redirect |
| Mada | Hosted | Level 1 | Token-based |
| Apple Pay | Hosted | Level 1 | Device token |
| STC Pay | Hosted | Level 1 | Token-based |

---

## 5. Anti-Money Laundering (AML)

### 5.1 KYC Requirements
| Customer Type | Required Documents | Verification |
|--------------|-------------------|-------------|
| Individual | National ID / Iqama | Government database |
| Business | Commercial Registration | Government database |
| High-Value Customer | Additional ID + proof of address | Manual review |

### 5.2 Transaction Monitoring
| Rule | Threshold | Action |
|------|-----------|--------|
| Single transaction > 50,000 SAR | Amount | Report to SAMA |
| Multiple transactions > 100,000 SAR/day | Cumulative | Flag for review |
| Suspicious pattern | Pattern analysis | Report to compliance |
| New account high value | First 30 days | Enhanced due diligence |
| Cross-border transaction | Any | Additional verification |

### 5.3 AML Reporting
| Report Type | Frequency | Recipient |
|------------|-----------|-----------|
| Suspicious Transaction Report (STR) | As needed | SAMA |
| Cash Transaction Report (CTR) | As needed | SAMA |
| Large Transaction Report | Monthly | Internal compliance |
| Customer Due Diligence | Ongoing | Internal compliance |

---

## 6. Audit Requirements

### 6.1 Internal Audits
| Audit Type | Frequency | Scope | Owner |
|-----------|-----------|-------|-------|
| Security Audit | Quarterly | All systems | Security Team |
| Compliance Audit | Semi-annually | Regulatory | Compliance |
| Financial Audit | Annually | Financial systems | Finance |
| Operational Audit | Annually | Operations | Operations |
| Code Quality Audit | Quarterly | Codebase | Engineering |

### 6.2 External Audits
| Audit Type | Frequency | Auditor | Scope |
|-----------|-----------|---------|-------|
| PCI DSS Audit | Annually | QSA | Payment systems |
| Financial Audit | Annually | External auditor | Financials |
| Security Assessment | Annually | Third-party | Security posture |
| ZATCA Compliance | As required | ZATCA | E-invoicing |

### 6.3 Audit Trail Requirements
| Event Type | Data Captured | Retention |
|-----------|--------------|-----------|
| User Login | User, IP, timestamp, success | 1 year |
| Data Access | User, resource, action, timestamp | 1 year |
| Data Modification | User, field, old/new value, timestamp | 7 years |
| Payment Transaction | Transaction ID, amount, status | 7 years |
| Admin Action | Admin, action, target, timestamp | 7 years |
| API Access | Client, endpoint, status, timestamp | 1 year |

### 6.4 Audit Log Format
```json
{
  "timestamp": "ISO 8601",
  "eventType": "string",
  "userId": "string",
  "ipAddress": "string",
  "userAgent": "string",
  "resource": "string",
  "action": "string",
  "result": "success|failure",
  "details": {
    "oldValue": "any",
    "newValue": "any"
  },
  "metadata": {
    "requestId": "string",
    "sessionId": "string"
  }
}
```

---

## 7. Security Compliance

### 7.1 Security Standards
| Standard | Scope | Status |
|----------|-------|--------|
| OWASP Top 10 | Web application | Implemented |
| OWASP ASVS | Application security | In Progress |
| NIST CSF | Security framework | In Progress |
| ISO 27001 | Information security | Planned |

### 7.2 Security Controls
| Control | Implementation | Verification |
|---------|---------------|-------------|
| MFA | All admin accounts | Continuous |
| Password Policy | 12+ chars, complexity | Continuous |
| Session Management | JWT, 15min expiry | Continuous |
| Rate Limiting | API endpoints | Continuous |
| Input Validation | All inputs | Continuous |
| Output Encoding | All outputs | Continuous |
| SQL Injection Prevention | Parameterized queries | Continuous |
| XSS Prevention | CSP + encoding | Continuous |
| CSRF Prevention | Tokens | Continuous |

### 7.3 Vulnerability Management
| Activity | Frequency | Tool | SLA |
|----------|-----------|------|-----|
| SAST | Every build | SonarQube | Fix in 30 days |
| DAST | Weekly | OWASP ZAP | Fix in 30 days |
| Dependency Scan | Daily | Snyk | Critical: 24h, High: 7d |
| Container Scan | Every build | Trivy | Fix before deploy |
| Penetration Test | Quarterly | Third-party | Fix in 30 days |

---

## 8. Financial Compliance

### 8.1 VAT Compliance
| Requirement | Implementation | Frequency |
|------------|----------------|-----------|
| VAT Calculation | 15% on eligible items | Per transaction |
| VAT Reporting | ZATCA filing | Monthly |
| VAT Payment | SAMA transfer | Monthly |
| VAT Records | 7-year retention | Continuous |
| VAT Invoicing | ZATCA format | Per invoice |

### 8.2 Financial Reporting
| Report | Frequency | Recipient | Deadline |
|--------|-----------|-----------|----------|
| VAT Return | Monthly | ZATCA | 15th of month |
| Financial Statements | Annually | ZATCA | 31 March |
| Transfer Pricing | Annually | ZATCA | 31 March |
| Withholding Tax | Monthly | ZATCA | 15th of month |

---

## 9. Data Retention

### 9.1 Retention Schedule
| Data Type | Retention Period | Disposal Method |
|-----------|-----------------|-----------------|
| Customer PII | Account lifetime + 7 years | Secure deletion |
| Transaction Data | 7 years | Archive then delete |
| Financial Records | 7 years | Archive then delete |
| Audit Logs | 7 years | Secure deletion |
| Marketing Data | Until consent withdrawn | Anonymize |
| Analytics Data | 2 years | Anonymize |
| System Logs | 1 year | Delete |
| Backups | 1 year | Secure deletion |

### 9.2 Data Disposal Procedures
| Method | Data Type | Verification |
|--------|-----------|-------------|
| Cryptographic Erasure | Encrypted data | Certificate |
| Secure Deletion | Database records | Audit log |
| Physical Destruction | Media | Certificate |
| Anonymization | Analytics data | Verification |

---

## 10. Compliance Monitoring

### 10.1 Compliance Dashboard
| Metric | Target | Current | Status |
|--------|--------|---------|--------|
| ZATCA Compliance | 100% | Track | Green/Red |
| PDPL Compliance | 100% | Track | Green/Red |
| PCI DSS Compliance | 100% | Track | Green/Red |
| AML Compliance | 100% | Track | Green/Red |
| Security Posture | A rating | Track | Green/Red |
| Audit Findings | 0 critical | Track | Green/Red |

### 10.2 Compliance Training
| Training | Audience | Frequency | Duration |
|----------|----------|-----------|----------|
| Security Awareness | All employees | Quarterly | 1 hour |
| Data Protection | All employees | Semi-annually | 2 hours |
| ZATCA Procedures | Finance team | As needed | 4 hours |
| AML/KYC | Customer service | Semi-annually | 2 hours |
| Incident Response | Engineering | Semi-annually | 4 hours |
