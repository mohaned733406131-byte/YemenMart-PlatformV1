# Responsive Design Strategy - YemenMart

## Overview

Mobile-first responsive design system for YemenMart across all frontends. Supports Arabic-first RTL layout, adaptive components, touch-friendly interactions, optimized image delivery, and progressive enhancement.

## Breakpoints

```typescript
// tailwind.config.ts
export default {
  theme: {
    screens: {
      'xs': '475px',    // Small phones
      'sm': '640px',    // Large phones
      'md': '768px',    // Tablets
      'lg': '1024px',   // Small laptops
      'xl': '1280px',   // Desktops
      '2xl': '1536px',  // Large screens
    },
  },
};
```

| Breakpoint | Width | Target Device |
|------------|-------|---------------|
| Default | < 475px | Small phones |
| xs | 475px+ | Large phones |
| sm | 640px+ | Large phones / small tablets |
| md | 768px+ | Tablets |
| lg | 1024px+ | Small laptops / landscape tablets |
| xl | 1280px+ | Desktops |
| 2xl | 1536px+ | Large desktops |

## Mobile-First Approach

```css
/* Base styles = mobile */
.product-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 0.75rem;
}

/* Tablet */
@media (min-width: 768px) {
  .product-grid {
    grid-template-columns: repeat(3, 1fr);
    gap: 1rem;
  }
}

/* Desktop */
@media (min-width: 1024px) {
  .product-grid {
    grid-template-columns: repeat(4, 1fr);
    gap: 1.5rem;
  }
}
```

```tsx
// React component example
function ProductGrid({ products }: { products: Product[] }) {
  return (
    <div className="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-3 md:gap-4 lg:gap-6">
      {products.map((product) => (
        <ProductCard key={product.id} product={product} />
      ))}
    </div>
  );
}
```

## Layout System

### Main Layout

```tsx
// components/layout/MainLayout.tsx
function MainLayout({ children }: { children: React.ReactNode }) {
  return (
    <div className="min-h-screen bg-white dark:bg-neutral-900">
      <Header />
      <div className="container mx-auto px-4 sm:px-6 lg:px-8">
        <main className="py-4 md:py-6 lg:py-8">
          {children}
        </main>
      </div>
      <Footer />
    </div>
  );
}
```

### Container Sizes

```css
.container {
  width: 100%;
  max-width: 100%;
  margin-left: auto;
  margin-right: auto;
  padding-left: 1rem;
  padding-right: 1rem;
}

@media (min-width: 640px) {
  .container {
    padding-left: 1.5rem;
    padding-right: 1.5rem;
  }
}

@media (min-width: 1024px) {
  .container {
    max-width: 1280px;
  }
}

@media (min-width: 1280px) {
  .container {
    max-width: 1400px;
  }
}
```

## Adaptive Grid Patterns

### Product Cards Grid

```tsx
// Responsive 2/3/4 column grid
<div className="grid grid-cols-2 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-3 sm:gap-4 lg:gap-6">
  {products.map(p => <ProductCard key={p.id} product={p} />)}
</div>
```

### Dashboard Stats

```tsx
// 1/2/4 column responsive stats
<div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-4">
  <StatsCard title="Sales" value={1234} />
  <StatsCard title="Orders" value={56} />
  <StatsCard title="Products" value={89} />
  <StatsCard title="Rating" value={4.8} />
</div>
```

### Content + Sidebar

```tsx
// Stack on mobile, side-by-side on desktop
<div className="flex flex-col lg:flex-row gap-6">
  <main className="flex-1 order-2 lg:order-1">{content}</main>
  <aside className="w-full lg:w-80 order-1 lg:order-2 shrink-0">{sidebar}</aside>
</div>
```

## Touch-Friendly Interactions

### Touch Target Sizes

