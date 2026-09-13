# User Flows — YemenMart

## 1. Overview

Key user journeys through the YemenMart platform. Each flow is designed for Arabic-first, mobile-optimized experience.

## 2. Customer Flows

### 2.1 Registration Flow

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│  Enter   │───>│  Send    │───>│  Verify  │───>│  Account │
│  Phone   │    │  OTP     │    │  Code    │    │ Created  │
└──────────┘    └──────────┘    └──────────┘    └──────────┘
     │              │               │                │
     │              │               │                ▼
     │              │               │         ┌──────────┐
     │              │               │         │  Create  │
     │              │               │         │ Password │
     │              │               │         └────┬─────┘
     │              │               │              │
     │              │               │              ▼
     │              │               │         ┌──────────┐
     │              │               │         │ Profile  │
     │              │               │         │  Setup   │
     │              │               │         └────┬─────┘
     │              │               │              │
     │              │               │              ▼
     │              │               │         ┌──────────┐
     │              │               │         │  Home    │
     │              │               │         │  Page    │
     │              │               │         └──────────┘
```

**Steps:**
1. User enters phone number (Yemeni format: +967XXXXXXXXX)
2. System sends 6-digit OTP via SMS (Telesom/Sabafon)
3. User enters OTP within 5 minutes
4. System validates OTP (3 attempts max)
5. User creates password (min 8 chars, complexity required)
6. User completes profile (name, email optional)
7. Wallet automatically created (0 YER balance)
8. Redirect to home page

### 2.2 Product Discovery Flow

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│  Home    │───>│  Search  │───>│  Results │───>│  Product │
│  Page    │    │  /Filter │    │  List    │    │  Detail  │
└──────────┘    └──────────┘    └──────────┘    └────┬─────┘
     │              │               │                 │
     │              │               │                 ▼
     │              │               │           ┌──────────┐
     │              │               │           │  Add to  │
     │              │               │           │  Cart    │
     │              │               │           └────┬─────┘
     │              │               │                │
     │              │               │                ▼
     │              │               │           ┌──────────┐
     │              │               │           │ Continue │
     │              │               │           │ Shopping │
     │              │               │           └──────────┘
```

**Steps:**
1. User browses homepage (featured products, categories, promotions)
2. User searches by keyword or filters by category/price/brand
3. Elasticsearch returns relevant results with Arabic text support
4. User views product detail (images, description, variants, reviews)
5. User selects variant (size, color, etc.)
6. User adds to cart
7. Cart badge updates with item count

### 2.3 Checkout Flow

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│  Review  │───>│  Login/  │───>│  Address │───>│  Payment │
│  Cart    │    │ Register │    │  Entry   │    │  Select  │
└──────────┘    └──────────┘    └──────────┘    └────┬─────┘
     │              │               │                 │
     │              │               │                 ▼
     │              │               │           ┌──────────┐
     │              │               │           │  Wallet  │
     │              │               │           │  Pay /   │
     │              │               │           │  COD     │
     │              │               │           └────┬─────┘
     │              │               │                │
     │              │               │                ▼
     │              │               │           ┌──────────┐
     │              │               │           │  Order   │
     │              │               │           │Confirmed │
     │              │               │           └──────────┘
```

**Steps:**
1. User reviews cart (items, quantities, prices, subtotals)
2. System validates stock availability
3. If guest: forced registration via phone OTP
4. If registered: verify session
5. User enters/selects delivery address
6. System calculates shipping cost per vendor
7. User selects payment method (Wallet or COD if enabled)
8. If wallet: check balance, hold escrow
9. System splits order into sub-orders per vendor
10. Order confirmation with tracking info

### 2.4 Order Tracking Flow

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│  Order   │───>│  Order   │───>│  Status  │───>│ Delivery │
│  History │    │  Detail  │    │  Updates │    │Received  │
└──────────┘    └──────────┘    └──────────┘    └────┬─────┘
     │              │               │                 │
     │              │               │                 ▼
     │              │               │           ┌──────────┐
     │              │               │           │  Enter   │
     │              │               │           │Delivery  │
     │              │               │           │  Code    │
     │              │               │           └────┬─────┘
     │              │               │                │
     │              │               │                ▼
     │              │               │           ┌──────────┐
     │              │               │           │  Rate &  │
     │              │               │           │  Review  │
     │              │               │           └──────────┘
```

**Steps:**
1. User views order history (sorted by date, filterable)
2. User selects order for detail view
3. System shows sub-order status per vendor
4. Real-time tracking via delivery provider integration
5. Customer receives delivery code via SMS
6. Driver collects code on delivery
7. Order marked as delivered
8. User prompted to rate products

## 3. Vendor Flows

