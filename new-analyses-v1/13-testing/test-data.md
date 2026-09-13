# Test Data Management — YemenMart v2

**Document Version:** 1.0
**Date:** 2026-09-13
**Scope:** Test environment data requirements, seed data, and masking strategy

---

## 1. Test Environment Setup

### 1.1 Environment Architecture

| Component | Test Environment | Configuration |
|-----------|-----------------|---------------|
| API Server | `api-test.yemenmart.com` | 2 instances, t3.medium |
| PostgreSQL | RDS db.t3.medium | Single instance, 50GB |
| Redis | ElastiCache t3.small | Single node, 2GB |
| Elasticsearch | t3.small.elasticsearch | Single node, 20GB |
| RabbitMQ | AmazonMQ t3.small | Single node |
| S3 | yemenmart-test-uploads | Versioning enabled |
| CDN | CloudFront (test distribution) | Origin: test ALB |

### 1.2 Environment Variables

```
NODE_ENV=test
DATABASE_URL=postgresql://ym_test:***@test-db.yemenmart.com:5432/yemenmart_test
REDIS_URL=redis://test-cache.yemenmart.com:6379
ELASTICSEARCH_URL=https://test-es.yemenmart.com:9200
RABBITMQ_URL=amqp://test-mq.yemenmart.com:5672
JWT_SECRET=test_jwt_secret_key_not_for_production
STRIPE_SECRET_KEY=sk_test_...
SMS_PROVIDER=test
EMAIL_PROVIDER=test
S3_BUCKET=yemenmart-test-uploads
```

### 1.3 Setup Commands

```bash
# Reset test database
npm run db:reset:test

# Seed test data
npm run db:seed:test

# Reindex Elasticsearch
npm run search:reindex:test

# Flush Redis cache
npm run cache:flush:test

# Run full setup
npm run test:setup
```

---

## 2. Test User Accounts

### 2.1 Admin Users

| Email | Password | Role | 2FA | Status | Notes |
|-------|----------|------|-----|--------|-------|
| `superadmin@yemenmart.test` | `SuperAdmin!2026` | super_admin | Enabled | active | Full system access |
| `admin@yemenmart.test` | `Admin!2026` | admin | Enabled | active | Standard admin |
| `finance@yemenmart.test` | `Finance!2026` | admin | Enabled | active | Finance role |
| `support@yemenmart.test` | `Support!2026` | admin | Disabled | active | Customer support |
| `pending-admin@yemenmart.test` | `Pending!2026` | admin | Disabled | pending_verification | Awaiting verification |

### 2.2 Vendor Users

| Email | Password | Business Name | Status | Rating | Notes |
|-------|----------|---------------|--------|--------|-------|
| `vendor1@yemenmart.test` | `Vendor1!2026` | Tech Store Yemen | approved | 4.7 | Electronics vendor |
| `vendor2@yemenmart.test` | `Vendor2!2026` | Fashion House Sana'a | approved | 4.3 | Clothing vendor |
| `vendor3@yemenmart.test` | `Vendor3!2026` | Fresh Market | approved | 4.8 | Grocery vendor |
| `vendor4@yemenmart.test` | `Vendor4!2026` | Home Essentials | approved | 4.1 | Home goods vendor |
| `vendor5@yemenmart.test` | `Vendor5!2026` | Book World | approved | 4.5 | Books vendor |
| `pending-vendor@yemenmart.test` | `Pending!2026` | New Shop | pending_review | — | Awaiting approval |
| `suspended-vendor@yemenmart.test` | `Suspended!2026` | Suspended Store | suspended | 2.1 | Suspended vendor |
| `rejected-vendor@yemenmart.test` | `Rejected!2026` | Rejected Shop | rejected | — | Rejected application |

### 2.3 Customer Users