```css
/* Minimum touch target: 44x44px (Apple HIG) / 48x48dp (Material) */
.touch-target {
  min-height: 44px;
  min-width: 44px;
  display: flex;
  align-items: center;
  justify-content: center;
}

/* Mobile-specific larger targets */
@media (max-width: 767px) {
  .touch-target {
    min-height: 48px;
    min-width: 48px;
  }
}
```

### Swipeable Components

```tsx
// components/ui/SwipeableCarousel.tsx
import { useRef } from 'react';
import { useDrag } from '@use-gesture/react';
import { useSpring, animated } from '@react-spring/web';

function SwipeableCarousel({ items }: { items: any[] }) {
  const [styles, api] = useSpring(() => ({ x: 0 }));
  const ref = useRef<HTMLDivElement>(null);

  const bind = useDrag(
    ({ movement: [mx], velocity: [vx], direction: [dx], cancel }) => {
      const threshold = ref.current ? ref.current.offsetWidth / 3 : 100;
      if (Math.abs(mx) > threshold || vx > 0.5) {
        // Handle swipe
        cancel();
      }
      api.start({ x: 0 });
    },
    { axis: 'x', filterTaps: true }
  );

  return (
    <div className="overflow-hidden touch-pan-x">
      <animated.div {...bind()} style={styles}
        className="flex gap-4 cursor-grab active:cursor-grabbing">
        {items.map((item, i) => (
          <div key={i} className="shrink-0 w-[80vw] sm:w-[60vw] md:w-[40vw] lg:w-[25vw]">
            {item}
          </div>
        ))}
      </animated.div>
    </div>
  );
}
```

### Pull-to-Refresh

```tsx
// components/ui/PullToRefresh.tsx
import { useState, useCallback } from 'react';
import { useDrag } from '@use-gesture/react';
import { useSpring, animated } from '@react-spring/web';

function PullToRefresh({ onRefresh, children }: { onRefresh: () => Promise<void>; children: React.ReactNode }) {
  const [refreshing, setRefreshing] = useState(false);
  const [styles, api] = useSpring(() => ({ y: 0 }));

  const bind = useDrag(
    async ({ movement: [, my], last, cancel }) => {
      if (my < 0) cancel();
      if (last && my > 80) {
        setRefreshing(true);
        api.start({ y: 60 });
        await onRefresh();
        setRefreshing(false);
        api.start({ y: 0 });
      } else {
        api.start({ y: Math.max(0, my * 0.5) });
      }
    },
    { axis: 'y', filterTaps: true, from: () => [0, 0] }
  );

  return (
    <div className="relative overflow-hidden">
      <animated.div {...bind()} style={styles}>
        {refreshing && (
          <div className="absolute top-0 left-0 right-0 flex justify-center py-4">
            <div className="animate-spin w-6 h-6 border-2 border-primary-600 border-t-transparent rounded-full" />
          </div>
        )}
        {children}
      </animated.div>
    </div>
  );
}
```

## RTL/LTR Responsive Layout

### Logical Properties

```css
/* Use CSS Logical Properties for automatic RTL/LTR */
.container-inline {
  margin-inline-start: auto;
  margin-inline-end: auto;
  padding-inline: 1rem;
}

.flex-direction {
  /* Automatically flips in RTL */
  flex-direction: row;
}

.text-align {
  text-align: start; /* Left in LTR, Right in RTL */
}

.border-direction {
  border-inline-start: 1px solid #e5e7eb;
  border-inline-end: 1px solid #e5e7eb;
}
```

### RTL-Aware Component Example

