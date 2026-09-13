# Notification Service

## Overview

Multi-channel notification system supporting SMS, WhatsApp, Email, Push, and In-App delivery. Features template management, retry logic, rate limiting, user preferences, and batch notifications.

---

## Multi-Channel Delivery

### Channel Providers

```typescript
export interface IChannelProvider {
  send(input: ChannelMessage): Promise<ChannelResult>;
  name: string;
}

export interface ChannelMessage {
  userId: string;
  to: string;
  type: NotificationType;
  subject?: string;
  body: string;
  htmlBody?: string;
  data?: Record<string, any>;
}

export interface ChannelResult {
  channel: NotificationChannel;
  success: boolean;
  messageId?: string;
  error?: string;
  deliveredAt?: Date;
}
```

```typescript
@injectable()
export class SmsProvider implements IChannelProvider {
  name = 'sms';

  constructor(@inject('Config') private config: SmsConfig) {}

  async send(input: ChannelMessage): Promise<ChannelResult> {
    try {
      const result = await this.config.client.messages.create({
        body: input.body,
        from: this.config.fromNumber,
        to: input.to,
      });
      return { channel: 'SMS', success: true, messageId: result.sid, deliveredAt: new Date() };
    } catch (error) {
      return { channel: 'SMS', success: false, error: error.message };
    }
  }
}
```

```typescript
@injectable()
export class WhatsAppProvider implements IChannelProvider {
  name = 'whatsapp';

  constructor(@inject('Config') private config: WhatsAppConfig) {}

  async send(input: ChannelMessage): Promise<ChannelResult> {
    try {
      const result = await this.config.client.messages.send({
        from: this.config.fromNumber,
        to: `whatsapp:${input.to}`,
        body: input.body,
      });
      return { channel: 'WHATSAPP', success: true, messageId: result.sid, deliveredAt: new Date() };
    } catch (error) {
      return { channel: 'WHATSAPP', success: false, error: error.message };
    }
  }
}
```

```typescript
@injectable()
export class EmailProvider implements IChannelProvider {
  name = 'email';

  constructor(@inject('Config') private config: EmailConfig) {}

  async send(input: ChannelMessage): Promise<ChannelResult> {
    try {
      const result = await this.config.transporter.sendMail({
        from: this.config.fromAddress,
        to: input.to,
        subject: input.subject,
        html: input.htmlBody,
        text: input.body,
      });
      return { channel: 'EMAIL', success: true, messageId: result.messageId, deliveredAt: new Date() };
    } catch (error) {
      return { channel: 'EMAIL', success: false, error: error.message };
    }
  }
}
```

```typescript
@injectable()
export class PushProvider implements IChannelProvider {
  name = 'push';

  constructor(@inject('Config') private config: PushConfig) {}

  async send(input: ChannelMessage): Promise<ChannelResult> {
    try {
      const messageId = await this.config.admin.messaging().send({
        token: input.to,
        notification: { title: input.subject, body: input.body },
        data: input.data as Record<string, string>,
      });
      return { channel: 'PUSH', success: true, messageId, deliveredAt: new Date() };
    } catch (error) {
      return { channel: 'PUSH', success: false, error: error.message };
    }
  }
}
```

```typescript
@injectable()
export class InAppProvider implements IChannelProvider {
  name = 'in_app';

  constructor(@inject('PrismaClient') private prisma: PrismaClient) {}

  async send(input: ChannelMessage): Promise<ChannelResult> {
    const notification = await this.prisma.notification.create({
      data: {
        userId: input.userId,
        type: input.type,
        channel: 'IN_APP',
        title: input.subject || '',
        body: input.body,
        data: input.data,
        status: 'SENT',
        sentAt: new Date(),
      },
    });
    return { channel: 'IN_APP', success: true, messageId: notification.id, deliveredAt: new Date() };
  }
}
```

---

## Template Management

