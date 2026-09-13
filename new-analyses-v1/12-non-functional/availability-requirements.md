# Availability Requirements - YemenMart

## Document Information
| Property | Value |
|----------|-------|
| Document ID | YEM-NFR-AVL-001 |
| Version | 1.0 |
| Status | Active |
| Last Updated | 2026-09-12 |

---

## 1. Availability Targets

### 1.1 Service-Level Availability
| Service | Availability Target | Downtime/Year | Downtime/Month |
|---------|--------------------|--------------:|---------------:|
| Core Platform | 99.99% | 52.6 min | 4.38 min |
| Payment Processing | 99.99% | 52.6 min | 4.38 min |
| Order Management | 99.99% | 52.6 min | 4.38 min |
| User Authentication | 99.99% | 52.6 min | 4.38 min |
| Product Catalog | 99.95% | 4.38 hrs | 21.9 min |
| Search Service | 99.95% | 4.38 hrs | 21.9 min |
| Notification Service | 99.9% | 8.77 hrs | 43.8 min |
| Admin Dashboard | 99.9% | 8.77 hrs | 43.8 min |
| Analytics Platform | 99.5% | 43.8 hrs | 3.65 hrs |
| Batch Processing | 99.0% | 87.7 hrs | 7.31 hrs |

### 1.2 Composite Availability
| Component | Weight | Availability | Contribution |
|-----------|--------|-------------|-------------|
| Core Platform | 30% | 99.99% | 29.997% |
| Payment Processing | 25% | 99.99% | 24.9975% |
| Order Management | 20% | 99.99% | 19.998% |
| Product Catalog | 15% | 99.95% | 14.9925% |
| Search Service | 10% | 99.95% | 9.995% |
| **Total** | **100%** | - | **99.98%** |

---

## 2. SLA Definitions

### 2.1 External SLA (Customer-Facing)
| SLA Metric | Target | Measurement Period | Exclusions |
|-----------|--------|-------------------|------------|
| Platform Uptime | 99.99% | Monthly | Scheduled maintenance |
| API Availability | 99.99% | Monthly | Scheduled maintenance |
| Order Processing | 99.99% | Monthly | Scheduled maintenance |
| Payment Processing | 99.99% | Monthly | Scheduled maintenance |
| Page Load Time | < 3s (p95) | Monthly | - |
| API Response Time | < 500ms (p95) | Monthly | - |

### 2.2 SLA Calculation
```
Availability = (Total Time - Downtime) / Total Time × 100%

Monthly Calculation:
- Total minutes: 43,200 (30 days)
- Allowed downtime for 99.99%: 4.32 minutes
- Allowed downtime for 99.95%: 21.6 minutes
- Allowed downtime for 99.9%: 43.2 minutes
```

### 2.3 SLA Exclusions
| Exclusion Type | Description | Notice Required |
|---------------|-------------|-----------------|
| Scheduled Maintenance | Planned updates | 72 hours |
| Force Majeure | Natural disasters | N/A |
| Customer Actions | Misconfiguration | N/A |
| Third-Party Failures | Vendor outages | Best effort |
| Security Incidents | Active response | 24 hours |

---

## 3. Monitoring and Alerting

### 3.1 Monitoring Stack
| Layer | Tool | Purpose |
|-------|------|---------|
| Infrastructure | Prometheus + Node Exporter | CPU, Memory, Disk, Network |
| Application | Prometheus + Custom Metrics | Request rates, latency, errors |
| Logs | ELK Stack | Centralized logging |
| Traces | Jaeger / Zipkin | Distributed tracing |
| Uptime | Pingdom / UptimeRobot | External availability |
| Real User | New Relic / Datadog | Real user monitoring |

### 3.2 Availability Metrics
| Metric | Description | Alert Threshold |
|--------|-------------|-----------------|
| Uptime Percentage | Successful requests / total requests | < 99.99% |
| Error Rate | Failed requests / total requests | > 0.1% |
| Request Success Rate | 2xx responses / total responses | < 99.9% |
| Health Check Status | Up/Down status | Any "Down" |
| SSL Certificate Expiry | Days until expiry | < 30 days |
| DNS Resolution | DNS lookup success rate | < 100% |

