# Analytics & Reporting — YemenMart

## 1. Overview

YemenMart's analytics platform provides actionable insights across sales, customer behavior, inventory, and financial operations. This document defines dashboards, reports, data export capabilities, and the technical architecture powering analytics.

---

## 2. Analytics Architecture

### 2.1 Data Pipeline
```
┌──────────┐    ┌──────────┐    ┌──────────────┐    ┌──────────────┐
│Production│───→│  CDC/ETL │───→│  Analytics   │───→│  BI Tools    │
│ Database │    │ Pipeline │    │  Data Store  │    │  (Dashboards)│
└──────────┘    └──────────┘    └──────────────┘    └──────────────┘
                      │                                    │
                      ↓                                    ↓
              ┌──────────────┐                    ┌──────────────┐
              │  Data Lake   │                    │  Export API  │
              │  (Raw Data)  │                    │  (CSV/Excel) │
              └──────────────┘                    └──────────────┘
```

### 2.2 Technology Stack
| Component | Technology | Purpose |
|---|---|---|
| OLTP Database | PostgreSQL 15 | Transactional data |
| OLAP Database | PostgreSQL (analytics schema) | Analytical queries |
| Search | Elasticsearch 8.x | Product/search analytics |
| Caching | Redis 7.x | Real-time metrics cache |
| BI Platform | Metabase / Grafana | Dashboards & reports |
| ETL | Python + Airflow | Scheduled data pipelines |
| Export | Custom API | CSV/Excel generation |

---

## 3. Dashboard Catalog

### 3.1 Executive Dashboard
| Metric | Visualization | Refresh | Owner |
|---|---|---|---|
| Daily Revenue | Time series (line) | 15 min | CFO |
| Monthly GMV | Big number + trend | Hourly | CEO |
| Order Volume | Bar chart (daily) | 15 min | COO |
| Active Users | Gauge | Hourly | CMO |
| Conversion Rate | Funnel | Daily | CMO |
| Customer Satisfaction | Score card | Weekly | COO |

### 3.2 Sales Dashboard
| Metric | Visualization | Refresh | Filters |
|---|---|---|---|
| Revenue by Category | Stacked bar | Hourly | Date, Category |
| Revenue by Region | Heat map | Daily | Region, City |
| Average Order Value | Line + target | 15 min | Date range |
| Top Products | Ranked table | Hourly | Category, Period |
| Orders by Status | Donut chart | Real-time | Status |
| Hourly Sales Trend | Area chart | 5 min | Date |

### 3.3 Inventory Dashboard
| Metric | Visualization | Refresh | Filters |
|---|---|---|---|
| Stock Levels | Bar chart (top N) | 15 min | Category, Warehouse |
| Low Stock Alerts | Alert list | Real-time | Threshold |
| Stock Turnover | KPI card | Daily | Period |
| Reorder Recommendations | Table | Daily | Supplier |
| Dead Stock | Ranked table | Weekly | Age threshold |
| Warehouse Utilization | Gauge | Hourly | Warehouse |

### 3.4 Customer Dashboard
| Metric | Visualization | Refresh | Filters |
|---|---|---|---|
| New vs Returning | Stacked area | Daily | Period |
| Customer Lifetime Value | Distribution | Weekly | Cohort |
| Churn Risk Score | Heat map | Weekly | Segment |
| Geographic Distribution | Map | Daily | Region |
| Registration Funnel | Funnel | Daily | Channel |
| Support Ticket Volume | Bar chart | Hourly | Category |

### 3.5 Financial Dashboard
| Metric | Visualization | Refresh | Filters |
|---|---|---|---|
| Revenue vs Target | Bullet chart | Daily | Period |
| Payment Success Rate | Gauge | Real-time | Method |
| Refund Rate | Trend line | Daily | Category |
| Tax Collection | Big number | Daily | Tax type |
| Currency Breakdown | Pie chart | Daily | Currency |
| Outstanding Payments | Aging report | Daily | Age bucket |

---

## 4. Standard Reports

