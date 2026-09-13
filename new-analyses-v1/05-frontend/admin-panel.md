# Admin Panel - YemenMart

## Overview

Admin management portal built with React 18 and Vite. Provides super-admins and support staff with full CRUD capabilities across all entities (customers, vendors, orders, products), financial management, coupon management, content management, system settings, support ticketing, security monitoring, and ZATCA invoice compliance. Arabic-first with RTL/LTR support.

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

## RBAC System

```typescript
// types/admin-roles.ts
export enum AdminRole {
  SUPER_ADMIN = 'super_admin',
  SUPPORT_ADMIN = 'support_admin',
  FINANCE_ADMIN = 'finance_admin',
  CONTENT_ADMIN = 'content_admin',
}

export enum AdminPermission {
  // Customers
  VIEW_CUSTOMERS = 'customers:view',
  EDIT_CUSTOMERS = 'customers:edit',
  BLOCK_CUSTOMERS = 'customers:block',
  DELETE_CUSTOMERS = 'customers:delete',

  // Vendors
  VIEW_VENDORS = 'vendors:view',
  APPROVE_VENDORS = 'vendors:approve',
  SUSPEND_VENDORS = 'vendors:suspend',
  DELETE_VENDORS = 'vendors:delete',
  VIEW_VENDOR_KYC = 'vendors:kyc:view',
  APPROVE_VENDOR_KYC = 'vendors:kyc:approve',

  // Orders
  VIEW_ORDERS = 'orders:view',
  EDIT_ORDERS = 'orders:edit',
  CANCEL_ORDERS = 'orders:cancel',
  ISSUE_REFUNDS = 'orders:refunds:issue',
  VIEW_ORDER_DETAILS = 'orders:details:view',

  // Products
  VIEW_PRODUCTS = 'products:view',
  APPROVE_PRODUCTS = 'products:approve',
  REJECT_PRODUCTS = 'products:reject',
  DELETE_PRODUCTS = 'products:delete',

  // Finance
  VIEW_FINANCE = 'finance:view',
  VIEW_TRANSACTIONS = 'finance:transactions:view',
  PROCESS_PAYOUTS = 'finance:payouts:process',
  VIEW_REPORTS = 'reports:view',
  EXPORT_REPORTS = 'reports:export',

  // Coupons
  VIEW_COUPONS = 'coupons:view',
  CREATE_COUPON = 'coupons:create',
  EDIT_COUPON = 'coupons:edit',
  DELETE_COUPON = 'coupons:delete',

  // Content
  VIEW_CONTENT = 'content:view',
  EDIT_BANNERS = 'content:banners:edit',
  EDIT_PAGES = 'content:pages:edit',
  MANAGE_CATEGORIES = 'content:categories:manage',

  // Settings
  VIEW_SETTINGS = 'settings:view',
  EDIT_SETTINGS = 'settings:edit',

  // Support
  VIEW_TICKETS = 'support:tickets:view',
  RESPOND_TICKETS = 'support:tickets:respond',
  CLOSE_TICKETS = 'support:tickets:close',

  // Security
  VIEW_AUDIT_LOG = 'security:audit:view',
  VIEW_SECURITY_EVENTS = 'security:events:view',
  MANAGE_RESTRICTIONS = 'security:restrictions:manage',

  // ZATCA
  VIEW_ZATCA = 'zatca:view',
  MANAGE_ZATCA = 'zatca:manage',
}

export const ADMIN_ROLE_PERMISSIONS: Record<AdminRole, AdminPermission[]> = {
  [AdminRole.SUPER_ADMIN]: Object.values(AdminPermission),
  [AdminRole.SUPPORT_ADMIN]: [
    AdminPermission.VIEW_CUSTOMERS,
    AdminPermission.EDIT_CUSTOMERS,
    AdminPermission.BLOCK_CUSTOMERS,
    AdminPermission.VIEW_VENDORS,
    AdminPermission.VIEW_ORDERS,
    AdminPermission.VIEW_ORDER_DETAILS,
    AdminPermission.EDIT_ORDERS,
    AdminPermission.VIEW_TICKETS,
    AdminPermission.RESPOND_TICKETS,
    AdminPermission.CLOSE_TICKETS,
    AdminPermission.VIEW_AUDIT_LOG,
  ],
  [AdminRole.FINANCE_ADMIN]: [
    AdminPermission.VIEW_CUSTOMERS,
    AdminPermission.VIEW_VENDORS,
    AdminPermission.VIEW_ORDERS,
    AdminPermission.VIEW_ORDER_DETAILS,
    AdminPermission.VIEW_FINANCE,
    AdminPermission.VIEW_TRANSACTIONS,
    AdminPermission.PROCESS_PAYOUTS,
    AdminPermission.VIEW_REPORTS,
    AdminPermission.EXPORT_REPORTS,
    AdminPermission.VIEW_ZATCA,
    AdminPermission.MANAGE_ZATCA,
  ],
  [AdminRole.CONTENT_ADMIN]: [
    AdminPermission.VIEW_CONTENT,
    AdminPermission.EDIT_BANNERS,
    AdminPermission.EDIT_PAGES,
    AdminPermission.MANAGE_CATEGORIES,
    AdminPermission.VIEW_PRODUCTS,
    AdminPermission.APPROVE_PRODUCTS,
    AdminPermission.REJECT_PRODUCTS,
    AdminPermission.VIEW_COUPONS,
    AdminPermission.CREATE_COUPON,
    AdminPermission.EDIT_COUPON,
  ],
};
```

### Admin Permission Hook

```typescript
// hooks/useAdminPermissions.ts
import { useAuthStore } from '@/stores/adminAuthStore';
import { AdminRole, AdminPermission, ADMIN_ROLE_PERMISSIONS } from '@/types/admin-roles';

export function useAdminPermissions() {
  const { user } = useAuthStore();

  const hasPermission = (permission: AdminPermission): boolean => {
    if (!user) return false;
    if (user.role === AdminRole.SUPER_ADMIN) return true;
    const perms = ADMIN_ROLE_PERMISSIONS[user.role as AdminRole] || [];
    return perms.includes(permission);
  };

  return {
    role: user?.role as AdminRole | undefined,
    hasPermission,
    isSuperAdmin: user?.role === AdminRole.SUPER_ADMIN,
    isSupportAdmin: user?.role === AdminRole.SUPPORT_ADMIN,
    isFinanceAdmin: user?.role === AdminRole.FINANCE_ADMIN,
    isContentAdmin: user?.role === AdminRole.CONTENT_ADMIN,
  };
}
```

## Pages

### 1. Admin Dashboard (`/admin/dashboard`)

```typescript
// pages/admin/Dashboard.tsx
import { useQuery } from '@tanstack/react-query';
import { StatsCard } from '@/components/admin/dashboard/StatsCard';
import { RevenueOverview } from '@/components/admin/dashboard/RevenueOverview';
import { OrdersChart } from '@/components/admin/dashboard/OrdersChart';
import { VendorActivity } from '@/components/admin/dashboard/VendorActivity';
import { CustomerGrowth } from '@/components/admin/dashboard/CustomerGrowth';
import { RecentActivity } from '@/components/admin/dashboard/RecentActivity';
import { SystemAlerts } from '@/components/admin/dashboard/SystemAlerts';
import { fetchAdminDashboardStats } from '@/lib/api/admin/dashboard';
import {
  Users, Store, ShoppingCart, DollarSign, TrendingUp, AlertTriangle,
} from 'lucide-react';

export function AdminDashboard() {
  const { data: stats, isLoading } = useQuery({
    queryKey: ['admin-dashboard-stats'],
    queryFn: fetchAdminDashboardStats,
    refetchInterval: 30000,
  });

  return (
    <div className="space-y-6">
      <div>
        <h1 className="text-2xl font-bold">لوحة تحكم الإدارة</h1>
        <p className="text-neutral-500">نظرة عامة على منصة YemenMart</p>
      </div>

      <SystemAlerts alerts={stats?.systemAlerts || []} />

      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4">
        <StatsCard title="الإجمالي المستلم" value={stats?.totalRevenue || 0} currency="YER"
          icon={DollarSign} change={stats?.revenueChange} loading={isLoading} />
        <StatsCard title="العملاء النشطون" value={stats?.activeCustomers || 0} icon={Users}
          change={stats?.customersChange} loading={isLoading} />
        <StatsCard title="المتاجر النشطة" value={stats?.activeVendors || 0} icon={Store}
          change={stats?.vendorsChange} loading={isLoading} />
        <StatsCard title="الطلبات هذا الشهر" value={stats?.monthlyOrders || 0} icon={ShoppingCart}
          change={stats?.ordersChange} loading={isLoading} />
      </div>

      <div className="grid grid-cols-1 lg:grid-cols-2 gap-6">
        <RevenueOverview data={stats?.revenueOverview || []} />
        <OrdersChart data={stats?.ordersChart || []} />
      </div>

      <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
        <div className="lg:col-span-2">
          <CustomerGrowth data={stats?.customerGrowth || []} />
        </div>
        <div className="space-y-6">
          <VendorActivity vendors={stats?.recentVendorActivity || []} />
          <RecentActivity activities={stats?.recentActivity || []} />
        </div>
      </div>
    </div>
  );
}
```

