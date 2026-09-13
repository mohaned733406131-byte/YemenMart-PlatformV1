# State Management Strategy - YemenMart

## Overview

YemenMart uses a layered state management approach separating concerns across server state, client state, form state, and URL state. Each layer uses the appropriate tool for its purpose, ensuring optimal performance, caching, and developer experience.

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────┐
│                    APPLICATION STATE                      │
├─────────────┬──────────────┬────────────┬───────────────┤
│ Server State│ Client State │ Form State │  URL State    │
│ React Query │   Zustand    │ React Hook │  Query Params │
│             │              │  Form+Zod  │               │
├─────────────┼──────────────┼────────────┼───────────────┤
│ API data    │ UI state     │ Form data  │ Filters       │
│ Caching     │ Auth tokens  │ Validation │ Pagination    │
│ Pagination  │ Cart         │ Errors     │ Search        │
│ Real-time   │ Preferences  │ Touched    │ Sort          │
└─────────────┴──────────────┴────────────┴───────────────┘
```

## 1. Server State: TanStack Query (React Query)

### Provider Setup

```typescript
// providers/QueryProvider.tsx
'use client';

import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';
import { useState } from 'react';

export function QueryProvider({ children }: { children: React.ReactNode }) {
  const [queryClient] = useState(
    () =>
      new QueryClient({
        defaultOptions: {
          queries: {
            staleTime: 5 * 60 * 1000,        // 5 minutes
            gcTime: 30 * 60 * 1000,           // 30 minutes (formerly cacheTime)
            retry: 2,
            retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 30000),
            refetchOnWindowFocus: false,
            refetchOnReconnect: 'always',
            throwOnError: false,
          },
          mutations: {
            retry: 1,
          },
        },
      })
  );

  return (
    <QueryClientProvider client={queryClient}>
      {children}
      {process.env.NODE_ENV === 'development' && <ReactQueryDevtools />}
    </QueryClientProvider>
  );
}
```

### Query Keys Convention

```typescript
// lib/queryKeys.ts
export const queryKeys = {
  // Products
  products: {
    all: ['products'] as const,
    lists: () => [...queryKeys.products.all, 'list'] as const,
    list: (filters: Record<string, any>) => [...queryKeys.products.lists(), filters] as const,
    details: () => [...queryKeys.products.all, 'detail'] as const,
    detail: (id: string) => [...queryKeys.products.details(), id] as const,
    bySlug: (slug: string) => [...queryKeys.products.all, 'slug', slug] as const,
    reviews: (id: string) => [...queryKeys.products.all, 'reviews', id] as const,
    featured: () => [...queryKeys.products.all, 'featured'] as const,
  },

  // Categories
  categories: {
    all: ['categories'] as const,
    lists: () => [...queryKeys.categories.all, 'list'] as const,
    detail: (id: string) => [...queryKeys.categories.all, 'detail', id] as const,
  },

  // Stores
  stores: {
    all: ['stores'] as const,
    lists: () => [...queryKeys.stores.all, 'list'] as const,
    list: (filters: Record<string, any>) => [...queryKeys.stores.lists(), filters] as const,
    detail: (id: string) => [...queryKeys.stores.all, 'detail', id] as const,
    bySlug: (slug: string) => [...queryKeys.stores.all, 'slug', slug] as const,
    products: (id: string) => [...queryKeys.stores.all, 'products', id] as const,
  },

  // Orders
  orders: {
    all: ['orders'] as const,
    lists: () => [...queryKeys.orders.all, 'list'] as const,
    list: (filters: Record<string, any>) => [...queryKeys.orders.lists(), filters] as const,
    detail: (id: string) => [...queryKeys.orders.all, 'detail', id] as const,
  },

  // Wallet
  wallet: {
    all: ['wallet'] as const,
    balance: () => [...queryKeys.wallet.all, 'balance'] as const,
    transactions: (filters?: Record<string, any>) => [...queryKeys.wallet.all, 'transactions', filters] as const,
  },

  // Search
  search: {
    all: ['search'] as const,
    results: (query: string, filters: Record<string, any>) => [...queryKeys.search.all, query, filters] as const,
  },

  // User
  user: {
    all: ['user'] as const,
    profile: () => [...queryKeys.user.all, 'profile'] as const,
    addresses: () => [...queryKeys.user.all, 'addresses'] as const,
    wishlist: () => [...queryKeys.user.all, 'wishlist'] as const,
  },

  // Vendor
  vendor: {
    dashboard: ['vendor', 'dashboard'] as const,
    products: (filters?: Record<string, any>) => ['vendor', 'products', filters] as const,
    orders: (filters?: Record<string, any>) => ['vendor', 'orders', filters] as const,
    inventory: (filters?: Record<string, any>) => ['vendor', 'inventory', filters] as const,
    coupons: ['vendor', 'coupons'] as const,
    analytics: (params: Record<string, any>) => ['vendor', 'analytics', params] as const,
    team: ['vendor', 'team'] as const,
  },

  // Admin
  admin: {
    dashboard: ['admin', 'dashboard'] as const,
    customers: (filters?: Record<string, any>) => ['admin', 'customers', filters] as const,
    vendors: (filters?: Record<string, any>) => ['admin', 'vendors', filters] as const,
    orders: (filters?: Record<string, any>) => ['admin', 'orders', filters] as const,
    products: (filters?: Record<string, any>) => ['admin', 'products', filters] as const,
    finance: (params: Record<string, any>) => ['admin', 'finance', params] as const,
    coupons: ['admin', 'coupons'] as const,
    tickets: (filters?: Record<string, any>) => ['admin', 'tickets', filters] as const,
    auditLog: (filters?: Record<string, any>) => ['admin', 'audit-log', filters] as const,
  },
} as const;
```

### API Functions with Caching

```typescript
// lib/api/products.ts
import { queryKeys } from '@/lib/queryKeys';

