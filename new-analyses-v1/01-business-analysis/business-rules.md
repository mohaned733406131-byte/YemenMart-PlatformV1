# Business Rules Catalog — YemenMart

## Overview

This document catalogs all business rules governing the YemenMart marketplace. Rules are organized by domain and include rule IDs, descriptions, conditions, and enforcement mechanisms.

---

## 1. Payment Rules

### PAY-001: Wallet-Only Payments
- **Description**: All payments on YemenMart must be made through integrated mobile wallets
- **Condition**: Payment method selected at checkout
- **Enforcement**: System rejects any non-wallet payment attempts
- **Exception**: None — no COD, credit cards, or bank transfers accepted
- **Rationale**: Low credit card penetration in Yemen; mobile wallets are the dominant digital payment method

### PAY-002: No Cash on Delivery (COD)
- **Description**: YemenMart does not support cash on delivery as a payment method
- **Condition**: Checkout flow
- **Enforcement**: COD option not presented; wallet payment required
- **Exception**: None currently; future consideration for rural areas
- **Rationale**: Reduces cash handling risk, enables escrow, simplifies settlement

### PAY-003: Escrow 7-Day Hold
- **Description**: Customer funds are held in escrow for 7 days after delivery confirmation
- **Condition**: Order status = Delivered
- **Enforcement**: Automatic escrow timer; funds released after 7 days if no dispute
- **Exception**: Instant release if vendor has "Trusted Seller" badge (>95% positive rating, >100 orders)
- **Rationale**: Protects customers against defective products; provides dispute window

### PAY-004: Multi-Currency Support
- **Description**: The platform supports YER (Yemeni Rial), SAR (Saudi Riyal), and USD (US Dollar)
- **Condition**: Wallet currency selection
- **Enforcement**: Real-time exchange rates updated daily; conversion at time of transaction
- **Exception**: Vendor receives settlement in their configured currency; platform handles conversion
- **Rationale**: Yemeni market uses multiple currencies; cross-border commerce with Saudi Arabia

### PAY-005: Wallet Minimum Balance
- **Description**: Customers must have sufficient wallet balance to cover the full order amount including delivery fees
- **Condition**: Checkout submission
- **Enforcement**: Balance check before order creation; insufficient balance blocks order
- **Exception**: None — partial payments not supported
- **Rationale**: Prevents failed transactions and delivery issues

### PAY-006: Refund to Wallet
- **Description**: All refunds are credited back to the customer's original wallet
- **Condition**: Approved refund/dispute resolution
- **Enforcement**: Automatic wallet credit upon refund approval
- **Exception**: None — no bank transfer refunds
- **Rationale**: Maintains wallet ecosystem; simplifies refund processing

### PAY-007: Vendor Settlement Cycle
- **Description**: Vendor earnings are settled every 7 days after escrow release
- **Condition**: Escrow period expired + no active dispute
- **Enforcement**: Automated settlement batch processing
- **Exception**: Weekly manual settlements available for vendors with "Premium" subscription
- **Rationale**: Predictable cash flow for vendors; reduces administrative overhead

### PAY-008: Platform Commission Deduction
- **Description**: Platform commission is deducted automatically from each transaction before vendor settlement
- **Condition**: Order completion
- **Enforcement**: Commission calculated and deducted during settlement
- **Exception**: Promotional periods may offer reduced or zero commission
- **Rationale**: Primary revenue stream for the platform

### PAY-009: Transaction Minimum Amount
- **Description**: Minimum transaction amount is 100 YER (or equivalent in other currencies)
- **Condition**: Checkout submission
- **Enforcement**: Order blocked if total below minimum
- **Exception**: Platform-initiated transactions (refunds, adjustments) have no minimum
- **Rationale**: Ensures transaction costs are economically viable

### PAY-010: Transaction Maximum Amount
- **Description**: Maximum single transaction amount is 5,000,000 YER (or equivalent)
- **Condition**: Checkout submission
- **Enforcement**: Order blocked if total exceeds maximum
- **Exception**: Vendor and admin can request limit increase with KYC upgrade
- **Rationale**: Fraud prevention; regulatory compliance

