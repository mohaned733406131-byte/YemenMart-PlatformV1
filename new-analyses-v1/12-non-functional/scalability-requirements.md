# Scalability Requirements - YemenMart

## Document Information
| Property | Value |
|----------|-------|
| Document ID | YEM-NFR-SCAL-001 |
| Version | 1.0 |
| Status | Active |
| Last Updated | 2026-09-12 |

---

## 1. Scaling Strategy Overview

### 1.1 Scaling Philosophy
YemenMart employs a **hybrid scaling strategy** combining horizontal and vertical scaling with an emphasis on **horizontal scalability** for stateless services and **vertical scaling** for stateful components (databases).

### 1.2 Scaling Principles
| Principle | Description |
|-----------|-------------|
| Stateless Services | All application services designed to be stateless |
| Shared-Nothing | Minimize shared state between instances |
| Data Partitioning | Shard data across multiple database instances |
| Cache-First | Aggressive caching to reduce database load |
| Async Processing | Offload non-critical operations to message queues |
| Graceful Degradation | Maintain core functionality under extreme load |

---

## 2. Horizontal Scaling

### 2.1 Application Tier Scaling
| Component | Min Instances | Max Instances | Scale-Up Trigger | Scale-Down Trigger |
|-----------|--------------|--------------|------------------|-------------------|
| API Gateway | 3 | 20 | CPU > 60% | CPU < 30% |
| Auth Service | 3 | 15 | CPU > 65% | CPU < 25% |
| Product Service | 3 | 20 | CPU > 60% | CPU < 30% |
| Order Service | 3 | 15 | CPU > 60% | CPU < 30% |
| Payment Service | 3 | 10 | CPU > 55% | CPU < 25% |
| Inventory Service | 2 | 10 | CPU > 60% | CPU < 30% |
| Search Service | 3 | 20 | CPU > 50% | CPU < 25% |
| Notification Service | 2 | 10 | Queue depth > 1000 | Queue depth < 100 |
| Admin Service | 2 | 5 | CPU > 70% | CPU < 30% |

### 2.2 Scaling Policies
| Policy Type | Configuration |
|------------|---------------|
| Target Tracking | CPU utilization at 60% |
| Step Scaling | +2 instances at 60%, +4 at 75%, +6 at 90% |
| Predictive Scaling | Enabled for known traffic patterns |
| Scheduled Scaling | Ramadan/Eid pre-scaling |
| Cooldown Period | 300 seconds |

### 2.3 Load Balancing
| Configuration | Value |
|---------------|-------|
| Algorithm | Least connections with zone awareness |
| Health Check Interval | 10 seconds |
| Health Check Timeout | 5 seconds |
| Unhealthy Threshold | 3 consecutive failures |
| Sticky Sessions | Disabled (stateless design) |
| Connection Draining | 30 seconds |
| Cross-Zone Load Balancing | Enabled |

---

## 3. Vertical Scaling

### 3.1 Database Scaling
| Component | Current | Scale-Up Trigger | Target Upgrade |
|-----------|---------|-----------------|----------------|
| Primary DB (Write) | db.r6g.2xlarge | CPU > 70%, Connections > 80% | db.r6g.4xlarge |
| Read Replica 1 | db.r6g.xlarge | CPU > 65% | db.r6g.2xlarge |
| Read Replica 2 | db.r6g.xlarge | CPU > 65% | db.r6g.2xlarge |
| Redis Primary | cache.r6g.xlarge | Memory > 75% | cache.r6g.2xlarge |
| Redis Replica | cache.r6g.large | Memory > 70% | cache.r6g.xlarge |

### 3.2 Cache Scaling
| Tier | Current | Upgrade Trigger | Target |
|------|---------|-----------------|--------|
| L1 (In-Memory) | 256MB | Hit rate < 80% | 512MB |
| L2 (Redis) | 16GB | Memory > 75% | 32GB |
| L3 (CDN) | 1TB | Origin requests > 1000/hr | 2TB |

---

## 4. Database Scalability

### 4.1 Read/Write Splitting
| Operation | Routing |
|-----------|---------|
| SELECT queries | Read replicas (round-robin) |
| INSERT/UPDATE/DELETE | Primary instance |
| Transactions | Primary instance |
| Reporting queries | Dedicated read replica |
| Full-text search | Elasticsearch cluster |

### 4.2 Sharding Strategy
| Shard Key | Shard Count | Rebalancing |
|-----------|-------------|-------------|
| orders | By customer_id (16 shards) | Consistent hashing |
| products | By category_id (8 shards) | Manual rebalance |
| users | By region (4 shards) | Zone-based |
| inventory | By warehouse_id (6 shards) | Manual rebalance |