| Email | Password | Name | Status | Wallet Balance | Notes |
|-------|----------|------|--------|----------------|-------|
| `customer1@yemenmart.test` | `Customer1!2026` | Mohammed Al-Sanabani | active | 250,000 YER | Active customer |
| `customer2@yemenmart.test` | `Customer2!2026` | Ahmed Al-Bahr | active | 100,000 YER | Repeat customer |
| `customer3@yemenmart.test` | `Customer3!2026` | Fatima Hassan | active | 50,000 YER | New customer |
| `customer4@yemenmart.test` | `Customer4!2026` | Omar Al-Dhaher | active | 500,000 YER | High-value customer |
| `customer5@yemenmart.test` | `Customer5!2026` | Nour Abdullah | active | 0 YER | Zero balance |
| `new-customer@yemenmart.test` | `NewCustomer!2026` | New User | pending_verification | 0 YER | Unverified email |
| `suspended-customer@yemenmart.test` | `Suspended!2026` | Bad Actor | suspended | 0 YER | Suspended account |
| `banned-customer@yemenmart.test` | `Banned!2026` | Fraud User | banned | 0 YER | Banned account |
| `2fa-customer@yemenmart.test` | `TwoFactor!2026` | Security User | active | 75,000 YER | 2FA enabled customer |

### 2.4 Rider Users

| Email | Password | Name | Status | Location | Notes |
|-------|----------|------|--------|----------|-------|
| `rider1@yemenmart.test` | `Rider1!2026` | Ali Mohammed | active | Sana'a | Full-time rider |
| `rider2@yemenmart.test` | `Rider2!2026` | Hassan Saleh | active | Aden | Full-time rider |
| `rider3@yemenmart.test` | `Rider3!2026` | Youssef Ahmed | active | Sana'a | Part-time rider |
| `offline-rider@yemenmart.test` | `Offline!2026` | Offline Rider | active | Sana'a | Currently offline |
| `pending-rider@yemenmart.test` | `Pending!2026` | New Rider | pending_verification | — | Awaiting approval |

---

## 3. Test Vendor Profiles

### 3.1 Vendor Details

| Vendor ID | Business Name | Type | Governorate | CR Number | Commission Rate |
|-----------|---------------|------|-------------|-----------|-----------------|
| `vnd_tech01` | Tech Store Yemen | electronics | Sana'a | CR-1001 | 8% |
| `vnd_fashion01` | Fashion House Sana'a | clothing | Sana'a | CR-1002 | 10% |
| `vnd_fresh01` | Fresh Market | groceries | Sana'a | CR-1003 | 5% |
| `vnd_home01` | Home Essentials | home_goods | Aden | CR-1004 | 10% |
| `vnd_book01` | Book World | books | Sana'a | CR-1005 | 7% |

### 3.2 Vendor Bank Accounts

| Vendor | Bank Name | Account Number | IBAN | Status |
|--------|-----------|----------------|------|--------|
| vnd_tech01 | Yemeni Gulf Bank | 1234567890 | YE01 YB00 1234 5678 9012 34 | verified |
| vnd_fashion01 | Saba Bank | 9876543210 | YE01 SB00 9876 5432 1098 76 | verified |
| vnd_fresh01 | Tadhamon Bank | 5555666677 | YE01 TD00 5555 6666 7788 99 | verified |
| vnd_home01 | Ahli Bank | 1111222233 | YE01 AB00 1111 2222 3344 55 | pending |
| vnd_book01 | Yemen Commercial Bank | 4444555566 | YE01 YC00 4444 5555 6677 88 | verified |

---

## 4. Test Product Catalog

### 4.1 Categories