### PAY-011: Wallet Top-Up Limits
- **Description**: Daily wallet top-up limit is 500,000 YER for standard users; 2,000,000 YER for verified users
- **Condition**: Wallet top-up request
- **Enforcement**: Daily cumulative tracking; limit enforced per calendar day
- **Exception**: None — applies to all users regardless of tier
- **Rationale**: Anti-money laundering compliance

### PAY-012: Failed Payment Retry
- **Description**: Failed payment attempts are limited to 3 per transaction before requiring user re-authentication
- **Condition**: Payment processing failure
- **Enforcement**: Counter tracks failures; locks after 3 attempts
- **Exception**: System errors (timeout, network) do not count toward limit
- **Rationale**: Prevents brute-force payment attempts

---

## 2. Authentication Rules

### AUTH-001: Primary Login — Phone + Password
- **Description**: All users authenticate using phone number + password as the primary login method
- **Condition**: Login attempt
- **Enforcement**: System verifies phone number exists and password hash matches
- **Exception**: None — all users must have a password set during registration
- **Rationale**: Password-based login is faster and more reliable than OTP; not dependent on SMS delivery

### AUTH-002: Secondary Login — Phone + OTP (Forgot Password Only)
- **Description**: Phone + OTP is available as a secondary method for password recovery only
- **Condition**: User clicks "Forgot Password" link
- **Enforcement**: System sends OTP via SMS/WhatsApp to registered phone number
- **Exception**: None — OTP is not used for regular login
- **Rationale**: OTP provides a secure backup when user forgets password

### AUTH-003: Password Requirements
- **Description**: Minimum 8 characters with at least one uppercase, one lowercase, and one number
- **Condition**: Registration or password change
- **Enforcement**: Password strength validation before hashing
- **Exception**: None — applies to all users
- **Rationale**: Ensures reasonable password security

### AUTH-004: OTP 3-Attempt Limit
- **Description**: Users have a maximum of 3 OTP entry attempts for password reset
- **Condition**: Incorrect OTP entered
- **Enforcement**: Counter increments on each failure; lockout at 3
- **Exception**: None — applies to forgot password flow only
- **Rationale**: Prevents brute-force OTP guessing

### AUTH-005: 15-Minute Lockout
- **Description**: After 5 failed login attempts or 3 failed OTP attempts, the account is locked for 15 minutes
- **Condition**: Failed attempts threshold reached
- **Enforcement**: Account locked; timer displayed to user
- **Exception**: Admin can manually unlock for verified users
- **Rationale**: Security measure; discourages unauthorized access attempts

### AUTH-006: OTP Expiry
- **Description**: OTPs expire after 5 minutes
- **Condition**: OTP generation timestamp
- **Enforcement**: System rejects expired OTPs; prompts for new code
- **Exception**: None — security standard
- **Rationale**: Limits window for OTP interception

### AUTH-007: Session Management
- **Description**: Active sessions are limited to 5 devices per user
- **Condition**: New device login
- **Enforcement**: Oldest session invalidated when 6th device attempts login
- **Exception**: None — applies to all users
- **Rationale**: Prevents session abuse; enhances account security

### AUTH-008: Sensitive Action Re-Authentication
- **Description**: Sensitive actions (password change, wallet withdrawal, profile update) require re-authentication
- **Condition**: User initiates sensitive action
- **Enforcement**: Password prompt before action execution
- **Exception**: Actions performed within 5 minutes of initial login do not require re-auth
- **Rationale**: Additional security layer for high-risk operations

---

## 3. Order Rules

### ORD-001: 17-State Order Lifecycle
- **Description**: Orders progress through 17 defined states from creation to completion
- **Condition**: Order state transitions
- **Enforcement**: State machine enforced; invalid transitions rejected
- **States**: Pending → Confirmed → Processing → Ready for Pickup → Dispatched → In Transit → Out for Delivery → Delivered → Completed | Cancelled → Refunding → Refunded | Disputed → Under Review → Resolved | Returned → Return Accepted → Return Completed
- **Rationale**: Comprehensive lifecycle tracking for all order scenarios

