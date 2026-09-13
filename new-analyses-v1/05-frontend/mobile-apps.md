# Mobile Apps - YemenMart

## Overview

React Native mobile applications for YemenMart covering both customer and vendor experiences. Built with Expo for cross-platform compatibility (iOS & Android), featuring push notifications, biometric authentication (planned), offline browsing support, and Arabic-first interface with full RTL support.

## Tech Stack

| Technology | Purpose |
|------------|---------|
| React Native (Expo) | Cross-platform framework |
| React Navigation v6 | Navigation |
| TanStack Query v5 | Server state management |
| Zustand | Client state management |
| React Hook Form + Zod | Forms and validation |
| React Native Reanimated | Animations |
| React Native Gesture Handler | Touch gestures |
| Expo Notifications | Push notifications |
| Expo SecureStore | Secure storage |
| AsyncStorage | Local persistence |
| MMKV | Fast key-value storage |
| React Native Fast Image | Optimized image loading |
| Lottie React Native | Animated illustrations |

## Project Structure

```
mobile-apps/
├── apps/
│   ├── customer/                  # Customer app
│   │   ├── app/                   # Expo Router
│   │   │   ├── (auth)/
│   │   │   │   ├── login.tsx
│   │   │   │   ├── register.tsx
│   │   │   │   └── _layout.tsx
│   │   │   ├── (tabs)/
│   │   │   │   ├── index.tsx      # Home
│   │   │   │   ├── search.tsx
│   │   │   │   ├── cart.tsx
│   │   │   │   ├── orders.tsx
│   │   │   │   └── account.tsx
│   │   │   ├── product/[id].tsx
│   │   │   ├── store/[id].tsx
│   │   │   ├── checkout/
│   │   │   │   ├── index.tsx
│   │   │   │   ├── address.tsx
│   │   │   │   ├── shipping.tsx
│   │   │   │   ├── payment.tsx
│   │   │   │   └── confirmation.tsx
│   │   │   ├── wallet/
│   │   │   │   ├── index.tsx
│   │   │   │   └── topup.tsx
│   │   │   ├── order/[id].tsx
│   │   │   ├── reviews/[productId].tsx
│   │   │   ├── favorites.tsx
│   │   │   ├── addresses.tsx
│   │   │   ├── settings.tsx
│   │   │   └── _layout.tsx
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── stores/
│   │   ├── lib/
│   │   └── app.json
│   └── vendor/                     # Vendor app
│       ├── app/
│       │   ├── (auth)/
│       │   │   └── login.tsx
│       │   ├── (tabs)/
│       │   │   ├── index.tsx      # Dashboard
│       │   │   ├── products.tsx
│       │   │   ├── orders.tsx
│       │   │   └── settings.tsx
│       │   ├── product/[id].tsx
│       │   ├── order/[id].tsx
│       │   └── _layout.tsx
│       ├── components/
│       ├── hooks/
│       ├── stores/
│       └── app.json
├── packages/
│   ├── shared/                    # Shared code
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── utils/
│   │   └── types/
│   └── api/                       # API client
│       ├── client.ts
│       ├── products.ts
│       ├── orders.ts
│       ├── wallet.ts
│       └── stores.ts
├── app.json
├── package.json
├── tsconfig.json
└── babel.config.js
```

## Customer App

### Home Screen

```typescript
// apps/customer/app/(tabs)/index.tsx
import { View, ScrollView, RefreshControl } from 'react-native';
import { SafeAreaView } from 'react-native-safe-area-context';
import { useQuery } from '@tanstack/react-query';
import { useTranslation } from 'react-i18next';
import { HeroBanner } from '@/components/home/HeroBanner';
import { CategoryCarousel } from '@/components/home/CategoryCarousel';
import { FeaturedProducts } from '@/components/home/FeaturedProducts';
import { FlashDeals } from '@/components/home/FlashDeals';
import { NearbyStores } from '@/components/home/NearbyStores';
import { PromoBanners } from '@/components/home/PromoBanners';
import { SearchBar } from '@/components/ui/SearchBar';
import { fetchHomepage } from '@/lib/api/home';
import { useState, useCallback } from 'react';

export default function HomeScreen() {
  const { t } = useTranslation();
  const [refreshing, setRefreshing] = useState(false);

  const { data, isLoading, refetch } = useQuery({
    queryKey: ['homepage'],
    queryFn: fetchHomepage,
    staleTime: 5 * 60 * 1000,
  });

  const onRefresh = useCallback(async () => {
    setRefreshing(true);
    await refetch();
    setRefreshing(false);
  }, [refetch]);

  return (
    <SafeAreaView className="flex-1 bg-white dark:bg-neutral-900">
      <ScrollView
        refreshControl={
          <RefreshControl refreshing={refreshing} onRefresh={onRefresh}
            tintColor="#2563eb" colors={['#2563eb']} />
        }
        showsVerticalScrollIndicator={false}
      >
        {/* Search Bar */}
        <SearchBar placeholder={t('home.search_placeholder')} />

        {/* Hero Banner */}
        <HeroBanner banners={data?.banners || []} />

        {/* Categories */}
        <CategoryCarousel categories={data?.categories || []} />

        {/* Flash Deals */}
        <FlashDeals deals={data?.flashDeals || []} />

        {/* Featured Products */}
        <FeaturedProducts products={data?.featuredProducts || []} />

        {/* Promo Banners */}
        <PromoBanners banners={data?.promoBanners || []} />

        {/* Nearby Stores */}
        <NearbyStores stores={data?.nearbyStores || []} />

        {/* Bottom Spacing */}
        <View className="h-20" />
      </ScrollView>
    </SafeAreaView>
  );
}
```

### Product Detail Screen