| Category ID | Name | Name (AR) | Parent | Product Count |
|-------------|------|-----------|--------|---------------|
| `cat_electronics` | Electronics | إلكترونيات | — | 15 |
| `cat_smartphones` | Smartphones | هواتف ذكية | cat_electronics | 8 |
| `cat_laptops` | Laptops | أجهزة محمولة | cat_electronics | 4 |
| `cat_accessories` | Accessories | إكسسوارات | cat_electronics | 3 |
| `cat_clothing` | Clothing | ملابس | — | 12 |
| `cat_mens` | Men's Wear | ملابس رجالية | cat_clothing | 6 |
| `cat_womens` | Women's Wear | ملابس نسائية | cat_clothing | 6 |
| `cat_groceries` | Groceries | بقالة | — | 10 |
| `cat_fresh` | Fresh Produce | فواكه وخضار | cat_groceries | 5 |
| `cat_staples` | Staples | أساسيات | cat_groceries | 5 |
| `cat_home` | Home & Living | домашний | — | 8 |
| `cat_kitchen` | Kitchen | مطبخ | cat_home | 4 |
| `cat_decor` | Decor | ديكور | cat_home | 4 |
| `cat_books` | Books | كتب | — | 5 |
| `cat_textbooks` | Textbooks | كتب مدرسية | cat_books | 3 |
| `cat_novels` | Novels | روايات | cat_books | 2 |

### 4.2 Test Products — Electronics

| Product ID | Name | Name (AR) | Vendor | Price (YER) | Stock | Rating |
|------------|------|-----------|--------|-------------|-------|--------|
| `prd_sam_a15` | Samsung Galaxy A15 128GB | سامسونج جالكسي A15 | vnd_tech01 | 85,000 | 45 | 4.5 |
| `prd_sam_a25` | Samsung Galaxy A25 256GB | سامسونج جالكسي A25 | vnd_tech01 | 120,000 | 30 | 4.6 |
| `prd_iphone_15` | iPhone 15 128GB | آيفون 15 | vnd_tech01 | 350,000 | 20 | 4.8 |
| `prd_redmi_13` | Xiaomi Redmi 13C | شاومي ريدمي 13C | vnd_tech01 | 45,000 | 60 | 4.2 |
| `prd_macbook_air` | MacBook Air M3 | ماك بوك إير M3 | vnd_tech01 | 850,000 | 10 | 4.9 |
| `prd_lenovo_i5` | Lenovo IdeaPad i5 | لينوفو آيداباد i5 | vnd_tech01 | 320,000 | 15 | 4.4 |
| `prd_airpods` | Apple AirPods Pro 2 | أبل إيربودز برو 2 | vnd_tech01 | 95,000 | 50 | 4.7 |
| `prd_charger` | Samsung 25W Fast Charger | شاحن سامسونج سريع | vnd_tech01 | 12,000 | 100 | 4.3 |

### 4.3 Test Products — Clothing

| Product ID | Name | Name (AR) | Vendor | Price (YER) | Stock | Rating |
|------------|------|-----------|--------|-------------|-------|--------|
| `prd_shirt_m` | Men's Casual Shirt | قميص رجالي كاجوال | vnd_fashion01 | 15,000 | 80 | 4.1 |
| `prd_dress_w` | Women's Summer Dress | فستان صيفي نسائي | vnd_fashion01 | 25,000 | 40 | 4.4 |
| `prd_jeans_m` | Men's Classic Jeans | جينز رجالي كلاسيك | vnd_fashion01 | 18,000 | 60 | 4.2 |
| `prd_abaya` | Women's Abaya | عباية نسائية | vnd_fashion01 | 35,000 | 25 | 4.6 |
| `prd_sneakers` | Unisex Sneakers | حذاء رياضي | vnd_fashion01 | 22,000 | 70 | 4.3 |

### 4.4 Test Products — Groceries

| Product ID | Name | Name (AR) | Vendor | Price (YER) | Stock | Rating |
|------------|------|-----------|--------|-------------|-------|--------|
| `prd_rice_5kg` | Basmati Rice 5kg | أرز بسمتي 5 كجم | vnd_fresh01 | 12,000 | 200 | 4.5 |
| `prd_oil_1L` | Sunflower Oil 1L | زيت عباد الشمس 1 لتر | vnd_fresh01 | 3,500 | 150 | 4.3 |
| `prd_sugar_2kg` | White Sugar 2kg | سكر أبيض 2 كجم | vnd_fresh01 | 2,800 | 180 | 4.4 |
| `prd_flour_2kg` | Wheat Flour 2kg | طحين أبيض 2 كجم | vnd_fresh01 | 2,200 | 200 | 4.2 |
| `prd_tea` | Yemeni Tea 100g | شاي يمني 100 جرام | vnd_fresh01 | 4,500 | 100 | 4.7 |

