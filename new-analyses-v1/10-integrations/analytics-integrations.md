# Analytics Integrations — YemenMart

## 1. Overview

YemenMart uses Google Analytics 4 for marketing analytics and a custom analytics engine for business intelligence. All analytics data flows through an event bus to decouple tracking from business operations.

```
┌─────────────────────────────────────────────────────────────────┐
│                    ANALYTICS ARCHITECTURE                        │
│                                                                 │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐                  │
│  │ Frontend │───>│ GA4      │───>│ Google   │                  │
│  │ Events   │    │ Tag      │    │ Analytics│                  │
│  └──────────┘    └──────────┘    └──────────┘                  │
│                                                                 │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐                  │
│  │ Backend  │───>│ Event    │───>│ Custom   │                  │
│  │ Events   │    │ Bus      │    │ Engine   │                  │
│  └──────────┘    └──────────┘    └────┬─────┘                  │
│                                       │                         │
│                  ┌────────────────────┼────────────────┐       │
│                  │                    │                │       │
│             ┌────▼─────┐        ┌────▼─────┐    ┌────▼────┐  │
│             │PostgreSQL│        │ Elastic- │    │  Redis  │  │
│             │          │        │ search   │    │(Cache)  │  │
│             └──────────┘        └──────────┘    └─────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

## 2. Google Analytics 4

### Configuration

| Property | Value |
|----------|-------|
| **Type** | Web Analytics Platform |
| **Measurement ID** | `G-XXXXXXXXXX` |
| **Data Stream** | Web (Customer Storefront) |
| **Enhanced E-commerce** | Enabled |
| **Custom Dimensions** | User role, Vendor ID, Order value range |
| **Data Retention** | 14 months |

### Events Tracked

#### E-commerce Events

| Event | Parameters | Trigger |
|-------|------------|---------|
| `view_item_list` | items, item_list_id, item_list_name | Product listing page view |
| `select_item` | items, item_list_id | Product click from listing |
| `view_item` | items, value, currency | Product detail page view |
| `add_to_cart` | items, value, currency | Add to cart action |
| `remove_from_cart` | items, value, currency | Remove from cart |
| `view_cart` | items, value, currency | Cart page view |
| `begin_checkout` | items, value, currency, coupon | Checkout initiated |
| `add_shipping_info` | items, value, currency, shipping_tier | Shipping info entered |
| `add_payment_info` | items, value, currency, payment_type | Payment info entered |
| `purchase` | transaction_id, value, tax, shipping, items | Order completed |
| `refund` | transaction_id, value, items | Refund processed |

#### User Events

| Event | Parameters | Trigger |
|-------|------------|---------|
| `sign_up` | method | User registration |
| `login` | method | User login |
| `search` | search_term, filters | Product search |
| `share` | method, content_type | Share product/store |

### Implementation

```typescript
// packages/analytics-module/src/ga4.service.ts
import { injectable, inject } from 'inversify';
import { gtag } from 'ga-gtag';

interface GA4Config {
  measurementId: string;
  customDimensions: CustomDimension[];
}

interface EcommerceItem {
  item_id: string;
  item_name: string;
  affiliation: string;
  coupon: string;
  discount: number;
  item_brand: string;
  item_category: string;
  item_category2: string;
  item_list_id: string;
  item_list_name: string;
  item_variant: string;
  price: number;
  quantity: number;
}

@injectable()
export class GA4Service {
  private config: GA4Config;

  constructor(@inject('GA4Config') config: GA4Config) {
    this.config = config;
    this.init();
  }

  private init() {
    gtag('js', new Date());
    gtag('config', this.config.measurementId, {
      custom_map: {
        dimension1: 'user_role',
        dimension2: 'vendor_id',
        dimension3: 'order_value_range',
      },
    });
  }

  trackViewItem(items: EcommerceItem[]) {
    gtag('event', 'view_item', {
      items,
      currency: 'YER',
    });
  }

  trackAddToCart(items: EcommerceItem[], value: number) {
    gtag('event', 'add_to_cart', {
      items,
      value,
      currency: 'YER',
    });
  }

  trackPurchase(transactionId: string, items: EcommerceItem[], value: number, tax: number, shipping: number) {
    gtag('event', 'purchase', {
      transaction_id: transactionId,
      value,
      tax,
      shipping,
      items,
      currency: 'YER',
    });
  }