```typescript
export interface NotificationTemplate {
  id: string;
  type: NotificationType;
  channel: NotificationChannel;
  subject?: string;
  body: string;
  htmlBody?: string;
}

const TEMPLATES: Record<string, NotificationTemplate> = {
  'ORDER_CONFIRMED:sms': {
    id: 'order_confirmed_sms',
    type: 'ORDER_CONFIRMED',
    channel: 'SMS',
    body: 'YemenMart: Your order #{orderId} confirmed. Total: {total} YER.',
  },
  'ORDER_CONFIRMED:email': {
    id: 'order_confirmed_email',
    type: 'ORDER_CONFIRMED',
    channel: 'EMAIL',
    subject: 'Order Confirmed - #{orderId}',
    body: 'Your order has been confirmed.',
    htmlBody: '<h2>Order Confirmed</h2><p>Order #{orderId}</p><p>Total: {total} YER</p>',
  },
  'ORDER_SHIPPED:sms': {
    id: 'order_shipped_sms',
    type: 'ORDER_SHIPPED',
    channel: 'SMS',
    body: 'YemenMart: Order #{orderId} shipped! Delivery code: {deliveryCode}',
  },
  'DELIVERY_CODE:sms': {
    id: 'delivery_code_sms',
    type: 'DELIVERY_CODE',
    channel: 'SMS',
    body: 'YemenMart delivery code: {code}. Valid 24h. Show to driver on delivery.',
  },
  'WALLET_TOPUP:sms': {
    id: 'wallet_topup_sms',
    type: 'WALLET_TOPUP',
    channel: 'SMS',
    body: 'YemenMart: Wallet credited with {amount} YER. Balance: {balance} YER.',
  },
  'PASSWORD_RESET:email': {
    id: 'password_reset_email',
    type: 'SYSTEM',
    channel: 'EMAIL',
    subject: 'Reset Your Password',
    body: 'Click the link to reset your password.',
    htmlBody: '<h2>Password Reset</h2><a href="{resetUrl}">Reset Password</a><p>Expires in 1 hour.</p>',
  },
  'PROMOTION:push': {
    id: 'promo_push',
    type: 'PROMOTION',
    channel: 'PUSH',
    subject: '{title}',
    body: '{message}',
  },
};

function renderTemplate(template: NotificationTemplate, variables: Record<string, string>): NotificationTemplate {
  let body = template.body;
  let subject = template.subject;
  let htmlBody = template.htmlBody;

  for (const [key, value] of Object.entries(variables)) {
    const placeholder = `{${key}}`;
    body = body.replace(new RegExp(placeholder, 'g'), value);
    if (subject) subject = subject.replace(new RegExp(placeholder, 'g'), value);
    if (htmlBody) htmlBody = htmlBody.replace(new RegExp(placeholder, 'g'), value);
  }

  return { ...template, body, subject, htmlBody };
}

export function getTemplate(type: NotificationType, channel: NotificationChannel): NotificationTemplate | null {
  return TEMPLATES[`${type}:${channel}`] || null;
}
```

---

## Core Notification Service

