# Style Guide — YemenMart

## 1. Overview

Writing style guide for all UI copy across YemenMart platforms. Ensures consistent, clear, Arabic-first communication.

## 2. Tone of Voice

| Attribute | Description | Example |
|-----------|-------------|---------|
| Friendly | Warm, approachable language | "مرحباً بك في YemenMart" |
| Clear | Simple, direct communication | "تم تأكيد طلبك" |
| Helpful | Guide users, don't just inform | "يمكنك تتبع طلبك من هنا" |
| Professional | Trustworthy, reliable | "تمت معالجة الدفع بنجاح" |
| Concise | Short, to the point | "تم الحفظ" |

## 3. Language Rules

### Arabic Writing Standards

| Rule | Correct | Incorrect |
|------|---------|-----------|
| Use Modern Standard Arabic | "تم تأكيد الطلب" | "أوكي الطلب جاهز" |
| Avoid transliteration | "البريد الإلكتروني" | "إيميل" |
| Use formal punctuation | "،" (Arabic comma) | "," (English comma) |
| Use Arabic-Indic numerals | "١٥,٠٠٠ يمني" | "15,000 يمني" |
| Right-to-left text direction | `dir="rtl"` on container | No direction specified |

### Bilingual Content

| Scenario | Treatment |
|----------|-----------|
| Brand names | Keep English: "YemenMart" |
| Technical terms | Arabic with English in parentheses: "الatchewan (Cache)" |
| Phone numbers | Keep Latin digits: "+967 77 123 4567" |
| Email addresses | Keep Latin: "support@yemenmart.com" |
| URLs | Keep Latin: "yemenmart.com" |
| Currency | Arabic: "١٥,٠٠٠ يمني" or "YER 15,000" |

## 4. Button Labels

### Primary Actions

| English | Arabic | Usage |
|---------|--------|-------|
| Add to Cart | أضف إلى السلة | Product page |
| Buy Now | اشترِ الآن | Product page |
| Checkout | إتمام الطلب | Cart page |
| Place Order | تأكيد الطلب | Checkout |
| Pay Now | ادفع الآن | Payment |
| Save | حفظ | Forms |
| Submit | إرسال | Forms |
| Confirm | تأكيد | Actions |
| Cancel | إلغاء | Actions |
| Delete | حذف | Actions |

### Navigation

| English | Arabic | Usage |
|---------|--------|-------|
| Home | الرئيسية | Navigation |
| Products | المنتجات | Navigation |
| Categories | الأقسام | Navigation |
| Cart | السلة | Navigation |
| Account | حسابي | Navigation |
| Orders | طلباتي | Navigation |
| Wallet | المحفظة | Navigation |
| Settings | الإعدادات | Navigation |
| Back | رجوع | Navigation |
| Next | التالي | Pagination |
| Previous | السابق | Pagination |

### Status Messages

| English | Arabic | Usage |
|---------|--------|-------|
| Success | نجح | Status |
| Error | خطأ | Status |
| Loading | جاري التحميل | Status |
| Processing | جاري المعالجة | Status |
| Saved | تم الحفظ | Status |
| Deleted | تم الحذف | Status |
| Copied | تم النسخ | Status |

## 5. Error Messages

### Format

```
[What happened] + [Why it happened] + [What to do]
```

### Examples

| Scenario | Message |
|----------|---------|
| Network error | "حدث خطأ في الاتصال. تحقق من اتصالك بالإنترنت وحاول مرة أخرى." |
| Server error | "حدث خطأ في الخادم. يرجى المحاولة مرة أخرى لاحقاً." |
| Validation error | "يرجى تعبئة جميع الحقول المطلوبة." |
| Phone format | "يرجى إدخال رقم هاتف صحيح بالصيغة: +967XXXXXXXXX" |
| Password weak | "كلمة المرور يجب أن تحتوي على 8 أحرف على الأقل، مع حرف كبير ورقم." |
| OTP expired | "انتهت صلاحية كود التأكيد. يرجى طلب كود جديد." |
| OTP wrong | "كود التأكيد غير صحيح. متبقي {count} محاولات." |
| Insufficient balance | "رصيد المحفظة غير كافٍ. يرجى شحن المحفظة أولاً." |
| Item out of stock | "المنتج غير متوفر حالياً. سنوافيك عند التوفر." |
| Delivery failed | "فشل التوصيل. يرجى التواصل مع الدعم للمساعدة." |

## 6. Success Messages

| Scenario | Message |
|----------|---------|
| Order placed | "تم تأكيد طلبك بنجاح! رقم الطلب: #{orderId}" |
| Payment complete | "تم الدفع بنجاح. رصيد المحفظة: {balance} يمني" |
| Profile updated | "تم تحديث الملف الشخصي بنجاح." |
| Address saved | "تم حفظ العنوان بنجاح." |
| Review submitted | "شكراً لمراجعتك! ستظهر بعد المراجعة." |
| Password changed | "تم تغيير كلمة المرور بنجاح." |
| Item added to cart | "تمت إضافة المنتج إلى السلة." |

