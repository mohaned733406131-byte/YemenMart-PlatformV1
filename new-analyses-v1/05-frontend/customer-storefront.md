# Customer Storefront - YemenMart

## Overview

Customer-facing frontend built with Next.js 15 (App Router), serving as the primary shopping interface for YemenMart. Arabic-first with full RTL/LTR support, mobile-first responsive design, and comprehensive e-commerce functionality.

## Tech Stack

| Technology | Purpose |
|------------|---------|
| Next.js 15 (App Router) | Framework with SSR/SSG/ISR |
| React 19 | UI library |
| Tailwind CSS 4 | Utility-first styling |
| TanStack Query (React Query) v5 | Server state management |
| Zustand | Client state management |
| React Hook Form + Zod | Form handling and validation |
| next-intl | Internationalization (AR/EN) |
| next-themes | Theme management (dark/light) |
| Lucide React | Icon library |
| Embla Carousel | Carousel/slider component |

## Internationalization

### RTL/LTR Support

```typescript
// next-intl configuration
// app/i18n/request.ts
import { getRequestConfig } from 'next-intl/request';
import { routing } from './routing';

export default getRequestConfig(async ({ requestLocale }) => {
  let locale = await requestLocale;
  if (!locale || !routing.locales.includes(locale as any)) {
    locale = routing.defaultLocale;
  }

  return {
    locale,
    messages: (await import(`../messages/${locale}.json`)).default,
    timeZone: 'Asia/Aden'
  };
});
```

### Locale Structure

```json
{
  "common": {
    "home": "الرئيسية",
    "search": "بحث",
    "cart": "سلة التسوق",
    "account": "حسابي",
    "login": "تسجيل الدخول",
    "register": "إنشاء حساب",
    "logout": "تسجيل الخروج",
    "orders": "طلباتي",
    "wallet": "المحفظة",
    "settings": "الإعدادات"
  },
  "homepage": {
    "hero_title": "تسوّق بأفضل الأسعار",
    "hero_subtitle": "آلاف المنتجات من أفضل المتاجر في اليمن",
    "featured_products": "منتجات مميزة",
    "categories": "الأقسام",
    "nearby_stores": "متاجر قريبة",
    "deals_of_day": "عروض اليوم"
  },
  "product": {
    "add_to_cart": "أضف إلى السلة",
    "buy_now": "اشترِ الآن",
    "reviews": "التقييمات",
    "description": "الوصف",
    "specifications": "المواصفات",
    "shipping": "الشحن",
    "seller": "البائع",
    "in_stock": "متوفر",
    "out_of_stock": "غير متوفر",
    "quantity": "الكمية"
  },
  "checkout": {
    "step_address": "عنوان الشحن",
    "step_shipping": "طريقة الشحن",
    "step_payment": "الدفع",
    "step_review": "مراجعة الطلب",
    "step_confirm": "التأكيد",
    "step_tracking": "تتبع",
    "step_complete": "اكتمل"
  }
}
```

### Language Toggle Component

```typescript
// components/LanguageToggle.tsx
'use client';

import { useLocale, useTranslations } from 'next-intl';
import { useRouter, usePathname } from 'next/navigation';
import { Globe } from 'lucide-react';

export function LanguageToggle() {
  const locale = useLocale();
  const router = useRouter();
  const pathname = usePathname();

  const toggleLocale = () => {
    const newLocale = locale === 'ar' ? 'en' : 'ar';
    router.push(`/${newLocale}${pathname}`);
  };

  return (
    <button
      onClick={toggleLocale}
      className="flex items-center gap-2 px-3 py-2 rounded-lg
                 hover:bg-neutral-100 dark:hover:bg-neutral-800
                 transition-colors"
      aria-label={locale === 'ar' ? 'Switch to English' : 'التبديل إلى العربية'}
    >
      <Globe className="w-4 h-4" />
      <span className="text-sm font-medium">
        {locale === 'ar' ? 'EN' : 'عربي'}
      </span>
    </button>
  );
}
```

## Pages

### 1. Homepage (`/`)

```typescript
// app/[locale]/(main)/page.tsx
import { Suspense } from 'react';
import { HeroSection } from '@/components/home/HeroSection';
import { FeaturedProducts } from '@/components/home/FeaturedProducts';
import { CategoryGrid } from '@/components/home/CategoryGrid';
import { NearbyStores } from '@/components/home/NearbyStores';
import { DealsOfDay } from '@/components/home/DealsOfDay';
import { PromoBanners } from '@/components/home/PromoBanners';
import { NewsletterSignup } from '@/components/home/NewsletterSignup';

export default function HomePage() {
  return (
    <main className="min-h-screen">
      <HeroSection />
      <PromoBanners />
      <Suspense fallback={<ProductSkeleton />}>
        <FeaturedProducts />
      </Suspense>
      <CategoryGrid />
      <Suspense fallback={<StoresSkeleton />}>
        <NearbyStores />
      </Suspense>
      <DealsOfDay />
      <NewsletterSignup />
    </main>
  );
}
```

#### Homepage Sections

```typescript
// components/home/HeroSection.tsx
'use client';

import { useTranslations } from 'next-intl';
import { Search } from 'lucide-react';
import { useRouter } from 'next/navigation';

export function HeroSection() {
  const t = useTranslations('homepage');
  const router = useRouter();

  return (
    <section className="relative bg-gradient-to-br from-primary-600 to-primary-800
                        text-white overflow-hidden">
      <div className="container mx-auto px-4 py-16 md:py-24 relative z-10">
        <h1 className="text-3xl md:text-5xl lg:text-6xl font-bold mb-4
                       text-center md:text-start">
          {t('hero_title')}
        </h1>
        <p className="text-lg md:text-xl text-primary-100 mb-8
                      text-center md:text-start max-w-2xl">
          {t('hero_subtitle')}
        </p>

        {/* Search Bar */}
        <div className="max-w-2xl mx-auto md:mx-0">
          <div className="relative">
            <Search className="absolute right-4 top-1/2 -translate-y-1/2
                              w-5 h-5 text-neutral-400" />
            <input
              type="search"
              placeholder={t('search_placeholder')}
              className="w-full py-4 px-12 rounded-xl text-neutral-900
                         text-lg shadow-lg focus:outline-none focus:ring-4
                         focus:ring-primary-300"
              onKeyDown={(e) => {
                if (e.key === 'Enter') {
                  router.push(`/search?q=${e.currentTarget.value}`);
                }
              }}
            />
          </div>
        </div>
      </div>

      {/* Background decoration */}
      <div className="absolute inset-0 opacity-10">
        <div className="absolute -top-40 -right-40 w-80 h-80 bg-white rounded-full" />
        <div className="absolute -bottom-40 -left-40 w-96 h-96 bg-white rounded-full" />
      </div>
    </section>
  );
}
```

```typescript
// components/home/FeaturedProducts.tsx
'use client';

import { useQuery } from '@tanstack/react-query';
import { ProductCard } from '@/components/product/ProductCard';
import { ProductCardSkeleton } from '@/components/skeletons/ProductCardSkeleton';
import { useTranslations } from 'next-intl';
import { fetchFeaturedProducts } from '@/lib/api/products';

export function FeaturedProducts() {
  const t = useTranslations('homepage');
  const { data: products, isLoading } = useQuery({
    queryKey: ['featured-products'],
    queryFn: fetchFeaturedProducts,
    staleTime: 10 * 60 * 1000, // 10 minutes
  });

  return (
    <section className="container mx-auto px-4 py-12">
      <h2 className="text-2xl md:text-3xl font-bold mb-8">
        {t('featured_products')}
      </h2>

      {isLoading ? (
        <div className="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-4">
          {Array.from({ length: 8 }).map((_, i) => (
            <ProductCardSkeleton key={i} />
          ))}
        </div>
      ) : (
        <div className="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-4">
          {products?.map((product) => (
            <ProductCard key={product.id} product={product} />
          ))}
        </div>
      )}
    </section>
  );
}
```

### 2. Search Page (`/search`)