### 4.5 Test Products — Home & Kitchen

| Product ID | Name | Name (AR) | Vendor | Price (YER) | Stock | Rating |
|------------|------|-----------|--------|-------------|-------|--------|
| `prd_cookware` | 5-Piece Cookware Set | طقم قدور 5 قطع | vnd_home01 | 45,000 | 20 | 4.5 |
| `prd_vacuum` | Robot Vacuum Cleaner | مكنسة كهربائية روبوت | vnd_home01 | 180,000 | 8 | 4.3 |
| `prd_lamp` | LED Floor Lamp | لمبة أرضية LED | vnd_home01 | 25,000 | 30 | 4.4 |
| `prd_pillow` | Memory Foam Pillow | وسادة ميموري فوم | vnd_home01 | 15,000 | 50 | 4.6 |

### 4.6 Test Products — Books

| Product ID | Name | Name (AR) | Vendor | Price (YER) | Stock | Rating |
|------------|------|-----------|--------|-------------|-------|--------|
| `prd_math_10` | Math Grade 10 | رياضيات الصف العاشر | vnd_book01 | 3,500 | 100 | 4.8 |
| `prd_arabic_novel` | Arabic Novel Collection | مجموعة روايات عربية | vnd_book01 | 8,000 | 40 | 4.5 |
| `prd_english_dict` | English-Arabic Dictionary | قاموس إنجليزي عربي | vnd_book01 | 12,000 | 25 | 4.6 |

### 4.7 Product Variants

| Product | Variant | SKU | Price | Stock |
|---------|---------|-----|-------|-------|
| prd_sam_a15 | Black / 128GB | SAM-A15-128-BLK | 85,000 | 20 |
| prd_sam_a15 | Blue / 128GB | SAM-A15-128-BLU | 85,000 | 15 |
| prd_sam_a15 | White / 128GB | SAM-A15-128-WHT | 85,000 | 10 |
| prd_iphone_15 | Black / 128GB | APH-15-128-BLK | 350,000 | 8 |
| prd_iphone_15 | Blue / 128GB | APH-15-128-BLU | 350,000 | 7 |
| prd_iphone_15 | Green / 128GB | APH-15-128-GRN | 350,000 | 5 |
| prd_shirt_m | S / White | SHRT-M-S-WHT | 15,000 | 20 |
| prd_shirt_m | M / White | SHRT-M-M-WHT | 15,000 | 25 |
| prd_shirt_m | L / White | SHRT-M-L-WHT | 15,000 | 20 |
| prd_shirt_m | XL / White | SHRT-M-XL-WHT | 15,000 | 15 |

### 4.8 Product Images (Test)

| Product | Image | URL | Primary |
|---------|-------|-----|---------|
| prd_sam_a15 | Front | `https://test-cdn.yemenmart.com/products/prd_sam_a15/front.webp` | Yes |
| prd_sam_a15 | Back | `https://test-cdn.yemenmart.com/products/prd_sam_a15/back.webp` | No |
| prd_sam_a15 | Side | `https://test-cdn.yemenmart.com/products/prd_sam_a15/side.webp` | No |
| prd_iphone_15 | Front | `https://test-cdn.yemenmart.com/products/prd_iphone_15/front.webp` | Yes |
| prd_iphone_15 | Box | `https://test-cdn.yemenmart.com/products/prd_iphone_15/box.webp` | No |

---

## 5. Test Addresses

### 5.1 User Addresses

