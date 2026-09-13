# YemenMart Design System

> Version 2.0 — Updated with corrected brand colors extracted from the official logo.

---

## 1. Brand Colors

The YemenMart logo uses two primary colors: a dark navy blue (cart outline, text, wheels) and a bright orange (the "M" letter, crown detail, decorative dots), set against a white background.

### Primary Palette

| Swatch | Name | Hex | Role |
|--------|------|-----|------|
| 🟦 | **Navy Blue** | `#1B2A4A` | Cart outline, text, headers, navigation |
| 🟧 | **Orange** | `#F57C20` | Call-to-action, highlights, active states |
| ⬜ | **White** | `#FFFFFF` | Backgrounds, cards |

### Derived Navy Blue Scale

| Token | Hex | Usage |
|-------|-----|-------|
| `navy-50` | `#f0f3f8` | Light navy backgrounds, hover states |
| `navy-100` | `#d9e0ed` | Borders, subtle highlights |
| `navy-200` | `#b3c1db` | Disabled states, muted elements |
| `navy-300` | `#8da2c9` | Placeholder text |
| `navy-400` | `#6783b7` | Secondary interactive elements |
| `navy-500` | `#4164a5` | Links, focused elements |
| `navy-600` | `#344f84` | Hover states for interactive elements |
| `navy-700` | `#273b63` | Active states |
| `navy-800` | `#1B2A4A` | **Primary** — headers, nav, main brand |
| `navy-900` | `#0f1a30` | Darkest navy, overlays |

### Derived Orange Scale

| Token | Hex | Usage |
|-------|-----|-------|
| `orange-50` | `#fff5eb` | Light orange backgrounds |
| `orange-100` | `#ffe6cc` | Hover states for light elements |
| `orange-200` | `#ffcc99` | Borders, subtle highlights |
| `orange-300` | `#ffb366` | Disabled CTA backgrounds |
| `orange-400` | `#ff9933` | Interactive hover states |
| `orange-500` | `#F57C20` | **Primary** — CTAs, buttons, active states |
| `orange-600` | `#e06518` | Hover states for CTAs |
| `orange-700` | `#c44e10` | Active/pressed states |
| `orange-800` | `#a83708` | Destructive orange actions |
| `orange-900` | `#8c2000` | Darkest orange, text on light bg |

### Semantic Colors

| Token | Hex | Usage |
|-------|-----|-------|
| `success-500` | `#22c55e` | Confirmations, positive feedback |
| `warning-500` | `#f59e0b` | Alerts, caution states |
| `error-500` | `#ef4444` | Errors, destructive actions |
| `info-500` | `#1B2A4A` | Informational — uses navy blue |

### Neutral Colors

| Token | Hex | Usage |
|-------|-----|-------|
| `neutral-50` | `#f9fafb` | Page backgrounds |
| `neutral-100` | `#f3f4f6` | Card backgrounds, subtle fills |
| `neutral-200` | `#e5e7eb` | Borders, dividers |
| `neutral-300` | `#d1d5db` | Disabled borders |
| `neutral-400` | `#9ca3af` | Placeholder text |
| `neutral-500` | `#6b7280` | Secondary text, captions |
| `neutral-600` | `#4b5563` | Body text (secondary) |
| `neutral-700` | `#374151` | Body text (primary) |
| `neutral-800` | `#1f2937` | Headings |
| `neutral-900` | `#111827` | High-contrast text |

### Color Usage Rules

| Context | Color | Value | Notes |
|---------|-------|-------|-------|
| Primary CTAs | `orange-500` | `#F57C20` | Buttons, links, active states, highlights |
| Headers / Nav | `navy-800` | `#1B2A4A` | Navigation bars, page headers, footers |
| Body Text | `neutral-900` | `#111827` | Headings, body text |
| Secondary Text | `neutral-500` | `#6b7280` | Captions, metadata |
| Background | `neutral-50` | `#f9fafb` | Page backgrounds |
| Card Background | `white` | `#FFFFFF` | Cards, modals |
| Borders | `neutral-200` | `#e5e7eb` | Dividers, borders |
| Success | `success-500` | `#22c55e` | Confirmations, positive feedback |
| Warning | `warning-500` | `#f59e0b` | Alerts, caution states |
| Error | `error-500` | `#ef4444` | Errors, destructive actions |
| Info | `navy-800` | `#1B2A4A` | Informational messages |

