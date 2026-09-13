# YemenMart Test Points Compendium

**Document ID:** YMP-TC-001  
**Version:** 1.0  
**Date:** 2026-09-13  
**Classification:** Confidential  
**Total Test Points:** 974  
**Blocks:** 13 + Cross-Block Integration  

---

## Priority Distribution

| Priority | Count | Percentage | Description |
|----------|-------|------------|-------------|
| P0 | 627 | 64% | Critical - Must pass for release |
| P1 | 334 | 34% | High - Should pass for release |
| P2 | 13 | 1% | Medium - Known issues, workaround exists |
| **Total** | **974** | **100%** | - |

## Block Summary

| Block | Name | Test Points | P0 | P1 | P2 |
|-------|------|-------------|-----|-----|-----|
| 01 | System Core | 27 | 17 | 9 | 1 |
| 02 | Marketplace & Vendors | 70 | 45 | 24 | 1 |
| 03 | Product Catalog | 74 | 47 | 26 | 1 |
| 04 | Orders & Checkout | 139 | 89 | 49 | 1 |
| 05 | Payments & Wallet | 72 | 46 | 25 | 1 |
| 06 | Finance & Accounting | 70 | 45 | 24 | 1 |
| 07 | Shipping & Delivery | 72 | 46 | 25 | 1 |
| 08 | Inventory Management | 72 | 46 | 25 | 1 |
| 09 | Customer Storefront | 74 | 47 | 26 | 1 |
| 10 | Reviews & Loyalty | 74 | 47 | 26 | 1 |
| 11 | Content & Notifications | 72 | 46 | 25 | 1 |
| 12 | Support & Analytics | 79 | 51 | 27 | 1 |
| 13 | Coupons & Discounts | 79 | 51 | 27 | 1 |
| - | Cross-Block Integration | 15 | 4 | 11 | 0 |
| **Total** | | **974** | **627** | **334** | **13** |

---

## Conventions

- **Test ID Format:** B{block:02d}-{sequence:03d} (e.g., B01-001)
- **Priority Levels:**
  - **P0 (Critical):** Blocks release if failing. Security, data loss, payment, order creation.
  - **P1 (High):** Should be resolved before release. Feature functionality, UX flows.
  - **P2 (Medium):** Known issues with workarounds. Cosmetic, edge cases.
- **Precondition Key:** [AUTH] Authenticated, [ADMIN] Admin role, [VENDOR] Vendor role, [DB] Database state, [CONFIG] System config

---


# Block 01 - System Core (27 Test Points)

## 1.1 Registration (8 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B01-001 | SMS OTP registration with valid Yemen mobile number | P0 | Guest user, valid +967XXXXXXXXX number | OTP sent within 5s, account created, JWT issued |
| B01-002 | WhatsApp OTP registration with valid number | P0 | Guest user, WhatsApp-capable number | WhatsApp message delivered, account created on verification |
| B01-003 | Email verification registration with valid email | P0 | Guest user, valid email format | Verification email sent, email verified, account activated |
| B01-004 | Guest-to-registered conversion preserves cart | P0 | Guest with items in cart, no account | Cart items migrated to registered account, no data loss |
| B01-005 | Duplicate phone number registration rejection | P0 | Existing account with same number | Error: phone already registered, no duplicate created |
| B01-006 | OTP expiry after 5 minutes | P1 | User requested OTP, 5+ minutes elapsed | OTP expired, new OTP required for verification |
| B01-007 | OTP resend rate limiting (max 3 per 10 min) | P1 | User sent 3 OTPs in 10 minutes | 4th request blocked with rate limit error |
| B01-008 | Registration with invalid phone format | P2 | Guest user, malformed phone number | Validation error: invalid phone format |

## 1.2 Login (6 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B01-009 | Phone + password login with valid credentials (PRIMARY) | P0 | Registered user with password | Login successful, JWT issued with correct roles |
| B01-010 | Failed login attempt counter | P0 | Registered user | Failed attempts tracked, user notified after 3 failures |
| B01-011 | Account lockout after 5 failed attempts | P0 | 4 previous failed attempts | 5th failure locks account for 15 minutes |
| B01-012 | Login with deactivated account | P0 | Account status: deactivated | Login rejected with appropriate error message |
| B01-013 | Login with incorrect password | P0 | Registered user, wrong password | Error: invalid credentials, attempt count incremented |
| B01-014 | Login with account that has no password set | P1 | Registered via OTP only, no password | Error: password not set, must create password first |

## 1.3 Password Management (5 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B01-015 | Forgot password via phone + OTP (SECONDARY) | P0 | Registered user, valid phone | OTP sent via SMS/WhatsApp, password reset flow initiated |
| B01-016 | Password reset with valid OTP | P0 | Valid OTP (< 5 minutes old) | Password updated, all existing sessions invalidated |
| B01-017 | Password reset with expired OTP | P1 | OTP older than 5 minutes | Error: OTP expired, new reset request required |
| B01-018 | Password history check (last 3) | P1 | User with password change history | Cannot reuse any of last 3 passwords |
| B01-019 | Minimum password strength enforcement | P0 | User setting new password | Reject passwords below 8 chars, requiring mixed case + numbers |

## 1.4 Session Management (4 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B01-020 | JWT token expiry after 24 hours | P0 | Active session | Token expired after 24h, re-authentication required |
| B01-021 | Session invalidation on logout | P0 | Active session | Token blacklisted, all API calls rejected |
| B01-022 | Concurrent session limit (5 max) | P1 | User with 5 active sessions | 6th session forces oldest session logout |
| B01-023 | Refresh token rotation | P1 | Active session, refresh token used | New refresh token issued, old one invalidated |

## 1.5 Audit Logging (4 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B01-024 | Login event audit log creation | P0 | Successful login | Audit log entry with timestamp, IP, user agent, location |
| B01-025 | Failed login audit log creation | P0 | Failed login attempt | Audit log entry with failure reason, IP, attempt count |
| B01-026 | Audit log immutability | P0 | Existing audit log entries | No user/admin can modify or delete audit entries |
| B01-027 | Audit log retention (90 days) | P1 | Audit logs older than 90 days | Automatic archival, no deletion within 90-day window |


---

# Block 02 - Marketplace & Vendors (70 Test Points)

## 2.1 Vendor Registration & KYC (12 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B02-001 | Vendor registration with valid business details | P0 | Authenticated user | Vendor application created, status: pending_kyc |
| B02-002 | KYC document submission (commercial registration) | P0 | Vendor with pending KYC | Documents uploaded, status: pending_review |
| B02-003 | KYC approval by admin | P0 | KYC documents submitted | Vendor status: approved, store creation enabled |
| B02-004 | KYC rejection with reason | P0 | KYC documents submitted | Vendor status: rejected, rejection reason provided |
| B02-005 | KYC resubmission after rejection | P1 | KYC rejected | New documents uploadable, status: pending_review |
| B02-006 | Duplicate vendor registration prevention | P0 | User already has vendor account | Error: vendor account already exists |
| B02-007 | KYC document format validation (PDF/JPG only) | P1 | Vendor uploading KYC docs | Non-PDF/JPG files rejected with format error |
| B02-008 | KYC document size limit (10MB per file) | P1 | Vendor uploading large file | Files > 10MB rejected with size error |
| B02-009 | Vendor profile completeness check | P1 | Vendor with incomplete profile | Warning: profile X% complete, missing fields listed |
| B02-010 | Tax registration number validation | P0 | Vendor entering tax ID | Valid Yemen tax ID format validated |
| B02-011 | Vendor terms acceptance required | P0 | New vendor registration | Cannot proceed without accepting vendor terms |
| B02-012 | KYC document expiry check | P1 | KYC with expired documents | Alert: documents expired, renewal required |

## 2.2 Store Management (12 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B02-013 | Store creation with unique slug | P0 | KYC-approved vendor | Store created, slug auto-generated, publicly accessible |
| B02-014 | Slug uniqueness enforcement | P0 | Vendor attempting duplicate slug | Error: slug already taken, alternative suggested |
| B02-015 | Store template selection | P1 | Vendor with created store | Template applied, store layout updated |
| B02-016 | Store settings update (name, description, logo) | P0 | Vendor with created store | Settings saved, store page reflects changes |
| B02-017 | Store deactivation by vendor | P1 | Active vendor store | Store hidden from directory, products unpublished |
| B02-018 | Store reactivation by vendor | P1 | Deactivated vendor store | Store restored, products republished |
| B02-019 | Store banner image upload | P1 | Vendor with created store | Banner uploaded, displayed on store page |
| B02-020 | Store business hours configuration | P1 | Vendor with created store | Hours saved and displayed on store page |
| B02-021 | Store contact info update | P0 | Vendor with created store | Phone/email/address updated on store page |
| B02-022 | Store location (governorate/district) setting | P0 | Vendor with created store | Location set, used for delivery zone calculation |
| B02-023 | Store policy pages (return, shipping) | P1 | Vendor with created store | Custom policy pages displayed on store |
| B02-024 | Store social media links | P2 | Vendor with created store | Social links displayed on store page |

## 2.3 Vendor Badges (6 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B02-025 | Badge award on meeting criteria | P0 | Vendor meets badge threshold | Badge automatically awarded, displayed on store |
| B02-026 | Badge criteria: 100+ orders, 4.5+ rating | P1 | Vendor with 100+ orders and 4.5+ avg rating | "Top Seller" badge awarded |
| B02-027 | Badge expiry after 90 days | P1 | Badge awarded 90+ days ago | Badge review triggered, renewal or removal |
| B02-028 | Badge removal on criteria failure | P0 | Badge holder drops below threshold | Badge removed within 24 hours |
| B02-029 | "Verified" badge on KYC completion | P0 | Vendor with completed KYC | Verified badge permanently displayed |
| B02-030 | Badge display on product listings | P1 | Vendor with active badges | Badge icons shown on all product cards |

## 2.4 Staff & RBAC (12 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B02-031 | Staff invitation by store owner | P0 | Vendor owner account | Invitation sent via email, pending acceptance |
| B02-032 | Staff role assignment (manager, support, viewer) | P0 | Staff invitation accepted | Role permissions applied correctly |
| B02-033 | Manager role: full product/order access | P0 | Staff with manager role | Can manage products, orders, inventory |
| B02-034 | Support role: order chat only | P1 | Staff with support role | Can only access order communications |
| B02-035 | Viewer role: read-only access | P1 | Staff with viewer role | Cannot modify any data, view-only |
| B02-036 | Staff removal by store owner | P0 | Active staff member | Staff removed, access revoked immediately |
| B02-037 | Owner cannot remove own access | P1 | Store owner account | Error: cannot remove primary owner |
| B02-038 | Maximum staff limit per store (10) | P1 | Store with 10 staff members | 11th invitation blocked |
| B02-039 | Staff activity logging | P0 | Staff performing actions | All actions logged with staff ID |
| B02-040 | Role permission boundary testing | P0 | Staff attempting out-of-scope action | Action blocked, 403 Forbidden returned |
| B02-041 | Staff invitation expiry (7 days) | P1 | Invitation sent 7+ days ago | Invitation expired, new one required |
| B02-042 | Multiple role assignment per staff | P1 | Staff needing multiple roles | Combined permissions applied, no conflicts |

## 2.5 Coupons & Merchant Offers (8 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B02-043 | Merchant coupon creation | P0 | Vendor with active store | Coupon created, pending admin approval |
| B02-044 | Admin coupon approval | P0 | Coupon pending approval | Coupon activated, available for customers |
| B02-045 | Admin coupon rejection with reason | P0 | Coupon pending approval | Coupon rejected, vendor notified with reason |
| B02-046 | Coupon usage limit configuration | P1 | Vendor creating coupon | Usage limit set, enforced on redemption |
| B02-047 | Coupon expiry date enforcement | P0 | Active coupon past expiry | Coupon rejected at checkout with expiry error |
| B02-048 | Store-specific coupon restriction | P1 | Vendor creating store coupon | Coupon only valid for products from that store |
| B02-049 | Coupon minimum order value | P1 | Coupon with minimum order set | Discount not applied if cart below minimum |
| B02-050 | Coupon code uniqueness across platform | P0 | Vendor creating coupon code | System checks uniqueness across all merchants |

## 2.6 Vendor Payout & Finance (6 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B02-051 | Vendor payout balance calculation | P0 | Vendor with completed orders | Balance = sales - commissions - refunds |
| B02-052 | Commission deduction on order completion | P0 | Order marked delivered | Commission (5-15%) deducted, vendor balance updated |
| B02-053 | Weekly payout processing | P0 | Vendor with positive balance | Payout initiated per weekly schedule |
| B02-054 | Payout method configuration | P1 | Vendor account | Bank details saved, validated |
| B02-055 | Minimum payout threshold (1000 YER) | P1 | Vendor balance below 1000 YER | Payout deferred until threshold met |
| B02-056 | Payout history and statement | P1 | Vendor with past payouts | Complete payout history viewable |

## 2.7 Vendor Analytics & Performance (8 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B02-057 | Vendor dashboard - sales overview | P0 | Vendor with sales data | Today/week/month sales figures displayed |
| B02-058 | Vendor dashboard - order statistics | P0 | Vendor with orders | Pending, shipped, delivered, returned counts |
| B02-059 | Vendor dashboard - revenue chart | P1 | Vendor with historical sales | Revenue trend chart with date range selector |
| B02-060 | Vendor dashboard - top products | P1 | Vendor with product sales | Top 10 products by revenue/quantity |
| B02-061 | Vendor dashboard - customer demographics | P1 | Vendor with customer data | Customer location/behavior breakdown |
| B02-062 | Vendor dashboard - conversion rate | P1 | Vendor with view/purchase data | Views to purchases conversion percentage |
| B02-063 | Vendor performance score calculation | P0 | Vendor with order/delivery data | Performance score (0-100) calculated |
| B02-064 | Vendor rating display and calculation | P0 | Vendor with customer reviews | Average rating displayed, updated on new reviews |