### 3.1 Vendor Registration Flow

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│  Phone   │───>│  KYC     │───>│  Store   │───>│  Store   │
│  Verify  │    │  Upload  │    │  Name    │    │Template  │
└──────────┘    └──────────┘    └──────────┘    └────┬─────┘
     │              │               │                 │
     │              │               │                 ▼
     │              │               │           ┌──────────┐
     │              │               │           │  Custom  │
     │              │               │           │ Branding │
     │              │               │           └────┬─────┘
     │              │               │                │
     │              │               │                ▼
     │              │               │           ┌──────────┐
     │              │               │           │  Admin   │
     │              │               │           │  Review  │
     │              │               │           └────┬─────┘
     │              │               │                │
     │              │               │                ▼
     │              │               │           ┌──────────┐
     │              │               │           │  Store   │
     │              │               │           │  Live    │
     │              │               │           └──────────┘
```

**Steps:**
1. Vendor registers with phone (OTP verification)
2. Vendor uploads KYC documents (trade license, ID)
3. Admin reviews KYC (approve/reject)
4. Vendor selects store template
5. Vendor customizes branding (logo, colors, description)
6. Admin approves store activation
7. Store goes live on marketplace

### 3.2 Product Listing Flow

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│  Add     │───>│  Enter   │───>│  Add     │───>│  Set     │
│  Product │    │  Details │    │  Images  │    │  Price   │
└──────────┘    └──────────┘    └──────────┘    └────┬─────┘
     │              │               │                 │
     │              │               │                 ▼
     │              │               │           ┌──────────┐
     │              │               │           │  Add     │
     │              │               │           │ Variants │
     │              │               │           └────┬─────┘
     │              │               │                │
     │              │               │                ▼
     │              │               │           ┌──────────┐
     │              │               │           │  Set     │
     │              │               │           │  Stock   │
     │              │               │           └────┬─────┘
     │              │               │                │
     │              │               │                ▼
     │              │               │           ┌──────────┐
     │              │               │           │ Publish  │
     │              │               │           │  Live    │
     │              │               │           └──────────┘
```

**Steps:**
1. Vendor clicks "Add Product"
2. Vendor enters product name (Arabic + English), description, category
3. Vendor uploads images (up to 10, max 5MB each)
4. Vendor sets base price (YER)
5. Vendor adds variants (size, color, material)
6. Vendor sets stock per variant
7. Product published to marketplace

### 3.3 Order Management Flow

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│  Order   │───>│  Accept/ │───>│  Pack    │───>│  Ship    │
│  Received│    │  Reject  │    │  Order   │    │  Order   │
└──────────┘    └──────────┘    └──────────┘    └────┬─────┘
     │              │               │                 │
     │              │               │                 ▼
     │              │               │           ┌──────────┐
     │              │               │           │  Update  │
     │              │               │           │ Tracking │
     │              │               │           └────┬─────┘
     │              │               │                │
     │              │               │                ▼
     │              │               │           ┌──────────┐
     │              │               │           │  Delivered│
     │              │               │           │  / Return │
     │              │               │           └──────────┘
```

**Steps:**
1. Vendor receives new order notification
2. Vendor reviews order details
3. Vendor accepts or rejects (with reason)
4. If accepted: vendor packs items
5. Vendor requests delivery pickup
6. Delivery provider assigns driver
7. Vendor updates tracking status
8. Customer receives delivery

## 4. Admin Flows

### 4.1 KYC Approval Flow

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│  KYC     │───>│  Review  │───>│  Approve │───>│  Vendor  │
│  Pending │    │  Docs    │    │ /Reject  │    │Notified  │
└──────────┘    └──────────┘    └──────────┘    └──────────┘
```

### 4.2 Vendor Payout Flow

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│  Payout  │───>│  Review  │───>│  Approve │───>│  Transfer│
│  Request │    │  Amount  │    │  Payout  │    │  Sent    │
└──────────┘    └──────────┘    └──────────┘    └──────────┘
```

## 5. Flow Metrics

| Flow | Target Time | Success Rate |
|------|-------------|--------------|
| Registration | < 2 minutes | 95% |
| Product Search | < 5 seconds | 90% |
| Checkout | < 3 minutes | 85% |
| Order Tracking | < 10 seconds | 95% |
| Vendor Registration | < 10 minutes | 90% |
| Product Listing | < 5 minutes | 85% |
| Order Processing | < 5 minutes | 95% |

## 6. Related Files

| File | Description |
|------|-------------|
| `design-system.md` | Design tokens and components |
| `wireframes/index.md` | Wireframe descriptions |
| `01-business-analysis/use-cases/customer-use-cases.md` | Customer use cases |
| `01-business-analysis/use-cases/vendor-use-cases.md` | Vendor use cases |
| `01-business-analysis/use-cases/admin-use-cases.md` | Admin use cases |

---

*Document Version: 1.0 | Last Updated: 2026-09-13 | Classification: Confidential*
