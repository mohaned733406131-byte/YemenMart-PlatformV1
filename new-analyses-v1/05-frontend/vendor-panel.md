# Vendor Panel - YemenMart

## Overview

Vendor management portal built with React 18 and Vite. Provides vendors with tools to manage their stores, products, orders, inventory, analytics, coupons, KYC, and team members. Supports RBAC with Vendor Owner and Vendor Staff permission levels, real-time notifications, and Arabic-first interface with RTL/LTR support.

## Tech Stack

| Technology | Purpose |
|------------|---------|
| React 18 | UI library |
| Vite | Build tool and dev server |
| TypeScript | Type safety |
| TanStack Query v5 | Server state management |
| Zustand | Client state management |
| React Router v6 | Client-side routing |
| Tailwind CSS 4 | Styling |
| React Hook Form + Zod | Forms and validation |
| Recharts | Charts and analytics |
| TanStack Table | Data tables |
| React Hot Toast | Notifications |
| Lucide React | Icons |
| i18next | Internationalization (AR/EN) |

## RBAC Permission System

### Role Definitions

```typescript
// types/roles.ts
export enum VendorRole {
  OWNER = 'vendor_owner',
  STAFF = 'vendor_staff',
}

export enum Permission {
  VIEW_STORE = 'store:view',
  EDIT_STORE = 'store:edit',
  VIEW_STORE_ANALYTICS = 'store:analytics:view',
  VIEW_PRODUCTS = 'products:view',
  CREATE_PRODUCT = 'products:create',
  EDIT_PRODUCT = 'products:edit',
  DELETE_PRODUCT = 'products:delete',
  MANAGE_INVENTORY = 'products:inventory:manage',
  VIEW_ORDERS = 'orders:view',
  PROCESS_ORDERS = 'orders:process',
  CANCEL_ORDERS = 'orders:cancel',
  VIEW_ORDER_DETAILS = 'orders:details:view',
  UPDATE_ORDER_STATUS = 'orders:status:update',
  ISSUE_REFUNDS = 'orders:refunds:issue',
  VIEW_COUPONS = 'coupons:view',
  CREATE_COUPON = 'coupons:create',
  EDIT_COUPON = 'coupons:edit',
  DELETE_COUPON = 'coupons:delete',
  VIEW_FINANCE = 'finance:view',
  VIEW_REPORTS = 'reports:view',
  EXPORT_REPORTS = 'reports:export',
  VIEW_TEAM = 'team:view',
  INVITE_TEAM_MEMBER = 'team:invite:edit',
  REMOVE_TEAM_MEMBER = 'team:remove',
  EDIT_TEAM_MEMBER_ROLE = 'team:role:edit',
  VIEW_KYC = 'kyc:view',
  SUBMIT_KYC = 'kyc:submit',
  UPDATE_KYC = 'kyc:update',
}

export const ROLE_PERMISSIONS: Record<VendorRole, Permission[]> = {
  [VendorRole.OWNER]: Object.values(Permission),
  [VendorRole.STAFF]: [
    Permission.VIEW_STORE,
    Permission.VIEW_STORE_ANALYTICS,
    Permission.VIEW_PRODUCTS,
    Permission.CREATE_PRODUCT,
    Permission.EDIT_PRODUCT,
    Permission.VIEW_ORDERS,
    Permission.PROCESS_ORDERS,
    Permission.VIEW_ORDER_DETAILS,
    Permission.UPDATE_ORDER_STATUS,
    Permission.VIEW_COUPONS,
    Permission.VIEW_FINANCE,
    Permission.VIEW_REPORTS,
  ],
};
```

### Permission Hook

```typescript
// hooks/usePermissions.ts
import { useAuthStore } from '@/stores/authStore';
import { VendorRole, Permission, ROLE_PERMISSIONS } from '@/types/roles';

export function usePermissions() {
  const { user } = useAuthStore();

  const hasPermission = (permission: Permission): boolean => {
    if (!user) return false;
    if (user.role === VendorRole.OWNER) return true;
    const rolePermissions = ROLE_PERMISSIONS[user.role as VendorRole] || [];
    return rolePermissions.includes(permission);
  };

  const hasAnyPermission = (permissions: Permission[]): boolean =>
    permissions.some((p) => hasPermission(p));

  const hasAllPermissions = (permissions: Permission[]): boolean =>
    permissions.every((p) => hasPermission(p));

  return {
    role: user?.role as VendorRole | undefined,
    hasPermission,
    hasAnyPermission,
    hasAllPermissions,
    isOwner: user?.role === VendorRole.OWNER,
    isStaff: user?.role === VendorRole.STAFF,
  };
}
```

### Permission Guard

```typescript
// components/auth/PermissionGuard.tsx
import { ReactNode } from 'react';
import { usePermissions } from '@/hooks/usePermissions';
import { Permission } from '@/types/roles';

interface PermissionGuardProps {
  permission: Permission;
  children: ReactNode;
  fallback?: ReactNode;
}

export function PermissionGuard({ permission, children, fallback = null }: PermissionGuardProps) {
  const { hasPermission } = usePermissions();
  if (!hasPermission(permission)) return <>{fallback}</>;
  return <>{children}</>;
}
```

## Pages

### 1. Dashboard (`/dashboard`)

```typescript
// pages/Dashboard.tsx
import { useQuery } from '@tanstack/react-query';
import { StatsCard } from '@/components/dashboard/StatsCard';
import { RecentOrders } from '@/components/dashboard/RecentOrders';
import { SalesChart } from '@/components/dashboard/SalesChart';
import { TopProducts } from '@/components/dashboard/TopProducts';
import { RevenueChart } from '@/components/dashboard/RevenueChart';
import { NotificationsList } from '@/components/dashboard/NotificationsList';
import { QuickActions } from '@/components/dashboard/QuickActions';
import { useAuthStore } from '@/stores/authStore';
import { DollarSign, ShoppingCart, Package, Star } from 'lucide-react';

export function Dashboard() {
  const { user } = useAuthStore();
  const { data: stats, isLoading } = useQuery({
    queryKey: ['vendor-dashboard-stats'],
    queryFn: fetchDashboardStats,
    refetchInterval: 30000,
  });

  return (
    <div className="space-y-6">
      <div className="flex items-center justify-between">
        <div>
          <h1 className="text-2xl font-bold">مرحباً، {user?.name}</h1>
          <p className="text-neutral-500">إليك ملخص نشاط متجرك اليوم</p>
        </div>
        <QuickActions />
      </div>

      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4">
        <StatsCard title="المبيعات اليوم" value={stats?.todaySales || 0} currency="YER"
          icon={DollarSign} change={stats?.salesChange} loading={isLoading} />
        <StatsCard title="الطلبات الجديدة" value={stats?.newOrders || 0}
          icon={ShoppingCart} change={stats?.ordersChange} loading={isLoading} />
        <StatsCard title="المنتجات النشطة" value={stats?.activeProducts || 0}
          icon={Package} loading={isLoading} />
        <StatsCard title="متوسط التقييم" value={stats?.avgRating || 0}
          icon={Star} change={stats?.ratingChange} loading={isLoading} format="decimal" />
      </div>

      <div className="grid grid-cols-1 lg:grid-cols-2 gap-6">
        <SalesChart data={stats?.salesChart || []} />
        <RevenueChart data={stats?.revenueChart || []} />
      </div>

      <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
        <div className="lg:col-span-2">
          <RecentOrders orders={stats?.recentOrders || []} />
        </div>
        <div className="space-y-6">
          <TopProducts products={stats?.topProducts || []} />
          <NotificationsList />
        </div>
      </div>
    </div>
  );
}
```

### 2. Products Management (`/products`)