```typescript
// app/[locale]/(main)/search/page.tsx
'use client';

import { useSearchParams } from 'next/navigation';
import { useQuery } from '@tanstack/react-query';
import { useState, useCallback } from 'react';
import { SearchFilters } from '@/components/search/SearchFilters';
import { SearchResults } from '@/components/search/SearchResults';
import { SearchSort } from '@/components/search/SearchSort';
import { fetchSearchResults } from '@/lib/api/search';

export default function SearchPage() {
  const searchParams = useSearchParams();
  const query = searchParams.get('q') || '';

  const [filters, setFilters] = useState({
    category: searchParams.get('category') || '',
    minPrice: searchParams.get('minPrice') || '',
    maxPrice: searchParams.get('maxPrice') || '',
    rating: searchParams.get('rating') || '',
    brand: searchParams.get('brand') || '',
    sort: searchParams.get('sort') || 'relevance',
    page: parseInt(searchParams.get('page') || '1'),
  });

  const { data, isLoading, isFetching } = useQuery({
    queryKey: ['search', query, filters],
    queryFn: () => fetchSearchResults({ query, ...filters }),
    keepPreviousData: true,
  });

  const handleFilterChange = useCallback((newFilters: Partial<typeof filters>) => {
    setFilters((prev) => ({ ...prev, ...newFilters, page: 1 }));
  }, []);

  const handlePageChange = useCallback((page: number) => {
    setFilters((prev) => ({ ...prev, page }));
    window.scrollTo({ top: 0, behavior: 'smooth' });
  }, []);

  return (
    <div className="container mx-auto px-4 py-6">
      <div className="flex flex-col lg:flex-row gap-6">
        {/* Sidebar Filters */}
        <aside className="w-full lg:w-64 shrink-0">
          <SearchFilters
            filters={filters}
            onFilterChange={handleFilterChange}
            resultCount={data?.total || 0}
          />
        </aside>

        {/* Main Content */}
        <div className="flex-1">
          <div className="flex items-center justify-between mb-4">
            <p className="text-sm text-neutral-500">
              {data?.total || 0} نتيجة لـ &quot;{query}&quot;
              {isFetching && <span className="mr-2">جاري التحديث...</span>}
            </p>
            <SearchSort
              value={filters.sort}
              onChange={(sort) => handleFilterChange({ sort })}
            />
          </div>

          <SearchResults
            products={data?.products || []}
            isLoading={isLoading}
            pagination={data?.pagination}
            onPageChange={handlePageChange}
          />
        </div>
      </div>
    </div>
  );
}
```

### 3. Categories Page (`/categories`)

```typescript
// app/[locale]/(main)/categories/page.tsx
import { Metadata } from 'next';
import { CategoryGrid } from '@/components/categories/CategoryGrid';
import { CategoryBreadcrumbs } from '@/components/categories/CategoryBreadcrumbs';

export const metadata: Metadata = {
  title: 'الأقسام | YemenMart',
  description: 'تصفح جميع الأقسام والتصنيفات في YemenMart',
};

export default function CategoriesPage() {
  return (
    <div className="container mx-auto px-4 py-6">
      <CategoryBreadcrumbs />
      <h1 className="text-3xl font-bold mb-8">الأقسام</h1>
      <CategoryGrid />
    </div>
  );
}
```

### 4. Product Detail Page (`/products/[slug]`)

```typescript
// app/[locale]/(main)/products/[slug]/page.tsx
import { Metadata } from 'next';
import { notFound } from 'next/navigation';
import { ProductGallery } from '@/components/product/ProductGallery';
import { ProductInfo } from '@/components/product/ProductInfo';
import { ProductTabs } from '@/components/product/ProductTabs';
import { RelatedProducts } from '@/components/product/RelatedProducts';
import { Breadcrumbs } from '@/components/ui/Breadcrumbs';
import { fetchProductBySlug } from '@/lib/api/products';

type Props = { params: Promise<{ slug: string; locale: string }> };

export async function generateMetadata({ params }: Props): Promise<Metadata> {
  const { slug } = await params;
  const product = await fetchProductBySlug(slug);
  if (!product) return { title: 'المنتج غير موجود' };

  return {
    title: `${product.name} | YemenMart`,
    description: product.description?.slice(0, 160),
    openGraph: {
      title: product.name,
      description: product.description,
      images: product.images?.map((img) => ({ url: img.url, alt: product.name })),
    },
  };
}

export default async function ProductPage({ params }: Props) {
  const { slug } = await params;
  const product = await fetchProductBySlug(slug);

  if (!product) notFound();

  return (
    <div className="container mx-auto px-4 py-6">
      <Breadcrumbs
        items={[
          { label: 'الرئيسية', href: '/' },
          { label: product.category?.name, href: `/categories/${product.category?.slug}` },
          { label: product.name },
        ]}
      />

      <div className="grid grid-cols-1 lg:grid-cols-2 gap-8 mt-6">
        <ProductGallery images={product.images} />
        <ProductInfo product={product} />
      </div>

      <ProductTabs
        description={product.description}
        specifications={product.specifications}
        reviews={product.reviews}
        productId={product.id}
      />

      <RelatedProducts
        categoryId={product.category?.id}
        currentProductId={product.id}
      />
    </div>
  );
}
```

### 5. Cart Page (`/cart`)

```typescript
// app/[locale]/(main)/cart/page.tsx
'use client';

import { useCartStore } from '@/stores/cartStore';
import { CartItem } from '@/components/cart/CartItem';
import { CartSummary } from '@/components/cart/CartSummary';
import { EmptyCart } from '@/components/cart/EmptyCart';
import { useTranslations } from 'next-intl';

export default function CartPage() {
  const t = useTranslations('cart');
  const { items, total, itemCount, removeItem, updateQuantity, clearCart } = useCartStore();

  if (items.length === 0) {
    return <EmptyCart />;
  }

  return (
    <div className="container mx-auto px-4 py-6">
      <h1 className="text-3xl font-bold mb-8">
        {t('title')} ({itemCount} {t('items')})
      </h1>

      <div className="grid grid-cols-1 lg:grid-cols-3 gap-8">
        {/* Cart Items */}
        <div className="lg:col-span-2 space-y-4">
          {items.map((item) => (
            <CartItem
              key={item.id}
              item={item}
              onUpdateQuantity={updateQuantity}
              onRemove={removeItem}
            />
          ))}

          <button
            onClick={clearCart}
            className="text-sm text-red-500 hover:text-red-600 transition-colors"
          >
            {t('clear_cart')}
          </button>
        </div>

        {/* Cart Summary */}
        <div className="lg:col-span-1">
          <CartSummary
            subtotal={total}
            itemCount={itemCount}
            onCheckout={() => window.location.href = '/checkout'}
          />
        </div>
      </div>
    </div>
  );
}
```

### 6. Checkout Page (`/checkout`) - 7-Step Wizard

