# Component Library - YemenMart

## Overview

Shared component library used across all YemenMart frontends (customer store, vendor panel, admin panel, mobile apps). Provides consistent, accessible, RTL-aware UI primitives with Tailwind CSS, theming, and Arabic-first design tokens.

## Tech Stack

- React 19 with TypeScript
- Tailwind CSS 4 for styling
- CVA (Class Variance Authority) for component variants
- clsx + tailwind-merge for class merging
- Radix UI Primitives for accessible primitives
- React Hook Form + Zod for forms
- Lucide React for icons

## Design Tokens

```typescript
// tokens/colors.ts
export const colors = {
  primary: {
    50: '#eff6ff', 100: '#dbeafe', 200: '#bfdbfe', 300: '#93c5fd',
    400: '#60a5fa', 500: '#3b82f6', 600: '#2563eb', 700: '#1d4ed8',
    800: '#1e40af', 900: '#1e3a8a',
  },
  neutral: {
    50: '#f9fafb', 100: '#f3f4f6', 200: '#e5e7eb', 300: '#d1d5db',
    400: '#9ca3af', 500: '#6b7280', 600: '#4b5563', 700: '#374151',
    800: '#1f2937', 900: '#111827',
  },
  success: '#22c55e', warning: '#f59e0b', error: '#ef4444', info: '#3b82f6',
};
```

## Utility: cn()

```typescript
import { clsx, type ClassValue } from 'clsx';
import { twMerge } from 'tailwind-merge';
export function cn(...inputs: ClassValue[]) { return twMerge(clsx(inputs)); }
```

---

## 1. Button

```typescript
import { forwardRef, ButtonHTMLAttributes } from 'react';
import { cva, type VariantProps } from 'class-variance-authority';
import { cn } from '@/lib/cn';
import { Loader2 } from 'lucide-react';

const buttonVariants = cva(
  'inline-flex items-center justify-center rounded-lg font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-primary-500 focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50',
  {
    variants: {
      variant: {
        primary: 'bg-primary-600 text-white hover:bg-primary-700 active:bg-primary-800',
        secondary: 'bg-neutral-100 text-neutral-900 hover:bg-neutral-200 dark:bg-neutral-800 dark:text-neutral-100',
        outline: 'border border-neutral-300 bg-transparent hover:bg-neutral-50 dark:border-neutral-600',
        ghost: 'bg-transparent hover:bg-neutral-100 dark:hover:bg-neutral-800',
        danger: 'bg-red-600 text-white hover:bg-red-700',
        success: 'bg-green-600 text-white hover:bg-green-700',
      },
      size: {
        sm: 'h-8 px-3 text-sm gap-1.5',
        md: 'h-10 px-4 text-sm gap-2',
        lg: 'h-12 px-6 text-base gap-2',
        xl: 'h-14 px-8 text-lg gap-3',
        icon: 'h-10 w-10',
      },
    },
    defaultVariants: { variant: 'primary', size: 'md' },
  }
);

interface ButtonProps extends ButtonHTMLAttributes<HTMLButtonElement>,
  VariantProps<typeof buttonVariants> { loading?: boolean; fullWidth?: boolean; }

const Button = forwardRef<HTMLButtonElement, ButtonProps>(
  ({ className, variant, size, loading, fullWidth, children, disabled, ...props }, ref) => (
    <button className={cn(buttonVariants({ variant, size, className }), fullWidth && 'w-full')}
      ref={ref} disabled={disabled || loading} aria-busy={loading} {...props}>
      {loading && <Loader2 className="h-4 w-4 animate-spin" />}
      {children}
    </button>
  )
);
Button.displayName = 'Button';
export { Button, buttonVariants };
```

---

## 2. Input

