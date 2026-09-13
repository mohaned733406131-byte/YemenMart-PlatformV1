# YemenMart Error States & Edge Cases

**Document Version:** 1.0
**Last Updated:** 2026-09-13
**Classification:** UI/UX Specification
**Applicable Portals:** Customer, Vendor, Admin, Delivery Provider

---

## Table of Contents

1. [Global Error Handling Strategy](#1-global-error-handling-strategy)
2. [Error Boundary Implementation](#2-error-boundary-implementation)
3. [Offline Behavior](#3-offline-behavior)
4. [Retry Mechanisms](#4-retry-mechanisms)
5. [Error Logging](#5-error-logging)
6. [HTTP Errors](#6-http-errors)
7. [Authentication Errors](#7-authentication-errors)
8. [Payment/Wallet Errors](#8-paymentwallet-errors)
9. [Order Errors](#9-order-errors)
10. [Delivery Errors](#10-delivery-errors)
11. [Product Errors](#11-product-errors)
12. [Network/Connectivity Errors](#12-networkconnectivity-errors)
13. [Validation Errors](#13-validation-errors)
14. [Empty States](#14-empty-states)
15. [Visual Treatment Reference](#15-visual-treatment-reference)
16. [Accessibility Reference](#16-accessibility-reference)

---

## 1. Global Error Handling Strategy

### 1.1 Error Handling Principles

| # | Principle | Description |
|---|-----------|-------------|
| 1 | **Arabic-First Messages** | All error messages display in Arabic by default. English available via toggle. |
| 2 | **RTL Compliance** | All error UIs respect RTL layout. Icons, progress indicators, and alignment mirror accordingly. |
| 3 | **User-Centric Language** | Error messages avoid technical jargon. Use simple, actionable Arabic. |
| 4 | **Non-Blocking Recovery** | Where possible, allow the user to continue partial tasks rather than blocking entirely. |
| 5 | **Graceful Degradation** | If a feature fails, degrade to a simpler version rather than showing a blank screen. |
| 6 | **No Silent Failures** | Every error must produce visible feedback. No error is swallowed without user notification. |
| 7 | **Context Preservation** | When an error occurs, preserve user input so they do not lose progress. |
| 8 | **Escalation Path** | Every error screen provides a path to support (chat, phone, or help center). |

### 1.2 Error Response Hierarchy

```
┌─────────────────────────────────────────────────┐
│              CRITICAL ERRORS                      │
│  (Blocking: payment, auth, data loss)            │
│  → Modal dialog, forces attention                │
│  → Cannot be dismissed without action            │
├─────────────────────────────────────────────────┤
│              HIGH ERRORS                          │
│  (Functional: failed actions, state conflicts)   │
│  → Banner or toast, blocks specific action       │
│  → Auto-dismiss after 8 seconds                  │
├─────────────────────────────────────────────────┤
│              MEDIUM ERRORS                        │
│  (Non-critical: validation, expired tokens)      │
│  → Inline error, field-level feedback            │
│  → Persists until corrected                      │
├─────────────────────────────────────────────────┤
│              LOW ERRORS                           │
│  (Informational: empty states, suggestions)      │
│  → Card or placeholder, non-urgent               │
│  → Persists until content available              │
└─────────────────────────────────────────────────┘
```

### 1.3 Error Display Rules

| Rule | Description |
|------|-------------|
| **Single error at a time** | Only one top-level error displays at a time. Multiple errors queue and display sequentially. |
| **Stacking limit** | Maximum 3 toast notifications visible simultaneously. Oldest dismissed when new arrives. |
| **No error loops** | If the same error triggers 3+ times consecutively, escalate to a modal with support contact. |
| **Timeout display** | Toast errors: 5 seconds. Banner errors: 8 seconds. Inline errors: persistent. Modal: until dismissed. |
| **Haptic feedback** | Critical and High errors trigger device vibration (200ms) on supported devices. |

---

## 2. Error Boundary Implementation

### 2.1 Error Boundary Scope

```
App Root
├── Auth Boundary (login/registration flows)
├── Portal Boundary (each portal isolated)
│   ├── Customer Portal Boundary
│   ├── Vendor Portal Boundary
│   ├── Admin Portal Boundary
│   └── Delivery Portal Boundary
├── Feature Boundary (per major feature)
│   ├── Cart & Checkout Boundary
│   ├── Order Management Boundary
│   ├── Wallet Boundary
│   └── Product Search Boundary
└── Component Boundary (critical widgets)
    ├── Payment Component
    ├── Map Component
    └── Image Upload Component
```

### 2.2 Error Boundary UI

**When a React error boundary catches an error:**

| Element | Content |
|---------|---------|
| **Icon** | Broken connection icon (_RF_ style) |
| **Arabic Title** | `حدث خطأ غير متوقع` |
| **English Title** | `Something went wrong` |
| **Arabic Message** | `نعتذر عن الإزعاج. يرجى المحاولة مرة أخرى أو التواصل مع الدعم الفني.` |
| **English Message** | `We apologize for the inconvenience. Please try again or contact support.` |
| **Primary Button** | `إعادة المحاولة` / `Try Again` |
| **Secondary Button** | `العودة للرئيسية` / `Go to Home` |
| **Support Link** | `الدعم الفني` / `Contact Support` |
| **Error ID** | Display a unique error reference ID (e.g., `ERR-20260913-A3F2`) for support tickets |

### 2.3 Recovery Actions per Boundary

| Boundary | Recovery Action |
|----------|-----------------|
| Auth | Redirect to login page. Clear stale auth state. |
| Portal | Redirect to portal home. Preserve deep link if possible. |
| Feature | Reload the feature component. Preserve form data in localStorage. |
| Component | Replace component with placeholder. Show retry button. |

---

## 3. Offline Behavior

### 3.1 Offline Detection

| Mechanism | Description |
|-----------|-------------|
| **Navigator.onLine** | Primary detection. Listen to `online`/`offline` events. |
| **Heartbeat ping** | Every 30 seconds, ping `/health`. If 3 consecutive pings fail, mark as offline. |
| **Request-level detection** | If a request fails with `ERR_NETWORK`, mark offline immediately. |

### 3.2 Offline UI State

| Element | Behavior |
|---------|----------|
| **Banner** | Persistent top banner: `لا يوجد اتصال بالإنترنت` / `No internet connection` with pulsing red dot |
| **Cached data** | Show cached product listings, order history, and profile data with `offline` badge |
| **Queued actions** | Queue write operations (add to cart, place order) with `pending sync` indicator |
| **Sync indicator** | When back online: `جاري المزامنة...` / `Syncing...` with progress bar |

### 3.3 Offline Capabilities by Feature

| Feature | Available Offline | Notes |
|---------|-------------------|-------|
| Browse products | Yes (cached) | Show cached listings, limit to last 50 products viewed |
| Product detail | Yes (cached) | Last viewed product details available |
| Cart | Yes (local) | Cart managed in localStorage. Syncs on reconnect. |
| Place order | No | Queue and notify user to retry when online |
| Wallet top-up | No | Requires real-time payment verification |
| Wallet balance | Yes (cached) | Show `last synced: X minutes ago` |
| Order history | Yes (cached) | Last 20 orders cached |
| Track order | No | Requires real-time delivery status |
| Search | No | Show `يتطلب اتصال بالإنترنت` message |
| Profile | Yes (cached) | Read-only. Edits queued for sync. |
| Reviews | No | Queue submission for sync |
| Chat/Support | No | Show `لا يمكن إرسال الرسائل بدون إنترنت` |

### 3.4 Offline Queue Rules

| Rule | Value |
|------|-------|
| Maximum queued actions | 20 |
| Queue expiry | 24 hours (actions older than 24h are discarded) |
| Conflict resolution | Last-write-wins for profile updates. Server-authoritative for orders. |
| User notification | Show `تم حفظ إجراءك. سيتم التنفيذ عند عودة الاتصال.` / `Your action is saved. It will execute when connection is restored.` |

---

## 4. Retry Mechanisms

### 4.1 Retry Strategy by Error Type

| Error Type | Retry? | Max Retries | Delay | Backoff |
|------------|--------|-------------|-------|---------|
| Network timeout | Yes | 3 | 1s, 2s, 4s | Exponential |
| Server 5xx | Yes | 3 | 2s, 4s, 8s | Exponential |
| Rate limit (429) | Yes | 1 | Per `Retry-After` header | Fixed |
| Auth token refresh | Yes | 2 | 500ms | Exponential |
| Payment processing | Yes | 2 | 3s, 6s | Exponential |
| OTP send | Yes | 2 | 5s, 10s | Exponential |
| Image upload | Yes | 2 | 2s, 4s | Exponential |
| Validation (400/422) | No | - | - | - |
| Conflict (409) | No | - | - | - |
| Server error (500) | Yes | 1 | 5s | Fixed |

### 4.2 Retry UX

| Scenario | UI Treatment |
|----------|--------------|
| **Auto-retry in progress** | Subtle spinner next to the action button with `جاري المحاولة...` / `Retrying...` |
| **Retry failed** | Toast: `فشلت المحاولة. يرجى المحاولة لاحقاً.` / `Retry failed. Please try again later.` |
| **Manual retry available** | Button changes to `إعادة المحاولة` / `Try Again` with refresh icon |
| **All retries exhausted** | Modal with support contact: `لم نتمكن من إتمام العملية. يرجى التواصل مع الدعم الفني.` |

### 4.3 Retry Exclusion Rules

| Rule | Description |
|------|-------------|
| **Destructive operations** | Never auto-retry DELETE or CANCEL operations |
| **Payment operations** | Never auto-retry payment deduction. Only auto-retry balance check. |
| **State-changing operations** | Never auto-retry ORDER PLACEMENT. User must explicitly retry. |
| **Maximum retry duration** | Total retry window capped at 30 seconds regardless of backoff |

---

## 5. Error Logging

### 5.1 Client-Side Error Logging

| Field | Description |
|-------|-------------|
| `error_id` | Unique identifier (ER-{TYPE}-{SEQUENCE}) |
| `timestamp` | ISO 8601 timestamp |
| `user_id` | Hashed user ID (never log raw phone numbers) |
| `session_id` | Current session ID |
| `portal` | customer / vendor / admin / delivery |
| `route` | Current route path |
| `action` | User action that triggered the error |
| `http_status` | HTTP status code (if applicable) |
| `error_message` | Error message (sanitized, no PII) |
| `stack_trace` | First 500 chars of stack trace |
| `device_info` | OS, browser, screen size |
| `network_type` | wifi / 4g / 3g / 2g / offline |
| `retry_count` | Number of retries attempted |
| `user_agent` | Sanitized user agent string |

### 5.2 Error Severity Mapping

| Severity | Action Required | Alert | SLA |
|----------|----------------|-------|-----|
| **Critical** | Immediate page, block user | PagerDuty + Slack | Response < 5 min |
| **High** | Log + alert on-call | Slack #alerts | Response < 15 min |
| **Medium** | Log + daily digest | Email digest | Fix within 24h |
| **Low** | Log only | Weekly report | Fix within sprint |

### 5.3 PII Protection Rules

| Rule | Description |
|------|-------------|
| **Never log** | Phone numbers, OTP codes, wallet balances, payment tokens |
| **Hash** | User IDs, session IDs, device IDs |
| **Sanitize** | Remove all Arabic text from error logs (use error codes only) |
| **Retention** | Error logs retained for 90 days. Critical errors for 1 year. |
| **Encryption** | All error logs encrypted at rest with AES-256 |

---

## 6. HTTP Errors

### ER-HTTP-001: 400 Bad Request

| Field | Value |
|-------|-------|
| **Error ID** | ER-HTTP-001 |
| **Trigger** | Server receives malformed request, invalid JSON, or missing required parameters |
| **Arabic UI Message** | `طلب غير صالح. يرجى التحقق من البيانات المدخلة والمحاولة مرة أخرى.` |
| **English UI Message** | `Bad request. Please check your input and try again.` |
| **Recovery** | Refresh the form. If persists, clear browser cache and retry. |
| **Severity** | Medium |
| **Visual Treatment** | Inline error on the form field(s) causing the issue. If field cannot be identified, show toast. |
| **Accessibility Announcement** | `خطأ في الطلب. يرجى التحقق من البيانات المدخلة.` (polite) |
| **Analytics Event** | `error_http_400` with `endpoint` and `field` properties |
| **Log Level** | WARN |

### ER-HTTP-002: 401 Unauthorized

| Field | Value |
|-------|-------|
| **Error ID** | ER-HTTP-002 |
| **Trigger** | Missing, expired, or invalid authentication token |
| **Arabic UI Message** | `جلستكنته قد انتهت. يرجى تسجيل الدخول مرة أخرى.` |
| **English UI Message** | `Your session has expired. Please log in again.` |
| **Recovery** | Redirect to login page. Preserve current route for post-login redirect. |
| **Severity** | High |
| **Visual Treatment** | Modal dialog (non-dismissible). Auto-redirect after 3 seconds with countdown. |
| **Accessibility Announcement** | `انتهت الجلسة. جاري التحويل إلى صفحة تسجيل الدخول.` (assertive) |
| **Analytics Event** | `error_http_401` with `expired_token_age` property |
| **Log Level** | INFO |

### ER-HTTP-003: 403 Forbidden

| Field | Value |
|-------|-------|
| **Error ID** | ER-HTTP-003 |
| **Trigger** | Authenticated user lacks permission for the requested resource |
| **Arabic UI Message** | `ليس لديك صلاحية للوصول إلى هذا المحتوى.` |
| **English UI Message** | `You don't have permission to access this content.` |
| **Recovery** | Navigate back. If this is the user's own resource, contact support. |
| **Severity** | Medium |
| **Visual Treatment** | Full-page illustration with locked padlock icon. Back button + support link. |
| **Accessibility Announcement** | `ليس لديك صلاحية للوصول إلى هذا المحتوى.` (polite) |
| **Analytics Event** | `error_http_403` with `resource` and `user_role` properties |
| **Log Level** | WARN (potential security issue if repeated) |

### ER-HTTP-004: 404 Not Found

| Field | Value |
|-------|-------|
| **Error ID** | ER-HTTP-004 |
| **Trigger** | Requested resource does not exist (product, order, page) |
| **Arabic UI Message** | `الصفحة المطلوبة غير موجودة. ربما تم حذفها أو تغيير رابطها.` |
| **English UI Message** | `The requested page was not found. It may have been deleted or moved.` |
| **Recovery** | Go to homepage. Use search to find the product/content. |
| **Severity** | Low |
| **Visual Treatment** | Custom illustration (confused cart icon). Search bar prominently displayed. |
| **Accessibility Announcement** | `الصفحة غير موجودة.` (polite) |
| **Analytics Event** | `error_http_404` with `attempted_url` and `referrer` properties |
| **Log Level** | INFO |

### ER-HTTP-005: 409 Conflict

| Field | Value |
|-------|-------|
| **Error ID** | ER-HTTP-005 |
| **Trigger** | Request conflicts with current state (e.g., order already cancelled, product already added) |
| **Arabic UI Message** | `تعذر تنفيذ الطلب لأن الحالة الحالية لا تسمح بذلك. يرجى تحديث الصفحة.` |
| **English UI Message** | `Could not complete the request due to a state conflict. Please refresh the page.` |
| **Recovery** | Refresh page to get latest state. If persists, contact support. |
| **Severity** | Medium |
| **Visual Treatment** | Toast notification with refresh icon. Auto-refresh option after 2 seconds. |
| **Accessibility Announcement** | `تعذر التنفيذ بسبب تعارض في الحالة. جاري تحديث الصفحة.` (polite) |
| **Analytics Event** | `error_http_409` with `conflict_type` and `resource_state` properties |
| **Log Level** | WARN |

### ER-HTTP-006: 422 Unprocessable Entity

| Field | Value |
|-------|-------|
| **Error ID** | ER-HTTP-006 |
| **Trigger** | Request is well-formed but semantically invalid (e.g., order amount below minimum) |
| **Arabic UI Message** | `البيانات المدخلة غير صالحة. يرجى مراجعة الحقول المميزة.` |
| **English UI Message** | `The data is invalid. Please review the highlighted fields.` |
| **Recovery** | Review highlighted fields, correct input, and resubmit. |
| **Severity** | Medium |
| **Visual Treatment** | Inline field-level errors with red border. Summary banner at top of form. |
| **Accessibility Announcement** | `خطأ في البيانات. يرجى مراجعة الحقول المميزة.` (polite) |
| **Analytics Event** | `error_http_422` with `validation_errors` array property |
| **Log Level** | INFO |

### ER-HTTP-007: 429 Rate Limited

| Field | Value |
|-------|-------|
| **Error ID** | ER-HTTP-007 |
| **Trigger** | Too many requests sent. Exceeds rate limit (OTP: 3/min, API: 60/min) |
| **Arabic UI Message** | `تم تجاوز الحد المسموح. يرجى الانتظار {duration} قبل المحاولة مرة أخرى.` |
| **English UI Message** | `Rate limit exceeded. Please wait {duration} before trying again.` |
| **Recovery** | Wait for the specified duration. Disable the triggering button with countdown. |
| **Severity** | Medium |
| **Visual Treatment** | Toast with countdown timer. Button disabled with `متاح بعد {seconds}s` / `Available in {seconds}s`. |
| **Accessibility Announcement** | `تم تجاوز الحد المسموح. يرجى الانتظار {duration}.` (polite) |
| **Analytics Event** | `error_http_429` with `rate_limit_type` and `retry_after` properties |
| **Log Level** | WARN |

### ER-HTTP-008: 500 Internal Server Error

| Field | Value |
|-------|-------|
| **Error ID** | ER-HTTP-008 |
| **Trigger** | Unexpected server-side failure |
| **Arabic UI Message** | `حدث خطأ في الخادم. فريقنا يعمل على إصلاح المشكلة. يرجى المحاولة لاحقاً.` |
| **English UI Message** | `An internal server error occurred. Our team is working on it. Please try again later.` |
| **Recovery** | Retry after 5 seconds. If persists, report issue with auto-generated error reference. |
| **Severity** | High |
| **Visual Treatment** | Full-page error with apology illustration. Retry button + support contact + error reference ID. |
| **Accessibility Announcement** | `خطأ في الخادم. يرجى المحاولة لاحقاً.` (assertive) |
| **Analytics Event** | `error_http_500` with `endpoint` and `correlation_id` properties |
| **Log Level** | ERROR (triggers alert) |

### ER-HTTP-009: 503 Service Unavailable

| Field | Value |
|-------|-------|
| **Error ID** | ER-HTTP-009 |
| **Trigger** | Server is temporarily unavailable (maintenance, overload) |
| **Arabic UI Message** | `الخدمة غير متوفرة حالياً للصيانة. يرجى المحاولة بعد بضع دقائق.` |
| **English UI Message** | `The service is temporarily unavailable for maintenance. Please try again in a few minutes.` |
| **Recovery** | Auto-retry every 30 seconds. Show maintenance countdown if ETA available. |
| **Severity** | High |
| **Visual Treatment** | Full-page maintenance screen with progress bar (if ETA available) or spinning gear animation. |
| **Accessibility Announcement** | `الخدمة غير متوفرة حالياً. يرجى المحاولة لاحقاً.` (assertive) |
| **Analytics Event** | `error_http_503` with `maintenance_eta` and `service` properties |
| **Log Level** | ERROR (triggers alert) |

---

## 7. Authentication Errors

### ER-AUTH-001: Invalid Phone Number Format

| Field | Value |
|-------|-------|
| **Error ID** | ER-AUTH-001 |
| **Trigger** | Phone number does not match Yemen format: +967 followed by 9 digits (e.g., +967 7XX XXX XXX) |
| **Arabic UI Message** | `رقم الهاتف غير صحيح. يرجى إدخال رقم يمني يبدأ بـ 967.` |
| **English UI Message** | `Invalid phone number. Please enter a valid Yemeni number starting with 967.` |
| **Recovery** | Correct the phone number. Use the country code selector for guidance. |
| **Severity** | Medium |
| **Visual Treatment** | Inline error below input field with red border. Phone format hint visible. |
| **Accessibility Announcement** | `رقم الهاتف غير صحيح. يرجى إدخال رقم يمني صالح.` (polite) |
| **Analytics Event** | `error_auth_invalid_phone` with `input_length` and `has_country_code` properties |
| **Log Level** | INFO |

### ER-AUTH-002: OTP Expired

| Field | Value |
|-------|-------|
| **Error ID** | ER-AUTH-002 |
| **Trigger** | OTP code entered after expiration (5-minute validity window) |
| **Arabic UI Message** | `رمز التحقق منتهي الصلاحية. يرجى طلب رمز جديد.` |
| **English UI Message** | `The verification code has expired. Please request a new code.` |
| **Recovery** | Tap "إعادة الإرسال" / "Resend" button. New OTP sent via SMS. |
| **Severity** | Medium |
| **Visual Treatment** | Toast notification. Resend button enabled with countdown timer reset. |
| **Accessibility Announcement** | `انتهت صلاحية رمز التحقق. يرجى طلب رمز جديد.` (polite) |
| **Analytics Event** | `error_auth_otp_expired` with `otp_age_seconds` property |
| **Log Level** | INFO |

### ER-AUTH-003: OTP Incorrect

| Field | Value |
|-------|-------|
| **Error ID** | ER-AUTH-003 |
| **Trigger** | Incorrect OTP entered. Shows remaining attempts: ` Attempts remaining: {count}/3` |
| **Arabic UI Message** | `رمز التحقق غير صحيح. متبقي {count} محاولة من أصل 3.` |
| **English UI Message** | `Incorrect verification code. {count} attempts remaining out of 3.` |
| **Recovery** | Re-enter OTP. After 3 failures, account locks for 15 minutes. |
| **Severity** | High |
| **Visual Treatment** | Inline error with remaining attempts counter. Input field shakes (CSS animation). |
| **Accessibility Announcement** | `رمز غير صحيح. متبقي {count} محاولات.` (assertive) |
| **Analytics Event** | `error_auth_otp_incorrect` with `attempt_number` and `remaining` properties |
| **Log Level** | WARN |

### ER-AUTH-004: Account Locked

| Field | Value |
|-------|-------|
| **Error ID** | ER-AUTH-004 |
| **Trigger** | 5 consecutive failed login attempts. Account locked for 15 minutes. |
| **Arabic UI Message** | `تم قفل الحساب مؤقتاً بسبب محاولات تسجيل دخول فاشلة متعددة. يرجى المحاولة بعد 15 دقيقة أو التواصل مع الدعم الفني.` |
| **English UI Message** | `Account temporarily locked due to multiple failed login attempts. Please try again in 15 minutes or contact support.` |
| **Recovery** | Wait 15 minutes. Contact support if locked out. Cannot bypass lock. |
| **Severity** | Critical |
| **Visual Treatment** | Modal dialog (non-dismissible). Countdown timer. Support phone number displayed. |
| **Accessibility Announcement** | `تم قفل الحساب. يرجى الانتظار 15 دقيقة.` (assertive) |
| **Analytics Event** | `error_auth_account_locked` with `failed_attempts` and `lock_duration` properties |
| **Log Level** | ERROR (triggers security alert) |

### ER-AUTH-005: Session Expired

| Field | Value |
|-------|-------|
| **Error ID** | ER-AUTH-005 |
| **Trigger** | JWT token expired (24-hour validity). Refresh token also expired or invalid. |
| **Arabic UI Message** | `انتهت صلاحية الجلسة. يرجى تسجيل الدخول مرة أخرى.` |
| **English UI Message** | `Your session has expired. Please log in again.` |
| **Recovery** | Redirect to login. Preserve destination route. Show "تسجيل الدخول" / "Log In" button. |
| **Severity** | High |
| **Visual Treatment** | Full-screen overlay with login prompt. 3-second countdown before redirect. |
| **Accessibility Announcement** | `انتهت الجلسة. جاري التحويل إلى صفحة تسجيل الدخول.` (assertive) |
| **Analytics Event** | `error_auth_session_expired` with `session_duration` and `token_type` properties |
| **Log Level** | INFO |

### ER-AUTH-006: Token Refresh Failed

| Field | Value |
|-------|-------|
| **Error ID** | ER-AUTH-006 |
| **Trigger** | Silent token refresh fails (network error, server error, invalid refresh token) |
| **Arabic UI Message** | `تعذر تحديث الجلسة. يرجى تسجيل الدخول مرة أخرى.` |
| **English UI Message** | `Could not refresh your session. Please log in again.` |
| **Recovery** | Redirect to login. Attempt refresh once more before redirect. |
| **Severity** | High |
| **Visual Treatment** | Toast notification, then redirect to login after 2 seconds. |
| **Accessibility Announcement** | `تعذر تحديث الجلسة. يرجى إعادة تسجيل الدخول.` (assertive) |
| **Analytics Event** | `error_auth_refresh_failed` with `refresh_token_age` and `failure_reason` properties |
| **Log Level** | WARN |

### ER-AUTH-007: Phone Number Already Registered

| Field | Value |
|-------|-------|
| **Error ID** | ER-AUTH-007 |
| **Trigger** | User attempts to register with an existing phone number |
| **Arabic UI Message** | `رقم الهاتف مسجل بالفعل. يرجى تسجيل الدخول بدلاً من ذلك.` |
| **English UI Message** | `This phone number is already registered. Please log in instead.` |
| **Recovery** | Tap "تسجيل الدخول" / "Log In" link to switch to login flow. |
| **Severity** | Medium |
| **Visual Treatment** | Inline error with clickable link to login page. |
| **Accessibility Announcement** | `رقم الهاتف مسجل بالفعل. يرجى تسجيل الدخول.` (polite) |
| **Analytics Event** | `error_auth_phone_exists` with `registration_source` property |
| **Log Level** | INFO |

### ER-AUTH-008: Password Too Weak

| Field | Value |
|-------|-------|
| **Error ID** | ER-AUTH-008 |
| **Trigger** | Password does not meet minimum requirements: 8+ chars, 1 uppercase, 1 lowercase, 1 number |
| **Arabic UI Message** | `كلمة المرور ضعيفة. يجب أن تحتوي على 8 أحرف على الأقل، حرف كبير، حرف صغير، ورقم.` |
| **English UI Message** | `Weak password. Must contain at least 8 characters, one uppercase, one lowercase, and one number.` |
| **Recovery** | Strengthen password according to requirements. Show real-time strength indicator. |
| **Severity** | Medium |
| **Visual Treatment** | Inline error with password strength meter (red → yellow → green). Requirements checklist. |
| **Accessibility Announcement** | `كلمة المرور ضعيفة. يرجى تقويتها وفقاً للمتطلبات.` (polite) |
| **Analytics Event** | `error_auth_weak_password` with `strength_score` property |
| **Log Level** | INFO |

### ER-AUTH-009: Rate Limit Exceeded

| Field | Value |
|-------|-------|
| **Error ID** | ER-AUTH-009 |
| **Trigger** | Too many OTP requests (max 3 per minute) or login attempts (max 5 per 15 minutes) |
| **Arabic UI Message** | `تم تجاوز عدد المحاولات المسموح. يرجى الانتظار {duration} قبل المحاولة مرة أخرى.` |
| **English UI Message** | `Too many attempts. Please wait {duration} before trying again.` |
| **Recovery** | Wait for countdown. OTP buttons disabled during cooldown. |
| **Severity** | High |
| **Visual Treatment** | Toast with countdown timer. Input and buttons disabled. Red warning icon. |
| **Accessibility Announcement** | `تم تجاوز الحد المسموح. يرجى الانتظار {duration}.` (assertive) |
| **Analytics Event** | `error_auth_rate_limited` with `limit_type` and `cooldown_seconds` properties |
| **Log Level** | WARN |

---

## 8. Payment/Wallet Errors

### ER-PAY-001: Insufficient Wallet Balance

| Field | Value |
|-------|-------|
| **Error ID** | ER-PAY-001 |
| **Trigger** | Wallet balance is less than order total at time of checkout |
| **Arabic UI Message** | `رصيد المحفظة غير كافٍ. الرصيد الحالي: {balance} يمني. المطلوب: {required} يمني.` |
| **English UI Message** | `Insufficient wallet balance. Current: {balance} YER. Required: {required} YER.` |
| **Recovery** | Top up wallet. Show top-up button prominently. Offer to reduce cart. |
| **Severity** | Critical |
| **Visual Treatment** | Modal dialog with wallet icon, balance display, and "شحن المحفظة" / "Top Up Wallet" primary button. |
| **Accessibility Announcement** | `رصيد المحفظة غير كافٍ. الرصيد الحالي {balance} يمني.` (assertive) |
| **Analytics Event** | `error_pay_insufficient` with `balance` and `required` properties |
| **Log Level** | INFO |

### ER-PAY-002: Wallet Top-Up Failed

| Field | Value |
|-------|-------|
| **Error ID** | ER-PAY-002 |
| **Trigger** | Top-up transaction failed (payment gateway error, network timeout, bank rejection) |
| **Arabic UI Message** | `فشل شحن المحفظة. لم يتم خصم أي مبلغ. يرجى المحاولة مرة أخرى أو استخدام طريقة دفع أخرى.` |
| **English UI Message** | `Wallet top-up failed. No amount was charged. Please try again or use a different payment method.` |
| **Recovery** | Retry top-up. Try different amount. Contact bank if card was charged. |
| **Severity** | Critical |
| **Visual Treatment** | Modal dialog with warning icon. "إعادة المحاولة" / "Retry" and "طرق الدفع الأخرى" / "Other Methods" buttons. |
| **Accessibility Announcement** | `فشل شحن المحفظة. لم يتم خصم أي مبلغ.` (assertive) |
| **Analytics Event** | `error_pay_topup_failed` with `amount` and `failure_reason` properties |
| **Log Level** | ERROR |

### ER-PAY-003: Transaction Timeout

| Field | Value |
|-------|-------|
| **Error ID** | ER-PAY-003 |
| **Trigger** | Payment processing exceeded 30-second timeout |
| **Arabic UI Message** | `انتهت مهلة المعاملة. يرجى التحقق من رصيد المحفظة قبل المحاولة مرة أخرى.` |
| **English UI Message** | `Transaction timed out. Please check your wallet balance before trying again.` |
| **Recovery** | Check wallet balance. If charged, contact support with transaction reference. |
| **Severity** | Critical |
| **Visual Treatment** | Modal dialog with clock icon. Show transaction reference ID. "التحقق من الرصيد" / "Check Balance" button. |
| **Accessibility Announcement** | `انتهت مهلة المعاملة. يرجى التحقق من الرصيد.` (assertive) |
| **Analytics Event** | `error_pay_timeout` with `timeout_duration` and `endpoint` properties |
| **Log Level** | ERROR |

### ER-PAY-004: Double-Spend Attempt

| Field | Value |
|-------|-------|
| **Error ID** | ER-PAY-004 |
| **Trigger** | System detects attempt to use the same wallet balance for two concurrent transactions |
| **Arabic UI Message** | `تم اكتشاف محاولة دفع مزدوجة. لم يتم تنفيذ أي معاملة. يرجى المحاولة مرة أخرى.` |
| **English UI Message** | `Double-spend attempt detected. No transaction was executed. Please try again.` |
| **Recovery** | Wait 5 seconds and retry. Check transaction history for any completed charge. |
| **Severity** | Critical |
| **Visual Treatment** | Modal dialog with shield icon (security). Strong emphasis on "no money was taken." |
| **Accessibility Announcement** | `تم اكتشاف محاولة دفع مزدوجة. لم يتم خصم أي مبلغ.` (assertive) |
| **Analytics Event** | `error_pay_double_spend` with `transaction_ids` array property |
| **Log Level** | ERROR (triggers security alert) |

### ER-PAY-005: Invalid Payment Method

| Field | Value |
|-------|-------|
| **Error ID** | ER-PAY-005 |
| **Trigger** | Selected payment method is not supported or has been deactivated |
| **Arabic UI Message** | `طريقة الدفع المحددة غير مدعومة حالياً. يرجى اختيار طريقة أخرى.` |
| **English UI Message** | `The selected payment method is not currently supported. Please choose another.` |
| **Recovery** | Select wallet as payment method (primary method). Contact support for alternatives. |
| **Severity** | Medium |
| **Visual Treatment** | Inline error on payment method selector. Auto-select wallet if available. |
| **Accessibility Announcement** | `طريقة الدفع غير مدعومة. يرجى اختيار طريقة أخرى.` (polite) |
| **Analytics Event** | `error_pay_invalid_method` with `method_type` property |
| **Log Level** | INFO |

### ER-PAY-006: Escrow Hold Error

| Field | Value |
|-------|-------|
| **Error ID** | ER-PAY-006 |
| **Trigger** | System fails to place funds in escrow during order placement (insufficient balance race condition, system error) |
| **Arabic UI Message** | `تعذر حجز المبلغ في حساب الضمان. يرجى المحاولة مرة أخرى. لم يتم خصم أي مبلغ.` |
| **English UI Message** | `Could not hold funds in escrow. Please try again. No amount was charged.` |
| **Recovery** | Retry order placement. If balance decreased without order, contact support immediately. |
| **Severity** | Critical |
| **Visual Treatment** | Modal dialog with lock icon. Clear "no charge" messaging. Retry and support buttons. |
| **Accessibility Announcement** | `تعذر حجز المبلغ. لم يتم خصم أي مبلغ. يرجى المحاولة مرة أخرى.` (assertive) |
| **Analytics Event** | `error_pay_escrow_hold` with `order_amount` and `wallet_balance` properties |
| **Log Level** | ERROR (triggers alert) |

### ER-PAY-007: Refund Processing Error

| Field | Value |
|-------|-------|
| **Error ID** | ER-PAY-007 |
| **Trigger** | Refund fails to process back to wallet (system error, wallet frozen, balance conflict) |
| **Arabic UI Message** | `تعذر معالجة الاسترداد. تم تسجيل طلبك وسيرتم معالجته خلال 24 ساعة. يرجى التواصل مع الدعم الفني إذا لم يظهر المبلغ.` |
| **English UI Message** | `Refund processing failed. Your request has been logged and will be processed within 24 hours. Contact support if the amount doesn't appear.` |
| **Recovery** | Refund queued for manual processing. Support ticket auto-created. Check wallet in 24 hours. |
| **Severity** | High |
| **Visual Treatment** | Toast notification with info icon. Support ticket reference number displayed. |
| **Accessibility Announcement** | `تعذر معالجة الاسترداد. تم تسجيل الطلب وسيرتم المعالجة خلال 24 ساعة.` (assertive) |
| **Analytics Event** | `error_pay_refund_failed` with `refund_amount` and `order_id` properties |
| **Log Level** | ERROR |

---

## 9. Order Errors

### ER-ORD-001: Product Out of Stock

| Field | Value |
|-------|-------|
| **Error ID** | ER-ORD-001 |
| **Trigger** | Product becomes unavailable after adding to cart or during checkout |
| **Arabic UI Message** | `المنتج "{product_name}" لم يعد متوفراً. يرجى إزالته من السلة أو اختيار بديل.` |
| **English UI Message** | `Product "{product_name}" is no longer available. Please remove it from your cart or choose an alternative.` |
| **Recovery** | Remove product from cart. Browse similar products. Vendor may restock. |
| **Severity** | High |
| **Visual Treatment** | Inline cart error with product card showing "غير متوفر" / "Unavailable" badge. Suggest alternatives. |
| **Accessibility Announcement** | `المنتج {product_name} غير متوفر.` (polite) |
| **Analytics Event** | `error_ord_out_of_stock` with `product_id` and `vendor_id` properties |
| **Log Level** | INFO |

### ER-ORD-002: Invalid Order State Transition

| Field | Value |
|-------|-------|
| **Error ID** | ER-ORD-002 |
| **Trigger** | Attempted order state change violates the 17-state order lifecycle |
| **Arabic UI Message** | `لا يمكن تغيير حالة الطلب من "{current_state}" إلى "{target_state}".` |
| **English UI Message** | `Cannot change order status from "{current_state}" to "{target_state}".` |
| **Recovery** | Refresh order details to see current valid actions. Contact support if state appears incorrect. |
| **Severity** | Medium |
| **Visual Treatment** | Toast notification. Invalid action button disabled with tooltip explaining valid transitions. |
| **Accessibility Announcement** | `لا يمكن تغيير حالة الطلب. يرجى تحديث الصفحة.` (polite) |
| **Analytics Event** | `error_ord_invalid_transition` with `current_state` and `target_state` properties |
| **Log Level** | WARN |

### ER-ORD-003: Order Cancellation Not Allowed

| Field | Value |
|-------|-------|
| **Error ID** | ER-ORD-003 |
| **Trigger** | Customer tries to cancel order that has already been shipped or is in a non-cancellable state |
| **Arabic UI Message** | `لا يمكن إلغاء الطلب في هذه المرحلة. يمكنك استلام الطلب ثم طلب الإرجاع.` |
| **English UI Message** | `Cannot cancel the order at this stage. You can receive the order and then request a return.` |
| **Recovery** | Wait for delivery. Initiate return process within return window. |
| **Severity** | Medium |
| **Visual Treatment** | Toast with info icon. Show "طلب الإرجاع" / "Request Return" option if delivery is imminent. |
| **Accessibility Announcement** | `لا يمكن إلغاء الطلب. يمكنك طلب الإرجاع بعد الاستلام.` (polite) |
| **Analytics Event** | `error_ord_cancel_not_allowed` with `current_order_state` property |
| **Log Level** | INFO |

### ER-ORD-004: Return Window Expired

| Field | Value |
|-------|-------|
| **Error ID** | ER-ORD-004 |
| **Trigger** | Return request submitted after the return window has expired (typically 3-7 days depending on category) |
| **Arabic UI Message** | `انتهت مهلة الإرجاع لهذا الطلب. ينتهي حقك في الإرجاع بعد {days} أيام من الاستلام.` |
| **English UI Message** | `Return window has expired for this order. Return rights expire {days} days after delivery.` |
| **Recovery** | Contact vendor directly for goodwill return. Contact support for dispute. |
| **Severity** | Medium |
| **Visual Treatment** | Modal with calendar icon. Show original delivery date and return deadline. |
| **Accessibility Announcement** | `انتهت مهلة الإرجاع لهذا الطلب.` (polite) |
| **Analytics Event** | `error_ord_return_expired` with `delivery_date` and `return_deadline` properties |
| **Log Level** | INFO |

### ER-ORD-005: Maximum Items Exceeded

| Field | Value |
|-------|-------|
| **Error ID** | ER-ORD-005 |
| **Trigger** | Cart contains more than 50 unique items (platform constraint) |
| **Arabic UI Message** | `تم الوصول للحد الأقصى من المنتجات (50 منتج). يرجى إزالة بعض المنتجات قبل الإضافة.` |
| **English UI Message** | `Cart limit reached (50 items max). Please remove some items before adding more.` |
| **Recovery** | Remove items from cart. Split into multiple orders. |
| **Severity** | Medium |
| **Visual Treatment** | Toast notification. Add-to-cart button disabled. Show current count: `{count}/50`. |
| **Accessibility Announcement** | `تم الوصول للحد الأقصى من المنتجات. الحد هو 50 منتج.` (polite) |
| **Analytics Event** | `error_ord_max_items` with `current_count` property |
| **Log Level** | INFO |

### ER-ORD-006: Maximum Quantity Exceeded

| Field | Value |
|-------|-------|
| **Error ID** | ER-ORD-006 |
| **Trigger** | Single product quantity exceeds 10 units (platform constraint) |
| **Arabic UI Message** | `الحد الأقصى لكمية منتج واحد هو 10 وحدات.` |
| **English UI Message** | `Maximum quantity for a single product is 10 units.` |
| **Recovery** | Reduce quantity to 10 or less. Place bulk order via vendor contact. |
| **Severity** | Medium |
| **Visual Treatment** | Inline error on quantity input. Quantity stepper maxes at 10. |
| **Accessibility Announcement** | `الحد الأقصى هو 10 وحدات لكل منتج.` (polite) |
| **Analytics Event** | `error_ord_max_quantity` with `product_id` and `attempted_quantity` properties |
| **Log Level** | INFO |

### ER-ORD-007: Minimum Order Amount

| Field | Value |
|-------|-------|
| **Error ID** | ER-ORD-007 |
| **Trigger** | Order total is below 500 YER minimum (platform constraint) |
| **Arabic UI Message** | `الحد الأدنى للطلب هو 500 يمني. المبلغ الحالي: {amount} يمني. يرجى إضافة منتجات إضافية.` |
| **English UI Message** | `Minimum order amount is 500 YER. Current total: {amount} YER. Please add more items.` |
| **Recovery** | Add more products. Browse recommended items to reach minimum. |
| **Severity** | Medium |
| **Visual Treatment** | Inline warning in checkout. Progress bar showing how much more is needed. Suggested products below. |
| **Accessibility Announcement** | `الحد الأدنى للطلب 500 يمني. المبلغ الحالي {amount} يمني.` (polite) |
| **Analytics Event** | `error_ord_min_amount` with `current_total` and `minimum_required` properties |
| **Log Level** | INFO |

### ER-ORD-008: Maximum Order Amount

| Field | Value |
|-------|-------|
| **Error ID** | ER-ORD-008 |
| **Trigger** | Order total exceeds 5,000,000 YER maximum (platform constraint) |
| **Arabic UI Message** | `تم تجاوز الحد الأقصى للطلب (5,000,000 يمني). المبلغ الحالي: {amount} يمني. يرجى تقسيم الطلب.` |
| **English UI Message** | `Order exceeds maximum amount (5,000,000 YER). Current: {amount} YER. Please split into multiple orders.` |
| **Recovery** | Split order into smaller orders. Remove high-value items. Contact vendor for bulk arrangement. |
| **Severity** | High |
| **Visual Treatment** | Modal dialog with split order suggestion. Auto-calculate suggested splits. |
| **Accessibility Announcement** | `تم تجاوز الحد الأقصى للطلب. يرجى تقسيم الطلب.` (assertive) |
| **Analytics Event** | `error_ord_max_amount` with `current_total` and `maximum_allowed` properties |
| **Log Level** | WARN |

---

## 10. Delivery Errors

### ER-DEL-001: Delivery Code Incorrect

| Field | Value |
|-------|-------|
| **Error ID** | ER-DEL-001 |
| **Trigger** | Customer enters incorrect delivery confirmation code. Shows remaining attempts: `{count}/3` |
| **Arabic UI Message** | `رمز التسليم غير صحيح. متبقي {count} محاولة من أصل 3.` |
| **English UI Message** | `Incorrect delivery code. {count} attempts remaining out of 3.` |
| **Recovery** | Re-enter code from SMS. If locked, contact vendor or support for manual confirmation. |
| **Severity** | High |
| **Visual Treatment** | Inline error with remaining attempts counter. Input field shakes. Red border. |
| **Accessibility Announcement** | `رمز التسليم غير صحيح. متبقي {count} محاولات.` (assertive) |
| **Analytics Event** | `error_del_code_incorrect` with `attempt_number` and `remaining` properties |
| **Log Level** | WARN |

### ER-DEL-002: Delivery Code Expired

| Field | Value |
|-------|-------|
| **Error ID** | ER-DEL-002 |
| **Trigger** | Delivery code entered after expiration (typically 24-hour validity) |
| **Arabic UI Message** | `رمز التسليم منتهي الصلاحية. يرجى طلب رمز جديد من البائع.` |
| **English UI Message** | `Delivery code has expired. Please request a new code from the vendor.` |
| **Recovery** | Request new code from vendor. Contact support if vendor is unresponsive. |
| **Severity** | Medium |
| **Visual Treatment** | Toast notification. "طلب رمز جديد" / "Request New Code" button. |
| **Accessibility Announcement** | `انتهت صلاحية رمز التسليم. يرجى طلب رمز جديد.` (polite) |
| **Analytics Event** | `error_del_code_expired` with `code_age_hours` property |
| **Log Level** | INFO |

### ER-DEL-003: Delivery Area Not Supported

| Field | Value |
|-------|-------|
| **Error ID** | ER-DEL-003 |
| **Trigger** | Delivery address is outside the supported delivery areas for the vendor or platform |
| **Arabic UI Message** | `التوصيل غير متوفر لهذا العنوان. يرجى اختيار عنوان آخر أو التواصل مع البائع.` |
| **English UI Message** | `Delivery is not available for this address. Please choose another address or contact the vendor.` |
| **Recovery** | Change delivery address. Contact vendor for alternative arrangement. Pick up from vendor location. |
| **Severity** | Medium |
| **Visual Treatment** | Map pin with red circle showing unsupported area. Address selector modal. |
| **Accessibility Announcement** | `التوصيل غير متوفر لهذا العنوان.` (polite) |
| **Analytics Event** | `error_del_area_unsupported` with `area_code` and `vendor_id` properties |
| **Log Level** | INFO |

### ER-DEL-004: Driver Lockout

| Field | Value |
|-------|-------|
| **Error ID** | ER-DEL-004 |
| **Trigger** | Delivery driver locked out after 3 failed delivery code attempts. Locked for 24 hours. |
| **Arabic UI Message** | `تم حظر حساب السائق لمدة 24 ساعة بسبب محاولات تسليم فاشلة متعددة. يرجى التواصل مع الإدارة.` |
| **English UI Message** | `Driver account locked for 24 hours due to multiple failed delivery attempts. Please contact administration.` |
| **Recovery** | Driver cannot access delivery features for 24 hours. Admin can manually unlock. |
| **Severity** | Critical |
| **Visual Treatment** | Full-screen modal with lock icon. Countdown timer. Admin contact displayed. |
| **Accessibility Announcement** | `تم حظر حساب السائق لمدة 24 ساعة.` (assertive) |
| **Analytics Event** | `error_del_driver_lockout` with `driver_id` and `lockout_duration` properties |
| **Log Level** | ERROR (triggers security alert) |

---

## 11. Product Errors

### ER-PRD-001: Product Not Found

| Field | Value |
|-------|-------|
| **Error ID** | ER-PRD-001 |
| **Trigger** | Requested product does not exist, has been removed, or URL is invalid |
| **Arabic UI Message** | `المنتج غير موجود. ربما تم حذفه أو لم يعد متاحاً.` |
| **English UI Message** | `Product not found. It may have been removed or is no longer available.` |
| **Recovery** | Browse similar products. Use search. Return to category page. |
| **Severity** | Low |
| **Visual Treatment** | Product card with "غير متاح" / "Unavailable" badge. Suggested similar products below. |
| **Accessibility Announcement** | `المنتج غير موجود.` (polite) |
| **Analytics Event** | `error_prd_not_found` with `product_id` and `referrer` properties |
| **Log Level** | INFO |

### ER-PRD-002: Product Variation Unavailable

| Field | Value |
|-------|-------|
| **Error ID** | ER-PRD-002 |
| **Trigger** | Selected product variation (size, color, etc.) is out of stock |
| **Arabic UI Message** | `الخيار المحدد "{variation}" غير متوفر. يرجى اختيار خيار آخر.` |
| **English UI Message** | `Selected option "{variation}" is unavailable. Please choose another option.` |
| **Recovery** | Select different variation. Check availability of other options. |
| **Severity** | Medium |
| **Visual Treatment** | Inline error on variation selector. Unavailable options grayed out with strikethrough. |
| **Accessibility Announcement** | `الخيار {variation} غير متوفر. يرجى اختيار خيار آخر.` (polite) |
| **Analytics Event** | `error_prd_variation_unavailable` with `product_id` and `variation_id` properties |
| **Log Level** | INFO |

### ER-PRD-003: Image Upload Failed

| Field | Value |
|-------|-------|
| **Error ID** | ER-PRD-003 |
| **Trigger** | Product image upload fails (file too large >5MB, invalid format, network error, server rejection) |
| **Arabic UI Message** | `فشل رفع الصورة. يرجى التحقق من الحجم (أقل من 5 ميجا) والصيغة (JPG, PNG) والمحاولة مرة أخرى.` |
| **English UI Message** | `Image upload failed. Please check size (under 5MB) and format (JPG, PNG) and try again.` |
| **Recovery** | Compress image. Convert to supported format. Retry upload. |
| **Severity** | Medium |
| **Visual Treatment** | Inline error on image upload component. Show file requirements. Retry button on failed image. |
| **Accessibility Announcement** | `فشل رفع الصورة. يرجى التحقق من الحجم والصيغة.` (polite) |
| **Analytics Event** | `error_prd_image_upload` with `file_size` and `file_type` properties |
| **Log Level** | INFO |

### ER-PRD-004: Product Moderation Rejected

| Field | Value |
|-------|-------|
| **Error ID** | ER-PRD-004 |
| **Trigger** | Product listing rejected by admin moderation (policy violation, inappropriate content, misleading info) |
| **Arabic UI Message** | `تم رفض المنتج "{product_name}" أثناء المراجعة. السبب: {reason}. يرجى تعديل المنتج وإعادة الإرسال.` |
| **English UI Message** | `Product "{product_name}" was rejected during moderation. Reason: {reason}. Please edit and resubmit.` |
| **Recovery** | Review rejection reason. Edit product details. Resubmit for moderation. |
| **Severity** | Medium |
| **Visual Treatment** | Notification banner on vendor dashboard. Product card shows "مرفوض" / "Rejected" badge. Edit button prominent. |
| **Accessibility Announcement** | `تم رفض المنتج. السبب: {reason}. يرجى التعديل وإعادة الإرسال.` (polite) |
| **Analytics Event** | `error_prd_moderation_rejected` with `product_id` and `rejection_reason` properties |
| **Log Level** | INFO |

---

## 12. Network/Connectivity Errors

### ER-NET-001: Network Disconnected

| Field | Value |
|-------|-------|
| **Error ID** | ER-NET-001 |
| **Trigger** | Device loses internet connection (WiFi/4G/3G drop) |
| **Arabic UI Message** | `لا يوجد اتصال بالإنترنت. بعض الميزات قد لا تعمل حتى عودة الاتصال.` |
| **English UI Message** | `No internet connection. Some features may not work until connection is restored.` |
| **Recovery** | Check device settings. Move to area with better signal. Enable WiFi. |
| **Severity** | High |
| **Visual Treatment** | Persistent top banner (red/orange) with connection status icon. Dismissable but reappears. |
| **Accessibility Announcement** | `لا يوجد اتصال بالإنترنت.` (assertive) |
| **Analytics Event** | `error_net_disconnected` with `last_online` and `network_type` properties |
| **Log Level** | INFO |

### ER-NET-002: Request Timeout

| Field | Value |
|-------|-------|
| **Error ID** | ER-NET-002 |
| **Trigger** | HTTP request exceeds timeout threshold (30 seconds for standard, 60 seconds for file uploads) |
| **Arabic UI Message** | `انتهت مهلة الطلب. يرجى التحقق من اتصالك بالإنترنت والمحاولة مرة أخرى.` |
| **English UI Message** | `Request timed out. Please check your internet connection and try again.` |
| **Recovery** | Check connection. Retry the action. If persists, switch network. |
| **Severity** | Medium |
| **Visual Treatment** | Toast notification with retry button. Loading state replaced with timeout message. |
| **Accessibility Announcement** | `انتهت مهلة الطلب. يرجى المحاولة مرة أخرى.` (polite) |
| **Analytics Event** | `error_net_timeout` with `timeout_duration` and `endpoint` properties |
| **Log Level** | WARN |

### ER-NET-003: Server Unreachable

| Field | Value |
|-------|-------|
| **Error ID** | ER-NET-003 |
| **Trigger** | DNS resolution fails or server IP is unreachable |
| **Arabic UI Message** | `تعذر الاتصال بالخادم. يرجى التحقق من اتصالك بالإنترنت أو المحاولة لاحقاً.` |
| **English UI Message** | `Cannot reach the server. Please check your internet connection or try again later.` |
| **Recovery** | Check DNS settings. Try different network. Wait and retry. |
| **Severity** | High |
| **Visual Treatment** | Full-page error with connection illustration. Auto-retry every 30 seconds with status indicator. |
| **Accessibility Announcement** | `تعذر الاتصال بالخادم.` (assertive) |
| **Analytics Event** | `error_net_unreachable` with `target_host` and `dns_resolution_time` properties |
| **Log Level** | ERROR |

### ER-NET-004: Slow Connection

| Field | Value |
|-------|-------|
| **Error ID** | ER-NET-004 |
| **Trigger** | Connection speed below threshold (<100kbps) or high latency (>3000ms) |
| **Arabic UI Message** | `اتصالك بطيء. قد تستغرق بعض العمليات وقتاً أطول. يرجى التحول إلى اتصال أسرع إن أمكن.` |
| **English UI Message** | `Your connection is slow. Some operations may take longer. Please switch to a faster connection if possible.` |
| **Recovery** | Switch to WiFi. Move to better signal area. Close other bandwidth-heavy apps. |
| **Severity** | Low |
| **Visual Treatment** | Subtle toast notification. Loading states show extended progress bars. No blocking UI. |
| **Accessibility Announcement** | `اتصالك بطيء. قد يستغرق التنفيذ وقتاً أطول.` (polite) |
| **Analytics Event** | `error_net_slow` with `latency_ms` and `bandwidth_kbps` properties |
| **Log Level** | INFO |

---

## 13. Validation Errors

### ER-VAL-001: Required Field Missing

| Field | Value |
|-------|-------|
| **Error ID** | ER-VAL-001 |
| **Trigger** | User submits form with required fields left empty |
| **Arabic UI Message** | `الحقل "{field_name}" مطلوب. يرجى إدخال قيمة.` |
| **English UI Message** | `Field "{field_name}" is required. Please enter a value.` |
| **Recovery** | Fill in the required field. Form cannot be submitted until all required fields are completed. |
| **Severity** | Medium |
| **Visual Treatment** | Inline field error with red border and asterisk. Summary banner at top of form listing all missing fields. |
| **Accessibility Announcement** | `الحقل {field_name} مطلوب.` (polite) |
| **Analytics Event** | `error_val_required` with `field_name` and `form_id` properties |
| **Log Level** | INFO |

### ER-VAL-002: Invalid Format

| Field | Value |
|-------|-------|
| **Error ID** | ER-VAL-002 |
| **Trigger** | Input does not match expected format (email, phone, price, date) |
| **Arabic UI Message** | `الحقل "{field_name}" بتنسيق غير صحيح. يرجى اتباع الصيغة المطلوبة.` |
| **English UI Message** | `Field "{field_name}" has an invalid format. Please follow the required format.` |
| **Recovery** | Correct format according to hint. Show expected format example below field. |
| **Severity** | Medium |
| **Visual Treatment** | Inline field error with format hint displayed. Expected format shown in gray below field. |
| **Accessibility Announcement** | `تنسيق الحقل {field_name} غير صحيح.` (polite) |
| **Analytics Event** | `error_val_format` with `field_name` and `expected_format` properties |
| **Log Level** | INFO |

### ER-VAL-003: Field Too Long

| Field | Value |
|-------|-------|
| **Error ID** | ER-VAL-003 |
| **Trigger** | Input exceeds maximum character/length limit |
| **Arabic UI Message** | `الحقل "{field_name}" طويل جداً. الحد الأقصى {max} حرف. المدخل: {current} حرف.` |
| **English UI Message** | `Field "{field_name}" is too long. Maximum {max} characters. Entered: {current} characters.` |
| **Recovery** | Shorten input to within limit. Character counter shown below field. |
| **Severity** | Medium |
| **Visual Treatment** | Inline field error with character counter in red. Counter shows `{current}/{max}`. |
| **Accessibility Announcement** | `الحقل {field_name} طويل جداً. الحد الأقصى {max} حرف.` (polite) |
| **Analytics Event** | `error_val_too_long` with `field_name` and `excess_chars` properties |
| **Log Level** | INFO |

### ER-VAL-004: Field Too Short

| Field | Value |
|-------|-------|
| **Error ID** | ER-VAL-004 |
| **Trigger** | Input is below minimum character/length requirement |
| **Arabic UI Message** | `الحقل "{field_name}" قصير جداً. الحد الأدنى {min} حرف. المدخل: {current} حرف.` |
| **English UI Message** | `Field "{field_name}" is too short. Minimum {min} characters. Entered: {current} characters.` |
| **Recovery** | Add more characters to meet minimum. Character counter shown below field. |
| **Severity** | Medium |
| **Visual Treatment** | Inline field error with character counter in red. Counter shows `{current}/{min}`. |
| **Accessibility Announcement** | `الحقل {field_name} قصير جداً. الحد الأدنى {min} حرف.` (polite) |
| **Analytics Event** | `error_val_too_short` with `field_name` and `deficit_chars` properties |
| **Log Level** | INFO |

### ER-VAL-005: Cross-Field Validation

| Field | Value |
|-------|-------|
| **Error ID** | ER-VAL-005 |
| **Trigger** | Fields are individually valid but conflict with each other (e.g., delivery date before order date, price range min > max) |
| **Arabic UI Message** | `تعارض في البيانات: "{field_1}" لا يتوافق مع "{field_2}". يرجى مراجعة القيمتين.` |
| **English UI Message** | `Data conflict: "{field_1}" does not match "{field_2}". Please review both values.` |
| **Recovery** | Review both fields. Adjust values to be consistent with each other. |
| **Severity** | Medium |
| **Visual Treatment** | Both conflicting fields highlighted with connecting line/arrow. Summary error at top of form. |
| **Accessibility Announcement** | `تعارض في البيانات بين {field_1} و {field_2}. يرجى المراجعة.` (polite) |
| **Analytics Event** | `error_val_cross_field` with `field_1` and `field_2` properties |
| **Log Level** | INFO |

---

## 14. Empty States

### ER-EMP-001: No Products Found

| Field | Value |
|-------|-------|
| **Error ID** | ER-EMP-001 |
| **Trigger** | Category page or filtered view has no products matching criteria |
| **Arabic UI Message** | `لا توجد منتجات في هذا القسم حالياً. يرجى تصفح أقسام أخرى أو التحقق لاحقاً.` |
| **English UI Message** | `No products in this category right now. Please browse other categories or check back later.` |
| **Recovery** | Browse other categories. Clear filters. Check back later. |
| **Severity** | Low |
| **Visual Treatment** | Illustration (empty shelf). Category suggestions. Search bar prominent. |
| **Accessibility Announcement** | `لا توجد منتجات في هذا القسم.` (polite) |
| **Analytics Event** | `empty_products` with `category_id` and `filter_params` properties |
| **Log Level** | INFO |

### ER-EMP-002: No Search Results

| Field | Value |
|-------|-------|
| **Error ID** | ER-EMP-002 |
| **Trigger** | Search query returns zero results |
| **Arabic UI Message** | `لا توجد نتائج لـ "{query}". جرّب كلمات بحث مختلفة أو تصفح الفئات.` |
| **English UI Message** | `No results for "{query}". Try different keywords or browse categories.` |
| **Recovery** | Modify search term. Check spelling. Browse categories. Use popular searches. |
| **Severity** | Low |
| **Visual Treatment** | Illustration (magnifying glass). Popular search suggestions. Category quick links. |
| **Accessibility Announcement** | `لا توجد نتائج للبحث عن {query}.` (polite) |
| **Analytics Event** | `empty_search` with `query` and `result_count` properties |
| **Log Level** | INFO |

### ER-EMP-003: No Orders

| Field | Value |
|-------|-------|
| **Error ID** | ER-EMP-003 |
| **Trigger** | Customer opens order history with no previous orders |
| **Arabic UI Message** | `لا توجد طلبات بعد. ابدأ التسوق الآن!` |
| **English UI Message** | `No orders yet. Start shopping now!` |
| **Recovery** | Browse products. View deals and recommendations. |
| **Severity** | Low |
| **Visual Treatment** | Illustration (empty package). "ابدأ التسوق" / "Start Shopping" primary CTA. Featured products below. |
| **Accessibility Announcement** | `لا توجد طلبات بعد. ابدأ التسوق الآن!` (polite) |
| **Analytics Event** | `empty_orders` with `user_segment` property |
| **Log Level** | INFO |

### ER-EMP-004: No Notifications

| Field | Value |
|-------|-------|
| **Error ID** | ER-EMP-004 |
| **Trigger** | User opens notification center with no notifications |
| **Arabic UI Message** | `لا توجد إشعارات جديدة.` |
| **English UI Message** | `No new notifications.` |
| **Recovery** | No action needed. Notifications will appear as events occur. |
| **Severity** | Low |
| **Visual Treatment** | Illustration (bell icon). Minimal text. No CTA needed. |
| **Accessibility Announcement** | `لا توجد إشعارات جديدة.` (polite) |
| **Analytics Event** | `empty_notifications` property |
| **Log Level** | INFO |

### ER-EMP-005: Empty Cart

| Field | Value |
|-------|-------|
| **Error ID** | ER-EMP-005 |
| **Trigger** | Customer opens cart with no items |
| **Arabic UI Message** | `سلتك فارغة. اضف منتجات للبدء!` |
| **English UI Message** | `Your cart is empty. Add items to get started!` |
| **Recovery** | Browse products. View recommendations based on browsing history. |
| **Severity** | Low |
| **Visual Treatment** | Illustration (empty cart). "تسوّق الآن" / "Shop Now" CTA. Recently viewed products below. |
| **Accessibility Announcement** | `سلتك فارغة. اضف منتجات للبدء!` (polite) |
| **Analytics Event** | `empty_cart` with `referrer` property |
| **Log Level** | INFO |

### ER-EMP-006: No Transactions

| Field | Value |
|-------|-------|
| **Error ID** | ER-EMP-006 |
| **Trigger** | Wallet transaction history is empty |
| **Arabic UI Message** | `لا توجد معاملات في سجل المحفظة.` |
| **English UI Message** | `No transactions in wallet history.` |
| **Recovery** | Top up wallet to see transactions. Transactions appear after first use. |
| **Severity** | Low |
| **Visual Treatment** | Illustration (wallet icon). "شحن المحفظة" / "Top Up Wallet" CTA. |
| **Accessibility Announcement** | `لا توجد معاملات في سجل المحفظة.` (polite) |
| **Analytics Event** | `empty_transactions` property |
| **Log Level** | INFO |

### ER-EMP-007: No Reviews

| Field | Value |
|-------|-------|
| **Error ID** | ER-EMP-007 |
| **Trigger** | Product has no customer reviews yet |
| **Arabic UI Message** | `لا توجد مراجعات لهذا المنتج بعد. كن أول من يقيّم!` |
| **English UI Message** | `No reviews for this product yet. Be the first to review!` |
| **Recovery** | View product details. Purchase and review after delivery. |
| **Severity** | Low |
| **Visual Treatment** | Illustration (star icons). "اكتب مراجعة" / "Write a Review" CTA (if user has purchased). |
| **Accessibility Announcement** | `لا توجد مراجعات. كن أول من يقيّم!` (polite) |
| **Analytics Event** | `empty_reviews` with `product_id` property |
| **Log Level** | INFO |

### ER-EMP-008: No Addresses

| Field | Value |
|-------|-------|
| **Error ID** | ER-EMP-008 |
| **Trigger** | Customer opens address book with no saved addresses |
| **Arabic UI Message** | `لا توجد عناوين محفوظة. أضف عنوانك الأول!` |
| **English UI Message** | `No saved addresses. Add your first address!` |
| **Recovery** | Add new address. Use current location. |
| **Severity** | Low |
| **Visual Treatment** | Illustration (map pin). "إضافة عنوان" / "Add Address" primary CTA. Location permission prompt. |
| **Accessibility Announcement** | `لا توجد عناوين محفوظة. أضف عنوانك الأول!` (polite) |
| **Analytics Event** | `empty_addresses` with `user_segment` property |
| **Log Level** | INFO |

---

## 15. Visual Treatment Reference

### 15.1 Toast Notification

| Property | Value |
|----------|-------|
| **Position** | Top-right (RTL: top-left) |
| **Max width** | 400px |
| **Auto-dismiss** | 5 seconds (Low/Medium), 8 seconds (High) |
| **Stacking** | Max 3 visible. New pushes oldest off. |
| **Animation** | Slide in from right (RTL: left), fade out |
| **Z-index** | 9999 |
| **Icon** | Left of text (RTL: right) — error: red X, warning: orange !, info: blue i |

```
┌──────────────────────────────────────────┐
│ ⚠️  رمز التحقق غير صحيح. متبقي 2 محاولة.  │
└──────────────────────────────────────────┘
```

### 15.2 Modal Dialog

| Property | Value |
|----------|-------|
| **Position** | Center of viewport |
| **Max width** | 480px |
| **Overlay** | Semi-transparent black (#00000080) |
| **Dismissable** | Critical: No. High/Medium: Yes (X button + overlay click) |
| **Animation** | Scale from 0.9 to 1.0 + fade in |
| **Z-index** | 10000 |
| **Focus trap** | Yes — Tab cycles within modal only |

```
┌──────────────────────────────────────┐
│            ⚠️  خطأ في الدفع          │
│                                      │
│  رصيد المحفظة غير كافٍ.              │
│  الرصيد الحالي: 2,500 يمني           │
│  المطلوب: 4,200 يمني                 │
│                                      │
│  ┌──────────────┐  ┌──────────────┐  │
│  │  شحن المحفظة  │  │    إلغاء     │  │
│  └──────────────┘  └──────────────┘  │
└──────────────────────────────────────┘
```

### 15.3 Inline Error

| Property | Value |
|----------|-------|
| **Position** | Below the field |
| **Color** | Text: #DC3545 (red). Border: #DC3545. Background: #FFF5F5. |
| **Icon** | Small red exclamation circle |
| **Animation** | Slide down from field + fade in |
| **Persistence** | Until field is corrected or form is reset |

```
┌──────────────────────────────────────┐
│  رقم الهاتف                           │
│  ┌──────────────────────────────────┐│
│  │ +967 712 345                     ││
│  └──────────────────────────────────┘│
│  ⚠️ رقم الهاتف غير صحيح. يرجى إدخال  │
│     رقم يمني يبدأ بـ 967.            │
└──────────────────────────────────────┘
```

### 15.4 Banner Error

| Property | Value |
|----------|-------|
| **Position** | Top of page, below header |
| **Full width** | Yes |
| **Color** | Background: #FFF3CD (warning) or #F8D7DA (error). Border-left: 4px solid. |
| **Dismissable** | Yes (X button) |
| **Auto-dismiss** | 8 seconds (High), persistent (Critical) |

```
┌──────────────────────────────────────────────────────┐
│ ⚠️ لا يوجد اتصال بالإنترنت. بعض الميزات قد لا تعمل.  │
│                                              [×]     │
└──────────────────────────────────────────────────────┘
```

### 15.5 Full-Page Error

| Property | Value |
|----------|-------|
| **Layout** | Centered content, max 500px width |
| **Illustration** | Custom SVG, 200x200px, above title |
| **Title** | 24px, bold, Arabic primary |
| **Message** | 16px, regular, Arabic secondary |
| **Actions** | Primary button (blue) + secondary link |
| **Support** | Phone number and chat link always visible |

```
┌──────────────────────────────────────┐
│                                      │
│            [Illustration]            │
│                                      │
│        حدث خطأ غير متوقع             │
│                                      │
│  نعتذر عن الإزعاج. يرجى المحاولة    │
│  مرة أخرى أو التواصل مع الدعم الفني. │
│                                      │
│        ┌──────────────────┐          │
│        │  إعادة المحاولة   │          │
│        └──────────────────┘          │
│                                      │
│     العودة للرئيسية | الدعم الفني    │
│                                      │
│     مرجع الخطأ: ERR-20260913-A3F2   │
└──────────────────────────────────────┘
```

---

## 16. Accessibility Reference

### 16.1 ARIA Live Regions

| Error Severity | ARIA Attribute | Behavior |
|----------------|----------------|----------|
| Critical | `aria-live="assertive"` | Announces immediately, interrupts current announcement |
| High | `aria-live="assertive"` | Announces immediately |
| Medium | `aria-live="polite"` | Announces after current announcement completes |
| Low | `aria-live="polite"` | Announces after current announcement completes |

### 16.2 Focus Management

| Error Type | Focus Behavior |
|------------|----------------|
| Modal error | Focus moves to modal. Trapped within modal until dismissed. |
| Toast error | No focus change. Screen reader announces via live region. |
| Inline error | Focus remains on the field. Error announced via live region. |
| Full-page error | Focus moves to the error title (h1/h2). |
| Auth redirect | Focus moves to login form on redirect completion. |

### 16.3 Keyboard Navigation

| Key | Behavior |
|-----|----------|
| `Escape` | Dismiss modal/toast errors |
| `Tab` | Cycle through interactive elements in error UI |
| `Enter` | Activate primary action button |
| `Space` | Activate buttons |
| `Arrow keys` | Navigate within radio groups or option lists |

### 16.4 Screen Reader Announcements

| Error | Announcement (Arabic) |
|-------|----------------------|
| ER-AUTH-003 | `رمز غير صحيح. متبقي محاولتان.` |
| ER-PAY-001 | `خطأ في الدفع. رصيد المحفظة غير كافٍ.` |
| ER-ORD-001 | `تنبيه. المنتج لم يعد متوفراً.` |
| ER-DEL-001 | `خطأ في التسليم. رمز غير صحيح.` |
| ER-NET-001 | `خطأ في الاتصال. لا يوجد اتصال بالإنترنت.` |

### 16.5 Color and Contrast

| Element | Minimum Contrast Ratio |
|---------|----------------------|
| Error text on white background | 4.5:1 (WCAG AA) |
| Error icon on white background | 3:1 (WCAG AA) |
| Error border on white background | 3:1 (WCAG AA) |
| Error background tint | Sufficient contrast with text |
| Toast notification text | 4.5:1 on toast background |
| Banner text | 4.5:1 on banner background |

### 16.6 Reduced Motion

| Animation | Fallback |
|-----------|----------|
| Toast slide-in | Instant appear (no slide) |
| Modal scale | Instant appear (no scale) |
| Shake animation | No animation, border color change only |
| Loading spinner | Static progress text |
| Banner slide-down | Instant appear |

---

## Appendix A: Error Code Quick Reference

| Category | Prefix | Count | Examples |
|----------|--------|-------|----------|
| HTTP | ER-HTTP | 9 | 400, 401, 403, 404, 409, 422, 429, 500, 503 |
| Authentication | ER-AUTH | 9 | Invalid phone, OTP expired, account locked |
| Payment | ER-PAY | 7 | Insufficient balance, top-up failed, timeout |
| Order | ER-ORD | 8 | Out of stock, invalid transition, min/max amounts |
| Delivery | ER-DEL | 4 | Code incorrect, area unsupported, driver lockout |
| Product | ER-PRD | 4 | Not found, variation unavailable, moderation rejected |
| Network | ER-NET | 4 | Disconnected, timeout, unreachable, slow |
| Validation | ER-VAL | 5 | Required, format, length, cross-field |
| Empty State | ER-EMP | 8 | No products, no results, no orders, etc. |
| **Total** | | **58** | |

---

## Appendix B: Severity Distribution

| Severity | Count | Percentage |
|----------|-------|------------|
| Critical | 8 | 13.8% |
| High | 14 | 24.1% |
| Medium | 26 | 44.8% |
| Low | 10 | 17.2% |
| **Total** | **58** | **100%** |

---

## Appendix C: Platform Constraints Reference

| # | Constraint | Related Error IDs |
|---|------------|-------------------|
| 1 | Arabic-first, RTL design | All errors |
| 2 | Wallet-only payments | ER-PAY-001 through ER-PAY-007 |
| 3 | SMS-only authentication | ER-AUTH-001 through ER-AUTH-009 |
| 4 | 17 order states | ER-ORD-002, ER-ORD-003 |
| 5 | 5 actors | Portal-specific error isolation |
| 6 | Max 50 items per cart | ER-ORD-005 |
| 7 | Max 10 quantity per product | ER-ORD-006 |
| 8 | Min order 500 YER | ER-ORD-007 |
| 9 | Max order 5,000,000 YER | ER-ORD-008 |
| 10 | OTP: 3 attempts before lock | ER-AUTH-003, ER-AUTH-004 |
| 11 | Delivery code: 3 attempts | ER-DEL-001, ER-DEL-004 |
| 12 | Driver lockout: 24 hours | ER-DEL-004 |
| 13 | Account lockout: 15 minutes | ER-AUTH-004 |
| 14 | OTP validity: 5 minutes | ER-AUTH-002 |
| 15 | Delivery code validity: 24 hours | ER-DEL-002 |

---

*End of Error States & Edge Cases Document*
