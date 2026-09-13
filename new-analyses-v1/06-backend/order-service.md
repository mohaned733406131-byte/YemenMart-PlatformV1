# Order Service

## Overview

Order management handles the full lifecycle from cart to delivery. Master/sub-order architecture splits each customer order by seller. A 17-state state machine governs all transitions.

---

## Cart to Order Conversion

### Add to Cart

```typescript
// packages/order-module/src/cart.service.ts
@injectable()
export class CartService {
  constructor(
    @inject('PrismaClient') private prisma: PrismaClient,
    @inject('ProductService') private products: IProductService,
  ) {}

  async addToCart(userId: string, input: AddToCartInput): Promise<Cart> {
    // Validate product exists and is active
    const product = await this.products.getProduct(input.productId);
    if (product.status !== 'ACTIVE') throw new ProductNotAvailableError(input.productId);

    // Check stock
    const available = await this.products.getAvailableStock(input.productId, input.variantId);
    if (available < input.quantity) {
      throw new InsufficientStockError(input.productId, input.quantity, available);
    }

    // Upsert cart item
    const cartItem = await this.prisma.cartItem.upsert({
      where: {
        userId_productId_variantId: {
          userId,
          productId: input.productId,
          variantId: input.variantId || null,
        },
      },
      create: {
        userId,
        productId: input.productId,
        variantId: input.variantId,
        quantity: input.quantity,
        price: product.basePrice,
        name: product.name,
        storeId: product.storeId,
        imageUrl: product.images[0]?.url,
      },
      update: {
        quantity: { increment: input.quantity },
      },
    });

    return this.getCart(userId);
  }

  async getCart(userId: string): Promise<Cart> {
    const items = await this.prisma.cartItem.findMany({
      where: { userId },
      include: { product: { select: { status: true, stockQuantity: true } } },
      orderBy: { createdAt: 'desc' },
    });

    // Validate stock for each item
    const validatedItems = items.map(item => ({
      ...item,
      inStock: item.product.status === 'ACTIVE' && item.product.stockQuantity >= item.quantity,
    }));

    const subtotal = validatedItems.reduce(
      (sum, item) => sum + Number(item.price) * item.quantity,
      0,
    );

    // Group by store for display
    const itemsByStore = groupBy(validatedItems, 'storeId');

    return {
      items: validatedItems,
      itemCount: validatedItems.reduce((sum, i) => sum + i.quantity, 0),
      subtotal: new Money(subtotal, 'YER'),
      itemsByStore,
      hasUnavailableItems: validatedItems.some(i => !i.inStock),
    };
  }

  async updateQuantity(userId: string, cartItemId: string, quantity: number): Promise<Cart> {
    if (quantity <= 0) return this.removeItem(userId, cartItemId);

    const item = await this.prisma.cartItem.findFirst({
      where: { id: cartItemId, userId },
    });
    if (!item) throw new CartItemNotFoundError();

    // Check stock
    const available = await this.products.getAvailableStock(item.productId, item.variantId);
    if (available < quantity) {
      throw new InsufficientStockError(item.productId, quantity, available);
    }

    await this.prisma.cartItem.update({
      where: { id: cartItemId },
      data: { quantity },
    });

    return this.getCart(userId);
  }

  async removeItem(userId: string, cartItemId: string): Promise<Cart> {
    await this.prisma.cartItem.deleteMany({
      where: { id: cartItemId, userId },
    });
    return this.getCart(userId);
  }

  async clearCart(userId: string): Promise<void> {
    await this.prisma.cartItem.deleteMany({ where: { userId } });
  }
}
```

---

## Stock Soft-Hold (15 Minutes)