```typescript
import { forwardRef, InputHTMLAttributes, ReactNode } from 'react';
import { cn } from '@/lib/cn';

interface InputProps extends InputHTMLAttributes<HTMLInputElement> {
  label?: string; error?: string; hint?: string; icon?: ReactNode; rightIcon?: ReactNode;
}

const Input = forwardRef<HTMLInputElement, InputProps>(
  ({ className, label, error, hint, icon, rightIcon, id, ...props }, ref) => {
    const inputId = id || label?.toLowerCase().replace(/\s+/g, '-');
    return (
      <div className="w-full">
        {label && (
          <label htmlFor={inputId} className="block text-sm font-medium text-neutral-700 dark:text-neutral-300 mb-1.5">
            {label}
          </label>
        )}
        <div className="relative">
          {icon && <div className="absolute left-3 top-1/2 -translate-y-1/2 text-neutral-400">{icon}</div>}
          <input ref={ref} id={inputId}
            className={cn(
              'w-full rounded-lg border bg-white px-3 py-2 text-sm transition-colors',
              'focus:outline-none focus:ring-2 focus:ring-primary-500 focus:border-transparent',
              'dark:bg-neutral-800 dark:text-neutral-100',
              icon && 'pl-10', rightIcon && 'pr-10',
              error ? 'border-red-500 focus:ring-red-500' : 'border-neutral-300 dark:border-neutral-600',
              'disabled:cursor-not-allowed disabled:opacity-50', className
            )} {...props} />
          {rightIcon && <div className="absolute right-3 top-1/2 -translate-y-1/2 text-neutral-400">{rightIcon}</div>}
        </div>
        {hint && !error && <p className="text-xs text-neutral-500 mt-1">{hint}</p>}
        {error && <p className="text-xs text-red-500 mt-1">{error}</p>}
      </div>
    );
  }
);
Input.displayName = 'Input';
export { Input };
```

---

## 3. Select

```typescript
import { forwardRef, SelectHTMLAttributes } from 'react';
import { cn } from '@/lib/cn';
import { ChevronDown } from 'lucide-react';

interface SelectOption { value: string; label: string; disabled?: boolean; }
interface SelectProps extends SelectHTMLAttributes<HTMLSelectElement> {
  label?: string; error?: string; options: SelectOption[]; placeholder?: string;
}

const Select = forwardRef<HTMLSelectElement, SelectProps>(
  ({ className, label, error, options, placeholder, id, ...props }, ref) => {
    const selectId = id || label?.toLowerCase().replace(/\s+/g, '-');
    return (
      <div className="w-full">
        {label && (
          <label htmlFor={selectId} className="block text-sm font-medium text-neutral-700 dark:text-neutral-300 mb-1.5">
            {label}
          </label>
        )}
        <div className="relative">
          <select ref={ref} id={selectId}
            className={cn(
              'w-full appearance-none rounded-lg border bg-white px-3 py-2 pr-10 text-sm',
              'focus:outline-none focus:ring-2 focus:ring-primary-500 focus:border-transparent',
              'dark:bg-neutral-800 dark:text-neutral-100',
              error ? 'border-red-500' : 'border-neutral-300 dark:border-neutral-600',
              'disabled:cursor-not-allowed disabled:opacity-50', className
            )} {...props}>
            {placeholder && <option value="">{placeholder}</option>}
            {options.map((opt) => (
              <option key={opt.value} value={opt.value} disabled={opt.disabled}>{opt.label}</option>
            ))}
          </select>
          <ChevronDown className="absolute right-3 top-1/2 -translate-y-1/2 h-4 w-4 text-neutral-400 pointer-events-none" />
        </div>
        {error && <p className="text-xs text-red-500 mt-1">{error}</p>}
      </div>
    );
  }
);
Select.displayName = 'Select';
export { Select };
```

---

## 4. Textarea