  trackSearch(searchTerm: string, filters?: Record<string, string>) {
    gtag('event', 'search', {
      search_term: searchTerm,
      ...filters,
    });
  }

  setUserProperties(userId: string, role: string, vendorId?: string) {
    gtag('set', 'user_properties', {
      user_id: userId,
      user_role: role,
      vendor_id: vendorId,
    });
  }
}
```

## 3. Custom Analytics Engine

### Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                   CUSTOM ANALYTICS ENGINE                        │
│                                                                 │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐                  │
│  │ Domain   │───>│ Event    │───>│ Material-│                  │
│  │ Events   │    │ Store    │    │ ization  │                  │
│  └──────────┘    └──────────┘    └────┬─────┘                  │
│                                       │                         │
│                  ┌────────────────────┼────────────────┐       │
│                  │                    │                │       │
│             ┌────▼─────┐        ┌────▼─────┐    ┌────▼────┐  │
│             │  Time-   │        │ Elastic- │    │  Redis  │  │
│             │  Series  │        │ search   │    │(Dashbrd)│  │
│             │  Store   │        │          │    │         │  │
│             └──────────┘        └──────────┘    └─────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

### Event Store

```typescript
// packages/analytics-module/src/event-store.service.ts
import { injectable, inject } from 'inversify';

interface AnalyticsEvent {
  id: string;
  type: string;
  aggregateId: string;
  aggregateType: string;
  data: Record<string, any>;
  metadata: {
    userId?: string;
    role?: string;
    ip?: string;
    userAgent?: string;
    timestamp: Date;
  };
  version: number;
}

@injectable()
export class EventStore {
  constructor(
    @inject('PrismaClient') private prisma: PrismaClient,
    @inject('ElasticsearchClient') private elastic: Elasticsearch,
  ) {}

  async append(event: AnalyticsEvent): Promise<void> {
    // Store in PostgreSQL for durability
    await this.prisma.analyticsEvent.create({
      data: {
        id: event.id,
        type: event.type,
        aggregateId: event.aggregateId,
        aggregateType: event.aggregateType,
        data: event.data,
        metadata: event.metadata,
        version: event.version,
      },
    });

    // Index in Elasticsearch for searching and aggregation
    await this.elastic.index({
      index: 'analytics-events',
      id: event.id,
      body: {
        ...event,
        '@timestamp': event.metadata.timestamp,
      },
    });
  }

  async query(query: AnalyticsQuery): Promise<AnalyticsResult> {
    // Use Elasticsearch for complex aggregations
    const result = await this.elastic.search({
      index: 'analytics-events',
      body: {
        query: this.buildQuery(query),
        aggs: this.buildAggregations(query),
        size: query.limit || 0,
      },
    });

    return {
      aggregations: result.aggregations,
      total: result.hits.total.value,
      data: result.hits.hits.map(hit => hit._source),
    };
  }

  private buildQuery(query: AnalyticsQuery) {
    const must: any[] = [];

    if (query.types?.length) {
      must.push({ terms: { type: query.types } });
    }

    if (query.dateRange) {
      must.push({
        range: {
          '@timestamp': {
            gte: query.dateRange.start,
            lte: query.dateRange.end,
          },
        },
      });
    }

    if (query.aggregateType) {
      must.push({ term: { aggregateType: query.aggregateType } });
    }

    return { bool: { must } };
  }

  private buildAggregations(query: AnalyticsQuery) {
    if (!query.aggregations) return {};

    const aggs: Record<string, any> = {};

    for (const agg of query.aggregations) {
      switch (agg.type) {
        case 'date_histogram':
          aggs[agg.name] = {
            date_histogram: {
              field: '@timestamp',
              calendar_interval: agg.interval || 'day',
            },
          };
          break;
        case 'terms':
          aggs[agg.name] = {
            terms: { field: agg.field, size: agg.size || 10 },
          };
          break;
        case 'sum':
          aggs[agg.name] = {
            sum: { field: agg.field },
          };
          break;
        case 'avg':
          aggs[agg.name] = {
            avg: { field: agg.field },
          };
          break;
        case 'cardinality':
          aggs[agg.name] = {
            cardinality: { field: agg.field },
          };
          break;
      }
    }

    return aggs;
  }
}
```

### Dashboard Service

```typescript
// packages/analytics-module/src/dashboard.service.ts
@injectable()
export class DashboardService {
  constructor(
    @inject('EventStore') private eventStore: EventStore,
    @inject('RedisClient') private redis: Redis,
  ) {}