```typescript
// apps/customer/app/product/[id].tsx
import { View, ScrollView, Text, Image, Pressable, Dimensions } from 'react-native';
import { useLocalSearchParams, useRouter } from 'expo-router';
import { SafeAreaView } from 'react-native-safe-area-context';
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { useTranslation } from 'react-i18next';
import { useState, useRef } from 'react';
import { fetchProduct, addReview } from '@/lib/api/products';
import { useCartStore } from '@/stores/cartStore';
import { formatPrice } from '@/lib/utils';
import { Rating } from '@/components/ui/Rating';
import { Button } from '@/components/ui/Button';
import { QuantitySelector } from '@/components/ui/QuantitySelector';
import { ReviewCard } from '@/components/product/ReviewCard';
import { WriteReviewModal } from '@/components/product/WriteReviewModal';
import { Ionicons } from '@expo/vector-icons';

const { width } = Dimensions.get('window');

export default function ProductDetailScreen() {
  const { id } = useLocalSearchParams<{ id: string }>();
  const router = useRouter();
  const { t } = useTranslation();
  const addItem = useCartStore((s) => s.addItem);
  const [quantity, setQuantity] = useState(1);
  const [selectedImageIndex, setSelectedImageIndex] = useState(0);
  const [showReviewModal, setShowReviewModal] = useState(false);
  const scrollRef = useRef<ScrollView>(null);

  const { data: product, isLoading } = useQuery({
    queryKey: ['product', id],
    queryFn: () => fetchProduct(id!),
    enabled: !!id,
  });

  const reviewMutation = useMutation({
    mutationFn: addReview,
    onSuccess: () => setShowReviewModal(false),
  });

  if (isLoading || !product) {
    return <View className="flex-1 items-center justify-center"><Text>جاري التحميل...</Text></View>;
  }

  return (
    <SafeAreaView className="flex-1 bg-white dark:bg-neutral-900">
      <ScrollView ref={scrollRef} showsVerticalScrollIndicator={false}>
        {/* Image Gallery */}
        <View className="relative">
          <ScrollView horizontal pagingEnabled
            onMomentumScrollEnd={(e) => {
              const index = Math.round(e.nativeEvent.contentOffset.x / width);
              setSelectedImageIndex(index);
            }}>
            {product.images.map((image: any, index: number) => (
              <Image key={index} source={{ uri: image.url }}
                style={{ width, height: width }} resizeMode="cover" />
            ))}
          </ScrollView>

          {/* Image Indicators */}
          <View className="absolute bottom-4 flex-row self-center gap-1">
            {product.images.map((_: any, index: number) => (
              <View key={index}
                className={`w-2 h-2 rounded-full ${
                  index === selectedImageIndex ? 'bg-primary-600' : 'bg-white/50'
                }`} />
            ))}
          </View>

          {/* Back Button */}
          <Pressable onPress={() => router.back()}
            className="absolute top-4 left-4 p-2 bg-black/30 rounded-full">
            <Ionicons name="arrow-back" size={24} color="white" />
          </Pressable>

          {/* Share Button */}
          <Pressable className="absolute top-4 right-4 p-2 bg-black/30 rounded-full">
            <Ionicons name="share-outline" size={24} color="white" />
          </Pressable>
        </View>

        {/* Product Info */}
        <View className="p-4">
          <Text className="text-xl font-bold mb-2">{product.name}</Text>

          <View className="flex-row items-center gap-2 mb-3">
            <Rating value={product.rating} />
            <Text className="text-sm text-neutral-500">({product.reviewCount} تقييم)</Text>
          </View>

          {/* Price */}
          <View className="flex-row items-baseline gap-2 mb-4">
            <Text className="text-2xl font-bold text-primary-600">
              {formatPrice(product.price)}
            </Text>
            {product.originalPrice && (
              <Text className="text-lg text-neutral-400 line-through">
                {formatPrice(product.originalPrice)}
              </Text>
            )}
          </View>

          {/* Stock Status */}
          <View className="flex-row items-center gap-2 mb-4">
            <View className={`w-2 h-2 rounded-full ${product.inStock ? 'bg-green-500' : 'bg-red-500'}`} />
            <Text className={`text-sm ${product.inStock ? 'text-green-600' : 'text-red-600'}`}>
              {product.inStock ? t('product.in_stock') : t('product.out_of_stock')}
            </Text>
          </View>

          {/* Store Info */}
          <Pressable onPress={() => router.push(`/store/${product.storeId}`)}
            className="flex-row items-center gap-3 p-3 bg-neutral-50 dark:bg-neutral-800 rounded-xl mb-4">
            <Image source={{ uri: product.storeLogo }} className="w-10 h-10 rounded-lg" />
            <View className="flex-1">
              <Text className="font-medium">{product.storeName}</Text>
              <Text className="text-sm text-neutral-500">{product.storeRating} ⭐</Text>
            </View>
            <Ionicons name="chevron-forward" size={20} color="#9ca3af" />
          </Pressable>

          {/* Description */}
          <Text className="font-semibold text-lg mb-2">{t('product.description')}</Text>
          <Text className="text-neutral-600 dark:text-neutral-400 leading-6 mb-6">
            {product.description}
          </Text>

          {/* Specifications */}
          {product.specifications && product.specifications.length > 0 && (
            <>
              <Text className="font-semibold text-lg mb-2">{t('product.specifications')}</Text>
              <View className="mb-6 bg-neutral-50 dark:bg-neutral-800 rounded-xl p-4">
                {product.specifications.map((spec: any, index: number) => (
                  <View key={index}
                    className={`flex-row justify-between py-2 ${
                      index < product.specifications.length - 1 ? 'border-b border-neutral-200 dark:border-neutral-700' : ''
                    }`}>
                    <Text className="text-neutral-500">{spec.label}</Text>
                    <Text className="font-medium">{spec.value}</Text>
                  </View>
                ))}
              </View>
            </>
          )}

          {/* Reviews */}
          <View className="flex-row items-center justify-between mb-4">
            <Text className="font-semibold text-lg">{t('product.reviews')}</Text>
            <Pressable onPress={() => setShowReviewModal(true)}>
              <Text className="text-primary-600 font-medium">اكتب تقييم</Text>
            </Pressable>
          </View>

          {product.reviews?.map((review: any) => (
            <ReviewCard key={review.id} review={review} />
          ))}

          <View className="h-32" />
        </View>
      </ScrollView>

      {/* Bottom Bar */}
      <View className="absolute bottom-0 left-0 right-0 bg-white dark:bg-neutral-900 border-t border-neutral-200 dark:border-neutral-700 p-4">
        <View className="flex-row items-center gap-4">
          <QuantitySelector value={quantity} onChange={setQuantity}
            min={1} max={product.stock || 10} />
          <Button
            className="flex-1"
            onPress={() => {
              addItem({
                id: product.id,
                name: product.name,
                price: product.price,
                image: product.images[0]?.url,
                quantity,
              });
              router.push('/cart');
            }}
            disabled={!product.inStock}
          >
            {t('product.add_to_cart')}
          </Button>
        </View>
      </View>

      {/* Review Modal */}
      <WriteReviewModal
        visible={showReviewModal}
        productId={product.id}
        onClose={() => setShowReviewModal(false)}
        onSubmit={(data) => reviewMutation.mutate(data)}
      />
    </SafeAreaView>
  );
}
```

