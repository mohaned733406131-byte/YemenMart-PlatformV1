# Scalability Strategy — YemenMart

## 1. Scaling Philosophy

YemenMart starts with a modular monolith optimized for a small team. The scaling strategy follows three principles:

1. **Scale horizontally before vertically** — add instances before upgrading hardware
2. **Cache aggressively, query carefully** — push reads to cache/CDN before hitting the database
3. **Measure before optimizing** — use metrics to identify bottlenecks, not assumptions

---

## 2. Horizontal Scaling

### 2.1 Application Servers

```
                    ┌──────────────┐
                    │ Load Balancer│
                    │   (Nginx/    │
                    │    ALB)      │
                    └──────┬───────┘
                           │
              ┌────────────┼────────────┐
              │            │            │
         ┌────▼────┐  ┌────▼────┐  ┌────▼────┐
         │ App  1  │  │ App  2  │  │ App  N  │
         │(Node.js)│  │(Node.js)│  │(Node.js)│
         └────┬────┘  └────┬────┘  └────┬────┘
              │            │            │
              └────────────┼────────────┘
                           │
                    ┌──────▼───────┐
                    │   Shared     │
                    │   Redis      │
                    │   Cluster    │
                    └──────────────┘
```

**Configuration:**

```yaml
# docker-compose.prod.yml
services:
  app:
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: '2'
          memory: 2G
        reservations:
          cpus: '1'
          memory: 1G
      restart_policy:
        condition: on-failure
        max_attempts: 3
```

**Scaling Triggers:**

| Metric | Threshold | Action |
|---|---|---|
| CPU utilization | > 70% sustained | Add instance |
| Memory usage | > 80% | Add instance |
| Request latency p95 | > 500ms | Add instance |
| Queue depth | > 1000 pending | Add worker |

### 2.2 Queue Workers

```
┌──────────────────────────────────────────────────┐
│              BULLMQ WORKER POOL                    │
│                                                   │
│  High Priority Workers (dedicated):               │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐            │
│  │Payment  │ │Order    │ │Delivery │            │
│  │Worker x2│ │Worker x3│ │Worker x2│            │
│  └─────────┘ └─────────┘ └─────────┘            │
│                                                   │
│  Normal Priority Workers (pooled):                │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐            │
│  │Notif.   │ │Inventory│ │Search   │            │
│  │Worker x4│ │Worker x3│ │Worker x2│            │
│  └─────────┘ └─────────┘ └─────────┘            │
│                                                   │
│  Background Workers (low priority):               │
│  ┌─────────┐ ┌─────────┐                         │
│  │Analytics│ │Cleanup  │                         │
│  │Worker x2│ │Worker x1│                         │
│  └─────────┘ └─────────┘                         │
└──────────────────────────────────────────────────┘
```

---

## 3. Vertical Scaling

### 3.1 PostgreSQL Optimization

```sql
-- postgresql.conf (production tuning)
shared_buffers = 8GB           # 25% of RAM
effective_cache_size = 24GB    # 75% of RAM
work_mem = 256MB               # Per-operation sort memory
maintenance_work_mem = 2GB     # VACUUM, CREATE INDEX
max_connections = 100          # Via PgBouncer
wal_buffers = 64MB
checkpoint_completion_target = 0.9
random_page_cost = 1.1        # SSD-optimized
effective_io_concurrency = 200 # SSD-optimized
```

### 3.2 Connection Pooling (PgBouncer)

```
Application Servers
        │
        ▼
┌─────────────────┐
│    PgBouncer    │
│  (Transaction   │
│   Pooling)      │
│                 │
│  max_client_conn = 1000
│  default_pool_size = 25
│  reserve_pool_size = 5
└────────┬────────┘
         │
    ┌────▼────┐
    │PostgreSQL│
    │ Primary  │
    └─────────┘
```

### 3.3 Redis Memory Optimization

```yaml
# redis.conf
maxmemory: 8gb
maxmemory-policy: allkeys-lru
save: "900 1"
save: "300 10"
save: "60 10000"
tcp-keepalive: 300
timeout: 300
```

---

## 4. Database Scaling

### 4.1 Read Replicas

```
┌──────────────────┐
│  Application     │
│  Servers         │
└──────┬───────────┘
       │
       ├──────────────────────┐
       │                      │
  ┌────▼─────┐          ┌────▼─────┐
  │ Primary  │─────────▶│ Replica 1│ (Reporting)
  │(Read/Write│  WAL     │(Read Only)│
  └──────────┘ Stream   └──────────┘
       │
       └──────────────────────┐
                              │
                         ┌────▼─────┐
                         │ Replica 2│ (Search sync)
                         │(Read Only)│
                         └──────────┘
```