```typescript
@injectable()
export class NotificationService implements INotificationService {
  private providers: Map<NotificationChannel, IChannelProvider>;
  private readonly MAX_RETRIES = 3;
  private readonly RETRY_DELAYS = [1000, 5000, 15000]; // exponential backoff

  constructor(
    @inject('SmsProvider') sms: SmsProvider,
    @inject('WhatsAppProvider') whatsapp: WhatsAppProvider,
    @inject('EmailProvider') email: EmailProvider,
    @inject('PushProvider') push: PushProvider,
    @inject('InAppProvider') inApp: InAppProvider,
    @inject('PrismaClient') private prisma: PrismaClient,
    @inject('RedisClient') private redis: Redis,
    @inject('Logger') private logger: ILogger,
  ) {
    this.providers = new Map([
      ['SMS', sms],
      ['WHATSAPP', whatsapp],
      ['EMAIL', email],
      ['PUSH', push],
      ['IN_APP', inApp],
    ]);
  }

  async send(input: NotificationInput): Promise<NotificationResult> {
    // Check user preferences
    const prefs = await this.getPreferences(input.userId);
    if (!this.isChannelAllowed(input.channel, prefs)) {
      return { success: false, channel: input.channel, skipped: true, reason: 'user_preference' };
    }

    // Rate limiting
    const allowed = await this.checkRateLimit(input.userId, input.channel);
    if (!allowed) {
      return { success: false, channel: input.channel, skipped: true, reason: 'rate_limit' };
    }

    // Get template and render
    const template = getTemplate(input.type, input.channel);
    const rendered = template
      ? renderTemplate(template, input.variables || {})
      : { body: input.body, subject: input.subject };

    // Resolve recipient
    const recipient = await this.resolveRecipient(input.userId, input.channel);

    // Create notification record
    const record = await this.prisma.notification.create({
      data: {
        userId: input.userId,
        type: input.type,
        channel: input.channel,
        title: rendered.subject || input.type,
        body: rendered.body,
        data: input.variables,
        status: 'PENDING',
      },
    });

    // Send with retry
    const result = await this.sendWithRetry(record.id, recipient, {
      ...input,
      ...rendered,
    });

    return result;
  }

  async sendBatch(inputs: NotificationInput[]): Promise<NotificationResult[]> {
    const results: NotificationResult[] = [];
    const batches = this.chunkArray(inputs, 50);

    for (const batch of batches) {
      const batchResults = await Promise.allSettled(
        batch.map(input => this.send(input)),
      );

      for (const result of batchResults) {
        if (result.status === 'fulfilled') {
          results.push(result.value);
        } else {
          results.push({ success: false, channel: 'SMS', error: result.reason.message });
        }
      }
    }

    return results;
  }

  private async sendWithRetry(
    notificationId: string,
    recipient: string,
    input: NotificationInput,
  ): Promise<NotificationResult> {
    const provider = this.providers.get(input.channel);
    if (!provider) {
      throw new UnsupportedChannelError(input.channel);
    }

    let lastError: string = '';

    for (let attempt = 0; attempt <= this.MAX_RETRIES; attempt++) {
      try {
        const result = await provider.send({
          userId: input.userId,
          to: recipient,
          type: input.type,
          subject: input.subject,
          body: input.body,
        });

        if (result.success) {
          await this.prisma.notification.update({
            where: { id: notificationId },
            data: { status: 'SENT', sentAt: new Date() },
          });
          return { success: true, channel: input.channel, messageId: result.messageId };
        }

        lastError = result.error || 'Unknown error';
        this.logger.warn(`Notification attempt ${attempt + 1} failed`, {
          notificationId,
          channel: input.channel,
          error: lastError,
        });
      } catch (error) {
        lastError = error.message;
      }

      // Wait before retry (exponential backoff)
      if (attempt < this.MAX_RETRIES) {
        await this.sleep(this.RETRY_DELAYS[attempt]);
      }
    }

    // All retries exhausted
    await this.prisma.notification.update({
      where: { id: notificationId },
      data: { status: 'FAILED', error: lastError, retries: this.MAX_RETRIES },
    });

    return { success: false, channel: input.channel, error: lastError };
  }

  private async checkRateLimit(userId: string, channel: NotificationChannel): Promise<boolean> {
    const key = `notif:ratelimit:${userId}:${channel}`;
    const count = await this.redis.incr(key);
    if (count === 1) {
      await this.redis.expire(key, 3600); // 1 hour window
    }
    const limits: Record<string, number> = {
      SMS: 10,
      WHATSAPP: 20,
      EMAIL: 30,
      PUSH: 50,
      IN_APP: 100,
    };
    return count <= (limits[channel] || 50);
  }

  private async resolveRecipient(userId: string, channel: NotificationChannel): Promise<string> {
    const user = await this.prisma.user.findUnique({ where: { id: userId } });
    if (!user) throw new UserNotFoundError(userId);

    switch (channel) {
      case 'SMS':
      case 'WHATSAPP':
        return user.phone;
      case 'EMAIL':
        if (!user.email) throw new EmailNotSetError(userId);
        return user.email;
      case 'PUSH': {
        const device = await this.prisma.userDevice.findFirst({
          where: { userId },
          orderBy: { lastActiveAt: 'desc' },
        });
        if (!device) throw new DeviceNotRegisteredError(userId);
        return device.fcmToken;
      }
      case 'IN_APP':
        return userId;
      default:
        throw new UnsupportedChannelError(channel);
    }
  }

  private isChannelAllowed(channel: NotificationChannel, prefs: NotificationPreferences): boolean {
    const channelMap: Record<string, boolean> = {
      SMS: prefs.sms,
      WHATSAPP: prefs.whatsapp,
      EMAIL: prefs.email,
      PUSH: prefs.push,
      IN_APP: true, // Always allowed
    };
    return channelMap[channel] ?? true;
  }

  private chunkArray<T>(arr: T[], size: number): T[][] {
    const chunks: T[][] = [];
    for (let i = 0; i < arr.length; i += size) {
      chunks.push(arr.slice(i, i + size));
    }
    return chunks;
  }

  private sleep(ms: number): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
}
```

