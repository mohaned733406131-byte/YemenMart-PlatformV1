# Infrastructure Overview - YemenMart

## Document Information
| Property | Value |
|----------|-------|
| Document ID | YEM-INF-OVR-001 |
| Version | 1.0 |
| Status | Active |
| Last Updated | 2026-09-12 |

---

## 1. Infrastructure Architecture

### 1.1 High-Level Architecture
```
                         ┌─────────────────┐
                         │   Route 53 DNS  │
                         └────────┬────────┘
                                  │
                         ┌────────▼────────┐
                         │  CloudFront CDN │
                         └────────┬────────┘
                                  │
                    ┌─────────────▼─────────────┐
                    │     Application Load      │
                    │        Balancer           │
                    └─────────────┬─────────────┘
                                  │
        ┌─────────────────────────┼─────────────────────────┐
        │                         │                         │
┌───────▼───────┐       ┌───────▼───────┐       ┌───────▼───────┐
│   AZ-1        │       │   AZ-2        │       │   AZ-3        │
│               │       │               │       │               │
│ ┌───────────┐ │       │ ┌───────────┐ │       │ ┌───────────┐ │
│ │ App Pod 1 │ │       │ │ App Pod 3 │ │       │ │ App Pod 5 │ │
│ └───────────┘ │       │ └───────────┘ │       │ └───────────┘ │
│ ┌───────────┐ │       │ ┌───────────┐ │       │ ┌───────────┐ │
│ │ App Pod 2 │ │       │ │ App Pod 4 │ │       │ │ App Pod 6 │ │
│ └───────────┘ │       │ └───────────┘ │       │ └───────────┘ │
└───────┬───────┘       └───────┬───────┘       └───────┬───────┘
        │                       │                       │
        └───────────────────────┼───────────────────────┘
                                │
                    ┌───────────▼───────────┐
                    │   Data Layer          │
                    │                       │
                    │ ┌───────────────────┐ │
                    │ │ Aurora PostgreSQL │ │
                    │ │ Primary + Replicas│ │
                    │ └───────────────────┘ │
                    │                       │
                    │ ┌───────────────────┐ │
                    │ │ Redis Cluster     │ │
                    │ └───────────────────┘ │
                    │                       │
                    │ ┌───────────────────┐ │
                    │ │ Elasticsearch     │ │
                    │ └───────────────────┘ │
                    └───────────────────────┘
```

### 1.2 Cloud Provider
| Aspect | Selection |
|--------|-----------|
| Primary Provider | AWS (Middle East - Bahrain) |
| Secondary Region | AWS Europe (Frankfurt) |
| Tertiary Region | AWS Asia Pacific (Mumbai) |
| CDN | CloudFront |
| DNS | Route 53 |

### 1.3 Region Selection Rationale
| Factor | Bahrain | Frankfurt | Mumbai |
|--------|---------|-----------|--------|
| Latency to Saudi | < 20ms | ~80ms | ~120ms |
| Data Sovereignty | Compliant | Compliant | Compliant |
| Service Availability | Full | Full | Full |
| Cost | Medium | Medium | Low |

---

## 2. Compute Resources

### 2.1 Kubernetes Cluster (EKS)
| Component | Specification |
|-----------|--------------|
| Cluster Name | yemenmart-prod |
| Kubernetes Version | 1.28 |
| Node Groups | System, Application, Data |
| Pod CIDR | 10.0.0.0/16 |
| Service CIDR | 10.100.0.0/16 |
| Cluster Autoscaler | Enabled |

### 2.2 Node Groups
| Node Group | Instance Type | Min | Max | Purpose |
|-----------|--------------|-----|-----|---------|
| System | m6i.xlarge | 3 | 5 | System pods, monitoring |
| Application | m6i.2xlarge | 6 | 50 | Application workloads |
| Data | r6i.2xlarge | 3 | 15 | Database, cache, search |