**Routing Rules:**

| Operation | Target | Rationale |
|---|---|---|
| CRUD operations | Primary | Write consistency |
| Product browsing | Replica 1 | Read-heavy, eventual consistency OK |
| Order history | Replica 1 | Non-critical reads |
| Search indexing | Replica 2 | No impact on primary |
| Analytics queries | Replica 1 | Heavy reads offloaded |
| Financial reports | Primary | Requires consistent reads |

### 4.2 Table Partitioning

```sql
-- Partition orders by month for query performance
CREATE TABLE orders (
    id UUID PRIMARY KEY,
    customer_id UUID NOT NULL,
    status VARCHAR(20) NOT NULL,
    total_amount DECIMAL(12,2) NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
) PARTITION BY RANGE (created_at);

-- Create monthly partitions
CREATE TABLE orders_2026_01 PARTITION OF orders
    FOR VALUES FROM ('2026-01-01') TO ('2026-02-01');
CREATE TABLE orders_2026_02 PARTITION OF orders
    FOR VALUES FROM ('2026-02-01') TO ('2026-03-01');
-- ... auto-generated via partition manager
```

### 4.3 Materialized Views for Analytics

```sql
-- Pre-computed daily sales summary
CREATE MATERIALIZED VIEW mv_daily_sales AS
SELECT
    DATE_TRUNC('day', o.created_at) AS sale_date,
    o.vendor_id,
    COUNT(DISTINCT o.id) AS order_count,
    SUM(o.total_amount) AS total_revenue,
    AVG(o.total_amount) AS avg_order_value
FROM orders o
WHERE o.status = 'COMPLETED'
GROUP BY DATE_TRUNC('day', o.created_at), o.vendor_id;

-- Refresh every 5 minutes via scheduled job
REFRESH MATERIALIZED VIEW CONCURRENTLY mv_daily_sales;
```

---

## 5. Redis Scaling

### 5.1 Redis Cluster

```
┌────────────────────────────────────────────────────────────┐
│                   REDIS CLUSTER                             │
│                                                             │
│  Master Nodes:                                              │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐                   │
│  │ Master 1│  │ Master 2│  │ Master 3│                   │
│  │0-5460   │  │5461-10922│ │10923-16383│                  │
│  └────┬────┘  └────┬────┘  └────┬────┘                   │
│       │            │            │                           │
│  Replica Nodes:                                             │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐                   │
│  │Replica 1│  │Replica 2│  │Replica 3│                   │
│  └─────────┘  └─────────┘  └─────────┘                   │
│                                                             │
│  Hash Slots: 16384 total                                   │
│  Replication Factor: 1 (1 master + 1 replica per shard)   │
└────────────────────────────────────────────────────────────┘
```

### 5.2 Cache Strategy

```
┌────────────────────────────────────────────────────────────┐
│                    CACHE ARCHITECTURE                        │
│                                                             │
│  L1: Application Cache (in-memory Map, TTL 60s)            │
│  ┌─────────────────────────────────────────────┐           │
│  │  Hot products, categories, config            │           │
│  └──────────────────────┬──────────────────────┘           │
│                          │ miss                             │
│  L2: Redis Cache (distributed, TTL varies)                  │
│  ┌─────────────────────────────────────────────┐           │
│  │  Sessions, carts, product data, search cache │           │
│  └──────────────────────┬──────────────────────┘           │
│                          │ miss                             │
│  L3: Database (PostgreSQL)                                  │
│  ┌─────────────────────────────────────────────┐           │
│  │  Source of truth                            │           │
│  └─────────────────────────────────────────────┘           │
└────────────────────────────────────────────────────────────┘
```

**Cache Invalidation Strategy:**

| Data Type | Invalidation | TTL |
|---|---|---|
| Product detail | Write-through on update | 1 hour |
| Category tree | TTL-based (infrequent changes) | 24 hours |
| Search results | TTL-based | 5 minutes |
| User session | Explicit on logout | 24 hours |
| Shopping cart | TTL-based + explicit on checkout | 30 minutes |
| Rate limiter | Sliding window (Redis) | 1 minute |
| Inventory count | Write-through on stock change | 10 minutes |

### 5.3 Cache Implementation