### 4.3 Connection Pooling
| Component | Min | Max | Overflow | Timeout |
|-----------|-----|-----|----------|---------|
| API Gateway | 10 | 50 | 20 | 30s |
| Auth Service | 5 | 30 | 10 | 30s |
| Product Service | 10 | 60 | 20 | 30s |
| Order Service | 10 | 50 | 15 | 30s |
| Payment Service | 5 | 30 | 10 | 30s |

---

## 5. Load Balancing Architecture

### 5.1 Multi-Tier Load Balancing
```
Internet
    |
[CloudFront CDN]
    |
[ALB - Regional]
    |
[Service Mesh / Istio]
    |
[Application Pods]
    |
[Database Connection Pool]
    |
[RDS Aurora Cluster]
```

### 5.2 Geographic Load Balancing
| Region | Priority | Weight | Failover |
|--------|----------|--------|----------|
| Middle East (Bahrain) | Primary | 70% | - |
| Europe (Frankfurt) | Secondary | 20% | Failover from ME |
| Asia Pacific (Mumbai) | Tertiary | 10% | Failover from EU |

### 5.3 Health Checks
| Component | Check Type | Interval | Threshold |
|-----------|-----------|----------|-----------|
| HTTP Service | HTTP 200 on /health | 10s | 3 failures |
| gRPC Service | gRPC health check | 10s | 3 failures |
| Database | TCP + query test | 30s | 3 failures |
| Cache | PING command | 10s | 3 failures |
| Queue | Queue depth check | 30s | 5 failures |

---

## 6. Growth Projections

### 6.1 User Growth
| Metric | Current | Year 1 | Year 2 | Year 3 |
|--------|---------|--------|--------|--------|
| Registered Users | 50,000 | 200,000 | 500,000 | 1,200,000 |
| Daily Active Users | 5,000 | 25,000 | 75,000 | 200,000 |
| Monthly Active Users | 20,000 | 100,000 | 300,000 | 750,000 |
| Peak Concurrent | 1,000 | 8,000 | 25,000 | 60,000 |

### 6.2 Transaction Growth
| Metric | Current | Year 1 | Year 2 | Year 3 |
|--------|---------|--------|--------|--------|
| Daily Orders | 500 | 3,000 | 12,000 | 35,000 |
| Monthly Revenue (SAR) | 500K | 3M | 12M | 35M |
| Average Order Value (SAR) | 150 | 180 | 200 | 220 |
| Items per Order | 3.2 | 3.5 | 3.8 | 4.0 |

### 6.3 Data Growth
| Data Type | Current | Year 1 | Year 2 | Year 3 |
|-----------|---------|--------|--------|--------|
| Product Catalog | 50K items | 200K items | 500K items | 1M items |
| Product Images | 250GB | 1TB | 3TB | 8TB |
| Database Size | 20GB | 100GB | 400GB | 1TB |
| Order History | 10GB | 50GB | 200GB | 500GB |
| User Data | 5GB | 25GB | 80GB | 200GB |
| Log Data | 100GB/month | 500GB/month | 2TB/month | 5TB/month |

### 6.4 Infrastructure Scaling
| Component | Current | Year 1 | Year 2 | Year 3 |
|-----------|---------|--------|--------|--------|
| App Servers | 6 | 20 | 50 | 100 |
| DB Instances | 3 | 8 | 15 | 30 |
| Redis Nodes | 2 | 6 | 12 | 24 |
| Elasticsearch Nodes | 3 | 6 | 12 | 20 |
| Storage (TB) | 1 | 5 | 15 | 40 |

---

## 7. Auto-Scaling Configuration

### 7.1 Scaling Metrics
| Metric | Scale Up Threshold | Scale Down Threshold | Cooldown |
|--------|-------------------|---------------------|----------|
| CPU Utilization | > 60% | < 30% | 300s |
| Memory Utilization | > 70% | < 40% | 300s |
| Request Count | > 1000/min | < 200/min | 180s |
| Queue Depth | > 500 | < 50 | 120s |
| Response Time (p95) | > 500ms | < 100ms | 300s |
| Error Rate | > 1% | < 0.1% | 300s |

### 7.2 Scaling Limits
| Resource | Minimum | Maximum | Burst Limit |
|----------|---------|---------|-------------|
| Total Instances | 20 | 150 | 200 (temporary) |
| Database Connections | 100 | 1000 | 1500 (temporary) |
| Redis Memory | 16GB | 128GB | - |
| S3 Requests | Unlimited | Unlimited | - |
| SQS Messages | Unlimited | Unlimited | - |

