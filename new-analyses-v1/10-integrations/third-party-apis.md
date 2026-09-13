# Third-Party APIs — YemenMart

## 1. Overview

Complete documentation of all third-party APIs consumed by YemenMart. Each integration is wrapped in an adapter for provider swapping and testing.

## 2. API Registry

| ID | System | Category | Base URL | Auth | Status |
|----|--------|----------|----------|------|--------|
| TPA-01 | m-Floos | Payment | `https://api.m-floos.com/v1` | mTLS + API key | Active |
| TPA-02 | OneCash | Payment | `https://api.onecash.ye/v2` | mTLS + API key | Active |
| TPA-03 | Telesom SMS | SMS | `https://sms.telesom.com/api/v1` | API key | Active |
| TPA-04 | Sabafon SMS | SMS | `https://api.sabafon.com/sms/v1` | API key | Active |
| TPA-05 | WhatsApp Business | Messaging | `https://graph.facebook.com/v17.0/{phone_id}` | Bearer token | Active |
| TPA-06 | Firebase FCM | Push | `https://fcm.googleapis.com/v1/projects/{project}/messages:send` | Service account | Active |
| TPA-07 | ZATCA | Compliance | `https://gw-fatoora.zatca.gov.sa/e-invoicing/shipment` | mTLS | Active |
| TPA-08 | Elasticsearch 8 | Search | `https://es-cluster.yemenmart.com` | API key | Active |
| TPA-09 | MinIO | Storage | `https://minio.yemenmart.com` | S3 access key | Active |
| TPA-10 | SendGrid | Email | `https://api.sendgrid.com/v3` | API key | Active |
| TPA-11 | SMSA | Delivery | `https://api.smsa.net/v2` | API key | Active |
| TPA-12 | Aramex | Delivery | `https://api.aramex.com/v2` | API key | Active |
| TPA-13 | DHL | Delivery | `https://api.dhl.com/mydhlapi/v2` | OAuth2 | Active |

## 3. API Connection Standards

### Request Configuration

```typescript
// packages/shared-module/src/http-client.ts
import axios, { AxiosInstance, AxiosRequestConfig } from 'axios';

interface ApiClientConfig {
  baseURL: string;
  timeout: number;
  retryConfig: RetryConfig;
  certificate?: {
    cert: Buffer;
    key: Buffer;
  };
  headers?: Record<string, string>;
}

interface RetryConfig {
  maxRetries: number;
  retryDelay: number;
  maxRetryDelay: number;
  retryableStatuses: number[];
}

export function createApiClient(config: ApiClientConfig): AxiosInstance {
  const axiosConfig: AxiosRequestConfig = {
    baseURL: config.baseURL,
    timeout: config.timeout,
    headers: {
      'Content-Type': 'application/json',
      'X-Request-ID': generateRequestId(),
      ...config.headers,
    },
  };

  if (config.certificate) {
    axiosConfig.httpsAgent = new https.Agent({
      cert: config.certificate.cert,
      key: config.certificate.key,
      rejectUnauthorized: true,
    });
  }

  const client = axios.create(axiosConfig);

  // Retry interceptor
  client.interceptors.response.use(
    (response) => response,
    async (error) => {
      const config = error.config;
      if (!config || !config.retryConfig) throw error;

      const { maxRetries, retryDelay, maxRetryDelay, retryableStatuses } = config.retryConfig;
      const retryCount = config._retryCount || 0;

      if (
        retryCount < maxRetries &&
        retryableStatuses.includes(error.response?.status || 500)
      ) {
        config._retryCount = retryCount + 1;
        const delay = Math.min(retryDelay * Math.pow(2, retryCount), maxRetryDelay);
        await new Promise(resolve => setTimeout(resolve, delay));
        return client(config);
      }

      throw error;
    }
  );

  return client;
}

// Usage
const mfloosClient = createApiClient({
  baseURL: process.env.MFLOOS_BASE_URL,
  timeout: 30000,
  retryConfig: {
    maxRetries: 3,
    retryDelay: 1000,
    maxRetryDelay: 4000,
    retryableStatuses: [408, 429, 500, 502, 503, 504],
  },
  certificate: {
    cert: Buffer.from(process.env.MFLOOS_CLIENT_CERT),
    key: Buffer.from(process.env.MFLOOS_CLIENT_KEY),
  },
  headers: {
    'X-API-Key': process.env.MFLOOS_API_KEY,
  },
});
```

### Response Mapping

```typescript
// packages/shared-module/src/api-response.mapper.ts
export interface StandardApiResponse<T> {
  success: boolean;
  data?: T;
  error?: {
    code: string;
    message: string;
    details?: any;
  };
  metadata: {
    requestId: string;
    timestamp: Date;
    provider: string;
  };
}

export function mapProviderResponse<T>(
  providerName: string,
  providerResponse: any,
  mapper: (data: any) => T,
): StandardApiResponse<T> {
  return {
    success: providerResponse.status === 'success' || providerResponse.status === 'completed',
    data: providerResponse.data ? mapper(providerResponse.data) : undefined,
    error: providerResponse.error ? {
      code: providerResponse.error.code || providerResponse.error.error_code,
      message: providerResponse.error.message || providerResponse.error.error,
    } : undefined,
    metadata: {
      requestId: generateRequestId(),
      timestamp: new Date(),
      provider: providerName,
    },
  };
}
```

