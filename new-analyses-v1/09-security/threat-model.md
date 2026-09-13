# Threat Model — YemenMart v2

**Document Version:** 1.0
**Date:** 2026-09-13
**Methodology:** STRIDE
**Scope:** Full application stack — mobile apps, web frontend, API gateway, backend services, databases, third-party integrations

---

## 1. System Context

### 1.1 Trust Boundaries

| Boundary | Description |
|----------|-------------|
| TB1 | Client ↔ API Gateway (internet) |
| TB2 | API Gateway ↔ Internal Services (VPC) |
| TB3 | Application ↔ Database (private subnet) |
| TB4 | Application ↔ Third-party APIs (payment, SMS, email) |
| TB5 | Admin Panel ↔ Admin API (elevated privilege) |
| TB6 | Rider App ↔ Delivery Service (location data) |

### 1.2 Data Classification

| Classification | Examples | Protection |
|---------------|----------|------------|
| **Critical** | Passwords, 2FA secrets, payment cards, encryption keys | AES-256 encryption, HSM, never logged |
| **Confidential** | PII (name, email, phone, address), order history | Encryption at rest, access controls, audit |
| **Internal** | Vendor financials, commission rates, analytics | Internal access only |
| **Public** | Product listings, categories, reviews, vendor profiles | Integrity protection, rate limiting |

---

## 2. STRIDE Analysis

### 2.1 SPOOFING — Impersonation Attacks

| ID | Threat | Severity | Likelihood | Component | Impact |
|----|--------|----------|------------|-----------|--------|
| S-01 | Account takeover via credential stuffing | HIGH | HIGH | Auth service | Full account compromise |
| S-02 | JWT token theft and reuse | HIGH | MEDIUM | API layer | Impersonation of any user |
| S-03 | Session hijacking via XSS | HIGH | MEDIUM | Web client | Account impersonation |
| S-04 | Vendor impersonation (fake store) | MEDIUM | MEDIUM | Vendor platform | Customer fraud |
| S-05 | Admin account compromise | CRITICAL | LOW | Admin panel | Full system compromise |
| S-06 | Payment gateway webhook spoofing | HIGH | MEDIUM | Payment service | Fraudulent order completion |
| S-07 | Social engineering of customer support | MEDIUM | MEDIUM | Support operations | Account takeover |
| S-08 | Fake rider impersonation | MEDIUM | MEDIUM | Delivery service | Package theft |

**S-01 Mitigations:**

| Control | Implementation | Priority |
|---------|---------------|----------|
| Rate limiting | 5 failed attempts per email per 15 min | P0 |
| Account lockout | Temporary lock after 10 failures | P0 |
| CAPTCHA | After 3 failed attempts | P0 |
| Breach DB check | HaveIBeenPwned API on registration/reset | P0 |
| Login notifications | Email/push on new device login | P1 |
| Device fingerprinting | Track known devices per user | P1 |

**S-02 Mitigations:**

| Control | Implementation | Priority |
|---------|---------------|----------|
| Short-lived tokens | 15-minute access token TTL | P0 |
| Token binding | Bind tokens to device fingerprint | P0 |
| Refresh token rotation | Single-use refresh tokens | P0 |
| Secure storage | Keychain (iOS), EncryptedSharedPreferences (Android) | P0 |
| Token revocation | Redis-based token blacklist | P0 |
| HttpOnly cookies | Web client refresh tokens | P1 |

**S-05 Mitigations:**

| Control | Implementation | Priority |
|---------|---------------|----------|
| Mandatory 2FA | TOTP-based 2FA for all admin accounts | P0 |
| IP allowlisting | Restrict admin access to office IPs | P0 |
| Session timeout | 30-minute idle timeout | P0 |
| Separate admin domain | admin.yemenmart.com | P0 |
| Privilege audit | Weekly review of admin access logs | P1 |

**S-06 Mitigations:**