### Cart Screen

```typescript
// apps/customer/app/(tabs)/cart.tsx
import { View, FlatList, Text, Image, Pressable } from 'react-native';
import { SafeAreaView } from 'react-native-safe-area-context';
import { useTranslation } from 'react-i18next';
import { useRouter } from 'expo-router';
import { useCartStore } from '@/stores/cartStore';
import { formatPrice } from '@/lib/utils';
import { Button } from '@/components/ui/Button';
import { QuantitySelector } from '@/components/ui/QuantitySelector';
import { Ionicons } from '@expo/vector-icons';
import Animated, { FadeInRight, Layout } from 'react-native-reanimated';

export default function CartScreen() {
  const { t } = useTranslation();
  const router = useRouter();
  const { items, total, itemCount, removeItem, updateQuantity, clearCart } = useCartStore();

  if (items.length === 0) {
    return (
      <SafeAreaView className="flex-1 items-center justify-center bg-white dark:bg-neutral-900">
        <Ionicons name="cart-outline" size={80} color="#d1d5db" />
        <Text className="text-xl font-semibold mt-4 mb-2">{t('cart.empty')}</Text>
        <Text className="text-neutral-500 mb-6">{t('cart.empty_message')}</Text>
        <Button onPress={() => router.push('/')}>{t('cart.start_shopping')}</Button>
      </SafeAreaView>
    );
  }

  return (
    <SafeAreaView className="flex-1 bg-white dark:bg-neutral-900">
      {/* Header */}
      <View className="flex-row items-center justify-between px-4 py-3 border-b border-neutral-200 dark:border-neutral-700">
        <Text className="text-lg font-bold">{t('cart.title')} ({itemCount})</Text>
        <Pressable onPress={clearCart}>
          <Text className="text-red-500 text-sm">{t('cart.clear_all')}</Text>
        </Pressable>
      </View>

      {/* Cart Items */}
      <FlatList
        data={items}
        keyExtractor={(item) => item.id}
        contentContainerStyle={{ padding: 16 }}
        renderItem={({ item }) => (
          <Animated.View
            entering={FadeInRight}
            layout={Layout}
            className="flex-row gap-3 bg-neutral-50 dark:bg-neutral-800 rounded-xl p-3 mb-3"
          >
            <Image source={{ uri: item.image }} className="w-20 h-20 rounded-lg" />
            <View className="flex-1">
              <Text className="font-medium" numberOfLines={2}>{item.name}</Text>
              <Text className="text-primary-600 font-bold mt-1">
                {formatPrice(item.price)}
              </Text>
              <View className="flex-row items-center justify-between mt-2">
                <QuantitySelector
                  value={item.quantity}
                  onChange={(qty) => updateQuantity(item.id, qty)}
                  min={1}
                  max={item.maxQuantity || 10}
                />
                <Pressable onPress={() => removeItem(item.id)}
                  className="p-2">
                  <Ionicons name="trash-outline" size={18} color="#ef4444" />
                </Pressable>
              </View>
            </View>
          </Animated.View>
        )}
      />

      {/* Summary & Checkout */}
      <View className="border-t border-neutral-200 dark:border-neutral-700 p-4 bg-white dark:bg-neutral-900">
        <View className="flex-row justify-between mb-2">
          <Text className="text-neutral-500">{t('cart.subtotal')}</Text>
          <Text className="font-medium">{formatPrice(total)}</Text>
        </View>
        <View className="flex-row justify-between mb-4">
          <Text className="text-neutral-500">{t('cart.shipping')}</Text>
          <Text className="font-medium text-green-600">{t('cart.free')}</Text>
        </View>
        <View className="flex-row justify-between mb-4 pt-2 border-t border-neutral-200 dark:border-neutral-700">
          <Text className="text-lg font-bold">{t('cart.total')}</Text>
          <Text className="text-lg font-bold text-primary-600">{formatPrice(total)}</Text>
        </View>
        <Button onPress={() => router.push('/checkout')}>
          {t('cart.checkout')}
        </Button>
      </View>
    </SafeAreaView>
  );
}
```

### Checkout Screen

