# Logging Strategy - YemenMart

## Document Information
| Property | Value |
|----------|-------|
| Document ID | YEM-INF-LOG-001 |
| Version | 1.0 |
| Status | Active |
| Last Updated | 2026-09-12 |

---

## 1. Logging Architecture

### 1.1 Log Stack Overview
```
┌─────────────────────────────────────────────────────────┐
│                    Logging Architecture                  │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌───────────┐  ┌───────────┐  ┌───────────┐          │
│  │ App Pod 1 │  │ App Pod 2 │  │ App Pod N │          │
│  └─────┬─────┘  └─────┬─────┘  └─────┬─────┘          │
│        │              │              │                  │
│        └──────────────┼──────────────┘                  │
│                       │                                  │
│              ┌────────▼────────┐                        │
│              │   Fluentd       │                        │
│              │  (Collection)   │                        │
│              └────────┬────────┘                        │
│                       │                                  │
│        ┌──────────────┼──────────────┐                  │
│        │              │              │                  │
│  ┌─────▼─────┐  ┌─────▼─────┐  ┌─────▼─────┐          │
│  │Elasticsearch│  │    S3     │  │ CloudWatch│          │
│  │ (Hot/Warm) │  │ (Archive) │  │  (Alerts) │          │
│  └─────┬─────┘  └───────────┘  └───────────┘          │
│        │                                                  │
│  ┌─────▼─────┐                                          │
│  │  Kibana   │                                          │
│  │(Analyze)  │                                          │
│  └───────────┘                                          │
└─────────────────────────────────────────────────────────┘
```

### 1.2 Components
| Component | Tool | Version | Purpose |
|-----------|------|---------|---------|
| Collection | Fluentd | 1.16 | Log aggregation |
| Processing | Fluentd | 1.16 | Parsing, filtering, enrichment |
| Storage | Elasticsearch | 8.11 | Searchable storage |
| Archive | S3 + Glacier | - | Long-term storage |
| Visualization | Kibana | 8.11 | Log exploration |
| Alerting | ElastAlert2 | - | Log-based alerts |

---

## 2. Log Sources

### 2.1 Application Logs
| Source | Format | Level | Retention |
|--------|--------|-------|-----------|
| API Gateway | JSON | INFO+ | 30 days |
| Auth Service | JSON | INFO+ | 30 days |
| Product Service | JSON | INFO+ | 30 days |
| Order Service | JSON | INFO+ | 30 days |
| Payment Service | JSON | INFO+ | 30 days |
| Inventory Service | JSON | INFO+ | 30 days |
| Search Service | JSON | INFO+ | 30 days |
| Notification Service | JSON | INFO+ | 30 days |
| Admin Service | JSON | INFO+ | 30 days |
| Web Frontend | JSON | ERROR+ | 30 days |

### 2.2 Infrastructure Logs
| Source | Format | Level | Retention |
|--------|--------|-------|-----------|
| Kubernetes Events | JSON | - | 7 days |
| Node System Logs | Syslog | - | 7 days |
| Container Logs | JSON | - | 7 days |
| Ingress Logs | Combined | - | 30 days |
| DNS Logs | JSON | - | 7 days |

### 2.3 Database Logs
| Source | Format | Level | Retention |
|--------|--------|-------|-----------|
| PostgreSQL Query | CSV | DEBUG+ | 7 days |
| PostgreSQL Error | CSV | ERROR+ | 30 days |
| PostgreSQL Audit | CSV | INFO+ | 30 days |
| Redis Slow Log | Native | - | 7 days |

### 2.4 Security Logs
| Source | Format | Level | Retention |
|--------|--------|-------|-----------|
| Authentication | JSON | INFO+ | 1 year |
| Authorization | JSON | INFO+ | 1 year |
| API Access | JSON | INFO+ | 1 year |
| Admin Actions | JSON | INFO+ | 1 year |
| Data Access | JSON | INFO+ | 1 year |

---

## 3. Log Format