```typescript
// packages/order-module/src/stock-hold.service.ts
@injectable()
export class StockHoldService {
  constructor(
    @inject('PrismaClient') private prisma: PrismaClient,
    @inject('RedisClient') private redis: Redis,
    @inject('EventBus') private events: EventBus,
  ) {}

  private HOLD_DURATION = 15 * 60; // 15 minutes in seconds

  async holdStock(
    items: OrderItemInput[],
    userId: string,
  ): Promise<StockHoldResult> {
    const holdId = uuidv4();

    return this.prisma.$transaction(async (tx) => {
      const holds: StockHold[] = [];

      for (const item of items) {
        const product = await tx.product.findUnique({
          where: { id: item.productId },
        });

        if (!product) throw new ProductNotFoundError(item.productId);

        const available = product.stockQuantity - product.softHoldQty;
        if (available < item.quantity) {
          throw new InsufficientStockError(item.productId, item.quantity, available);
        }

        // Create soft hold
        await tx.stockHold.create({
          data: {
            holdId,
            productId: item.productId,
            variantId: item.variantId,
            userId,
            quantity: item.quantity,
            expiresAt: new Date(Date.now() + this.HOLD_DURATION * 1000),
          },
        });

        // Increment soft hold quantity
        await tx.product.update({
          where: { id: item.productId },
          data: { softHoldQty: { increment: item.quantity } },
        });

        holds.push({
          productId: item.productId,
          quantity: item.quantity,
          expiresAt: new Date(Date.now() + this.HOLD_DURATION * 1000),
        });
      }

      // Store hold ID in Redis for quick lookup
      await this.redis.setex(
        `stock-hold:${holdId}`,
        this.HOLD_DURATION,
        JSON.stringify({ userId, items, createdAt: Date.now() }),
      );

      this.events.emit('stock.held', { holdId, userId, items });

      return { holdId, holds, expiresIn: this.HOLD_DURATION };
    });
  }

  async confirmHold(holdId: string): Promise<void> {
    const holds = await this.prisma.stockHold.findMany({
      where: { holdId, confirmed: false },
    });

    if (!holds.length) throw new StockHoldNotFoundError(holdId);

    return this.prisma.$transaction(async (tx) => {
      for (const hold of holds) {
        // Convert soft hold to actual stock decrement
        await tx.product.update({
          where: { id: hold.productId },
          data: {
            stockQuantity: { decrement: hold.quantity },
            softHoldQty: { decrement: hold.quantity },
          },
        });

        await tx.stockHold.update({
          where: { id: hold.id },
          data: { confirmed: true },
        });
      }

      await this.redis.del(`stock-hold:${holdId}`);
    });
  }

  async releaseHold(holdId: string): Promise<void> {
    const holds = await this.prisma.stockHold.findMany({
      where: { holdId, confirmed: false },
    });

    return this.prisma.$transaction(async (tx) => {
      for (const hold of holds) {
        await tx.product.update({
          where: { id: hold.productId },
          data: { softHoldQty: { decrement: hold.quantity } },
        });

        await tx.stockHold.delete({ where: { id: hold.id } });
      }

      await this.redis.del(`stock-hold:${holdId}`);
    });
  }

  // Background job: Release expired holds
  async releaseExpiredHolds(): Promise<number> {
    const expired = await this.prisma.stockHold.findMany({
      where: {
        confirmed: false,
        expiresAt: { lte: new Date() },
      },
    });

    let released = 0;
    for (const hold of expired) {
      try {
        await this.prisma.$transaction(async (tx) => {
          await tx.product.update({
            where: { id: hold.productId },
            data: { softHoldQty: { decrement: hold.quantity } },
          });
          await tx.stockHold.delete({ where: { id: hold.id } });
        });
        released++;
      } catch (error) {
        this.logger.error(`Failed to release stock hold ${hold.holdId}`, error);
      }
    }

    return released;
  }
}
```

---

## Order Creation