  async getRealtimeDashboard(): Promise<RealtimeDashboard> {
    const cacheKey = 'dashboard:realtime';
    const cached = await this.redis.get(cacheKey);
    if (cached) return JSON.parse(cached);

    const now = new Date();
    const oneHourAgo = new Date(now.getTime() - 60 * 60 * 1000);

    const [activeUsers, orders, revenue] = await Promise.all([
      this.getActiveUsers(oneHourAgo, now),
      this.getOrderCount(oneHourAgo, now),
      this.getRevenue(oneHourAgo, now),
    ]);

    const dashboard = { activeUsers, orders, revenue, timestamp: now };
    await this.redis.setex(cacheKey, 60, JSON.stringify(dashboard)); // Cache for 1 minute

    return dashboard;
  }

  async getVendorDashboard(vendorId: string, dateRange: DateRange): Promise<VendorDashboard> {
    const [sales, orders, topProducts, rating] = await Promise.all([
      this.getVendorSales(vendorId, dateRange),
      this.getVendorOrders(vendorId, dateRange),
      this.getVendorTopProducts(vendorId, dateRange),
      this.getVendorRating(vendorId),
    ]);

    return { sales, orders, topProducts, rating, dateRange };
  }

  async getAdminDashboard(dateRange: DateRange): Promise<AdminDashboard> {
    const [gmv, users, vendors, orders, topVendors, categoryBreakdown] = await Promise.all([
      this.getGMV(dateRange),
      this.getUserMetrics(dateRange),
      this.getVendorMetrics(dateRange),
      this.getOrderMetrics(dateRange),
      this.getTopVendors(dateRange),
      this.getCategoryBreakdown(dateRange),
    ]);

    return { gmv, users, vendors, orders, topVendors, categoryBreakdown, dateRange };
  }

  private async getGMV(dateRange: DateRange): Promise<GMVMetrics> {
    const result = await this.eventStore.query({
      types: ['order.completed'],
      dateRange,
      aggregations: [
        { name: 'total_gmv', type: 'sum', field: 'data.total' },
        { name: 'avg_order_value', type: 'avg', field: 'data.total' },
        { name: 'unique_customers', type: 'cardinality', field: 'metadata.userId' },
        { name: 'orders_by_day', type: 'date_histogram', field: '@timestamp', interval: 'day' },
      ],
    });

    return {
      totalGMV: result.aggregations.total_gmv.value,
      avgOrderValue: result.aggregations.avg_order_value.value,
      uniqueCustomers: result.aggregations.unique_customers.value,
      dailyBreakdown: result.aggregations.orders_by_day.buckets,
    };
  }