### 3.1 Structured Log Schema
```json
{
  "timestamp": "2026-09-12T10:30:00.123Z",
  "level": "INFO",
  "service": "yemenmart-api",
  "version": "1.2.3",
  "traceId": "abc123def456",
  "spanId": "789ghi012",
  "message": "Order created successfully",
  "logger": "com.yemenmart.order.OrderService",
  "thread": "http-nio-8080-exec-1",
  "host": "yemenmart-api-abc123",
  "kubernetes": {
    "namespace": "yemenmart-prod",
    "pod": "yemenmart-api-abc123",
    "container": "api",
    "node": "ip-10-0-1-100"
  },
  "context": {
    "userId": "USR-67890",
    "orderId": "ORD-12345",
    "sessionId": "sess-abc123",
    "requestId": "req-def456"
  },
  "metrics": {
    "duration": 150,
    "statusCode": 200,
    "method": "POST",
    "path": "/api/v1/orders"
  },
  "tags": ["order", "success"]
}
```

### 3.2 Log Level Usage
| Level | When to Use | Production |
|-------|-------------|------------|
| ERROR | System errors, exceptions | Enabled |
| WARN | Potential issues, degraded | Enabled |
| INFO | Business events, operations | Enabled |
| DEBUG | Detailed diagnostic info | Disabled |
| TRACE | Very detailed, method entry/exit | Disabled |

### 3.3 Log Level Configuration
| Environment | Level | Rationale |
|-------------|-------|-----------|
| Development | DEBUG | Full visibility |
| QA | INFO | Standard debugging |
| Staging | INFO | Pre-production validation |
| Production | WARN | Minimal performance impact |

---

## 4. Fluentd Configuration

### 4.1 Fluentd Setup
| Setting | Value |
|---------|-------|
| Version | 1.16 |
| Deployment | DaemonSet |
| Buffer Size | 64MB |
| Flush Interval | 5s |
| Retry Max | 10 |
| Parallel Workers | 4 |

### 4.2 Input Configuration
```xml
<source>
  @type tail
  path /var/log/containers/*.log
  pos_file /var/log/fluentd-containers.log.pos
  tag kubernetes.*
  read_from_head true
  <parse>
    @type json
    time_key time
    time_format %Y-%m-%dT%H:%M:%S.%NZ
  </parse>
</source>

<source>
  @type tail
  path /var/log/kubernetes/audit/*.log
  pos_file /var/log/fluentd-audit.log.pos
  tag audit.*
  <parse>
    @type json
  </parse>
</source>
```

### 4.3 Filter Configuration
```xml
<filter kubernetes.**>
  @type kubernetes_metadata
  @id filter_kube_metadata
  kubernetes_url "https://#{ENV['KUBERNETES_SERVICE_HOST']}:#{ENV['KUBERNETES_SERVICE_PORT']}"
  verify_ssl true
  ca_file /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
  bearer_token_file /var/run/secrets/kubernetes.io/serviceaccount/token
  skip_labels false
  skip_container_metadata false
  skip_namespace_metadata true
  skip_master_url true
</filter>

<filter kubernetes.**>
  @type record_transformer
  enable_ruby true
  <record>
    hostname "#{Socket.gethostname}"
    environment "#{ENV['ENVIRONMENT']}"
    service "#{ENV['SERVICE_NAME']}"
  </record>
</filter>
```

### 4.4 Output Configuration
```xml
<match kubernetes.**>
  @type elasticsearch
  host elasticsearch-master.monitoring
  port 9200
  scheme https
  ssl_verify false
  index_name logs-${tag_parts[2]}-${time_slice}
  type_name _doc
  include_tag_key true
  logstash_format true
  logstash_prefix logs
  logstash_dateformat %Y.%m.%d
  request_timeout 30s
  reload_connections false
  reconnect_on_error true
  reload_on_failure true
  <buffer tag, time>
    @type file
    path /var/log/fluentd-buffers/kubernetes.buffer
    flush_mode interval
    flush_thread_count 4
    flush_interval 5s
    retry_type exponential_backoff
    retry_forever true
    retry_max_interval 30
    chunk_limit_size 64M
    queue_limit_length 128
    total_limit_size 8G
  </buffer>
</match>

<match audit.**>
  @type s3
  aws_key_id "#{ENV['AWS_ACCESS_KEY_ID']}"
  aws_secret_key "#{ENV['AWS_SECRET_ACCESS_KEY']}"
  s3_bucket yemenmart-prod-logs
  s3_region me-south-1
  path audit/%Y/%m/%d/
  buffer_path /var/log/fluentd-buffers/s3.buffer
  time_slice_format %Y%m%d%H
  time_slice_wait 10m
  <buffer time>
    @type file
    path /var/log/fluentd-buffers/s3.buffer
    flush_mode interval
    flush_interval 10m
    retry_type exponential_backoff
    retry_max 10
    chunk_limit_size 256M
    total_limit_size 2G
  </buffer>
</match>
```