| User | Label | Governorate | District | Street | Lat | Lon | Default |
|------|-------|-------------|----------|--------|-----|-----|---------|
| customer1 | Home | Sana'a | Al-Tahrir | Al-Zubairi St. 12 | 15.3694 | 44.1910 | Yes |
| customer1 | Work | Sana'a | Al-Sabaeen | University Rd. 5 | 15.3556 | 44.2066 | No |
| customer2 | Home | Aden | Crater | Al-Mualla St. 8 | 12.7854 | 45.0186 | Yes |
| customer3 | Home | Sana'a | Hadda | Hadda Rd. 20 | 15.3314 | 44.2185 | Yes |
| customer4 | Home | Taiz | Al-Qadisia | Al-Stah Rd. 3 | 13.5789 | 44.0219 | Yes |

### 5.2 Vendor Pickup Addresses

| Vendor | Governorate | District | Street | Lat | Lon |
|--------|-------------|----------|--------|-----|-----|
| vnd_tech01 | Sana'a | Al-Tahrir | Mutanabi St. 7 | 15.3710 | 44.1935 |
| vnd_fashion01 | Sana'a | Al-Maidan | Al-Maidan St. 15 | 15.3625 | 44.2015 |
| vnd_fresh01 | Sana'a | Bab Al-Yemen | Old City Market | 15.3544 | 44.2083 |
| vnd_home01 | Aden | Khormaksar | Al-Aroob St. 10 | 12.7900 | 45.0250 |
| vnd_book01 | Sana'a | Al-Qadisia | Book Market St. 2 | 15.3490 | 44.1975 |

---

## 6. Test Orders

### 6.1 Order Scenarios

| Order ID | Customer | Items | Total (YER) | Status | Payment | Notes |
|----------|----------|-------|-------------|--------|---------|-------|
| `ord_test01` | customer1 | prd_sam_a15 × 1 | 85,000 + 5,000 ship | delivered | card | Complete order flow |
| `ord_test02` | customer2 | prd_rice_5kg × 2, prd_oil_1L × 3 | 34,500 + 5,000 ship | delivered | cod | Cash on delivery |
| `ord_test03` | customer3 | prd_dress_w × 1, prd_abaya × 1 | 60,000 + 5,000 ship | shipped | wallet | Paid from wallet |
| `ord_test04` | customer4 | prd_macbook_air × 1 | 850,000 + 5,000 ship | processing | card | High-value order |
| `ord_test05` | customer1 | prd_airpods × 1, prd_charger × 2 | 119,000 + 5,000 ship | confirmed | card | Multi-vendor |
| `ord_test06` | customer2 | prd_shirt_m × 3 | 45,000 + 5,000 ship | pending_payment | — | Unpaid (pending) |
| `ord_test07` | customer3 | prd_cookware × 1 | 45,000 + 5,000 ship | cancelled | — | Cancelled by customer |
| `ord_test08` | customer4 | prd_iphone_15 × 1 | 350,000 + 5,000 ship | returned | refunded | Returned item |
| `ord_test09` | customer1 | prd_vacuum × 1 | 180,000 + 5,000 ship | in_transit | card | Rider assigned |
| `ord_test10` | customer3 | prd_math_10 × 5 | 17,500 + 5,000 ship | delivered | wallet | Bulk book order |

### 6.2 Order Status History (ord_test01)

| Status | Timestamp | Actor | Notes |
|--------|-----------|-------|-------|
| pending_payment | 2026-09-10 10:00:00 | customer1 | Order placed |
| confirmed | 2026-09-10 10:05:00 | system | Payment confirmed |
| processing | 2026-09-10 14:00:00 | vendor | Vendor acknowledged |
| shipped | 2026-09-11 09:00:00 | vendor | Shipped via YemenPost |
| in_transit | 2026-09-12 08:00:00 | rider1 | Picked up by rider |
| delivered | 2026-09-12 14:00:00 | rider1 | Delivered with photo proof |

---

## 7. Test Wallet Data