## 2.8 Vendor Compliance & Policies (6 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B02-065 | Vendor terms of service acceptance | P0 | New vendor registration | Terms accepted before account activation |
| B02-066 | Prohibited items policy enforcement | P0 | Vendor listing restricted item | Item blocked, vendor warned |
| B02-067 | Price gouging detection | P0 | Vendor setting excessive prices | Flagged for admin review, auto-rejected if >200% market avg |
| B02-068 | Vendor response time SLA (24h) | P1 | Vendor receiving customer inquiry | Response within 24h tracked, escalation on breach |
| B02-069 | Vendor account suspension for violations | P0 | Vendor with multiple violations | Account suspended, all products unpublished |
| B02-070 | Vendor account reinstatement after appeal | P1 | Suspended vendor with successful appeal | Account reactivated, restrictions lifted |


---

# Block 03 - Product Catalog (74 Test Points)

## 3.1 Product Creation (14 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B03-001 | 9-step product creation wizard initiation | P0 | Vendor with approved store | Wizard loads with step 1 (basic info) |
| B03-002 | Step 1: Basic info (name, description, category) | P0 | Vendor in wizard step 1 | Required fields validated, step advances |
| B03-003 | Step 2: Pricing (base price, compare-at price) | P0 | Vendor in wizard step 2 | Price validation, compare-at > base price |
| B03-004 | Step 3: Variants (size, color, material) | P0 | Vendor in wizard step 3 | Variant matrix generated, SKU combinations created |
| B03-005 | Step 4: Media upload (photos, video) | P0 | Vendor in wizard step 4 | Min 3, max 10 images uploaded, video optional |
| B03-006 | Step 5: Inventory & shipping | P0 | Vendor in wizard step 5 | Stock quantity, weight, dimensions saved |
| B03-007 | Step 6: SEO settings (title, meta, slug) | P1 | Vendor in wizard step 6 | SEO fields saved, slug uniqueness checked |
| B03-008 | Step 7: Attributes & specifications | P1 | Vendor in wizard step 7 | Custom attributes saved, displayed on product page |
| B03-009 | Step 8: Pricing tiers (bulk discounts) | P1 | Vendor in wizard step 8 | Volume pricing rules saved |
| B03-010 | Step 9: Review & submit | P0 | Vendor in wizard step 9 | Product preview shown, submission queued for approval |
| B03-011 | Product auto-scoring algorithm | P0 | Product submitted | Score calculated: completeness, media quality, SEO, price competitiveness |
| B03-012 | Product approval by admin | P0 | Product pending approval | Product published, visible in storefront |
| B03-013 | Product rejection with feedback | P0 | Product pending approval | Rejection reason provided, vendor notified |
| B03-014 | Product draft save and resume | P1 | Vendor in middle of wizard | Draft saved, can resume from same step |

## 3.2 Product Variants (10 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B03-015 | SKU uniqueness across platform | P0 | Vendor creating variant | SKU checked globally, duplicates rejected |
| B03-016 | Variant price override | P0 | Product with variants | Each variant can have unique price |
| B03-017 | Variant stock quantity tracking | P0 | Product with variants | Stock tracked per variant independently |
| B03-018 | Variant image assignment | P1 | Product with variants | Specific images mapped to variant combinations |
| B03-019 | Variant availability toggle | P1 | Product with variants | Individual variants can be enabled/disabled |
| B03-020 | Default variant selection | P1 | Product with variants | First variant pre-selected on product page |
| B03-021 | Variant combination limits (max 100) | P1 | Product with many options | Error when combinations exceed 100 |
| B03-022 | Variant stock sync with parent | P0 | Parent product stock change | All variants reflect updated stock status |
| B03-023 | Out-of-stock variant handling | P0 | Variant with zero stock | Variant shows "Out of Stock", excluded from checkout |
| B03-024 | Low stock alert per variant | P1 | Variant stock below threshold | Vendor notified of low stock per variant |

## 3.3 Categories & Hierarchy (10 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B03-025 | Category hierarchy (3 levels max) | P0 | Admin managing categories | Parent > Child > Subchild structure enforced |
| B03-026 | Product-to-category assignment | P0 | Vendor creating product | Product assigned to leaf category only |
| B03-027 | Category filtering on storefront | P0 | Customer browsing | Products filtered correctly by selected category |
| B03-028 | Multi-category assignment (max 3) | P1 | Vendor editing product | Product assigned to max 3 categories |
| B03-029 | Category sort order | P1 | Admin managing categories | Categories display in configured order |
| B03-030 | Category icon/banner upload | P1 | Admin managing categories | Visual assets saved and displayed |
| B03-031 | Category product count display | P1 | Category with products | Accurate product count shown |
| B03-032 | Category slug generation | P0 | Admin creating category | Auto-generated slug, uniqueness enforced |
| B03-033 | Category deactivation | P1 | Admin managing categories | Category hidden, products reassigned or unpublished |
| B03-034 | Category description SEO field | P1 | Admin creating category | Description saved, used for category page SEO |

## 3.4 Product Media (8 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B03-035 | Image upload (JPEG, PNG, WebP) | P0 | Vendor editing product | Valid formats accepted, others rejected |
| B03-036 | Image size limit (5MB per image) | P0 | Vendor uploading image | Files > 5MB rejected with error |
| B03-037 | Image auto-resize to 1000x1000 | P1 | Vendor uploading large image | Image resized, aspect ratio preserved |
| B03-038 | Image compression for web delivery | P1 | Image uploaded | Compressed to < 200KB without visible quality loss |
| B03-039 | Minimum 3 images per product | P0 | Vendor submitting product | Submission blocked if < 3 images |
| B03-040 | Maximum 10 images per product | P1 | Vendor uploading images | Upload blocked after 10 images |
| B03-041 | Product video upload (max 60s, 50MB) | P1 | Vendor adding video | Video uploaded, duration/size validated |
| B03-042 | Image reordering via drag-and-drop | P1 | Vendor with multiple images | Image order updated, first image = thumbnail |

## 3.5 Product Search & Discovery (8 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B03-043 | Full-text search across products | P0 | Products in catalog | Relevant results returned for keyword search |
| B03-044 | Search by category filter | P0 | Customer searching | Results filtered to selected category |
| B03-045 | Search by price range | P0 | Customer searching | Results filtered to price range |
| B03-046 | Search by vendor/store | P1 | Customer searching | Results filtered to specific vendor |
| B03-047 | Search by availability (in-stock only) | P1 | Customer searching | Out-of-stock products excluded |
| B03-048 | Search sort (relevance, price, rating, newest) | P1 | Customer searching | Results sorted by selected criteria |
| B03-049 | Search autocomplete suggestions | P1 | Customer typing in search | Suggestions appear after 2+ characters |
| B03-050 | Empty search results handling | P1 | No products match search | "No results found" with suggestions displayed |

## 3.6 Offers & Promotions (12 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B03-051 | Flash sale creation by admin | P0 | Admin, active products | Flash sale created with start/end time |
| B03-052 | Flash sale price activation | P0 | Flash sale scheduled, start time reached | Discounted price auto-activated |
| B03-053 | Flash sale price deactivation | P0 | Flash sale active, end time reached | Price reverts to original |
| B03-054 | BOGO (Buy One Get One) offer | P0 | Vendor creating promotion | BOGO rules applied correctly at checkout |
| B03-055 | Bundle pricing (2+ items) | P1 | Vendor creating bundle | Bundle discount applied when all items in cart |
| B03-056 | Offer stacking rules (no double discount) | P0 | Product with multiple offers | Only best offer applied, not cumulative |
| B03-057 | Offer inventory limit | P1 | Offer with limited quantity | Offer disabled when quantity exhausted |
| B03-058 | Offer per-customer limit | P1 | Offer with per-user limit | Limit enforced, additional purchases at full price |
| B03-059 | Offer preview on product page | P1 | Active offer on product | Discounted price and savings displayed |
| B03-060 | Offer countdown timer | P1 | Flash sale active | Countdown displayed on product page |
| B03-061 | Offer performance tracking | P1 | Active offer with sales data | Views, conversions, revenue attributed to offer |
| B03-062 | Scheduled offer auto-activation | P0 | Offer with future start date | Offer activates automatically at scheduled time |

## 3.7 Product Status & Lifecycle (12 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B03-063 | Product draft status | P0 | Vendor saving draft | Product saved, not visible to customers |
| B03-064 | Product pending approval status | P0 | Vendor submitting product | Product queued for admin review |
| B03-065 | Product approved/published status | P0 | Admin approving product | Product visible in storefront and search |
| B03-066 | Product rejected status | P0 | Admin rejecting product | Product returned to vendor with feedback |
| B03-067 | Product suspended status | P0 | Admin suspending product | Product hidden, vendor notified |
| B03-068 | Product archived status | P1 | Vendor archiving product | Product hidden, data preserved |
| B03-069 | Product deletion (soft delete) | P0 | Vendor deleting product | Soft-deleted, recoverable within 30 days |
| B03-070 | Product permanent deletion | P1 | Admin confirming deletion | Data permanently removed after 30-day grace |
| B03-071 | Product update triggers re-approval | P0 | Vendor editing published product | Product queued for re-approval |
| B03-072 | Bulk product operations | P1 | Vendor with many products | Bulk status change, price update, deletion |
| B03-073 | Product import/export (CSV) | P1 | Vendor with catalog data | CSV import/export with validation |
| B03-074 | Product activity log | P0 | Any product modification | All changes logged with timestamp and user |


---

# Block 04 - Orders & Checkout (139 Test Points)

## 4.1 Cart Management (18 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B04-001 | Add single item to cart | P0 | Authenticated user, product in stock | Item added with quantity 1, cart total updated |
| B04-002 | Add multiple items to cart | P0 | Authenticated user, multiple products | All items added, cart total calculated |
| B04-003 | Update item quantity in cart | P0 | Item in cart | Quantity updated, stock checked, total recalculated |
| B04-004 | Remove item from cart | P0 | Item in cart | Item removed, total recalculated |
| B04-005 | Cart persists across sessions | P0 | Authenticated user with items | Cart data persisted in database, restored on login |
| B04-006 | Guest cart merge on registration | P0 | Guest with cart items, registering | Guest cart merged with existing registered cart |
| B04-007 | Stock validation on cart load | P0 | Items in cart, stock changed | Out-of-stock items flagged, quantity capped |
| B04-008 | Price change detection in cart | P0 | Items in cart, price changed | Customer notified of price change before checkout |
| B04-009 | Apply coupon to cart | P0 | Valid coupon code | Discount applied, total recalculated |
| B04-010 | Remove coupon from cart | P0 | Applied coupon | Coupon removed, total recalculated |
| B04-011 | Cart maximum items limit (50) | P1 | Cart approaching 50 items | Warning at 45, block at 50 |
| B04-012 | Cart maximum quantity per item (10) | P1 | Cart with item quantity > 10 | Quantity capped at 10 |
| B04-013 | Cart subtotal calculation accuracy | P0 | Cart with items, tax, discounts | Subtotal = sum(item_price * quantity) |
| B04-014 | Multi-vendor cart grouping | P0 | Cart with items from 3+ vendors | Items grouped by vendor for sub-orders |
| B04-015 | Cart expiry (abandoned after 7 days) | P1 | Cart inactive for 7+ days | Cart items cleared, soft notification sent |
| B04-016 | Add out-of-stock item to cart | P0 | Customer adding unavailable product | Error: product out of stock |
| B04-017 | Cart delivery fee preview | P0 | Cart with items, location set | Estimated delivery fee displayed |
| B04-018 | Cart wallet balance display | P0 | User with wallet balance | Available balance shown during cart review |

## 4.2 Checkout Flow (22 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B04-019 | Checkout step 1: Delivery address | P0 | Items in cart | Address form displayed, validation enforced |
| B04-020 | Checkout step 2: Delivery method selection | P0 | Valid delivery address | Available delivery options displayed with fees |
| B04-021 | Checkout step 3: Delivery time slot | P0 | Delivery method selected | Available time slots displayed |
| B04-022 | Checkout step 4: Payment method | P0 | Delivery configured | Payment options: Wallet only |
| B04-023 | Checkout step 5: Order summary | P0 | Payment method selected | Full breakdown: items, fees, discounts, total |
| B04-024 | Checkout step 6: Review & confirm | P0 | Summary displayed | All details editable, confirm button enabled |
| B04-025 | Checkout step 7: Order placement | P0 | User confirms order | Order created, payment initiated, confirmation sent |
| B04-026 | Wallet balance validation at checkout | P0 | User selecting wallet payment | Balance must cover full order amount |
| B04-027 | Wallet-only order creation | P0 | User selecting wallet payment | Order created, payment processed from wallet |
| B04-028 | Bank transfer order creation | P0 | User selecting bank transfer | Order created, payment proof upload enabled |
| B04-029 | Checkout session timeout (15 min) | P1 | User idle in checkout for 15 min | Session expired, return to cart |
| B04-030 | Forced registration for checkout | P0 | Guest user at checkout | Redirect to registration, cart preserved |
| B04-031 | Address validation (governorate/district) | P0 | User entering address | Valid Yemen governorate and district required |
| B04-032 | Delivery zone availability check | P0 | Address submitted | Check if delivery available to selected area |
| B04-033 | Minimum order value enforcement | P1 | Cart below minimum | Error: minimum order not met |
| B04-034 | Maximum order value limit | P1 | Cart above maximum | Error: order exceeds maximum limit |
| B04-035 | Order placement inventory reservation | P0 | User confirming order | Stock reserved immediately, not deducted |
| B04-036 | Checkout error recovery | P0 | Error during checkout | User returns to last valid step, data preserved |
| B04-037 | Single payment method (wallet only) | P1 | User at checkout | Only wallet payment available |
| B04-038 | Checkout analytics tracking | P1 | User progressing through checkout | Funnel events logged for each step |
| B04-039 | Order confirmation email/SMS | P0 | Order successfully placed | Confirmation sent with order ID and summary |
| B04-040 | Checkout CSRF protection | P0 | User submitting checkout form | CSRF token validated, replay attacks blocked |