```typescript
import { forwardRef, TextareaHTMLAttributes } from 'react';
import { cn } from '@/lib/cn';

interface TextareaProps extends TextareaHTMLAttributes<HTMLTextAreaElement> {
  label?: string; error?: string; hint?: string;
}

const Textarea = forwardRef<HTMLTextAreaElement, TextareaProps>(
  ({ className, label, error, hint, id, ...props }, ref) => {
    const textareaId = id || label?.toLowerCase().replace(/\s+/g, '-');
    return (
      <div className="w-full">
        {label && (
          <label htmlFor={textareaId} className="block text-sm font-medium text-neutral-700 dark:text-neutral-300 mb-1.5">
            {label}
          </label>
        )}
        <textarea ref={ref} id={textareaId}
          className={cn(
            'w-full rounded-lg border bg-white px-3 py-2 text-sm transition-colors resize-y',
            'focus:outline-none focus:ring-2 focus:ring-primary-500 focus:border-transparent',
            'dark:bg-neutral-800 dark:text-neutral-100',
            error ? 'border-red-500' : 'border-neutral-300 dark:border-neutral-600',
            'disabled:cursor-not-allowed disabled:opacity-50', className
          )} {...props} />
        {hint && !error && <p className="text-xs text-neutral-500 mt-1">{hint}</p>}
        {error && <p className="text-xs text-red-500 mt-1">{error}</p>}
      </div>
    );
  }
);
Textarea.displayName = 'Textarea';
export { Textarea };
```

---

## 5. Checkbox & Radio

```typescript
import { forwardRef, InputHTMLAttributes } from 'react';
import { cn } from '@/lib/cn';

interface CheckboxProps extends Omit<InputHTMLAttributes<HTMLInputElement>, 'type'> {
  label?: string; error?: string;
}

const Checkbox = forwardRef<HTMLInputElement, CheckboxProps>(
  ({ className, label, error, id, ...props }, ref) => {
    const checkboxId = id || label?.toLowerCase().replace(/\s+/g, '-');
    return (
      <div className="flex items-start gap-2">
        <input ref={ref} type="checkbox" id={checkboxId}
          className={cn(
            'h-4 w-4 mt-0.5 rounded border-neutral-300 text-primary-600 focus:ring-primary-500',
            'dark:border-neutral-600 dark:bg-neutral-700',
            error && 'border-red-500', className
          )} {...props} />
        {label && (
          <label htmlFor={checkboxId} className="text-sm text-neutral-700 dark:text-neutral-300">
            {label}
          </label>
        )}
        {error && <p className="text-xs text-red-500">{error}</p>}
      </div>
    );
  }
);
Checkbox.displayName = 'Checkbox';

interface RadioProps extends Omit<InputHTMLAttributes<HTMLInputElement>, 'type'> {
  label?: string; error?: string;
}

const Radio = forwardRef<HTMLInputElement, RadioProps>(
  ({ className, label, error, id, ...props }, ref) => {
    const radioId = id || label?.toLowerCase().replace(/\s+/g, '-');
    return (
      <div className="flex items-start gap-2">
        <input ref={ref} type="radio" id={radioId}
          className={cn(
            'h-4 w-4 mt-0.5 border-neutral-300 text-primary-600 focus:ring-primary-500',
            error && 'border-red-500', className
          )} {...props} />
        {label && (
          <label htmlFor={radioId} className="text-sm text-neutral-700 dark:text-neutral-300">
            {label}
          </label>
        )}
      </div>
    );
  }
);
Radio.displayName = 'Radio';
export { Checkbox, Radio };
```

---

## 6. Card

```typescript
import { HTMLAttributes, ReactNode } from 'react';
import { cn } from '@/lib/cn';

interface CardProps extends HTMLAttributes<HTMLDivElement> { children: ReactNode; }

function Card({ className, children, ...props }: CardProps) {
  return (
    <div className={cn('bg-white dark:bg-neutral-800 rounded-xl border border-neutral-200 dark:border-neutral-700 shadow-sm', className)} {...props}>
      {children}
    </div>
  );
}

function CardHeader({ className, children, ...props }: CardProps) {
  return <div className={cn('px-6 py-4 border-b border-neutral-200 dark:border-neutral-700', className)} {...props}>{children}</div>;
}

function CardContent({ className, children, ...props }: CardProps) {
  return <div className={cn('px-6 py-4', className)} {...props}>{children}</div>;
}

function CardFooter({ className, children, ...props }: CardProps) {
  return <div className={cn('px-6 py-4 border-t border-neutral-200 dark:border-neutral-700', className)} {...props}>{children}</div>;
}

export { Card, CardHeader, CardContent, CardFooter };
```

---

## 7. Modal