```typescript
// pages/Products.tsx
import { useState, useMemo } from 'react';
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { useReactTable, getCoreRowModel, getPaginationRowModel } from '@tanstack/react-table';
import { DataTable } from '@/components/ui/DataTable';
import { Button } from '@/components/ui/Button';
import { Input } from '@/components/ui/Input';
import { Select } from '@/components/ui/Select';
import { Badge } from '@/components/ui/Badge';
import { Modal } from '@/components/ui/Modal';
import { ProductForm } from '@/components/products/ProductForm';
import { ProductDeleteDialog } from '@/components/products/ProductDeleteDialog';
import { Permission } from '@/types/roles';
import { PermissionGuard } from '@/components/auth/PermissionGuard';
import { fetchProducts, deleteProduct } from '@/lib/api/vendor/products';
import { toast } from 'react-hot-toast';
import { Plus, Search, Edit, Trash2 } from 'lucide-react';

export function Products() {
  const queryClient = useQueryClient();
  const [search, setSearch] = useState('');
  const [category, setCategory] = useState('');
  const [status, setStatus] = useState('');
  const [isCreateModalOpen, setIsCreateModalOpen] = useState(false);
  const [editingProduct, setEditingProduct] = useState<any>(null);
  const [deletingProduct, setDeletingProduct] = useState<any>(null);

  const { data, isLoading } = useQuery({
    queryKey: ['vendor-products', { search, category, status }],
    queryFn: () => fetchProducts({ search, category, status }),
  });

  const deleteMutation = useMutation({
    mutationFn: deleteProduct,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['vendor-products'] });
      toast.success('تم حذف المنتج بنجاح');
      setDeletingProduct(null);
    },
    onError: () => toast.error('فشل حذف المنتج'),
  });

  const columns = useMemo(() => [
    {
      accessorKey: 'image',
      header: 'الصورة',
      cell: ({ row }) => (
        <img src={row.original.images[0]?.url || '/placeholder.png'}
          alt={row.original.name} className="w-12 h-12 rounded-lg object-cover" />
      ),
    },
    {
      accessorKey: 'name',
      header: 'اسم المنتج',
      cell: ({ row }) => (
        <div>
          <p className="font-medium">{row.original.name}</p>
          <p className="text-sm text-neutral-500">{row.original.sku}</p>
        </div>
      ),
    },
    { accessorKey: 'category', header: 'القسم', cell: ({ row }) => row.original.category?.name },
    {
      accessorKey: 'price',
      header: 'السعر',
      cell: ({ row }) => (
        <span className="font-medium">{row.original.price.toLocaleString('ar-YE')} يمني</span>
      ),
    },
    {
      accessorKey: 'stock',
      header: 'المخزون',
      cell: ({ row }) => (
        <Badge variant={row.original.stock === 0 ? 'error' : row.original.stock < 10 ? 'warning' : 'success'}>
          {row.original.stock === 0 ? 'نفد' : row.original.stock}
        </Badge>
      ),
    },
    {
      accessorKey: 'status',
      header: 'الحالة',
      cell: ({ row }) => (
        <Badge variant={row.original.isActive ? 'success' : 'neutral'}>
          {row.original.isActive ? 'نشط' : 'غير نشط'}
        </Badge>
      ),
    },
    {
      id: 'actions',
      header: 'الإجراءات',
      cell: ({ row }) => (
        <div className="flex items-center gap-2">
          <PermissionGuard permission={Permission.EDIT_PRODUCT}>
            <button onClick={() => setEditingProduct(row.original)}
              className="p-2 hover:bg-neutral-100 rounded-lg">
              <Edit className="w-4 h-4" />
            </button>
          </PermissionGuard>
          <PermissionGuard permission={Permission.DELETE_PRODUCT}>
            <button onClick={() => setDeletingProduct(row.original)}
              className="p-2 hover:bg-red-50 text-red-500 rounded-lg">
              <Trash2 className="w-4 h-4" />
            </button>
          </PermissionGuard>
        </div>
      ),
    },
  ], []);

  const table = useReactTable({
    data: data?.products || [],
    columns,
    getCoreRowModel: getCoreRowModel(),
    getPaginationRowModel: getPaginationRowModel(),
  });

  return (
    <div className="space-y-6">
      <div className="flex items-center justify-between">
        <h1 className="text-2xl font-bold">المنتجات</h1>
        <PermissionGuard permission={Permission.CREATE_PRODUCT}>
          <Button onClick={() => setIsCreateModalOpen(true)}>
            <Plus className="w-4 h-4 ml-2" />إضافة منتج
          </Button>
        </PermissionGuard>
      </div>

      <div className="flex flex-wrap items-center gap-3">
        <Input placeholder="بحث عن منتج..." value={search}
          onChange={(e) => setSearch(e.target.value)}
          icon={<Search className="w-4 h-4" />} className="max-w-sm" />
        <Select value={category} onChange={setCategory} className="w-40"
          options={[
            { value: '', label: 'جميع الأقسام' },
            { value: 'electronics', label: 'إلكترونيات' },
            { value: 'clothing', label: 'ملابس' },
            { value: 'home', label: 'منزل ومطبخ' },
          ]} />
        <Select value={status} onChange={setStatus} className="w-40"
          options={[
            { value: '', label: 'جميع الحالات' },
            { value: 'active', label: 'نشط' },
            { value: 'inactive', label: 'غير نشط' },
            { value: 'out_of_stock', label: 'نفد المخزون' },
          ]} />
      </div>

      <DataTable table={table} isLoading={isLoading} emptyMessage="لا توجد منتجات" />

      <Modal isOpen={isCreateModalOpen} onClose={() => setIsCreateModalOpen(false)}
        title="إضافة منتج جديد" size="xl">
        <ProductForm onSubmit={() => {
          queryClient.invalidateQueries({ queryKey: ['vendor-products'] });
          setIsCreateModalOpen(false);
        }} onCancel={() => setIsCreateModalOpen(false)} />
      </Modal>

      {editingProduct && (
        <Modal isOpen={true} onClose={() => setEditingProduct(null)}
          title="تعديل المنتج" size="xl">
          <ProductForm initialData={editingProduct} onSubmit={() => {
            queryClient.invalidateQueries({ queryKey: ['vendor-products'] });
            setEditingProduct(null);
          }} onCancel={() => setEditingProduct(null)} />
        </Modal>
      )}

      {deletingProduct && (
        <ProductDeleteDialog product={deletingProduct}
          onConfirm={() => deleteMutation.mutate(deletingProduct.id)}
          onCancel={() => setDeletingProduct(null)} isLoading={deleteMutation.isLoading} />
      )}
    </div>
  );
}
```

### 3. Orders Management (`/orders`)