### 3.3 Alert Configuration
| Alert Type | Severity | Channel | Response Time |
|-----------|----------|---------|---------------|
| Platform Down | Critical | Phone + Slack + Email | Immediate |
| High Error Rate | High | Slack + Email | 5 minutes |
| Slow Response | Medium | Slack | 15 minutes |
| Certificate Expiry | Medium | Email + Slack | 24 hours |
| Disk Space Low | Low | Email | 24 hours |

---

## 4. Health Check System

### 4.1 Health Check Endpoints
| Endpoint | Method | Response | Timeout |
|----------|--------|----------|---------|
| /health | GET | { status: "UP" } | 5s |
| /health/ready | GET | Readiness status | 5s |
| /health/live | GET | Liveness status | 5s |
| /health/startup | GET | Startup status | 30s |

### 4.2 Health Check Components
| Component | Check Method | Threshold | Action on Failure |
|-----------|-------------|-----------|-------------------|
| Database | Query execution | < 1s | Mark unhealthy |
| Redis | PING command | < 100ms | Mark unhealthy |
| Elasticsearch | Cluster health | < 1s | Mark unhealthy |
| Message Queue | Queue depth check | < 10,000 | Mark unhealthy |
| Disk Space | Filesystem check | > 10% free | Warning |
| Memory | Usage check | < 90% | Warning |
| CPU | Usage check | < 80% | Warning |

### 4.3 Health Check Configuration
```yaml
healthCheck:
  interval: 10s
  timeout: 5s
  healthyThreshold: 2
  unhealthyThreshold: 3
  successCodes: [200]
  failureCodes: [500, 503]
```

---

## 5. Scheduled Maintenance

### 5.1 Maintenance Windows
| Maintenance Type | Schedule | Duration | Notice Period |
|-----------------|----------|----------|---------------|
| Security Patches | Weekly (Tue 2:00 AM AST) | 30 min | 48 hours |
| Minor Updates | Bi-weekly (Wed 2:00 AM AST) | 1 hour | 72 hours |
| Major Releases | Monthly (1st Sat 2:00 AM AST) | 2 hours | 1 week |
| Database Maintenance | Weekly (Mon 3:00 AM AST) | 1 hour | 48 hours |
| Infrastructure Updates | Quarterly | 4 hours | 2 weeks |

### 5.2 Maintenance Procedures
| Step | Action | Owner | Validation |
|------|--------|-------|------------|
| 1 | Notify stakeholders | Release Manager | Notification sent |
| 2 | Create backup | DBA | Backup verified |
| 3 | Enable maintenance mode | DevOps | Page shows notice |
| 4 | Perform maintenance | Engineering | Tasks completed |
| 5 | Run smoke tests | QA | All tests pass |
| 6 | Disable maintenance mode | DevOps | Site accessible |
| 7 | Monitor for issues | SRE | No alerts triggered |
| 8 | Communicate completion | Release Manager | Notification sent |

### 5.3 Maintenance Mode
| Feature | During Maintenance | After Maintenance |
|---------|-------------------|-------------------|
| Browse Products | Available | Available |
| Search | Available | Available |
| Add to Cart | Available | Available |
| Checkout | Disabled | Available |
| Order Status | Read-only | Available |
| Admin Panel | Disabled | Available |
| API Access | Read-only | Available |

---

## 6. Redundancy Architecture

### 6.1 Multi-AZ Deployment
| Component | AZ-1 | AZ-2 | AZ-3 | Failover |
|-----------|------|------|------|----------|
| Application Servers | 3 | 3 | 3 | Automatic |
| Database (Primary) | Primary | - | - | Automatic to AZ-2 |
| Database (Replicas) | 1 | 1 | 1 | Automatic |
| Redis (Primary) | Primary | - | - | Automatic to AZ-2 |
| Redis (Replicas) | 1 | 1 | 1 | Automatic |
| Load Balancer | Active | Active | Active | Automatic |

### 6.2 Multi-Region Deployment
| Region | Role | Capacity | Failover |
|--------|------|----------|----------|
| Middle East (Bahrain) | Primary | 100% | - |
| Europe (Frankfurt) | Secondary | 50% | Automatic |
| Asia Pacific (Mumbai) | Tertiary | 25% | Manual |

