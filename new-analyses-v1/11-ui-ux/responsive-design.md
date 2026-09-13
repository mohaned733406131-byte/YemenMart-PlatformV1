# Responsive Design — YemenMart

## 1. Overview

YemenMart uses a **mobile-first** responsive design approach. All interfaces are optimized for mobile devices (320px+) and enhanced for larger screens.

## 2. Breakpoints

| Name | Min Width | Max Width | Target Device |
|------|-----------|-----------|---------------|
| xs | 320px | 639px | Small phones |
| sm | 640px | 767px | Large phones |
| md | 768px | 1023px | Tablets portrait |
| lg | 1024px | 1279px | Tablets landscape, small laptops |
| xl | 1280px | 1535px | Desktops |
| 2xl | 1536px | — | Large screens |

### Tailwind Configuration

```javascript
// tailwind.config.js
module.exports = {
  theme: {
    screens: {
      'xs': '320px',
      'sm': '640px',
      'md': '768px',
      'lg': '1024px',
      'xl': '1280px',
      '2xl': '1536px',
    },
  },
}
```

## 3. Grid System

### 12-Column Grid

```css
.grid-container {
  width: 100%;
  margin: 0 auto;
  padding: 0 var(--spacing-4);
}

/* Mobile: 4 columns */
@media (max-width: 639px) {
  .grid-container { max-width: 100%; }
  .col-1, .col-2, .col-3, .col-4 { width: 100%; }
}

/* Tablet: 8 columns */
@media (min-width: 640px) and (max-width: 1023px) {
  .grid-container { max-width: 768px; }
}

/* Desktop: 12 columns */
@media (min-width: 1024px) {
  .grid-container { max-width: 1280px; }
}
```

### Column Spacing

| Breakpoint | Gutter | Margin |
|------------|--------|--------|
| Mobile (xs-sm) | 16px | 16px |
| Tablet (md) | 20px | 24px |
| Desktop (lg+) | 24px | 32px |

## 4. Layout Patterns

### Mobile Layout (320-639px)

```
┌──────────────────┐
│  Hamburger Menu  │
│  Logo            │
│  Search Icon     │
├──────────────────┤
│                  │
│  Content         │
│  (Full Width)    │
│                  │
├──────────────────┤
│  Bottom Nav      │
│  Home|Cart|Account│
└──────────────────┘
```

### Tablet Layout (640-1023px)

```
┌────────────────────────────┐
│  Logo  Search  Cart Account │
├────────────────────────────┤
│                            │
│  Content (2-column grid)   │
│  ┌──────────┐ ┌──────────┐│
│  │  Item 1  │ │  Item 2  ││
│  └──────────┘ └──────────┘│
│  ┌──────────┐ ┌──────────┐│
│  │  Item 3  │ │  Item 4  ││
│  └──────────┘ └──────────┘│
│                            │
└────────────────────────────┘
```

### Desktop Layout (1024px+)

```
┌──────────────────────────────────────────┐
│  Logo  Search Bar  Cart  Account  Lang   │
├──────────────────────────────────────────┤
│                                          │
│  Content (3-4 column grid)               │
│  ┌────────┐ ┌────────┐ ┌────────┐       │
│  │ Item 1 │ │ Item 2 │ │ Item 3 │       │
│  └────────┘ └────────┘ └────────┘       │
│  ┌────────┐ ┌────────┐ ┌────────┐       │
│  │ Item 4 │ │ Item 5 │ │ Item 6 │       │
│  └────────┘ └────────┘ └────────┘       │
│                                          │
└──────────────────────────────────────────┘
```

## 5. Component Responsiveness

### Product Grid

| Breakpoint | Columns | Card Width | Image Aspect Ratio |
|------------|---------|------------|-------------------|
| Mobile | 2 | 50% | 1:1 |
| Tablet | 3 | 33.33% | 4:3 |
| Desktop | 4 | 25% | 4:3 |
| Large | 5 | 20% | 4:3 |

### Navigation

| Breakpoint | Navigation Style |
|------------|-----------------|
| Mobile | Hamburger menu + bottom nav |
| Tablet | Compact top nav with icons |
| Desktop | Full top nav with labels |

### Product Detail Page

| Breakpoint | Layout |
|------------|--------|
| Mobile | Single column, image gallery on top |
| Tablet | 2 columns (image left, details right) |
| Desktop | 3 columns (image gallery, details, actions) |

### Cart / Checkout