```typescript
// pages/Orders.tsx
import { useState } from 'react';
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { DataTable } from '@/components/ui/DataTable';
import { Badge } from '@/components/ui/Badge';
import { OrderDetailDrawer } from '@/components/orders/OrderDetailDrawer';
import { OrderStatusUpdate } from '@/components/orders/OrderStatusUpdate';
import { Permission } from '@/types/roles';
import { PermissionGuard } from '@/components/auth/PermissionGuard';
import { fetchVendorOrders, updateOrderStatus } from '@/lib/api/vendor/orders';
import { toast } from 'react-hot-toast';
import { Eye, CheckCircle, Truck, Package, Clock } from 'lucide-react';

type OrderStatus = 'pending' | 'confirmed' | 'processing' | 'shipped' | 'delivered' | 'cancelled';

export function Orders() {
  const queryClient = useQueryClient();
  const [statusFilter, setStatusFilter] = useState<OrderStatus | ''>('');
  const [selectedOrder, setSelectedOrder] = useState<any>(null);
  const [updatingOrder, setUpdatingOrder] = useState<any>(null);

  const { data, isLoading } = useQuery({
    queryKey: ['vendor-orders', { status: statusFilter }],
    queryFn: () => fetchVendorOrders({ status: statusFilter }),
    refetchInterval: 15000,
  });

  const statusMutation = useMutation({
    mutationFn: ({ orderId, status }: { orderId: string; status: OrderStatus }) =>
      updateOrderStatus(orderId, status),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['vendor-orders'] });
      toast.success('تم تحديث حالة الطلب بنجاح');
      setUpdatingOrder(null);
    },
    onError: () => toast.error('فشل تحديث حالة الطلب'),
  });

  const getStatusBadge = (status: OrderStatus) => {
    const variants: Record<OrderStatus, string> = {
      pending: 'warning', confirmed: 'info', processing: 'info',
      shipped: 'primary', delivered: 'success', cancelled: 'error',
    };
    const labels: Record<OrderStatus, string> = {
      pending: 'قيد الانتظار', confirmed: 'مؤكد', processing: 'قيد التجهيز',
      shipped: 'تم الشحن', delivered: 'تم التوصيل', cancelled: 'ملغي',
    };
    return <Badge variant={variants[status]}>{labels[status]}</Badge>;
  };

  const columns = [
    { accessorKey: 'orderNumber', header: 'رقم الطلب',
      cell: ({ row }) => <span className="font-mono text-sm">#{row.original.orderNumber}</span> },
    { accessorKey: 'customer', header: 'العميل',
      cell: ({ row }) => (
        <div>
          <p className="font-medium">{row.original.customerName}</p>
          <p className="text-sm text-neutral-500">{row.original.customerPhone}</p>
        </div>
      ) },
    { accessorKey: 'items', header: 'المنتجات',
      cell: ({ row }) => <span>{row.original.itemCount} منتجات</span> },
    { accessorKey: 'total', header: 'المبلغ',
      cell: ({ row }) => (
        <span className="font-medium">{row.original.total.toLocaleString('ar-YE')} يمني</span>
      ) },
    { accessorKey: 'status', header: 'الحالة',
      cell: ({ row }) => getStatusBadge(row.original.status) },
    { accessorKey: 'createdAt', header: 'التاريخ',
      cell: ({ row }) => new Date(row.original.createdAt).toLocaleDateString('ar-YE') },
    { id: 'actions', header: 'الإجراءات',
      cell: ({ row }) => (
        <div className="flex items-center gap-1">
          <button onClick={() => setSelectedOrder(row.original)}
            className="p-2 hover:bg-neutral-100 rounded-lg">
            <Eye className="w-4 h-4" />
          </button>
          <PermissionGuard permission={Permission.UPDATE_ORDER_STATUS}>
            <button onClick={() => setUpdatingOrder(row.original)}
              className="p-2 hover:bg-neutral-100 rounded-lg">
              <CheckCircle className="w-4 h-4" />
            </button>
          </PermissionGuard>
        </div>
      ) },
  ];

  return (
    <div className="space-y-6">
      <h1 className="text-2xl font-bold">الطلبات</h1>

      <div className="flex flex-wrap gap-2">
        {[
          { value: '', label: 'الكل', icon: null },
          { value: 'pending', label: 'قيد الانتظار', icon: Clock },
          { value: 'confirmed', label: 'مؤكد', icon: CheckCircle },
          { value: 'processing', label: 'قيد التجهيز', icon: Package },
          { value: 'shipped', label: 'تم الشحن', icon: Truck },
          { value: 'delivered', label: 'تم التوصيل', icon: CheckCircle },
        ].map((filter) => (
          <button key={filter.value}
            onClick={() => setStatusFilter(filter.value as OrderStatus | '')}
            className={`flex items-center gap-2 px-4 py-2 rounded-lg text-sm font-medium transition-colors ${
              statusFilter === filter.value ? 'bg-primary-600 text-white' : 'bg-neutral-100 hover:bg-neutral-200'
            }`}>
            {filter.icon && <filter.icon className="w-4 h-4" />}
            {filter.label}
          </button>
        ))}
      </div>

      <DataTable data={data?.orders || []} columns={columns} isLoading={isLoading}
        emptyMessage="لا توجد طلبات" pagination={data?.pagination} />

      {selectedOrder && (
        <OrderDetailDrawer order={selectedOrder} onClose={() => setSelectedOrder(null)} />
      )}

      {updatingOrder && (
        <OrderStatusUpdate order={updatingOrder}
          onUpdate={(status) => statusMutation.mutate({ orderId: updatingOrder.id, status })}
          onCancel={() => setUpdatingOrder(null)} isLoading={statusMutation.isLoading} />
      )}
    </div>
  );
}
```

### 4. Inventory Management (`/inventory`)

```typescript
// pages/Inventory.tsx
import { useState } from 'react';
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { DataTable } from '@/components/ui/DataTable';
import { Button } from '@/components/ui/Button';
import { Input } from '@/components/ui/Input';
import { Badge } from '@/components/ui/Badge';
import { BulkStockUpdate } from '@/components/inventory/BulkStockUpdate';
import { StockHistory } from '@/components/inventory/StockHistory';
import { fetchInventory, updateStock } from '@/lib/api/vendor/inventory';
import { toast } from 'react-hot-toast';
import { AlertTriangle, TrendingDown, Upload, History } from 'lucide-react';

export function Inventory() {
  const queryClient = useQueryClient();
  const [search, setSearch] = useState('');
  const [showBulkUpdate, setShowBulkUpdate] = useState(false);
  const [showHistory, setShowHistory] = useState(false);
  const [selectedProduct, setSelectedProduct] = useState<any>(null);

  const { data, isLoading } = useQuery({
    queryKey: ['vendor-inventory', { search }],
    queryFn: () => fetchInventory({ search }),
  });

  const updateStockMutation = useMutation({
    mutationFn: ({ productId, quantity }: { productId: string; quantity: number }) =>
      updateStock(productId, quantity),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['vendor-inventory'] });
      toast.success('تم تحديث المخزون بنجاح');
    },
    onError: () => toast.error('فشل تحديث المخزون'),
  });

  const lowStockItems = data?.items?.filter((item: any) => item.stock < 10) || [];
  const outOfStockItems = data?.items?.filter((item: any) => item.stock === 0) || [];

  const columns = [
    { accessorKey: 'product', header: 'المنتج',
      cell: ({ row }) => (
        <div className="flex items-center gap-3">
          <img src={row.original.images[0]?.url || '/placeholder.png'}
            alt={row.original.name} className="w-10 h-10 rounded-lg object-cover" />
          <div>
            <p className="font-medium">{row.original.name}</p>
            <p className="text-sm text-neutral-500">{row.original.sku}</p>
          </div>
        </div>
      ) },
    { accessorKey: 'stock', header: 'المخزون الحالي',
      cell: ({ row }) => (
        <Badge variant={row.original.stock === 0 ? 'error' : row.original.stock < 10 ? 'warning' : 'success'}>
          {row.original.stock}
        </Badge>
      ) },
    { accessorKey: 'reservedStock', header: 'محجوز',
      cell: ({ row }) => row.original.reservedStock || 0 },
    { accessorKey: 'availableStock', header: 'متاح',
      cell: ({ row }) => (
        <span className="font-medium">{row.original.stock - (row.original.reservedStock || 0)}</span>
      ) },
    { id: 'actions', header: 'الإجراءات',
      cell: ({ row }) => (
        <div className="flex items-center gap-2">
          <Input type="number" defaultValue={row.original.stock} className="w-20"
            onBlur={(e) => {
              const newStock = parseInt(e.target.value);
              if (newStock !== row.original.stock) {
                updateStockMutation.mutate({ productId: row.original.id, quantity: newStock });
              }
            }} />
          <button onClick={() => { setSelectedProduct(row.original); setShowHistory(true); }}
            className="p-2 hover:bg-neutral-100 rounded-lg">
            <History className="w-4 h-4" />
          </button>
        </div>
      ) },
  ];

  return (
    <div className="space-y-6">
      <div className="flex items-center justify-between">
        <h1 className="text-2xl font-bold">إدارة المخزون</h1>
        <Button onClick={() => setShowBulkUpdate(true)}>
          <Upload className="w-4 h-4 ml-2" />تحديث جماعي
        </Button>
      </div>

      {(lowStockItems.length > 0 || outOfStockItems.length > 0) && (
        <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
          {outOfStockItems.length > 0 && (
            <div className="bg-red-50 dark:bg-red-900/20 border border-red-200 rounded-xl p-4">
              <div className="flex items-center gap-2 text-red-600">
                <AlertTriangle className="w-5 h-5" />
                <span className="font-medium">منتجات نفدت ({outOfStockItems.length})</span>
              </div>
            </div>
          )}
          {lowStockItems.length > 0 && (
            <div className="bg-yellow-50 dark:bg-yellow-900/20 border border-yellow-200 rounded-xl p-4">
              <div className="flex items-center gap-2 text-yellow-600">
                <TrendingDown className="w-5 h-5" />
                <span className="font-medium">مخزون منخفض ({lowStockItems.length})</span>
              </div>
            </div>
          )}
        </div>
      )}

      <Input placeholder="بحث في المخزون..." value={search}
        onChange={(e) => setSearch(e.target.value)} className="max-w-sm" />

      <DataTable data={data?.items || []} columns={columns} isLoading={isLoading}
        emptyMessage="لا توجد منتجات" />

      {showBulkUpdate && (
        <BulkStockUpdate onClose={() => setShowBulkUpdate(false)} onSuccess={() => {
          queryClient.invalidateQueries({ queryKey: ['vendor-inventory'] });
          setShowBulkUpdate(false);
        }} />
      )}

      {showHistory && selectedProduct && (
        <StockHistory product={selectedProduct} onClose={() => {
          setShowHistory(false);
          setSelectedProduct(null);
        }} />
      )}
    </div>
  );
}
```