```typescript
// app/[locale]/(main)/checkout/page.tsx
'use client';

import { useState } from 'react';
import { useRouter } from 'next/navigation';
import { useCartStore } from '@/stores/cartStore';
import { useCheckoutStore } from '@/stores/checkoutStore';
import { CheckoutStepper } from '@/components/checkout/CheckoutStepper';
import { AddressStep } from '@/components/checkout/steps/AddressStep';
import { ShippingStep } from '@/components/checkout/steps/ShippingStep';
import { PaymentStep } from '@/components/checkout/steps/PaymentStep';
import { ReviewStep } from '@/components/checkout/steps/ReviewStep';
import { ConfirmationStep } from '@/components/checkout/steps/ConfirmationStep';
import { TrackingStep } from '@/components/checkout/steps/TrackingStep';
import { CompletionStep } from '@/components/checkout/steps/CompletionStep';

const CHECKOUT_STEPS = [
  { id: 1, key: 'address', label: 'عنوان الشحن' },
  { id: 2, key: 'shipping', label: 'طريقة الشحن' },
  { id: 3, key: 'payment', label: 'الدفع' },
  { id: 4, key: 'review', label: 'مراجعة الطلب' },
  { id: 5, key: 'confirm', label: 'التأكيد' },
  { id: 6, key: 'tracking', label: 'تتبع' },
  { id: 7, key: 'complete', label: 'اكتمل' },
];

export default function CheckoutPage() {
  const router = useRouter();
  const { items, total, clearCart } = useCartStore();
  const { currentStep, setStep, checkoutData, submitOrder } = useCheckoutStore();
  const [isSubmitting, setIsSubmitting] = useState(false);

  const handleNextStep = () => {
    if (currentStep < 7) {
      setStep(currentStep + 1);
    }
  };

  const handlePrevStep = () => {
    if (currentStep > 1) {
      setStep(currentStep - 1);
    }
  };

  const handleSubmitOrder = async () => {
    setIsSubmitting(true);
    try {
      await submitOrder();
      clearCart();
      handleNextStep();
    } catch (error) {
      console.error('Order submission failed:', error);
    } finally {
      setIsSubmitting(false);
    }
  };

  if (items.length === 0 && currentStep < 5) {
    router.push('/cart');
    return null;
  }

  return (
    <div className="container mx-auto px-4 py-6 max-w-4xl">
      <h1 className="text-3xl font-bold mb-8 text-center">إتمام الطلب</h1>

      <CheckoutStepper
        steps={CHECKOUT_STEPS}
        currentStep={currentStep}
      />

      <div className="mt-8">
        {currentStep === 1 && (
          <AddressStep
            data={checkoutData.address}
            onNext={handleNextStep}
          />
        )}
        {currentStep === 2 && (
          <ShippingStep
            data={checkoutData.shipping}
            cartTotal={total}
            onNext={handleNextStep}
            onBack={handlePrevStep}
          />
        )}
        {currentStep === 3 && (
          <PaymentStep
            data={checkoutData.payment}
            onNext={handleNextStep}
            onBack={handlePrevStep}
          />
        )}
        {currentStep === 4 && (
          <ReviewStep
            checkoutData={checkoutData}
            items={items}
            total={total}
            onNext={handleSubmitOrder}
            onBack={handlePrevStep}
            isSubmitting={isSubmitting}
          />
        )}
        {currentStep === 5 && (
          <ConfirmationStep
            orderId={checkoutData.orderId}
            onNext={handleNextStep}
          />
        )}
        {currentStep === 6 && (
          <TrackingStep
            orderId={checkoutData.orderId}
            onNext={handleNextStep}
          />
        )}
        {currentStep === 7 && (
          <CompletionStep
            orderId={checkoutData.orderId}
          />
        )}
      </div>
    </div>
  );
}
```

#### Checkout Steps Components

```typescript
// components/checkout/steps/AddressStep.tsx
'use client';

import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';
import { Input } from '@/components/ui/Input';
import { Select } from '@/components/ui/Select';
import { Button } from '@/components/ui/Button';

const addressSchema = z.object({
  fullName: z.string().min(2, 'الاسم الكامل مطلوب'),
  phone: z.string().regex(/^7[0-9]{8}$/, 'رقم الهاتف غير صحيح'),
  alternatePhone: z.string().optional(),
  governorate: z.string().min(1, 'المحافظة مطلوبة'),
  district: z.string().min(1, 'المنطقة مطلوبة'),
  address: z.string().min(5, 'العنوان التفصيلي مطلوب'),
  buildingNumber: z.string().optional(),
  floorNumber: z.string().optional(),
  apartmentNumber: z.string().optional(),
  landmark: z.string().optional(),
  isDefault: z.boolean().default(false),
});

type AddressFormData = z.infer<typeof addressSchema>;

const governorates = [
  { value: 'sanaa', label: 'صنعاء' },
  { value: 'aden', label: 'عدن' },
  { value: 'taiz', label: 'تعز' },
  { value: 'hodeidah', label: 'الحديدة' },
  { value: 'marib', label: 'مأرب' },
  { value: 'hadhramaut', label: 'حضرموت' },
  { value: 'ibb', label: 'إب' },
  { value: 'dhamar', label: 'ذيمار' },
  { value: 'al-hudaydah', label: 'لحج' },
  { value: 'shabwa', label: 'شبوة' },
  { value: 'socotra', label: 'سقطرى' },
  { value: 'al-jawf', label: 'الجوف' },
  { value: 'sada', label: 'صعدة' },
  { value: 'raymah', label: 'ريمة' },
  { value: 'almahwit', label: 'المحويت' },
  { value: 'amran', label: 'عمران' },
  { value: 'hajjah', label: 'حجة' },
  { value: 'al-dhale', label: 'الضالع' },
  { value: 'dhale', label: 'الضالع' },
  { value: 'bayda', label: 'بيضاء' },
];

interface AddressStepProps {
  data?: any;
  onNext: (data: AddressFormData) => void;
}

export function AddressStep({ data, onNext }: AddressStepProps) {
  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm<AddressFormData>({
    resolver: zodResolver(addressSchema),
    defaultValues: data || {},
  });

  return (
    <form onSubmit={handleSubmit(onNext)} className="space-y-6">
      <div className="bg-white dark:bg-neutral-800 rounded-xl p-6 shadow-sm">
        <h2 className="text-xl font-semibold mb-6">عنوان الشحن</h2>

        <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
          <Input
            label="الاسم الكامل"
            {...register('fullName')}
            error={errors.fullName?.message}
            dir="rtl"
          />
          <Input
            label="رقم الجوال"
            type="tel"
            {...register('phone')}
            error={errors.phone?.message}
            placeholder="7XXXXXXXX"
            dir="ltr"
          />
          <Input
            label="رقم الجوال البديل (اختياري)"
            type="tel"
            {...register('alternatePhone')}
            dir="ltr"
          />
          <Select
            label="المحافظة"
            {...register('governorate')}
            options={governorates}
            error={errors.governorate?.message}
          />
          <Input
            label="المنطقة/المديرية"
            {...register('district')}
            error={errors.district?.message}
          />
          <Input
            label="رقم المبنى"
            {...register('buildingNumber')}
          />
          <Input
            label="الدور"
            {...register('floorNumber')}
          />
          <Input
            label="رقم الشقة"
            {...register('apartmentNumber')}
          />
        </div>

        <div className="mt-4">
          <Input
            label="العنوان التفصيلي"
            {...register('address')}
            error={errors.address?.message}
            placeholder="الشارع، المنطقة، أي علامات مميزة"
          />
        </div>

        <div className="mt-4">
          <Input
            label="علامة مميزة (اختياري)"
            {...register('landmark')}
            placeholder="بجوار معلم معروف"
          />
        </div>
      </div>

      <div className="flex justify-end">
        <Button type="submit" size="lg">
          متابعة
        </Button>
      </div>
    </form>
  );
}
```

### 7. Account Pages

```typescript
// app/[locale]/(main)/account/layout.tsx
'use client';

import { useAuth } from '@/hooks/useAuth';
import { AccountSidebar } from '@/components/account/AccountSidebar';
import { redirect } from 'next/navigation';

export default function AccountLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  const { user, isAuthenticated } = useAuth();

  if (!isAuthenticated) {
    redirect('/login?redirect=/account');
  }

  return (
    <div className="container mx-auto px-4 py-6">
      <div className="flex flex-col md:flex-row gap-6">
        <AccountSidebar user={user!} />
        <main className="flex-1">{children}</main>
      </div>
    </div>
  );
}
```

```typescript
// app/[locale]/(main)/account/page.tsx
'use client';

import { useAuth } from '@/hooks/useAuth';
import { useQuery } from '@tanstack/react-query';
import { ProfileForm } from '@/components/account/ProfileForm';
import { OrderSummary } from '@/components/account/OrderSummary';
import { WalletBalance } from '@/components/account/WalletBalance';
import { fetchAccountSummary } from '@/lib/api/account';

export default function AccountPage() {
  const { user } = useAuth();
  const { data: summary, isLoading } = useQuery({
    queryKey: ['account-summary'],
    queryFn: fetchAccountSummary,
  });

  return (
    <div className="space-y-6">
      <h1 className="text-2xl font-bold">مرحباً، {user?.name}</h1>

      <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
        <WalletBalance balance={summary?.walletBalance || 0} />
        <OrderSummary
          pending={summary?.pendingOrders || 0}
          delivered={summary?.deliveredOrders || 0}
        />
        <div className="bg-white dark:bg-neutral-800 rounded-xl p-6 shadow-sm">
          <p className="text-sm text-neutral-500">نقاط الولاء</p>
          <p className="text-2xl font-bold text-primary-600">
            {summary?.loyaltyPoints || 0}
          </p>
        </div>
      </div>

      <ProfileForm user={user!} />
    </div>
  );
}
```

### 8. Orders Page (`/account/orders`)