```tsx
// components/product/ProductCard.tsx
function ProductCard({ product }: { product: Product }) {
  const { isRtl } = useRtl();

  return (
    <div className="group bg-white dark:bg-neutral-800 rounded-xl shadow-sm
                    hover:shadow-md transition-all overflow-hidden">
      {/* Image - naturally flows the same in RTL/LTR */}
      <div className="aspect-square relative overflow-hidden">
        <img src={product.image} alt={product.name}
          className="w-full h-full object-cover group-hover:scale-105 transition-transform duration-300" />
        {/* Discount badge - uses logical positioning */}
        {product.discount && (
          <span className="absolute top-2 inline-start-2 bg-red-500 text-white text-xs
                           font-bold px-2 py-1 rounded">
            -{product.discount}%
          </span>
        )}
      </div>

      {/* Content */}
      <div className="p-3">
        {/* Uses text-start for proper RTL text alignment */}
        <h3 className="text-sm font-medium line-clamp-2 mb-2 text-start">
          {product.name}
        </h3>

        {/* Price row */}
        <div className="flex items-baseline gap-2 mb-2">
          <span className="text-lg font-bold text-primary-600">
            {formatPrice(product.price)}
          </span>
          {product.originalPrice && (
            <span className="text-sm text-neutral-400 line-through">
              {formatPrice(product.originalPrice)}
            </span>
          )}
        </div>

        {/* Store name - truncated naturally */}
        <p className="text-xs text-neutral-500 truncate">{product.storeName}</p>
      </div>
    </div>
  );
}
```

## Image Optimization

### Responsive Images

```tsx
// components/ui/OptimizedImage.tsx
import Image from 'next/image';

interface OptimizedImageProps {
  src: string;
  alt: string;
  width?: number;
  height?: number;
  className?: string;
  priority?: boolean;
  sizes?: string;
}

function OptimizedImage({
  src, alt, width = 400, height = 400, className, priority, sizes,
}: OptimizedImageProps) {
  return (
    <Image
      src={src}
      alt={alt}
      width={width}
      height={height}
      priority={priority}
      className={className}
      sizes={sizes || '(max-width: 640px) 50vw, (max-width: 1024px) 33vw, 25vw'}
      placeholder="blur"
      blurDataURL="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVR42mNk+M9QDwADhgGAWjR9awAAAABJRU5ErkJggg=="
    />
  );
}
```

### Image Size Strategy

```typescript
// lib/imageSizes.ts
export const imageSizes = {
  thumbnail: { width: 100, height: 100 },
  card: { width: 400, height: 400 },
  detail: { width: 800, height: 800 },
  hero: { width: 1920, height: 600 },
  avatar: { width: 200, height: 200 },
  banner: { width: 1200, height: 400 },
} as const;

// Responsive sizes attribute
export const responsiveSizes = {
  productGrid: '(max-width: 640px) 50vw, (max-width: 1024px) 33vw, 25vw',
  productDetail: '(max-width: 768px) 100vw, 50vw',
  banner: '100vw',
  avatar: '(max-width: 768px) 48px, 64px',
} as const;
```

### Lazy Loading

```tsx
// components/ui/LazyImage.tsx
'use client';

import { useState, useRef, useEffect } from 'react';

function LazyImage({ src, alt, className }: { src: string; alt: string; className?: string }) {
  const [loaded, setLoaded] = useState(false);
  const [inView, setInView] = useState(false);
  const ref = useRef<HTMLDivElement>(null);

  useEffect(() => {
    const observer = new IntersectionObserver(
      ([entry]) => { if (entry.isIntersecting) setInView(true); },
      { rootMargin: '200px' }
    );
    if (ref.current) observer.observe(ref.current);
    return () => observer.disconnect();
  }, []);

  return (
    <div ref={ref} className={`relative overflow-hidden ${className}`}>
      {inView && (
        <img
          src={src}
          alt={alt}
          loading="lazy"
          onLoad={() => setLoaded(true)}
          className={`w-full h-full object-cover transition-opacity duration-300 ${loaded ? 'opacity-100' : 'opacity-0'}`}
        />
      )}
      {!loaded && (
        <div className="absolute inset-0 bg-neutral-200 dark:bg-neutral-700 animate-pulse" />
      )}
    </div>
  );
}
```

## Responsive Typography