---

## 5. Elasticsearch Configuration

### 5.1 Cluster Setup
| Setting | Value |
|---------|-------|
| Version | 8.11 |
| Nodes | 3 data + 3 master |
| Shards per Index | 3 |
| Replicas per Shard | 1 |
| Refresh Interval | 30s |

### 5.2 Index Lifecycle Management (ILM)
```json
{
  "policy": {
    "phases": {
      "hot": {
        "min_age": "0ms",
        "actions": {
          "rollover": {
            "max_primary_shard_size": "50gb",
            "max_age": "1d"
          },
          "set_priority": {
            "priority": 100
          }
        }
      },
      "warm": {
        "min_age": "7d",
        "actions": {
          "shrink": {
            "number_of_shards": 1
          },
          "forcemerge": {
            "max_num_segments": 1
          },
          "set_priority": {
            "priority": 50
          }
        }
      },
      "cold": {
        "min_age": "30d",
        "actions": {
          "set_priority": {
            "priority": 0
          }
        }
      },
      "delete": {
        "min_age": "90d",
        "actions": {
          "delete": {}
        }
      }
    }
  }
}
```

### 5.3 Index Templates
```json
{
  "index_patterns": ["logs-*"],
  "template": {
    "settings": {
      "number_of_shards": 3,
      "number_of_replicas": 1,
      "index.lifecycle.name": "logs-policy",
      "index.lifecycle.rollover_alias": "logs"
    },
    "mappings": {
      "properties": {
        "@timestamp": { "type": "date" },
        "level": { "type": "keyword" },
        "service": { "type": "keyword" },
        "message": { "type": "text" },
        "traceId": { "type": "keyword" },
        "spanId": { "type": "keyword" },
        "host": { "type": "keyword" },
        "kubernetes": {
          "properties": {
            "namespace": { "type": "keyword" },
            "pod": { "type": "keyword" },
            "container": { "type": "keyword" }
          }
        }
      }
    }
  }
}
```

---

## 6. Kibana Configuration

### 6.1 Index Patterns
| Pattern | Purpose | Time Field |
|---------|---------|------------|
| logs-* | Application logs | @timestamp |
| audit-* | Audit logs | @timestamp |
| nginx-* | Ingress logs | @timestamp |
| postgres-* | Database logs | @timestamp |

### 6.2 Saved Searches
| Search | Query | Columns |
|--------|-------|---------|
| Errors | level:ERROR | timestamp, service, message |
| Warnings | level:WARN | timestamp, service, message |
| Slow Requests | duration:>1000 | timestamp, service, path, duration |
| Failed Orders | orderId:* AND level:ERROR | timestamp, orderId, message |
| Auth Failures | message:"authentication failed" | timestamp, userId, ip |

### 6.3 Dashboards
| Dashboard | Panels | Refresh |
|-----------|--------|---------|
| Application Overview | 10 | 30s |
| Error Analysis | 8 | 1m |
| Performance Metrics | 6 | 30s |
| Security Events | 8 | 1m |
| Business Events | 6 | 5m |

---

## 7. Log-Based Alerting

### 7.1 Alert Rules
| Alert | Query | Threshold | Severity |
|-------|-------|-----------|----------|
| High Error Rate | level:ERROR | > 10/min | Critical |
| Authentication Failures | message:"auth failed" | > 5/min | Warning |
| Slow Queries | message:"slow query" | > 3/min | Warning |
| Payment Failures | service:payment AND level:ERROR | > 2/min | Critical |
| Disk Space | message:"disk space" | > 0 | Warning |

### 7.2 Alert Configuration
```yaml
name: High Error Rate
type: frequency
index: logs-*
num_events: 10
timeframe:
  minutes: 1
filter:
  - term:
      level: ERROR
alert:
  - "slack://#alerts-critical"
subject: "High Error Rate Detected"
```

---

## 8. Log Retention

### 8.1 Retention Policy
| Log Type | Hot | Warm | Cold | Archive | Total |
|----------|-----|------|------|---------|-------|
| Application | 7 days | 23 days | 60 days | 1 year | 1 year |
| Audit | 30 days | 60 days | 1 year | 7 years | 7 years |
| Security | 30 days | 60 days | 1 year | 7 years | 7 years |
| Infrastructure | 7 days | 23 days | 60 days | - | 90 days |
| Database | 7 days | 23 days | - | - | 30 days |

