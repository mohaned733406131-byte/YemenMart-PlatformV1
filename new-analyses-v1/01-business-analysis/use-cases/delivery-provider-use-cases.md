# Delivery Provider Use Cases — YemenMart

## Overview

47 use cases for delivery provider-facing functionality organized by category. Each use case includes ID, name, description, actor, preconditions, main flow, and alternatives.

---

## 1. Account Management (UC-D-001 to UC-D-008)

### UC-D-001: Register as Delivery Provider
- **Actor**: New Delivery Provider
- **Precondition**: No existing delivery provider account
- **Main Flow**: Tap "Become a Delivery Partner" → Enter business details and phone → Set password → Account created → Pending KYC
- **Alternative**: Already registered → Login instead

### UC-D-002: Complete KYC Verification
- **Actor**: Registered Delivery Provider
- **Precondition**: Account pending verification
- **Main Flow**: Navigate to KYC section → Upload license, insurance, vehicle registration → Submit → Await review (24–48 hours)
- **Alternative**: Documents rejected → Re-upload → Resubmit

### UC-D-003: Login to Provider Dashboard
- **Actor**: Verified Delivery Provider
- **Precondition**: KYC approved
- **Main Flow**: Enter credentials → OTP verification → Access provider dashboard
- **Alternative**: 5 failed attempts → 15-minute lockout

### UC-D-004: Update Provider Profile
- **Actor**: Verified Delivery Provider
- **Precondition**: Active account
- **Main Flow**: Navigate to settings → Update name, phone, vehicle details, coverage area → Save
- **Alternative**: Major changes → Re-verification required

### UC-D-005: Change Password
- **Actor**: Verified Delivery Provider
- **Precondition**: Active session
- **Main Flow**: Navigate to security → Enter current password → Enter new password → Confirm → Updated
- **Alternative**: Forgot password → OTP reset flow

### UC-D-006: View Provider Dashboard
- **Actor**: Verified Delivery Provider
- **Precondition**: Active account
- **Main Flow**: Login → View dashboard overview (pending deliveries, completed today, earnings, rating)
- **Alternative**: New provider → Onboarding tour

### UC-D-007: Set Availability Status
- **Actor**: Verified Delivery Provider
- **Precondition**: Active account
- **Main Flow**: Toggle availability (Online/Offline) → When online, receive delivery requests
- **Alternative**: Auto-offline after inactivity (30 minutes)

### UC-D-008: Deactivate Provider Account
- **Actor**: Verified Delivery Provider
- **Precondition**: No active deliveries
- **Main Flow**: Navigate to settings → Request deactivation → Confirm → Account deactivated
- **Alternative**: Active deliveries exist → Cannot deactivate until completed

---

## 2. Delivery Zone Management (UC-D-009 to UC-D-014)

### UC-D-009: View Assigned Zones
- **Actor**: Verified Delivery Provider
- **Precondition**: Active account
- **Main Flow**: Navigate to zones → View assigned delivery zones on map → See zone boundaries and rules
- **Alternative**: No zones assigned → Request zone assignment

### UC-D-010: Request Zone Assignment
- **Actor**: Verified Delivery Provider
- **Precondition**: Active account, no zones assigned
- **Main Flow**: Navigate to zones → Request assignment for specific zones → Submit → Admin reviews
- **Alternative**: Zone not available → Join waitlist

### UC-D-011: View Zone Rules
- **Actor**: Verified Delivery Provider
- **Precondition**: Zones assigned
- **Main Flow**: Select zone → View rules (delivery hours, fee structure, restrictions)
- **Alternative**: None

### UC-D-012: Update Zone Coverage
- **Actor**: Verified Delivery Provider
- **Precondition**: Active account
- **Main Flow**: Navigate to zones → Update preferred zones → Submit changes → Admin reviews
- **Alternative**: Cannot reduce below minimum zones → Show warning

### UC-D-013: View Zone Performance
- **Actor**: Verified Delivery Provider
- **Precondition**: Has delivery history in zone
- **Main Flow**: Select zone → View performance metrics (deliveries completed, earnings, rating per zone)
- **Alternative**: No data in zone → Show empty state

### UC-D-014: View Zone Map
- **Actor**: Verified Delivery Provider
- **Precondition**: Zones assigned
- **Main Flow**: Navigate to zones → View interactive map showing all assigned zones with boundaries
- **Alternative**: Map fails to load → Retry

---

## 3. Delivery Bids (UC-D-015 to UC-D-020)

### UC-D-015: View Available Delivery Requests
- **Actor**: Verified Delivery Provider
- **Precondition**: Online and in assigned zone
- **Main Flow**: View incoming delivery requests → See order value, pickup location, delivery location, estimated fee
- **Alternative**: No requests available → Wait for new requests

