# 09 - DATA FLOW MAPPING

## YemenMart Platform — Comprehensive Data Flow Documentation

| Field | Value |
|-------|-------|
| Document ID | YM-DF-00 |
| Version | 1.0 |
| Last Updated | 2026-09-13 |
| Scope | All major pages and their data flows |
| Data Flow ID Format | `DF-{PAGE}-{SEQUENCE}` |

---

## Table of Contents

1. [Homepage Data Flows](#1-homepage-data-flows)
2. [Product Search & Filtering](#2-product-search--filtering)
3. [Product Detail](#3-product-detail)
4. [Cart Operations](#4-cart-operations)
5. [Checkout Flow](#5-checkout-flow)
6. [Order History & Tracking](#6-order-history--tracking)
7. [Wallet Operations](#7-wallet-operations)
8. [Vendor Dashboard](#8-vendor-dashboard)
9. [Vendor Product Management](#9-vendor-product-management)
10. [Vendor Order Management](#10-vendor-order-management)
11. [Admin Dashboard](#11-admin-dashboard)
12. [Admin User/Vendor/Order Management](#12-admin-uservendororder-management)
13. [Authentication Flow](#13-authentication-flow)

---

## 1. Homepage Data Flows

### DF-HOME-001 — Featured Products Loading

| Field | Value |
|-------|-------|
| Data Flow ID | `DF-HOME-001` |
| Page | Homepage (`/`) |
| Widget | `FeaturedProductsCarousel` |
| Trigger | Page mount (initial load) |
| Request | `GET /api/v1/products?featured=true&limit=10&lang=ar` |
| Response | `{ products: Product[], total: number }` |
| Transformation | Map API `Product[]` → `FeaturedProductCard[]`; extract `primary_image.url`, `price_formatted`, `rating` |
| Loading Strategy | Initial load with skeleton placeholders |
| Cache Config | `staleTime: 5min`, `cacheTime: 30min` |
| Error Handling | Fallback to empty carousel with retry button; log to Sentry |
| Database Entity | `products` WHERE `is_featured = true` JOIN `product_images` |
| Security | Public — no auth required |

### DF-HOME-002 — Categories Grid Loading

| Field | Value |
|-------|-------|
| Data Flow ID | `DF-HOME-002` |
| Page | Homepage (`/`) |
| Widget | `CategoriesGrid` |
| Trigger | Page mount (initial load) |
| Request | `GET /api/v1/categories?include_counts=true&lang=ar` |
| Response | `{ categories: Category[] }` |
| Transformation | Build tree structure from flat array using `parent_id`; compute depth |
| Loading Strategy | Initial load with skeleton grid |
| Cache Config | `staleTime: 15min`, `cacheTime: 60min` |
| Error Handling | Show generic "Categories unavailable" banner; retry on refetch |
| Database Entity | `categories` WHERE `is_active = true` |
| Security | Public |

### DF-HOME-003 — Active Promotions / Banners

| Field | Value |
|-------|-------|
| Data Flow ID | `DF-HOME-003` |
| Page | Homepage (`/`) |
| Widget | `PromotionsBanner` / `HeroSlider` |
| Trigger | Page mount (initial load) |
| Request | `GET /api/v1/promotions?active=true&placement=homepage&lang=ar` |
| Response | `{ promotions: Promotion[] }` |
| Transformation | Sort by `priority` descending; map to `BannerSlide[]` with `image_url`, `link_url`, `cta_text` |
| Loading Strategy | Initial load with fade-in animation |
| Cache Config | `staleTime: 10min`, `cacheTime: 30min` |
| Error Handling | Hide banner section entirely on failure |
| Database Entity | `promotions` WHERE `is_active = true AND start_date <= NOW() AND end_date >= NOW()` |
| Security | Public |

### DF-HOME-004 — Flash Sale / Timer Data

| Field | Value |
|-------|-------|
| Data Flow ID | `DF-HOME-004` |
| Page | Homepage (`/`) |
| Widget | `FlashSaleSection` |
| Trigger | Page mount (initial load) |
| Request | `GET /api/v1/flash-sales?active=true&lang=ar` |
| Response | `{ flash_sale: FlashSale, products: FlashSaleProduct[] }` |
| Transformation | Compute remaining seconds from `end_date`; map products with `discount_percentage` |
| Loading Strategy | Initial load; client-side countdown timer (no polling) |
| Cache Config | `staleTime: 1min`, `cacheTime: 5min` |
| Error Handling | Hide flash sale section on failure |
| Database Entity | `flash_sales` JOIN `flash_sale_products` JOIN `products` |
| Security | Public |

---

## 2. Product Search & Filtering

### DF-SEARCH-001 — Search Query Execution

| Field | Value |
|-------|-------|
| Data Flow ID | `DF-SEARCH-001` |
| Page | Search Results (`/search?q={query}`) |
| Widget | `SearchResultsList` |
| Trigger | URL search param `q` change; debounced input (300ms) |
| Request | `GET /api/v1/products?search={query}&page={page}&per_page=20&lang=ar` |
| Response | `{ products: Product[], total: number, page: number, per_page: number }` |
| Transformation | Map to `SearchResultCard[]`; highlight matching terms in `name` |
| Loading Strategy | Debounced search; infinite scroll pagination |
| Cache Config | `staleTime: 2min`, `cacheTime: 10min`; query-key includes `q`, `page`, filters |
| Error Handling | Show "No results found" with suggestions; retry button on network error |
| Database Entity | `products` via Elasticsearch full-text index |
| Security | Public |

### DF-SEARCH-002 — Category Filter

| Field | Value |
|-------|-------|
| Data Flow ID | `DF-SEARCH-002` |
| Page | Search Results (`/search`) |
| Widget | `CategoryFilter` (sidebar) |
| Trigger | Category checkbox click |
| Request | `GET /api/v1/products?category_ids={ids}&page=1&per_page=20&lang=ar` |
| Response | `{ products: Product[], total: number }` |
| Transformation | Rebuild product list; update URL params without navigation |
| Loading Strategy | Optimistic — immediately apply filter; invalidate stale cache |
| Cache Config | `staleTime: 2min`, `cacheTime: 10min` |
| Error Handling | Revert to previous filter state on failure |
| Database Entity | `products` JOIN `product_categories` |
| Security | Public |

### DF-SEARCH-003 — Price Range Filter

| Field | Value |
|-------|-------|
| Data Flow ID | `DF-SEARCH-003` |
| Page | Search Results (`/search`) |
| Widget | `PriceRangeFilter` (slider) |
| Trigger | Price range slider change (debounced 500ms) |
| Request | `GET /api/v1/products?min_price={min}&max_price={max}&page=1&per_page=20&lang=ar` |
| Response | `{ products: Product[], total: number }` |
| Transformation | Clamp price values; format price display |
| Loading Strategy | Debounced; URL param sync |
| Cache Config | `staleTime: 2min`, `cacheTime: 10min` |
| Error Handling | Snap slider to last valid position |
| Database Entity | `products` WHERE `price BETWEEN min AND max` |
| Security | Public |

### DF-SEARCH-004 — Sort Change

| Field | Value |
|-------|-------|
| Data Flow ID | `DF-SEARCH-004` |
| Page | Search Results (`/search`) |
| Widget | `SortDropdown` |
| Trigger | Sort option selection |
| Request | `GET /api/v1/products?sort={sort_by}&order={order}&page=1&per_page=20&lang=ar` |
| Response | `{ products: Product[], total: number }` |
| Transformation | Re-render list with new order |
| Loading Strategy | Immediate; no debounce needed |
| Cache Config | `staleTime: 2min`, `cacheTime: 10min` |
| Error Handling | Revert sort to previous value |
| Database Entity | `products` ORDER BY `{sort_by} {order}` |
| Security | Public |

---

## 3. Product Detail

### DF-PRODUCT-001 — Product Data Loading

| Field | Value |
|-------|-------|
| Data Flow ID | `DF-PRODUCT-001` |
| Page | Product Detail (`/products/[id]`) |
| Widget | `ProductDetailPage` (root) |
| Trigger | Route navigation to `/products/{id}` |
| Request | `GET /api/v1/products/{id}?lang=ar` |
| Response | `{ product: ProductDetail }` |
| Transformation | Map to page-level data model; extract `images[]`, `description` (HTML sanitization), `variants[]` |
| Loading Strategy | Initial load with full-page skeleton |
| Cache Config | `staleTime: 3min`, `cacheTime: 15min` |
| Error Handling | 404 → redirect to not-found page; 500 → retry with backoff |
| Database Entity | `products` JOIN `product_images`, `vendors`, `categories` |
| Security | Public |

### DF-PRODUCT-002 — Product Variants

| Field | Value |
|-------|-------|
| Data Flow ID | `DF-PRODUCT-002` |
| Page | Product Detail (`/products/[id]`) |
| Widget | `VariantSelector` |
| Trigger | Parent product loaded (DF-PRODUCT-001) |
| Request | `GET /api/v1/products/{id}/variants?lang=ar` |
| Response | `{ variants: Variant[] }` |
| Transformation | Group by `attribute_type` (color, size); build option matrix |
| Loading Strategy | Sequential — load after product data; skeleton for variant options |
| Cache Config | `staleTime: 3min`, `cacheTime: 15min` |
| Error Handling | Show "Variants unavailable" with basic product info |
| Database Entity | `product_variants` JOIN `variant_attributes` |
| Security | Public |

### DF-PRODUCT-003 — Product Reviews

| Field | Value |
|-------|-------|
| Data Flow ID | `DF-PRODUCT-003` |
| Page | Product Detail (`/products/[id]`) |
| Widget | `ReviewsSection` |
| Trigger | Scroll into viewport (Intersection Observer) |
| Request | `GET /api/v1/products/{id}/reviews?page=1&per_page=5&lang=ar` |
| Response | `{ reviews: Review[], average_rating: number, total_reviews: number }` |
| Transformation | Map to `ReviewCard[]`; compute rating distribution histogram |
| Loading Strategy | Lazy load on scroll; infinite scroll for pagination |
| Cache Config | `staleTime: 5min`, `cacheTime: 30min` |
| Error Handling | Show "Reviews unavailable" with product rating display |
| Database Entity | `reviews` JOIN `customers` WHERE `product_id = {id}` |
| Security | Public |

### DF-PRODUCT-004 — Related Products

| Field | Value |
|-------|-------|
| Data Flow ID | `DF-PRODUCT-004` |
| Page | Product Detail (`/products/[id]`) |
| Widget | `RelatedProductsCarousel` |
| Trigger | Scroll into viewport |
| Request | `GET /api/v1/products/{id}/related?limit=8&lang=ar` |
| Response | `{ products: Product[] }` |
| Transformation | Map to `ProductCard[]` |
| Loading Strategy | Lazy load; prefetch next 4 on hover |
| Cache Config | `staleTime: 10min`, `cacheTime: 60min` |
| Error Handling | Hide section on failure |
| Database Entity | `products` WHERE `category_id` matches; fallback to tag-based similarity |
| Security | Public |

---

## 4. Cart Operations

### DF-CART-001 — Cart Load (Authenticated)

| Field | Value |
|-------|-------|
| Data Flow ID | `DF-CART-001` |
| Page | Any (persistent cart) |
| Widget | `CartSidebar` / `CartPage` |
| Trigger | App mount (if authenticated); auth state change |
| Request | `GET /api/v1/cart` |
| Response | `{ cart: Cart, items: CartItem[], total: number, item_count: number }` |
| Transformation | Compute `subtotal`, `discount`, `total`; map items to `CartItemComponent[]` |
| Loading Strategy | Initial load on auth; background refresh every 5min |
| Cache Config | `staleTime: 0` (always fresh), `cacheTime: 5min` |
| Error Handling | Show cached cart from Zustand if network fails; sync on reconnect |
| Database Entity | `carts` JOIN `cart_items` JOIN `products` JOIN `product_variants` |
| Security | Authenticated — Bearer token required |

### DF-CART-002 — Add to Cart

| Field | Value |
|-------|-------|
| Data Flow ID | `DF-CART-002` |
| Page | Product Detail / Product List |
| Widget | `AddToCartButton` |
| Trigger | Button click |
| Request | `POST /api/v1/cart` with `{ product_id, variant_id?, quantity }` |
| Response | `{ cart: Cart, item: CartItem, item_count: number }` |
| Transformation | Optimistic update: immediately increment cart badge count; append item to local state |
| Loading Strategy | Optimistic update; rollback on error |
| Cache Config | Invalidate `cart` query; refetch on success |
| Error Handling | Rollback optimistic update; show toast "Failed to add to cart" with retry |
| Database Entity | `carts` INSERT/UPDATE `cart_items` |
| Security | Authenticated |

### DF-CART-003 — Update Cart Item Quantity

| Field | Value |
|-------|-------|
| Data Flow ID | `DF-CART-003` |
| Page | Cart Page / Cart Sidebar |
| Widget | `QuantityStepper` |
| Trigger | +/- button click or manual input |
| Request | `PUT /api/v1/cart` with `{ item_id, quantity }` |
| Response | `{ cart: Cart, item: CartItem, total: number }` |
| Transformation | Optimistic: update local quantity and recalculate subtotal |
| Loading Strategy | Optimistic update; debounce rapid clicks (300ms) |
| Cache Config | Invalidate `cart` query |
| Error Handling | Revert to previous quantity; show error toast |
| Database Entity | `cart_items` UPDATE `quantity` |
| Security | Authenticated |

### DF-CART-004 — Remove Cart Item

| Field | Value |
|-------|-------|
| Data Flow ID | `DF-CART-004` |
| Page | Cart Page / Cart Sidebar |
| Widget | `RemoveItemButton` |
| Trigger | Delete icon click + confirmation |
| Request | `DELETE /api/v1/cart` with `{ item_id }` |
| Response | `{ cart: Cart, total: number, item_count: number }` |
| Transformation | Optimistic: remove item from local state; recalculate totals |
| Loading Strategy | Optimistic with confirmation dialog |
| Cache Config | Invalidate `cart` query |
| Error Handling | Restore item on failure; show error toast |
| Database Entity | `cart_items` DELETE |
| Security | Authenticated |

### DF-CART-005 — Cart Sync (Guest → Authenticated)

| Field | Value |
|-------|-------|
| Data Flow ID | `DF-CART-005` |
| Page | Login / Register |
| Widget | `AuthForm` (post-auth hook) |
| Trigger | Successful authentication |
| Request | `POST /api/v1/cart/sync` with `{ guest_cart_items: CartItem[] }` |
| Response | `{ cart: Cart, merged: boolean, conflict_items: CartItem[] }` |
| Transformation | Merge guest localStorage cart with server cart; handle quantity conflicts |
| Loading Strategy | Background sync; show merge dialog if conflicts |
| Cache Config | Invalidate `cart` query post-merge |
| Error Handling | Keep both carts; prompt user to manually resolve conflicts |
| Database Entity | `carts` MERGE logic on `cart_items` |
| Security | Authenticated |

---

## 5. Checkout Flow

### DF-CHECKOUT-001 — Checkout Initialization

| Field | Value |
|-------|-------|
| Data Flow ID | `DF-CHECKOUT-001` |
| Page | Checkout (`/checkout`) |
| Widget | `CheckoutPage` (root) |
| Trigger | Route navigation from cart |
| Request | `GET /api/v1/cart` (verify cart not empty) |
| Response | `{ cart: Cart, items: CartItem[], total: number }` |
| Transformation | Validate stock availability; redirect to cart if empty |
| Loading Strategy | Initial load; block checkout if cart empty |
| Cache Config | `staleTime: 0` |
| Error Handling | Redirect to cart page with message |
| Database Entity | `carts` JOIN `cart_items` JOIN `products` |
| Security | Authenticated |

### DF-CHECKOUT-002 — Address Selection

| Field | Value |
|-------|-------|
| Data Flow ID | `DF-CHECKOUT-002` |
| Page | Checkout (`/checkout`) |
| Widget | `AddressSelector` |
| Trigger | Checkout page mount |
| Request | `GET /api/v1/customers/addresses` |
| Response | `{ addresses: Address[] }` |
| Transformation | Mark default address; map to `AddressCard[]` with `is_default` badge |
| Loading Strategy | Initial load; lazy-load address form on "Add New" |
| Cache Config | `staleTime: 5min`, `cacheTime: 15min` |
| Error Handling | Show empty state with "Add Address" CTA |
| Database Entity | `customer_addresses` WHERE `customer_id = current_user` |
| Security | Authenticated |

### DF-CHECKOUT-003 — Shipping Method Calculation

| Field | Value |
|-------|-------|
| Data Flow ID | `DF-CHECKOUT-003` |
| Page | Checkout (`/checkout`) |
| Widget | `ShippingMethodSelector` |
| Trigger | Address selection (DF-CHECKOUT-002) |
| Request | `POST /api/v1/shipping/calculate` with `{ address_id, items: [{product_id, quantity}] }` |
| Response | `{ methods: ShippingMethod[], estimated_delivery: string }` |
| Transformation | Sort by `price` ascending; format delivery date in Arabic |
| Loading Strategy | Triggered on address change; loading skeleton |
| Cache Config | `staleTime: 0` (always recalculate) |
| Error Handling | Show "Shipping unavailable for this address" with address change option |
| Database Entity | `shipping_methods` JOIN `shipping_zones` |
| Security | Authenticated |

### DF-CHECKOUT-004 — Coupon Validation

| Field | Value |
|-------|-------|
| Data Flow ID | `DF-CHECKOUT-004` |
| Page | Checkout (`/checkout`) |
| Widget | `CouponInput` |
| Trigger | Coupon code submit button |
| Request | `POST /api/v1/coupons/validate` with `{ code, cart_total, items }` |
| Response | `{ valid: boolean, discount: number, discount_type: string, message: string }` |
| Transformation | Apply discount to order summary; update `discount_total` |
| Loading Strategy | On-demand; debounce 500ms |
| Cache Config | No cache — always validate server-side |
| Error Handling | Show validation message; clear coupon on invalid |
| Database Entity | `coupons` WHERE `code = ? AND is_active = true` |
| Security | Authenticated |

### DF-CHECKOUT-005 — Order Creation

| Field | Value |
|-------|-------|
| Data Flow ID | `DF-CHECKOUT-005` |
| Page | Checkout (`/checkout`) |
| Widget | `PlaceOrderButton` |
| Trigger | "Place Order" button click |
| Request | `POST /api/v1/orders` with `{ address_id, shipping_method_id, payment_method, coupon_code?, notes? }` |
| Response | `{ order: Order, payment_url?: string }` |
| Transformation | Clear cart; redirect to payment gateway or order confirmation |
| Loading Strategy | Blocking — disable button during request; full-page loading overlay |
| Cache Config | Invalidate `cart`, `orders` queries |
| Error Handling | Show specific errors (stock unavailable, payment failed); allow retry |
| Database Entity | `orders` INSERT; `order_items` INSERT; `cart_items` DELETE; `inventory` UPDATE |
| Security | Authenticated |

### DF-CHECKOUT-006 — Payment Processing

| Field | Value |
|-------|-------|
| Data Flow ID | `DF-CHECKOUT-006` |
| Page | Payment Gateway Callback |
| Widget | `PaymentResultPage` |
| Trigger | Payment gateway redirect/callback |
| Request | `GET /api/v1/orders/{id}/payment-status` |
| Response | `{ order_id: string, status: string, transaction_id: string }` |
| Transformation | Map gateway status to internal status; show success/failure page |
| Loading Strategy | Initial load on callback |
| Cache Config | `staleTime: 0` |
| Error Handling | Show payment pending page; allow manual status check |
| Database Entity | `orders` UPDATE `payment_status`; `payment_transactions` INSERT |
| Security | Authenticated; webhook signature verification for gateway |

---

## 6. Order History & Tracking

### DF-ORDERS-001 — Order List Loading

| Field | Value |
|-------|-------|
| Data Flow ID | `DF-ORDERS-001` |
| Page | Order History (`/orders`) |
| Widget | `OrderList` |
| Trigger | Page mount; tab change (all/pending/delivered/cancelled) |
| Request | `GET /api/v1/orders?status={filter}&page={page}&per_page=10` |
| Response | `{ orders: OrderSummary[], total: number, page: number }` |
| Transformation | Map to `OrderCard[]` with status badge, date, total, item preview |
| Loading Strategy | Pagination (page buttons); skeleton loading |
| Cache Config | `staleTime: 1min`, `cacheTime: 5min` |
| Error Handling | Show "No orders found" for empty; retry on network error |
| Database Entity | `orders` JOIN `order_items` JOIN `vendors` WHERE `customer_id = current_user` |
| Security | Authenticated — own orders only |

### DF-ORDERS-002 — Order Detail Loading

| Field | Value |
|-------|-------|
| Data Flow ID | `DF-ORDERS-002` |
| Page | Order Detail (`/orders/[id]`) |
| Widget | `OrderDetailPage` |
| Trigger | Route navigation to `/orders/{id}` |
| Request | `GET /api/v1/orders/{id}` |
| Response | `{ order: OrderDetail }` |
| Transformation | Map to detail sections: items, address, shipping, payment, timeline |
| Loading Strategy | Initial load with skeleton |
| Cache Config | `staleTime: 1min`, `cacheTime: 5min` |
| Error Handling | 404 → redirect to order list; 403 → unauthorized message |
| Database Entity | `orders` JOIN `order_items`, `order_status_history`, `vendors`, `customer_addresses` |
| Security | Authenticated — own orders only |

### DF-ORDERS-003 — Cancel Order

| Field | Value |
|-------|-------|
| Data Flow ID | `DF-ORDERS-003` |
| Page | Order Detail (`/orders/[id]`) |
| Widget | `CancelButton` |
| Trigger | Button click + confirmation dialog |
| Request | `POST /api/v1/orders/{id}/cancel` with `{ reason }` |
| Response | `{ order: Order }` |
| Transformation | Optimistic: update status badge to "Cancelled"; disable cancel button |
| Loading Strategy | Optimistic update; confirmation dialog |
| Cache Config | Invalidate `order` and `orders` queries |
| Error Handling | Revert status on failure; show error toast |
| Database Entity | `orders` UPDATE `status = 'cancelled'`; `order_status_history` INSERT |
| Security | Authenticated — own orders only; status must be cancellable |

### DF-ORDERS-004 — Confirm Delivery

| Field | Value |
|-------|-------|
| Data Flow ID | `DF-ORDERS-004` |
| Page | Order Detail (`/orders/[id]`) |
| Widget | `ConfirmDeliveryButton` |
| Trigger | Button click |
| Request | `POST /api/v1/orders/{id}/confirm-delivery` |
| Response | `{ order: Order }` |
| Transformation | Update status to "Delivered"; enable "Return" and "Review" buttons |
| Loading Strategy | Optimistic update |
| Cache Config | Invalidate `order` and `orders` queries |
| Error Handling | Revert on failure |
| Database Entity | `orders` UPDATE `status = 'delivered'`, `delivered_at = NOW()`; `order_status_history` INSERT |
| Security | Authenticated — own orders only; status must be "shipped" |

### DF-ORDERS-005 — Request Return

| Field | Value |
|-------|-------|
| Data Flow ID | `DF-ORDERS-005` |
| Page | Order Detail (`/orders/[id]`) |
| Widget | `ReturnRequestForm` |
| Trigger | Button click → modal open → form submit |
| Request | `POST /api/v1/orders/{id}/return` with `{ reason, items: [{ item_id, quantity, reason }] }` |
| Response | `{ return_request: ReturnRequest }` |
| Transformation | Show return request confirmation; update order status |
| Loading Strategy | Form submission; loading overlay |
| Cache Config | Invalidate `order` query |
| Error Handling | Show specific return policy violations |
| Database Entity | `return_requests` INSERT; `order_items` UPDATE `return_status` |
| Security | Authenticated — own orders; within return window |

---

## 7. Wallet Operations

### DF-WALLET-001 — Balance Loading

| Field | Value |
|-------|-------|
| Data Flow ID | `DF-WALLET-001` |
| Page | Wallet (`/wallet`) |
| Widget | `WalletBalance` |
| Trigger | Page mount |
| Request | `GET /api/v1/wallet/balance` |
| Response | `{ balance: number, currency: string, formatted_balance: string }` |
| Transformation | Display formatted balance with currency symbol |
| Loading Strategy | Initial load; background refresh every 30s |
| Cache Config | `staleTime: 0`, `cacheTime: 1min` |
| Error Handling | Show masked balance with "Unable to load" message |
| Database Entity | `wallets` WHERE `customer_id = current_user` |
| Security | Authenticated |

### DF-WALLET-002 — Transaction History

| Field | Value |
|-------|-------|
| Data Flow ID | `DF-WALLET-002` |
| Page | Wallet (`/wallet`) |
| Widget | `TransactionList` |
| Trigger | Page mount; tab change (all/credit/debit) |
| Request | `GET /api/v1/wallet/transactions?type={filter}&page={page}&per_page=20` |
| Response | `{ transactions: Transaction[], total: number, page: number }` |
| Transformation | Map to `TransactionRow[]` with icon, description, amount (+/-), date |
| Loading Strategy | Pagination; skeleton loading |
| Cache Config | `staleTime: 1min`, `cacheTime: 5min` |
| Error Handling | Show "No transactions" for empty; retry on error |
| Database Entity | `wallet_transactions` WHERE `wallet_id = current_wallet` |
| Security | Authenticated |

### DF-WALLET-003 — Top-Up Wallet

| Field | Value |
|-------|-------|
| Data Flow ID | `DF-WALLET-003` |
| Page | Wallet Top-Up (`/wallet/topup`) |
| Widget | `TopUpForm` |
| Trigger | Form submit |
| Request | `POST /api/v1/wallet/topup` with `{ amount, payment_method }` |
| Response | `{ transaction: Transaction, payment_url?: string }` |
| Transformation | Redirect to payment gateway; show "Processing" state |
| Loading Strategy | Blocking form submission |
| Cache Config | Invalidate `wallet-balance` and `wallet-transactions` queries |
| Error Handling | Show payment failure message; allow retry |
| Database Entity | `wallet_transactions` INSERT; `payment_transactions` INSERT |
| Security | Authenticated |

### DF-WALLET-004 — Transfer Funds

| Field | Value |
|-------|-------|
| Data Flow ID | `DF-WALLET-004` |
| Page | Wallet Transfer (`/wallet/transfer`) |
| Widget | `TransferForm` |
| Trigger | Form submit |
| Request | `POST /api/v1/wallet/transfer` with `{ recipient_id, amount, note? }` |
| Response | `{ transaction: Transaction, new_balance: number }` |
| Transformation | Optimistic: update balance display; show success toast |
| Loading Strategy | Blocking form submission; confirmation dialog for large amounts |
| Cache Config | Invalidate `wallet-balance` and `wallet-transactions` queries |
| Error Handling | Show insufficient balance; recipient not found errors |
| Database Entity | `wallet_transactions` INSERT (debit + credit pair); `wallets` UPDATE balance |
| Security | Authenticated; transfer limit checks |

---

## 8. Vendor Dashboard

### DF-VENDASH-001 — Dashboard Overview

| Field | Value |
|-------|-------|
| Data Flow ID | `DF-VENDASH-001` |
| Page | Vendor Dashboard (`/vendor/dashboard`) |
| Widget | `DashboardOverview` |
| Trigger | Page mount |
| Request | `GET /api/v1/vendors/dashboard` |
| Response | `{ stats: DashboardStats, recent_orders: OrderSummary[], sales_chart: ChartData }` |
| Transformation | Map stats to `StatCard[]`; format chart data for recharts |
| Loading Strategy | Initial load with skeleton |
| Cache Config | `staleTime: 1min`, `cacheTime: 5min` |
| Error Handling | Show partial data if some stats fail; retry button |
| Database Entity | `vendors` JOIN `orders`, `order_items`, `products` aggregated |
| Security | Authenticated — vendor role required |

### DF-VENDASH-002 — Sales Analytics

| Field | Value |
|-------|-------|
| Data Flow ID | `DF-VENDASH-002` |
| Page | Vendor Dashboard (`/vendor/dashboard`) |
| Widget | `SalesChart` |
| Trigger | Date range picker change |
| Request | `GET /api/v1/vendors/dashboard/analytics?start={date}&end={date}&period={daily|weekly|monthly}` |
| Response | `{ data_points: AnalyticsPoint[], totals: AnalyticsTotals }` |
| Transformation | Map to chart series; compute deltas (vs previous period) |
| Loading Strategy | On-demand; date range change triggers refetch |
| Cache Config | `staleTime: 5min`, `cacheTime: 30min` |
| Error Handling | Show "Analytics unavailable" with last cached data |
| Database Entity | `order_items` JOIN `orders` aggregated by date |
| Security | Authenticated — vendor role |

### DF-VENDASH-003 — Recent Orders Widget

| Field | Value |
|-------|-------|
| Data Flow ID | `DF-VENDASH-003` |
| Page | Vendor Dashboard (`/vendor/dashboard`) |
| Widget | `RecentOrdersWidget` |
| Trigger | Page mount; WebSocket new order event |
| Request | `GET /api/v1/vendors/orders?limit=5&sort=created_at&order=desc` |
| Response | `{ orders: OrderSummary[] }` |
| Transformation | Map to compact order row with status badge |
| Loading Strategy | Initial load; real-time via WebSocket push |
| Cache Config | `staleTime: 0`, `cacheTime: 2min` |
| Error Handling | Show "No recent orders" |
| Database Entity | `orders` JOIN `order_items` WHERE `vendor_id = current_vendor` |
| Security | Authenticated — vendor role |

---

## 9. Vendor Product Management

### DF-VPROD-001 — Product List Loading

| Field | Value |
|-------|-------|
| Data Flow ID | `DF-VPROD-001` |
| Page | Vendor Products (`/vendor/products`) |
| Widget | `ProductTable` |
| Trigger | Page mount; filter/search change |
| Request | `GET /api/v1/vendors/products?status={filter}&search={query}&page={page}&per_page=20` |
| Response | `{ products: VendorProduct[], total: number }` |
| Transformation | Map to table rows with status badge, stock count, actions |
| Loading Strategy | Pagination; debounced search |
| Cache Config | `staleTime: 30s`, `cacheTime: 5min` |
| Error Handling | Show "No products" for empty; retry on error |
| Database Entity | `products` WHERE `vendor_id = current_vendor` JOIN `product_images`, `inventory` |
| Security | Authenticated — vendor role |

### DF-VPROD-002 — Create Product

| Field | Value |
|-------|-------|
| Data Flow ID | `DF-VPROD-002` |
| Page | Vendor Product Create (`/vendor/products/new`) |
| Widget | `ProductForm` |
| Trigger | Form submit |
| Request | `POST /api/v1/vendors/products` with multipart form data `{ name, description, category_id, price, images[], variants[], stock, ... }` |
| Response | `{ product: VendorProduct }` |
| Transformation | Redirect to product list with success toast; optimistic add to list |
| Loading Strategy | Blocking form submission; image upload progress |
| Cache Config | Invalidate `vendor-products` query |
| Error Handling | Show field-specific validation errors; preserve form state |
| Database Entity | `products` INSERT; `product_images` INSERT; `product_variants` INSERT; `inventory` INSERT |
| Security | Authenticated — vendor role |

### DF-VPROD-003 — Edit Product

| Field | Value |
|-------|-------|
| Data Flow ID | `DF-VPROD-003` |
| Page | Vendor Product Edit (`/vendor/products/[id]/edit`) |
| Widget | `ProductForm` (pre-filled) |
| Trigger | Form submit |
| Request | `PUT /api/v1/vendors/products/{id}` with updated fields |
| Response | `{ product: VendorProduct }` |
| Transformation | Show success toast; update list cache |
| Loading Strategy | Blocking form submission |
| Cache Config | Invalidate `vendor-products` and `vendor-product-{id}` queries |
| Error Handling | Show validation errors; preserve form state |
| Database Entity | `products` UPDATE; `product_images` UPSERT/DELETE |
| Security | Authenticated — vendor role; own products only |

### DF-VPROD-004 — Delete Product

| Field | Value |
|-------|-------|
| Data Flow ID | `DF-VPROD-004` |
| Page | Vendor Products (`/vendor/products`) |
| Widget | `DeleteButton` (table row action) |
| Trigger | Button click + confirmation dialog |
| Request | `DELETE /api/v1/vendors/products/{id}` |
| Response | `{ success: boolean }` |
| Transformation | Optimistic: remove row from table; show undo toast (5s) |
| Loading Strategy | Optimistic with undo capability |
| Cache Config | Invalidate `vendor-products` query |
| Error Handling | Restore row on failure; show error toast |
| Database Entity | `products` SOFT DELETE (`is_deleted = true`) |
| Security | Authenticated — vendor role; own products only |

---

## 10. Vendor Order Management

### DF-VORD-001 — Order List Loading

| Field | Value |
|-------|-------|
| Data Flow ID | `DF-VORD-001` |
| Page | Vendor Orders (`/vendor/orders`) |
| Widget | `OrderTable` |
| Trigger | Page mount; status filter change |
| Request | `GET /api/v1/vendors/orders?status={filter}&page={page}&per_page=20` |
| Response | `{ orders: VendorOrder[], total: number }` |
| Transformation | Map to table rows with customer info, items, total, status |
| Loading Strategy | Pagination; skeleton loading |
| Cache Config | `staleTime: 30s`, `cacheTime: 5min` |
| Error Handling | Show "No orders" for empty |
| Database Entity | `orders` JOIN `order_items`, `customers`, `customer_addresses` WHERE `vendor_id = current_vendor` |
| Security | Authenticated — vendor role |

### DF-VORD-002 — Confirm Order

| Field | Value |
|-------|-------|
| Data Flow ID | `DF-VORD-002` |
| Page | Vendor Orders (`/vendor/orders`) |
| Widget | `ConfirmOrderButton` |
| Trigger | Button click |
| Request | `PUT /api/v1/vendors/orders/{id}/confirm` |
| Response | `{ order: VendorOrder }` |
| Transformation | Optimistic: update status badge to "Confirmed"; disable confirm button |
| Loading Strategy | Optimistic update |
| Cache Config | Invalidate `vendor-orders` query |
| Error Handling | Revert on failure |
| Database Entity | `orders` UPDATE `status = 'confirmed'`; `order_status_history` INSERT |
| Security | Authenticated — vendor role; own orders only |

### DF-VORD-003 — Order Status Update

| Field | Value |
|-------|-------|
| Data Flow ID | `DF-VORD-003` |
| Page | Vendor Order Detail (`/vendor/orders/[id]`) |
| Widget | `StatusUpdateDropdown` |
| Trigger | Status selection from dropdown |
| Request | `PUT /api/v1/vendors/orders/{id}/status` with `{ status, note? }` |
| Response | `{ order: VendorOrder }` |
| Transformation | Update status timeline; notify customer via WebSocket |
| Loading Strategy | On-demand |
| Cache Config | Invalidate `vendor-order-{id}` query |
| Error Handling | Show status transition error messages |
| Database Entity | `orders` UPDATE `status`; `order_status_history` INSERT |
| Security | Authenticated — vendor role; valid status transition only |

---

## 11. Admin Dashboard

### DF-ADMIN-001 — Dashboard Overview

| Field | Value |
|-------|-------|
| Data Flow ID | `DF-ADMIN-001` |
| Page | Admin Dashboard (`/admin/dashboard`) |
| Widget | `AdminDashboardOverview` |
| Trigger | Page mount |
| Request | `GET /api/v1/admin/dashboard` |
| Response | `{ stats: AdminStats, recent_activity: Activity[], alerts: Alert[] }` |
| Transformation | Map to stat cards, activity feed, alert banners |
| Loading Strategy | Initial load with skeleton |
| Cache Config | `staleTime: 1min`, `cacheTime: 5min` |
| Error Handling | Show partial data; retry button |
| Database Entity | Aggregated from `orders`, `customers`, `vendors`, `products`, `wallets` |
| Security | Authenticated — admin role required |

### DF-ADMIN-002 — Revenue Analytics

| Field | Value |
|-------|-------|
| Data Flow ID | `DF-ADMIN-002` |
| Page | Admin Dashboard (`/admin/dashboard`) |
| Widget | `RevenueChart` |
| Trigger | Date range picker change |
| Request | `GET /api/v1/admin/analytics/revenue?start={date}&end={date}` |
| Response | `{ data_points: RevenuePoint[], totals: RevenueTotals }` |
| Transformation | Map to line/bar chart; compute YoY, MoM deltas |
| Loading Strategy | On-demand; date range triggers refetch |
| Cache Config | `staleTime: 5min`, `cacheTime: 30min` |
| Error Handling | Show "Analytics unavailable" |
| Database Entity | `orders` JOIN `order_items` JOIN `payment_transactions` aggregated |
| Security | Authenticated — admin role |

### DF-ADMIN-003 — Platform Metrics

| Field | Value |
|-------|-------|
| Data Flow ID | `DF-ADMIN-003` |
| Page | Admin Dashboard (`/admin/dashboard`) |
| Widget | `PlatformMetrics` |
| Trigger | Page mount |
| Request | `GET /api/v1/admin/metrics` |
| Response | `{ metrics: { total_customers, active_vendors, total_products, pending_orders, ... } }` |
| Transformation | Map to metric cards with trend indicators |
| Loading Strategy | Initial load; background refresh every 1min |
| Cache Config | `staleTime: 30s`, `cacheTime: 2min` |
| Error Handling | Show cached values with "stale" indicator |
| Database Entity | Aggregated counts from multiple tables |
| Security | Authenticated — admin role |

---

## 12. Admin User/Vendor/Order Management

### DF-ADMINUSR-001 — Customer List

| Field | Value |
|-------|-------|
| Data Flow ID | `DF-ADMINUSR-001` |
| Page | Admin Customers (`/admin/customers`) |
| Widget | `CustomerTable` |
| Trigger | Page mount; search/filter change |
| Request | `GET /api/v1/admin/customers?search={query}&status={filter}&page={page}&per_page=20` |
| Response | `{ customers: Customer[], total: number }` |
| Transformation | Map to table rows with name, email, phone, orders count, status |
| Loading Strategy | Pagination; debounced search |
| Cache Config | `staleTime: 30s`, `cacheTime: 5min` |
| Error Handling | Show "No customers" for empty |
| Database Entity | `customers` LEFT JOIN `orders` aggregated |
| Security | Authenticated — admin role |

### DF-ADMINUSR-002 — Vendor List

| Field | Value |
|-------|-------|
| Data Flow ID | `DF-ADMINUSR-002` |
| Page | Admin Vendors (`/admin/vendors`) |
| Widget | `VendorTable` |
| Trigger | Page mount; search/filter change |
| Request | `GET /api/v1/admin/vendors?search={query}&status={filter}&page={page}&per_page=20` |
| Response | `{ vendors: Vendor[], total: number }` |
| Transformation | Map to table rows with store name, owner, products count, revenue, status |
| Loading Strategy | Pagination; debounced search |
| Cache Config | `staleTime: 30s`, `cacheTime: 5min` |
| Error Handling | Show "No vendors" for empty |
| Database Entity | `vendors` LEFT JOIN `products`, `orders` aggregated |
| Security | Authenticated — admin role |

### DF-ADMINUSR-003 — Admin Order List

| Field | Value |
|-------|-------|
| Data Flow ID | `DF-ADMINUSR-003` |
| Page | Admin Orders (`/admin/orders`) |
| Widget | `AdminOrderTable` |
| Trigger | Page mount; status filter change |
| Request | `GET /api/v1/admin/orders?status={filter}&page={page}&per_page=20` |
| Response | `{ orders: AdminOrder[], total: number }` |
| Transformation | Map to table rows with order ID, customer, vendor, total, status, date |
| Loading Strategy | Pagination; skeleton loading |
| Cache Config | `staleTime: 30s`, `cacheTime: 5min` |
| Error Handling | Show "No orders" for empty |
| Database Entity | `orders` JOIN `order_items`, `customers`, `vendors` |
| Security | Authenticated — admin role |

### DF-ADMINUSR-004 — User Detail (Customer/Vendor)

| Field | Value |
|-------|-------|
| Data Flow ID | `DF-ADMINUSR-004` |
| Page | Admin User Detail (`/admin/customers/[id]` or `/admin/vendors/[id]`) |
| Widget | `UserDetailPage` |
| Trigger | Route navigation |
| Request | `GET /api/v1/admin/customers/{id}` or `GET /api/v1/admin/vendors/{id}` |
| Response | `{ user: CustomerDetail | VendorDetail, recent_activity: Activity[] }` |
| Transformation | Map to detail sections: profile, orders, wallet, activity |
| Loading Strategy | Initial load with skeleton |
| Cache Config | `staleTime: 1min`, `cacheTime: 5min` |
| Error Handling | 404 → redirect to list |
| Database Entity | `customers`/`vendors` JOIN related tables |
| Security | Authenticated — admin role |

### DF-ADMINUSR-005 — Admin User Status Update

| Field | Value |
|-------|-------|
| Data Flow ID | `DF-ADMINUSR-005` |
| Page | Admin User Detail |
| Widget | `StatusToggleButton` |
| Trigger | Toggle click + confirmation |
| Request | `PUT /api/v1/admin/customers/{id}/status` with `{ status }` or `PUT /api/v1/admin/vendors/{id}/status` |
| Response | `{ user: Customer | Vendor }` |
| Transformation | Optimistic: update status badge |
| Loading Strategy | Optimistic with confirmation dialog |
| Cache Config | Invalidate user detail and list queries |
| Error Handling | Revert on failure |
| Database Entity | `customers`/`vendors` UPDATE `status` |
| Security | Authenticated — admin role |

---

## 13. Authentication Flow

### DF-AUTH-001 — Register

| Field | Value |
|-------|-------|
| Data Flow ID | `DF-AUTH-001` |
| Page | Register (`/register`) |
| Widget | `RegisterForm` |
| Trigger | Form submit |
| Request | `POST /api/v1/auth/register` with `{ name, email, phone, password, password_confirmation }` |
| Response | `{ user: User, requires_verification: boolean }` |
| Transformation | If `requires_verification` → redirect to OTP page; else → store tokens and redirect to home |
| Loading Strategy | Blocking form submission |
| Cache Config | No cache |
| Error Handling | Show field-specific validation errors (email exists, phone exists) |
| Database Entity | `customers` INSERT |
| Security | Public; rate limited (5 attempts/min) |

### DF-AUTH-002 — Login

| Field | Value |
|-------|-------|
| Data Flow ID | `DF-AUTH-002` |
| Page | Login (`/login`) |
| Widget | `LoginForm` |
| Trigger | Form submit |
| Request | `POST /api/v1/auth/login` with `{ identifier (email/phone), password }` |
| Response | `{ user: User, access_token, refresh_token, requires_verification: boolean }` |
| Transformation | Store tokens in secure HTTP-only cookies; update auth state in Zustand; redirect to previous page or home |
| Loading Strategy | Blocking form submission |
| Cache Config | No cache |
| Error Handling | Show "Invalid credentials" with rate limit warning after 5 attempts |
| Database Entity | `customers` SELECT; `refresh_tokens` INSERT |
| Security | Public; rate limited (5 attempts/min, 15 lockout) |

### DF-AUTH-003 — OTP Verification

| Field | Value |
|-------|-------|
| Data Flow ID | `DF-AUTH-003` |
| Page | OTP Verify (`/verify-otp`) |
| Widget | `OTPForm` |
| Trigger | Form submit |
| Request | `POST /api/v1/auth/verify-otp` with `{ phone, code }` |
| Response | `{ user: User, access_token, refresh_token }` |
| Transformation | Store tokens; update auth state; redirect to home |
| Loading Strategy | Blocking form submission; auto-submit on 6 digits |
| Cache Config | No cache |
| Error Handling | Show "Invalid code" with resend option after 60s cooldown |
| Database Entity | `otp_codes` SELECT + DELETE; `customers` UPDATE `is_verified = true` |
| Security | Public; rate limited (3 attempts/min, 5 lockout) |

### DF-AUTH-004 — Forgot Password

| Field | Value |
|-------|-------|
| Data Flow ID | `DF-AUTH-004` |
| Page | Forgot Password (`/forgot-password`) |
| Widget | `ForgotPasswordForm` |
| Trigger | Form submit |
| Request | `POST /api/v1/auth/forgot-password` with `{ email }` |
| Response | `{ message: "Reset link sent" }` |
| Transformation | Show success message regardless of email existence (prevents enumeration) |
| Loading Strategy | Blocking form submission |
| Cache Config | No cache |
| Error Handling | Show generic success message even on error |
| Database Entity | `password_resets` INSERT |
| Security | Public; rate limited (3 attempts/min) |

### DF-AUTH-005 — Reset Password

| Field | Value |
|-------|-------|
| Data Flow ID | `DF-AUTH-005` |
| Page | Reset Password (`/reset-password?token={token}`) |
| Widget | `ResetPasswordForm` |
| Trigger | Form submit |
| Request | `POST /api/v1/auth/reset-password` with `{ token, password, password_confirmation }` |
| Response | `{ message: "Password reset successful" }` |
| Transformation | Redirect to login with success message |
| Loading Strategy | Blocking form submission |
| Cache Config | No cache |
| Error Handling | Show "Invalid or expired token" on failure |
| Database Entity | `password_resets` SELECT + DELETE; `customers` UPDATE `password` |
| Security | Public; token expires in 1 hour |

### DF-AUTH-006 — Refresh Token

| Field | Value |
|-------|-------|
| Data Flow ID | `DF-AUTH-006` |
| Page | Any (intercepted by Axios interceptor) |
| Widget | N/A (background) |
| Trigger | 401 response from any API call |
| Request | `POST /api/v1/auth/refresh-token` with `{ refresh_token }` |
| Response | `{ access_token, refresh_token }` |
| Transformation | Update stored tokens; retry original request |
| Loading Strategy | Background; automatic |
| Cache Config | No cache |
| Error Handling | If refresh fails → logout and redirect to login |
| Database Entity | `refresh_tokens` SELECT + UPDATE |
| Security | Refresh token required |

### DF-AUTH-007 — Logout

| Field | Value |
|-------|-------|
| Data Flow ID | `DF-AUTH-007` |
| Page | Any (header/profile menu) |
| Widget | `LogoutButton` |
| Trigger | Button click |
| Request | `POST /api/v1/auth/logout` |
| Response | `{ message: "Logged out" }` |
| Transformation | Clear tokens; reset auth state; clear query cache; redirect to login |
| Loading Strategy | Immediate; optimistic |
| Cache Config | Clear all cached queries |
| Error Handling | Force logout even if API call fails |
| Database Entity | `refresh_tokens` DELETE WHERE `customer_id = current_user` |
| Security | Authenticated |

---

## Appendix A: Data Flow Cross-Reference Matrix

| Page | Data Flow IDs | Total Flows |
|------|--------------|-------------|
| Homepage | DF-HOME-001, 002, 003, 004 | 4 |
| Search | DF-SEARCH-001, 002, 003, 004 | 4 |
| Product Detail | DF-PRODUCT-001, 002, 003, 004 | 4 |
| Cart | DF-CART-001, 002, 003, 004, 005 | 5 |
| Checkout | DF-CHECKOUT-001, 002, 003, 004, 005, 006 | 6 |
| Orders | DF-ORDERS-001, 002, 003, 004, 005 | 5 |
| Wallet | DF-WALLET-001, 002, 003, 004 | 4 |
| Vendor Dashboard | DF-VENDASH-001, 002, 003 | 3 |
| Vendor Products | DF-VPROD-001, 002, 003, 004 | 4 |
| Vendor Orders | DF-VORD-001, 002, 003 | 3 |
| Admin Dashboard | DF-ADMIN-001, 002, 003 | 3 |
| Admin Management | DF-ADMINUSR-001, 002, 003, 004, 005 | 5 |
| Authentication | DF-AUTH-001, 002, 003, 004, 005, 006, 007 | 7 |
| **TOTAL** | | **57** |

## Appendix B: Loading Strategy Reference

| Strategy | Description | Use Cases |
|----------|-------------|-----------|
| Initial | Load on page mount | Page data, dashboards |
| Lazy | Load on viewport intersection | Reviews, related products |
| Pagination | Page-based data fetching | Lists, tables |
| Infinite Scroll | Continuous loading on scroll | Product feeds, transaction history |
| Refresh | Periodic background refetch | Wallet balance, order status |
| Polling | Fixed-interval refetch | Flash sale timer, delivery status |
| WebSocket | Real-time push updates | New orders, notifications |
| Optimistic | Update UI before server response | Cart, status changes |
| Cache | Serve from cache-first | Categories, static config |

## Appendix C: Cache Configuration Summary

| Query | staleTime | cacheTime | Rationale |
|-------|-----------|-----------|-----------|
| Cart | 0 | 5min | Always fresh; critical for checkout |
| Wallet balance | 0 | 1min | Financial data; high freshness |
| Orders list | 1min | 5min | Moderate freshness |
| Product detail | 3min | 15min | Relatively stable |
| Categories | 15min | 60min | Rarely changes |
| Featured products | 5min | 30min | Updates daily |
| Search results | 2min | 10min | Depends on inventory |
| Vendor products | 30s | 5min | Vendor edits frequently |
| Promotions | 10min | 30min | Scheduled changes |

## Appendix D: Error Handling Patterns

| Error Code | Pattern | User Experience |
|------------|---------|-----------------|
| 400 | Show field-level validation errors | Inline error messages under form fields |
| 401 | Trigger token refresh (DF-AUTH-006); if fails → logout | Seamless if refresh succeeds; redirect to login |
| 403 | Show "Unauthorized" message | Block access; suggest contact admin |
| 404 | Redirect to not-found page or show empty state | Contextual 404 with back navigation |
| 409 | Show conflict message with resolution options | "Item already in cart" with increment option |
| 422 | Show business logic errors | "Insufficient stock", "Coupon expired" |
| 429 | Show rate limit message with cooldown | "Too many attempts. Try again in X seconds" |
| 500 | Show generic error with retry button | "Something went wrong. Please try again." |
| Network | Show offline indicator; queue actions | Toast "You're offline. Changes will sync." |