---

## 2. CSS Custom Properties

```css
:root {
  /* ─── Brand: Navy Blue ─── */
  --ym-navy-50: #f0f3f8;
  --ym-navy-100: #d9e0ed;
  --ym-navy-200: #b3c1db;
  --ym-navy-300: #8da2c9;
  --ym-navy-400: #6783b7;
  --ym-navy-500: #4164a5;
  --ym-navy-600: #344f84;
  --ym-navy-700: #273b63;
  --ym-navy-800: #1B2A4A;
  --ym-navy-900: #0f1a30;

  /* ─── Brand: Orange ─── */
  --ym-orange-50: #fff5eb;
  --ym-orange-100: #ffe6cc;
  --ym-orange-200: #ffcc99;
  --ym-orange-300: #ffb366;
  --ym-orange-400: #ff9933;
  --ym-orange-500: #F57C20;
  --ym-orange-600: #e06518;
  --ym-orange-700: #c44e10;
  --ym-orange-800: #a83708;
  --ym-orange-900: #8c2000;

  /* ─── Semantic Colors ─── */
  --ym-success-50: #f0fdf4;
  --ym-success-500: #22c55e;
  --ym-success-700: #15803d;

  --ym-warning-50: #fffbeb;
  --ym-warning-500: #f59e0b;
  --ym-warning-700: #b45309;

  --ym-error-50: #fef2f2;
  --ym-error-500: #ef4444;
  --ym-error-700: #b91c1c;

  --ym-info-50: #f0f3f8;
  --ym-info-500: #1B2A4A;
  --ym-info-700: #0f1a30;

  /* ─── Neutrals ─── */
  --ym-neutral-50: #f9fafb;
  --ym-neutral-100: #f3f4f6;
  --ym-neutral-200: #e5e7eb;
  --ym-neutral-300: #d1d5db;
  --ym-neutral-400: #9ca3af;
  --ym-neutral-500: #6b7280;
  --ym-neutral-600: #4b5563;
  --ym-neutral-700: #374151;
  --ym-neutral-800: #1f2937;
  --ym-neutral-900: #111827;

  /* ─── Semantic Aliases ─── */
  --ym-color-primary: var(--ym-navy-800);
  --ym-color-primary-hover: var(--ym-navy-700);
  --ym-color-accent: var(--ym-orange-500);
  --ym-color-accent-hover: var(--ym-orange-600);
  --ym-color-bg: var(--ym-neutral-50);
  --ym-color-surface: #ffffff;
  --ym-color-text: var(--ym-neutral-900);
  --ym-color-text-secondary: var(--ym-neutral-500);
  --ym-color-border: var(--ym-neutral-200);
  --ym-color-success: var(--ym-success-500);
  --ym-color-warning: var(--ym-warning-500);
  --ym-color-error: var(--ym-error-500);
  --ym-color-info: var(--ym-info-500);

  /* ─── Typography ─── */
  --ym-font-family-ar: 'Tajawal', 'Segoe UI', Tahoma, sans-serif;
  --ym-font-family-en: 'Inter', 'Segoe UI', Roboto, sans-serif;
  --ym-font-family-mono: 'JetBrains Mono', 'Fira Code', monospace;

  --ym-font-size-xs: 0.75rem;
  --ym-font-size-sm: 0.875rem;
  --ym-font-size-base: 1rem;
  --ym-font-size-lg: 1.125rem;
  --ym-font-size-xl: 1.25rem;
  --ym-font-size-2xl: 1.5rem;
  --ym-font-size-3xl: 1.875rem;
  --ym-font-size-4xl: 2.25rem;
  --ym-font-size-5xl: 3rem;

  --ym-line-height-tight: 1.25;
  --ym-line-height-normal: 1.5;
  --ym-line-height-relaxed: 1.75;

  --ym-font-weight-normal: 400;
  --ym-font-weight-medium: 500;
  --ym-font-weight-semibold: 600;
  --ym-font-weight-bold: 700;

  /* ─── Spacing ─── */
  --ym-space-0: 0;
  --ym-space-0-5: 0.125rem;
  --ym-space-1: 0.25rem;
  --ym-space-1-5: 0.375rem;
  --ym-space-2: 0.5rem;
  --ym-space-2-5: 0.625rem;
  --ym-space-3: 0.75rem;
  --ym-space-3-5: 0.875rem;
  --ym-space-4: 1rem;
  --ym-space-5: 1.25rem;
  --ym-space-6: 1.5rem;
  --ym-space-7: 1.75rem;
  --ym-space-8: 2rem;
  --ym-space-9: 2.25rem;
  --ym-space-10: 2.5rem;
  --ym-space-12: 3rem;
  --ym-space-14: 3.5rem;
  --ym-space-16: 4rem;
  --ym-space-20: 5rem;
  --ym-space-24: 6rem;

  /* ─── Border Radius ─── */
  --ym-radius-none: 0;
  --ym-radius-sm: 0.25rem;
  --ym-radius-md: 0.375rem;
  --ym-radius-lg: 0.5rem;
  --ym-radius-xl: 0.75rem;
  --ym-radius-2xl: 1rem;
  --ym-radius-3xl: 1.5rem;
  --ym-radius-full: 9999px;

  /* ─── Shadows ─── */
  --ym-shadow-xs: 0 1px 2px 0 rgba(0, 0, 0, 0.05);
  --ym-shadow-sm: 0 1px 3px 0 rgba(0, 0, 0, 0.1), 0 1px 2px -1px rgba(0, 0, 0, 0.1);
  --ym-shadow-md: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -2px rgba(0, 0, 0, 0.1);
  --ym-shadow-lg: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -4px rgba(0, 0, 0, 0.1);
  --ym-shadow-xl: 0 20px 25px -5px rgba(0, 0, 0, 0.1), 0 8px 10px -6px rgba(0, 0, 0, 0.1);
  --ym-shadow-2xl: 0 25px 50px -12px rgba(0, 0, 0, 0.25);
  --ym-shadow-inner: inset 0 2px 4px 0 rgba(0, 0, 0, 0.05);
  --ym-shadow-focus: 0 0 0 3px rgba(245, 124, 32, 0.4);

  /* ─── Z-Index Scale ─── */
  --ym-z-base: 0;
  --ym-z-dropdown: 1000;
  --ym-z-sticky: 1100;
  --ym-z-fixed: 1200;
  --ym-z-backdrop: 1300;
  --ym-z-modal: 1400;
  --ym-z-popover: 1500;
  --ym-z-tooltip: 1600;
  --ym-z-toast: 1700;

  /* ─── Transitions ─── */
  --ym-duration-fast: 100ms;
  --ym-duration-normal: 200ms;
  --ym-duration-slow: 300ms;
  --ym-duration-slower: 500ms;
  --ym-ease-default: cubic-bezier(0.4, 0, 0.2, 1);
  --ym-ease-in: cubic-bezier(0.4, 0, 1, 1);
  --ym-ease-out: cubic-bezier(0, 0, 0.2, 1);
  --ym-ease-in-out: cubic-bezier(0.4, 0, 0.2, 1);

  /* ─── Breakpoints (reference only) ─── */
  /* sm: 640px | md: 768px | lg: 1024px | xl: 1280px | 2xl: 1536px */
}

/* ─── Dark Mode ─── */
[data-theme="dark"] {
  --ym-color-primary: var(--ym-navy-200);
  --ym-color-primary-hover: var(--ym-navy-100);
  --ym-color-accent: var(--ym-orange-400);
  --ym-color-accent-hover: var(--ym-orange-300);
  --ym-color-bg: #0f1a30;
  --ym-color-surface: #1a2540;
  --ym-color-text: var(--ym-neutral-50);
  --ym-color-text-secondary: var(--ym-neutral-400);
  --ym-color-border: #2a3550;
  --ym-color-success: #4ade80;
  --ym-color-warning: #fbbf24;
  --ym-color-error: #f87171;
  --ym-color-info: var(--ym-navy-200);

  --ym-shadow-sm: 0 1px 3px 0 rgba(0, 0, 0, 0.3), 0 1px 2px -1px rgba(0, 0, 0, 0.3);
  --ym-shadow-md: 0 4px 6px -1px rgba(0, 0, 0, 0.3), 0 2px 4px -2px rgba(0, 0, 0, 0.3);
  --ym-shadow-lg: 0 10px 15px -3px rgba(0, 0, 0, 0.4), 0 4px 6px -4px rgba(0, 0, 0, 0.3);
  --ym-shadow-xl: 0 20px 25px -5px rgba(0, 0, 0, 0.5), 0 8px 10px -6px rgba(0, 0, 0, 0.4);
}
```