---

## 8. Content Delivery Network (CDN)

### 8.1 CDN Configuration
| Setting | Value |
|---------|-------|
| Provider | CloudFront |
| Edge Locations | 400+ globally |
| Origin Shield | Enabled |
| Cache Behaviors | Per content type |
| TTL (Static Assets) | 24 hours |
| TTL (API Responses) | 5 minutes |
| Compression | Brotli + Gzip |
| HTTP/2 | Enabled |
| HTTP/3 (QUIC) | Enabled |

### 8.2 Cache Invalidation
| Content Type | Invalidation Method | Frequency |
|-------------|---------------------|-----------|
| Static Assets | Versioned URLs | On deploy |
| Product Images | Prefix invalidation | On update |
| API Responses | TTL-based | Automatic |
| Configuration | Manual invalidation | On change |

---

## 9. Message Queue Scalability

### 9.1 Queue Configuration
| Queue | Current Throughput | Max Throughput | Retention |
|-------|-------------------|----------------|-----------|
| order-events | 100 msg/s | 10,000 msg/s | 7 days |
| payment-events | 50 msg/s | 5,000 msg/s | 7 days |
| inventory-updates | 200 msg/s | 20,000 msg/s | 3 days |
| notifications | 100 msg/s | 10,000 msg/s | 3 days |
| analytics-events | 500 msg/s | 50,000 msg/s | 1 day |
| dead-letter-queue | - | - | 30 days |

### 9.2 Consumer Scaling
| Queue | Min Consumers | Max Consumers | Batch Size |
|-------|--------------|--------------|------------|
| order-events | 5 | 50 | 10 |
| payment-events | 3 | 30 | 5 |
| inventory-updates | 5 | 40 | 20 |
| notifications | 3 | 30 | 25 |
| analytics-events | 10 | 100 | 100 |

---

## 10. Search Scalability

### 10.1 Elasticsearch Cluster
| Node Type | Count | Purpose | Replicas |
|-----------|-------|---------|----------|
| Master | 3 | Cluster management | - |
| Data Hot | 3 | Recent data (< 30 days) | 1 |
| Data Warm | 2 | Older data (30-90 days) | 1 |
| Ingest | 2 | Data ingestion | - |
| Coordinating | 2 | Query routing | - |

### 10.2 Index Strategy
| Index | Shards | Replicas | Rollover |
|-------|--------|----------|----------|
| products-current | 5 | 1 | Daily |
| products-archive | 3 | 1 | Monthly |
| orders-current | 8 | 1 | Daily |
| orders-archive | 5 | 1 | Monthly |
| search-logs | 3 | 0 | Daily |

---

## 11. Disaster Recovery Scaling

### 11.1 Multi-Region Setup
| Region | Role | Instances | RPO | RTO |
|--------|------|-----------|-----|-----|
| Middle East (Primary) | Active | Full capacity | 0 | 0 |
| Europe (Secondary) | Warm Standby | 50% capacity | 1 hour | 15 min |
| Asia Pacific (Tertiary) | Cold Standby | 20% capacity | 24 hours | 4 hours |

### 11.2 Failover Procedures
| Failure Type | Detection | Automated Response | Manual Action |
|-------------|-----------|-------------------|---------------|
| Single Instance | Health check | Auto-replace | Review |
| Availability Zone | Multi-AZ | Traffic reroute | Scale up |
| Region | Route53 failover | DNS failover | Verify |
| Database | Aurora failover | Auto-promote replica | Verify data |

---

## 12. Capacity Planning

### 12.1 Monthly Capacity Reviews
| Review Area | Metrics | Action Items |
|------------|---------|--------------|
| Compute | CPU, Memory, Instances | Right-size or scale |
| Storage | Disk usage, IOPS | Add storage or optimize |
| Network | Bandwidth, Latency | Optimize or upgrade |
| Database | Connections, Query time | Tune or scale |
| Cache | Hit rate, Memory | Expand or optimize |

### 12.2 Capacity Alerts
| Metric | Warning (80%) | Critical (90%) | Emergency (95%) |
|--------|--------------|----------------|-----------------|
| CPU | Notify team | Auto-scale + alert | Page on-call |
| Memory | Notify team | Auto-scale + alert | Page on-call |
| Storage | Notify team | Auto-expand | Page on-call |
| DB Connections | Notify team | Add replica | Page on-call |
| Cache Memory | Notify team | Add node | Page on-call |