```css
/* Mobile-first typography scale */
.text-responsive {
  font-size: 0.875rem;   /* 14px base */
  line-height: 1.25rem;
}

@media (min-width: 768px) {
  .text-responsive {
    font-size: 1rem;      /* 16px */
    line-height: 1.5rem;
  }
}

@media (min-width: 1024px) {
  .text-responsive {
    font-size: 1.125rem;  /* 18px */
    line-height: 1.75rem;
  }
}
```

```tsx
// Responsive heading component
function ResponsiveHeading({ level = 1, children }: { level?: 1|2|3|4; children: React.ReactNode }) {
  const classes = {
    1: 'text-2xl sm:text-3xl md:text-4xl font-bold',
    2: 'text-xl sm:text-2xl md:text-3xl font-bold',
    3: 'text-lg sm:text-xl md:text-2xl font-semibold',
    4: 'text-base sm:text-lg md:text-xl font-semibold',
  };
  return React.createElement(`h${level}`, { className: classes[level] }, children);
}
```

## Hidden/Visible Patterns

```tsx
// Show/hide at breakpoints
<div className="md:hidden">Mobile only</div>
<div className="hidden md:block">Desktop only</div>
<div className="sm:hidden lg:block">Visible on sm and lg+</div>
<div className="hidden sm:block lg:hidden">Tablet only</div>

// Responsive flex direction
<div className="flex flex-col md:flex-row gap-4">
  <div className="w-full md:w-1/3">Sidebar</div>
  <div className="w-full md:w-2/3">Content</div>
</div>

// Responsive text alignment
<div className="text-center md:text-start">Aligned text</div>
```

## Component Patterns

### Responsive Navigation

```tsx
// components/layout/ResponsiveNav.tsx
function ResponsiveNav() {
  const [mobileOpen, setMobileOpen] = useState(false);

  return (
    <>
      {/* Desktop nav */}
      <nav className="hidden md:flex items-center gap-4">
        <NavLink href="/">Home</NavLink>
        <NavLink href="/categories">Categories</NavLink>
        <NavLink href="/stores">Stores</NavLink>
      </nav>

      {/* Mobile hamburger */}
      <button className="md:hidden p-2" onClick={() => setMobileOpen(true)}>
        <Menu className="w-6 h-6" />
      </button>

      {/* Mobile drawer */}
      {mobileOpen && (
        <div className="fixed inset-0 z-50 md:hidden">
          <div className="absolute inset-0 bg-black/50" onClick={() => setMobileOpen(false)} />
          <nav className="absolute top-0 right-0 h-full w-80 bg-white dark:bg-neutral-900 p-6">
            <NavLink href="/">Home</NavLink>
            <NavLink href="/categories">Categories</NavLink>
            <NavLink href="/stores">Stores</NavLink>
          </nav>
        </div>
      )}
    </>
  );
}
```

### Responsive Cart

```tsx
// Mobile: Bottom drawer | Desktop: Sidebar
function CartDrawer({ isOpen, onClose }: CartDrawerProps) {
  return (
    <>
      {/* Overlay */}
      <div className="fixed inset-0 bg-black/50 z-40" onClick={onClose} />

      {/* Drawer - bottom on mobile, right side on desktop */}
      <div className="fixed bg-white dark:bg-neutral-900 z-50
                      bottom-0 left-0 right-0 max-h-[85vh]
                      md:top-0 md:bottom-0 md:end-0 md:start-auto md:w-96
                      rounded-t-2xl md:rounded-none shadow-2xl
                      flex flex-col">
        {/* Drag handle for mobile */}
        <div className="flex justify-center py-2 md:hidden">
          <div className="w-10 h-1 bg-neutral-300 rounded-full" />
        </div>

        {/* Content */}
        <div className="flex-1 overflow-y-auto p-4">
          <CartContent />
        </div>
      </div>
    </>
  );
}
```

### Responsive Tables