```typescript
// apps/customer/app/checkout/index.tsx
import { View, Text, ScrollView, Pressable } from 'react-native';
import { SafeAreaView } from 'react-native-safe-area-context';
import { useState } from 'react';
import { useTranslation } from 'react-i18next';
import { useRouter } from 'expo-router';
import { useCartStore } from '@/stores/cartStore';
import { useCheckoutStore } from '@/stores/checkoutStore';
import { AddressStep } from '@/components/checkout/AddressStep';
import { ShippingStep } from '@/components/checkout/ShippingStep';
import { PaymentStep } from '@/components/checkout/PaymentStep';
import { ReviewStep } from '@/components/checkout/ReviewStep';
import { ConfirmationStep } from '@/components/checkout/ConfirmationStep';
import { Ionicons } from '@expo/vector-icons';

const STEPS = [
  { key: 'address', label: 'العنوان' },
  { key: 'shipping', label: 'الشحن' },
  { key: 'payment', label: 'الدفع' },
  { key: 'review', label: 'المراجعة' },
  { key: 'confirmation', label: 'التأكيد' },
];

export default function CheckoutScreen() {
  const { t } = useTranslation();
  const router = useRouter();
  const { items, total } = useCartStore();
  const { currentStep, setStep } = useCheckoutStore();

  return (
    <SafeAreaView className="flex-1 bg-white dark:bg-neutral-900">
      {/* Header */}
      <View className="flex-row items-center gap-3 px-4 py-3 border-b border-neutral-200 dark:border-neutral-700">
        <Pressable onPress={() => currentStep > 1 ? setStep(currentStep - 1) : router.back()}>
          <Ionicons name="arrow-back" size={24} />
        </Pressable>
        <Text className="text-lg font-bold">{t('checkout.title')}</Text>
      </View>

      {/* Step Indicator */}
      <View className="flex-row items-center justify-between px-4 py-3">
        {STEPS.map((step, index) => (
          <View key={step.key} className="flex-1 items-center">
            <View className={`w-8 h-8 rounded-full flex items-center justify-center ${
              index + 1 <= currentStep ? 'bg-primary-600' : 'bg-neutral-200 dark:bg-neutral-700'
            }`}>
              {index + 1 < currentStep ? (
                <Ionicons name="checkmark" size={16} color="white" />
              ) : (
                <Text className={`text-sm font-medium ${
                  index + 1 <= currentStep ? 'text-white' : 'text-neutral-500'
                }`}>{index + 1}</Text>
              )}
            </View>
            <Text className={`text-xs mt-1 ${
              index + 1 <= currentStep ? 'text-primary-600 font-medium' : 'text-neutral-500'
            }`}>{step.label}</Text>
          </View>
        ))}
      </View>

      <ScrollView className="flex-1 px-4">
        {currentStep === 1 && <AddressStep onNext={() => setStep(2)} />}
        {currentStep === 2 && <ShippingStep onNext={() => setStep(3)} onBack={() => setStep(1)} />}
        {currentStep === 3 && <PaymentStep onNext={() => setStep(4)} onBack={() => setStep(2)} />}
        {currentStep === 4 && (
          <ReviewStep
            items={items}
            total={total}
            onNext={() => setStep(5)}
            onBack={() => setStep(3)}
          />
        )}
        {currentStep === 5 && <ConfirmationStep />}
      </ScrollView>
    </SafeAreaView>
  );
}
```

### Orders Screen

```typescript
// apps/customer/app/(tabs)/orders.tsx
import { View, FlatList, Text, Pressable } from 'react-native';
import { SafeAreaView } from 'react-native-safe-area-context';
import { useQuery } from '@tanstack/react-query';
import { useTranslation } from 'react-i18next';
import { useRouter } from 'expo-router';
import { useState } from 'react';
import { fetchOrders } from '@/lib/api/orders';
import { OrderCard } from '@/components/orders/OrderCard';
import { TabSelector } from '@/components/ui/TabSelector';

export default function OrdersScreen() {
  const { t } = useTranslation();
  const router = useRouter();
  const [status, setStatus] = useState('');

  const { data, isLoading } = useQuery({
    queryKey: ['orders', { status }],
    queryFn: () => fetchOrders({ status }),
  });

  const tabs = [
    { key: '', label: t('orders.all') },
    { key: 'pending', label: t('orders.pending') },
    { key: 'processing', label: t('orders.processing') },
    { key: 'shipped', label: t('orders.shipped') },
    { key: 'delivered', label: t('orders.delivered') },
  ];

  return (
    <SafeAreaView className="flex-1 bg-white dark:bg-neutral-900">
      <View className="px-4 py-3 border-b border-neutral-200 dark:border-neutral-700">
        <Text className="text-lg font-bold">{t('orders.title')}</Text>
      </View>

      <TabSelector tabs={tabs} value={status} onChange={setStatus} />

      <FlatList
        data={data?.orders || []}
        keyExtractor={(item) => item.id}
        contentContainerStyle={{ padding: 16 }}
        renderItem={({ item }) => (
          <Pressable onPress={() => router.push(`/order/${item.id}`)}>
            <OrderCard order={item} />
          </Pressable>
        )}
        ListEmptyComponent={
          <View className="items-center justify-center py-20">
            <Text className="text-neutral-500">{t('orders.empty')}</Text>
          </View>
        }
      />
    </SafeAreaView>
  );
}
```

### Wallet Screen

```typescript
// apps/customer/app/wallet/index.tsx
import { View, Text, FlatList, Pressable } from 'react-native';
import { SafeAreaView } from 'react-native-safe-area-context';
import { useQuery } from '@tanstack/react-query';
import { useTranslation } from 'react-i18next';
import { useRouter } from 'expo-router';
import { fetchWallet } from '@/lib/api/wallet';
import { formatPrice } from '@/lib/utils';
import { Button } from '@/components/ui/Button';
import { Ionicons } from '@expo/vector-icons';

export default function WalletScreen() {
  const { t } = useTranslation();
  const router = useRouter();

  const { data: wallet } = useQuery({
    queryKey: ['wallet'],
    queryFn: fetchWallet,
  });

  return (
    <SafeAreaView className="flex-1 bg-white dark:bg-neutral-900">
      <View className="px-4 py-3 border-b border-neutral-200 dark:border-neutral-700">
        <Text className="text-lg font-bold">{t('wallet.title')}</Text>
      </View>

      {/* Balance Card */}
      <View className="m-4 bg-gradient-to-br from-primary-600 to-primary-800 rounded-2xl p-6">
        <Text className="text-white/70 mb-1">{t('wallet.balance')}</Text>
        <Text className="text-3xl font-bold text-white mb-4">
          {formatPrice(wallet?.balance || 0)}
        </Text>
        <Button variant="secondary" className="bg-white/20"
          onPress={() => router.push('/wallet/topup')}>
          <Ionicons name="add-circle-outline" size={20} color="white" />
          <Text className="text-white ml-2">{t('wallet.topup')}</Text>
        </Button>
      </View>

      {/* Transactions */}
      <View className="px-4 mb-2">
        <Text className="font-semibold">{t('wallet.transactions')}</Text>
      </View>

      <FlatList
        data={wallet?.transactions || []}
        keyExtractor={(item) => item.id}
        contentContainerStyle={{ paddingHorizontal: 16 }}
        renderItem={({ item }) => (
          <View className="flex-row items-center justify-between py-3 border-b border-neutral-100 dark:border-neutral-800">
            <View className="flex-row items-center gap-3">
              <View className={`w-10 h-10 rounded-full flex items-center justify-center ${
                item.type === 'credit' ? 'bg-green-100' : 'bg-red-100'
              }`}>
                <Ionicons
                  name={item.type === 'credit' ? 'arrow-down' : 'arrow-up'}
                  size={18}
                  color={item.type === 'credit' ? '#22c55e' : '#ef4444'}
                />
              </View>
              <View>
                <Text className="font-medium">{item.description}</Text>
                <Text className="text-xs text-neutral-500">
                  {new Date(item.createdAt).toLocaleDateString('ar-YE')}
                </Text>
              </View>
            </View>
            <Text className={`font-bold ${
              item.type === 'credit' ? 'text-green-600' : 'text-red-600'
            }`}>
              {item.type === 'credit' ? '+' : '-'}{formatPrice(item.amount)}
            </Text>
          </View>
        )}
      />
    </SafeAreaView>
  );
}
```

