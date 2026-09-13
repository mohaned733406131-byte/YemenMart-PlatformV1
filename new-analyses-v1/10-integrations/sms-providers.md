# SMS Providers — YemenMart

## 1. Overview

YemenMart uses dual SMS providers (Telesom and Sabafon) for redundancy. SMS is critical for OTP delivery, transactional notifications, and delivery codes. Both providers are Yemeni local carriers with native network support.

```
┌─────────────────────────────────────────────────────────────────┐
│                      SMS GATEWAY ARCHITECTURE                    │
│                                                                 │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐                  │
│  │  B01     │───>│   SMS    │───>│ Provider │                  │
│  │ Identity │    │  Router  │    │ Selector │                  │
│  └──────────┘    └──────────┘    └────┬─────┘                  │
│                                       │                         │
│                  ┌────────────────────┼────────────────┐       │
│                  │                    │                │       │
│             ┌────▼─────┐        ┌────▼─────┐          │       │
│             │ Telesom  │        │ Sabafon  │          │       │
│             │   SMS    │        │   SMS    │          │       │
│             │  (Pri.)  │        │ (Backup) │          │       │
│             └────┬─────┘        └────┬─────┘          │       │
│                  │                    │                │       │
│                  └────────────────────┼────────────────┘       │
│                                       │                         │
│                                       ▼                         │
│                               ┌──────────┐                     │
│                               │  Customer│                     │
│                               │  Device  │                     │
│                               └──────────┘                     │
└─────────────────────────────────────────────────────────────────┘
```

## 2. Telesom SMS (Primary)

### Connection Details

| Property | Value |
|----------|-------|
| **Type** | SMS Gateway Provider |
| **Protocol** | REST API over HTTPS |
| **Authentication** | API key in `X-API-Key` header |
| **Base URL (Production)** | `https://sms.telesom.com/api/v1` |
| **Base URL (Sandbox)** | `https://sandbox.sms.telesom.com/api/v1` |
| **Integrating Block** | B01 Identity |
| **Timeout** | 10s |
| **Retry** | 2 attempts, linear backoff (2s, 4s) |
| **Rate Limit** | 10 SMS/second |
| **Circuit Breaker** | Open after 3 failures; half-open after 30s |

### API Endpoints

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/send` | POST | Send OTP or transactional SMS |
| `/status/{id}` | GET | Check delivery status |

### Request/Response Schemas

#### Send SMS Request

```json
{
  "idempotency_key": "uuid-v4",
  "to": "+967771234567",
  "message": "YemenMart OTP: 482916. Valid 5 minutes.",
  "sender_id": "YemenMart",
  "priority": "high",
  "callback_url": "https://api.yemenmart.com/webhooks/sms/telesom"
}
```

#### Send SMS Response (Success)

```json
{
  "status": "queued",
  "message_id": "tsm_msg_abc123",
  "to": "+967771234567",
  "cost": 10,
  "currency": "YER"
}
```

#### Delivery Status Response

```json
{
  "message_id": "tsm_msg_abc123",
  "status": "delivered",
  "delivered_at": "2026-09-13T14:30:05Z",
  "error_code": null
}
```

### Adapter Implementation

```typescript
// packages/notification-module/src/adapters/telesom-sms.adapter.ts
import { injectable, inject } from 'inversify';
import { ISmsAdapter, SmsRequest, SmsResult } from '../interfaces/sms-adapter.interface';

interface TelesomConfig {
  baseUrl: string;
  apiKey: string;
  senderId: string;
  timeout: number;
}

@injectable()
export class TelesomSmsAdapter implements ISmsAdapter {
  name = 'telesom';
  private httpClient: HttpClient;

  constructor(@inject('TelesomConfig') config: TelesomConfig) {
    this.httpClient = new HttpClient({
      baseURL: config.baseUrl,
      timeout: config.timeout,
      headers: {
        'X-API-Key': config.apiKey,
        'Content-Type': 'application/json',
      },
    });
  }