## 4.3 Order State Machine (35 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B04-041 | State: pending → confirmed (auto for wallet) | P0 | Wallet payment successful | Order auto-confirmed |
| B04-042 | State: pending → confirmed (auto for wallet) | P0 | Wallet payment successful | Order auto-confirmed |
| B04-043 | State: pending → confirmed (bank transfer) | P0 | Bank transfer selected | Order pending payment verification |
| B04-044 | State: confirmed → processing | P0 | Vendor accepts order | Vendor begins processing, status updated |
| B04-045 | State: confirmed → cancelled (vendor) | P0 | Vendor rejects order | Order cancelled, refund initiated if paid |
| B04-046 | State: processing → shipped | P0 | Vendor ships order | Tracking info added, customer notified |
| B04-047 | State: shipped → out_for_delivery | P0 | Courier picks up package | Status updated, delivery ETA provided |
| B04-048 | State: out_for_delivery → delivered | P0 | Customer confirms receipt | Order marked delivered, payment released |
| B04-049 | State: delivered → return_requested | P0 | Customer requests return within 7 days | Return request created, pending vendor review |
| B04-050 | State: return_requested → return_approved | P0 | Vendor approves return | Return label generated, pickup scheduled |
| B04-051 | State: return_requested → return_rejected | P0 | Vendor rejects return | Reason provided, customer can appeal |
| B04-052 | State: return_approved → returned | P0 | Customer ships back item | Item received, condition verified |
| B04-053 | State: returned → refund_pending | P0 | Return verified | Refund queued for processing |
| B04-054 | State: refund_pending → refund_processed | P0 | Refund processed | Amount credited to wallet/payment method |
| B04-055 | State: delivered → review_left | P1 | Customer leaves review | Review attached to order, no further state change |
| B04-056 | State: confirmed → cancelled (customer) | P0 | Customer cancels pre-shipment order | Order cancelled, full refund |
| B04-057 | State: shipped → cancelled (customer blocked) | P0 | Customer attempts to cancel shipped order | Error: cannot cancel after shipment |
| B04-058 | State transition: invalid transition blocked | P0 | Attempting invalid state change | Error: invalid transition, state unchanged |
| B04-059 | Master order status calculation | P0 | Multi-vendor master order | Master status = worst sub-order status |
| B04-060 | Sub-order independent state progression | P0 | Master order with 3 sub-orders | Each sub-order progresses independently |
| B04-061 | Master order completion | P0 | All sub-orders delivered | Master order marked completed |
| B04-062 | Partial cancellation (one sub-order) | P0 | One sub-order cancelled | Master order continues with remaining sub-orders |
| B04-063 | State transition timestamp logging | P0 | Any state change | Timestamp, user, and action logged |
| B04-064 | State transition notification triggers | P0 | State change occurs | Appropriate notifications sent for each transition |
| B04-065 | Auto-cancel unpaid orders (24h) | P0 | Order pending payment for 24h | Order auto-cancelled, stock released |
| B04-066 | Auto-confirm unpaid orders (24h vendor response) | P1 | Order pending for 24h | Vendor response deadline, escalation if no response |
| B04-067 | Auto-deliver after 14 days (no customer action) | P0 | Order shipped for 14+ days | Auto-delivered, payment released |
| B04-068 | Return window (7 days from delivery) | P0 | Delivered order | Return requested within 7 days allowed |
| B04-069 | Return window expired | P1 | Delivered order 8+ days ago | Return request blocked |
| B04-070 | Refund wallet credit (instant) | P0 | Refund processed | Wallet balance updated instantly |
| B04-071 | Refund bank transfer (3-5 days) | P0 | Refund to bank | Refund initiated, expected completion in 3-5 days |
| B04-072 | Partial refund (damaged item) | P0 | Return with partial damage | Partial refund calculated and processed |
| B04-073 | Full refund (complete return) | P0 | Complete item return | Full amount refunded |
| B04-074 | Refund status tracking | P0 | Refund initiated | Customer can track refund status |
| B04-075 | Order status page for customer | P0 | Customer with active order | Real-time status displayed with timeline |

## 4.4 Cancellation (12 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B04-076 | Pre-payment cancellation (instant) | P0 | Order with wallet payment, not yet processed | Order cancelled, wallet not charged |
| B04-077 | Post-payment cancellation (wallet) | P0 | Wallet-paid order, vendor not shipped | Order cancelled, full wallet refund |
| B04-078 | Post-payment cancellation (wallet) | P0 | Wallet-paid order, not yet shipped | Order cancelled, refund to wallet |
| B04-079 | Partial cancellation (remove item) | P0 | Multi-item order | Item removed, order total adjusted |
| B04-080 | Partial cancellation (reduce quantity) | P0 | Multi-quantity item | Quantity reduced, total adjusted |
| B04-081 | Cancellation reason selection | P0 | Customer cancelling | Reason required from predefined list |
| B04-082 | Cancellation confirmation | P0 | Customer initiating cancel | Confirmation dialog before final cancellation |
| B04-083 | Vendor-initiated cancellation | P0 | Vendor cancelling order | Reason required, customer notified |
| B04-084 | Admin force cancellation | P0 | Admin cancelling any order | Override capability with audit log |
| B04-085 | Cancellation stock release | P0 | Order cancelled | Reserved stock released back to available |
| B04-086 | Cancellation coupon restoration | P0 | Order with coupon cancelled | Coupon usage count decremented |
| B04-087 | Cancellation loyalty points reversal | P1 | Order with loyalty points earned | Points deducted if order cancelled |

## 4.5 Delivery Code System (18 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B04-088 | Delivery code generation (6-digit) | P0 | Order shipped, out for delivery | Unique 6-digit code generated and sent to customer |
| B04-089 | Delivery code via SMS | P0 | Code generated | SMS with code sent to customer's phone |
| B04-090 | Delivery code via WhatsApp | P0 | Code generated | WhatsApp message with code sent |
| B04-091 | Delivery code via email | P1 | Code generated | Email with code sent to customer |
| B04-092 | Delivery code verification by courier | P0 | Courier at delivery location | Code verified, delivery confirmed |
| B04-093 | Delivery code incorrect entry (attempt 1/5) | P0 | Courier enters wrong code | Error: incorrect code, attempt count shown |
| B04-094 | Delivery code lockout (5 failed attempts) | P0 | 5 incorrect code attempts | Code locked, admin intervention required |
| B04-095 | Delivery code expiry (24h) | P1 | Code generated 24+ hours ago | Code expired, new code generated |
| B04-096 | Delivery code resend request | P1 | Customer requesting new code | New code generated and sent |
| B04-097 | Delivery code used confirmation | P0 | Code verified successfully | Delivery confirmed, order status updated |
| B04-098 | Delivery code reuse prevention | P0 | Already-used code | Error: code already used |
| B04-099 | Delivery code for partial delivery | P0 | Multi-item order, partial delivery | Separate codes for each delivery batch |
| B04-100 | Courier location verification (GPS) | P0 | Courier verifying delivery | GPS coordinates logged within delivery radius |
| B04-101 | Delivery proof photo capture | P0 | Delivery confirmed | Photo required, stored with delivery record |
| B04-102 | Delivery proof signature capture | P1 | Delivery confirmed | Digital signature captured |
| B04-103 | Delivery code mobile app integration | P0 | Courier using mobile app | Code entry and verification in-app |
| B04-104 | Delivery code admin override | P0 | Code locked, admin resolving | Admin can manually confirm delivery |
| B04-105 | Delivery attempt tracking | P1 | Courier attempting delivery | Failed attempts logged, rescheduling enabled |

## 4.6 Master/Sub-Order Architecture (14 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B04-106 | Master order creation from cart | P0 | Cart with items from 3 vendors | Master order with 3 sub-orders created |
| B04-107 | Sub-order linked to correct vendor | P0 | Multi-vendor order | Each sub-order shows correct vendor |
| B04-108 | Sub-order independent delivery tracking | P0 | Multi-vendor order | Each sub-order tracks delivery independently |
| B04-109 | Master order total = sum of sub-orders | P0 | Multi-vendor order | Master total equals sum of all sub-order totals |
| B04-110 | Master order discount distribution | P0 | Order with platform coupon | Discount proportionally distributed across sub-orders |
| B04-111 | Master order delivery fee calculation | P0 | Multi-vendor order | Delivery fee calculated per vendor zone |
| B04-112 | Sub-order vendor commission calculation | P0 | Sub-order delivered | Commission calculated on sub-order total |
| B04-113 | Master order invoice generation | P0 | Master order completed | Single invoice with all sub-orders listed |
| B04-114 | Sub-order invoice per vendor | P0 | Sub-order completed | Vendor receives sub-order invoice |
| B04-115 | Master order cancellation cascading | P0 | Master order cancelled | All sub-orders cancelled, refunds initiated |
| B04-116 | Sub-order independent cancellation | P0 | One sub-order cancelled | Other sub-orders unaffected |
| B04-117 | Master order status display | P0 | Customer viewing order | Combined status with per-vendor breakdown |
| B04-118 | Vendor sees only their sub-order | P0 | Vendor dashboard | Vendor views only their sub-order data |
| B04-119 | Master order return handling (per item) | P0 | Customer returning item | Return processed per sub-order/item |


---

# Block 05 - Payments & Wallet (72 Test Points)

## 5.1 Wallet Management (14 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B05-001 | Wallet creation on user registration | P0 | New user registration | Wallet created with 0 balance |
| B05-002 | Wallet balance display | P0 | User with wallet | Accurate balance displayed in dashboard |
| B05-003 | Multi-currency wallet (YER, USD) | P0 | User with wallet | Balance tracked per currency |
| B05-004 | Currency conversion display | P1 | User viewing wallet | Real-time conversion rates displayed |
| B05-005 | Wallet transaction history | P0 | User with transactions | Complete transaction history with filters |
| B05-006 | Wallet balance atomic deduction | P0 | Wallet with balance | Balance deducted atomically, no overdraft |
| B05-007 | Wallet balance atomic credit | P0 | Wallet receiving credit | Balance credited atomically |
| B05-008 | Wallet balance consistency check | P0 | Any wallet operation | Balance always matches sum of transactions |
| B05-009 | Wallet minimum balance (0 YER) | P0 | Wallet deduction attempt | Cannot go below zero |
| B05-010 | Wallet maximum balance (10,000,000 YER) | P1 | Wallet credit attempt | Cannot exceed maximum limit |
| B05-011 | Wallet deactivation | P1 | User requesting deactivation | Wallet frozen, no transactions allowed |
| B05-012 | Wallet reactivation | P1 | Deactivated wallet | Reactivation requires KYC re-verification |
| B05-013 | Wallet statement export (PDF) | P1 | User requesting statement | PDF statement generated with date range |
| B05-014 | Wallet security PIN | P0 | User enabling PIN | PIN required for transactions > 50,000 YER |

## 5.2 Wallet Top-up (14 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B05-015 | Top-up via payment gateway | P0 | User initiating top-up | Payment gateway redirect, amount specified |
| B05-016 | Top-up success callback | P0 | Gateway confirms payment | Wallet credited, transaction logged |
| B05-017 | Top-up failure handling | P0 | Gateway rejects payment | Error displayed, no wallet change |
| B05-018 | Top-up idempotency (duplicate prevention) | P0 | User double-clicking top-up | Single transaction processed |
| B05-019 | Admin manual top-up | P0 | Admin crediting wallet | Admin can add balance with reason |
| B05-020 | Admin manual deduction | P0 | Admin debiting wallet | Admin can deduct balance with reason |
| B05-021 | Top-up minimum amount (1,000 YER) | P1 | User entering small amount | Minimum enforced, error if below |
| B05-022 | Top-up maximum amount per transaction | P1 | User entering large amount | Maximum per transaction enforced |
| B05-023 | Daily top-up limit | P1 | User exceeding daily limit | Limit enforced, next-day reset |
| B05-024 | Top-up verification for large amounts | P0 | Top-up > 1,000,000 YER | Additional verification required |
| B05-025 | Top-up receipt generation | P1 | Successful top-up | Receipt generated and accessible |
| B05-026 | Top-up notification (SMS + email) | P0 | Successful top-up | Notification sent on successful top-up |
| B05-027 | Top-up pending state handling | P0 | Gateway processing | Pending state displayed, polling for status |
| B05-028 | Top-up reversal on gateway error | P0 | Gateway timeout after charge | Amount reversed if charge succeeded but callback failed |

## 5.3 Payment Processing (16 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B05-029 | Payment idempotency key enforcement | P0 | Payment request | Idempotency key required, duplicate rejected |
| B05-030 | Payment atomic operation (debit wallet) | P0 | Wallet payment | Debit is atomic: balance checked + deducted in single transaction |
| B05-031 | Payment failure state recovery | P0 | Payment fails mid-process | System returns to pre-payment state, stock released |
| B05-032 | Payment timeout handling (30s) | P0 | Gateway not responding | Timeout after 30s, order returned to pending |
| B05-033 | Payment concurrent access (race condition) | P0 | Two payments for same wallet | Only one succeeds, second gets insufficient balance |
| B05-034 | Wallet payment recording | P0 | Order paid via wallet | Payment recorded in transaction ledger |
| B05-035 | Bank transfer payment recording | P0 | Bank transfer confirmed | Payment recorded with transfer reference |
| B05-036 | Payment gateway webhook verification | P0 | Webhook received | Signature verified, payload validated |
| B05-037 | Payment receipt generation | P0 | Payment successful | Receipt with transaction ID generated |
| B05-038 | Payment reversal (admin) | P0 | Admin reversing payment | Reversal processed, wallet refunded |
| B05-039 | Payment dispute handling | P0 | Customer disputing payment | Dispute created, payment held pending review |
| B05-040 | Payment retry after failure | P1 | Failed payment | User can retry with same or different method |
| B05-041 | Multi-currency payment conversion | P0 | Payment in different currency | Converted at current rate, displayed to user |
| B05-042 | Payment audit trail | P0 | Any payment event | Complete audit trail with timestamps |
| B05-043 | Payment method validation | P0 | User selecting payment method | Method valid for user's region and order type |
| B05-044 | Payment security encryption | P0 | Payment data transmitted | All payment data encrypted in transit and at rest |