```typescript
// app/[locale]/(main)/account/orders/page.tsx
'use client';

import { useQuery } from '@tanstack/react-query';
import { useState } from 'react';
import { OrderCard } from '@/components/orders/OrderCard';
import { OrderFilters } from '@/components/orders/OrderFilters';
import { OrderDetail } from '@/components/orders/OrderDetail';
import { Pagination } from '@/components/ui/Pagination';
import { fetchOrders } from '@/lib/api/orders';

export default function OrdersPage() {
  const [status, setStatus] = useState<string>('');
  const [page, setPage] = useState(1);
  const [selectedOrder, setSelectedOrder] = useState<string | null>(null);

  const { data, isLoading } = useQuery({
    queryKey: ['orders', { status, page }],
    queryFn: () => fetchOrders({ status, page, limit: 10 }),
    keepPreviousData: true,
  });

  return (
    <div className="space-y-6">
      <h1 className="text-2xl font-bold">طلباتي</h1>

      <OrderFilters
        status={status}
        onStatusChange={(s) => { setStatus(s); setPage(1); }}
      />

      {isLoading ? (
        <div className="space-y-4">
          {Array.from({ length: 3 }).map((_, i) => (
            <div key={i} className="h-32 bg-neutral-100 dark:bg-neutral-800
                                    rounded-xl animate-pulse" />
          ))}
        </div>
      ) : data?.orders.length === 0 ? (
        <div className="text-center py-12">
          <p className="text-neutral-500">لا توجد طبات</p>
        </div>
      ) : (
        <>
          <div className="space-y-4">
            {data?.orders.map((order) => (
              <OrderCard
                key={order.id}
                order={order}
                onSelect={() => setSelectedOrder(order.id)}
              />
            ))}
          </div>

          {data?.pagination && (
            <Pagination
              currentPage={data.pagination.currentPage}
              totalPages={data.pagination.totalPages}
              onPageChange={setPage}
            />
          )}
        </>
      )}

      {selectedOrder && (
        <OrderDetail
          orderId={selectedOrder}
          onClose={() => setSelectedOrder(null)}
        />
      )}
    </div>
  );
}
```

### 9. Wallet Page (`/account/wallet`)

```typescript
// app/[locale]/(main)/account/wallet/page.tsx
'use client';

import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { useState } from 'react';
import { WalletBalance } from '@/components/wallet/WalletBalance';
import { TransactionList } from '@/components/wallet/TransactionList';
import { TopUpModal } from '@/components/wallet/TopUpModal';
import { Button } from '@/components/ui/Button';
import { Plus } from 'lucide-react';
import { fetchWalletData, topUpWallet } from '@/lib/api/wallet';

export default function WalletPage() {
  const queryClient = useQueryClient();
  const [showTopUp, setShowTopUp] = useState(false);

  const { data: walletData, isLoading } = useQuery({
    queryKey: ['wallet'],
    queryFn: fetchWalletData,
  });

  const topUpMutation = useMutation({
    mutationFn: topUpWallet,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['wallet'] });
      setShowTopUp(false);
    },
  });

  return (
    <div className="space-y-6">
      <div className="flex items-center justify-between">
        <h1 className="text-2xl font-bold">المحفظة</h1>
        <Button onClick={() => setShowTopUp(true)}>
          <Plus className="w-4 h-4 ml-2" />
          شحن المحفظة
        </Button>
      </div>

      <WalletBalance
        balance={walletData?.balance || 0}
        currency="YER"
      />

      <TransactionList
        transactions={walletData?.transactions || []}
        isLoading={isLoading}
      />

      {showTopUp && (
        <TopUpModal
          onClose={() => setShowTopUp(false)}
          onSubmit={(amount) => topUpMutation.mutate(amount)}
          isSubmitting={topUpMutation.isLoading}
        />
      )}
    </div>
  );
}
```

### 10. Store Directory (`/stores`)

```typescript
// app/[locale]/(main)/stores/page.tsx
'use client';

import { useQuery } from '@tanstack/react-query';
import { useState } from 'react';
import { StoreCard } from '@/components/stores/StoreCard';
import { StoreSearch } from '@/components/stores/StoreSearch';
import { StoreFilters } from '@/components/stores/StoreFilters';
import { Pagination } from '@/components/ui/Pagination';
import { fetchStores } from '@/lib/api/stores';

export default function StoreDirectoryPage() {
  const [search, setSearch] = useState('');
  const [category, setCategory] = useState('');
  const [governorate, setGovernorate] = useState('');
  const [page, setPage] = useState(1);

  const { data, isLoading } = useQuery({
    queryKey: ['stores', { search, category, governorate, page }],
    queryFn: () => fetchStores({ search, category, governorate, page, limit: 12 }),
    keepPreviousData: true,
  });

  return (
    <div className="container mx-auto px-4 py-6">
      <h1 className="text-3xl font-bold mb-8">المتاجر</h1>

      <div className="flex flex-col md:flex-row gap-4 mb-6">
        <StoreSearch value={search} onChange={setSearch} />
        <StoreFilters
          category={category}
          onCategoryChange={setCategory}
          governorate={governorate}
          onGovernorateChange={setGovernorate}
        />
      </div>

      {isLoading ? (
        <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
          {Array.from({ length: 6 }).map((_, i) => (
            <div key={i} className="h-64 bg-neutral-100 dark:bg-neutral-800
                                    rounded-xl animate-pulse" />
          ))}
        </div>
      ) : (
        <>
          <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
            {data?.stores.map((store) => (
              <StoreCard key={store.id} store={store} />
            ))}
          </div>

          {data?.pagination && (
            <Pagination
              currentPage={data.pagination.currentPage}
              totalPages={data.pagination.totalPages}
              onPageChange={setPage}
            />
          )}
        </>
      )}
    </div>
  );
}
```

### 11. Store Page (`/stores/[slug]`)

```typescript
// app/[locale]/(main)/stores/[slug]/page.tsx
import { Metadata } from 'next';
import { notFound } from 'next/navigation';
import { StoreHeader } from '@/components/stores/StoreHeader';
import { StoreProducts } from '@/components/stores/StoreProducts';
import { StoreInfo } from '@/components/stores/StoreInfo';
import { fetchStoreBySlug } from '@/lib/api/stores';

type Props = { params: Promise<{ slug: string }> };

export async function generateMetadata({ params }: Props): Promise<Metadata> {
  const { slug } = await params;
  const store = await fetchStoreBySlug(slug);
  if (!store) return { title: 'المتجر غير موجود' };

  return {
    title: `${store.name} | YemenMart`,
    description: store.description,
  };
}

export default async function StorePage({ params }: Props) {
  const { slug } = await params;
  const store = await fetchStoreBySlug(slug);

  if (!store) notFound();

  return (
    <div>
      <StoreHeader store={store} />

      <div className="container mx-auto px-4 py-6">
        <div className="grid grid-cols-1 lg:grid-cols-4 gap-8">
          <div className="lg:col-span-3">
            <StoreProducts storeId={store.id} />
          </div>
          <aside className="lg:col-span-1">
            <StoreInfo store={store} />
          </aside>
        </div>
      </div>
    </div>
  );
}
```

## Components

### 1. Header Component