### ORD-002: Master/Sub-Order Architecture
- **Description**: A customer order (master order) contains one or more sub-orders, one per vendor
- **Condition**: Cart contains items from multiple vendors
- **Enforcement**: System automatically splits cart into vendor-specific sub-orders
- **Exception**: Single-vendor orders have master = sub-order
- **Rationale**: Enables independent fulfillment per vendor; accurate commission tracking

### ORD-003: 15-Minute Stock Hold
- **Description**: Items in a pending order are held for 15 minutes, reserving stock
- **Condition**: Order created (Pending state)
- **Enforcement**: Stock decremented on order creation; restored on timeout or cancellation
- **Exception**: "Buy Now" orders extend hold to 30 minutes
- **Rationale**: Prevents overselling during checkout; gives customers time to complete payment

### ORD-004: Minimum Order Value
- **Description**: Minimum order value per vendor is 100 YER (or equivalent)
- **Condition**: Cart total per vendor
- **Enforcement**: Vendor minimum enforced at checkout; blocked if below threshold
- **Exception**: Platform-initiated orders (admin, support) have no minimum
- **Rationale**: Ensures vendor economic viability per order

### ORD-005: Maximum Items Per Order
- **Description**: Maximum 50 items per master order (across all vendors)
- **Condition**: Cart addition
- **Enforcement**: Cart limit enforced; error displayed when exceeded
- **Exception**: None — applies to all customers
- **Rationale**: Prevents abuse; ensures manageable fulfillment

### ORD-006: Order Cancellation Window
- **Description**: Customers can cancel orders within 15 minutes of creation, or before vendor confirmation
- **Condition**: Order in Pending state
- **Enforcement**: Cancel button available; automatic refund initiated
- **Exception**: Orders in Processing or later states cannot be cancelled by customer
- **Rationale**: Balances customer flexibility with vendor operational needs

### ORD-007: Vendor Confirmation SLA
- **Description**: Vendors must confirm or reject orders within 24 hours
- **Condition**: Order in Pending state
- **Enforcement**: Auto-reject and refund if not confirmed within 24 hours
- **Exception**: "Trusted Sellers" have 48-hour window
- **Rationale**: Ensures customers receive timely responses; prevents indefinite holds

### ORD-008: Delivery Attempt Limit
- **Description**: Maximum 3 delivery attempts per order
- **Condition**: Delivery attempt failure
- **Enforcement**: Counter tracks attempts; after 3 failures, order returned to vendor
- **Exception**: Customer can request additional attempt via support ticket
- **Rationale**: Balances delivery success with logistics efficiency

### ORD-009: Order Auto-Completion
- **Description**: Orders auto-complete 7 days after delivery confirmation if no dispute is raised
- **Condition**: Delivery confirmed + 7 days elapsed + no dispute
- **Enforcement**: System auto-completes order; escrow released to vendor
- **Exception**: Open dispute prevents auto-completion
- **Rationale**: Ensures timely vendor settlement; prevents indefinite holds

---

## 4. Delivery Rules

### DEL-001: OTP Verification >50,000 YER
- **Description**: Orders with value exceeding 50,000 YER require OTP verification at delivery
- **Condition**: Order total > 50,000 YER + delivery attempted
- **Enforcement**: Delivery person requests OTP from customer; delivery completes only with valid OTP
- **Exception**: None — security requirement
- **Rationale**: Prevents unauthorized delivery of high-value items

### DEL-002: Signature Required >100,000 YER
- **Description**: Orders with value exceeding 100,000 YER require signature capture at delivery
- **Condition**: Order total > 100,000 YER + delivery attempted
- **Enforcement**: Delivery app captures customer signature (digital or photo); delivery incomplete without signature
- **Exception**: None — security requirement
- **Rationale**: Legal proof of delivery for high-value items

### DEL-003: Delivery Zone Restrictions
- **Description**: Deliveries are limited to zones where delivery providers operate
- **Condition**: Delivery address zone check
- **Enforcement**: System checks delivery availability before order confirmation
- **Exception**: Admin can override for special arrangements
- **Rationale**: Ensures realistic delivery expectations

