# Delivery Providers — YemenMart

## 1. Overview

YemenMart integrates with multiple delivery providers to serve all 17 Yemeni governates. Providers submit bids on orders, and the system assigns based on price, coverage, and performance metrics.

```
┌─────────────────────────────────────────────────────────────────┐
│                    DELIVERY MARKETPLACE                          │
│                                                                 │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐                  │
│  │  Order   │───>│  Bid     │───>│  Assign  │                  │
│  │ Created  │    │  Period  │    │  Winner  │                  │
│  └──────────┘    └──────────┘    └────┬─────┘                  │
│                                       │                         │
│                  ┌────────────────────┼────────────────┐       │
│                  │                    │                │       │
│             ┌────▼─────┐        ┌────▼─────┐    ┌────▼────┐  │
│             │  SMSA    │        │  Aramex  │    │   DHL   │  │
│             │  API     │        │  API     │    │   API   │  │
│             └────┬─────┘        └────┬─────┘    └────┬────┘  │
│                  │                    │                │       │
│                  └────────────────────┼────────────────┘       │
│                                       │                         │
│                                       ▼                         │
│                               ┌──────────┐                     │
│                               │ Delivery │                     │
│                               │ Tracking │                     │
│                               └──────────┘                     │
└─────────────────────────────────────────────────────────────────┘
```

## 2. Provider Registry

### SMSA

| Property | Value |
|----------|-------|
| **Type** | Logistics Provider |
| **Protocol** | REST API over HTTPS |
| **Authentication** | API key in `Authorization` header |
| **Base URL (Production)** | `https://api.smsa.net/v2` |
| **Base URL (Sandbox)** | `https://sandbox.smsa.net/v2` |
| **Integrating Block** | B07 Logistics |
| **Timeout** | 15s |
| **Retry** | 2 attempts, linear backoff |
| **Coverage** | All 17 governates |

### Aramex

