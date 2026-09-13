# System Use Cases — YemenMart

## Overview

70 system (automated) use cases organized by category. These are automated processes triggered by the system without direct user interaction.

---

## 1. Stock Management (UC-S-001 to UC-S-006)

### UC-S-001: Stock Hold on Order Creation
- **Trigger**: Order created (Pending state)
- **Main Flow**: System decrements stock for ordered items → Stock held for 15 minutes → If order not confirmed within 15 minutes, stock restored
- **Alternative**: Buy Now order extends hold to 30 minutes

### UC-S-002: Stock Release on Order Cancellation
- **Trigger**: Order cancelled
- **Main Flow**: System restores stock for cancelled items → Stock available for other customers
- **Alternative**: Stock already sold to another customer → No action needed

### UC-S-003: Low Stock Alert
- **Trigger**: Product stock falls below threshold (configurable, default 5 units)
- **Main Flow**: System sends notification to vendor → "Low stock alert for [product name]"
- **Alternative**: Vendor has disabled notifications → Alert logged only

### UC-S-004: Out of Stock Update
- **Trigger**: Product stock reaches zero
- **Main Flow**: System marks product as "Out of Stock" → Hides from search results → Notifies vendor
- **Alternative**: Product has "Notify Me" subscribers → Send restock notifications

### UC-S-005: Restock Notification
- **Trigger**: Out-of-stock product restocked
- **Main Flow**: System notifies all customers who opted for "Notify Me" → Product restored to search results
- **Alternative**: No subscribers → No notifications sent

### UC-S-006: Stock Synchronization
- **Trigger**: Scheduled (every 5 minutes)
- **Main Flow**: System reconciles stock across all orders and vendor updates → Resolves any discrepancies → Logs corrections
- **Alternative**: Discrepancy exceeds threshold → Alert admin

---

## 2. Escrow Management (UC-S-007 to UC-S-012)

### UC-S-007: Escrow Fund Collection
- **Trigger**: Customer payment successful
- **Main Flow**: System moves funds from customer wallet to escrow account → Escrow record created with 7-day timer
- **Alternative**: Payment fails → Order cancelled → Stock restored

### UC-S-008: Escrow Release on Delivery Confirmation
- **Trigger**: Delivery confirmed + 7-day window expired + no dispute
- **Main Flow**: System releases escrow funds to vendor wallet → Deducts platform commission → Settlement record created
- **Alternative**: Dispute opened → Escrow frozen until resolution

### UC-S-009: Escrow Freeze on Dispute
- **Trigger**: Dispute opened by customer or vendor
- **Main Flow**: System freezes escrow funds → No release until dispute resolved → Both parties notified
- **Alternative**: Dispute resolved → Follow resolution decision

### UC-S-010: Escrow Refund on Order Cancellation
- **Trigger**: Order cancelled before delivery
- **Main Flow**: System refunds escrow funds to customer wallet → Escrow record closed
- **Alternative**: Partial cancellation → Partial refund

### UC-S-011: Escrow Refund on Dispute Resolution
- **Trigger**: Dispute resolved in customer's favor
- **Main Flow**: System refunds escrow funds to customer wallet (full or partial based on resolution) → Vendor notified
- **Alternative**: Dispute resolved in vendor's favor → Escrow released to vendor

### UC-S-012: Escrow Expiry Notification
- **Trigger**: Escrow approaching 7-day expiry (6 days elapsed)
- **Main Flow**: System sends reminder to vendor → "Escrow will be released in 24 hours if no dispute"
- **Alternative**: Vendor already notified → No duplicate notification

---

## 3. Notification Management (UC-S-013 to UC-S-020)

### UC-S-013: Order Status Notification
- **Trigger**: Order status change
- **Main Flow**: System sends notification to customer via preferred channel (SMS/WhatsApp/push) → Includes order number and new status
- **Alternative**: Customer opted out of channel → Try alternate channel

### UC-S-014: Vendor Order Notification
- **Trigger**: New order received
- **Main Flow**: System sends notification to vendor → "New order received! Order #[number]"
- **Alternative**: Vendor opted out → Log notification only