## 4. Circuit Breaker Pattern

```typescript
// packages/shared-module/src/circuit-breaker.ts
import { injectable, inject } from 'inversify';

interface CircuitBreakerConfig {
  failureThreshold: number;
  openDuration: number;
  halfOpenRequests: number;
}

interface CircuitBreakerState {
  status: 'closed' | 'open' | 'half-open';
  failureCount: number;
  lastFailureTime: number;
  halfOpenAttempts: number;
}

@injectable()
export class CircuitBreaker {
  private states: Map<string, CircuitBreakerState> = new Map();
  private config: Map<string, CircuitBreakerConfig> = new Map();

  constructor(@inject('RedisClient') private redis: Redis) {}

  register(name: string, config: CircuitBreakerConfig) {
    this.config.set(name, config);
    this.states.set(name, {
      status: 'closed',
      failureCount: 0,
      lastFailureTime: 0,
      halfOpenAttempts: 0,
    });
  }

  async execute<T>(name: string, fn: () => Promise<T>): Promise<T> {
    const state = this.getState(name);
    const config = this.config.get(name)!;

    if (state.status === 'open') {
      if (Date.now() - state.lastFailureTime > config.openDuration) {
        state.status = 'half-open';
        state.halfOpenAttempts = 0;
      } else {
        throw new CircuitBreakerOpenError(name);
      }
    }

    if (state.status === 'half-open' && state.halfOpenAttempts >= config.halfOpenRequests) {
      throw new CircuitBreakerOpenError(name);
    }

    try {
      const result = await fn();
      this.onSuccess(name);
      return result;
    } catch (error) {
      this.onFailure(name);
      throw error;
    }
  }

  async isAvailable(name: string): Promise<boolean> {
    const state = this.getState(name);
    const config = this.config.get(name)!;

    if (state.status === 'closed') return true;
    if (state.status === 'open' && Date.now() - state.lastFailureTime > config.openDuration) {
      return true;
    }
    return false;
  }

  private getState(name: string): CircuitBreakerState {
    if (!this.states.has(name)) {
      this.states.set(name, {
        status: 'closed',
        failureCount: 0,
        lastFailureTime: 0,
        halfOpenAttempts: 0,
      });
    }
    return this.states.get(name)!;
  }

  private onSuccess(name: string) {
    const state = this.getState(name);
    state.failureCount = 0;
    state.status = 'closed';
  }

  private onFailure(name: string) {
    const state = this.getState(name);
    const config = this.config.get(name)!;

    state.failureCount++;
    state.lastFailureTime = Date.now();

    if (state.failureCount >= config.failureThreshold) {
      state.status = 'open';
      logger.warn(`Circuit breaker opened for ${name}`, { failureCount: state.failureCount });
    }
  }
}
```

## 5. Webhook Verification

```typescript
// packages/shared-module/src/webhook-verification.ts
import * as crypto from 'crypto';

export function verifyWebhookSignature(
  payload: Buffer | string,
  signature: string,
  secret: string,
  algorithm: 'sha256' | 'sha512' = 'sha256',
): boolean {
  const expectedSignature = crypto
    .createHmac(algorithm, secret)
    .update(payload)
    .digest('hex');

  return crypto.timingSafeEqual(
    Buffer.from(signature),
    Buffer.from(expectedSignature)
  );
}

export function verifyWhatsAppSignature(payload: Buffer, signature: string): boolean {
  return verifyWebhookSignature(payload, signature, process.env.WHATSAPP_APP_SECRET);
}

export function verifyMFloosSignature(payload: Buffer, signature: string): boolean {
  return verifyWebhookSignature(payload, signature, process.env.MFLOOS_WEBHOOK_SECRET);
}

export function verifyOneCashSignature(payload: Buffer, signature: string): boolean {
  return verifyWebhookSignature(payload, signature, process.env.ONECASH_WEBHOOK_SECRET);
}

export function verifySmsaSignature(payload: Buffer, signature: string): boolean {
  return verifyWebhookSignature(payload, signature, process.env.SMSA_WEBHOOK_SECRET);
}

export function verifyAramexSignature(payload: Buffer, signature: string): boolean {
  return verifyWebhookSignature(payload, signature, process.env.ARAMEX_WEBHOOK_SECRET);
}
```

## 6. Idempotency Key Management