```typescript
// packages/order-module/src/order.service.ts
@injectable()
export class OrderService implements IOrderService {
  constructor(
    @inject('PrismaClient') private prisma: PrismaClient,
    @inject('StockHoldService') private stockHolds: StockHoldService,
    @inject('PaymentService') private payments: IPaymentService,
    @inject('CommissionService') private commissions: ICommissionService,
    @inject('CouponService') private coupons: ICouponService,
    @inject('LoyaltyService') private loyalty: ILoyaltyService,
    @inject('EventBus') private events: EventBus,
  ) {}

  async createOrder(input: CreateOrderInput): Promise<MasterOrder> {
    return this.prisma.$transaction(async (tx) => {
      // 1. Validate idempotency
      const existing = await tx.order.findFirst({
        where: { userId: input.userId, idempotencyKey: input.idempotencyKey },
      });
      if (existing) throw new DuplicateOrderError(input.idempotencyKey);

      // 2. Hold stock
      const stockHold = await this.stockHolds.holdStock(input.items, input.userId);

      // 3. Calculate subtotal
      const subtotal = input.items.reduce(
        (sum, item) => sum + item.price * item.quantity,
        0,
      );

      // 4. Apply coupon
      let couponDiscount = 0;
      if (input.couponCode) {
        const couponResult = await this.coupons.validateAndApply(
          input.couponCode, input.items, input.userId, tx,
        );
        couponDiscount = couponResult.discount.value;
      }

      // 5. Calculate delivery fee
      const deliveryFee = await this.calculateDeliveryFee(input.addressId, input.items);

      // 6. Calculate loyalty redemption
      let pointsDiscount = 0;
      if (input.usePoints) {
        pointsDiscount = await this.loyalty.calculateRedemption(input.userId, subtotal);
      }

      // 7. Calculate total
      const total = subtotal + deliveryFee - couponDiscount - pointsDiscount;

      // 8. Debit wallet
      await this.payments.debitWallet(input.userId, {
        amount: new Money(total, 'YER'),
        orderId: 'pending', // Will be updated
        idempotencyKey: `${input.idempotencyKey}:payment`,
        description: `Order payment`,
      }, tx);

      // 9. Group items by store for sub-orders
      const itemsByStore = groupBy(input.items, 'storeId');

      // 10. Create master order
      const masterOrder = await tx.order.create({
        data: {
          userId: input.userId,
          type: 'MASTER',
          status: 'CONFIRMED',
          total,
          currency: 'YER',
          subtotal,
          deliveryFee,
          discount: couponDiscount,
          couponCode: input.couponCode,
          pointsUsed: input.usePoints ? pointsDiscount * 100 : 0,
          addressId: input.addressId,
          notes: input.notes,
          idempotencyKey: input.idempotencyKey,
        },
      });

      // 11. Create sub-orders per seller
      const subOrders: SubOrder[] = [];
      for (const [storeId, items] of Object.entries(itemsByStore)) {
        const subTotal = items.reduce((sum, i) => sum + i.price * i.quantity, 0);

        // Calculate commission
        const commission = await this.commissions.calculate(storeId, new Money(subTotal, 'YER'));

        const sellerPayout = subTotal - commission.totalFee;

        const subOrder = await tx.order.create({
          data: {
            userId: input.userId,
            type: 'SUB',
            parentId: masterOrder.id,
            storeId,
            status: 'CONFIRMED',
            total: subTotal,
            currency: 'YER',
            subtotal: subTotal,
            commission: commission.totalFee,
            sellerPayout,
            items: {
              create: items.map(item => ({
                productId: item.productId,
                variantId: item.variantId,
                name: item.name,
                price: item.price,
                quantity: item.quantity,
                total: item.price * item.quantity,
              })),
            },
          },
          include: { items: true },
        });

        subOrders.push(subOrder as SubOrder);
      }

      // 12. Hold escrow for 7 days
      await this.payments.holdEscrow(masterOrder.id, new Money(total, 'YER'), { days: 7 });

      // 13. Apply coupon usage
      if (input.couponCode) {
        await this.coupons.recordUsage(input.couponCode, input.userId, masterOrder.id);
      }

      // 14. Award loyalty points
      if (!input.usePoints) {
        await this.loyalty.awardPoints(input.userId, new Money(total, 'YER'), masterOrder.id);
      }

      // 15. Confirm stock hold
      await this.stockHolds.confirmHold(stockHold.holdId);

      // 16. Clear cart
      await tx.cartItem.deleteMany({ where: { userId: input.userId } });

      // 17. Publish event
      this.events.emit('order.placed', {
        orderId: masterOrder.id,
        userId: input.userId,
        total,
        subOrders: subOrders.map(s => s.id),
      });

      return { ...masterOrder, subOrders } as MasterOrder;
    });
  }

  async getOrder(orderId: string, userId: string): Promise<OrderDetail> {
    const order = await this.prisma.order.findFirst({
      where: { id: orderId, userId, type: 'MASTER' },
      include: {
        items: true,
        subOrders: {
          include: {
            items: true,
            store: { select: { id: true, name: true, slug: true, logo: true } },
          },
        },
        statusHistory: { orderBy: { createdAt: 'desc' }, take: 10 },
        deliveryCode: { select: { expiresAt: true, used: true } },
        escrow: true,
      },
    });

    if (!order) throw new OrderNotFoundError(orderId);

    return order as OrderDetail;
  }
}
```

---

## 17-State State Machine

