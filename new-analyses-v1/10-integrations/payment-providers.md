# Payment Providers — YemenMart

## 1. Overview

YemenMart operates a **wallet-only payment system**. No credit/debit card processors, no BNPL, no installments. All monetary operations flow through the internal wallet system, funded via mobile wallet top-up (m-Floos, OneCash) or bank transfer.

```
┌─────────────────────────────────────────────────────────────────┐
│                      PAYMENT FLOW                               │
│                                                                 │
│  Customer ──[Top-Up]──> Wallet ──[Pay]──> Escrow ──[Release]──>│
│                          │                                      │
│  ┌──────────┐            │    ┌──────────┐                      │
│  │  m-Floos │◄───────────┤    │  OneCash │                      │
│  └──────────┘            │    └──────────┘                      │
│                          │                                      │
│  ┌──────────┐            │                                      │
│  │   Bank   │◄───────────┤   (Manual Verification)              │
│  │ Transfer │            │                                      │
│  └──────────┘            │                                      │
└─────────────────────────────────────────────────────────────────┘
```

## 2. m-Floos Integration

### Connection Details

| Property | Value |
|----------|-------|
| **Type** | Mobile Wallet Payment Gateway |
| **Protocol** | REST API over HTTPS |
| **Authentication** | mTLS (client certificate) + API key |
| **Base URL (Production)** | `https://api.m-floos.com/v1` |
| **Base URL (Sandbox)** | `https://sandbox.m-floos.com/v1` |
| **Integrating Block** | B05 Payment |
| **Timeout** | 30s (charge), 60s (refund) |
| **Circuit Breaker** | Open after 5 consecutive failures; half-open after 60s |

### API Endpoints

| Endpoint | Method | Purpose | Idempotent |
|----------|--------|---------|------------|
| `/charge` | POST | Charge customer wallet | Yes |
| `/refund` | POST | Refund to customer wallet | Yes |
| `/balance` | GET | Check wallet balance | No |
| `/transfer` | POST | Transfer between wallets | Yes |
| `/status/{ref}` | GET | Check transaction status | No |

### Request/Response Schemas

#### Charge Request

```json
{
  "idempotency_key": "uuid-v4",
  "amount": 15000,
  "currency": "YER",
  "phone": "+967771234567",
  "description": "YemenMart Wallet Top-Up",
  "metadata": {
    "order_id": "YM-ORD-2026-001",
    "user_id": "usr_uuid"
  }
}
```

#### Charge Response (Success)

```json
{
  "status": "success",
  "transaction_id": "mfl_txn_abc123",
  "amount": 15000,
  "currency": "YER",
  "phone": "+967771234567",
  "timestamp": "2026-09-13T14:30:00Z"
}
```

#### Charge Response (Failure)

```json
{
  "status": "failed",
  "error_code": "INSUFFICIENT_BALANCE",
  "message": "Insufficient balance in m-Floos wallet",
  "transaction_id": null
}
```

### Adapter Implementation

```typescript
// packages/payment-module/src/adapters/mfloos.adapter.ts
import { injectable, inject } from 'inversify';
import { IPaymentAdapter, TopUpRequest, TopUpResult, RefundRequest, RefundResult } from '../interfaces/payment-adapter.interface';

interface MFloosConfig {
  baseUrl: string;
  apiKey: string;
  clientCert: Buffer;
  clientKey: Buffer;
  timeout: number;
}

@injectable()
export class MFloosAdapter implements IPaymentAdapter {
  name = 'm-floos';
  private config: MFloosConfig;
  private httpClient: HttpClient;

  constructor(@inject('MFloosConfig') config: MFloosConfig) {
    this.config = config;
    this.httpClient = new HttpClient({
      baseURL: config.baseUrl,
      timeout: config.timeout,
      httpsAgent: new https.Agent({
        cert: config.clientCert,
        key: config.clientKey,
        rejectUnauthorized: true,
      }),
      headers: {
        'X-API-Key': config.apiKey,
        'Content-Type': 'application/json',
      },
    });
  }

  async topUp(request: TopUpRequest): Promise<TopUpResult> {
    try {
      const response = await this.httpClient.post('/charge', {
        idempotency_key: request.idempotencyKey,
        amount: request.amount.value,
        currency: request.amount.currency,
        phone: request.phone,
        description: request.description,
        metadata: request.metadata,
      });

      return {
        success: response.data.status === 'success',
        transactionId: response.data.transaction_id,
        amount: new Money(response.data.amount, response.data.currency),
        providerReference: response.data.transaction_id,
        raw: response.data,
      };
    } catch (error) {
      if (error.response?.status === 402) {
        return { success: false, error: 'INSUFFICIENT_BALANCE', raw: error.response.data };
      }
      throw new PaymentProviderError('m-floos', error);
    }
  }

  async refund(request: RefundRequest): Promise<RefundResult> {
    try {
      const response = await this.httpClient.post('/refund', {
        idempotency_key: request.idempotencyKey,
        original_transaction_id: request.originalTransactionId,
        amount: request.amount.value,
        currency: request.amount.currency,
        reason: request.reason,
      });

      return {
        success: response.data.status === 'success',
        refundId: response.data.refund_id,
        amount: new Money(response.data.amount, response.data.currency),
        raw: response.data,
      };
    } catch (error) {
      throw new PaymentProviderError('m-floos', error);
    }
  }

  async getBalance(phone: string): Promise<Money> {
    const response = await this.httpClient.get(`/balance?phone=${phone}`);
    return new Money(response.data.balance, response.data.currency);
  }

  async getTransactionStatus(transactionId: string): Promise<TransactionStatus> {
    const response = await this.httpClient.get(`/status/${transactionId}`);
    return {
      status: response.data.status,
      amount: new Money(response.data.amount, response.data.currency),
      timestamp: response.data.timestamp,
    };
  }
}
```

