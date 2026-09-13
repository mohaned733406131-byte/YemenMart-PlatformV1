# RTL Design — YemenMart

## 1. Overview

YemenMart is an **Arabic-first** platform with full RTL (Right-to-Left) support. All interfaces default to RTL layout, with seamless LTR support for bilingual content.

## 2. RTL Principles

| Principle | Description |
|-----------|-------------|
| Arabic Default | All layouts start as RTL |
| Mirrored Icons | Directional icons flip in RTL |
| Logical Properties | Use `start`/`end` instead of `left`/`right` |
| Bidirectional Text | Support mixed Arabic/English content |
| Number Handling | Arabic-Indic numerals support |

## 3. CSS Implementation

### HTML Direction

```html
<html lang="ar" dir="rtl">
```

### Logical Properties

```css
/* OLD - Don't use */
padding-left: 16px;
margin-right: 16px;
text-align: left;
border-left: 2px solid;

/* NEW - Use logical properties */
padding-inline-start: 16px;
margin-inline-end: 16px;
text-align: start;
border-inline-start: 2px solid;
```

### Tailwind RTL Classes

```html
<!-- Spacing -->
<div class="ms-4">  <!-- margin-inline-start -->
<div class="me-4">  <!-- margin-inline-end -->
<div class="ps-4">  <!-- padding-inline-start -->
<div class="pe-4">  <!-- padding-inline-end -->

<!-- Text alignment -->
<p class="text-start">Text aligned to start</p>
<p class="text-end">Text aligned to end</p>

<!-- Borders -->
<div class="border-s-2">  <!-- border-inline-start -->
<div class="border-e-2">  <!-- border-inline-end -->

<!-- Positioning -->
<div class="start-4">  <!-- inset-inline-start -->
<div class="end-4">    <!-- inset-inline-end -->
```

### Direction Switching

```css
/* Base styles (RTL) */
.container {
  padding-inline-start: 16px;
  padding-inline-end: 16px;
}

/* Override for LTR if needed */
[dir="ltr"] .container {
  /* Specific LTR overrides */
}

/* Flip elements based on direction */
[dir="rtl"] .icon-arrow {
  transform: rotate(180deg);
}

/* Arrow that points correct direction */
.icon-arrow-forward::before {
  content: '→';
}

[dir="rtl"] .icon-arrow-forward::before {
  content: '←';
}
```

## 4. Component RTL Patterns

### Navigation

```html
<!-- RTL Navigation -->
<nav class="flex items-center gap-4">
  <a href="/" class="text-start">الرئيسية</a>
  <a href="/products" class="text-start">المنتجات</a>
  <a href="/cart" class="text-start">السلة</a>
</nav>

<!-- Back button (flips direction) -->
<button class="flex items-center gap-2">
  <svg class="w-5 h-5 rotate-180" /> <!-- Arrow flips in RTL -->
  <span>رجوع</span>
</button>
```

### Forms

```html
<!-- RTL Form -->
<form class="space-y-4">
  <div>
    <label class="block text-sm font-medium text-start mb-1.5">
      اسم المستخدم
    </label>
    <input class="w-full text-start" dir="auto" />
  </div>
  
  <div class="flex items-center gap-2">
    <input type="checkbox" id="remember" />
    <label for="remember" class="text-start">تذكرني</label>
  </div>
</form>
```

### Product Cards

```html
<!-- RTL Product Card -->
<div class="flex gap-4">
  <img class="w-32 h-32 object-cover rounded-lg" />
  <div class="flex-1 text-start">
    <h3 class="font-semibold">اسم المنتج</h3>
    <p class="text-neutral-500">اسم المتجر</p>
    <p class="text-primary-600 font-bold">١٥,٠٠٠ يمني</p>
  </div>
</div>
```

### Cart Layout

```html
<!-- RTL Cart -->
<div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
  <!-- Items (right side in RTL) -->
  <div class="lg:col-span-2 space-y-4">
    <div class="flex gap-4">
      <img class="w-24 h-24" />
      <div class="flex-1 text-start">
        <h3>اسم المنتج</h3>
        <p class="text-primary-600">١٥,٠٠٠ يمني</p>
      </div>
      <button class="text-red-500">
        <svg class="w-5 h-5" /> <!-- Delete icon -->
      </button>
    </div>
  </div>
  
  <!-- Summary (left side in RTL) -->
  <div class="bg-neutral-50 p-4 rounded-xl">
    <h3>ملخص الطلب</h3>
    <!-- Summary content -->
  </div>
</div>
```

### Table