## 7. Confirmation Dialogs

### Format

```
[Title: Action being confirmed]
[Body: Consequence of the action]
[Confirm: Action verb in imperative]
[Cancel: Dismissive text]
```

### Examples

| Scenario | Title | Body | Confirm | Cancel |
|----------|-------|------|---------|--------|
| Delete product | "حذف المنتج" | "هل أنت متأكد من حذف هذا المنتج؟ لا يمكن التراجع عن هذا الإجراء." | "حذف" | "إلغاء" |
| Cancel order | "إلغاء الطلب" | "هل تريد إلغاء الطلب #{orderId}؟ سيتم استرداد المبلغ إلى محفظتك." | "نعم، إلغاء" | "الاحتفاظ بالطلب" |
| Logout | "تسجيل الخروج" | "هل تريد تسجيل الخروج من حسابك؟" | "تسجيل الخروج" | "إلغاء" |

## 8. Form Labels

### Input Labels

| Field | Label | Placeholder | Hint |
|-------|-------|-------------|------|
| Phone | رقم الهاتف | +967 77 123 4567 | "أدخل رقم الهاتف المسجل" |
| Password | كلمة المرور | — | "٨ أحرف على الأقل" |
| Full Name | الاسم الكامل | "محمد أحمد" | — |
| Email | البريد الإلكتروني | "user@example.com" | "اختياري" |
| Address | العنوان | "شارع..." | "العنوان التفصيلي" |
| City | المدينة | "صنعاء" | — |
| Notes | ملاحظات | "أي ملاحظات إضافية..." | "اختياري" |

### Validation Messages

| Field | Error | Success |
|-------|-------|---------|
| Phone | "رقم الهاتف غير صحيح" | ✓ |
| Password | "كلمة المرور قصيرة جداً" | ✓ |
| Email | "البريد الإلكتروني غير صحيح" | ✓ |
| Required | "هذا الحقل مطلوب" | — |
| Max length | "النص طويل جداً (حد أقصى {max} حرف)" | — |

## 9. Notifications

### Push Notification Format

```
[Title: Category/Source]
[Body: Concise message, max 65 chars]
[Action: What to do next]
```

### Examples

| Type | Title | Body |
|------|-------|------|
| Order update | "تحديث الطلب" | "طلبك #{orderId} تم شحنه" |
| Delivery | "جاهز للتوصيل" | "طلبك في الطريق. كود التوصيل: {code}" |
| Wallet | "شحن المحفظة" | "تم شحن محفظتك بمبلغ {amount} يمني" |
| Promotion | "عرض خاص" | "خصم {discount}% على جميع المنتجات" |
| Low stock | "Alert" | "المنتج {product} على وشك النفاد" |

## 10. Date & Time Format

| Context | Format | Example |
|---------|--------|---------|
| Short date | day/month/year | "١٣/٩/٢٠٢٦" |
| Long date | day month year | "١٣ سبتمبر ٢٠٢٦" |
| Time | hour:minute | "٢:٣٠ م" |
| Relative | X ago/after | "منذ ٥ دقائق" |
| Duration | Xh Xm | "ساعة و ٣٠ دقيقة" |

## 11. Currency Format

```typescript
function formatYER(amount: number): string {
  return new Intl.NumberFormat('ar-YE', {
    style: 'decimal',
    minimumFractionDigits: 0,
  }).format(amount) + ' يمني';
}

// Examples
formatYER(1500);     // "١٬٥٠٠ يمني"
formatYER(15000);    // "١٥٬٠٠٠ يمني"
formatYER(150000);   // "١٥٠٬٠٠٠ يمني"
formatYER(1500000);  // "١٬٥٠٠٬٠٠٠ يمني"
```

## 12. Accessibility in Copy

| Rule | Example |
|------|---------|
| Avoid jargon | "钱包" → "المحفظة" not "البورصة" |
| Use simple sentences | "تم تأكيد الطلب" not "تمت عملية تأكيد الطلب بنجاح" |
| Be specific | "٣ منتجات في السلة" not "لديك منتجات" |
| Use active voice | "تم الشحن" not "تم شحنه" |
| Avoid negatives | "أضف منتجاً" not "السلة فارغة" |

## 13. Related Files

| File | Description |
|------|-------------|
| `design-system.md` | Design tokens and components |
| `rtl-design.md` | RTL layout implementation |
| `accessibility.md` | WCAG 2.1 AA requirements |
| `05-frontend/component-library.md` | UI components |
| `22-glossary/terminology.md` | Arabic terminology glossary |

---

*Document Version: 1.0 | Last Updated: 2026-09-13 | Classification: Confidential*
