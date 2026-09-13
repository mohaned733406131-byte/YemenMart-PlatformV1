# Backup & Disaster Recovery - YemenMart

## Document Information
| Property | Value |
|----------|-------|
| Document ID | YEM-INF-BDR-001 |
| Version | 1.0 |
| Status | Active |
| Last Updated | 2026-09-12 |

---

## 1. DR Strategy Overview

### 1.1 Recovery Objectives
| System | RPO | RTO | Strategy |
|--------|-----|-----|----------|
| Core Platform | 0 | 5 min | Multi-AZ + Auto-failover |
| Payment Processing | 0 | 5 min | Multi-AZ + Auto-failover |
| Order Management | 0 | 5 min | Multi-AZ + Auto-failover |
| Product Catalog | 5 min | 15 min | Multi-AZ + Cache |
| Search Service | 5 min | 15 min | Index rebuild |
| User Service | 0 | 5 min | Multi-AZ + Auto-failover |
| Analytics | 1 hour | 1 hour | Backup restore |
| Admin Dashboard | 30 min | 30 min | Manual failover |
| Notification Service | 5 min | 15 min | Queue replay |

### 1.2 DR Architecture
```
┌─────────────────────────────────────────────────────────┐
│                    DR Architecture                       │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  PRIMARY REGION: Middle East (Bahrain)                  │
│  ┌─────────────────────────────────────────────┐        │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐    │        │
│  │  │  AZ-1   │  │  AZ-2   │  │  AZ-3   │    │        │
│  │  │ App Pods│  │ App Pods│  │ App Pods│    │        │
│  │  └─────────┘  └─────────┘  └─────────┘    │        │
│  │         │            │            │         │        │
│  │  ┌──────────────────────────────────────┐   │        │
│  │  │      Aurora PostgreSQL Cluster        │   │        │
│  │  │  Primary(AZ-1) → Replica(AZ-2,3)    │   │        │
│  │  └──────────────────────────────────────┘   │        │
│  └─────────────────────────────────────────────┘        │
│                         │                                │
│                    [Cross-Region Replication]            │
│                         │                                │
│  SECONDARY REGION: Europe (Frankfurt)                   │
│  ┌─────────────────────────────────────────────┐        │
│  │  ┌─────────┐  ┌─────────┐                  │        │
│  │  │  AZ-1   │  │  AZ-2   │                  │        │
│  │  │ App Pods│  │ App Pods│                  │        │
│  │  └─────────┘  └─────────┘                  │        │
│  │         │            │                     │        │
│  │  ┌──────────────────────────────────────┐   │        │
│  │  │      Aurora PostgreSQL Replica        │   │        │
│  │  └──────────────────────────────────────┘   │        │
│  └─────────────────────────────────────────────┘        │
│                                                         │
│  TERTIARY REGION: Asia Pacific (Mumbai)                 │
│  ┌─────────────────────────────────────────────┐        │
│  │  Cold Standby - Minimal Infrastructure      │        │
│  └─────────────────────────────────────────────┘        │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## 2. Backup Strategy

### 2.1 Database Backups
| Backup Type | Frequency | Retention | Storage | Recovery |
|------------|-----------|-----------|---------|----------|
| Automated Snapshot | Daily | 30 days | S3 | Point-in-time |
| Manual Snapshot | Weekly | 90 days | S3 + Glacier | Full restore |
| Transaction Log | Continuous | 7 days | S3 | Point-in-time |
| Cross-Region Snapshot | Daily | 30 days | S3 (Frankfurt) | DR restore |
| Logical Dump | Weekly | 30 days | S3 | Schema restore |

### 2.2 Backup Schedule
| Backup | Schedule (AST) | Duration | Owner |
|--------|---------------|----------|-------|
| DB Automated Snapshot | 02:00 daily | ~30 min | Automated |
| DB Manual Snapshot | Sunday 03:00 | ~1 hour | DBA |
| DB Cross-Region | 04:00 daily | ~1 hour | Automated |
| Redis Snapshot | Every 6 hours | ~10 min | Automated |
| S3 Versioning | Continuous | - | Automated |
| Elasticsearch Snapshot | Daily 05:00 | ~2 hours | Automated |
| Configuration Backup | On change | ~5 min | Automated |

### 2.3 Backup Verification
| Verification | Frequency | Method | Action on Failure |
|-------------|-----------|--------|-------------------|
| Snapshot Integrity | Daily | Checksum validation | Alert + rebackup |
| Restore Test | Weekly | Restore to dev | Alert + investigate |
| Cross-Region Sync | Daily | Compare snapshots | Alert + resync |
| Point-in-Time Recovery | Monthly | Full test | Alert + remediate |
| DR Failover Test | Quarterly | Full DR drill | Update runbook |

### 2.4 Backup Configuration
```yaml
# Aurora Backup Configuration
backup:
  automated_backups: enabled
  backup_retention_period: 35 days
  preferred_backup_window: "02:00-03:00"
  preferred_maintenance_window: "sun:03:00-sun:04:00"
  copy_tags_to_snapshot: true
  cross_region_backup: enabled
  cross_region_destination: eu-central-1

