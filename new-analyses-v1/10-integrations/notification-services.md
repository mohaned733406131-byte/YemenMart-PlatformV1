# Notification Services — YemenMart

## 1. Overview

YemenMart delivers notifications through five channels: SMS, WhatsApp, Email, Push (FCM), and In-App. Each channel is managed through a unified notification service with per-user preferences, rate limiting, template rendering, and retry logic.

```
┌─────────────────────────────────────────────────────────────────┐
│                   NOTIFICATION ARCHITECTURE                      │
│                                                                 │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐                  │
│  │ Business │───>│ Notif.   │───>│ Channel  │                  │
│  │ Events   │    │ Service  │    │ Router   │                  │
│  └──────────┘    └──────────┘    └────┬─────┘                  │
│                                       │                         │
│  ┌────────────────────────────────────┼────────────────────┐   │
│  │                 │                  │                    │   │
│  │  ┌──────────┐  │  ┌──────────┐   │  ┌──────────┐      │   │
│  │  │   SMS    │  │  │ WhatsApp │   │  │  Email   │      │   │
│  │  │ Provider │  │  │ Business │   │  │ Provider │      │   │
│  │  └──────────┘  │  └──────────┘   │  └──────────┘      │   │
│  │                 │                  │                    │   │
│  │  ┌──────────┐  │  ┌──────────┐   │                    │   │
│  │  │   Push   │  │  │ In-App   │   │                    │   │
│  │  │ (FCM)    │  │  │ Service  │   │                    │   │
│  │  └──────────┘  │  └──────────┘   │                    │   │
│  │                 │                  │                    │   │
│  └─────────────────┼──────────────────┼────────────────────┘   │
│                    │                  │                          │
│                    ▼                  ▼                          │
│             ┌──────────┐      ┌──────────┐                     │
│             │ Customer │      │  Vendor  │                     │
│             │  Device  │      │  Device  │                     │
│             └──────────┘      └──────────┘                     │
└─────────────────────────────────────────────────────────────────┘
```

## 2. WhatsApp Business API

### Connection Details

| Property | Value |
|----------|-------|
| **Type** | Messaging Platform |
| **Protocol** | Cloud API (HTTPS) |
| **Authentication** | Bearer token |
| **Base URL** | `https://graph.facebook.com/v17.0/{phone_number_id}` |
| **Integrating Block** | B11 Content |
| **Timeout** | 15s |
| **Retry** | 3 attempts, exponential backoff |
| **Webhook** | Inbound messages verified via X-Hub-Signature |

### API Endpoints

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/messages` | POST | Send template/free-form message |
| `/media` | POST | Send media (images, documents) |
| Webhook (inbound) | POST | Receive customer replies |

### Message Templates

```typescript
const WHATSAPP_TEMPLATES = {
  ORDER_CONFIRMED: {
    name: 'order_confirmed',
    language: { code: 'ar' },
    components: [
      {
        type: 'body',
        parameters: [
          { type: 'text', text: '{orderId}' },
          { type: 'text', text: '{total}' },
          { type: 'text', text: '{vendorName}' },
        ],
      },
    ],
  },
  ORDER_SHIPPED: {
    name: 'order_shipped',
    language: { code: 'ar' },
    components: [
      {
        type: 'body',
        parameters: [
          { type: 'text', text: '{orderId}' },
          { type: 'text', text: '{deliveryCode}' },
          { type: 'text', text: '{estimatedDelivery}' },
        ],
      },
    ],
  },
  DELIVERY_REMINDER: {
    name: 'delivery_reminder',
    language: { code: 'ar' },
    components: [
      {
        type: 'body',
        parameters: [
          { type: 'text', text: '{customerName}' },
          { type: 'text', text: '{orderId}' },
          { type: 'text', text: '{deliveryCode}' },
        ],
      },
    ],
  },
};
```

### Adapter Implementation

```typescript
// packages/notification-module/src/adapters/whatsapp.adapter.ts
import { injectable, inject } from 'inversify';
import { IChannelProvider, ChannelMessage, ChannelResult } from '../interfaces/channel.interface';