```typescript
// components/layout/Header.tsx
'use client';

import { useState } from 'react';
import Link from 'next/link';
import { useTranslations, useLocale } from 'next-intl';
import { useCartStore } from '@/stores/cartStore';
import { useAuth } from '@/hooks/useAuth';
import { LanguageToggle } from '@/components/LanguageToggle';
import { ThemeToggle } from '@/components/ThemeToggle';
import { CartDrawer } from '@/components/cart/CartDrawer';
import { UserMenu } from '@/components/layout/UserMenu';
import { MobileMenu } from '@/components/layout/MobileMenu';
import {
  ShoppingCart,
  Search,
  Menu,
  MapPin,
  Bell,
} from 'lucide-react';

export function Header() {
  const t = useTranslations('common');
  const locale = useLocale();
  const { itemCount } = useCartStore();
  const { user } = useAuth();
  const [isMobileMenuOpen, setIsMobileMenuOpen] = useState(false);
  const [isCartOpen, setIsCartOpen] = useState(false);
  const isRtl = locale === 'ar';

  const navLinks = [
    { href: '/', label: t('home') },
    { href: '/categories', label: 'الأقسام' },
    { href: '/stores', label: 'المتاجر' },
    { href: '/deals', label: 'العروض' },
  ];

  return (
    <header className="sticky top-0 z-50 bg-white dark:bg-neutral-900
                       border-b border-neutral-200 dark:border-neutral-800
                       shadow-sm">
      {/* Top Bar */}
      <div className="bg-primary-600 text-white text-sm">
        <div className="container mx-auto px-4 py-2 flex items-center
                        justify-between">
          <div className="flex items-center gap-4">
            <span className="hidden md:inline">شحن مجاني للطلبات فوق 50,000 يمني</span>
          </div>
          <div className="flex items-center gap-2">
            <LanguageToggle />
            <ThemeToggle />
          </div>
        </div>
      </div>

      {/* Main Header */}
      <div className="container mx-auto px-4 py-3">
        <div className="flex items-center justify-between gap-4">
          {/* Logo */}
          <Link href="/" className="flex items-center gap-2 shrink-0">
            <div className="w-10 h-10 bg-primary-600 rounded-lg flex items-center
                            justify-center text-white font-bold text-xl">
              Y
            </div>
            <span className="text-xl font-bold hidden sm:inline">
              YemenMart
            </span>
          </Link>

          {/* Search Bar - Desktop */}
          <div className="hidden md:flex flex-1 max-w-xl">
            <div className="relative w-full">
              <Search className={`absolute top-1/2 -translate-y-1/2 w-5 h-5
                                  text-neutral-400 ${isRtl ? 'right-4' : 'left-4'}`} />
              <input
                type="search"
                placeholder="ابحث عن منتجات، متاجر، أو أقسام..."
                className="w-full py-3 px-12 rounded-xl border border-neutral-200
                           dark:border-neutral-700 bg-neutral-50 dark:bg-neutral-800
                           focus:outline-none focus:ring-2 focus:ring-primary-500
                           focus:border-transparent"
              />
            </div>
          </div>

          {/* Right Actions */}
          <div className="flex items-center gap-2">
            {/* Location */}
            <button className="hidden md:flex items-center gap-2 px-3 py-2
                               rounded-lg hover:bg-neutral-100
                               dark:hover:bg-neutral-800 transition-colors">
              <MapPin className="w-5 h-5 text-primary-600" />
              <span className="text-sm">🇾🇪 صنعاء</span>
            </button>

            {/* Notifications */}
            {user && (
              <button className="relative p-2 rounded-lg hover:bg-neutral-100
                                 dark:hover:bg-neutral-800 transition-colors">
                <Bell className="w-5 h-5" />
                <span className="absolute -top-0.5 -right-0.5 w-4 h-4 bg-red-500
                                 rounded-full text-white text-xs flex items-center
                                 justify-center">
                  3
                </span>
              </button>
            )}

            {/* Cart */}
            <button
              onClick={() => setIsCartOpen(true)}
              className="relative p-2 rounded-lg hover:bg-neutral-100
                         dark:hover:bg-neutral-800 transition-colors"
            >
              <ShoppingCart className="w-5 h-5" />
              {itemCount > 0 && (
                <span className="absolute -top-0.5 -right-0.5 w-5 h-5 bg-primary-600
                                 rounded-full text-white text-xs flex items-center
                                 justify-center font-medium">
                  {itemCount}
                </span>
              )}
            </button>

            {/* User Menu */}
            {user ? (
              <UserMenu user={user} />
            ) : (
              <Link
                href="/login"
                className="hidden md:inline-flex px-4 py-2 bg-primary-600
                           text-white rounded-lg hover:bg-primary-700
                           transition-colors font-medium"
              >
                {t('login')}
              </Link>
            )}

            {/* Mobile Menu Toggle */}
            <button
              onClick={() => setIsMobileMenuOpen(true)}
              className="md:hidden p-2 rounded-lg hover:bg-neutral-100
                         dark:hover:bg-neutral-800 transition-colors"
            >
              <Menu className="w-5 h-5" />
            </button>
          </div>
        </div>

        {/* Navigation - Desktop */}
        <nav className="hidden md:flex items-center gap-1 mt-3">
          {navLinks.map((link) => (
            <Link
              key={link.href}
              href={link.href}
              className="px-4 py-2 text-sm font-medium rounded-lg
                         hover:bg-neutral-100 dark:hover:bg-neutral-800
                         transition-colors"
            >
              {link.label}
            </Link>
          ))}
        </nav>
      </div>

      {/* Cart Drawer */}
      <CartDrawer isOpen={isCartOpen} onClose={() => setIsCartOpen(false)} />

      {/* Mobile Menu */}
      <MobileMenu
        isOpen={isMobileMenuOpen}
        onClose={() => setIsMobileMenuOpen(false)}
        links={navLinks}
      />
    </header>
  );
}
```

### 2. Footer Component

```typescript
// components/layout/Footer.tsx
import Link from 'next/link';
import { useTranslations } from 'next-intl';

export function Footer() {
  const t = useTranslations('common');

  return (
    <footer className="bg-neutral-900 text-neutral-300">
      <div className="container mx-auto px-4 py-12">
        <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-8">
          {/* Brand */}
          <div>
            <div className="flex items-center gap-2 mb-4">
              <div className="w-10 h-10 bg-primary-600 rounded-lg flex items-center
                              justify-center text-white font-bold text-xl">
                Y
              </div>
              <span className="text-xl font-bold text-white">YemenMart</span>
            </div>
            <p className="text-sm leading-relaxed mb-4">
              السوق الإلكتروني الأول في اليمن. تسوّق من آلاف المنتجات بأفضل الأسعار
              مع توصيل إلى جميع المحافظات.
            </p>
          </div>

          {/* Quick Links */}
          <div>
            <h3 className="text-white font-semibold mb-4">روابط سريعة</h3>
            <ul className="space-y-2 text-sm">
              <li>
                <Link href="/about" className="hover:text-white transition-colors">
                  من نحن
                </Link>
              </li>
              <li>
                <Link href="/contact" className="hover:text-white transition-colors">
                  تواصل معنا
                </Link>
              </li>
              <li>
                <Link href="/stores" className="hover:text-white transition-colors">
                  افتح متجرك
                </Link>
              </li>
              <li>
                <Link href="/careers" className="hover:text-white transition-colors">
                  الوظائف
                </Link>
              </li>
            </ul>
          </div>

          {/* Customer Service */}
          <div>
            <h3 className="text-white font-semibold mb-4">خدمة العملاء</h3>
            <ul className="space-y-2 text-sm">
              <li>
                <Link href="/help" className="hover:text-white transition-colors">
                  مركز المساعدة
                </Link>
              </li>
              <li>
                <Link href="/shipping" className="hover:text-white transition-colors">
                  سياسة الشحن
                </Link>
              </li>
              <li>
                <Link href="/returns" className="hover:text-white transition-colors">
                  سياسة الإرجاع
                </Link>
              </li>
              <li>
                <Link href="/privacy" className="hover:text-white transition-colors">
                  الخصوصية
                </Link>
              </li>
            </ul>
          </div>

          {/* Contact */}
          <div>
            <h3 className="text-white font-semibold mb-4">تواصل معنا</h3>
            <ul className="space-y-2 text-sm">
              <li>📞 +967 1 234 567</li>
              <li>📧 support@yemenmart.com</li>
              <li>📍 صنعاء، اليمن</li>
            </ul>
            <div className="flex gap-3 mt-4">
              {/* Social icons */}
            </div>
          </div>
        </div>

        <div className="border-t border-neutral-800 mt-8 pt-8 text-center text-sm">
          <p>© {new Date().getFullYear()} YemenMart. جميع الحقوق محفوظة.</p>
        </div>
      </div>
    </footer>
  );
}
```

### 3. Product Card Component