### 5. Store Settings (`/settings`)

```typescript
// pages/StoreSettings.tsx
import { useState } from 'react';
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';
import { Tabs, TabsContent, TabsList, TabsTrigger } from '@/components/ui/Tabs';
import { Input } from '@/components/ui/Input';
import { Textarea } from '@/components/ui/Textarea';
import { Button } from '@/components/ui/Button';
import { ImageUpload } from '@/components/ui/ImageUpload';
import { TimePicker } from '@/components/ui/TimePicker';
import { fetchStoreSettings, updateStoreSettings } from '@/lib/api/vendor/store';
import { toast } from 'react-hot-toast';

const storeSchema = z.object({
  name: z.string().min(2, 'اسم المتجر مطلوب'),
  description: z.string().optional(),
  logo: z.string().optional(),
  banner: z.string().optional(),
  phone: z.string().regex(/^7[0-9]{8}$/, 'رقم الهاتف غير صحيح'),
  email: z.string().email('البريد الإلكتروني غير صحيح'),
  address: z.string().min(5, 'العنوان مطلوب'),
  governorate: z.string().min(1, 'المحافظة مطلوبة'),
  district: z.string().min(1, 'المنطقة مطلوبة'),
  openingTime: z.string(),
  closingTime: z.string(),
  workingDays: z.array(z.string()),
  shippingEnabled: z.boolean(),
  freeShippingThreshold: z.number().optional(),
  returnPolicy: z.string().optional(),
});

type StoreFormData = z.infer<typeof storeSchema>;

export function StoreSettings() {
  const queryClient = useQueryClient();
  const [activeTab, setActiveTab] = useState('general');

  const { data: settings } = useQuery({
    queryKey: ['vendor-store-settings'],
    queryFn: fetchStoreSettings,
  });

  const updateMutation = useMutation({
    mutationFn: updateStoreSettings,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['vendor-store-settings'] });
      toast.success('تم حفظ الإعدادات بنجاح');
    },
    onError: () => toast.error('فشل حفظ الإعدادات'),
  });

  const { register, handleSubmit, formState: { errors }, watch, setValue } = useForm<StoreFormData>({
    resolver: zodResolver(storeSchema),
    values: settings,
  });

  const onSubmit = (data: StoreFormData) => updateMutation.mutate(data);

  return (
    <div className="space-y-6">
      <h1 className="text-2xl font-bold">إعدادات المتجر</h1>
      <Tabs value={activeTab} onValueChange={setActiveTab}>
        <TabsList>
          <TabsTrigger value="general">عام</TabsTrigger>
          <TabsTrigger value="appearance">المظهر</TabsTrigger>
          <TabsTrigger value="hours">ساعات العمل</TabsTrigger>
          <TabsTrigger value="shipping">الشحن</TabsTrigger>
          <TabsTrigger value="policies">السياسات</TabsTrigger>
        </TabsList>

        <form onSubmit={handleSubmit(onSubmit)}>
          <TabsContent value="general" className="space-y-4">
            <div className="bg-white dark:bg-neutral-800 rounded-xl p-6 space-y-4">
              <Input label="اسم المتجر" {...register('name')} error={errors.name?.message} />
              <Textarea label="وصف المتجر" {...register('description')} rows={4} />
              <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
                <Input label="رقم الجوال" type="tel" {...register('phone')} error={errors.phone?.message} dir="ltr" />
                <Input label="البريد الإلكتروني" type="email" {...register('email')} error={errors.email?.message} dir="ltr" />
              </div>
              <Input label="العنوان" {...register('address')} error={errors.address?.message} />
              <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
                <Input label="المحافظة" {...register('governorate')} error={errors.governorate?.message} />
                <Input label="المنطقة" {...register('district')} error={errors.district?.message} />
              </div>
            </div>
          </TabsContent>

          <TabsContent value="appearance" className="space-y-4">
            <div className="bg-white dark:bg-neutral-800 rounded-xl p-6 space-y-4">
              <ImageUpload label="شعار المتجر" value={watch('logo')} onChange={(url) => setValue('logo', url)} aspectRatio={1} />
              <ImageUpload label="بانر المتجر" value={watch('banner')} onChange={(url) => setValue('banner', url)} aspectRatio={3} />
            </div>
          </TabsContent>

          <TabsContent value="hours" className="space-y-4">
            <div className="bg-white dark:bg-neutral-800 rounded-xl p-6 space-y-4">
              <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
                <TimePicker label="وقت الفتح" {...register('openingTime')} />
                <TimePicker label="وقت الإغلاق" {...register('closingTime')} />
              </div>
              <div>
                <label className="block text-sm font-medium mb-2">أيام العمل</label>
                <div className="flex flex-wrap gap-2">
                  {['السبت', 'الأحد', 'الاثنين', 'الثلاثاء', 'الأربعاء', 'الخميس', 'الجمعة'].map((day) => (
                    <label key={day} className="flex items-center gap-2">
                      <input type="checkbox" value={day} {...register('workingDays')} className="rounded" />
                      {day}
                    </label>
                  ))}
                </div>
              </div>
            </div>
          </TabsContent>

          <TabsContent value="shipping" className="space-y-4">
            <div className="bg-white dark:bg-neutral-800 rounded-xl p-6 space-y-4">
              <label className="flex items-center gap-3">
                <input type="checkbox" {...register('shippingEnabled')} className="rounded" />
                <span>تفعيل خدمة الشحن</span>
              </label>
              {watch('shippingEnabled') && (
                <Input label="الحد الأدنى للشحن المجاني (يمني)" type="number"
                  {...register('freeShippingThreshold', { valueAsNumber: true })} />
              )}
            </div>
          </TabsContent>

          <TabsContent value="policies" className="space-y-4">
            <div className="bg-white dark:bg-neutral-800 rounded-xl p-6">
              <Textarea label="سياسة الإرجاع والاستبدال" {...register('returnPolicy')} rows={6} />
            </div>
          </TabsContent>

          <div className="flex justify-end mt-6">
            <Button type="submit" loading={updateMutation.isLoading}>حفظ الإعدادات</Button>
          </div>
        </form>
      </Tabs>
    </div>
  );
}
```

### 6. Coupons Management (`/coupons`)