## 5.4 Escrow System (14 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B05-045 | Escrow hold on order placement | P0 | Wallet-funded order confirmed | Amount held in escrow, balance unavailable |
| B05-046 | Escrow release on delivery confirmation | P0 | Order marked delivered | Escrow released to vendor (minus commission) |
| B05-047 | Escrow hold duration (max 30 days) | P0 | Order pending delivery 30+ days | Escrow auto-released or returned to customer |
| B05-048 | Escrow release to vendor | P0 | Delivery confirmed | Vendor receives: escrow amount - commission |
| B05-049 | Escrow refund to customer | P0 | Order cancelled/refunded | Full amount returned from escrow to customer |
| B05-050 | Escrow dispute hold | P0 | Customer opens dispute | Escrow frozen until dispute resolved |
| B05-051 | Escrow dispute resolution (customer wins) | P0 | Dispute resolved in customer favor | Full refund from escrow to customer |
| B05-052 | Escrow dispute resolution (vendor wins) | P0 | Dispute resolved in vendor favor | Escrow released to vendor |
| B05-053 | Escrow balance tracking | P0 | Multiple active escrows | Total escrowed amount tracked accurately |
| B05-054 | Escrow report for finance team | P1 | Finance team viewing escrow | All escrowed amounts with order details displayed |
| B05-055 | Escrow auto-release scheduler | P0 | Escrow past auto-release date | Scheduled job releases escrow |
| B05-056 | Escrow partial release (partial delivery) | P0 | Partial order delivered | Proportional escrow released |
| B05-057 | Escrow commission deduction | P0 | Escrow release to vendor | Platform commission deducted before release |
| B05-058 | Escrow audit log | P0 | Any escrow event | All escrow operations logged immutably |

## 5.4 Refund Processing (14 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B05-059 | Refund to wallet (instant credit) | P0 | Refund processed | Wallet credited immediately |
| B05-060 | Refund amount calculation (full) | P0 | Complete return | Full order amount refunded |
| B05-061 | Refund amount calculation (partial) | P0 | Partial return | Proportional amount refunded |
| B05-062 | Refund to original payment method | P0 | Refund processed | Refunded to same method used for payment |
| B05-063 | Refund idempotency | P0 | Duplicate refund request | Only one refund processed |
| B05-064 | Refund status tracking | P0 | Refund initiated | Customer can track: pending, processing, completed |
| B05-065 | Refund notification | P0 | Refund processed | Customer notified via SMS + email |
| B05-066 | Refund report for finance | P1 | Finance team viewing refunds | Complete refund history with reasons |
| B05-067 | Refund reversal (admin) | P0 | Admin reversing incorrect refund | Refund reversed, amount deducted |
| B05-068 | Refund to wallet | P0 | Order refund | Refund credited to wallet |
| B05-069 | Refund processing time SLA (3 days) | P1 | Refund initiated | Refund completed within 3 business days |
| B05-070 | Refund with coupon restoration | P0 | Refund on coupon order | Coupon usage decremented, available again |
| B05-071 | Refund with loyalty points reversal | P1 | Refund on loyalty-earned order | Points deducted from customer account |
| B05-072 | Refund audit trail | P0 | Any refund event | Complete audit trail with timestamps and amounts |


---

# Block 06 - Finance & Accounting (70 Test Points)

## 6.1 Commission System (12 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B06-001 | Commission rate configuration per category | P0 | Admin setting rates | Rate saved per category (5-15% range) |
| B06-002 | Commission calculation on order delivery | P0 | Order delivered | Commission = sub-order total * category rate |
| B06-003 | Commission applied to correct vendor | P0 | Multi-vendor order | Commission deducted from correct vendor balance |
| B06-004 | Commission exemption for certain vendors | P1 | Admin configuring exemptions | Exempt vendors pay zero commission |
| B06-005 | Commission rate change (new orders only) | P0 | Admin changing rate | Only new orders affected, existing unaffected |
| B06-006 | Commission dispute by vendor | P0 | Vendor disputing commission | Dispute created, commission held pending review |
| B06-007 | Commission adjustment by admin | P0 | Admin correcting commission | Manual adjustment with reason documented |
| B06-008 | Commission report generation | P0 | Finance team viewing | Commission report by vendor, period, category |
| B06-009 | Commission invoice generation | P0 | Commission calculated | Invoice generated for vendor's commission |
| B06-010 | Commission tax calculation | P0 | Commission calculated | Applicable taxes calculated on commission |
| B06-011 | Commission monthly reconciliation | P1 | Month-end processing | All commissions reconciled against orders |
| B06-012 | Commission historical rate lookup | P1 | Commission query | Historical rates used for past orders |

## 6.2 Payout Processing (10 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B06-013 | Weekly payout batch processing | P0 | Week-end, vendor balances > 0 | Payout batch generated for all eligible vendors |
| B06-014 | Payout calculation: balance - pending refunds | P0 | Payout batch processing | Pending refunds deducted from payout |
| B06-015 | Payout bank transfer execution | P0 | Payout batch approved | Transfers initiated to vendor bank accounts |
| B06-016 | Payout confirmation from bank | P0 | Transfer initiated | Bank confirmation received, payout marked complete |
| B06-017 | Payout failure handling | P0 | Bank rejects transfer | Payout failed, amount returned to platform |
| B06-018 | Payout retry for failed transfers | P1 | Failed payout | Retry mechanism, max 3 attempts |
| B06-019 | Payout report for finance | P0 | Finance team viewing | Complete payout history with status |
| B06-020 | Payout statement for vendor | P1 | Vendor viewing payouts | Vendor sees their payout history |
| B06-021 | Payout tax withholding | P0 | Payout processed | Applicable tax withheld before transfer |
| B06-022 | Payout minimum threshold enforcement | P1 | Vendor balance below minimum | Payout deferred until threshold met |

## 6.3 Invoice & ZATCA Compliance (14 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B06-023 | Invoice generation on order delivery | P0 | Order delivered | Invoice created with sequential number |
| B06-024 | Invoice sequential numbering | P0 | Invoice generated | Number incremented atomically, no gaps |
| B06-025 | Invoice ZATCA compliance fields | P0 | Invoice generated | All ZATCA required fields present |
| B06-026 | Invoice QR code generation | P0 | Invoice generated | QR code containing invoice data embedded |
| B06-027 | Invoice PDF generation | P0 | Invoice generated | PDF with all required fields generated |
| B06-028 | Invoice Arabic/English bilingual | P0 | Invoice generated | Both Arabic and English text on invoice |
| B06-029 | Invoice VAT calculation | P0 | Invoice generated | VAT (5%) calculated and displayed |
| B06-030 | Invoice line item details | P0 | Invoice generated | Each item with quantity, price, tax breakdown |
| B06-031 | Invoice customer copy delivery | P0 | Invoice generated | PDF sent to customer via email |
| B06-032 | Invoice vendor copy delivery | P1 | Invoice generated | PDF sent to vendor |
| B06-033 | Invoice admin access | P0 | Admin viewing invoices | All invoices accessible with search/filter |
| B06-034 | Invoice amendment (credit note) | P0 | Refund processed | Credit note generated linked to original invoice |
| B06-035 | Invoice archive (7-year retention) | P1 | Invoice older than 1 year | Archived but retrievable |
| B06-036 | Invoice export for accounting software | P1 | Finance team exporting | CSV/JSON export compatible with accounting tools |

## 6.4 Tax Management (12 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B06-037 | VAT rate configuration (5%) | P0 | Admin setting tax rates | VAT rate saved and applied |
| B06-038 | VAT calculation on each order | P0 | Order placed | VAT calculated per item based on category |
| B06-039 | VAT-exclusive pricing display | P0 | Product page | Price shown as ex-VAT, VAT added at checkout |
| B06-040 | VAT-inclusive pricing option | P1 | Admin configuration | Toggle between exclusive/inclusive display |
| B06-041 | VAT report generation | P0 | Finance team requesting | VAT collected report by period |
| B06-042 | VAT return preparation data | P0 | Quarter-end | Data formatted for ZATCA submission |
| B06-043 | Zero-rated items handling | P0 | Products exempt from VAT | VAT = 0 for exempt items |
| B06-044 | Tax-exempt customer handling | P1 | Government entity purchasing | Tax exemption applied with certificate |
| B06-045 | Multi-governorate tax rules | P1 | Orders from different governorates | Local tax rules applied where applicable |
| B06-046 | Tax rate change history | P1 | Admin changing rates | Historical rates preserved, applied to correct periods |
| B06-047 | Tax audit trail | P0 | Any tax calculation | Complete audit trail for all tax events |
| B06-048 | Annual tax summary report | P1 | Year-end processing | Complete annual tax summary generated |

## 6.5 General Ledger (12 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B06-049 | Double-entry bookkeeping | P0 | Any financial transaction | Every transaction has debit and credit entries |
| B06-050 | Journal entry creation | P0 | Financial event | Journal entry with date, accounts, amounts |
| B06-051 | Chart of accounts maintenance | P1 | Admin managing accounts | Accounts CRUD with hierarchy |
| B06-052 | Account balance calculation | P0 | Journal entries posted | Balances calculated from entries |
| B06-053 | Trial balance generation | P1 | Finance requesting | Trial balance shows all account balances |
| B06-054 | Income statement generation | P0 | Finance requesting | Revenue, expenses, net income calculated |
| B06-055 | Balance sheet generation | P0 | Finance requesting | Assets, liabilities, equity balanced |
| B06-056 | Cash flow statement | P1 | Finance requesting | Operating, investing, financing activities |
| B06-057 | Bank reconciliation | P0 | Finance reconciling | Platform records matched against bank statements |
| B06-058 | Reconciliation discrepancy flagging | P0 | Mismatch detected | Discrepancies flagged for investigation |
| B06-059 | Financial period close | P1 | Month/quarter end | Period closed, no backdated entries allowed |
| B06-060 | Financial report export | P1 | Finance team | Reports exported as PDF/Excel |

## 6.6 Financial Controls (10 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B06-061 | Daily revenue reconciliation | P0 | Day-end processing | All orders reconciled against revenue |
| B06-062 | Discrepancy alert (amount > 0.01 YER) | P0 | Reconciliation discrepancy | Alert sent to finance team |
| B06-063 | Financial data access control | P0 | User accessing financial data | Only finance role can access |
| B06-064 | Financial data encryption at rest | P0 | Financial data stored | All financial data encrypted |
| B06-065 | Transaction integrity check | P0 | Any transaction | No orphaned or partial transactions |
| B06-066 | Daily transaction summary | P1 | Day-end processing | Summary of all transactions generated |
| B06-067 | Revenue by vendor report | P1 | Finance requesting | Revenue breakdown per vendor |
| B06-068 | Revenue by category report | P1 | Finance requesting | Revenue breakdown per category |
| B06-069 | Financial year configuration | P1 | Admin setting fiscal year | Financial year dates saved |
| B06-070 | Audit trail for all financial changes | P0 | Any financial modification | Complete, immutable audit trail |


---

# Block 07 - Shipping & Delivery (72 Test Points)

## 7.1 Delivery Provider Management (12 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B07-001 | Provider registration with credentials | P0 | Admin adding provider | Provider account created with API keys |
| B07-002 | Provider zone configuration | P0 | Provider registered | Delivery zones mapped to governorates |
| B07-003 | Provider rating system | P0 | Provider with delivery history | Rating calculated from customer feedback |
| B07-004 | Provider performance metrics | P1 | Provider with history | On-time rate, success rate, avg delivery time |
| B07-005 | Provider API integration test | P0 | Provider configured | API connectivity verified |
| B07-006 | Provider deactivation | P1 | Admin managing providers | Provider hidden from assignment, existing deliveries continue |
| B07-007 | Provider fee structure configuration | P0 | Provider registered | Base fee, per-kg fee, zone-based fees saved |
| B07-008 | Provider capacity limits | P1 | Provider configured | Max daily deliveries enforced |
| B07-009 | Provider service area mapping | P0 | Provider registered | Geographic coverage area defined |
| B07-010 | Provider SLA configuration | P1 | Provider registered | Delivery time commitments defined per zone |
| B07-011 | Provider insurance configuration | P1 | Provider registered | Coverage limits for lost/damaged items |
| B07-012 | Provider blacklist management | P1 | Admin managing providers | Problematic providers blocked |

## 7.2 Delivery Assignment (12 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B07-013 | Auto-assignment on order confirmation | P0 | Order confirmed, shipped | Best provider assigned based on zone/cost/performance |
| B07-014 | Assignment algorithm: zone match | P0 | Multiple providers available | Provider with delivery zone coverage selected |
| B07-015 | Assignment algorithm: cost optimization | P0 | Multiple zone-matched providers | Lowest cost provider selected |
| B07-016 | Assignment algorithm: performance weighting | P1 | Multiple providers available | Higher-rated providers prioritized |
| B07-017 | Manual override by admin | P0 | Order with assigned provider | Admin can reassign to different provider |
| B07-018 | Manual override by vendor | P1 | Order with assigned provider | Vendor can request provider change |
| B07-019 | Assignment failure handling | P0 | No provider available | Order held, admin notified for manual assignment |
| B07-020 | Multi-vendor order assignment | P0 | Order with items from multiple vendors | Each sub-order assigned to appropriate provider |
| B07-021 | Assignment notification to provider | P0 | Provider assigned | Provider notified with order details |
| B07-022 | Assignment notification to customer | P0 | Provider assigned | Customer receives provider name and tracking info |
| B07-023 | Reassignment tracking | P1 | Provider reassigned | Both providers notified, audit trail maintained |
| B07-024 | Provider acceptance/rejection | P0 | Provider assigned | Provider can accept or reject assignment |