```typescript
// packages/order-module/src/order-state-machine.ts
@injectable()
export class OrderStateMachine {
  private transitions: Record<OrderStatus, OrderStatus[]> = {
    PENDING:           ['CONFIRMED', 'CANCELLED', 'ON_HOLD'],
    ON_HOLD:           ['CONFIRMED', 'CANCELLED'],
    CONFIRMED:         ['PROCESSING', 'CANCELLED'],
    PROCESSING:        ['SHIPPED', 'CANCELLED'],
    SHIPPED:           ['OUT_FOR_DELIVERY', 'RETURNED'],
    OUT_FOR_DELIVERY:  ['DELIVERED', 'DELIVERY_FAILED'],
    DELIVERED:         ['COMPLETED', 'RETURN_REQUESTED', 'DISPUTE_OPENED'],
    DELIVERY_FAILED:   ['SHIPPED', 'CANCELLED'],
    COMPLETED:         ['RETURN_REQUESTED'],
    RETURN_REQUESTED:  ['RETURN_APPROVED', 'RETURN_REJECTED'],
    RETURN_APPROVED:   ['RETURNED'],
    RETURN_REJECTED:   ['COMPLETED'],
    RETURNED:          ['REFUNDING'],
    REFUNDING:         ['REFUNDED'],
    REFUNDED:          [],
    CANCELLED:         [],
    DISPUTE_OPENED:    ['DISPUTE_RESOLVED'],
    DISPUTE_RESOLVED:  [],
  };

  private sideEffects: Partial<Record<OrderStatus, SideEffectFn[]>> = {
    CONFIRMED: [this.onConfirmed.bind(this)],
    SHIPPED: [this.onShipped.bind(this)],
    DELIVERED: [this.onDelivered.bind(this)],
    COMPLETED: [this.onCompleted.bind(this)],
    CANCELLED: [this.onCancelled.bind(this)],
    RETURN_APPROVED: [this.onReturnApproved.bind(this)],
    REFUNDED: [this.onRefunded.bind(this)],
  };

  async transition(
    orderId: string,
    toStatus: OrderStatus,
    actor: ActorInfo,
    metadata?: TransitionMetadata,
  ): Promise<Order> {
    const order = await this.prisma.order.findUnique({ where: { id: orderId } });
    if (!order) throw new OrderNotFoundError(orderId);

    const fromStatus = order.status;
    if (!this.canTransition(fromStatus, toStatus)) {
      throw new InvalidTransitionError(fromStatus, toStatus);
    }

    // Update status
    const updated = await this.prisma.order.update({
      where: { id: orderId },
      data: { status: toStatus, updatedAt: new Date() },
    });

    // Record history
    await this.prisma.orderStatusHistory.create({
      data: {
        orderId,
        from: fromStatus,
        to: toStatus,
        actorId: actor.id,
        actorRole: actor.role,
        notes: metadata?.notes,
      },
    });

    // Execute side effects
    const effects = this.sideEffects[toStatus] || [];
    for (const effect of effects) {
      await effect(updated, metadata);
    }

    return updated;
  }

  canTransition(from: OrderStatus, to: OrderStatus): boolean {
    return this.transitions[from]?.includes(to) ?? false;
  }

  private async onConfirmed(order: Order): Promise<void> {
    this.events.emit('order.confirmed', { orderId: order.id });
  }

  private async onShipped(order: Order): Promise<void> {
    // Generate delivery code
    await this.deliveryService.generateDeliveryCode(order.id);
    this.events.emit('order.shipped', { orderId: order.id });
  }

  private async onDelivered(order: Order): Promise<void> {
    this.events.emit('order.delivered', { orderId: order.id, userId: order.userId });
  }

  private async onCompleted(order: Order): Promise<void> {
    // Release escrow to sellers
    await this.payments.releaseEscrow(order.id);
    this.events.emit('order.completed', { orderId: order.id });
  }

  private async onCancelled(order: Order): Promise<void> {
    // Release held stock
    await this.stockHolds.releaseAllHolds(order.id);
    // Refund buyer
    await this.refunds.processRefund(order.id, { reason: 'ORDER_CANCELLED', partial: false });
    this.events.emit('order.cancelled', { orderId: order.id });
  }

  private async onReturnApproved(order: Order): Promise<void> {
    this.events.emit('order.return_approved', { orderId: order.id });
  }

  private async onRefunded(order: Order): Promise<void> {
    this.events.emit('order.refunded', { orderId: order.id });
  }
}
```

### Status Update Endpoint

```typescript
// PUT /orders/:id/status
@AuthGuard()
async updateOrderStatus(req: Request, res: Response) {
  const { status, notes } = req.body;
  const order = await this.orderStateMachine.transition(
    req.params.id,
    status,
    { id: req.user.sub, role: req.user.role },
    { notes },
  );
  res.json({ success: true, data: order });
}
```

---

## Return Request Handling