interface WhatsAppConfig {
  phoneNumberId: string;
  accessToken: string;
  verifyToken: string;
  appSecret: string;
  baseUrl: string;
}

@injectable()
export class WhatsAppProvider implements IChannelProvider {
  name = 'whatsapp';
  private httpClient: HttpClient;

  constructor(@inject('WhatsAppConfig') config: WhatsAppConfig) {
    this.httpClient = new HttpClient({
      baseURL: `${config.baseUrl}/${config.phoneNumberId}`,
      timeout: 15000,
      headers: {
        'Authorization': `Bearer ${config.accessToken}`,
        'Content-Type': 'application/json',
      },
    });
  }

  async send(input: ChannelMessage): Promise<ChannelResult> {
    try {
      const result = await this.httpClient.post('/messages', {
        messaging_product: 'whatsapp',
        to: input.to.replace('+', ''),
        type: 'template',
        template: input.template || {
          name: 'generic_message',
          language: { code: 'ar' },
          components: [
            {
              type: 'body',
              parameters: [{ type: 'text', text: input.body }],
            },
          ],
        },
      });

      return {
        channel: 'WHATSAPP',
        success: true,
        messageId: result.data.messages[0].id,
        deliveredAt: new Date(),
      };
    } catch (error) {
      return {
        channel: 'WHATSAPP',
        success: false,
        error: error.response?.data?.error?.message || error.message,
      };
    }
  }

  async sendMedia(input: ChannelMessage, mediaUrl: string, mediaType: string): Promise<ChannelResult> {
    try {
      // Upload media first
      const mediaResponse = await this.httpClient.post('/media', {
        messaging_product: 'whatsapp',
        type: mediaType,
        url: mediaUrl,
      });

      const mediaId = mediaResponse.data.id;

      // Send message with media
      const result = await this.httpClient.post('/messages', {
        messaging_product: 'whatsapp',
        to: input.to.replace('+', ''),
        type: 'image',
        image: { id: mediaId },
      });

      return {
        channel: 'WHATSAPP',
        success: true,
        messageId: result.data.messages[0].id,
        deliveredAt: new Date(),
      };
    } catch (error) {
      return {
        channel: 'WHATSAPP',
        success: false,
        error: error.message,
      };
    }
  }

  verifyWebhook(mode: string, token: string, challenge: string): string | null {
    if (mode === 'subscribe' && token === process.env.WHATSAPP_VERIFY_TOKEN) {
      return challenge;
    }
    return null;
  }

  verifySignature(payload: Buffer, signature: string): boolean {
    const expected = crypto
      .createHmac('sha256', process.env.WHATSAPP_APP_SECRET)
      .update(payload)
      .digest('hex');
    return crypto.timingSafeEqual(Buffer.from(signature), Buffer.from(expected));
  }
}
```

## 3. Firebase Cloud Messaging (Push)

### Connection Details

| Property | Value |
|----------|-------|
| **Type** | Push Notification Service |
| **Protocol** | REST API over HTTPS |
| **Authentication** | Service account (Firebase Admin SDK) |
| **Timeout** | 10s |
| **Retry** | 2 attempts |

### Push Notification Templates

```typescript
const PUSH_TEMPLATES = {
  ORDER_STATUS_CHANGED: {
    title: 'تحديث حالة الطلب',
    body: 'طلبك #{orderId} الآن: {status}',
    data: { type: 'order_update', orderId: '{orderId}' },
  },
  NEW_MESSAGE: {
    title: 'رسالة جديدة',
    body: '{senderName}: {preview}',
    data: { type: 'message', conversationId: '{conversationId}' },
  },
  PROMOTION: {
    title: '{promotionTitle}',
    body: '{promotionMessage}',
    data: { type: 'promotion', promotionId: '{promotionId}' },
  },
  WALLET_LOW: {
    title: 'رصيد المحفظة منخفض',
    body: 'رصيدك الحالي: {balance} يمني. اشحن محفظتك الآن.',
    data: { type: 'wallet_low' },
  },
  DELIVERY_READY: {
    title: 'جاهز للتوصيل',
    body: 'طلبك #{orderId} جاهز. كود التوصيل: {deliveryCode}',
    data: { type: 'delivery', orderId: '{orderId}' },
  },
};
```

### Adapter Implementation

```typescript
// packages/notification-module/src/adapters/push-fcm.adapter.ts
import { injectable, inject } from 'inversify';
import { IChannelProvider, ChannelMessage, ChannelResult } from '../interfaces/channel.interface';
import * as admin from 'firebase-admin';