---

## 3. Tailwind Configuration

```js
// tailwind.config.js
module.exports = {
  darkMode: ['class', '[data-theme="dark"]'],
  content: ['./src/**/*.{js,ts,jsx,tsx,html}'],
  theme: {
    extend: {
      colors: {
        navy: {
          50: '#f0f3f8',
          100: '#d9e0ed',
          200: '#b3c1db',
          300: '#8da2c9',
          400: '#6783b7',
          500: '#4164a5',
          600: '#344f84',
          700: '#273b63',
          800: '#1B2A4A',
          900: '#0f1a30',
        },
        orange: {
          50: '#fff5eb',
          100: '#ffe6cc',
          200: '#ffcc99',
          300: '#ffb366',
          400: '#ff9933',
          500: '#F57C20',
          600: '#e06518',
          700: '#c44e10',
          800: '#a83708',
          900: '#8c2000',
        },
        ym: {
          primary: '#1B2A4A',
          accent: '#F57C20',
          success: '#22c55e',
          warning: '#f59e0b',
          error: '#ef4444',
          info: '#1B2A4A',
        },
      },
      fontFamily: {
        sans: ['Inter', 'Segoe UI', 'Roboto', 'sans-serif'],
        'sans-ar': ['Tajawal', 'Segoe UI', 'Tahoma', 'sans-serif'],
        mono: ['JetBrains Mono', 'Fira Code', 'monospace'],
      },
      fontSize: {
        '2xs': ['0.625rem', { lineHeight: '0.875rem' }],
        xs: ['0.75rem', { lineHeight: '1rem' }],
        sm: ['0.875rem', { lineHeight: '1.25rem' }],
        base: ['1rem', { lineHeight: '1.5rem' }],
        lg: ['1.125rem', { lineHeight: '1.75rem' }],
        xl: ['1.25rem', { lineHeight: '1.75rem' }],
        '2xl': ['1.5rem', { lineHeight: '2rem' }],
        '3xl': ['1.875rem', { lineHeight: '2.25rem' }],
        '4xl': ['2.25rem', { lineHeight: '2.5rem' }],
        '5xl': ['3rem', { lineHeight: '1' }],
      },
      spacing: {
        0.5: '0.125rem',
        1.5: '0.375rem',
        2.5: '0.625rem',
        3.5: '0.875rem',
      },
      borderRadius: {
        sm: '0.25rem',
        DEFAULT: '0.375rem',
        md: '0.375rem',
        lg: '0.5rem',
        xl: '0.75rem',
        '2xl': '1rem',
        '3xl': '1.5rem',
      },
      boxShadow: {
        xs: '0 1px 2px 0 rgba(0, 0, 0, 0.05)',
        sm: '0 1px 3px 0 rgba(0, 0, 0, 0.1), 0 1px 2px -1px rgba(0, 0, 0, 0.1)',
        DEFAULT: '0 1px 3px 0 rgba(0, 0, 0, 0.1), 0 1px 2px -1px rgba(0, 0, 0, 0.1)',
        md: '0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -2px rgba(0, 0, 0, 0.1)',
        lg: '0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -4px rgba(0, 0, 0, 0.1)',
        xl: '0 20px 25px -5px rgba(0, 0, 0, 0.1), 0 8px 10px -6px rgba(0, 0, 0, 0.1)',
        '2xl': '0 25px 50px -12px rgba(0, 0, 0, 0.25)',
        inner: 'inset 0 2px 4px 0 rgba(0, 0, 0, 0.05)',
        focus: '0 0 0 3px rgba(245, 124, 32, 0.4)',
      },
      zIndex: {
        base: '0',
        dropdown: '1000',
        sticky: '1100',
        fixed: '1200',
        backdrop: '1300',
        modal: '1400',
        popover: '1500',
        tooltip: '1600',
        toast: '1700',
      },
      animation: {
        'fade-in': 'fadeIn var(--ym-duration-normal) var(--ym-ease-out)',
        'fade-out': 'fadeOut var(--ym-duration-normal) var(--ym-ease-in)',
        'slide-up': 'slideUp var(--ym-duration-slow) var(--ym-ease-out)',
        'slide-down': 'slideDown var(--ym-duration-slow) var(--ym-ease-out)',
        'slide-in-right': 'slideInRight var(--ym-duration-slow) var(--ym-ease-out)',
        'slide-in-left': 'slideInLeft var(--ym-duration-slow) var(--ym-ease-out)',
        'scale-in': 'scaleIn var(--ym-duration-normal) var(--ym-ease-out)',
        'pulse-subtle': 'pulseSubtle 2s var(--ym-ease-in-out) infinite',
      },
      keyframes: {
        fadeIn: {
          from: { opacity: '0' },
          to: { opacity: '1' },
        },
        fadeOut: {
          from: { opacity: '1' },
          to: { opacity: '0' },
        },
        slideUp: {
          from: { transform: 'translateY(10px)', opacity: '0' },
          to: { transform: 'translateY(0)', opacity: '1' },
        },
        slideDown: {
          from: { transform: 'translateY(-10px)', opacity: '0' },
          to: { transform: 'translateY(0)', opacity: '1' },
        },
        slideInRight: {
          from: { transform: 'translateX(10px)', opacity: '0' },
          to: { transform: 'translateX(0)', opacity: '1' },
        },
        slideInLeft: {
          from: { transform: 'translateX(-10px)', opacity: '0' },
          to: { transform: 'translateX(0)', opacity: '1' },
        },
        scaleIn: {
          from: { transform: 'scale(0.95)', opacity: '0' },
          to: { transform: 'scale(1)', opacity: '1' },
        },
        pulseSubtle: {
          '0%, 100%': { opacity: '1' },
          '50%': { opacity: '0.7' },
        },
      },
    },
  },
  plugins: [],
};
```