```typescript
// pages/Coupons.tsx
import { useState } from 'react';
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { DataTable } from '@/components/ui/DataTable';
import { Button } from '@/components/ui/Button';
import { Badge } from '@/components/ui/Badge';
import { Modal } from '@/components/ui/Modal';
import { CouponForm } from '@/components/coupons/CouponForm';
import { Permission } from '@/types/roles';
import { PermissionGuard } from '@/components/auth/PermissionGuard';
import { fetchCoupons, deleteCoupon } from '@/lib/api/vendor/coupons';
import { toast } from 'react-hot-toast';
import { Plus, Trash2, Edit, Copy } from 'lucide-react';

export function Coupons() {
  const queryClient = useQueryClient();
  const [isCreateModalOpen, setIsCreateModalOpen] = useState(false);
  const [editingCoupon, setEditingCoupon] = useState<any>(null);

  const { data, isLoading } = useQuery({
    queryKey: ['vendor-coupons'],
    queryFn: fetchCoupons,
  });

  const deleteMutation = useMutation({
    mutationFn: deleteCoupon,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['vendor-coupons'] });
      toast.success('تم حذف الكوبون بنجاح');
    },
  });

  const columns = [
    { accessorKey: 'code', header: 'الكود',
      cell: ({ row }) => (
        <div className="flex items-center gap-2">
          <code className="bg-neutral-100 dark:bg-neutral-700 px-2 py-1 rounded font-mono text-sm">
            {row.original.code}
          </code>
          <button onClick={() => { navigator.clipboard.writeText(row.original.code); toast.success('تم النسخ'); }}
            className="p-1 hover:bg-neutral-100 rounded">
            <Copy className="w-3 h-3" />
          </button>
        </div>
      ) },
    { accessorKey: 'type', header: 'النوع',
      cell: ({ row }) => (
        <Badge variant={row.original.type === 'percentage' ? 'info' : 'success'}>
          {row.original.type === 'percentage' ? 'نسبة مئوية' : 'مبلغ ثابت'}
        </Badge>
      ) },
    { accessorKey: 'value', header: 'القيمة',
      cell: ({ row }) => row.original.type === 'percentage' ? `${row.original.value}%`
        : `${row.original.value.toLocaleString('ar-YE')} يمني` },
    { accessorKey: 'usageCount', header: 'الاستخدام',
      cell: ({ row }) => `${row.original.usageCount}/${row.original.usageLimit || '∞'}` },
    { accessorKey: 'expiresAt', header: 'ينتهي في',
      cell: ({ row }) => new Date(row.original.expiresAt).toLocaleDateString('ar-YE') },
    { accessorKey: 'isActive', header: 'الحالة',
      cell: ({ row }) => (
        <Badge variant={row.original.isActive ? 'success' : 'neutral'}>
          {row.original.isActive ? 'نشط' : 'غير نشط'}
        </Badge>
      ) },
    { id: 'actions', header: 'الإجراءات',
      cell: ({ row }) => (
        <div className="flex items-center gap-2">
          <PermissionGuard permission={Permission.EDIT_COUPON}>
            <button onClick={() => setEditingCoupon(row.original)}
              className="p-2 hover:bg-neutral-100 rounded-lg"><Edit className="w-4 h-4" /></button>
          </PermissionGuard>
          <PermissionGuard permission={Permission.DELETE_COUPON}>
            <button onClick={() => { if (confirm('هل أنت متأكد من حذف هذا الكوبون؟')) deleteMutation.mutate(row.original.id); }}
              className="p-2 hover:bg-red-50 text-red-500 rounded-lg"><Trash2 className="w-4 h-4" /></button>
          </PermissionGuard>
        </div>
      ) },
  ];

  return (
    <div className="space-y-6">
      <div className="flex items-center justify-between">
        <h1 className="text-2xl font-bold">الكوبونات</h1>
        <PermissionGuard permission={Permission.CREATE_COUPON}>
          <Button onClick={() => setIsCreateModalOpen(true)}>
            <Plus className="w-4 h-4 ml-2" />إنشاء كوبون
          </Button>
        </PermissionGuard>
      </div>

      <DataTable data={data?.coupons || []} columns={columns} isLoading={isLoading}
        emptyMessage="لا توجد كوبونات" />

      <Modal isOpen={isCreateModalOpen} onClose={() => setIsCreateModalOpen(false)} title="إنشاء كوبون جديد">
        <CouponForm onSubmit={() => {
          queryClient.invalidateQueries({ queryKey: ['vendor-coupons'] });
          setIsCreateModalOpen(false);
        }} onCancel={() => setIsCreateModalOpen(false)} />
      </Modal>

      {editingCoupon && (
        <Modal isOpen={true} onClose={() => setEditingCoupon(null)} title="تعديل الكوبون">
          <CouponForm initialData={editingCoupon} onSubmit={() => {
            queryClient.invalidateQueries({ queryKey: ['vendor-coupons'] });
            setEditingCoupon(null);
          }} onCancel={() => setEditingCoupon(null)} />
        </Modal>
      )}
    </div>
  );
}
```

### 7. Analytics Page (`/analytics`)

```typescript
// pages/Analytics.tsx
import { useState } from 'react';
import { useQuery } from '@tanstack/react-query';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/Card';
import { Select } from '@/components/ui/Select';
import { DateRangePicker } from '@/components/ui/DateRangePicker';
import { SalesChart } from '@/components/analytics/SalesChart';
import { OrdersChart } from '@/components/analytics/OrdersChart';
import { ProductsChart } from '@/components/analytics/ProductsChart';
import { TopProductsTable } from '@/components/analytics/TopProductsTable';
import { RevenueMetrics } from '@/components/analytics/RevenueMetrics';
import { ConversionFunnel } from '@/components/analytics/ConversionFunnel';
import { Permission } from '@/types/roles';
import { PermissionGuard } from '@/components/auth/PermissionGuard';
import { fetchAnalytics } from '@/lib/api/vendor/analytics';
import { Button } from '@/components/ui/Button';
import { Download } from 'lucide-react';

export function Analytics() {
  const [dateRange, setDateRange] = useState<{ from: Date; to: Date }>({
    from: new Date(Date.now() - 30 * 24 * 60 * 60 * 1000),
    to: new Date(),
  });
  const [period, setPeriod] = useState('daily');

  const { data, isLoading } = useQuery({
    queryKey: ['vendor-analytics', { dateRange, period }],
    queryFn: () => fetchAnalytics({ dateRange, period }),
  });

  return (
    <div className="space-y-6">
      <div className="flex items-center justify-between">
        <h1 className="text-2xl font-bold">التحليلات</h1>
        <div className="flex items-center gap-3">
          <DateRangePicker value={dateRange} onChange={setDateRange} />
          <Select value={period} onChange={setPeriod} className="w-32"
            options={[{ value: 'daily', label: 'يومي' }, { value: 'weekly', label: 'أسبوعي' }, { value: 'monthly', label: 'شهري' }]} />
          <PermissionGuard permission={Permission.EXPORT_REPORTS}>
            <Button variant="outline"><Download className="w-4 h-4 ml-2" />تصدير التقرير</Button>
          </PermissionGuard>
        </div>
      </div>

      <RevenueMetrics data={data?.metrics} isLoading={isLoading} />

      <div className="grid grid-cols-1 lg:grid-cols-2 gap-6">
        <SalesChart data={data?.salesChart || []} />
        <OrdersChart data={data?.ordersChart || []} />
      </div>

      <div className="grid grid-cols-1 lg:grid-cols-2 gap-6">
        <ProductsChart data={data?.productsChart || []} />
        <Card>
          <CardHeader><CardTitle>قمع التحويل</CardTitle></CardHeader>
          <CardContent><ConversionFunnel data={data?.funnel} /></CardContent>
        </Card>
      </div>

      <Card>
        <CardHeader><CardTitle>المنتجات الأكثر مبيعاً</CardTitle></CardHeader>
        <CardContent><TopProductsTable products={data?.topProducts || []} /></CardContent>
      </Card>
    </div>
  );
}
```

### 8. KYC Management (`/kyc`)

