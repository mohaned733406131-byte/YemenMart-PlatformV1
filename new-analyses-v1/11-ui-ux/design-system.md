# Design System — YemenMart

## 1. Overview

YemenMart's design system provides a consistent, accessible, Arabic-first design language across all platforms. Built on Tailwind CSS 4 with custom design tokens, it ensures visual consistency and RTL support.

## 2. Design Principles

| Principle | Description |
|-----------|-------------|
| Arabic-First | RTL layout as default; Arabic typography primary |
| Mobile-First | Designed for 320px+ screens, enhanced for larger |
| Accessible | WCAG 2.1 AA compliance throughout |
| Consistent | Shared tokens, components, and patterns |
| Performant | Minimal CSS, optimized assets |

## 3. Color System

### Brand Colors

```css
:root {
  /* Primary - YemenMart Blue */
  --color-primary-50: #eff6ff;
  --color-primary-100: #dbeafe;
  --color-primary-200: #bfdbfe;
  --color-primary-300: #93c5fd;
  --color-primary-400: #60a5fa;
  --color-primary-500: #3b82f6;
  --color-primary-600: #2563eb;
  --color-primary-700: #1d4ed8;
  --color-primary-800: #1e40af;
  --color-primary-900: #1e3a8a;

  /* Secondary - Accent */
  --color-secondary-50: #faf5ff;
  --color-secondary-100: #f3e8ff;
  --color-secondary-200: #e9d5ff;
  --color-secondary-300: #d8b4fe;
  --color-secondary-400: #c084fc;
  --color-secondary-500: #a855f7;
  --color-secondary-600: #9333ea;
  --color-secondary-700: #7e22ce;
  --color-secondary-800: #6b21a8;
  --color-secondary-900: #581c87;

  /* Semantic Colors */
  --color-success-50: #f0fdf4;
  --color-success-100: #dcfce7;
  --color-success-500: #22c55e;
  --color-success-600: #16a34a;
  --color-success-700: #15803d;

  --color-warning-50: #fffbeb;
  --color-warning-100: #fef3c7;
  --color-warning-500: #f59e0b;
  --color-warning-600: #d97706;
  --color-warning-700: #b45309;

  --color-error-50: #fef2f2;
  --color-error-100: #fee2e2;
  --color-error-500: #ef4444;
  --color-error-600: #dc2626;
  --color-error-700: #b91c1c;

  /* Neutral */
  --color-neutral-50: #f9fafb;
  --color-neutral-100: #f3f4f6;
  --color-neutral-200: #e5e7eb;
  --color-neutral-300: #d1d5db;
  --color-neutral-400: #9ca3af;
  --color-neutral-500: #6b7280;
  --color-neutral-600: #4b5563;
  --color-neutral-700: #374151;
  --color-neutral-800: #1f2937;
  --color-neutral-900: #111827;
}
```

### Color Usage

| Context | Color | Usage |
|---------|-------|-------|
| Primary CTAs | primary-600 | Buttons, links, active states |
| Success | success-500 | Confirmations, positive feedback |
| Warning | warning-500 | Alerts, caution states |
| Error | error-500 | Errors, destructive actions |
| Text Primary | neutral-900 | Headings, body text |
| Text Secondary | neutral-500 | Captions, metadata |
| Background | neutral-50 | Page backgrounds |
| Card Background | white | Cards, modals |
| Borders | neutral-200 | Dividers, borders |

## 4. Typography

### Font Stack

```css
/* Arabic - Primary */
font-family: 'Tajawal', 'Cairo', 'Noto Sans Arabic', system-ui, sans-serif;

/* English - Secondary */
font-family: 'Inter', 'Roboto', system-ui, sans-serif;

/* Monospace - Code */
font-family: 'JetBrains Mono', 'Fira Code', monospace;
```

### Type Scale

| Token | Size | Line Height | Weight | Usage |
|-------|------|-------------|--------|-------|
| display-lg | 3rem (48px) | 1.1 | 700 | Hero headlines |
| display-md | 2.25rem (36px) | 1.2 | 700 | Section headlines |
| display-sm | 1.875rem (30px) | 1.25 | 600 | Page titles |
| h1 | 1.5rem (24px) | 1.33 | 600 | Main headings |
| h2 | 1.25rem (20px) | 1.4 | 600 | Sub-headings |
| h3 | 1.125rem (18px) | 1.5 | 600 | Card titles |
| body-lg | 1rem (16px) | 1.5 | 400 | Body text |
| body-md | 0.875rem (14px) | 1.5 | 400 | Standard text |
| body-sm | 0.75rem (12px) | 1.5 | 400 | Small text |
| caption | 0.6875rem (11px) | 1.4 | 400 | Captions, labels |

