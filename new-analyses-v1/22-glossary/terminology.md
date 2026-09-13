# YemenMart Glossary & Terminology

## Overview

This glossary defines **50+ terms** used throughout the YemenMart platform documentation and implementation. Terms are organized alphabetically with Arabic translations where applicable.

---

## A

### Acceptance Criteria (معايير القبول)
Predefined conditions that a user story or feature must meet to be considered complete. YemenMart has 62 security acceptance criteria and 477 use case acceptance criteria.

### Actor (الشخصية)
A role played by a user or system that interacts with the platform. YemenMart has 7 actors: Customer, Merchant, Delivery Agent, Admin, Super Admin, Moderator, Support Agent.

### Admin Panel (لوحة التحكم)
The backend management interface for platform administrators. Module M23 handles system configuration, user management, and compliance reporting.

### Analytics Engine (محرك التحليلات)
Module M22 responsible for generating merchant dashboards, sales reports, and platform-wide analytics.

### API (واجهة برمجة التطبيقات)
Application Programming Interface. YemenMart uses RESTful APIs with versioning. All API responses must meet p95 < 200ms.

### Arabic-First (العربية أولاً)
Design principle where Arabic is the primary language and default locale. RTL layout is enforced across all UI components.

---

## B

### Bilingual (ثنائي اللغة)
Support for both Arabic and English. All content, UI elements, and documentation must be available in both languages.

### Block (الكتلة)
A high-level architectural grouping. YemenMart has 13 building blocks.

### BNPL (اشترِ الآن وادفع لاحقاً)
Buy Now Pay Later. **Explicitly prohibited** on YemenMart (Constraint 1).

### Branding Violations (انتهاكات العلامات التجارية)
Use of third-party brand names, logos, or trademarks without authorization. Prohibited by Constraint 18.

### Building Block (الكتلة البنائية)
See Block. The 13 blocks organize the 23 modules into logical groupings.

---

## C

### Cart (سلة المشتريات)
Module M11. Temporary storage for items a customer intends to purchase. Maximum 50 items, max 10 units per product.

### Cart Validation (التحقق من السلة)
Process ensuring cart contents meet platform constraints: item count (≤50), quantity per product (≤10), and order value range (500–5,000,000 YER).

### Category (الفئة)
Product classification. System defines 40+ service categories (Constraint 15). Categories require admin approval for creation/modification.

### Checkout (الدفع)
Module M12. Process of converting cart contents into an order, including address selection, wallet verification, and payment processing.

### CMS (نظام إدارة المحتوى)
Content Management System. Manages banners, pages, promotions, and static content.

### Compliance (الامتثال)
Adherence to regulatory requirements including ZATCA invoicing, 15% VAT, and 5-year data retention.

### Content Moderation (إدارة المحتوى)
Review process for product listings, reviews, and store content. Conducted by Moderators (A-06) to prevent branding violations.

### Constraint (قيود)
Non-negotiable platform rule. YemenMart has 26 constraints that must be honored in every module.

### Courier (السائق)
See Delivery Agent.

### CRUD (إنشاء قراءة تحديث حذف)
Create, Read, Update, Delete. Basic data operation pattern used across all modules.

### Customer (العميل)
End user who purchases products. Actor A-01. Authenticates via phone OTP. No KYC required.

---

## D

### Delivery Agent (وكيل التوصيل)
Actor A-03. Responsible for picking up packages from merchants and delivering to customers. Assigned via platform.

### Delivery Code (رمز التوصيل)
Unique code sent to customer via SMS/WhatsApp for delivery verification. Customer must enter this code to confirm receipt. Maximum 3 attempts before lockout.

### Delivery Code Lockout (قفل رمز التوصيل)
After 3 failed delivery code attempts, the customer account is locked for 24 hours. A support ticket is auto-generated.

### Double-Entry Bookkeeping (المحاسبة المزدوجة)
Financial recording method where every transaction has equal debits and credits. Enforced by the ledger engine (Constraint 22).

### DTO (كائن نقل البيانات)
Data Transfer Object. Used for input validation and API contract enforcement.

---

## E

### E2E Tests (اختبارات نهاية إلى نهاية)
End-to-End tests simulating complete user journeys. 49 E2E tests in the testing strategy (5% of total).

### Encryption at Rest (التشفير أثناء التخزين)
AES-256 encryption for data stored in databases and files.

### Encryption in Transit (التشفير أثناء النقل)
TLS 1.3 encryption for all data transmitted between client and server.

### Escrow (الضمان المالي)
Fund holding mechanism. Customer payments held for 7 days after delivery before release to merchant.

### Escrow Engine (محرك الضمان)
Module M16. Manages fund holding, release, and refund within the escrow period.

---

## F

### Forgot Password (نسيت كلمة المرور)
Password recovery flow using phone OTP only. No email-based recovery. No security questions.

### Fulfillment (التنفيذ)
Process of completing an order from merchant confirmation to customer delivery.

---

## G