```typescript
import { useEffect, useRef, ReactNode } from 'react';
import { cn } from '@/lib/cn';
import { X } from 'lucide-react';
import { createPortal } from 'react-dom';

interface ModalProps {
  isOpen: boolean; onClose: () => void; title?: string; children: ReactNode;
  size?: 'sm' | 'md' | 'lg' | 'xl'; showClose?: boolean;
}

function Modal({ isOpen, onClose, title, children, size = 'md', showClose = true }: ModalProps) {
  const overlayRef = useRef<HTMLDivElement>(null);

  useEffect(() => {
    const handleEscape = (e: KeyboardEvent) => { if (e.key === 'Escape') onClose(); };
    if (isOpen) document.addEventListener('keydown', handleEscape);
    return () => document.removeEventListener('keydown', handleEscape);
  }, [isOpen, onClose]);

  useEffect(() => {
    if (isOpen) document.body.style.overflow = 'hidden';
    else document.body.style.overflow = '';
    return () => { document.body.style.overflow = ''; };
  }, [isOpen]);

  if (!isOpen) return null;

  const sizeClasses = { sm: 'max-w-sm', md: 'max-w-md', lg: 'max-w-lg', xl: 'max-w-xl' };

  return createPortal(
    <div ref={overlayRef} className="fixed inset-0 z-50 flex items-center justify-center p-4 bg-black/50"
      onClick={(e) => { if (e.target === overlayRef.current) onClose(); }}>
      <div className={cn('bg-white dark:bg-neutral-900 rounded-xl shadow-2xl w-full max-h-[90vh] overflow-y-auto', sizeClasses[size])}>
        {(title || showClose) && (
          <div className="flex items-center justify-between px-6 py-4 border-b border-neutral-200 dark:border-neutral-700">
            {title && <h2 className="text-lg font-semibold">{title}</h2>}
            {showClose && (
              <button onClick={onClose} className="p-1 rounded-lg hover:bg-neutral-100 dark:hover:bg-neutral-800">
                <X className="w-5 h-5" />
              </button>
            )}
          </div>
        )}
        <div className="px-6 py-4">{children}</div>
      </div>
    </div>,
    document.body
  );
}

export { Modal };
```

---

## 8. Toast / Alert

```typescript
import { cn } from '@/lib/cn';
import { CheckCircle, XCircle, AlertTriangle, Info, X } from 'lucide-react';

interface ToastProps {
  type?: 'success' | 'error' | 'warning' | 'info'; title: string; message?: string; onClose?: () => void;
}

function Toast({ type = 'info', title, message, onClose }: ToastProps) {
  const config = {
    success: { icon: CheckCircle, bg: 'bg-green-50 dark:bg-green-900/20', border: 'border-green-200 dark:border-green-800', text: 'text-green-800 dark:text-green-200' },
    error: { icon: XCircle, bg: 'bg-red-50 dark:bg-red-900/20', border: 'border-red-200 dark:border-red-800', text: 'text-red-800 dark:text-red-200' },
    warning: { icon: AlertTriangle, bg: 'bg-yellow-50 dark:bg-yellow-900/20', border: 'border-yellow-200 dark:border-yellow-800', text: 'text-yellow-800 dark:text-yellow-200' },
    info: { icon: Info, bg: 'bg-blue-50 dark:bg-blue-900/20', border: 'border-blue-200 dark:border-blue-800', text: 'text-blue-800 dark:text-blue-200' },
  };
  const { icon: Icon, bg, border, text } = config[type];

  return (
    <div className={cn('flex items-start gap-3 p-4 rounded-xl border shadow-lg', bg, border)}>
      <Icon className={cn('w-5 h-5 mt-0.5 shrink-0', text)} />
      <div className="flex-1">
        <p className={cn('font-medium', text)}>{title}</p>
        {message && <p className="text-sm mt-1 opacity-80">{message}</p>}
      </div>
      {onClose && (
        <button onClick={onClose} className="p-1 rounded-lg hover:bg-black/10"><X className="w-4 h-4" /></button>
      )}
    </div>
  );
}

interface AlertProps { variant?: 'default' | 'info' | 'success' | 'warning' | 'error'; children: React.ReactNode; }

function Alert({ variant = 'default', children }: AlertProps) {
  const variants = {
    default: 'bg-neutral-100 dark:bg-neutral-800 text-neutral-900 dark:text-neutral-100',
    info: 'bg-blue-50 dark:bg-blue-900/20 text-blue-900 dark:text-blue-100 border border-blue-200 dark:border-blue-800',
    success: 'bg-green-50 dark:bg-green-900/20 text-green-900 dark:text-green-100 border border-green-200',
    warning: 'bg-yellow-50 dark:bg-yellow-900/20 text-yellow-900 dark:text-yellow-100 border border-yellow-200',
    error: 'bg-red-50 dark:bg-red-900/20 text-red-900 dark:text-red-100 border border-red-200',
  };
  return <div className={cn('rounded-xl p-4', variants[variant])}>{children}</div>;
}

export { Toast, Alert };
```