```typescript
// components/product/ProductCard.tsx
'use client';

import Link from 'next/link';
import Image from 'next/image';
import { useTranslations } from 'next-intl';
import { useCartStore } from '@/stores/cartStore';
import { formatPrice } from '@/lib/utils/format';
import { Heart, ShoppingCart, Star } from 'lucide-react';
import { useState } from 'react';

interface Product {
  id: string;
  slug: string;
  name: string;
  price: number;
  originalPrice?: number;
  images: { url: string; alt: string }[];
  rating: number;
  reviewCount: number;
  storeName: string;
  inStock: boolean;
}

interface ProductCardProps {
  product: Product;
}

export function ProductCard({ product }: ProductCardProps) {
  const t = useTranslations('product');
  const addItem = useCartStore((state) => state.addItem);
  const [isWishlisted, setIsWishlisted] = useState(false);

  const discount = product.originalPrice
    ? Math.round(((product.originalPrice - product.price) / product.originalPrice) * 100)
    : 0;

  return (
    <div className="group bg-white dark:bg-neutral-800 rounded-xl shadow-sm
                    hover:shadow-md transition-all duration-200 overflow-hidden">
      {/* Image */}
      <Link href={`/products/${product.slug}`} className="block relative">
        <div className="aspect-square bg-neutral-100 dark:bg-neutral-700
                        overflow-hidden">
          <Image
            src={product.images[0]?.url || '/placeholder-product.png'}
            alt={product.images[0]?.alt || product.name}
            fill
            className="object-cover group-hover:scale-105 transition-transform
                       duration-300"
            sizes="(max-width: 640px) 50vw, (max-width: 1024px) 33vw, 25vw"
          />
        </div>

        {/* Discount Badge */}
        {discount > 0 && (
          <span className="absolute top-2 right-2 bg-red-500 text-white
                           text-xs font-bold px-2 py-1 rounded">
            -{discount}%
          </span>
        )}

        {/* Wishlist Button */}
        <button
          onClick={(e) => {
            e.preventDefault();
            setIsWishlisted(!isWishlisted);
          }}
          className="absolute top-2 left-2 p-2 bg-white/80 dark:bg-neutral-800/80
                     rounded-full opacity-0 group-hover:opacity-100
                     transition-opacity"
        >
          <Heart
            className={`w-4 h-4 ${isWishlisted ? 'fill-red-500 text-red-500' : 'text-neutral-500'}`}
          />
        </button>
      </Link>

      {/* Content */}
      <div className="p-3">
        <Link href={`/products/${product.slug}`}>
          <h3 className="text-sm font-medium line-clamp-2 mb-2 hover:text-primary-600
                         transition-colors min-h-[2.5rem]">
            {product.name}
          </h3>
        </Link>

        {/* Rating */}
        <div className="flex items-center gap-1 mb-2">
          <div className="flex items-center">
            {Array.from({ length: 5 }).map((_, i) => (
              <Star
                key={i}
                className={`w-3 h-3 ${
                  i < Math.round(product.rating)
                    ? 'fill-yellow-400 text-yellow-400'
                    : 'text-neutral-300'
                }`}
              />
            ))}
          </div>
          <span className="text-xs text-neutral-500">
            ({product.reviewCount})
          </span>
        </div>

        {/* Price */}
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

        {/* Store */}
        <p className="text-xs text-neutral-500 mb-3 truncate">
          {product.storeName}
        </p>

        {/* Add to Cart */}
        <button
          onClick={() => {
            if (product.inStock) {
              addItem({
                id: product.id,
                name: product.name,
                price: product.price,
                image: product.images[0]?.url,
                quantity: 1,
              });
            }
          }}
          disabled={!product.inStock}
          className={`w-full py-2 rounded-lg text-sm font-medium transition-colors
                      flex items-center justify-center gap-2
                      ${product.inStock
                        ? 'bg-primary-600 text-white hover:bg-primary-700'
                        : 'bg-neutral-200 text-neutral-500 cursor-not-allowed'
                      }`}
        >
          <ShoppingCart className="w-4 h-4" />
          {product.inStock ? t('add_to_cart') : t('out_of_stock')}
        </button>
      </div>
    </div>
  );
}
```

### 4. Cart Drawer Component

```typescript
// components/cart/CartDrawer.tsx
'use client';

import { useTranslations } from 'next-intl';
import { useCartStore } from '@/stores/cartStore';
import { CartDrawerItem } from '@/components/cart/CartDrawerItem';
import { formatPrice } from '@/lib/utils/format';
import { X, ShoppingCart, ArrowLeft } from 'lucide-react';
import Link from 'next/link';

interface CartDrawerProps {
  isOpen: boolean;
  onClose: () => void;
}

export function CartDrawer({ isOpen, onClose }: CartDrawerProps) {
  const t = useTranslations('cart');
  const { items, total, itemCount, removeItem, updateQuantity } = useCartStore();

  if (!isOpen) return null;

  return (
    <>
      {/* Overlay */}
      <div
        className="fixed inset-0 bg-black/50 z-50"
        onClick={onClose}
      />

      {/* Drawer */}
      <div className={`fixed top-0 h-full w-full max-w-md bg-white
                       dark:bg-neutral-900 shadow-2xl z-50
                       flex flex-col
                       ${useLocale() === 'ar' ? 'left-0' : 'right-0'}`}>
        {/* Header */}
        <div className="flex items-center justify-between p-4 border-b
                        border-neutral-200 dark:border-neutral-800">
          <div className="flex items-center gap-2">
            <ShoppingCart className="w-5 h-5" />
            <h2 className="text-lg font-semibold">
              {t('title')} ({itemCount})
            </h2>
          </div>
          <button
            onClick={onClose}
            className="p-2 rounded-lg hover:bg-neutral-100
                       dark:hover:bg-neutral-800 transition-colors"
          >
            <X className="w-5 h-5" />
          </button>
        </div>

        {/* Items */}
        <div className="flex-1 overflow-y-auto p-4">
          {items.length === 0 ? (
            <div className="flex flex-col items-center justify-center h-full
                            text-neutral-500">
              <ShoppingCart className="w-16 h-16 mb-4 opacity-50" />
              <p>{t('empty_cart')}</p>
            </div>
          ) : (
            <div className="space-y-4">
              {items.map((item) => (
                <CartDrawerItem
                  key={item.id}
                  item={item}
                  onUpdateQuantity={updateQuantity}
                  onRemove={removeItem}
                />
              ))}
            </div>
          )}
        </div>

        {/* Footer */}
        {items.length > 0 && (
          <div className="border-t border-neutral-200 dark:border-neutral-800
                          p-4 space-y-4">
            <div className="flex items-center justify-between">
              <span className="text-neutral-500">{t('subtotal')}</span>
              <span className="text-xl font-bold">
                {formatPrice(total)}
              </span>
            </div>

            <Link
              href="/checkout"
              onClick={onClose}
              className="flex items-center justify-center gap-2 w-full py-3
                         bg-primary-600 text-white rounded-xl hover:bg-primary-700
                         transition-colors font-medium"
            >
              {t('checkout')}
              <ArrowLeft className="w-4 h-4" />
            </Link>
          </div>
        )}
      </div>
    </>
  );
}
```

### 5. Order Timeline Component

```typescript
// components/orders/OrderTimeline.tsx
'use client';

import { useLocale } from 'next-intl';
import { Check, Clock, Package, Truck, MapPin, XCircle } from 'lucide-react';

interface TimelineStep {
  status: string;
  label: string;
  timestamp?: string;
  completed: boolean;
  current?: boolean;
}

interface OrderTimelineProps {
  steps: TimelineStep[];
}

export function OrderTimeline({ steps }: OrderTimelineProps) {
  const locale = useLocale();
  const isRtl = locale === 'ar';

  const getIcon = (status: string) => {
    switch (status) {
      case 'placed': return Clock;
      case 'confirmed': return Check;
      case 'processing': return Package;
      case 'shipped': return Truck;
      case 'delivered': return MapPin;
      case 'cancelled': return XCircle;
      default: return Clock;
    }
  };

  return (
    <div className="relative">
      {steps.map((step, index) => {
        const Icon = getIcon(step.status);
        const isLast = index === steps.length - 1;

        return (
          <div
            key={step.status}
            className={`flex gap-4 ${!isLast ? 'pb-8' : ''} relative`}
          >
            {/* Line */}
            {!isLast && (
              <div className={`absolute top-8 ${isRtl ? 'right-4' : 'left-4'}
                               w-0.5 h-full ${
                                 step.completed
                                   ? 'bg-primary-600'
                                   : 'bg-neutral-200 dark:bg-neutral-700'
                               }`} />
            )}

            {/* Icon */}
            <div className={`relative z-10 w-8 h-8 rounded-full flex items-center
                             justify-center shrink-0
                             ${step.completed
                               ? 'bg-primary-600 text-white'
                               : step.current
                                 ? 'bg-primary-100 text-primary-600 ring-4 ring-primary-200'
                                 : 'bg-neutral-200 dark:bg-neutral-700 text-neutral-500'
                             }`}>
              <Icon className="w-4 h-4" />
            </div>

            {/* Content */}
            <div className="flex-1">
              <p className={`font-medium ${
                step.completed || step.current
                  ? 'text-neutral-900 dark:text-white'
                  : 'text-neutral-500'
              }`}>
                {step.label}
              </p>
              {step.timestamp && (
                <p className="text-sm text-neutral-500 mt-1">
                  {new Date(step.timestamp).toLocaleDateString('ar-YE', {
                    year: 'numeric',
                    month: 'long',
                    day: 'numeric',
                    hour: '2-digit',
                    minute: '2-digit',
                  })}
                </p>
              )}
            </div>
          </div>
        );
      })}
    </div>
  );
}
```