  async send(request: SmsRequest): Promise<SmsResult> {
    try {
      const response = await this.httpClient.post('/send', {
        idempotency_key: request.idempotencyKey,
        to: request.phone,
        message: request.message,
        sender_id: request.senderId || 'YemenMart',
        priority: request.priority || 'normal',
        callback_url: request.callbackUrl,
      });

      return {
        success: true,
        messageId: response.data.message_id,
        status: response.data.status,
        cost: response.data.cost,
        provider: 'telesom',
      };
    } catch (error) {
      return {
        success: false,
        error: error.response?.data?.error_code || error.message,
        provider: 'telesom',
      };
    }
  }

  async getStatus(messageId: string): Promise<SmsDeliveryStatus> {
    const response = await this.httpClient.get(`/status/${messageId}`);
    return {
      messageId: response.data.message_id,
      status: response.data.status,
      deliveredAt: response.data.delivered_at,
      errorCode: response.data.error_code,
    };
  }
}
```

## 3. Sabafon SMS (Backup)

### Connection Details

| Property | Value |
|----------|-------|
| **Type** | SMS Gateway Provider |
| **Protocol** | REST API over HTTPS |
| **Authentication** | API key in `Authorization` header |
| **Base URL (Production)** | `https://api.sabafon.com/sms/v1` |
| **Base URL (Sandbox)** | `https://sandbox.sabafon.com/sms/v1` |
| **Integrating Block** | B01 Identity |
| **Timeout** | 10s |
| **Retry** | 2 attempts, linear backoff (2s, 4s) |
| **Rate Limit** | 10 SMS/second |
| **Circuit Breaker** | Open after 3 failures; half-open after 30s |

### API Endpoints

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/send` | POST | Send OTP or transactional SMS |
| `/status/{id}` | GET | Check delivery status |

### Request/Response Schemas

#### Send SMS Request

```json
{
  "idempotency_key": "uuid-v4",
  "destination": "+967771234567",
  "text": "YemenMart OTP: 482916. Valid 5 minutes.",
  "originator": "YemenMart",
  "type": "text",
  "callback_url": "https://api.yemenmart.com/webhooks/sms/sabafon"
}
```

#### Send SMS Response (Success)

```json
{
  "result": "success",
  "id": "sbf_msg_xyz789",
  "destination": "+967771234567"
}
```

### Adapter Implementation

```typescript
// packages/notification-module/src/adapters/sabafon-sms.adapter.ts
import { injectable, inject } from 'inversify';
import { ISmsAdapter, SmsRequest, SmsResult } from '../interfaces/sms-adapter.interface';

interface SabafonConfig {
  baseUrl: string;
  apiKey: string;
  senderId: string;
  timeout: number;
}

@injectable()
export class SabafonSmsAdapter implements ISmsAdapter {
  name = 'sabafon';
  private httpClient: HttpClient;

  constructor(@inject('SabafonConfig') config: SabafonConfig) {
    this.httpClient = new HttpClient({
      baseURL: config.baseUrl,
      timeout: config.timeout,
      headers: {
        'Authorization': `Bearer ${config.apiKey}`,
        'Content-Type': 'application/json',
      },
    });
  }

  async send(request: SmsRequest): Promise<SmsResult> {
    try {
      const response = await this.httpClient.post('/send', {
        idempotency_key: request.idempotencyKey,
        destination: request.phone,
        text: request.message,
        originator: request.senderId || 'YemenMart',
        type: 'text',
        callback_url: request.callbackUrl,
      });

      return {
        success: true,
        messageId: response.data.id,
        status: 'queued',
        provider: 'sabafon',
      };
    } catch (error) {
      return {
        success: false,
        error: error.response?.data?.error || error.message,
        provider: 'sabafon',
      };
    }
  }

  async getStatus(messageId: string): Promise<SmsDeliveryStatus> {
    const response = await this.httpClient.get(`/status/${messageId}`);
    return {
      messageId: response.data.id,
      status: response.data.status,
      deliveredAt: response.data.delivered_at,
      errorCode: response.data.error_code,
    };
  }
}
```

## 4. SMS Provider Adapter Interface

```typescript
// packages/notification-module/src/interfaces/sms-adapter.interface.ts

