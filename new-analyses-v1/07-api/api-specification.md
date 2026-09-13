# API Specification — YemenMart v2

**Document Version:** 1.0
**Date:** 2026-09-13
**Base URL:** `https://api.yemenmart.com/api/v1`
**OpenAPI Version:** 3.0.3

---

## 1. Overview

All endpoints require Bearer JWT authentication unless explicitly marked as public.
Requests must include `Content-Type: application/json` for body payloads.

### Rate Limit Headers (all responses)

| Header | Description |
|--------|-------------|
| `X-RateLimit-Limit` | Max requests per window |
| `X-RateLimit-Remaining` | Remaining in current window |
| `X-RateLimit-Reset` | Unix timestamp when window resets |
| `Retry-After` | Seconds until retry (429 responses) |

### Standard Error Response

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "The request body is invalid.",
    "details": [
      {
        "field": "email",
        "message": "Must be a valid email address"
      }
    ]
  },
  "request_id": "req_abc123def456",
  "timestamp": "2026-09-13T10:30:00Z"
}
```

### Success Response Envelope

```json
{
  "success": true,
  "data": { },
  "meta": {
    "page": 1,
    "per_page": 20,
    "total": 150,
    "total_pages": 8
  },
  "request_id": "req_abc123def456"
}
```

---

## 2. Authentication Endpoints (B01)

### `POST /auth/register`
Register a new customer account.

**Request:**
```json
{
  "email": "user@example.com",
  "phone": "+967712345678",
  "password": "SecureP@ss123",
  "first_name": "Mohammed",
  "last_name": "Al-Sanabani",
  "locale": "ar",
  "currency": "YER"
}
```

**Response 201:**
```json
{
  "success": true,
  "data": {
    "user_id": "usr_abc123",
    "email": "user@example.com",
    "phone": "+967712345678",
    "first_name": "Mohammed",
    "last_name": "Al-Sanabani",
    "role": "customer",
    "email_verified": false,
    "phone_verified": false,
    "created_at": "2026-09-13T10:30:00Z"
  }
}
```

**Errors:** `400 VALIDATION_ERROR`, `409 EMAIL_EXISTS`, `409 PHONE_EXISTS`

---

### `POST /auth/login`
Authenticate user and return JWT tokens.

**Request:**
```json
{
  "email": "user@example.com",
  "password": "SecureP@ss123",
  "device_id": "device_abc123"
}
```

**Response 200:**
```json
{
  "success": true,
  "data": {
    "access_token": "eyJhbGciOiJSUzI1NiIs...",
    "refresh_token": "ref_xyz789...",
    "expires_in": 900,
    "token_type": "Bearer",
    "user": {
      "user_id": "usr_abc123",
      "email": "user@example.com",
      "role": "customer",
      "2fa_required": false
    }
  }
}
```

**Errors:** `401 INVALID_CREDENTIALS`, `403 ACCOUNT_LOCKED`, `403 EMAIL_NOT_VERIFIED`, `429 TOO_MANY_ATTEMPTS`

---

### `POST /auth/refresh`
Refresh an expired access token.

**Request:**
```json
{
  "refresh_token": "ref_xyz789..."
}
```

**Response 200:**
```json
{
  "success": true,
  "data": {
    "access_token": "eyJhbGciOiJSUzI1NiIs...",
    "expires_in": 900,
    "token_type": "Bearer"
  }
}
```

**Errors:** `401 INVALID_REFRESH_TOKEN`, `401 TOKEN_EXPIRED`

---

### `POST /auth/logout`
Invalidate current session.

**Headers:** `Authorization: Bearer <token>`

**Response 204:** No content.

---

### `POST /auth/forgot-password`
Send password reset email.

**Request:**
```json
{
  "email": "user@example.com"
}
```

**Response 200:**
```json
{
  "success": true,
  "data": {
    "message": "If an account exists with this email, a reset link has been sent."
  }
}
```

---

### `POST /auth/reset-password`
Reset password using token from email.

**Request:**
```json
{
  "token": "rst_abc123...",
  "password": "NewSecureP@ss456"
}
```

**Response 200:**
```json
{
  "success": true,
  "data": {
    "message": "Password has been reset successfully."
  }
}
```

**Errors:** `400 INVALID_TOKEN`, `400 TOKEN_EXPIRED`

---

### `POST /auth/verify-email`
Verify email address using token.

**Request:**
```json
{
  "token": "vrf_xyz789..."
}
```

**Response 200:** `message: "Email verified successfully."`

---

### `POST /auth/2fa/enable`
Enable two-factor authentication (admin/vendor).

**Headers:** `Authorization: Bearer <token>`

**Response 200:**
```json
{
  "success": true,
  "data": {
    "secret": "JBSWY3DPEHPK3PXP",
    "qr_code": "data:image/png;base64,...",
    "backup_codes": ["ABCD-EFGH", "IJKL-MNOP"]
  }
}
```

---

### `POST /auth/2fa/verify`
Verify 2FA code during login.

**Request:**
```json
{
  "code": "123456",
  "session_token": "sess_abc123..."
}
```

**Response 200:**
```json
{
  "success": true,
  "data": {
    "access_token": "eyJhbGciOiJSUzI1NiIs...",
    "refresh_token": "ref_xyz789...",
    "expires_in": 900
  }
}
```

---

### `POST /auth/2fa/disable`
Disable two-factor authentication.

**Request:**
```json
{
  "password": "SecureP@ss123",
  "code": "123456"
}
```

**Response 200:** `message: "2FA disabled successfully."`

---

## 3. User Profile Endpoints (B02)

### `GET /users/me`
Get current user profile.

**Response 200:**
```json
{
  "success": true,
  "data": {
    "user_id": "usr_abc123",
    "email": "user@example.com",
    "phone": "+967712345678",
    "first_name": "Mohammed",
    "last_name": "Al-Sanabani",
    "avatar_url": "https://cdn.yemenmart.com/avatars/usr_abc123.jpg",
    "role": "customer",
    "locale": "ar",
    "currency": "YER",
    "email_verified": true,
    "phone_verified": true,
    "2fa_enabled": false,
    "created_at": "2026-01-15T08:00:00Z"
  }
}
```

---

### `PUT /users/me`
Update current user profile.

**Request:**
```json
{
  "first_name": "Mohammed",
  "last_name": "Al-Sanabani",
  "locale": "en",
  "currency": "USD"
}
```

**Response 200:** Updated user object.

---

### `PUT /users/me/avatar`
Upload user avatar.

**Headers:** `Content-Type: multipart/form-data`

**Body:** `avatar` (file, max 5MB, jpg/png/webp)

**Response 200:**
```json
{
  "success": true,
  "data": {
    "avatar_url": "https://cdn.yemenmart.com/avatars/usr_abc123.jpg?v=2"
  }
}
```

---

### `PUT /users/me/password`
Change user password.

**Request:**
```json
{
  "current_password": "OldP@ss123",
  "new_password": "NewSecureP@ss456"
}
```

**Response 200:** `message: "Password updated successfully."`

---

### `DELETE /users/me`
Soft-delete user account.

**Request:**
```json
{
  "password": "SecureP@ss123",
  "reason": "No longer using the service"
}
```

**Response 200:** `message: "Account scheduled for deletion in 30 days."`

---

## 4. Address Endpoints (B02)

### `GET /users/me/addresses`
List user addresses.

**Response 200:**
```json
{
  "success": true,
  "data": [
    {
      "address_id": "addr_abc123",
      "label": "Home",
      "full_name": "Mohammed Al-Sanabani",
      "phone": "+967712345678",
      "governorate": "Sana'a",
      "district": "Al-Tahrir",
      "street": "Al-Zubairi Street",
      "building": "12",
      "floor": "3",
      "apartment": "2",
      "landmark": "Near Al-Saleh Mosque",
      "latitude": 15.3694,
      "longitude": 44.1910,
      "is_default": true,
      "delivery_instructions": "Ring doorbell twice"
    }
  ]
}
```

---

### `POST /users/me/addresses`
Create new address.

**Request:**
```json
{
  "label": "Work",
  "full_name": "Mohammed Al-Sanabani",
  "phone": "+967712345678",
  "governorate": "Sana'a",
  "district": "Al-Tahrir",
  "street": "Al-Zubairi Street",
  "building": "12",
  "floor": "3",
  "apartment": "2",
  "landmark": "Near Al-Saleh Mosque",
  "latitude": 15.3694,
  "longitude": 44.1910,
  "is_default": false,
  "delivery_instructions": ""
}
```

**Response 201:** Address object with `address_id`.

---

### `PUT /users/me/addresses/:address_id`
Update address.

**Response 200:** Updated address object.

---

### `DELETE /users/me/addresses/:address_id`
Delete address.

**Response 204:** No content.

---

## 5. Product Endpoints (B03)

### `GET /products`
Search and list products.

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `q` | string | — | Search query |
| `category_id` | string | — | Filter by category |
| `vendor_id` | string | — | Filter by vendor |
| `min_price` | integer | — | Minimum price (YER) |
| `max_price` | integer | — | Maximum price (YER) |
| `in_stock` | boolean | — | In-stock only |
| `rating_min` | number | — | Minimum average rating |
| `sort` | string | `relevance` | `relevance`, `price_asc`, `price_desc`, `rating`, `newest`, `popular` |
| `page` | integer | 1 | Page number |
| `per_page` | integer | 20 | Results per page (max 100) |

**Response 200:**
```json
{
  "success": true,
  "data": [
    {
      "product_id": "prd_abc123",
      "slug": "samsung-galaxy-a15",
      "name": "Samsung Galaxy A15",
      "name_ar": "سامسونج جالكسي A15",
      "description": "Samsung Galaxy A15 128GB...",
      "description_ar": "سامسونج جالكسي A15 بسعة 128 جيجا...",
      "category": {
        "category_id": "cat_smartphones",
        "name": "Smartphones",
        "name_ar": "هواتف ذكية"
      },
      "vendor": {
        "vendor_id": "vnd_abc123",
        "name": "Tech Store Yemen",
        "rating": 4.7,
        "verified": true
      },
      "price": {
        "amount": 85000,
        "currency": "YER",
        "original_amount": 95000,
        "discount_percent": 11
      },
      "images": [
        {
          "url": "https://cdn.yemenmart.com/products/prd_abc123/main.webp",
          "alt": "Samsung Galaxy A15 front view",
          "is_primary": true
        }
      ],
      "stock": {
        "quantity": 45,
        "in_stock": true,
        "low_stock": false
      },
      "rating": {
        "average": 4.5,
        "count": 128
      },
      "attributes": {
        "brand": "Samsung",
        "color": "Black",
        "storage": "128GB",
        "ram": "4GB"
      },
      "tags": ["bestseller", "sale"],
      "created_at": "2026-03-15T10:00:00Z"
    }
  ],
  "meta": {
    "page": 1,
    "per_page": 20,
    "total": 1500,
    "total_pages": 75,
    "filters_applied": ["category_id:cat_smartphones"],
    "query_time_ms": 45
  }
}
```

---

### `GET /products/:product_id`
Get product details.

**Response 200:** Full product object with:
- `variants[]` — array of product variants (size, color, etc.)
- `reviews[]` — latest 10 reviews with pagination
- `related_products[]` — up to 12 related products
- `seller_policies` — return policy, warranty info
- `shipping_info` — estimated delivery, cost by governorate

---

### `GET /products/:product_id/variants`
List product variants.

**Response 200:**
```json
{
  "success": true,
  "data": [
    {
      "variant_id": "var_abc123",
      "sku": "SAM-A15-128-BLK",
      "name": "Black / 128GB",
      "price": {
        "amount": 85000,
        "currency": "YER"
      },
      "stock": {
        "quantity": 20,
        "in_stock": true
      },
      "attributes": {
        "color": "Black",
        "storage": "128GB"
      },
      "image_url": "https://cdn.yemenmart.com/products/prd_abc123/black-128.webp"
    }
  ]
}
```

---

### `POST /products/:product_id/reviews`
Create product review.

**Request:**
```json
{
  "rating": 5,
  "title": "Excellent phone!",
  "comment": "Great value for the price...",
  "images": ["https://..."]
}
```

**Response 201:** Review object.

**Errors:** `400 ALREADY_REVIEWED`, `400 ORDER_NOT_DELIVERED`

---

### `GET /products/:product_id/reviews`
List product reviews.

**Query:** `page`, `per_page`, `sort` (`newest`, `highest`, `lowest`, `most_helpful`)

---

### `POST /products/:product_id/reviews/:review_id/helpful`
Mark review as helpful.

**Response 200:** `{ "helpful_count": 12 }`

---

## 6. Category Endpoints (B03)

### `GET /categories`
List all categories (tree structure).

**Response 200:**
```json
{
  "success": true,
  "data": [
    {
      "category_id": "cat_electronics",
      "name": "Electronics",
      "name_ar": "إلكترونيات",
      "slug": "electronics",
      "icon_url": "https://cdn.yemenmart.com/icons/electronics.svg",
      "product_count": 12500,
      "children": [
        {
          "category_id": "cat_smartphones",
          "name": "Smartphones",
          "name_ar": "هواتف ذكية",
          "slug": "smartphones",
          "product_count": 3200,
          "children": []
        }
      ]
    }
  ]
}
```

---

### `GET /categories/:category_id`
Get category details with breadcrumb and filters.

---

### `GET /categories/:category_id/filters`
Get available filters for a category.

**Response 200:**
```json
{
  "success": true,
  "data": [
    {
      "filter_id": "brand",
      "name": "Brand",
      "name_ar": "العلامة التجارية",
      "type": "multi_select",
      "options": [
        { "value": "Samsung", "count": 520 },
        { "value": "Apple", "count": 380 }
      ]
    },
    {
      "filter_id": "price_range",
      "name": "Price Range",
      "name_ar": "نطاق السعر",
      "type": "range",
      "min": 10000,
      "max": 500000,
      "currency": "YER"
    }
  ]
}
```

---

## 7. Cart Endpoints (B04)

### `GET /cart`
Get current user's cart.

**Response 200:**
```json
{
  "success": true,
  "data": {
    "cart_id": "cart_abc123",
    "items": [
      {
        "item_id": "item_abc123",
        "product_id": "prd_abc123",
        "variant_id": "var_abc123",
        "product_name": "Samsung Galaxy A15",
        "variant_name": "Black / 128GB",
        "image_url": "https://cdn.yemenmart.com/...",
        "price": {
          "amount": 85000,
          "currency": "YER"
        },
        "quantity": 2,
        "subtotal": 170000,
        "in_stock": true,
        "max_quantity": 10,
        "vendor_id": "vnd_abc123",
        "vendor_name": "Tech Store Yemen"
      }
    ],
    "summary": {
      "item_count": 2,
      "subtotal": 170000,
      "shipping_estimate": 5000,
      "tax_estimate": 8500,
      "total": 183500,
      "currency": "YER"
    },
    "updated_at": "2026-09-13T10:30:00Z"
  }
}
```

---

### `POST /cart/items`
Add item to cart.

**Request:**
```json
{
  "product_id": "prd_abc123",
  "variant_id": "var_abc123",
  "quantity": 2
}
```

**Response 200:** Updated cart object.

**Errors:** `400 OUT_OF_STOCK`, `400 EXCEEDS_MAX_QUANTITY`, `400 VENDOR_RESTRICTED`

---

### `PUT /cart/items/:item_id`
Update cart item quantity.

**Request:**
```json
{
  "quantity": 3
}
```

---

### `DELETE /cart/items/:item_id`
Remove item from cart.

**Response 204:** No content.

---

### `DELETE /cart`
Clear entire cart.

**Response 204:** No content.

---

### `POST /cart/apply-coupon`
Apply coupon to cart.

**Request:**
```json
{
  "coupon_code": "RAMADAN2026"
}
```

**Response 200:**
```json
{
  "success": true,
  "data": {
    "coupon": {
      "code": "RAMADAN2026",
      "description": "Ramadan Sale — 15% off",
      "discount_type": "percentage",
      "discount_value": 15,
      "applied_amount": 25500,
      "currency": "YER"
    },
    "summary": {
      "subtotal": 170000,
      "discount": 25500,
      "shipping": 5000,
      "tax": 7225,
      "total": 146725
    }
  }
}
```

**Errors:** `400 INVALID_COUPON`, `400 COUPON_EXPIRED`, `400 COUPON_MIN_NOT_MET`

---

### `DELETE /cart/coupon`
Remove applied coupon.

---

## 8. Wishlist Endpoints (B04)

### `GET /wishlist`
List wishlist items.

---

### `POST /wishlist`
Add product to wishlist.

**Request:** `{ "product_id": "prd_abc123" }`

---

### `DELETE /wishlist/:product_id`
Remove from wishlist.

---

### `GET /wishlist/check/:product_id`
Check if product is in wishlist.

**Response 200:** `{ "in_wishlist": true }`

---

## 9. Order Endpoints (B05)

### `POST /orders`
Place a new order.

**Request:**
```json
{
  "shipping_address_id": "addr_abc123",
  "payment_method": "card",
  "coupon_code": "RAMADAN2026",
  "notes": "Please leave at door",
  "items": [
    {
      "product_id": "prd_abc123",
      "variant_id": "var_abc123",
      "quantity": 2
    }
  ]
}
```

**Response 201:**
```json
{
  "success": true,
  "data": {
    "order_id": "ord_abc123",
    "order_number": "YM-2026-00001234",
    "status": "pending_payment",
    "payment": {
      "method": "card",
      "amount": 146725,
      "currency": "YER",
      "payment_url": "https://pay.yemenmart.com/..."
    },
    "items": [...],
    "shipping": {
      "address": { ... },
      "estimated_delivery": "2026-09-16",
      "cost": 5000
    },
    "summary": {
      "subtotal": 170000,
      "discount": 25500,
      "shipping": 5000,
      "tax": 7225,
      "total": 146725
    },
    "created_at": "2026-09-13T10:30:00Z"
  }
}
```

**Errors:** `400 INSUFFICIENT_STOCK`, `400 INVALID_ADDRESS`, `400 CART_EMPTY`

---

### `GET /orders`
List user orders.

**Query:** `status`, `page`, `per_page`, `sort` (`newest`, `oldest`)

**Response:** Array of order summaries (without full item details).

---

### `GET /orders/:order_id`
Get order details.

**Response 200:** Full order object with:
- `items[]` — ordered products with current prices
- `status_history[]` — all status transitions with timestamps
- `payment` — payment details
- `shipping` — delivery tracking info
- `timeline` — estimated milestones

---

### `POST /orders/:order_id/cancel`
Cancel an order (only if status is `pending_payment` or `confirmed`).

**Request:**
```json
{
  "reason": "Found a better price elsewhere"
}
```

**Response 200:** Updated order with `status: "cancelled"`.

---

### `POST /orders/:order_id/return`
Request order return (within 7 days of delivery).

**Request:**
```json
{
  "reason": "wrong_item",
  "description": "Received wrong color",
  "images": ["https://..."]
}
```

**Response 201:** Return request object.

---

### `GET /orders/:order_id/tracking`
Get real-time delivery tracking.

**Response 200:**
```json
{
  "success": true,
  "data": {
    "order_id": "ord_abc123",
    "status": "in_transit",
    "delivery": {
      "delivery_id": "dlv_abc123",
      "rider": {
        "name": "Ahmed",
        "phone": "+967771234567",
        "avatar_url": "..."
      },
      "current_location": {
        "latitude": 15.3700,
        "longitude": 44.1950,
        "updated_at": "2026-09-13T14:00:00Z"
      },
      "estimated_arrival": "2026-09-13T15:00:00Z",
      "timeline": [
        {
          "status": "picked_up",
          "timestamp": "2026-09-13T12:00:00Z",
          "location": "Tech Store Warehouse"
        },
        {
          "status": "in_transit",
          "timestamp": "2026-09-13T13:00:00Z",
          "location": "Al-Tahrir District"
        }
      ]
    }
  }
}
```

---

## 10. Payment Endpoints (B06)

### `GET /payment/methods`
List available payment methods.

**Response 200:**
```json
{
  "success": true,
  "data": [
    {
      "method_id": "card",
      "name": "Credit/Debit Card",
      "name_ar": "بطاقة ائتمان/خصم",
      "icon_url": "...",
      "currencies": ["YER", "SAR", "USD"],
      "enabled": true
    },
    {
      "method_id": "wallet",
      "name": "YemenMart Wallet",
      "name_ar": "محفظة يمن مارت",
      "icon_url": "...",
      "currencies": ["YER"],
      "enabled": true
    },
    {
      "method_id": "cod",
      "name": "Cash on Delivery",
      "name_ar": "الدفع عند الاستلام",
      "icon_url": "...",
      "currencies": ["YER"],
      "enabled": true
    },
    {
      "method_id": "bank_transfer",
      "name": "Bank Transfer",
      "name_ar": "تحويل بنكي",
      "icon_url": "...",
      "currencies": ["YER"],
      "enabled": true
    }
  ]
}
```

---

### `POST /payment/intent`
Create payment intent (Stripe/custom gateway).

**Request:**
```json
{
  "order_id": "ord_abc123",
  "method": "card",
  "currency": "YER"
}
```

**Response 200:**
```json
{
  "success": true,
  "data": {
    "payment_id": "pay_abc123",
    "client_secret": "pi_abc123_secret_xyz...",
    "amount": 146725,
    "currency": "YER",
    "expires_at": "2026-09-13T11:00:00Z"
  }
}
```

---

### `POST /payment/webhook`
Payment provider webhook callback (internal).

**Headers:** `X-Webhook-Signature: sha256=...`

---

### `GET /payments`
List payment history.

**Query:** `status`, `page`, `per_page`

---

### `GET /payments/:payment_id`
Get payment details.

---

### `POST /payments/:payment_id/retry`
Retry a failed payment.

---

### `POST /payments/:payment_id/refund`
Request payment refund (admin/vendor).

**Request:**
```json
{
  "amount": 85000,
  "reason": "Product out of stock"
}
```

---

## 11. Wallet Endpoints (B07)

### `GET /wallet`
Get wallet balance and summary.

**Response 200:**
```json
{
  "success": true,
  "data": {
    "wallet_id": "wal_abc123",
    "balance": 250000,
    "currency": "YER",
    "pending_balance": 15000,
    "total_earned": 500000,
    "total_spent": 250000,
    "last_transaction_at": "2026-09-12T18:00:00Z"
  }
}
```

---

### `GET /wallet/transactions`
List wallet transactions.

**Query:** `type` (`credit`, `debit`, `refund`, `cashback`, `withdrawal`), `from_date`, `to_date`, `page`, `per_page`

**Response 200:**
```json
{
  "success": true,
  "data": [
    {
      "transaction_id": "txn_abc123",
      "type": "credit",
      "amount": 25000,
      "balance_after": 250000,
      "description": "Order #YM-2026-00001234 refund",
      "reference_type": "order",
      "reference_id": "ord_abc123",
      "created_at": "2026-09-12T18:00:00Z"
    }
  ],
  "meta": { "page": 1, "per_page": 20, "total": 45 }
}
```

---

### `POST /wallet/topup`
Initiate wallet top-up.

**Request:**
```json
{
  "amount": 100000,
  "currency": "YER",
  "payment_method": "card"
}
```

**Response 200:** Payment intent for the top-up.

---

### `POST /wallet/transfer`
Transfer between wallets (user-to-user).

**Request:**
```json
{
  "recipient_id": "usr_xyz789",
  "amount": 10000,
  "note": "Split dinner bill"
}
```

**Response 200:** Transfer confirmation.

---

### `POST /wallet/withdraw`
Request wallet withdrawal to bank account.

**Request:**
```json
{
  "amount": 50000,
  "bank_account_id": "bank_abc123"
}
```

---

### `GET /wallet/bank-accounts`
List linked bank accounts.

---

### `POST /wallet/bank-accounts`
Link a bank account.

---

### `DELETE /wallet/bank-accounts/:bank_account_id`
Remove linked bank account.

---

## 12. Vendor Endpoints (B08)

### `GET /vendors`
List all vendors (public).

**Query:** `q`, `category_id`, `rating_min`, `page`, `per_page`, `sort`

---

### `GET /vendors/:vendor_id`
Get vendor public profile.

**Response 200:**
```json
{
  "success": true,
  "data": {
    "vendor_id": "vnd_abc123",
    "name": "Tech Store Yemen",
    "name_ar": "متجر تك اليمن",
    "description": "...",
    "avatar_url": "...",
    "banner_url": "...",
    "rating": {
      "average": 4.7,
      "count": 1250
    },
    "product_count": 320,
    "established_year": 2020,
    "verified": true,
    "location": {
      "governorate": "Sana'a",
      "district": "Al-Tahrir"
    },
    "policies": {
      "return_days": 14,
      "warranty_days": 365
    },
    "response_time": "Within 2 hours",
    "shipping": {
      "free_shipping_min": 200000,
      "default_cost": 5000
    }
  }
}
```

---

### `GET /vendors/:vendor_id/products`
List vendor products.

---

### `POST /vendor/register`
Register as a vendor.

**Request:**
```json
{
  "business_name": "Tech Store Yemen",
  "business_name_ar": "متجر تك اليمن",
  "business_type": "electronics",
  "commercial_registration": "CR-12345",
  "tax_id": "123456789",
  "governorate": "Sana'a",
  "district": "Al-Tahrir",
  "phone": "+967712345678",
  "email": "vendor@example.com",
  "bank_account": {
    "bank_name": "Yemeni Gulf Bank",
    "account_number": "12345678901234",
    "account_name": "Tech Store LLC"
  },
  "documents": {
    "commercial_registration_doc": "https://...",
    "tax_cert_doc": "https://...",
    "national_id_doc": "https://..."
  }
}
```

**Response 201:** Vendor application with `status: "pending_review"`.

---

### `GET /vendor/dashboard`
Vendor dashboard summary (authenticated vendor).

**Response 200:**
```json
{
  "success": true,
  "data": {
    "today": {
      "orders": 12,
      "revenue": 850000,
      "pending_shipments": 5
    },
    "this_week": {
      "orders": 78,
      "revenue": 5200000,
      "new_reviews": 8
    },
    "overall": {
      "total_products": 320,
      "active_products": 295,
      "pending_orders": 18,
      "return_requests": 2,
      "wallet_balance": 12500000,
      "rating": 4.7
    },
    "recent_orders": [
      {
        "order_id": "ord_xyz789",
        "order_number": "YM-2026-00005678",
        "customer": "Ahmed",
        "total": 120000,
        "status": "confirmed",
        "created_at": "2026-09-13T10:00:00Z"
      }
    ]
  }
}
```

---

### `GET /vendor/orders`
List vendor orders.

**Query:** `status`, `page`, `per_page`

---

### `PUT /vendor/orders/:order_id/status`
Update order status (vendor).

**Request:**
```json
{
  "status": "processing",
  "note": "Preparing for shipment"
}
```

**Valid transitions:** `confirmed` → `processing` → `shipped` → `delivered`

---

### `POST /vendor/orders/:order_id/ship`
Mark order as shipped.

**Request:**
```json
{
  "tracking_number": "YMSHP-123456",
  "carrier": "YemenPost",
  "estimated_delivery": "2026-09-16"
}
```

---

### `GET /vendor/products`
List vendor products (vendor portal).

---

### `POST /vendor/products`
Create new product.

**Request:** Full product object (see Product schema).

**Response 201:** Created product with `status: "pending_review"`.

---

### `PUT /vendor/products/:product_id`
Update product.

---

### `DELETE /vendor/products/:product_id`
Soft-delete product.

---

### `GET /vendor/wallet`
Get vendor wallet balance.

---

### `GET /vendor/wallet/transactions`
List vendor wallet transactions.

---

### `POST /vendor/wallet/withdraw`
Request vendor wallet withdrawal.

---

### `GET /vendor/reviews`
List vendor reviews.

---

### `POST /vendor/reviews/:review_id/reply`
Reply to customer review.

---

### `GET /vendor/analytics`
Vendor analytics (sales, traffic, conversion).

---

### `GET /vendor/analytics/products`
Per-product analytics.

---

## 13. Admin Endpoints (B09)

### `GET /admin/dashboard`
Admin dashboard overview.

**Response 200:**
```json
{
  "success": true,
  "data": {
    "today": {
      "orders": 450,
      "revenue": 25000000,
      "new_users": 120,
      "active_users": 3500
    },
    "this_month": {
      "orders": 12500,
      "revenue": 750000000,
      "new_users": 3500,
      "vendor_applications": 45
    },
    "system": {
      "api_p95": 145,
      "error_rate": 0.12,
      "uptime": 99.99
    }
  }
}
```

---

### `GET /admin/users`
List all users.

**Query:** `role`, `status`, `q`, `page`, `per_page`

---

### `GET /admin/users/:user_id`
Get user details.

---

### `PUT /admin/users/:user_id/status`
Update user status (`active`, `suspended`, `banned`).

**Request:**
```json
{
  "status": "suspended",
  "reason": "Fraudulent activity detected",
  "notify_user": true
}
```

---

### `GET /admin/vendors`
List all vendor applications.

**Query:** `status` (`pending_review`, `approved`, `rejected`, `suspended`), `page`, `per_page`

---

### `GET /admin/vendors/:vendor_id`
Get vendor details (full, including financials).

---

### `PUT /admin/vendors/:vendor_id/approve`
Approve vendor application.

---

### `PUT /admin/vendors/:vendor_id/reject`
Reject vendor application.

**Request:**
```json
{
  "reason": "Incomplete documentation",
  "details": "Missing commercial registration document"
}
```

---

### `PUT /admin/vendors/:vendor_id/suspend`
Suspend vendor.

---

### `GET /admin/orders`
List all platform orders.

**Query:** `status`, `vendor_id`, `date_from`, `date_to`, `page`, `per_page`

---

### `GET /admin/orders/:order_id`
Get order details (admin view).

---

### `POST /admin/orders/:order_id/refund`
Process refund (admin override).

---

### `GET /admin/products`
List all products (admin view).

**Query:** `status` (`active`, `pending_review`, `rejected`, `flagged`), `category_id`, `vendor_id`, `page`, `per_page`

---

### `PUT /admin/products/:product_id/approve`
Approve product listing.

---

### `PUT /admin/products/:product_id/reject`
Reject product listing.

---

### `GET /admin/transactions`
List all transactions.

**Query:** `type`, `status`, `date_from`, `date_to`, `page`, `per_page`

---

### `GET /admin/finance/summary`
Finance summary (revenue, commissions, payouts).

---

### `GET /admin/finance/vendor-payouts`
List vendor payouts.

---

### `POST /admin/finance/vendor-payouts/:payout_id/process`
Process vendor payout.

---

### `GET /admin/finance/commission-settings`
Get commission rate settings.

---

### `PUT /admin/finance/commission-settings`
Update commission rates.

**Request:**
```json
{
  "default_rate": 10,
  "category_rates": {
    "cat_electronics": 8,
    "cat_groceries": 5
  }
}
```

---

### `GET /admin/reports/sales`
Sales report with filters.

**Query:** `date_from`, `date_to`, `group_by` (`day`, `week`, `month`), `category_id`, `vendor_id`

---

### `GET /admin/reports/vendors`
Vendor performance report.

---

### `GET /admin/reports/products`
Product performance report.

---

### `GET /admin/settings`
Get platform settings.

---

### `PUT /admin/settings`
Update platform settings.

**Request:**
```json
{
  "maintenance_mode": false,
  "min_order_amount": 5000,
  "max_order_amount": 5000000,
  "free_shipping_threshold": 200000,
  "delivery_fee": 5000,
  "tax_rate": 5,
  "allowed_payment_methods": ["card", "wallet", "cod", "bank_transfer"]
}
```

---

### `GET /admin/coupons`
List all coupons.

---

### `POST /admin/coupons`
Create new coupon.

**Request:**
```json
{
  "code": "WELCOME2026",
  "description": "Welcome discount — 10% off first order",
  "discount_type": "percentage",
  "discount_value": 10,
  "min_order_amount": 50000,
  "max_discount": 25000,
  "usage_limit": 1000,
  "per_user_limit": 1,
  "valid_from": "2026-09-01T00:00:00Z",
  "valid_until": "2026-12-31T23:59:59Z",
  "applicable_categories": ["all"],
  "applicable_vendors": ["all"]
}
```

---

### `PUT /admin/coupons/:coupon_id`
Update coupon.

---

### `DELETE /admin/coupons/:coupon_id`
Deactivate coupon.

---

### `GET /admin/coupons/:coupon_id/stats`
Coupon usage statistics.

---

## 14. Delivery Endpoints (B10)

### `GET /deliveries/rider/assignments`
List rider delivery assignments (authenticated rider).

---

### `PUT /deliveries/:delivery_id/accept`
Rider accepts delivery assignment.

---

### `PUT /deliveries/:delivery_id/pickup`
Mark delivery as picked up.

---

### `PUT /deliveries/:delivery_id/deliver`
Mark delivery as completed.

**Request:**
```json
{
  "proof_of_delivery": {
    "photo_url": "https://...",
    "signature_url": "https://...",
    "latitude": 15.3694,
    "longitude": 44.1910
  }
}
```

---

### `PUT /deliveries/:delivery_id/failed`
Mark delivery as failed.

**Request:**
```json
{
  "reason": "customer_unavailable",
  "notes": "Customer not at address",
  "photo_url": "https://..."
}
```

---

### `GET /deliveries/:delivery_id/route`
Get optimized delivery route.

---

### `GET /rider/dashboard`
Rider dashboard (authenticated rider).

---

### `GET /rider/earnings`
Rider earnings summary.

---

## 15. Notification Endpoints (B11)

### `GET /notifications`
List user notifications.

**Query:** `type`, `read` (`true`/`false`), `page`, `per_page`

---

### `PUT /notifications/:notification_id/read`
Mark notification as read.

---

### `PUT /notifications/read-all`
Mark all notifications as read.

---

### `GET /notifications/preferences`
Get notification preferences.

---

### `PUT /notifications/preferences`
Update notification preferences.

**Request:**
```json
{
  "email": {
    "order_updates": true,
    "promotions": false,
    "newsletter": true
  },
  "push": {
    "order_updates": true,
    "promotions": true,
    "price_drops": true
  },
  "sms": {
    "order_updates": true,
    "promotions": false
  }
}
```

---

### `POST /notifications/fcm-token`
Register FCM device token.

**Request:** `{ "token": "dF7x...", "platform": "android" }`

---

### `DELETE /notifications/fcm-token/:token`
Unregister FCM device token.

---

## 16. Search & Discovery Endpoints (B12)

### `GET /search`
Full-text product search.

**Query:** `q`, `category_id`, `sort`, `page`, `per_page`, `facets` (comma-separated)

---

### `GET /search/autocomplete`
Search autocomplete suggestions.

**Query:** `q` (min 2 chars), `locale`

**Response 200:**
```json
{
  "success": true,
  "data": {
    "suggestions": [
      { "type": "product", "text": "Samsung Galaxy A15", "product_id": "prd_abc123", "image_url": "..." },
      { "type": "category", "text": "Smartphones", "category_id": "cat_smartphones" },
      { "type": "brand", "text": "Samsung" }
    ]
  }
}
```

---

### `GET /search/trending`
Get trending search terms.

---

### `GET /home`
Homepage data (personalized).

**Response 200:**
```json
{
  "success": true,
  "data": {
    "banners": [...],
    "flash_sales": [...],
    "popular_categories": [...],
    "trending_products": [...],
    "recommended_for_you": [...],
    "new_arrivals": [...],
    "top_vendors": [...]
  }
}
```

---

### `GET /recommendations`
Product recommendations.

**Query:** `type` (`similar`, `also_bought`, `trending`, `for_you`), `product_id` (for similar/also_bought)

---

## 17. Coupon Endpoints (B13)

### `POST /coupons/validate`
Validate a coupon code.

**Request:**
```json
{
  "code": "WELCOME2026",
  "cart_total": 150000,
  "items": [
    { "product_id": "prd_abc123", "category_id": "cat_smartphones", "vendor_id": "vnd_abc123", "price": 85000 }
  ]
}
```

**Response 200:**
```json
{
  "success": true,
  "data": {
    "valid": true,
    "coupon_code": "WELCOME2026",
    "description": "Welcome discount — 10% off",
    "discount_type": "percentage",
    "discount_value": 10,
    "max_discount": 25000,
    "calculated_discount": 15000,
    "applicable_items": 1,
    "minimum_met": true
  }
}
```

**Errors:** `400 INVALID_COUPON`, `400 COUPON_EXPIRED`, `400 MINIMUM_NOT_MET`, `400 USAGE_LIMIT_REACHED`, `400 NOT_APPLICABLE`

---

## 18. File Upload Endpoints (B13)

### `POST /uploads/presigned-url`
Get presigned upload URL.

**Request:**
```json
{
  "file_type": "image",
  "content_type": "image/jpeg",
  "file_size": 2048000
}
```

**Response 200:**
```json
{
  "success": true,
  "data": {
    "upload_url": "https://s3.amazonaws.com/...",
    "file_url": "https://cdn.yemenmart.com/uploads/...",
    "expires_at": "2026-09-13T11:00:00Z",
    "max_file_size": 10485760
  }
}
```

---

### `POST /uploads/confirm`
Confirm upload completion.

**Request:**
```json
{
  "file_url": "https://cdn.yemenmart.com/uploads/...",
  "file_type": "product_image",
  "metadata": {
    "product_id": "prd_abc123",
    "is_primary": true
  }
}
```

---

## 19. ZATCA E-Invoice Endpoints (B13)

### `POST /invoices/generate`
Generate ZATCA-compliant invoice.

**Request:**
```json
{
  "order_id": "ord_abc123"
}
```

**Response 200:**
```json
{
  "success": true,
  "data": {
    "invoice_id": "inv_abc123",
    "invoice_number": "YM-INV-2026-0001234",
    "xml": "<?xml version=\"1.0\"...>",
    "qr_code": "data:image/png;base64,...",
    "digital_signature": "...",
    "zatca_uuid": "...",
    "issued_at": "2026-09-13T10:30:00Z"
  }
}
```

---

### `GET /invoices/:invoice_id`
Get invoice details.

---

### `GET /invoices/:invoice_id/pdf`
Download invoice PDF.

---

### `POST /invoices/:invoice_id/void`
Void an invoice (admin only).

---

## 20. Health & Status Endpoints

### `GET /health`
Service health check (no auth).

**Response 200:**
```json
{
  "status": "healthy",
  "version": "2.0.0",
  "timestamp": "2026-09-13T10:30:00Z"
}
```

---

### `GET /status`
Detailed system status (internal).

---

## Appendix A: HTTP Status Codes

| Code | Usage |
|------|-------|
| 200 | Successful GET/PUT |
| 201 | Successful POST (resource created) |
| 204 | Successful DELETE (no content) |
| 400 | Validation error / bad request |
| 401 | Authentication required or invalid |
| 403 | Insufficient permissions |
| 404 | Resource not found |
| 409 | Conflict (duplicate, state conflict) |
| 413 | Payload too large |
| 415 | Unsupported media type |
| 422 | Unprocessable entity (semantic errors) |
| 429 | Rate limit exceeded |
| 500 | Internal server error |
| 502 | Bad gateway |
| 503 | Service unavailable |
| 504 | Gateway timeout |

---

## Appendix B: Authentication Flow

```
1. Client → POST /auth/login → { access_token, refresh_token }
2. Client → GET /orders → Authorization: Bearer <access_token>
3. Token expires (401) → Client → POST /auth/refresh → { new access_token }
4. Refresh expires → Client → POST /auth/login → new token pair
5. Logout → POST /auth/logout → tokens invalidated server-side
```

---

## Appendix C: Pagination Standard

All list endpoints support:

| Parameter | Type | Default | Max |
|-----------|------|---------|-----|
| `page` | integer | 1 | — |
| `per_page` | integer | 20 | 100 |

Response includes `meta` object with `total`, `total_pages`, `page`, `per_page`.