### 2. Customer Management (`/admin/customers`)

```typescript
// pages/admin/Customers.tsx
import { useState } from 'react';
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { DataTable } from '@/components/ui/DataTable';
import { Button } from '@/components/ui/Button';
import { Input } from '@/components/ui/Input';
import { Badge } from '@/components/ui/Badge';
import { Modal } from '@/components/ui/Modal';
import { CustomerDetailDrawer } from '@/components/admin/customers/CustomerDetailDrawer';
import { AdminPermission } from '@/types/admin-roles';
import { PermissionGuard } from '@/components/admin/auth/PermissionGuard';
import { fetchCustomers, blockCustomer, unblockCustomer, deleteCustomer } from '@/lib/api/admin/customers';
import { toast } from 'react-hot-toast';
import { Search, Eye, Ban, CheckCircle, Trash2, Download } from 'lucide-react';

export function AdminCustomers() {
  const queryClient = useQueryClient();
  const [search, setSearch] = useState('');
  const [status, setStatus] = useState('');
  const [selectedCustomer, setSelectedCustomer] = useState<any>(null);

  const { data, isLoading } = useQuery({
    queryKey: ['admin-customers', { search, status }],
    queryFn: () => fetchCustomers({ search, status }),
  });

  const blockMutation = useMutation({
    mutationFn: blockCustomer,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['admin-customers'] });
      toast.success('تم حظر العميل بنجاح');
    },
  });

  const unblockMutation = useMutation({
    mutationFn: unblockCustomer,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['admin-customers'] });
      toast.success('تم فك حظر العميل بنجاح');
    },
  });

  const deleteMutation = useMutation({
    mutationFn: deleteCustomer,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['admin-customers'] });
      toast.success('تم حذف العميل بنجاح');
    },
  });

  const columns = [
    { accessorKey: 'name', header: 'الاسم',
      cell: ({ row }) => (
        <div>
          <p className="font-medium">{row.original.name}</p>
          <p className="text-sm text-neutral-500">{row.original.email}</p>
        </div>
      ) },
    { accessorKey: 'phone', header: 'الجوال', cell: ({ row }) => row.original.phone },
    { accessorKey: 'orders', header: 'الطلبات',
      cell: ({ row }) => <span>{row.original.orderCount} طلب</span> },
    { accessorKey: 'totalSpent', header: 'إجمالي الشراء',
      cell: ({ row }) => (
        <span className="font-medium">{row.original.totalSpent.toLocaleString('ar-YE')} يمني</span>
      ) },
    { accessorKey: 'walletBalance', header: 'المحفظة',
      cell: ({ row }) => `${row.original.walletBalance.toLocaleString('ar-YE')} يمني` },
    { accessorKey: 'status', header: 'الحالة',
      cell: ({ row }) => (
        <Badge variant={row.original.isBlocked ? 'error' : 'success'}>
          {row.original.isBlocked ? 'محظور' : 'نشط'}
        </Badge>
      ) },
    { accessorKey: 'createdAt', header: 'تاريخ التسجيل',
      cell: ({ row }) => new Date(row.original.createdAt).toLocaleDateString('ar-YE') },
    { id: 'actions', header: 'الإجراءات',
      cell: ({ row }) => (
        <div className="flex items-center gap-1">
          <button onClick={() => setSelectedCustomer(row.original)}
            className="p-2 hover:bg-neutral-100 rounded-lg">
            <Eye className="w-4 h-4" />
          </button>
          <PermissionGuard permission={AdminPermission.BLOCK_CUSTOMERS}>
            {row.original.isBlocked ? (
              <button onClick={() => unblockMutation.mutate(row.original.id)}
                className="p-2 hover:bg-green-50 text-green-500 rounded-lg">
                <CheckCircle className="w-4 h-4" />
              </button>
            ) : (
              <button onClick={() => blockMutation.mutate(row.original.id)}
                className="p-2 hover:bg-red-50 text-red-500 rounded-lg">
                <Ban className="w-4 h-4" />
              </button>
            )}
          </PermissionGuard>
          <PermissionGuard permission={AdminPermission.DELETE_CUSTOMERS}>
            <button onClick={() => { if (confirm('هل أنت متأكد من حذف هذا العميل؟')) deleteMutation.mutate(row.original.id); }}
              className="p-2 hover:bg-red-50 text-red-500 rounded-lg">
              <Trash2 className="w-4 h-4" />
            </button>
          </PermissionGuard>
        </div>
      ) },
  ];

  return (
    <div className="space-y-6">
      <div className="flex items-center justify-between">
        <h1 className="text-2xl font-bold">العملاء</h1>
        <Button variant="outline"><Download className="w-4 h-4 ml-2" />تصدير</Button>
      </div>

      <div className="flex flex-wrap items-center gap-3">
        <Input placeholder="بحث بالاسم، البريد، أو الجوال..." value={search}
          onChange={(e) => setSearch(e.target.value)}
          icon={<Search className="w-4 h-4" />} className="max-w-sm" />
        <select value={status} onChange={(e) => setStatus(e.target.value)}
          className="px-4 py-2 rounded-lg border border-neutral-200 dark:border-neutral-700">
          <option value="">جميع الحالات</option>
          <option value="active">نشط</option>
          <option value="blocked">محظور</option>
        </select>
      </div>

      <DataTable data={data?.customers || []} columns={columns} isLoading={isLoading}
        emptyMessage="لا يوجد عملاء" pagination={data?.pagination} />

      {selectedCustomer && (
        <CustomerDetailDrawer customer={selectedCustomer}
          onClose={() => setSelectedCustomer(null)} />
      )}
    </div>
  );
}
```

### 3. Vendor Management (`/admin/vendors`)

```typescript
// pages/admin/Vendors.tsx
import { useState } from 'react';
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { DataTable } from '@/components/ui/DataTable';
import { Button } from '@/components/ui/Button';
import { Input } from '@/components/ui/Input';
import { Badge } from '@/components/ui/Badge';
import { VendorDetailDrawer } from '@/components/admin/vendors/VendorDetailDrawer';
import { KYCReviewModal } from '@/components/admin/vendors/KYCReviewModal';
import { AdminPermission } from '@/types/admin-roles';
import { PermissionGuard } from '@/components/admin/auth/PermissionGuard';
import { fetchVendors, approveVendor, suspendVendor } from '@/lib/api/admin/vendors';
import { toast } from 'react-hot-toast';
import { Search, Eye, CheckCircle, XCircle, Shield, Download } from 'lucide-react';

export function AdminVendors() {
  const queryClient = useQueryClient();
  const [search, setSearch] = useState('');
  const [status, setStatus] = useState('');
  const [kycStatus, setKycStatus] = useState('');
  const [selectedVendor, setSelectedVendor] = useState<any>(null);
  const [reviewingKYC, setReviewingKYC] = useState<any>(null);

  const { data, isLoading } = useQuery({
    queryKey: ['admin-vendors', { search, status, kycStatus }],
    queryFn: () => fetchVendors({ search, status, kycStatus }),
  });

  const suspendMutation = useMutation({
    mutationFn: suspendVendor,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['admin-vendors'] });
      toast.success('تم تعليق المتجر بنجاح');
    },
  });

  const columns = [
    { accessorKey: 'storeName', header: 'اسم المتجر',
      cell: ({ row }) => (
        <div className="flex items-center gap-3">
          <img src={row.original.logo || '/placeholder-store.png'} alt=""
            className="w-10 h-10 rounded-lg object-cover" />
          <div>
            <p className="font-medium">{row.original.storeName}</p>
            <p className="text-sm text-neutral-500">{row.original.ownerName}</p>
          </div>
        </div>
      ) },
    { accessorKey: 'products', header: 'المنتجات',
      cell: ({ row }) => row.original.productCount },
    { accessorKey: 'orders', header: 'الطلبات',
      cell: ({ row }) => row.original.orderCount },
    { accessorKey: 'revenue', header: 'الإيرادات',
      cell: ({ row }) => (
        <span className="font-medium">{row.original.totalRevenue.toLocaleString('ar-YE')} يمني</span>
      ) },
    { accessorKey: 'kycStatus', header: 'KYC',
      cell: ({ row }) => (
        <Badge variant={
          row.original.kycStatus === 'verified' ? 'success' :
          row.original.kycStatus === 'pending' ? 'warning' :
          row.original.kycStatus === 'rejected' ? 'error' : 'neutral'
        }>
          {row.original.kycStatus === 'verified' ? 'موثق' :
           row.original.kycStatus === 'pending' ? 'قيد المراجعة' :
           row.original.kycStatus === 'rejected' ? 'مرفوض' : 'غير مقدم'}
        </Badge>
      ) },
    { accessorKey: 'status', header: 'الحالة',
      cell: ({ row }) => (
        <Badge variant={row.original.isActive ? 'success' : 'error'}>
          {row.original.isActive ? 'نشط' : 'معلق'}
        </Badge>
      ) },
    { accessorKey: 'rating', header: 'التقييم',
      cell: ({ row }) => `${row.original.rating} ⭐` },
    { id: 'actions', header: 'الإجراءات',
      cell: ({ row }) => (
        <div className="flex items-center gap-1">
          <button onClick={() => setSelectedVendor(row.original)}
            className="p-2 hover:bg-neutral-100 rounded-lg"><Eye className="w-4 h-4" /></button>
          <PermissionGuard permission={AdminPermission.APPROVE_VENDOR_KYC}>
            {row.original.kycStatus === 'pending' && (
              <button onClick={() => setReviewingKYC(row.original)}
                className="p-2 hover:bg-blue-50 text-blue-500 rounded-lg">
                <Shield className="w-4 h-4" />
              </button>
            )}
          </PermissionGuard>
          <PermissionGuard permission={AdminPermission.SUSPEND_VENDORS}>
            {row.original.isActive && (
              <button onClick={() => suspendMutation.mutate(row.original.id)}
                className="p-2 hover:bg-red-50 text-red-500 rounded-lg">
                <XCircle className="w-4 h-4" />
              </button>
            )}
          </PermissionGuard>
        </div>
      ) },
  ];

  return (
    <div className="space-y-6">
      <div className="flex items-center justify-between">
        <h1 className="text-2xl font-bold">المتاجر</h1>
        <Button variant="outline"><Download className="w-4 h-4 ml-2" />تصدير</Button>
      </div>

      <div className="flex flex-wrap items-center gap-3">
        <Input placeholder="بحث بالاسم، المالك..." value={search}
          onChange={(e) => setSearch(e.target.value)}
          icon={<Search className="w-4 h-4" />} className="max-w-sm" />
        <select value={status} onChange={(e) => setStatus(e.target.value)}
          className="px-4 py-2 rounded-lg border">
          <option value="">جميع الحالات</option>
          <option value="active">نشط</option>
          <option value="suspended">معلق</option>
        </select>
        <select value={kycStatus} onChange={(e) => setKycStatus(e.target.value)}
          className="px-4 py-2 rounded-lg border">
          <option value="">جميع KYC</option>
          <option value="verified">موثق</option>
          <option value="pending">قيد المراجعة</option>
          <option value="rejected">مرفوض</option>
        </select>
      </div>

      <DataTable data={data?.vendors || []} columns={columns} isLoading={isLoading}
        emptyMessage="لا يوجد متاجر" pagination={data?.pagination} />

      {selectedVendor && (
        <VendorDetailDrawer vendor={selectedVendor} onClose={() => setSelectedVendor(null)} />
      )}

      {reviewingKYC && (
        <KYCReviewModal vendor={reviewingKYC}
          onClose={() => setReviewingKYC(null)}
          onSuccess={() => {
            queryClient.invalidateQueries({ queryKey: ['admin-vendors'] });
            setReviewingKYC(null);
          }} />
      )}
    </div>
  );
}
```