### Gate (البوابة)
Quality checkpoint in the development pipeline. YemenMart has pre-commit, CI/CD, pre-production, and production gates.

### GPS Tracking (تتبع الموقع)
**Explicitly prohibited** on YemenMart (Constraint 10). Delivery status communicated through delivery codes only.

---

## H

### HSM (وحدة حماية الأجهزة)
Hardware Security Module. Used for encryption key management with 90-day rotation.

### HMAC (كود التحقق المُسوّق)
Hash-based Message Authentication Code. Used for JWT signature verification.

---

## I

### IDOR (التحديد غير المصرح به للموارد)
Insecure Direct Object Reference. Authorization vulnerability where user can access resources by manipulating IDs. Mitigated by ownership validation.

### i18n (التدويل)
Internationalization. Framework supporting Arabic (primary) and English (secondary) with RTL layout.

### Inventory (المخزون)
Module M06. Tracks product stock levels, reservations, and movements.

### Invoice (الفاتورة)
ZATCA-compliant document generated for every transaction. Must be retained for 5 years.

---

## J

### JWT (رمز الوصول JSON)
JSON Web Token. Used for session management with 24-hour expiry and HMAC signature.

---

## K

### k6 (أداة اختبار الأداء)
Load testing tool used for API performance testing. Targets: p95 < 200ms, throughput > 1000 req/s.

### KYC (اعرف عميلك)
Know Your Customer. Mandatory verification for merchants (Constraint 19). Includes identity documents and business information.

---

## L

### Ledger (دفتر الأستاذ)
Immutable financial record maintaining double-entry bookkeeping. All balance changes logged.

### Localization (توطين)
Adaptation of content for specific languages and regions. YemenMart localizes for Arabic (default) and English.

### Lockout (القفل)
Temporary account restriction after security threshold exceeded. Examples: OTP lockout (3 attempts), password lockout (5 attempts), delivery code lockout (3 attempts).

---

## M

### Master Order (الأمر الرئيسي)
Top-level order created by customer. Contains sub-orders for each merchant. Unique master order ID links all sub-orders.

### Merchant (التاجر)
See Vendor.

### Moderation (الإشراف)
Content review process conducted by Moderators (A-06) to ensure policy compliance.

### Module (الوحدة)
Logical grouping of related functionality. YemenMart has 23 modules across 13 blocks.

### MVP (الحد الأدنى القابل للتطبيق)
Minimum Viable Product. Initial release containing core features.

---

## N

### Notification Hub (مركز الإشعارات)
Module M21. Multi-channel notification system supporting SMS, WhatsApp, push notifications, and in-app messages.

---

## O

### OTP (كلمة مرور لمرة واحدة)
One-Time Password. Sent via SMS or WhatsApp for phone-based authentication. Expires after 5 minutes.

### Order (الأمر)
Customer purchase request. Progresses through 17 defined states. Split into master order and sub-orders per merchant.

### Order Lifecycle (دورة حياة الأمر)
The complete journey of an order through 17 states: PENDING → PAID → CONFIRMED → ... → COMPLETED or CANCELLED or REFUNDED.

---

## P

### Paywall (جدار الدفع)
Restriction requiring payment before accessing content. **Not used** on YemenMart.

### PII (معلومات شخصية قابلة للتحديد)
Personally Identifiable Information. Encrypted at rest, masked in logs.

### Platform Fee (رسوم المنصة)
Commission deducted from merchant settlement before escrow release.

### Playwright (أداة اختبار المتصفح)
Browser automation framework used for E2E testing. Supports Chromium, Firefox, and WebKit.

### Product (المنتج)
Item listed by merchant for sale. Each product belongs to a category and has inventory tracking.

### Promotion (الترويج)
Marketing campaign affecting product pricing. VAT calculated on discounted price (Constraint 20).

---

## Q

### QR Code (رمز الاستجابة السريع)
Quick Response code on ZATCA-compliant invoices for verification purposes.

### Quality Gate (بوابة الجودة)
See Gate.

---

## R

### RBAC (التحكم في الوصول القائم على الأدوار)
Role-Based Access Control. Permission system defining what each actor can access. 7 actors with distinct permission levels.

### Recommendation Engine (محرك التوصيات)
Module M10. Generates product suggestions based on browsing and purchase history.

### Regression Testing (اختبار التراجع)
Re-running existing tests to ensure new changes don't break existing functionality. Full regression: weekly, ~4 hours.

### Return (إرجاع)
Product return process. Must comply with merchant's `isReturnable` and `returnPeriodDays` policies.

### Return Policy (سياسة الإرجاع)
Merchant-defined settings: `isReturnable` (boolean) and `returnPeriodDays` (integer).

### RTL (من اليمين إلى اليسار)
Right-to-Left. Layout direction for Arabic text. Enforced as default across all UI components.

---

## S

### Session (الجلزة)
Authenticated user state maintained via JWT token. Expires after 24 hours of inactivity.

