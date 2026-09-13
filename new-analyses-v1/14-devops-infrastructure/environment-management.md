# Environment Management - YemenMart

## Document Information
| Property | Value |
|----------|-------|
| Document ID | YEM-INF-ENV-001 |
| Version | 1.0 |
| Status | Active |
| Last Updated | 2026-09-12 |

---

## 1. Environment Overview

### 1.1 Environment Summary
| Environment | Purpose | Infrastructure | Data | Access |
|-------------|---------|---------------|------|--------|
| Development | Feature development | Shared cluster | Synthetic | All developers |
| QA | Quality assurance | Dedicated cluster | Synthetic | QA team |
| Staging | Pre-production | Production mirror | Anonymized | Limited |
| Production | Live system | Full HA | Real | Restricted |

### 1.2 Environment URLs
| Environment | URL | SSL |
|------------|-----|-----|
| Development | dev.yemenmart.com | Wildcard |
| QA | qa.yemenmart.com | Wildcard |
| Staging | staging.yemenmart.com | Wildcard |
| Production | yemenmart.com | Dedicated |

---

## 2. Development Environment

### 2.1 Purpose
| Aspect | Description |
|--------|-------------|
| Primary Use | Feature development and debugging |
| Users | All developers |
| Availability | On-demand |
| Data | Synthetic/local |
| Deploy | Manual/on-commit |

### 2.2 Infrastructure
| Component | Specification |
|-----------|--------------|
| Cluster | Shared EKS (dev) |
| Nodes | 3x m6i.large |
| Database | PostgreSQL (shared, per-developer schema) |
| Cache | Redis (shared, per-developer DB) |
| Search | Elasticsearch (shared) |
| Storage | S3 (dev bucket) |

### 2.3 Local Development Setup
```
docker-compose.yml:
- api (Node.js 20, port 8080)
- db (PostgreSQL 15, port 5432)
- redis (Redis 7, port 6379)
- elasticsearch (8.11, port 9200)
- mailhog (ports 1025, 8025)
```

### 2.4 Development Configuration
| Setting | Value |
|---------|-------|
| Log Level | DEBUG |
| Hot Reload | Enabled |
| Source Maps | Enabled |
| API Mocking | Enabled |
| Database Seeding | Auto |
| Test Coverage | Optional |

---

## 3. QA Environment

### 3.1 Purpose
| Aspect | Description |
|--------|-------------|
| Primary Use | Quality assurance testing |
| Users | QA team, developers |
| Availability | Business hours |
| Data | Synthetic test data |
| Deploy | Automated on PR merge |

### 3.2 Infrastructure
| Component | Specification |
|-----------|--------------|
| Cluster | Dedicated EKS (qa) |
| Nodes | 3x m6i.xlarge |
| Database | PostgreSQL (dedicated) |
| Cache | Redis (dedicated) |
| Search | Elasticsearch (dedicated) |
| Storage | S3 (qa bucket) |

### 3.3 QA Tools
| Tool | Purpose |
|------|---------|
| Cypress | E2E testing |
| Jest | Unit testing |
| k6 | Performance testing |
| OWASP ZAP | Security testing |
| Postman | API testing |
| TestRail | Test management |

### 3.4 QA Workflow
| Step | Action | Frequency |
|------|--------|-----------|
| 1 | Deploy from develop branch | Daily |
| 2 | Run automated test suite | Every deploy |
| 3 | Execute manual test cases | Weekly |
| 4 | Performance testing | Bi-weekly |
| 5 | Security scanning | Weekly |
| 6 | UAT testing | Per release |
| 7 | Sign-off | Per release |

### 3.5 QA Configuration
| Setting | Value |
|---------|-------|
| Log Level | INFO |
| API Mocking | Partial |
| External Services | Mocked |
| Email Sending | Disabled |
| Payment Processing | Test mode |
| SMS Sending | Disabled |

---

## 4. Staging Environment

### 4.1 Purpose
| Aspect | Description |
|--------|-------------|
| Primary Use | Pre-production validation |
| Users | Limited (release team) |
| Availability | Always on |
| Data | Anonymized production data |
| Deploy | Automated on main branch |