## 7.3 Delivery Operations (16 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B07-025 | Delivery code generation | P0 | Order out for delivery | 6-digit code generated and sent to customer |
| B07-026 | Delivery code verification | P0 | Courier at delivery | Code verified, delivery confirmed |
| B07-027 | Delivery code failed attempts (5 max) | P0 | Courier entering wrong code | Locked after 5 failures |
| B07-028 | Delivery proof: photo required | P0 | Delivery confirmed | Photo of delivered package required |
| B07-029 | Delivery proof: signature capture | P0 | Delivery confirmed | Customer signature captured |
| B07-030 | Delivery proof: GPS coordinates | P0 | Delivery confirmed | Courier GPS coordinates logged |
| B07-031 | Delivery attempt tracking | P1 | Courier attempting delivery | Failed attempts logged, rescheduling enabled |
| B07-032 | Delivery rescheduling | P1 | Failed delivery attempt | Customer can reschedule delivery |
| B07-033 | Delivery location verification | P0 | Delivery confirmed | GPS coordinates within delivery radius |
| B07-034 | Delivery time tracking | P0 | Delivery in progress | Time from pickup to delivery tracked |
| B07-035 | Delivery status updates to customer | P0 | Status change | Customer receives real-time status updates |
| B07-036 | Delivery status updates to vendor | P1 | Status change | Vendor sees delivery status in dashboard |
| B07-037 | Delivery failure: customer not available | P0 | Courier at location, customer absent | Failed attempt logged, notification sent |
| B07-038 | Delivery failure: wrong address | P0 | Courier cannot locate address | Failed attempt logged, address flagged |
| B07-039 | Delivery failure: package damaged | P0 | Package damaged in transit | Incident logged, replacement/refund initiated |
| B07-040 | Delivery success confirmation | P0 | Delivery verified | Order status updated to delivered |

## 7.4 Delivery Tracking (8 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B07-041 | Real-time tracking page | P0 | Order shipped | Customer can track delivery in real-time |
| B07-042 | Tracking timeline display | P0 | Order with tracking events | Timeline shows all events with timestamps |
| B07-043 | Estimated delivery date | P0 | Order shipped | ETA displayed based on provider SLA |
| B07-044 | Delivery notification (SMS) | P0 | Status change | SMS notification sent to customer |
| B07-045 | Delivery notification (email) | P0 | Status change | Email notification sent to customer |
| B07-046 | Delivery notification (push) | P1 | Status change | Push notification sent to app |
| B07-047 | Tracking link sharing | P1 | Order shipped | Customer can share tracking link |
| B07-048 | Tracking data retention (90 days) | P1 | Tracking data older than 90 days | Archived, accessible on request |

## 7.5 Return Pickup (12 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B07-049 | Return pickup scheduling | P0 | Return approved | Pickup scheduled with date/time window |
| B07-050 | Return pickup provider assignment | P0 | Return scheduled | Provider assigned for pickup |
| B07-051 | Return pickup confirmation | P0 | Courier at pickup location | Pickup confirmed with item count |
| B07-052 | Return item condition verification | P0 | Item picked up | Condition documented (photos + notes) |
| B07-053 | Return pickup failed (item not available) | P1 | Courier at pickup, item missing | Failed pickup logged, customer contacted |
| B07-054 | Return item packaging requirements | P1 | Customer returning item | Packaging requirements communicated |
| B07-055 | Return tracking to vendor | P0 | Return in transit | Vendor can track return shipment |
| B07-056 | Return received confirmation | P0 | Return delivered to vendor | Return confirmed received |
| B07-057 | Return inspection by vendor | P0 | Return received | Vendor inspects and confirms condition |
| B07-058 | Return acceptance by vendor | P0 | Inspection complete | Return accepted, refund initiated |
| B07-059 | Return rejection by vendor | P0 | Inspection complete | Return rejected with reason, item returned to customer |
| B07-060 | Return shipping cost responsibility | P0 | Return processed | Cost attributed based on return reason |

## 7.6 Delivery Zone Management (12 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B07-061 | Zone definition by governorate | P0 | Admin configuring zones | Zones mapped to governorates |
| B07-062 | Zone definition by district | P0 | Admin configuring zones | Sub-zones within governorates defined |
| B07-063 | Zone-based pricing | P0 | Zones configured | Different delivery fees per zone |
| B07-064 | Zone availability check | P0 | Customer entering address | System checks delivery availability |
| B07-065 | Zone-specific delivery time | P0 | Zone selected | Estimated delivery time per zone displayed |
| B07-066 | Zone exclusion (no delivery areas) | P0 | Admin configuring zones | Areas marked as undeliverable |
| B07-067 | Zone overlap resolution | P1 | Overlapping zones defined | Clear precedence rules applied |
| B07-068 | Zone pricing update | P0 | Admin updating fees | New prices apply to new orders only |
| B07-069 | Zone coverage by provider | P0 | Provider and zones configured | Provider linked to covered zones |
| B07-070 | Zone performance reporting | P1 | Finance viewing | Delivery metrics per zone |
| B07-071 | Zone expansion by admin | P1 | Admin adding new areas | New zones available immediately |
| B07-072 | Zone deprecation | P1 | Admin removing zones | Deprecated zones hidden from selection |


---

# Block 08 - Inventory Management (72 Test Points)

## 8.1 Stock Management (14 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B08-001 | Stock quantity update by vendor | P0 | Vendor with product | Stock quantity updated, reflected on storefront |
| B08-002 | Stock decrement on order confirmation | P0 | Order confirmed | Stock reduced atomically |
| B08-003 | Stock restoration on cancellation | P0 | Order cancelled | Stock restored to available pool |
| B08-004 | Stock restoration on return | P0 | Return processed | Stock restored based on item condition |
| B08-005 | Stock check before add to cart | P0 | Customer adding item | Available stock validated before cart addition |
| B08-006 | Stock check at checkout | P0 | Customer checking out | Final stock validation before order creation |
| B08-007 | Stock display on product page | P0 | Product in stock | Accurate stock level displayed |
| B08-008 | Low stock threshold configuration | P1 | Vendor setting thresholds | Custom low-stock alert threshold saved |
| B08-009 | Out-of-stock product handling | P0 | Product with zero stock | "Out of Stock" badge, add-to-cart disabled |
| B08-010 | Back-in-stock notification | P1 | Out-of-stock product | Customers notified when stock replenished |
| B08-011 | Stock adjustment audit trail | P0 | Any stock change | All adjustments logged with reason |
| B08-012 | Stock export for reconciliation | P1 | Vendor requesting export | CSV/Excel export of all stock levels |
| B08-013 | Stock import via CSV | P1 | Vendor importing stock | Bulk stock update via CSV upload |
| B08-014 | Negative stock prevention | P0 | Stock deduction attempt | Cannot reduce below zero |

## 8.2 Stock Reservation (12 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B08-015 | 15-minute soft hold on checkout | P0 | Customer in checkout | Stock reserved for 15 minutes |
| B08-016 | Reservation expiry and release | P0 | Reservation past 15 minutes | Stock released back to available pool |
| B08-017 | Reservation on order placement | P0 | Order placed | Stock converted from soft hold to hard reserve |
| B08-018 | Hard reserve on confirmed order | P0 | Order confirmed | Stock deducted from available |
| B08-019 | Reservation cleanup scheduler | P0 | Expired reservations | Background job cleans up expired holds |
| B08-020 | Reservation for multi-item order | P0 | Order with 5 items | All 5 items reserved atomically |
| B08-021 | Reservation failure (insufficient stock) | P0 | Item stock below cart quantity | Reservation failed, customer notified |
| B08-022 | Concurrent reservation handling | P0 | Two users reserving same stock | Only one succeeds, other gets stock error |
| B08-023 | Reservation visibility (reserved stock not available) | P0 | Stock reserved for order | Reserved stock hidden from available count |
| B08-024 | Reservation extension on checkout delay | P1 | Checkout taking > 15 minutes | Automatic extension if user actively checkout |
| B08-025 | Reservation cancellation on cart clear | P0 | User clearing cart | Reserved stock released |
| B08-026 | Reservation reporting | P1 | Admin viewing reservations | All active reservations displayed |

## 8.3 Multi-Vendor Stock (12 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B08-027 | Vendor stock segregation | P0 | Multiple vendors | Each vendor sees only their stock |
| B08-028 | Vendor cannot modify other vendor stock | P0 | Vendor attempting cross-vendor edit | Access denied, 403 Forbidden |
| B08-029 | Multi-vendor stock check for same product | P0 | Same product from different vendors | Stock checked per vendor |
| B08-030 | Vendor stock reporting | P0 | Vendor requesting report | Accurate stock levels for their products |
| B08-031 | Admin stock visibility across vendors | P0 | Admin viewing stock | All vendor stock visible to admin |
| B08-032 | Vendor stock threshold alerts | P1 | Vendor stock below threshold | Vendor notified of low stock |
| B08-033 | Vendor stock update via API | P1 | Vendor with API access | Stock updated via API call |
| B08-034 | Vendor stock history | P1 | Vendor viewing stock history | Complete history of stock changes |
| B08-035 | Stock transfer between vendors | P1 | Admin transferring stock | Stock moved between vendor inventories |
| B08-036 | Vendor stock freeze (admin) | P0 | Admin freezing vendor stock | Vendor cannot modify frozen stock |
| B08-037 | Vendor stock reconciliation | P1 | Finance reconciling | Stock matches order quantities |
| B08-038 | Vendor stock export | P1 | Vendor requesting export | Stock data exported for reconciliation |

## 8.4 Stock Alerts & Notifications (8 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B08-039 | Low stock alert (below threshold) | P0 | Stock below configured threshold | Alert sent to vendor |
| B08-040 | Critical stock alert (below 5 units) | P0 | Stock below 5 units | Urgent alert sent to vendor |
| B08-041 | Out-of-stock alert | P0 | Stock reaches zero | Alert sent to vendor + admin |
| B08-042 | Reorder suggestion | P1 | Low stock detected | Reorder quantity suggested based on sales velocity |
| B08-043 | Restock notification to customers | P1 | Product restocked | Back-in-stock notifications sent |
| B08-044 | Stock alert email delivery | P0 | Alert triggered | Email delivered within 1 minute |
| B08-045 | Stock alert SMS delivery | P1 | Alert triggered | SMS delivered within 1 minute |
| B08-046 | Stock alert aggregation (not spam) | P1 | Multiple low-stock alerts | Alerts aggregated, sent as daily digest |

## 8.5 Purchase Orders (12 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B08-047 | Purchase order creation | P0 | Vendor creating PO | PO created with supplier details |
| B08-048 | Purchase order item specification | P0 | Vendor creating PO | Items, quantities, unit prices specified |
| B08-049 | Purchase order approval | P0 | PO submitted | PO approved by admin |
| B08-050 | Purchase order status tracking | P0 | PO created | Status: draft → submitted → approved → received |
| B08-051 | Purchase order receiving | P0 | Physical stock received | PO marked as received, stock updated |
| B08-052 | Partial receiving | P0 | Partial shipment received | PO partially received, remaining pending |
| B08-053 | Receiving discrepancy flagging | P0 | Received qty != ordered qty | Discrepancy flagged for investigation |
| B08-054 | Purchase order PDF generation | P1 | PO created | PDF generated for supplier |
| B08-055 | Purchase order history | P1 | Vendor viewing POs | Complete PO history displayed |
| B08-056 | Purchase order cancellation | P0 | PO in draft/submitted | PO cancelled, supplier notified |
| B08-057 | Purchase order cost tracking | P1 | PO received | Total cost tracked against budget |
| B08-058 | Purchase order auto-stock update | P0 | PO fully received | Stock levels updated automatically |

## 8.6 Stock Counts (8 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B08-059 | Physical stock count initiation | P0 | Admin starting count | Count session created |
| B08-060 | Stock count by vendor | P0 | Vendor counting stock | Vendor counts their own inventory |
| B08-061 | Stock count by product | P0 | Count in progress | Count entered per product |
| B08-062 | Stock count variance detection | P0 | Count completed | Variance between system and physical count flagged |
| B08-063 | Stock count reconciliation | P0 | Variance detected | Admin reviews and approves adjustment |
| B08-064 | Stock count approval workflow | P0 | Variance submitted | Admin approves/rejects adjustment |
| B08-065 | Stock count audit trail | P0 | Count completed | Complete audit trail with timestamps |
| B08-066 | Stock count scheduling | P1 | Admin scheduling | Recurring count schedules configured |

## 8.7 Inventory Analytics (8 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B08-067 | Stock turnover rate calculation | P1 | Vendor with sales history | Turnover rate calculated |
| B08-068 | Days of stock remaining | P1 | Vendor with sales data | Estimated days until stockout |
| B08-069 | Dead stock identification | P1 | Products with no sales in 90 days | Dead stock items flagged |
| B08-070 | Stock valuation report | P1 | Finance requesting | Total inventory value calculated |
| B08-071 | Stock movement history | P0 | Vendor viewing product | Complete history of stock ins/outs |
| B08-072 | Inventory aging report | P1 | Finance requesting | Stock grouped by age |
| B08-073 | Demand forecasting suggestion | P1 | Vendor with sales history | Reorder quantity suggested |
| B08-074 | Stock accuracy report | P1 | Admin viewing | Accuracy percentage by vendor/category |


---

# Block 09 - Customer Storefront (74 Test Points)

## 9.1 Guest Browsing (10 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B09-001 | Guest can browse products | P0 | No authentication | Products visible without login |
| B09-002 | Guest can search products | P0 | No authentication | Search results displayed |
| B09-003 | Guest can view product details | P0 | No authentication | Product page with full details displayed |
| B09-004 | Guest can filter by category | P0 | No authentication | Category filter applied correctly |
| B09-005 | Guest can sort products | P1 | No authentication | Sort options work correctly |
| B09-006 | Guest can view store pages | P0 | No authentication | Vendor store pages visible |
| B09-007 | Guest sees promotional banners | P1 | No authentication | Active promotions displayed |
| B09-008 | Guest session tracking (anonymous) | P1 | Guest browsing | Anonymous session tracked for analytics |
| B09-009 | Guest search autocomplete | P1 | Guest typing in search | Suggestions appear after 2 characters |
| B09-010 | Guest browsing speed (< 2s load) | P0 | Guest loading pages | Pages load within 2 seconds |

## 9.2 Product Search & Filtering (14 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B09-011 | Full-text product search | P0 | Products in catalog | Relevant results returned |
| B09-012 | Search by product name | P0 | User searching | Results match product name |
| B09-013 | Search by brand | P1 | User searching | Results filtered by brand |
| B09-014 | Search by price range (min-max) | P0 | User filtering | Products within price range shown |
| B09-015 | Search by category | P0 | User filtering | Products in selected category |
| B09-016 | Search by vendor/store | P1 | User filtering | Products from specific vendor |
| B09-017 | Search by availability | P1 | User filtering | Only in-stock products shown |
| B09-018 | Search by rating | P1 | User filtering | Products with minimum rating |
| B09-019 | Search sort: relevance | P0 | User sorting | Results sorted by relevance |
| B09-020 | Search sort: price (low to high) | P0 | User sorting | Results sorted by price ascending |
| B09-021 | Search sort: price (high to low) | P0 | User sorting | Results sorted by price descending |
| B09-022 | Search sort: newest | P1 | User sorting | Newest products first |
| B09-023 | Search sort: rating | P1 | User sorting | Highest rated first |
| B09-024 | Search results pagination | P0 | Many results | Pagination with 20 items per page |