### 4. Order Management (`/admin/orders`)

```typescript
// pages/admin/Orders.tsx
import { useState } from 'react';
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { DataTable } from '@/components/ui/DataTable';
import { Button } from '@/components/ui/Button';
import { Input } from '@/components/ui/Input';
import { Badge } from '@/components/ui/Badge';
import { AdminOrderDetail } from '@/components/admin/orders/AdminOrderDetail';
import { AdminPermission } from '@/types/admin-roles';
import { PermissionGuard } from '@/components/admin/auth/PermissionGuard';
import { fetchAdminOrders, cancelAdminOrder, issueRefund } from '@/lib/api/admin/orders';
import { toast } from 'react-hot-toast';
import { Search, Eye, Ban, Refund, Download } from 'lucide-react';

export function AdminOrders() {
  const queryClient = useQueryClient();
  const [search, setSearch] = useState('');
  const [status, setStatus] = useState('');
  const [dateRange, setDateRange] = useState({ from: '', to: '' });
  const [selectedOrder, setSelectedOrder] = useState<any>(null);

  const { data, isLoading } = useQuery({
    queryKey: ['admin-orders', { search, status, dateRange }],
    queryFn: () => fetchAdminOrders({ search, status, dateRange }),
  });

  const cancelMutation = useMutation({
    mutationFn: cancelAdminOrder,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['admin-orders'] });
      toast.success('تم إلغاء الطلب بنجاح');
    },
  });

  const refundMutation = useMutation({
    mutationFn: issueRefund,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['admin-orders'] });
      toast.success('تم إصدار المبلغ المسترجع بنجاح');
    },
  });

  const columns = [
    { accessorKey: 'orderNumber', header: 'رقم الطلب',
      cell: ({ row }) => <span className="font-mono">#{row.original.orderNumber}</span> },
    { accessorKey: 'customer', header: 'العميل',
      cell: ({ row }) => (
        <div>
          <p className="font-medium">{row.original.customerName}</p>
          <p className="text-sm text-neutral-500">{row.original.customerPhone}</p>
        </div>
      ) },
    { accessorKey: 'vendor', header: 'المتجر',
      cell: ({ row }) => row.original.vendorName },
    { accessorKey: 'items', header: 'المنتجات',
      cell: ({ row }) => `${row.original.itemCount} منتجات` },
    { accessorKey: 'total', header: 'المبلغ',
      cell: ({ row }) => (
        <span className="font-medium">{row.original.total.toLocaleString('ar-YE')} يمني</span>
      ) },
    { accessorKey: 'paymentMethod', header: 'الدفع',
      cell: ({ row }) => (
        <Badge variant={row.original.paymentMethod === 'cod' ? 'warning' : 'info'}>
          {row.original.paymentMethod === 'cod' ? 'الدفع عند الاستلام' : 'محفظة'}
        </Badge>
      ) },
    { accessorKey: 'status', header: 'الحالة',
      cell: ({ row }) => (
        <Badge variant={
          row.original.status === 'delivered' ? 'success' :
          row.original.status === 'cancelled' ? 'error' :
          row.original.status === 'refunded' ? 'warning' : 'info'
        }>
          {row.original.statusLabel}
        </Badge>
      ) },
    { accessorKey: 'createdAt', header: 'التاريخ',
      cell: ({ row }) => new Date(row.original.createdAt).toLocaleDateString('ar-YE') },
    { id: 'actions', header: 'الإجراءات',
      cell: ({ row }) => (
        <div className="flex items-center gap-1">
          <button onClick={() => setSelectedOrder(row.original)}
            className="p-2 hover:bg-neutral-100 rounded-lg"><Eye className="w-4 h-4" /></button>
          <PermissionGuard permission={AdminPermission.CANCEL_ORDERS}>
            {['pending', 'confirmed'].includes(row.original.status) && (
              <button onClick={() => { if (confirm('هل أنت متأكد من إلغاء الطلب؟')) cancelMutation.mutate(row.original.id); }}
                className="p-2 hover:bg-red-50 text-red-500 rounded-lg"><Ban className="w-4 h-4" /></button>
            )}
          </PermissionGuard>
          <PermissionGuard permission={AdminPermission.ISSUE_REFUNDS}>
            {row.original.status === 'delivered' && row.original.paymentMethod !== 'cod' && (
              <button onClick={() => refundMutation.mutate({ orderId: row.original.id, reason: 'Admin refund' })}
                className="p-2 hover:bg-yellow-50 text-yellow-500 rounded-lg"><Refund className="w-4 h-4" /></button>
            )}
          </PermissionGuard>
        </div>
      ) },
  ];

  return (
    <div className="space-y-6">
      <div className="flex items-center justify-between">
        <h1 className="text-2xl font-bold">الطلبات</h1>
        <Button variant="outline"><Download className="w-4 h-4 ml-2" />تصدير التقرير</Button>
      </div>

      <div className="flex flex-wrap items-center gap-3">
        <Input placeholder="بحث برقم الطلب، اسم العميل..." value={search}
          onChange={(e) => setSearch(e.target.value)}
          icon={<Search className="w-4 h-4" />} className="max-w-sm" />
        <select value={status} onChange={(e) => setStatus(e.target.value)}
          className="px-4 py-2 rounded-lg border">
          <option value="">جميع الحالات</option>
          <option value="pending">قيد الانتظار</option>
          <option value="confirmed">مؤكد</option>
          <option value="processing">قيد التجهيز</option>
          <option value="shipped">تم الشحن</option>
          <option value="delivered">تم التوصيل</option>
          <option value="cancelled">ملغي</option>
          <option value="refunded">مسترجع</option>
        </select>
      </div>

      <DataTable data={data?.orders || []} columns={columns} isLoading={isLoading}
        emptyMessage="لا توجد طلبات" pagination={data?.pagination} />

      {selectedOrder && (
        <AdminOrderDetail order={selectedOrder} onClose={() => setSelectedOrder(null)} />
      )}
    </div>
  );
}
```

### 5. Product Moderation (`/admin/products`)