### SIM Swap (تبديل بطاقة SIM)
Attack where attacker takes over victim's phone number. Mitigated by device binding and velocity checks.

### SLA (اتفاقية مستوى الختم)
Service Level Agreement. Defines response/resolution times for different severity levels.

### SMS (رسالة نصية قصيرة)
Short Message Service. Primary channel for OTP delivery and critical notifications.

### Sub-Order (الأمر الفرعي)
Order segment containing items from a single merchant within a master order. Independent lifecycle tracking.

### Store Template (قالب المتجر)
Pre-designed storefront layout. 10+ templates available (Constraint 14).

### Storefront (واجهة المتجر)
Merchant's public-facing store page. URL format: yemenmart.com/store/{slug}.

### Super Admin (المدير الأعلى)
Actor A-05. Unrestricted system access. System bootstrap only.

---

## T

### Test Point (نقطة اختبار)
Individual test case within the testing strategy. YemenMart has 974 test points across 13 blocks.

### Testing Pyramid (هرم الاختبار)
Test distribution strategy: 80% unit, 15% integration, 5% E2E.

### Threat Model (نموذج التهديد)
Security analysis identifying 55 potential threats across 7 categories.

### TLS (طبقة المقابس الآمنة)
Transport Layer Security. Version 1.3 required for all data in transit.

### Transaction (المعاملة)
Wallet operation including funding, transfer, payment, or refund. All transactions follow double-entry bookkeeping.

---

## U

### UAT (اختبار قبول المستخدم)
User Acceptance Testing. Final validation by stakeholders before production release.

### Unit Test (اختبار الوحدة)
Test for individual function, method, or component. 779 unit tests (80% of total).

### Use Case (حالة استخدام)
Specific scenario describing actor interaction with the system. YemenMart has 477 use cases.

---

## V

### Validation (التحقق)
Input verification ensuring data meets format, range, and business rule requirements.

### Vendor (المزود)
Merchant who lists and sells products on the platform. Actor A-02. KYC mandatory.

### View (العرض)
Read-only access to data or resources.

---

## W

### Wallet (المحفظة)
Virtual currency storage for all platform transactions. Module M15. Wallet-only payments (Constraint 1). No card/BNPL support.

### Wallet Balance (رصيد المحفظة)
Amount of virtual currency available in a user's wallet. Displayed in real-time.

### WhatsApp (واتساب)
Messaging application used as alternative channel for OTP delivery and notifications.

---

## X

### XSS (برمجة نصوص المواقع عبر الموقع)
Cross-Site Scripting. Injection attack mitigated by input sanitization and CSP headers.

---

## Z

### ZATCA (هيئة الزكاة والضريبة والجمارك)
Zakat, Tax and Customs Authority. Saudi regulatory body. Invoices must be ZATCA-compliant (Constraint 21).

### Zero-Day (يوم صفر)
Previously unknown vulnerability. Mitigated by regular security audits and penetration testing.

---

## Acronym Reference

| Acronym | Full Term (EN) | Full Term (AR) |
|---------|----------------|----------------|
| AC | Acceptance Criteria | معايير القبول |
| API | Application Programming Interface | واجهة برمجة التطبيقات |
| BNPL | Buy Now Pay Later | اشترِ الآن وادفع لاحقاً |
| CMS | Content Management System | نظام إدارة المحتوى |
| CRUD | Create, Read, Update, Delete | إنشاء قراءة تحديث حذف |
| DTO | Data Transfer Object | كائن نقل البيانات |
| E2E | End-to-End | من النهاية إلى النهاية |
| HSM | Hardware Security Module | وحدة حماية الأجهزة |
| HMAC | Hash-based Message Authentication Code | كود التحقق المُسوّق |
| i18n | Internationalization | التدويل |
| IDOR | Insecure Direct Object Reference | التحديد غير المصرح به للموارد |
| JWT | JSON Web Token | رمز الوصول JSON |
| KYC | Know Your Customer | اعرف عميلك |
| MVP | Minimum Viable Product | الحد الأدنى القابل للتطبيق |
| OTP | One-Time Password | كلمة مرور لمرة واحدة |
| PII | Personally Identifiable Information | معلومات شخصية قابلة للتحديد |
| RBAC | Role-Based Access Control | التحكم في الوصول القائم على الأدوار |
| RTL | Right-to-Left | من اليمين إلى اليسار |
| SLA | Service Level Agreement | اتفاقية مستوى الختم |
| SMS | Short Message Service | رسالة نصية قصيرة |
| TLS | Transport Layer Security | طبقة المقابس الآمنة |
| UAT | User Acceptance Testing | اختبار قبول المستخدم |
| XSS | Cross-Site Scripting | برمجة نصوص المواقع عبر الموقع |
| YER | Yemeni Rial | ريال يمني |
| ZATCA | Zakat, Tax and Customs Authority | هيئة الزكاة والضريبة والجمارك |

---

*Document Version: 1.0 | Last Updated: 2026-09-13 | Classification: Confidential*