### Error Handling

| Error Code | HTTP Status | Description | Action |
|------------|-------------|-------------|--------|
| `INSUFFICIENT_BALANCE` | 402 | Wallet has insufficient funds | Prompt user to top-up |
| `INVALID_PHONE` | 400 | Phone number format invalid | Validate input |
| `PROVIDER_UNAVAILABLE` | 503 | m-Floos service down | Retry, then failover to OneCash |
| `TIMEOUT` | 408 | Request timed out | Retry with backoff |
| `DUPLICATE_REQUEST` | 409 | Idempotency key already used | Return cached result |

## 3. OneCash Integration

### Connection Details

| Property | Value |
|----------|-------|
| **Type** | Mobile Wallet Payment Gateway |
| **Protocol** | REST API over HTTPS |
| **Authentication** | mTLS (client certificate) + API key |
| **Base URL (Production)** | `https://api.onecash.ye/v2` |
| **Base URL (Sandbox)** | `https://sandbox.onecash.ye/v2` |
| **Integrating Block** | B05 Payment |
| **Timeout** | 30s (charge), 60s (refund) |
| **Circuit Breaker** | Open after 5 consecutive failures; half-open after 60s |

### API Endpoints

| Endpoint | Method | Purpose | Idempotent |
|----------|--------|---------|------------|
| `/payment/charge` | POST | Charge customer wallet | Yes |
| `/payment/refund` | POST | Refund to customer wallet | Yes |
| `/wallet/balance` | GET | Check wallet balance | No |
| `/payment/status/{id}` | GET | Check transaction status | No |

### Request/Response Schemas

#### Charge Request

```json
{
  "idempotency_key": "uuid-v4",
  "amount": 15000,
  "currency": "YER",
  "phone": "+967771234567",
  "description": "YemenMart Wallet Top-Up",
  "reference": "YM-TOPUP-2026-001",
  "metadata": {
    "user_id": "usr_uuid"
  }
}
```

#### Charge Response (Success)

```json
{
  "status": "completed",
  "id": "oc_pay_xyz789",
  "amount": 15000,
  "currency": "YER",
  "phone": "+967771234567",
  "created_at": "2026-09-13T14:30:00Z"
}
```

### Adapter Implementation