### Reviews Screen

```typescript
// apps/customer/app/reviews/[productId].tsx
import { View, Text, FlatList } from 'react-native';
import { useLocalSearchParams } from 'expo-router';
import { SafeAreaView } from 'react-native-safe-area-context';
import { useQuery } from '@tanstack/react-query';
import { useTranslation } from 'react-i18next';
import { fetchProductReviews } from '@/lib/api/products';
import { ReviewCard } from '@/components/product/ReviewCard';
import { Rating } from '@/components/ui/Rating';

export default function ReviewsScreen() {
  const { productId } = useLocalSearchParams<{ productId: string }>();
  const { t } = useTranslation();

  const { data, isLoading } = useQuery({
    queryKey: ['product-reviews', productId],
    queryFn: () => fetchProductReviews(productId!),
  });

  return (
    <SafeAreaView className="flex-1 bg-white dark:bg-neutral-900">
      <View className="px-4 py-3 border-b border-neutral-200 dark:border-neutral-700">
        <Text className="text-lg font-bold">{t('reviews.title')}</Text>
      </View>

      {/* Rating Summary */}
      {data?.summary && (
        <View className="flex-row items-center gap-4 px-4 py-4 border-b border-neutral-200 dark:border-neutral-700">
          <Text className="text-4xl font-bold">{data.summary.average}</Text>
          <View>
            <Rating value={data.summary.average} size={20} />
            <Text className="text-sm text-neutral-500 mt-1">
              {data.summary.total} تقييم
            </Text>
          </View>
        </View>
      )}

      <FlatList
        data={data?.reviews || []}
        keyExtractor={(item) => item.id}
        contentContainerStyle={{ padding: 16 }}
        renderItem={({ item }) => <ReviewCard review={item} />}
      />
    </SafeAreaView>
  );
}
```

## Vendor App

### Vendor Dashboard

```typescript
// apps/vendor/app/(tabs)/index.tsx
import { View, Text, ScrollView, RefreshControl } from 'react-native';
import { SafeAreaView } from 'react-native-safe-area-context';
import { useQuery } from '@tanstack/react-query';
import { useTranslation } from 'react-i18next';
import { useState, useCallback } from 'react';
import { fetchVendorDashboard } from '@/lib/api/vendor';
import { StatsCard } from '@/components/dashboard/StatsCard';
import { formatPrice } from '@/lib/utils';
import { Ionicons } from '@expo/vector-icons';

export default function VendorDashboard() {
  const { t } = useTranslation();
  const [refreshing, setRefreshing] = useState(false);

  const { data, isLoading, refetch } = useQuery({
    queryKey: ['vendor-dashboard'],
    queryFn: fetchVendorDashboard,
    refetchInterval: 30000,
  });

  const onRefresh = useCallback(async () => {
    setRefreshing(true);
    await refetch();
    setRefreshing(false);
  }, [refetch]);

  return (
    <SafeAreaView className="flex-1 bg-neutral-50 dark:bg-neutral-900">
      <ScrollView
        refreshControl={<RefreshControl refreshing={refreshing} onRefresh={onRefresh} />}
        showsVerticalScrollIndicator={false}
      >
        {/* Header */}
        <View className="px-4 py-4">
          <Text className="text-2xl font-bold">{t('vendor.dashboard')}</Text>
          <Text className="text-neutral-500">{t('vendor.welcome')}</Text>
        </View>

        {/* Stats Grid */}
        <View className="flex-row flex-wrap px-4 gap-3">
          <StatsCard
            title={t('vendor.today_sales')}
            value={formatPrice(data?.todaySales || 0)}
            icon="cash"
            color="green"
          />
          <StatsCard
            title={t('vendor.new_orders')}
            value={data?.newOrders || 0}
            icon="cart"
            color="blue"
          />
          <StatsCard
            title={t('vendor.products')}
            value={data?.activeProducts || 0}
            icon="package"
            color="purple"
          />
          <StatsCard
            title={t('vendor.rating')}
            value={data?.avgRating || 0}
            icon="star"
            color="yellow"
          />
        </View>

        {/* Recent Orders */}
        <View className="px-4 mt-6">
          <View className="flex-row items-center justify-between mb-3">
            <Text className="font-semibold text-lg">{t('vendor.recent_orders')}</Text>
            <Text className="text-primary-600">{t('vendor.view_all')}</Text>
          </View>
          {data?.recentOrders?.map((order: any) => (
            <View key={order.id}
              className="bg-white dark:bg-neutral-800 rounded-xl p-4 mb-3 shadow-sm">
              <View className="flex-row items-center justify-between">
                <View>
                  <Text className="font-medium">#{order.orderNumber}</Text>
                  <Text className="text-sm text-neutral-500">{order.customerName}</Text>
                </View>
                <View className="items-end">
                  <Text className="font-bold">{formatPrice(order.total)}</Text>
                  <Text className="text-xs text-neutral-500">{order.itemCount} منتجات</Text>
                </View>
              </View>
            </View>
          ))}
        </View>

        <View className="h-20" />
      </ScrollView>
    </SafeAreaView>
  );
}
```