```typescript
// pages/KYC.tsx
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/Card';
import { Input } from '@/components/ui/Input';
import { Button } from '@/components/ui/Button';
import { Badge } from '@/components/ui/Badge';
import { FileUpload } from '@/components/ui/FileUpload';
import { fetchKYC, submitKYC } from '@/lib/api/vendor/kyc';
import { toast } from 'react-hot-toast';
import { Shield, CheckCircle, Clock, XCircle, Upload } from 'lucide-react';

const kycSchema = z.object({
  businessName: z.string().min(2, 'اسم النشاط مطلوب'),
  businessType: z.enum(['individual', 'company', 'government']),
  registrationNumber: z.string().min(5, 'رقم السجل التجاري مطلوب'),
  taxId: z.string().min(10, 'الرقم الضريبي مطلوب'),
  idFront: z.string().url('صورة الهوية الأمامية مطلوبة'),
  idBack: z.string().url('صورة الهوية الخلفية مطلوبة'),
  commercialRegister: z.string().url().optional(),
  bankStatement: z.string().url().optional(),
});

type KYCFormData = z.infer<typeof kycSchema>;

export function KYC() {
  const queryClient = useQueryClient();
  const { data: kycData } = useQuery({ queryKey: ['vendor-kyc'], queryFn: fetchKYC });

  const submitMutation = useMutation({
    mutationFn: submitKYC,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['vendor-kyc'] });
      toast.success('تم تقديم طلب التحقق بنجاح');
    },
    onError: () => toast.error('فشل تقديم طلب التحقق'),
  });

  const { register, handleSubmit, formState: { errors }, watch, setValue } = useForm<KYCFormData>({
    resolver: zodResolver(kycSchema),
  });

  const onSubmit = (data: KYCFormData) => submitMutation.mutate(data);

  const getStatusBadge = (status: string) => {
    switch (status) {
      case 'verified': return <Badge variant="success"><CheckCircle className="w-4 h-4 ml-1" />موثق</Badge>;
      case 'pending': return <Badge variant="warning"><Clock className="w-4 h-4 ml-1" />قيد المراجعة</Badge>;
      case 'rejected': return <Badge variant="error"><XCircle className="w-4 h-4 ml-1" />مرفوض</Badge>;
      default: return <Badge variant="neutral">غير مقدم</Badge>;
    }
  };

  return (
    <div className="space-y-6">
      <div className="flex items-center justify-between">
        <h1 className="text-2xl font-bold">التحقق من الهوية (KYC)</h1>
        {kycData && getStatusBadge(kycData.status)}
      </div>

      {kycData?.status === 'pending' && (
        <Card className="border-yellow-200 bg-yellow-50 dark:bg-yellow-900/20">
          <CardContent className="flex items-center gap-3 py-4">
            <Clock className="w-5 h-5 text-yellow-600" />
            <p>طلبك قيد المراجعة. سيتم إشعارك خلال 2-3 أيام عمل.</p>
          </CardContent>
        </Card>
      )}

      {kycData?.status === 'rejected' && (
        <Card className="border-red-200 bg-red-50 dark:bg-red-900/20">
          <CardContent className="py-4">
            <div className="flex items-center gap-3 mb-2">
              <XCircle className="w-5 h-5 text-red-600" />
              <p className="font-medium">تم رفض طلب التحقق</p>
            </div>
            <p className="text-sm text-neutral-600">السبب: {kycData.rejectionReason}</p>
          </CardContent>
        </Card>
      )}

      <form onSubmit={handleSubmit(onSubmit)} className="space-y-6">
        <Card>
          <CardHeader>
            <CardTitle className="flex items-center gap-2">
              <Shield className="w-5 h-5" />معلومات النشاط التجاري
            </CardTitle>
          </CardHeader>
          <CardContent className="space-y-4">
            <Input label="اسم النشاط التجاري" {...register('businessName')} error={errors.businessName?.message} />
            <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
              <Input label="رقم السجل التجاري" {...register('registrationNumber')} error={errors.registrationNumber?.message} />
              <Input label="الرقم الضريبي" {...register('taxId')} error={errors.taxId?.message} />
            </div>
          </CardContent>
        </Card>

        <Card>
          <CardHeader>
            <CardTitle className="flex items-center gap-2">
              <Upload className="w-5 h-5" />المستندات المطلوبة
            </CardTitle>
          </CardHeader>
          <CardContent className="space-y-4">
            <FileUpload label="الهوية الوطنية - الأمام" accept="image/*"
              value={watch('idFront')} onChange={(url) => setValue('idFront', url)} error={errors.idFront?.message} />
            <FileUpload label="الهوية الوطنية - الخلف" accept="image/*"
              value={watch('idBack')} onChange={(url) => setValue('idBack', url)} error={errors.idBack?.message} />
            <FileUpload label="السجل التجاري (اختياري)" accept="image/*,.pdf"
              value={watch('commercialRegister')} onChange={(url) => setValue('commercialRegister', url)} />
            <FileUpload label="كشف الحساب البنكي (اختياري)" accept="image/*,.pdf"
              value={watch('bankStatement')} onChange={(url) => setValue('bankStatement', url)} />
          </CardContent>
        </Card>

        <div className="flex justify-end">
          <Button type="submit" loading={submitMutation.isLoading}
            disabled={kycData?.status === 'verified' || kycData?.status === 'pending'}>
            تقديم طلب التحقق
          </Button>
        </div>
      </form>
    </div>
  );
}
```

### 9. Team Management (`/team`)

```typescript
// pages/Team.tsx
import { useState } from 'react';
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { DataTable } from '@/components/ui/DataTable';
import { Button } from '@/components/ui/Button';
import { Badge } from '@/components/ui/Badge';
import { Modal } from '@/components/ui/Modal';
import { InviteMemberForm } from '@/components/team/InviteMemberForm';
import { Permission } from '@/types/roles';
import { PermissionGuard } from '@/components/auth/PermissionGuard';
import { VendorRole } from '@/types/roles';
import { fetchTeamMembers, removeTeamMember, updateMemberRole } from '@/lib/api/vendor/team';
import { toast } from 'react-hot-toast';
import { UserPlus, Trash2, Shield } from 'lucide-react';

export function Team() {
  const queryClient = useQueryClient();
  const [isInviteModalOpen, setIsInviteModalOpen] = useState(false);
  const [updatingMember, setUpdatingMember] = useState<any>(null);

  const { data: members, isLoading } = useQuery({
    queryKey: ['vendor-team'],
    queryFn: fetchTeamMembers,
  });

  const removeMutation = useMutation({
    mutationFn: removeTeamMember,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['vendor-team'] });
      toast.success('تم إزالة العضو بنجاح');
    },
  });

  const roleMutation = useMutation({
    mutationFn: ({ memberId, role }: { memberId: string; role: VendorRole }) =>
      updateMemberRole(memberId, role),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['vendor-team'] });
      toast.success('تم تحديث الصلاحية بنجاح');
      setUpdatingMember(null);
    },
  });

  const columns = [
    { accessorKey: 'name', header: 'الاسم',
      cell: ({ row }) => (
        <div className="flex items-center gap-3">
          <div className="w-10 h-10 rounded-full bg-primary-100 dark:bg-primary-900 flex items-center justify-center text-primary-600 font-medium">
            {row.original.name.charAt(0)}
          </div>
          <div>
            <p className="font-medium">{row.original.name}</p>
            <p className="text-sm text-neutral-500">{row.original.email}</p>
          </div>
        </div>
      ) },
    { accessorKey: 'phone', header: 'الجوال', cell: ({ row }) => row.original.phone },
    { accessorKey: 'role', header: 'الصلاحية',
      cell: ({ row }) => (
        <Badge variant={row.original.role === VendorRole.OWNER ? 'primary' : 'neutral'}>
          <Shield className="w-3 h-3 ml-1" />
          {row.original.role === VendorRole.OWNER ? 'المالك' : 'موظف'}
        </Badge>
      ) },
    { accessorKey: 'status', header: 'الحالة',
      cell: ({ row }) => (
        <Badge variant={row.original.isActive ? 'success' : 'neutral'}>
          {row.original.isActive ? 'نشط' : 'غير نشط'}
        </Badge>
      ) },
    { id: 'actions', header: 'الإجراءات',
      cell: ({ row }) => {
        if (row.original.role === VendorRole.OWNER) return null;
        return (
          <div className="flex items-center gap-2">
            <PermissionGuard permission={Permission.EDIT_TEAM_MEMBER_ROLE}>
              <button onClick={() => setUpdatingMember(row.original)}
                className="p-2 hover:bg-neutral-100 rounded-lg"><Shield className="w-4 h-4" /></button>
            </PermissionGuard>
            <PermissionGuard permission={Permission.REMOVE_TEAM_MEMBER}>
              <button onClick={() => { if (confirm('هل أنت متأكد من إزالة هذا العضو؟')) removeMutation.mutate(row.original.id); }}
                className="p-2 hover:bg-red-50 text-red-500 rounded-lg"><Trash2 className="w-4 h-4" /></button>
            </PermissionGuard>
          </div>
        );
      } },
  ];

  return (
    <div className="space-y-6">
      <div className="flex items-center justify-between">
        <div>
          <h1 className="text-2xl font-bold">فريق العمل</h1>
          <p className="text-neutral-500">إدارة أعضاء فريق العمل وصلاحياتهم</p>
        </div>
        <PermissionGuard permission={Permission.INVITE_TEAM_MEMBER}>
          <Button onClick={() => setIsInviteModalOpen(true)}>
            <UserPlus className="w-4 h-4 ml-2" />دعوة عضو
          </Button>
        </PermissionGuard>
      </div>

      <DataTable data={members || []} columns={columns} isLoading={isLoading}
        emptyMessage="لا يوجد أعضاء في الفريق" />

      <Modal isOpen={isInviteModalOpen} onClose={() => setIsInviteModalOpen(false)} title="دعوة عضو جديد">
        <InviteMemberForm onSubmit={() => {
          queryClient.invalidateQueries({ queryKey: ['vendor-team'] });
          setIsInviteModalOpen(false);
        }} onCancel={() => setIsInviteModalOpen(false)} />
      </Modal>

      {updatingMember && (
        <Modal isOpen={true} onClose={() => setUpdatingMember(null)} title="تغيير الصلاحية">
          <div className="space-y-4">
            <p>تغيير صلاحية <span className="font-medium">{updatingMember.name}</span></p>
            <Button variant={updatingMember.role === VendorRole.STAFF ? 'primary' : 'outline'}
              onClick={() => roleMutation.mutate({ memberId: updatingMember.id, role: VendorRole.STAFF })}>
              موظف
            </Button>
          </div>
        </Modal>
      )}
    </div>
  );
}
```

