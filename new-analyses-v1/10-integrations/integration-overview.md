# Integration Overview — YemenMart

## 1. Integration Architecture

YemenMart integrates with external systems through an **Adapter Pattern** — each external system is wrapped in an anti-corruption layer that translates between external APIs and internal domain interfaces. This enables provider swapping without touching business logic.

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        YEMENMART PLATFORM                               │
│                                                                         │
│  ┌───────────────────────────────────────────────────────────────────┐  │
│  │                    ADAPTER / ACL LAYER                            │  │
│  │                                                                   │  │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐            │  │
│  │  │ Payment  │ │ Delivery │ │   SMS    │ │Notifica- │            │  │
│  │  │ Adapter  │ │ Adapter  │ │ Adapter  │ │tion Adpt.│            │  │
│  │  └────┬─────┘ └────┬─────┘ └────┬─────┘ └────┬─────┘            │  │
│  └───────┼─────────────┼───────────┼─────────────┼──────────────────┘  │
│          │             │           │             │                      │
└──────────┼─────────────┼───────────┼─────────────┼──────────────────────┘
           │             │           │             │
           ▼             ▼           ▼             ▼
    ┌──────────┐  ┌──────────┐ ┌──────────┐ ┌──────────┐
    │ m-Floos  │  │  SMSA    │ │ Telesom  │ │ WhatsApp │
    │ OneCash  │  │  Aramex  │ │ Sabafon  │ │ Business │
    │ Bank     │  │  DHL     │ │          │ │ Firebase │
    └──────────┘  └──────────┘ └──────────┘ └──────────┘
```

## 2. Integration Categories

| Category | Providers | Block | Protocol | Auth Method |
|----------|-----------|-------|----------|-------------|
| Payment | m-Floos, OneCash | B05 Payment | REST/mTLS | Client cert + API key |
| Delivery | SMSA, Aramex, DHL | B07 Logistics | REST/HTTPS | API key |
| SMS | Telesom, Sabafon | B01 Identity | REST/HTTPS | API key |
| Messaging | WhatsApp Business | B11 Content | Cloud API/HTTPS | Bearer token |
| Push | Firebase Cloud Messaging | B11 Content | REST/HTTPS | Service account |
| E-Invoicing | ZATCA | B06 Finance | REST/mTLS | Client cert |
| Search | Elasticsearch 8 | B03/B09 | REST/HTTP | API key |
| Object Storage | MinIO (S3-compatible) | Infrastructure | S3 API | Access key + secret |
| Email | SMTP / SendGrid | B11 Content | SMTP/REST | API key |

## 3. Integration Principles

| Principle | Description |
|-----------|-------------|
| Adapter Pattern | Every external system wrapped in a provider adapter |
| Circuit Breaker | All external calls protected; open after 3-5 failures |
| Retry with Backoff | Exponential or linear backoff per provider SLA |
| Idempotency | All payment and financial calls use idempotency keys |
| Fallback | SMS providers failover; payment methods degrade gracefully |
| Timeout Budget | Each integration has defined timeout (5s-60s) |
| Dead Letter Queue | Failed async jobs routed to DLQ for manual intervention |
| Signature Verification | All inbound webhooks validated by HMAC signature |

## 4. Integration Registry

| ID | System | Owner | Status | SLA |
|----|--------|-------|--------|-----|
| INT-01 | m-Floos | B05 Payment | Active | 99.9% |
| INT-02 | OneCash | B05 Payment | Active | 99.9% |
| INT-03 | SMSA | B07 Logistics | Active | 99.5% |
| INT-04 | Aramex | B07 Logistics | Active | 99.5% |
| INT-05 | DHL | B07 Logistics | Active | 99.5% |
| INT-06 | Telesom SMS | B01 Identity | Active | 99.0% |
| INT-07 | Sabafon SMS | B01 Identity | Active | 99.0% |
| INT-08 | WhatsApp Business | B11 Content | Active | 99.5% |
| INT-09 | Firebase FCM | B11 Content | Active | 99.9% |
| INT-10 | ZATCA | B06 Finance | Active | 99.0% |
| INT-11 | Elasticsearch 8 | B03/B09 | Active | 99.9% |
| INT-12 | MinIO | Infrastructure | Active | 99.9% |
| INT-13 | SendGrid | B11 Content | Active | 99.5% |

## 5. Resilience Patterns

### Circuit Breaker Matrix

| Integration | Failure Threshold | Open Duration | Half-Open Requests |
|-------------|-------------------|---------------|-------------------|
| m-Floos | 5 consecutive | 60s | 3 |
| OneCash | 5 consecutive | 60s | 3 |
| SMSA | 3 consecutive | 30s | 2 |
| Aramex | 3 consecutive | 30s | 2 |
| DHL | 3 consecutive | 30s | 2 |
| Telesom SMS | 3 consecutive | 30s | 2 |
| Sabafon SMS | 3 consecutive | 30s | 2 |
| WhatsApp | 5 consecutive | 60s | 3 |
| ZATCA | 3 consecutive | 120s | 2 |
| Elasticsearch | 5 consecutive | 30s | 3 |

### Retry Policies

| Integration | Max Retries | Strategy | Initial Delay | Max Delay |
|-------------|-------------|----------|---------------|-----------|
| m-Floos | 3 | Exponential | 1s | 4s |
| OneCash | 3 | Exponential | 1s | 4s |
| SMSA | 2 | Linear | 2s | 4s |
| Aramex | 2 | Linear | 2s | 4s |
| DHL | 2 | Linear | 2s | 4s |
| Telesom SMS | 2 | Linear | 2s | 4s |
| Sabafon SMS | 2 | Linear | 2s | 4s |
| WhatsApp | 3 | Exponential | 1s | 8s |
| ZATCA | 1 | None (idempotent) | — | — |
| Elasticsearch | 2 | Linear | 1s | 2s |

### Dead Letter Queue Routing

| Source Queue | DLQ Queue | Retry Policy | Alert Threshold |
|-------------|-----------|-------------|-----------------|
| yemenmart:orders | yemenmart:orders:dlq | Manual retry | 3 failures |
| yemenmart:payments | yemenmart:payments:dlq | Auto-retry 3x | 5 failures |
| yemenmart:inventory | yemenmart:inventory:dlq | Manual retry | 3 failures |
| yemenmart:logistics | yemenmart:logistics:dlq | Auto-retry 3x | 5 failures |
| yemenmart:notifications | yemenmart:notifications:dlq | Auto-retry 5x | 10 failures |
| yemenmart:finance | yemenmart:finance:dlq | Manual retry | 1 failure |

## 6. Environment Configuration

| Environment | Base URLs | Certificates | API Keys |
|-------------|-----------|--------------|----------|
| Development | Sandbox URLs | Self-signed | Test keys |
| Staging | Staging URLs | Staging certs | Staging keys |
| Production | Production URLs | Production certs | Production keys |

### Secrets Management

All integration credentials stored in HashiCorp Vault:

```yaml
Vault Paths:
  secret/data/yemenmart/payments/m-floos:
    - api_key
    - client_cert
    - client_key
    - webhook_secret

  secret/data/yemenmart/payments/onecash:
    - api_key
    - client_cert
    - client_key
    - webhook_secret

  secret/data/yemenmart/sms/telesom:
    - api_key
    - sender_id

  secret/data/yemenmart/sms/sabafon:
    - api_key
    - sender_id

  secret/data/yemenmart/whatsapp:
    - access_token
    - phone_number_id
    - verify_token
    - app_secret

  secret/data/yemenmart/zatca:
    - client_cert
    - client_key
    - csid
