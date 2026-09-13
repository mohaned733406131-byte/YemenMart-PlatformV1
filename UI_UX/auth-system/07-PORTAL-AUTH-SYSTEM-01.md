# 07 — YemenMart Authentication System — UI/UX Specification

| Field | Value |
|---|---|
| **Module** | Portal Authentication System |
| **Version** | 1.0 |
| **Date** | 2026-09-13 |
| **Status** | Draft |
| **Design Language** | YemenMart Design System (YMDS) |
| **Direction** | RTL-first, Arabic-primary |
| **Auth Model** | SMS OTP → Password (hybrid) |

---

## Table of Contents

1. [Global Conventions](#1-global-conventions)
2. [AUTH-LO-001 — Login Page (Phone Entry)](#2-auth-lo-001--login-page-phone-entry)
3. [AUTH-LO-002 — OTP Verification Page (Login)](#3-auth-lo-002--otp-verification-page-login)
4. [AUTH-RE-001 — Registration Page (Phone Entry)](#4-auth-re-001--registration-page-phone-entry)
5. [AUTH-RE-002 — Registration OTP Verification](#5-auth-re-002--registration-otp-verification)
6. [AUTH-RE-003 — Profile Setup (Post-Registration)](#6-auth-re-003--profile-setup-post-registration)
7. [AUTH-FP-001 — Forgot Password (Phone Entry)](#7-auth-fp-001--forgot-password-phone-entry)
8. [AUTH-FP-002 — Forgot Password OTP Verification](#8-auth-fp-002--forgot-password-otp-verification)
9. [AUTH-FP-003 — New Password Entry](#9-auth-fp-003--new-password-entry)
10. [AUTH-2F-001 — Two-Factor Authentication](#10-auth-2f-001--two-factor-authentication)
11. [AUTH-SE-001 — Session Expired Page](#11-auth-se-001--session-expired-page)
12. [AUTH-AL-001 — Account Locked Page](#12-auth-al-001--account-locked-page)
13. [AUTH-AS-001 — Account Suspended Page](#13-auth-as-001--account-suspended-page)
14. [AUTH-EV-001 — Email Verification Page](#14-auth-ev-001--email-verification-page)
15. [State Machines](#15-state-machines)
16. [Cross-Cutting Concerns](#16-cross-cutting-concerns)

---

## 1. Global Conventions

### 1.1 Design Tokens

| Token | Value |
|---|---|
| Primary Color | `#0D9488` (Teal 600) |
| Primary Hover | `#0F766E` (Teal 700) |
| Error | `#DC2626` (Red 600) |
| Warning | `#D97706` (Amber 600) |
| Success | `#16A34A` (Green 600) |
| Background | `#F9FAFB` (Gray 50) |
| Card Background | `#FFFFFF` |
| Text Primary | `#111827` (Gray 900) |
| Text Secondary | `#6B7280` (Gray 500) |
| Border | `#D1D5DB` (Gray 300) |
| Border Focus | `#0D9488` (Teal 600) |
| Font Family (AR) | `IBM Plex Sans Arabic`, `Noto Sans Arabic`, sans-serif |
| Font Family (EN) | `Inter`, `IBM Plex Sans`, sans-serif |
| Border Radius | `8px` (cards), `6px` (inputs), `9999px` (pills) |
| Spacing Unit | `4px` base |
| Max Width (Auth Pages) | `420px` |
| Max Width (Profile Setup) | `560px` |

### 1.2 RTL Layout Rules

- All content flows **right-to-left**.
- Input fields, labels, and helper text are **right-aligned**.
- Phone input prefix `+967` is placed on the **left** side of the field (trailing edge in RTL).
- Back navigation icon (arrow) appears on the **right** side of the header.
- Validation error icons appear on the **left** side of the input (trailing edge).
- Progress indicators fill **right-to-left**.
- Password strength meter fills **right-to-left**.

### 1.3 Phone Number Format

- Country code: `+967` (Yemen) — hardcoded, non-editable prefix.
- Local number: 9 digits, starting with `7` (e.g., `7XXXXXXXX`).
- Display format: `+967 7XX XXX XXX`.
- API format: `+9677XXXXXXXX` (E.164 without spaces).

### 1.4 OTP Configuration

| Parameter | Value |
|---|---|
| OTP Length | 6 digits |
| OTP Delivery | SMS only |
| Delivery SLA | ≤ 30 seconds |
| Expiry Window | 5 minutes |
| Max Resend Attempts | 3 per 10-minute window |
| Resend Cooldown | 60 seconds between attempts |
| Character Type | Numeric only |

### 1.5 Authentication Token Configuration

| Parameter | Value |
|---|---|
| Algorithm | RS256 (RSA + SHA-256) |
| Access Token Lifetime | 15 minutes |
| Refresh Token Lifetime | 7 days |
| Session Inactivity Timeout | 24 hours |
| Token Storage | HTTP-only Secure Cookie + Memory |
| Refresh Strategy | Silent refresh via refresh token |

---

## 2. AUTH-LO-001 — Login Page (Phone Entry)

### 2.1 Page Metadata

| Field | Value |
|---|---|
| **Page ID** | AUTH-LO-001 |
| **Page Name** | تسجيل الدخول |
| **Page Name (EN)** | Login |
| **Route** | `/auth/login` |
| **Requires Auth** | No |
| **Priority** | P0 — Critical |
| **Last Updated** | 2026-09-13 |

### 2.2 Page Purpose

Allow existing YemenMart users to initiate authentication by entering their registered phone number. The system determines whether the user has a password set and routes accordingly (password login or OTP login).

### 2.3 Entry Points

| Source | Mechanism |
|---|---|
| Homepage header "تسجيل الدخول" button | Direct navigation |
| Protected page redirect | `?redirect=/previous-path` |
| Registration success | Post-registration flow completion |
| Deep link | `/auth/login?phone=+967XXXXXXXXX` |
| Session expired redirect | `/auth/login?reason=session_expired` |

### 2.4 Exit Points

| Destination | Trigger |
|---|---|
| AUTH-LO-002 (OTP Verification) | OTP-only user submits phone |
| AUTH-LO-002 (Password Entry) | Password-enabled user submits phone |
| AUTH-RE-001 (Registration) | "ليس لديك حساب؟" link clicked |
| AUTH-FP-001 (Forgot Password) | "نسيت كلمة المرور؟" link clicked |
| Homepage `/` | Back/close action |
| Previous page | `?redirect=` parameter |

### 2.5 Information Architecture

```
Login Page
├── Header
│   ├── Logo (YemenMart)
│   ├── Back Button (if navigated from protected page)
│   └── Language Toggle (AR/EN)
├── Main Content
│   ├── Page Title: "تسجيل الدخول"
│   ├── Subtitle: "أدخل رقم هاتفك المحدد عند التسجيل"
│   ├── Phone Input Group
│   │   ├── Country Code Badge: "+967"
│   │   ├── Phone Number Field
│   │   ├── Helper Text
│   │   └── Validation Error
│   ├── Primary CTA: "إرسال رمز التحقق"
│   └── Divider: "أو"
├── Footer Links
│   ├── "ليس لديك حساب؟ أنشئ حساباً" → AUTH-RE-001
│   └── "نسيت كلمة المرور؟" → AUTH-FP-001
└── Footer
    └── Terms & Privacy links
```

### 2.6 Layout Structure — ASCII Wireframe

```
┌─────────────────────────────────────────────────────────────┐
│ ◀ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─  [AR | EN] │
│                          ◀ (back)          │    │           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│                     ┌───────────────┐                       │
│                     │   YemenMart   │                       │
│                     │     Logo      │                       │
│                     └───────────────┘                       │
│                                                             │
│                     تسجيل الدخول                             │
│              أدخل رقم هاتفك المحدد عند التسجيل              │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │ +967  │  7XX  XXX  XXX                             │  │
│    │───────│─────────────────────────────────────────────│  │
│    └─────────────────────────────────────────────────────┘  │
│     ▲                                                       │
│     └── 546- 7XX- XXX- XXX                                   │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │              إرسال رمز التحقق                       │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│                    ──── أو ────                              │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │        تسجيل الدخول بكلمة المرور                    │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│         ليس لديك حساب؟  أنشئ حساباً                         │
│                                                             │
│              نسيت كلمة المرور؟                               │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│  شروط الاستخدام  ·  سياسة الخصوصية                          │
└─────────────────────────────────────────────────────────────┘
```

### 2.7 Responsive Design

| Breakpoint | Behavior |
|---|---|
| **Mobile** (< 640px) | Full-width card, centered vertically, no side decoration |
| **Tablet** (640–1024px) | Centered card, max-width 420px, light background pattern |
| **Desktop** (> 1024px) | Two-column: left = illustration, right = form (flipped in RTL: illustration left, form right) |

### 2.8 Widget Inventory

| Widget ID | Widget Type | Purpose |
|---|---|---|
| W-LO-001 | Logo | Brand identity |
| W-LO-002 | Back Button | Navigation back |
| W-LO-003 | Language Toggle | Switch AR/EN |
| W-LO-004 | Phone Input | Enter phone number |
| W-LO-005 | Primary Button | Submit phone |
| W-LO-006 | Password Login Button | Switch to password mode |
| W-LO-007 | Link — Register | Navigate to registration |
| W-LO-008 | Link — Forgot Password | Navigate to forgot password |
| W-LO-009 | Terms Links | Open terms/privacy pages |

### 2.9 Widget Specification

#### W-LO-004 — Phone Input

| Property | Value |
|---|---|
| Type | Composite Input Group |
| Components | Country Code Badge + Text Input |
| Input Mode | `tel` |
| Pattern | `[7][0-9]{8}` |
| Max Length | 9 (local number) |
| Prefix | `+967` (non-editable badge) |
| Placeholder | `7XX XXX XXX` |
| Autofocus | Yes |
| Keyboard | Numeric pad on mobile |
| Autocomplete | `tel-national` |

**States:**

| State | Visual |
|---|---|
| Default | Gray border, placeholder visible |
| Focused | Teal border, border-width 2px, subtle shadow |
| Filled | Gray border, value visible |
| Error | Red border, error icon left, error message below |
| Disabled | Gray background, reduced opacity |
| Loading | Subtle pulse animation on border |

#### W-LO-005 — Primary Button

| Property | Value |
|---|---|
| Label | "إرسال رمز التحقق" |
| Label (EN) | "Send Verification Code" |
| Full Width | Yes |
| Height | 48px |
| Background | `#0D9488` |
| Text Color | `#FFFFFF` |
| Loading State | Spinner replaces text, button disabled |
| Disabled State | `opacity: 0.5`, no pointer events |

### 2.10 Data Requirements

| Endpoint | Method | Payload | Response |
|---|---|---|---|
| `POST /api/v1/auth/check-phone` | POST | `{ "phone": "+9677XXXXXXXX" }` | `{ "exists": true, "hasPassword": true/false }` |

**Request:**
```json
{
  "phone": "+967712345678"
}
```

**Response (200):**
```json
{
  "exists": true,
  "hasPassword": true,
  "maskedPhone": "+967 7XX *** **8"
}
```

**Response (200 — not registered):**
```json
{
  "exists": false,
  "hasPassword": false,
  "maskedPhone": "+967 7XX *** **8"
}
```

### 2.11 Data Loading Strategy

- No data pre-loaded on page mount.
- Phone check request sent on form submit (debounced 300ms).
- Response cached in component state for 30 seconds to prevent duplicate checks.
- If network error, display generic error with retry.

### 2.12 User Actions

| # | Action | Description |
|---|---|---|
| 1 | Enter phone number | User types 9-digit local number |
| 2 | Submit phone | Clicks "إرسال رمز التحقق" |
| 3 | Toggle to password login | Clicks "تسجيل الدخول بكلمة المرور" |
| 4 | Navigate to register | Clicks "أنشئ حساباً" |
| 5 | Navigate to forgot password | Clicks "نسيت كلمة المرور؟" |
| 6 | Toggle language | Clicks language toggle |
| 7 | Go back | Clicks back button or browser back |

### 2.13 Form Specification

```yaml
form:
  id: login-phone-form
  fields:
    - name: phone
      type: tel
      label: "رقم الهاتف"
      label_en: "Phone Number"
      required: true
      prefix: "+967"
      placeholder: "7XX XXX XXX"
      maxLength: 9
      pattern: "^7[0-9]{8}$"
      inputMode: "numeric"
      autocomplete: "tel-national"
      aria-describedby: "phone-helper"
      aria-invalid: false  # dynamic
  submitButton:
    label: "إرسال رمز التحقق"
    label_en: "Send Verification Code"
    fullWidth: true
    loadingLabel: "جارٍ الإرسال..."
```

### 2.14 Validation Rules

| Rule | Condition | Error Message (AR) | Error Message (EN) |
|---|---|---|---|
| Required | Empty field | "رقم الهاتف مطلوب" | "Phone number is required" |
| Format | Not matching `^7[0-9]{8}$` | "رقم الهاتف غير صحيح. يجب أن يبدأ بـ 7 ويحتوي على 9 أرقام" | "Invalid phone number. Must start with 7 and contain 9 digits" |
| Length | < 9 digits | "رقم الهاتف يجب أن يكون 9 أرقام" | "Phone number must be 9 digits" |

### 2.15 State Machine — Login Phone Entry

```
[IDLE] ──user types phone──▶ [FILLED] ──user clears──▶ [IDLE]
   │                              │
   │                              ▼
   │                         [SUBMITTING]
   │                              │
   │              ┌───────────────┼───────────────┐
   │              ▼               ▼               ▼
   │         [SUCCESS]       [ERROR]         [RATE_LIMITED]
   │              │               │               │
   │              ▼               ▼               ▼
   │    ┌─────────────┐   [FILLED]          [LOCKED]
   │    │ hasPassword? │     │                 │
   │    └──────┬──────┘     │                 ▼
   │      ┌────┴────┐       │           [ACCOUNT_LOCKED]
   │      ▼         ▼       │
   │    [YES]      [NO]     │
   │      │         │       │
   │      ▼         ▼       │
   │  [PASSWORD]  [OTP]     │
   │  [MODE]      [MODE]    │
   └────────────────────────┘
```

### 2.16 Error States

| Error Code | Condition | UI Treatment |
|---|---|---|
| `AUTH_PHONE_INVALID` | Invalid format | Inline error below input |
| `AUTH_PHONE_NOT_FOUND` | Number not registered | Redirect to AUTH-RE-001 with phone pre-filled |
| `AUTH_RATE_LIMITED` | Too many requests | Toast notification + 60s cooldown timer on button |
| `AUTH_ACCOUNT_LOCKED` | Too many failed attempts | Redirect to AUTH-AL-001 |
| `AUTH_NETWORK_ERROR` | No connection | Toast: "تحقق من اتصالك بالإنترنت" |
| `AUTH_SERVER_ERROR` | 500 | Toast: "حدث خطأ. حاول مرة أخرى" |

### 2.17 Empty States

- **Initial Load:** Phone input empty with placeholder, autofocused.
- **After Clear:** Returns to placeholder state.

### 2.18 Loading States

| State | Visual |
|---|---|
| Checking phone | Button shows spinner, input disabled |
| Network request pending | Overlay with subtle spinner |

### 2.19 Success States

- On phone check success with `exists=true`: Navigate to AUTH-LO-002 with phone number in state (not URL).
- On phone check success with `exists=false`: Navigate to AUTH-RE-001 with phone number pre-filled.

### 2.20 Security UX

- Phone number is never stored in `localStorage`.
- Phone number passed between pages via encrypted session state only.
- Rate limit counter displayed as subtle hint: "المحاولات المتبقية: 2".
- No phone number in URL query parameters.

### 2.21 RTL/LTR Behavior

- All text right-aligned.
- Phone prefix `+967` on left side of input.
- Back arrow on right side.
- Error text right-aligned, error icon on left.

### 2.22 Internationalization

| Key | AR | EN |
|---|---|---|
| `auth.login.title` | تسجيل الدخول | Login |
| `auth.login.subtitle` | أدخل رقم هاتفك المحدد عند التسجيل | Enter your registered phone number |
| `auth.login.phone_label` | رقم الهاتف | Phone Number |
| `auth.login.phone_placeholder` | 7XX XXX XXX | 7XX XXX XXX |
| `auth.login.submit_otp` | إرسال رمز التحقق | Send Verification Code |
| `auth.login.submit_password` | تسجيل الدخول بكلمة المرور | Login with Password |
| `auth.login.no_account` | ليس لديك حساب؟ | Don't have an account? |
| `auth.login.register` | أنشئ حساباً | Create one |
| `auth.login.forgot_password` | نسيت كلمة المرور؟ | Forgot password? |
| `auth.login.or` | أو | or |

### 2.23 Accessibility

| Requirement | Implementation |
|---|---|
| ARIA labels | `aria-label="رقم الهاتف"` on input |
| ARIA live regions | Validation errors announced via `aria-live="polite"` |
| Focus management | Auto-focus on phone input on page load |
| Keyboard navigation | Tab order: Phone → Submit → Register → Forgot Password |
| Screen reader | Country code announced as "م_prefijo_prefijo +967" |
| Color contrast | All text ≥ 4.5:1 ratio, buttons ≥ 3:1 |
| Touch targets | Minimum 44×44px for all interactive elements |

### 2.24 Performance Requirements

| Metric | Target |
|---|---|
| First Contentful Paint | ≤ 1.2s |
| Time to Interactive | ≤ 2.0s |
| Phone check API response | ≤ 500ms (p95) |
| Form validation (client) | ≤ 50ms |
| Page weight | ≤ 150KB (gzipped) |

---

## 3. AUTH-LO-002 — OTP Verification Page (Login)

### 3.1 Page Metadata

| Field | Value |
|---|---|
| **Page ID** | AUTH-LO-002 |
| **Page Name** | التحقق من الرمز |
| **Page Name (EN)** | OTP Verification |
| **Route** | `/auth/login/verify-otp` |
| **Requires Auth** | No |
| **Priority** | P0 — Critical |
| **Last Updated** | 2026-09-13 |

### 3.2 Page Purpose

Verify the user's identity by validating the 6-digit OTP sent via SMS. Upon successful verification, issue authentication tokens and redirect to the dashboard or the `redirect` URL.

### 3.3 Entry Points

| Source | Mechanism |
|---|---|
| AUTH-LO-001 (phone submit) | Successful phone check |
| Deep link (with token) | `/auth/login/verify-otp?token=xxx` (resend link) |

### 3.4 Exit Points

| Destination | Trigger |
|---|---|
| Dashboard `/` | Successful OTP verification |
| AUTH-FP-003 (if reset flow) | Password reset flow |
| AUTH-LO-001 | Back button / manual navigation |
| Previous page | `redirect` parameter |

### 3.5 Information Architecture

```
OTP Verification (Login)
├── Header
│   ├── Logo
│   ├── Back Button
│   └── Language Toggle
├── Main Content
│   ├── Page Title: "التحقق من الرمز"
│   ├── Subtitle: "أدخل الرمز المكون من 6 أرقام المرسل إلى +967 7XX *** **X"
│   ├── OTP Input Group (6 individual digit boxes)
│   ├── Timer: "إعادة الإرسال خلال 00:45"
│   ├── Resend Button: "إعادة إرسال الرمز" (disabled during countdown)
│   ├── Error Display Area
│   └── Verify Button (auto-submit on 6 digits)
├── Footer
│   ├── "تغيير رقم الهاتف" → AUTH-LO-001
│   └── Attempt Counter: "المحاولات المتبقية: 3"
```

### 3.6 Layout Structure — ASCII Wireframe

```
┌─────────────────────────────────────────────────────────────┐
│ ◀ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─  [AR | EN] │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│                     ┌───────────────┐                       │
│                     │   YemenMart   │                       │
│                     └───────────────┘                       │
│                                                             │
│                     التحقق من الرمز                          │
│         أدخل الرمز المكون من 6 أرقام المرسل إلى            │
│                     +967 7XX *** **8                         │
│                                                             │
│    ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐       │
│    │  6  │ │  6  │ │  6  │ │  6  │ │  6  │ │  6  │       │
│    └─────┘ └─────┘ └─────┘ └─────┘ └─────┘ └─────┘       │
│                                                             │
│         ◀──  إعادة الإرسال خلال 01:23  ──▶                   │
│                                                             │
│         إعادة إرسال الرمز  (available after countdown)     │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │                   تحقق                              │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│              المحاولات المتبقية: 3                           │
│              تغيير رقم الهاتف                                │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 3.7 Responsive Design

| Breakpoint | Behavior |
|---|---|
| **Mobile** | Full-width OTP boxes, stacked layout |
| **Tablet** | Centered, max-width 420px |
| **Desktop** | Centered card with illustration panel |

### 3.8 Widget Inventory

| Widget ID | Widget Type | Purpose |
|---|---|---|
| W-OTP-001 | Logo | Brand identity |
| W-OTP-002 | Back Button | Navigate back to phone entry |
| W-OTP-003 | OTP Input (6-box) | Enter verification code |
| W-OTP-004 | Resend Timer | Countdown to resend availability |
| W-OTP-005 | Resend Button | Request new OTP |
| W-OTP-006 | Auto-submit Handler | Submit on 6th digit |
| W-OTP-007 | Attempt Counter | Display remaining attempts |
| W-OTP-008 | Change Phone Link | Navigate to phone entry |

### 3.9 Widget Specification

#### W-OTP-003 — OTP Input (6-Box)

| Property | Value |
|---|---|
| Type | 6 individual input fields |
| Input Mode | `numeric` |
| Pattern | `[0-9]` per box |
| Max Length | 1 per box |
| Auto-advance | Yes (auto-focus next on input) |
| Auto-backspace | Yes (focus previous on delete) |
| Paste Support | Yes (distribute digits across boxes) |
| Keyboard | Numeric pad on mobile |
| Autofocus | First box |

**States:**

| State | Visual |
|---|---|
| Default | Gray border, centered digit |
| Focused | Teal border, border-width 2px, subtle glow |
| Filled | Gray border, bold digit |
| Error | Red border, shake animation |
| Verified | Green border, checkmark icon |

### 3.10 Data Requirements

| Endpoint | Method | Payload | Response |
|---|---|---|---|
| `POST /api/v1/auth/verify-otp` | POST | `{ "phone": "+9677XXXXXXXX", "otp": "123456", "purpose": "login" }` | `{ "accessToken": "...", "refreshToken": "...", "user": {...} }` |
| `POST /api/v1/auth/resend-otp` | POST | `{ "phone": "+9677XXXXXXXX", "purpose": "login" }` | `{ "expiresIn": 300, "resendAfter": 60 }` |

**Verify OTP Response (200):**
```json
{
  "accessToken": "eyJhbGciOiJSUzI1NiIs...",
  "refreshToken": "dGhpcyBpcyBhIHJlZn...",
  "tokenType": "Bearer",
  "expiresIn": 900,
  "user": {
    "id": "usr_xxxxxxxx",
    "phone": "+967712345678",
    "name": "محمد أحمد",
    "role": "buyer",
    "emailVerified": false,
    "phoneVerified": true
  }
}
```

**Resend OTP Response (200):**
```json
{
  "success": true,
  "expiresIn": 300,
  "resendAfter": 60,
  "attemptsRemaining": 2
}
```

### 3.11 Data Loading Strategy

- OTP page loads with countdown timer initialized from `expiresIn` (300s).
- Timer runs client-side with `setInterval(1000)`.
- Resend button enabled after `resendAfter` seconds.
- Auto-submit triggered on 6th digit entry.
- No polling; event-driven verification.

### 3.12 User Actions

| # | Action | Description |
|---|---|---|
| 1 | Enter OTP | Type 6 digits, auto-advancing |
| 2 | Paste OTP | Paste 6-digit code, auto-distribute |
| 3 | Resend OTP | Click resend after countdown |
| 4 | Change phone | Navigate back to phone entry |
| 5 | Go back | Back button |

### 3.13 Form Specification

```yaml
form:
  id: otp-verify-form
  fields:
    - name: otp
      type: otp-group
      label: "رمز التحقق"
      label_en: "Verification Code"
      required: true
      length: 6
      inputMode: "numeric"
      pattern: "^[0-9]{6}$"
      autoAdvance: true
      autoSubmit: true
      pasteDistribute: true
      aria-describedby: "otp-helper"
```

### 3.14 Validation Rules

| Rule | Condition | Error Message (AR) | Error Message (EN) |
|---|---|---|---|
| Required | Empty field | "أدخل رمز التحقق" | "Enter verification code" |
| Length | < 6 digits | "الرمز يجب أن يكون 6 أرقام" | "Code must be 6 digits" |
| Format | Non-numeric | "الرمز يجب أن يحتوي على أرقام فقط" | "Code must contain digits only" |
| Invalid OTP | Server returns 401 | "الرمز غير صحيح. حاول مرة أخرى" | "Invalid code. Try again" |
| Expired OTP | Server returns 410 | "انتهت صلاحية الرمز. اطلب رمزاً جديداً" | "Code expired. Request a new one" |
| Max Attempts | Server returns 429 | "تم تجاوز الحد الأقصى للمحاولات. حاول لاحقاً" | "Max attempts exceeded. Try later" |

### 3.15 State Machine — OTP Verification

```
[IDLE] ──user types digits──▶ [PARTIAL] ──6th digit──▶ [AUTO_SUBMIT]
   │                               │                        │
   │                               │                   ┌────┼────┐
   │                               │                   ▼    ▼    ▼
   │                               │              [SUCCESS][VERIFYING][ERROR]
   │                               │                   │         │
   │                               │                   ▼         ▼
   │                               │              [REDIRECT] [PARTIAL]
   │                               │                          (retry)
   │                               │
   │              resend available after cooldown
   │                               │
   │              [COUNTDOWN] ────▶ [RESEND_READY] ──click──▶ [RESENDING]
   │                                    │                           │
   │                                    │                    ┌──────┼──────┐
   │                                    │                    ▼      ▼      ▼
   │                                    │               [SUCCESS][ERROR][RATE_LIMITED]
   │                                    │                    │      │
   │                                    │                    ▼      ▼
   │                                    │              [COUNTDOWN] [LOCKED]
   │                                    │                    │
   └────────────────────────────────────┘                    │
                                                             ▼
                                                      [ACCOUNT_LOCKED]
```

### 3.16 Error States

| Error Code | Condition | UI Treatment |
|---|---|---|
| `AUTH_OTP_INVALID` | Wrong OTP | Inline error + shake animation on OTP boxes |
| `AUTH_OTP_EXPIRED` | Past 5 minutes | Full error banner + auto-resend option |
| `AUTH_OTP_MAX_ATTEMPTS` | > 3 wrong attempts in session | Redirect to AUTH-AL-001 |
| `AUTH_OTP_RESEND_LIMIT` | > 3 resends in 10 minutes | Toast: "تم تجاوز حد الإرسال. حاول بعد 10 دقائق" |
| `AUTH_OTP_RATE_LIMIT` | Resend before cooldown | Timer remains, button disabled |
| `AUTH_NETWORK_ERROR` | No connection | Toast: "تحقق من اتصالك بالإنترنت" |

### 3.17 Empty States

- All OTP boxes empty with placeholder dots.
- Timer showing full countdown `05:00`.
- Resend button disabled with countdown.

### 3.18 Loading States

| State | Visual |
|---|---|
| Verifying OTP | Spinner overlay on OTP boxes, input disabled |
| Resending | Spinner on resend button, button disabled |
| Auto-submitting | Brief loading state between last digit and verification |

### 3.19 Success States

- OTP boxes turn green with checkmark.
- Brief 500ms hold on success state.
- Navigate to dashboard with smooth transition.

### 3.20 Security UX

- OTP digits masked after 2 seconds (show briefly, then mask).
- OTP not stored in any client-side storage.
- Anti-automation: subtle CAPTCHA after 2 failed attempts.
- Rate limit warnings shown proactively.
- Phone number masked in subtitle: `+967 7XX *** **8`.

### 3.21 RTL/LTR Behavior

- OTP boxes flow right-to-left (first digit on right).
- Timer text right-aligned.
- Error text right-aligned.
- Back button on right side.

### 3.22 Internationalization

| Key | AR | EN |
|---|---|---|
| `auth.otp.title` | التحقق من الرمز | OTP Verification |
| `auth.otp.subtitle` | أدخل الرمز المكون من 6 أرقام المرسل إلى | Enter the 6-digit code sent to |
| `auth.otp.resend` | إعادة إرسال الرمز | Resend Code |
| `auth.otp.resend_in` | إعادة الإرسال خلال | Resend in |
| `auth.otp.change_phone` | تغيير رقم الهاتف | Change phone number |
| `auth.otp.attempts` | المحاولات المتبقية | Attempts remaining |
| `auth.otp.verifying` | جارٍ التحقق... | Verifying... |

### 3.23 Accessibility

| Requirement | Implementation |
|---|---|
| ARIA labels | `aria-label="رقم الرمز"` on OTP group |
| ARIA live | Error/success messages announced |
| Focus management | Auto-focus first box, advance on input |
| Keyboard | Backspace moves to previous box |
| Screen reader | Each box announced as "رقم X من 6" |
| High contrast | OTP boxes visible in high contrast mode |

### 3.24 Performance Requirements

| Metric | Target |
|---|---|
| Auto-submit latency | ≤ 200ms from last digit |
| OTP verification API | ≤ 800ms (p95) |
| Resend API | ≤ 500ms (p95) |
| Timer accuracy | ≤ 100ms drift per minute |

---

## 4. AUTH-RE-001 — Registration Page (Phone Entry)

### 4.1 Page Metadata

| Field | Value |
|---|---|
| **Page ID** | AUTH-RE-001 |
| **Page Name** | إنشاء حساب جديد |
| **Page Name (EN)** | Create Account |
| **Route** | `/auth/register` |
| **Requires Auth** | No |
| **Priority** | P0 — Critical |
| **Last Updated** | 2026-09-13 |

### 4.2 Page Purpose

Allow new users to register by entering their phone number. The system sends an OTP for verification. Account is created **only after OTP verification** (Constraint: account created after OTP verification).

### 4.3 Entry Points

| Source | Mechanism |
|---|---|
| AUTH-LO-001 (no account link) | Direct navigation |
| Homepage "إنشاء حساب" button | Direct navigation |
| Deep link | `/auth/register?phone=+967XXXXXXXXX` |

### 4.4 Exit Points

| Destination | Trigger |
|---|---|
| AUTH-RE-002 (OTP Verification) | Phone submitted successfully |
| AUTH-LO-001 (Login) | "لديك حساب بالفعل؟" link |
| Homepage `/` | Back button |

### 4.5 Information Architecture

```
Registration Page
├── Header
│   ├── Logo
│   ├── Back Button
│   └── Language Toggle
├── Main Content
│   ├── Page Title: "إنشاء حساب جديد"
│   ├── Subtitle: "أدخل رقم هاتفك للبدء"
│   ├── Phone Input Group (same as AUTH-LO-001)
│   ├── Terms Checkbox: "أوافق على شروط الاستخدام وسياسة الخصوصية"
│   ├── Primary CTA: "التالي"
│   └── Divider: "أو"
├── Footer Links
│   └── "لديك حساب بالفعل؟ تسجيل الدخول" → AUTH-LO-001
└── Footer
    └── Terms & Privacy links
```

### 4.6 Layout Structure — ASCII Wireframe

```
┌─────────────────────────────────────────────────────────────┐
│ ◀ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─  [AR | EN] │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│                     ┌───────────────┐                       │
│                     │   YemenMart   │                       │
│                     └───────────────┘                       │
│                                                             │
│                   إنشاء حساب جديد                           │
│               أدخل رقم هاتفك للبدء                          │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │ +967  │  7XX  XXX  XXX                             │  │
│    │───────│─────────────────────────────────────────────│  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│    ☐ أوافق على شروط الاستخدام وسياسة الخصوصية             │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │                      التالي                         │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│                    ──── أو ────                              │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │        تسجيل الدخول بحساب موجود                     │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│         لديك حساب بالفعل؟  تسجيل الدخول                     │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│  شروط الاستخدام  ·  سياسة الخصوصية                          │
└─────────────────────────────────────────────────────────────┘
```

### 4.7 Responsive Design

Same as AUTH-LO-001.

### 4.8 Widget Inventory

| Widget ID | Widget Type | Purpose |
|---|---|---|
| W-RE-001 | Logo | Brand identity |
| W-RE-002 | Back Button | Navigation |
| W-RE-003 | Language Toggle | Switch AR/EN |
| W-RE-004 | Phone Input | Enter phone number |
| W-RE-005 | Terms Checkbox | Accept terms |
| W-RE-006 | Primary Button | Submit registration |
| W-RE-007 | Login Link | Navigate to login |

### 4.9 Widget Specification

#### W-RE-005 — Terms Checkbox

| Property | Value |
|---|---|
| Type | Checkbox with link |
| Label | "أوافق على شروط الاستخدام وسياسة الخصوصية" |
| Required | Yes |
| Links | "شروط الاستخدام" → `/terms`, "سياسة الخصوصية" → `/privacy` |
| Default | Unchecked |

### 4.10 Data Requirements

| Endpoint | Method | Payload | Response |
|---|---|---|---|
| `POST /api/v1/auth/register` | POST | `{ "phone": "+9677XXXXXXXX", "acceptTerms": true }` | `{ "otpSent": true, "expiresIn": 300, "maskedPhone": "..." }` |

### 4.11 Data Loading Strategy

- No pre-loaded data.
- Registration request sent on form submit.
- Terms checkbox must be checked before submit.
- Rate limiting applied: 3 registrations per phone per hour.

### 4.12 User Actions

| # | Action | Description |
|---|---|---|
| 1 | Enter phone | Type 9-digit number |
| 2 | Accept terms | Check terms checkbox |
| 3 | Submit | Click "التالي" |
| 4 | Navigate to login | Click "تسجيل الدخول" |
| 5 | View terms | Click terms/privacy links |

### 4.13 Form Specification

```yaml
form:
  id: register-phone-form
  fields:
    - name: phone
      type: tel
      label: "رقم الهاتف"
      required: true
      prefix: "+967"
      placeholder: "7XX XXX XXX"
      maxLength: 9
      pattern: "^7[0-9]{8}$"
      inputMode: "numeric"
    - name: acceptTerms
      type: checkbox
      label: "أوافق على شروط الاستخدام وسياسة الخصوصية"
      required: true
      links:
        - text: "شروط الاستخدام"
          url: "/terms"
        - text: "سياسة الخصوصية"
          url: "/privacy"
```

### 4.14 Validation Rules

| Rule | Condition | Error Message (AR) |
|---|---|---|
| Phone Required | Empty | "رقم الهاتف مطلوب" |
| Phone Format | Invalid | "رقم الهاتف غير صحيح" |
| Terms Required | Unchecked | "يجب الموافقة على الشروط" |
| Phone Exists | Already registered | "رقم الهاتف مسجل بالفعل. تسجيل الدخول" |

### 4.15 State Machine — Registration Phone Entry

```
[IDLE] ──user types──▶ [FILLED] ──checks terms──▶ [READY]
   │                      │                          │
   │                      │                          ▼
   │                      │                     [SUBMITTING]
   │                      │                          │
   │                      │              ┌────────────┼────────────┐
   │                      │              ▼            ▼            ▼
   │                      │         [SUCCESS]    [ERROR]    [PHONE_EXISTS]
   │                      │              │            │            │
   │                      │              ▼            ▼            ▼
   │                      │         [REDIRECT    [FILLED]    [REDIRECT
   │                      │          to RE-002]              to LO-001]
   │                      │
   └──────────────────────┘
```

### 4.16 Error States

| Error Code | Condition | UI Treatment |
|---|---|---|
| `AUTH_PHONE_EXISTS` | Already registered | Toast + suggestion to login |
| `AUTH_TERMS_REQUIRED` | Checkbox unchecked | Inline error below checkbox |
| `AUTH_RATE_LIMITED` | Too many registrations | Toast with cooldown |
| `AUTH_NETWORK_ERROR` | No connection | Toast with retry |

### 4.17–4.24

*Follow same patterns as AUTH-LO-001 with registration-specific values.*

---

## 5. AUTH-RE-002 — Registration OTP Verification

### 5.1 Page Metadata

| Field | Value |
|---|---|
| **Page ID** | AUTH-RE-002 |
| **Page Name** | التحقق من رقم الهاتف |
| **Page Name (EN)** | Phone Verification |
| **Route** | `/auth/register/verify-otp` |
| **Requires Auth** | No |
| **Priority** | P0 — Critical |
| **Last Updated** | 2026-09-13 |

### 5.2 Page Purpose

Verify the new user's phone number via OTP. **Account is created only upon successful OTP verification** (Constraint 3: phone is primary identifier, Constraint: account created after OTP verification).

### 5.3 Entry Points

| Source | Mechanism |
|---|---|
| AUTH-RE-001 (phone submit) | Successful registration request |

### 5.4 Exit Points

| Destination | Trigger |
|---|---|
| AUTH-RE-003 (Profile Setup) | Successful OTP verification |
| AUTH-RE-001 | Back / change phone |

### 5.5 Information Architecture

```
Registration OTP Verification
├── Header
│   ├── Logo
│   ├── Back Button
│   └── Language Toggle
├── Main Content
│   ├── Page Title: "التحقق من رقم الهاتف"
│   ├── Subtitle: "أدخل الرمز المرسل إلى +967 7XX *** **X"
│   ├── OTP Input Group (6-box)
│   ├── Timer: "إعادة الإرسال خلال 01:23"
│   ├── Resend Button
│   ├── Attempt Counter
│   └── Error Display
├── Footer
│   └── "تغيير رقم الهاتف" → AUTH-RE-001
```

### 5.6 Layout Structure — ASCII Wireframe

```
┌─────────────────────────────────────────────────────────────┐
│ ◀ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─  [AR | EN] │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│                     ┌───────────────┐                       │
│                     │   YemenMart   │                       │
│                     └───────────────┘                       │
│                                                             │
│                  التحقق من رقم الهاتف                        │
│          أدخل الرمز المرسل إلى +967 7XX *** **8             │
│                                                             │
│    ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐       │
│    │     │ │     │ │     │ │     │ │     │ │     │       │
│    └─────┘ └─────┘ └─────┘ └─────┘ └─────┘ └─────┘       │
│                                                             │
│         ◀──  إعادة الإرسال خلال 01:23  ──▶                   │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │                    تحقق                             │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│         المحاولات المتبقية: 3                               │
│         تغيير رقم الهاتف                                    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 5.7–5.24

*Same structure as AUTH-LO-002 with these differences:*

**API Endpoint:**
```yaml
endpoint: POST /api/v1/auth/verify-otp
payload:
  phone: "+9677XXXXXXXX"
  otp: "123456"
  purpose: "registration"
response:
  accountCreated: true
  accessToken: "..."
  refreshToken: "..."
  user: { ... }
  requiresProfileSetup: true
```

**Post-Verification Flow:**
- On success → Check `requiresProfileSetup` flag.
- If `true` → Navigate to AUTH-RE-003.
- If `false` → Navigate to dashboard.

---

## 6. AUTH-RE-003 — Profile Setup (Post-Registration)

### 6.1 Page Metadata

| Field | Value |
|---|---|
| **Page ID** | AUTH-RE-003 |
| **Page Name** | إعداد الملف الشخصي |
| **Page Name (EN)** | Profile Setup |
| **Route** | `/auth/register/profile` |
| **Requires Auth** | Yes (temp token from OTP) |
| **Priority** | P0 — Critical |
| **Last Updated** | 2026-09-13 |

### 6.2 Page Purpose

Collect essential profile information from newly registered users before granting full access. Minimal required fields to reduce friction.

### 6.3 Entry Points

| Source | Mechanism |
|---|---|
| AUTH-RE-002 (OTP success) | Auto-redirect after verification |

### 6.4 Exit Points

| Destination | Trigger |
|---|---|
| Homepage `/` | Profile saved successfully |
| AUTH-LO-001 | Skip / logout |

### 6.5 Information Architecture

```
Profile Setup
├── Header
│   ├── Logo
│   └── Language Toggle
├── Progress Bar (Step 1 of 1)
├── Main Content
│   ├── Page Title: "إعداد ملفك الشخصي"
│   ├── Subtitle: "أخبرنا عن نفسك لتخصيص تجربتك"
│   ├── Form
│   │   ├── Name Input (Arabic)
│   │   ├── Name Input (English) — optional
│   │   ├── Account Type Selector
│   │   │   ├── شاري (Buyer)
│   │   │   ├── بائع (Seller)
│   │   │   └── كلهم (Both)
│   │   ├── City/Region Selector
│   │   └── Email Input — optional (Constraint 4)
│   ├── Primary CTA: "حفظ والمتابعة"
│   └── Skip Link: "تخطي — إعداد لاحقاً"
└── Footer
```

### 6.6 Layout Structure — ASCII Wireframe

```
┌─────────────────────────────────────────────────────────────┐
│                      ┌───────────────┐              [AR | EN]│
│                      │   YemenMart   │                      │
│                      └───────────────┘                      │
│                                                             │
│ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │
│                                                             │
│                   إعداد ملفك الشخصي                          │
│             أخبرنا عن نفسك لتخصيص تجربتك                    │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │ الاسم بالعربي (مطلوب)                               │  │
│    │ محمد أحمد                                            │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │ الاسم بالإنجليزي (اختياري)                          │  │
│    │ Mohammed Ahmed                                       │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│    نوع الحساب                                               │
│    ┌──────────┐ ┌──────────┐ ┌──────────┐                  │
│    │  شاري    │ │  بائع    │ │  كلهم    │                  │
│    └──────────┘ └──────────┘ └──────────┘                  │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │ المدينة / المنطقة                                    │  │
│    │ صنعاء                                    ▼           │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │ البريد الإلكتروني (اختياري)                          │  │
│    │ example@email.com                                    │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │              حفظ والمتابعة                           │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│              تخطي — إعداد لاحقاً                              │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 6.7 Responsive Design

| Breakpoint | Behavior |
|---|---|
| **Mobile** | Full-width form, stacked fields |
| **Tablet** | Centered, max-width 560px |
| **Desktop** | Centered, max-width 560px, illustration panel |

### 6.8 Widget Inventory

| Widget ID | Widget Type | Purpose |
|---|---|---|
| W-PR-001 | Name Input (AR) | Enter Arabic name |
| W-PR-002 | Name Input (EN) | Enter English name (optional) |
| W-PR-003 | Account Type Selector | Choose buyer/seller/both |
| W-PR-004 | City Selector | Choose city/region |
| W-PR-005 | Email Input | Enter email (optional) |
| W-PR-006 | Primary Button | Save and continue |
| W-PR-007 | Skip Link | Skip setup |

### 6.9 Widget Specification

#### W-PR-003 — Account Type Selector

| Property | Value |
|---|---|
| Type | Segmented Control (3 options) |
| Options | شاري (Buyer), بائع (Seller), كلهم (Both) |
| Default | شاري (Buyer) |
| Layout | Horizontal, equal-width segments |
| Selection Indicator | Teal background, white text |

#### W-PR-004 — City Selector

| Property | Value |
|---|---|
| Type | Dropdown / Searchable Select |
| Options | Yemeni governorates (21) |
| Default | None (required) |
| Search | Yes (Arabic search) |
| Keyboard | Type-ahead search |

**Yemeni Governorates:**
```
عدن، أبين، صنعاء، صنعاء (عاصمة)، حضرموت، شبوة، 
الحديدة، تعز، إب، ذمار، مأرب، صور، سقطرى، 
حجة، المحويت، عمران، الجوف، البيضاء، لحج، ضاله، المهرة
```

### 6.10 Data Requirements

| Endpoint | Method | Payload | Response |
|---|---|---|---|
| `POST /api/v1/auth/profile` | POST | `{ "name": "...", "nameEn": "...", "accountType": "buyer", "city": "صنعاء", "email": "..." }` | `{ "success": true, "user": {...} }` |

### 6.11–6.24

*Follow established patterns. Key validation: Name required (2-50 chars), City required, Email optional but must be valid format if provided.*

---

## 7. AUTH-FP-001 — Forgot Password (Phone Entry)

### 7.1 Page Metadata

| Field | Value |
|---|---|
| **Page ID** | AUTH-FP-001 |
| **Page Name** | استعادة كلمة المرور |
| **Page Name (EN)** | Forgot Password |
| **Route** | `/auth/forgot-password` |
| **Requires Auth** | No |
| **Priority** | P0 — Critical |
| **Last Updated** | 2026-09-13 |

### 7.2 Page Purpose

Initiate password reset flow. User enters registered phone number, receives OTP for verification (Constraint 5: forgot password uses phone OTP only).

### 7.3 Entry Points

| Source | Mechanism |
|---|---|
| AUTH-LO-001 (forgot password link) | Direct navigation |
| Password error page | Link after failed password attempts |

### 7.4 Exit Points

| Destination | Trigger |
|---|---|
| AUTH-FP-002 (OTP Verification) | Phone submitted |
| AUTH-LO-001 (Login) | Back / " remembers password" |

### 7.5 Information Architecture

```
Forgot Password
├── Header
│   ├── Logo
│   ├── Back Button
│   └── Language Toggle
├── Main Content
│   ├── Page Title: "استعادة كلمة المرور"
│   ├── Subtitle: "أدخل رقم هاتفك وسنرسل لك رمز التحقق"
│   ├── Phone Input (same as AUTH-LO-001)
│   └── Primary CTA: "إرسال رمز التحقق"
├── Footer Links
│   └── "تتذكر كلمة المرور؟ تسجيل الدخول" → AUTH-LO-001
```

### 7.6 Layout Structure — ASCII Wireframe

```
┌─────────────────────────────────────────────────────────────┐
│ ◀ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─  [AR | EN] │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│                     ┌───────────────┐                       │
│                     │   YemenMart   │                       │
│                     └───────────────┘                       │
│                                                             │
│                  استعادة كلمة المرور                          │
│        أدخل رقم هاتفك وسنرسل لك رمز التحقق                 │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │ +967  │  7XX  XXX  XXX                             │  │
│    │───────│─────────────────────────────────────────────│  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │              إرسال رمز التحقق                       │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│         تتذكر كلمة المرور؟ تسجيل الدخول                     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 7.7–7.24

*Same structure as AUTH-LO-001 with password-reset-specific API:*

```yaml
endpoint: POST /api/v1/auth/forgot-password
payload:
  phone: "+9677XXXXXXXX"
response:
  success: true
  maskedPhone: "+967 7XX *** **8"
  expiresIn: 300
  resendAfter: 60
```

---

## 8. AUTH-FP-002 — Forgot Password OTP Verification

### 8.1 Page Metadata

| Field | Value |
|---|---|
| **Page ID** | AUTH-FP-002 |
| **Page Name** | التحقق من الرمز |
| **Page Name (EN)** | Verify Reset Code |
| **Route** | `/auth/forgot-password/verify-otp` |
| **Requires Auth** | No |
| **Priority** | P0 — Critical |
| **Last Updated** | 2026-09-13 |

### 8.2 Page Purpose

Verify OTP for password reset. Upon success, issue a temporary reset token (short-lived, single-use) and redirect to new password entry.

### 8.3 Entry Points

| Source | Mechanism |
|---|---|
| AUTH-FP-001 (phone submit) | Successful phone check |

### 8.4 Exit Points

| Destination | Trigger |
|---|---|
| AUTH-FP-003 (New Password) | OTP verified |
| AUTH-FP-001 | Back / change phone |

### 8.5 Layout Structure — ASCII Wireframe

```
┌─────────────────────────────────────────────────────────────┐
│ ◀ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─  [AR | EN] │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│                     ┌───────────────┐                       │
│                     │   YemenMart   │                       │
│                     └───────────────┘                       │
│                                                             │
│                     التحقق من الرمز                          │
│         أدخل الرمز المكون من 6 أرقام المرسل إلى            │
│                     +967 7XX *** **8                         │
│                                                             │
│    ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐       │
│    │     │ │     │ │     │ │     │ │     │ │     │       │
│    └─────┘ └─────┘ └─────┘ └─────┘ └─────┘ └─────┘       │
│                                                             │
│         ◀──  إعادة الإرسال خلال 01:23  ──▶                   │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │                    تحقق                             │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│         المحاولات المتبقية: 3                               │
│         تغيير رقم الهاتف                                    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 8.6–8.24

*Same as AUTH-LO-002 with:*

```yaml
endpoint: POST /api/v1/auth/verify-otp
payload:
  phone: "+9677XXXXXXXX"
  otp: "123456"
  purpose: "password_reset"
response:
  resetToken: "temporary_single_use_token"
  expiresIn: 600
```

---

## 9. AUTH-FP-003 — New Password Entry

### 9.1 Page Metadata

| Field | Value |
|---|---|
| **Page ID** | AUTH-FP-003 |
| **Page Name** | كلمة المرور الجديدة |
| **Page Name (EN)** | New Password |
| **Route** | `/auth/forgot-password/new-password` |
| **Requires Auth** | Reset token (from AUTH-FP-002) |
| **Priority** | P0 — Critical |
| **Last Updated** | 2026-09-13 |

### 9.2 Page Purpose

Allow user to set a new password after successful OTP verification. Password must meet complexity requirements.

### 9.3 Entry Points

| Source | Mechanism |
|---|---|
| AUTH-FP-002 (OTP success) | Auto-redirect with reset token |

### 9.4 Exit Points

| Destination | Trigger |
|---|---|
| AUTH-LO-001 (Login) | Password reset success |
| AUTH-LO-001 | Back button |

### 9.5 Information Architecture

```
New Password Entry
├── Header
│   ├── Logo
│   └── Language Toggle
├── Main Content
│   ├── Page Title: "كلمة المرور الجديدة"
│   ├── Subtitle: "اختر كلمة مرور قوية لحماية حسابك"
│   ├── Form
│   │   ├── New Password Input (with show/hide toggle)
│   │   │   ├── Password Strength Meter
│   │   │   └── Requirement Checklist
│   │   └── Confirm Password Input (with show/hide toggle)
│   └── Primary CTA: "حفظ كلمة المرور الجديدة"
├── Success State (post-reset)
│   ├── Success Icon
│   ├── Message: "تم تغيير كلمة المرور بنجاح"
│   └── CTA: "تسجيل الدخول"
```

### 9.6 Layout Structure — ASCII Wireframe

```
┌─────────────────────────────────────────────────────────────┐
│                      ┌───────────────┐              [AR | EN]│
│                      │   YemenMart   │                      │
│                      └───────────────┘                      │
│                                                             │
│                 كلمة المرور الجديدة                           │
│          اختر كلمة مرور قوية لحماية حسابك                   │
│                                                             │
│    ┌─────────────────────────────────────────┬──────┐       │
│    │ كلمة المرور الجديدة                     │ 👁   │       │
│    └─────────────────────────────────────────┴──────┘       │
│                                                             │
│    ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ (strength bar)   │
│                                                             │
│    ☑ 8 أحرف على الأقل                                       │
│    ☑ حرف كبير واحد على الأقل                                │
│    ☑ رقم واحد على الأقل                                     │
│    ☐ رمز خاص واحد على الأقل (!@#$%^&*)                       │
│                                                             │
│    ┌─────────────────────────────────────────┬──────┐       │
│    │ تأكيد كلمة المرور                       │ 👁   │       │
│    └─────────────────────────────────────────┴──────┘       │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │           حفظ كلمة المرور الجديدة                    │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 9.7–9.9

*Standard responsive patterns. Widgets: Password Input (2x), Show/Hide Toggle (2x), Strength Meter, Requirement Checklist, Submit Button.*

### 9.10 Data Requirements

| Endpoint | Method | Payload | Response |
|---|---|---|---|
| `POST /api/v1/auth/reset-password` | POST | `{ "resetToken": "...", "newPassword": "...", "confirmPassword": "..." }` | `{ "success": true }` |

### 9.11–9.13

*Standard form handling with real-time validation.*

### 9.14 Validation Rules

| Rule | Condition | Error Message (AR) |
|---|---|---|
| Min Length | < 8 chars | "كلمة المرور يجب أن تكون 8 أحرف على الأقل" |
| Uppercase | No uppercase letter | "يجب أن تحتوي على حرف كبير واحد على الأقل" |
| Lowercase | No lowercase letter | "يجب أن تحتوي على حرف صغير واحد على الأقل" |
| Number | No digit | "يجب أن تحتوي على رقم واحد على الأقل" |
| Special Char | No special char (optional) | "للأمان الأفضل، أضف رمزاً خاصاً (!@#$%^&*)" |
| Match | Passwords don't match | "كلمتا المرور غير متطابقتين" |
| Common | Common password | "كلمة المرور هذه شائعة. اختر كلمة أقوى" |

### 9.15–9.24

*Standard patterns. Password strength meter uses zxcvbn algorithm. Success state shows confirmation with redirect to login.*

---

## 10. AUTH-2F-001 — Two-Factor Authentication

### 10.1 Page Metadata

| Field | Value |
|---|---|
| **Page ID** | AUTH-2F-001 |
| **Page Name** | التحقق بخطوتين |
| **Page Name (EN)** | Two-Factor Authentication |
| **Route** | `/auth/2fa` |
| **Requires Auth** | Yes (partial — after password) |
| **Priority** | P1 — High |
| **Last Updated** | 2026-09-13 |

### 10.2 Page Purpose

Additional security layer for high-risk actions (password change, payment, account deletion). Requires OTP verification via SMS.

### 10.3 Entry Points

| Source | Mechanism |
|---|---|
| Password-protected action trigger | After successful password entry for sensitive action |
| Admin-initiated security challenge | System-initiated |

### 10.4 Exit Points

| Destination | Trigger |
|---|---|
| Previous action page | OTP verified |
| Dashboard | Cancel |

### 10.5 Layout Structure — ASCII Wireframe

```
┌─────────────────────────────────────────────────────────────┐
│ ◀ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─  [AR | EN] │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│                     ┌───────────────┐                       │
│                     │   YemenMart   │                       │
│                     └───────────────┘                       │
│                                                             │
│                   🔒 التحقق بخطوتين                         │
│        أدخل الرمز المرسل إلى +967 7XX *** **8              │
│                                                             │
│    ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐       │
│    │     │ │     │ │     │ │     │ │     │ │     │       │
│    └─────┘ └─────┘ └─────┘ └─────┘ └─────┘ └─────┘       │
│                                                             │
│         ◀──  إعادة الإرسال خلال 01:23  ──▶                   │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │                    تحقق                             │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│              إلغاء                                          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 10.6–10.24

*Same OTP verification patterns as AUTH-LO-002. Additional security: 2FA token has 5-minute expiry, single-use, tied to specific action.*

---

## 11. AUTH-SE-001 — Session Expired Page

### 11.1 Page Metadata

| Field | Value |
|---|---|
| **Page ID** | AUTH-SE-001 |
| **Page Name** | انتهت الجلسة |
| **Page Name (EN)** | Session Expired |
| **Route** | `/auth/session-expired` |
| **Requires Auth** | No |
| **Priority** | P1 — High |
| **Last Updated** | 2026-09-13 |

### 11.2 Page Purpose

Inform user that their session has expired (24-hour inactivity timeout) and prompt re-authentication.

### 11.3 Entry Points

| Source | Mechanism |
|---|---|
| Auth middleware | Session token expired during request |
| Inactivity timeout | 24-hour inactivity threshold reached |
| Refresh token expired | 7-day refresh token lifetime exceeded |

### 11.4 Exit Points

| Destination | Trigger |
|---|---|
| AUTH-LO-001 (Login) | "تسجيل الدخول" button |
| Homepage `/` | Back button |

### 11.5 Layout Structure — ASCII Wireframe

```
┌─────────────────────────────────────────────────────────────┐
│                      ┌───────────────┐                      │
│                      │   YemenMart   │                      │
│                      └───────────────┘                      │
│                                                             │
│                      ⏰ انتهت الجلسة                         │
│                                                             │
│         لحماية حسابك، انتهت صلاحية جلستك                    │
│         بسبب عدم النشاط لفترة طويلة.                         │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │              تسجيل الدخول مرة أخرى                  │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│         العودة إلى الصفحة الرئيسية                           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 11.6–11.24

*Static page, minimal complexity. No form validation. Focus on clear messaging and easy re-authentication path.*

---

## 12. AUTH-AL-001 — Account Locked Page

### 12.1 Page Metadata

| Field | Value |
|---|---|
| **Page ID** | AUTH-AL-001 |
| **Page Name** | الحساب مقفل |
| **Page Name (EN)** | Account Locked |
| **Route** | `/auth/account-locked` |
| **Requires Auth** | No |
| **Priority** | P1 — High |
| **Last Updated** | 2026-09-13 |

### 12.2 Page Purpose

Inform user that their account is temporarily locked due to too many failed authentication attempts. Display lock duration and contact support option.

### 12.3 Entry Points

| Source | Mechanism |
|---|---|
| AUTH-LO-002 | Max OTP attempts exceeded |
| AUTH-LO-001 | Multiple failed login attempts |
| AUTH-FP-002 | Max OTP attempts exceeded |

### 12.4 Exit Points

| Destination | Trigger |
|---|---|
| AUTH-LO-001 (Login) | After lock period expires |
| Support page | "تواصل معنا" link |
| Homepage `/` | Back button |

### 12.5 Layout Structure — ASCII Wireframe

```
┌─────────────────────────────────────────────────────────────┐
│                      ┌───────────────┐                      │
│                      │   YemenMart   │                      │
│                      └───────────────┘                      │
│                                                             │
│                      🔒 الحساب مقفل                          │
│                                                             │
│         تم قفل حسابك مؤقتاً بسبب محاولات                     │
│         تسجيل دخول فاشلة متعددة.                             │
│                                                             │
│         مدة القفل: 30 دقيقة                                 │
│         الوقت المتبقي: 28:45                                 │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │          العودة إلى تسجيل الدخول                     │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│    هل تحتاج مساعدة؟ تواصل معنا                              │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 12.6–12.24

*Static page with countdown timer. Lock duration: 30 minutes (configurable). Contact support link available.*

---

## 13. AUTH-AS-001 — Account Suspended Page

### 13.1 Page Metadata

| Field | Value |
|---|---|
| **Page ID** | AUTH-AS-001 |
| **Page Name** | الحساب معلّق |
| **Page Name (EN)** | Account Suspended |
| **Route** | `/auth/account-suspended` |
| **Requires Auth** | No |
| **Priority** | P1 — High |
| **Last Updated** | 2026-09-13 |

### 13.2 Page Purpose

Inform user that their account has been permanently suspended due to policy violation. Provide appeal option.

### 13.3 Entry Points

| Source | Mechanism |
|---|---|
| Auth middleware | Account status = `suspended` |
| Login attempt | Account flagged as suspended |

### 13.4 Exit Points

| Destination | Trigger |
|---|---|
| Appeal form | "تقديم استئناف" button |
| Homepage `/` | Back button |

### 13.5 Layout Structure — ASCII Wireframe

```
┌─────────────────────────────────────────────────────────────┐
│                      ┌───────────────┐                      │
│                      │   YemenMart   │                      │
│                      └───────────────┘                      │
│                                                             │
│                      ⚠️ الحساب معلّق                         │
│                                                             │
│         تم تعليق حسابك بسبب مخالفة                          │
│         لشروط الاستخدام.                                     │
│                                                             │
│         السبب: [reason displayed]                            │
│         رقم الحادثة: [ticket number]                         │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │              تقديم استئناف                           │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│    العودة إلى الصفحة الرئيسية                                │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 13.6–13.24

*Static page. Appeal link opens support form. No re-authentication possible.*

---

## 14. AUTH-EV-001 — Email Verification Page

### 14.1 Page Metadata

| Field | Value |
|---|---|
| **Page ID** | AUTH-EV-001 |
| **Page Name** | تأكيد البريد الإلكتروني |
| **Page Name (EN)** | Email Verification |
| **Route** | `/auth/verify-email` |
| **Requires Auth** | Yes |
| **Priority** | P2 — Medium |
| **Last Updated** | 2026-09-13 |

### 14.2 Page Purpose

Optional email verification (Constraint 4: email is optional). Display verification status and allow user to verify email if provided.

### 14.3 Entry Points

| Source | Mechanism |
|---|---|
| Profile page | "تأكيد البريد" link |
| Email change notification | After email update |
| Deep link | `/auth/verify-email?token=xxx` |

### 14.4 Exit Points

| Destination | Trigger |
|---|---|
| Profile page | Back button |
| Dashboard | After verification |

### 14.5 Layout Structure — ASCII Wireframe

```
┌─────────────────────────────────────────────────────────────┐
│                      ┌───────────────┐              [AR | EN]│
│                      │   YemenMart   │                      │
│                      └───────────────┘                      │
│                                                             │
│                  📧 تأكيد البريد الإلكتروني                   │
│                                                             │
│         أرسلنا رابط التأكيد إلى                              │
│         m***@example.com                                    │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │          إعادة إرسال البريد                          │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│         تحقق من بريدك Spam/ Junk                            │
│                                                             │
│         تخطي —不确定性 later                                 │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 14.6–14.24

*Simple verification flow. Email verification link sent via email, not SMS. Token valid for 24 hours.*

---

## 15. State Machines

### 15.1 Master Authentication State Machine

```
                    ┌──────────────────────────┐
                    │                          │
                    ▼                          │
              ┌──────────┐                     │
              │  GUEST   │───── visit ────────▶│
              └────┬─────┘                     │
                   │                           │
         ┌─────────┼─────────┐                 │
         ▼         ▼         ▼                 │
    ┌─────────┐ ┌────────┐ ┌────────┐         │
    │  LOGIN  │ │REGISTER│ │ FORGOT │         │
    │  FLOW   │ │  FLOW  │ │PASSWORD│         │
    └────┬────┘ └───┬────┘ └───┬────┘         │
         │          │          │               │
         ▼          ▼          ▼               │
    ┌─────────────────────────────────┐       │
    │        OTP VERIFICATION         │       │
    │    (purpose: login/register/    │       │
    │     password_reset)             │       │
    └──────────────┬──────────────────┘       │
                   │                           │
         ┌─────────┼─────────┐                 │
         ▼         ▼         ▼                 │
    ┌────────┐ ┌────────┐ ┌────────┐         │
    │ SUCCESS│ │ FAILED │ │EXPIRED │         │
    └───┬────┘ └───┬────┘ └───┬────┘         │
        │          │          │               │
        ▼          │          │               │
   ┌─────────┐     │          │               │
   │AUTHENTICATED│  │          │               │
   │  (JWT)   │     │          │               │
   └────┬─────┘     │          │               │
        │           │          │               │
   ┌────┴────┐      │          │               │
   │         │      │          │               │
   ▼         ▼      │          │               │
┌──────┐ ┌──────┐   │          │               │
│DASHBOARD│ │2FA  │   │          │               │
│(normal) │ │REQUIRED│ │          │               │
└──────┘ └──────┘   │          │               │
                     │          │               │
                     └──────────┘               │
                            │                   │
                            └───────────────────┘
```

### 15.2 Session State Machine

```
[FRESH] ──login──▶ [ACTIVE] ──activity──▶ [ACTIVE] (refresh timeout)
                       │                        │
                       │                   24h inactivity
                       │                        │
                       │                        ▼
                       │                  [EXPIRED]
                       │                        │
                       │                        ▼
                       │                  [AUTH_SE-001]
                       │
                  7 days max
                       │
                       ▼
                 [REFRESH_NEEDED]
                       │
              ┌────────┼────────┐
              ▼                 ▼
         [REFRESHED]      [REFRESH_FAILED]
              │                 │
              ▼                 ▼
          [ACTIVE]        [EXPIRED]
```

### 15.3 OTP Delivery State Machine

```
[IDLE] ──request──▶ [SENDING] ──success──▶ [SENT]
                        │                     │
                        │                     ▼
                        ▼              ┌──────────────┐
                   [FAILED]           │ COUNTDOWN    │
                        │             │ (60s resend) │
                        ▼             └──────┬───────┘
                   [RETRY]                   │
                        │              60s elapsed
                        │                    │
                        │                    ▼
                        │             [RESEND_READY]
                        │                    │
                        │              resend clicked
                        │                    │
                        │                    ▼
                        │             [RESENDING] ──success──▶ [SENT]
                        │                    │
                        │              resend failed
                        │                    │
                        │                    ▼
                        │             [RESEND_FAILED]
                        │
                   10 min window
                        │
                        ▼
                  [RATE_LIMITED] ──10 min──▶ [IDLE]
```

---

## 16. Cross-Cutting Concerns

### 16.1 Error Toast Specifications

| Toast Type | Position | Duration | Dismissible |
|---|---|---|---|
| Error | Bottom-center (mobile), Top-right (desktop) | 5 seconds | Yes |
| Success | Bottom-center | 3 seconds | Yes |
| Warning | Bottom-center | 4 seconds | Yes |
| Info | Bottom-center | 3 seconds | Yes |

### 16.2 Error Toast Structure (RTL)

```
┌─────────────────────────────────────────────────────────┐
│ ⚠  [Error message text here]                        ✕  │
└─────────────────────────────────────────────────────────┘
  ▲                    ▲                                   │
  │                    │                                   │
  icon               text                              close
  (left)            (right-aligned)                    (left)
```

### 16.3 Security Headers

| Header | Value |
|---|---|
| `X-Content-Type-Options` | `nosniff` |
| `X-Frame-Options` | `DENY` |
| `X-XSS-Protection` | `1; mode=block` |
| `Strict-Transport-Security` | `max-age=31536000; includeSubDomains` |
| `Content-Security-Policy` | `default-src 'self'; script-src 'self'` |
| `Referrer-Policy` | `strict-origin-when-cross-origin` |

### 16.4 Token Storage Strategy

| Token | Storage | Lifetime | Security |
|---|---|---|---|
| Access Token | HTTP-only Secure Cookie + Memory | 15 minutes | HttpOnly, Secure, SameSite=Strict |
| Refresh Token | HTTP-only Secure Cookie | 7 days | HttpOnly, Secure, SameSite=Strict |
| Reset Token | Memory only (session) | 10 minutes | Single-use, short-lived |
| 2FA Token | Memory only | 5 minutes | Single-use, action-bound |

### 16.5 Rate Limiting UI

| Action | Limit | Window | UI Treatment |
|---|---|---|---|
| Phone check | 10 requests | 1 minute | Cooldown timer on button |
| OTP send | 3 requests | 10 minutes | Cooldown timer + counter |
| OTP verify | 5 attempts | 10 minutes | Attempt counter + lock |
| Password login | 5 attempts | 15 minutes | Account lock warning |
| Password reset | 3 requests | 1 hour | Cooldown + toast |
| Registration | 3 requests | 1 hour | Cooldown + toast |

### 16.6 Analytics Events

| Event Name | Parameters | Trigger |
|---|---|---|
| `auth_page_view` | `page_id`, `referrer` | Page load |
| `auth_phone_submit` | `page_id`, `method` | Phone form submit |
| `auth_otp_sent` | `purpose`, `delivery_time_ms` | OTP sent successfully |
| `auth_otp_verify` | `purpose`, `attempts`, `success` | OTP verification |
| `auth_otp_resend` | `purpose`, `attempt_number` | OTP resend |
| `auth_login_success` | `method` (otp/password) | Successful login |
| `auth_login_failure` | `error_code`, `method` | Failed login |
| `auth_password_reset` | `success` | Password reset completion |
| `auth_session_expired` | `inactive_duration` | Session timeout |
| `auth_account_locked` | `lock_reason` | Account lock |

### 16.7 Loading Skeleton Pattern

```
┌─────────────────────────────────────────────────────────┐
│ ████  (Logo skeleton)                                   │
│                                                         │
│ ████████████████  (Title skeleton)                       │
│ ██████████████████████  (Subtitle skeleton)              │
│                                                         │
│ ┌─────────────────────────────────────────────────────┐ │
│ │ ████████████████████████████████████████████████    │ │
│ └─────────────────────────────────────────────────────┘ │
│                                                         │
│ ┌─────────────────────────────────────────────────────┐ │
│ │ ████████████████████████████████████████████████    │ │
│ └─────────────────────────────────────────────────────┘ │
│                                                         │
│ ████████████████████████████████████████████████████    │
└─────────────────────────────────────────────────────────┘
```

### 16.8 Error Boundary

All auth pages wrapped in `AuthErrorBoundary` component that catches:
- React rendering errors
- Unhandled promise rejections
- Network failures during page load

Fallback UI: Generic error page with retry button and support link.

### 16.9 Navigation Guards

| Guard | Condition | Action |
|---|---|---|
| Auth Required | No valid token | Redirect to `/auth/login?redirect={current}` |
| Guest Only | Valid token on auth pages | Redirect to dashboard |
| OTP Guard | No pending OTP session | Redirect to phone entry page |
| Reset Guard | No valid reset token | Redirect to `/auth/forgot-password` |
| 2FA Guard | No valid 2FA session | Redirect to appropriate auth page |

### 16.10 Performance Budget

| Resource | Budget |
|---|---|
| Total JS (auth pages) | ≤ 80KB gzipped |
| Total CSS (auth pages) | ≤ 20KB gzipped |
| Images (logo, illustrations) | ≤ 50KB (SVG preferred) |
| Fonts | ≤ 100KB (Arabic subset) |
| Time to First Byte | ≤ 200ms |
| First Contentful Paint | ≤ 1.2s |
| Largest Contentful Paint | ≤ 2.5s |
| Cumulative Layout Shift | ≤ 0.1 |
| First Input Delay | ≤ 100ms |
| Time to Interactive | ≤ 3.0s |

---

*End of Specification — 07-PORTAL-AUTH-SYSTEM-01.md*