### 6.3 Data Redundancy
| Data Type | Replication | Consistency | RPO |
|-----------|-------------|-------------|-----|
| Database | Synchronous (Multi-AZ) | Strong | 0 |
| Database | Asynchronous (Cross-Region) | Eventual | 1s |
| File Storage | S3 Cross-Region | Eventual | 0 |
| Cache | Redis Cluster | Eventual | 0 |
| Logs | CloudWatch Logs | Eventual | 0 |

---

## 7. Incident Management

### 7.1 Incident Severity Levels
| Severity | Description | Response Time | Resolution Target |
|----------|-------------|---------------|-------------------|
| SEV-1 | Platform completely down | 5 minutes | 30 minutes |
| SEV-2 | Major feature unavailable | 15 minutes | 2 hours |
| SEV-3 | Minor feature degradation | 1 hour | 8 hours |
| SEV-4 | Cosmetic or low impact | 4 hours | 24 hours |

### 7.2 Incident Response Process
| Step | Action | Owner | Timeline |
|------|--------|-------|----------|
| 1 | Detect & Alert | Monitoring | Immediate |
| 2 | Triage & Classify | On-call Engineer | 5 min |
| 3 | Assemble War Room | Incident Commander | 10 min |
| 4 | Communicate Status | Communications | 15 min |
| 5 | Investigate & Mitigate | Engineering | 30 min |
| 6 | Resolve & Verify | Engineering | As needed |
| 7 | Post-mortem | All Teams | 24-48 hours |
| 8 | Implement Fixes | Engineering | 1 week |

### 7.3 Communication Templates
| Scenario | Channel | Message |
|----------|---------|---------|
| Incident Start | Slack + Email | "We're investigating issues with [service]" |
| Status Update | Slack + Email | "We've identified the cause and are working on a fix" |
| Resolution | Slack + Email | "The issue has been resolved. [Brief summary]" |
| Post-mortem | Wiki | Full incident report |

---

## 8. Service Level Objectives (SLOs)

### 8.1 SLO Definitions
| SLO | Target | Error Budget | Measurement |
|-----|--------|-------------|-------------|
| Availability | 99.99% | 4.32 min/month | Successful requests / total |
| Latency (p50) | < 200ms | - | Request duration |
| Latency (p95) | < 500ms | - | Request duration |
| Latency (p99) | < 1000ms | - | Request duration |
| Error Rate | < 0.1% | - | 5xx responses / total |
| Throughput | > 5000 RPS | - | Requests per second |

### 8.2 Error Budget Policy
| Error Budget Remaining | Action |
|----------------------|--------|
| > 50% | Normal development velocity |
| 25-50% | Increase testing, reduce deployment risk |
| 10-25% | Freeze non-critical changes, focus on reliability |
| < 10% | Feature freeze, dedicated reliability sprint |
| 0% | Complete feature freeze, all hands on reliability |

---

## 9. Disaster Recovery

### 9.1 DR Strategy
| Component | Strategy | RPO | RTO |
|-----------|----------|-----|-----|
| Application | Multi-region deployment | 0 | 5 min |
| Database | Cross-region replication | 1 min | 15 min |
| Cache | Multi-region cluster | 0 | 5 min |
| File Storage | S3 cross-region | 0 | 5 min |
| DNS | Multi-provider | 0 | 5 min |

### 9.2 DR Testing
| Test Type | Frequency | Scope | Duration |
|-----------|-----------|-------|----------|
| Backup Restore | Weekly | Database | 1 hour |
| Failover Test | Monthly | Single service | 30 min |
| DR Drill | Quarterly | Full system | 4 hours |
| Full DR Test | Annually | All systems | 8 hours |

---

## 10. Availability Reporting

### 10.1 Monthly Availability Report
| Metric | Target | Actual | Status |
|--------|--------|--------|--------|
| Platform Uptime | 99.99% | Monitor | Green/Red |
| API Availability | 99.99% | Monitor | Green/Red |
| Error Rate | < 0.1% | Monitor | Green/Red |
| P95 Latency | < 500ms | Monitor | Green/Red |
| Incidents | < 2 | Track | Green/Red |
| MTTR | < 15 min | Calculate | Green/Red |

### 10.2 SLA Compliance
| SLA Metric | Target | Current Month | YTD |
|-----------|--------|---------------|-----|
| Uptime | 99.99% | Track | Track |
| Response Time | < 500ms | Track | Track |
| Error Rate | < 0.1% | Track | Track |
| Incident Resolution | < 30 min | Track | Track |