```html
<!-- RTL Table -->
<div class="overflow-x-auto">
  <table class="w-full text-sm">
    <thead>
      <tr>
        <th class="text-start px-4 py-3">رقم الطلب</th>
        <th class="text-start px-4 py-3">التاريخ</th>
        <th class="text-start px-4 py-3">المبلغ</th>
        <th class="text-start px-4 py-3">الحالة</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td class="px-4 py-3">YM-001</td>
        <td class="px-4 py-3">١٣ سبتمبر ٢٠٢٦</td>
        <td class="px-4 py-3">١٥,٠٠٠ يمني</td>
        <td class="px-4 py-3">
          <span class="badge-success">مكتمل</span>
        </td>
      </tr>
    </tbody>
  </table>
</div>
```

## 5. Icon Direction

### Flipped Icons (Directional)

| Icon | LTR | RTL | Usage |
|------|-----|-----|-------|
| Arrow Left | ← | → | Back navigation |
| Arrow Right | → | ← | Forward navigation |
| Chevron Left | ◂ | ▸ | Previous page |
| Chevron Right | ▸ | ◂ | Next page |
| Share | → | ← | Share direction |
| Reply | ← | → | Reply direction |

### Non-Flipped Icons (Non-Directional)

| Icon | Usage |
|------|-------|
| Search | Universal symbol |
| Heart | Universal symbol |
| Star | Universal symbol |
| Cart | Universal symbol |
| Home | Universal symbol |
| Settings | Universal symbol |

### Icon Component

```typescript
// components/ui/Icon.tsx
interface IconProps {
  name: string;
  size?: 'sm' | 'md' | 'lg';
  className?: string;
}

function Icon({ name, size = 'md', className }: IconProps) {
  const { isRtl } = useRtl();
  const sizes = { sm: 'w-4 h-4', md: 'w-5 h-5', lg: 'w-6 h-6' };
  
  // Icons that should flip in RTL
  const flipIcons = ['arrow-left', 'arrow-right', 'chevron-left', 'chevron-right', 'share', 'reply'];
  
  const shouldFlip = flipIcons.includes(name) && isRtl;
  
  return (
    <svg 
      className={cn(
        sizes[size],
        shouldFlip && 'scale-x-[-1]',
        className
      )}
      aria-hidden="true"
    >
      {/* SVG content */}
    </svg>
  );
}
```

## 6. Number Formatting

### Arabic-Indic Numerals

```typescript
// lib/number-format.ts
function toArabicIndic(num: number): string {
  const arabicIndic = ['٠', '١', '٢', '٣', '٤', '٥', '٦', '٧', '٨', '٩'];
  return num.toString().replace(/\d/g, (d) => arabicIndic[parseInt(d)]);
}

function formatPrice(amount: number, currency: string = 'YER'): string {
  const formatted = new Intl.NumberFormat('ar-YE', {
    style: 'decimal',
    minimumFractionDigits: 0,
    maximumFractionDigits: 0,
  }).format(amount);
  
  return `${formatted} يمني`;
}

// Usage
formatPrice(15000); // "١٥٬٠٠٠ يمني"
formatPrice(1500000); // "١٬٥٠٠٬٠٠٠ يمني"
```

### Date Formatting

```typescript
// lib/date-format.ts
function formatDate(date: Date): string {
  return new Intl.DateTimeFormat('ar-YE', {
    year: 'numeric',
    month: 'long',
    day: 'numeric',
  }).format(date);
}

function formatTime(date: Date): string {
  return new Intl.DateTimeFormat('ar-YE', {
    hour: '2-digit',
    minute: '2-digit',
  }).format(date);
}

// Usage
formatDate(new Date()); // "١٣ سبتمبر ٢٠٢٦"
formatTime(new Date()); // "٢:٣٠ م"
```

## 7. RTL Testing

### Visual Testing Checklist

| Element | RTL Behavior | Status |
|---------|-------------|--------|
| Navigation | Flipped horizontally | Required |
| Forms | Labels and inputs aligned start | Required |
| Buttons | Text and icon order flipped | Required |
| Cards | Image and content positions flipped | Required |
| Tables | Column order flipped | Required |
| Modals | Close button position flipped | Required |
| Tooltips | Position adjusted | Required |
| Dropdowns | Position adjusted | Required |
| Pagination | Previous/Next flipped | Required |
| Bread crumbs | Separator direction flipped | Required |

### Functional Testing

| Test Case | Expected Result |
|-----------|----------------|
| Switch language to English | Layout flips to LTR |
| Switch language to Arabic | Layout flips to RTL |
| Mixed content rendering | Arabic and English display correctly |
| Number input | Arabic-Indic and Western numerals accepted |
| Date input | Arabic date format displayed |
| Copy-paste mixed text | Text direction preserved |

## 8. Related Files

| File | Description |
|------|-------------|
| `design-system.md` | Design tokens and components |
| `accessibility.md` | WCAG 2.1 AA requirements |
| `responsive-design.md` | Responsive patterns |
| `05-frontend/component-library.md` | RTL-aware components |
| `11-ui-ux/wireframes/index.md` | RTL wireframes |

---

*Document Version: 1.0 | Last Updated: 2026-09-13 | Classification: Confidential*