---

## 4. Typography

### Font Stack

| Context | Font Family | Fallback |
|---------|-------------|----------|
| Arabic (`dir="rtl"`) | Tajawal | Segoe UI, Tahoma, sans-serif |
| English (`dir="ltr"`) | Inter | Segoe UI, Roboto, sans-serif |
| Code / Monospace | JetBrains Mono | Fira Code, monospace |

### Type Scale

| Token | Size | Line Height | Weight | Usage |
|-------|------|-------------|--------|-------|
| `display-lg` | 3rem / 48px | 1 | Bold | Hero headings |
| `display-md` | 2.25rem / 36px | 1.25 | Bold | Page titles |
| `display-sm` | 1.875rem / 30px | 1.25 | Bold | Section headings |
| `heading-lg` | 1.5rem / 24px | 1.33 | Semibold | Card titles |
| `heading-md` | 1.25rem / 20px | 1.4 | Semibold | Subsection headings |
| `heading-sm` | 1.125rem / 18px | 1.5 | Semibold | Small headings |
| `body-lg` | 1rem / 16px | 1.5 | Normal | Large body text |
| `body-md` | 0.875rem / 14px | 1.5 | Normal | Default body text |
| `body-sm` | 0.75rem / 12px | 1.5 | Normal | Captions, metadata |
| `label-lg` | 0.875rem / 14px | 1.25 | Medium | Button text |
| `label-md` | 0.75rem / 12px | 1.25 | Medium | Input labels |
| `label-sm` | 0.625rem / 10px | 1.25 | Medium | Badge text |