```typescript
// pages/admin/Products.tsx
import { useState } from 'react';
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { DataTable } from '@/components/ui/DataTable';
import { Badge } from '@/components/ui/Badge';
import { AdminPermission } from '@/types/admin-roles';
import { PermissionGuard } from '@/components/admin/auth/PermissionGuard';
import { fetchAdminProducts, approveProduct, rejectProduct, deleteProduct } from '@/lib/api/admin/products';
import { toast } from 'react-hot-toast';
import { Eye, CheckCircle, XCircle, Trash2 } from 'lucide-react';

export function AdminProducts() {
  const queryClient = useQueryClient();
  const [status, setStatus] = useState('pending');

  const { data, isLoading } = useQuery({
    queryKey: ['admin-products', { status }],
    queryFn: () => fetchAdminProducts({ status }),
  });

  const approveMutation = useMutation({
    mutationFn: approveProduct,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['admin-products'] });
      toast.success('تم اعتماد المنتج بنجاح');
    },
  });

  const rejectMutation = useMutation({
    mutationFn: rejectProduct,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['admin-products'] });
      toast.success('تم رفض المنتج بنجاح');
    },
  });

  const columns = [
    { accessorKey: 'image', header: 'الصورة',
      cell: ({ row }) => (
        <img src={row.original.images[0]?.url || '/placeholder.png'} alt=""
          className="w-12 h-12 rounded-lg object-cover" />
      ) },
    { accessorKey: 'name', header: 'المنتج',
      cell: ({ row }) => (
        <div>
          <p className="font-medium">{row.original.name}</p>
          <p className="text-sm text-neutral-500">{row.original.vendorName}</p>
        </div>
      ) },
    { accessorKey: 'category', header: 'القسم',
      cell: ({ row }) => row.original.category?.name },
    { accessorKey: 'price', header: 'السعر',
      cell: ({ row }) => `${row.original.price.toLocaleString('ar-YE')} يمني` },
    { accessorKey: 'status', header: 'الحالة',
      cell: ({ row }) => (
        <Badge variant={
          row.original.status === 'approved' ? 'success' :
          row.original.status === 'pending' ? 'warning' :
          row.original.status === 'rejected' ? 'error' : 'neutral'
        }>
          {row.original.statusLabel}
        </Badge>
      ) },
    { accessorKey: 'createdAt', header: 'التاريخ',
      cell: ({ row }) => new Date(row.original.createdAt).toLocaleDateString('ar-YE') },
    { id: 'actions', header: 'الإجراءات',
      cell: ({ row }) => (
        <div className="flex items-center gap-1">
          <PermissionGuard permission={AdminPermission.APPROVE_PRODUCTS}>
            {row.original.status === 'pending' && (
              <>
                <button onClick={() => approveMutation.mutate(row.original.id)}
                  className="p-2 hover:bg-green-50 text-green-500 rounded-lg">
                  <CheckCircle className="w-4 h-4" />
                </button>
                <button onClick={() => rejectMutation.mutate(row.original.id)}
                  className="p-2 hover:bg-red-50 text-red-500 rounded-lg">
                  <XCircle className="w-4 h-4" />
                </button>
              </>
            )}
          </PermissionGuard>
          <PermissionGuard permission={AdminPermission.DELETE_PRODUCTS}>
            <button onClick={() => { if (confirm('هل أنت متأكد من حذف هذا المنتج؟')) deleteProduct(row.original.id); }}
              className="p-2 hover:bg-red-50 text-red-500 rounded-lg">
              <Trash2 className="w-4 h-4" />
            </button>
          </PermissionGuard>
        </div>
      ) },
  ];

  return (
    <div className="space-y-6">
      <h1 className="text-2xl font-bold">إدارة المنتجات</h1>

      <div className="flex gap-2">
        {[
          { value: 'pending', label: 'قيد المراجعة', count: data?.pendingCount },
          { value: 'approved', label: 'معتمد', count: data?.approvedCount },
          { value: 'rejected', label: 'مرفوض', count: data?.rejectedCount },
        ].map((tab) => (
          <button key={tab.value} onClick={() => setStatus(tab.value)}
            className={`px-4 py-2 rounded-lg text-sm font-medium transition-colors ${
              status === tab.value ? 'bg-primary-600 text-white' : 'bg-neutral-100 hover:bg-neutral-200'
            }`}>
            {tab.label} {tab.count !== undefined && `(${tab.count})`}
          </button>
        ))}
      </div>

      <DataTable data={data?.products || []} columns={columns} isLoading={isLoading}
        emptyMessage="لا توجد منتجات" />
    </div>
  );
}
```

### 6. Finance Management (`/admin/finance`)

```typescript
// pages/admin/Finance.tsx
import { useState } from 'react';
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/Card';
import { DataTable } from '@/components/ui/DataTable';
import { Button } from '@/components/ui/Button';
import { Badge } from '@/components/ui/Badge';
import { Tabs, TabsContent, TabsList, TabsTrigger } from '@/components/ui/Tabs';
import { DateRangePicker } from '@/components/ui/DateRangePicker';
import { RevenueChart } from '@/components/admin/finance/RevenueChart';
import { PayoutQueue } from '@/components/admin/finance/PayoutQueue';
import { AdminPermission } from '@/types/admin-roles';
import { PermissionGuard } from '@/components/admin/auth/PermissionGuard';
import { fetchFinanceOverview, fetchTransactions, fetchPayouts, processPayout } from '@/lib/api/admin/finance';
import { toast } from 'react-hot-toast';
import { DollarSign, TrendingUp, CreditCard, Download, CheckCircle } from 'lucide-react';

export function AdminFinance() {
  const queryClient = useQueryClient();
  const [activeTab, setActiveTab] = useState('overview');
  const [dateRange, setDateRange] = useState<{ from: Date; to: Date }>({
    from: new Date(Date.now() - 30 * 24 * 60 * 60 * 1000),
    to: new Date(),
  });

  const { data: overview, isLoading: overviewLoading } = useQuery({
    queryKey: ['admin-finance-overview', { dateRange }],
    queryFn: () => fetchFinanceOverview({ dateRange }),
  });

  const { data: transactions } = useQuery({
    queryKey: ['admin-transactions', { dateRange }],
    queryFn: () => fetchTransactions({ dateRange }),
  });

  const { data: payouts } = useQuery({
    queryKey: ['admin-payouts'],
    queryFn: fetchPayouts,
  });

  const processPayoutMutation = useMutation({
    mutationFn: processPayout,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['admin-payouts'] });
      toast.success('تم معالجة الدفعة بنجاح');
    },
  });

  return (
    <div className="space-y-6">
      <div className="flex items-center justify-between">
        <h1 className="text-2xl font-bold">الإدارة المالية</h1>
        <DateRangePicker value={dateRange} onChange={setDateRange} />
      </div>

      <Tabs value={activeTab} onValueChange={setActiveTab}>
        <TabsList>
          <TabsTrigger value="overview">نظرة عامة</TabsTrigger>
          <TabsTrigger value="transactions">المعاملات</TabsTrigger>
          <TabsTrigger value="payouts">المدفوعات</TabsTrigger>
        </TabsList>

        <TabsContent value="overview" className="space-y-6">
          <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
            <Card>
              <CardContent className="p-6">
                <div className="flex items-center gap-4">
                  <div className="p-3 bg-green-100 dark:bg-green-900/20 rounded-lg">
                    <DollarSign className="w-6 h-6 text-green-600" />
                  </div>
                  <div>
                    <p className="text-sm text-neutral-500">إجمالي الإيرادات</p>
                    <p className="text-2xl font-bold">{overview?.totalRevenue?.toLocaleString('ar-YE')} يمني</p>
                  </div>
                </div>
              </CardContent>
            </Card>
            <Card>
              <CardContent className="p-6">
                <div className="flex items-center gap-4">
                  <div className="p-3 bg-blue-100 dark:bg-blue-900/20 rounded-lg">
                    <CreditCard className="w-6 h-6 text-blue-600" />
                  </div>
                  <div>
                    <p className="text-sm text-neutral-500">أرباح المنصة</p>
                    <p className="text-2xl font-bold">{overview?.platformRevenue?.toLocaleString('ar-YE')} يمني</p>
                  </div>
                </div>
              </CardContent>
            </Card>
            <Card>
              <CardContent className="p-6">
                <div className="flex items-center gap-4">
                  <div className="p-3 bg-orange-100 dark:bg-orange-900/20 rounded-lg">
                    <TrendingUp className="w-6 h-6 text-orange-600" />
                  </div>
                  <div>
                    <p className="text-sm text-neutral-500">بانتظار التحويل</p>
                    <p className="text-2xl font-bold">{overview?.pendingPayouts?.toLocaleString('ar-YE')} يمني</p>
                  </div>
                </div>
              </CardContent>
            </Card>
          </div>

          <RevenueChart data={overview?.revenueChart || []} />
        </TabsContent>

        <TabsContent value="transactions">
          <DataTable data={transactions?.items || []} columns={[
            { accessorKey: 'id', header: 'رقم المعاملة' },
            { accessorKey: 'type', header: 'النوع',
              cell: ({ row }) => (
                <Badge variant={row.original.type === 'sale' ? 'success' : 'warning'}>
                  {row.original.type === 'sale' ? 'مبيعات' : 'استرداد'}
                </Badge>
              ) },
            { accessorKey: 'amount', header: 'المبلغ',
              cell: ({ row }) => `${row.original.amount.toLocaleString('ar-YE')} يمني` },
            { accessorKey: 'vendor', header: 'المتجر' },
            { accessorKey: 'orderNumber', header: 'رقم الطلب' },
            { accessorKey: 'createdAt', header: 'التاريخ',
              cell: ({ row }) => new Date(row.original.createdAt).toLocaleDateString('ar-YE') },
          ]} emptyMessage="لا توجد معاملات" pagination={transactions?.pagination} />
        </TabsContent>

        <TabsContent value="payouts">
          <PermissionGuard permission={AdminPermission.PROCESS_PAYOUTS}>
            <PayoutQueue payouts={payouts?.pending || []}
              onProcess={(id) => processPayoutMutation.mutate(id)} />
          </PermissionGuard>

          <div className="mt-6">
            <h3 className="text-lg font-semibold mb-4">سجل الدفعات</h3>
            <DataTable data={payouts?.completed || []} columns={[
              { accessorKey: 'vendorName', header: 'المتجر' },
              { accessorKey: 'amount', header: 'المبلغ',
                cell: ({ row }) => `${row.original.amount.toLocaleString('ar-YE')} يمني` },
              { accessorKey: 'status', header: 'الحالة',
                cell: ({ row }) => (
                  <Badge variant="success"><CheckCircle className="w-3 h-3 ml-1" />مكتمل</Badge>
                ) },
              { accessorKey: 'processedAt', header: 'التاريخ',
                cell: ({ row }) => new Date(row.original.processedAt).toLocaleDateString('ar-YE') },
            ]} emptyMessage="لا توجد مدفوعات مكتملة" />
          </div>
        </TabsContent>
      </Tabs>
    </div>
  );
}
```

