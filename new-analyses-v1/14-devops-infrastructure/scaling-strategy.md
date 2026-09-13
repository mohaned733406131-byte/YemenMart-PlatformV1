# Scaling Strategy - YemenMart

## Document Information
| Property | Value |
|----------|-------|
| Document ID | YEM-INF-SCL-001 |
| Version | 1.0 |
| Status | Active |
| Last Updated | 2026-09-12 |

---

## 1. Scaling Strategy Overview

### 1.1 Scaling Philosophy
| Principle | Description |
|-----------|-------------|
| Design for Scale | Build systems that scale horizontally from day one |
| Automate Everything | Auto-scaling for all components |
| Scale Independently | Each service scales based on its own metrics |
| Cache Aggressively | Reduce database load through caching |
| Async Processing | Offload non-critical work to queues |
| Measure and Monitor | Track scaling metrics continuously |

### 1.2 Scaling Dimensions
| Dimension | Approach |
|-----------|----------|
| Horizontal | Add more instances |
| Vertical | Increase instance size |
| Data | Shard and partition |
| Geographic | Multi-region deployment |
| Functional | Microservice decomposition |

---

## 2. Auto-Scaling Configuration

### 2.1 EKS Cluster Autoscaler
| Setting | Value |
|---------|-------|
| Min Nodes | 6 |
| Max Nodes | 50 |
| Scale-down Delay | 10 min |
| Scale-down Utilization | 0.5 |
| Skip Nodes with Local Storage | true |
| Balance Similar Node Groups | true |

### 2.2 Horizontal Pod Autoscaler (HPA)
| Service | Min | Max | CPU Target | Memory Target |
|---------|-----|-----|------------|---------------|
| API Gateway | 3 | 20 | 60% | 70% |
| Auth Service | 3 | 15 | 60% | 70% |
| Product Service | 3 | 20 | 60% | 70% |
| Order Service | 3 | 15 | 60% | 70% |
| Payment Service | 3 | 10 | 55% | 70% |
| Inventory Service | 2 | 10 | 60% | 70% |
| Search Service | 3 | 20 | 50% | 70% |
| Notification Service | 2 | 10 | 60% | 70% |
| Admin Service | 2 | 5 | 70% | 80% |
| Web Frontend | 3 | 10 | 60% | 70% |

### 2.3 Vertical Pod Autoscaler (VPA)
| Service | Current | Max | Recommendation Mode |
|---------|---------|-----|-------------------|
| Database | db.r6g.2xlarge | db.r6g.4xlarge | Off (manual) |
| Redis | cache.r6g.xlarge | cache.r6g.2xlarge | Off (manual) |
| Elasticsearch | r6i.xlarge.search | r6i.2xlarge.search | Off (manual) |

### 2.4 Scaling Behaviors
```yaml
# API Gateway HPA Behavior
behavior:
  scaleUp:
    stabilizationWindowSeconds: 60
    policies:
      - type: Percent
        value: 100
        periodSeconds: 60
      - type: Pods
        value: 4
        periodSeconds: 60
    selectPolicy: Max
  scaleDown:
    stabilizationWindowSeconds: 300
    policies:
      - type: Percent
        value: 10
        periodSeconds: 120
    selectPolicy: Min
```

---

## 3. Scaling Metrics

### 3.1 Primary Metrics
| Metric | Source | Scale Up | Scale Down |
|--------|--------|----------|------------|
| CPU Utilization | Prometheus | > 60% | < 30% |
| Memory Utilization | Prometheus | > 70% | < 40% |
| Request Rate | Prometheus | > 1000/pod | < 200/pod |
| Response Time (p95) | Prometheus | > 500ms | < 100ms |
| Queue Depth | Prometheus | > 500 | < 50 |
| Error Rate | Prometheus | > 1% | < 0.1% |

### 3.2 Custom Metrics
| Metric | Description | Target |
|--------|-------------|--------|
| orders_per_second | Order creation rate | 100/pod |
| search_queries_per_second | Search rate | 500/pod |
| active_websocket_connections | WebSocket load | 1000/pod |
| cart_operations_per_second | Cart operations | 200/pod |
| payment_processing_rate | Payment throughput | 50/pod |

### 3.3 Scaling Triggers
| Trigger | Condition | Action | Cooldown |
|---------|-----------|--------|----------|
| High CPU | > 60% for 60s | Scale up 50% | 60s |
| High Memory | > 70% for 60s | Scale up 50% | 60s |
| High Latency | p95 > 500ms for 5m | Scale up 100% | 120s |
| Queue Backlog | > 1000 for 5m | Scale up consumers | 60s |
| Low Traffic | < 20% for 30m | Scale down 20% | 300s |

---

## 4. Scheduled Scaling

### 4.1 Business Hours Scaling
| Time (AST) | Scale Factor | Services |
|------------|-------------|----------|
| 00:00-06:00 | 0.5x | All (except critical) |
| 06:00-09:00 | 0.75x | All |
| 09:00-18:00 | 1.0x | All |
| 18:00-22:00 | 1.25x | All (peak) |
| 22:00-00:00 | 0.75x | All |