### 4.1 Operational Reports
| Report | Frequency | Format | Recipients |
|---|---|---|---|
| Daily Sales Summary | Daily 08:00 | Email + PDF | Management |
| Order Processing Status | Every 4 hours | Dashboard | Operations |
| Inventory Snapshot | Daily 06:00 | CSV + Email | Warehouse |
| Delivery Performance | Daily 20:00 | Dashboard | Logistics |
| Payment Reconciliation | Daily EOD | Excel | Finance |
| Customer Support Summary | Daily 09:00 | Email | Support Lead |

### 4.2 Financial Reports
| Report | Frequency | Format | Recipients |
|---|---|---|---|
| Revenue by Category | Weekly | Excel | CFO |
| Profit & Loss by Product | Monthly | PDF + Excel | Finance |
| Tax Summary | Monthly | PDF | Finance |
| Commission Report | Monthly | Excel | Partners |
| Expense Breakdown | Monthly | Dashboard | CFO |
| Cash Flow Forecast | Weekly | Excel | CFO |

### 4.3 Marketing Reports
| Report | Frequency | Format | Recipients |
|---|---|---|---|
| Campaign Performance | Per campaign | Dashboard | Marketing |
| Customer Acquisition Cost | Monthly | Excel | Marketing |
| Channel Attribution | Monthly | PDF | Marketing |
| Cohort Retention | Monthly | Dashboard | Marketing |
| Promotional Impact | Per promo | Excel | Marketing |
| Brand Search Trends | Weekly | Dashboard | Marketing |

### 4.4 Compliance Reports
| Report | Frequency | Format | Recipients |
|---|---|---|---|
| Transaction Audit Trail | On demand | CSV | Auditor |
| Data Access Logs | Monthly | Excel | DPO |
| GDPR Request Summary | Monthly | PDF | Legal |
| Tax Compliance | Quarterly | PDF | Finance |
| Vendor Payment Report | Monthly | Excel | Finance |

---

## 5. Report Generation

### 5.1 Generation Pipeline
```python
# Report Generation Flow
1. Scheduler triggers report job (Airflow DAG)
2. ETL extracts data from source tables
3. Transformations applied (aggregations, filters)
4. Data validated against business rules
5. Report rendered (PDF/Excel/Dashboard)
6. Distribution (email, download, dashboard)
7. Archive for compliance retention
```

### 5.2 Report Parameters
| Parameter | Options | Default |
|---|---|---|
| Date Range | Today, 7d, 30d, 90d, YTD, Custom | Last 30 days |
| Granularity | Hourly, Daily, Weekly, Monthly | Daily |
| Grouping | Category, Region, Channel, Supplier | Category |
| Format | PDF, Excel, CSV, Dashboard | PDF |
| Delivery | Email, Download, Slack, API | Email |

### 5.3 Report Templates
- **PDF Reports**: Branded header/footer, charts, executive summary
- **Excel Reports**: Multi-sheet, pivot-ready, formula-linked
- **CSV Exports**: Raw data for external analysis
- **Dashboard Links**: Live, interactive views

---

## 6. Data Export API

### 6.1 Export Endpoints
```
GET /api/v1/export/sales?from=2024-01-01&to=2024-01-31&format=csv
GET /api/v1/export/orders?status=delivered&format=excel
GET /api/v1/export/customers?segment=premium&format=csv
GET /api/v1/export/products?category=electronics&format=csv
GET /api/v1/export/inventory?warehouse=main&format=excel
GET /api/v1/export/financials?period=monthly&format=pdf
```

### 6.2 Export Limits
| Tier | Daily Exports | Max Rows | File Size |
|---|---|---|---|
| Admin | Unlimited | 1M | 100 MB |
| Manager | 50 | 100K | 50 MB |
| Analyst | 20 | 50K | 25 MB |
| Viewer | 5 | 10K | 10 MB |

### 6.3 Export Response Format
```json
{
  "export_id": "exp_abc123",
  "status": "completed",
  "format": "csv",
  "row_count": 45230,
  "file_size": "12.4 MB",
  "download_url": "/api/v1/exports/exp_abc123/download",
  "expires_at": "2024-02-01T12:00:00Z",
  "generated_at": "2024-01-31T12:00:00Z"
}
```