@injectable()
export class PushProvider implements IChannelProvider {
  name = 'push';
  private messaging: admin.messaging.Messaging;

  constructor(@inject('FirebaseAdmin') firebase: admin.app.App) {
    this.messaging = firebase.messaging();
  }

  async send(input: ChannelMessage): Promise<ChannelResult> {
    try {
      const message: admin.messaging.Message = {
        token: input.to,
        notification: {
          title: input.subject,
          body: input.body,
        },
        data: input.data as Record<string, string>,
        android: {
          priority: 'high',
          notification: {
            channelId: input.data?.channel || 'default',
            sound: 'default',
          },
        },
        apns: {
          payload: {
            aps: {
              sound: 'default',
              badge: 1,
            },
          },
        },
      };

      const messageId = await this.messaging.send(message);

      return {
        channel: 'PUSH',
        success: true,
        messageId,
        deliveredAt: new Date(),
      };
    } catch (error) {
      return {
        channel: 'PUSH',
        success: false,
        error: error.message,
      };
    }
  }

  async sendToTopic(topic: string, title: string, body: string, data?: Record<string, string>): Promise<ChannelResult> {
    try {
      const messageId = await this.messaging.send({
        topic,
        notification: { title, body },
        data,
      });

      return {
        channel: 'PUSH',
        success: true,
        messageId,
        deliveredAt: new Date(),
      };
    } catch (error) {
      return {
        channel: 'PUSH',
        success: false,
        error: error.message,
      };
    }
  }

  async subscribeToTopic(tokens: string[], topic: string): Promise<void> {
    await this.messaging.subscribeToTopic(tokens, topic);
  }

  async unsubscribeFromTopic(tokens: string[], topic: string): Promise<void> {
    await this.messaging.unsubscribeFromTopic(tokens, topic);
  }
}
```

## 4. Email Provider

### Connection Details

| Property | Value |
|----------|-------|
| **Type** | Email Delivery Service |
| **Protocol** | SMTP / REST API |
| **Authentication** | API key |
| **Provider** | SendGrid |
| **Timeout** | 15s |
| **Retry** | 2 attempts |

### Email Templates

```typescript
const EMAIL_TEMPLATES = {
  WELCOME: {
    subject: 'مرحباً بك في YemenMart',
    html: `
      <div dir="rtl" style="font-family: Tajawal, sans-serif;">
        <h1>مرحباً {name}</h1>
        <p>شكراً لك للانضمام إلى YemenMart.</p>
        <p>ابدأ التسوق الآن واستمتع بآلاف المنتجات.</p>
        <a href="{shopUrl}" style="background: #2563eb; color: white; padding: 12px 24px; text-decoration: none; border-radius: 8px;">
          تصفح المنتجات
        </a>
      </div>
    `,
  },
  ORDER_CONFIRMATION: {
    subject: 'تأكيد الطلب #{orderId}',
    html: `
      <div dir="rtl" style="font-family: Tajawal, sans-serif;">
        <h1>تم تأكيد طلبك</h1>
        <p>رقم الطلب: {orderId}</p>
        <p>المبلغ الإجمالي: {total} يمني</p>
        <p>التوصيل المتوقع: {estimatedDelivery}</p>
        <table style="width: 100%; border-collapse: collapse;">
          {items}
        </table>
      </div>
    `,
  },
  PASSWORD_RESET: {
    subject: 'إعادة تعيين كلمة المرور',
    html: `
      <div dir="rtl" style="font-family: Tajawal, sans-serif;">
        <h1>إعادة تعيين كلمة المرور</h1>
        <p>نقر على الرابط لإعادة تعيين كلمة المرور:</p>
        <a href="{resetUrl}" style="background: #2563eb; color: white; padding: 12px 24px; text-decoration: none; border-radius: 8px;">
          إعادة التعيين
        </a>
        <p>صالح لمدة ساعة واحدة.</p>
      </div>
    `,
  },
};
```

### Adapter Implementation

```typescript
// packages/notification-module/src/adapters/email.adapter.ts
import { injectable, inject } from 'inversify';
import { IChannelProvider, ChannelMessage, ChannelResult } from '../interfaces/channel.interface';
import * as sgMail from '@sendgrid/mail';