```typescript
// packages/payment-module/src/adapters/onecash.adapter.ts
import { injectable, inject } from 'inversify';
import { IPaymentAdapter, TopUpRequest, TopUpResult, RefundRequest, RefundResult } from '../interfaces/payment-adapter.interface';

interface OneCashConfig {
  baseUrl: string;
  apiKey: string;
  clientCert: Buffer;
  clientKey: Buffer;
  timeout: number;
}

@injectable()
export class OneCashAdapter implements IPaymentAdapter {
  name = 'onecash';
  private config: OneCashConfig;
  private httpClient: HttpClient;

  constructor(@inject('OneCashConfig') config: OneCashConfig) {
    this.config = config;
    this.httpClient = new HttpClient({
      baseURL: config.baseUrl,
      timeout: config.timeout,
      httpsAgent: new https.Agent({
        cert: config.clientCert,
        key: config.clientKey,
        rejectUnauthorized: true,
      }),
      headers: {
        'Authorization': `Bearer ${config.apiKey}`,
        'Content-Type': 'application/json',
      },
    });
  }

  async topUp(request: TopUpRequest): Promise<TopUpResult> {
    try {
      const response = await this.httpClient.post('/payment/charge', {
        idempotency_key: request.idempotencyKey,
        amount: request.amount.value,
        currency: request.amount.currency,
        phone: request.phone,
        description: request.description,
        reference: request.metadata?.order_id,
        metadata: request.metadata,
      });

      return {
        success: response.data.status === 'completed',
        transactionId: response.data.id,
        amount: new Money(response.data.amount, response.data.currency),
        providerReference: response.data.id,
        raw: response.data,
      };
    } catch (error) {
      if (error.response?.status === 402) {
        return { success: false, error: 'INSUFFICIENT_BALANCE', raw: error.response.data };
      }
      throw new PaymentProviderError('onecash', error);
    }
  }

  async refund(request: RefundRequest): Promise<RefundResult> {
    try {
      const response = await this.httpClient.post('/payment/refund', {
        idempotency_key: request.idempotencyKey,
        payment_id: request.originalTransactionId,
        amount: request.amount.value,
        currency: request.amount.currency,
        reason: request.reason,
      });

      return {
        success: response.data.status === 'completed',
        refundId: response.data.id,
        amount: new Money(response.data.amount, response.data.currency),
        raw: response.data,
      };
    } catch (error) {
      throw new PaymentProviderError('onecash', error);
    }
  }

  async getBalance(phone: string): Promise<Money> {
    const response = await this.httpClient.get(`/wallet/balance?phone=${phone}`);
    return new Money(response.data.balance, response.data.currency);
  }

  async getTransactionStatus(transactionId: string): Promise<TransactionStatus> {
    const response = await this.httpClient.get(`/payment/status/${transactionId}`);
    return {
      status: response.data.status,
      amount: new Money(response.data.amount, response.data.currency),
      timestamp: response.data.created_at,
    };
  }
}
```

### Error Handling

| Error Code | HTTP Status | Description | Action |
|------------|-------------|-------------|--------|
| `INSUFFICIENT_FUNDS` | 402 | Wallet has insufficient funds | Prompt user to top-up |
| `INVALID_ACCOUNT` | 400 | Phone number not registered | Validate input |
| `SERVICE_UNAVAILABLE` | 503 | OneCash service down | Retry, then failover to m-Floos |
| `REQUEST_TIMEOUT` | 408 | Request timed out | Retry with backoff |
| `DUPLICATE_TRANSACTION` | 409 | Idempotency key already used | Return cached result |

## 4. Bank Transfer Integration

### Flow

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│ Customer │───>│ Generate │───>│ Transfer │───>│  Admin   │
│ Requests │    │ Reference│    │   Bank   │    │ Verifies │
│ Top-Up   │    │   Code   │    │          │    │ Transfer │
└──────────┘    └──────────┘    └──────────┘    └────┬─────┘
                                                     │
                                                     ▼
                                                ┌──────────┐
                                                │  Wallet  │
                                                │ Credited │
                                                └──────────┘
```

### Reference Code Format

```
YM-{YYYYMMDD}-{RANDOM6}
Example: YM-20260913-A3K9F2
```

### Webhook (Optional)

For banks supporting webhooks, the system verifies:

```typescript
function verifyBankWebhook(payload: Buffer, signature: string, secret: string): boolean {
  const expected = crypto
    .createHmac('sha256', secret)
    .update(payload)
    .digest('hex');
  
  return crypto.timingSafeEqual(
    Buffer.from(signature),
    Buffer.from(expected)
  );
}
```

### Manual Verification

For banks without webhook support:

1. Admin receives transfer notification via SMS/email
2. Admin cross-references reference code in admin panel
3. Admin approves transfer → wallet credited
4. Customer receives confirmation notification

## 5. Payment Adapter Interface

```typescript
// packages/payment-module/src/interfaces/payment-adapter.interface.ts

export interface IPaymentAdapter {
  name: string;
  topUp(request: TopUpRequest): Promise<TopUpResult>;
  refund(request: RefundRequest): Promise<RefundResult>;
  getBalance(phone: string): Promise<Money>;
  getTransactionStatus(transactionId: string): Promise<TransactionStatus>;
}

export interface TopUpRequest {
  idempotencyKey: string;
  amount: Money;
  phone: string;
  description: string;
  metadata?: Record<string, any>;
}

export interface TopUpResult {
  success: boolean;
  transactionId?: string;
  amount?: Money;
  providerReference?: string;
  error?: string;
  raw?: any;
}

export interface RefundRequest {
  idempotencyKey: string;
  originalTransactionId: string;
  amount: Money;
  reason: string;
}

export interface RefundResult {
  success: boolean;
  refundId?: string;
  amount?: Money;
  error?: string;
  raw?: any;
}