## State Management

### Cart Store (Zustand)

```typescript
// stores/cartStore.ts
import { create } from 'zustand';
import { persist } from 'zustand/middleware';
import { immer } from 'zustand/middleware/immer';

interface CartItem {
  id: string;
  name: string;
  price: number;
  image?: string;
  quantity: number;
  maxQuantity?: number;
}

interface CartState {
  items: CartItem[];
  total: number;
  itemCount: number;
  addItem: (item: CartItem) => void;
  removeItem: (id: string) => void;
  updateQuantity: (id: string, quantity: number) => void;
  clearCart: () => void;
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
            existing.quantity += item.quantity;
          } else {
            state.items.push(item);
          }
          state.total = state.items.reduce(
            (sum, i) => sum + i.price * i.quantity,
            0
          );
          state.itemCount = state.items.reduce(
            (sum, i) => sum + i.quantity,
            0
          );
        }),

      removeItem: (id) =>
        set((state) => {
          state.items = state.items.filter((i) => i.id !== id);
          state.total = state.items.reduce(
            (sum, i) => sum + i.price * i.quantity,
            0
          );
          state.itemCount = state.items.reduce(
            (sum, i) => sum + i.quantity,
            0
          );
        }),

      updateQuantity: (id, quantity) =>
        set((state) => {
          const item = state.items.find((i) => i.id === id);
          if (item) {
            item.quantity = quantity;
          }
          state.total = state.items.reduce(
            (sum, i) => sum + i.price * i.quantity,
            0
          );
          state.itemCount = state.items.reduce(
            (sum, i) => sum + i.quantity,
            0
          );
        }),

      clearCart: () =>
        set((state) => {
          state.items = [];
          state.total = 0;
          state.itemCount = 0;
        }),
    })),
    {
      name: 'yemenmart-cart',
    }
  )
);
```

### Checkout Store (Zustand)

```typescript
// stores/checkoutStore.ts
import { create } from 'zustand';

interface Address {
  fullName: string;
  phone: string;
  alternatePhone?: string;
  governorate: string;
  district: string;
  address: string;
  buildingNumber?: string;
  floorNumber?: string;
  apartmentNumber?: string;
  landmark?: string;
}

interface Shipping {
  method: 'standard' | 'express' | 'same-day';
  cost: number;
  estimatedDays: number;
}

interface Payment {
  method: 'cod' | 'wallet' | 'card';
  cardLast4?: string;
}

interface CheckoutState {
  currentStep: number;
  checkoutData: {
    address: Address | null;
    shipping: Shipping | null;
    payment: Payment | null;
    orderId: string | null;
  };
  setStep: (step: number) => void;
  setAddress: (address: Address) => void;
  setShipping: (shipping: Shipping) => void;
  setPayment: (payment: Payment) => void;
  setOrderId: (orderId: string) => void;
  submitOrder: () => Promise<void>;
  reset: () => void;
}

export const useCheckoutStore = create<CheckoutState>((set, get) => ({
  currentStep: 1,
  checkoutData: {
    address: null,
    shipping: null,
    payment: null,
    orderId: null,
  },

  setStep: (step) => set({ currentStep: step }),

  setAddress: (address) =>
    set((state) => ({
      checkoutData: { ...state.checkoutData, address },
    })),

  setShipping: (shipping) =>
    set((state) => ({
      checkoutData: { ...state.checkoutData, shipping },
    })),

  setPayment: (payment) =>
    set((state) => ({
      checkoutData: { ...state.checkoutData, payment },
    })),

  setOrderId: (orderId) =>
    set((state) => ({
      checkoutData: { ...state.checkoutData, orderId },
    })),

  submitOrder: async () => {
    const { checkoutData } = get();
    const response = await fetch('/api/orders', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(checkoutData),
    });
    const data = await response.json();
    set((state) => ({
      checkoutData: { ...state.checkoutData, orderId: data.orderId },
    }));
  },

  reset: () =>
    set({
      currentStep: 1,
      checkoutData: {
        address: null,
        shipping: null,
        payment: null,
        orderId: null,
      },
    }),
}));
```

## Responsive Design

### Breakpoint Strategy

```css
/* tailwind.config.ts */
import type { Config } from 'tailwindcss';

const config: Config = {
  content: [
    './pages/**/*.{js,ts,jsx,tsx,mdx}',
    './components/**/*.{js,ts,jsx,tsx,mdx}',
    './app/**/*.{js,ts,jsx,tsx,mdx}',
  ],
  theme: {
    extend: {
      screens: {
        'xs': '475px',
        'sm': '640px',
        'md': '768px',
        'lg': '1024px',
        'xl': '1280px',
        '2xl': '1536px',
      },
      colors: {
        primary: {
          50: '#eff6ff',
          100: '#dbeafe',
          200: '#bfdbfe',
          300: '#93c5fd',
          400: '#60a5fa',
          500: '#3b82f6',
          600: '#2563eb',
          700: '#1d4ed8',
          800: '#1e40af',
          900: '#1e3a8a',
          950: '#172554',
        },
      },
    },
  },
  plugins: [],
};

export default config;
```

## Project Structure