## 9.3 Cart & Session (12 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B09-025 | Add to cart (authenticated) | P0 | Logged-in user | Item added, cart badge updated |
| B09-026 | Add to cart (guest) | P0 | Guest user | Item added to session cart |
| B09-027 | Cart badge count display | P0 | User with items in cart | Accurate item count shown |
| B09-028 | Cart sidebar/drawer | P1 | User clicking cart icon | Cart preview shown without page navigation |
| B09-029 | View full cart page | P0 | User clicking view cart | Full cart page displayed |
| B09-030 | Update quantity in cart | P0 | Item in cart | Quantity updated, total recalculated |
| B09-031 | Remove item from cart | P0 | Item in cart | Item removed, total recalculated |
| B09-032 | Cart persistence (cookie/localStorage) | P0 | Guest user | Cart persisted across page refreshes |
| B09-033 | Cart migration on login | P0 | Guest with cart, logging in | Guest cart merged with account cart |
| B09-034 | Cart expiry after 7 days (guest) | P1 | Guest cart inactive 7+ days | Cart cleared |
| B09-035 | Cart minimum order value message | P1 | Cart below minimum | Message displayed about minimum order |
| B09-036 | Cart delivery estimate display | P0 | Cart with items | Estimated delivery time shown |

## 9.4 Checkout Experience (14 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B09-037 | Checkout requires login/registration | P0 | Guest at checkout | Redirect to login/register |
| B09-038 | Saved address selection | P0 | Logged-in user with addresses | Saved addresses available for selection |
| B09-039 | New address entry during checkout | P0 | User entering new address | Address validated and saved |
| B09-040 | Address edit during checkout | P1 | User with saved address | Address can be edited inline |
| B09-041 | Delivery method selection | P0 | Address confirmed | Available delivery options shown |
| B09-042 | Delivery fee display | P0 | Delivery method selected | Fee shown in order summary |
| B09-043 | Payment method selection | P0 | Delivery selected | Payment options displayed |
| B09-044 | Order summary review | P0 | Payment selected | Full breakdown shown before confirmation |
| B09-045 | Apply coupon during checkout | P0 | User with coupon code | Discount applied, total updated |
| B09-046 | Wallet balance display during checkout | P0 | User with wallet | Balance shown, option to use for payment |
| B09-047 | Order confirmation page | P0 | Order placed | Confirmation with order ID and summary |
| B09-048 | Checkout progress indicator | P1 | User in checkout | Step indicator shows current position |
| B09-049 | Checkout error messages | P0 | Validation error | Clear error message for each field |
| B09-050 | Checkout mobile optimization | P0 | Mobile device | Checkout fully functional on mobile |

## 9.5 Store Directory (12 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B09-051 | Store directory page | P0 | Customer browsing | All active stores listed |
| B09-052 | Store search by name | P1 | Customer searching | Stores matching name shown |
| B09-053 | Store filter by category | P1 | Customer filtering | Stores with relevant products shown |
| B09-054 | Store filter by location | P1 | Customer filtering | Stores in selected area shown |
| B09-055 | Store page rendering | P0 | Customer clicking store | Store page loads with branding |
| B09-056 | Store product listing | P0 | Customer on store page | All store products displayed |
| B09-057 | Store follow/unfollow | P1 | Logged-in customer | Can follow/unfollow stores |
| B09-058 | Followed stores feed | P1 | Customer following stores | Products from followed stores highlighted |
| B09-059 | Store ratings display | P0 | Customer on store page | Store rating shown |
| B09-060 | Store reviews display | P1 | Customer on store page | Recent reviews displayed |
| B09-061 | Store contact information | P0 | Customer on store page | Contact details visible |
| B09-062 | Store template rendering | P0 | Customer on store page | Store template applied correctly |

## 9.6 Wishlist (8 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B09-063 | Add product to wishlist | P0 | Logged-in user | Product added to wishlist |
| B09-064 | Remove from wishlist | P0 | Product in wishlist | Product removed |
| B09-065 | View wishlist page | P0 | User with wishlist items | All saved products displayed |
| B09-066 | Move wishlist item to cart | P0 | Item in wishlist | Item moved to cart |
| B09-067 | Wishlist item price change notification | P1 | Saved product price changed | Notification sent |
| B09-068 | Wishlist item out-of-stock handling | P1 | Saved product out of stock | "Out of Stock" badge on wishlist item |
| B09-069 | Wishlist persistence across sessions | P0 | User logging in from different device | Wishlist synchronized |
| B09-070 | Wishlist item count display | P1 | User with wishlist | Count shown in navigation |

## 9.7 Homepage & Navigation (4 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B09-071 | Homepage loads correctly | P0 | Customer visiting site | Homepage renders with all sections |
| B09-072 | Category navigation menu | P0 | Customer browsing | Category menu with sub-categories |
| B09-073 | Featured products display | P1 | Admin configuring featured | Featured products shown on homepage |
| B09-074 | Responsive design (mobile/tablet) | P0 | Any device | Layout adapts to screen size |


---

# Block 10 - Reviews & Loyalty (74 Test Points)

## 10.1 Reviews Submission (14 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B10-001 | Review submission after delivery | P0 | Order delivered, within 30 days | Review form accessible |
| B10-002 | Review with rating (1-5 stars) | P0 | Review form | Rating saved with review |
| B10-003 | Review with text comment | P0 | Review form | Comment text saved |
| B10-004 | Review with photo upload | P1 | Review form | Photo attached to review |
| B10-005 | Review with video upload | P2 | Review form | Video attached to review |
| B10-006 | One review per order per customer | P0 | Customer reviewing | Duplicate review blocked |
| B10-007 | Review edit within 24 hours | P1 | Customer with review | Review editable within 24h |
| B10-008 | Review delete by author | P1 | Customer with review | Review deleted, rating removed |
| B10-009 | Review for specific variant | P0 | Product with variants | Review linked to purchased variant |
| B10-010 | Review submission notification to vendor | P0 | Review submitted | Vendor notified of new review |
| B10-011 | Review submission notification to admin | P1 | Review submitted | Admin notified for moderation |
| B10-012 | Review anonymous option | P1 | Customer submitting review | Option to hide name, show "Anonymous" |
| B10-013 | Review helpfulness voting | P1 | Other customers viewing review | Helpful/not helpful voting works |
| B10-014 | Review sorting (newest, helpful, rating) | P1 | Customer viewing reviews | Sort options functional |

## 10.2 Review Moderation (12 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B10-015 | Auto-moderation (profanity filter) | P0 | Review with profanity | Review flagged for manual review |
| B10-016 | Auto-moderation (spam detection) | P0 | Spam review | Review auto-rejected |
| B10-017 | Manual moderation approval | P0 | Flagged review | Admin approves, review published |
| B10-018 | Manual moderation rejection | P0 | Flagged review | Admin rejects, author notified |
| B10-019 | Moderation queue management | P0 | Multiple flagged reviews | Queue sorted by date, manageable |
| B10-020 | Review content policy (no competitor refs) | P1 | Review mentioning competitors | Flagged for moderation |
| B10-021 | Review language detection (Arabic/English) | P1 | Review submitted | Language detected, appropriate filtering |
| B10-022 | Review reporting by customers | P1 | Inappropriate review | Report submitted, review flagged |
| B10-023 | Review appeal process | P1 | Rejected review | Author can appeal rejection |
| B10-024 | Moderation audit trail | P0 | Any moderation action | Action logged with admin ID |
| B10-025 | Bulk moderation actions | P1 | Multiple reviews | Approve/reject multiple at once |
| B10-026 | Moderation SLA (24h response) | P1 | Review pending moderation | Review processed within 24h |

## 10.3 Vendor Reply (8 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B10-027 | Vendor can reply to reviews | P0 | Vendor with published review | Reply form accessible |
| B10-028 | Vendor reply published immediately | P1 | Vendor submitting reply | Reply visible without moderation |
| B10-029 | Vendor reply edit within 24h | P1 | Vendor with reply | Reply editable within 24h |
| B10-030 | Vendor reply character limit (500) | P1 | Vendor submitting reply | Limit enforced |
| B10-031 | Customer notification on vendor reply | P0 | Vendor replying | Customer notified of reply |
| B10-032 | Vendor reply display on review | P0 | Vendor has replied | Reply shown below review |
| B10-033 | Vendor cannot reply to own reviews | P1 | Vendor attempting self-reply | Blocked |
| B10-034 | Admin can remove vendor replies | P0 | Inappropriate reply | Admin removes reply |

## 10.4 Ratings System (10 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B10-035 | Average rating calculation | P0 | Product with reviews | Average = sum(ratings) / count |
| B10-036 | Rating distribution display | P0 | Product with reviews | 5-star to 1-star distribution shown |
| B10-037 | Rating update on new review | P0 | New review submitted | Average recalculated instantly |
| B10-038 | Rating update on review edit | P1 | Review edited with new rating | Average recalculated |
| B10-039 | Rating update on review deletion | P1 | Review deleted | Average recalculated without deleted review |
| B10-040 | Minimum reviews for rating display | P1 | Product with < 3 reviews | "Not enough reviews" shown |
| B10-041 | Rating sort on category page | P0 | Category page | Products sortable by rating |
| B10-042 | Vendor average rating calculation | P0 | Vendor with reviews | Vendor rating = avg of all product ratings |
| B10-043 | Store average rating display | P0 | Vendor store page | Store rating displayed |
| B10-044 | Rating filtering (minimum stars) | P1 | Customer filtering reviews | Reviews filtered by star count |

## 10.5 Loyalty Points System (16 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B10-045 | Points earning on order delivery | P0 | Order delivered | Points = order total * earning rate |
| B10-046 | Points earning rate configuration | P0 | Admin setting rate | Rate saved (e.g., 1 point per 100 YER) |
| B10-047 | Points balance display | P0 | User with points | Current balance shown in dashboard |
| B10-048 | Points redemption at checkout | P0 | User with points | Option to use points for discount |
| B10-049 | Points redemption value calculation | P0 | User redeeming points | Value = points * redemption rate |
| B10-050 | Minimum points for redemption (100) | P1 | User with < 100 points | Redemption option disabled |
| B10-051 | Maximum redemption per order (50%) | P0 | User redeeming points | Cannot exceed 50% of order total |
| B10-052 | Points expiry (12 months) | P1 | Points older than 12 months | Expired points removed |
| B10-053 | Points transaction history | P0 | User with points | Complete history of earning/redemption |
| B10-054 | Points on cancelled order reversal | P0 | Order cancelled | Points deducted back |
| B10-055 | Points on returned order reversal | P0 | Order returned | Points deducted back |
| B10-056 | Points notification (earned) | P0 | Points earned | Notification sent with amount |
| B10-057 | Points notification (redeemed) | P0 | Points redeemed | Notification sent with amount |
| B10-058 | Points notification (expiring) | P1 | Points expiring in 30 days | Warning notification sent |
| B10-059 | Points non-transferable | P0 | User attempting transfer | Transfer blocked |
| B10-060 | Points earning on wallet top-up | P1 | Wallet top-up completed | Bonus points awarded |

## 10.6 Tier System (8 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B10-061 | Tier levels: Bronze, Silver, Gold, Platinum | P0 | Tier system configured | 4 tiers available |
| B10-062 | Tier upgrade criteria | P0 | Customer with purchase history | Tier upgraded based on total spend |
| B10-063 | Tier benefits display | P0 | Customer viewing profile | Current tier and benefits shown |
| B10-064 | Tier-specific discounts | P1 | Tier customer | Tier-specific pricing applied |
| B10-065 | Tier-specific early access | P1 | Tier customer | Early access to sales for higher tiers |
| B10-066 | Tier downgrade after inactivity (6 months) | P1 | Tier customer inactive 6 months | Tier reviewed for downgrade |
| B10-067 | Tier upgrade notification | P0 | Customer upgraded | Notification sent with new benefits |
| B10-068 | Tier display on vendor page | P1 | Customer with tier | Tier badge shown to vendor |

## 10.7 Referral System (6 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B10-069 | Referral code generation | P0 | User requesting referral code | Unique code generated |
| B10-070 | Referral code sharing | P1 | User with referral code | Code shareable via link/SMS |
| B10-071 | Referral bonus on friend's first order | P0 | Friend uses code, completes order | Both parties receive bonus |
| B10-072 | Referral bonus amount configuration | P0 | Admin setting bonus | Amount saved (e.g., 5,000 YER each) |
| B10-073 | Referral tracking | P1 | Referral used | Referrer sees successful referrals |
| B10-074 | Referral fraud prevention (self-referral) | P0 | User referring themselves | Blocked, same IP/device detected |


---

# Block 11 - Content & Notifications (72 Test Points)

## 11.1 CMS - Page Management (12 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B11-001 | CMS page creation | P0 | Admin creating page | Page created with title, content, slug |
| B11-002 | CMS page editing | P0 | Admin editing page | Content updated, preview available |
| B11-003 | CMS page publishing | P0 | Page in draft | Page published, accessible via URL |
| B11-004 | CMS page unpublishing | P0 | Published page | Page hidden from public |
| B11-005 | CMS page deletion | P0 | Admin deleting page | Page removed, redirect configured |
| B11-006 | CMS page slug uniqueness | P0 | Admin creating page | Slug checked for uniqueness |
| B11-007 | CMS page SEO fields | P1 | Admin creating page | Meta title, description, keywords saved |
| B11-008 | CMS page ordering | P1 | Admin managing pages | Pages display in configured order |
| B11-009 | CMS page template selection | P1 | Admin creating page | Template applied to page layout |
| B11-010 | CMS page version history | P1 | Page edited multiple times | Previous versions accessible |
| B11-011 | CMS page autosave | P1 | Admin editing page | Content autosaved every 30 seconds |
| B11-012 | CMS page preview | P1 | Admin editing page | Preview renders current draft |