| Property | Value |
|----------|-------|
| **Type** | Logistics Provider |
| **Protocol** | REST API over HTTPS |
| **Authentication** | API key + Client ID |
| **Base URL (Production)** | `https://api.aramex.com/v2` |
| **Base URL (Sandbox)** | `https://sandbox.aramex.com/v2` |
| **Integrating Block** | B07 Logistics |
| **Timeout** | 15s |
| **Retry** | 2 attempts, linear backoff |
| **Coverage** | Major cities (Sana'a, Aden, Taiz, Hudaydah) |

### DHL

| Property | Value |
|----------|-------|
| **Type** | International Logistics Provider |
| **Protocol** | REST API over HTTPS |
| **Authentication** | API key (OAuth2) |
| **Base URL (Production)** | `https://api.dhl.com/mydhlapi/v2` |
| **Base URL (Sandbox)** | `https://api-sandbox.dhl.com/mydhlapi/v2` |
| **Integrating Block** | B07 Logistics |
| **Timeout** | 20s |
| **Retry** | 2 attempts, linear backoff |
| **Coverage** | International shipments only |

## 3. API Endpoints

### SMSA Endpoints

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/shipments` | POST | Create shipment |
| `/shipments/{id}` | GET | Track shipment |
| `/shipments/{id}/cancel` | POST | Cancel shipment |
| `/rates` | POST | Get shipping rates |
| `/serviceable-areas` | GET | Check coverage |

### Aramex Endpoints

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/shipments` | POST | Create shipment |
| `/shipments/{id}/track` | GET | Track shipment |
| `/shipments/{id}` | DELETE | Cancel shipment |
| `/calculateshipment` | POST | Calculate rates |
| `/pickup` | POST | Schedule pickup |

### DHL Endpoints

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/shipments` | POST | Create shipment |
| `/shipments/{id}/tracking` | GET | Track shipment |
| `/shipments/{id}` | DELETE | Cancel shipment |
| `/rates` | POST | Get rates |
| `/locations` | GET | Find service points |

## 4. Delivery Provider Adapter Interface

```typescript
// packages/logistics-module/src/interfaces/delivery-adapter.interface.ts

export interface IDeliveryAdapter {
  name: string;
  createShipment(request: CreateShipmentRequest): Promise<ShipmentResult>;
  trackShipment(trackingNumber: string): Promise<TrackingResult>;
  cancelShipment(trackingNumber: string): Promise<CancelResult>;
  getRates(request: RateRequest): Promise<RateResult[]>;
  checkCoverage(address: Address): Promise<boolean>;
}

export interface CreateShipmentRequest {
  idempotencyKey: string;
  orderId: string;
  subOrderId: string;
  sender: Address;
  recipient: Address;
  package: PackageDetails;
  serviceType: 'standard' | 'express';
  codAmount?: Money;
  notes?: string;
}

export interface ShipmentResult {
  success: boolean;
  trackingNumber?: string;
  estimatedDelivery?: Date;
  cost?: Money;
  labelUrl?: string;
  error?: string;
}

export interface TrackingResult {
  status: DeliveryStatus;
  location?: string;
  timestamp: Date;
  history: TrackingEvent[];
  estimatedDelivery?: Date;
}

export interface RateRequest {
  sender: Address;
  recipient: Address;
  package: PackageDetails;
  serviceType: 'standard' | 'express';
}

export interface RateResult {
  provider: string;
  serviceType: string;
  cost: Money;
  estimatedDays: number;
}
```

## 5. Adapter Implementations

### SMSA Adapter

```typescript
@injectable()
export class SmsaAdapter implements IDeliveryAdapter {
  name = 'smsa';
  private httpClient: HttpClient;

  constructor(@inject('SmsaConfig') config: SmsaConfig) {
    this.httpClient = new HttpClient({
      baseURL: config.baseUrl,
      timeout: config.timeout,
      headers: {
        'Authorization': `Bearer ${config.apiKey}`,
        'Content-Type': 'application/json',
      },
    });
  }

  async createShipment(request: CreateShipmentRequest): Promise<ShipmentResult> {
    try {
      const response = await this.httpClient.post('/shipments', {
        idempotency_key: request.idempotencyKey,
        order_id: request.orderId,
        shipper: {
          name: request.sender.name,
          phone: request.sender.phone,
          address: this.formatAddress(request.sender),
        },
        consignee: {
          name: request.recipient.name,
          phone: request.recipient.phone,
          address: this.formatAddress(request.recipient),
        },
        parcels: [{
          weight: request.package.weightKg,
          dimensions: {
            length: request.package.lengthCm,
            width: request.package.widthCm,
            height: request.package.heightCm,
          },
        }],
        service_type: request.serviceType,
        cod_amount: request.codAmount?.value,
        cod_currency: request.codAmount?.currency,
        reference: request.subOrderId,
      });

      return {
        success: true,
        trackingNumber: response.data.tracking_number,
        estimatedDelivery: new Date(response.data.estimated_delivery),
        cost: new Money(response.data.cost, response.data.currency),
        labelUrl: response.data.label_url,
      };
    } catch (error) {
      return { success: false, error: error.message };
    }
  }

  async trackShipment(trackingNumber: string): Promise<TrackingResult> {
    const response = await this.httpClient.get(`/shipments/${trackingNumber}`);
    return this.mapTrackingStatus(response.data);
  }

  async cancelShipment(trackingNumber: string): Promise<CancelResult> {
    try {
      await this.httpClient.post(`/shipments/${trackingNumber}/cancel`);
      return { success: true };
    } catch (error) {
      return { success: false, error: error.message };
    }
  }

  async getRates(request: RateRequest): Promise<RateResult[]> {
    const response = await this.httpClient.post('/rates', {
      shipper: this.formatAddress(request.sender),
      consignee: this.formatAddress(request.recipient),
      weight: request.package.weightKg,
      service_type: request.serviceType,
    });

    return response.data.rates.map((rate: any) => ({
      provider: 'smsa',
      serviceType: rate.service_type,
      cost: new Money(rate.cost, rate.currency),
      estimatedDays: rate.estimated_days,
    }));
  }

  async checkCoverage(address: Address): Promise<boolean> {
    const response = await this.httpClient.get('/serviceable-areas');
    return response.data.areas.some((area: string) =>
      address.governate.includes(area)
    );
  }

  private mapTrackingStatus(data: any): TrackingResult {
    const statusMap: Record<string, DeliveryStatus> = {
      'pending': 'PENDING',
      'picked_up': 'PICKED_UP',
      'in_transit': 'IN_TRANSIT',
      'out_for_delivery': 'OUT_FOR_DELIVERY',
      'delivered': 'DELIVERED',
      'failed': 'FAILED',
      'returned': 'RETURNED',
    };

    return {
      status: statusMap[data.status] || 'UNKNOWN',
      location: data.current_location,
      timestamp: new Date(data.last_update),
      history: data.history?.map((h: any) => ({
        status: statusMap[h.status] || 'UNKNOWN',
        location: h.location,
        timestamp: new Date(h.timestamp),
        description: h.description,
      })) || [],
      estimatedDelivery: data.estimated_delivery ? new Date(data.estimated_delivery) : undefined,
    };
  }

  private formatAddress(address: Address): string {
    return `${address.street}, ${address.district}, ${address.city}, ${address.governate}, Yemen`;
  }
}
```

### Aramex Adapter

```typescript
@injectable()
export class AramexAdapter implements IDeliveryAdapter {
  name = 'aramex';
  private httpClient: HttpClient;

  constructor(@inject('AramexConfig') config: AramexConfig) {
    this.httpClient = new HttpClient({
      baseURL: config.baseUrl,
      timeout: config.timeout,
      headers: {
        'Authorization': `Bearer ${config.apiKey}`,
        'X-Client-ID': config.clientId,
        'Content-Type': 'application/json',
      },
    });
  }

  async createShipment(request: CreateShipmentRequest): Promise<ShipmentResult> {
    try {
      const response = await this.httpClient.post('/shipments', {
        idempotency_key: request.idempotencyKey,
        shipper: this.mapAddress(request.sender),
        consignee: this.mapAddress(request.recipient),
        details: [{
          number_of_pieces: 1,
          weight: request.package.weightKg,
          description: 'YemenMart Order',
        }],
        service_type: request.serviceType === 'express' ? 'P' : 'E',
        cod_amount: request.codAmount?.value,
        reference: request.subOrderId,
      });

      return {
        success: true,
        trackingNumber: response.data.tracking_number,
        estimatedDelivery: new Date(response.data.estimated_delivery),
        cost: new Money(response.data.cost, response.data.currency),
        labelUrl: response.data.label_url,
      };
    } catch (error) {
      return { success: false, error: error.message };
    }
  }

  async trackShipment(trackingNumber: string): Promise<TrackingResult> {
    const response = await this.httpClient.get(`/shipments/${trackingNumber}/track`);
    return this.mapTrackingStatus(response.data);
  }

  async cancelShipment(trackingNumber: string): Promise<CancelResult> {
    try {
      await this.httpClient.delete(`/shipments/${trackingNumber}`);
      return { success: true };
    } catch (error) {
      return { success: false, error: error.message };
    }
  }

  async getRates(request: RateRequest): Promise<RateResult[]> {
    const response = await this.httpClient.post('/calculateshipment', {
      shipper: this.mapAddress(request.sender),
      consignee: this.mapAddress(request.recipient),
      weight: request.package.weightKg,
      service_type: request.serviceType === 'express' ? 'P' : 'E',
    });

    return response.data.rates.map((rate: any) => ({
      provider: 'aramex',
      serviceType: rate.service_type,
      cost: new Money(rate.cost, rate.currency),
      estimatedDays: rate.estimated_days,
    }));
  }

  async checkCoverage(address: Address): Promise<boolean> {
    const response = await this.httpClient.get('/serviceable-areas');
    return response.data.areas.includes(address.city);
  }

  private mapAddress(address: Address) {
    return {
      name: address.name,
      phone: address.phone,
      line1: address.street,
      city: address.city,
      state: address.governate,
      country: 'YE',
      postal_code: address.postalCode,
    };
  }

  private mapTrackingStatus(data: any): TrackingResult {
    const statusMap: Record<string, DeliveryStatus> = {
      'SH': 'PENDING',
      'PU': 'PICKED_UP',
      'IT': 'IN_TRANSIT',
      'OD': 'OUT_FOR_DELIVERY',
      'DL': 'DELIVERED',
      'NF': 'FAILED',
      'RT': 'RETURNED',
    };

    return {
      status: statusMap[data.status_code] || 'UNKNOWN',
      location: data.current_location,
      timestamp: new Date(data.timestamp),
      history: data.scan_history?.map((h: any) => ({
        status: statusMap[h.status_code] || 'UNKNOWN',
        location: h.location,
        timestamp: new Date(h.timestamp),
        description: h.description,
      })) || [],
      estimatedDelivery: data.estimated_delivery ? new Date(data.estimated_delivery) : undefined,
    };
  }
}
```

### DHL Adapter

```typescript
@injectable()
export class DhlAdapter implements IDeliveryAdapter {
  name = 'dhl';
  private httpClient: HttpClient;

  constructor(@inject('DhlConfig') config: DhlConfig) {
    this.httpClient = new HttpClient({
      baseURL: config.baseUrl,
      timeout: config.timeout,
      headers: {
        'Authorization': `Bearer ${config.apiKey}`,
        'Content-Type': 'application/json',
      },
    });
  }

  async createShipment(request: CreateShipmentRequest): Promise<ShipmentResult> {
    try {
      const response = await this.httpClient.post('/shipments', {
        idempotency_key: request.idempotencyKey,
        plannedShippingDate: new Date().toISOString(),
        shipper: {
          shipperName: request.sender.name,
          registeredAccount: request.sender.dhlAccount,
          address: this.mapAddress(request.sender),
        },
        recipient: {
          recipientName: request.recipient.name,
          address: this.mapAddress(request.recipient),
        },
        packages: [{
          weight: request.package.weightKg,
          dimensions: {
            length: request.package.lengthCm,
            width: request.package.widthCm,
            height: request.package.heightCm,
          },
        }],
        serviceType: request.serviceType === 'express' ? 'P' : 'E',
        codAmount: request.codAmount?.value,
        codCurrency: request.codAmount?.currency,
      });

      return {
        success: true,
        trackingNumber: response.data.shipmentTrackingNumber,
        estimatedDelivery: new Date(response.data.estimatedDeliveryDate),
        cost: new Money(response.data.totalPrice, response.data.currency),
        labelUrl: response.data.labelUrl,
      };
    } catch (error) {
      return { success: false, error: error.message };
    }
  }

  async trackShipment(trackingNumber: string): Promise<TrackingResult> {
    const response = await this.httpClient.get(`/shipments/${trackingNumber}/tracking`);
    return this.mapTrackingStatus(response.data);
  }

  async cancelShipment(trackingNumber: string): Promise<CancelResult> {
    try {
      await this.httpClient.delete(`/shipments/${trackingNumber}`);
      return { success: true };
    } catch (error) {
      return { success: false, error: error.message };
    }
  }

  async getRates(request: RateRequest): Promise<RateResult[]> {
    const response = await this.httpClient.post('/rates', {
      plannedShippingDate: new Date().toISOString(),
      originCountryCode: 'YE',
      destinationCountryCode: 'YE',
      weight: request.package.weightKg,
      isCustomsDeclarable: false,
    });

    return response.data.rates.map((rate: any) => ({
      provider: 'dhl',
      serviceType: rate.serviceType,
      cost: new Money(rate.totalPrice, rate.currency),
      estimatedDays: rate.estimatedDeliveryDays,
    }));
  }

  async checkCoverage(address: Address): Promise<boolean> {
    const response = await this.httpClient.get('/locations', {
      params: { countryCode: 'YE', city: address.city },
    });
    return response.data.locations.length > 0;
  }

  private mapAddress(address: Address) {
    return {
      street: address.street,
      city: address.city,
      state: address.governate,
      countryCode: 'YE',
      postalCode: address.postalCode,
    };
  }

  private mapTrackingStatus(data: any): TrackingResult {
    const statusMap: Record<string, DeliveryStatus> = {
      'pre-transit': 'PENDING',
      'transit': 'IN_TRANSIT',
      'out-for-delivery': 'OUT_FOR_DELIVERY',
      'delivered': 'DELIVERED',
      'failure': 'FAILED',
      'returned': 'RETURNED',
    };

    return {
      status: statusMap[data.status?.toLowerCase()] || 'UNKNOWN',
      location: data.location,
      timestamp: new Date(data.timestamp),
      history: data.scan_events?.map((e: any) => ({
        status: statusMap[e.status?.toLowerCase()] || 'UNKNOWN',
        location: e.location,
        timestamp: new Date(e.timestamp),
        description: e.description,
      })) || [],
      estimatedDelivery: data.estimatedDeliveryDate ? new Date(data.estimatedDeliveryDate) : undefined,
    };
  }
}
```

## 6. Delivery Provider Router

```typescript
@injectable()
export class DeliveryProviderRouter {
  private providers: Map<string, IDeliveryAdapter>;

  constructor(
    @inject('SmsaAdapter') private smsa: SmsaAdapter,
    @inject('AramexAdapter') private aramex: AramexAdapter,
    @inject('DhlAdapter') private dhl: DhlAdapter,
    @inject('CircuitBreaker') private circuitBreaker: CircuitBreaker,
    @inject('Logger') private logger: ILogger,
  ) {
    this.providers = new Map([
      ['smsa', smsa],
      ['aramex', aramex],
      ['dhl', dhl],
    ]);
  }

  async selectProvider(address: Address): Promise<IDeliveryAdapter> {
    // Priority: SMSA (best coverage) > Aramex > DHL
    const priority = ['smsa', 'aramex', 'dhl'];
    
    for (const providerName of priority) {
      const adapter = this.providers.get(providerName)!;
      const isAvailable = await this.circuitBreaker.isAvailable(providerName);
      
      if (isAvailable) {
        const hasCoverage = await adapter.checkCoverage(address);
        if (hasCoverage) return adapter;
      }
    }

    throw new NoDeliveryProviderError(address.governate);
  }

  async getRatesForAllProviders(request: RateRequest): Promise<RateResult[]> {
    const allRates: RateResult[] = [];
    
    for (const [name, adapter] of this.providers) {
      try {
        const rates = await adapter.getRates(request);
        allRates.push(...rates);
      } catch (error) {
        this.logger.warn(`Failed to get rates from ${name}`, { error });
      }
    }

    return allRates.sort((a, b) => a.cost.value - b.cost.value);
  }
}
```

## 7. Webhook Handlers

### Delivery Status Webhook

```typescript
app.post('/webhooks/delivery/:provider', express.raw({ type: 'application/json' }), async (req, res) => {
  const provider = req.params.provider;
  const signature = req.headers['x-webhook-signature'] as string;

  if (!verifyWebhookSignature(req.body, signature, process.env[`${provider.toUpperCase()}_WEBHOOK_SECRET`])) {
    logger.warn(`Invalid ${provider} webhook signature`, { ip: req.ip });
    return res.status(401).json({ error: 'Invalid signature' });
  }

  const event = JSON.parse(req.body);
  const eventId = event.event_id;

  if (await isWebhookProcessed(eventId)) {
    return res.status(200).json({ received: true });
  }

  switch (event.type) {
    case 'shipment.picked_up':
      await handleDeliveryPickedUp(event.data);
      break;
    case 'shipment.in_transit':
      await handleDeliveryInTransit(event.data);
      break;
    case 'shipment.delivered':
      await handleDeliveryDelivered(event.data);
      break;
    case 'shipment.failed':
      await handleDeliveryFailed(event.data);
      break;
    case 'shipment.returned':
      await handleDeliveryReturned(event.data);
      break;
  }

  await markWebhookProcessed(eventId);
  return res.status(200).json({ received: true });
});
```

## 8. Coverage Matrix

| Governate | SMSA | Aramex | DHL | Notes |
|-----------|:----:|:------:|:---:|-------|
| Sana'a | Yes | Yes | Yes | Capital city |
| Aden | Yes | Yes | Yes | Commercial capital |
| Taiz | Yes | Yes | Partial | Major city |
| Hudaydah | Yes | Yes | Partial | Port city |
| Ibb | Yes | Yes | No | Central highlands |
| Dhamar | Yes | Partial | No | Central highlands |
| Hadramaut | Yes | Yes | Yes | Mukalla hub |
| Hadhramaut | Yes | Yes | Yes | Mukalla hub |
| Marib | Yes | Partial | No | Oil region |
| Al Jawf | Yes | No | No | Northern region |
| Saada | Yes | No | No | Northern region |
| Lahij | Yes | Partial | No | Southern region |
| Abyan | Yes | Partial | No | Southern region |
| Shabwah | Yes | Partial | No | Central-eastern |
| Al Mahrah | Yes | No | No | Easternmost |
| Socotra | No | No | No | Island (special handling) |
| Raymah | Yes | No | No | Small governate |

## 9. Rate Calculation

```typescript
export interface DeliveryPricing {
  baseRate: Money;
  perKgRate: Money;
  perKmRate: Money;
  codFee: Money;
  expressMultiplier: number;
}

const PRICING: Record<string, DeliveryPricing> = {
  smsa: {
    baseRate: new Money(500, 'YER'),
    perKgRate: new Money(100, 'YER'),
    perKmRate: new Money(5, 'YER'),
    codFee: new Money(200, 'YER'),
    expressMultiplier: 1.5,
  },
  aramex: {
    baseRate: new Money(600, 'YER'),
    perKgRate: new Money(120, 'YER'),
    perKmRate: new Money(6, 'YER'),
    codFee: new Money(250, 'YER'),
    expressMultiplier: 1.5,
  },
  dhl: {
    baseRate: new Money(1500, 'YER'),
    perKgRate: new Money(200, 'YER'),
    perKmRate: new Money(10, 'YER'),
    codFee: new Money(500, 'YER'),
    expressMultiplier: 2.0,
  },
};
```

## 10. Related Files

| File | Description |
|------|-------------|
| `integration-overview.md` | Integration architecture overview |
| `06-backend/notification-service.md` | Delivery notifications |
| `03-system-analysis/integration-points.md` | Integration point catalog |
| `01-business-analysis/business-rules.md` | Delivery business rules |

---

*Document Version: 1.0 | Last Updated: 2026-09-13 | Classification: Confidential*