# S3 Backup Configuration
s3_versioning: enabled
s3_lifecycle:
  - transition_to_ia: 30 days
  - transition_to_glacier: 90 days
  - expiration: 365 days
```

---

## 3. Disaster Recovery Procedures

### 3.1 Failure Scenarios
| Scenario | Detection | Response | RTO |
|----------|-----------|----------|-----|
| Single Instance | Health check | Auto-replace | < 1 min |
| Availability Zone | Multi-AZ | Traffic reroute | < 5 min |
| Region Primary | Route53 failover | DNS failover | < 15 min |
| Database Failure | Aurora failover | Auto-promote | < 1 min |
| Cache Failure | Sentinel | Auto-promote | < 1 min |
| Storage Failure | S3 alerts | Cross-region | < 5 min |

### 3.2 DR Runbook: Region Failure
| Step | Action | Owner | Validation |
|------|--------|-------|------------|
| 1 | Detect failure | Monitoring | Alerts firing |
| 2 | Declare incident | Incident Commander | War room created |
| 3 | Assess impact | SRE Team | Impact documented |
| 4 | Initiate DR | SRE Team | DR procedure started |
| 5 | Update DNS | SRE Team | Route53 failover |
| 6 | Scale secondary | SRE Team | Capacity adequate |
| 7 | Verify services | QA Team | All services healthy |
| 8 | Notify stakeholders | Communications | Notifications sent |
| 9 | Monitor stability | SRE Team | No new alerts |
| 10 | Document incident | Incident Commander | Post-mortem drafted |

### 3.3 Failover Decision Tree
```
Failure Detected
    │
    ├── Is it a single instance?
    │   └── Yes → Auto-replace (no manual action)
    │
    ├── Is it an AZ failure?
    │   └── Yes → Multi-AZ routing (automatic)
    │
    ├── Is it a region failure?
    │   └── Yes → DR Failover Procedure
    │       │
    │       ├── Is database accessible?
    │       │   └── Yes → Promote read replica
    │       │   └── No → Restore from snapshot
    │       │
    │       ├── Is cache accessible?
    │       │   └── Yes → Use existing
    │       │   └── No → Rebuild from persistence
    │       │
    │       └── Is storage accessible?
    │           └── Yes → Use cross-region
    │           └── No → Restore from backup
    │
    └── Is it a data center issue?
        └── Yes → Contact AWS Support + DR