## 11.2 Banner Management (8 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B11-013 | Banner creation with image | P0 | Admin creating banner | Image uploaded, banner created |
| B11-014 | Banner scheduling (start/end date) | P0 | Admin configuring banner | Banner activates/deactivates per schedule |
| B11-015 | Banner placement configuration | P0 | Admin creating banner | Placement (homepage, category, etc.) set |
| B11-016 | Banner link/click-through URL | P1 | Admin creating banner | Click navigates to configured URL |
| B11-017 | Banner priority/ordering | P1 | Multiple banners | Banners display in priority order |
| B11-018 | Banner A/B testing | P2 | Admin creating banners | Two versions shown to different users |
| B11-019 | Banner analytics (impressions, clicks) | P1 | Banner active | Impression and click counts tracked |
| B11-020 | Banner responsive images | P1 | Admin uploading banner | Different sizes for desktop/mobile |

## 11.3 Notification System (18 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B11-021 | SMS notification delivery | P0 | Notification triggered | SMS delivered within 30 seconds |
| B11-022 | Email notification delivery | P0 | Notification triggered | Email delivered within 60 seconds |
| B11-023 | WhatsApp notification delivery | P0 | Notification triggered | WhatsApp message delivered |
| B11-024 | Push notification delivery | P1 | Notification triggered | Push notification delivered |
| B11-025 | Notification retry on failure | P0 | Delivery failed | Retry 3 times with exponential backoff |
| B11-026 | Notification preference management | P0 | User managing preferences | User can enable/disable channels |
| B11-027 | Notification template variable substitution | P0 | Template with variables | Variables replaced with actual values |
| B11-028 | Notification Arabic/English localization | P0 | Notification triggered | Sent in user's preferred language |
| B11-029 | Notification delivery status tracking | P0 | Notification sent | Status: sent, delivered, failed tracked |
| B11-030 | Notification history per user | P1 | User viewing notifications | Complete history accessible |
| B11-031 | Notification read/unread status | P0 | User with notifications | Read status tracked |
| B11-032 | Notification batch sending | P1 | Admin sending bulk | Batch processed in background |
| B11-033 | Notification rate limiting | P0 | User receiving notifications | Max 5 SMS/hour, max 10 email/hour |
| B11-034 | Notification unsubscribe (email) | P0 | User clicking unsubscribe | Email removed from future emails |
| B11-035 | Notification block (SMS) | P1 | User blocking SMS | No further SMS notifications |
| B11-036 | Notification delivery report | P0 | Admin viewing | Delivery rates by channel displayed |
| B11-037 | Notification failure alert | P0 | High failure rate | Admin alerted to delivery issues |
| B11-038 | Notification queue management | P0 | High volume | Queue processed in order, no messages lost |

## 11.4 Notification Templates (8 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B11-039 | Template creation | P0 | Admin creating template | Template with subject, body, variables created |
| B11-040 | Template editing | P0 | Admin editing template | Template updated |
| B11-041 | Template preview | P1 | Admin editing template | Preview with sample data shown |
| B11-042 | Template variable validation | P0 | Template with variables | Required variables validated |
| B11-043 | Template Arabic version | P0 | Template created | Arabic version saved separately |
| B11-044 | Template English version | P0 | Template created | English version saved separately |
| B11-045 | Template activation/deactivation | P0 | Admin managing templates | Templates can be enabled/disabled |
| B11-046 | Template audit trail | P0 | Template modified | All changes logged |

## 11.5 Content Delivery (8 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B11-047 | Static page caching | P0 | CMS page published | Page served from cache |
| B11-048 | Cache invalidation on update | P0 | CMS page updated | Cache cleared, fresh content served |
| B11-049 | CDN delivery for media | P0 | Banner/page with images | Images served via CDN |
| B11-050 | Image optimization (WebP) | P1 | Image uploaded | Converted to WebP for faster delivery |
| B11-051 | Content loading speed (< 2s) | P0 | Customer loading page | Page loads within 2 seconds |
| B11-052 | Content accessibility (WCAG 2.1) | P1 | CMS page published | Meets WCAG 2.1 AA standards |
| B11-053 | Content mobile responsiveness | P0 | CMS page on mobile | Layout adapts to mobile |
| B11-054 | Content search indexing | P1 | CMS page published | Indexed for internal search |

## 11.6 Bulk Operations (10 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B11-055 | Bulk email sending (max 1000/batch) | P0 | Admin sending campaign | Batch processed in background |
| B11-056 | Bulk SMS sending (max 500/batch) | P0 | Admin sending campaign | Batch processed in background |
| B11-057 | Bulk sending rate limiting | P0 | Bulk operation | Rate limits enforced per batch |
| B11-058 | Bulk sending progress tracking | P1 | Bulk operation in progress | Progress bar with completion % |
| B11-059 | Bulk sending failure handling | P0 | Some deliveries fail | Failed items logged, retry scheduled |
| B11-060 | Bulk sending report | P1 | Bulk operation completed | Report with delivery statistics |
| B11-061 | Bulk template assignment | P1 | Admin sending bulk | Template selected for batch |
| B11-062 | Bulk audience segmentation | P1 | Admin targeting users | Users filtered by segment |
| B11-063 | Bulk scheduling (future send) | P1 | Admin scheduling campaign | Campaign queued for future |
| B11-064 | Bulk cancellation | P1 | Scheduled campaign | Campaign cancelled before sending |

## 11.7 SEO & Metadata (8 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B11-065 | Meta title configuration | P0 | Admin creating page | Title saved, displayed in browser tab |
| B11-066 | Meta description configuration | P0 | Admin creating page | Description saved, shown in search results |
| B11-067 | Canonical URL configuration | P1 | Admin creating page | Canonical URL set |
| B11-068 | Open Graph tags | P1 | Admin creating page | OG tags for social sharing |
| B11-069 | Structured data (JSON-LD) | P1 | Product page | Schema.org markup embedded |
| B11-070 | Sitemap auto-generation | P0 | CMS pages published | Sitemap.xml updated automatically |
| B11-071 | Robots.txt management | P1 | Admin managing SEO | Robots.txt configurable |
| B11-072 | URL redirect management | P0 | Page URL changed | 301 redirect from old to new URL |


---

# Block 12 - Support & Analytics (79 Test Points)

## 12.1 Support Ticket System (18 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B12-001 | Ticket creation by customer | P0 | Logged-in customer | Ticket created with subject, description, category |
| B12-002 | Ticket creation by vendor | P0 | Logged-in vendor | Ticket created for platform issues |
| B12-003 | Ticket category selection | P0 | Creating ticket | Category: Order, Payment, Delivery, Account, Other |
| B12-004 | Ticket priority assignment | P0 | Creating ticket | Priority: Low, Medium, High, Critical |
| B12-005 | Ticket attachment upload | P1 | Creating ticket | Files attached (max 5, 10MB total) |
| B12-006 | Ticket auto-assignment to agent | P0 | Ticket created | Assigned to available agent based on category |
| B12-007 | Ticket status tracking | P0 | Customer with ticket | Status displayed: Open, In Progress, Resolved, Closed |
| B12-008 | Ticket response by agent | P0 | Agent viewing ticket | Response added, customer notified |
| B12-009 | Ticket response by customer | P0 | Customer with ticket | Response added, agent notified |
| B12-010 | Ticket escalation | P0 | Ticket not resolved in SLA | Auto-escalated to supervisor |
| B12-011 | Ticket resolution | P0 | Agent resolving ticket | Status: Resolved, satisfaction survey sent |
| B12-012 | Ticket closure | P0 | Customer confirming resolution | Status: Closed |
| B12-013 | Ticket reopening | P1 | Customer reopening resolved ticket | Status: Reopened, re-assigned |
| B12-014 | Ticket SLA tracking | P0 | Ticket created | SLA timer started, breached tickets flagged |
| B12-015 | Ticket satisfaction survey | P0 | Ticket resolved | 1-5 star survey sent to customer |
| B12-016 | Ticket internal notes (agent-only) | P1 | Agent viewing ticket | Private notes visible only to agents |
| B12-017 | Ticket merge (duplicate) | P1 | Agent identifying duplicate | Tickets merged, original preserved |
| B12-018 | Ticket export (CSV) | P1 | Admin requesting export | Ticket data exported as CSV |

## 12.2 Support Analytics (8 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B12-019 | Ticket volume dashboard | P0 | Admin viewing | Tickets by day/week/month displayed |
| B12-020 | Average resolution time | P0 | Admin viewing | Mean time to resolution calculated |
| B12-021 | Agent performance metrics | P1 | Admin viewing | Tickets resolved, avg time, satisfaction per agent |
| B12-022 | Category distribution report | P1 | Admin viewing | Tickets broken down by category |
| B12-023 | SLA compliance rate | P0 | Admin viewing | % tickets resolved within SLA |
| B12-024 | Customer satisfaction score | P0 | Admin viewing | Average satisfaction rating |
| B12-025 | Escalation rate report | P1 | Admin viewing | % tickets escalated |
| B12-026 | Ticket trend analysis | P1 | Admin viewing | Volume trends over time |

## 12.3 Analytics Dashboard (16 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B12-027 | Dashboard overview | P0 | Admin logging in | Key metrics: revenue, orders, users, conversion |
| B12-028 | Revenue dashboard | P0 | Admin viewing | Revenue by day/week/month, trend chart |
| B12-029 | Orders dashboard | P0 | Admin viewing | Orders by status, conversion funnel |
| B12-030 | Users dashboard | P0 | Admin viewing | New users, active users, retention |
| B12-031 | Products dashboard | P1 | Admin viewing | Top products, low stock, new listings |
| B12-032 | Vendors dashboard | P1 | Admin viewing | Vendor performance, top vendors |
| B12-033 | Real-time active users | P1 | Admin viewing | Current active sessions displayed |
| B12-034 | Date range filtering | P0 | Admin viewing dashboard | Custom date range selectable |
| B12-035 | Dashboard widget customization | P2 | Admin configuring | Widgets can be added/removed/reordered |
| B12-036 | Dashboard export (PDF) | P1 | Admin exporting | Dashboard snapshot exported as PDF |
| B12-037 | Dashboard export (Excel) | P1 | Admin exporting | Data exported as Excel spreadsheet |
| B12-038 | Dashboard refresh rate (30s) | P1 | Admin viewing | Data refreshes every 30 seconds |
| B12-039 | Dashboard mobile view | P1 | Admin on mobile | Dashboard responsive on mobile |
| B12-040 | Dashboard access control | P0 | User accessing dashboard | Only authorized roles can access |
| B12-041 | Dashboard data accuracy | P0 | Admin comparing data | Dashboard matches raw data |
| B12-042 | Dashboard performance (< 3s load) | P0 | Admin loading dashboard | Loads within 3 seconds |

## 12.4 Reports (14 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B12-043 | Sales report generation | P0 | Admin requesting | Sales data by period, category, vendor |
| B12-044 | Financial report generation | P0 | Finance requesting | Revenue, expenses, profit report |
| B12-045 | Inventory report generation | P0 | Admin requesting | Stock levels, movement, valuation |
| B12-046 | Customer report generation | P1 | Admin requesting | Customer acquisition, demographics, behavior |
| B12-047 | Vendor performance report | P0 | Admin requesting | Vendor scores, sales, ratings |
| B12-048 | Delivery performance report | P1 | Admin requesting | Delivery times, success rates by provider |
| B12-049 | Marketing campaign report | P1 | Marketing requesting | Campaign ROI, conversion tracking |
| B12-050 | Report scheduling (daily/weekly/monthly) | P1 | Admin configuring | Reports auto-generated and sent |
| B12-051 | Report email delivery | P1 | Report scheduled | Report sent to configured email |
| B12-052 | Report format (PDF, Excel, CSV) | P1 | Admin requesting | Format selectable before generation |
| B12-053 | Report data accuracy | P0 | Admin verifying | Report data matches raw data |
| B12-054 | Report generation speed (< 30s) | P0 | Admin requesting | Report generated within 30 seconds |
| B12-055 | Report history | P1 | Admin viewing | Previous reports accessible |
| B12-056 | Custom report builder | P2 | Admin creating custom | Custom fields and filters configurable |

## 12.5 Service Management (10 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B12-057 | Service category management | P0 | Admin creating category | Service category created with name, description |
| B12-058 | Service listing creation | P0 | Vendor creating service | Service listed with pricing, availability |
| B12-059 | Service booking by customer | P0 | Customer selecting service | Booking created with date/time |
| B12-060 | Service booking confirmation | P0 | Booking created | Confirmation sent to customer and vendor |
| B12-061 | Service booking cancellation | P0 | Customer cancelling | Booking cancelled, refund initiated |
| B12-062 | Service completion marking | P0 | Vendor completing service | Service marked as completed |
| B12-063 | Service review submission | P1 | Service completed | Review submitted for service |
| B12-064 | Service availability calendar | P1 | Vendor managing services | Calendar shows available slots |
| B12-065 | Service pricing configuration | P0 | Vendor creating service | Pricing rules saved |
| B12-066 | Service search and filtering | P1 | Customer searching | Services filtered by category, location |

## 12.6 Admin Panel (14 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B12-067 | Admin dashboard login | P0 | Admin credentials | Dashboard accessible |
| B12-068 | Admin role management | P0 | Super admin | Roles created, permissions assigned |
| B12-069 | Admin user management | P0 | Admin | Users listed, can edit/deactivate |
| B12-070 | Admin vendor management | P0 | Admin | Vendors listed, KYC reviewed |
| B12-071 | Admin product management | P0 | Admin | Products listed, can approve/reject |
| B12-072 | Admin order management | P0 | Admin | Orders listed, can intervene |
| B12-073 | Admin financial overview | P0 | Admin | Revenue, payouts, commission overview |
| B12-074 | Admin system configuration | P0 | Super admin | System settings configurable |
| B12-075 | Admin audit log viewer | P0 | Admin | All audit logs searchable |
| B12-076 | Admin notification center | P1 | Admin | Platform notifications managed |
| B12-077 | Admin backup management | P1 | Super admin | Backups initiated, restored |
| B12-078 | Admin feature flag management | P1 | Super admin | Feature flags toggled |
| B12-079 | Admin activity log | P0 | Admin performing actions | All admin actions logged |