export interface TransactionStatus {
  status: string;
  amount: Money;
  timestamp: string;
}
```

## 6. Payment Provider Router

```typescript
// packages/payment-module/src/payment-provider.router.ts
import { injectable, inject } from 'inversify';

@injectable()
export class PaymentProviderRouter {
  private providers: Map<string, IPaymentAdapter>;

  constructor(
    @inject('MFloosAdapter') private mfloos: MFloosAdapter,
    @inject('OneCashAdapter') private onecash: OneCashAdapter,
    @inject('CircuitBreaker') private circuitBreaker: CircuitBreaker,
    @inject('Logger') private logger: ILogger,
  ) {
    this.providers = new Map([
      ['m-floos', mfloos],
      ['onecash', onecash],
    ]);
  }

  async topUp(provider: string, request: TopUpRequest): Promise<TopUpResult> {
    const adapter = this.providers.get(provider);
    if (!adapter) throw new UnsupportedPaymentProviderError(provider);

    try {
      return await this.circuitBreaker.execute(provider, () => adapter.topUp(request));
    } catch (error) {
      this.logger.error(`Payment provider ${provider} failed`, { error, request });
      
      // Failover to secondary provider
      const fallback = this.getFallbackProvider(provider);
      if (fallback) {
        this.logger.info(`Falling back to ${fallback.name}`);
        return await fallback.topUp(request);
      }
      
      throw error;
    }
  }

  private getFallbackProvider(failed: string): IPaymentAdapter | null {
    const fallbacks: Record<string, string> = {
      'm-floos': 'onecash',
      'onecash': 'm-floos',
    };
    const fallbackName = fallbacks[failed];
    return fallbackName ? this.providers.get(fallbackName) || null : null;
  }
}
```

## 7. Webhook Handlers

### m-Floos Webhook

```typescript
app.post('/webhooks/m-floos', express.raw({ type: 'application/json' }), async (req, res) => {
  const signature = req.headers['x-mfloos-signature'] as string;
  
  if (!verifyWebhookSignature(req.body, signature, process.env.MFLOOS_WEBHOOK_SECRET)) {
    logger.warn('Invalid m-Floos webhook signature', { ip: req.ip });
    return res.status(401).json({ error: 'Invalid signature' });
  }

  const event = JSON.parse(req.body);
  const eventId = event.event_id;

  if (await isWebhookProcessed(eventId)) {
    return res.status(200).json({ received: true });
  }

  switch (event.type) {
    case 'charge.completed':
      await handleTopUpCompleted(event.data);
      break;
    case 'charge.failed':
      await handleTopUpFailed(event.data);
      break;
    case 'refund.completed':
      await handleRefundCompleted(event.data);
      break;
  }

  await markWebhookProcessed(eventId);
  return res.status(200).json({ received: true });
});
```

### OneCash Webhook

```typescript
app.post('/webhooks/onecash', express.raw({ type: 'application/json' }), async (req, res) => {
  const signature = req.headers['x-onecash-signature'] as string;
  
  if (!verifyWebhookSignature(req.body, signature, process.env.ONECASH_WEBHOOK_SECRET)) {
    logger.warn('Invalid OneCash webhook signature', { ip: req.ip });
    return res.status(401).json({ error: 'Invalid signature' });
  }

  const event = JSON.parse(req.body);
  const eventId = event.id;

  if (await isWebhookProcessed(eventId)) {
    return res.status(200).json({ received: true });
  }

  switch (event.type) {
    case 'payment.completed':
      await handleTopUpCompleted(event.data);
      break;
    case 'payment.failed':
      await handleTopUpFailed(event.data);
      break;
    case 'refund.completed':
      await handleRefundCompleted(event.data);
      break;
  }

  await markWebhookProcessed(eventId);
  return res.status(200).json({ received: true });
});
```

## 8. Security Requirements

| Requirement | Implementation |
|-------------|----------------|
| mTLS | Client certificates for m-Floos and OneCash |
| Idempotency | All POST requests require idempotency key |
| Webhook Verification | HMAC-SHA256 signature validation |
| Rate Limiting | Per-user top-up limits enforced |
| Encryption | All PII encrypted at rest (AES-256-GCM) |
| Audit Trail | All payment events logged with full trace |
| IP Whitelisting | Webhook endpoints restricted to provider IPs |

## 9. Related Files

| File | Description |
|------|-------------|
| `integration-overview.md` | Integration architecture overview |
| `06-backend/payment-service.md` | Payment service implementation |
| `09-security/security-overview.md` | Payment security requirements |
| `03-system-analysis/integration-points.md` | Integration point catalog |

---

*Document Version: 1.0 | Last Updated: 2026-09-13 | Classification: Confidential*