### DEL-004: Delivery Fee Calculation
- **Description**: Delivery fees are calculated based on vendor zone, customer zone, weight, and distance
- **Condition**: Cart checkout
- **Enforcement**: Fee calculated automatically; displayed before payment
- **Exception**: Promotional free delivery campaigns
- **Rationale**: Transparent pricing; cost recovery for delivery providers

### DEL-005: Real-Time Tracking
- **Description**: All deliveries must provide real-time GPS tracking when in transit
- **Condition**: Delivery status = In Transit
- **Enforcement**: Delivery app shares GPS coordinates; customer sees live map
- **Exception**: Tracking gaps in low-connectivity areas are acceptable
- **Rationale**: Customer transparency; delivery accountability

### DEL-006: Return Handling
- **Description**: Returned items must be picked up within 48 hours of return approval
- **Condition**: Return approved
- **Enforcement**: Delivery provider assigned; 48-hour pickup window enforced
- **Exception**: Extended window for rural areas (72 hours)
- **Rationale**: Timely return processing; vendor inventory recovery

### DEL-007: Delivery Photo Proof
- **Description**: Delivery persons must capture a photo of the delivered package at the delivery location
- **Condition**: Delivery completion
- **Enforcement**: Photo required in delivery app; delivery marked incomplete without photo
- **Exception**: None — dispute evidence requirement
- **Rationale**: Proof of delivery for dispute resolution

### DEL-008: Fragile Item Handling
- **Description**: Orders containing items marked as fragile require special handling and signature
- **Condition**: Product tagged as "Fragile" in catalog
- **Enforcement**: Delivery app prompts for careful handling; signature always required
- **Exception**: None
- **Rationale**: Reduces damage claims; ensures proper handling

---

## 5. Coupon Rules

### CPN-001: Discount Range 1–90%
- **Description**: Coupon discounts must be between 1% and 90% of the product price
- **Condition**: Coupon creation
- **Enforcement**: System validates discount range; rejects outside bounds
- **Exception**: Platform-wide flash sales may exceed 90% with admin approval
- **Rationale**: Prevents unrealistic discounts; maintains marketplace trust

### CPN-002: No Coupon Stacking
- **Description**: Only one coupon can be applied per order
- **Condition**: Multiple coupons attempted at checkout
- **Enforcement**: System replaces existing coupon with newly entered one
- **Exception**: None — strict no-stacking policy
- **Rationale**: Simplifies discount calculation; prevents excessive discounting

### CPN-003: Merchant-Created Coupons
- **Description**: Vendors can create coupons for their own products only
- **Condition**: Vendor coupon creation
- **Enforcement**: Coupon restricted to vendor's products; cannot be applied to other vendors' items
- **Exception**: None
- **Rationale**: Vendor-controlled promotions; reduces platform liability

### CPN-004: Admin-Created Coupons
- **Description**: Platform administrators can create coupons applicable across all vendors
- **Condition**: Admin coupon creation
- **Enforcement**: Admin coupons can target specific vendors, categories, or all products
- **Exception**: None
- **Rationale**: Platform-wide promotional campaigns

### CPN-005: Coupon Expiry
- **Description**: All coupons have a mandatory expiry date (maximum 90 days from creation)
- **Condition**: Coupon creation
- **Enforcement**: System enforces expiry; coupons auto-deactivate after expiry date
- **Exception**: "Evergreen" coupons created by admin with quarterly renewal
- **Rationale**: Prevents coupon abuse; ensures promotional freshness

### CPN-006: Minimum Order for Coupon
- **Description**: Coupons can require a minimum order value to be applicable
- **Condition**: Coupon configuration
- **Enforcement**: Minimum order value checked at checkout; coupon rejected if below threshold
- **Exception**: None
- **Rationale**: Encourages higher order values; protects vendor margins

---

## 6. Loyalty Rules

### LOY-001: All Wallet Payments Qualify
- **Description**: All wallet payments (not just completed orders) earn loyalty points
- **Condition**: Successful wallet transaction
- **Enforcement**: Points credited proportionally to payment amount
- **Exception**: Refunded transactions have points deducted
- **Rationale**: Encourages wallet usage; rewards all spending behavior