### UC-S-015: Delivery Update Notification
- **Trigger**: Delivery status change
- **Main Flow**: System sends notification to customer → "Your order is [status]" → Include tracking link if in transit
- **Alternative**: Notification fails → Retry after 5 minutes

### UC-S-016: Payment Confirmation Notification
- **Trigger**: Payment processed
- **Main Flow**: System sends payment confirmation to customer → "Payment of [amount] received for order #[number]"
- **Alternative**: Notification fails → Log only

### UC-S-017: Settlement Notification
- **Trigger**: Vendor settlement processed
- **Main Flow**: System sends notification to vendor → "Settlement of [amount] processed to your wallet"
- **Alternative**: Notification fails → Log only

### UC-S-018: OTP Delivery Notification
- **Trigger**: OTP requested for authentication or delivery verification
- **Main Flow**: System generates OTP → Sends via SMS and/or WhatsApp → OTP expires after 5 minutes
- **Alternative**: SMS fails → Try WhatsApp → Both fail → "Unable to send OTP"

### UC-S-019: Promotional Notification
- **Trigger**: Platform promotion or sale event
- **Main Flow**: System sends promotional notification to opted-in customers → "New sale! Up to [X]% off"
- **Alternative**: Customer opted out → Not sent

### UC-S-020: Account Security Notification
- **Trigger**: Sensitive account activity (login from new device, password change, large transaction)
- **Main Flow**: System sends security alert via all channels → "New [activity] detected on your account"
- **Alternative**: Cannot reach customer → Log security event

---

## 4. Commission Management (UC-S-021 to UC-S-025)

### UC-S-021: Commission Calculation
- **Trigger**: Order completed (delivery confirmed + 7 days)
- **Main Flow**: System calculates commission based on order value and vendor category rate → Records commission amount
- **Alternative**: Promotional rate active → Apply promotional rate

### UC-S-022: Commission Deduction from Settlement
- **Trigger**: Settlement processed
- **Main Flow**: System deducts commission from vendor settlement → Records deduction → Updates vendor balance
- **Alternative**: Insufficient settlement → Hold commission for next cycle

### UC-S-023: Commission Report Generation
- **Trigger**: Scheduled (daily)
- **Main Flow**: System generates daily commission report → Total commissions earned, by vendor, by category → Stored for admin review
- **Alternative**: Report generation fails → Retry

### UC-S-024: Commission Rate Change
- **Trigger**: Admin updates commission rate
- **Main Flow**: System applies new rate to all future transactions → Old rate honored for in-progress orders
- **Alternative**: Rate effective date in future → Schedule change

### UC-S-025: Commission Dispute Handling
- **Trigger**: Vendor disputes commission charge
- **Main Flow**: System freezes disputed commission → Notifies admin → Awaits manual review
- **Alternative**: Auto-resolution not possible → Manual process

---

## 5. Loyalty Management (UC-S-026 to UC-S-030)

### UC-S-026: Points Accrual
- **Trigger**: Successful wallet payment
- **Main Flow**: System calculates points earned (based on payment amount) → Credits to customer loyalty account → Notification sent
- **Alternative**: Refunded transaction → Points deducted

### UC-S-027: Points Redemption at Checkout
- **Trigger**: Customer applies points at checkout
- **Main Flow**: System validates points balance → Calculates discount (max 30% of order) → Deducts points → Applies discount
- **Alternative**: Insufficient points → "Not enough points" error

### UC-S-028: Tier Upgrade Evaluation
- **Trigger**: Monthly evaluation (1st of each month)
- **Main Flow**: System evaluates cumulative spending per customer → Upgrades tier if threshold met → Notifies customer
- **Alternative**: Tier downgrade if spending dropped → Notify with reason

### UC-S-029: Points Expiry
- **Trigger**: Daily evaluation
- **Main Flow**: System checks for inactive accounts (12 months no qualifying transaction) → Deducts expired points → Sends 30-day warning before expiry
- **Alternative**: Platinum members → No expiry applied