export async function fetchProducts(params: ProductQueryParams) {
  const searchParams = new URLSearchParams();
  Object.entries(params).forEach(([key, value]) => {
    if (value !== undefined && value !== null && value !== '') {
      searchParams.set(key, String(value));
    }
  });
  const response = await fetch(`/api/products?${searchParams}`);
  if (!response.ok) throw new Error('Failed to fetch products');
  return response.json();
}

export async function fetchProductBySlug(slug: string) {
  const response = await fetch(`/api/products/slug/${slug}`);
  if (!response.ok) throw new Error('Product not found');
  return response.json();
}

// Custom hooks with pre-configured cache
export function useProducts(params: ProductQueryParams) {
  return useQuery({
    queryKey: queryKeys.products.list(params),
    queryFn: () => fetchProducts(params),
    keepPreviousData: true,
    staleTime: 2 * 60 * 1000, // 2 minutes for product lists
  });
}

export function useProduct(id: string) {
  return useQuery({
    queryKey: queryKeys.products.detail(id),
    queryFn: () => fetchProduct(id),
    staleTime: 10 * 60 * 1000, // 10 minutes for individual products
    enabled: !!id,
  });
}

export function useFeaturedProducts() {
  return useQuery({
    queryKey: queryKeys.products.featured(),
    queryFn: fetchFeaturedProducts,
    staleTime: 10 * 60 * 1000,
    refetchOnMount: false,
  });
}
```

### Prefetching Strategies

```typescript
// hooks/usePrefetch.ts
import { useQueryClient } from '@tanstack/react-query';
import { queryKeys } from '@/lib/queryKeys';
import { fetchProduct, fetchProducts } from '@/lib/api/products';
import { useCallback } from 'react';