### Vendor Products Screen

```typescript
// apps/vendor/app/(tabs)/products.tsx
import { View, Text, FlatList, Pressable, Image } from 'react-native';
import { SafeAreaView } from 'react-native-safe-area-context';
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { useTranslation } from 'react-i18next';
import { useRouter } from 'expo-router';
import { useState } from 'react';
import { fetchVendorProducts, deleteProduct } from '@/lib/api/vendor/products';
import { formatPrice } from '@/lib/utils';
import { Button } from '@/components/ui/Button';
import { Badge } from '@/components/ui/Badge';
import { SearchBar } from '@/components/ui/SearchBar';
import { Ionicons } from '@expo/vector-icons';
import { Alert } from 'react-native';

export default function VendorProducts() {
  const { t } = useTranslation();
  const router = useRouter();
  const queryClient = useQueryClient();
  const [search, setSearch] = useState('');

  const { data, isLoading } = useQuery({
    queryKey: ['vendor-products', { search }],
    queryFn: () => fetchVendorProducts({ search }),
  });

  const deleteMutation = useMutation({
    mutationFn: deleteProduct,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['vendor-products'] });
    },
  });

  return (
    <SafeAreaView className="flex-1 bg-neutral-50 dark:bg-neutral-900">
      <View className="flex-row items-center justify-between px-4 py-3">
        <Text className="text-lg font-bold">{t('vendor.products')}</Text>
        <Button onPress={() => router.push('/product/new')} size="sm">
          <Ionicons name="add" size={18} color="white" />
          <Text className="text-white ml-1">{t('vendor.add_product')}</Text>
        </Button>
      </View>

      <SearchBar value={search} onChange={setSearch} placeholder={t('vendor.search_products')} />

      <FlatList
        data={data?.products || []}
        keyExtractor={(item) => item.id}
        contentContainerStyle={{ padding: 16 }}
        renderItem={({ item }) => (
          <View className="flex-row bg-white dark:bg-neutral-800 rounded-xl p-3 mb-3 shadow-sm">
            <Image source={{ uri: item.images[0]?.url }} className="w-16 h-16 rounded-lg" />
            <View className="flex-1 ml-3">
              <Text className="font-medium" numberOfLines={1}>{item.name}</Text>
              <Text className="text-primary-600 font-bold mt-1">{formatPrice(item.price)}</Text>
              <View className="flex-row items-center gap-2 mt-1">
                <Badge variant={item.stock > 0 ? 'success' : 'error'}>
                  {t('vendor.stock')}: {item.stock}
                </Badge>
                <Badge variant={item.isActive ? 'success' : 'neutral'}>
                  {item.isActive ? t('vendor.active') : t('vendor.inactive')}
                </Badge>
              </View>
            </View>
            <View className="justify-between items-end">
              <Pressable onPress={() => router.push(`/product/${item.id}`)}
                className="p-2">
                <Ionicons name="create-outline" size={20} color="#6b7280" />
              </Pressable>
              <Pressable onPress={() => {
                Alert.alert('حذف المنتج', 'هل أنت متأكد؟', [
                  { text: 'إلغاء', style: 'cancel' },
                  { text: 'حذف', style: 'destructive',
                    onPress: () => deleteMutation.mutate(item.id) },
                ]);
              }} className="p-2">
                <Ionicons name="trash-outline" size={20} color="#ef4444" />
              </Pressable>
            </View>
          </View>
        )}
      />
    </SafeAreaView>
  );
}
```

### Vendor Orders Screen