### 7. Coupon Management (`/admin/coupons`)

```typescript
// pages/admin/Coupons.tsx
import { useState } from 'react';
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { DataTable } from '@/components/ui/DataTable';
import { Button } from '@/components/ui/Button';
import { Badge } from '@/components/ui/Badge';
import { Modal } from '@/components/ui/Modal';
import { AdminCouponForm } from '@/components/admin/coupons/AdminCouponForm';
import { AdminPermission } from '@/types/admin-roles';
import { PermissionGuard } from '@/components/admin/auth/PermissionGuard';
import { fetchAdminCoupons, deleteAdminCoupon } from '@/lib/api/admin/coupons';
import { toast } from 'react-hot-toast';
import { Plus, Edit, Trash2, Copy } from 'lucide-react';

export function AdminCoupons() {
  const queryClient = useQueryClient();
  const [isCreateModalOpen, setIsCreateModalOpen] = useState(false);
  const [editingCoupon, setEditingCoupon] = useState<any>(null);

  const { data, isLoading } = useQuery({
    queryKey: ['admin-coupons'],
    queryFn: fetchAdminCoupons,
  });

  const deleteMutation = useMutation({
    mutationFn: deleteAdminCoupon,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['admin-coupons'] });
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
            className="p-1 hover:bg-neutral-100 rounded"><Copy className="w-3 h-3" /></button>
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
    { accessorKey: 'scope', header: 'النطاق',
      cell: ({ row }) => (
        <Badge variant={row.original.scope === 'platform' ? 'primary' : 'neutral'}>
          {row.original.scope === 'platform' ? 'المنصة' : 'متجر محدد'}
        </Badge>
      ) },
    { accessorKey: 'usageCount', header: 'الاستخدام',
      cell: ({ row }) => `${row.original.usageCount}/${row.original.usageLimit || '∞'}` },
    { accessorKey: 'expiresAt', header: 'ينتهي في',
      cell: ({ row }) => new Date(row.original.expiresAt).toLocaleDateString('ar-YE') },
    { id: 'actions', header: 'الإجراءات',
      cell: ({ row }) => (
        <div className="flex items-center gap-2">
          <PermissionGuard permission={AdminPermission.EDIT_COUPON}>
            <button onClick={() => setEditingCoupon(row.original)}
              className="p-2 hover:bg-neutral-100 rounded-lg"><Edit className="w-4 h-4" /></button>
          </PermissionGuard>
          <PermissionGuard permission={AdminPermission.DELETE_COUPON}>
            <button onClick={() => { if (confirm('هل أنت متأكد؟')) deleteMutation.mutate(row.original.id); }}
              className="p-2 hover:bg-red-50 text-red-500 rounded-lg"><Trash2 className="w-4 h-4" /></button>
          </PermissionGuard>
        </div>
      ) },
  ];

  return (
    <div className="space-y-6">
      <div className="flex items-center justify-between">
        <h1 className="text-2xl font-bold">الكوبونات</h1>
        <PermissionGuard permission={AdminPermission.CREATE_COUPON}>
          <Button onClick={() => setIsCreateModalOpen(true)}>
            <Plus className="w-4 h-4 ml-2" />إنشاء كوبون
          </Button>
        </PermissionGuard>
      </div>

      <DataTable data={data?.coupons || []} columns={columns} isLoading={isLoading}
        emptyMessage="لا توجد كوبونات" />

      <Modal isOpen={isCreateModalOpen} onClose={() => setIsCreateModalOpen(false)}
        title="إنشاء كوبون جديد">
        <AdminCouponForm onSubmit={() => {
          queryClient.invalidateQueries({ queryKey: ['admin-coupons'] });
          setIsCreateModalOpen(false);
        }} onCancel={() => setIsCreateModalOpen(false)} />
      </Modal>

      {editingCoupon && (
        <Modal isOpen={true} onClose={() => setEditingCoupon(null)} title="تعديل الكوبون">
          <AdminCouponForm initialData={editingCoupon} onSubmit={() => {
            queryClient.invalidateQueries({ queryKey: ['admin-coupons'] });
            setEditingCoupon(null);
          }} onCancel={() => setEditingCoupon(null)} />
        </Modal>
      )}
    </div>
  );
}
```

### 8. Content Management (`/admin/content`)

```typescript
// pages/admin/Content.tsx
import { useState } from 'react';
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { Tabs, TabsContent, TabsList, TabsTrigger } from '@/components/ui/Tabs';
import { BannerManager } from '@/components/admin/content/BannerManager';
import { CategoryManager } from '@/components/admin/content/CategoryManager';
import { PageManager } from '@/components/admin/content/PageManager';
import { NotificationTemplates } from '@/components/admin/content/NotificationTemplates';

export function AdminContent() {
  const [activeTab, setActiveTab] = useState('banners');

  return (
    <div className="space-y-6">
      <h1 className="text-2xl font-bold">إدارة المحتوى</h1>

      <Tabs value={activeTab} onValueChange={setActiveTab}>
        <TabsList>
          <TabsTrigger value="banners">البانرات</TabsTrigger>
          <TabsTrigger value="categories">الأقسام</TabsTrigger>
          <TabsTrigger value="pages">الصفحات الثابتة</TabsTrigger>
          <TabsTrigger value="templates">قوالب الإشعارات</TabsTrigger>
        </TabsList>

        <TabsContent value="banners"><BannerManager /></TabsContent>
        <TabsContent value="categories"><CategoryManager /></TabsContent>
        <TabsContent value="pages"><PageManager /></TabsContent>
        <TabsContent value="templates"><NotificationTemplates /></TabsContent>
      </Tabs>
    </div>
  );
}
```

### 9. Settings (`/admin/settings`)

```typescript
// pages/admin/Settings.tsx
import { useState } from 'react';
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { Tabs, TabsContent, TabsList, TabsTrigger } from '@/components/ui/Tabs';
import { GeneralSettings } from '@/components/admin/settings/GeneralSettings';
import { PaymentSettings } from '@/components/admin/settings/PaymentSettings';
import { ShippingSettings } from '@/components/admin/settings/ShippingSettings';
import { NotificationSettings } from '@/components/admin/settings/NotificationSettings';
import { PlatformFees } from '@/components/admin/settings/PlatformFees';
import { fetchAdminSettings, updateAdminSettings } from '@/lib/api/admin/settings';
import { toast } from 'react-hot-toast';

export function AdminSettings() {
  const queryClient = useQueryClient();
  const [activeTab, setActiveTab] = useState('general');

  const { data: settings } = useQuery({
    queryKey: ['admin-settings'],
    queryFn: fetchAdminSettings,
  });

  const updateMutation = useMutation({
    mutationFn: updateAdminSettings,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['admin-settings'] });
      toast.success('تم حفظ الإعدادات بنجاح');
    },
  });

  return (
    <div className="space-y-6">
      <h1 className="text-2xl font-bold">إعدادات المنصة</h1>

      <Tabs value={activeTab} onValueChange={setActiveTab}>
        <TabsList>
          <TabsTrigger value="general">عام</TabsTrigger>
          <TabsTrigger value="payment">المدفوعات</TabsTrigger>
          <TabsTrigger value="shipping">الشحن</TabsTrigger>
          <TabsTrigger value="notifications">الإشعارات</TabsTrigger>
          <TabsTrigger value="fees">رسوم المنصة</TabsTrigger>
        </TabsList>

        <TabsContent value="general">
          <GeneralSettings settings={settings?.general} onSave={(data) => updateMutation.mutate({ section: 'general', data })} />
        </TabsContent>
        <TabsContent value="payment">
          <PaymentSettings settings={settings?.payment} onSave={(data) => updateMutation.mutate({ section: 'payment', data })} />
        </TabsContent>
        <TabsContent value="shipping">
          <ShippingSettings settings={settings?.shipping} onSave={(data) => updateMutation.mutate({ section: 'shipping', data })} />
        </TabsContent>
        <TabsContent value="notifications">
          <NotificationSettings settings={settings?.notifications} onSave={(data) => updateMutation.mutate({ section: 'notifications', data })} />
        </TabsContent>
        <TabsContent value="fees">
          <PlatformFees settings={settings?.fees} onSave={(data) => updateMutation.mutate({ section: 'fees', data })} />
        </TabsContent>
      </Tabs>
    </div>
  );
}
```

