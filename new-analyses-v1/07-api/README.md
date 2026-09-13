# 07 - API

**Category:** API  
**Purpose:** REST API specifications, endpoints, and contracts

---

## Contents

- `api-overview.md` - API architecture and design principles
- `authentication-endpoints.md` - Auth and session management APIs
- `customer-endpoints.md` - Customer-facing APIs
- `vendor-endpoints.md` - Vendor management APIs
- `admin-endpoints.md` - Admin control APIs
- `payment-endpoints.md` - Wallet and payment APIs
- `order-endpoints.md` - Order management APIs
- `api-versioning.md` - Version strategy (v1, v2, etc.)
- `error-handling.md` - Standard error responses
- `rate-limiting.md` - Rate limit policies

---

## API Design

### Base URL
```
Production: https://api.yemenmart.com/v1
Staging: https://api-staging.yemenmart.com/v1
Development: http://localhost:9000/v1
```

### Authentication
- **Method:** JWT Bearer Token
- **Acquisition:** SMS OTP → JWT token
- **Refresh:** Refresh token rotation
- **Expiry:** Access token 1 hour, Refresh token 30 days

### Standards
- **Protocol:** REST over HTTPS
- **Format:** JSON (application/json)
- **Versioning:** URL-based (/v1, /v2)
- **Rate Limiting:** 100 req/min for authenticated, 20 req/min for anonymous
- **Pagination:** Cursor-based for large datasets
- **Filtering:** Query parameters (?status=active&limit=20)
- **Sorting:** Query parameters (?sort=created_at:desc)

---

## Endpoint Groups

### Authentication (Public)
- `POST /v1/auth/register` - SMS OTP registration
- `POST /v1/auth/verify-otp` - OTP verification
- `POST /v1/auth/login` - SMS OTP login
- `POST /v1/auth/refresh` - Token refresh
- `POST /v1/auth/logout` - Logout

### Customers
- `GET /v1/customers/me` - Current customer profile
- `PUT /v1/customers/me` - Update profile
- `GET /v1/customers/orders` - Order history
- `GET /v1/customers/wallet` - Wallet balance

### Products
- `GET /v1/products` - Product listing (paginated)
- `GET /v1/products/:id` - Product details
- `GET /v1/products/search` - Full-text search

### Orders
- `POST /v1/orders` - Create order
- `GET /v1/orders/:id` - Order details
- `PUT /v1/orders/:id/cancel` - Cancel order
- `POST /v1/orders/:id/confirm-delivery` - Delivery confirmation

### Payments
- `GET /v1/wallet/balance` - Wallet balance
- `POST /v1/wallet/topup` - Initiate bank transfer top-up
- `GET /v1/wallet/transactions` - Transaction history

---

## Error Responses

Standard error format:
```json
{
  "error": {
    "code": "INVALID_OTP",
    "message": "The OTP code provided is invalid or expired",
    "details": {},
    "timestamp": "2026-09-15T10:30:00Z"
  }
}
```

HTTP Status Codes:
- `200` - Success
- `201` - Created
- `400` - Bad Request
- `401` - Unauthorized
- `403` - Forbidden
- `404` - Not Found
- `429` - Too Many Requests
- `500` - Internal Server Error

---

## Related Categories
- `06-backend` - Backend implementation
- `09-security` - API security
- `13-testing` - API testing

---

*Source: API specifications from analayesev2/07-API and design documents*