```typescript
// apps/vendor/app/(tabs)/orders.tsx
import { View, Text, FlatList, Pressable } from 'react-native';
import { SafeAreaView } from 'react-native-safe-area-context';
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { useTranslation } from 'react-i18next';
import { useRouter } from 'expo-router';
import { useState } from 'react';
import { fetchVendorOrders, updateVendorOrderStatus } from '@/lib/api/vendor/orders';
import { formatPrice } from '@/lib/utils';
import { Badge } from '@/components/ui/Badge';
import { Ionicons } from '@expo/vector-icons';
import { Alert } from 'react-native';

type OrderStatus = 'pending' | 'confirmed' | 'processing' | 'shipped' | 'delivered';

export default function VendorOrders() {
  const { t } = useTranslation();
  const router = useRouter();
  const queryClient = useQueryClient();
  const [status, setStatus] = useState<OrderStatus | ''>('');

  const { data, isLoading } = useQuery({
    queryKey: ['vendor-orders', { status }],
    queryFn: () => fetchVendorOrders({ status: status as string }),
    refetchInterval: 15000,
  });

  const updateStatusMutation = useMutation({
    mutationFn: ({ orderId, newStatus }: { orderId: string; newStatus: OrderStatus }) =>
      updateVendorOrderStatus(orderId, newStatus),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['vendor-orders'] });
    },
  });

  const getNextStatus = (current: OrderStatus): OrderStatus | null => {
    const flow: Record<string, OrderStatus> = {
      pending: 'confirmed',
      confirmed: 'processing',
      processing: 'shipped',
      shipped: 'delivered',
    };
    return flow[current] || null;
  };

  const statusLabel: Record<string, string> = {
    pending: 'قيد الانتظار',
    confirmed: 'مؤكد',
    processing: 'قيد التجهيز',
    shipped: 'تم الشحن',
    delivered: 'تم التوصيل',
  };

  const statusVariant: Record<string, string> = {
    pending: 'warning',
    confirmed: 'info',
    processing: 'info',
    shipped: 'primary',
    delivered: 'success',
  };

  return (
    <SafeAreaView className="flex-1 bg-neutral-50 dark:bg-neutral-900">
      <View className="px-4 py-3">
        <Text className="text-lg font-bold">{t('vendor.orders')}</Text>
      </View>

      {/* Status Filter */}
      <View className="flex-row px-4 gap-2 mb-4">
        {['', 'pending', 'confirmed', 'processing', 'shipped', 'delivered'].map((s) => (
          <Pressable key={s} onPress={() => setStatus(s as OrderStatus | '')}
            className={`px-3 py-1.5 rounded-full ${
              status === s ? 'bg-primary-600' : 'bg-neutral-200 dark:bg-neutral-700'
            }`}>
            <Text className={`text-xs font-medium ${
              status === s ? 'text-white' : 'text-neutral-600'
            }`}>
              {s ? statusLabel[s] : 'الكل'}
            </Text>
          </Pressable>
        ))}
      </View>

      <FlatList
        data={data?.orders || []}
        keyExtractor={(item) => item.id}
        contentContainerStyle={{ paddingHorizontal: 16 }}
        renderItem={({ item }) => {
          const next = getNextStatus(item.status);
          return (
            <Pressable onPress={() => router.push(`/order/${item.id}`)}
              className="bg-white dark:bg-neutral-800 rounded-xl p-4 mb-3 shadow-sm">
              <View className="flex-row items-center justify-between mb-2">
                <Text className="font-medium">#{item.orderNumber}</Text>
                <Badge variant={statusVariant[item.status]}>{statusLabel[item.status]}</Badge>
              </View>
              <View className="flex-row items-center justify-between">
                <View>
                  <Text className="text-sm text-neutral-500">{item.customerName}</Text>
                  <Text className="text-sm text-neutral-500">{item.itemCount} منتجات</Text>
                </View>
                <Text className="font-bold">{formatPrice(item.total)}</Text>
              </View>
              {next && (
                <Pressable onPress={() => {
                  Alert.alert('تحديث الحالة', `تأكيد ${statusLabel[next]}؟`, [
                    { text: 'إلغاء', style: 'cancel' },
                    { text: 'تأكيد', onPress: () =>
                      updateStatusMutation.mutate({ orderId: item.id, newStatus: next }) },
                  ]);
                }}
                  className="mt-3 bg-primary-50 dark:bg-primary-900/20 rounded-lg py-2 items-center">
                  <Text className="text-primary-600 font-medium text-sm">
                    تحديث إلى: {statusLabel[next]}
                  </Text>
                </Pressable>
              )}
            </Pressable>
          );
        }}
      />
    </SafeAreaView>
  );
}
```

## Push Notifications

```typescript
// packages/shared/notifications/setup.ts
import * as Notifications from 'expo-notifications';
import * as Device from 'expo-device';
import { Platform } from 'react-native';
import { apiClient } from '@/lib/api/client';

Notifications.setNotificationHandler({
  handleNotification: async () => ({
    shouldShowAlert: true,
    shouldPlaySound: true,
    shouldSetBadge: true,
  }),
});

export async function registerForPushNotifications(userId: string, userType: 'customer' | 'vendor') {
  if (!Device.isDevice) {
    console.log('Push notifications require a physical device');
    return null;
  }

  const { status: existingStatus } = await Notifications.getPermissionsAsync();
  let finalStatus = existingStatus;

  if (existingStatus !== 'granted') {
    const { status } = await Notifications.requestPermissionsAsync();
    finalStatus = status;
  }

  if (finalStatus !== 'granted') {
    console.log('Push notification permission not granted');
    return null;
  }

  const token = await Notifications.getExpoPushTokenAsync({
    projectId: 'your-project-id',
  });

  if (Platform.OS === 'android') {
    Notifications.setNotificationChannelAsync('default', {
      name: 'default',
      importance: Notifications.AndroidImportance.MAX,
      vibrationPattern: [0, 250, 250, 250],
      lightColor: '#2563eb',
    });

    Notifications.setNotificationChannelAsync('orders', {
      name: 'الطلبات',
      importance: Notifications.AndroidImportance.HIGH,
      vibrationPattern: [0, 250, 250, 250],
    });
  }

  // Send token to backend
  await apiClient.post('/notifications/register', {
    token: token.data,
    userId,
    userType,
    platform: Platform.OS,
  });

  return token.data;
}

export function setupNotificationListeners(
  onNotificationReceived: (notification: Notifications.Notification) => void,
  onNotificationTapped: (response: Notifications.NotificationResponse) => void
) {
  const receivedSubscription = Notifications.addNotificationReceivedListener(
    onNotificationReceived
  );

  const responseSubscription = Notifications.addNotificationResponseReceivedListener(
    onNotificationTapped
  );

  return () => {
    receivedSubscription.remove();
    responseSubscription.remove();
  };
}
```

## Biometric Authentication (Future)

```typescript
// packages/shared/auth/biometrics.ts
import * as LocalAuthentication from 'expo-local-authentication';
import * as SecureStore from 'expo-secure-store';

export async function isBiometricsAvailable(): Promise<boolean> {
  const compatible = await LocalAuthentication.hasHardwareAsync();
  const enrolled = await LocalAuthentication.isEnrolledAsync();
  return compatible && enrolled;
}

export async function authenticateWithBiometrics(): Promise<boolean> {
  const result = await LocalAuthentication.authenticateAsync({
    promptMessage: 'تأكيد الهوية',
    cancelLabel: 'إلغاء',
    disableDeviceFallback: false,
    fallbackLabel: 'استخدام كلمة المرور',
  });
  return result.success;
}

export async function saveBiometricCredentials(userId: string, token: string): Promise<void> {
  await SecureStore.setItemAsync(`biometric_${userId}`, token, {
    keychainAccessible: SecureStore.WHEN_UNLOCKED_THIS_DEVICE_ONLY,
  });
}

export async function getBiometricToken(userId: string): Promise<string | null> {
  return await SecureStore.getItemAsync(`biometric_${userId}`);
}

export async function removeBiometricCredentials(userId: string): Promise<void> {
  await SecureStore.deleteItemAsync(`biometric_${userId}`);
}
```