### 4.2 Infrastructure
| Component | Specification |
|-----------|--------------|
| Cluster | Dedicated EKS (staging) |
| Nodes | 6x m6i.2xlarge |
| Database | Aurora PostgreSQL (production mirror) |
| Cache | Redis Cluster (production mirror) |
| Search | Elasticsearch (production mirror) |
| Storage | S3 (staging bucket) |

### 4.3 Staging Configuration
| Setting | Value |
|---------|-------|
| Log Level | INFO |
| API Mocking | None (real services) |
| External Services | Test/sandbox mode |
| Email Sending | Staging mailbox |
| Payment Processing | Test mode |
| SMS Sending | Staging numbers |

### 4.4 Data Management
| Data Type | Source | Refresh |
|-----------|--------|---------|
| Products | Production | Weekly |
| Users | Anonymized production | Weekly |
| Orders | Anonymized production | Weekly |
| Configuration | Production | Daily |

---

## 5. Production Environment

### 5.1 Purpose
| Aspect | Description |
|--------|-------------|
| Primary Use | Live system serving customers |
| Users | All customers |
| Availability | 99.99% uptime |
| Data | Real production data |
| Deploy | Manual with approval |

### 5.2 Infrastructure
| Component | Specification |
|-----------|--------------|
| Cluster | EKS (prod) - Multi-AZ |
| Nodes | 6-50x m6i.2xlarge (auto-scaling) |
| Database | Aurora PostgreSQL (Multi-AZ) |
| Cache | Redis Cluster (Multi-AZ) |
| Search | Elasticsearch (3 nodes) |
| Storage | S3 (production bucket) |
| CDN | CloudFront |
| Load Balancer | ALB |

### 5.3 Production Configuration
| Setting | Value |
|---------|-------|
| Log Level | WARN |
| API Mocking | None |
| External Services | Production |
| Email Sending | Production SMTP |
| Payment Processing | Live mode |
| SMS Sending | Production gateway |
| Monitoring | Full |
| Alerting | Full |

### 5.4 Production Deployment
| Step | Action | Owner | Approval |
|------|--------|-------|----------|
| 1 | Create release branch | Developer | - |
| 2 | Bump version | Release Manager | - |
| 3 | Update CHANGELOG | Release Manager | - |
| 4 | Create PR | Developer | - |
| 5 | Code review | Team Lead | - |
| 6 | Merge to main | Release Manager | - |
| 7 | Deploy to staging | Automated | - |
| 8 | Staging validation | QA | - |
| 9 | Production approval | Engineering Manager | Required |
| 10 | Deploy to production | Release Manager | - |
| 11 | Smoke tests | Automated | - |
| 12 | Monitor | SRE | - |

### 5.5 Production Safeguards
| Safeguard | Implementation |
|-----------|---------------|
| Blue-Green Deployment | Zero-downtime deploy |
| Canary Release | 5% - 25% - 100% |
| Feature Flags | Gradual feature rollout |
| Rollback Ability | Instant rollback |
| Health Checks | Continuous monitoring |
| Rate Limiting | Protection against spikes |

---

## 6. Environment Isolation

### 6.1 Isolation Strategy
| Aspect | Isolation Method |
|--------|-----------------|
| Compute | Separate EKS clusters |
| Network | Separate VPCs |
| Database | Separate RDS instances |
| Cache | Separate ElastiCache |
| Storage | Separate S3 buckets |
| Secrets | Separate Secret Manager |
| DNS | Separate subdomains |

### 6.2 Network Isolation
| Environment | VPC CIDR | Subnets |
|------------|----------|---------|
| Development | 10.0.0.0/16 | 3 public, 3 private |
| QA | 10.1.0.0/16 | 3 public, 3 private |
| Staging | 10.2.0.0/16 | 3 public, 3 private |
| Production | 10.3.0.0/16 | 3 public, 3 private, 3 data |

### 6.3 DNS Isolation
| Environment | Domain | SSL |
|------------|--------|-----|
| Development | dev.yemenmart.com | Wildcard |
| QA | qa.yemenmart.com | Wildcard |
| Staging | staging.yemenmart.com | Wildcard |
| Production | yemenmart.com | Dedicated |