| Control | Implementation | Priority |
|---------|---------------|----------|
| Signature verification | HMAC-SHA256 on all webhooks | P0 |
| IP allowlisting | Payment gateway source IPs only | P0 |
| Idempotency | Prevent duplicate processing | P0 |
| Timestamp validation | Reject webhooks older than 5 minutes | P0 |

---

### 2.2 TAMPERING — Data Integrity Attacks

| ID | Threat | Severity | Likelihood | Component | Impact |
|----|--------|----------|------------|-----------|--------|
| T-01 | SQL injection in search/queries | CRITICAL | LOW | Database | Data breach, modification |
| T-02 | Price manipulation on checkout | HIGH | MEDIUM | Order service | Financial loss |
| T-03 | Coupon/discount abuse | MEDIUM | HIGH | Promo engine | Revenue loss |
| T-04 | Product listing tampering | MEDIUM | LOW | Product service | Customer fraud |
| T-05 | Order status manipulation | HIGH | LOW | Order service | Business logic bypass |
| T-06 | Payment amount tampering | CRITICAL | LOW | Payment service | Direct financial loss |
| T-07 | Wallet balance manipulation | CRITICAL | LOW | Wallet service | Financial fraud |
| T-08 | Review manipulation (fake reviews) | MEDIUM | MEDIUM | Review service | Trust erosion |
| T-09 | API request parameter tampering | HIGH | MEDIUM | API gateway | Unauthorized access |
| T-10 | Man-in-the-middle on mobile API | HIGH | LOW | Transport layer | Data interception |

**T-01 Mitigations:**

| Control | Implementation | Priority |
|---------|---------------|----------|
| Parameterized queries | ORM exclusively, no raw SQL | P0 |
| Input validation | Server-side allowlisting for all inputs | P0 |
| WAF rules | OWASP SQL injection rules on WAF | P0 |
| Database permissions | Application DB user has minimal privileges | P0 |
| SAST scanning | SQL injection detection in CI pipeline | P0 |

**T-02 Mitigations:**

| Control | Implementation | Priority |
|---------|---------------|----------|
| Server-side validation | Recalculate prices from product DB on checkout | P0 |
| Price snapshot | Store product price at time of order creation | P0 |
| Cart integrity | Hash cart contents, validate on submit | P0 |
| Audit logging | Log all price-related changes | P0 |

**T-06 Mitigations:**

| Control | Implementation | Priority |
|---------|---------------|----------|
| Server-side calculation | Gateway amount always derived server-side | P0 |
| Payment signature | Sign payment requests with HMAC | P0 |
| Webhook verification | Validate webhook signatures | P0 |
| Reconciliation | Daily automated payment reconciliation | P0 |

---

### 2.3 REPUDIATION — Denial of Actions

| ID | Threat | Severity | Likelihood | Component | Impact |
|----|--------|----------|------------|-----------|--------|
| R-01 | Customer denies placing order | MEDIUM | MEDIUM | Order service | Dispute resolution |
| R-02 | Vendor denies shipping order | MEDIUM | MEDIUM | Delivery service | Customer complaints |
| R-03 | Admin denies performing action | HIGH | LOW | Admin operations | Internal fraud |
| R-04 | Rider denies receiving package | MEDIUM | MEDIUM | Delivery service | Package loss disputes |
| R-05 | Customer denies receiving refund | LOW | LOW | Wallet service | Financial disputes |
| R-06 | Vendor disputes commission calculation | LOW | LOW | Finance service | Vendor relations |

**R-01 Mitigations:**

| Control | Implementation | Priority |
|---------|---------------|----------|
| Order confirmation | Email + SMS confirmation with order details | P0 |
| Digital receipts | PDF invoices with ZATCA compliance | P0 |
| Audit trail | Complete order status history with timestamps | P0 |
| Login audit | Log device, IP, and timestamp for order placement | P0 |
| Delivery confirmation | Photo + GPS proof of delivery | P0 |

**R-03 Mitigations:**

| Control | Implementation | Priority |
|---------|---------------|----------|
| Audit logging | Every admin action logged with user, timestamp, IP | P0 |
| Immutable logs | Write audit logs to append-only storage | P0 |
| Admin action alerts | Notify super_admin on critical changes | P0 |
| 4-eyes principle | Critical actions require two admin approvals | P1 |

