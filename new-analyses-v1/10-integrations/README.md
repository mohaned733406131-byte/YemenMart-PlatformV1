# 10 - Integrations

**Category:** Integrations  
**Purpose:** Payment providers, delivery services, external APIs

---

## Contents

- `integration-overview.md` - Integration architecture
- `payment-providers.md` - m-Floos, OneCash, bank transfer integrations
- `delivery-providers.md` - Delivery marketplace provider APIs
- `sms-providers.md` - SMS gateway integrations (Twilio, local providers)
- `notification-services.md` - Push notifications, email services
- `analytics-integrations.md` - Google Analytics, custom analytics
- `third-party-apis.md` - External API dependencies

---

## Payment Integrations

### Wallet Top-Up (Bank Transfer Only)
- **m-Floos** - Mobile wallet integration
- **OneCash** - Mobile wallet integration
- **Bank Transfer** - Direct bank transfer (manual verification)

**NO credit/debit card processors, NO BNPL, NO installments**

### Payment Flow
1. Customer initiates wallet top-up
2. System generates bank transfer reference
3. Customer transfers to platform bank account
4. Admin verifies transfer manually or via webhook
5. Wallet credited after verification

---

## SMS Integrations

### Primary SMS Gateway
- **Provider:** Twilio / Local Yemeni SMS provider
- **Purpose:** OTP delivery, notifications, alerts
- **Fallback:** Secondary provider for redundancy
- **Rate Limits:** 1 OTP per 60 seconds per phone number

### SMS Templates
- Registration OTP
- Login OTP
- Password reset OTP
- Order confirmation
- Delivery updates
- Payment notifications

---

## Delivery Provider Integration

### Marketplace API
- **Registration:** Provider registers with API key
- **Bid Submission:** POST /delivery-bids with pricing
- **Assignment:** System assigns winning bid
- **Status Updates:** Provider sends tracking updates
- **Proof of Delivery:** Upload delivery code confirmation

### Provider Requirements
- RESTful API endpoint for webhooks
- Authentication via API keys
- Standard response formats
- Rate limiting compliance

---

## Analytics Integrations

### Google Analytics 4
- **Tracking:** Page views, events, conversions
- **E-commerce:** Product impressions, cart events, purchases
- **Custom Dimensions:** User role, vendor ID, order value

### Custom Analytics
- **Backend Service:** Custom analytics engine
- **Metrics:** Real-time dashboards, vendor performance, financial reports
- **Storage:** PostgreSQL + Elasticsearch

---

## Integration Patterns

### API Adapter Pattern
All external integrations use adapter pattern for:
- **Consistency:** Standardized internal interfaces
- **Flexibility:** Easy provider switching
- **Testing:** Mock adapters for testing

### Webhook Handling
- **Verification:** Signature validation for all webhooks
- **Retry Logic:** Exponential backoff for failed webhooks
- **Idempotency:** Duplicate webhook detection

---

## Related Categories
- `06-backend` - Integration implementation
- `07-api` - API specifications
- `09-security` - Integration security

---

*Source: Integration requirements from design documents and architecture*