---

## 7. Configuration Management

### 7.1 Configuration Hierarchy
| Level | Source | Priority |
|-------|--------|----------|
| Base | values.yaml | Lowest |
| Environment | values-{env}.yaml | Medium |
| Secret | Secret Manager | Highest |
| Override | CLI flags | Highest |

### 7.2 Configuration Files
| File | Purpose |
|------|---------|
| values.yaml | Base Helm values |
| values-dev.yaml | Development overrides |
| values-qa.yaml | QA overrides |
| values-staging.yaml | Staging overrides |
| values-prod.yaml | Production overrides |

### 7.3 Environment Variables
| Variable | Dev | QA | Staging | Production |
|----------|-----|-----|---------|------------|
| NODE_ENV | development | test | staging | production |
| LOG_LEVEL | debug | info | info | warn |
| DATABASE_URL | local | qa-db | staging-db | prod-db |
| REDIS_URL | local | qa-redis | staging-redis | prod-redis |
| API_URL | localhost | api.qa | api.staging | api.yemenmart |

---

## 8. Secrets Management

### 8.1 Secret Storage
| Environment | Storage | Access |
|------------|---------|--------|
| Development | .env file | Local developer |
| QA | AWS Secrets Manager | CI/CD + QA team |
| Staging | AWS Secrets Manager | CI/CD + limited |
| Production | AWS Secrets Manager | CI/CD + SRE |

### 8.2 Secret Rotation
| Secret Type | Rotation Period | Method |
|------------|----------------|--------|
| Database Password | 90 days | Automatic |
| API Keys | 90 days | Manual |
| TLS Certificates | 1 year | Automatic |
| JWT Secret | 1 year | Manual |

### 8.3 Secret Access
| Role | Dev | QA | Staging | Production |
|------|-----|-----|---------|------------|
| Developer | Read | - | - | - |
| QA Engineer | - | Read | - | - |
| DevOps | Read | Read | Read | Read |
| SRE | Read | Read | Read | Read |

---

## 9. Database Management

### 9.1 Database Per Environment
| Environment | Instance | Size | Backup |
|------------|----------|------|--------|
| Development | PostgreSQL (shared) | db.t3.medium | Daily |
| QA | Aurora PostgreSQL | db.r6g.large | Daily |
| Staging | Aurora PostgreSQL | db.r6g.xlarge | Daily |
| Production | Aurora PostgreSQL | db.r6g.2xlarge | Continuous |

### 9.2 Data Management
| Operation | Dev | QA | Staging | Production |
|-----------|-----|-----|---------|------------|
| Schema Changes | Manual | Automated | Automated | Automated |
| Data Seeding | Auto | Auto | Weekly | - |
| Data Anonymization | - | - | Weekly | - |
| Backup Restore | On-demand | Weekly | Weekly | DR only |

### 9.3 Migration Strategy
| Step | Dev | QA | Staging | Production |
|------|-----|-----|---------|------------|
| 1 | Local test | Auto deploy | Auto deploy | Manual approval |
| 2 | Commit | Run tests | Run tests | Run tests |
| 3 | - | Validate | Validate | Validate |
| 4 | - | - | - | Deploy with rollback |

---

## 10. Monitoring Per Environment

### 10.1 Monitoring Configuration
| Feature | Dev | QA | Staging | Production |
|---------|-----|-----|---------|------------|
| Metrics | Basic | Basic | Full | Full |
| Logging | stdout | Fluentd | Fluentd | Fluentd + S3 |
| Tracing | None | Sampled | Sampled | Full |
| Alerting | None | Slack | Slack | Slack + PagerDuty |
| Uptime | None | None | Internal | External |

### 10.2 Dashboard Access
| Dashboard | Dev | QA | Staging | Production |
|-----------|-----|-----|---------|------------|
| Infrastructure | Read | Read | Read | Read + Alert |
| Application | Read | Read | Read | Read + Alert |
| Business | None | Read | Read | Read + Alert |
| Security | None | None | Read | Read + Alert |

---

## 11. Cost Management