---

### 2.4 INFORMATION DISCLOSURE — Data Exposure

| ID | Threat | Severity | Likelihood | Component | Impact |
|----|--------|----------|------------|-----------|--------|
| I-01 | API exposes excessive user data | HIGH | MEDIUM | API responses | PII leakage |
| I-02 | Error messages leak system internals | MEDIUM | MEDIUM | Error handling | Reconnaissance aid |
| I-03 | Logs contain sensitive data | HIGH | MEDIUM | Logging system | Data breach |
| I-04 | Database backup exposure | CRITICAL | LOW | Backup storage | Full data breach |
| I-05 | Third-party data sharing | HIGH | LOW | Integrations | Privacy violation |
| I-06 | Cache poisoning leaks data | MEDIUM | LOW | CDN/Cache | Cross-user data leakage |
| I-08 | Vendor data cross-exposure | HIGH | MEDIUM | Vendor portal | Competitive intelligence leak |
| I-09 | Payment card data exposure | CRITICAL | LOW | Payment flow | PCI-DSS violation |
| I-10 | Rider location data exposure | MEDIUM | MEDIUM | Delivery service | Privacy violation |

**I-01 Mitigations:**

| Control | Implementation | Priority |
|---------|---------------|----------|
| Response filtering | API returns only necessary fields per role | P0 |
| DTO/serialization layer | Explicit field whitelisting in responses | P0 |
| API review process | Security review for all new endpoints | P0 |
| Field-level encryption | Sensitive fields encrypted at rest | P0 |

**I-03 Mitigations:**

| Control | Implementation | Priority |
|---------|---------------|----------|
| Log scrubbing | Remove PII, tokens, passwords from logs | P0 |
| Log levels | PII only at DEBUG level (disabled in prod) | P0 |
| Centralized logging | Logs stored in encrypted, access-controlled storage | P0 |
| Log retention | Auto-delete after 30 days | P0 |

**I-09 Mitigations:**

| Control | Implementation | Priority |
|---------|---------------|----------|
| Tokenization | Card data never touches our servers | P0 |
| PCI-DSS compliance | Level 1 SAQ-A compliance via gateway | P0 |
| No card storage | Only last 4 digits and brand stored | P0 |
| Encrypted transmission | TLS 1.3 for all card data | P0 |

---

### 2.5 DENIAL OF SERVICE — Availability Attacks

| ID | Threat | Severity | Likelihood | Component | Impact |
|----|--------|----------|------------|-----------|--------|
| D-01 | Volumetric DDoS attack | HIGH | MEDIUM | CDN/Network | Site outage |
| D-02 | Application-layer DDoS (Slowloris) | HIGH | MEDIUM | API servers | Service degradation |
| D-03 | Database overload (expensive queries) | HIGH | HIGH | Database | Slow response, outage |
| D-04 | Memory exhaustion (large payloads) | MEDIUM | MEDIUM | API servers | Server crash |
| D-05 | Search index overload | MEDIUM | MEDIUM | Elasticsearch | Search downtime |
| D-06 | Message queue saturation | MEDIUM | MEDIUM | RabbitMQ | Background job delays |
| D-07 | Storage exhaustion (uploads) | MEDIUM | LOW | S3/Storage | Upload failures |
| D-08 | Payment gateway abuse | MEDIUM | MEDIUM | Payment service | Failed transactions |
| D-09 | Resource exhaustion via file uploads | MEDIUM | MEDIUM | Upload service | Storage/cost impact |
| D-10 | Cache stampede | MEDIUM | LOW | Redis/Cache | Database overload |

**D-01 Mitigations:**

| Control | Implementation | Priority |
|---------|---------------|----------|
| AWS Shield Advanced | DDoS protection at network edge | P0 |
| CloudFront | CDN absorbs volumetric attacks | P0 |
| Rate limiting | Per-IP, per-user, per-endpoint | P0 |
| Auto-scaling | Horizontal scaling on traffic spike | P0 |
| Geographic filtering | Block traffic from non-target regions | P1 |