### 4.2 Special Events Scaling
| Event | Pre-scale | Duration | Scale Factor |
|-------|-----------|----------|-------------|
| Ramadan | 1 week before | 30 days | 1.5x |
| Eid al-Fitr | 1 week before | 3 days | 2.0x |
| Eid al-Adha | 1 week before | 4 days | 2.0x |
| White Friday | 1 week before | 3 days | 3.0x |
| National Day | 1 week before | 1 day | 1.5x |
| Flash Sales | 1 hour before | 2-4 hours | 2.0x |

### 4.3 Cron-Based Scaling
```yaml
# Kubernetes CronJobs for scaling
apiVersion: batch/v1
kind: CronJob
metadata:
  name: scale-up-morning
spec:
  schedule: "0 6 * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
            - name: scale
              image: bitnami/kubectl
              command:
                - kubectl
                - scale
                - deployment/yemenmart-api
                - --replicas=10
                - -n=yemenmart-prod
          restartPolicy: OnFailure
```

---

## 5. Database Scaling

### 5.1 Read/Write Splitting
| Operation | Route | Instance |
|-----------|-------|----------|
| SELECT | Read replicas | Round-robin |
| INSERT | Primary | Direct |
| UPDATE | Primary | Direct |
| DELETE | Primary | Direct |
| Transactions | Primary | Direct |
| Reporting | Dedicated replica | Direct |

### 5.2 Connection Pooling
| Component | Min | Max | Overflow | Timeout |
|-----------|-----|-----|----------|---------|
| API Gateway | 10 | 50 | 20 | 30s |
| Auth Service | 5 | 30 | 10 | 30s |
| Product Service | 10 | 60 | 20 | 30s |
| Order Service | 10 | 50 | 15 | 30s |
| Payment Service | 5 | 30 | 10 | 30s |

### 5.3 Database Scaling Triggers
| Metric | Threshold | Action |
|--------|-----------|--------|
| CPU > 70% | 5 min | Add read replica |
| Connections > 80% | 5 min | Increase pool |
| Replication Lag > 30s | 5 min | Investigate |
| Query Time > 1s | 5 min | Optimize query |

---

## 6. Cache Scaling

### 6.1 Redis Cluster Scaling
| Metric | Threshold | Action |
|--------|-----------|--------|
| Memory > 75% | 5 min | Add shard |
| CPU > 70% | 5 min | Add replica |
| Connections > 80% | 5 min | Increase pool |
| Hit Rate < 80% | 1 hour | Review TTLs |

### 6.2 Cache Tier Scaling
| Tier | Current | Scale Trigger | Target |
|------|---------|---------------|--------|
| L1 (In-Memory) | 256MB | Hit rate < 80% | 512MB |
| L2 (Redis) | 16GB | Memory > 75% | 32GB |
| L3 (CDN) | 1TB | Origin requests > 1000/hr | 2TB |

---

## 7. Search Scaling

### 7.1 Elasticsearch Scaling
| Metric | Threshold | Action |
|--------|-----------|--------|
| CPU > 70% | 5 min | Add data node |
| Storage > 75% | 1 hour | Add storage |
| Query Time > 500ms | 5 min | Add node |
| Indexing Rate > 80% | 5 min | Add ingest node |

### 7.2 Index Strategy
| Index | Shards | Replicas | Rollover |
|-------|--------|----------|----------|
| products-current | 5 | 1 | Daily |
| orders-current | 8 | 1 | Daily |
| search-logs | 3 | 0 | Daily |

---

## 8. Message Queue Scaling

### 8.1 Queue Consumers
| Queue | Min | Max | Batch Size | Scale Trigger |
|-------|-----|-----|------------|---------------|
| order-events | 5 | 50 | 10 | Queue depth > 1000 |
| payment-events | 3 | 30 | 5 | Queue depth > 500 |
| inventory-updates | 5 | 40 | 20 | Queue depth > 2000 |
| notifications | 3 | 30 | 25 | Queue depth > 1000 |
| analytics-events | 10 | 100 | 100 | Queue depth > 5000 |

### 8.2 Queue Scaling Strategy
| Metric | Scale Up | Scale Down | Cooldown |
|--------|----------|------------|----------|
| Queue Depth | > 1000 for 5min | < 50 for 30min | 60s |
| Consumer Lag | > 5min | < 1min for 30min | 120s |
| Processing Time | > 10s per message | < 1s for 30min | 60s |

---

## 9. CDN Scaling

### 9.1 CloudFront Configuration
| Setting | Value |
|---------|-------|
| Edge Locations | 400+ |
| Origin Shield | Enabled |
| Cache Behaviors | Per content type |
| TTL (Static) | 24 hours |
| TTL (Dynamic) | 5 minutes |
| Compression | Brotli + Gzip |
| HTTP/2 | Enabled |
| HTTP/3 (QUIC) | Enabled |