---

# Block 13 - Coupons & Discounts (79 Test Points)

## 13.1 Coupon Creation (16 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B13-001 | Platform coupon creation by admin | P0 | Admin creating coupon | Coupon created with code, discount, limits |
| B13-002 | Merchant coupon creation by vendor | P0 | Vendor with active store | Coupon created, pending admin approval |
| B13-003 | Coupon code format validation (alphanumeric, 6-20 chars) | P0 | Creating coupon | Format validated, invalid codes rejected |
| B13-004 | Coupon code uniqueness enforcement | P0 | Creating coupon | Duplicate codes rejected across platform |
| B13-005 | Coupon discount type: percentage | P0 | Creating coupon | Percentage discount (1-99%) saved |
| B13-006 | Coupon discount type: fixed amount | P0 | Creating coupon | Fixed amount discount saved in YER |
| B13-007 | Coupon minimum order value | P1 | Creating coupon | Minimum order value saved |
| B13-008 | Coupon maximum discount cap | P1 | Creating coupon | Maximum discount amount saved |
| B13-009 | Coupon usage limit (total) | P0 | Creating coupon | Total usage limit saved |
| B13-010 | Coupon usage limit (per user) | P1 | Creating coupon | Per-user limit saved |
| B13-011 | Coupon start date | P0 | Creating coupon | Start date saved |
| B13-012 | Coupon end date | P0 | Creating coupon | End date saved |
| B13-013 | Coupon active/inactive toggle | P0 | Coupon created | Can be enabled/disabled |
| B13-014 | Coupon vendor restriction | P1 | Vendor creating coupon | Coupon restricted to vendor's products |
| B13-015 | Coupon category restriction | P1 | Creating coupon | Coupon restricted to specific categories |
| B13-016 | Coupon product restriction | P1 | Creating coupon | Coupon restricted to specific products |

## 13.2 Coupon Validation (14 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B13-017 | Coupon valid (all criteria met) | P0 | Valid coupon code | Discount applied successfully |
| B13-018 | Coupon expired | P0 | Coupon past end date | Error: coupon expired |
| B13-019 | Coupon not yet active | P0 | Coupon before start date | Error: coupon not active |
| B13-020 | Coupon usage limit reached | P0 | Coupon at max uses | Error: coupon usage limit reached |
| B13-021 | Coupon per-user limit reached | P0 | User at max uses | Error: coupon already used maximum times |
| B13-022 | Coupon minimum order not met | P0 | Cart below minimum | Error: minimum order value not met |
| B13-023 | Coupon vendor restriction check | P0 | Coupon restricted, cart has other vendor items | Discount applied only to eligible items |
| B13-024 | Coupon category restriction check | P0 | Coupon restricted, cart has ineligible items | Discount applied only to eligible items |
| B13-025 | Coupon product restriction check | P0 | Coupon restricted, cart has ineligible items | Discount applied only to eligible items |
| B13-026 | Coupon invalid code | P0 | Non-existent coupon code | Error: invalid coupon code |
| B13-027 | Coupon case-insensitive matching | P1 | Coupon code entered in different case | Coupon matched regardless of case |
| B13-028 | Coupon one-time-use enforcement | P0 | Single-use coupon, already used | Error: coupon already used |
| B13-029 | Coupon concurrent usage (race condition) | P0 | Multiple users using last coupon | Only one succeeds, others get limit error |
| B13-030 | Coupon validation timing (before payment) | P0 | Coupon applied at checkout | Validated again before payment processing |

## 13.3 Coupon Usage Tracking (10 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B13-031 | Coupon usage count increment | P0 | Coupon used in order | Usage count incremented atomically |
| B13-032 | Coupon usage per-user tracking | P0 | User using coupon | Usage recorded per user |
| B13-033 | Coupon usage history | P1 | Admin viewing coupon | Complete usage history with orders |
| B13-034 | Coupon revenue impact report | P1 | Finance viewing | Total discounts given per coupon |
| B13-035 | Coupon conversion tracking | P1 | Admin viewing | Orders using each coupon tracked |
| B13-036 | Coupon ROI calculation | P1 | Admin viewing | Revenue generated vs discount given |
| B13-037 | Coupon usage chart | P1 | Admin viewing | Usage over time displayed |
| B13-038 | Coupon top users report | P1 | Admin viewing | Users who used coupon most |
| B13-039 | Coupon export (CSV) | P1 | Admin exporting | Coupon usage data exported |
| B13-040 | Coupon analytics dashboard | P1 | Admin viewing | Coupon performance dashboard |

## 13.4 Discount Types (12 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B13-041 | Percentage discount calculation | P0 | 10% coupon on 1000 YER item | 100 YER discount applied |
| B13-042 | Fixed amount discount calculation | P0 | 500 YER coupon on 1000 YER item | 500 YER discount applied |
| B13-043 | Discount cannot exceed item price | P0 | 2000 YER coupon on 1000 YER item | Discount capped at 1000 YER |
| B13-044 | Discount on multi-item order | P0 | Coupon on 3 items | Discount applied proportionally or to eligible items |
| B13-045 | Free shipping coupon | P0 | Free shipping coupon applied | Delivery fee waived |
| B13-046 | BOGO coupon (Buy One Get One) | P0 | BOGO coupon applied | Second item free |
| B13-047 | Bundle discount coupon | P1 | Bundle coupon applied | Bundle discount applied |
| B13-048 | Tiered discount (spend X get Y% off) | P1 | Tiered coupon configured | Discount applied based on total |
| B13-049 | Discount rounding (whole YER) | P0 | Discount calculation | Rounded to nearest whole YER |
| B13-050 | Discount decimal precision (2 places) | P0 | Discount calculation | Maximum 2 decimal places |
| B13-051 | Discount display on product page | P1 | Product with active discount | Original and discounted price shown |
| B13-052 | Discount in cart summary | P0 | Cart with coupon | Discount shown in order breakdown |

## 13.5 Coupon Stacking & Rules (12 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B13-053 | One coupon per order (default) | P0 | Two coupons applied | Second coupon rejected |
| B13-054 | Stacking allowed (admin config) | P0 | Admin enabling stacking | Multiple coupons applied per order |
| B13-055 | Stack priority (highest discount first) | P0 | Multiple coupons stacked | Highest discount applied first |
| B13-056 | Stack total discount cap (50% max) | P0 | Stacked coupons > 50% discount | Discount capped at 50% |
| B13-057 | Platform + vendor coupon stacking | P0 | Platform and vendor coupons | Both applied if stacking enabled |
| B13-058 | Coupon + loyalty points (allowed) | P0 | Coupon and points redemption | Both applied |
| B13-059 | Coupon + flash sale (best price) | P0 | Coupon and flash sale | Best price for customer applied |
| B13-060 | Stacking order optimization | P1 | Multiple stacked coupons | Optimized for maximum customer savings |
| B13-061 | Stacking audit trail | P0 | Coupons stacked | All applied coupons logged |
| B13-062 | Stacking validation (no negative total) | P0 | Stacking coupons | Total cannot go below zero |
| B13-063 | Stacking limit enforcement (max 3) | P1 | Stacking enabled | Maximum 3 coupons per order |
| B13-064 | Stacking display in cart | P1 | Multiple coupons applied | All applied coupons shown in breakdown |

## 13.6 Admin Coupon Management (8 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B13-065 | Admin coupon list view | P0 | Admin managing coupons | All coupons listed with status and stats |
| B13-066 | Admin coupon search/filter | P0 | Admin managing coupons | Search by code, vendor, status |
| B13-067 | Admin coupon edit | P0 | Admin editing coupon | Coupon details updated |
| B13-068 | Admin coupon delete (soft) | P0 | Admin deleting coupon | Coupon deactivated, not deleted |
| B13-069 | Admin coupon override (force apply) | P0 | Admin overriding coupon | Coupon forced regardless of rules |
| B13-070 | Admin coupon restriction management | P0 | Admin managing restrictions | Restrictions configured and enforced |
| B13-071 | Admin merchant coupon approval queue | P0 | Merchant coupons pending | Queue displayed for review |
| B13-072 | Admin bulk coupon operations | P1 | Admin managing multiple coupons | Bulk activate/deactivate/delete |

## 13.7 Coupon Communication (7 Test Points)

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| B13-073 | Coupon notification to eligible customers | P1 | New coupon created | Targeted customers notified |
| B13-074 | Coupon expiry reminder | P1 | Coupon expiring in 3 days | Reminder sent to eligible users |
| B13-075 | Coupon usage confirmation | P0 | Coupon used in order | Customer sees discount applied |
| B13-076 | Coupon failure message | P0 | Coupon validation fails | Clear error message explaining why |
| B13-077 | Coupon terms display | P1 | Coupon applied | Terms and conditions visible |
| B13-078 | Coupon share functionality | P1 | Customer with active coupon | Coupon shareable via link |
| B13-079 | Coupon page (active promotions) | P1 | Customer viewing promotions | List of active coupons displayed |


---

# Cross-Block Integration (15 Test Points)

## End-to-End Flows

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| CB-001 | Full order lifecycle: browse → cart → checkout → payment → delivery → review | P0 | Customer, vendor, courier all active | Complete flow from discovery to post-delivery review |
| CB-002 | Vendor onboarding: register → KYC → store → product → first order | P0 | New vendor, platform active | Full vendor journey from registration to first sale |
| CB-003 | Wallet top-up → order payment → delivery → escrow release → vendor payout | P0 | Customer with wallet, vendor with store | Complete financial flow from funding to vendor payout |
| CB-004 | Return request → approval → pickup → inspection → refund | P0 | Delivered order, customer requesting return | Complete return lifecycle with refund |
| CB-005 | Coupon creation → admin approval → customer redemption → discount application | P0 | Vendor creating coupon, admin approving | Full coupon lifecycle |

## Cross-Module Data Consistency

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| CB-006 | Order creation consistency: cart → stock reservation → payment → order record | P0 | Customer placing order | All modules updated atomically |
| CB-007 | Multi-vendor order: master/sub-order sync across inventory, payment, shipping | P0 | Multi-vendor order placed | All vendor systems synchronized |
| CB-008 | Cancellation cascade: order → stock release → payment refund → notification | P0 | Order cancelled | All modules notified and updated |
| CB-009 | Delivery confirmation cascade: delivery → payment release → notification → review prompt | P0 | Delivery confirmed | All downstream actions triggered |

## Notification Cross-Block Verification

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| CB-010 | Order state change triggers correct notifications across all channels | P0 | Order state transition | SMS, email, push all sent correctly |
| CB-011 | Vendor receives notifications for all relevant events | P0 | Events affecting vendor | Vendor notified of orders, reviews, payouts |

## Security Cross-Block Verification

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| CB-012 | Role-based access enforced across all modules | P0 | User with specific role | Access controls consistent everywhere |
| CB-013 | Audit trail completeness across all financial operations | P0 | Any financial transaction | Audit logs present in all relevant modules |

## Performance Cross-Block Verification

| Test ID | Description | Priority | Precondition | Expected Result |
|---------|-------------|----------|--------------|-----------------|
| CB-014 | Concurrent operations (100 simultaneous orders) | P0 | 100 users ordering simultaneously | All orders processed correctly, no data corruption |
| CB-015 | System recovery after partial failure | P0 | Payment succeeds, notification fails | System recovers, notification retried, data consistent |


---

# Summary Statistics

## Total Test Points by Block

| Block | Name | Test Points | P0 | P1 | P2 |
|-------|------|-------------|-----|-----|-----|
| 01 | System Core | 27 | 17 | 9 | 1 |
| 02 | Marketplace & Vendors | 70 | 45 | 24 | 1 |
| 03 | Product Catalog | 74 | 47 | 26 | 1 |
| 04 | Orders & Checkout | 139 | 89 | 49 | 1 |
| 05 | Payments & Wallet | 72 | 46 | 25 | 1 |
| 06 | Finance & Accounting | 70 | 45 | 24 | 1 |
| 07 | Shipping & Delivery | 72 | 46 | 25 | 1 |
| 08 | Inventory Management | 72 | 46 | 25 | 1 |
| 09 | Customer Storefront | 74 | 47 | 26 | 1 |
| 10 | Reviews & Loyalty | 74 | 47 | 26 | 1 |
| 11 | Content & Notifications | 72 | 46 | 25 | 1 |
| 12 | Support & Analytics | 79 | 51 | 27 | 1 |
| 13 | Coupons & Discounts | 79 | 51 | 27 | 1 |
| - | Cross-Block Integration | 15 | 4 | 11 | 0 |
| **Total** | | **974** | **627** | **334** | **13** |

## Priority Distribution

| Priority | Count | Percentage |
|----------|-------|------------|
| P0 (Critical) | 627 | 64.4% |
| P1 (High) | 334 | 34.3% |
| P2 (Medium) | 13 | 1.3% |
| **Total** | **974** | **100%** |

## Coverage by Category

| Category | Test Points | Percentage |
|----------|-------------|------------|
| Authentication & Security | 27 | 2.8% |
| Business Logic (Orders, Payments, Inventory) | 453 | 46.5% |
| User Experience (Storefront, Search, Reviews) | 220 | 22.6% |
| Financial Operations | 142 | 14.6% |
| Content & Notifications | 72 | 7.4% |
| Cross-Block Integration | 15 | 1.5% |
| Support & Operations | 45 | 4.6% |

## Testing Execution Recommendations

| Phase | Focus | Duration | Test Points |
|-------|-------|----------|-------------|
| Phase 1 | P0 Critical Path | 2 weeks | 627 |
| Phase 2 | P1 Feature Coverage | 2 weeks | 334 |
| Phase 3 | P2 Edge Cases | 1 week | 13 |
| Phase 4 | Cross-Block Integration | 1 week | 15 |
| Phase 5 | Regression & Performance | 1 week | All |

---

*Document Version: 1.0 | Last Updated: 2026-09-13 | Classification: Confidential*  
*Total Test Points: 974 | Blocks: 13 + Cross-Block Integration*