---

## 9. Badge

```typescript
import { cn } from '@/lib/cn';

interface BadgeProps { variant?: 'default' | 'primary' | 'success' | 'warning' | 'error' | 'info' | 'neutral'; children: React.ReactNode; className?: string; }

function Badge({ variant = 'default', children, className }: BadgeProps) {
  const variants = {
    default: 'bg-neutral-100 text-neutral-800 dark:bg-neutral-700 dark:text-neutral-200',
    primary: 'bg-primary-100 text-primary-800 dark:bg-primary-900/20 dark:text-primary-300',
    success: 'bg-green-100 text-green-800 dark:bg-green-900/20 dark:text-green-300',
    warning: 'bg-yellow-100 text-yellow-800 dark:bg-yellow-900/20 dark:text-yellow-300',
    error: 'bg-red-100 text-red-800 dark:bg-red-900/20 dark:text-red-300',
    info: 'bg-blue-100 text-blue-800 dark:bg-blue-900/20 dark:text-blue-300',
    neutral: 'bg-neutral-200 text-neutral-600 dark:bg-neutral-600 dark:text-neutral-300',
  };
  return (
    <span className={cn('inline-flex items-center rounded-full px-2.5 py-0.5 text-xs font-medium', variants[variant], className)}>
      {children}
    </span>
  );
}

export { Badge };
```

---

## 10. Table

```typescript
import { ReactNode } from 'react';
import { cn } from '@/lib/cn';

interface TableProps { children: ReactNode; className?: string; }
function Table({ children, className }: TableProps) {
  return (
    <div className="w-full overflow-x-auto">
      <table className={cn('w-full text-sm', className)}>{children}</table>
    </div>
  );
}

function TableHeader({ children, className }: TableProps) {
  return <thead className={cn('bg-neutral-50 dark:bg-neutral-800 border-b border-neutral-200 dark:border-neutral-700', className)}>{children}</thead>;
}

function TableBody({ children, className }: TableProps) {
  return <tbody className={cn('divide-y divide-neutral-200 dark:divide-neutral-700', className)}>{children}</tbody>;
}

function TableRow({ children, className, onClick }: TableProps & { onClick?: () => void }) {
  return (
    <tr className={cn('hover:bg-neutral-50 dark:hover:bg-neutral-800/50 transition-colors', onClick && 'cursor-pointer', className)} onClick={onClick}>
      {children}
    </tr>
  );
}

function TableHead({ children, className }: TableProps) {
  return <th className={cn('px-4 py-3 text-start text-xs font-medium text-neutral-500 uppercase tracking-wider', className)}>{children}</th>;
}

function TableCell({ children, className }: TableProps) {
  return <td className={cn('px-4 py-3 whitespace-nowrap', className)}>{children}</td>;
}

export { Table, TableHeader, TableBody, TableRow, TableHead, TableCell };
```

---

## 11. Pagination