### 10. Support Tickets (`/admin/support`)

```typescript
// pages/admin/Support.tsx
import { useState } from 'react';
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { DataTable } from '@/components/ui/DataTable';
import { Badge } from '@/components/ui/Badge';
import { Button } from '@/components/ui/Button';
import { Input } from '@/components/ui/Input';
import { Textarea } from '@/components/ui/Textarea';
import { Modal } from '@/components/ui/Modal';
import { TicketDetail } from '@/components/admin/support/TicketDetail';
import { AdminPermission } from '@/types/admin-roles';
import { PermissionGuard } from '@/components/admin/auth/PermissionGuard';
import { fetchTickets, respondToTicket, closeTicket } from '@/lib/api/admin/support';
import { toast } from 'react-hot-toast';
import { MessageSquare, CheckCircle, Clock, AlertCircle } from 'lucide-react';

export function AdminSupport() {
  const queryClient = useQueryClient();
  const [status, setStatus] = useState('');
  const [priority, setPriority] = useState('');
  const [selectedTicket, setSelectedTicket] = useState<any>(null);
  const [respondingTicket, setRespondingTicket] = useState<any>(null);
  const [response, setResponse] = useState('');

  const { data, isLoading } = useQuery({
    queryKey: ['admin-tickets', { status, priority }],
    queryFn: () => fetchTickets({ status, priority }),
  });

  const respondMutation = useMutation({
    mutationFn: ({ ticketId, message }: { ticketId: string; message: string }) =>
      respondToTicket(ticketId, message),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['admin-tickets'] });
      toast.success('تم إرسال الرد بنجاح');
      setRespondingTicket(null);
      setResponse('');
    },
  });

  const closeMutation = useMutation({
    mutationFn: closeTicket,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['admin-tickets'] });
      toast.success('تم إغلاق التذكرة بنجاح');
    },
  });

  const columns = [
    { accessorKey: 'ticketNumber', header: 'رقم التذكرة',
      cell: ({ row }) => <span className="font-mono">#{row.original.ticketNumber}</span> },
    { accessorKey: 'subject', header: 'الموضوع',
      cell: ({ row }) => (
        <div>
          <p className="font-medium">{row.original.subject}</p>
          <p className="text-sm text-neutral-500">{row.original.customerName}</p>
        </div>
      ) },
    { accessorKey: 'category', header: 'الفئة',
      cell: ({ row }) => row.original.category },
    { accessorKey: 'priority', header: 'الأولوية',
      cell: ({ row }) => (
        <Badge variant={
          row.original.priority === 'high' ? 'error' :
          row.original.priority === 'medium' ? 'warning' : 'info'
        }>
          {row.original.priority === 'high' ? 'عالية' :
           row.original.priority === 'medium' ? 'متوسطة' : 'منخفضة'}
        </Badge>
      ) },
    { accessorKey: 'status', header: 'الحالة',
      cell: ({ row }) => (
        <Badge variant={
          row.original.status === 'open' ? 'warning' :
          row.original.status === 'in_progress' ? 'info' :
          row.original.status === 'resolved' ? 'success' : 'neutral'
        }>
          {row.original.statusLabel}
        </Badge>
      ) },
    { accessorKey: 'createdAt', header: 'التاريخ',
      cell: ({ row }) => new Date(row.original.createdAt).toLocaleDateString('ar-YE') },
    { id: 'actions', header: 'الإجراءات',
      cell: ({ row }) => (
        <div className="flex items-center gap-1">
          <button onClick={() => setSelectedTicket(row.original)}
            className="p-2 hover:bg-neutral-100 rounded-lg"><MessageSquare className="w-4 h-4" /></button>
          <PermissionGuard permission={AdminPermission.RESPOND_TICKETS}>
            {row.original.status !== 'closed' && (
              <button onClick={() => setRespondingTicket(row.original)}
                className="p-2 hover:bg-blue-50 text-blue-500 rounded-lg">
                <Clock className="w-4 h-4" />
              </button>
            )}
          </PermissionGuard>
          <PermissionGuard permission={AdminPermission.CLOSE_TICKETS}>
            {row.original.status !== 'closed' && (
              <button onClick={() => closeMutation.mutate(row.original.id)}
                className="p-2 hover:bg-green-50 text-green-500 rounded-lg">
                <CheckCircle className="w-4 h-4" />
              </button>
            )}
          </PermissionGuard>
        </div>
      ) },
  ];

  return (
    <div className="space-y-6">
      <h1 className="text-2xl font-bold">تذاكر الدعم</h1>

      <div className="flex gap-2">
        {[
          { value: '', label: 'الكل' },
          { value: 'open', label: 'مفتوحة' },
          { value: 'in_progress', label: 'قيد المعالجة' },
          { value: 'resolved', label: 'محلولة' },
          { value: 'closed', label: 'مغلقة' },
        ].map((filter) => (
          <button key={filter.value} onClick={() => setStatus(filter.value)}
            className={`px-4 py-2 rounded-lg text-sm font-medium ${
              status === filter.value ? 'bg-primary-600 text-white' : 'bg-neutral-100 hover:bg-neutral-200'
            }`}>
            {filter.label}
          </button>
        ))}
      </div>

      <DataTable data={data?.tickets || []} columns={columns} isLoading={isLoading}
        emptyMessage="لا توجد تذاكر" pagination={data?.pagination} />

      {selectedTicket && (
        <TicketDetail ticket={selectedTicket} onClose={() => setSelectedTicket(null)} />
      )}

      {respondingTicket && (
        <Modal isOpen={true} onClose={() => setRespondingTicket(null)} title="الرد على التذكرة">
          <div className="space-y-4">
            <p className="text-sm text-neutral-500">
              التذكرة: #{respondingTicket.ticketNumber} - {respondingTicket.subject}
            </p>
            <Textarea placeholder="اكتب ردك هنا..." value={response}
              onChange={(e) => setResponse(e.target.value)} rows={6} />
            <div className="flex justify-end gap-2">
              <Button variant="outline" onClick={() => setRespondingTicket(null)}>إلغاء</Button>
              <Button onClick={() => respondMutation.mutate({ ticketId: respondingTicket.id, message: response })}
                loading={respondMutation.isLoading}>
                إرسال الرد
              </Button>
            </div>
          </div>
        </Modal>
      )}
    </div>
  );
}
```

### 11. Security & Audit Log (`/admin/security`)