export function usePrefetch() {
  const queryClient = useQueryClient();

  const prefetchProduct = useCallback(
    (id: string) => {
      queryClient.prefetchQuery({
        queryKey: queryKeys.products.detail(id),
        queryFn: () => fetchProduct(id),
        staleTime: 5 * 60 * 1000,
      });
    },
    [queryClient]
  );

  const prefetchProducts = useCallback(
    (params: Record<string, any>) => {
      queryClient.prefetchInfiniteQuery({
        queryKey: queryKeys.products.list(params),
        queryFn: ({ pageParam }) => fetchProducts({ ...params, page: pageParam }),
        getNextPageParam: (lastPage) => lastPage.nextCursor,
        staleTime: 2 * 60 * 1000,
      });
    },
    [queryClient]
  );

  return { prefetchProduct, prefetchProducts };
}
```

---

## 2. Client State: Zustand

### Cart Store

```typescript
// stores/cartStore.ts
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';
import { immer } from 'zustand/middleware/immer';

export interface CartItem {
  id: string;
  name: string;
  price: number;
  image?: string;
  quantity: number;
  maxQuantity?: number;
  storeId?: string;
  storeName?: string;
}

interface CartState {
  items: CartItem[];
  total: number;
  itemCount: number;
  addItem: (item: CartItem) => void;
  removeItem: (id: string) => void;
  updateQuantity: (id: string, quantity: number) => void;
  clearCart: () => void;
  getItem: (id: string) => CartItem | undefined;
}

function recalculateTotals(items: CartItem[]) {
  return {
    total: items.reduce((sum, item) => sum + item.price * item.quantity, 0),
    itemCount: items.reduce((sum, item) => sum + item.quantity, 0),
  };
}

export const useCartStore = create<CartState>()(
  persist(
    immer((set, get) => ({
      items: [],
      total: 0,
      itemCount: 0,

      addItem: (item) =>
        set((state) => {
          const existing = state.items.find((i) => i.id === item.id);
          if (existing) {
            const maxQty = existing.maxQuantity || 99;
            existing.quantity = Math.min(existing.quantity + item.quantity, maxQty);
          } else {
            state.items.push(item);
          }
          const totals = recalculateTotals(state.items);
          state.total = totals.total;
          state.itemCount = totals.itemCount;
        }),

      removeItem: (id) =>
        set((state) => {
          state.items = state.items.filter((i) => i.id !== id);
          const totals = recalculateTotals(state.items);
          state.total = totals.total;
          state.itemCount = totals.itemCount;
        }),

      updateQuantity: (id, quantity) =>
        set((state) => {
          const item = state.items.find((i) => i.id === id);
          if (item) {
            const maxQty = item.maxQuantity || 99;
            item.quantity = Math.max(1, Math.min(quantity, maxQty));
          }
          const totals = recalculateTotals(state.items);
          state.total = totals.total;
          state.itemCount = totals.itemCount;
        }),

      clearCart: () =>
        set((state) => {
          state.items = [];
          state.total = 0;
          state.itemCount = 0;
        }),

      getItem: (id) => get().items.find((i) => i.id === id),
    })),
    {
      name: 'yemenmart-cart',
      storage: createJSONStorage(() => localStorage),
      partialize: (state) => ({ items: state.items }),
    }
  )
);
```

### Auth Store

```typescript
// stores/authStore.ts
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

interface User {
  id: string;
  name: string;
  email: string;
  phone: string;
  avatar?: string;
  role: string;
}

interface AuthState {
  user: User | null;
  token: string | null;
  refreshToken: string | null;
  isAuthenticated: boolean;
  setAuth: (user: User, token: string, refreshToken: string) => void;
  updateUser: (user: Partial<User>) => void;
  logout: () => void;
}

export const useAuthStore = create<AuthState>()(
  persist(
    (set) => ({
      user: null,
      token: null,
      refreshToken: null,
      isAuthenticated: false,

      setAuth: (user, token, refreshToken) =>
        set({ user, token, refreshToken, isAuthenticated: true }),

      updateUser: (userData) =>
        set((state) => ({
          user: state.user ? { ...state.user, ...userData } : null,
        })),

      logout: () => set({
        user: null, token: null, refreshToken: null, isAuthenticated: false,
      }),
    }),
    { name: 'yemenmart-auth' }
  )
);
```

### UI Store

```typescript
// stores/uiStore.ts
import { create } from 'zustand';