### UC-S-030: Referral Bonus Processing
- **Trigger**: Referred user completes first purchase
- **Main Flow**: System credits referrer bonus → Credits referred user welcome bonus → Both notified
- **Alternative**: Monthly referral limit reached (5) → No bonus credited

---

## 6. Inventory Management (UC-S-031 to UC-S-034)

### UC-S-031: Inventory Sync
- **Trigger**: Scheduled (every 15 minutes)
- **Main Flow**: System synchronizes inventory across all vendor products → Updates stock levels → Resolves conflicts
- **Alternative**: Sync failure → Alert admin → Retry

### UC-S-032: Out of Stock Auto-Deactivate
- **Trigger**: Product stock reaches zero
- **Main Flow**: System deactivates product listing → Removes from search → Notifies vendor
- **Alternative**: Vendor restocks within 24 hours → Reactivate automatically

### UC-S-033: Overstock Alert
- **Trigger**: Product stock exceeds threshold (configurable, default 1000 units)
- **Main Flow**: System sends alert to vendor → "High stock level for [product] - consider promotion"
- **Alternative**: Vendor has disabled alerts → Log only

### UC-S-034: Inventory Reconciliation
- **Trigger**: Scheduled (daily)
- **Main Flow**: System reconciles inventory across orders, returns, and vendor updates → Generates discrepancy report → Alerts admin for major differences
- **Alternative**: No discrepancies → Log successful reconciliation

---

## 7. KYC Management (UC-S-035 to UC-S-038)

### UC-S-035: KYC Document Validation
- **Trigger**: KYC document uploaded
- **Main Flow**: System validates document format, size, and readability → Flags issues → Queues for manual review
- **Alternative**: Document invalid → Reject with reason → Notify vendor

### UC-S-036: KYC Expiry Reminder
- **Trigger**: KYC approaching expiry (30 days before)
- **Main Flow**: System sends reminder to vendor → "Your KYC verification expires in 30 days. Please renew."
- **Alternative**: Vendor already renewed → No reminder

### UC-S-037: KYC Expiry Enforcement
- **Trigger**: KYC expired
- **Main Flow**: System deactivates vendor store → Suspends product listings → Notifies vendor → Awaits re-verification
- **Alternative**: Vendor re-submits within 7 days → Expedited review

### UC-S-038: KYC Approval Notification
- **Trigger**: KYC approved by admin
- **Main Flow**: System activates vendor store → Publishes product listings → Sends welcome notification → Onboarding sequence triggered
- **Alternative**: KYC rejected → Sends rejection notification with reason

---

## 8. Badge Management (UC-S-039 to UC-S-041)

### UC-S-039: Trusted Seller Badge Evaluation
- **Trigger**: Weekly evaluation
- **Main Flow**: System evaluates vendors against criteria (>95% positive rating, >100 orders, <2% cancellation rate) → Awards badge to qualifying vendors
- **Alternative**: Badge criteria not met → Badge removed if previously awarded

### UC-S-040: New Seller Badge
- **Trigger**: Vendor KYC approved
- **Main Flow**: System awards "New Seller" badge → Badge valid for first 30 days → Auto-removed after period
- **Alternative**: Vendor reaches 100 orders before 30 days → Badge replaced with Trusted Seller if criteria met

### UC-S-041: Badge Notification
- **Trigger**: Badge awarded or removed
- **Main Flow**: System notifies vendor → "Congratulations! You've earned the [badge name]" or "Your [badge name] has been removed"
- **Alternative**: Notification fails → Log event

---

## 9. Reporting & Analytics (UC-S-042 to UC-S-046)

### UC-S-042: Daily Sales Report
- **Trigger**: Scheduled (end of each day)
- **Main Flow**: System generates daily sales report → Total orders, revenue, new users, new vendors, top products → Stored and available in admin dashboard
- **Alternative**: Report generation fails → Retry → Alert admin