```typescript
// infrastructure/cache/CacheService.ts
export class CacheService {
  constructor(
    private redis: RedisCluster,
    private localCache: Map<string, { value: any; expiry: number }>,
  ) {}

  async get<T>(key: string): Promise<T | null> {
    // L1: Check local cache
    const local = this.localCache.get(key);
    if (local && local.expiry > Date.now()) {
      return local.value as T;
    }

    // L2: Check Redis
    const redisValue = await this.redis.get(key);
    if (redisValue) {
      const parsed = JSON.parse(redisValue) as T;
      // Populate L1
      this.localCache.set(key, { value: parsed, expiry: Date.now() + 60_000 });
      return parsed;
    }

    return null;
  }

  async set(key: string, value: any, ttlSeconds: number): Promise<void> {
    // L1
    this.localCache.set(key, { value, expiry: Date.now() + 60_000 });
    // L2
    await this.redis.setex(key, ttlSeconds, JSON.stringify(value));
  }

  async invalidate(pattern: string): Promise<void> {
    // Invalidate L1
    for (const key of this.localCache.keys()) {
      if (key.startsWith(pattern)) {
        this.localCache.delete(key);
      }
    }
    // Invalidate L2
    const keys = await this.redis.keys(`${pattern}*`);
    if (keys.length > 0) {
      await this.redis.del(...keys);
    }
  }
}
```

---

## 6. Elasticsearch Scaling

### 6.1 Cluster Topology

```
┌────────────────────────────────────────────────────────────┐
│              ELASTICSEARCH CLUSTER                          │
│                                                             │
│  Master Nodes:                                              │
│  ┌─────────┐  ┌─────────┐                                 │
│  │Master 1 │  │Master 2 │  (dedicated, no data)           │
│  └─────────┘  └─────────┘                                 │
│                                                             │
│  Data Nodes:                                                │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐                   │
│  │Data  1  │  │Data  2  │  │Data  3  │                   │
│  │Products │  │Orders   │  │Analytics │                   │
│  └─────────┘  └─────────┘  └─────────┘                   │
│                                                             │
│  Ingest Node:                                               │
│  ┌─────────┐                                               │
│  │ Ingest  │  (pipeline processing)                        │
│  └─────────┘                                               │
└────────────────────────────────────────────────────────────┘
```

### 6.2 Sharding Strategy

| Index | Primary Shards | Replicas | Routing |
|---|---|---|---|
| `products` | 3 | 1 | By vendor_id |
| `orders` | 5 | 1 | By customer_id |
| `reviews` | 3 | 1 | By product_id |
| `analytics_events` | 5 | 0 | By event_type |

### 6.3 Index Lifecycle Management

```
Hot → Warm → Cold → Delete

Hot (0-7 days):    SSD, full indexing, fast queries
Warm (7-30 days):  HDD, read-only, force merge
Cold (30-90 days): SSD, compressed, minimal queries
Delete (>90 days): Archived to S3, index deleted
```

---

## 7. CDN Strategy

### 7.1 Static Assets

```
┌──────────┐     ┌──────────┐     ┌──────────┐
│  Client  │────▶│   CDN    │────▶│  Origin  │
│ (Browser)│     │(CloudFlare│    │  (App)   │
└──────────┘     │  /AWS)   │     └──────────┘
                 └──────────┘
```

**Cache Rules:**

| Asset Type | CDN TTL | Origin TTL | Notes |
|---|---|---|---|
| JS/CSS bundles | 1 year | 1 year | Content-hashed filenames |
| Images | 30 days | 7 days | Cache-Control headers |
| Fonts | 1 year | 1 year | Immutable |
| HTML pages | 5 min | — | ISR for product pages |
| API responses | 0 | — | Never cached by CDN |

### 7.2 Image Optimization

```
Original Upload (MinIO/S3)
        │
        ▼
┌──────────────────┐
│  Image Processor │
│  (Sharp/Lambda)  │
└────────┬─────────┘
         │
    ┌────┼────┬────────┐
    │    │    │        │
    ▼    ▼    ▼        ▼
  Thumb  WebP  Medium  Large
  150px  auto  600px   1200px
    │    │    │        │
    └────┴────┴────────┘
         │
         ▼
    CDN Edge Cache
```

---

## 8. Performance Targets

| Metric | Target | Current | Strategy |
|---|---|---|---|
| API response (p50) | < 50ms | — | Redis cache, query optimization |
| API response (p95) | < 200ms | — | Connection pooling, read replicas |
| API response (p99) | < 500ms | — | Circuit breakers, graceful degradation |
| Page load (FCP) | < 1.5s | — | CDN, ISR, code splitting |
| Page load (LCP) | < 2.5s | — | Image optimization, preloading |
| Time to Interactive | < 3.5s | — | Lazy loading, tree shaking |
| Search latency | < 100ms | — | Elasticsearch, result caching |
| Queue processing | < 5s | — | Worker scaling, priority queues |