### Typography Rules

- **Arabic text**: Use `font-family: var(--ym-font-family-ar)` and always wrap with `dir="rtl"`.
- **English text**: Use `font-family: var(--ym-font-family-en)` and always wrap with `dir="ltr"`.
- **Mixed content**: Arabic should be primary; English is secondary. Both can coexist within the same layout but must respect the document's base direction.
- **Base font size**: 16px (1rem) for body content.

---

## 5. Spacing System

The spacing scale is based on a **4px base unit**. All spacing values are multiples of 4px.

| Token | Value | Pixels | Common Usage |
|-------|-------|--------|--------------|
| `space-0` | 0 | 0px | Reset / collapse |
| `space-0.5` | 0.125rem | 2px | Micro gaps (icon-text) |
| `space-1` | 0.25rem | 4px | Tight spacing |
| `space-1.5` | 0.375rem | 6px | Compact elements |
| `space-2` | 0.5rem | 8px | Default small spacing |
| `space-2.5` | 0.625rem | 10px | Input padding |
| `space-3` | 0.75rem | 12px | Card inner padding |
| `space-3.5` | 0.875rem | 14px | Medium padding |
| `space-4` | 1rem | 16px | Standard spacing |
| `space-5` | 1.25rem | 20px | Medium-large spacing |
| `space-6` | 1.5rem | 24px | Section spacing |
| `space-8` | 2rem | 32px | Large spacing |
| `space-10` | 2.5rem | 40px | XL spacing |
| `space-12` | 3rem | 48px | Section gaps |
| `space-16` | 4rem | 64px | Page margins |
| `space-20` | 5rem | 80px | Hero spacing |
| `space-24` | 6rem | 96px | Max spacing |