### LOY-002: Tier System
- **Description**: Customers advance through tiers (Bronze → Silver → Gold → Platinum) based on cumulative spending
- **Condition**: Cumulative spending thresholds met
- **Enforcement**: Automatic tier upgrade; benefits applied immediately
- **Tier Thresholds**: Bronze (0–50,000 YER), Silver (50,001–200,000 YER), Gold (200,001–500,0000 YER), Platinum (500,001+ YER)
- **Exception**: Tier review quarterly; downgrade if spending drops below threshold
- **Rationale**: Gamification; customer retention

### LOY-003: Referral Program
- **Description**: Referring customers earn bonus points when referred users make their first purchase
- **Condition**: New user registers via referral link and completes first order
- **Enforcement**: Referrer receives fixed bonus; referred user receives welcome bonus
- **Exception**: Referral bonus limited to 5 successful referrals per month
- **Rationale**: Organic growth; customer acquisition cost reduction

### LOY-004: Points Redemption
- **Description**: Loyalty points can be redeemed for discounts at checkout (100 points = 1 YER)
- **Condition**: Customer has redeemable points
- **Enforcement**: Points-to-discount conversion at checkout; maximum 30% of order value
- **Exception**: None — points cannot be withdrawn as cash
- **Rationale**: Encourages repeat purchases; tangible reward value

### LOY-005: Points Expiry
- **Description**: Loyalty points expire after 12 months of account inactivity
- **Condition**: No qualifying transaction for 12 months
- **Enforcement**: System deducts expired points; notification sent 30 days before expiry
- **Exception**: Platinum members have no points expiry
- **Rationale**: Encourages regular engagement; manages loyalty liability

---

## 7. Product Rules

### PRD-001: Arabic Product Name Required
- **Description**: Every product must have an Arabic name (minimum 3 characters)
- **Condition**: Product creation or update
- **Enforcement**: System rejects products without valid Arabic name
- **Exception**: None — Arabic name is mandatory for all listings
- **Rationale**: Arabic-first platform; ensures searchability for Arabic-speaking users

### PRD-002: Auto-Scoring System
- **Description**: Products receive an automatic quality score based on completeness, images, reviews, and sales
- **Condition**: Product listing
- **Enforcement**: Score calculated continuously; influences search ranking
- **Scoring Factors**: Image count (20%), description quality (15%), review rating (25%), sales velocity (20%), completeness (20%)
- **Exception**: None — all products scored equally
- **Rationale**: Incentivizes quality listings; improves customer experience

### PRD-003: Return Policy
- **Description**: All products have a 7-day return policy from delivery date
- **Condition**: Return request within 7 days of delivery
- **Enforcement**: Return button available for 7 days; auto-hides after
- **Exception**: "No Return" products (clearly marked) and perishable goods
- **Rationale**: Customer protection; builds trust

### PRD-004: Product Images
- **Description**: Products must have at least 1 image; maximum 10 images allowed
- **Condition**: Product creation
- **Enforcement**: Minimum 1 image required; upload limit enforced
- **Exception**: None — images are mandatory for product listing
- **Rationale**: Visual commerce; reduces returns from mismatched expectations

### PRD-005: Category Assignment
- **Description**: Products must be assigned to exactly one primary category and up to 2 secondary categories
- **Condition**: Product creation
- **Enforcement**: Primary category required; secondary categories optional
- **Exception**: None
- **Rationale**: Organized catalog; accurate search and filtering

### PRD-006: Price Validation
- **Description**: Product price must be greater than 0 and not exceed 10,000,000 YER
- **Condition**: Product creation or price update
- **Enforcement**: Price range validated; rejected if outside bounds
- **Exception**: Admin can set prices outside range for special products
- **Rationale**: Prevents erroneous pricing; fraud prevention

### PRD-007: Stock Management
- **Description**: Stock quantity must be maintained for each product variant
- **Condition**: Product creation
- **Enforcement**: Stock count tracked; zero-stock products marked "Out of Stock"
- **Exception**: "Made to Order" products can have unlimited stock
- **Rationale**: Accurate availability; prevents overselling