## Offline Support

```typescript
// packages/shared/offline/useOfflineStatus.ts
import { useEffect, useState } from 'react';
import NetInfo from '@react-native-community/netinfo';

export function useOfflineStatus() {
  const [isOffline, setIsOffline] = useState(false);

  useEffect(() => {
    const unsubscribe = NetInfo.addEventListener((state) => {
      setIsOffline(!state.isConnected);
    });

    return () => unsubscribe();
  }, []);

  return { isOffline };
}
```

```typescript
// packages/shared/offline/cacheProducts.ts
import AsyncStorage from '@react-native-async-storage/async-storage';

const CACHE_KEY = 'cached_products';
const CACHE_EXPIRY = 30 * 60 * 1000; // 30 minutes

export async function cacheProducts(products: any[]) {
  const data = {
    products,
    timestamp: Date.now(),
  };
  await AsyncStorage.setItem(CACHE_KEY, JSON.stringify(data));
}

export async function getCachedProducts(): Promise<any[] | null> {
  const raw = await AsyncStorage.getItem(CACHE_KEY);
  if (!raw) return null;

  const { products, timestamp } = JSON.parse(raw);
  if (Date.now() - timestamp > CACHE_EXPIRY) {
    await AsyncStorage.removeItem(CACHE_KEY);
    return null;
  }

  return products;
}

export async function clearProductCache() {
  await AsyncStorage.removeItem(CACHE_KEY);
}
```

```typescript
// packages/shared/hooks/useCachedQuery.ts
import { useQuery } from '@tanstack/react-query';
import { useOfflineStatus } from './useOfflineStatus';
import { getCachedProducts, cacheProducts } from './offline/cacheProducts';

export function useCachedQuery<T>(
  queryKey: string[],
  queryFn: () => Promise<T>,
  cacheFn?: (data: T) => Promise<void>,
  getCachedFn?: () => Promise<T | null>
) {
  const { isOffline } = useOfflineStatus();

  return useQuery({
    queryKey,
    queryFn: async () => {
      const data = await queryFn();
      if (cacheFn) await cacheFn(data);
      return data;
    },
    enabled: !isOffline,
    placeholderData: () => {
      // Return cached data as placeholder when offline
      return getCachedFn ? undefined : undefined;
    },
    staleTime: isOffline ? Infinity : 5 * 60 * 1000,
  });
}
```

## API Client

```typescript
// packages/api/client.ts
import * as SecureStore from 'expo-secure-store';
import { Platform } from 'react-native';

const BASE_URL = Platform.select({
  android: 'http://10.0.2.2:3000/api', // Android emulator
  ios: 'http://localhost:3000/api',     // iOS simulator
  default: 'https://api.yemenmart.com/api', // Production
});

interface RequestOptions {
  method?: string;
  body?: any;
  headers?: Record<string, string>;
}

export async function apiRequest<T>(
  endpoint: string,
  options: RequestOptions = {}
): Promise<T> {
  const token = await SecureStore.getItemAsync('auth_token');

  const headers: Record<string, string> = {
    'Content-Type': 'application/json',
    ...options.headers,
  };

  if (token) {
    headers['Authorization'] = `Bearer ${token}`;
  }

  const response = await fetch(`${BASE_URL}${endpoint}`, {
    method: options.method || 'GET',
    headers,
    body: options.body ? JSON.stringify(options.body) : undefined,
  });

  if (!response.ok) {
    const error = await response.json().catch(() => ({}));
    throw new Error(error.message || `API Error: ${response.status}`);
  }

  return response.json();
}

export const apiClient = {
  get: <T>(endpoint: string) => apiRequest<T>(endpoint),
  post: <T>(endpoint: string, body: any) =>
    apiRequest<T>(endpoint, { method: 'POST', body }),
  put: <T>(endpoint: string, body: any) =>
    apiRequest<T>(endpoint, { method: 'PUT', body }),
  delete: <T>(endpoint: string) =>
    apiRequest<T>(endpoint, { method: 'DELETE' }),
};
```

## Configuration

```json
// apps/customer/app.json
{
  "expo": {
    "name": "YemenMart",
    "slug": "yemenmart-customer",
    "version": "1.0.0",
    "orientation": "portrait",
    "icon": "./assets/icon.png",
    "scheme": "yemenmart",
    "userInterfaceStyle": "automatic",
    "splash": {
      "image": "./assets/splash.png",
      "resizeMode": "contain",
      "backgroundColor": "#2563eb"
    },
    "assetBundlePatterns": ["**/*"],
    "ios": {
      "supportsTablet": true,
      "bundleIdentifier": "com.yemenmart.customer"
    },
    "android": {
      "adaptiveIcon": {
        "foregroundImage": "./assets/adaptive-icon.png",
        "backgroundColor": "#2563eb"
      },
      "package": "com.yemenmart.customer"
    },
    "plugins": [
      "expo-router",
      "expo-secure-store",
      "expo-notifications",
      [
        "expo-local-authentication",
        {
          "faceIDPermission": "السماح لـ YemenMart باستخدام Face ID لتأكيد هويتك"
        }
      ]
    ]
  }
}
```

```json
// apps/vendor/app.json
{
  "expo": {
    "name": "YemenMart Vendor",
    "slug": "yemenmart-vendor",
    "version": "1.0.0",
    "orientation": "portrait",
    "icon": "./assets/icon.png",
    "scheme": "yemenmart-vendor",
    "userInterfaceStyle": "automatic",
    "splash": {
      "image": "./assets/splash.png",
      "resizeMode": "contain",
      "backgroundColor": "#2563eb"
    },
    "ios": {
      "bundleIdentifier": "com.yemenmart.vendor"
    },
    "android": {
      "package": "com.yemenmart.vendor"
    },
    "plugins": [
      "expo-router",
      "expo-secure-store",
      "expo-notifications"
    ]
  }
}
```