---

## 6. Border Radius

| Token | Value | Usage |
|-------|-------|-------|
| `radius-none` | 0 | No rounding |
| `radius-sm` | 0.25rem (4px) | Badges, small elements |
| `radius-md` | 0.375rem (6px) | Buttons, inputs |
| `radius-lg` | 0.5rem (8px) | Cards, panels |
| `radius-xl` | 0.75rem (12px) | Modals, dropdowns |
| `radius-2xl` | 1rem (16px) | Large cards, dialogs |
| `radius-3xl` | 1.5rem (24px) | Feature cards |
| `radius-full` | 9999px | Avatars, pills, circular |

---

## 7. Shadows

| Token | Description | Usage |
|-------|-------------|-------|
| `shadow-xs` | Minimal lift | Subtle borders |
| `shadow-sm` | Low elevation | Dropdown menus |
| `shadow-md` | Medium elevation | Cards, popovers |
| `shadow-lg` | High elevation | Modals, drawers |
| `shadow-xl` | Very high elevation | Toast notifications |
| `shadow-2xl` | Maximum elevation | Floating elements |
| `shadow-inner` | Inset shadow | Input focus states |
| `shadow-focus` | Orange glow ring | Focus indicators |

**Note**: `shadow-focus` uses `rgba(245, 124, 32, 0.4)` — derived from the brand orange for consistent focus indicators.

---

## 8. Breakpoints

| Token | Width | Target |
|-------|-------|--------|
| `sm` | 640px | Large phones (landscape) |
| `md` | 768px | Tablets (portrait) |
| `lg` | 1024px | Tablets (landscape), small laptops |
| `xl` | 1280px | Desktops |
| `2xl` | 1536px | Large desktops |