```typescript
import { cn } from '@/lib/cn';
import { ChevronRight, ChevronLeft } from 'lucide-react';

interface PaginationProps { currentPage: number; totalPages: number; onPageChange: (page: number) => void; }

function Pagination({ currentPage, totalPages, onPageChange }: PaginationProps) {
  const pages = Array.from({ length: Math.min(totalPages, 7) }, (_, i) => {
    if (totalPages <= 7) return i + 1;
    if (currentPage <= 4) return i + 1;
    if (currentPage >= totalPages - 3) return totalPages - 6 + i;
    return currentPage - 3 + i;
  });

  return (
    <div className="flex items-center justify-center gap-1">
      <button onClick={() => onPageChange(currentPage - 1)} disabled={currentPage === 1}
        className="p-2 rounded-lg hover:bg-neutral-100 dark:hover:bg-neutral-800 disabled:opacity-50 disabled:cursor-not-allowed">
        <ChevronRight className="w-4 h-4" />
      </button>
      {pages.map((page) => (
        <button key={page} onClick={() => onPageChange(page)}
          className={cn('w-10 h-10 rounded-lg text-sm font-medium transition-colors',
            page === currentPage ? 'bg-primary-600 text-white' : 'hover:bg-neutral-100 dark:hover:bg-neutral-800'
          )}>{page}</button>
      ))}
      <button onClick={() => onPageChange(currentPage + 1)} disabled={currentPage === totalPages}
        className="p-2 rounded-lg hover:bg-neutral-100 dark:hover:bg-neutral-800 disabled:opacity-50">
        <ChevronLeft className="w-4 h-4" />
      </button>
    </div>
  );
}

export { Pagination };
```

---

## 12. Loading & Empty States

```typescript
import { cn } from '@/lib/cn';
import { Loader2, Inbox } from 'lucide-react';

function Spinner({ size = 'md', className }: { size?: 'sm' | 'md' | 'lg'; className?: string }) {
  const sizes = { sm: 'h-4 w-4', md: 'h-6 w-6', lg: 'h-8 w-8' };
  return <Loader2 className={cn('animate-spin text-primary-600', sizes[size], className)} />;
}

function Skeleton({ className }: { className?: string }) {
  return <div className={cn('animate-pulse rounded-lg bg-neutral-200 dark:bg-neutral-700', className)} />;
}

function SkeletonCard() {
  return (
    <div className="bg-white dark:bg-neutral-800 rounded-xl p-4 shadow-sm">
      <Skeleton className="h-40 w-full rounded-lg mb-3" />
      <Skeleton className="h-4 w-3/4 mb-2" />
      <Skeleton className="h-4 w-1/2 mb-3" />
      <Skeleton className="h-8 w-full rounded-lg" />
    </div>
  );
}

function EmptyState({ icon: Icon = Inbox, title, description, action }: {
  icon?: typeof Inbox; title: string; description?: string; action?: React.ReactNode;
}) {
  return (
    <div className="flex flex-col items-center justify-center py-12 text-center">
      <Icon className="w-16 h-16 text-neutral-300 dark:text-neutral-600 mb-4" />
      <h3 className="text-lg font-semibold mb-2">{title}</h3>
      {description && <p className="text-neutral-500 max-w-md mb-4">{description}</p>}
      {action}
    </div>
  );
}

export { Spinner, Skeleton, SkeletonCard, EmptyState };
```

---

## 13. RTL-Aware Utilities

```typescript
// lib/rtl.ts
import { useLocale } from 'next-intl';

export function useRtl() {
  const locale = useLocale();
  return { isRtl: locale === 'ar', dir: locale === 'ar' ? 'rtl' : 'ltr' as const };
}

// Tailwind RTL utilities (already built-in via tailwindcss-rtl plugin)
// Or use logical properties: ms-*, me-*, ps-*, pe-*, start-*, end-*

// Example usage in components:
// <div className="ms-4"> (margin-inline-start, works in both RTL/LTR)
// <div className="ps-4"> (padding-inline-start)
// <div className="text-start"> (text-align: start)
```

---

## 14. Tabs