@injectable()
export class EmailProvider implements IChannelProvider {
  name = 'email';

  constructor(@inject('EmailConfig') private config: EmailConfig) {
    sgMail.setApiKey(config.apiKey);
  }

  async send(input: ChannelMessage): Promise<ChannelResult> {
    try {
      const msg = {
        to: input.to,
        from: { email: this.config.fromAddress, name: 'YemenMart' },
        subject: input.subject,
        html: input.htmlBody || input.body,
        text: input.body,
      };

      const result = await sgMail.send(msg);

      return {
        channel: 'EMAIL',
        success: true,
        messageId: result[0].headers['x-message-id'],
        deliveredAt: new Date(),
      };
    } catch (error) {
      return {
        channel: 'EMAIL',
        success: false,
        error: error.message,
      };
    }
  }

  async sendBatch(inputs: ChannelMessage[]): Promise<ChannelResult[]> {
    const messages = inputs.map(input => ({
      to: input.to,
      from: { email: this.config.fromAddress, name: 'YemenMart' },
      subject: input.subject,
      html: input.htmlBody || input.body,
      text: input.body,
    }));

    try {
      await sgMail.send(messages);
      return inputs.map(() => ({
        channel: 'EMAIL' as NotificationChannel,
        success: true,
        deliveredAt: new Date(),
      }));
    } catch (error) {
      return inputs.map(() => ({
        channel: 'EMAIL' as NotificationChannel,
        success: false,
        error: error.message,
      }));
    }
  }
}
```

## 5. Channel Provider Interface

```typescript
// packages/notification-module/src/interfaces/channel.interface.ts

export interface IChannelProvider {
  name: string;
  send(input: ChannelMessage): Promise<ChannelResult>;
}

export interface ChannelMessage {
  userId: string;
  to: string;
  type: NotificationType;
  subject?: string;
  body: string;
  htmlBody?: string;
  template?: any;
  data?: Record<string, any>;
}

export interface ChannelResult {
  channel: NotificationChannel;
  success: boolean;
  messageId?: string;
  error?: string;
  deliveredAt?: Date;
}

export type NotificationChannel = 'SMS' | 'WHATSAPP' | 'EMAIL' | 'PUSH' | 'IN_APP';

export type NotificationType =
  | 'OTP'
  | 'ORDER_CONFIRMED'
  | 'ORDER_SHIPPED'
  | 'ORDER_DELIVERED'
  | 'ORDER_CANCELLED'
  | 'DELIVERY_CODE'
  | 'WALLET_TOPUP'
  | 'WALLET_LOW'
  | 'PROMOTION'
  | 'SYSTEM'
  | 'KYC_UPDATE'
  | 'VENDOR_ALERT';
```

## 6. In-App Notification Service

```typescript
@injectable()
export class InAppProvider implements IChannelProvider {
  name = 'in_app';

  constructor(
    @inject('PrismaClient') private prisma: PrismaClient,
    @inject('RedisClient') private redis: Redis,
  ) {}

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

    // Publish to Redis for real-time delivery via WebSocket
    await this.redis.publish(`notifications:${input.userId}`, JSON.stringify({
      id: notification.id,
      type: input.type,
      title: input.subject,
      body: input.body,
      data: input.data,
      createdAt: notification.createdAt,
    }));

