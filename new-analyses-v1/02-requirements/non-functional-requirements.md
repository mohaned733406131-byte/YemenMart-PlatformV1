# Non-Functional Requirements — YemenMart v2

**Document Version:** 1.0
**Date:** 2026-09-13
**Status:** Draft

---

## 1. Performance Requirements

### 1.1 API Response Times

| Metric | Target | Measurement |
|--------|--------|-------------|
| API p50 latency | < 100ms | Server-side metrics (APM) |
| API p95 latency | < 200ms | Server-side metrics (APM) |
| API p99 latency | < 500ms | Server-side metrics (APM) |
| API max latency (non-streaming) | < 1s | Server-side metrics (APM) |

### 1.2 Page Load Performance

| Metric | Target | Measurement |
|--------|--------|-------------|
| First Contentful Paint (FCP) | < 1.5s | Lighthouse / CrUX |
| Largest Contentful Paint (LCP) | < 2.5s | Lighthouse / CrUX |
| Total page load (mobile 4G) | < 3s | Synthetic monitoring |
| Time to Interactive (TTI) | < 3.5s | Lighthouse / CrUX |
| Cumulative Layout Shift (CLS) | < 0.1 | Lighthouse / CrUX |
| First Input Delay (FID) | < 100ms | Real User Monitoring |

### 1.3 Search Performance

| Metric | Target | Measurement |
|--------|--------|-------------|
| Product search response | < 500ms | Elasticsearch metrics |
| Autocomplete suggestions | < 150ms | Elasticsearch metrics |
| Search result set (10k+ products) | < 800ms | Elasticsearch metrics |
| Faceted filter computation | < 200ms | Elasticsearch metrics |

### 1.4 Database Performance

| Metric | Target | Measurement |
|--------|--------|-------------|
| Simple query execution | < 10ms | pg_stat_statements |
| Complex joins (5+ tables) | < 50ms | pg_stat_statements |
| Transaction commit (single row) | < 5ms | pg_stat_statements |
| Connection pool utilization | < 70% | PgBouncer metrics |

### 1.5 Background Processing

| Metric | Target | Measurement |
|--------|--------|-------------|
| Order processing queue latency | < 2s | RabbitMQ metrics |
| Email delivery queue | < 30s | RabbitMQ metrics |
| Image processing (resize) | < 5s per image | Worker metrics |
| Payment webhook processing | < 3s end-to-end | Worker metrics |

---

## 2. Scalability Requirements

### 2.1 User Capacity

| Metric | Target | Notes |
|--------|--------|-------|
| Concurrent authenticated users | 10,000 | Peak hour estimate |
| Concurrent anonymous browsers | 50,000 | Product browsing |
| Peak concurrent orders | 500/min | Flash sale scenarios |
| Total registered users | 1,000,000 | 2-year projection |

### 2.2 Horizontal Scaling

| Component | Scaling Strategy | Trigger |
|-----------|-----------------|---------|
| API servers | Horizontal (Kubernetes HPA) | CPU > 70% or p95 > 200ms |
| Database | Read replicas + vertical | Connection pool > 80% |
| Search cluster | Horizontal (Elasticsearch) | Index size > 500GB |
| Cache layer | Horizontal (Redis Cluster) | Memory > 75% |
| Message queues | Horizontal (RabbitMQ cluster) | Queue depth > 10,000 |

### 2.3 Data Scaling

| Data Type | Initial | 2-Year Projection | Strategy |
|-----------|---------|-------------------|----------|
| Products | 50,000 | 500,000 | Elasticsearch sharding |
| Orders | 100,000/month | 1,000,000/month | Table partitioning by month |
| User sessions | 10,000/day | 100,000/day | Redis TTL + pagination |
| Product images | 200,000 | 2,000,000 | CDN + S3 storage |
| Audit logs | 500,000/month | 5,000,000/month | Archival to cold storage |

### 2.4 Geographic Scaling

- Primary region: `me-central-1` (Bahrain) for MENA proximity
- CDN edge nodes: Middle East, Africa, South Asia
- Database: Multi-AZ with cross-region read replica for DR
- Static assets: CloudFront with 99%+ cache hit ratio

---

## 3. Availability Requirements

### 3.1 Uptime Targets