### PRD-008: Product Variants
- **Description**: Products can have up to 5 variant dimensions (size, color, etc.) with up to 50 combinations
- **Condition**: Product creation
- **Enforcement**: Variant combinations validated; price and stock per variant
- **Exception**: None — simplified variant system
- **Rationale**: Supports product variety without excessive complexity

---

## 8. Vendor Rules

### VND-001: KYC Verification Required
- **Description**: Vendors must complete KYC verification before listing products
- **Condition**: Vendor registration
- **Enforcement**: Store goes live only after KYC approval; pending status otherwise
- **Required Documents**: Commercial registration, ID proof, bank/wallet details
- **Exception**: "Test Store" mode allows 10 product listings during verification
- **Rationale**: Trust and compliance; prevents fraudulent vendors

### VND-002: Store Templates
- **Description**: Vendors can choose from predefined store templates for their storefront
- **Condition**: Store setup
- **Enforcement**: Template selection required; customization within template constraints
- **Exception**: Premium subscribers can request custom templates
- **Rationale**: Professional appearance without design expertise

### VND-003: Staff RBAC (Role-Based Access Control)
- **Description**: Vendors can invite staff with role-based permissions (Viewer, Editor, Manager)
- **Condition**: Staff invitation
- **Enforcement**: Roles enforced; permissions validated on each action
- **Roles**: Viewer (read-only), Editor (products, orders), Manager (all except financials)
- **Exception**: Store owner retains all permissions
- **Rationale**: Operational flexibility; security for sensitive data

### VND-004: Vendor Rating System
- **Description**: Vendors receive ratings based on order fulfillment, customer reviews, and response time
- **Condition**: Ongoing operations
- **Enforcement**: Rating displayed on store page; influences search ranking
- **Rating Factors**: Order fulfillment (40%), customer review (35%), response time (25%)
- **Exception**: New vendors receive provisional "New Seller" badge for first 30 days
- **Rationale**: Quality benchmarking; customer trust

### VND-005: Product Listing Limits
- **Description**: Vendor product listing limits depend on subscription tier
- **Condition**: Product creation
- **Enforcement**: Limit enforced per subscription plan
- **Limits**: Basic (100 products), Standard (500 products), Premium (unlimited)
- **Exception**: None — hard limit per plan
- **Rationale**: Tiered value proposition; revenue optimization

### VND-006: Vendor Payout Threshold
- **Description**: Minimum payout amount is 1,000 YER; amounts below are rolled over
- **Condition**: Settlement cycle
- **Enforcement**: Payout below threshold held until threshold reached
- **Exception**: Vendor can request payout below threshold with 50 YER fee
- **Rationale**: Reduces transaction costs; predictable settlements

---

## 9. Additional Rules

### SYS-001: Exchange Rate Updates
- **Description**: Exchange rates are updated every 24 hours from a trusted source
- **Condition**: Daily scheduler
- **Enforcement**: Rates cached and used for conversions; stale rates rejected
- **Exception**: Admin can manually update rates for emergency situations
- **Rationale**: Accurate multi-currency support

### SYS-002: Notification Preferences
- **Description**: Users can customize notification preferences per channel (SMS, WhatsApp, push, email)
- **Condition**: User settings
- **Enforcement**: Notifications sent only to opted-in channels
- **Exception**: Critical notifications (security, fraud) sent regardless of preferences
- **Rationale**: User control; reduced notification fatigue

### SYS-003: Content Moderation
- **Description**: All product images and descriptions are subject to automated and manual moderation
- **Condition**: Product submission
- **Enforcement**: AI screening for prohibited content; manual review queue for flagged items
- **Exception**: None — all content moderated
- **Rationale**: Platform safety; regulatory compliance

### SYS-004: Data Retention
- **Description**: User data is retained for 5 years after account deletion for compliance purposes
- **Condition**: Account deletion request
- **Enforcement**: Account anonymized; data retained in secure archive
- **Exception**: Transaction records retained for 7 years per financial regulations
- **Rationale**: Regulatory compliance; audit requirements