```tsx
// Desktop: Full table | Mobile: Card layout
function ResponsiveTable({ data, columns }: TableProps) {
  const isMobile = useMediaQuery('(max-width: 767px)');

  if (isMobile) {
    return (
      <div className="space-y-3">
        {data.map((row, i) => (
          <div key={i} className="bg-white dark:bg-neutral-800 rounded-xl p-4 shadow-sm">
            {columns.map((col) => (
              <div key={col.key} className="flex justify-between py-1">
                <span className="text-sm text-neutral-500">{col.label}</span>
                <span className="text-sm font-medium">{col.render(row)}</span>
              </div>
            ))}
          </div>
        ))}
      </div>
    );
  }

  return (
    <Table>
      <TableHeader>
        <TableRow>
          {columns.map((col) => <TableHead key={col.key}>{col.label}</TableHead>)}
        </TableRow>
      </TableHeader>
      <TableBody>
        {data.map((row, i) => (
          <TableRow key={i}>
            {columns.map((col) => <TableCell key={col.key}>{col.render(row)}</TableCell>)}
          </TableRow>
        ))}
      </TableBody>
    </Table>
  );
}
```

### Responsive Form

```tsx
// Mobile: Single column | Desktop: Multi-column
function ResponsiveForm() {
  return (
    <form className="space-y-4">
      {/* Full width always */}
      <Input label="Full Name" />

      {/* Side by side on desktop */}
      <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
        <Input label="Email" />
        <Input label="Phone" />
      </div>

      {/* Three columns on desktop */}
      <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-4">
        <Input label="City" />
        <Input label="District" />
        <Input label="Street" />
      </div>

      {/* Full width textarea */}
      <Textarea label="Address Details" rows={3} />

      {/* Responsive button layout */}
      <div className="flex flex-col-reverse sm:flex-row gap-3 sm:justify-end">
        <Button variant="outline" className="w-full sm:w-auto">Cancel</Button>
        <Button className="w-full sm:w-auto">Submit</Button>
      </div>
    </form>
  );
}
```

## Touch vs Mouse Detection

```typescript
// hooks/useInputMode.ts
import { useEffect, useState } from 'react';

export function useInputMode() {
  const [mode, setMode] = useState<'touch' | 'mouse'>('mouse');

  useEffect(() => {
    const handleTouchStart = () => setMode('touch');
    const handleMouseDown = () => setMode('mouse');

    window.addEventListener('touchstart', handleTouchStart, { once: true, passive: true });
    window.addEventListener('mousedown', handleMouseDown, { once: true });

    return () => {
      window.removeEventListener('touchstart', handleTouchStart);
      window.removeEventListener('mousedown', handleMouseDown);
    };
  }, []);

  return mode;
}

// Usage
function ProductCard({ product }: { product: Product }) {
  const inputMode = useInputMode();
  return (
    <div className={`${inputMode === 'touch' ? 'p-4' : 'p-3 hover:shadow-md'}`}>
      {/* Larger padding and no hover on touch */}
    </div>
  );
}
```

## Viewport Height Fix (Mobile)

```css
/* Fix for mobile browsers where 100vh includes the address bar */
.min-h-dvh {
  min-height: 100dvh; /* Dynamic viewport height */
}

/* Fallback for browsers without dvh support */
@supports not (min-height: 100dvh) {
  .min-h-dvh {
    min-height: -webkit-fill-available;
    min-height: fill-available;
  }
}
```

```tsx
// Layout uses dynamic viewport height
function Layout({ children }: { children: React.ReactNode }) {
  return (
    <div className="min-h-dvh flex flex-col">
      <Header />
      <main className="flex-1">{children}</main>
      <Footer />
    </div>
  );
}
```

## Orientation Handling

```css
/* Different layouts for landscape on mobile */
@media (max-width: 767px) and (orientation: landscape) {
  .product-detail-layout {
    flex-direction: row;
  }
  .product-gallery {
    width: 40%;
  }
  .product-info {
    width: 60%;
  }
}
```

## Print Styles

```css
@media print {
  .no-print { display: none !important; }
  .print-only { display: block !important; }
  body { font-size: 12pt; }
  a { text-decoration: underline; color: #000; }
  img { max-width: 100%; }
  .container { max-width: 100%; padding: 0; }
}
```