## Layout & Navigation

### Vendor Layout

```typescript
// layouts/VendorLayout.tsx
import { useState } from 'react';
import { Outlet } from 'react-router-dom';
import { Sidebar } from '@/components/layout/Sidebar';
import { TopBar } from '@/components/layout/TopBar';
import { NotificationPanel } from '@/components/notifications/NotificationPanel';
import { useNotifications } from '@/hooks/useNotifications';

export function VendorLayout() {
  const [sidebarOpen, setSidebarOpen] = useState(true);
  const [notificationsOpen, setNotificationsOpen] = useState(false);
  const { unreadCount } = useNotifications();

  return (
    <div className="flex h-screen bg-neutral-50 dark:bg-neutral-900">
      <Sidebar isOpen={sidebarOpen} onToggle={() => setSidebarOpen(!sidebarOpen)} />
      <div className="flex-1 flex flex-col overflow-hidden">
        <TopBar onMenuToggle={() => setSidebarOpen(!sidebarOpen)}
          onNotificationsToggle={() => setNotificationsOpen(!notificationsOpen)}
          unreadNotifications={unreadCount} />
        <main className="flex-1 overflow-y-auto p-6">
          <Outlet />
        </main>
      </div>
      <NotificationPanel isOpen={notificationsOpen} onClose={() => setNotificationsOpen(false)} />
    </div>
  );
}
```

### Sidebar Navigation

```typescript
// components/layout/Sidebar.tsx
import { NavLink, useLocation } from 'react-router-dom';
import { usePermissions } from '@/hooks/usePermissions';
import { Permission } from '@/types/roles';
import { PermissionGuard } from '@/components/auth/PermissionGuard';
import {
  LayoutDashboard, Package, ShoppingCart, Warehouse, Store,
  Tag, BarChart3, Shield, Users, Settings, ChevronLeft, ChevronRight,
} from 'lucide-react';

interface SidebarProps { isOpen: boolean; onToggle: () => void; }

const navigation = [
  { name: 'لوحة التحكم', href: '/dashboard', icon: LayoutDashboard, permission: null },
  { name: 'المنتجات', href: '/products', icon: Package, permission: Permission.VIEW_PRODUCTS },
  { name: 'الطلبات', href: '/orders', icon: ShoppingCart, permission: Permission.VIEW_ORDERS },
  { name: 'المخزون', href: '/inventory', icon: Warehouse, permission: Permission.MANAGE_INVENTORY },
  { name: 'إعدادات المتجر', href: '/settings', icon: Store, permission: Permission.EDIT_STORE },
  { name: 'الكوبونات', href: '/coupons', icon: Tag, permission: Permission.VIEW_COUPONS },
  { name: 'التحليلات', href: '/analytics', icon: BarChart3, permission: Permission.VIEW_STORE_ANALYTICS },
  { name: 'التحقق من الهوية', href: '/kyc', icon: Shield, permission: Permission.VIEW_KYC },
  { name: 'فريق العمل', href: '/team', icon: Users, permission: Permission.VIEW_TEAM },
  { name: 'الإعدادات', href: '/account', icon: Settings, permission: null },
];

export function Sidebar({ isOpen, onToggle }: SidebarProps) {
  const location = useLocation();

  return (
    <aside className={`${isOpen ? 'w-64' : 'w-16'} bg-white dark:bg-neutral-800
                       border-l border-neutral-200 dark:border-neutral-700
                       transition-all duration-300 flex flex-col`}>
      <div className="flex items-center justify-between p-4 border-b border-neutral-200 dark:border-neutral-700">
        {isOpen && <span className="text-xl font-bold">Vendor Panel</span>}
        <button onClick={onToggle} className="p-2 rounded-lg hover:bg-neutral-100 dark:hover:bg-neutral-700">
          {isOpen ? <ChevronRight className="w-5 h-5" /> : <ChevronLeft className="w-5 h-5" />}
        </button>
      </div>

      <nav className="flex-1 p-2 space-y-1">
        {navigation.map((item) => (
          <PermissionGuard key={item.href} permission={item.permission}>
            <NavLink to={item.href}
              className={({ isActive }) =>
                `flex items-center gap-3 px-3 py-2 rounded-lg transition-colors ${
                  isActive ? 'bg-primary-50 dark:bg-primary-900/20 text-primary-600' : 'hover:bg-neutral-100 dark:hover:bg-neutral-700'
                }`
              }>
              <item.icon className="w-5 h-5 shrink-0" />
              {isOpen && <span className="text-sm font-medium">{item.name}</span>}
            </NavLink>
          </PermissionGuard>
        ))}
      </nav>
    </aside>
  );
}
```

### TopBar

```typescript
// components/layout/TopBar.tsx
import { Bell, Menu, Search, User, LogOut } from 'lucide-react';
import { useAuthStore } from '@/stores/authStore';
import { LanguageToggle } from '@/components/LanguageToggle';
import { ThemeToggle } from '@/components/ThemeToggle';

interface TopBarProps {
  onMenuToggle: () => void;
  onNotificationsToggle: () => void;
  unreadNotifications: number;
}

export function TopBar({ onMenuToggle, onNotificationsToggle, unreadNotifications }: TopBarProps) {
  const { user, logout } = useAuthStore();

  return (
    <header className="h-16 bg-white dark:bg-neutral-800 border-b border-neutral-200 dark:border-neutral-700 flex items-center justify-between px-4">
      <div className="flex items-center gap-4">
        <button onClick={onMenuToggle} className="p-2 rounded-lg hover:bg-neutral-100 dark:hover:bg-neutral-700 md:hidden">
          <Menu className="w-5 h-5" />
        </button>
        <div className="relative hidden md:block">
          <Search className="absolute left-3 top-1/2 -translate-y-1/2 w-4 h-4 text-neutral-400" />
          <input type="search" placeholder="بحث..."
            className="pl-10 pr-4 py-2 rounded-lg border border-neutral-200 dark:border-neutral-700 bg-neutral-50 dark:bg-neutral-900 focus:outline-none focus:ring-2 focus:ring-primary-500 w-64" />
        </div>
      </div>

      <div className="flex items-center gap-2">
        <LanguageToggle />
        <ThemeToggle />
        <button onClick={onNotificationsToggle} className="relative p-2 rounded-lg hover:bg-neutral-100 dark:hover:bg-neutral-700">
          <Bell className="w-5 h-5" />
          {unreadNotifications > 0 && (
            <span className="absolute -top-0.5 -right-0.5 w-4 h-4 bg-red-500 rounded-full text-white text-xs flex items-center justify-center">
              {unreadNotifications}
            </span>
          )}
        </button>
        <div className="flex items-center gap-2 px-3 py-2">
          <div className="w-8 h-8 rounded-full bg-primary-100 dark:bg-primary-900 flex items-center justify-center">
            <User className="w-4 h-4 text-primary-600" />
          </div>
          <div className="hidden md:block">
            <p className="text-sm font-medium">{user?.name}</p>
            <p className="text-xs text-neutral-500">{user?.role === 'vendor_owner' ? 'المالك' : 'موظف'}</p>
          </div>
        </div>
        <button onClick={logout} className="p-2 rounded-lg hover:bg-neutral-100 dark:hover:bg-neutral-700 text-red-500">
          <LogOut className="w-5 h-5" />
        </button>
      </div>
    </header>
  );
}
```