```typescript
// pages/admin/Security.tsx
import { useState } from 'react';
import { useQuery } from '@tanstack/react-query';
import { Tabs, TabsContent, TabsList, TabsTrigger } from '@/components/ui/Tabs';
import { DataTable } from '@/components/ui/DataTable';
import { Badge } from '@/components/ui/Badge';
import { Input } from '@/components/ui/Input';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/Card';
import { AuditLogViewer } from '@/components/admin/security/AuditLogViewer';
import { RestrictionManager } from '@/components/admin/security/RestrictionManager';
import { SecurityEvents } from '@/components/admin/security/SecurityEvents';
import { AdminPermission } from '@/types/admin-roles';
import { PermissionGuard } from '@/components/admin/auth/PermissionGuard';
import { fetchAuditLog, fetchSecurityEvents } from '@/lib/api/admin/security';
import { Shield, AlertTriangle, Activity, Ban } from 'lucide-react';

export function AdminSecurity() {
  const [activeTab, setActiveTab] = useState('audit');
  const [search, setSearch] = useState('');

  const { data: auditLog, isLoading: auditLoading } = useQuery({
    queryKey: ['admin-audit-log', { search }],
    queryFn: () => fetchAuditLog({ search }),
  });

  const { data: securityEvents } = useQuery({
    queryKey: ['admin-security-events'],
    queryFn: fetchSecurityEvents,
  });

  return (
    <div className="space-y-6">
      <h1 className="text-2xl font-bold">الأمان والمراقبة</h1>

      <div className="grid grid-cols-1 md:grid-cols-4 gap-4">
        <Card>
          <CardContent className="p-4">
            <div className="flex items-center gap-3">
              <Shield className="w-8 h-8 text-green-500" />
              <div>
                <p className="text-sm text-neutral-500">الجلسات النشطة</p>
                <p className="text-xl font-bold">{securityEvents?.activeSessions || 0}</p>
              </div>
            </div>
          </CardContent>
        </Card>
        <Card>
          <CardContent className="p-4">
            <div className="flex items-center gap-3">
              <AlertTriangle className="w-8 h-8 text-yellow-500" />
              <div>
                <p className="text-sm text-neutral-500">محاولات فاشلة</p>
                <p className="text-xl font-bold">{securityEvents?.failedAttempts || 0}</p>
              </div>
            </div>
          </CardContent>
        </Card>
        <Card>
          <CardContent className="p-4">
            <div className="flex items-center gap-3">
              <Activity className="w-8 h-8 text-blue-500" />
              <div>
                <p className="text-sm text-neutral-500">أحداث اليوم</p>
                <p className="text-xl font-bold">{securityEvents?.todayEvents || 0}</p>
              </div>
            </div>
          </CardContent>
        </Card>
        <Card>
          <CardContent className="p-4">
            <div className="flex items-center gap-3">
              <Ban className="w-8 h-8 text-red-500" />
              <div>
                <p className="text-sm text-neutral-500">العناصر المحظورة</p>
                <p className="text-xl font-bold">{securityEvents?.blockedEntities || 0}</p>
              </div>
            </div>
          </CardContent>
        </Card>
      </div>

      <Tabs value={activeTab} onValueChange={setActiveTab}>
        <TabsList>
          <TabsTrigger value="audit">سجل التدقيق</TabsTrigger>
          <TabsTrigger value="events">الأحداث الأمنية</TabsTrigger>
          <TabsTrigger value="restrictions">القيود</TabsTrigger>
        </TabsList>

        <TabsContent value="audit">
          <Input placeholder="بحث في سجل التدقيق..." value={search}
            onChange={(e) => setSearch(e.target.value)} className="max-w-sm mb-4" />
          <AuditLogViewer data={auditLog?.logs || []} isLoading={auditLoading} />
        </TabsContent>

        <TabsContent value="events">
          <SecurityEvents events={securityEvents?.events || []} />
        </TabsContent>

        <TabsContent value="restrictions">
          <PermissionGuard permission={AdminPermission.MANAGE_RESTRICTIONS}>
            <RestrictionManager />
          </PermissionGuard>
        </TabsContent>
      </Tabs>
    </div>
  );
}
```

### 12. ZATCA Invoice Management (`/admin/zatca`)

```typescript
// pages/admin/ZATCA.tsx
import { useState } from 'react';
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/Card';
import { DataTable } from '@/components/ui/DataTable';
import { Badge } from '@/components/ui/Badge';
import { Button } from '@/components/ui/Button';
import { Input } from '@/components/ui/Input';
import { Tabs, TabsContent, TabsList, TabsTrigger } from '@/components/ui/Tabs';
import { AdminPermission } from '@/types/admin-roles';
import { PermissionGuard } from '@/components/admin/auth/PermissionGuard';
import { fetchZATCAInvoices, fetchZATCAStatus, syncZATCA } from '@/lib/api/admin/zatca';
import { toast } from 'react-hot-toast';
import { FileText, RefreshCw, CheckCircle, XCircle, AlertTriangle, Download } from 'lucide-react';

export function AdminZATCA() {
  const queryClient = useQueryClient();
  const [activeTab, setActiveTab] = useState('invoices');
  const [status, setStatus] = useState('');
  const [search, setSearch] = useState('');

  const { data: zatcaStatus } = useQuery({
    queryKey: ['admin-zatca-status'],
    queryFn: fetchZATCAStatus,
  });

  const { data, isLoading } = useQuery({
    queryKey: ['admin-zatca-invoices', { status, search }],
    queryFn: () => fetchZATCAInvoices({ status, search }),
  });

  const syncMutation = useMutation({
    mutationFn: syncZATCA,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['admin-zatca-invoices'] });
      toast.success('تمت المزامنة بنجاح');
    },
    onError: () => toast.error('فشلت المزامنة'),
  });

  return (
    <div className="space-y-6">
      <div className="flex items-center justify-between">
        <h1 className="text-2xl font-bold">إدارة الفواتير (ZATCA)</h1>
        <div className="flex items-center gap-2">
          <Button variant="outline" onClick={() => syncMutation.mutate()}
            loading={syncMutation.isLoading}>
            <RefreshCw className="w-4 h-4 ml-2" />مزامنة
          </Button>
          <Button variant="outline"><Download className="w-4 h-4 ml-2" />تصدير</Button>
        </div>
      </div>

      {/* ZATCA Status */}
      <div className="grid grid-cols-1 md:grid-cols-4 gap-4">
        <Card>
          <CardContent className="p-4">
            <div className="flex items-center gap-3">
              {zatcaStatus?.connected ? (
                <CheckCircle className="w-8 h-8 text-green-500" />
              ) : (
                <XCircle className="w-8 h-8 text-red-500" />
              )}
              <div>
                <p className="text-sm text-neutral-500">حالة الاتصال</p>
                <p className="font-medium">{zatcaStatus?.connected ? 'متصل' : 'غير متصل'}</p>
              </div>
            </div>
          </CardContent>
        </Card>
        <Card>
          <CardContent className="p-4">
            <div className="flex items-center gap-3">
              <FileText className="w-8 h-8 text-blue-500" />
              <div>
                <p className="text-sm text-neutral-500">الفواتير المزامنة</p>
                <p className="text-xl font-bold">{zatcaStatus?.syncedInvoices || 0}</p>
              </div>
            </div>
          </CardContent>
        </Card>
        <Card>
          <CardContent className="p-4">
            <div className="flex items-center gap-3">
              <AlertTriangle className="w-8 h-8 text-yellow-500" />
              <div>
                <p className="text-sm text-neutral-500">بانتظار المزامنة</p>
                <p className="text-xl font-bold">{zatcaStatus?.pendingInvoices || 0}</p>
              </div>
            </div>
          </CardContent>
        </Card>
        <Card>
          <CardContent className="p-4">
            <div className="flex items-center gap-3">
              <XCircle className="w-8 h-8 text-red-500" />
              <div>
                <p className="text-sm text-neutral-500">فواتير مرفوضة</p>
                <p className="text-xl font-bold">{zatcaStatus?.failedInvoices || 0}</p>
              </div>
            </div>
          </CardContent>
        </Card>
      </div>

      <Tabs value={activeTab} onValueChange={setActiveTab}>
        <TabsList>
          <TabsTrigger value="invoices">الفواتير</TabsTrigger>
          <TabsTrigger value="settings">إعدادات ZATCA</TabsTrigger>
        </TabsList>

        <TabsContent value="invoices">
          <div className="flex gap-3 mb-4">
            <Input placeholder="بحث برقم الفاتورة..." value={search}
              onChange={(e) => setSearch(e.target.value)} className="max-w-sm" />
            <select value={status} onChange={(e) => setStatus(e.target.value)}
              className="px-4 py-2 rounded-lg border">
              <option value="">جميع الحالات</option>
              <option value="synced">مزامنة</option>
              <option value="pending">بانتظار المزامنة</option>
              <option value="failed">مرفوضة</option>
            </select>
          </div>

          <DataTable data={data?.invoices || []} columns={[
            { accessorKey: 'invoiceNumber', header: 'رقم الفاتورة' },
            { accessorKey: 'orderNumber', header: 'رقم الطلب' },
            { accessorKey: 'vendorName', header: 'المتجر' },
            { accessorKey: 'amount', header: 'المبلغ',
              cell: ({ row }) => `${row.original.amount.toLocaleString('ar-YE')} يمني` },
            { accessorKey: 'status', header: 'الحالة',
              cell: ({ row }) => (
                <Badge variant={
                  row.original.status === 'synced' ? 'success' :
                  row.original.status === 'failed' ? 'error' : 'warning'
                }>
                  {row.original.status === 'synced' ? 'مزامنة' :
                   row.original.status === 'failed' ? 'مرفوضة' : 'بانتظار المزامنة'}
                </Badge>
              ) },
            { accessorKey: 'zatcaUuid', header: 'ZATCA UUID',
              cell: ({ row }) => (
                <span className="font-mono text-xs">{row.original.zatcaUuid || '-'}</span>
              ) },
            { accessorKey: 'createdAt', header: 'التاريخ',
              cell: ({ row }) => new Date(row.original.createdAt).toLocaleDateString('ar-YE') },
          ]} isLoading={isLoading} emptyMessage="لا توجد فواتير" pagination={data?.pagination} />
        </TabsContent>

        <TabsContent value="settings">
          <Card>
            <CardHeader><CardTitle>إعدادات ZATCA</CardTitle></CardHeader>
            <CardContent className="space-y-4">
              <Input label="API Key" type="password" defaultValue={zatcaStatus?.apiKey || ''} dir="ltr" />
              <Input label="API Secret" type="password" defaultValue={zatcaStatus?.apiSecret || ''} dir="ltr" />
              <Input label="Seller Name" defaultValue={zatcaStatus?.sellerName || ''} />
              <Input label="Seller VAT Number" defaultValue={zatcaStatus?.vatNumber || ''} dir="ltr" />
              <div className="flex justify-end">
                <Button>حفظ الإعدادات</Button>
              </div>
            </CardContent>
          </Card>
        </TabsContent>
      </Tabs>
    </div>
  );
}
```