### 9.2 CDN Scaling Metrics
| Metric | Threshold | Action |
|--------|-----------|--------|
| Origin Requests | > 1000/hr | Increase TTL |
| Cache Hit Rate | < 90% | Optimize caching |
| Bandwidth | > 1 Gbps | Enable more origins |

---

## 10. Geographic Scaling

### 10.1 Multi-Region Setup
| Region | Role | Capacity | Failover |
|--------|------|----------|----------|
| Middle East (Bahrain) | Primary | 100% | - |
| Europe (Frankfurt) | Secondary | 50% | Automatic |
| Asia Pacific (Mumbai) | Tertiary | 25% | Manual |

### 10.2 Geographic Load Balancing
| Strategy | Configuration |
|----------|--------------|
| Latency-based routing | Route to nearest region |
| Failover routing | Automatic failover |
| Weighted routing | Traffic distribution |
| Geolocation routing | Country-based routing |

---

## 11. Scaling Monitoring

### 11.1 Scaling Dashboard
| Panel | Metric | Refresh |
|-------|--------|---------|
| Current Replicas | All services | 30s |
| Scaling Events | Scale up/down | 1m |
| Resource Usage | CPU, Memory | 30s |
| Queue Depths | All queues | 30s |
| Database Connections | All instances | 30s |
| Cache Hit Rate | Redis | 30s |

### 11.2 Scaling Alerts
| Alert | Condition | Severity |
|-------|-----------|----------|
| Scaling Event | Any scale up | Info |
| Scaling Failed | Scale up failed | Critical |
| Max Replicas Reached | At max capacity | Warning |
| Scaling Thrash | > 5 events/hour | Warning |
| Resource Exhausted | CPU/Memory > 90% | Critical |

---

## 12. Cost Optimization

### 12.1 Scaling Cost Model
| Scenario | Instances | Cost/Hour | Cost/Day |
|----------|-----------|-----------|----------|
| Off-peak (00:00-06:00) | 15 | $15 | $90 |
| Normal (06:00-18:00) | 30 | $30 | $360 |
| Peak (18:00-22:00) | 45 | $45 | $180 |
| Flash Sale | 60 | $60 | $240 |
| Eid/Ramadan | 75 | $75 | $1,800 |

### 12.2 Cost Optimization Strategies
| Strategy | Expected Savings |
|----------|-----------------|
| Spot Instances | 60-70% |
| Reserved Instances | 30-40% |
| Scheduled Scaling | 20-30% |
| Right-sizing | 10-20% |
| Auto-scaling | 20-30% |

### 12.3 Cost Monitoring
| Metric | Target | Alert |
|--------|--------|-------|
| Cost per Request | < $0.0001 | > $0.0002 |
| Cost per Order | < $0.50 | > $1.00 |
| Monthly Spend | < $25,000 | > $30,000 |
| Spot Instance Ratio | > 40% | < 30% |

---

## 13. Scaling Testing

### 13.1 Load Testing
| Test Type | Frequency | Tool | Duration |
|-----------|-----------|------|----------|
| Baseline | Weekly | k6 | 30 min |
| Stress | Bi-weekly | k6 | 15 min |
| Spike | Monthly | k6 | 5 min |
| Endurance | Monthly | k6 | 4 hours |
| Scalability | Quarterly | k6 | 1 hour |

### 13.2 Scaling Test Scenarios
| Scenario | Users | Duration | Expected Behavior |
|----------|-------|----------|-------------------|
| Normal Load | 5,000 | 30 min | Stable performance |
| Peak Load | 15,000 | 30 min | Auto-scale to handle |
| Flash Sale | 50,000 | 5 min | Rapid scale up |
| Sustained Peak | 30,000 | 4 hours | Maintain capacity |
| Recovery | 50,000 → 5,000 | 30 min | Scale down gracefully |

### 13.3 Scaling Test Metrics
| Metric | Target | Measurement |
|--------|--------|-------------|
| Scale-up Time | < 2 min | Time to add instances |
| Scale-down Time | < 5 min | Time to remove instances |
| Stability | No thrashing | < 5 scale events/hour |
| Performance | < 500ms p95 | Response time during scaling |

---

## 14. Scaling Roadmap

### 14.1 Phase 1: Foundation (Month 1-2)
- Implement HPA for all services
- Configure cluster autoscaler
- Set up monitoring dashboards
- Establish scaling metrics

### 14.2 Phase 2: Optimization (Month 3-4)
- Implement scheduled scaling
- Add custom metrics scaling
- Optimize database connection pooling
- Implement cache scaling

### 14.3 Phase 3: Advanced (Month 5-6)
- Implement predictive scaling
- Add multi-region scaling
- Implement chaos testing for scaling
- Optimize cost optimization

### 14.4 Phase 4: Continuous (Ongoing)
- Weekly scaling reviews
- Monthly optimization
- Quarterly scaling tests
- Annual architecture review