  private async getVendorSales(vendorId: string, dateRange: DateRange): Promise<SalesMetrics> {
    const result = await this.eventStore.query({
      types: ['order.completed'],
      dateRange,
      aggregateType: 'vendor',
      aggregateId: vendorId,
      aggregations: [
        { name: 'total_sales', type: 'sum', field: 'data.sellerPayout' },
        { name: 'total_orders', type: 'cardinality', field: 'aggregateId' },
        { name: 'avg_order_value', type: 'avg', field: 'data.sellerPayout' },
        { name: 'sales_by_day', type: 'date_histogram', field: '@timestamp', interval: 'day' },
      ],
    });

    return {
      totalSales: result.aggregations.total_sales.value,
      totalOrders: result.aggregations.total_orders.value,
      avgOrderValue: result.aggregations.avg_order_value.value,
      dailyBreakdown: result.aggregations.sales_by_day.buckets,
    };
  }
}
```

## 4. Metrics Definitions

### Business Metrics

| Metric | Definition | Calculation |
|--------|------------|-------------|
| GMV | Gross Merchandise Value | Sum of all order totals |
| Revenue | Platform revenue | Sum of commissions + fees |
| AOV | Average Order Value | GMV / Order count |
| ARPU | Average Revenue Per User | Revenue / Active users |
| LTV | Customer Lifetime Value | AOV × Purchase frequency × Retention |
| CAC | Customer Acquisition Cost | Marketing spend / New customers |
| Retention Rate | % customers returning | (Returning / Total) × 100 |
| Churn Rate | % customers leaving | (Lost / Total) × 100 |

### Vendor Metrics

| Metric | Definition | Calculation |
|--------|------------|-------------|
| Vendor GMV | Vendor's total sales | Sum of vendor's order totals |
| Vendor Rating | Average product rating | Avg of all product reviews |
| Fill Rate | Orders fulfilled on time | (On-time / Total) × 100 |
| Return Rate | Orders returned | (Returns / Total) × 100 |
| Commission Paid | Total commission to platform | Sum of commission amounts |

### Platform Metrics

| Metric | Definition | Calculation |
|--------|------------|-------------|
| DAU | Daily Active Users | Unique users with activity |
| MAU | Monthly Active Users | Unique users with activity |
| Session Duration | Average session time | Sum / Session count |
| Bounce Rate | Single-page sessions | (Bounced / Total) × 100 |
| Conversion Rate | Orders / Visits | (Orders / Visits) × 100 |
| Search Success Rate | Searches leading to purchase | (Search purchases / Searches) × 100 |

## 5. Data Pipeline

```typescript
// packages/analytics-module/src/pipeline.service.ts
@injectable()
export class AnalyticsPipeline {
  constructor(
    @inject('EventBus') private events: EventBus,
    @inject('EventStore') private eventStore: EventStore,
    @inject('GA4Service') private ga4: GA4Service,
    @inject('Logger') private logger: ILogger,
  ) {
    this.setupConsumers();
  }

  private setupConsumers() {
    // Consume domain events and route to analytics
    this.events.on('order.completed', (event) => this.processOrderCompleted(event));
    this.events.on('product.viewed', (event) => this.processProductViewed(event));
    this.events.on('user.registered', (event) => this.processUserRegistered(event));
    this.events.on('search.executed', (event) => this.processSearch(event));
  }

  private async processOrderCompleted(event: OrderCompletedEvent) {
    // Store in event store
    await this.eventStore.append({
      id: uuid(),
      type: 'order.completed',
      aggregateId: event.orderId,
      aggregateType: 'order',
      data: {
        total: event.total,
        items: event.items,
        vendorId: event.vendorId,
        paymentMethod: event.paymentMethod,
      },
      metadata: {
        userId: event.userId,
        role: 'customer',
        timestamp: new Date(),
      },
      version: 1,
    });

    // Send to GA4
    this.ga4.trackPurchase(event.orderId, event.items.map(item => ({
      item_id: item.variantId,
      item_name: item.name,
      affiliation: event.storeName,
      price: item.price,
      quantity: item.quantity,
    })), event.total, event.tax, event.shipping);
  }

  private async processProductViewed(event: ProductViewedEvent) {
    await this.eventStore.append({
      id: uuid(),
      type: 'product.viewed',
      aggregateId: event.productId,
      aggregateType: 'product',
      data: {
        variantId: event.variantId,
        storeId: event.storeId,
        price: event.price,
      },
      metadata: {
        userId: event.userId,
        timestamp: new Date(),
      },
      version: 1,
    });
  }
}
```

## 6. Reporting API

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| GET | /analytics/realtime | Real-time dashboard | Admin |
| GET | /analytics/gmv | GMV metrics | Admin |
| GET | /analytics/vendors/:id | Vendor performance | Admin/Vendor |
| GET | /analytics/products | Product analytics | Admin/Vendor |
| GET | /analytics/orders | Order analytics | Admin |
| GET | /analytics/users | User analytics | Admin |
| GET | /analytics/search | Search analytics | Admin |
| GET | /analytics/reports | Generate reports | Admin |

## 7. Related Files

| File | Description |
|------|-------------|
| `integration-overview.md` | Integration architecture overview |
| `third-party-apis.md` | Third-party API documentation |
| `03-system-analysis/integration-points.md` | Integration point catalog |

---

*Document Version: 1.0 | Last Updated: 2026-09-13 | Classification: Confidential*