    return {
      channel: 'IN_APP',
      success: true,
      messageId: notification.id,
      deliveredAt: new Date(),
    };
  }

  async getNotifications(userId: string, pagination: PaginationInput) {
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
}
```

## 7. User Preference Management

```typescript
@injectable()
export class NotificationPreferenceService {
  constructor(@inject('PrismaClient') private prisma: PrismaClient) {}

  async getPreferences(userId: string): Promise<NotificationPreferences> {
    const prefs = await this.prisma.notificationPreference.findUnique({
      where: { userId },
    });

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

  async isChannelAllowed(userId: string, channel: NotificationChannel): Promise<boolean> {
    const prefs = await this.getPreferences(userId);
    const channelMap: Record<string, boolean> = {
      SMS: prefs.sms,
      WHATSAPP: prefs.whatsapp,
      EMAIL: prefs.email,
      PUSH: prefs.push,
      IN_APP: true, // Always allowed
    };
    return channelMap[channel] ?? true;
  }
}
```

## 8. Notification Routing Rules

| Notification Type | SMS | WhatsApp | Email | Push | In-App |
|-------------------|:---:|:--------:|:-----:|:----:|:------:|
| OTP | Yes | No | No | No | No |
| Order Confirmed | Yes | Yes | Yes | Yes | Yes |
| Order Shipped | Yes | Yes | Yes | Yes | Yes |
| Order Delivered | Yes | Yes | Yes | Yes | Yes |
| Order Cancelled | Yes | Yes | Yes | Yes | Yes |
| Delivery Code | Yes | Yes | No | Yes | Yes |
| Wallet Top-Up | Yes | No | Yes | Yes | Yes |
| Wallet Low | No | No | No | Yes | Yes |
| Promotions | No | Yes | Yes | Yes | Yes |
| KYC Update | No | No | Yes | No | Yes |
| Vendor Alert | Yes | No | Yes | Yes | Yes |

## 9. Rate Limiting

```typescript
const CHANNEL_RATE_LIMITS: Record<NotificationChannel, { max: number; window: number }> = {
  SMS: { max: 10, window: 3600 },      // 10 per hour per user
  WHATSAPP: { max: 20, window: 3600 }, // 20 per hour per user
  EMAIL: { max: 30, window: 3600 },    // 30 per hour per user
  PUSH: { max: 50, window: 3600 },     // 50 per hour per user
  IN_APP: { max: 100, window: 3600 },  // 100 per hour per user
};

async function checkChannelRateLimit(
  userId: string,
  channel: NotificationChannel,
  redis: Redis,
): Promise<boolean> {
  const limits = CHANNEL_RATE_LIMITS[channel];
  const key = `notif:ratelimit:${userId}:${channel}`;
  
  const count = await redis.incr(key);
  if (count === 1) {
    await redis.expire(key, limits.window);
  }
  
  return count <= limits.max;
}
```

## 10. API Endpoints

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| GET | /notifications | Get in-app notifications | User |
| GET | /notifications/unread-count | Get unread count | User |
| PUT | /notifications/:id/read | Mark as read | User |
| PUT | /notifications/read-all | Mark all as read | User |
| GET | /notifications/preferences | Get notification prefs | User |
| PUT | /notifications/preferences | Update notification prefs | User |
| POST | /notifications/send | Send notification (admin) | Admin |
| POST | /notifications/batch | Batch send (admin) | Admin |
| POST | /webhooks/whatsapp | WhatsApp inbound webhook | Verified |
| POST | /webhooks/sms/:provider | SMS delivery webhook | Verified |

## 11. Related Files

| File | Description |
|------|-------------|
| `sms-providers.md` | SMS gateway integration |
| `integration-overview.md` | Integration architecture overview |
| `06-backend/notification-service.md` | Notification service implementation |
| `03-system-analysis/integration-points.md` | Integration point catalog |
| `01-business-analysis/business-rules.md` | Notification business rules |

---

*Document Version: 1.0 | Last Updated: 2026-09-13 | Classification: Confidential*