```

## 7. Monitoring & Observability

### Metrics

| Metric | Description | Alert Threshold |
|--------|-------------|-----------------|
| integration_request_total | Total requests per integration | — |
| integration_request_duration_seconds | Request latency histogram | p95 > 5s |
| integration_error_total | Error count per integration | > 5/min |
| integration_circuit_breaker_open | Circuit breaker state change | Any open |
| integration_retry_total | Retry attempt count | > 10/min |
| integration_dlq_size | Dead letter queue depth | > 100 |

### Dashboards

- **Integration Health Panel**: Real-time status of all external systems
- **Payment Flow Dashboard**: m-Floos/OneCash success rates, latency, volume
- **SMS Delivery Dashboard**: Telesom/Sabafon delivery rates, costs, failures
- **Logistics Dashboard**: Delivery provider performance, SLA compliance
- **Error Rate Dashboard**: Integration error trends, circuit breaker events

## 8. Related Files

| File | Description |
|------|-------------|
| `payment-providers.md` | m-Floos and OneCash integration details |
| `delivery-providers.md` | SMSA, Aramex, DHL carrier integration |
| `sms-providers.md` | Telesom and Sabafon SMS gateway integration |
| `notification-services.md` | WhatsApp Business API, Push notification integration |
| `analytics-integrations.md` | Analytics and reporting integrations |
| `third-party-apis.md` | All third-party API documentation |
| `03-system-analysis/integration-points.md` | Complete integration point catalog |
| `09-security/security-overview.md` | Integration security requirements |
| `06-backend/payment-service.md` | Payment service implementation |
| `06-backend/notification-service.md` | Notification service implementation |

---

*Document Version: 1.0 | Last Updated: 2026-09-13 | Classification: Confidential*