### 7.1 Wallet Balances

| User | Balance (YER) | Pending | Total Earned | Total Spent |
|------|---------------|---------|--------------|-------------|
| customer1 | 250,000 | 15,000 | 50,000 | 300,000 |
| customer2 | 100,000 | 0 | 25,000 | 125,000 |
| customer3 | 50,000 | 10,000 | 15,000 | 65,000 |
| customer4 | 500,000 | 0 | 100,000 | 600,000 |
| customer5 | 0 | 0 | 0 | 0 |
| vendor1 | 12,500,000 | 500,000 | 25,000,000 | 12,500,000 |
| vendor2 | 8,200,000 | 300,000 | 15,000,000 | 6,800,000 |
| vendor3 | 5,800,000 | 200,000 | 10,000,000 | 4,200,000 |

### 7.2 Wallet Transaction History (customer1)

| Transaction ID | Type | Amount | Description | Date |
|----------------|------|--------|-------------|------|
| `txn_test01` | topup | +100,000 | Card top-up | 2026-09-01 |
| `txn_test02` | debit | -85,000 | Order #YM-2026-0001 payment | 2026-09-03 |
| `txn_test03` | credit | +25,000 | Order #YM-2026-0002 refund | 2026-09-05 |
| `txn_test04` | topup | +200,000 | Card top-up | 2026-09-08 |
| `txn_test05` | debit | -60,000 | Order #YM-2026-0003 payment | 2026-09-10 |
| `txn_test06` | cashback | +10,000 | Ramadan cashback reward | 2026-09-11 |
| `txn_test07` | transfer_out | -15,000 | Transfer to Ahmed | 2026-09-12 |

---

## 8. Test Coupons

| Code | Type | Value | Min Order | Max Discount | Usage Limit | Valid | Status |
|------|------|-------|-----------|--------------|-------------|-------|--------|
| `WELCOME2026` | percentage | 10% | 50,000 | 25,000 | 1000 | 2026-01-01 to 2026-12-31 | active |
| `RAMADAN2026` | percentage | 15% | 100,000 | 50,000 | 500 | 2026-03-01 to 2026-04-30 | expired |
| `FLAT5000` | fixed_amount | 5,000 | 30,000 | 5,000 | 200 | 2026-09-01 to 2026-09-30 | active |
| `FREESHIP` | free_shipping | 100% | 150,000 | 10,000 | unlimited | 2026-01-01 to 2026-12-31 | active |
| `VIP20` | percentage | 20% | 500,000 | 100,000 | 50 | 2026-01-01 to 2026-12-31 | active |
| `USEDUP` | percentage | 10% | 50,000 | 20,000 | 1 | 2026-01-01 to 2026-12-31 | active (exhausted) |
| `EXPIRED` | fixed_amount | 10,000 | 100,000 | 10,000 | 100 | 2025-01-01 to 2025-12-31 | expired |

---

## 9. Test Reviews

| Review ID | Product | User | Rating | Title | Verified | Date |
|-----------|---------|------|--------|-------|----------|------|
| `rev_test01` | prd_sam_a15 | customer2 | 5 | Excellent phone! | Yes | 2026-09-05 |
| `rev_test02` | prd_sam_a15 | customer3 | 4 | Good value for money | Yes | 2026-09-06 |
| `rev_test03` | prd_iphone_15 | customer4 | 5 | Best iPhone yet | Yes | 2026-09-07 |
| `rev_test04` | prd_rice_5kg | customer1 | 5 | Fresh and high quality | Yes | 2026-09-04 |
| `rev_test05` | prd_shirt_m | customer3 | 3 | Average quality fabric | Yes | 2026-09-08 |
| `rev_test06` | prd_macbook_air | customer4 | 5 | Perfect for work | Yes | 2026-09-09 |
| `rev_test07` | prd_math_10 | customer3 | 5 | Essential for students | Yes | 2026-09-10 |
| `rev_test08` | prd_cookware | customer1 | 4 | Good quality set | Yes | 2026-09-11 |
| `rev_test09` | prd_sam_a15 | customer5 | 1 | Arrived damaged | Yes | 2026-09-12 |
| `rev_test10` | prd_airpods | customer2 | 5 | Amazing sound quality | Yes | 2026-09-13 |