| Metric | Target | Measurement |
|--------|--------|-------------|
| Monthly uptime percentage | 99.99% | Synthetic + Real User |
| Maximum monthly downtime | 4.38 minutes | Excluding planned maintenance |
| Planned maintenance window | 2 hrs/month (low traffic) | Scheduled via status page |
| Disaster recovery RPO | 5 minutes | Point-in-time recovery |
| Disaster recovery RTO | 1 hour | Full restoration time |

### 3.2 High Availability Architecture

| Component | HA Strategy | Failover Time |
|-----------|-------------|---------------|
| API servers | Multi-AZ, minimum 3 replicas | < 30 seconds |
| PostgreSQL | Primary + 2 read replicas, Patroni | < 30 seconds |
| Redis | Cluster mode, 3 shards, 3 replicas | < 10 seconds |
| Elasticsearch | 3-node cluster, 1 replica per index | < 60 seconds |
| RabbitMQ | 3-node mirrored queue cluster | < 15 seconds |
| Load balancer | ALB with cross-zone | Automatic |

### 3.3 Resilience Patterns

| Pattern | Implementation | Purpose |
|---------|---------------|---------|
| Circuit breaker | Resilience4j / Polly | Prevent cascade failures |
| Retry with backoff | Exponential + jitter | Transient fault recovery |
| Bulkhead isolation | Separate thread pools | Failure containment |
| Timeout | 30s API, 5s internal | Resource protection |
| Graceful degradation | Cached responses, queue fallback | Maintain partial service |
| Health checks | Liveness + readiness probes | Kubernetes orchestration |

### 3.4 Backup & Recovery

| Data | Backup Frequency | Retention | Recovery Test |
|------|-----------------|-----------|---------------|
| PostgreSQL full | Daily | 30 days | Weekly restore test |
| PostgreSQL WAL | Continuous | 7 days | Continuous verification |
| Redis snapshots | Every 6 hours | 7 days | Monthly restore test |
| Elasticsearch index | Daily | 14 days | Bi-weekly restore test |
| S3 product images | Versioned | 90 days | Monthly spot check |
| Configuration files | On change + daily | 90 days | Quarterly DR drill |

---

## 4. Security Requirements

### 4.1 Authentication & Authorization

| Requirement | Implementation |
|-------------|---------------|
| Multi-factor authentication | TOTP-based 2FA for admin/vendor |
| JWT token lifetime | 15-minute access, 7-day refresh |
| Password policy | Min 12 chars, complexity rules, breach DB check |
| Session management | Device fingerprinting, concurrent session limits |
| OAuth2 social login | Google, Apple (via Firebase Auth) |
| Role-based access control | 12 predefined roles, custom role support |
| API key management | Scoped keys with rotation, usage tracking |

### 4.2 Data Protection

| Requirement | Implementation |
|-------------|---------------|
| Encryption at rest | AES-256 for all databases and storage |
| Encryption in transit | TLS 1.3 enforced, HSTS headers |
| PII encryption | Column-level encryption for sensitive fields |
| Data masking | Non-production environments masked |
| Data retention | Configurable per data type (see compliance) |
| Right to deletion | 30-day GDPR deletion workflow |

### 4.3 Application Security

| Requirement | Implementation |
|-------------|---------------|
| OWASP Top 10 | Annual pen test, SAST/DAST pipeline |
| Input validation | Server-side + client-side, allowlisting |
| SQL injection prevention | Parameterized queries, ORM only |
| XSS prevention | CSP headers, output encoding, React DOMPurify |
| CSRF protection | SameSite cookies + CSRF tokens |
| Rate limiting | Per-user, per-endpoint, per-IP |
| API security | Request signing, replay protection |

### 4.4 Infrastructure Security

| Requirement | Implementation |
|-------------|---------------|
| Container scanning | Trivy in CI/CD pipeline |
| Dependency scanning | Dependabot / Snyk |
| Network isolation | VPC, security groups, NACLs |
| Secrets management | AWS Secrets Manager / HashiCorp Vault |
| WAF | AWS WAF with OWASP managed rules |
| DDoS protection | AWS Shield Advanced |
| Audit logging | CloudTrail + application audit log |

---

## 5. Usability Requirements

### 5.1 Accessibility