### UC-D-016: Place Delivery Bid
- **Actor**: Verified Delivery Provider
- **Precondition**: Delivery request available
- **Main Flow**: Select request → Review details → Place bid with proposed fee → Submit → Await vendor/customer acceptance
- **Alternative**: Bid rejected → Try another request

### UC-D-017: View Active Bids
- **Actor**: Verified Delivery Provider
- **Precondition**: Has active bids
- **Main Flow**: Navigate to bids → View all pending bids with status (Pending, Accepted, Rejected)
- **Alternative**: No active bids → Show empty state

### UC-D-018: Cancel Bid
- **Actor**: Verified Delivery Provider
- **Precondition**: Bid in Pending status
- **Main Flow**: Select bid → Tap cancel → Confirm → Bid removed
- **Alternative**: Bid already accepted → Cannot cancel

### UC-D-019: View Bid History
- **Actor**: Verified Delivery Provider
- **Precondition**: Has bid history
- **Main Flow**: Navigate to bids → View all past bids with outcomes (Accepted, Rejected, Completed)
- **Alternative**: No history → Show empty state

### UC-D-020: Accept Delivery Assignment
- **Actor**: Verified Delivery Provider
- **Precondition**: Bid accepted by vendor/customer
- **Main Flow**: Receive notification → Accept assignment → Delivery added to active deliveries → Proceed to pickup
- **Alternative**: Reject assignment → Return to available requests

---

## 4. Delivery Operations (UC-D-021 to UC-D-032)

### UC-D-021: View Active Deliveries
- **Actor**: Verified Delivery Provider
- **Precondition**: Has assigned deliveries
- **Main Flow**: Navigate to deliveries → View all active deliveries with status, pickup, and delivery locations
- **Alternative**: No active deliveries → Show empty state

### UC-D-022: Navigate to Pickup
- **Actor**: Verified Delivery Provider
- **Precondition**: Active delivery assigned
- **Main Flow**: Select delivery → Tap navigate → Open maps with pickup location → Navigate to vendor
- **Alternative**: Maps not available → Show address details

### UC-D-023: Confirm Pickup
- **Actor**: Verified Delivery Provider
- **Precondition**: At pickup location
- **Main Flow**: Arrive at vendor → Tap confirm pickup → Verify items match order → Confirm pickup → Status updated to In Transit
- **Alternative**: Items missing → Report issue → Contact vendor

### UC-D-024: Navigate to Delivery
- **Actor**: Verified Delivery Provider
- **Precondition**: Items picked up
- **Main Flow**: Select delivery → Tap navigate → Open maps with delivery location → Navigate to customer
- **Alternative**: Maps not available → Show address details

### UC-D-025: Request OTP from Customer
- **Actor**: Verified Delivery Provider
- **Precondition**: Order value exceeds 50,000 YER
- **Main Flow**: Arrive at delivery location → Request OTP from customer → Customer provides OTP → Enter OTP in app → Delivery proceeds
- **Alternative**: Customer cannot provide OTP → Contact support

### UC-D-026: Capture Customer Signature
- **Actor**: Verified Delivery Provider
- **Precondition**: Order value exceeds 100,000 YER
- **Main Flow**: Arrive at delivery location → Present signature pad → Customer signs → Capture signature → Delivery proceeds
- **Alternative**: Customer refuses signature → Contact support

### UC-D-027: Capture Delivery Photo
- **Actor**: Verified Delivery Provider
- **Precondition**: At delivery location
- **Main Flow**: After customer confirmation → Take photo of delivered package at delivery location → Upload photo → Proof recorded
- **Alternative**: Camera unavailable → Delivery marked incomplete → Retry

### UC-D-028: Confirm Delivery Completion
- **Actor**: Verified Delivery Provider
- **Precondition**: All verification steps complete
- **Main Flow**: All steps done (OTP/signature if required, photo captured) → Tap confirm delivery → Delivery marked complete → Payment released to escrow
- **Alternative**: Customer refuses delivery → Mark as failed → Return to vendor

### UC-D-029: Report Failed Delivery Attempt
- **Actor**: Verified Delivery Provider
- **Precondition**: Delivery cannot be completed
- **Main Flow**: Arrive at location → Customer unavailable or address incorrect → Tap report failure → Enter reason → Photo of location → Mark as failed attempt
- **Alternative**: None

### UC-D-030: Return Items to Vendor
- **Actor**: Verified Delivery Provider
- **Precondition**: 3 failed delivery attempts
- **Main Flow**: After 3rd failed attempt → Items marked for return → Navigate to vendor location → Return items → Confirm return
- **Alternative**: Vendor unavailable → Leave at designated return point