### 2.3 Application Instances
| Service | Replicas | CPU Request | Memory Request | CPU Limit | Memory Limit |
|---------|----------|-------------|----------------|-----------|--------------|
| API Gateway | 3-20 | 500m | 512Mi | 1000m | 1Gi |
| Auth Service | 3-15 | 250m | 256Mi | 500m | 512Mi |
| Product Service | 3-20 | 500m | 512Mi | 1000m | 1Gi |
| Order Service | 3-15 | 500m | 512Mi | 1000m | 1Gi |
| Payment Service | 3-10 | 250m | 256Mi | 500m | 512Mi |
| Inventory Service | 2-10 | 250m | 256Mi | 500m | 512Mi |
| Search Service | 3-20 | 500m | 512Mi | 1000m | 1Gi |
| Notification Service | 2-10 | 250m | 256Mi | 500m | 512Mi |
| Admin Service | 2-5 | 250m | 256Mi | 500m | 512Mi |
| Web Frontend | 3-10 | 250m | 256Mi | 500m | 512Mi |

---

## 3. Data Layer

### 3.1 Database (Aurora PostgreSQL)
| Component | Specification |
|-----------|--------------|
| Engine | PostgreSQL 15 |
| Instance Class (Primary) | db.r6g.2xlarge |
| Instance Class (Replicas) | db.r6g.xlarge |
| Storage | 100GB GP3, auto-scaling to 1TB |
| Multi-AZ | Yes (3 AZs) |
| Read Replicas | 2 |
| Backup Retention | 35 days |
| Encryption | AES-256 |

### 3.2 Cache (ElastiCache Redis)
| Component | Specification |
|-----------|--------------|
| Engine | Redis 7.0 |
| Node Type | cache.r6g.xlarge |
| Shards | 3 |
| Replicas per Shard | 2 |
| Total Memory | 48GB |
| Encryption | In-transit + at-rest |
| Multi-AZ | Yes |

### 3.3 Search (OpenSearch)
| Component | Specification |
|-----------|--------------|
| Engine | OpenSearch 2.x |
| Instance Type | r6i.xlarge.search |
| Data Nodes | 3 |
| Master Nodes | 3 |
| Storage | 500GB per node |
| Dedicated Master | Yes |
| UltraWarm | For older indices |

### 3.4 Object Storage (S3)
| Bucket | Purpose | Lifecycle |
|--------|---------|-----------|
| yemenmart-prod-assets | Product images, static assets | Standard |
| yemenmart-prod-backups | Database backups | Glacier after 30 days |
| yemenmart-prod-logs | Application logs | Glacier after 90 days |
| yemenmart-prod-reports | Generated reports | Glacier after 30 days |

---

## 4. Networking

### 4.1 VPC Configuration
| Component | CIDR | Purpose |
|-----------|------|---------|
| VPC | 10.0.0.0/16 | Main VPC |
| Public Subnet AZ-1 | 10.0.1.0/24 | ALB, NAT Gateway |
| Public Subnet AZ-2 | 10.0.2.0/24 | ALB, NAT Gateway |
| Public Subnet AZ-3 | 10.0.3.0/24 | ALB, NAT Gateway |
| Private Subnet AZ-1 | 10.0.10.0/24 | Application pods |
| Private Subnet AZ-2 | 10.0.20.0/24 | Application pods |
| Private Subnet AZ-3 | 10.0.30.0/24 | Application pods |
| Data Subnet AZ-1 | 10.0.100.0/24 | Database, cache |
| Data Subnet AZ-2 | 10.0.200.0/24 | Database, cache |
| Data Subnet AZ-3 | 10.0.300.0/24 | Database, cache |

### 4.2 Security Groups
| Group | Inbound | Outbound |
|-------|---------|----------|
| ALB SG | 80, 443 from 0.0.0.0/0 | To App SG |
| App SG | 8080 from ALB SG | To Data SG, 443 to internet |
| Data SG | 5432, 6379, 9200 from App SG | None |
| Bastion SG | 22 from office IPs | To App/Data SG |

### 4.3 Load Balancing
| Component | Type | Configuration |
|-----------|------|---------------|
| Application LB | ALB | External, HTTPS |
| Internal LB | NLB | Internal, TCP |
| gRPC LB | NLB | Internal, TCP |

---

## 5. Monitoring and Observability

### 5.1 Monitoring Stack
| Layer | Tool | Purpose |
|-------|------|---------|
| Metrics | Prometheus | Time-series metrics |
| Visualization | Grafana | Dashboards |
| Logging | Fluentd + Elasticsearch | Centralized logs |
| Tracing | Jaeger | Distributed tracing |
| Alerting | Alertmanager | Alert routing |
| Uptime | Pingdom | External monitoring |