## Audit Log Viewer Component

```typescript
// components/admin/security/AuditLogViewer.tsx
import { Badge } from '@/components/ui/Badge';

interface AuditLogEntry {
  id: string;
  action: string;
  entity: string;
  entityId: string;
  userId: string;
  userName: string;
  userRole: string;
  details: Record<string, any>;
  ipAddress: string;
  userAgent: string;
  createdAt: string;
}

interface AuditLogViewerProps {
  data: AuditLogEntry[];
  isLoading: boolean;
}

export function AuditLogViewer({ data, isLoading }: AuditLogViewerProps) {
  const getActionBadge = (action: string) => {
    switch (action) {
      case 'create': return <Badge variant="success">إنشاء</Badge>;
      case 'update': return <Badge variant="info">تعديل</Badge>;
      case 'delete': return <Badge variant="error">حذف</Badge>;
      case 'login': return <Badge variant="primary">دخول</Badge>;
      case 'logout': return <Badge variant="neutral">خروج</Badge>;
      default: return <Badge>{action}</Badge>;
    }
  };

  if (isLoading) return <div className="animate-pulse space-y-4">...</div>;

  return (
    <div className="space-y-2">
      {data.map((entry) => (
        <div key={entry.id} className="flex items-start gap-4 p-4 bg-white dark:bg-neutral-800 rounded-xl border border-neutral-200 dark:border-neutral-700">
          <div className="shrink-0">
            {getActionBadge(entry.action)}
          </div>
          <div className="flex-1 min-w-0">
            <p className="text-sm">
              <span className="font-medium">{entry.userName}</span>
              {' '}({entry.userRole}){' '}
              <span className="text-neutral-500">{entry.action}</span>
              {' '}<span className="font-medium">{entry.entity}</span>
              {' '}<span className="text-neutral-500">#{entry.entityId}</span>
            </p>
            <p className="text-xs text-neutral-400 mt-1">
              {new Date(entry.createdAt).toLocaleString('ar-YE')} | {entry.ipAddress}
            </p>
          </div>
        </div>
      ))}
    </div>
  );
}
```

## Restriction Manager

```typescript
// components/admin/security/RestrictionManager.tsx
import { useState } from 'react';
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { Button } from '@/components/ui/Button';
import { Input } from '@/components/ui/Input';
import { Badge } from '@/components/ui/Badge';
import { fetchRestrictions, addRestriction, removeRestriction } from '@/lib/api/admin/security';
import { toast } from 'react-hot-toast';
import { Plus, Trash2, Globe, Hash, Smartphone } from 'lucide-react';

type RestrictionType = 'ip' | 'phone' | 'email' | 'device';

export function RestrictionManager() {
  const queryClient = useQueryClient();
  const [type, setType] = useState<RestrictionType>('ip');
  const [value, setValue] = useState('');
  const [reason, setReason] = useState('');

  const { data: restrictions } = useQuery({
    queryKey: ['admin-restrictions'],
    queryFn: fetchRestrictions,
  });

  const addMutation = useMutation({
    mutationFn: addRestriction,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['admin-restrictions'] });
      toast.success('تمت إضافة القيود بنجاح');
      setValue('');
      setReason('');
    },
  });

  const removeMutation = useMutation({
    mutationFn: removeRestriction,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['admin-restrictions'] });
      toast.success('تمت إزالة القيود بنجاح');
    },
  });

  const icons: Record<RestrictionType, typeof Globe> = {
    ip: Globe,
    phone: Smartphone,
    email: Hash,
    device: Hash,
  };

  const labels: Record<RestrictionType, string> = {
    ip: 'عنوان IP',
    phone: 'رقم الجوال',
    email: 'البريد الإلكتروني',
    device: 'معرف الجهاز',
  };

  return (
    <div className="space-y-6">
      <div className="bg-white dark:bg-neutral-800 rounded-xl p-6">
        <h3 className="text-lg font-semibold mb-4">إضافة قيد جديد</h3>
        <div className="flex flex-wrap gap-3">
          <select value={type} onChange={(e) => setType(e.target.value as RestrictionType)}
            className="px-4 py-2 rounded-lg border">
            {Object.entries(labels).map(([key, label]) => (
              <option key={key} value={key}>{label}</option>
            ))}
          </select>
          <Input placeholder={labels[type]} value={value}
            onChange={(e) => setValue(e.target.value)} className="flex-1" />
          <Input placeholder="السبب" value={reason}
            onChange={(e) => setReason(e.target.value)} className="flex-1" />
          <Button onClick={() => addMutation.mutate({ type, value, reason })}
            loading={addMutation.isLoading}>
            <Plus className="w-4 h-4 ml-2" />إضافة
          </Button>
        </div>
      </div>

      <div className="space-y-2">
        {restrictions?.map((restriction: any) => {
          const Icon = icons[restriction.type];
          return (
            <div key={restriction.id}
              className="flex items-center justify-between p-4 bg-white dark:bg-neutral-800 rounded-xl border">
              <div className="flex items-center gap-3">
                <Icon className="w-5 h-5 text-neutral-500" />
                <div>
                  <p className="font-medium">{restriction.value}</p>
                  <p className="text-sm text-neutral-500">{restriction.reason}</p>
                </div>
              </div>
              <button onClick={() => removeMutation.mutate(restriction.id)}
                className="p-2 hover:bg-red-50 text-red-500 rounded-lg">
                <Trash2 className="w-4 h-4" />
              </button>
            </div>
          );
        })}
      </div>
    </div>
  );
}
```

## Project Structure

```
admin-panel/
├── src/
│   ├── pages/admin/
│   │   ├── Dashboard.tsx
│   │   ├── Customers.tsx
│   │   ├── Vendors.tsx
│   │   ├── Orders.tsx
│   │   ├── Products.tsx
│   │   ├── Finance.tsx
│   │   ├── Coupons.tsx
│   │   ├── Content.tsx
│   │   ├── Settings.tsx
│   │   ├── Support.tsx
│   │   ├── Security.tsx
│   │   ├── ZATCA.tsx
│   │   └── Login.tsx
│   ├── layouts/
│   │   └── AdminLayout.tsx
│   ├── components/admin/
│   │   ├── layout/
│   │   │   ├── Sidebar.tsx
│   │   │   └── TopBar.tsx
│   │   ├── dashboard/
│   │   │   ├── StatsCard.tsx
│   │   │   ├── RevenueOverview.tsx
│   │   │   ├── OrdersChart.tsx
│   │   │   ├── VendorActivity.tsx
│   │   │   ├── CustomerGrowth.tsx
│   │   │   ├── RecentActivity.tsx
│   │   │   └── SystemAlerts.tsx
│   │   ├── customers/
│   │   │   └── CustomerDetailDrawer.tsx
│   │   ├── vendors/
│   │   │   ├── VendorDetailDrawer.tsx
│   │   │   └── KYCReviewModal.tsx
│   │   ├── orders/
│   │   │   └── AdminOrderDetail.tsx
│   │   ├── coupons/
│   │   │   └── AdminCouponForm.tsx
│   │   ├── finance/
│   │   │   ├── RevenueChart.tsx
│   │   │   └── PayoutQueue.tsx
│   │   ├── content/
│   │   │   ├── BannerManager.tsx
│   │   │   ├── CategoryManager.tsx
│   │   │   ├── PageManager.tsx
│   │   │   └── NotificationTemplates.tsx
│   │   ├── settings/
│   │   │   ├── GeneralSettings.tsx
│   │   │   ├── PaymentSettings.tsx
│   │   │   ├── ShippingSettings.tsx
│   │   │   ├── NotificationSettings.tsx
│   │   │   └── PlatformFees.tsx
│   │   ├── support/
│   │   │   └── TicketDetail.tsx
│   │   ├── security/
│   │   │   ├── AuditLogViewer.tsx
│   │   │   ├── RestrictionManager.tsx
│   │   │   └── SecurityEvents.tsx
│   │   ├── zatca/
│   │   │   └── ZATCASettings.tsx
│   │   └── auth/
│   │       └── PermissionGuard.tsx
│   ├── stores/
│   │   └── adminAuthStore.ts
│   ├── hooks/
│   │   └── useAdminPermissions.ts
│   ├── types/
│   │   └── admin-roles.ts
│   ├── lib/api/admin/
│   │   ├── dashboard.ts
│   │   ├── customers.ts
│   │   ├── vendors.ts
│   │   ├── orders.ts
│   │   ├── products.ts
│   │   ├── finance.ts
│   │   ├── coupons.ts
│   │   ├── content.ts
│   │   ├── settings.ts
│   │   ├── support.ts
│   │   ├── security.ts
│   │   └── zatca.ts
│   ├── App.tsx
│   └── main.tsx
├── package.json
├── tsconfig.json
├── vite.config.ts
├── tailwind.config.ts
└── .env.local
```