### UC-D-031: View Delivery History
- **Actor**: Verified Delivery Provider
- **Precondition**: Has completed deliveries
- **Main Flow**: Navigate to deliveries → View all past deliveries with dates, statuses, earnings
- **Alternative**: No history → Show empty state

### UC-D-032: Update Delivery Status
- **Actor**: Verified Delivery Provider
- **Precondition**: Active delivery
- **Main Flow**: Select delivery → Update status (Picked Up, In Transit, Out for Delivery, Delivered) → Customer notified
- **Alternative**: Status invalid → Show error

---

## 5. Proof of Delivery (UC-D-033 to UC-D-036)

### UC-D-033: Capture GPS Location
- **Actor**: Verified Delivery Provider
- **Precondition**: Active delivery
- **Main Flow**: App automatically captures GPS at pickup and delivery → Location stored as proof
- **Alternative**: GPS unavailable → Manual location entry → Photo required

### UC-D-034: Capture Timestamp
- **Actor**: Verified Delivery Provider
- **Precondition**: Active delivery
- **Main Flow**: App automatically records timestamp at each delivery step → Stored as proof
- **Alternative**: None

### UC-D-035: View Delivery Proof
- **Actor**: Verified Delivery Provider
- **Precondition**: Completed delivery
- **Main Flow**: Select delivery → View proof (photo, GPS, timestamp, OTP, signature if applicable)
- **Alternative**: No proof → "Proof not captured" warning

### UC-D-036: Resubmit Delivery Proof
- **Actor**: Verified Delivery Provider
- **Precondition**: Proof incomplete
- **Main Flow**: Receive notification of incomplete proof → Navigate to delivery → Resubmit missing proof elements
- **Alternative**: Proof window expired → Contact support

---

## 6. Earnings & Payments (UC-D-037 to UC-D-041)

### UC-D-037: View Earnings Dashboard
- **Actor**: Verified Delivery Provider
- **Precondition**: Has completed deliveries
- **Main Flow**: Navigate to earnings → View total earnings, pending payouts, earnings per delivery
- **Alternative**: No earnings → Show empty state

### UC-D-038: View Earnings History
- **Actor**: Verified Delivery Provider
- **Precondition**: Has earnings
- **Main Flow**: Navigate to earnings → View all past earnings with dates, delivery details, amounts
- **Alternative**: No history → Show empty state

### UC-D-039: Request Payout
- **Actor**: Verified Delivery Provider
- **Precondition**: Available balance exceeds minimum payout
- **Main Flow**: Navigate to earnings → Tap request payout → Enter amount → Confirm → Payout processed
- **Alternative**: Below minimum → "Minimum payout not reached"

### UC-D-040: Set Payout Method
- **Actor**: Verified Delivery Provider
- **Precondition**: Active account
- **Main Flow**: Navigate to earnings → Set payout method (wallet, bank transfer) → Enter details → Save
- **Alternative**: Bank transfer → Additional verification required

### UC-D-041: View Payout History
- **Actor**: Verified Delivery Provider
- **Precondition**: Has payouts
- **Main Flow**: Navigate to earnings → View all past payouts with dates, amounts, methods
- **Alternative**: No payouts → Show empty state

---

## 7. Performance & Rating (UC-D-042 to UC-D-045)

### UC-D-042: View Performance Dashboard
- **Actor**: Verified Delivery Provider
- **Precondition**: Active account
- **Main Flow**: Navigate to performance → View metrics (on-time rate, success rate, customer rating, response time)
- **Alternative**: Insufficient data → "Metrics available after 10 deliveries"

### UC-D-043: View Customer Ratings
- **Actor**: Verified Delivery Provider
- **Precondition**: Has completed deliveries
- **Main Flow**: Navigate to performance → View all customer ratings and comments
- **Alternative**: No ratings → Show empty state

### UC-D-044: View Rating Trends
- **Actor**: Verified Delivery Provider
- **Precondition**: Has ratings
- **Main Flow**: Navigate to performance → View rating trends over time → See improvement areas
- **Alternative**: Insufficient data → Show message

### UC-D-045: Respond to Rating
- **Actor**: Verified Delivery Provider
- **Precondition**: Has rating with comment
- **Main Flow**: Select rating → Write response → Submit → Response displayed
- **Alternative**: Rating period expired → Cannot respond

---

## 8. Support (UC-D-046 to UC-D-047)

### UC-D-046: Contact Platform Support
- **Actor**: Verified Delivery Provider
- **Precondition**: Has issue
- **Main Flow**: Navigate to support → Select category → Describe issue → Submit → Ticket created
- **Alternative**: No internet → SMS support available

### UC-D-047: View Support Tickets
- **Actor**: Verified Delivery Provider
- **Precondition**: Has support tickets
- **Main Flow**: Navigate to support → View ticket list → Select for details and updates
- **Alternative**: No tickets → Show empty state