### 8.2 Storage Tiers
| Tier | Storage | Cost | Query Speed |
|------|---------|------|-------------|
| Hot | SSD | $$$ | < 100ms |
| Warm | HDD | $$ | < 500ms |
| Cold | S3 Standard | $ | < 5s |
| Archive | S3 Glacier | $ | Minutes to hours |

### 8.3 S3 Lifecycle Rules
```json
{
  "Rules": [
    {
      "ID": "Logs Lifecycle",
      "Status": "Enabled",
      "Filter": { "Prefix": "logs/" },
      "Transitions": [
        { "Days": 30, "StorageClass": "STANDARD_IA" },
        { "Days": 90, "StorageClass": "GLACIER" },
        { "Days": 365, "StorageClass": "DEEP_ARCHIVE" }
      ],
      "Expiration": { "Days": 2555 }
    }
  ]
}
```

---

## 9. Log Security

### 9.1 Sensitive Data Handling
| Data Type | Treatment | Example |
|-----------|-----------|---------|
| Passwords | Redact | `***REDACTED***` |
| Credit Cards | Mask | `****-****-****-1234` |
| National ID | Mask | `****-****-1234` |
| Email | Partial mask | `u***@example.com` |
| Phone | Mask | `****-1234` |
| JWT Tokens | Redact | `***REDACTED***` |
| API Keys | Redact | `***REDACTED***` |

### 9.2 Log Sanitization Rules
```ruby
# Fluentd filter for sanitization
<filter app.**>
  @type record_transformer
  enable_ruby true
  <record>
    message ${record["message"].gsub(/password=.*/, "password=***REDACTED***")}
    message ${record["message"].gsub(/credit_card=.*/, "credit_card=***REDACTED***")}
    message ${record["message"].gsub(/national_id=.*/, "national_id=***REDACTED***")}
  </record>
</filter>
```

### 9.3 Access Control
| Role | Access | Duration |
|------|--------|----------|
| Developer | Application logs (own service) | 30 days |
| SRE | All infrastructure logs | 90 days |
| Security | Security + audit logs | 1 year |
| Compliance | Audit logs | 7 years |
| Admin | Full access | 7 days |

---

## 10. Log Monitoring

### 10.1 Key Metrics
| Metric | Target | Alert |
|--------|--------|-------|
| Log Ingestion Rate | Steady | Sudden drop/spike |
| Elasticsearch Index Size | Growing predictable | Unusual growth |
| Elasticsearch Query Time | < 1s | > 5s |
| Fluentd Buffer Usage | < 80% | > 90% |
| Log Error Rate | < 0.1% | > 1% |

### 10.2 Health Checks
| Component | Check | Interval |
|-----------|-------|----------|
| Fluentd | Pod status | 30s |
| Elasticsearch | Cluster health | 30s |
| Kibana | HTTP 200 | 1m |
| S3 | Bucket access | 5m |

---

## 11. Disaster Recovery

### 11.1 Backup Strategy
| Component | Method | Frequency | Retention |
|-----------|--------|-----------|-----------|
| Elasticsearch Snapshots | S3 Snapshot | Daily | 30 days |
| Fluentd Configuration | Git | Real-time | Indefinite |
| Kibana Saved Objects | Export | Daily | 30 days |
| ILM Policies | Export | Daily | 30 days |

### 11.2 Recovery Procedures
| Scenario | Method | RTO |
|----------|--------|-----|
| Pod Failure | Auto-restart | < 1 min |
| Node Failure | Reschedule | < 5 min |
| Elasticsearch Failure | Restore snapshot | < 1 hour |
| Complete Loss | Rebuild from backup | < 4 hours |

---

## 12. Cost Optimization

### 12.1 Cost Breakdown
| Component | Monthly Cost | Optimization |
|-----------|-------------|-------------|
| Elasticsearch | $2,000 | ILM, rollover |
| S3 Storage | $200 | Lifecycle policies |
| Fluentd | $100 | DaemonSet |
| Kibana | $200 | - |
| **Total** | **$2,500** | - |

### 12.2 Optimization Strategies
| Strategy | Expected Savings |
|----------|-----------------|
| ILM Policies | 40% |
| Log Level Filtering | 30% |
| Sampling (DEBUG/TRACE) | 20% |
| Compression | 10% |