---

## 9. Auto-Scaling Rules

### 9.1 Kubernetes HPA

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: yemenmart-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: yemenmart-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80
    - type: Pods
      pods:
        metric:
          name: http_requests_per_second
        target:
          type: AverageValue
          averageValue: "100"
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
        - type: Percent
          value: 50
          periodSeconds: 60
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
        - type: Percent
          value: 25
          periodSeconds: 120
```

### 9.2 Queue Worker Auto-Scaling

```yaml
# BullMQ worker scaling based on queue depth
scaling_rules:
  - queue: payment.process
    min_workers: 2
    max_workers: 5
    scale_up_threshold: 10
    scale_down_threshold: 2
  - queue: notification.send
    min_workers: 2
    max_workers: 8
    scale_up_threshold: 50
    scale_down_threshold: 5
  - queue: search.reindex
    min_workers: 1
    max_workers: 3
    scale_up_threshold: 20
    scale_down_threshold: 3
```

---

## 10. Disaster Recovery

### 10.1 Backup Strategy

| Component | Method | Frequency | Retention |
|---|---|---|---|
| PostgreSQL | pg_basebackup + WAL | Daily full, continuous WAL | 30 days |
| Redis | RDB + AOF | Every 6 hours | 7 days |
| Elasticsearch | Snapshot to S3 | Daily | 30 days |
| MinIO | Replication + snapshot | Continuous | 30 days |
| Application config | Git | On change | Indefinite |

### 10.2 Recovery Targets

| Scenario | RPO | RTO | Strategy |
|---|---|---|---|
| Single node failure | 0 | < 1 min | Auto-failover to replica |
| Database corruption | 1 hour | < 30 min | Point-in-time recovery |
| Full region outage | 1 hour | < 4 hours | Cross-region backup restore |
| Data center failure | 0 | < 5 min | Multi-AZ deployment |

### 10.3 Health Checks

```typescript
// infrastructure/health/HealthCheck.ts
export class HealthCheck {
  async check(): Promise<HealthStatus> {
    const checks = await Promise.allSettled([
      this.checkDatabase(),
      this.checkRedis(),
      this.checkElasticsearch(),
      this.checkMinIO(),
      this.checkBullMQ(),
    ]);

    return {
      status: checks.every((c) => c.status === 'fulfilled') ? 'healthy' : 'degraded',
      checks: {
        database: checks[0],
        redis: checks[1],
        elasticsearch: checks[2],
        storage: checks[3],
        queue: checks[4],
      },
      timestamp: new Date(),
    };
  }
}
```

---

## 11. Cost Optimization

| Strategy | Estimated Savings | Implementation |
|---|---|---|
| Reserved instances (DB) | 30-50% | 1-year commitment |
| Spot instances (workers) | 60-70% | Fault-tolerant jobs only |
| Auto-scaling (downscale) | 20-30% | Scale to zero off-peak |
| CDN caching | 40-60% bandwidth | Aggressive cache headers |
| Connection pooling | 30% DB connections | PgBouncer |
| Query optimization | 50% DB CPU | Indexes, query plans |
| Image compression | 60% storage | WebP, responsive sizes |

---

## 12. Monitoring & Alerting

### 12.1 Key Dashboards

| Dashboard | Metrics |
|---|---|
| Application Health | Request rate, error rate, latency p50/p95/p99 |
| Database Performance | QPS, connections, cache hit ratio, slow queries |
| Queue Health | Queue depth, processing time, failure rate |
| Infrastructure | CPU, memory, disk, network per node |
| Business KPIs | Orders/min, revenue/min, conversion rate |

### 12.2 Alert Rules

| Alert | Condition | Severity | Action |
|---|---|---|---|
| High error rate | > 5% 5xx for 5 min | Critical | Page on-call |
| High latency | p95 > 1s for 5 min | Warning | Auto-scale |
| Database connections | > 80% pool | Critical | Scale DB |
| Queue backlog | > 5000 for 10 min | Warning | Scale workers |
| Disk usage | > 85% | Warning | Cleanup /扩容 |
| Memory usage | > 90% for 5 min | Critical | Restart / scale |
| Cache hit ratio | < 70% for 15 min | Warning | Review cache keys |