### Arabic Typography Rules

```css
/* Arabic text requires larger line height */
[dir="rtl"] {
  line-height: 1.7; /* Increased for Arabic readability */
}

/* Arabic numerals */
[dir="rtl"] .arabic-numerals {
  font-feature-settings: "ss01"; /* Arabic-Indic numerals */
}

/* Mixed content */
[dir="rtl"] .mixed-content {
  direction: rtl;
  unicode-bidi: bidi-override;
}
```

## 5. Spacing System

### Spacing Scale

| Token | Value | Usage |
|-------|-------|-------|
| spacing-0 | 0 | Reset |
| spacing-1 | 4px | Tight spacing |
| spacing-2 | 8px | Small spacing |
| spacing-3 | 12px | Default spacing |
| spacing-4 | 16px | Medium spacing |
| spacing-5 | 20px | Section spacing |
| spacing-6 | 24px | Large spacing |
| spacing-8 | 32px | XL spacing |
| spacing-10 | 40px | 2XL spacing |
| spacing-12 | 48px | 3XL spacing |
| spacing-16 | 64px | 4XL spacing |
| spacing-20 | 80px | 5XL spacing |

### Grid System

```css
/* 12-column grid */
.grid {
  display: grid;
  grid-template-columns: repeat(12, 1fr);
  gap: var(--spacing-6);
}

/* Breakpoints */
/* Mobile: 320px - 767px */
.grid { grid-template-columns: repeat(4, 1fr); gap: var(--spacing-4); }

/* Tablet: 768px - 1023px */
@media (min-width: 768px) { .grid { grid-template-columns: repeat(8, 1fr); gap: var(--spacing-5); } }

/* Desktop: 1024px+ */
@media (min-width: 1024px) { .grid { grid-template-columns: repeat(12, 1fr); gap: var(--spacing-6); } }
```

## 6. Border Radius

| Token | Value | Usage |
|-------|-------|-------|
| radius-sm | 4px | Small elements |
| radius-md | 8px | Buttons, inputs |
| radius-lg | 12px | Cards, modals |
| radius-xl | 16px | Large cards |
| radius-2xl | 24px | Feature cards |
| radius-full | 9999px | Avatars, badges |

## 7. Shadows

| Token | Value | Usage |
|-------|-------|-------|
| shadow-sm | 0 1px 2px rgba(0,0,0,0.05) | Subtle elevation |
| shadow-md | 0 4px 6px rgba(0,0,0,0.1) | Cards, dropdowns |
| shadow-lg | 0 10px 15px rgba(0,0,0,0.1) | Modals, popovers |
| shadow-xl | 0 20px 25px rgba(0,0,0,0.15) | Floating elements |

## 8. Breakpoints

| Name | Min Width | Target |
|------|-----------|--------|
| xs | 320px | Small phones |
| sm | 640px | Large phones |
| md | 768px | Tablets |
| lg | 1024px | Small laptops |
| xl | 1280px | Desktops |
| 2xl | 1536px | Large screens |

## 9. Z-Index Scale

| Token | Value | Usage |
|-------|-------|-------|
| z-base | 0 | Default |
| z-dropdown | 1000 | Dropdowns, tooltips |
| z-sticky | 1020 | Sticky headers |
| z-fixed | 1030 | Fixed navigation |
| z-modal-backdrop | 1040 | Modal overlays |
| z-modal | 1050 | Modals |
| z-popover | 1060 | Popovers |
| z-tooltip | 1070 | Toolbars |
| z-toast | 1080 | Toast notifications |

## 10. Animation & Motion

```css
/* Transitions */
:root {
  --transition-fast: 150ms ease;
  --transition-normal: 200ms ease;
  --transition-slow: 300ms ease;
}

/* Reduced motion preference */
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

## 11. Dark Mode

```css
/* Dark mode tokens */
@media (prefers-color-scheme: dark) {
  :root {
    --bg-primary: #111827;
    --bg-secondary: #1f2937;
    --bg-tertiary: #374151;
    --text-primary: #f9fafb;
    --text-secondary: #9ca3af;
    --border-color: #374151;
  }
}
```

## 12. Related Files

| File | Description |
|------|-------------|
| `05-frontend/component-library.md` | React component library |
| `rtl-design.md` | RTL layout implementation |
| `accessibility.md` | WCAG 2.1 AA requirements |
| `responsive-design.md` | Responsive patterns |
| `style-guide.md` | Writing style guide |

---

*Document Version: 1.0 | Last Updated: 2026-09-13 | Classification: Confidential*