export interface ISmsAdapter {
  name: string;
  send(request: SmsRequest): Promise<SmsResult>;
  getStatus(messageId: string): Promise<SmsDeliveryStatus>;
}

export interface SmsRequest {
  idempotencyKey: string;
  phone: string;
  message: string;
  senderId?: string;
  priority?: 'low' | 'normal' | 'high';
  callbackUrl?: string;
}

export interface SmsResult {
  success: boolean;
  messageId?: string;
  status?: string;
  cost?: number;
  error?: string;
  provider: string;
}

export interface SmsDeliveryStatus {
  messageId: string;
  status: 'queued' | 'sent' | 'delivered' | 'failed' | 'expired';
  deliveredAt?: string;
  errorCode?: string;
}
```

## 5. SMS Provider Router

```typescript
@injectable()
export class SmsProviderRouter {
  private providers: Map<string, ISmsAdapter>;
  private priority: string[] = ['telesom', 'sabafon'];

  constructor(
    @inject('TelesomSmsAdapter') private telesom: TelesomSmsAdapter,
    @inject('SabafonSmsAdapter') private sabafon: SabafonSmsAdapter,
    @inject('CircuitBreaker') private circuitBreaker: CircuitBreaker,
    @inject('RedisClient') private redis: Redis,
    @inject('Logger') private logger: ILogger,
  ) {
    this.providers = new Map([
      ['telesom', telesom],
      ['sabafon', sabafon],
    ]);
  }

  async send(request: SmsRequest): Promise<SmsResult> {
    // Check rate limit per phone number
    const rateKey = `sms:ratelimit:${request.phone}`;
    const count = await this.redis.incr(rateKey);
    if (count === 1) {
      await this.redis.expire(rateKey, 60); // 1 minute window
    }
    if (count > 3) {
      return { success: false, error: 'RATE_LIMITED', provider: 'none' };
    }

    // Try providers in priority order
    for (const providerName of this.priority) {
      const adapter = this.providers.get(providerName)!;
      const isAvailable = await this.circuitBreaker.isAvailable(providerName);

      if (!isAvailable) {
        this.logger.warn(`SMS provider ${providerName} circuit breaker open`, { phone: request.phone });
        continue;
      }

      try {
        const result = await adapter.send(request);
        if (result.success) {
          this.logger.info(`SMS sent via ${providerName}`, { phone: request.phone, messageId: result.messageId });
          return result;
        }
        this.logger.warn(`SMS send failed via ${providerName}`, { error: result.error, phone: request.phone });
      } catch (error) {
        this.logger.error(`SMS provider ${providerName} error`, { error, phone: request.phone });
      }
    }

    this.logger.error('All SMS providers failed', { phone: request.phone });
    return { success: false, error: 'ALL_PROVIDERS_FAILED', provider: 'none' };
  }

  async getStatus(messageId: string, provider: string): Promise<SmsDeliveryStatus> {
    const adapter = this.providers.get(provider);
    if (!adapter) throw new UnknownSmsProviderError(provider);
    return adapter.getStatus(messageId);
  }
}
```

## 6. SMS Templates

### OTP Templates

```typescript
const SMS_TEMPLATES = {
  REGISTRATION_OTP: {
    ar: 'YemenMart: كود التحقق الخاص بك هو {code}. صالح لمدة 5 دقائق.',
    en: 'YemenMart: Your verification code is {code}. Valid for 5 minutes.',
  },
  LOGIN_OTP: {
    ar: 'YemenMart: كود تسجيل الدخول الخاص بك هو {code}. صالح لمدة 5 دقائق.',
    en: 'YemenMart: Your login code is {code}. Valid for 5 minutes.',
  },
  PASSWORD_RESET_OTP: {
    ar: 'YemenMart: كود إعادة تعيين كلمة المرور هو {code}. صالح لمدة 5 دقائق.',
    en: 'YemenMart: Your password reset code is {code}. Valid for 5 minutes.',
  },
};