**D-03 Mitigations:**

| Control | Implementation | Priority |
|---------|---------------|----------|
| Query timeout | 30-second max query timeout | P0 |
| Connection pooling | PgBouncer with 200 max connections | P0 |
| Read replicas | Distribute read queries across replicas | P0 |
| Query analysis | pg_stat_statements for slow query detection | P0 |
| Result pagination | Enforce max 100 records per page | P0 |
| Slow query alerts | Alert on queries > 500ms | P1 |

**D-06 Mitigations:**

| Control | Implementation | Priority |
|---------|---------------|----------|
| Queue depth monitoring | Alert on queue depth > 10,000 | P0 |
| Dead letter queues | Capture failed messages for investigation | P0 |
| Priority queues | Separate high/low priority job queues | P1 |
| Consumer auto-scaling | Scale consumers based on queue depth | P0 |

---

### 2.6 ELEVATION OF PRIVILEGE — Unauthorized Access

| ID | Threat | Severity | Likelihood | Component | Impact |
|----|--------|----------|------------|-----------|--------|
| E-01 | IDOR — access other users' orders | HIGH | MEDIUM | API endpoints | Data breach |
| E-02 | Role escalation (customer → admin) | CRITICAL | LOW | Auth system | Full system compromise |
| E-03 | Vendor access to other vendors' data | HIGH | MEDIUM | Vendor portal | Competitive data leak |
| E-04 | Customer accessing vendor admin APIs | HIGH | MEDIUM | API authorization | Business logic bypass |
| E-05 | Rider accessing order payment data | MEDIUM | MEDIUM | Delivery service | Financial data exposure |
| E-06 | SSRF via user-controlled input | HIGH | LOW | URL handling | Internal network access |
| E-07 | Path traversal in file downloads | HIGH | LOW | File service | Arbitrary file read |
| E-08 | JWT claim manipulation | CRITICAL | LOW | Auth system | Privilege escalation |
| E-09 | GraphQL introspection in production | MEDIUM | LOW | API layer | Schema disclosure |
| E-10 | Insecure direct object reference in uploads | MEDIUM | MEDIUM | Upload service | Unauthorized file access |

**E-01 Mitigations:**

| Control | Implementation | Priority |
|---------|---------------|----------|
| UUID identifiers | Non-guessable resource IDs (UUIDv4) | P0 |
| Authorization checks | Every endpoint validates resource ownership | P0 |
| Middleware enforcement | Authorization middleware on all protected routes | P0 |
| Automated IDOR testing | Security scan includes IDOR test cases | P1 |

**E-02 Mitigations:**

| Control | Implementation | Priority |
|---------|---------------|----------|
| Role-based access control | Server-side RBAC enforcement, never client-side | P0 |
| Admin registration approval | Manual approval for admin role assignment | P0 |
| Privilege audit | Weekly review of role assignments | P0 |
| Separation of duties | No single user can self-promote | P0 |
| JWT signing | RS256 asymmetric signing, keys in HSM | P0 |

**E-06 Mitigations:**

| Control | Implementation | Priority |
|---------|---------------|----------|
| URL validation | Allowlist of permitted external domains | P0 |
| Internal network isolation | No route from app tier to internal metadata | P0 |
| Input sanitization | Strip/encode user-controlled URLs | P0 |
| egress filtering | Block outbound traffic to internal IPs | P0 |

**E-07 Mitigations:**

| Control | Implementation | Priority |
|---------|---------------|----------|
| Path normalization | Resolve and validate file paths | P0 |
| Chroot/sandboxing | File service runs in isolated container | P0 |
| Filename allowlisting | Only permitted characters in filenames | P0 |
| Access control | Verify user has access to requested file | P0 |

---

## 3. Risk Matrix