```

---

## 4. Data Protection

### 4.1 Encryption
| Data Type | Encryption Method | Key Management |
|-----------|------------------|----------------|
| Database (at rest) | AES-256 | AWS KMS |
| Database (in transit) | TLS 1.3 | AWS Certificate Manager |
| File Storage | AES-256 | AWS KMS |
| Cache (at rest) | AES-256 | AWS KMS |
| Cache (in transit) | TLS 1.3 | AWS KMS |
| Backups | AES-256 | AWS KMS |
| Logs | AES-256 | AWS KMS |

### 4.2 Key Management
| Key Type | Rotation | Storage | Access |
|----------|----------|---------|--------|
| Master Key | Annual | AWS KMS | HSM-backed |
| Data Key | Daily | AWS KMS | Auto-rotation |
| Backup Key | Annual | AWS KMS | Separate key |
| DR Key | Annual | AWS KMS (Frankfurt) | Separate key |

### 4.3 Data Integrity
| Check | Method | Frequency |
|-------|--------|-----------|
| Checksum Validation | SHA-256 | Per backup |
| Replication Lag | Monitoring | Real-time |
| Consistency Check | Row counts + checksums | Weekly |
| Backup Verification | Restore test | Weekly |

---

## 5. Communication Plan

### 5.1 Incident Communication
| Severity | Internal | External | Frequency |
|----------|----------|----------|-----------|
| SEV-1 | Slack + Phone | Status page | Every 15 min |
| SEV-2 | Slack + Email | Status page | Every 30 min |
| SEV-3 | Slack | Status page | Every hour |
| SEV-4 | Slack | - | Daily |

### 5.2 Stakeholder Notification
| Stakeholder | Channel | Timing |
|------------|---------|--------|
| Engineering Team | Slack + PagerDuty | Immediate |
| Management | Email + Phone | 15 min |
| Customer Support | Slack + Email | 30 min |
| Customers | Status page + Email | 1 hour |
| Partners | Email | 2 hours |
| Media | Press release | As needed |

### 5.3 Status Page
| Component | Status Page | Update Frequency |
|-----------|-------------|------------------|
| Platform Status | status.yemenmart.com | Real-time |
| API Status | status.yemenmart.com/api | Real-time |
| Payment Status | status.yemenmart.com/payments | Real-time |

---

## 6. DR Testing

### 6.1 Test Schedule
| Test Type | Frequency | Scope | Duration |
|-----------|-----------|-------|----------|
| Backup Restore | Weekly | Single database | 1 hour |
| Failover Test | Monthly | Single service | 30 min |
| AZ Failover | Quarterly | Full AZ | 2 hours |
| Region Failover | Semi-annually | Full system | 4 hours |
| Full DR Test | Annually | All systems | 8 hours |

### 6.2 DR Test Checklist
| Step | Action | Expected Result |
|------|--------|-----------------|
| 1 | Pre-test backup | Backup successful |
| 2 | Simulate failure | Target system unavailable |
| 3 | Initiate DR | DR procedure started |
| 4 | Verify failover | Services running in DR |
| 5 | Test functionality | All features working |
| 6 | Verify data | Data integrity maintained |
| 7 | Measure RTO | Within target |
| 8 | Measure RPO | Within target |
| 9 | Failback | Return to primary |
| 10 | Verify primary | Primary fully operational |

### 6.3 DR Metrics
| Metric | Target | Current | Status |
|--------|--------|---------|--------|
| RTO (Application) | < 5 min | Track | Green/Red |
| RTO (Database) | < 15 min | Track | Green/Red |
| RPO (Application) | 0 | Track | Green/Red |
| RPO (Database) | < 5 min | Track | Green/Red |
| DR Test Success | 100% | Track | Green/Red |

---

## 7. Backup Storage

### 7.1 Storage Architecture
| Storage | Purpose | Location | Redundancy |
|---------|---------|----------|------------|
| S3 Standard | Active backups | Bahrain | Multi-AZ |
| S3 IA | 30-90 day backups | Bahrain | Multi-AZ |
| S3 Glacier | 90-365 day backups | Bahrain | - |
| S3 Deep Archive | > 1 year backups | Bahrain | - |
| S3 Cross-Region | DR backups | Frankfurt | Multi-AZ |

### 7.2 Storage Costs
| Tier | Cost/GB/Month | Use Case |
|------|---------------|----------|
| S3 Standard | $0.023 | Recent backups |
| S3 IA | $0.0125 | 30-90 day backups |
| S3 Glacier | $0.004 | 90-365 day backups |
| S3 Deep Archive | $0.00099 | > 1 year backups |

### 7.3 Backup Size Estimates
| Data Type | Current Size | Year 1 | Year 2 | Year 3 |
|-----------|-------------|--------|--------|--------|
| Database Full | 20GB | 100GB | 400GB | 1TB |
| Database Incremental | 5GB/day | 20GB/day | 50GB/day | 100GB/day |
| File Storage | 250GB | 1TB | 3TB | 8TB |
| Logs | 100GB/month | 500GB/month | 2TB/month | 5TB/month |

---

## 8. AWS Services for DR

### 8.1 DR Services
| Service | Purpose | Configuration |
|---------|---------|---------------|
| Route 53 | DNS failover | Health checks + failover |
| Aurora | Database DR | Cross-region read replica |
| S3 | File storage DR | Cross-region replication |
| ElastiCache | Cache DR | Global Datastore |
| CloudFront | CDN | Multi-origin |
| ELB | Load balancing | Cross-region |

### 8.2 DR Infrastructure
| Component | Primary (Bahrain) | Secondary (Frankfurt) |
|-----------|-------------------|----------------------|
| EKS Cluster | 6 nodes | 3 nodes |
| Aurora | 3 instances | 1 replica |
| Redis | 3 shards + 6 replicas | 1 shard + 2 replicas |
| Elasticsearch | 3 nodes | 1 node |
| S3 | Active | Cross-region replica |

---

## 9. Monitoring and Alerting

### 9.1 DR Monitoring
| Metric | Alert Threshold | Action |
|--------|----------------|--------|
| Replication Lag | > 30 seconds | Page SRE |
| Backup Failure | Any failure | Page DBA |
| Cross-Region Sync | > 5 minutes | Alert SRE |
| DR Health Check | Failure | Page SRE |
| Storage Capacity | > 80% | Alert SRE |

### 9.2 DR Alerts
| Alert | Condition | Severity | Response |
|-------|-----------|----------|----------|
| Replication Lag High | > 60s | Critical | Investigate |
| Backup Failed | Backup status != success | Critical | Investigate |
| DR Region Unhealthy | Health check fails | Critical | DR procedure |
| Cross-Region Sync Delay | > 5 min | Warning | Monitor |
| DR Test Failed | Test != success | Critical | Investigate |

---

## 10. Documentation

### 10.1 DR Documentation
| Document | Owner | Update Frequency |
|----------|-------|------------------|
| DR Runbook | SRE Team | After each incident |
| Backup Procedures | DBA Team | Quarterly |
| Failover Procedures | SRE Team | Quarterly |
| Communication Plan | Communications | Semi-annually |
| Contact List | Management | Monthly |

### 10.2 DR Contacts
| Role | Primary | Secondary | Escalation |
|------|---------|-----------|------------|
| Incident Commander | SRE Lead | SRE Manager | CTO |
| Database DR | DBA Lead | Senior DBA | VP Engineering |
| Application DR | Tech Lead | Senior Engineer | VP Engineering |
| Communications | PR Manager | Marketing Lead | CEO |

---

## 11. Continuous Improvement

### 11.1 Post-Incident Review
| Activity | Timeline | Participants |
|----------|----------|--------------|
| Incident Review | 24 hours | Engineering |
| Post-mortem | 48 hours | All teams |
| Action Items | 1 week | Assigned owners |
| DR Runbook Update | 1 week | SRE Team |
| Lessons Learned | 2 weeks | All teams |

### 11.2 DR Maturity Model
| Level | Description | Current |
|-------|-------------|---------|
| 1 | Basic backup/restore | - |
| 2 | Pilot light | - |
| 3 | Warm standby | ✓ |
| 4 | Multi-site active-active | - |
| 5 | Multi-region active-active | - |

### 11.3 DR Improvement Roadmap
| Phase | Focus | Timeline |
|-------|-------|----------|
| Phase 1 | Automate failover | Q1 |
| Phase 2 | Add tertiary region | Q2 |
| Phase 3 | Implement chaos engineering | Q3 |
| Phase 4 | Achieve multi-site active-active | Q4 |