### UC-S-043: Weekly Performance Report
- **Trigger**: Scheduled (every Monday)
- **Main Flow**: System generates weekly performance report → Week-over-week comparison, trends, alerts → Stored and emailed to admin
- **Alternative**: Report generation fails → Retry

### UC-S-044: Monthly Financial Report
- **Trigger**: Scheduled (1st of each month)
- **Main Flow**: System generates monthly financial report → Revenue, commissions, refunds, settlements, profit/loss → Stored and emailed to admin
- **Alternative**: Report generation fails → Retry

### UC-S-045: Vendor Performance Report
- **Trigger**: Scheduled (monthly)
- **Main Flow**: System evaluates all vendors against performance metrics → Generates report → Flags underperforming vendors
- **Alternative**: No vendors → Skip report

### UC-S-046: Customer Analytics Report
- **Trigger**: Scheduled (monthly)
- **Main Flow**: System generates customer analytics report → Registration trends, retention, LTV, segment analysis → Stored in admin dashboard
- **Alternative**: No customers → Skip report

---

## 10. Backup & Recovery (UC-S-047 to UC-S-049)

### UC-S-047: Database Backup
- **Trigger**: Scheduled (daily at 2 AM)
- **Main Flow**: System performs full database backup → Stores in secure offsite location → Logs backup success/failure
- **Alternative**: Backup fails → Retry immediately → Alert admin if second attempt fails

### UC-S-048: Transaction Log Backup
- **Trigger**: Scheduled (every 6 hours)
- **Main Flow**: System backs up transaction logs → Stores securely → Maintains 30-day retention
- **Alternative**: Backup fails → Retry → Alert admin

### UC-S-049: Disaster Recovery Test
- **Trigger**: Scheduled (monthly)
- **Main Flow**: System performs automated disaster recovery test → Restores backup to test environment → Validates data integrity → Reports results
- **Alternative**: Test fails → Alert admin → Manual intervention required

---

## 11. Exchange Rate Management (UC-S-050 to UC-S-052)

### UC-S-050: Exchange Rate Update
- **Trigger**: Scheduled (every 24 hours)
- **Main Flow**: System fetches latest exchange rates from trusted source → Updates cache → Applies to all currency conversions
- **Alternative**: Source unavailable → Use last known rates → Alert admin

### UC-S-051: Exchange Rate Alert
- **Trigger**: Exchange rate changes by more than 5% in 24 hours
- **Main Flow**: System sends alert to admin → "Exchange rate [currency] changed by [X]% in 24 hours"
- **Alternative**: Rate within normal range → No alert

### UC-S-052: Exchange Rate Logging
- **Trigger**: Each rate update
- **Main Flow**: System logs all exchange rate changes with timestamp, source, old rate, new rate → Maintains audit trail
- **Alternative**: None

---

## 12. Delivery Code Management (UC-S-053 to UC-S-055)

### UC-S-053: Generate Delivery OTP
- **Trigger**: Delivery person requests OTP for high-value order (>50,000 YER)
- **Main Flow**: System generates 6-digit OTP → Sends to customer via SMS/WhatsApp → OTP valid for 10 minutes
- **Alternative**: OTP delivery fails → Retry → Fallback to backup method

### UC-S-054: Validate Delivery OTP
- **Trigger**: Delivery person enters OTP
- **Main Flow**: System validates OTP → If valid, marks OTP as used → Delivery proceeds → If invalid, increments attempt counter
- **Alternative**: 3 invalid attempts → OTP invalidated → New OTP generated

### UC-S-055: Delivery OTP Expiry
- **Trigger**: OTP expires (10 minutes)
- **Main Flow**: System invalidates expired OTP → Notifies delivery person → New OTP can be requested
- **Alternative**: None

---

## 13. Reconciliation (UC-S-056 to UC-S-058)

### UC-S-056: Daily Transaction Reconciliation
- **Trigger**: Scheduled (daily at 3 AM)
- **Main Flow**: System reconciles all transactions against wallet balances → Identifies discrepancies → Generates reconciliation report
- **Alternative**: Discrepancy found → Alert admin → Manual resolution required