---

## 10. Test Deliveries

| Delivery ID | Order ID | Rider | Status | Pickup | Delivery | Fee |
|-------------|----------|-------|--------|--------|----------|-----|
| `dlv_test01` | ord_test01 | rider1 | delivered | vnd_tech01 | customer1 (Sana'a) | 5,000 |
| `dlv_test02` | ord_test02 | rider2 | delivered | vnd_fresh01 | customer2 (Aden) | 8,000 |
| `dlv_test03` | ord_test03 | rider3 | in_transit | vnd_fashion01 | customer3 (Sana'a) | 5,000 |
| `dlv_test04` | ord_test04 | rider1 | pending | vnd_tech01 | customer4 (Taiz) | 12,000 |
| `dlv_test05` | ord_test09 | rider3 | picked_up | vnd_home01 | customer1 (Sana'a) | 5,000 |

### 10.1 Rider Locations

| Rider | Latitude | Longitude | Speed | Online | Last Update |
|-------|----------|-----------|-------|--------|-------------|
| rider1 | 15.3710 | 44.1935 | 25 km/h | Yes | 2026-09-13 14:00:00 |
| rider2 | 12.7854 | 45.0186 | 0 km/h | Yes | 2026-09-13 14:00:00 |
| rider3 | 15.3625 | 44.2015 | 30 km/h | Yes | 2026-09-13 14:00:00 |
| offline-rider | 15.3556 | 44.2066 | 0 km/h | No | 2026-09-13 12:00:00 |

---

## 11. Test Notifications

| Notification ID | User | Type | Title | Read | Date |
|-----------------|------|------|-------|------|------|
| `ntf_test01` | customer1 | order_update | Order shipped | Yes | 2026-09-11 |
| `ntf_test02` | customer1 | promotion | Ramadan sale starts! | Yes | 2026-09-01 |
| `ntf_test03` | customer2 | payment | Payment confirmed | Yes | 2026-09-10 |
| `ntf_test04` | customer3 | delivery | Rider on the way | No | 2026-09-13 |
| `ntf_test05` | vendor1 | order_new | New order received | No | 2026-09-13 |
| `ntf_test06` | vendor1 | review | New 5-star review | No | 2026-09-12 |
| `ntf_test07` | admin | system | Vendor application pending | No | 2026-09-13 |

---

## 12. Data Masking Strategy

### 12.1 Non-Production Data Masking Rules

| Data Type | Production Value | Masked Value | Method |
|-----------|-----------------|--------------|--------|
| Email addresses | real@email.com | test_customer_01@yemenmart.test | Synthetic generation |
| Phone numbers | +967712345678 | +967700000001 | Sequential pattern |
| Names | Real names | Faker-generated names | Faker library |
| Addresses | Real addresses | Test addresses with real governorates | Synthetic with real geography |
| Passwords | Real hashes | Bcrypt of known test passwords | Pre-computed hashes |
| Payment cards | Real card numbers | Stripe test card numbers | Test gateway tokens |
| IP addresses | Real IPs | 192.168.x.x / 10.x.x.x | Private ranges |
| GPS coordinates | Exact locations | ±0.01 degree offset | Randomized |
| API keys | Production keys | Test/sandbox keys | Environment separation |
| Encryption keys | Real keys | Test-only keys | Separate key management |

### 12.2 Masking Implementation

| Layer | Approach | Tool |
|-------|----------|------|
| Database | Synthetic data generation | Faker.js + custom seed scripts |
| API responses | DTO filtering | Never return raw sensitive data |
| Logs | Pattern-based scrubbing | Custom log middleware |
| Test environments | Full masking on clone | ETL pipeline with masking rules |
| Local development | Synthetic only | No production data access |

### 12.3 Data Sensitivity Matrix

| Data | Production | Staging | Development | Test |
|------|-----------|---------|-------------|------|
| User PII | Real (encrypted) | Masked | Synthetic | Synthetic |
| Passwords | Bcrypt hash | Bcrypt hash | Bcrypt hash | Known test passwords |
| Payment data | Tokenized | Test tokens | Test tokens | Test tokens |
| Order data | Real | Masked | Synthetic | Synthetic |
| Wallet balances | Real | Masked | Synthetic | Synthetic |
| Vendor financials | Real | Masked | N/A | Synthetic |
| Audit logs | Real | Masked | N/A | Synthetic |
| Search queries | Real | Anonymized | N/A | Synthetic |

---

## 13. Seed Data Scripts

### 13.1 Script Organization

```
test/
├── seed/
│   ├── 00-reset.sql          -- Truncate all tables
│   ├── 01-users.sql          -- User accounts
│   ├── 02-vendors.sql        -- Vendor profiles
│   ├── 03-categories.sql     -- Category tree
│   ├── 04-products.sql       -- Product catalog
│   ├── 05-variants.sql       -- Product variants
│   ├── 06-images.sql         -- Product images
│   ├── 07-addresses.sql      -- User addresses
│   ├── 08-orders.sql         -- Test orders
│   ├── 09-payments.sql       -- Payment records
│   ├── 10-wallets.sql        -- Wallet balances
│   ├── 11-transactions.sql   -- Wallet transactions
│   ├── 12-coupons.sql        -- Coupon definitions
│   ├── 13-reviews.sql        -- Product reviews
│   ├── 14-deliveries.sql     -- Delivery assignments
│   ├── 15-notifications.sql  -- Test notifications
│   └── 16-settings.sql       -- Platform settings
├── fixtures/
│   ├── products.json         -- Product fixture data
│   ├── orders.json           -- Order fixture data
│   └── users.json            -- User fixture data
└── helpers/
    ├── reset-db.sh           -- Database reset script
    ├── seed-all.sh           -- Full seed script
    └── generate-test-data.js -- Dynamic test data generator
```

### 13.2 Execution Order

```bash
# Full reset and seed
npm run test:setup

# Individual scripts
npm run test:seed:users
npm run test:seed:products
npm run test:seed:orders

# Verify seed data
npm run test:verify

# Generate additional random data
npm run test:generate -- --count 1000 --type users
npm run test:generate -- --count 5000 --type products
npm run test:generate -- --count 10000 --type orders
```

---

## 14. Test Data Maintenance

### 14.1 Refresh Schedule

| Environment | Refresh Frequency | Method |
|-------------|-------------------|--------|
| Local development | On demand | `npm run test:setup` |
| CI/CD pipeline | Per build | Automatic reset + seed |
| Integration test env | Daily | Scheduled reset at 02:00 UTC |
| Staging | Weekly | Friday 22:00 UTC |
| Performance test env | Before load tests | Full refresh + data generation |

### 14.2 Data Volume Targets

| Entity | Target Count | Purpose |
|--------|-------------|---------|
| Users | 10,000 | Pagination, search testing |
| Products | 50,000 | Search, filtering, performance |
| Orders | 100,000 | Reporting, analytics |
| Reviews | 200,000 | Search, sorting |
| Transactions | 500,000 | Wallet, reporting |
| Notifications | 1,000,000 | Pagination, cleanup |

### 14.3 Test Data Quality Checks

| Check | Frequency | Action on Failure |
|-------|-----------|-------------------|
| Referential integrity | Every seed | Abort seed, fix foreign keys |
| Unique constraints | Every seed | Log duplicate, skip entry |
| Not-null constraints | Every seed | Log error, fix data |
| Business rule validation | Daily | Alert team, fix seed scripts |
| Data freshness | Weekly | Refresh from latest templates |