```
customer-storefront/
├── app/
│   ├── [locale]/
│   │   ├── (auth)/
│   │   │   ├── login/page.tsx
│   │   │   ├── register/page.tsx
│   │   │   └── layout.tsx
│   │   ├── (main)/
│   │   │   ├── page.tsx
│   │   │   ├── layout.tsx
│   │   │   ├── search/page.tsx
│   │   │   ├── categories/
│   │   │   │   ├── page.tsx
│   │   │   │   └── [slug]/page.tsx
│   │   │   ├── products/
│   │   │   │   └── [slug]/page.tsx
│   │   │   ├── cart/page.tsx
│   │   │   ├── checkout/page.tsx
│   │   │   ├── stores/
│   │   │   │   ├── page.tsx
│   │   │   │   └── [slug]/page.tsx
│   │   │   ├── account/
│   │   │   │   ├── layout.tsx
│   │   │   │   ├── page.tsx
│   │   │   │   ├── orders/
│   │   │   │   │   ├── page.tsx
│   │   │   │   │   └── [id]/page.tsx
│   │   │   │   ├── wallet/page.tsx
│   │   │   │   ├── addresses/page.tsx
│   │   │   │   ├── wishlist/page.tsx
│   │   │   │   └── settings/page.tsx
│   │   │   └── help/page.tsx
│   │   └── layout.tsx
│   ├── api/ (Next.js API routes)
│   └── globals.css
├── components/
│   ├── layout/
│   │   ├── Header.tsx
│   │   ├── Footer.tsx
│   │   ├── Navigation.tsx
│   │   ├── MobileMenu.tsx
│   │   └── UserMenu.tsx
│   ├── home/
│   │   ├── HeroSection.tsx
│   │   ├── FeaturedProducts.tsx
│   │   ├── CategoryGrid.tsx
│   │   ├── NearbyStores.tsx
│   │   ├── DealsOfDay.tsx
│   │   ├── PromoBanners.tsx
│   │   └── NewsletterSignup.tsx
│   ├── product/
│   │   ├── ProductCard.tsx
│   │   ├── ProductGallery.tsx
│   │   ├── ProductInfo.tsx
│   │   ├── ProductTabs.tsx
│   │   └── RelatedProducts.tsx
│   ├── cart/
│   │   ├── CartItem.tsx
│   │   ├── CartDrawer.tsx
│   │   ├── CartDrawerItem.tsx
│   │   ├── CartSummary.tsx
│   │   └── EmptyCart.tsx
│   ├── checkout/
│   │   ├── CheckoutStepper.tsx
│   │   └── steps/
│   │       ├── AddressStep.tsx
│   │       ├── ShippingStep.tsx
│   │       ├── PaymentStep.tsx
│   │       ├── ReviewStep.tsx
│   │       ├── ConfirmationStep.tsx
│   │       ├── TrackingStep.tsx
│   │       └── CompletionStep.tsx
│   ├── orders/
│   │   ├── OrderCard.tsx
│   │   ├── OrderDetail.tsx
│   │   ├── OrderTimeline.tsx
│   │   └── OrderFilters.tsx
│   ├── account/
│   │   ├── AccountSidebar.tsx
│   │   ├── ProfileForm.tsx
│   │   ├── OrderSummary.tsx
│   │   └── WalletBalance.tsx
│   ├── wallet/
│   │   ├── WalletBalance.tsx
│   │   ├── TransactionList.tsx
│   │   └── TopUpModal.tsx
│   ├── stores/
│   │   ├── StoreCard.tsx
│   │   ├── StoreHeader.tsx
│   │   ├── StoreProducts.tsx
│   │   ├── StoreInfo.tsx
│   │   ├── StoreSearch.tsx
│   │   └── StoreFilters.tsx
│   ├── search/
│   │   ├── SearchFilters.tsx
│   │   ├── SearchResults.tsx
│   │   └── SearchSort.tsx
│   ├── ui/
│   │   ├── Button.tsx
│   │   ├── Input.tsx
│   │   ├── Select.tsx
│   │   ├── Modal.tsx
│   │   ├── Pagination.tsx
│   │   ├── Breadcrumbs.tsx
│   │   ├── Skeleton.tsx
│   │   ├── Badge.tsx
│   │   ├── Tooltip.tsx
│   │   └── Toast.tsx
│   ├── skeletons/
│   │   └── ProductCardSkeleton.tsx
│   └── providers/
│       ├── QueryProvider.tsx
│       ├── ThemeProvider.tsx
│       └── AuthProvider.tsx
├── stores/
│   ├── cartStore.ts
│   ├── checkoutStore.ts
│   ├── uiStore.ts
│   └── authStore.ts
├── hooks/
│   ├── useAuth.ts
│   ├── useDebounce.ts
│   ├── useMediaQuery.ts
│   └── useIntersection.ts
├── lib/
│   ├── api/
│   │   ├── products.ts
│   │   ├── categories.ts
│   │   ├── stores.ts
│   │   ├── orders.ts
│   │   ├── wallet.ts
│   │   └── search.ts
│   ├── utils/
│   │   ├── format.ts
│   │   ├── validate.ts
│   │   └── helpers.ts
│   └── constants/
│       ├── config.ts
│       └── routes.ts
├── messages/
│   ├── ar.json
│   └── en.json
├── public/
│   ├── images/
│   └── icons/
├── middleware.ts
├── next.config.ts
├── tailwind.config.ts
├── tsconfig.json
├── package.json
└── .env.local
```

## Performance Optimization

### Image Optimization

```typescript
// next.config.ts
import type { NextConfig } from 'next';

const nextConfig: NextConfig = {
  images: {
    remotePatterns: [
      {
        protocol: 'https',
        hostname: 'images.yemenmart.com',
      },
      {
        protocol: 'https',
        hostname: 'cdn.yemenmart.com',
      },
    ],
    formats: ['image/avif', 'image/webp'],
    deviceSizes: [640, 750, 828, 1080, 1200, 1920, 2048],
    imageSizes: [16, 32, 48, 64, 96, 128, 256, 384],
  },
};

export default nextConfig;
```

### Code Splitting

```typescript
// Lazy load heavy components
const CheckoutSteps = dynamic(
  () => import('@/components/checkout/CheckoutStepper'),
  { loading: () => <CheckoutSkeleton /> }
);

const ProductGallery = dynamic(
  () => import('@/components/product/ProductGallery'),
  { ssr: false }
);
```

### Caching Strategy

```typescript
// lib/api/products.ts
import { QueryClient } from '@tanstack/react-query';

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000, // 5 minutes
      gcTime: 30 * 60 * 1000, // 30 minutes (formerly cacheTime)
      retry: 2,
      refetchOnWindowFocus: false,
    },
  },
});

export async function fetchProducts(params: ProductQueryParams) {
  const searchParams = new URLSearchParams();
  Object.entries(params).forEach(([key, value]) => {
    if (value !== undefined && value !== null) {
      searchParams.set(key, String(value));
    }
  });

  const response = await fetch(`/api/products?${searchParams}`);
  if (!response.ok) throw new Error('Failed to fetch products');
  return response.json();
}
```

## Accessibility

```typescript
// components/ui/Button.tsx
import { forwardRef } from 'react';
import { Slot } from '@radix-ui/react-slot';

interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: 'primary' | 'secondary' | 'outline' | 'ghost';
  size?: 'sm' | 'md' | 'lg';
  loading?: boolean;
  asChild?: boolean;
}

export const Button = forwardRef<HTMLButtonElement, ButtonProps>(
  ({ className, variant = 'primary', size = 'md', loading, asChild, children, disabled, ...props }, ref) => {
    const Comp = asChild ? Slot : 'button';

    return (
      <Comp
        ref={ref}
        className={cn(
          'inline-flex items-center justify-center rounded-lg font-medium',
          'transition-colors focus-visible:outline-none focus-visible:ring-2',
          'focus-visible:ring-primary-500 focus-visible:ring-offset-2',
          'disabled:pointer-events-none disabled:opacity-50',
          // Variant styles
          variant === 'primary' && 'bg-primary-600 text-white hover:bg-primary-700',
          variant === 'secondary' && 'bg-neutral-100 text-neutral-900 hover:bg-neutral-200',
          variant === 'outline' && 'border border-neutral-300 hover:bg-neutral-50',
          variant === 'ghost' && 'hover:bg-neutral-100',
          // Size styles
          size === 'sm' && 'h-8 px-3 text-sm',
          size === 'md' && 'h-10 px-4 text-sm',
          size === 'lg' && 'h-12 px-6 text-base',
          className
        )}
        disabled={disabled || loading}
        aria-busy={loading}
        {...props}
      >
        {loading && (
          <svg
            className="animate-spin -ml-1 mr-2 h-4 w-4"
            xmlns="http://www.w3.org/2000/svg"
            fill="none"
            viewBox="0 0 24 24"
            aria-hidden="true"
          >
            <circle
              className="opacity-25"
              cx="12"
              cy="12"
              r="10"
              stroke="currentColor"
              strokeWidth="4"
            />
            <path
              className="opacity-75"
              fill="currentColor"
              d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4z"
            />
          </svg>
        )}
        {children}
      </Comp>
    );
  }
);

Button.displayName = 'Button';
```

## Testing Strategy

```typescript
// __tests__/components/ProductCard.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { ProductCard } from '@/components/product/ProductCard';
import { useCartStore } from '@/stores/cartStore';

const mockProduct = {
  id: '1',
  slug: 'test-product',
  name: 'Test Product',
  price: 1000,
  originalPrice: 1500,
  images: [{ url: '/test.png', alt: 'Test' }],
  rating: 4.5,
  reviewCount: 100,
  storeName: 'Test Store',
  inStock: true,
};

describe('ProductCard', () => {
  it('renders product information', () => {
    render(<ProductCard product={mockProduct} />);

    expect(screen.getByText('Test Product')).toBeInTheDocument();
    expect(screen.getByText('1,000 يمني')).toBeInTheDocument();
    expect(screen.getByText('Test Store')).toBeInTheDocument();
  });

  it('shows discount badge', () => {
    render(<ProductCard product={mockProduct} />);
    expect(screen.getByText('-33%')).toBeInTheDocument();
  });

  it('adds to cart on button click', () => {
    const addItem = jest.fn();
    useCartStore.setState({ addItem });

    render(<ProductCard product={mockProduct} />);
    fireEvent.click(screen.getByText('أضف إلى السلة'));

    expect(addItem).toHaveBeenCalledWith(
      expect.objectContaining({ id: '1', name: 'Test Product' })
    );
  });

  it('disables add to cart when out of stock', () => {
    render(<ProductCard product={{ ...mockProduct, inStock: false }} />);
    expect(screen.getByText('غير متوفر')).toBeDisabled();
  });
});
```