### UC-S-057: Escrow Reconciliation
- **Trigger**: Scheduled (daily)
- **Main Flow**: System reconciles escrow balances against orders → Verifies all held funds match order values → Flags discrepancies
- **Alternative**: Discrepancy found → Alert admin → Freeze affected escrows

### UC-S-058: Vendor Settlement Reconciliation
- **Trigger**: Scheduled (after each settlement batch)
- **Main Flow**: System reconciles settlement amounts against vendor earnings → Verifies commission deductions → Flags discrepancies
- **Alternative**: Discrepancy found → Hold settlement → Alert admin

---

## 14. Scheduler & Automation (UC-S-059 to UC-S-062)

### UC-S-059: Expired Stock Hold Cleanup
- **Trigger**: Scheduled (every minute)
- **Main Flow**: System checks for expired stock holds (15 minutes elapsed) → Releases held stock → Restores availability
- **Alternative**: None

### UC-S-060: Order Auto-Completion
- **Trigger**: Scheduled (every hour)
- **Main Flow**: System checks for orders delivered 7+ days ago with no dispute → Auto-completes order → Triggers escrow release
- **Alternative**: Dispute exists → Skip order

### UC-S-061: Vendor Auto-Rejection
- **Trigger**: Scheduled (every hour)
- **Main Flow**: System checks for pending orders older than 24 hours without vendor confirmation → Auto-rejects → Refunds customer
- **Alternative**: Trusted seller → 48-hour window applied

### UC-S-062: Inactive Session Cleanup
- **Trigger**: Scheduled (every 30 minutes)
- **Main Flow**: System invalidates sessions inactive for 30+ minutes → Clears session tokens → Logs cleanup
- **Alternative**: None

---

## 15. Health Monitoring (UC-S-063 to UC-S-066)

### UC-S-063: API Health Check
- **Trigger**: Scheduled (every 5 minutes)
- **Main Flow**: System pings all critical API endpoints → Measures response time → Logs health status → Alerts if response time exceeds threshold
- **Alternative**: All endpoints healthy → Log success

### UC-S-064: Database Health Check
- **Trigger**: Scheduled (every 5 minutes)
- **Main Flow**: System checks database connection, query performance, storage usage → Logs health → Alerts on issues
- **Alternative**: Database healthy → Log success

### UC-S-065: Payment Provider Health Check
- **Trigger**: Scheduled (every 5 minutes)
- **Main Flow**: System pings payment provider APIs → Measures response time → Alerts if unavailable
- **Alternative**: Provider healthy → Log success

### UC-S-066: Error Rate Monitoring
- **Trigger**: Scheduled (every 15 minutes)
- **Main Flow**: System calculates error rate across all services → If error rate exceeds threshold (5%), sends alert to admin
- **Alternative**: Error rate normal → Log metrics

---

## 16. Analytics Computation (UC-S-067 to UC-S-070)

### UC-S-067: Product Score Recalculation
- **Trigger**: Scheduled (daily)
- **Main Flow**: System recalculates auto-scores for all products based on images, description, reviews, sales → Updates search ranking
- **Alternative**: Calculation fails → Retry → Use previous scores

### UC-S-068: Vendor Rating Recalculation
- **Trigger**: Scheduled (daily)
- **Main Flow**: System recalculates vendor ratings based on fulfillment rate, customer reviews, response time → Updates vendor scores
- **Alternative**: Calculation fails → Retry → Use previous ratings

### UC-S-069: Search Index Update
- **Trigger**: Scheduled (every 30 minutes) or on product change
- **Main Flow**: System updates search index with new/modified products → Rebuilds index if needed → Optimizes search performance
- **Alternative**: Index build fails → Retry → Alert admin

### UC-S-070: Dashboard Data Aggregation
- **Trigger**: Scheduled (every 15 minutes)
- **Main Flow**: System aggregates dashboard metrics → Total orders, revenue, users, vendors → Updates admin dashboard cache
- **Alternative**: Aggregation fails → Use stale data → Alert admin