### 11.1 Cost Allocation
| Environment | Monthly Cost | Percentage |
|------------|-------------|------------|
| Development | $500 | 3% |
| QA | $1,000 | 6% |
| Staging | $3,000 | 18% |
| Production | $12,500 | 73% |
| **Total** | **$17,000** | **100%** |

### 11.2 Cost Optimization per Environment
| Environment | Strategy | Expected Savings |
|------------|----------|-----------------|
| Development | Spot instances, auto-shutdown | 50% |
| QA | Scheduled scaling | 30% |
| Staging | Right-sizing | 20% |
| Production | Reserved instances, auto-scaling | 40% |

### 11.3 Cost Monitoring
| Metric | Target | Alert |
|--------|--------|-------|
| Monthly Spend | < $20,000 | > $25,000 |
| Cost per Environment | Within budget | > 10% over |
| Idle Resources | < 5% | > 10% |

---

## 12. Environment Promotion

### 12.1 Promotion Flow
```
Development -> QA -> Staging -> Production
     |           |          |           |
     v           v          v           v
  Unit Test  Integration  E2E Test  Smoke Test
              Test
```

### 12.2 Promotion Gates
| Gate | Criteria | Owner |
|------|----------|-------|
| Dev to QA | All unit tests pass | Automated |
| QA to Staging | QA sign-off | QA Lead |
| Staging to Production | Performance + Security pass | Engineering Manager |

### 12.3 Rollback Procedures
| Environment | Method | Time |
|------------|--------|------|
| Development | Git revert | 5 min |
| QA | Redeploy previous | 5 min |
| Staging | ArgoCD rollback | 2 min |
| Production | Blue-green switch | 1 min |

---

## 13. Compliance Per Environment

### 13.1 Compliance Requirements
| Requirement | Dev | QA | Staging | Production |
|------------|-----|-----|---------|------------|
| Data Encryption | Optional | Required | Required | Required |
| Access Logging | Optional | Required | Required | Required |
| Audit Trail | Optional | Optional | Required | Required |
| Backup | Optional | Daily | Daily | Continuous |
| DR Testing | - | - | Monthly | Quarterly |

### 13.2 Audit Requirements
| Audit Type | Dev | QA | Staging | Production |
|-----------|-----|-----|---------|------------|
| Access Audit | - | Monthly | Monthly | Weekly |
| Security Scan | - | Weekly | Weekly | Daily |
| Compliance Check | - | Quarterly | Quarterly | Monthly |
| Penetration Test | - | - | Quarterly | Quarterly |

---

## 14. Disaster Recovery Per Environment

### 14.1 DR Strategy
| Environment | RPO | RTO | Strategy |
|------------|-----|-----|----------|
| Development | 24 hours | 4 hours | Backup restore |
| QA | 24 hours | 4 hours | Backup restore |
| Staging | 1 hour | 1 hour | Backup restore |
| Production | 0 | 5 minutes | Multi-AZ + failover |

### 14.2 DR Testing
| Environment | Frequency | Scope |
|------------|-----------|-------|
| Development | - | - |
| QA | - | - |
| Staging | Monthly | Single service |
| Production | Quarterly | Full system |

---

## 15. Environment Maintenance

### 15.1 Maintenance Windows
| Environment | Window | Duration | Notice |
|------------|--------|----------|--------|
| Development | Anytime | - | - |
| QA | Weekends | 4 hours | 24 hours |
| Staging | Weekends | 2 hours | 48 hours |
| Production | Tue/Wed 2AM AST | 1 hour | 72 hours |

### 15.2 Cleanup Procedures
| Task | Frequency | Environment |
|------|-----------|-------------|
| Log rotation | Daily | All |
| Old snapshots | Weekly | Dev, QA |
| Stale data | Monthly | QA, Staging |
| Resource cleanup | Weekly | Dev, QA |

### 15.3 Environment Refresh
| Source | Target | Frequency | Method |
|--------|--------|-----------|--------|
| Production | Staging | Weekly | Data anonymization |
| Production | QA | Monthly | Data anonymization |
| Production | Dev | On-demand | Subset extraction |