```typescript
// packages/shared-module/src/idempotency.ts
import { injectable, inject } from 'inversify';

@injectable()
export class IdempotencyManager {
  constructor(
    @inject('RedisClient') private redis: Redis,
    @inject('PrismaClient') private prisma: PrismaClient,
  ) {}

  async checkAndReserve(key: string, ttl: number = 86400): Promise<boolean> {
    // Check Redis first (fast)
    const exists = await this.redis.exists(`idempotent:${key}`);
    if (exists) return false;

    // Reserve in Redis
    const reserved = await this.redis.set(`idempotent:${key}`, 'reserved', 'EX', ttl, 'NX');
    if (!reserved) return false;

    // Persist to database
    await this.prisma.idempotencyKey.create({
      data: {
        key,
        status: 'reserved',
        expiresAt: new Date(Date.now() + ttl * 1000),
      },
    });

    return true;
  }

  async complete(key: string, response: any): Promise<void> {
    await this.redis.setex(`idempotent:${key}`, 86400, JSON.stringify(response));
    await this.prisma.idempotencyKey.update({
      where: { key },
      data: { status: 'completed', response },
    });
  }

  async getResponse(key: string): Promise<any | null> {
    const cached = await this.redis.get(`idempotent:${key}`);
    return cached ? JSON.parse(cached) : null;
  }
}
```

## 7. API Rate Limiting

```typescript
// packages/shared-module/src/rate-limiter.ts
export const API_RATE_LIMITS: Record<string, RateLimitConfig> = {
  'm-floos': {
    requests: 100,
    window: 60, // seconds
    burst: 20,
  },
  'onecash': {
    requests: 100,
    window: 60,
    burst: 20,
  },
  'telesom': {
    requests: 10,
    window: 1,
    burst: 10,
  },
  'sabafon': {
    requests: 10,
    window: 1,
    burst: 10,
  },
  'whatsapp': {
    requests: 80,
    window: 60,
    burst: 20,
  },
  'smsa': {
    requests: 50,
    window: 60,
    burst: 10,
  },
  'aramex': {
    requests: 50,
    window: 60,
    burst: 10,
  },
  'dhl': {
    requests: 30,
    window: 60,
    burst: 10,
  },
};
```

## 8. Error Classification

| Error Category | HTTP Status | Retryable | Action |
|----------------|-------------|-----------|--------|
| Authentication Failed | 401 | No | Check credentials |
| Rate Limited | 429 | Yes (after delay) | Wait for retry-after header |
| Server Error | 500-503 | Yes | Retry with backoff |
| Timeout | 408 | Yes | Retry with backoff |
| Not Found | 404 | No | Check endpoint |
| Bad Request | 400 | No | Check request payload |
| Insufficient Funds | 402 | No | Prompt user |
| Circuit Open | 503 | No | Use fallback provider |

## 9. Environment Configuration

```yaml
# config/integrations.yml
integrations:
  m-floos:
    production:
      baseUrl: https://api.m-floos.com/v1
      timeout: 30000
      certPath: /vault/secrets/mfloos/client.crt
      keyPath: /vault/secrets/mfloos/client.key
    sandbox:
      baseUrl: https://sandbox.m-floos.com/v1
      timeout: 30000

  onecash:
    production:
      baseUrl: https://api.onecash.ye/v2
      timeout: 30000
      certPath: /vault/secrets/onecash/client.crt
      keyPath: /vault/secrets/onecash/client.key
    sandbox:
      baseUrl: https://sandbox.onecash.ye/v2
      timeout: 30000

  telesom:
    production:
      baseUrl: https://sms.telesom.com/api/v1
      timeout: 10000
    sandbox:
      baseUrl: https://sandbox.sms.telesom.com/api/v1
      timeout: 10000

  sabafon:
    production:
      baseUrl: https://api.sabafon.com/sms/v1
      timeout: 10000
    sandbox:
      baseUrl: https://sandbox.sabafon.com/sms/v1
      timeout: 10000

  whatsapp:
    production:
      baseUrl: https://graph.facebook.com/v17.0
      timeout: 15000

  smsa:
    production:
      baseUrl: https://api.smsa.net/v2
      timeout: 15000
    sandbox:
      baseUrl: https://sandbox.smsa.net/v2
      timeout: 15000

  aramex:
    production:
      baseUrl: https://api.aramex.com/v2
      timeout: 15000
    sandbox:
      baseUrl: https://sandbox.aramex.com/v2
      timeout: 15000

  dhl:
    production:
      baseUrl: https://api.dhl.com/mydhlapi/v2
      timeout: 20000
    sandbox:
      baseUrl: https://api-sandbox.dhl.com/mydhlapi/v2
      timeout: 20000
```

## 10. Related Files

| File | Description |
|------|-------------|
| `integration-overview.md` | Integration architecture overview |
| `payment-providers.md` | Payment provider details |
| `delivery-providers.md` | Delivery provider details |
| `sms-providers.md` | SMS provider details |
| `notification-services.md` | Notification channel details |
| `analytics-integrations.md` | Analytics integrations |
| `03-system-analysis/integration-points.md` | Integration point catalog |

---

*Document Version: 1.0 | Last Updated: 2026-09-13 | Classification: Confidential*