interface UIState {
  sidebarOpen: boolean;
  cartDrawerOpen: boolean;
  searchFocused: boolean;
  locale: 'ar' | 'en';
  theme: 'light' | 'dark' | 'system';
  toggleSidebar: () => void;
  setSidebarOpen: (open: boolean) => void;
  toggleCartDrawer: () => void;
  setCartDrawerOpen: (open: boolean) => void;
  setSearchFocused: (focused: boolean) => void;
  setLocale: (locale: 'ar' | 'en') => void;
  setTheme: (theme: 'light' | 'dark' | 'system') => void;
}

export const useUIStore = create<UIState>((set) => ({
  sidebarOpen: true,
  cartDrawerOpen: false,
  searchFocused: false,
  locale: 'ar',
  theme: 'system',

  toggleSidebar: () => set((s) => ({ sidebarOpen: !s.sidebarOpen })),
  setSidebarOpen: (open) => set({ sidebarOpen: open }),
  toggleCartDrawer: () => set((s) => ({ cartDrawerOpen: !s.cartDrawerOpen })),
  setCartDrawerOpen: (open) => set({ cartDrawerOpen: open }),
  setSearchFocused: (focused) => set({ searchFocused: focused }),
  setLocale: (locale) => set({ locale }),
  setTheme: (theme) => set({ theme }),
}));
```

### Checkout Store

```typescript
// stores/checkoutStore.ts
import { create } from 'zustand';

interface CheckoutData {
  address: any | null;
  shipping: { method: string; cost: number; estimatedDays: number } | null;
  payment: { method: string; cardLast4?: string } | null;
  orderId: string | null;
}

interface CheckoutState {
  currentStep: number;
  data: CheckoutData;
  setStep: (step: number) => void;
  setAddress: (address: any) => void;
  setShipping: (shipping: CheckoutData['shipping']) => void;
  setPayment: (payment: CheckoutData['payment']) => void;
  setOrderId: (orderId: string) => void;
  reset: () => void;
}

const initialData: CheckoutData = { address: null, shipping: null, payment: null, orderId: null };

export const useCheckoutStore = create<CheckoutState>((set) => ({
  currentStep: 1,
  data: initialData,
  setStep: (step) => set({ currentStep: step }),
  setAddress: (address) => set((s) => ({ data: { ...s.data, address } })),
  setShipping: (shipping) => set((s) => ({ data: { ...s.data, shipping } })),
  setPayment: (payment) => set((s) => ({ data: { ...s.data, payment } })),
  setOrderId: (orderId) => set((s) => ({ data: { ...s.data, orderId } })),
  reset: () => set({ currentStep: 1, data: initialData }),
}));
```

---

## 3. Form State: React Hook Form + Zod

### Configuration

```typescript
// lib/forms/config.ts
import { zodResolver } from '@hookform/resolvers/zod';
import { useForm, UseFormProps } from 'react-hook-form';
import { z } from 'zod';

// Common validation schemas
export const phoneSchema = z.string().regex(/^7[0-9]{8}$/, 'رقم الهاتف غير صحيح');
export const emailSchema = z.string().email('البريد الإلكتروني غير صحيح');
export const passwordSchema = z.string().min(8, 'كلمة المرور يجب أن تكون 8 أحرف على الأقل');
export const nameSchema = z.string().min(2, 'الاسم مطلوب').max(100);
export const addressSchema = z.string().min(5, 'العنوان مطلوب');
export const priceSchema = z.number().min(0, 'السعر يجب أن يكون موجباً');

// Form hook factory
export function useZodForm<T extends z.ZodType>(
  schema: T,
  options?: UseFormProps<z.infer<T>>
) {
  return useForm<z.infer<T>>({
    resolver: zodResolver(schema),
    mode: 'onBlur',
    reValidateMode: 'onChange',
    ...options,
  });
}
```

### Address Form Example

```typescript
// components/forms/AddressForm.tsx
import { useZodForm } from '@/lib/forms/config';
import { z } from 'zod';
import { Input } from '@/components/ui/Input';
import { Select } from '@/components/ui/Select';
import { Button } from '@/components/ui/Button';