### 5.2 Key Dashboards
| Dashboard | Metrics | Refresh |
|-----------|---------|---------|
| Infrastructure | CPU, Memory, Disk, Network | 30s |
| Application | Request rate, latency, errors | 15s |
| Database | Connections, queries, replication | 30s |
| Business | Orders, revenue, users | 60s |

---

## 6. Security Infrastructure

### 6.1 Security Services
| Service | Purpose | Configuration |
|---------|---------|---------------|
| AWS WAF | Web application firewall | ALB integration |
| AWS Shield | DDoS protection | Standard + Advanced |
| KMS | Key management | Customer-managed keys |
| Secrets Manager | Secret storage | Automatic rotation |
| GuardDuty | Threat detection | All accounts |
| Security Hub | Security posture | All accounts |
| CloudTrail | Audit logging | All regions |

### 6.2 Network Security
| Control | Implementation |
|---------|---------------|
| WAF Rules | OWASP Top 10, rate limiting |
| Network ACLs | Public subnet filtering |
| Security Groups | Instance-level filtering |
| VPC Flow Logs | Traffic logging |
| Private Link | AWS service access |
| VPN | Office connectivity |

---

## 7. CI/CD Infrastructure

### 7.1 CI/CD Tools
| Tool | Purpose |
|------|---------|
| GitHub Actions | CI/CD pipeline |
| ArgoCD | GitOps deployment |
| Helm | Package management |
| Terraform | Infrastructure as Code |
| Ansible | Configuration management |

### 7.2 Pipeline Infrastructure
| Component | Specification |
|-----------|--------------|
| Runner | Self-hosted (EKS) |
| Container Registry | ECR |
| Artifact Storage | S3 |
| Secret Management | AWS Secrets Manager |

---

## 8. Disaster Recovery

### 8.1 DR Infrastructure
| Component | Primary | Secondary | RPO | RTO |
|-----------|---------|-----------|-----|-----|
| Application | Bahrain | Frankfurt | 0 | 5 min |
| Database | Bahrain | Frankfurt | 1 min | 15 min |
| Cache | Bahrain | Frankfurt | 0 | 5 min |
| Files | Bahrain | Frankfurt | 0 | 0 |
| DNS | Global | Global | 0 | 5 min |

### 8.2 Backup Infrastructure
| Backup Type | Storage | Retention | Encryption |
|------------|---------|-----------|------------|
| Database Full | S3 + Glacier | 30 days + 1 year | AES-256 |
| Database Incremental | S3 | 7 days | AES-256 |
| Application Config | S3 + Git | Indefinite | AES-256 |
| Container Images | ECR | Indefinite | AES-256 |

---

## 9. Cost Optimization

### 9.1 Cost Allocation
| Category | Monthly Estimate | Optimization |
|----------|-----------------|-------------|
| Compute (EKS) | $8,000 | Spot instances, auto-scaling |
| Database (Aurora) | $5,000 | Reserved instances |
| Cache (Redis) | $2,000 | Reserved instances |
| Storage (S3) | $500 | Lifecycle policies |
| CDN (CloudFront) | $1,000 | Cache optimization |
| Networking | $1,500 | VPC endpoints |
| Monitoring | $1,000 | Retention policies |
| **Total** | **$19,000** | - |

### 9.2 Cost Optimization Strategies
| Strategy | Expected Savings |
|----------|-----------------|
| Reserved Instances (1yr) | 30-40% |
| Spot Instances | 60-70% |
| Auto-scaling | 20-30% |
| S3 Lifecycle | 40-50% |
| Right-sizing | 10-20% |

---

## 10. Environment Overview

### 10.1 Environment Architecture
| Environment | Purpose | Infrastructure | Data |
|-------------|---------|---------------|------|
| Development | Feature development | Shared cluster | Synthetic |
| QA | Quality assurance | Dedicated cluster | Synthetic |
| Staging | Pre-production | Production mirror | Anonymized |
| Production | Live system | Full HA | Real data |

### 10.2 Environment Isolation
| Aspect | Isolation Method |
|--------|-----------------|
| Cluster | Separate EKS clusters |
| Database | Separate RDS instances |
| Cache | Separate ElastiCache |
| Network | Separate VPCs |
| Secrets | Separate Secret Manager |
| DNS | Separate subdomains |
