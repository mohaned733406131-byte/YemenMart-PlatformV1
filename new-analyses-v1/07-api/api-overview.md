# YemenMart REST API Specification v1.0

**Base URL:** `https://api.yemenmart.com/api/v1`  
**Content-Type:** `application/json`  
**Last Updated:** September 2026

---

## Table of Contents

1. [Global Standards](#global-standards)
2. [Authentication](#authentication)
3. [Customers](#customers)
4. [Products](#products)
5. [Vendors](#vendors)
6. [Orders](#orders)
7. [Payments & Wallet](#payments--wallet)
8. [Cart](#cart)
9. [Reviews](#reviews)
10. [Coupons](#coupons)
11. [Support Tickets](#support-tickets)
12. [Admin](#admin)
13. [Notifications](#notifications)
14. [Delivery](#delivery)

---

## Global Standards

### Authentication Header

```
Authorization: Bearer <access_token>
```

### Pagination Format

```json
{
  "data": [],
  "pagination": {
    "current_page": 1,
    "per_page": 20,
    "total_pages": 10,
    "total_items": 195,
    "has_next": true,
    "has_prev": false
  }
}
```

### Standard Success Response

```json
{
  "success": true,
  "message": "Operation successful",
  "data": {}
}
```

### Error Response Format

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input data",
    "details": [
      {
        "field": "phone",
        "message": "Phone number must be in format +967XXXXXXXXX"
      }
    ]
  },
  "request_id": "req_abc123xyz"
}
```

### Common Error Codes

| HTTP Status | Code | Description |
|-------------|------|-------------|
| 400 | VALIDATION_ERROR | Invalid request body |
| 401 | UNAUTHORIZED | Missing or invalid token |
| 403 | FORBIDDEN | Insufficient permissions |
| 404 | NOT_FOUND | Resource not found |
| 409 | CONFLICT | Resource already exists |
| 422 | UNPROCESSABLE | Business logic error |
| 429 | RATE_LIMITED | Too many requests |
| 500 | INTERNAL_ERROR | Server error |

### Rate Limiting Headers

```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1726233600
X-RateLimit-Policy: 100;w=60
```

### Request ID Header

```
X-Request-ID: req_abc123xyz
```

---

## 1. Authentication

### POST /api/v1/auth/register

**Description:** Register a new customer account via phone number  
**Authentication:** None  
**Rate Limit:** 5 requests per hour per IP

**Request Body:**

```json
{
  "phone": "+967771234567",
  "full_name": "Mohammed Ali",
  "password": "SecureP@ss123",
  "delivery_method": "sms",
  "referral_code": "ABC123"
}
```

**Validation Rules:**
- `phone`: Required, E.164 format, Yemen prefix (+967)
- `full_name`: Required, 2-100 characters
- `password`: Required, min 8 chars, must include uppercase, lowercase, number
- `delivery_method`: Required, enum: `sms`, `whatsapp` (for OTP verification only)
- `referral_code`: Optional, 6 alphanumeric characters

**Success Response (201):**

```json
{
  "success": true,
  "message": "OTP sent to your phone for verification",
  "data": {
    "user_id": "usr_abc123",
    "phone": "+967771234567",
    "otp_expires_in": 300,
    "delivery_method": "sms"
  }
}
```

**Error Responses:**

| Status | Code | Message |
|--------|------|---------|
| 409 | CONFLICT | Phone number already registered |
| 422 | VALIDATION_ERROR | Invalid phone format |
| 429 | RATE_LIMITED | Too many registration attempts |

---

### POST /api/v1/auth/login

**Description:** Login with phone number + password (PRIMARY method)  
**Authentication:** None  
**Rate Limit:** 5 requests per hour per IP

**Request Body:**

```json
{
  "phone": "+967771234567",
  "password": "SecureP@ss123"
}
```

**Validation Rules:**
- `phone`: Required, E.164 format
- `password`: Required

**Success Response (200):**

```json
{
  "success": true,
  "message": "Login successful",
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIs...",
    "refresh_token": "eyJhbGciOiJIUzI1NiIs...",
    "token_type": "Bearer",
    "expires_in": 900,
    "user": {
      "id": "usr_abc123",
      "phone": "+967771234567",
      "full_name": "Mohammed Ali",
      "role": "BUYER"
    }
  }
}
```

**Error Responses:**

| Status | Code | Message |
|--------|------|---------|
| 401 | UNAUTHORIZED | Invalid phone or password |
| 423 | LOCKED | Account locked due to too many failed attempts |

---

### POST /api/v1/auth/forgot-password

**Description:** Request OTP for password reset (SECONDARY method)  
**Authentication:** None  
**Rate Limit:** 3 requests per hour per IP

**Request Body:**

```json
{
  "phone": "+967771234567",
  "delivery_method": "sms"
}
```

**Validation Rules:**
- `phone`: Required, E.164 format
- `delivery_method`: Required, enum: `sms`, `whatsapp`

**Success Response (200):**

```json
{
  "success": true,
  "message": "If the phone number exists, a verification code has been sent",
  "data": {
    "otp_expires_in": 300,
    "delivery_method": "sms"
  }
}
```

**Error Responses:**

| Status | Code | Message |
|--------|------|---------|
| 429 | RATE_LIMITED | Too many password reset requests |

---

### POST /api/v1/auth/reset-password

**Description:** Reset password using OTP verification  
**Authentication:** None  
**Rate Limit:** 5 requests per hour per IP

**Request Body:**

```json
{
  "phone": "+967771234567",
  "otp": "482956",
  "new_password": "NewSecureP@ss123"
}
```

**Validation Rules:**
- `phone`: Required, E.164 format
- `otp`: Required, exactly 6 digits
- `new_password`: Required, min 8 chars, must include uppercase, lowercase, number

**Success Response (200):**

```json
{
  "success": true,
  "message": "Password reset successful. All sessions have been invalidated."
}
```

**Error Responses:**

| Status | Code | Message |
|--------|------|---------|
| 400 | VALIDATION_ERROR | Invalid OTP |
| 400 | EXPIRED_OTP | OTP has expired |
| 400 | WEAK_PASSWORD | Password does not meet strength requirements |
| 429 | RATE_LIMITED | Too many verification attempts |

---

### POST /api/v1/auth/forgot-password

**Description:** Request password reset OTP  
**Authentication:** None  
**Rate Limit:** 3 requests per hour per IP

**Request Body:**

```json
{
  "phone": "+967771234567"
}
```

**Success Response (200):**

```json
{
  "success": true,
  "message": "Password reset OTP sent",
  "data": {
    "user_id": "usr_abc123",
    "otp_expires_in": 300
  }
}
```

---

### POST /api/v1/auth/reset-password

**Description:** Reset password using verified OTP  
**Authentication:** None (requires valid OTP session)  
**Rate Limit:** 5 requests per hour per IP

**Request Body:**

```json
{
  "user_id": "usr_abc123",
  "otp": "482956",
  "new_password": "NewSecureP@ss456"
}
```

**Success Response (200):**

```json
{
  "success": true,
  "message": "Password reset successful. Please login with new password."
}
```

---

### POST /api/v1/auth/refresh-token

**Description:** Get new access token using refresh token  
**Authentication:** Refresh token in body  
**Rate Limit:** 20 requests per hour

**Request Body:**

```json
{
  "refresh_token": "rt_def456ghi789"
}
```

**Success Response (200):**

```json
{
  "success": true,
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIs...",
    "expires_in": 3600
  }
}
```

**Error Responses:**

| Status | Code | Message |
|--------|------|---------|
| 401 | INVALID_TOKEN | Refresh token expired or invalid |
| 401 | TOKEN_REVOKED | Refresh token has been revoked |

---

### POST /api/v1/auth/logout

**Description:** Revoke current session  
**Authentication:** Bearer token required  
**Rate Limit:** 30 requests per hour

**Request Body:**

```json
{
  "refresh_token": "rt_def456ghi789",
  "all_devices": false
}
```

**Success Response (200):**

```json
{
  "success": true,
  "message": "Logged out successfully"
}
```

---

## 2. Customers

### GET /api/v1/customers/profile

**Description:** Get authenticated customer's profile  
**Authentication:** Bearer token required  
**Rate Limit:** 60 requests per hour

**Success Response (200):**

```json
{
  "success": true,
  "data": {
    "id": "usr_abc123",
    "phone": "+967771234567",
    "full_name": "Mohammed Ali",
    "email": "mohammed@example.com",
    "avatar_url": "https://cdn.yemenmart.com/avatars/usr_abc123.jpg",
    "is_verified": true,
    "wallet_balance": 15000.00,
    "loyalty_points": 320,
    "created_at": "2026-01-15T10:30:00Z",
    "addresses_count": 3
  }
}
```

---

### PUT /api/v1/customers/profile

**Description:** Update customer profile  
**Authentication:** Bearer token required  
**Rate Limit:** 10 requests per hour

**Request Body:**

```json
{
  "full_name": "Mohammed Ali Hassan",
  "email": "mohammed@example.com",
  "avatar_url": "https://cdn.yemenmart.com/avatars/usr_abc123.jpg"
}
```

**Validation Rules:**
- `full_name`: Optional, 2-100 characters
- `email`: Optional, valid email format
- `avatar_url`: Optional, valid URL

**Success Response (200):**

```json
{
  "success": true,
  "message": "Profile updated successfully",
  "data": {
    "id": "usr_abc123",
    "full_name": "Mohammed Ali Hassan",
    "email": "mohammed@example.com"
  }
}
```

---

### GET /api/v1/customers/addresses

**Description:** List all saved addresses  
**Authentication:** Bearer token required  
**Rate Limit:** 60 requests per hour

**Success Response (200):**

```json
{
  "success": true,
  "data": [
    {
      "id": "addr_001",
      "label": "Home",
      "full_name": "Mohammed Ali",
      "phone": "+967771234567",
      "governorate": "Sana'a",
      "district": "Al-Sabaeen",
      "street": "Al-Sabaeen Street, Building 45",
      "landmark": "Near Al-Sabaeen Park",
      "latitude": 15.3547,
      "longitude": 44.2066,
      "is_default": true,
      "created_at": "2026-03-10T08:00:00Z"
    },
    {
      "id": "addr_002",
      "label": "Office",
      "full_name": "Mohammed Ali",
      "phone": "+967771234567",
      "governorate": "Aden",
      "district": "Crater",
      "street": "Crater Road, Office 12",
      "landmark": "Next to Aden Mall",
      "latitude": 12.7855,
      "longitude": 45.0187,
      "is_default": false,
      "created_at": "2026-04-20T14:30:00Z"
    }
  ]
}
```

---

### POST /api/v1/customers/addresses

**Description:** Add a new address  
**Authentication:** Bearer token required  
**Rate Limit:** 20 requests per hour

**Request Body:**

```json
{
  "label": "Home",
  "full_name": "Mohammed Ali",
  "phone": "+967771234567",
  "governorate": "Sana'a",
  "district": "Al-Sabaeen",
  "street": "Al-Sabaeen Street, Building 45",
  "landmark": "Near Al-Sabaeen Park",
  "latitude": 15.3547,
  "longitude": 44.2066,
  "is_default": true
}
```

**Validation Rules:**
- `label`: Required, 1-50 characters (e.g., Home, Office, Other)
- `governorate`: Required, must be valid Yemen governorate
- `district`: Required, must be valid district within governorate
- `street`: Required, 5-200 characters
- `latitude`: Optional, decimal (-90 to 90)
- `longitude`: Optional, decimal (-180 to 180)
- `is_default`: Optional, boolean, default false

**Business Rules:**
- Maximum 10 addresses per customer
- Setting `is_default: true` unsets previous default

**Success Response (201):**

```json
{
  "success": true,
  "message": "Address added successfully",
  "data": {
    "id": "addr_003",
    "label": "Home",
    "governorate": "Sana'a",
    "is_default": true
  }
}
```

---

### PUT /api/v1/customers/addresses/:id

**Description:** Update an existing address  
**Authentication:** Bearer token required  
**Rate Limit:** 20 requests per hour

**Path Parameters:**
- `id`: Address ID (addr_xxx)

**Request Body:**

```json
{
  "label": "New Home",
  "street": "Updated Street Address",
  "is_default": true
}
```

**Success Response (200):**

```json
{
  "success": true,
  "message": "Address updated successfully",
  "data": {
    "id": "addr_001",
    "label": "New Home",
    "street": "Updated Street Address"
  }
}
```

**Error Responses:**

| Status | Code | Message |
|--------|------|---------|
| 404 | NOT_FOUND | Address not found |

---

### DELETE /api/v1/customers/addresses/:id

**Description:** Delete an address  
**Authentication:** Bearer token required  
**Rate Limit:** 20 requests per hour

**Path Parameters:**
- `id`: Address ID (addr_xxx)

**Business Rules:**
- Cannot delete address if it's the only one
- Cannot delete address with active orders

**Success Response (200):**

```json
{
  "success": true,
  "message": "Address deleted successfully"
}
```

**Error Responses:**

| Status | Code | Message |
|--------|------|---------|
| 404 | NOT_FOUND | Address not found |
| 422 | CANNOT_DELETE | Cannot delete only address |
| 422 | ACTIVE_ORDER | Address has active orders |

---

## 3. Products

### GET /api/v1/products

**Description:** Search and list products  
**Authentication:** None  
**Rate Limit:** 120 requests per hour

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `q` | string | - | Search query |
| `category_id` | string | - | Filter by category |
| `vendor_id` | string | - | Filter by vendor |
| `min_price` | number | - | Minimum price (YER) |
| `max_price` | number | - | Maximum price (YER) |
| `in_stock` | boolean | - | Only in-stock items |
| `rating_min` | number | - | Minimum rating (1-5) |
| `sort` | string | `relevance` | Sort: `relevance`, `price_asc`, `price_desc`, `newest`, `rating`, `sales` |
| `page` | integer | 1 | Page number |
| `per_page` | integer | 20 | Items per page (max 50) |

**Success Response (200):**

```json
{
  "success": true,
  "data": [
    {
      "id": "prod_xyz789",
      "name": "Samsung Galaxy S24 Ultra",
      "slug": "samsung-galaxy-s24-ultra",
      "description": "Latest Samsung flagship with S Pen",
      "short_description": "6.8\" Dynamic AMOLED, 200MP Camera",
      "price": 285000.00,
      "original_price": 320000.00,
      "currency": "YER",
      "discount_percentage": 11,
      "thumbnail_url": "https://cdn.yemenmart.com/products/prod_xyz789/thumb.jpg",
      "images": [
        "https://cdn.yemenmart.com/products/prod_xyz789/img1.jpg",
        "https://cdn.yemenmart.com/products/prod_xyz789/img2.jpg"
      ],
      "category": {
        "id": "cat_electronics",
        "name": "Electronics"
      },
      "vendor": {
        "id": "vnd_abc123",
        "name": "Tech Store Yemen",
        "rating": 4.8,
        "is_verified": true
      },
      "rating": 4.7,
      "reviews_count": 156,
      "sales_count": 423,
      "in_stock": true,
      "stock_quantity": 15,
      "variants_count": 3,
      "has_variants": true,
      "is_featured": true,
      "created_at": "2026-08-01T10:00:00Z"
    }
  ],
  "pagination": {
    "current_page": 1,
    "per_page": 20,
    "total_pages": 15,
    "total_items": 287,
    "has_next": true,
    "has_prev": false
  },
  "filters_applied": {
    "category_id": "cat_electronics",
    "in_stock": true
  }
}
```

---

### GET /api/v1/products/:id

**Description:** Get product details  
**Authentication:** None  
**Rate Limit:** 120 requests per hour

**Path Parameters:**
- `id`: Product ID (prod_xxx)

**Success Response (200):**

```json
{
  "success": true,
  "data": {
    "id": "prod_xyz789",
    "name": "Samsung Galaxy S24 Ultra",
    "slug": "samsung-galaxy-s24-ultra",
    "description": "Full product description with HTML support...",
    "short_description": "6.8\" Dynamic AMOLED, 200MP Camera",
    "price": 285000.00,
    "original_price": 320000.00,
    "currency": "YER",
    "discount_percentage": 11,
    "images": [
      {
        "id": "img_001",
        "url": "https://cdn.yemenmart.com/products/prod_xyz789/img1.jpg",
        "alt_text": "Samsung Galaxy S24 Ultra front view",
        "sort_order": 1
      }
    ],
    "category": {
      "id": "cat_electronics",
      "name": "Electronics",
      "breadcrumb": "Home > Electronics > Mobile Phones"
    },
    "vendor": {
      "id": "vnd_abc123",
      "name": "Tech Store Yemen",
      "rating": 4.8,
      "is_verified": true,
      "response_time": "< 2 hours"
    },
    "specifications": [
      {
        "group": "Display",
        "items": [
          { "key": "Size", "value": "6.8 inches" },
          { "key": "Type", "value": "Dynamic AMOLED 2X" },
          { "key": "Resolution", "value": "3120 x 1440" }
        ]
      },
      {
        "group": "Performance",
        "items": [
          { "key": "Processor", "value": "Snapdragon 8 Gen 3" },
          { "key": "RAM", "value": "12GB" },
          { "key": "Storage", "value": "256GB" }
        ]
      }
    ],
    "variants": [
      {
        "id": "var_001",
        "name": "256GB / Titanium Black",
        "price": 285000.00,
        "original_price": 320000.00,
        "sku": "SAM-S24U-256-BLK",
        "stock_quantity": 8,
        "in_stock": true,
        "attributes": [
          { "key": "Storage", "value": "256GB" },
          { "key": "Color", "value": "Titanium Black" }
        ]
      }
    ],
    "rating": 4.7,
    "reviews_count": 156,
    "sales_count": 423,
    "in_stock": true,
    "total_stock": 15,
    "weight": 232,
    "weight_unit": "g",
    "return_policy": "7-day return policy",
    "warranty": "1 year manufacturer warranty",
    "is_featured": true,
    "tags": ["samsung", "galaxy", "smartphone", "5g"],
    "created_at": "2026-08-01T10:00:00Z",
    "updated_at": "2026-09-01T15:30:00Z"
  }
}
```

**Error Responses:**

| Status | Code | Message |
|--------|------|---------|
| 404 | NOT_FOUND | Product not found |
| 410 | GONE | Product has been removed |

---

### GET /api/v1/products/:id/variants

**Description:** Get all variants for a product  
**Authentication:** None  
**Rate Limit:** 120 requests per hour

**Success Response (200):**

```json
{
  "success": true,
  "data": [
    {
      "id": "var_001",
      "name": "256GB / Titanium Black",
      "price": 285000.00,
      "original_price": 320000.00,
      "sku": "SAM-S24U-256-BLK",
      "stock_quantity": 8,
      "in_stock": true,
      "image_url": "https://cdn.yemenmart.com/products/prod_xyz789/var_001.jpg",
      "attributes": [
        { "key": "Storage", "value": "256GB", "group": "Storage" },
        { "key": "Color", "value": "Titanium Black", "group": "Color" }
      ]
    },
    {
      "id": "var_002",
      "name": "512GB / Titanium Gray",
      "price": 335000.00,
      "original_price": 375000.00,
      "sku": "SAM-S24U-512-GRY",
      "stock_quantity": 5,
      "in_stock": true,
      "image_url": "https://cdn.yemenmart.com/products/prod_xyz789/var_002.jpg",
      "attributes": [
        { "key": "Storage", "value": "512GB", "group": "Storage" },
        { "key": "Color", "value": "Titanium Gray", "group": "Color" }
      ]
    }
  ]
}
```

---

### GET /api/v1/products/:id/reviews

**Description:** Get product reviews  
**Authentication:** None  
**Rate Limit:** 60 requests per hour

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `rating` | integer | - | Filter by rating (1-5) |
| `sort` | string | `newest` | Sort: `newest`, `oldest`, `highest`, `lowest`, `helpful` |
| `page` | integer | 1 | Page number |
| `per_page` | integer | 10 | Items per page |

**Success Response (200):**

```json
{
  "success": true,
  "data": [
    {
      "id": "rev_001",
      "customer": {
        "id": "usr_def456",
        "full_name": "Ahmed S.",
        "avatar_url": "https://cdn.yemenmart.com/avatars/usr_def456.jpg",
        "is_verified": true
      },
      "rating": 5,
      "title": "Excellent phone!",
      "comment": "Best phone I've ever used. Camera quality is amazing.",
      "images": [
        "https://cdn.yemenmart.com/reviews/rev_001/img1.jpg"
      ],
      "variant_purchased": "256GB / Titanium Black",
      "is_verified_purchase": true,
      "helpful_count": 23,
      "vendor_reply": {
        "comment": "Thank you for your wonderful review!",
        "replied_at": "2026-08-15T12:00:00Z"
      },
      "created_at": "2026-08-10T14:30:00Z"
    }
  ],
  "summary": {
    "average_rating": 4.7,
    "total_reviews": 156,
    "rating_distribution": {
      "5": 98,
      "4": 35,
      "3": 12,
      "2": 7,
      "1": 4
    }
  },
  "pagination": {
    "current_page": 1,
    "per_page": 10,
    "total_pages": 16,
    "total_items": 156
  }
}
```

---

### GET /api/v1/categories

**Description:** Get all categories  
**Authentication:** None  
**Rate Limit:** 60 requests per hour

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `include_children` | boolean | true | Include subcategories |
| `parent_id` | string | - | Filter by parent category |

**Success Response (200):**

```json
{
  "success": true,
  "data": [
    {
      "id": "cat_electronics",
      "name": "Electronics",
      "slug": "electronics",
      "description": "Phones, laptops, and gadgets",
      "image_url": "https://cdn.yemenmart.com/categories/electronics.jpg",
      "icon": "device-phone",
      "product_count": 2847,
      "sort_order": 1,
      "is_active": true,
      "children": [
        {
          "id": "cat_mobile_phones",
          "name": "Mobile Phones",
          "slug": "mobile-phones",
          "product_count": 1256,
          "children": []
        },
        {
          "id": "cat_laptops",
          "name": "Laptops & Computers",
          "slug": "laptops-computers",
          "product_count": 834,
          "children": []
        }
      ]
    },
    {
      "id": "cat_fashion",
      "name": "Fashion",
      "slug": "fashion",
      "product_count": 5623,
      "children": []
    }
  ]
}
```

---

### GET /api/v1/categories/:id/products

**Description:** Get products in a category  
**Authentication:** None  
**Rate Limit:** 60 requests per hour

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `sort` | string | `relevance` | Sort: `relevance`, `price_asc`, `price_desc`, `newest`, `sales` |
| `page` | integer | 1 | Page number |
| `per_page` | integer | 20 | Items per page |

**Success Response (200):** Returns same format as `GET /api/v1/products`

---

## 4. Vendors

### POST /api/v1/vendors/register

**Description:** Register as a vendor  
**Authentication:** Bearer token required (customer account)  
**Rate Limit:** 3 requests per day

**Request Body:**

```json
{
  "store_name": "Tech World Yemen",
  "store_description": "Authorized Samsung and Apple reseller",
  "store_logo_url": "https://example.com/logo.png",
  "store_banner_url": "https://example.com/banner.jpg",
  "business_type": "retailer",
  "governorate": "Sana'a",
  "district": "Al-Sabaeen",
  "street_address": "Al-Sabaeen Street, Shop 12",
  "contact_phone": "+967771234567",
  "contact_email": "contact@techworld.ye",
  "category_ids": ["cat_electronics", "cat_accessories"]
}
```

**Validation Rules:**
- `store_name`: Required, 3-100 characters, unique
- `business_type`: Required, enum: `individual`, `retailer`, `wholesaler`, `manufacturer`
- `category_ids`: Required, at least 1 category

**Business Rules:**
- Customer account must be verified
- Only one store per user
- KYC required before store activation

**Success Response (201):**

```json
{
  "success": true,
  "message": "Vendor registration submitted. Complete KYC to activate.",
  "data": {
    "vendor_id": "vnd_new123",
    "store_name": "Tech World Yemen",
    "status": "pending_kyc",
    "kyc_required": true,
    "next_step": "POST /api/v1/vendors/kyc"
  }
}
```

---

### POST /api/v1/vendors/kyc

**Description:** Submit KYC verification documents  
**Authentication:** Bearer token required (vendor role)  
**Rate Limit:** 5 requests per day

**Request Body:**

```json
{
  "document_type": "national_id",
  "document_number": "1234567890",
  "document_front_url": "https://storage.yemenmart.com/kyc/abc/front.jpg",
  "document_back_url": "https://storage.yemenmart.com/kyc/abc/back.jpg",
  "selfie_url": "https://storage.yemenmart.com/kyc/abc/selfie.jpg",
  "business_registration_url": null,
  "commercial_register_number": null
}
```

**Validation Rules:**
- `document_type`: Required, enum: `national_id`, `passport`, `commercial_register`
- `document_number`: Required, valid format
- `document_front_url`: Required, valid URL
- `document_back_url`: Required for national_id/passport
- `selfie_url`: Required

**Business Rules:**
- KYC review takes 24-72 hours
- Store remains inactive until KYC approved

**Success Response (200):**

```json
{
  "success": true,
  "message": "KYC documents submitted for review",
  "data": {
    "vendor_id": "vnd_new123",
    "kyc_status": "pending_review",
    "estimated_review_time": "24-72 hours"
  }
}
```

---

### GET /api/v1/vendors/dashboard

**Description:** Get vendor dashboard statistics  
**Authentication:** Bearer token required (vendor role)  
**Rate Limit:** 30 requests per hour

**Success Response (200):**

```json
{
  "success": true,
  "data": {
    "store_status": "active",
    "store_name": "Tech World Yemen",
    "summary": {
      "total_products": 45,
      "active_products": 42,
      "out_of_stock": 3,
      "pending_orders": 12,
      "total_orders": 1567,
      "orders_this_month": 89
    },
    "financial": {
      "wallet_balance": 2500000.00,
      "pending_payouts": 450000.00,
      "total_earnings": 15600000.00,
      "earnings_this_month": 1250000.00,
      "commission_rate": 0.05
    },
    "performance": {
      "average_rating": 4.8,
      "total_reviews": 456,
      "response_rate": 0.95,
      "average_response_time": "1.5 hours",
      "fulfillment_rate": 0.98,
      "return_rate": 0.02
    },
    "recent_orders": [
      {
        "id": "ord_001",
        "customer_name": "Ahmed M.",
        "total": 125000.00,
        "status": "pending",
        "created_at": "2026-09-13T10:00:00Z"
      }
    ],
    "low_stock_alerts": [
      {
        "product_id": "prod_abc",
        "product_name": "iPhone 15 Case",
        "stock_quantity": 2
      }
    ]
  }
}
```

---

### GET /api/v1/vendors/products

**Description:** List vendor's products  
**Authentication:** Bearer token required (vendor role)  
**Rate Limit:** 60 requests per hour

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `status` | string | `all` | Filter: `active`, `draft`, `out_of_stock`, `all` |
| `sort` | string | `newest` | Sort: `newest`, `oldest`, `price_asc`, `price_desc`, `sales` |
| `page` | integer | 1 | Page number |
| `per_page` | integer | 20 | Items per page |

**Success Response (200):**

```json
{
  "success": true,
  "data": [
    {
      "id": "prod_xyz789",
      "name": "Samsung Galaxy S24 Ultra",
      "sku": "SAM-S24U-256-BLK",
      "price": 285000.00,
      "stock_quantity": 15,
      "status": "active",
      "sales_count": 423,
      "views_count": 12500,
      "thumbnail_url": "https://cdn.yemenmart.com/products/prod_xyz789/thumb.jpg",
      "created_at": "2026-08-01T10:00:00Z"
    }
  ],
  "pagination": {
    "current_page": 1,
    "per_page": 20,
    "total_pages": 3,
    "total_items": 45
  }
}
```

---

### POST /api/v1/vendors/products

**Description:** Create a new product  
**Authentication:** Bearer token required (vendor role)  
**Rate Limit:** 30 requests per hour

**Request Body:**

```json
{
  "name": "Samsung Galaxy S24 Ultra",
  "description": "Full product description...",
  "short_description": "6.8\" Dynamic AMOLED, 200MP Camera",
  "category_id": "cat_mobile_phones",
  "price": 285000.00,
  "original_price": 320000.00,
  "sku": "SAM-S24U-256-BLK",
  "stock_quantity": 15,
  "weight": 232,
  "weight_unit": "g",
  "images": [
    { "url": "https://storage.yemenmart.com/products/img1.jpg", "sort_order": 1 },
    { "url": "https://storage.yemenmart.com/products/img2.jpg", "sort_order": 2 }
  ],
  "specifications": [
    {
      "group": "Display",
      "items": [
        { "key": "Size", "value": "6.8 inches" }
      ]
    }
  ],
  "variants": [
    {
      "name": "256GB / Titanium Black",
      "sku": "SAM-S24U-256-BLK",
      "price": 285000.00,
      "stock_quantity": 8,
      "attributes": [
        { "key": "Storage", "value": "256GB" },
        { "key": "Color", "value": "Titanium Black" }
      ]
    }
  ],
  "return_policy": "7-day return policy",
  "warranty": "1 year manufacturer warranty",
  "tags": ["samsung", "galaxy", "smartphone"]
}
```

**Validation Rules:**
- `name`: Required, 3-200 characters
- `price`: Required, positive number, min 100 YER
- `stock_quantity`: Required for simple products, non-negative integer
- `images`: Required, at least 1 image
- `variants`: Optional, but if provided each must have name, SKU, price, stock

**Business Rules:**
- Vendor store must be active
- Product requires approval for first 30 days
- Duplicate SKUs not allowed within vendor

**Success Response (201):**

```json
{
  "success": true,
  "message": "Product created successfully. Pending approval.",
  "data": {
    "id": "prod_new456",
    "name": "Samsung Galaxy S24 Ultra",
    "status": "pending_approval",
    "estimated_approval_time": "24 hours"
  }
}
```

---

### PUT /api/v1/vendors/products/:id

**Description:** Update a product  
**Authentication:** Bearer token required (vendor role)  
**Rate Limit:** 30 requests per hour

**Path Parameters:**
- `id`: Product ID

**Request Body:** Same as create, all fields optional

**Business Rules:**
- Cannot edit product with pending orders
- Price changes limited to 20% per day

**Success Response (200):**

```json
{
  "success": true,
  "message": "Product updated successfully",
  "data": {
    "id": "prod_xyz789",
    "status": "active"
  }
}
```

---

### DELETE /api/v1/vendors/products/:id

**Description:** Delete/archive a product  
**Authentication:** Bearer token required (vendor role)  
**Rate Limit:** 10 requests per hour

**Business Rules:**
- Cannot delete product with pending/processing orders
- Product is archived, not permanently deleted

**Success Response (200):**

```json
{
  "success": true,
  "message": "Product archived successfully"
}
```

**Error Responses:**

| Status | Code | Message |
|--------|------|---------|
| 422 | ACTIVE_ORDERS | Cannot delete product with pending orders |

---

### GET /api/v1/vendors/orders

**Description:** List vendor's orders  
**Authentication:** Bearer token required (vendor role)  
**Rate Limit:** 60 requests per hour

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `status` | string | `all` | Filter: `pending`, `confirmed`, `processing`, `shipped`, `delivered`, `cancelled`, `returned` |
| `date_from` | string | - | ISO date (YYYY-MM-DD) |
| `date_to` | string | - | ISO date (YYYY-MM-DD) |
| `page` | integer | 1 | Page number |
| `per_page` | integer | 20 | Items per page |

**Success Response (200):**

```json
{
  "success": true,
  "data": [
    {
      "id": "ord_001",
      "order_number": "YM-2026-001234",
      "customer": {
        "id": "usr_abc123",
        "name": "Mohammed Ali",
        "phone": "+967771234567"
      },
      "items": [
        {
          "product_id": "prod_xyz789",
          "product_name": "Samsung Galaxy S24 Ultra",
          "variant": "256GB / Titanium Black",
          "quantity": 1,
          "unit_price": 285000.00,
          "total": 285000.00
        }
      ],
      "subtotal": 285000.00,
      "delivery_fee": 5000.00,
      "total": 290000.00,
      "currency": "YER",
      "payment_method": "wallet",
      "status": "pending",
      "delivery_address": {
        "governorate": "Sana'a",
        "district": "Al-Sabaeen",
        "street": "Al-Sabaeen Street, Building 45"
      },
      "created_at": "2026-09-13T10:30:00Z",
      "confirmed_at": null,
      "delivered_at": null
    }
  ],
  "pagination": {
    "current_page": 1,
    "per_page": 20,
    "total_pages": 5,
    "total_items": 89
  }
}
```

---

### PUT /api/v1/vendors/orders/:id/confirm

**Description:** Confirm an order  
**Authentication:** Bearer token required (vendor role)  
**Rate Limit:** 30 requests per hour

**Path Parameters:**
- `id`: Order ID

**Request Body:**

```json
{
  "estimated_preparation_time": "2-3 days",
  "notes": "Item in stock, ready to ship"
}
```

**Business Rules:**
- Must confirm within 24 hours or auto-cancelled
- Cannot confirm if product is out of stock

**Success Response (200):**

```json
{
  "success": true,
  "message": "Order confirmed successfully",
  "data": {
    "order_id": "ord_001",
    "status": "confirmed",
    "confirmed_at": "2026-09-13T11:00:00Z"
  }
}
```

**Error Responses:**

| Status | Code | Message |
|--------|------|---------|
| 422 | OUT_OF_STOCK | Product is out of stock |
| 422 | EXPIRED_WINDOW | Confirmation window expired (24h) |

---

### GET /api/v1/vendors/store

**Description:** Get vendor's public store information  
**Authentication:** None  
**Rate Limit:** 60 requests per hour

**Success Response (200):**

```json
{
  "success": true,
  "data": {
    "id": "vnd_abc123",
    "store_name": "Tech World Yemen",
    "store_description": "Authorized Samsung and Apple reseller",
    "store_logo_url": "https://cdn.yemenmart.com/stores/vnd_abc123/logo.jpg",
    "store_banner_url": "https://cdn.yemenmart.com/stores/vnd_abc123/banner.jpg",
    "rating": 4.8,
    "total_reviews": 456,
    "total_products": 42,
    "total_sales": 1567,
    "is_verified": true,
    "joined_date": "2025-06-15",
    "response_rate": 0.95,
    "average_response_time": "1.5 hours",
    "categories": [
      { "id": "cat_electronics", "name": "Electronics" }
    ]
  }
}
```

---

## 5. Orders

### POST /api/v1/orders

**Description:** Create a new order (checkout)  
**Authentication:** Bearer token required  
**Rate Limit:** 10 requests per hour

**Request Body:**

```json
{
  "address_id": "addr_001",
  "payment_method": "wallet",
  "coupon_code": "SAVE10",
  "notes": "Please deliver in the morning",
  "items": [
    {
      "product_id": "prod_xyz789",
      "variant_id": "var_001",
      "quantity": 1
    },
    {
      "product_id": "prod_abc123",
      "variant_id": null,
      "quantity": 2
    }
  ]
}
```

**Validation Rules:**
- `address_id`: Required, must be valid customer address
- `payment_method`: Required, enum: `wallet`, `cod` (cash on delivery), `card`
- `items`: Required, at least 1 item
- `quantity`: Required, positive integer, max 10 per item

**Business Rules:**
- All items must be from vendors with active stores
- COD requires minimum order 10,000 YER
- Maximum 5 vendors per order (splits into multiple orders if needed)
- Wallet balance must cover total

**Success Response (201):**

```json
{
  "success": true,
  "message": "Order placed successfully",
  "data": {
    "orders": [
      {
        "id": "ord_001",
        "order_number": "YM-2026-001234",
        "vendor": {
          "id": "vnd_abc123",
          "name": "Tech World Yemen"
        },
        "subtotal": 285000.00,
        "delivery_fee": 5000.00,
        "discount": 28500.00,
        "total": 261500.00,
        "currency": "YER",
        "status": "pending",
        "payment_method": "wallet",
        "payment_status": "paid",
        "estimated_delivery": "2026-09-16"
      }
    ],
    "total_amount": 261500.00,
    "wallet_deducted": 261500.00,
    "remaining_balance": 1238500.00,
    "created_at": "2026-09-13T10:30:00Z"
  }
}
```

**Error Responses:**

| Status | Code | Message |
|--------|------|---------|
| 400 | INSUFFICIENT_BALANCE | Wallet balance insufficient |
| 400 | MINIMUM_ORDER | COD requires minimum 10,000 YER |
| 422 | OUT_OF_STOCK | One or more items out of stock |
| 422 | INVALID_COUPON | Coupon is invalid or expired |
| 422 | DELIVERY_AREA | Delivery not available in this area |

---

### GET /api/v1/orders

**Description:** List customer's orders  
**Authentication:** Bearer token required  
**Rate Limit:** 60 requests per hour

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `status` | string | `all` | Filter by status |
| `page` | integer | 1 | Page number |
| `per_page` | integer | 20 | Items per page |

**Success Response (200):**

```json
{
  "success": true,
  "data": [
    {
      "id": "ord_001",
      "order_number": "YM-2026-001234",
      "vendors": [
        {
          "id": "vnd_abc123",
          "name": "Tech World Yemen",
          "items_count": 1
        }
      ],
      "items_summary": {
        "total_items": 3,
        "thumbnail_url": "https://cdn.yemenmart.com/products/prod_xyz789/thumb.jpg"
      },
      "subtotal": 285000.00,
      "delivery_fee": 5000.00,
      "total": 290000.00,
      "currency": "YER",
      "status": "shipped",
      "status_history": [
        { "status": "pending", "timestamp": "2026-09-13T10:30:00Z" },
        { "status": "confirmed", "timestamp": "2026-09-13T11:00:00Z" },
        { "status": "shipped", "timestamp": "2026-09-14T09:00:00Z" }
      ],
      "tracking": {
        "carrier": "YemenExpress",
        "tracking_number": "YE123456789"
      },
      "created_at": "2026-09-13T10:30:00Z"
    }
  ],
  "pagination": {
    "current_page": 1,
    "per_page": 20,
    "total_pages": 8,
    "total_items": 156
  }
}
```

---

### GET /api/v1/orders/:id

**Description:** Get order details  
**Authentication:** Bearer token required  
**Rate Limit:** 60 requests per hour

**Success Response (200):**

```json
{
  "success": true,
  "data": {
    "id": "ord_001",
    "order_number": "YM-2026-001234",
    "customer": {
      "id": "usr_abc123",
      "name": "Mohammed Ali",
      "phone": "+967771234567"
    },
    "vendor": {
      "id": "vnd_abc123",
      "name": "Tech World Yemen",
      "phone": "+967779876543"
    },
    "items": [
      {
        "product_id": "prod_xyz789",
        "product_name": "Samsung Galaxy S24 Ultra",
        "variant": "256GB / Titanium Black",
        "quantity": 1,
        "unit_price": 285000.00,
        "total": 285000.00,
        "thumbnail_url": "https://cdn.yemenmart.com/products/prod_xyz789/thumb.jpg"
      }
    ],
    "subtotal": 285000.00,
    "delivery_fee": 5000.00,
    "discount": 0,
    "total": 290000.00,
    "currency": "YER",
    "payment_method": "wallet",
    "payment_status": "paid",
    "status": "shipped",
    "status_history": [
      { "status": "pending", "timestamp": "2026-09-13T10:30:00Z", "note": "Order placed" },
      { "status": "confirmed", "timestamp": "2026-09-13T11:00:00Z", "note": "Vendor confirmed" },
      { "status": "processing", "timestamp": "2026-09-13T14:00:00Z", "note": "Preparing for shipment" },
      { "status": "shipped", "timestamp": "2026-09-14T09:00:00Z", "note": "Handed to courier" }
    ],
    "delivery_address": {
      "label": "Home",
      "full_name": "Mohammed Ali",
      "phone": "+967771234567",
      "governorate": "Sana'a",
      "district": "Al-Sabaeen",
      "street": "Al-Sabaeen Street, Building 45",
      "landmark": "Near Al-Sabaeen Park"
    },
    "tracking": {
      "carrier": "YemenExpress",
      "tracking_number": "YE123456789",
      "tracking_url": "https://yemenexpress.ye/track/YE123456789",
      "estimated_delivery": "2026-09-16"
    },
    "notes": "Please deliver in the morning",
    "created_at": "2026-09-13T10:30:00Z",
    "updated_at": "2026-09-14T09:00:00Z"
  }
}
```

---

### POST /api/v1/orders/:id/cancel

**Description:** Cancel an order  
**Authentication:** Bearer token required  
**Rate Limit:** 10 requests per hour

**Path Parameters:**
- `id`: Order ID

**Request Body:**

```json
{
  "reason": "Changed my mind",
  "details": "Found a better price elsewhere"
}
```

**Validation Rules:**
- `reason`: Required, enum: `changed_mind`, `found_better_price`, `duplicate_order`, `delivery_too_slow`, `other`
- `details`: Optional, max 500 characters

**Business Rules:**
- Can only cancel if status is `pending` or `confirmed`
- Wallet refunded immediately for prepaid orders
- COD orders simply marked as cancelled

**Success Response (200):**

```json
{
  "success": true,
  "message": "Order cancelled successfully",
  "data": {
    "order_id": "ord_001",
    "status": "cancelled",
    "refund_amount": 290000.00,
    "refund_method": "wallet",
    "cancelled_at": "2026-09-13T12:00:00Z"
  }
}
```

**Error Responses:**

| Status | Code | Message |
|--------|------|---------|
| 422 | CANNOT_CANCEL | Order cannot be cancelled in current status |

---

### POST /api/v1/orders/:id/return

**Description:** Request a return  
**Authentication:** Bearer token required  
**Rate Limit:** 5 requests per hour

**Path Parameters:**
- `id`: Order ID

**Request Body:**

```json
{
  "reason": "defective_product",
  "details": "Screen has a dead pixel",
  "items": [
    {
      "order_item_id": "item_001",
      "quantity": 1,
      "reason": "Defective screen"
    }
  ],
  "images": [
    "https://storage.yemenmart.com/returns/img1.jpg",
    "https://storage.yemenmart.com/returns/img2.jpg"
  ]
}
```

**Validation Rules:**
- `reason`: Required, enum: `defective_product`, `wrong_item`, `not_as_described`, `damaged_in_transit`, `other`
- `items`: Required, at least 1 item
- `images`: Recommended, max 5 images

**Business Rules:**
- Return window: 7 days after delivery
- Must provide evidence (photos) for defective/damaged
- Refund to wallet within 3-5 business days

**Success Response (201):**

```json
{
  "success": true,
  "message": "Return request submitted",
  "data": {
    "return_id": "ret_001",
    "order_id": "ord_001",
    "status": "pending_review",
    "estimated_refund": 290000.00,
    "review_time": "24-48 hours"
  }
}
```

---

### POST /api/v1/orders/:id/confirm-delivery

**Description:** Confirm order received  
**Authentication:** Bearer token required  
**Rate Limit:** 10 requests per hour

**Path Parameters:**
- `id`: Order ID

**Business Rules:**
- Auto-confirmed after 7 days if not manually confirmed
- Triggers vendor payout

**Success Response (200):**

```json
{
  "success": true,
  "message": "Delivery confirmed. You can now leave a review.",
  "data": {
    "order_id": "ord_001",
    "status": "delivered",
    "confirmed_at": "2026-09-16T14:00:00Z",
    "can_review": true
  }
}
```

---

## 6. Payments & Wallet

### GET /api/v1/wallet/balance

**Description:** Get wallet balance  
**Authentication:** Bearer token required  
**Rate Limit:** 60 requests per hour

**Success Response (200):**

```json
{
  "success": true,
  "data": {
    "balance": 1500000.00,
    "currency": "YER",
    "pending_transactions": 2,
    "last_updated": "2026-09-13T10:30:00Z"
  }
}
```

---

### GET /api/v1/wallet/transactions

**Description:** List wallet transactions  
**Authentication:** Bearer token required  
**Rate Limit:** 60 requests per hour

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `type` | string | `all` | Filter: `credit`, `debit`, `all` |
| `category` | string | `all` | Filter: `topup`, `purchase`, `refund`, `transfer`, `commission`, `all` |
| `date_from` | string | - | ISO date |
| `date_to` | string | - | ISO date |
| `page` | integer | 1 | Page number |
| `per_page` | integer | 20 | Items per page |

**Success Response (200):**

```json
{
  "success": true,
  "data": [
    {
      "id": "txn_001",
      "type": "credit",
      "category": "topup",
      "amount": 500000.00,
      "balance_after": 1500000.00,
      "description": "Wallet top-up via Fawri",
      "reference": "FAWRI-2026-001",
      "status": "completed",
      "created_at": "2026-09-12T15:00:00Z"
    },
    {
      "id": "txn_002",
      "type": "debit",
      "category": "purchase",
      "amount": -290000.00,
      "balance_after": 1000000.00,
      "description": "Order YM-2026-001234",
      "reference": "ord_001",
      "status": "completed",
      "created_at": "2026-09-13T10:30:00Z"
    }
  ],
  "pagination": {
    "current_page": 1,
    "per_page": 20,
    "total_pages": 12,
    "total_items": 234
  }
}
```

---

### POST /api/v1/wallet/topup

**Description:** Top up wallet  
**Authentication:** Bearer token required  
**Rate Limit:** 10 requests per hour

**Request Body:**

```json
{
  "amount": 500000.00,
  "payment_method": "fawri",
  "phone": "+967771234567",
  "reference": "FAWRI-2026-001"
}
```

**Validation Rules:**
- `amount`: Required, positive, min 1000 YER, max 5000000 YER
- `payment_method`: Required, enum: `fawri`, `cash_card`, `bank_transfer`, `credit_card`
- `phone`: Required for mobile money

**Business Rules:**
- Fawri: Instant processing
- Bank transfer: 1-24 hours
- Credit card: Instant with 2.5% fee

**Success Response (201):**

```json
{
  "success": true,
  "message": "Top-up initiated",
  "data": {
    "transaction_id": "txn_003",
    "amount": 500000.00,
    "payment_method": "fawri",
    "status": "processing",
    "reference": "FAWRI-2026-001",
    "estimated_completion": "Instant"
  }
}
```

---

### POST /api/v1/wallet/transfer

**Description:** Transfer funds to another wallet  
**Authentication:** Bearer token required  
**Rate Limit:** 5 requests per hour

**Request Body:**

```json
{
  "recipient_phone": "+967779876543",
  "amount": 50000.00,
  "note": "Payment for dinner"
}
```

**Validation Rules:**
- `recipient_phone`: Required, must be registered user
- `amount`: Required, positive, min 1000 YER
- Cannot transfer to self

**Business Rules:**
- No fees for transfers
- Instant processing
- Daily transfer limit: 2,000,000 YER

**Success Response (200):**

```json
{
  "success": true,
  "message": "Transfer successful",
  "data": {
    "transaction_id": "txn_004",
    "recipient": {
      "id": "usr_xyz789",
      "name": "Ali Hassan",
      "phone": "+967779876543"
    },
    "amount": 50000.00,
    "note": "Payment for dinner",
    "new_balance": 950000.00,
    "created_at": "2026-09-13T12:00:00Z"
  }
}
```

---

### GET /api/v1/escrow

**Description:** Get escrow held for active orders  
**Authentication:** Bearer token required (vendor role)  
**Rate Limit:** 30 requests per hour

**Success Response (200):**

```json
{
  "success": true,
  "data": {
    "total_in_escrow": 2500000.00,
    "currency": "YER",
    "holdings": [
      {
        "order_id": "ord_001",
        "order_number": "YM-2026-001234",
        "amount": 290000.00,
        "status": "shipped",
        "expected_release": "2026-09-16",
        "vendor_commission": 14500.00
      }
    ]
  }
}
```

---

## 7. Cart

### GET /api/v1/cart

**Description:** Get current cart  
**Authentication:** Bearer token required  
**Rate Limit:** 60 requests per hour

**Success Response (200):**

```json
{
  "success": true,
  "data": {
    "id": "cart_abc123",
    "items": [
      {
        "id": "cart_item_001",
        "product": {
          "id": "prod_xyz789",
          "name": "Samsung Galaxy S24 Ultra",
          "thumbnail_url": "https://cdn.yemenmart.com/products/prod_xyz789/thumb.jpg",
          "in_stock": true
        },
        "variant": {
          "id": "var_001",
          "name": "256GB / Titanium Black"
        },
        "vendor": {
          "id": "vnd_abc123",
          "name": "Tech World Yemen"
        },
        "quantity": 1,
        "unit_price": 285000.00,
        "total": 285000.00,
        "max_quantity": 15,
        "added_at": "2026-09-12T18:00:00Z"
      },
      {
        "id": "cart_item_002",
        "product": {
          "id": "prod_abc123",
          "name": "USB-C Cable 2m",
          "thumbnail_url": "https://cdn.yemenmart.com/products/prod_abc123/thumb.jpg",
          "in_stock": true
        },
        "variant": null,
        "vendor": {
          "id": "vnd_def456",
          "name": "Accessories Hub"
        },
        "quantity": 2,
        "unit_price": 5000.00,
        "total": 10000.00,
        "max_quantity": 50,
        "added_at": "2026-09-12T18:05:00Z"
      }
    ],
    "summary": {
      "total_items": 3,
      "subtotal": 295000.00,
      "delivery_fee_estimate": 10000.00,
      "discount_estimate": 0,
      "total_estimate": 305000.00,
      "items_by_vendor": [
        {
          "vendor_id": "vnd_abc123",
          "vendor_name": "Tech World Yemen",
          "items_count": 1,
          "subtotal": 285000.00
        },
        {
          "vendor_id": "vnd_def456",
          "vendor_name": "Accessories Hub",
          "items_count": 2,
          "subtotal": 10000.00
        }
      ]
    },
    "coupon": null,
    "updated_at": "2026-09-12T18:05:00Z"
  }
}
```

---

### POST /api/v1/cart/items

**Description:** Add item to cart  
**Authentication:** Bearer token required  
**Rate Limit:** 60 requests per hour

**Request Body:**

```json
{
  "product_id": "prod_xyz789",
  "variant_id": "var_001",
  "quantity": 1
}
```

**Validation Rules:**
- `product_id`: Required
- `variant_id`: Required if product has variants
- `quantity`: Required, positive integer

**Business Rules:**
- Maximum 50 unique items in cart
- Maximum quantity per item: 10
- Validates stock availability
- Merges if same product+variant already in cart

**Success Response (201):**

```json
{
  "success": true,
  "message": "Item added to cart",
  "data": {
    "cart_item_id": "cart_item_001",
    "cart_summary": {
      "total_items": 3,
      "subtotal": 295000.00
    }
  }
}
```

**Error Responses:**

| Status | Code | Message |
|--------|------|---------|
| 422 | OUT_OF_STOCK | Product is out of stock |
| 422 | EXCEEDS_STOCK | Requested quantity exceeds available stock |
| 422 | CART_FULL | Cart cannot hold more unique items |

---

### PUT /api/v1/cart/items/:id

**Description:** Update cart item quantity  
**Authentication:** Bearer token required  
**Rate Limit:** 60 requests per hour

**Path Parameters:**
- `id`: Cart item ID

**Request Body:**

```json
{
  "quantity": 3
}
```

**Success Response (200):**

```json
{
  "success": true,
  "message": "Cart updated",
  "data": {
    "cart_item_id": "cart_item_001",
    "quantity": 3,
    "total": 855000.00,
    "cart_summary": {
      "total_items": 5,
      "subtotal": 865000.00
    }
  }
}
```

---

### DELETE /api/v1/cart/items/:id

**Description:** Remove item from cart  
**Authentication:** Bearer token required  
**Rate Limit:** 60 requests per hour

**Path Parameters:**
- `id`: Cart item ID

**Success Response (200):**

```json
{
  "success": true,
  "message": "Item removed from cart",
  "data": {
    "cart_summary": {
      "total_items": 2,
      "subtotal": 10000.00
    }
  }
}
```

---

### POST /api/v1/cart/coupon

**Description:** Apply coupon to cart  
**Authentication:** Bearer token required  
**Rate Limit:** 10 requests per hour

**Request Body:**

```json
{
  "coupon_code": "SAVE10"
}
```

**Validation Rules:**
- `coupon_code`: Required, alphanumeric

**Business Rules:**
- One coupon per cart
- Validates minimum order amount
- Validates category/product restrictions
- Validates usage limits

**Success Response (200):**

```json
{
  "success": true,
  "message": "Coupon applied successfully",
  "data": {
    "coupon_code": "SAVE10",
    "discount_type": "percentage",
    "discount_value": 10,
    "max_discount": 50000.00,
    "applied_discount": 29500.00,
    "cart_summary": {
      "subtotal": 295000.00,
      "discount": 29500.00,
      "total": 265500.00
    }
  }
}
```

**Error Responses:**

| Status | Code | Message |
|--------|------|---------|
| 404 | NOT_FOUND | Coupon not found |
| 422 | EXPIRED | Coupon has expired |
| 422 | MINIMUM_NOT_MET | Minimum order amount not met |
| 422 | USAGE_EXCEEDED | Coupon usage limit reached |
| 422 | NOT_APPLICABLE | Coupon not applicable to cart items |

---

## 8. Reviews

### GET /api/v1/reviews

**Description:** List reviews by current user  
**Authentication:** Bearer token required  
**Rate Limit:** 60 requests per hour

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `page` | integer | 1 | Page number |
| `per_page` | integer | 20 | Items per page |

**Success Response (200):**

```json
{
  "success": true,
  "data": [
    {
      "id": "rev_001",
      "product": {
        "id": "prod_xyz789",
        "name": "Samsung Galaxy S24 Ultra",
        "thumbnail_url": "https://cdn.yemenmart.com/products/prod_xyz789/thumb.jpg"
      },
      "rating": 5,
      "title": "Excellent phone!",
      "comment": "Best phone I've ever used.",
      "images": ["https://cdn.yemenmart.com/reviews/rev_001/img1.jpg"],
      "helpful_count": 23,
      "vendor_reply": {
        "comment": "Thank you for your wonderful review!",
        "replied_at": "2026-08-15T12:00:00Z"
      },
      "created_at": "2026-08-10T14:30:00Z",
      "can_edit": true
    }
  ],
  "pagination": {
    "current_page": 1,
    "per_page": 20,
    "total_pages": 3,
    "total_items": 45
  }
}
```

---

### POST /api/v1/reviews

**Description:** Create a review  
**Authentication:** Bearer token required  
**Rate Limit:** 10 requests per hour

**Request Body:**

```json
{
  "product_id": "prod_xyz789",
  "order_id": "ord_001",
  "rating": 5,
  "title": "Excellent phone!",
  "comment": "Best phone I've ever used. Camera quality is amazing and battery lasts all day.",
  "images": [
    "https://storage.yemenmart.com/reviews/user/img1.jpg"
  ]
}
```

**Validation Rules:**
- `product_id`: Required
- `order_id`: Required, must be delivered order containing the product
- `rating`: Required, integer 1-5
- `title`: Optional, max 100 characters
- `comment`: Optional, max 2000 characters
- `images`: Optional, max 5 images

**Business Rules:**
- Can only review products from delivered orders
- One review per product per order
- Can edit within 30 days
- Images uploaded separately, URLs passed here

**Success Response (201):**

```json
{
  "success": true,
  "message": "Review submitted successfully",
  "data": {
    "review_id": "rev_002",
    "rating": 5,
    "status": "published",
    "created_at": "2026-09-13T14:00:00Z"
  }
}
```

---

### PUT /api/v1/reviews/:id

**Description:** Update a review  
**Authentication:** Bearer token required  
**Rate Limit:** 10 requests per hour

**Path Parameters:**
- `id`: Review ID

**Request Body:**

```json
{
  "rating": 4,
  "title": "Updated title",
  "comment": "Updated comment text"
}
```

**Business Rules:**
- Can only edit own reviews
- Must be within 30 days of creation

**Success Response (200):**

```json
{
  "success": true,
  "message": "Review updated successfully",
  "data": {
    "review_id": "rev_002",
    "rating": 4,
    "updated_at": "2026-09-13T15:00:00Z"
  }
}
```

---

### DELETE /api/v1/reviews/:id

**Description:** Delete a review  
**Authentication:** Bearer token required  
**Rate Limit:** 10 requests per hour

**Path Parameters:**
- `id`: Review ID

**Success Response (200):**

```json
{
  "success": true,
  "message": "Review deleted successfully"
}
```

---

## 9. Coupons

### POST /api/v1/coupons/validate

**Description:** Validate a coupon code  
**Authentication:** Bearer token required  
**Rate Limit:** 20 requests per hour

**Request Body:**

```json
{
  "code": "SAVE10",
  "cart_total": 295000.00,
  "items": [
    {
      "product_id": "prod_xyz789",
      "quantity": 1,
      "price": 285000.00
    }
  ]
}
```

**Success Response (200):**

```json
{
  "success": true,
  "data": {
    "valid": true,
    "coupon_code": "SAVE10",
    "discount_type": "percentage",
    "discount_value": 10,
    "max_discount": 50000.00,
    "calculated_discount": 29500.00,
    "minimum_order": 100000.00,
    "valid_until": "2026-12-31T23:59:59Z",
    "usage_remaining": 45,
    "description": "10% off on orders above 100,000 YER"
  }
}
```

**Error Response (200):**

```json
{
  "success": true,
  "data": {
    "valid": false,
    "error_code": "EXPIRED",
    "error_message": "Coupon has expired"
  }
}
```

---

### GET /api/v1/coupons

**Description:** List available coupons for customer  
**Authentication:** Bearer token required  
**Rate Limit:** 30 requests per hour

**Success Response (200):**

```json
{
  "success": true,
  "data": [
    {
      "id": "coup_001",
      "code": "WELCOME15",
      "discount_type": "percentage",
      "discount_value": 15,
      "max_discount": 75000.00,
      "minimum_order": 50000.00,
      "description": "15% off for new customers",
      "valid_until": "2026-12-31T23:59:59Z",
      "usage_limit": 1,
      "usage_remaining": 1,
      "applicable_categories": ["cat_electronics", "cat_fashion"],
      "terms": "Valid for first order only"
    }
  ]
}
```

---

## 10. Support Tickets

### GET /api/v1/tickets

**Description:** List support tickets  
**Authentication:** Bearer token required  
**Rate Limit:** 60 requests per hour

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `status` | string | `all` | Filter: `open`, `in_progress`, `resolved`, `closed`, `all` |
| `page` | integer | 1 | Page number |
| `per_page` | integer | 20 | Items per page |

**Success Response (200):**

```json
{
  "success": true,
  "data": [
    {
      "id": "tkt_001",
      "subject": "Order not received",
      "category": "order_issue",
      "status": "open",
      "priority": "high",
      "order_id": "ord_001",
      "last_message": {
        "sender": "customer",
        "message": "My order was supposed to arrive yesterday but I haven't received it.",
        "created_at": "2026-09-13T10:00:00Z"
      },
      "unread_count": 1,
      "created_at": "2026-09-12T15:00:00Z",
      "updated_at": "2026-09-13T10:00:00Z"
    }
  ],
  "pagination": {
    "current_page": 1,
    "per_page": 20,
    "total_pages": 2,
    "total_items": 15
  }
}
```

---

### POST /api/v1/tickets

**Description:** Create a support ticket  
**Authentication:** Bearer token required  
**Rate Limit:** 10 requests per hour

**Request Body:**

```json
{
  "subject": "Order not received",
  "category": "order_issue",
  "priority": "high",
  "order_id": "ord_001",
  "message": "My order was supposed to arrive yesterday but I haven't received it yet. The tracking shows it's out for delivery.",
  "attachments": [
    "https://storage.yemenmart.com/tickets/img1.jpg"
  ]
}
```

**Validation Rules:**
- `subject`: Required, 5-200 characters
- `category`: Required, enum: `order_issue`, `payment_issue`, `product_quality`, `delivery_issue`, `account_issue`, `refund`, `other`
- `priority`: Required, enum: `low`, `medium`, `high`, `urgent`
- `order_id`: Optional, required if category is order/payment/refund related
- `message`: Required, 10-5000 characters
- `attachments`: Optional, max 5 files

**Business Rules:**
- One ticket per issue
- Vendor-related issues forwarded to vendor
- Auto-escalation after 48 hours without response

**Success Response (201):**

```json
{
  "success": true,
  "message": "Support ticket created",
  "data": {
    "ticket_id": "tkt_002",
    "status": "open",
    "estimated_response": "4-24 hours",
    "created_at": "2026-09-13T12:00:00Z"
  }
}
```

---

### POST /api/v1/tickets/:id/messages

**Description:** Send message in ticket  
**Authentication:** Bearer token required  
**Rate Limit:** 30 requests per hour

**Path Parameters:**
- `id`: Ticket ID

**Request Body:**

```json
{
  "message": "I also tried contacting the courier but no response.",
  "attachments": [
    "https://storage.yemenmart.com/tickets/img2.jpg"
  ]
}
```

**Success Response (201):**

```json
{
  "success": true,
  "message": "Message sent",
  "data": {
    "message_id": "msg_002",
    "ticket_status": "open",
    "created_at": "2026-09-13T14:00:00Z"
  }
}
```

---

### GET /api/v1/tickets/:id

**Description:** Get ticket details with messages  
**Authentication:** Bearer token required  
**Rate Limit:** 60 requests per hour

**Path Parameters:**
- `id`: Ticket ID

**Success Response (200):**

```json
{
  "success": true,
  "data": {
    "id": "tkt_001",
    "subject": "Order not received",
    "category": "order_issue",
    "status": "open",
    "priority": "high",
    "order": {
      "id": "ord_001",
      "order_number": "YM-2026-001234"
    },
    "messages": [
      {
        "id": "msg_001",
        "sender": {
          "type": "customer",
          "id": "usr_abc123",
          "name": "Mohammed Ali"
        },
        "message": "My order was supposed to arrive yesterday but I haven't received it yet.",
        "attachments": [],
        "created_at": "2026-09-12T15:00:00Z"
      },
      {
        "id": "msg_002",
        "sender": {
          "type": "support",
          "id": "usr_support_001",
          "name": "YemenMart Support"
        },
        "message": "We're looking into this. Let me check with the courier.",
        "attachments": [],
        "created_at": "2026-09-12T16:00:00Z"
      }
    ],
    "created_at": "2026-09-12T15:00:00Z",
    "updated_at": "2026-09-12T16:00:00Z"
  }
}
```

---

## 11. Admin

### GET /api/v1/admin/dashboard

**Description:** Get admin dashboard overview  
**Authentication:** Bearer token required (admin role)  
**Rate Limit:** 30 requests per hour

**Success Response (200):**

```json
{
  "success": true,
  "data": {
    "period": "2026-09",
    "overview": {
      "total_users": 125600,
      "new_users_today": 245,
      "total_vendors": 3450,
      "active_vendors": 2890,
      "total_products": 89000,
      "total_orders": 456789,
      "orders_today": 1234,
      "revenue_today": 156000000.00,
      "revenue_this_month": 2450000000.00
    },
    "orders_by_status": {
      "pending": 234,
      "confirmed": 567,
      "processing": 123,
      "shipped": 890,
      "delivered": 12340,
      "cancelled": 345,
      "returned": 89
    },
    "top_categories": [
      { "id": "cat_electronics", "name": "Electronics", "orders": 45678, "revenue": 890000000.00 },
      { "id": "cat_fashion", "name": "Fashion", "orders": 34567, "revenue": 456000000.00 }
    ],
    "top_vendors": [
      { "id": "vnd_001", "name": "Tech World", "orders": 1234, "revenue": 234000000.00 }
    ],
    "financial_summary": {
      "total_wallet_balance": 8900000000.00,
      "total_escrow": 1230000000.00,
      "total_commission_earned": 345000000.00,
      "pending_payouts": 567000000.00
    },
    "support_summary": {
      "open_tickets": 123,
      "avg_response_time": "2.5 hours",
      "satisfaction_rate": 0.92
    }
  }
}
```

---

### GET /api/v1/admin/customers

**Description:** List all customers  
**Authentication:** Bearer token required (admin role)  
**Rate Limit:** 60 requests per hour

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `search` | string | - | Search by name, phone, email |
| `status` | string | `all` | Filter: `active`, `suspended`, `all` |
| `page` | integer | 1 | Page number |
| `per_page` | integer | 20 | Items per page |

**Success Response (200):**

```json
{
  "success": true,
  "data": [
    {
      "id": "usr_abc123",
      "full_name": "Mohammed Ali",
      "phone": "+967771234567",
      "email": "mohammed@example.com",
      "status": "active",
      "is_verified": true,
      "wallet_balance": 1500000.00,
      "total_orders": 56,
      "total_spent": 8900000.00,
      "loyalty_points": 320,
      "created_at": "2026-01-15T10:30:00Z",
      "last_active": "2026-09-13T08:00:00Z"
    }
  ],
  "pagination": {
    "current_page": 1,
    "per_page": 20,
    "total_pages": 6280,
    "total_items": 125600
  }
}
```

---

### GET /api/v1/admin/vendors

**Description:** List all vendors  
**Authentication:** Bearer token required (admin role)  
**Rate Limit:** 60 requests per hour

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `search` | string | - | Search by store name, owner name |
| `status` | string | `all` | Filter: `active`, `pending_kyc`, `suspended`, `all` |
| `page` | integer | 1 | Page number |
| `per_page` | integer | 20 | Items per page |

**Success Response (200):**

```json
{
  "success": true,
  "data": [
    {
      "id": "vnd_abc123",
      "store_name": "Tech World Yemen",
      "owner": {
        "id": "usr_xyz789",
        "full_name": "Ali Hassan",
        "phone": "+967779876543"
      },
      "kyc_status": "approved",
      "status": "active",
      "rating": 4.8,
      "total_products": 42,
      "total_orders": 1567,
      "total_revenue": 234000000.00,
      "commission_rate": 0.05,
      "joined_date": "2025-06-15",
      "last_active": "2026-09-13T10:00:00Z"
    }
  ],
  "pagination": {
    "current_page": 1,
    "per_page": 20,
    "total_pages": 173,
    "total_items": 3450
  }
}
```

---

### PUT /api/v1/admin/vendors/:id/kyc

**Description:** Update vendor KYC status  
**Authentication:** Bearer token required (admin role)  
**Rate Limit:** 30 requests per hour

**Path Parameters:**
- `id`: Vendor ID

**Request Body:**

```json
{
  "status": "approved",
  "notes": "Documents verified successfully",
  "rejection_reason": null
}
```

**Validation Rules:**
- `status`: Required, enum: `approved`, `rejected`
- `notes`: Optional, max 500 characters
- `rejection_reason`: Required if status is rejected

**Business Rules:**
- Approving KYC activates vendor store
- Rejection sends notification with reason
- Vendor can resubmit after rejection

**Success Response (200):**

```json
{
  "success": true,
  "message": "KYC status updated",
  "data": {
    "vendor_id": "vnd_abc123",
    "kyc_status": "approved",
    "store_status": "active",
    "updated_at": "2026-09-13T14:00:00Z"
  }
}
```

---

### GET /api/v1/admin/orders

**Description:** List all orders  
**Authentication:** Bearer token required (admin role)  
**Rate Limit:** 60 requests per hour

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `status` | string | `all` | Filter by status |
| `vendor_id` | string | - | Filter by vendor |
| `customer_id` | string | - | Filter by customer |
| `date_from` | string | - | ISO date |
| `date_to` | string | - | ISO date |
| `min_amount` | number | - | Minimum order amount |
| `max_amount` | number | - | Maximum order amount |
| `page` | integer | 1 | Page number |
| `per_page` | integer | 20 | Items per page |

**Success Response (200):** Returns same format as vendor orders endpoint

---

### GET /api/v1/admin/finance/commissions

**Description:** Get commission reports  
**Authentication:** Bearer token required (admin role)  
**Rate Limit:** 30 requests per hour

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `period` | string | `monthly` | Filter: `daily`, `weekly`, `monthly` |
| `vendor_id` | string | - | Filter by vendor |
| `date_from` | string | - | ISO date |
| `date_to` | string | - | ISO date |
| `page` | integer | 1 | Page number |

**Success Response (200):**

```json
{
  "success": true,
  "data": {
    "summary": {
      "total_sales": 15600000000.00,
      "total_commission": 780000000.00,
      "average_commission_rate": 0.05,
      "period": "2026-09"
    },
    "by_vendor": [
      {
        "vendor_id": "vnd_001",
        "vendor_name": "Tech World Yemen",
        "total_sales": 234000000.00,
        "commission": 11700000.00,
        "rate": 0.05,
        "orders_count": 1234,
        "status": "active"
      }
    ],
    "pagination": {
      "current_page": 1,
      "per_page": 20,
      "total_pages": 173,
      "total_items": 3450
    }
  }
}
```

---

### POST /api/v1/admin/coupons

**Description:** Create a coupon  
**Authentication:** Bearer token required (admin role)  
**Rate Limit:** 10 requests per hour

**Request Body:**

```json
{
  "code": "EID2026",
  "description": "Eid celebration discount",
  "discount_type": "percentage",
  "discount_value": 20,
  "max_discount": 100000.00,
  "minimum_order": 50000.00,
  "usage_limit": 1000,
  "per_user_limit": 1,
  "start_date": "2026-09-15T00:00:00Z",
  "end_date": "2026-09-30T23:59:59Z",
  "applicable_categories": ["cat_electronics", "cat_fashion"],
  "applicable_vendors": [],
  "is_active": true
}
```

**Validation Rules:**
- `code`: Required, unique, 4-20 alphanumeric characters
- `discount_type`: Required, enum: `percentage`, `fixed`
- `discount_value`: Required, positive
- `max_discount`: Required for percentage type
- `minimum_order`: Optional
- `start_date`: Required, must be future
- `end_date`: Required, must be after start_date

**Success Response (201):**

```json
{
  "success": true,
  "message": "Coupon created successfully",
  "data": {
    "coupon_id": "coup_002",
    "code": "EID2026",
    "is_active": true,
    "created_at": "2026-09-13T14:00:00Z"
  }
}
```

---

## 12. Notifications

### GET /api/v1/notifications

**Description:** Get user notifications  
**Authentication:** Bearer token required  
**Rate Limit:** 60 requests per hour

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `is_read` | boolean | - | Filter by read status |
| `type` | string | `all` | Filter: `order`, `promotion`, `system`, `wallet`, `all` |
| `page` | integer | 1 | Page number |
| `per_page` | integer | 20 | Items per page |

**Success Response (200):**

```json
{
  "success": true,
  "data": {
    "unread_count": 12,
    "notifications": [
      {
        "id": "notif_001",
        "type": "order",
        "title": "Order Shipped",
        "message": "Your order YM-2026-001234 has been shipped.",
        "data": {
          "order_id": "ord_001",
          "order_number": "YM-2026-001234",
          "tracking_number": "YE123456789"
        },
        "image_url": "https://cdn.yemenmart.com/notifications/order_shipped.png",
        "is_read": false,
        "action_url": "/orders/ord_001",
        "created_at": "2026-09-14T09:00:00Z"
      },
      {
        "id": "notif_002",
        "type": "promotion",
        "title": "Flash Sale - 50% Off Electronics",
        "message": "Hurry! Limited time offer on selected electronics.",
        "data": {
          "campaign_id": "camp_001",
          "category_id": "cat_electronics"
        },
        "image_url": "https://cdn.yemenmart.com/promotions/electronics_sale.jpg",
        "is_read": true,
        "action_url": "/categories/electronics",
        "created_at": "2026-09-13T10:00:00Z"
      }
    ],
    "pagination": {
      "current_page": 1,
      "per_page": 20,
      "total_pages": 5,
      "total_items": 89
    }
  }
}
```

---

### PUT /api/v1/notifications/:id/read

**Description:** Mark notification as read  
**Authentication:** Bearer token required  
**Rate Limit:** 120 requests per hour

**Path Parameters:**
- `id`: Notification ID

**Success Response (200):**

```json
{
  "success": true,
  "message": "Notification marked as read"
}
```

---

### PUT /api/v1/notifications/preferences

**Description:** Update notification preferences  
**Authentication:** Bearer token required  
**Rate Limit:** 10 requests per hour

**Request Body:**

```json
{
  "order_updates": {
    "push": true,
    "sms": true,
    "whatsapp": false
  },
  "promotions": {
    "push": true,
    "sms": false,
    "whatsapp": true
  },
  "wallet": {
    "push": true,
    "sms": true,
    "whatsapp": false
  },
  "system": {
    "push": true,
    "sms": false,
    "whatsapp": false
  },
  "quiet_hours": {
    "enabled": true,
    "start": "22:00",
    "end": "07:00",
    "timezone": "Asia/Aden"
  }
}
```

**Success Response (200):**

```json
{
  "success": true,
  "message": "Notification preferences updated",
  "data": {
    "updated_at": "2026-09-13T14:00:00Z"
  }
}
```

---

## 13. Delivery

### GET /api/v1/delivery/assignments

**Description:** List delivery assignments for driver  
**Authentication:** Bearer token required (driver role)  
**Rate Limit:** 60 requests per hour

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `status` | string | `all` | Filter: `assigned`, `picked_up`, `in_transit`, `delivered`, `all` |
| `page` | integer | 1 | Page number |
| `per_page` | integer | 20 | Items per page |

**Success Response (200):**

```json
{
  "success": true,
  "data": [
    {
      "id": "del_001",
      "order": {
        "id": "ord_001",
        "order_number": "YM-2026-001234"
      },
      "vendor": {
        "id": "vnd_abc123",
        "name": "Tech World Yemen",
        "address": "Al-Sabaeen Street, Shop 12",
        "phone": "+967779876543"
      },
      "customer": {
        "id": "usr_abc123",
        "name": "Mohammed Ali",
        "phone": "+967771234567"
      },
      "pickup_address": {
        "governorate": "Sana'a",
        "district": "Al-Sabaeen",
        "street": "Al-Sabaeen Street, Shop 12"
      },
      "delivery_address": {
        "governorate": "Sana'a",
        "district": "Al-Sabaeen",
        "street": "Al-Sabaeen Street, Building 45",
        "landmark": "Near Al-Sabaeen Park",
        "latitude": 15.3547,
        "longitude": 44.2066
      },
      "items_count": 1,
      "total_amount": 290000.00,
      "delivery_fee": 5000.00,
      "status": "assigned",
      "assigned_at": "2026-09-14T08:00:00Z",
      "estimated_delivery": "2026-09-14T18:00:00Z",
      "distance_km": 3.5
    }
  ],
  "summary": {
    "assigned": 3,
    "in_transit": 2,
    "delivered_today": 8,
    "earnings_today": 45000.00
  },
  "pagination": {
    "current_page": 1,
    "per_page": 20,
    "total_pages": 1,
    "total_items": 5
  }
}
```

---

### POST /api/v1/delivery/assignments/:id/confirm

**Description:** Confirm pickup of delivery  
**Authentication:** Bearer token required (driver role)  
**Rate Limit:** 30 requests per hour

**Path Parameters:**
- `id`: Delivery assignment ID

**Request Body:**

```json
{
  "action": "pickup",
  "notes": "Picked up from vendor"
}
```

**Validation Rules:**
- `action`: Required, enum: `pickup`, `in_transit`, `delivered`
- `notes`: Optional, max 500 characters

**Success Response (200):**

```json
{
  "success": true,
  "message": "Pickup confirmed",
  "data": {
    "delivery_id": "del_001",
    "status": "picked_up",
    "picked_up_at": "2026-09-14T10:00:00Z"
  }
}
```

---

### POST /api/v1/delivery/assignments/:id/proof

**Description:** Submit proof of delivery  
**Authentication:** Bearer token required (driver role)  
**Rate Limit:** 20 requests per hour

**Path Parameters:**
- `id`: Delivery assignment ID

**Request Body:**

```json
{
  "proof_type": "photo",
  "photo_url": "https://storage.yemenmart.com/delivery/del_001/proof.jpg",
  "signature_url": null,
  "recipient_name": "Mohammed Ali",
  "recipient_phone": "+967771234567",
  "notes": "Left at door as requested"
}
```

**Validation Rules:**
- `proof_type`: Required, enum: `photo`, `signature`, `pin`, `photo_and_signature`
- `photo_url`: Required if proof_type includes photo
- `signature_url`: Required if proof_type includes signature
- `recipient_name`: Required, name of person who received
- `recipient_phone`: Optional

**Business Rules:**
- Photo must show delivered package at location
- Triggers order status update to "delivered"
- Triggers vendor payout
- Auto-confirmation after 7 days if customer doesn't confirm

**Success Response (200):**

```json
{
  "success": true,
  "message": "Proof of delivery submitted",
  "data": {
    "delivery_id": "del_001",
    "status": "delivered",
    "delivered_at": "2026-09-14T14:00:00Z",
    "proof_url": "https://storage.yemenmart.com/delivery/del_001/proof.jpg",
    "delivery_fee_earned": 5000.00
  }
}
```

---

## Appendix A: Yemen Governorates Reference

| Code | Name (Arabic) | Name (English) |
|------|---------------|----------------|
| SA | صنعاء | Sana'a |
| AD | عدن | Aden |
| HU | تعز | Taiz |
| IM | إب | Ibb |
| LA | لحج | Lahij |
| BA | برع | Al Bayda |
| DA | دار سعد | Dhale |
| HA | حضرموت | Hadhramaut |
| SH | شبوة | Shabwah |
| MA | مأرب | Marib |
| JB | جبلة | Al Jawf |
| MR | المهرة | Al Mahrah |
| HD | حديدة | Al Hudaydah |
| MW | المحويت | Al Mahwit |
| AM | عمران | Amran |
| DH | ذمار | Dhamar |
| BD | بني دﻬplib | Bani Dhahban |
| RA | ريمه | Raymah |
| SI | صعده | Saada |
| TC | تعز | Taizz |
| WA | وادي عوض | Al Wadi |
| HR | الحديدة | Al Hudaydah |

---

## Appendix B: Currency Formatting

All monetary values are in **Yemeni Rial (YER)**:
- Format: `{amount}.00`
- Minimum order: varies by category
- COD minimum: 10,000 YER
- Maximum single transaction: 10,000,000 YER

---

## Appendix C: Delivery Status Flow

```
pending → confirmed → processing → shipped → in_transit → delivered
   ↓          ↓           ↓           ↓           ↓            ↓
cancelled  cancelled  cancelled   cancelled   cancelled    returned
```

---

## Appendix D: Payment Methods

| Method | Description | Processing Time | Fee |
|--------|-------------|-----------------|-----|
| `wallet` | YemenMart wallet | Instant | Free |
| `cod` | Cash on delivery | On delivery | Free |
| `fawri` | Fawri mobile money | Instant | 1.5% |
| `cash_card` | Cash Card | Instant | 1% |
| `bank_transfer` | Bank transfer | 1-24 hours | Free |
| `credit_card` | Credit/Debit card | Instant | 2.5% |

---

## Appendix E: Rate Limiting Summary

| Endpoint Category | Requests per Hour | Burst |
|-------------------|-------------------|-------|
| Auth (register/login) | 5 | 2/min |
| Auth (verify/refresh) | 20 | 5/min |
| Products (read) | 120 | 20/min |
| Cart/Orders | 60 | 10/min |
| Vendor operations | 30 | 5/min |
| Admin operations | 60 | 10/min |
| Wallet operations | 10 | 3/min |
| Notifications | 120 | 20/min |

---

## Appendix F: Webhook Events (Future)

| Event | Trigger |
|-------|---------|
| `order.created` | New order placed |
| `order.confirmed` | Vendor confirms order |
| `order.shipped` | Order handed to courier |
| `order.delivered` | Delivery confirmed |
| `order.cancelled` | Order cancelled |
| `payment.completed` | Payment processed |
| `payment.refunded` | Refund processed |
| `kyc.approved` | Vendor KYC approved |
| `kyc.rejected` | Vendor KYC rejected |
| `review.created` | New review posted |
| `ticket.created` | New support ticket |

---

*End of YemenMart API Specification v1.0*