---

## Preference Management

```typescript
async getPreferences(userId: string): Promise<NotificationPreferences> {
  const prefs = await this.prisma.notificationPreference.findUnique({ where: { userId } });
  if (!prefs) {
    return this.prisma.notificationPreference.create({
      data: {
        userId,
        email: true,
        push: true,
        sms: true,
        whatsapp: false,
        orderUpdates: true,
        promotions: false,
        priceAlerts: true,
      },
    });
  }
  return prefs;
}

async updatePreferences(userId: string, input: NotificationPreferencesInput): Promise<void> {
  await this.prisma.notificationPreference.upsert({
    where: { userId },
    create: { userId, ...input },
    update: input,
  });
}
```

---

## In-App Notification Queries

```typescript
async getInAppNotifications(
  userId: string,
  pagination: PaginationInput,
): Promise<PaginatedResult<Notification>> {
  const [data, total, unreadCount] = await Promise.all([
    this.prisma.notification.findMany({
      where: { userId, channel: 'IN_APP' },
      orderBy: { createdAt: 'desc' },
      skip: (pagination.page - 1) * pagination.limit,
      take: pagination.limit,
    }),
    this.prisma.notification.count({
      where: { userId, channel: 'IN_APP' },
    }),
    this.prisma.notification.count({
      where: { userId, channel: 'IN_APP', readAt: null },
    }),
  ]);

  return { data, total, unreadCount, page: pagination.page, limit: pagination.limit };
}

async markAsRead(userId: string, notificationId: string): Promise<void> {
  await this.prisma.notification.updateMany({
    where: { id: notificationId, userId, readAt: null },
    data: { readAt: new Date() },
  });
}

async markAllAsRead(userId: string): Promise<void> {
  await this.prisma.notification.updateMany({
    where: { userId, channel: 'IN_APP', readAt: null },
    data: { readAt: new Date() },
  });
}
```

---

## API Endpoints

| Method | Endpoint                        | Description               | Auth   |
|--------|---------------------------------|---------------------------|--------|
| GET    | /notifications                  | Get in-app notifications  | User   |
| GET    | /notifications/unread-count     | Get unread count          | User   |
| PUT    | /notifications/:id/read         | Mark as read              | User   |
| PUT    | /notifications/read-all         | Mark all as read          | User   |
| GET    | /notifications/preferences      | Get notification prefs    | User   |
| PUT    | /notifications/preferences      | Update notification prefs | User   |
| POST   | /notifications/send             | Send notification         | Admin  |
| POST   | /notifications/batch            | Batch send                | Admin  |