| Severity \ Likelihood | LOW | MEDIUM | HIGH |
|----------------------|-----|--------|------|
| **CRITICAL** | T-01, T-06, T-07, E-02, E-08, I-04, I-09 | — | S-01 |
| **HIGH** | T-05, T-10, E-06, E-07 | S-02, S-03, S-06, T-02, T-09, D-01, D-02, E-01, E-03, E-04, I-01, I-05, I-08 | D-03 |
| **MEDIUM** | D-07, I-06, E-09 | S-04, S-07, S-08, T-03, T-04, T-08, R-01, R-02, R-04, I-02, I-10, D-04, D-05, D-08, D-09, E-05, E-10 | — |
| **LOW** | R-05, R-06 | — | — |

---

## 4. Security Controls Summary

### 4.1 Authentication & Authorization Controls

| Control | Status | Priority |
|---------|--------|----------|
| JWT with RS256 signing | Planned | P0 |
| 15-minute access token TTL | Planned | P0 |
| Single-use refresh token rotation | Planned | P0 |
| RBAC with 12 predefined roles | Planned | P0 |
| Mandatory 2FA for admin/vendor | Planned | P0 |
| Rate limiting (per-user, per-IP) | Planned | P0 |
| Account lockout after failures | Planned | P0 |
| Device fingerprinting | Planned | P1 |

### 4.2 Data Protection Controls

| Control | Status | Priority |
|---------|--------|----------|
| AES-256 encryption at rest | Planned | P0 |
| TLS 1.3 encryption in transit | Planned | P0 |
| Column-level encryption for PII | Planned | P0 |
| Data masking in non-production | Planned | P0 |
| Secure key management (HSM/Vault) | Planned | P0 |
| Automated data retention policies | Planned | P1 |

### 4.3 Application Security Controls

| Control | Status | Priority |
|---------|--------|----------|
| OWASP Top 10 compliance | Planned | P0 |
| SAST in CI/CD pipeline | Planned | P0 |
| DAST scanning (monthly) | Planned | P0 |
| Container image scanning (Trivy) | Planned | P0 |
| Dependency vulnerability scanning | Planned | P0 |
| CSP headers | Planned | P0 |
| CSRF protection | Planned | P0 |
| Input validation (server-side) | Planned | P0 |

### 4.4 Infrastructure Security Controls

| Control | Status | Priority |
|---------|--------|----------|
| WAF (AWS WAF + OWASP rules) | Planned | P0 |
| DDoS protection (AWS Shield) | Planned | P0 |
| VPC network isolation | Planned | P0 |
| Security group least privilege | Planned | P0 |
| Secrets management (Vault) | Planned | P0 |
| Audit logging (CloudTrail) | Planned | P0 |
| Incident response runbooks | Planned | P1 |

---

## 5. Threat Tracking

| Threat ID | Status | Owner | Due Date | Notes |
|-----------|--------|-------|----------|-------|
| S-01 | Open | Security Team | 2026-Q3 | Implement rate limiting + lockout |
| S-02 | Open | Security Team | 2026-Q3 | JWT token binding |
| S-05 | Open | Security Team | 2026-Q3 | Mandatory 2FA for admin |
| T-01 | Open | Dev Team | 2026-Q3 | ORM enforcement |
| T-02 | Open | Dev Team | 2026-Q3 | Server-side price validation |
| T-06 | Open | Dev Team | 2026-Q3 | Payment signature verification |
| I-09 | Open | Security Team | 2026-Q3 | PCI-DSS SAQ-A attestation |
| D-01 | Open | Infra Team | 2026-Q3 | AWS Shield Advanced |
| E-01 | Open | Dev Team | 2026-Q3 | IDOR testing automation |
| E-02 | Open | Security Team | 2026-Q3 | RBAC enforcement audit |

---

## 6. Review Schedule

| Activity | Frequency | Responsible |
|----------|-----------|-------------|
| Threat model review | Quarterly | Security Team |
| Penetration testing | Bi-annually | External vendor |
| OWASP Top 10 audit | Annually | Security Team |
| Code security review | Per sprint | Dev Team |
| Dependency audit | Weekly (automated) | CI/CD pipeline |
| Incident response drill | Quarterly | Security Team |
| Security training | Annually | All staff |