const addressSchema = z.object({
  fullName: z.string().min(2, 'الاسم الكامل مطلوب'),
  phone: z.string().regex(/^7[0-9]{8}$/, 'رقم الهاتف غير صحيح'),
  governorate: z.string().min(1, 'المحافظة مطلوبة'),
  district: z.string().min(1, 'المنطقة مطلوبة'),
  address: z.string().min(5, 'العنوان التفصيلي مطلوب'),
  landmark: z.string().optional(),
});

type AddressFormData = z.infer<typeof addressSchema>;

interface AddressFormProps {
  initialData?: Partial<AddressFormData>;
  onSubmit: (data: AddressFormData) => void;
  onCancel?: () => void;
}

export function AddressForm({ initialData, onSubmit, onCancel }: AddressFormProps) {
  const { register, handleSubmit, formState: { errors, isSubmitting } } = useZodForm(addressSchema, {
    defaultValues: initialData,
  });

  return (
    <form onSubmit={handleSubmit(onSubmit)} className="space-y-4">
      <Input label="الاسم الكامل" {...register('fullName')} error={errors.fullName?.message} />
      <Input label="رقم الجوال" type="tel" {...register('phone')} error={errors.phone?.message} dir="ltr" />
      <Select label="المحافظة" {...register('governorate')} error={errors.governorate?.message}
        options={governorates} />
      <Input label="المنطقة" {...register('district')} error={errors.district?.message} />
      <Input label="العنوان التفصيلي" {...register('address')} error={errors.address?.message} />
      <Input label="علامة مميزة" {...register('landmark')} />

      <div className="flex justify-end gap-2 pt-4">
        {onCancel && <Button type="button" variant="outline" onClick={onCancel}>إلغاء</Button>}
        <Button type="submit" loading={isSubmitting}>حفظ</Button>
      </div>
    </form>
  );
}
```

### Product Form Example

```typescript
const productSchema = z.object({
  name: z.string().min(2, 'اسم المنتج مطلوب').max(200),
  nameEn: z.string().optional(),
  description: z.string().min(10, 'الوصف مطلوب'),
  price: z.number().min(1, 'السعر مطلوب'),
  comparePrice: z.number().optional(),
  sku: z.string().min(3, 'رمز SKU مطلوب'),
  stock: z.number().min(0, 'المخزون مطلوب'),
  categoryId: z.string().min(1, 'القسم مطلوب'),
  images: z.array(z.string()).min(1, 'صورة واحدة على الأقل مطلوبة'),
  specifications: z.array(z.object({
    label: z.string(),
    value: z.string(),
  })).optional(),
});

export function ProductForm({ initialData, onSubmit }: ProductFormProps) {
  const { register, handleSubmit, control, formState: { errors, isSubmitting }, watch } = useZodForm(productSchema, {
    defaultValues: initialData,
  });

  return (
    <form onSubmit={handleSubmit(onSubmit)} className="space-y-6">
      <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
        <Input label="اسم المنتج (عربي)" {...register('name')} error={errors.name?.message} />
        <Input label="اسم المنتج (إنجليزي)" {...register('nameEn')} />
      </div>
      <Textarea label="الوصف" {...register('description')} error={errors.description?.message} rows={4} />
      <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
        <Input label="السعر (يمني)" type="number" {...register('price', { valueAsNumber: true })}
          error={errors.price?.message} />
        <Input label="السعر قبل الخصم" type="number" {...register('comparePrice', { valueAsNumber: true })} />
        <Input label="الكمية" type="number" {...register('stock', { valueAsNumber: true })}
          error={errors.stock?.message} />
      </div>
      <Input label="رمز SKU" {...register('sku')} error={errors.sku?.message} dir="ltr" />
      <Select label="القسم" {...register('categoryId')} error={errors.categoryId?.message}
        options={categories} />
      <Button type="submit" loading={isSubmitting} fullWidth>{initialData ? 'تحديث' : 'إنشاء'}</Button>
    </form>
  );
}
```

---

## 4. URL State: Query Parameters

```typescript
// hooks/useURLFilters.ts
'use client';