```typescript
import { useState, createContext, useContext, ReactNode } from 'react';
import { cn } from '@/lib/cn';

const TabsContext = createContext<{ value: string; onChange: (v: string) => void }>({ value: '', onChange: () => {} });

function Tabs({ value, onValueChange, children, className }: {
  value: string; onValueChange: (v: string) => void; children: ReactNode; className?: string;
}) {
  return (
    <TabsContext.Provider value={{ value, onChange: onValueChange }}>
      <div className={className}>{children}</div>
    </TabsContext.Provider>
  );
}

function TabsList({ children, className }: { children: ReactNode; className?: string }) {
  return (
    <div className={cn('flex border-b border-neutral-200 dark:border-neutral-700', className)} role="tablist">
      {children}
    </div>
  );
}

function TabsTrigger({ value, children, className }: { value: string; children: ReactNode; className?: string }) {
  const ctx = useContext(TabsContext);
  const isActive = ctx.value === value;
  return (
    <button role="tab" aria-selected={isActive}
      onClick={() => ctx.onChange(value)}
      className={cn('px-4 py-2 text-sm font-medium border-b-2 transition-colors',
        isActive ? 'border-primary-600 text-primary-600' : 'border-transparent text-neutral-500 hover:text-neutral-700',
        className
      )}>{children}</button>
  );
}

function TabsContent({ value, children, className }: { value: string; children: ReactNode; className?: string }) {
  const ctx = useContext(TabsContext);
  if (ctx.value !== value) return null;
  return <div role="tabpanel" className={cn('py-4', className)}>{children}</div>;
}

export { Tabs, TabsList, TabsTrigger, TabsContent };
```

---

## 15. Tooltip

```typescript
import { useState, ReactNode } from 'react';
import { cn } from '@/lib/cn';

interface TooltipProps { content: string; children: ReactNode; position?: 'top' | 'bottom' | 'left' | 'right'; }

function Tooltip({ content, children, position = 'top' }: TooltipProps) {
  const [show, setShow] = useState(false);
  const positions = {
    top: 'bottom-full left-1/2 -translate-x-1/2 mb-2',
    bottom: 'top-full left-1/2 -translate-x-1/2 mt-2',
    left: 'right-full top-1/2 -translate-y-1/2 mr-2',
    right: 'left-full top-1/2 -translate-y-1/2 ml-2',
  };
  return (
    <div className="relative inline-flex" onMouseEnter={() => setShow(true)} onMouseLeave={() => setShow(false)}>
      {children}
      {show && (
        <div className={cn('absolute z-50 px-2 py-1 text-xs text-white bg-neutral-900 rounded whitespace-nowrap', positions[position])}>
          {content}
        </div>
      )}
    </div>
  );
}

export { Tooltip };
```

---

## Component Export Index

```typescript
// components/ui/index.ts
export { Button, buttonVariants } from './Button';
export { Input } from './Input';
export { Select } from './Select';
export { Textarea } from './Textarea';
export { Checkbox, Radio } from './Checkbox';
export { Card, CardHeader, CardContent, CardFooter } from './Card';
export { Modal } from './Modal';
export { Toast, Alert } from './Toast';
export { Badge } from './Badge';
export { Table, TableHeader, TableBody, TableRow, TableHead, TableCell } from './Table';
export { Pagination } from './Pagination';
export { Spinner, Skeleton, SkeletonCard, EmptyState } from './LoadingStates';
export { Tabs, TabsList, TabsTrigger, TabsContent } from './Tabs';
export { Tooltip } from './Tooltip';
```

## Testing Components

```typescript
// __tests__/components/Button.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { Button } from '@/components/ui/Button';

describe('Button', () => {
  it('renders with text', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByText('Click me')).toBeInTheDocument();
  });

  it('applies variant styles', () => {
    render(<Button variant="danger">Delete</Button>);
    expect(screen.getByText('Delete')).toHaveClass('bg-red-600');
  });

  it('shows loading spinner', () => {
    render(<Button loading>Submit</Button>);
    expect(screen.getByRole('button')).toBeDisabled();
    expect(screen.getByText('Submit')).toBeInTheDocument();
  });

  it('calls onClick handler', () => {
    const onClick = vi.fn();
    render(<Button onClick={onClick}>Click</Button>);
    fireEvent.click(screen.getByText('Click'));
    expect(onClick).toHaveBeenCalledTimes(1);
  });

  it('is disabled when loading', () => {
    render(<Button loading>Submit</Button>);
    expect(screen.getByRole('button')).toBeDisabled();
  });
});
```