## Performance Budget

| Metric | Mobile | Desktop |
|--------|--------|---------|
| First Contentful Paint | < 1.5s | < 1.0s |
| Largest Contentful Paint | < 2.5s | < 1.5s |
| Time to Interactive | < 3.0s | < 2.0s |
| Cumulative Layout Shift | < 0.1 | < 0.1 |
| Total Bundle Size | < 200KB | < 300KB |
| Image Size (avg) | < 100KB | < 200KB |

## Testing Responsiveness

```typescript
// __tests__/responsive/ProductGrid.test.tsx
import { render, screen } from '@testing-library/react';
import { ProductGrid } from '@/components/product/ProductGrid';
import { useMediaQuery } from '@/hooks/useMediaQuery';

jest.mock('@/hooks/useMediaQuery');

describe('ProductGrid', () => {
  it('renders 2 columns on mobile', () => {
    (useMediaQuery as jest.Mock).mockReturnValue(true); // isMobile = true
    const { container } = render(<ProductGrid products={mockProducts} />);
    expect(container.querySelector('.grid-cols-2')).toBeInTheDocument();
  });

  it('renders 4 columns on desktop', () => {
    (useMediaQuery as jest.Mock).mockReturnValue(false);
    const { container } = render(<ProductGrid products={mockProducts} />);
    expect(container.querySelector('.lg:grid-cols-4')).toBeInTheDocument();
  });
});
```

```typescript
// Playwright responsive test
import { test, expect } from '@playwright/test';

const viewports = [
  { name: 'iPhone SE', width: 375, height: 667 },
  { name: 'iPad', width: 768, height: 1024 },
  { name: 'Desktop', width: 1280, height: 720 },
];

for (const vp of viewports) {
  test(`Homepage renders correctly on ${vp.name}`, async ({ page }) => {
    await page.setViewportSize({ width: vp.width, height: vp.height });
    await page.goto('/');
    await expect(page).toHaveTitle(/YemenMart/);

    // Check grid columns adapt
    const grid = page.locator('.product-grid');
    await expect(grid).toBeVisible();
  });
}
```

## Accessibility at All Viewports

```tsx
// Accessible responsive component
function AccessibleResponsiveNav() {
  const [isOpen, setIsOpen] = useState(false);
  const buttonRef = useRef<HTMLButtonElement>(null);
  const navRef = useRef<HTMLElement>(null);

  // Trap focus in mobile menu
  useEffect(() => {
    if (isOpen) {
      const focusable = navRef.current?.querySelectorAll('a, button');
      focusable?.[0]?.focus();
    }
  }, [isOpen]);

  return (
    <>
      <button ref={buttonRef}
        className="md:hidden p-2 min-h-[44px] min-w-[44px]"
        onClick={() => setIsOpen(true)}
        aria-expanded={isOpen}
        aria-controls="mobile-nav"
        aria-label="Open navigation menu">
        <Menu className="w-6 h-6" />
      </button>

      {isOpen && (
        <nav ref={navRef} id="mobile-nav" role="navigation" aria-label="Mobile navigation"
          className="fixed inset-0 z-50 bg-white dark:bg-neutral-900 md:hidden">
          <div className="flex items-center justify-between p-4">
            <span className="text-lg font-bold">Menu</span>
            <button onClick={() => { setIsOpen(false); buttonRef.current?.focus(); }}
              className="p-2 min-h-[44px] min-w-[44px]"
              aria-label="Close navigation menu">
              <X className="w-6 h-6" />
            </button>
          </div>
          <div className="p-4 space-y-2">
            <a href="/" className="block p-3 min-h-[44px] rounded-lg hover:bg-neutral-100">Home</a>
            <a href="/categories" className="block p-3 min-h-[44px] rounded-lg hover:bg-neutral-100">Categories</a>
          </div>
        </nav>
      )}
    </>
  );
}
```