// Transactional Templates
const TRANSACTIONAL_TEMPLATES = {
  ORDER_CONFIRMED: {
    ar: 'YemenMart: تم تأكيد طلبك #{orderId}. المبلغ الإجمالي: {total} يمني.',
    en: 'YemenMart: Your order #{orderId} confirmed. Total: {total} YER.',
  },
  ORDER_SHIPPED: {
    ar: 'YemenMart: تم شحن طلبك #{orderId}. كود التوصيل: {deliveryCode}',
    en: 'YemenMart: Your order #{orderId} shipped! Delivery code: {deliveryCode}',
  },
  DELIVERY_CODE: {
    ar: 'YemenMart كود التوصيل: {code}. صالح لمدة 24 ساعة. اعرضه للسائق عند التوصيل.',
    en: 'YemenMart delivery code: {code}. Valid 24h. Show to driver on delivery.',
  },
  WALLET_TOPUP: {
    ar: 'YemenMart: تم شحن محفظتك بمبلغ {amount} يمني. الرصيد: {balance} يمني.',
    en: 'YemenMart: Wallet credited with {amount} YER. Balance: {balance} YER.',
  },
  ORDER_CANCELLED: {
    ar: 'YemenMart: تم إلغاء طلبك #{orderId}. السبب: {reason}',
    en: 'YemenMart: Your order #{orderId} cancelled. Reason: {reason}',
  },
  VENDOR_KYC_APPROVED: {
    ar: 'YemenMart: تم الموافقة على مستندات التحقق الخاصة بك. يمكنك الآن إدارة متجرك.',
    en: 'YemenMart: Your KYC documents approved. You can now manage your store.',
  },
};
```

## 7. SMS Security

### Rate Limiting

| Scenario | Limit | Window | Action |
|----------|-------|--------|--------|
| OTP requests per phone | 3 | 1 minute | Reject with rate limit error |
| OTP requests per phone | 5 | 10 minutes | Lock phone for 15 minutes |
| Bulk SMS per minute | 100 | 1 minute | Queue for processing |
| All SMS per second | 10 | 1 second | Queue excess messages |

### Phone Number Validation

```typescript
function validateYemeniPhone(phone: string): boolean {
  // Yemen phone format: +967 followed by 9 digits
  // Mobile: +967 7X XXX XXXX (70-79)
  const yemenPhoneRegex = /^\+967[7][0-9]{8}$/;
  return yemenPhoneRegex.test(phone);
}

function normalizePhone(phone: string): string {
  // Remove spaces, dashes, parentheses
  let normalized = phone.replace(/[\s\-\(\)]/g, '');
  
  // Add country code if missing
  if (normalized.startsWith('7')) {
    normalized = '+967' + normalized;
  }
  
  return normalized;
}
```

### SMS Content Security

- OTP codes are never logged in plaintext
- Phone numbers are masked in logs: `+96777****67`
- SMS content never includes full names or sensitive data
- All SMS requests include idempotency keys to prevent duplicates

## 8. Delivery Monitoring

### Metrics

| Metric | Description | Alert Threshold |
|--------|-------------|-----------------|
| sms_send_total | Total SMS sent per provider | — |
| sms_delivery_rate | Percentage delivered | < 95% |
| sms_latency_seconds | Time from send to delivery | p95 > 30s |
| sms_error_total | Failed SMS per provider | > 5/min |
| sms_rate_limit_total | Rate limited requests | > 10/hour |
| sms_cost_total | Total SMS cost per provider | > budget |

### Dashboard Panels

- **SMS Delivery Rate**: Real-time delivery rates per provider
- **SMS Cost Tracking**: Cost per message, total daily/monthly spend
- **Provider Health**: Circuit breaker status, error rates
- **Geographic Coverage**: SMS delivery success by governate

## 9. Related Files

| File | Description |
|------|-------------|
| `notification-services.md` | Multi-channel notification integration |
| `integration-overview.md` | Integration architecture overview |
| `06-backend/notification-service.md` | Notification service implementation |
| `03-system-analysis/integration-points.md` | Integration point catalog |
| `01-business-analysis/business-rules.md` | OTP and SMS business rules |

---

*Document Version: 1.0 | Last Updated: 2026-09-13 | Classification: Confidential*