| Requirement | Standard | Target |
|-------------|----------|--------|
| WCAG compliance | WCAG 2.1 AA | Full compliance |
| Screen reader support | ARIA 1.2 | All interactive elements |
| Keyboard navigation | WCAG 2.1 | Full keyboard operability |
| Color contrast ratio | WCAG 2.1 | Minimum 4.5:1 text, 3:1 UI |
| Touch target size | WCAG 2.1 | Minimum 44x44px |
| Alternative text | WCAG 2.1 | All meaningful images |
| Form labels | WCAG 2.1 | Explicit labels for all inputs |
| Error identification | WCAG 2.1 | Multiple identification methods |

### 5.2 Responsive Design

| Breakpoint | Target | Behavior |
|------------|--------|----------|
| < 375px | Mobile S | Single column, stacked layout |
| 375–767px | Mobile M/L | Single column, bottom nav |
| 768–1023px | Tablet | Two column, side nav |
| 1024–1439px | Desktop | Three column, full nav |
| ≥ 1440px | Wide | Max-width container, full nav |

### 5.3 Mobile-First Design

- Progressive enhancement from mobile baseline
- Touch-optimized interactions (swipe, pull-to-refresh)
- Offline capability for product browsing (Service Worker)
- PWA features: install prompt, splash screen, badge support
- Camera integration for barcode scanning and product search
- Location services for delivery address auto-detect

### 5.4 RTL Support

| Feature | Implementation |
|---------|---------------|
| Text direction | CSS logical properties (`margin-inline-start`) |
| Layout mirroring | Full horizontal flip for Arabic |
| Typography | Arabic-first font stack (Noto Kufi Arabic, Inter) |
| Iconography | RTL-aware icon sets (mirrored where appropriate) |
| Date/time | Arabic locale with Hijri calendar option |
| Number formatting | Arabic-Indic numerals option |

### 5.5 Internationalization

| Feature | Implementation |
|---------|---------------|
| String externalization | JSON resource bundles per locale |
| Pluralization | ICU message format |
| Date formatting | Moment.js / date-fns with locale |
| Currency formatting | Locale-aware with currency symbols |
| Address formatting | Locale-specific address templates |
| Phone formatting | Yemeni + international formats |

---

## 6. Localization Requirements

### 6.1 Language Support

| Language | Priority | Scope | Direction |
|----------|----------|-------|-----------|
| Arabic (ar-YE) | Primary | 100% of UI | RTL |
| English (en) | Secondary | 100% of UI | LTR |

### 6.2 Currency Support

| Currency | Code | Usage | Precision |
|----------|------|-------|-----------|
| Yemeni Rial | YER | Primary pricing | 0 decimals |
| Saudi Riyal | SAR | Cross-border pricing | 2 decimals |
| US Dollar | USD | International pricing | 2 decimals |
| Cryptocurrency | BTC/ETH | Future integration | 8 decimals |

### 6.3 Number & Date Formatting

| Format | Arabic (ar-YE) | English (en) |
|--------|----------------|--------------|
| Thousands separator | ٫ (Arabic decimal) | , |
| Decimal point | . (period) | . |
| Date format | d MMMM yyyy | MMMM d, yyyy |
| Time format | h:mm a | h:mm A |
| Phone format | +967 XX XXX XXXX | +967 XX XXX XXXX |

### 6.4 Content Localization

- All product descriptions: Arabic + English
- Legal documents: Arabic (binding), English (reference)
- Marketing content: Bilingual with locale preference
- Push notifications: User's preferred language
- Email templates: Bilingual with language selector
- Error messages: Bilingual with fallback to English

---

## 7. Compliance Requirements

### 7.1 ZATCA E-Invoicing

| Requirement | Implementation | Deadline |
|-------------|---------------|----------|
| Phase 1 (Integration) | API integration with ZATCA platform | Immediate |
| Phase 2 (Reporting) | Real-time invoice reporting | 2026-Q4 |
| Invoice format | XML/JSON per ZATCA schema | Immediate |
| QR code generation | Base64-encoded invoice data | Immediate |
| Digital signature | ZATCA-approved certificate | Immediate |
| Archive retention | 10 years | Ongoing |

### 7.2 Tax Compliance

| Tax Type | Rate | Implementation |
|----------|------|---------------|
| VAT (standard) | 5% | Auto-calculate on checkout |
| VAT (zero-rated) | 0% | Export items, medical supplies |
| VAT (exempt) | N/A | Selected basic food items |
| Withholding tax | Variable | Vendor payout processing |

