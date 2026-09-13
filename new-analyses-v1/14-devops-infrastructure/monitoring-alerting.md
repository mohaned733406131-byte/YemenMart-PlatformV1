# Monitoring & Alerting - YemenMart

## Document Information
| Property | Value |
|----------|-------|
| Document ID | YEM-INF-MON-001 |
| Version | 1.0 |
| Status | Active |
| Last Updated | 2026-09-12 |

---

## 1. Monitoring Architecture

### 1.1 Monitoring Stack
```
┌─────────────────────────────────────────────────────────┐
│                    Monitoring Stack                      │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐ │
│  │ Prometheus  │───▶│  Grafana    │    │ Alertmanager│ │
│  │ (Metrics)   │    │(Dashboard) │    │  (Alerts)   │ │
│  └─────────────┘    └─────────────┘    └─────────────┘ │
│         │                                    │         │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐ │
│  │  Node       │    │  Jaeger     │    │  PagerDuty  │ │
│  │  Exporter   │    │  (Traces)   │    │  (Incident) │ │
│  └─────────────┘    └─────────────┘    └─────────────┘ │
│                                                         │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐ │
│  │  Fluentd    │───▶│Elasticsearch│───▶│   Kibana    │ │
│  │  (Logs)     │    │  (Storage)  │    │  (Vizualize)│ │
│  └─────────────┘    └─────────────┘    └─────────────┘ │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### 1.2 Monitoring Layers
| Layer | Tools | Purpose |
|-------|-------|---------|
| Infrastructure | Prometheus + Node Exporter | CPU, Memory, Disk, Network |
| Application | Prometheus + Custom Metrics | Request rate, latency, errors |
| Business | Custom Metrics | Orders, revenue, users |
| Logs | Fluentd + ELK | Centralized logging |
| Traces | Jaeger | Distributed tracing |
| Uptime | Pingdom / UptimeRobot | External availability |

---

## 2. Prometheus Configuration

### 2.1 Prometheus Setup
| Setting | Value |
|---------|-------|
| Version | 2.47.x |
| Storage | 15-day retention |
| Scrape Interval | 15s |
| Evaluation Interval | 15s |
| TSDB Retention | 15 days |
| Remote Write | Thanos / Mimir |

### 2.2 Scrape Targets
| Target | Interval | Metrics |
|--------|----------|---------|
| Kubernetes Nodes | 15s | node_exporter |
| Kubernetes Pods | 15s | Application metrics |
| PostgreSQL | 30s | pg_exporter |
| Redis | 15s | redis_exporter |
| Elasticsearch | 30s | elasticsearch_exporter |
| Nginx Ingress | 15s | nginx_ingress |
| API Gateway | 15s | Custom metrics |

### 2.3 ServiceMonitor Configuration
```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: yemenmart-api
  namespace: monitoring
  labels:
    release: prometheus
spec:
  namespaceSelector:
    matchNames:
      - yemenmart-prod
  selector:
    matchLabels:
      app: yemenmart-api
  endpoints:
    - port: http
      path: /metrics
      interval: 15s