| Breakpoint | Layout |
|------------|--------|
| Mobile | Single column, stacked sections |
| Tablet | 2 columns (items left, summary right) |
| Desktop | 3 columns (items, address, payment) |

## 6. Touch Targets

| Element | Minimum Size | Recommended Size |
|---------|-------------|-----------------|
| Buttons | 44x44px | 48x48px |
| Links | 44x44px | 48x48px |
| Form inputs | 44px height | 48px height |
| Checkboxes | 44x44px | 48x48px |
| Radio buttons | 44x44px | 48x48px |
| Icons | 44x44px | 48x48px |

## 7. Responsive Utilities

```typescript
// hooks/useBreakpoint.ts
import { useState, useEffect } from 'react';

type Breakpoint = 'xs' | 'sm' | 'md' | 'lg' | 'xl' | '2xl';

const BREAKPOINTS: Record<Breakpoint, number> = {
  xs: 320,
  sm: 640,
  md: 768,
  lg: 1024,
  xl: 1280,
  '2xl': 1536,
};

export function useBreakpoint(): Breakpoint {
  const [breakpoint, setBreakpoint] = useState<Breakpoint>('xs');

  useEffect(() => {
    function handleResize() {
      const width = window.innerWidth;
      if (width >= 1536) setBreakpoint('2xl');
      else if (width >= 1280) setBreakpoint('xl');
      else if (width >= 1024) setBreakpoint('lg');
      else if (width >= 768) setBreakpoint('md');
      else if (width >= 640) setBreakpoint('sm');
      else setBreakpoint('xs');
    }

    handleResize();
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  return breakpoint;
}

// hooks/useMediaQuery.ts
export function useMediaQuery(query: string): boolean {
  const [matches, setMatches] = useState(false);

  useEffect(() => {
    const media = window.matchMedia(query);
    setMatches(media.matches);
    
    const listener = (e: MediaQueryListEvent) => setMatches(e.matches);
    media.addEventListener('change', listener);
    return () => media.removeEventListener('change', listener);
  }, [query]);

  return matches;
}

// Usage
const isMobile = useMediaQuery('(max-width: 639px)');
const isTablet = useMediaQuery('(min-width: 640px) and (max-width: 1023px)');
const isDesktop = useMediaQuery('(min-width: 1024px)');
```

## 8. Image Responsiveness

```html
<!-- Responsive images with art direction -->
<picture>
  <source media="(min-width: 1024px)" srcset="product-desktop.webp" />
  <source media="(min-width: 640px)" srcset="product-tablet.webp" />
  <img 
    src="product-mobile.webp" 
    alt="Product name"
    loading="lazy"
    width="400"
    height="300"
  />
</picture>

<!-- Responsive image grid -->
<img
  srcset="image-320w.jpg 320w, image-640w.jpg 640w, image-1280w.jpg 1280w"
  sizes="(max-width: 639px) 100vw, (max-width: 1023px) 50vw, 25vw"
  src="image-640w.jpg"
  alt="Product image"
  loading="lazy"
/>
```

## 9. Performance Optimization

| Strategy | Implementation |
|----------|----------------|
| Critical CSS | Inline above-the-fold CSS |
| Lazy loading | `loading="lazy"` on images |
| Code splitting | Route-based lazy loading |
| Image optimization | WebP format, responsive sizes |
| Font loading | `font-display: swap` |
| Prefetching | `<link rel="prefetch">` for next pages |

## 10. Testing Matrix

| Device | OS | Browser | Resolution | Status |
|--------|-----|---------|------------|--------|
| iPhone SE | iOS 16 | Safari | 375x667 | Required |
| iPhone 14 | iOS 16 | Safari | 390x844 | Required |
| Samsung Galaxy S21 | Android 13 | Chrome | 360x800 | Required |
| iPad | iPadOS 16 | Safari | 768x1024 | Required |
| iPad Pro | iPadOS 16 | Safari | 1024x1366 | Required |
| Windows Laptop | Windows 11 | Chrome | 1366x768 | Required |
| Windows Desktop | Windows 11 | Chrome | 1920x1080 | Required |
| MacBook Air | macOS 13 | Safari | 1280x800 | Required |

## 11. Related Files

| File | Description |
|------|-------------|
| `design-system.md` | Design tokens and components |
| `rtl-design.md` | RTL layout implementation |
| `accessibility.md` | WCAG 2.1 AA requirements |
| `05-frontend/component-library.md` | Responsive components |
| `05-frontend/mobile-apps.md` | Mobile app specifics |

---

*Document Version: 1.0 | Last Updated: 2026-09-13 | Classification: Confidential*