**Mobile-first approach**: All base styles apply to screens < 640px. Use `sm:`, `md:`, `lg:`, `xl:`, `2xl:` prefixes to override at each breakpoint.

---

## 9. Z-Index Scale

| Token | Value | Usage |
|-------|-------|-------|
| `z-base` | 0 | Default layer |
| `z-dropdown` | 1000 | Dropdown menus, select options |
| `z-sticky` | 1100 | Sticky headers, navigation |
| `z-fixed` | 1200 | Fixed elements (navbar, sidebar) |
| `z-backdrop` | 1300 | Modal backdrops, overlays |
| `z-modal` | 1400 | Modal dialogs, drawers |
| `z-popover` | 1500 | Popovers, tooltips |
| `z-tooltip` | 1600 | Tooltips, help text |
| `z-toast` | 1700 | Toast notifications |

**Rule**: Never use arbitrary z-index values. Always reference this scale.

---

## 10. Animation & Motion

### Duration Scale

| Token | Value | Usage |
|-------|-------|-------|
| `duration-fast` | 100ms | Micro-interactions (color changes, opacity) |
| `duration-normal` | 200ms | Standard transitions (hover states) |
| `duration-slow` | 300ms | Page transitions, drawer animations |
| `duration-slower` | 500ms | Complex multi-step animations |

### Easing Curves

| Token | Value | Usage |
|-------|-------|-------|
| `ease-default` | `cubic-bezier(0.4, 0, 0.2, 1)` | General-purpose transitions |
| `ease-in` | `cubic-bezier(0.4, 0, 1, 1)` | Elements leaving the screen |
| `ease-out` | `cubic-bezier(0, 0, 0.2, 1)` | Elements entering the screen |
| `ease-in-out` | `cubic-bezier(0.4, 0, 0.2, 1)` | Elements moving within the screen |

### Animation Presets

| Name | Effect | Usage |
|------|--------|-------|
| `fade-in` | Opacity 0 → 1 | Modal appear, content load |
| `fade-out` | Opacity 1 → 0 | Modal dismiss |
| `slide-up` | 10px up + opacity | Dropdown, popover appear |
| `slide-down` | 10px down + opacity | Accordion expand |
| `slide-in-right` | 10px right + opacity | Toast notification (LTR) |
| `slide-in-left` | 10px left + opacity | Toast notification (RTL) |
| `scale-in` | 95% → 100% + opacity | Button press feedback |
| `pulse-subtle` | Opacity pulse | Loading indicators |

### Motion Principles

1. **Purposeful**: Every animation should communicate something (success, error, navigation, feedback).
2. **Consistent**: Use the same animation for the same type of action across the app.
3. **Quick**: Prefer `duration-normal` (200ms). Only use longer durations for page transitions.
4. **Respectful**: Check `prefers-reduced-motion` and disable animations when the user has requested reduced motion.

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

---

## 11. Dark Mode

### Strategy

Dark mode uses a `[data-theme="dark"]` selector on the `<html>` element, toggled via JavaScript. CSS custom properties are overridden in the dark mode block (see Section 2).

### Dark Mode Color Mapping

| Light Mode | Dark Mode | Purpose |
|------------|-----------|---------|
| `--ym-color-bg: #f9fafb` | `--ym-color-bg: #0f1a30` | Page background |
| `--ym-color-surface: #ffffff` | `--ym-color-surface: #1a2540` | Card/modal surface |
| `--ym-color-text: #111827` | `--ym-color-text: #f9fafb` | Primary text |
| `--ym-color-text-secondary: #6b7280` | `--ym-color-text-secondary: #9ca3af` | Secondary text |
| `--ym-color-border: #e5e7eb` | `--ym-color-border: #2a3550` | Borders |
| `--ym-color-primary: #1B2A4A` | `--ym-color-primary: #b3c1db` | Primary actions |
| `--ym-color-accent: #F57C20` | `--ym-color-accent: #ff9933` | Accent actions |

