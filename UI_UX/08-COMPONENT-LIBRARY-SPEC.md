# YemenMart Component Library Specification

> **Version:** 1.0.0
> **Last Updated:** 2026-09-13
> **Design System:** YemenMart DS v2
> **RTL-First:** Yes
> **Framework:** React 19 + TypeScript + Tailwind CSS 4 + CVA + Radix UI

---

## Table of Contents

1. [Design Tokens Reference](#design-tokens-reference)
2. [Component Naming Convention](#component-naming-convention)
3. [Navigation Components](#navigation-components)
4. [Form Components](#form-components)
5. [Button Components](#button-components)
6. [Table Components](#table-components)
7. [Card Components](#card-components)
8. [Modal/Overlay Components](#modaloverlay-components)
9. [Feedback Components](#feedback-components)
10. [Chip/Badge Components](#chipbadge-components)
11. [Search Components](#search-components)
12. [Layout Components](#layout-components)
13. [Action Components](#action-components)
14. [Image Components](#image-components)
15. [Chart Components](#chart-components)
16. [Domain-Specific Components](#domain-specific-components)
17. [Shared Types and Utilities](#shared-types--utilities)

---

## Design Tokens Reference

```typescript
// tokens/design-tokens.ts
export const tokens = {
  colors: {
    primary: {
      50: '#eff6ff', 100: '#dbeafe', 200: '#bfdbfe', 300: '#93c5fd',
      400: '#60a5fa', 500: '#1B2A4A', 600: '#2563eb', 700: '#1d4ed8',
      800: '#1e40af', 900: '#1e3a8a',
    },
    success: { 50: '#f0fdf4', 100: '#dcfce7', 500: '#22c55e', 600: '#16a34a', 700: '#15803d' },
    warning: { 50: '#fffbeb', 100: '#fef3c7', 500: '#f59e0b', 600: '#d97706', 700: '#b45309' },
    error: { 50: '#fef2f2', 100: '#fee2e2', 500: '#ef4444', 600: '#dc2626', 700: '#b91c1c' },
    neutral: {
      50: '#f9fafb', 100: '#f3f4f6', 200: '#e5e7eb', 300: '#d1d5db',
      400: '#9ca3af', 500: '#6b7280', 600: '#4b5563', 700: '#374151',
      800: '#1f2937', 900: '#111827',
    },
  },
  spacing: { xs: '0.25rem', sm: '0.5rem', md: '1rem', lg: '1.5rem', xl: '2rem', '2xl': '3rem' },
  radii: { sm: '0.375rem', md: '0.5rem', lg: '0.75rem', xl: '1rem', full: '9999px' },
  typography: {
    fontFamily: { sans: ['IBM Plex Sans Arabic', 'system-ui', 'sans-serif'] },
    fontSize: {
      xs: ['0.75rem', { lineHeight: '1rem' }],
      sm: ['0.875rem', { lineHeight: '1.25rem' }],
      base: ['1rem', { lineHeight: '1.5rem' }],
      lg: ['1.125rem', { lineHeight: '1.75rem' }],
      xl: ['1.25rem', { lineHeight: '1.75rem' }],
      '2xl': ['1.5rem', { lineHeight: '2rem' }],
    },
  },
  shadows: {
    sm: '0 1px 2px 0 rgb(0 0 0 / 0.05)',
    md: '0 4px 6px -1px rgb(0 0 0 / 0.1)',
    lg: '0 10px 15px -3px rgb(0 0 0 / 0.1)',
  },
} as const;

export type Direction = 'ltr' | 'rtl';
export type Size = 'xs' | 'sm' | 'md' | 'lg' | 'xl';
export type Variant = 'primary' | 'secondary' | 'outline' | 'ghost' | 'danger' | 'success';
export type Status = 'default' | 'hover' | 'focus' | 'active' | 'selected' | 'disabled' | 'loading' | 'error' | 'empty' | 'success' | 'readonly';

export interface ComponentMeta {
  id: string; name: string; category: string; purpose: string; usageLocations: string[];
}

export interface ResponsiveValue<T> {
  base: T; sm?: T; md?: T; lg?: T; xl?: T;
}
```

---

## Component Naming Convention

| Pattern | Example | Description |
|---------|---------|-------------|
| `WG-XXX-NNN` | `WG-NAV-001` | Widget ID prefix |
| `WgComponentName` | `WgHeaderNav` | React component name |
| `wg-component-name` | `wg-header-nav` | CSS class prefix |
| `IgComponentNameProps` | `IgHeaderNavProps` | TypeScript props interface |

---

## Navigation Components

---

### WG-NAV-001: Header/Navigation Bar

| Field | Value |
|-------|-------|
| **ID** | WG-NAV-001 |
| **Name** | Header Navigation Bar |
| **Category** | Navigation |
| **Purpose** | Primary navigation bar with logo, search, cart, and user menu |
| **Usage Locations** | All pages, fixed at top |

#### Visual Definition

| Property | Value |
|----------|-------|
| Height | 64px (desktop), 56px (mobile) |
| Background | `white` / `neutral-50` |
| Border | Bottom 1px `neutral-200` |
| Elevation | `shadow-sm` |
| Padding | `px-4 md:px-6 lg:px-8` |
| Position | `fixed top-0 inset-x-0 z-50` |

#### Props/Inputs

```typescript
interface WgHeaderNavProps {
  logo: string;
  logoAlt: string;
  showSearch?: boolean;
  showCart?: boolean;
  showUserMenu?: boolean;
  cartItemCount?: number;
  user?: { name: string; avatar?: string; role: 'customer' | 'vendor' | 'admin' };
  onSearchClick?: () => void;
  onCartClick?: () => void;
  onLogoClick?: () => void;
  onUserMenuClick?: () => void;
  onNotificationClick?: () => void;
  notificationCount?: number;
  variant?: 'default' | 'transparent' | 'dark';
  sticky?: boolean;
  className?: string;
}
```

#### States

| State | Visual Treatment |
|-------|-----------------|
| Default | White background, full opacity |
| Scrolled | Add `shadow-md`, slight blur backdrop |
| Transparent | No background, white text (landing pages) |

#### Interactions

| Trigger | Action |
|---------|--------|
| Click Logo | Navigate to home `/` |
| Click Search | Open search overlay/modal |
| Click Cart | Navigate to `/cart` |
| Click User | Toggle user dropdown menu |
| Click Notification | Navigate to `/notifications` |

#### Accessibility

```typescript
{
  role: 'banner',
  ariaLabel: 'التنقل الرئيسي',
  landmark: 'header',
  skipLink: 'تخطي إلى المحتوى الرئيسي',
  focusOrder: ['skip-link', 'logo', 'search', 'cart', 'notifications', 'user-menu'],
}
```

#### RTL Behavior

- Logo: Right side (`mr-auto` in RTL, `ml-auto` in LTR)
- Search: Centered
- Actions (cart, user): Left side (`ml-auto` in RTL, `mr-auto` in LTR)
- Cart badge: Positioned top-left in RTL, top-right in LTR

#### Responsive Behavior

| Breakpoint | Behavior |
|------------|----------|
| Desktop (>=1024px) | Full nav with inline search bar |
| Tablet (768-1023px) | Condensed, search becomes icon |
| Mobile (<768px) | Hamburger menu, search icon only |

#### TypeScript Implementation

```typescript
import { cva, type VariantProps } from 'class-variance-authority';
import { cn } from '@/lib/utils';
import { Search, ShoppingCart, Bell, User } from 'lucide-react';

const headerNavVariants = cva(
  'fixed inset-x-0 top-0 z-50 flex items-center transition-all duration-200',
  {
    variants: {
      variant: {
        default: 'bg-white border-b border-neutral-200 shadow-sm',
        transparent: 'bg-transparent text-white',
        dark: 'bg-neutral-900 text-white',
      },
    },
    defaultVariants: { variant: 'default' },
  }
);

export function WgHeaderNav({
  logo, logoAlt, showSearch = true, showCart = true, showUserMenu = true,
  cartItemCount = 0, user, onSearchClick, onCartClick, onLogoClick,
  onUserMenuClick, onNotificationClick, notificationCount = 0,
  variant = 'default', sticky = true, className,
}: WgHeaderNavProps) {
  return (
    <header className={cn(headerNavVariants({ variant }), className)} role="banner" aria-label="التنقل الرئيسي">
      <div className="mx-auto flex w-full max-w-7xl items-center justify-between px-4 md:px-6 lg:px-8">
        <a href="/" onClick={(e) => { e.preventDefault(); onLogoClick?.(); }}
           className="flex items-center gap-2" aria-label="YemenMart - الصفحة الرئيسية">
          <img src={logo} alt={logoAlt} className="h-8 w-auto" />
        </a>

        {showSearch && (
          <button onClick={onSearchClick}
            className="hidden md:flex items-center gap-2 rounded-lg border border-neutral-200 bg-neutral-50 px-4 py-2 text-sm text-neutral-500 transition-colors hover:bg-neutral-100"
            aria-label="بحث">
            <Search className="h-4 w-4" />
            <span>بحث في المنتجات...</span>
          </button>
        )}

        <div className="flex items-center gap-2">
          {showSearch && (
            <button onClick={onSearchClick} className="md:hidden p-2 rounded-lg hover:bg-neutral-100" aria-label="بحث">
              <Search className="h-5 w-5" />
            </button>
          )}
          {showCart && (
            <button onClick={onCartClick} className="relative p-2 rounded-lg hover:bg-neutral-100"
              aria-label={cartItemCount > 0 ? "سلة التسوق - " + cartItemCount + " منتجات" : "سلة التسوق"}>
              <ShoppingCart className="h-5 w-5" />
              {cartItemCount > 0 && (
                <span className="absolute -left-1 -top-1 flex h-5 w-5 items-center justify-center rounded-full bg-error-500 text-xs text-white">
                  {cartItemCount > 99 ? '99+' : cartItemCount}
                </span>
              )}
            </button>
          )}
          <button onClick={onNotificationClick} className="relative p-2 rounded-lg hover:bg-neutral-100"
            aria-label={notificationCount > 0 ? "إشعارات - " + notificationCount + " جديدة" : "إشعارات"}>
            <Bell className="h-5 w-5" />
            {notificationCount > 0 && (
              <span className="absolute -left-1 -top-1 flex h-5 w-5 items-center justify-center rounded-full bg-primary-500 text-xs text-white">
                {notificationCount > 99 ? '99+' : notificationCount}
              </span>
            )}
          </button>
          {showUserMenu && user && (
            <button onClick={onUserMenuClick} className="flex items-center gap-2 rounded-lg p-1 hover:bg-neutral-100"
              aria-label="قائمة المستخدم" aria-haspopup="true">
              {user.avatar ? (
                <img src={user.avatar} alt="" className="h-8 w-8 rounded-full object-cover" />
              ) : (
                <div className="flex h-8 w-8 items-center justify-center rounded-full bg-primary-100 text-primary-700">
                  <User className="h-4 w-4" />
                </div>
              )}
            </button>
          )}
        </div>
      </div>
    </header>
  );
}
```

---

### WG-NAV-002: Sidebar Navigation

| Field | Value |
|-------|-------|
| **ID** | WG-NAV-002 |
| **Name** | Sidebar Navigation |
| **Category** | Navigation |
| **Purpose** | Vertical navigation for admin/vendor dashboards |
| **Usage Locations** | Vendor dashboard, Admin panel, Account settings |

#### Visual Definition

| Property | Value |
|----------|-------|
| Width | 260px (expanded), 72px (collapsed) |
| Background | `white` |
| Border | Left 1px `neutral-200` (RTL: right) |
| Item Height | 44px |
| Active Indicator | 3px border on start side + `primary-50` background |

#### Props/Inputs

```typescript
interface WgSidebarNavProps {
  items: SidebarNavItem[];
  activeItemId: string;
  collapsed?: boolean;
  logo?: string;
  logoAlt?: string;
  header?: React.ReactNode;
  footer?: React.ReactNode;
  onToggleCollapse?: () => void;
  onItemClick?: (item: SidebarNavItem) => void;
  variant?: 'default' | 'dark' | 'floating';
  width?: number;
  collapsedWidth?: number;
  className?: string;
}

interface SidebarNavItem {
  id: string; label: string;
  icon: React.ComponentType<{ className?: string }>;
  href?: string; badge?: number | string;
  badgeVariant?: 'default' | 'success' | 'warning' | 'error';
  children?: SidebarNavItem[]; disabled?: boolean;
  tooltip?: string; dividerAfter?: boolean;
}
```

#### RTL Behavior

- Active border indicator: Right side in RTL, left side in LTR
- Icons: Same position (start-aligned)
- Text alignment: Right-aligned in RTL

---

### WG-NAV-003: Bottom Navigation (Mobile)

| Field | Value |
|-------|-------|
| **ID** | WG-NAV-003 |
| **Name** | Bottom Navigation |
| **Category** | Navigation |
| **Purpose** | Mobile primary navigation with 3-5 tab items |
| **Usage Locations** | Mobile app shell, mobile web |

#### Visual Definition

| Property | Value |
|----------|-------|
| Height | 64px |
| Background | `white` |
| Border | Top 1px `neutral-200` |
| Max Items | 5 |
| Badge Position | Top-end of icon |

#### Props/Inputs

```typescript
interface WgBottomNavProps {
  items: BottomNavItem[];
  activeItemId: string;
  onItemClick?: (item: BottomNavItem) => void;
  showLabels?: boolean;
  safeArea?: boolean;
  className?: string;
}

interface BottomNavItem {
  id: string; label: string;
  icon: React.ComponentType<{ className?: string }>;
  activeIcon?: React.ComponentType<{ className?: string }>;
  href?: string; badge?: number;
}
```

---

### WG-NAV-004: Breadcrumbs

| Field | Value |
|-------|-------|
| **ID** | WG-NAV-004 |
| **Name** | Breadcrumbs |
| **Category** | Navigation |
| **Purpose** | Hierarchical page navigation trail |
| **Usage Locations** | Product pages, category pages, settings, order details |

#### Props/Inputs

```typescript
interface WgBreadcrumbsProps {
  items: BreadcrumbItem[];
  separator?: 'chevron' | 'slash' | 'dot';
  maxItems?: number;
  onItemClick?: (item: BreadcrumbItem) => void;
  className?: string;
}

interface BreadcrumbItem {
  id: string; label: string; href?: string;
  icon?: React.ComponentType<{ className?: string }>;
  isCurrent?: boolean;
}
```

#### RTL Behavior

- Separator icons flipped: ChevronLeft in RTL, ChevronRight in LTR
- Items flow from right to left

---

### WG-NAV-005: Language Toggle

| Field | Value |
|-------|-------|
| **ID** | WG-NAV-005 |
| **Name** | Language Toggle |
| **Category** | Navigation |
| **Purpose** | Switch between Arabic and English |
| **Usage Locations** | Header, footer, settings page |

#### Props/Inputs

```typescript
interface WgLanguageToggleProps {
  currentLocale: 'ar' | 'en';
  onLocaleChange: (locale: 'ar' | 'en') => void;
  variant?: 'dropdown' | 'toggle' | 'inline';
  showFlag?: boolean;
  showLabel?: boolean;
  className?: string;
}
```

#### RTL Behavior

- Toggle position flips (Arabic on right in RTL, on left in LTR)
- Text direction changes globally: `document.documentElement.dir`
- Font family switches: IBM Plex Sans Arabic / system sans-serif

---

## Form Components

---

### WG-FRM-001: Text Input

| Field | Value |
|-------|-------|
| **ID** | WG-FRM-001 |
| **Name** | Text Input |
| **Category** | Form |
| **Purpose** | Standard text input field |
| **Usage Locations** | Registration, login, checkout, profile, product forms |

#### Visual Definition

| Property | Value |
|----------|-------|
| Height | 40px (md), 32px (sm), 48px (lg) |
| Border | 1px `neutral-300` |
| Border Radius | `rounded-lg` |
| Focus Ring | `ring-2 ring-primary-500 ring-offset-2` |

#### Props/Inputs

```typescript
interface WgTextInputProps extends React.InputHTMLAttributes<HTMLInputElement> {
  label: string;
  name: string;
  helperText?: string;
  errorMessage?: string;
  hint?: string;
  leftIcon?: React.ComponentType<{ className?: string }>;
  rightIcon?: React.ComponentType<{ className?: string }>;
  rightElement?: React.ReactNode;
  size?: 'sm' | 'md' | 'lg';
  required?: boolean;
  disabled?: boolean;
  readOnly?: boolean;
  loading?: boolean;
  dir?: 'ltr' | 'rtl' | 'auto';
  locale?: 'ar' | 'en';
  onChange?: (e: React.ChangeEvent<HTMLInputElement>) => void;
  onBlur?: (e: React.FocusEvent<HTMLInputElement>) => void;
}
```

#### States

| State | Visual Treatment |
|-------|-----------------|
| Default | Border `neutral-300`, bg `white` |
| Hover | Border `neutral-400` |
| Focus | Border `primary-500`, ring `primary-500`/20 |
| Disabled | Bg `neutral-50`, text `neutral-400` |
| Error | Border `error-500`, ring `error-500`/20 |
| Success | Border `success-500`, check icon right |
| ReadOnly | Bg `neutral-50`, no focus ring |
| Loading | Spinner on right side |

#### RTL Behavior

- Text alignment: `text-end` in RTL
- Icons: Flipped (left icon becomes right icon)
- `dir="rtl"` applied when locale is Arabic

---

### WG-FRM-002: Select/Dropdown

| Field | Value |
|-------|-------|
| **ID** | WG-FRM-002 |
| **Name** | Select/Dropdown |
| **Category** | Form |
| **Purpose** | Single or multi-select dropdown |
| **Usage Locations** | Category selection, governorate/city, filters, forms |

#### Props/Inputs

```typescript
interface WgSelectProps {
  label: string; name: string; options: SelectOption[];
  value?: string | string[]; defaultValue?: string;
  placeholder?: string; multiple?: boolean;
  searchable?: boolean; clearable?: boolean;
  disabled?: boolean; required?: boolean;
  error?: string; helperText?: string;
  size?: 'sm' | 'md' | 'lg'; loading?: boolean;
  locale?: 'ar' | 'en';
  onChange?: (value: string | string[]) => void;
  onOpenChange?: (open: boolean) => void;
  className?: string;
}

interface SelectOption {
  value: string; label: string; disabled?: boolean;
  icon?: React.ComponentType<{ className?: string }>;
  group?: string;
}
```

#### RTL Behavior

- Dropdown opens below (or above if near bottom)
- Checkmark icon on selected item: right side in RTL
- Search input text-aligned to the right

---

### WG-FRM-003: Checkbox

| Field | Value |
|-------|-------|
| **ID** | WG-FRM-003 |
| **Name** | Checkbox |
| **Category** | Form |
| **Purpose** | Boolean selection, multi-select options |
| **Usage Locations** | Filters, terms acceptance, settings, permissions |

#### Props/Inputs

```typescript
interface WgCheckboxProps {
  label: string; name: string; checked?: boolean;
  defaultChecked?: boolean; indeterminate?: boolean;
  disabled?: boolean; required?: boolean;
  error?: string; description?: string;
  size?: 'sm' | 'md' | 'lg';
  onChange?: (checked: boolean) => void;
  className?: string;
}
```

#### States

| State | Visual Treatment |
|-------|-----------------|
| Unchecked | Border `neutral-300`, empty |
| Checked | Bg `primary-500`, white checkmark |
| Indeterminate | Bg `primary-500`, minus icon |
| Disabled | Bg `neutral-100`, border `neutral-200` |
| Error | Border `error-500` |

#### Keyboard Interaction

| Key | Action |
|-----|--------|
| Space | Toggle checked state |
| Tab | Move focus to next checkbox |

---

### WG-FRM-004: Radio Button

| Field | Value |
|-------|-------|
| **ID** | WG-FRM-004 |
| **Name** | Radio Button |
| **Category** | Form |
| **Purpose** | Single selection from a set |
| **Usage Locations** | Payment method, delivery options, form options |

#### Props/Inputs

```typescript
interface WgRadioGroupProps {
  label: string; name: string; options: RadioOption[];
  value?: string; defaultValue?: string;
  orientation?: 'vertical' | 'horizontal';
  disabled?: boolean; error?: string;
  onChange?: (value: string) => void;
  className?: string;
}

interface RadioOption {
  value: string; label: string; description?: string;
  disabled?: boolean;
  icon?: React.ComponentType<{ className?: string }>;
}
```

---

### WG-FRM-005: Textarea

| Field | Value |
|-------|-------|
| **ID** | WG-FRM-005 |
| **Name** | Textarea |
| **Category** | Form |
| **Purpose** | Multi-line text input |
| **Usage Locations** | Product descriptions, reviews, messages, support tickets |

#### Props/Inputs

```typescript
interface WgTextareaProps extends React.TextareaHTMLAttributes<HTMLTextAreaElement> {
  label: string; name: string;
  helperText?: string; errorMessage?: string;
  maxLength?: number; showCounter?: boolean;
  autoResize?: boolean; minRows?: number; maxRows?: number;
  required?: boolean; locale?: 'ar' | 'en';
  onChange?: (e: React.ChangeEvent<HTMLTextAreaElement>) => void;
}
```

---

### WG-FRM-006: Phone Input (with +967 prefix)

| Field | Value |
|-------|-------|
| **ID** | WG-FRM-006 |
| **Name** | Phone Input |
| **Category** | Form |
| **Purpose** | Phone number input with Yemen +967 prefix |
| **Usage Locations** | Registration, login, checkout, profile, vendor registration |

#### Props/Inputs

```typescript
interface WgPhoneInputProps {
  label: string; name: string; value?: string;
  defaultCountry?: string; countries?: CountryCode[];
  placeholder?: string; required?: boolean; disabled?: boolean;
  error?: string; helperText?: string; locale?: 'ar' | 'en';
  onChange?: (value: string, countryCode: string) => void;
  onBlur?: (e: React.FocusEvent<HTMLInputElement>) => void;
  className?: string;
}

interface CountryCode {
  code: string; name: string; dialCode: string;
  flag: string; maxLength: number;
}
```

#### RTL Behavior

- Country code selector on the left (RTL: right)
- Phone number input fills remaining space
- LTR direction for phone number digits regardless of locale

---

### WG-FRM-007: OTP Input (6-digit)

| Field | Value |
|-------|-------|
| **ID** | WG-FRM-007 |
| **Name** | OTP Input |
| **Category** | Form |
| **Purpose** | 6-digit one-time password verification |
| **Usage Locations** | Login verification, registration, phone verification, password reset |

#### Props/Inputs

```typescript
interface WgOtpInputProps {
  length?: number; value?: string; autoFocus?: boolean;
  disabled?: boolean; loading?: boolean; error?: string;
  size?: 'sm' | 'md' | 'lg'; locale?: 'ar' | 'en';
  onComplete?: (otp: string) => void; onChange?: (otp: string) => void;
  onResend?: () => void; resendTimer?: number;
  label?: string; className?: string;
}
```

#### Keyboard Interaction

| Key | Action |
|-----|--------|
| 0-9 | Enter digit, auto-advance |
| Backspace | Clear current, move to previous |
| Delete | Clear current |
| ArrowLeft/Right | Move between inputs |
| Paste (Ctrl+V) | Fill all inputs from clipboard |

---

### WG-FRM-008: Date Picker

| Field | Value |
|-------|-------|
| **ID** | WG-FRM-008 |
| **Name** | Date Picker |
| **Category** | Form |
| **Purpose** | Date selection (Gregorian and Hijri) |
| **Usage Locations** | Checkout delivery date, birth date, order filters, product expiry |

#### Props/Inputs

```typescript
interface WgDatePickerProps {
  label: string; name: string; value?: Date;
  placeholder?: string; minDate?: Date; maxDate?: Date;
  disabled?: boolean; required?: boolean; error?: string;
  locale?: 'ar' | 'en'; calendarType?: 'gregorian' | 'hijri';
  showTime?: boolean; format?: string;
  onChange?: (date: Date | null) => void;
  className?: string;
}
```

#### RTL Behavior

- Calendar opens below input (or above if near bottom)
- Day headers: Saturday to Friday (Arabic week)
- Month/year navigation: Flipped arrows

---

### WG-FRM-009: File Upload

| Field | Value |
|-------|-------|
| **ID** | WG-FRM-009 |
| **Name** | File Upload |
| **Category** | Form |
| **Purpose** | Single or multiple file upload with preview |
| **Usage Locations** | Profile avatar, product images, store logo, documents |

#### Props/Inputs

```typescript
interface WgFileUploadProps {
  label: string; name: string; accept?: string;
  multiple?: boolean; maxSize?: number; maxFiles?: number;
  disabled?: boolean; required?: boolean; error?: string;
  variant?: 'dropzone' | 'button' | 'avatar';
  preview?: boolean; locale?: 'ar' | 'en';
  onUpload?: (files: File[]) => void;
  onRemove?: (index: number) => void;
  className?: string;
}
```

#### States

| State | Visual Treatment |
|-------|-----------------|
| Default | Dashed border, icon + text |
| Hover/DragOver | Border `primary-500`, bg `primary-50` |
| Uploading | Progress bar overlay |
| Error | Border `error-500`, error message |
| Success | Checkmark, file thumbnail |

---

### WG-FRM-010: Search Input

| Field | Value |
|-------|-------|
| **ID** | WG-FRM-010 |
| **Name** | Search Input |
| **Category** | Form |
| **Purpose** | Text input with search icon, clear button, and debounced search |
| **Usage Locations** | Header search, product search, customer search |

#### Props/Inputs

```typescript
interface WgSearchInputProps {
  placeholder?: string; value?: string; debounceMs?: number;
  loading?: boolean; showRecent?: boolean; recentSearches?: string[];
  size?: 'sm' | 'md' | 'lg'; autoFocus?: boolean;
  locale?: 'ar' | 'en';
  onSearch?: (query: string) => void; onClear?: () => void;
  onSelectRecent?: (query: string) => void;
  className?: string;
}
```

#### Keyboard Interaction

| Key | Action |
|-----|--------|
| Enter | Trigger search |
| Escape | Clear input or close suggestions |
| ArrowUp/Down | Navigate suggestions |
| Backspace on empty | Clear and close |

---

## Button Components

---

### WG-BTN-001: Button

| Field | Value |
|-------|-------|
| **ID** | WG-BTN-001 |
| **Name** | Button |
| **Category** | Button |
| **Purpose** | Primary action trigger with multiple variants |
| **Usage Locations** | All pages, forms, modals, navigation |

#### Props/Inputs

```typescript
interface WgButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: 'primary' | 'secondary' | 'outline' | 'ghost' | 'danger' | 'success';
  size?: 'xs' | 'sm' | 'md' | 'lg' | 'xl';
  loading?: boolean; loadingText?: string;
  leftIcon?: React.ComponentType<{ className?: string }>;
  rightIcon?: React.ComponentType<{ className?: string }>;
  fullWidth?: boolean; asChild?: boolean;
  locale?: 'ar' | 'en';
}
```

#### States

| State | Visual Treatment |
|-------|-----------------|
| Default | Full opacity, bg per variant |
| Hover | Darker shade, slight scale(1.02) |
| Focus | Ring `primary-500`, ring-offset 2 |
| Active/Pressed | Scale(0.98), darker shade |
| Loading | Spinner, text change, disabled |
| Disabled | Opacity 0.5, cursor not-allowed |

#### TypeScript Implementation

```typescript
import { forwardRef } from 'react';
import { cva, type VariantProps } from 'class-variance-authority';
import { cn } from '@/lib/utils';
import { Loader2 } from 'lucide-react';

const buttonVariants = cva(
  'inline-flex items-center justify-center gap-2 rounded-lg font-medium transition-all focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50 active:scale-[0.98]',
  {
    variants: {
      variant: {
        primary: 'bg-primary-500 text-white hover:bg-primary-600 focus-visible:ring-primary-500',
        secondary: 'bg-neutral-100 text-neutral-700 hover:bg-neutral-200 focus-visible:ring-neutral-400',
        outline: 'border border-neutral-300 bg-white text-neutral-700 hover:bg-neutral-50 focus-visible:ring-primary-500',
        ghost: 'text-neutral-600 hover:bg-neutral-100 hover:text-neutral-900 focus-visible:ring-neutral-400',
        danger: 'bg-error-500 text-white hover:bg-error-600 focus-visible:ring-error-500',
        success: 'bg-success-500 text-white hover:bg-success-600 focus-visible:ring-success-500',
      },
      size: {
        xs: 'h-7 px-2 text-xs rounded-md',
        sm: 'h-8 px-3 text-sm',
        md: 'h-10 px-4 text-sm',
        lg: 'h-12 px-6 text-base',
        xl: 'h-14 px-8 text-lg',
      },
      fullWidth: { true: 'w-full' },
    },
    defaultVariants: { variant: 'primary', size: 'md' },
  }
);

export const WgButton = forwardRef<HTMLButtonElement, WgButtonProps>(
  ({ className, variant, size, fullWidth, loading = false, loadingText, leftIcon: LeftIcon, rightIcon: RightIcon, disabled, children, ...props }, ref) => {
    return (
      <button ref={ref} className={cn(buttonVariants({ variant, size, fullWidth, className }))} disabled={disabled || loading} aria-busy={loading} {...props}>
        {loading ? (<><Loader2 className="h-4 w-4 animate-spin" />{loadingText && <span>{loadingText}</span></>)
          : (<>{LeftIcon && <LeftIcon className="h-4 w-4" />}{children}{RightIcon && <RightIcon className="h-4 w-4" />}</>)}
      </button>
    );
  }
);
WgButton.displayName = 'WgButton';
```

---

### WG-BTN-002: Icon Button

| Field | Value |
|-------|-------|
| **ID** | WG-BTN-002 |
| **Name** | Icon Button |
| **Category** | Button |
| **Purpose** | Square button with only an icon |
| **Usage Locations** | Toolbars, card actions, form actions |

#### Props/Inputs

```typescript
interface WgIconButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  icon: React.ComponentType<{ className?: string }>;
  variant?: 'primary' | 'secondary' | 'outline' | 'ghost' | 'danger';
  size?: 'xs' | 'sm' | 'md' | 'lg';
  tooltip?: string; loading?: boolean;
  'aria-label': string;
}
```

---

### WG-BTN-003: Floating Action Button

| Field | Value |
|-------|-------|
| **ID** | WG-BTN-003 |
| **Name** | Floating Action Button |
| **Category** | Button |
| **Purpose** | Primary action floating at bottom of screen |
| **Usage Locations** | Mobile: add product, create order, contact support |

#### Props/Inputs

```typescript
interface WgFabProps {
  icon: React.ComponentType<{ className?: string }>;
  label: string; onClick?: () => void;
  variant?: 'primary' | 'secondary' | 'success' | 'danger';
  size?: 'md' | 'lg'; extended?: boolean;
  position?: 'bottom-end' | 'bottom-center';
  className?: string;
}
```

#### RTL Behavior

- Default position: `bottom-end` (bottom-right in LTR, bottom-left in RTL)

---

## Table Components

---

### WG-TBL-001: Data Table

| Field | Value |
|-------|-------|
| **ID** | WG-TBL-001 |
| **Name** | Data Table |
| **Category** | Table |
| **Purpose** | Tabular data display with sorting, filtering, pagination |
| **Usage Locations** | Orders list, products list, customers, vendors, reports |

#### Props/Inputs

```typescript
interface WgDataTableProps<T> {
  columns: ColumnDef<T>[]; data: T[];
  loading?: boolean; totalRows?: number;
  selectable?: boolean; selectableKey?: keyof T;
  onSelectionChange?: (selectedRows: T[]) => void;
  emptyState?: React.ReactNode; stickyHeader?: boolean;
  striped?: boolean; compact?: boolean;
  locale?: 'ar' | 'en'; className?: string;
}

interface ColumnDef<T> {
  id: string; header: string | React.ReactNode;
  accessorKey?: keyof T; accessorFn?: (row: T) => React.ReactNode;
  sortable?: boolean; filterable?: boolean;
  width?: string | number; align?: 'start' | 'center' | 'end';
  cell?: (row: T) => React.ReactNode;
}
```

#### RTL Behavior

- Sticky columns: First column sticky to the right in RTL
- Sort icons: Flipped (ascending arrow points up in both)
- Horizontal scroll: Starts from the right in RTL

---

### WG-TBL-002: Table Pagination

| Field | Value |
|-------|-------|
| **ID** | WG-TBL-002 |
| **Name** | Table Pagination |
| **Category** | Table |
| **Purpose** | Page navigation for table data |
| **Usage Locations** | Below data tables |

#### Props/Inputs

```typescript
interface WgTablePaginationProps {
  currentPage: number; totalPages: number; totalItems: number;
  pageSize: number; pageSizeOptions?: number[];
  showPageSize?: boolean; showTotal?: boolean;
  locale?: 'ar' | 'en';
  onPageChange?: (page: number) => void;
  onPageSizeChange?: (size: number) => void;
  className?: string;
}
```

---

### WG-TBL-003: Table Filters

| Field | Value |
|-------|-------|
| **ID** | WG-TBL-003 |
| **Name** | Table Filters |
| **Category** | Table |
| **Purpose** | Filter controls for table data |
| **Usage Locations** | Above data tables (orders, products, customers) |

#### Props/Inputs

```typescript
interface WgTableFiltersProps {
  filters: FilterConfig[]; values: Record<string, any>;
  onChange: (filters: Record<string, any>) => void;
  onReset?: () => void; locale?: 'ar' | 'en';
  className?: string;
}

interface FilterConfig {
  id: string; label: string;
  type: 'text' | 'select' | 'date-range' | 'number-range' | 'multi-select';
  options?: { value: string; label: string }[];
  placeholder?: string;
}
```

---

### WG-TBL-004: Table Sort

| Field | Value |
|-------|-------|
| **ID** | WG-TBL-004 |
| **Name** | Table Sort |
| **Category** | Table |
| **Purpose** | External sort controls for table columns |
| **Usage Locations** | Advanced table headers, mobile sort options |

#### Props/Inputs

```typescript
interface WgTableSortProps {
  columns: { id: string; label: string }[];
  activeSort?: { key: string; direction: 'asc' | 'desc' };
  onSortChange?: (sort: { key: string; direction: 'asc' | 'desc' } | null) => void;
  locale?: 'ar' | 'en'; className?: string;
}
```

---

## Card Components

---

### WG-CRD-001: Product Card

| Field | Value |
|-------|-------|
| **ID** | WG-CRD-001 |
| **Name** | Product Card |
| **Category** | Card |
| **Purpose** | Display product info in grid/list views |
| **Usage Locations** | Category pages, search results, home page, wishlists |

#### Visual Definition

| Property | Value |
|----------|-------|
| Border Radius | `rounded-xl` |
| Border | 1px `neutral-200` |
| Shadow | `shadow-sm` (hover: `shadow-md`) |
| Image Aspect | 4:3 or 1:1 |

#### Props/Inputs

```typescript
interface WgProductCardProps {
  product: {
    id: string; name: string; nameAr?: string; slug: string;
    image: string; images?: string[];
    price: number; originalPrice?: number; currency?: string;
    rating?: number; reviewCount?: number;
    seller?: { name: string; avatar?: string; verified?: boolean };
    badge?: string; badgeVariant?: 'new' | 'sale' | 'bestseller';
    inStock?: boolean; discount?: number;
  };
  variant?: 'grid' | 'list' | 'compact';
  showSeller?: boolean; showRating?: boolean;
  showBadge?: boolean; showDiscount?: boolean;
  locale?: 'ar' | 'en';
  onAddToCart?: (productId: string) => void;
  onAddToWishlist?: (productId: string) => void;
  onShare?: (productId: string) => void;
  onClick?: (product: any) => void;
  isWishlisted?: boolean; className?: string;
}
```

#### States

| State | Visual Treatment |
|-------|-----------------|
| Default | White bg, neutral border |
| Hover | Shadow increases, slight translate-y(-2px) |
| Out of stock | Image grayscale, overlay |
| Loading | Skeleton loader placeholder |

#### RTL Behavior

- Image: Same position (top in grid, start in list)
- Text: Always right-aligned
- Sale badge: Top-left in RTL, top-right in LTR
- Price: Always left-aligned (numbers are LTR)

---

### WG-CRD-002: Store Card

| Field | Value |
|-------|-------|
| **ID** | WG-CRD-002 |
| **Name** | Store Card |
| **Category** | Card |
| **Purpose** | Display vendor store information |
| **Usage Locations** | Store listings, vendor search, featured stores |

#### Props/Inputs

```typescript
interface WgStoreCardProps {
  store: {
    id: string; name: string; nameAr?: string;
    logo: string; banner?: string; description?: string;
    rating?: number; reviewCount?: number;
    productCount?: number; followerCount?: number;
    verified?: boolean; category?: string; location?: string;
  };
  variant?: 'featured' | 'compact' | 'detailed';
  locale?: 'ar' | 'en';
  onFollow?: (storeId: string) => void;
  onClick?: (store: any) => void;
  isFollowing?: boolean; className?: string;
}
```

---

### WG-CRD-003: Order Card

| Field | Value |
|-------|-------|
| **ID** | WG-CRD-003 |
| **Name** | Order Card |
| **Category** | Card |
| **Purpose** | Display order summary with status |
| **Usage Locations** | My orders, order history, order tracking |

#### Props/Inputs

```typescript
interface WgOrderCardProps {
  order: {
    id: string; orderNumber: string; date: string;
    status: 'pending' | 'confirmed' | 'processing' | 'shipped' | 'delivered' | 'cancelled' | 'returned';
    items: { name: string; image: string; quantity: number; price: number }[];
    total: number; currency?: string; storeName?: string;
    deliveryDate?: string; trackingNumber?: string;
  };
  locale?: 'ar' | 'en';
  onTrack?: (orderId: string) => void;
  onReorder?: (orderId: string) => void;
  onCancel?: (orderId: string) => void;
  onReview?: (orderId: string) => void;
  onClick?: (order: any) => void; className?: string;
}
```

#### Status Colors

| Status | Color | Label (AR) |
|--------|-------|------------|
| pending | Warning | قيد الانتظار |
| confirmed | Primary | مؤكد |
| processing | Primary | قيد التجهيز |
| shipped | Info | تم الشحن |
| delivered | Success | تم التوصيل |
| cancelled | Error | ملغي |
| returned | Neutral | مرتجع |

---

### WG-CRD-004: User Card

| Field | Value |
|-------|-------|
| **ID** | WG-CRD-004 |
| **Name** | User Card |
| **Category** | Card |
| **Purpose** | Display user/customer/vendor profile summary |
| **Usage Locations** | Customer management, vendor management, team members |

#### Props/Inputs

```typescript
interface WgUserCardProps {
  user: {
    id: string; name: string; email: string; phone?: string;
    avatar?: string; role: 'customer' | 'vendor' | 'admin' | 'driver';
    status: 'active' | 'inactive' | 'suspended';
    joinDate: string; orderCount?: number; totalSpent?: number;
  };
  variant?: 'compact' | 'detailed'; locale?: 'ar' | 'en';
  onEdit?: (userId: string) => void;
  onSuspend?: (userId: string) => void;
  onClick?: (user: any) => void; className?: string;
}
```

---

### WG-CRD-005: Stat Card (KPI)

| Field | Value |
|-------|-------|
| **ID** | WG-CRD-005 |
| **Name** | Stat Card |
| **Category** | Card |
| **Purpose** | Display key performance indicators |
| **Usage Locations** | Dashboard, analytics, reports |

#### Props/Inputs

```typescript
interface WgStatCardProps {
  title: string; value: string | number;
  change?: { value: number; direction: 'up' | 'down' | 'neutral'; period?: string };
  icon?: React.ComponentType<{ className?: string }>; iconBg?: string;
  format?: 'number' | 'currency' | 'percentage'; currency?: string;
  loading?: boolean; locale?: 'ar' | 'en';
  onClick?: () => void; className?: string;
}
```

#### RTL Behavior

- Icon on the right, text on the left (swapped from LTR)
- Change indicator: Arrow direction unchanged, but positioned correctly
---

## Modal/Overlay Components

---

### WG-MDL-001: Modal Dialog

| Field | Value |
|-------|-------|
| **ID** | WG-MDL-001 |
| **Name** | Modal Dialog |
| **Category** | Modal |
| **Purpose** | Focused overlay for important actions |
| **Usage Locations** | Product quick view, forms, confirmations, filters |

#### Props/Inputs

```typescript
interface WgModalProps {
  open: boolean; onClose: () => void;
  title?: string; description?: string;
  size?: 'sm' | 'md' | 'lg' | 'xl' | 'full';
  closeOnOverlayClick?: boolean;
  closeOnEscape?: boolean;
  showCloseButton?: boolean;
  footer?: React.ReactNode; locale?: 'ar' | 'en';
  children: React.ReactNode; className?: string;
}
```

#### Accessibility

- role: dialog, ariaModal: true
- ariaLabelledBy: 'modal-title'
- ariaDescribedBy: 'modal-description'
- focusTrap: true, returnFocus: true
- escapeKey: 'close'

#### Keyboard Interaction

| Key | Action |
|-----|--------|
| Escape | Close modal |
| Tab | Cycle through focusable elements (trapped) |
| Shift+Tab | Reverse cycle |

---

### WG-MDL-002: Drawer (Side Panel)

| Field | Value |
|-------|-------|
| **ID** | WG-MDL-002 |
| **Name** | Drawer |
| **Category** | Modal |
| **Purpose** | Slide-in panel for supplementary content |
| **Usage Locations** | Filters, cart preview, product details, settings |

#### Props/Inputs

```typescript
interface WgDrawerProps {
  open: boolean; onClose: () => void;
  title?: string; description?: string;
  side?: 'start' | 'end' | 'top' | 'bottom';
  size?: 'sm' | 'md' | 'lg' | 'xl' | 'full';
  closeOnOverlayClick?: boolean;
  footer?: React.ReactNode; locale?: 'ar' | 'en';
  children: React.ReactNode; className?: string;
}
```

#### RTL Behavior

- `side="start"`: Right side in RTL, left in LTR
- `side="end"`: Left side in RTL, right in LTR

---

### WG-MDL-003: Bottom Sheet

| Field | Value |
|-------|-------|
| **ID** | WG-MDL-003 |
| **Name** | Bottom Sheet |
| **Category** | Modal |
| **Purpose** | Mobile-first slide-up panel |
| **Usage Locations** | Mobile filters, mobile actions, mobile menus |

#### Props/Inputs

```typescript
interface WgBottomSheetProps {
  open: boolean; onClose: () => void;
  title?: string; snapPoints?: number[];
  initialSnap?: number; dismissible?: boolean;
  footer?: React.ReactNode; locale?: 'ar' | 'en';
  children: React.ReactNode; className?: string;
}
```

#### Interactions

| Gesture | Action |
|---------|--------|
| Swipe up | Expand to next snap point |
| Swipe down | Collapse to previous snap point or dismiss |
| Tap overlay | Dismiss |

---

### WG-MDL-004: Confirmation Dialog

| Field | Value |
|-------|-------|
| **ID** | WG-MDL-004 |
| **Name** | Confirmation Dialog |
| **Category** | Modal |
| **Purpose** | Confirm destructive or important actions |
| **Usage Locations** | Delete actions, cancel orders, remove items |

#### Props/Inputs

```typescript
interface WgConfirmDialogProps {
  open: boolean; onClose: () => void; onConfirm: () => void;
  title: string; message: string;
  confirmLabel?: string; cancelLabel?: string;
  variant?: 'danger' | 'warning' | 'info';
  loading?: boolean; locale?: 'ar' | 'en';
}
```

---

### WG-MDL-005: Alert Dialog

| Field | Value |
|-------|-------|
| **ID** | WG-MDL-005 |
| **Name** | Alert Dialog |
| **Category** | Modal |
| **Purpose** | Non-dismissable important notification |
| **Usage Locations** | System errors, maintenance notices, required actions |

#### Props/Inputs

```typescript
interface WgAlertDialogProps {
  open: boolean; onAction: () => void;
  title: string; message: string;
  actionLabel?: string;
  variant?: 'error' | 'warning' | 'info' | 'success';
  icon?: React.ComponentType<{ className?: string }>;
  locale?: 'ar' | 'en';
}
```

---

## Feedback Components

---

### WG-FDB-001: Toast Notification

| Field | Value |
|-------|-------|
| **ID** | WG-FDB-001 |
| **Name** | Toast Notification |
| **Category** | Feedback |
| **Purpose** | Brief, non-blocking notification |
| **Usage Locations** | Success actions, error messages, updates |

#### Props/Inputs

```typescript
interface WgToastProps {
  id: string; title?: string; message: string;
  variant?: 'success' | 'error' | 'warning' | 'info';
  duration?: number;
  action?: { label: string; onClick: () => void };
  dismissible?: boolean;
  position?: 'top-start' | 'top-center' | 'top-end' | 'bottom-start' | 'bottom-center' | 'bottom-end';
  locale?: 'ar' | 'en';
  onDismiss?: (id: string) => void;
}
```

#### RTL Behavior

- Position: `top-end` becomes top-left in RTL
- Swipe to dismiss: Swipe left in RTL, right in LTR

---

### WG-FDB-002: Alert Banner

| Field | Value |
|-------|-------|
| **ID** | WG-FDB-002 |
| **Name** | Alert Banner |
| **Category** | Feedback |
| **Purpose** | Page-level alert with dismiss and action |
| **Usage Locations** | System notifications, warnings, maintenance notices |

#### Props/Inputs

```typescript
interface WgAlertBannerProps {
  variant?: 'info' | 'success' | 'warning' | 'error';
  title?: string; message: string;
  action?: { label: string; onClick: () => void };
  dismissible?: boolean;
  icon?: React.ComponentType<{ className?: string }>;
  locale?: 'ar' | 'en';
  onDismiss?: () => void; className?: string;
}
```

---

### WG-FDB-003: Snackbar

| Field | Value |
|-------|-------|
| **ID** | WG-FDB-003 |
| **Name** | Snackbar |
| **Category** | Feedback |
| **Purpose** | Bottom-anchored notification with optional action |
| **Usage Locations** | Undo actions, quick confirmations |

#### Props/Inputs

```typescript
interface WgSnackbarProps {
  open: boolean; message: string;
  action?: { label: string; onClick: () => void };
  duration?: number; dismissible?: boolean;
  position?: 'bottom-center' | 'bottom-start' | 'bottom-end';
  locale?: 'ar' | 'en';
  onDismiss?: () => void; className?: string;
}
```

---

### WG-FDB-004: Progress Bar

| Field | Value |
|-------|-------|
| **ID** | WG-FDB-004 |
| **Name** | Progress Bar |
| **Category** | Feedback |
| **Purpose** | Visual indicator of completion or loading |
| **Usage Locations** | File uploads, multi-step forms, downloads |

#### Props/Inputs

```typescript
interface WgProgressBarProps {
  value: number; max?: number;
  variant?: 'default' | 'success' | 'warning' | 'error' | 'striped';
  size?: 'sm' | 'md' | 'lg';
  showLabel?: boolean; label?: string;
  indeterminate?: boolean;
  locale?: 'ar' | 'en'; className?: string;
}
```

---

### WG-FDB-005: Spinner/Loader

| Field | Value |
|-------|-------|
| **ID** | WG-FDB-005 |
| **Name** | Spinner/Loader |
| **Category** | Feedback |
| **Purpose** | Inline or full-page loading indicator |
| **Usage Locations** | Buttons, page transitions, data fetching |

#### Props/Inputs

```typescript
interface WgSpinnerProps {
  size?: 'xs' | 'sm' | 'md' | 'lg' | 'xl';
  variant?: 'default' | 'primary' | 'white';
  label?: string;
  overlay?: boolean;
  className?: string;
}
```

---

### WG-FDB-006: Skeleton Loader

| Field | Value |
|-------|-------|
| **ID** | WG-FDB-006 |
| **Name** | Skeleton Loader |
| **Category** | Feedback |
| **Purpose** | Placeholder content during loading |
| **Usage Locations** | Product cards, table rows, feed items |

#### Props/Inputs

```typescript
interface WgSkeletonProps {
  variant?: 'text' | 'circular' | 'rectangular' | 'card';
  width?: string | number; height?: string | number;
  lines?: number; animate?: boolean;
  className?: string;
}
```

---

### WG-FDB-007: Empty State

| Field | Value |
|-------|-------|
| **ID** | WG-FDB-007 |
| **Name** | Empty State |
| **Category** | Feedback |
| **Purpose** | Display when no data is available |
| **Usage Locations** | Empty lists, no search results, first-time users |

#### Props/Inputs

```typescript
interface WgEmptyStateProps {
  icon?: React.ComponentType<{ className?: string }>;
  title: string; description?: string;
  action?: { label: string; onClick: () => void };
  illustration?: string;
  variant?: 'default' | 'compact' | 'illustrated';
  locale?: 'ar' | 'en'; className?: string;
}
```

---

### WG-FDB-008: Error State

| Field | Value |
|-------|-------|
| **ID** | WG-FDB-008 |
| **Name** | Error State |
| **Category** | Feedback |
| **Purpose** | Display when an error occurs |
| **Usage Locations** | Failed loads, API errors, 404 pages |

#### Props/Inputs

```typescript
interface WgErrorStateProps {
  title?: string; message: string;
  code?: string | number;
  retry?: { label?: string; onClick: () => void };
  support?: { label?: string; onClick: () => void };
  illustration?: string;
  locale?: 'ar' | 'en'; className?: string;
}
```

---

## Chip/Badge Components

---

### WG-CHP-001: Badge/Tag

| Field | Value |
|-------|-------|
| **ID** | WG-CHP-001 |
| **Name** | Badge/Tag |
| **Category** | Chip |
| **Purpose** | Status indicator, category label, count display |
| **Usage Locations** | Product tags, notification counts, status indicators |

#### Props/Inputs

```typescript
interface WgBadgeProps {
  children: React.ReactNode;
  variant?: 'default' | 'primary' | 'success' | 'warning' | 'error' | 'info' | 'outline';
  size?: 'sm' | 'md' | 'lg';
  removable?: boolean; onRemove?: () => void;
  icon?: React.ComponentType<{ className?: string }>;
  dot?: boolean; className?: string;
}
```

---

### WG-CHP-002: Status Chip

| Field | Value |
|-------|-------|
| **ID** | WG-CHP-002 |
| **Name** | Status Chip |
| **Category** | Chip |
| **Purpose** | Visual status indicator with color coding |
| **Usage Locations** | Order status, delivery status, account status |

#### Props/Inputs

```typescript
interface WgStatusChipProps {
  status: 'pending' | 'confirmed' | 'processing' | 'shipped' | 'delivered' | 'cancelled' | 'returned' | 'active' | 'inactive' | 'suspended';
  size?: 'sm' | 'md' | 'lg';
  showIcon?: boolean; locale?: 'ar' | 'en';
  className?: string;
}
```

#### Status Configuration

| Status | Color | Label (AR) |
|--------|-------|------------|
| pending | warning | قيد الانتظار |
| confirmed | primary | مؤكد |
| processing | primary | قيد التجهيز |
| shipped | info | تم الشحن |
| delivered | success | تم التوصيل |
| cancelled | error | ملغي |
| returned | default | مرتجع |
| active | success | نشط |
| inactive | default | غير نشط |
| suspended | error | معلق |

---

### WG-CHP-003: Rating Stars

| Field | Value |
|-------|-------|
| **ID** | WG-CHP-003 |
| **Name** | Rating Stars |
| **Category** | Chip |
| **Purpose** | Display or input star ratings |
| **Usage Locations** | Product ratings, review forms, store ratings |

#### Props/Inputs

```typescript
interface WgRatingStarsProps {
  value: number; max?: number;
  size?: 'sm' | 'md' | 'lg';
  readonly?: boolean;
  showValue?: boolean; showCount?: boolean; count?: number;
  onRate?: (value: number) => void;
  locale?: 'ar' | 'en'; className?: string;
}
```

---

### WG-CHP-004: Price Display

| Field | Value |
|-------|-------|
| **ID** | WG-CHP-004 |
| **Name** | Price Display |
| **Category** | Chip |
| **Purpose** | Formatted price with currency and optional discount |
| **Usage Locations** | Product cards, cart, checkout, invoices |

#### Props/Inputs

```typescript
interface WgPriceDisplayProps {
  price: number; originalPrice?: number;
  currency?: string;
  size?: 'sm' | 'md' | 'lg';
  showCurrency?: boolean; showDiscount?: boolean;
  discountPercentage?: boolean;
  locale?: 'ar' | 'en'; className?: string;
}
```

#### RTL Behavior

- Numbers always LTR
- Currency code always follows the number
- Discount badge positioned on the start side

---

### WG-CHP-005: Countdown Timer

| Field | Value |
|-------|-------|
| **ID** | WG-CHP-005 |
| **Name** | Countdown Timer |
| **Category** | Chip |
| **Purpose** | Display remaining time for offers/deadlines |
| **Usage Locations** | Flash sales, delivery estimates, offer expiration |

#### Props/Inputs

```typescript
interface WgCountdownTimerProps {
  targetDate: Date | string;
  format?: 'detailed' | 'compact' | 'minimal';
  size?: 'sm' | 'md' | 'lg';
  showLabels?: boolean; locale?: 'ar' | 'en';
  onComplete?: () => void; className?: string;
}
```

---

## Search Components

---

### WG-SCH-001: Search Bar

| Field | Value |
|-------|-------|
| **ID** | WG-SCH-001 |
| **Name** | Search Bar |
| **Category** | Search |
| **Purpose** | Full-featured search with filters and suggestions |
| **Usage Locations** | Home page, category pages, global search |

#### Props/Inputs

```typescript
interface WgSearchBarProps {
  placeholder?: string; value?: string;
  recentSearches?: string[];
  suggestions?: SearchSuggestion[];
  showFilters?: boolean; filters?: FilterConfig[];
  loading?: boolean; autoFocus?: boolean;
  debounceMs?: number; locale?: 'ar' | 'en';
  onSearch?: (query: string) => void;
  onFilterChange?: (filters: Record<string, any>) => void;
  onSelectSuggestion?: (suggestion: SearchSuggestion) => void;
  onClear?: () => void; className?: string;
}

interface SearchSuggestion {
  id: string; text: string;
  type: 'product' | 'category' | 'store' | 'recent';
  image?: string; category?: string;
}

interface FilterConfig {
  id: string; label: string;
  type: 'text' | 'select' | 'range';
  options?: { value: string; label: string }[];
}
```

---

### WG-SCH-002: Autocomplete

| Field | Value |
|-------|-------|
| **ID** | WG-SCH-002 |
| **Name** | Autocomplete |
| **Category** | Search |
| **Purpose** | Type-ahead suggestions with keyboard navigation |
| **Usage Locations** | Search inputs, address fields, product name entry |

#### Props/Inputs

```typescript
interface WgAutocompleteProps {
  value?: string; placeholder?: string;
  suggestions: AutocompleteOption[];
  loading?: boolean; minChars?: number;
  maxSuggestions?: number; debounceMs?: number;
  locale?: 'ar' | 'en';
  onSelect?: (option: AutocompleteOption) => void;
  onSearch?: (query: string) => void;
  className?: string;
}

interface AutocompleteOption {
  id: string; label: string; sublabel?: string;
  icon?: React.ComponentType<{ className?: string }>;
  image?: string;
}
```

#### Keyboard Interaction

| Key | Action |
|-----|--------|
| ArrowDown | Highlight next suggestion |
| ArrowUp | Highlight previous suggestion |
| Enter | Select highlighted suggestion |
| Escape | Close suggestions |

---

### WG-SCH-003: Filter Panel

| Field | Value |
|-------|-------|
| **ID** | WG-SCH-003 |
| **Name** | Filter Panel |
| **Category** | Search |
| **Purpose** | Expandable filter controls for product/search results |
| **Usage Locations** | Product listing, search results, admin data views |

#### Props/Inputs

```typescript
interface WgFilterPanelProps {
  filters: FilterSection[]; values: Record<string, any>;
  collapsible?: boolean; showClearAll?: boolean;
  activeCount?: number;
  layout?: 'sidebar' | 'horizontal' | 'modal';
  locale?: 'ar' | 'en';
  onChange?: (filters: Record<string, any>) => void;
  onClearAll?: () => void; className?: string;
}

interface FilterSection {
  id: string; label: string;
  type: 'checkbox' | 'radio' | 'range' | 'color' | 'rating';
  options?: { value: string; label: string; count?: number; color?: string }[];
  min?: number; max?: number; expanded?: boolean;
}
```

---

### WG-SCH-004: Sort Dropdown

| Field | Value |
|-------|-------|
| **ID** | WG-SCH-004 |
| **Name** | Sort Dropdown |
| **Category** | Search |
| **Purpose** | Sort results by various criteria |
| **Usage Locations** | Product listings, search results, data tables |

#### Props/Inputs

```typescript
interface WgSortDropdownProps {
  options: SortOption[]; value?: string;
  direction?: 'asc' | 'desc';
  locale?: 'ar' | 'en';
  onChange?: (sort: { value: string; direction: 'asc' | 'desc' }) => void;
  className?: string;
}

interface SortOption {
  value: string; label: string;
  defaultDirection?: 'asc' | 'desc';
}
```

---

## Layout Components

---

### WG-LYT-001: Page Layout Shell

| Field | Value |
|-------|-------|
| **ID** | WG-LYT-001 |
| **Name** | Page Layout Shell |
| **Category** | Layout |
| **Purpose** | Main page structure with header, sidebar, content |
| **Usage Locations** | All authenticated pages, dashboard |

#### Props/Inputs

```typescript
interface WgPageLayoutProps {
  header?: React.ReactNode; sidebar?: React.ReactNode;
  footer?: React.ReactNode; breadcrumbs?: React.ReactNode;
  title?: string; actions?: React.ReactNode;
  maxWidth?: 'sm' | 'md' | 'lg' | 'xl' | 'full';
  padding?: boolean; locale?: 'ar' | 'en';
  children: React.ReactNode; className?: string;
}
```

---

### WG-LYT-002: Card Container

| Field | Value |
|-------|-------|
| **ID** | WG-LYT-002 |
| **Name** | Card Container |
| **Category** | Layout |
| **Purpose** | Generic content container with optional header/footer |
| **Usage Locations** | Settings panels, form sections, detail views |

#### Props/Inputs

```typescript
interface WgCardProps {
  header?: React.ReactNode; footer?: React.ReactNode;
  title?: string; description?: string;
  variant?: 'default' | 'outlined' | 'elevated';
  padding?: boolean | 'sm' | 'md' | 'lg';
  locale?: 'ar' | 'en';
  children: React.ReactNode; className?: string;
}
```

---

### WG-LYT-003: Tabs

| Field | Value |
|-------|-------|
| **ID** | WG-LYT-003 |
| **Name** | Tabs |
| **Category** | Layout |
| **Purpose** | Switch between content panels |
| **Usage Locations** | Product details, settings, profile sections |

#### Props/Inputs

```typescript
interface WgTabsProps {
  tabs: TabItem[];
  activeTab?: string;
  defaultTab?: string;
  variant?: 'underline' | 'pills' | 'enclosed';
  size?: 'sm' | 'md' | 'lg';
  locale?: 'ar' | 'en';
  onChange?: (tabId: string) => void;
  className?: string;
}

interface TabItem {
  id: string; label: string;
  icon?: React.ComponentType<{ className?: string }>;
  content?: React.ReactNode;
  badge?: number | string;
  disabled?: boolean;
}
```

---

### WG-LYT-004: Accordion

| Field | Value |
|-------|-------|
| **ID** | WG-LYT-004 |
| **Name** | Accordion |
| **Category** | Layout |
| **Purpose** | Collapsible content sections |
| **Usage Locations** | FAQ, product details, settings sections |

#### Props/Inputs

```typescript
interface WgAccordionProps {
  items: AccordionItem[];
  multiple?: boolean;
  defaultOpen?: string[];
  locale?: 'ar' | 'en';
  className?: string;
}

interface AccordionItem {
  id: string; title: string;
  content: React.ReactNode;
  icon?: React.ComponentType<{ className?: string }>;
  disabled?: boolean;
}
```

---

### WG-LYT-005: Stepper/Wizard

| Field | Value |
|-------|-------|
| **ID** | WG-LYT-005 |
| **Name** | Stepper/Wizard |
| **Category** | Layout |
| **Purpose** | Multi-step form/process indicator |
| **Usage Locations** | Checkout, registration, vendor onboarding |

#### Props/Inputs

```typescript
interface WgStepperProps {
  steps: StepItem[];
  currentStep: number;
  variant?: 'horizontal' | 'vertical';
  size?: 'sm' | 'md' | 'lg';
  locale?: 'ar' | 'en';
  onStepClick?: (step: number) => void;
  className?: string;
}

interface StepItem {
  id: string; label: string;
  description?: string;
  icon?: React.ComponentType<{ className?: string }>;
  status?: 'pending' | 'current' | 'completed' | 'error';
}
```

---

### WG-LYT-006: Grid Layout

| Field | Value |
|-------|-------|
| **ID** | WG-LYT-006 |
| **Name** | Grid Layout |
| **Category** | Layout |
| **Purpose** | Responsive grid for product listings, cards |
| **Usage Locations** | Product listings, category pages, dashboard widgets |

#### Props/Inputs

```typescript
interface WgGridProps {
  columns?: ResponsiveValue<number>;
  gap?: ResponsiveValue<number>;
  minItemWidth?: number;
  locale?: 'ar' | 'en';
  children: React.ReactNode;
  className?: string;
}

interface ResponsiveValue<T> {
  base: T; sm?: T; md?: T; lg?: T; xl?: T;
}
```

---

## Action Components

---

### WG-ACT-001: Dropdown Menu

| Field | Value |
|-------|-------|
| **ID** | WG-ACT-001 |
| **Name** | Dropdown Menu |
| **Category** | Action |
| **Purpose** | Contextual action menu |
| **Usage Locations** | Table row actions, user menu, settings menu |

#### Props/Inputs

```typescript
interface WgDropdownMenuProps {
  trigger: React.ReactNode;
  items: DropdownMenuItem[];
  align?: 'start' | 'center' | 'end';
  side?: 'top' | 'bottom';
  locale?: 'ar' | 'en';
  className?: string;
}

interface DropdownMenuItem {
  id: string; label: string;
  icon?: React.ComponentType<{ className?: string }>;
  onClick?: () => void;
  variant?: 'default' | 'danger';
  disabled?: boolean; divider?: boolean;
  badge?: string;
}
```

---

### WG-ACT-002: Context Menu

| Field | Value |
|-------|-------|
| **ID** | WG-ACT-002 |
| **Name** | Context Menu |
| **Category** | Action |
| **Purpose** | Right-click context menu |
| **Usage Locations** | Table rows, file management, admin panels |

#### Props/Inputs

```typescript
interface WgContextMenuProps {
  items: ContextMenuItem[];
  locale?: 'ar' | 'en';
  children: React.ReactNode;
}

interface ContextMenuItem {
  id: string; label: string;
  icon?: React.ComponentType<{ className?: string }>;
  onClick?: () => void;
  variant?: 'default' | 'danger';
  disabled?: boolean; divider?: boolean;
}
```

---

### WG-ACT-003: Tooltip

| Field | Value |
|-------|-------|
| **ID** | WG-ACT-003 |
| **Name** | Tooltip |
| **Category** | Action |
| **Purpose** | Informational hover tooltip |
| **Usage Locations** | Icon buttons, links, form fields |

#### Props/Inputs

```typescript
interface WgTooltipProps {
  content: string;
  side?: 'top' | 'bottom' | 'start' | 'end';
  align?: 'start' | 'center' | 'end';
  delay?: number;
  locale?: 'ar' | 'en';
  children: React.ReactNode;
  className?: string;
}
```

---

### WG-ACT-004: Popover

| Field | Value |
|-------|-------|
| **ID** | WG-ACT-004 |
| **Name** | Popover |
| **Category** | Action |
| **Purpose** | Rich content tooltip with interactions |
| **Usage Locations** | Filter previews, quick actions, date pickers |

#### Props/Inputs

```typescript
interface WgPopoverProps {
  trigger: React.ReactNode;
  content: React.ReactNode;
  side?: 'top' | 'bottom' | 'start' | 'end';
  align?: 'start' | 'center' | 'end';
  closeOnInteraction?: boolean;
  locale?: 'ar' | 'en';
  className?: string;
}
```

---

### WG-ACT-005: Swipe Action

| Field | Value |
|-------|-------|
| **ID** | WG-ACT-005 |
| **Name** | Swipe Action |
| **Category** | Action |
| **Purpose** | Swipe-to-reveal actions on mobile |
| **Usage Locations** | Order items, message list, contact list |

#### Props/Inputs

```typescript
interface WgSwipeActionProps {
  leftActions?: SwipeAction[];
  rightActions?: SwipeAction[];
  locale?: 'ar' | 'en';
  children: React.ReactNode;
  className?: string;
}

interface SwipeAction {
  id: string; label: string;
  icon?: React.ComponentType<{ className?: string }>;
  onClick: () => void;
  variant?: 'default' | 'danger' | 'success';
}
```

---

## Image Components

---

### WG-IMG-001: Image Gallery

| Field | Value |
|-------|-------|
| **ID** | WG-IMG-001 |
| **Name** | Image Gallery |
| **Category** | Image |
| **Purpose** | Product image gallery with zoom and thumbnails |
| **Usage Locations** | Product detail page, store gallery |

#### Props/Inputs

```typescript
interface WgImageGalleryProps {
  images: GalleryImage[];
  initialIndex?: number;
  variant?: 'stacked' | 'thumbnails' | 'lightbox';
  showCounter?: boolean;
  showZoom?: boolean;
  locale?: 'ar' | 'en';
  onChange?: (index: number) => void;
  className?: string;
}

interface GalleryImage {
  src: string; alt: string;
  thumb?: string;
  width?: number; height?: number;
}
```

#### RTL Behavior

- Thumbnail navigation arrows: Flipped
- Image counter: RTL format
- Zoom controls: Mirrored position

---

### WG-IMG-002: Avatar

| Field | Value |
|-------|-------|
| **ID** | WG-IMG-002 |
| **Name** | Avatar |
| **Category** | Image |
| **Purpose** | User/vendor profile image |
| **Usage Locations** | User profiles, comments, reviews, headers |

#### Props/Inputs

```typescript
interface WgAvatarProps {
  src?: string; alt?: string;
  fallback?: string;
  size?: 'xs' | 'sm' | 'md' | 'lg' | 'xl';
  shape?: 'circle' | 'square';
  status?: 'online' | 'offline' | 'busy' | 'away';
  badge?: React.ReactNode;
  className?: string;
}
```

---

### WG-IMG-003: Thumbnail

| Field | Value |
|-------|-------|
| **ID** | WG-IMG-003 |
| **Name** | Thumbnail |
| **Category** | Image |
| **Purpose** | Small preview image with optional overlay |
| **Usage Locations** | Product lists, file lists, chat messages |

#### Props/Inputs

```typescript
interface WgThumbnailProps {
  src: string; alt: string;
  size?: 'sm' | 'md' | 'lg';
  shape?: 'circle' | 'square' | 'rounded';
  overlay?: React.ReactNode;
  loading?: 'lazy' | 'eager';
  onClick?: () => void;
  className?: string;
}
```

---

## Chart Components

---

### WG-CHT-001: Line Chart

| Field | Value |
|-------|-------|
| **ID** | WG-CHT-001 |
| **Name** | Line Chart |
| **Category** | Chart |
| **Purpose** | Display trends over time |
| **Usage Locations** | Sales analytics, traffic reports, performance metrics |

#### Props/Inputs

```typescript
interface WgLineChartProps {
  data: ChartDataPoint[];
  series?: ChartSeries[];
  xKey: string; yKey?: string;
  height?: number;
  showGrid?: boolean;
  showTooltip?: boolean;
  showLegend?: boolean;
  locale?: 'ar' | 'en';
  className?: string;
}

interface ChartDataPoint {
  [key: string]: string | number;
}

interface ChartSeries {
  key: string; label: string;
  color?: string; type?: 'line' | 'area';
}
```

---

### WG-CHT-002: Bar Chart

| Field | Value |
|-------|-------|
| **ID** | WG-CHT-002 |
| **Name** | Bar Chart |
| **Category** | Chart |
| **Purpose** | Compare values across categories |
| **Usage Locations** | Sales by category, vendor performance, regional data |

#### Props/Inputs

```typescript
interface WgBarChartProps {
  data: ChartDataPoint[];
  series?: ChartSeries[];
  xKey: string; yKey?: string;
  height?: number;
  orientation?: 'vertical' | 'horizontal';
  showGrid?: boolean;
  showTooltip?: boolean;
  showLegend?: boolean;
  stacked?: boolean;
  locale?: 'ar' | 'en';
  className?: string;
}
```

---

### WG-CHT-003: Pie Chart

| Field | Value |
|-------|-------|
| **ID** | WG-CHT-003 |
| **Name** | Pie Chart |
| **Category** | Chart |
| **Purpose** | Show proportional data |
| **Usage Locations** | Sales distribution, payment methods, category breakdown |

#### Props/Inputs

```typescript
interface WgPieChartProps {
  data: PieChartData[];
  variant?: 'pie' | 'donut';
  height?: number;
  showTooltip?: boolean;
  showLegend?: boolean;
  showLabels?: boolean;
  innerRadius?: number;
  locale?: 'ar' | 'en';
  className?: string;
}

interface PieChartData {
  name: string; value: number;
  color?: string;
}
```

---

### WG-CHT-004: Area Chart

| Field | Value |
|-------|-------|
| **ID** | WG-CHT-004 |
| **Name** | Area Chart |
| **Category** | Chart |
| **Purpose** | Display volume trends over time |
| **Usage Locations** | Revenue trends, user growth, order volume |

#### Props/Inputs

```typescript
interface WgAreaChartProps {
  data: ChartDataPoint[];
  series?: ChartSeries[];
  xKey: string; yKey?: string;
  height?: number;
  stacked?: boolean;
  showGrid?: boolean;
  showTooltip?: boolean;
  showLegend?: boolean;
  locale?: 'ar' | 'en';
  className?: string;
}
```

---

## Domain-Specific Components

---

### WG-OTR-001: Wallet Balance Display

| Field | Value |
|-------|-------|
| **ID** | WG-OTR-001 |
| **Name** | Wallet Balance Display |
| **Category** | Other |
| **Purpose** | Show vendor/customer wallet balance |
| **Usage Locations** | Dashboard, profile, withdrawal page |

#### Props/Inputs

```typescript
interface WgWalletBalanceProps {
  balance: number;
  currency?: string;
  pendingBalance?: number;
  showActions?: boolean;
  onWithdraw?: () => void;
  onTopUp?: () => void;
  locale?: 'ar' | 'en';
  className?: string;
}
```

---

### WG-OTR-002: Delivery Code Display

| Field | Value |
|-------|-------|
| **ID** | WG-OTR-002 |
| **Name** | Delivery Code Display |
| **Category** | Other |
| **Purpose** | Show and verify delivery OTP code |
| **Usage Locations** | Driver app, customer app during delivery |

#### Props/Inputs

```typescript
interface WgDeliveryCodeProps {
  code: string;
  orderNumber?: string;
  onVerify?: (code: string) => void;
  onResend?: () => void;
  loading?: boolean;
  error?: string;
  locale?: 'ar' | 'en';
  className?: string;
}
```

---

### WG-OTR-003: Order Status Timeline

| Field | Value |
|-------|-------|
| **ID** | WG-OTR-003 |
| **Name** | Order Status Timeline |
| **Category** | Other |
| **Purpose** | Visual timeline of order progress |
| **Usage Locations** | Order details, order tracking page |

#### Props/Inputs

```typescript
interface WgOrderTimelineProps {
  steps: TimelineStep[];
  currentStep: number;
  variant?: 'vertical' | 'horizontal';
  locale?: 'ar' | 'en';
  className?: string;
}

interface TimelineStep {
  id: string; label: string;
  description?: string;
  timestamp?: string;
  status?: 'pending' | 'current' | 'completed' | 'error';
  icon?: React.ComponentType<{ className?: string }>;
}
```

---

### WG-OTR-004: Star Rating Input

| Field | Value |
|-------|-------|
| **ID** | WG-OTR-004 |
| **Name** | Star Rating Input |
| **Category** | Other |
| **Purpose** | Interactive star rating selection |
| **Usage Locations** | Review forms, feedback forms |

#### Props/Inputs

```typescript
interface WgStarRatingInputProps {
  value?: number;
  max?: number;
  size?: 'sm' | 'md' | 'lg';
  readonly?: boolean;
  showValue?: boolean;
  labels?: string[];
  onRate?: (value: number) => void;
  locale?: 'ar' | 'en';
  className?: string;
}
```

---

### WG-OTR-005: Quantity Selector

| Field | Value |
|-------|-------|
| **ID** | WG-OTR-005 |
| **Name** | Quantity Selector |
| **Category** | Other |
| **Purpose** | Increment/decrement quantity input |
| **Usage Locations** | Product detail, cart, checkout |

#### Props/Inputs

```typescript
interface WgQuantitySelectorProps {
  value: number;
  min?: number; max?: number; step?: number;
  size?: 'sm' | 'md' | 'lg';
  disabled?: boolean;
  locale?: 'ar' | 'en';
  onChange?: (value: number) => void;
  className?: string;
}
```

#### RTL Behavior

- Minus button on the right, plus on the left (RTL: minus left, plus right)
- Number always centered

#### Keyboard Interaction

| Key | Action |
|-----|--------|
| ArrowUp | Increment value |
| ArrowDown | Decrement value |
| Home | Set to min |
| End | Set to max |

---

## Shared Types and Utilities

### Common Types

```typescript
// Common types used across all components
export type Direction = 'ltr' | 'rtl';
export type Locale = 'ar' | 'en';
export type Size = 'xs' | 'sm' | 'md' | 'lg' | 'xl';

export interface ResponsiveValue<T> {
  base: T;
  sm?: T;
  md?: T;
  lg?: T;
  xl?: T;
}

export interface ComponentMeta {
  id: string;
  name: string;
  category: string;
  purpose: string;
  usageLocations: string[];
}

// Currency formatting
export function formatPrice(price: number, currency = 'YER'): string {
  return new Intl.NumberFormat('ar-YE', {
    style: 'decimal',
    maximumFractionDigits: 0,
  }).format(price) + ' ' + currency;
}

// RTL direction helper
export function getDirection(locale: Locale): Direction {
  return locale === 'ar' ? 'rtl' : 'ltr';
}

// Merge class names utility
export function cn(...classes: (string | undefined | null | false)[]): string {
  return classes.filter(Boolean).join(' ');
}
```

---

## Component Statistics

| Category | Components | Total |
|----------|------------|-------|
| Navigation | WG-NAV-001 to WG-NAV-005 | 5 |
| Form | WG-FRM-001 to WG-FRM-010 | 10 |
| Button | WG-BTN-001 to WG-BTN-003 | 3 |
| Table | WG-TBL-001 to WG-TBL-004 | 4 |
| Card | WG-CRD-001 to WG-CRD-005 | 5 |
| Modal | WG-MDL-001 to WG-MDL-005 | 5 |
| Feedback | WG-FDB-001 to WG-FDB-008 | 8 |
| Chip | WG-CHP-001 to WG-CHP-005 | 5 |
| Search | WG-SCH-001 to WG-SCH-004 | 4 |
| Layout | WG-LYT-001 to WG-LYT-006 | 6 |
| Action | WG-ACT-001 to WG-ACT-005 | 5 |
| Image | WG-IMG-001 to WG-IMG-003 | 3 |
| Chart | WG-CHT-001 to WG-CHT-004 | 4 |
| Other | WG-OTR-001 to WG-OTR-005 | 5 |
| **Total** | | **67** |

---

## Appendix A: Component Category Map

`
Component Library
├── Navigation (5)
│   ├── WG-NAV-001: Header/Navigation Bar
│   ├── WG-NAV-002: Sidebar Navigation
│   ├── WG-NAV-003: Bottom Navigation (Mobile)
│   ├── WG-NAV-004: Breadcrumbs
│   └── WG-NAV-005: Language Toggle
├── Form (10)
│   ├── WG-FRM-001: Text Input
│   ├── WG-FRM-002: Select/Dropdown
│   ├── WG-FRM-003: Checkbox
│   ├── WG-FRM-004: Radio Button
│   ├── WG-FRM-005: Textarea
│   ├── WG-FRM-006: Phone Input (+967)
│   ├── WG-FRM-007: OTP Input (6-digit)
│   ├── WG-FRM-008: Date Picker
│   ├── WG-FRM-009: File Upload
│   └── WG-FRM-010: Search Input
├── Button (3)
│   ├── WG-BTN-001: Button
│   ├── WG-BTN-002: Icon Button
│   └── WG-BTN-003: Floating Action Button
├── Table (4)
│   ├── WG-TBL-001: Data Table
│   ├── WG-TBL-002: Table Pagination
│   ├── WG-TBL-003: Table Filters
│   └── WG-TBL-004: Table Sort
├── Card (5)
│   ├── WG-CRD-001: Product Card
│   ├── WG-CRD-002: Store Card
│   ├── WG-CRD-003: Order Card
│   ├── WG-CRD-004: User Card
│   └── WG-CRD-005: Stat Card (KPI)
├── Modal (5)
│   ├── WG-MDL-001: Modal Dialog
│   ├── WG-MDL-002: Drawer (Side Panel)
│   ├── WG-MDL-003: Bottom Sheet
│   ├── WG-MDL-004: Confirmation Dialog
│   └── WG-MDL-005: Alert Dialog
├── Feedback (8)
│   ├── WG-FDB-001: Toast Notification
│   ├── WG-FDB-002: Alert Banner
│   ├── WG-FDB-003: Snackbar
│   ├── WG-FDB-004: Progress Bar
│   ├── WG-FDB-005: Spinner/Loader
│   ├── WG-FDB-006: Skeleton Loader
│   ├── WG-FDB-007: Empty State
│   └── WG-FDB-008: Error State
├── Chip (5)
│   ├── WG-CHP-001: Badge/Tag
│   ├── WG-CHP-002: Status Chip
│   ├── WG-CHP-003: Rating Stars
│   ├── WG-CHP-004: Price Display
│   └── WG-CHP-005: Countdown Timer
├── Search (4)
│   ├── WG-SCH-001: Search Bar
│   ├── WG-SCH-002: Autocomplete
│   ├── WG-SCH-003: Filter Panel
│   └── WG-SCH-004: Sort Dropdown
├── Layout (6)
│   ├── WG-LYT-001: Page Layout Shell
│   ├── WG-LYT-002: Card Container
│   ├── WG-LYT-003: Tabs
│   ├── WG-LYT-004: Accordion
│   ├── WG-LYT-005: Stepper/Wizard
│   └── WG-LYT-006: Grid Layout
├── Action (5)
│   ├── WG-ACT-001: Dropdown Menu
│   ├── WG-ACT-002: Context Menu
│   ├── WG-ACT-003: Tooltip
│   ├── WG-ACT-004: Popover
│   └── WG-ACT-005: Swipe Action
├── Image (3)
│   ├── WG-IMG-001: Image Gallery
│   ├── WG-IMG-002: Avatar
│   └── WG-IMG-003: Thumbnail
├── Chart (4)
│   ├── WG-CHT-001: Line Chart
│   ├── WG-CHT-002: Bar Chart
│   ├── WG-CHT-003: Pie Chart
│   └── WG-CHT-004: Area Chart
└── Other (5)
    ├── WG-OTR-001: Wallet Balance Display
    ├── WG-OTR-002: Delivery Code Display
    ├── WG-OTR-003: Order Status Timeline
    ├── WG-OTR-004: Star Rating Input
    └── WG-OTR-005: Quantity Selector
`

---

*End of YemenMart Component Library Specification*