```typescript
// packages/order-module/src/return.service.ts
@injectable()
export class ReturnService {
  constructor(
    @inject('PrismaClient') private prisma: PrismaClient,
    @inject('OrderStateMachine') private stateMachine: OrderStateMachine,
    @inject('RefundService') private refunds: RefundService,
  ) {}

  async requestReturn(
    orderId: string,
    userId: string,
    input: ReturnInput,
  ): Promise<ReturnRequest> {
    const order = await this.prisma.order.findFirst({
      where: { id: orderId, userId, type: 'MASTER' },
    });

    if (!order) throw new OrderNotFoundError(orderId);
    if (!this.stateMachine.canTransition(order.status, 'RETURN_REQUESTED')) {
      throw new ReturnNotAllowedError(order.status);
    }

    // Check return window (7 days from delivery)
    const deliveredAt = order.updatedAt;
    const returnDeadline = new Date(deliveredAt);
    returnDeadline.setDate(returnDeadline.getDate() + 7);
    if (new Date() > returnDeadline) {
      throw new ReturnWindowExpiredError();
    }

    const returnRequest = await this.prisma.returnRequest.create({
      data: {
        orderId,
        userId,
        reason: input.reason,
        description: input.description,
        items: input.items, // { productId, quantity, reason }
        status: 'REQUESTED',
      },
    });

    await this.stateMachine.transition(orderId, 'RETURN_REQUESTED', {
      id: userId,
      role: 'BUYER',
    }, { returnId: returnRequest.id });

    return returnRequest;
  }

  async approveReturn(
    returnId: string,
    adminId: string,
    resolution: string,
  ): Promise<void> {
    const returnRequest = await this.prisma.returnRequest.findUnique({
      where: { id: returnId },
      include: { order: true },
    });

    if (!returnRequest) throw new ReturnNotFoundError(returnId);

    await this.prisma.$transaction(async (tx) => {
      await tx.returnRequest.update({
        where: { id: returnId },
        data: {
          status: 'APPROVED',
          resolution,
          reviewedAt: new Date(),
          reviewedBy: adminId,
        },
      });

      await this.stateMachine.transition(
        returnRequest.orderId,
        'RETURN_APPROVED',
        { id: adminId, role: 'ADMIN' },
        { returnId },
      );

      // Initiate refund
      await this.refunds.processRefund(
        returnRequest.orderId,
        { reason: returnRequest.reason, partial: false },
      );
    });
  }

  async rejectReturn(
    returnId: string,
    adminId: string,
    reason: string,
  ): Promise<void> {
    const returnRequest = await this.prisma.returnRequest.findUnique({
      where: { id: returnId },
    });

    if (!returnRequest) throw new ReturnNotFoundError(returnId);

    await this.prisma.$transaction(async (tx) => {
      await tx.returnRequest.update({
        where: { id: returnId },
        data: {
          status: 'REJECTED',
          resolution: reason,
          reviewedAt: new Date(),
          reviewedBy: adminId,
        },
      });

      await this.stateMachine.transition(
        returnRequest.orderId,
        'RETURN_REJECTED',
        { id: adminId, role: 'ADMIN' },
        { returnId },
      );
    });
  }
}
```

---

## Delivery Fee Calculation

```typescript
private async calculateDeliveryFee(
  addressId: string,
  items: OrderItemInput[],
): Promise<number> {
  const address = await this.prisma.address.findUnique({ where: { id: addressId } });
  if (!address) throw new AddressNotFoundError(addressId);

  // Get store locations for items
  const storeIds = [...new Set(items.map(i => i.storeId))];
  const stores = await this.prisma.store.findMany({
    where: { id: { in: storeIds } },
  });

  // Calculate distance-based fee (simplified)
  const baseFee = 500; // Base delivery fee in YER
  const perItemFee = 100; // Additional per-item fee
  const totalItems = items.reduce((sum, i) => sum + i.quantity, 0);

  return baseFee + (totalItems * perItemFee);
}
```

---

## API Endpoints

| Method | Endpoint                          | Description              | Auth     |
|--------|-----------------------------------|--------------------------|----------|
| POST   | /cart/items                       | Add item to cart         | Buyer    |
| GET    | /cart                             | Get cart                 | Buyer    |
| PUT    | /cart/items/:id                   | Update item quantity     | Buyer    |
| DELETE | /cart/items/:id                   | Remove item from cart    | Buyer    |
| POST   | /orders                           | Create order from cart   | Buyer    |
| GET    | /orders                           | List user orders         | Buyer    |
| GET    | /orders/:id                       | Get order detail         | Buyer    |
| PUT    | /orders/:id/cancel                | Cancel order             | Buyer    |
| POST   | /orders/:id/return                | Request return           | Buyer    |
| GET    | /stores/:id/orders                | List store orders        | Seller   |
| PUT    | /orders/:id/status                | Update order status      | Seller   |
| POST   | /orders/:id/delivery/verify       | Verify delivery code     | Agent    |
| POST   | /admin/orders/:id/status          | Admin status update      | Admin    |
| POST   | /admin/returns/:id/approve        | Approve return           | Admin    |
| POST   | /admin/returns/:id/reject         | Reject return            | Admin    |