```

---

## 3. Key Metrics

### 3.1 RED Method Metrics
| Metric | Description | Type |
|--------|-------------|------|
| Rate | Requests per second | Counter |
| Errors | Error rate (5xx / total) | Counter |
| Duration | Response time (histogram) | Histogram |

### 3.2 USE Method Metrics
| Metric | Description | Type |
|--------|-------------|------|
| Utilization | CPU/Memory/Disk usage % | Gauge |
| Saturation | Queue depth, pending requests | Gauge |
| Errors | Error counts | Counter |

### 3.3 Application Metrics
| Metric | Labels | Description |
|--------|--------|-------------|
| http_requests_total | method, path, status | Total HTTP requests |
| http_request_duration_seconds | method, path | Request duration |
| http_request_size_bytes | method, path | Request size |
| http_response_size_bytes | method, path | Response size |
| http_requests_in_flight | - | Current requests |
| business_orders_total | status | Total orders |
| business_revenue_total | currency | Total revenue |
| business_users_total | type | Total users |

### 3.4 Infrastructure Metrics
| Metric | Description | Source |
|--------|-------------|--------|
| node_cpu_seconds_total | CPU usage | node_exporter |
| node_memory_MemAvailable_bytes | Available memory | node_exporter |
| node_filesystem_avail_bytes | Disk space | node_exporter |
| node_network_receive_bytes_total | Network traffic | node_exporter |
| container_cpu_usage_seconds_total | Container CPU | cAdvisor |
| container_memory_usage_bytes | Container memory | cAdvisor |

---

## 4. Grafana Dashboards

### 4.1 Dashboard Inventory
| Dashboard | Audience | Refresh | Panels |
|-----------|----------|---------|--------|
| Infrastructure Overview | SRE | 30s | 12 |
| Application Performance | Engineering | 15s | 15 |
| Database Performance | DBA | 30s | 10 |
| Business Metrics | Management | 60s | 8 |
| Kubernetes Cluster | SRE | 30s | 20 |
| Security Overview | Security | 60s | 10 |

### 4.2 Infrastructure Dashboard
| Panel | Metric | Visualization |
|-------|--------|--------------|
| CPU Usage | avg(rate(node_cpu_seconds_total[5m])) | Time series |
| Memory Usage | node_memory_MemAvailable_bytes | Gauge |
| Disk Usage | node_filesystem_avail_bytes | Gauge |
| Network I/O | rate(node_network_receive_bytes_total[5m]) | Time series |
| Pod Count | count(kube_pod_info) | Stat |
| Node Status | kube_node_status_condition | Status history |

### 4.3 Application Dashboard
| Panel | Metric | Visualization |
|-------|--------|--------------|
| Request Rate | sum(rate(http_requests_total[5m])) | Time series |
| Error Rate | sum(rate(http_requests_total{status=~"5.."}[5m])) | Time series |
| Response Time (p50) | histogram_quantile(0.5, ...) | Time series |
| Response Time (p95) | histogram_quantile(0.95, ...) | Time series |
| Response Time (p99) | histogram_quantile(0.99, ...) | Time series |
| Apdex Score | Custom calculation | Gauge |
| Active Connections | http_connections | Time series |
| Request Size | http_request_size_bytes | Histogram |

### 4.4 Business Dashboard
| Panel | Metric | Visualization |
|-------|--------|--------------|
| Orders per Hour | business_orders_total | Time series |
| Revenue per Hour | business_revenue_total | Time series |
| Cart Abandonment | business_cart_abandonment | Gauge |
| Conversion Rate | business_conversion_rate | Gauge |
| Active Users | business_users_active | Stat |
| Top Products | business_products_views | Bar chart |

---

## 5. Alerting Rules

### 5.1 Alert Severity Levels
| Severity | Description | Response Time | Notification |
|----------|-------------|---------------|--------------|
| Critical | System down, data loss | Immediate | Phone + Slack + Email |
| High | Major feature unavailable | 5 min | Slack + Email |
| Medium | Degraded performance | 15 min | Slack |
| Low | Minor issue, warning | 1 hour | Email |
| Info | Informational | Next business day | Dashboard |

### 5.2 Infrastructure Alerts
| Alert | Condition | Duration | Severity |
|-------|-----------|----------|----------|
| NodeDown | node_up == 0 | 5m | Critical |
| HighCPU | avg(rate(node_cpu_seconds_total[5m])) > 0.8 | 5m | Warning |
| HighMemory | (1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) > 0.85 | 5m | Warning |
| DiskSpaceLow | node_filesystem_avail_bytes / node_filesystem_size_bytes < 0.2 | 5m | Warning |
| DiskSpaceCritical | node_filesystem_avail_bytes / node_filesystem_size_bytes < 0.1 | 5m | Critical |
| NetworkHigh | rate(node_network_receive_bytes_total[5m]) > 100000000 | 5m | Warning |

### 5.3 Application Alerts
| Alert | Condition | Duration | Severity |
|-------|-----------|----------|----------|
| HighErrorRate | sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m])) > 0.01 | 5m | Critical |
| HighLatency | histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m]))) > 0.5 | 5m | Warning |
| RequestRateHigh | sum(rate(http_requests_total[5m])) > 10000 | 5m | Warning |
| PodCrashLooping | rate(kube_pod_container_status_restarts_total[15m]) > 0 | 15m | Critical |
| PodNotReady | kube_pod_status_ready{condition="false"} == 1 | 5m | Warning |
| DeploymentUnavailable | kube_deployment_status_available_replicas < kube_deployment_spec_replicas | 5m | Warning |

### 5.4 Database Alerts
| Alert | Condition | Duration | Severity |
|-------|-----------|----------|----------|
| ConnectionPoolExhausted | pg_stat_activity_count > pg_settings_max_connections * 0.8 | 5m | Critical |
| ReplicationLag | pg_replication_lag > 30 | 5m | Critical |
| SlowQueries | rate(pg_stat_activity_max_tx_duration[5m]) > 60 | 5m | Warning |
| Deadlocks | increase(pg_stat_database_deadlocks[5m]) > 0 | 5m | Warning |
| DiskSpaceLow | pg_database_size_bytes > pg_tablespace_size_bytes * 0.8 | 5m | Warning |

### 5.5 Business Alerts
| Alert | Condition | Duration | Severity |
|-------|-----------|----------|----------|
| OrdersDropped | rate(business_orders_total[1h]) < 10 | 1h | Critical |
| PaymentFailures | rate(business_payment_failures_total[5m]) > 0.05 | 5m | Critical |
| CartAbandonmentHigh | business_cart_abandonment_rate > 0.7 | 1h | Warning |
| ConversionRateLow | business_conversion_rate < 0.02 | 1h | Warning |

---

## 6. Alertmanager Configuration

### 6.1 Alertmanager Setup
| Setting | Value |
|---------|-------|
| Version | 0.26.x |
| Retention | 120 hours |
| Cluster | 3 replicas |

### 6.2 Alert Routing
```yaml
route:
  group_by: ['alertname', 'severity']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 4h
  receiver: 'slack-notifications'
  routes:
    - match:
        severity: critical
      receiver: 'pagerduty-critical'
      group_wait: 10s
    - match:
        severity: high
      receiver: 'slack-critical'
    - match:
        severity: medium
      receiver: 'slack-warning'