import { useRouter, useSearchParams, usePathname } from 'next/navigation';
import { useCallback, useMemo } from 'react';

interface FilterState {
  search?: string;
  category?: string;
  minPrice?: string;
  maxPrice?: string;
  sort?: string;
  page?: number;
  [key: string]: string | number | undefined;
}

export function useURLFilters<T extends FilterState>(defaults: T) {
  const router = useRouter();
  const pathname = usePathname();
  const searchParams = useSearchParams();

  const filters = useMemo(() => {
    const result = { ...defaults };
    searchParams.forEach((value, key) => {
      if (key === 'page') {
        result[key as keyof T] = parseInt(value) as any;
      } else {
        (result as any)[key] = value;
      }
    });
    return result;
  }, [searchParams, defaults]);

  const setFilter = useCallback(
    (key: string, value: string | number | undefined) => {
      const params = new URLSearchParams(searchParams.toString());
      if (value === undefined || value === '' || value === defaults[key as keyof T]) {
        params.delete(key);
      } else {
        params.set(key, String(value));
      }
      // Reset page when filters change
      if (key !== 'page') params.delete('page');
      router.push(`${pathname}?${params.toString()}`, { scroll: false });
    },
    [router, pathname, searchParams, defaults]
  );

  const setFilters = useCallback(
    (updates: Partial<FilterState>) => {
      const params = new URLSearchParams(searchParams.toString());
      Object.entries(updates).forEach(([key, value]) => {
        if (value === undefined || value === '' || value === defaults[key as keyof T]) {
          params.delete(key);
        } else {
          params.set(key, String(value));
        }
      });
      router.push(`${pathname}?${params.toString()}`, { scroll: false });
    },
    [router, pathname, searchParams, defaults]
  );

  const resetFilters = useCallback(() => {
    router.push(pathname, { scroll: false });
  }, [router, pathname]);

  return { filters, setFilter, setFilters, resetFilters };
}

// Usage example:
// const { filters, setFilter, resetFilters } = useURLFilters({
//   search: '', category: '', sort: 'relevance', page: 1,
// });
```

---

## 5. Cache Strategy: Stale-While-Revalidate

```
┌──────────────────────────────────────────────────────────────┐
│                  CACHE TIMING STRATEGY                        │
├──────────────────┬───────────┬───────────┬──────────────────┤
│ Data Type        │ staleTime │ gcTime    │ refetchInterval  │
├──────────────────┼───────────┼───────────┼──────────────────┤
│ Homepage         │ 5 min     │ 30 min    │ Off              │
│ Product Detail   │ 10 min    │ 30 min    │ Off              │
│ Product List     │ 2 min     │ 15 min    │ Off              │
│ Search Results   │ 1 min     │ 5 min     │ Off              │
│ Cart             │ Permanent │ Never     │ Off              │
│ User Profile     │ 5 min     │ 30 min    │ Off              │
│ Wallet Balance   │ 30 sec    │ 5 min     │ 30 sec           │
│ Orders List      │ 1 min     │ 10 min    │ 15 sec (active)  │
│ Vendor Dashboard │ 30 sec    │ 5 min     │ 30 sec           │
│ Notifications    │ 0         │ 5 min     │ 10 sec           │
│ Categories       │ 1 hour    │ 24 hours  │ Off              │
│ Static Content   │ 24 hours  │ 7 days    │ Off              │
└──────────────────┴───────────┴───────────┴──────────────────┘
```

```typescript
// lib/cache-strategy.ts
export const cacheStrategies = {
  // Static data - very long cache
  static: { staleTime: 24 * 60 * 60 * 1000, gcTime: 7 * 24 * 60 * 60 * 1000 },

  // Semi-static data
  semiStatic: { staleTime: 60 * 60 * 1000, gcTime: 24 * 60 * 60 * 1000 },

  // Regular data
  regular: { staleTime: 5 * 60 * 1000, gcTime: 30 * 60 * 1000 },

  // Frequent updates
  frequent: { staleTime: 2 * 60 * 1000, gcTime: 15 * 60 * 1000 },

  // Real-time data
  realtime: { staleTime: 0, gcTime: 5 * 60 * 1000, refetchInterval: 10000 },

  // Financial data - always fresh
  financial: { staleTime: 30 * 1000, gcTime: 5 * 60 * 1000, refetchInterval: 30000 },
} as const;
```

---

## 6. Store Persistence Configuration

```typescript
// stores/persistence.ts
import { createJSONStorage } from 'zustand/middleware';