## Real-time Notifications

```typescript
// hooks/useNotifications.ts
import { useEffect, useState } from 'react';
import { useQueryClient } from '@tanstack/react-query';
import { useAuthStore } from '@/stores/authStore';

export function useNotifications() {
  const [notifications, setNotifications] = useState<any[]>([]);
  const { user } = useAuthStore();
  const queryClient = useQueryClient();

  useEffect(() => {
    if (!user) return;

    const eventSource = new EventSource(`/api/notifications/stream?vendorId=${user.vendorId}`);

    eventSource.onmessage = (event) => {
      const data = JSON.parse(event.data);
      setNotifications((prev) => [data, ...prev]);

      // Invalidate relevant queries based on notification type
      if (data.type === 'new_order') {
        queryClient.invalidateQueries({ queryKey: ['vendor-orders'] });
        queryClient.invalidateQueries({ queryKey: ['vendor-dashboard-stats'] });
      } else if (data.type === 'product_review') {
        queryClient.invalidateQueries({ queryKey: ['vendor-products'] });
      }
    };

    eventSource.onerror = () => {
      eventSource.close();
      // Reconnect after 5 seconds
      setTimeout(() => {
        // Reconnect logic
      }, 5000);
    };

    return () => eventSource.close();
  }, [user, queryClient]);

  const unreadCount = notifications.filter((n) => !n.read).length;

  const markAsRead = async (id: string) => {
    await fetch(`/api/notifications/${id}/read`, { method: 'PUT' });
    setNotifications((prev) =>
      prev.map((n) => (n.id === id ? { ...n, read: true } : n))
    );
  };

  const markAllAsRead = async () => {
    await fetch('/api/notifications/read-all', { method: 'PUT' });
    setNotifications((prev) => prev.map((n) => ({ ...n, read: true })));
  };

  return { notifications, unreadCount, markAsRead, markAllAsRead };
}
```

## State Management

```typescript
// stores/authStore.ts
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

interface VendorUser {
  id: string;
  name: string;
  email: string;
  phone: string;
  role: string;
  vendorId: string;
  storeName: string;
  avatar?: string;
}

interface AuthState {
  user: VendorUser | null;
  token: string | null;
  isAuthenticated: boolean;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
  setUser: (user: VendorUser) => void;
}

export const useAuthStore = create<AuthState>()(
  persist(
    (set) => ({
      user: null,
      token: null,
      isAuthenticated: false,

      login: async (email: string, password: string) => {
        const response = await fetch('/api/vendor/auth/login', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ email, password }),
        });
        const data = await response.json();
        set({ user: data.user, token: data.token, isAuthenticated: true });
      },

      logout: () => {
        set({ user: null, token: null, isAuthenticated: false });
        window.location.href = '/login';
      },

      setUser: (user) => set({ user }),
    }),
    { name: 'vendor-auth' }
  )
);
```

## Project Structure

```
vendor-panel/
├── src/
│   ├── pages/
│   │   ├── Dashboard.tsx
│   │   ├── Products.tsx
│   │   ├── Orders.tsx
│   │   ├── Inventory.tsx
│   │   ├── StoreSettings.tsx
│   │   ├── Coupons.tsx
│   │   ├── Analytics.tsx
│   │   ├── KYC.tsx
│   │   ├── Team.tsx
│   │   └── Login.tsx
│   ├── layouts/
│   │   └── VendorLayout.tsx
│   ├── components/
│   │   ├── layout/
│   │   │   ├── Sidebar.tsx
│   │   │   └── TopBar.tsx
│   │   ├── dashboard/
│   │   │   ├── StatsCard.tsx
│   │   │   ├── RecentOrders.tsx
│   │   │   ├── SalesChart.tsx
│   │   │   ├── TopProducts.tsx
│   │   │   ├── RevenueChart.tsx
│   │   │   ├── NotificationsList.tsx
│   │   │   └── QuickActions.tsx
│   │   ├── products/
│   │   │   ├── ProductForm.tsx
│   │   │   └── ProductDeleteDialog.tsx
│   │   ├── orders/
│   │   │   ├── OrderDetailDrawer.tsx
│   │   │   └── OrderStatusUpdate.tsx
│   │   ├── inventory/
│   │   │   ├── BulkStockUpdate.tsx
│   │   │   └── StockHistory.tsx
│   │   ├── coupons/
│   │   │   └── CouponForm.tsx
│   │   ├── analytics/
│   │   │   ├── SalesChart.tsx
│   │   │   ├── OrdersChart.tsx
│   │   │   ├── ProductsChart.tsx
│   │   │   ├── TopProductsTable.tsx
│   │   │   ├── RevenueMetrics.tsx
│   │   │   └── ConversionFunnel.tsx
│   │   ├── team/
│   │   │   └── InviteMemberForm.tsx
│   │   ├── notifications/
│   │   │   └── NotificationPanel.tsx
│   │   ├── auth/
│   │   │   └── PermissionGuard.tsx
│   │   └── ui/
│   │       ├── Button.tsx
│   │       ├── Input.tsx
│   │       ├── Select.tsx
│   │       ├── Modal.tsx
│   │       ├── DataTable.tsx
│   │       ├── Badge.tsx
│   │       ├── Card.tsx
│   │       ├── Tabs.tsx
│   │       ├── Textarea.tsx
│   │       ├── ImageUpload.tsx
│   │       ├── FileUpload.tsx
│   │       ├── TimePicker.tsx
│   │       └── DateRangePicker.tsx
│   ├── stores/
│   │   └── authStore.ts
│   ├── hooks/
│   │   ├── usePermissions.ts
│   │   └── useNotifications.ts
│   ├── types/
│   │   └── roles.ts
│   ├── lib/
│   │   └── api/
│   │       └── vendor/
│   │           ├── products.ts
│   │           ├── orders.ts
│   │           ├── inventory.ts
│   │           ├── store.ts
│   │           ├── coupons.ts
│   │           ├── analytics.ts
│   │           ├── kyc.ts
│   │           └── team.ts
│   ├── App.tsx
│   ├── main.tsx
│   └── vite-env.d.ts
├── index.html
├── package.json
├── tsconfig.json
├── vite.config.ts
├── tailwind.config.ts
└── .env.local
```

## Testing

```typescript
// __tests__/pages/Products.test.tsx
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { Products } from '@/pages/Products';
import { server } from '@/mocks/server';
import { http, HttpResponse } from 'msw';

const queryClient = new QueryClient({
  defaultOptions: { queries: { retry: false } },
});

const wrapper = ({ children }: { children: React.ReactNode }) => (
  <QueryClientProvider client={queryClient}>{children}</QueryClientProvider>
);

describe('Products Page', () => {
  it('renders products list', async () => {
    render(<Products />, { wrapper });
    expect(screen.getByText('المنتجات')).toBeInTheDocument();
    await waitFor(() => {
      expect(screen.getByText('iPhone 15')).toBeInTheDocument();
    });
  });

  it('opens create modal', async () => {
    render(<Products />, { wrapper });
    await userEvent.click(screen.getByText('إضافة منتج'));
    expect(screen.getByText('إضافة منتج جديد')).toBeInTheDocument();
  });

  it('filters products by search', async () => {
    render(<Products />, { wrapper });
    await userEvent.type(screen.getByPlaceholderText('بحث عن منتج...'), 'iPhone');
    await waitFor(() => {
      expect(screen.getByText('iPhone 15')).toBeInTheDocument();
    });
  });
});
```