---

## 7. Real-time Analytics

### 7.1 Real-time Metrics
| Metric | Update Frequency | Source | Latency |
|---|---|---|---|
| Active Users | 30 seconds | Redis | < 1s |
| Live Orders | Real-time | WebSocket | < 1s |
| Cart Abandonment | 5 minutes | Kafka stream | < 5 min |
| Payment Success | Real-time | Webhook | < 1s |
| Server Health | 10 seconds | Prometheus | < 10s |
| Search Volume | 1 minute | Log aggregation | < 1 min |

### 7.2 Streaming Architecture
```
Events → Kafka/RabbitMQ → Stream Processor → Redis (real-time)
                         ↓
                    PostgreSQL (persisted)
                         ↓
                    Elasticsearch (searchable)
```

---

## 8. Analytics Data Model

### 8.1 Star Schema (Sales)
```
                    ┌─────────────┐
                    │ dim_date    │
                    └──────┬──────┘
                           │
┌─────────────┐    ┌──────┴──────┐    ┌─────────────┐
│ dim_product │────│ fact_sales  │────│ dim_customer │
└─────────────┘    └──────┬──────┘    └─────────────┘
                          │
                   ┌──────┴──────┐
                   │ dim_region  │
                   └─────────────┘
```

### 8.2 Key Dimensions
| Dimension | Key Columns | Grain |
|---|---|---|
| Date | date_key, year, month, week, day, is_holiday | One row per day |
| Product | product_key, category, subcategory, brand, supplier | One row per product |
| Customer | customer_key, segment, region, registration_date | One row per customer |
| Region | region_key, country, governorate, city, district | One row per location |
| Channel | channel_key, platform, device, source | One row per channel |

### 8.3 Key Facts
| Fact Table | Grain | Key Measures |
|---|---|---|
| fact_sales | One row per order line | quantity, revenue, cost, discount, tax |
| fact_orders | One row per order | total, items, status, delivery_time |
| fact_inventory | One row per product per day | stock_in, stock_out, balance |
| fact_sessions | One row per user session | duration, pages, conversions |
| fact_payments | One row per transaction | amount, method, status, fee |

---

## 9. Performance Optimization

### 9.1 Query Optimization
- **Materialized Views**: Pre-aggregated daily/weekly/monthly summaries
- **Partitioning**: Orders by month, products by category
- **Indexing**: Composite indexes on common filter combinations
- **Caching**: Redis cache for dashboard queries (5-min TTL)

### 9.2 Resource Allocation
| Resource | Development | Production | Analytics |
|---|---|---|---|
| CPU | 2 cores | 8 cores | 4 cores |
| RAM | 4 GB | 32 GB | 16 GB |
| Storage | 50 GB SSD | 500 GB SSD | 2 TB HDD |
| Connections | 10 | 200 | 50 |

---

## 10. Security & Access Control

### 10.1 Access Levels
| Role | Dashboards | Reports | Exports | Admin |
|---|---|---|---|---|
| CEO | All | All | Unlimited | Full |
| CFO | Financial, Sales | Financial | Unlimited | Financial |
| Manager | Department | Department | 50/day | Department |
| Analyst | Assigned | Assigned | 20/day | Read-only |
| Viewer | Limited | Limited | 5/day | None |

### 10.2 Data Masking
- Customer PII: Masked in non-admin views
- Payment details: Last 4 digits only
- Supplier pricing: Visible to finance only
- Employee data: HR reports only

---

## 11. Monitoring & Quality

### 11.1 Dashboard Health
| Check | Frequency | Threshold | Alert |
|---|---|---|---|
| Data Freshness | 5 min | > 15 min stale | Slack + Email |
| Query Performance | Continuous | > 30s load | Slack |
| Error Rate | Continuous | > 1% queries | PagerDuty |
| Cache Hit Rate | Hourly | < 80% | Slack |

### 11.2 Data Quality Checks
- Null value detection on required fields
- Outlier detection on revenue/quantity
- Duplicate order detection
- Cross-system reconciliation (orders vs payments)
- Daily automated quality scorecard
