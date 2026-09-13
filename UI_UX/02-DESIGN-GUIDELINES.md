# YemenMart Platform — Design Guidelines

> Version: 2.0  
> Last Updated: 2026-09-13  
> Status: Active

---

## Table of Contents

1. [RTL Layout Guidelines](#1-rtl-layout-guidelines)
2. [LTR Layout Guidelines](#2-ltr-layout-guidelines)
3. [Arabic UX Patterns](#3-arabic-ux-patterns)
4. [English UX Patterns](#4-english-ux-patterns)
5. [Typography Guidelines](#5-typography-guidelines)
6. [Layout Guidelines](#6-layout-guidelines)
7. [Spacing System](#7-spacing-system)
8. [Form Guidelines](#8-form-guidelines)
9. [Table Guidelines](#9-table-guidelines)
10. [Navigation Guidelines](#10-navigation-guidelines)
11. [Modal and Dialog Guidelines](#11-modal-and-dialog-guidelines)
12. [Feedback Guidelines](#12-feedback-guidelines)
13. [Accessibility Guidelines](#13-accessibility-guidelines)
14. [Responsive Design Guidelines](#14-responsive-design-guidelines)
15. [Mobile Behavior Guidelines](#15-mobile-behavior-guidelines)
16. [Touch Interaction Guidelines](#16-touch-interaction-guidelines)
17. [Localization Guidelines](#17-localization-guidelines)
18. [Date/Time Formatting](#18-datetime-formatting)
19. [Currency Formatting](#19-currency-formatting)
20. [Number Formatting](#20-number-formatting)

---

## 1. RTL Layout Guidelines

### 1.1 Core Principle

YemenMart is Arabic-first. The **default direction is RTL**. English is secondary and only applies when the user switches the interface language.

### 1.2 Logical Properties

Use CSS logical properties instead of physical properties. This ensures automatic mirroring when switching between RTL and LTR.

```css
/* AVOID physical properties */
margin-left: 16px;
padding-right: 12px;
border-left: 2px solid;
text-align: left;
float: right;

/* USE logical properties */
margin-inline-start: 16px;
padding-inline-end: 12px;
border-inline-start: 2px solid;
text-align: start;
float: inline-start;
```

**Required logical property mappings:**

| Physical (AVOID)         | Logical (USE)               |
|--------------------------|-----------------------------|
| `margin-left`            | `margin-inline-start`       |
| `margin-right`           | `margin-inline-end`         |
| `padding-left`           | `padding-inline-start`      |
| `padding-right`          | `padding-inline-end`        |
| `border-left`            | `border-inline-start`       |
| `border-right`           | `border-inline-end`         |
| `left`                   | `inset-inline-start`        |
| `right`                  | `inset-inline-end`          |
| `text-align: left`       | `text-align: start`         |
| `text-align: right`      | `text-align: end`           |
| `float: left`            | `float: inline-start`       |
| `float: right`           | `float: inline-end`         |

### 1.3 Icon Flipping

| Icon Type                        | RTL Behavior      |
|----------------------------------|--------------------|
| Directional arrows (← → ↑ ↓)   | **Flip horizontally** |
| Play/pause icons                | **Flip horizontally** |
| Checkmarks, close (×)           | **No flip**        |
| Search, home, cart              | **No flip**        |
| Chevron/back arrows             | **Flip horizontally** |
| Progress indicators (←→)        | **Flip horizontally** |
| Star ratings (★)               | **No flip**        |
| External link icon              | **Flip horizontally** |
| Upload/download arrows          | **Flip horizontally** |
| Back/forward buttons            | **Flip horizontally** |

**Tailwind approach:**

```html
<!-- Arrow that flips in RTL -->
<svg class="rtl:rotate-180 ltr:rotate-0">

<!-- Icon that never flips -->
<svg class="">

<!-- Back button -->
<button class="rtl:scale-x-[-1]">
```

### 1.4 Text Alignment

- Arabic text is **right-aligned** by default.
- Use `text-align: start` which resolves to `right` in RTL and `left` in LTR.
- Never hardcode `text-align: right` or `text-align: left` for body text.
- Numeric values may be center-aligned for readability in tables.

### 1.5 Page Layout Mirroring

```
RTL (Arabic default):                    LTR (English):
+---------------------------+           +---------------------------+
| [Logo]    [Nav]  [Icons]  |           | [Icons]  [Nav]    [Logo] |
+---------------------------+           +---------------------------+
| [Sidebar] | [Content]     |           | [Content] | [Sidebar]    |
|           |               |           |           |              |
+---------------------------+           +---------------------------+
```

- Sidebar moves from **right** (RTL) to **left** (LTR).
- Content moves from **left** (RTL) to **right** (LTR).
- The entire layout mirrors automatically when using logical properties.

---

## 2. LTR Layout Guidelines

### 2.1 When LTR Applies

- User switches language to English.
- System-generated emails/SMS with Latin content.
- Third-party integrations with fixed LTR layouts.

### 2.2 Transition from RTL

- All logical properties automatically resolve to LTR values.
- Icons that were flipped in RTL will now appear in their original orientation.
- Ensure `dir="ltr"` is set on the `<html>` element when English is active.

```html
<html lang="ar" dir="rtl"> <!-- Arabic -->
<html lang="en" dir="ltr"> <!-- English -->
```

### 2.3 Mixed Content Handling

When Arabic text appears in an LTR layout (or vice versa), use the `unicode-bidi` property:

```css
.mixed-content {
  unicode-bidi: embed;
  direction: inherit;
}
```

For standalone Arabic numbers within LTR text, no special handling is needed.

---

## 3. Arabic UX Patterns

### 3.1 Text Direction

- Paragraphs flow **right-to-left**.
- Line breaks occur at the start of the next line (right side).
- Text truncation (`text-overflow: ellipsis`) shows the **left** side of the text (end of string).

### 3.2 Number Handling

- Arabic-Indic numerals (٠١٢٣٤٥٦٧٨٩) should be used in Arabic mode.
- Western-Arabic numerals (0123456789) are acceptable for technical contexts.
- Use a formatter utility to convert based on locale:

```typescript
function formatNumber(num: number, locale: 'ar' | 'en'): string {
  return new Intl.NumberFormat(locale === 'ar' ? 'ar-YE' : 'en-US').format(num);
}
// Arabic: ١٬٢٣٤٬٥٦٧
// English: 1,234,567
```

### 3.3 Date Formatting

- Use Gregorian calendar by default.
- Optional Hijri calendar support for religious/commercial contexts.
- Arabic date format: `١٢ سبتمبر ٢٠٢٦` or `١٢/٠٩/٢٠٢٦`.
- Never use `mm/dd/yyyy` format. Use `dd/mm/yyyy` or `yyyy/mm/dd`.

### 3.4 Currency Display

- Amount appears **after** the number: `١٠٬٠٠٠ ر.ي`.
- Currency symbol is right-aligned after the value in Arabic.

### 3.5 Form Labels

- Labels appear **above** inputs (preferred) or to the **right** (inline).
- Placeholder text is in Arabic when Arabic mode is active.
- Field descriptions below inputs.

### 3.6 Time Display

- 12-hour format preferred: `٢:٣٠ م` (2:30 PM).
- 24-hour format acceptable: `14:30`.
- AM/PM labels in Arabic: `ص` (AM), `م` (PM).

### 3.7 Error Messages

- Write error messages in Arabic when Arabic mode is active.
- Error messages are concise and actionable.
- Error placement: directly below the input field.

```
❌ Incorrect:
"Error: The field is required"

✅ Correct:
"هذا الحقل مطلوب" (This field is required)
```

### 3.8 Greeting and Tone

- Use formal Arabic (فصحى) for system messages.
- Use friendly but professional tone for notifications.
- Welcome messages: `مرحبًا، [اسم المستخدم]`

---

## 4. English UX Patterns

### 4.1 Text Direction

- Paragraphs flow **left-to-right**.
- Truncation shows the **right** side of the text.

### 4.2 Number Handling

- Western-Arabic numerals only: `1,234,567.89`.
- Comma as thousands separator, period as decimal separator.

### 4.3 Date Formatting

- Use `MM/DD/YYYY` or `YYYY-MM-DD` (ISO 8601).
- Avoid `DD/MM/YYYY` in English to prevent confusion with US format.
- Prefer: `September 12, 2026` or `2026-09-12`.

### 4.4 Currency Display

- Amount appears **before** the currency: `$10,000` or `10,000 YER`.
- Space between number and currency symbol.

### 4.5 Form Labels

- Labels appear **above** inputs (preferred) or to the **left** (inline).
- Placeholder text in English.

### 4.6 Tone

- Professional and clear.
- Avoid jargon unless the audience is technical.
- Error messages: `This field is required`.

---

## 5. Typography Guidelines

### 5.1 Font Families

| Context     | Arabic Font Stack                                | English Font Stack              |
|-------------|--------------------------------------------------|----------------------------------|
| Body text   | Tajawal, Cairo, 'Noto Sans Arabic', sans-serif   | Inter, Roboto, sans-serif        |
| Headings    | Tajawal (Bold/Black), Cairo (Bold)               | Inter (Semibold/Bold)            |
| Monospace   | 'Noto Sans Arabic', monospace                    | 'Fira Code', monospace           |

### 5.2 Type Scale

| Token             | Size (px) | Size (rem) | Line Height | Use Case                    |
|-------------------|-----------|------------|-------------|-----------------------------|
| `text-xs`         | 12        | 0.75       | 16px        | Captions, helper text       |
| `text-sm`         | 14        | 0.875      | 20px        | Secondary text, labels      |
| `text-base`       | 16        | 1.0        | 24px        | Body text (default)         |
| `text-lg`         | 18        | 1.125      | 28px        | Large body, subheadings     |
| `text-xl`         | 20        | 1.25       | 28px        | Section headings            |
| `text-2xl`        | 24        | 1.5        | 32px        | Page headings               |
| `text-3xl`        | 30        | 1.875      | 36px        | Hero text, feature headings |
| `text-4xl`        | 36        | 2.25       | 40px        | Display headings            |

### 5.3 Arabic-Specific Typography

- Arabic text requires **larger line height** than English for readability: use `leading-relaxed` (1.625) or `leading-loose` (2) for Arabic body text.
- Arabic characters have descenders and ascenders that require more vertical space.
- Avoid tight letter-spacing (`tracking-tight`) on Arabic text.
- Use `tracking-normal` or `tracking-wide` for Arabic.

```css
/* Arabic body text */
font-family: 'Tajawal', 'Cairo', 'Noto Sans Arabic', sans-serif;
font-size: 1rem;
line-height: 1.75;
letter-spacing: normal;

/* English body text */
font-family: 'Inter', 'Roboto', sans-serif;
font-size: 1rem;
line-height: 1.5;
letter-spacing: normal;
```

### 5.4 Font Loading Strategy

```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Tajawal:wght@400;500;700;800&family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
```

- Load both Arabic and English fonts upfront.
- Use `font-display: swap` to prevent FOIT.
- Subset Tajawal to Arabic + Latin if file size is a concern.

### 5.5 Mixed Content

When Arabic and English appear together:

```html
<span class="font-arabic">السعر: </span>
<span class="font-english">1,000</span>
<span class="font-arabic"> ر.ي</span>
```

The number should render in the Latin numeral style even in Arabic text.

---

## 6. Layout Guidelines

### 6.1 Grid System

Use a **12-column responsive grid** based on Tailwind's grid utilities.

```html
<div class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-4">
  <!-- Grid items -->
</div>
```

| Breakpoint | Width      | Columns | Margins | Gutter |
|------------|------------|---------|---------|--------|
| xs         | ≥320px     | 1       | 16px    | 16px   |
| sm         | ≥640px     | 2       | 16px    | 16px   |
| md         | ≥768px     | 3       | 24px    | 24px   |
| lg         | ≥1024px    | 4       | 32px    | 32px   |
| xl         | ≥1280px    | 4       | 40px    | 32px   |
| 2xl        | ≥1536px    | 4       | auto    | 32px   |

### 6.2 Page Width

- **Mobile**: 100% width, 16px padding on each side.
- **Tablet**: Max 768px content width.
- **Desktop**: Max 1200px content width.
- **Wide**: Max 1400px content width.

```html
<main class="mx-auto max-w-7xl px-4 sm:px-6 lg:px-8">
  <!-- Page content -->
</main>
```

### 6.3 Container Behavior

```css
.container {
  width: 100%;
  margin-inline: auto;
  padding-inline: 1rem;
}

@media (min-width: 640px)  { .container { max-width: 640px;  } }
@media (min-width: 768px)  { .container { max-width: 768px;  } }
@media (min-width: 1024px) { .container { max-width: 1024px; } }
@media (min-width: 1280px) { .container { max-width: 1280px; } }
```

### 6.4 Content Hierarchy

```
+--------------------------------------------------+
| HEADER (fixed, 56px height mobile, 64px desktop) |
+--------------------------------------------------+
| NAVIGATION BAR (optional, 48px)                  |
+--------------------------------------------------+
|                                                  |
|  MAIN CONTENT AREA                               |
|  - Page title                                    |
|  - Breadcrumbs (optional)                        |
|  - Content blocks                                |
|                                                  |
+--------------------------------------------------+
| FOOTER                                           |
+--------------------------------------------------+
```

---

## 7. Spacing System

### 7.1 Spacing Scale

Based on a **4px base unit**:

| Token   | Value  | px   | Use Case                          |
|---------|--------|------|-----------------------------------|
| `p-0`   | 0      | 0    | Reset                             |
| `p-0.5` | 0.125  | 2    | Tight internal spacing            |
| `p-1`   | 0.25   | 4    | Minimum touch gap                 |
| `p-1.5` | 0.375  | 6    | Small element spacing             |
| `p-2`   | 0.5    | 8    | Compact element spacing           |
| `p-3`   | 0.75   | 12   | Card internal padding             |
| `p-4`   | 1      | 16   | Standard padding, mobile margins  |
| `p-5`   | 1.25   | 20   | Section internal padding          |
| `p-6`   | 1.5    | 24   | Medium section spacing            |
| `p-8`   | 2      | 32   | Large section spacing             |
| `p-10`  | 2.5    | 40   | Section margins                   |
| `p-12`  | 3      | 48   | Large section margins             |
| `p-16`  | 4      | 64   | Hero spacing                      |

### 7.2 Grid Gaps

| Context          | Gap (mobile) | Gap (tablet) | Gap (desktop) |
|------------------|--------------|--------------|---------------|
| Product grid     | 12px         | 16px         | 24px          |
| Form fields      | 16px         | 16px         | 20px          |
| Card grid        | 16px         | 24px         | 32px          |
| Navigation items | 4px          | 8px          | 8px           |
| Table rows       | 0            | 0            | 0             |

### 7.3 Component Spacing

- **Between sections**: 32px (mobile) → 48px (desktop).
- **Between related elements**: 8px or 12px.
- **Between independent sections**: 24px or 32px.
- **Card padding**: 16px (mobile) → 24px (desktop).

### 7.4 Responsive Spacing

Use Tailwind's responsive prefixes to adjust spacing:

```html
<div class="p-4 md:p-6 lg:p-8">
  <!-- Content with responsive padding -->
</div>

<section class="mb-8 md:mb-12 lg:mb-16">
  <!-- Section with responsive bottom margin -->
</section>
```

---

## 8. Form Guidelines

### 8.1 Input Design

```html
<div class="space-y-1">
  <label for="phone" class="block text-sm font-medium text-gray-700">
    رقم الهاتف
  </label>
  <input
    id="phone"
    type="tel"
    dir="ltr"
    class="block w-full rounded-lg border border-gray-300 px-3 py-2 text-base
           shadow-sm transition-colors
           placeholder:text-gray-400
           focus:border-navy-800 focus:outline-none focus:ring-2 focus:ring-navy-800/20
           disabled:cursor-not-allowed disabled:bg-gray-50 disabled:text-gray-500"
    placeholder="77X XXX XXXX"
  />
  <p class="text-sm text-gray-500">أدخل رقم هاتفك المحمول</p>
</div>
```

### 8.2 Input States

| State        | Visual Treatment                                      |
|--------------|-------------------------------------------------------|
| Default      | Gray border (`border-gray-300`), white background     |
| Focus        | Blue border (`border-navy-800`), blue ring            |
| Error        | Red border (`border-red-500`), red ring               |
| Success      | Green border (`border-green-500`)                     |
| Disabled     | Gray background (`bg-gray-50`), gray text             |
| Read-only    | Light gray background, no border change               |

### 8.3 SMS-Only Authentication Input

Phone number input should be LTR regardless of page direction:

```html
<input
  type="tel"
  dir="ltr"
  inputmode="numeric"
  pattern="[0-9]{9,15}"
  placeholder="77X XXX XXXX"
/>
```

### 8.4 Validation Messages

Place validation messages directly below the input:

```html
<!-- Error state -->
<div>
  <input class="border-red-500 ..." />
  <p class="mt-1 text-sm text-red-600" role="alert">
    رقم الهاتف غير صحيح
  </p>
</div>

<!-- Success state -->
<div>
  <input class="border-green-500 ..." />
  <p class="mt-1 text-sm text-green-600">
    ✓ تم التحقق من الرقم
  </p>
</div>
```

### 8.5 Input Types

| Field           | `type`    | `inputmode` | `dir`  | Notes                      |
|-----------------|-----------|-------------|--------|----------------------------|
| Phone number    | `tel`     | `numeric`   | `ltr`  | Always LTR                 |
| SMS OTP code    | `text`    | `numeric`   | `ltr`  | 4-6 digits, LTR            |
| Password        | `password`| `text`      | `ltr`  | LTR for passwords          |
| Name (Arabic)   | `text`    | `text`      | `rtl`  | RTL default                |
| Email           | `email`   | `email`     | `ltr`  | LTR always                 |
| Amount/Price    | `number`  | `decimal`   | `ltr`  | LTR, 2 decimal places      |
| Search          | `search`  | `text`      | `auto` | Follows page direction     |
| Date            | `date`    | `text`      | `auto` | Consider custom picker     |

### 8.6 Form Layout

```html
<form class="space-y-6">
  <div>
    <label class="block text-sm font-medium text-gray-700 mb-1">...</label>
    <input class="block w-full ..." />
  </div>
  <div>
    <label class="block text-sm font-medium text-gray-700 mb-1">...</label>
    <input class="block w-full ..." />
  </div>
  <div class="flex flex-col-reverse sm:flex-row sm:justify-end gap-3">
    <button type="button" class="btn-secondary">إلغاء</button>
    <button type="submit" class="btn-primary">حفظ</button>
  </div>
</form>
```

### 8.7 Required Fields

- Mark required fields with a red asterisk (`*`) **after** the label text in Arabic.
- In Arabic: `رقم الهاتف *`
- Never rely solely on color to indicate required fields.

### 8.8 Form Submission

- Disable the submit button during submission to prevent double-clicks.
- Show a loading spinner inside the button.
- Clear error messages when the user starts typing.
- Preserve form values on validation errors.

---

## 9. Table Guidelines

### 9.1 Column Order

**RTL (Arabic):**

| # | آخر تحديث | الحالة       | المبلغ      | المنتج        | #
|--|-----------|-------------|-------------|--------------|--
| 1 | 12/09     | قيد التنفيذ | ٥٬٠٠٠ ر.ي  | آيفون 15     | 1 |

- Action column is on the **left** (end) in RTL.
- Data columns flow right-to-left.

**LTR (English):**

| # | Product   | Amount     | Status      | Last Update | #
|--|-----------|------------|-------------|-------------|--
| 1 | iPhone 15 | 5,000 YER  | Processing  | 12/09       | 1 |

- Action column is on the **right** (end) in LTR.

### 9.2 Column Alignment

| Column Type        | Alignment          | Notes                      |
|--------------------|--------------------|-----------------------------|
| Text (Arabic)      | `text-align: start`| Right in RTL, left in LTR   |
| Text (English)     | `text-align: start`| Left in RTL, right in LTR   |
| Numbers            | `text-align: end`  | Consistent numeric alignment|
| Currency           | `text-align: end`  | Align decimal points         |
| Dates              | `text-align: start`| Follows text direction       |
| Status badges      | `text-align: center` | Centered                  |
| Action buttons     | `text-align: end`  | Always at the end            |
| Checkbox column    | `text-align: center` | Centered                 |

### 9.3 Responsive Tables

**Mobile-first approach**: Convert tables to cards on small screens.

```html
<!-- Desktop table -->
<table class="hidden md:table w-full text-sm">
  <thead>...</thead>
  <tbody>...</tbody>
</table>

<!-- Mobile card view -->
<div class="md:hidden space-y-4">
  <div class="rounded-lg border p-4">
    <div class="flex justify-between items-start">
      <span class="font-medium">Product Name</span>
      <span class="text-sm text-gray-500">12/09/2026</span>
    </div>
    <div class="mt-2 space-y-1">
      <div class="flex justify-between">
        <span class="text-gray-500">Amount:</span>
        <span>5,000 YER</span>
      </div>
      <div class="flex justify-between">
        <span class="text-gray-500">Status:</span>
        <span class="inline-flex items-center rounded-full bg-yellow-100 px-2 py-0.5 text-xs">Processing</span>
      </div>
    </div>
  </div>
</div>
```

### 9.4 Pagination

- Pagination controls appear at the **bottom** of the table.
- In RTL: Previous arrow points **right**, Next arrow points **left**.
- Show page numbers: `1 2 3 ... 10`.
- Include "Show per page" selector.
- Current page is highlighted.

```html
<nav class="flex items-center justify-between border-t border-gray-200 px-4 py-3">
  <span class="text-sm text-gray-500">عرض 1-10 من 50</span>
  <div class="flex gap-1">
    <button class="px-3 py-1 rounded border text-sm">السابق</button>
    <button class="px-3 py-1 rounded border text-sm bg-navy-800 text-white">1</button>
    <button class="px-3 py-1 rounded border text-sm">2</button>
    <button class="px-3 py-1 rounded border text-sm">التالي</button>
  </div>
</nav>
```

### 9.5 Table Sorting

- Sortable columns show an arrow icon.
- Arrow direction is flipped in RTL.
- Active sort column is highlighted.
- Show ascending/descending toggle on click.

---

## 10. Navigation Guidelines

### 10.1 Header / Top Bar

**Mobile (≤768px):**

```
+-----------------------------------------------+
| ☰  [Logo]  YemenMart              🔔  👤  🛒 |
+-----------------------------------------------+
```

- Hamburger menu on the **right** in RTL (left in LTR).
- Logo centered or at the start.
- Icons grouped at the **left** in RTL (right in LTR).

**Desktop (≥1024px):**

```
+---------------------------------------------------------------+
| [Logo]  الرئيسية  التصنيفات  الطلبات  المفضلة    🔔 👤 🛒  |
+---------------------------------------------------------------+
```

- Horizontal navigation links.
- Icons with optional labels.

### 10.2 Sidebar Navigation

**RTL Desktop:**

```
+----------+----------------------------------+
|          |                                  |
| الرئيسية |                                  |
| -------- |         MAIN CONTENT             |
| التصنيفات |                                  |
| -------- |                                  |
| الطلبات   |                                  |
| -------- |                                  |
| الإعدادات |                                  |
|          |                                  |
+----------+----------------------------------+
```

- Sidebar on the **right** in RTL.
- Active item highlighted with blue accent and start border.
- Collapsible on mobile (drawer).

### 10.3 Bottom Navigation (Mobile)

**RTL:**

```
+-----------------------------------------------+
|  🏠     📦     ➕     🛒     👤              |
| الرئيسية الطلبات  إضافة  السلة   حسابي      |
+-----------------------------------------------+
```

- 4-5 items maximum.
- Active item highlighted (color + label).
- Center "Add" button is larger/emphasized for vendors.
- Fixed at the bottom of the screen.
- Hidden when keyboard is open.

### 10.4 Breadcrumbs

```html
<nav aria-label="Breadcrumb" dir="rtl">
  <ol class="flex items-center gap-2 text-sm text-gray-500">
    <li><a href="/" class="hover:text-navy-900">الرئيسية</a></li>
    <li aria-hidden="true">
      <svg class="rtl:rotate-180 h-4 w-4"><!-- chevron --></svg>
    </li>
    <li><a href="/category" class="hover:text-navy-900">الإلكترونيات</a></li>
    <li aria-hidden="true">
      <svg class="rtl:rotate-180 h-4 w-4"><!-- chevron --></svg>
    </li>
    <li aria-current="page" class="font-medium text-gray-900">هواتف</li>
  </ol>
</nav>
```

- Chevron separators flip in RTL.
- Current page is not a link.

### 10.5 Tab Navigation

```html
<div class="border-b border-gray-200">
  <nav class="flex gap-8" aria-label="Tabs">
    <button class="border-b-2 border-navy-800 py-3 text-sm font-medium text-navy-900">
      المنتجات
    </button>
    <button class="border-b-2 border-transparent py-3 text-sm font-medium text-gray-500 hover:text-gray-700">
      التقييمات
    </button>
  </nav>
</div>
```

---

## 11. Modal and Dialog Guidelines

### 11.1 Modal Structure

```html
<!-- Backdrop -->
<div class="fixed inset-0 z-50 bg-black/50" aria-hidden="true"></div>

<!-- Modal panel -->
<div class="fixed inset-0 z-50 flex items-center justify-center p-4">
  <div
    role="dialog"
    aria-modal="true"
    aria-labelledby="modal-title"
    class="w-full max-w-md rounded-xl bg-white p-6 shadow-xl"
  >
    <div class="flex items-center justify-between mb-4">
      <h2 id="modal-title" class="text-lg font-bold">تأكيد الحذف</h2>
      <button
        aria-label="إغلاق"
        class="rounded-lg p-1 hover:bg-gray-100"
      >
        <svg class="h-5 w-5"><!-- X icon --></svg>
      </button>
    </div>
    <p class="text-gray-600 mb-6">هل أنت متأكد من حذف هذا المنتج؟</p>
    <div class="flex flex-col-reverse sm:flex-row sm:justify-end gap-3">
      <button class="btn-secondary">إلغاء</button>
      <button class="btn-danger">حذف</button>
    </div>
  </div>
</div>
```

### 11.2 Modal Behavior

- Focus is trapped inside the modal when open.
- `Escape` key closes the modal.
- Clicking the backdrop closes the modal.
- Scroll is locked on the body when modal is open.
- On mobile, modals slide up from the bottom (sheet pattern).
- Maximum width: 480px on mobile, 560px on tablet, 640px on desktop.

### 11.3 Confirmation Dialogs

- Destructive actions require confirmation.
- The destructive button is **red** and placed on the **left** (end) in RTL.
- Clear, concise message explaining the consequence.
- Include "Cancel" and "Confirm" (or action name) buttons.

### 11.4 Full-Screen Dialogs (Mobile)

On mobile, use full-screen dialogs for complex flows (checkout, forms):

```html
<div class="fixed inset-0 z-50 bg-white flex flex-col">
  <header class="flex items-center gap-4 border-b px-4 py-3">
    <button aria-label="رجوع"><!-- Back arrow --></button>
    <h2 class="text-lg font-bold">إتمام الطلب</h2>
  </header>
  <main class="flex-1 overflow-y-auto p-4">
    <!-- Content -->
  </main>
</div>
```

---

## 12. Feedback Guidelines

### 12.1 Toast Notifications

```html
<!-- Success toast -->
<div
  role="status"
  aria-live="polite"
  class="fixed bottom-4 left-4 right-4 z-50 flex items-center gap-3 rounded-lg bg-green-50 p-4 shadow-lg border border-green-200 sm:left-auto sm:right-4 sm:max-w-sm"
>
  <svg class="h-5 w-5 text-green-500"><!-- check circle --></svg>
  <p class="text-sm text-green-800">تمت إضافة المنتج إلى السلة بنجاح</p>
  <button class="ms-auto text-green-500 hover:text-green-700" aria-label="إغلاق">
    <svg class="h-4 w-4"><!-- X --></svg>
  </button>
</div>
```

**Toast types:**

| Type    | Icon       | Background    | Text Color    | Border Color  |
|---------|------------|---------------|---------------|---------------|
| Success | ✓ check    | `green-50`    | `green-800`   | `green-200`   |
| Error   | ✕ cross    | `red-50`      | `red-800`     | `red-200`     |
| Warning | ⚠ triangle | `yellow-50`   | `yellow-800`  | `yellow-200`  |
| Info    | ℹ circle   | `blue-50`     | `blue-800`    | `blue-200`    |

### 12.2 Toast Behavior

- Auto-dismiss after **5 seconds** for success/info.
- Error toasts stay until manually dismissed.
- Maximum 3 toasts visible at once.
- Newest toasts appear at the bottom in RTL, at the top in LTR (or consistent positioning).
- Toasts stack vertically with 8px gap.

### 12.3 Inline Alerts

```html
<!-- Warning alert -->
<div class="rounded-lg border border-yellow-200 bg-yellow-50 p-4" role="alert">
  <div class="flex gap-3">
    <svg class="h-5 w-5 text-yellow-500 flex-shrink-0"><!-- warning icon --></svg>
    <div>
      <h3 class="text-sm font-medium text-yellow-800">تنبيه</h3>
      <p class="mt-1 text-sm text-yellow-700">رصيد محفظتك أقل من ١٠٠ ر.ي</p>
    </div>
  </div>
</div>
```

### 12.4 Loading States

- **Skeleton screens** for content loading (preferred).
- **Spinners** for button actions.
- **Progress bars** for file uploads or multi-step processes.

```html
<!-- Skeleton loading -->
<div class="animate-pulse space-y-4">
  <div class="h-4 bg-gray-200 rounded w-3/4"></div>
  <div class="h-4 bg-gray-200 rounded w-1/2"></div>
  <div class="h-32 bg-gray-200 rounded"></div>
</div>
```

### 12.5 Empty States

```html
<div class="flex flex-col items-center justify-center py-12 text-center">
  <svg class="h-16 w-16 text-gray-300 mb-4"><!-- empty cart icon --></svg>
  <h3 class="text-lg font-medium text-gray-900 mb-2">السلة فارغة</h3>
  <p class="text-sm text-gray-500 mb-4">لم تقم بإضافة أي منتجات بعد</p>
  <a href="/products" class="btn-primary">تصفح المنتجات</a>
</div>
```

---

## 13. Accessibility Guidelines

### 13.1 WCAG 2.1 AA Target

All UI components must meet **WCAG 2.1 Level AA** compliance.

### 13.2 Color Contrast

| Element             | Minimum Ratio | Required                |
|---------------------|---------------|--------------------------|
| Normal text (<18px) | 4.5:1         | AA                       |
| Large text (≥18px bold or ≥24px) | 3:1 | AA                  |
| UI components       | 3:1           | AA                       |
| Focus indicators    | 3:1           | AA                       |

**Contrast checks for common combinations:**

| Foreground       | Background    | Ratio   | Pass? |
|------------------|---------------|---------|-------|
| `gray-900`       | `white`       | 15.4:1  | ✅    |
| `gray-500`       | `white`       | 4.6:1   | ✅    |
| `gray-400`       | `white`       | 3.0:1   | ❌ (large text only) |
| `navy-900`       | `white`       | 5.0:1   | ✅    |
| `red-600`        | `white`       | 4.5:1   | ✅    |
| `white`          | `navy-900`    | 5.0:1   | ✅    |

### 13.3 Keyboard Navigation

- All interactive elements must be focusable.
- Focus order follows visual/reading order.
- **Tab** moves forward, **Shift+Tab** moves backward.
- **Enter** or **Space** activates buttons and links.
- **Escape** closes modals, dropdowns, and popovers.
- Focus visible indicator: `ring-2 ring-navy-800 ring-offset-2`.

```css
/* Custom focus style */
:focus-visible {
  outline: 2px solid #1B2A4A;
  outline-offset: 2px;
}

/* Remove default outline for mouse users */
:focus:not(:focus-visible) {
  outline: none;
}
```

### 13.4 Screen Reader Support

- All images have descriptive `alt` text.
- Icon-only buttons have `aria-label`.
- Form inputs have associated `<label>` elements.
- ARIA landmarks: `<main>`, `<nav>`, `<header>`, `<footer>`.
- Dynamic content uses `aria-live` regions.
- Status messages use `role="status"` or `role="alert"`.

```html
<!-- Icon button with aria-label -->
<button aria-label="حذف المنتج" class="btn-icon">
  <svg class="h-5 w-5"><!-- trash icon --></svg>
</button>

<!-- Live region for dynamic updates -->
<div aria-live="polite" class="sr-only">
  تم تحديث السلة: ٣ منتجات، المجموع ١٥٬٠٠٠ ر.ي
</div>
```

### 13.5 Skip Links

```html
<body>
  <a
    href="#main-content"
    class="sr-only focus:not-sr-only focus:absolute focus:top-4 focus:start-4 focus:z-50 focus:rounded-lg focus:bg-navy-900 focus:px-4 focus:py-2 focus:text-white"
  >
    تخطي إلى المحتوى الرئيسي
  </a>
  <!-- Header -->
  <main id="main-content">...</main>
</body>
```

### 13.6 Form Accessibility

- Every input has a visible `<label>`.
- Error messages are associated with inputs via `aria-describedby`.
- Required fields use `aria-required="true"`.
- Input purposes are declared via `autocomplete` attributes.

```html
<div>
  <label for="phone">رقم الهاتف <span class="text-red-500" aria-hidden="true">*</span></label>
  <input
    id="phone"
    type="tel"
    autocomplete="tel"
    aria-required="true"
    aria-describedby="phone-error"
  />
  <p id="phone-error" class="text-sm text-red-600" role="alert">
    رقم الهاتف مطلوب
  </p>
</div>
```

### 13.7 Reduced Motion

```css
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

---

## 14. Responsive Design Guidelines

### 14.1 Mobile-First Approach

All styles start from mobile and scale up:

```css
/* Base = mobile */
.card { padding: 16px; }

/* sm = tablet */
@media (min-width: 640px) { .card { padding: 20px; } }

/* md = desktop */
@media (min-width: 768px) { .card { padding: 24px; } }
```

### 14.2 Breakpoint Behavior

| Breakpoint | Width   | Layout Changes                                              |
|------------|---------|-------------------------------------------------------------|
| xs         | ≥320px  | Single column, stacked elements, bottom nav                 |
| sm         | ≥640px  | 2-column grids, side-by-side elements                       |
| md         | ≥768px  | 3-column grids, sidebar may appear, tables visible          |
| lg         | ≥1024px | Full desktop layout, sidebar always visible, horizontal nav |
| xl         | ≥1280px | Wider content area, more columns in grids                   |
| 2xl        | ≥1536px | Max-width containers, additional padding                    |

### 14.3 Component Behavior by Breakpoint

| Component        | Mobile (xs-sm)          | Tablet (md)            | Desktop (lg+)             |
|------------------|-------------------------|------------------------|---------------------------|
| Navigation       | Bottom nav              | Bottom nav or sidebar  | Top nav + sidebar         |
| Product grid     | 1-2 columns             | 2-3 columns            | 3-4 columns               |
| Cart sidebar     | Full screen             | Slide-in panel         | Slide-in panel            |
| Table            | Card view               | Responsive table       | Full table                |
| Modal            | Full screen / bottom    | Centered               | Centered                  |
| Header           | Compact (56px)          | Standard (64px)        | Standard (64px)           |
| Sidebar          | Hidden (drawer)         | Toggleable             | Always visible            |
| Search bar       | Expandable (icon only)  | Visible                | Visible with filters      |

### 14.4 Container Queries (Future Enhancement)

```css
.card-container {
  container-type: inline-size;
}

@container (min-width: 400px) {
  .card {
    display: flex;
    gap: 16px;
  }
}
```

---

## 15. Mobile Behavior Guidelines

### 15.1 Touch Targets

- Minimum touch target size: **44px × 44px** (WCAG 2.5.8).
- Preferred touch target size: **48px × 48px**.
- Spacing between adjacent touch targets: **8px minimum**.

```html
<!-- Minimum touch target -->
<button class="h-11 min-w-[44px] px-4">
  سلة المشتريات
</button>

<!-- Icon-only button -->
<button class="h-12 w-12 flex items-center justify-center rounded-full">
  <svg class="h-6 w-6"><!-- icon --></svg>
</button>
```

### 15.2 Bottom Navigation

- Fixed to the bottom of the viewport.
- Height: 56px + safe area (for devices with home indicator).
- Safe area padding: `pb-[env(safe-area-inset-bottom)]`.
- 4-5 items maximum.
- Active state: filled icon + label color change.
- Inactive state: outline icon + gray label.

```html
<nav class="fixed bottom-0 inset-x-0 z-40 bg-white border-t border-gray-200 pb-[env(safe-area-inset-bottom)]">
  <div class="flex justify-around py-2">
    <a href="/" class="flex flex-col items-center gap-0.5 text-navy-900">
      <svg class="h-6 w-6"><!-- filled home --></svg>
      <span class="text-xs font-medium">الرئيسية</span>
    </a>
    <a href="/categories" class="flex flex-col items-center gap-0.5 text-gray-500">
      <svg class="h-6 w-6"><!-- outline grid --></svg>
      <span class="text-xs">التصنيفات</span>
    </a>
    <a href="/add" class="flex flex-col items-center gap-0.5 text-gray-500">
      <div class="h-12 w-12 -mt-6 flex items-center justify-center rounded-full bg-navy-900 text-white shadow-lg">
        <svg class="h-6 w-6"><!-- plus --></svg>
      </div>
    </a>
    <a href="/cart" class="relative flex flex-col items-center gap-0.5 text-gray-500">
      <svg class="h-6 w-6"><!-- cart --></svg>
      <span class="text-xs">السلة</span>
      <span class="absolute -top-1 -end-1 h-4 w-4 rounded-full bg-red-500 text-[10px] text-white flex items-center justify-center">3</span>
    </a>
    <a href="/profile" class="flex flex-col items-center gap-0.5 text-gray-500">
      <svg class="h-6 w-6"><!-- outline user --></svg>
      <span class="text-xs">حسابي</span>
    </a>
  </div>
</nav>
```

### 15.3 Swipe Gestures

- Swipe left/right on product cards for quick actions (add to cart, favorite).
- Swipe down to refresh (pull-to-refresh).
- Swipe up on bottom sheet to expand.
- Ensure swipe gestures don't conflict with browser back/forward navigation.

### 15.4 Scroll Behavior

- Use `overflow-y: auto` on scrollable areas, not `overflow: hidden`.
- Prevent background scroll when modal/drawer is open.
- Use `overscroll-behavior: contain` to prevent scroll chaining.
- Consider sticky headers: `sticky top-0 z-30`.

---

## 16. Touch Interaction Guidelines

### 16.1 Gesture Support

| Gesture              | Action                          | Context              |
|----------------------|--------------------------------|----------------------|
| Tap                  | Select / Activate              | All interactive      |
| Long press           | Context menu / Edit            | Products, orders     |
| Swipe left           | Quick action (delete/archive)  | List items           |
| Swipe right          | Quick action (favorite/bookmark)| List items          |
| Swipe up             | Expand bottom sheet            | Bottom sheets        |
| Swipe down           | Collapse / Pull-to-refresh     | Sheets / Lists       |
| Pinch                | Zoom image                     | Product images       |
| Double tap           | Zoom toggle                    | Product images       |
| Scroll               | Navigate content               | All pages            |

### 16.2 Haptic Feedback

- Use `navigator.vibrate()` for:
  - Successful actions (short pulse: 50ms).
  - Error feedback (double pulse: 100ms, 50ms).
  - Long-press activation (medium pulse: 100ms).

```typescript
function hapticFeedback(type: 'success' | 'error' | 'tap') {
  if (!navigator.vibrate) return;
  switch (type) {
    case 'success': navigator.vibrate(50); break;
    case 'error':   navigator.vibrate([100, 50, 100]); break;
    case 'tap':     navigator.vibrate(10); break;
  }
}
```

### 16.3 Pull-to-Refresh

```html
<main
  class="overflow-y-auto"
  style="overscroll-behavior-y: contain;"
>
  <!-- Pull indicator -->
  <div class="flex justify-center py-4" id="pull-indicator" hidden>
    <svg class="h-6 w-6 animate-spin text-navy-800"><!-- spinner --></svg>
  </div>
  <!-- Content -->
</main>
```

### 16.4 Swipe-to-Delete

```html
<div class="relative overflow-hidden">
  <!-- Action background -->
  <div class="absolute inset-0 flex items-center justify-end bg-red-500 text-white px-4">
    <span>حذف</span>
  </div>
  <!-- Card content -->
  <div class="relative bg-white p-4 transition-transform" data-swipeable>
    <!-- Product info -->
  </div>
</div>
```

---

## 17. Localization Guidelines

### 17.1 Translation Key Structure

Use **dot notation** with Arabic-first organization:

```typescript
const translations = {
  common: {
    save: { ar: 'حفظ', en: 'Save' },
    cancel: { ar: 'إلغاء', en: 'Cancel' },
    delete: { ar: 'حذف', en: 'Delete' },
    confirm: { ar: 'تأكيد', en: 'Confirm' },
    loading: { ar: 'جاري التحميل...', en: 'Loading...' },
    error: { ar: 'حدث خطأ', en: 'An error occurred' },
  },
  cart: {
    title: { ar: 'سلة المشتريات', en: 'Shopping Cart' },
    empty: { ar: 'السلة فارغة', en: 'Your cart is empty' },
    total: { ar: 'المجموع', en: 'Total' },
    checkout: { ar: 'إتمام الطلب', en: 'Checkout' },
  },
  auth: {
    phoneLabel: { ar: 'رقم الهاتف', en: 'Phone Number' },
    phonePlaceholder: { ar: '77X XXX XXXX', en: '77X XXX XXXX' },
    sendOtp: { ar: 'إرسال رمز التحقق', en: 'Send Verification Code' },
    otpLabel: { ar: 'رمز التحقق', en: 'Verification Code' },
  },
} as const;
```

### 17.2 Translation Function

```typescript
import { useI18n } from '@/i18n';

function MyComponent() {
  const { t, locale } = useI18n();

  return (
    <div dir={locale === 'ar' ? 'rtl' : 'ltr'}>
      <h1>{t('cart.title')}</h1>
      <p>{t('cart.empty')}</p>
      <button>{t('common.checkout')}</button>
    </div>
  );
}
```

### 17.3 RTL/LTR Conditional Styling

```typescript
// Utility for directional styles
function useDirection() {
  const { locale } = useI18n();
  return {
    isRTL: locale === 'ar',
    dir: locale === 'ar' ? 'rtl' : 'ltr',
    // Directional class helpers
    marginStart: (value: string) => locale === 'ar' ? `ms-${value}` : `ms-${value}`,
    paddingEnd: (value: string) => locale === 'ar' ? `pe-${value}` : `pe-${value}`,
  };
}
```

### 17.4 Hardcoded Strings

**NEVER** hardcode user-facing strings in components.

```typescript
// ❌ Wrong
<button>حفظ</button>
<p>السلة فارغة</p>

// ✅ Correct
<button>{t('common.save')}</button>
<p>{t('cart.empty')}</p>
```

### 17.5 Pluralization

```typescript
const translations = {
  cart: {
    itemCount: {
      ar: (count: number) => count === 1 ? 'منتج واحد' : `${count} منتجات`,
      en: (count: number) => count === 1 ? '1 item' : `${count} items`,
    },
  },
};
```

### 17.6 Number-Dependent Strings

```typescript
const translations = {
  order: {
    status: {
      ar: (status: string) => {
        const map: Record<string, string> = {
          pending: 'قيد الانتظار',
          processing: 'قيد التنفيذ',
          shipped: 'تم الشحن',
          delivered: 'تم التوصيل',
        };
        return map[status] || status;
      },
    },
  },
};
```

---

## 18. Date/Time Formatting

### 18.1 Arabic Date Format

```
١٢ سبتمبر ٢٠٢٦
(12 September 2026)
```

**Full format**: `day month year`  
**Short format**: `dd/mm/yyyy` → `١٢/٠٩/٢٠٢٦`

### 18.2 English Date Format

```
September 12, 2026
```

**Full format**: `month day, year`  
**Short format**: `YYYY-MM-DD` (ISO 8601) or `MM/DD/YYYY`

### 18.3 Implementation

```typescript
function formatDate(date: Date, locale: 'ar' | 'en'): string {
  return new Intl.DateTimeFormat(locale === 'ar' ? 'ar-YE' : 'en-US', {
    year: 'numeric',
    month: 'long',
    day: 'numeric',
  }).format(date);
}

function formatDateTime(date: Date, locale: 'ar' | 'en'): string {
  return new Intl.DateTimeFormat(locale === 'ar' ? 'ar-YE' : 'en-US', {
    year: 'numeric',
    month: 'short',
    day: 'numeric',
    hour: '2-digit',
    minute: '2-digit',
    hour12: locale === 'ar',
  }).format(date);
}

// Arabic: ١٢ سبتمبر ٢٠٢٦، ٢:٣٠ م
// English: Sep 12, 2026, 2:30 PM
```

### 18.4 Relative Time

```typescript
function formatRelativeTime(date: Date, locale: 'ar' | 'en'): string {
  const rtf = new Intl.RelativeTimeFormat(locale === 'ar' ? 'ar-YE' : 'en-US', {
    numeric: 'auto',
  });

  const diffInSeconds = Math.floor((date.getTime() - Date.now()) / 1000);

  if (Math.abs(diffInSeconds) < 60) return rtf.format(diffInSeconds, 'second');
  if (Math.abs(diffInSeconds) < 3600) return rtf.format(Math.floor(diffInSeconds / 60), 'minute');
  if (Math.abs(diffInSeconds) < 86400) return rtf.format(Math.floor(diffInSeconds / 3600), 'hour');
  return rtf.format(Math.floor(diffInSeconds / 86400), 'day');
}

// Arabic: منذ يومين
// English: 2 days ago
```

### 18.5 Hijri Calendar (Optional)

```typescript
function formatHijriDate(date: Date): string {
  return new Intl.DateTimeFormat('ar-SA-u-ca-islamic', {
    year: 'numeric',
    month: 'long',
    day: 'numeric',
  }).format(date);
}

// Example: ١٤ ربيع الآخر ١٤٤٨ هـ
```

### 18.6 Time Format

| Context        | Arabic Format     | English Format  |
|----------------|-------------------|-----------------|
| 12-hour        | `٢:٣٠ م`         | `2:30 PM`       |
| 24-hour        | `14:30`           | `14:30`         |
| Time ago       | `منذ ٥ دقائق`     | `5 minutes ago` |
| Scheduled      | `٢:٣٠ م، ١٣ سبتمبر` | `2:30 PM, Sep 13` |

---

## 19. Currency Formatting

### 19.1 Supported Currencies

| Code | Symbol | Name (Arabic)           | Name (English)        |
|------|--------|-------------------------|-----------------------|
| YER  | ر.ي    | ريال يمني               | Yemeni Rial           |
| SAR  | ر.س    | ريال سعودي              | Saudi Riyal           |
| USD  | $      | دولار أمريكي             | US Dollar             |

### 19.2 Formatting Rules

**Arabic:**

| Value          | Formatted         |
|----------------|-------------------|
| 1000           | ١٬٠٠٠ ر.ي        |
| 1234567.89     | ١٬٢٣٤٬٥٦٧٫٨٩ ر.ي |
| 500            | ٥٠٠ ر.ي          |

**English:**

| Value          | Formatted         |
|----------------|-------------------|
| 1000           | 1,000 YER         |
| 1234567.89     | 1,234,567.89 YER  |
| 500            | 500 YER           |

### 19.3 Implementation

```typescript
function formatCurrency(
  amount: number,
  currency: 'YER' | 'SAR' | 'USD',
  locale: 'ar' | 'en'
): string {
  return new Intl.NumberFormat(locale === 'ar' ? 'ar-YE' : 'en-US', {
    style: 'currency',
    currency: currency,
    minimumFractionDigits: currency === 'USD' ? 2 : 0,
    maximumFractionDigits: currency === 'USD' ? 2 : 0,
  }).format(amount);
}
```

### 19.4 Price Display in Product Cards

```html
<!-- Arabic product card -->
<div class="text-end">
  <span class="text-2xl font-bold text-gray-900">١٥٬٠٠٠</span>
  <span class="text-sm text-gray-500"> ر.ي</span>
</div>

<!-- English product card -->
<div class="text-start">
  <span class="text-2xl font-bold text-gray-900">$15,000</span>
  <span class="text-sm text-gray-500"> YER</span>
</div>
```

### 19.5 Currency Selector

```html
<select class="rounded-lg border border-gray-300 px-3 py-2 text-sm">
  <option value="YER">ر.ي — ريال يمني</option>
  <option value="SAR">ر.س — ريال سعودي</option>
  <option value="USD">$ — دولار أمريكي</option>
</select>
```

---

## 20. Number Formatting

### 20.1 Numeral Systems

| System             | Digits              | Use Case                         |
|--------------------|----------------------|----------------------------------|
| Arabic-Indic       | ٠١٢٣٤٥٦٧٨٩         | Arabic UI, customer-facing       |
| Western-Arabic     | 0123456789           | English UI, technical contexts   |

### 20.2 Implementation

```typescript
function formatNumber(num: number, locale: 'ar' | 'en'): string {
  return new Intl.NumberFormat(locale === 'ar' ? 'ar-YE' : 'en-US').format(num);
}

// Arabic: ١٢٬٣٤٥٬٦٧٨
// English: 12,345,678
```

### 20.3 Number Display Rules

| Context          | Arabic                  | English               |
|------------------|-------------------------|-----------------------|
| Price            | ١٥٬٠٠٠ ر.ي            | 15,000 YER            |
| Quantity         | ٣ منتجات                | 3 items               |
| Percentage       | %١٥                     | 15%                   |
| Rating           | ٤٫٥ من ٥               | 4.5 out of 5          |
| Phone number     | 77X XXX XXXX (Latin)    | 77X XXX XXXX          |
| OTP code         | 123456 (Latin)          | 123456                |
| Order number     | #١٢٣٤٥                 | #12345                |
| Quantity in cart | ٢                       | 2                     |

### 20.4 Decimal Separator

- Arabic uses **Arabic decimal separator** (`٫` U+066B) or period.
- English uses **period** (`.`).
- The `Intl.NumberFormat` API handles this automatically.

### 20.5 Thousands Separator

- Arabic uses **Arabic thousands separator** (`٬` U+066C).
- English uses **comma** (`,`).

### 20.6 Special Number Cases

**Phone numbers and OTP codes** always use Latin numerals, regardless of locale:

```typescript
// Always Latin numerals
function formatPhone(phone: string): string {
  return phone.replace(/(\d{3})(\d{3})(\d{3})/, '$1 $2 $3');
}

// 771234567 → 771 234 567
```

**Order numbers** use locale-specific numerals:

```typescript
function formatOrderNumber(id: number, locale: 'ar' | 'en'): string {
  const formatted = formatNumber(id, locale);
  return `#${formatted}`;
}

// Arabic: #١٢٣٤٥
// English: #12345
```

---

## Quick Reference: Design Token Summary

### Colors

| Token        | Hex       | Tailwind Class                |
|--------------|-----------|-------------------------------|
| Primary      | `#1B2A4A` | `navy-800`                    |
| Primary Dark | `#2563eb` | `navy-900`                    |
| Success      | `#22c55e` | `green-500`                   |
| Warning      | `#f59e0b` | `amber-500`                   |
| Error        | `#ef4444` | `red-500`                     |
| Info         | `#1B2A4A` | `navy-800`                    |
| Gray 50      | `#f9fafb` | `gray-50`                     |
| Gray 100     | `#f3f4f6` | `gray-100`                    |
| Gray 200     | `#e5e7eb` | `gray-200`                    |
| Gray 500     | `#6b7280` | `gray-500`                    |
| Gray 900     | `#111827` | `gray-900`                    |

### Typography

| Token        | Font Family (Arabic)           | Font Family (English)  |
|--------------|--------------------------------|------------------------|
| `font-sans`  | Tajawal, Cairo, Noto Sans Arabic | Inter, Roboto          |
| `font-mono`  | Noto Sans Arabic               | Fira Code              |

### Breakpoints

| Token | Width   | Tailwind Prefix |
|-------|---------|-----------------|
| xs    | 320px   | (default)       |
| sm    | 640px   | `sm:`           |
| md    | 768px   | `md:`           |
| lg    | 1024px  | `lg:`           |
| xl    | 1280px  | `xl:`           |
| 2xl   | 1536px  | `2xl:`          |

### Spacing

| Token   | Value  | Use Case                      |
|---------|--------|-------------------------------|
| `gap-1` | 4px    | Tight element spacing         |
| `gap-2` | 8px    | Default element spacing       |
| `gap-3` | 12px   | Related element spacing       |
| `gap-4` | 16px   | Standard spacing              |
| `gap-6` | 24px   | Section spacing               |
| `gap-8` | 32px   | Large section spacing         |

---

*End of Design Guidelines*