receivers:
  - name: 'slack-notifications'
    slack_configs:
      - channel: '#alerts'
        send_resolved: true

  - name: 'slack-critical'
    slack_configs:
      - channel: '#alerts-critical'
        send_resolved: true

  - name: 'pagerduty-critical'
    pagerduty_configs:
      - service_key: '<pagerduty-key>'

inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'instance']
```

### 6.3 Notification Channels
| Channel | Severity | Use Case |
|---------|----------|----------|
| Slack #alerts | All | General notifications |
| Slack #alerts-critical | Critical, High | Urgent issues |
| PagerDuty | Critical | Immediate response |
| Email | All | Documentation |
| SMS | Critical | Backup notification |

---

## 7. Distributed Tracing

### 7.1 Tracing Stack
| Component | Tool | Purpose |
|-----------|------|---------|
| Tracer | Jaeger | Trace collection |
| Propagation | W3C Trace Context | Context propagation |
| Sampling | Probabilistic (10%) | Cost optimization |
| Storage | Elasticsearch | Trace storage |

### 7.2 Trace Configuration
| Setting | Value |
|---------|-------|
| Endpoint | jaeger-collector.monitoring:14268 |
| Sampling Rate | 10% (production) |
| Max Spans per Trace | 1000 |
| Max Tag Length | 1024 |

### 7.3 Trace Sampling Rules
| Condition | Sampling Rate |
|-----------|--------------|
| Default | 10% |
| Error responses | 100% |
| Slow requests (> 1s) | 100% |
| Payment operations | 100% |
| Order operations | 50% |
| Search operations | 5% |

---

## 8. Uptime Monitoring

### 8.1 External Checks
| Check | URL/Endpoint | Interval | Regions |
|-------|-------------|----------|---------|
| Homepage | https://yemenmart.com | 1 min | SA, EU, US |
| API Health | https://api.yemenmart.com/health | 1 min | SA, EU, US |
| Login | https://yemenmart.com/login | 5 min | SA |
| Payment | https://api.yemenmart.com/payment/health | 1 min | SA |
| Search | https://api.yemenmart.com/search/health | 5 min | SA |

### 8.2 SLA Monitoring
| Metric | Target | Alert |
|--------|--------|-------|
| Uptime | 99.99% | < 99.95% |
| Response Time (p95) | < 2s | > 3s |
| SSL Certificate | Valid | < 30 days to expiry |
| DNS Resolution | Working | Any failure |

---

## 9. Log Management

### 9.1 Log Stack
| Component | Tool | Purpose |
|-----------|------|---------|
| Collection | Fluentd | Log aggregation |
| Storage | Elasticsearch | Log storage |
| Visualization | Kibana | Log exploration |
| Retention | ILM | Lifecycle management |

### 9.2 Log Levels
| Level | Usage | Production |
|-------|-------|------------|
| ERROR | System errors | Enabled |
| WARN | Potential issues | Enabled |
| INFO | Business events | Enabled |
| DEBUG | Detailed info | Disabled |
| TRACE | Very detailed | Disabled |

### 9.3 Structured Logging Format
```json
{
  "timestamp": "2026-09-12T10:30:00Z",
  "level": "INFO",
  "service": "yemenmart-api",
  "traceId": "abc123",
  "spanId": "def456",
  "message": "Order created",
  "orderId": "ORD-12345",
  "userId": "USR-67890",
  "amount": 150.00,
  "currency": "SAR"
}
```

---

## 10. Monitoring Deployment

### 10.1 Helm Chart Values
| Component | Replicas | CPU | Memory | Storage |
|-----------|----------|-----|--------|---------|
| Prometheus | 2 | 2 cores | 4Gi | 50GB |
| Grafana | 2 | 1 core | 2Gi | 10GB |
| Alertmanager | 3 | 500m | 1Gi | 5GB |
| Elasticsearch | 3 | 4 cores | 16Gi | 500GB |
| Kibana | 2 | 1 core | 2Gi | - |
| Jaeger | 2 | 1 core | 2Gi | 100GB |
| Fluentd | 3 | 1 core | 2Gi | - |

### 10.2 Resource Allocation
| Component | Request | Limit | Priority |
|-----------|---------|-------|----------|
| Prometheus | 2 CPU, 4Gi | 4 CPU, 8Gi | High |
| Grafana | 1 CPU, 2Gi | 2 CPU, 4Gi | Medium |
| Alertmanager | 500m, 1Gi | 1 CPU, 2Gi | High |
| Elasticsearch | 4 CPU, 16Gi | 8 CPU, 32Gi | High |

---

## 11. Incident Management Integration

### 11.1 Incident Workflow
| Step | Action | Tool |
|------|--------|------|
| 1 | Alert fires | Prometheus/Alertmanager |
| 2 | Create incident | PagerDuty |
| 3 | Notify team | Slack + Phone |
| 4 | Investigate | Grafana + Jaeger |
| 5 | Mitigate | Runbook |
| 6 | Resolve | PagerDuty |
| 7 | Post-mortem | Confluence |

### 11.2 On-Call Rotation
| Role | Schedule | Escalation |
|------|----------|------------|
| Primary | Weekly rotation | After 15 min |
| Secondary | Weekly rotation | After 30 min |
| Manager | Monthly rotation | After 1 hour |

---

## 12. Cost Optimization

### 12.1 Metrics Retention
| Metric Type | Retention | Storage |
|------------|-----------|---------|
| Raw metrics | 15 days | Prometheus |
| Downsampled (5m) | 30 days | Prometheus |
| Downsampled (1h) | 90 days | Thanos |
| Aggregated (1d) | 1 year | Thanos |

### 12.2 Cost Allocation
| Component | Monthly Cost | Optimization |
|-----------|-------------|-------------|
| Prometheus | $500 | Retention policy |
| Grafana | $200 | - |
| Elasticsearch | $1,500 | ILM, rollover |
| Jaeger | $300 | Sampling |
| Fluentd | $200 | - |
| **Total** | **$2,700** | - |