### Dark Mode Rules

- Never use hardcoded colors in dark mode components. Always use CSS custom properties.
- Navy blue background in dark mode (`#0f1a30`) maintains brand identity.
- Orange accent shifts to a lighter shade (`#ff9933`) for better contrast on dark backgrounds.
- Semantic colors (success, warning, error) have dedicated dark-mode variants for readability.
- Shadows are intensified in dark mode for depth perception.

---

## 12. RTL Implementation Rules

### General Rules

1. **Always use logical properties** instead of physical properties:
   - Use `margin-inline-start` instead of `margin-left`
   - Use `padding-inline-end` instead of `padding-right`
   - Use `border-inline-start` instead of `border-left`
   - Use `inset-inline-start` instead of `left`

2. **Document direction**: Set `dir="rtl"` on the `<html>` element for Arabic pages.

3. **Text alignment**: Use `text-align: start` / `text-align: end` instead of `left` / `right`.

4. **Flexbox**: Use `flex-start` / `flex-end` for `justify-content` and `align-items`.

5. **CSS Logical Properties**:
   ```css
   /* CORRECT */
   .card {
     margin-inline: auto;
     padding-inline-start: 1rem;
     padding-inline-end: 1rem;
     border-inline-start: 3px solid var(--ym-color-accent);
     text-align: start;
   }

   /* WRONG */
   .card {
     margin: 0 auto;
     padding-left: 1rem;
     padding-right: 1rem;
     border-left: 3px solid var(--ym-color-accent);
     text-align: left;
   }
   ```

6. **Transform flips**: For directional icons and illustrations:
   ```css
   [dir="rtl"] .icon-arrow {
     transform: scaleX(-1);
   }
   ```

7. **Grid alignment**: Use `justify-content: start` / `end` instead of `left` / `right`.

8. **Tailwind RTL**: Use the `rtl:` prefix for RTL-specific overrides:
   ```html
   <div class="ps-4 rtl:pe-4 rtl:ps-0">Content</div>
   ```

9. **Font switching**: Arabic text should use `font-family: var(--ym-font-family-ar)`. Apply this automatically via the `html[dir="rtl"]` selector:
   ```css
   html[dir="rtl"] body {
     font-family: var(--ym-font-family-ar);
   }
   html[dir="ltr"] body {
     font-family: var(--ym-font-family-en);
   }
   ```

10. **Number formatting**: Arabic numerals (٠١٢٣٤٥٦٧٨٩) vs Western numerals (0123456789). Use `font-variant-numeric: tabular-nums` for alignment in tables.

---

## Appendix A: Component Quick Reference

### Button

| Variant | Background | Text | Hover |
|---------|------------|------|-------|
| Primary | `orange-500` | White | `orange-600` |
| Secondary | `navy-800` | White | `navy-700` |
| Outline | Transparent | `orange-500` | `orange-50` bg |
| Ghost | Transparent | `navy-800` | `neutral-100` bg |
| Danger | `error-500` | White | `error-700` |

### Input

| State | Border | Focus Ring |
|-------|--------|------------|
| Default | `neutral-300` | None |
| Focus | `orange-500` | `shadow-focus` (orange) |
| Error | `error-500` | `error-500` glow |
| Success | `success-500` | `success-500` glow |
| Disabled | `neutral-200` | None |

### Card

- Background: `white`
- Border: `neutral-200`
- Border radius: `radius-xl`
- Shadow: `shadow-md`
- Hover: `shadow-lg`

---

## Appendix B: File Naming Convention

```
components/
  Button/
    Button.tsx
    Button.styles.ts
    Button.stories.tsx
    Button.test.tsx
  Card/
    Card.tsx
    Card.styles.ts
    ...
```

---

*This design system is version 2.0. Previous color values (#3b82f6 blue, etc.) have been corrected to match the official YemenMart logo branding.*