### 7.3 Data Protection

| Regulation | Scope | Implementation |
|-----------|-------|---------------|
| Yemen E-Transaction Law | User data, transactions | Consent management |
| PCI-DSS Level 1 | Payment card data | Stripe/card tokenization |
| GDPR (if applicable) | EU users | Data portability, deletion |
| Cloud data residency | All user data | `me-central-1` region |

### 7.4 Commercial Compliance

| Requirement | Implementation |
|-------------|---------------|
| Yemeni business license | Vendor verification workflow |
| Commercial registration | Vendor document upload + verification |
| Consumer protection | Return/refund policy enforcement |
| Anti-money laundering | Transaction monitoring, thresholds |
| Halal compliance | Product category flagging |

---

## 8. Observability Requirements

### 8.1 Monitoring

| Layer | Tool | Metrics |
|-------|------|---------|
| Infrastructure | CloudWatch / Prometheus | CPU, memory, disk, network |
| Application | Datadog / New Relic | Request rate, error rate, latency |
| Business | Custom dashboards | Orders, revenue, conversion |
| User experience | Real User Monitoring | Page load, errors, interactions |
| Security | GuardDuty / WAF logs | Threats, anomalies, violations |

### 8.2 Alerting

| Severity | Response Time | Escalation | Examples |
|----------|--------------|------------|----------|
| P1 Critical | < 5 min | Immediate page | Site down, data breach |
| P2 High | < 15 min | Page + Slack | Payment failure spike |
| P3 Medium | < 1 hour | Slack + email | Elevated error rate |
| P4 Low | < 4 hours | Email | Disk usage warning |

### 8.3 Logging

| Log Type | Retention | Storage | Format |
|----------|-----------|---------|--------|
| Application logs | 30 days | CloudWatch Logs | Structured JSON |
| Access logs | 90 days | S3 + Athena | Combined log format |
| Audit logs | 1 year | S3 Glacier | Application-specific |
| Security logs | 1 year | CloudTrail + S3 | AWS standard |

---

## 9. Disaster Recovery Requirements

### 9.1 Recovery Objectives

| Scenario | RPO | RTO | Strategy |
|----------|-----|-----|----------|
| Single AZ failure | 0 (synchronous) | < 30s | Multi-AZ failover |
| Region failure | 5 min | 1 hr | Cross-region replica + S3 CRR |
| Database corruption | 5 min | 2 hr | Point-in-time recovery |
| Ransomware/security breach | 5 min | 4 hr | Isolated backup restore |
| Complete infrastructure loss | 24 hr | 8 hr | Full DR site activation |

### 9.2 DR Testing

| Test Type | Frequency | Scope |
|-----------|-----------|-------|
| Backup restoration | Weekly | Database + config |
| Failover drill | Monthly | Full application stack |
| DR site activation | Quarterly | Complete site failover |
| Chaos engineering | Bi-weekly | Random component failures |

---

## 10. Development & Deployment Requirements

### 10.1 CI/CD Pipeline

| Stage | Tool | Target Duration |
|-------|------|----------------|
| Code compilation | Webpack / Turbopack | < 2 min |
| Unit tests | Jest / Vitest | < 3 min |
| Integration tests | Testcontainers | < 5 min |
| Security scan | Trivy + Snyk | < 3 min |
| Build & push | Docker + ECR | < 3 min |
| Deploy to staging | ArgoCD | < 5 min |
| E2E tests | Playwright / Cypress | < 10 min |
| Deploy to production | ArgoCD + canary | < 10 min |

### 10.2 Code Quality

| Metric | Target | Enforcement |
|--------|--------|-------------|
| Code coverage (unit) | > 80% | CI gate |
| Code coverage (integration) | > 60% | CI gate |
| Technical debt ratio | < 5% | SonarQube quality gate |
| Linting violations | 0 | ESLint + CI gate |
| Type errors | 0 | TypeScript strict mode |
| Bundle size | < 250KB gzip | CI gate |

### 10.3 Environment Management

| Environment | Purpose | Infrastructure |
|-------------|---------|---------------|
| Local development | Developer workstation | Docker Compose |
| Feature preview | PR validation | Ephemeral Kubernetes |
| Staging | Pre-production testing | Production mirror |
| Production | Live environment | Multi-AZ, HA |