// localStorage adapter
const localStorageAdapter = createJSONStorage(() => {
  if (typeof window === 'undefined') {
    return {
      getItem: () => null,
      setItem: () => {},
      removeItem: () => {},
    };
  }
  return localStorage;
});

// sessionStorage adapter
const sessionStorageAdapter = createJSONStorage(() => {
  if (typeof window === 'undefined') {
    return {
      getItem: () => null,
      setItem: () => {},
      removeItem: () => {},
    };
  }
  return sessionStorage;
});

export { localStorageAdapter, sessionStorageAdapter };
```

---

## 7. Hydration Handling

```typescript
// hooks/useHydration.ts
import { useState, useEffect } from 'react';

export function useHydration() {
  const [hydrated, setHydrated] = useState(false);

  useEffect(() => {
    setHydrated(true);
  }, []);

  return hydrated;
}

// Usage in layout:
// const hydrated = useHydration();
// if (!hydrated) return <Skeleton />; // Prevent hydration mismatch
```

---

## 8. Cross-Store Communication

```typescript
// stores/middleware/sync.ts
import { StateSync, StorageSync } from 'zustand-middlewares';

// Example: Sync cart to localStorage and notify other tabs
export const syncCartToStorage = (config) => (set, get, api) =>
  config(
    (...args) => {
      set(...args);
      // Broadcast to other tabs
      window.dispatchEvent(new CustomEvent('cart-updated', {
        detail: get().items,
      }));
    },
    get,
    api
  );

// Listen for cart updates from other tabs
if (typeof window !== 'undefined') {
  window.addEventListener('cart-updated', ((e: CustomEvent) => {
    useCartStore.setState({ items: e.detail });
  }) as EventListener);
}
```

---

## 9. DevTools Integration

```typescript
// stores/devtools.ts
import { devtools } from 'zustand/middleware';

// Enable devtools in development only
export const withDevtools = process.env.NODE_ENV === 'development'
  ? devtools
  : (fn: any) => fn;
```

---

## 10. Performance Patterns

### Selective Subscriptions

```typescript
// BAD - re-renders on any cart change
const { items, total, itemCount, addItem } = useCartStore();

// GOOD - only subscribes to specific fields
const items = useCartStore((s) => s.items);
const addItem = useCartStore((s) => s.addItem);

// BEST - use shallow comparison for objects
import { shallow } from 'zustand/shallow';
const { total, itemCount } = useCartStore(
  (s) => ({ total: s.total, itemCount: s.itemCount }),
  shallow
);
```

### Query Optimistic Updates

```typescript
// hooks/useOptimisticUpdate.ts
import { useMutation, useQueryClient } from '@tanstack/react-query';

export function useOptimisticCart() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: addToCartAPI,
    onMutate: async (newItem) => {
      await queryClient.cancelQueries({ queryKey: ['cart'] });
      const previous = queryClient.getQueryData(['cart']);
      queryClient.setQueryData(['cart'], (old: any) => ({
        ...old,
        items: [...old.items, newItem],
      }));
      return { previous };
    },
    onError: (_err, _vars, context) => {
      queryClient.setQueryData(['cart'], context?.previous);
    },
    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: ['cart'] });
    },
  });
}
```
