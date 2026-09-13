# Job Scheduling

## Overview

Background job processing using BullMQ (Redis-backed). Handles async tasks: notifications, commissions, payouts, reconciliation, cleanup, and scheduled reports. Includes dead letter queue, retry strategies, and monitoring.

---

## BullMQ Configuration

```typescript
// packages/jobs/src/queue-manager.ts
import { Queue, QueueScheduler, Worker, Job, JobOptions } from 'bullmq';
import { Redis } from 'ioredis';

const connection = new Redis({
  host: process.env.REDIS_HOST || 'localhost',
  port: parseInt(process.env.REDIS_PORT || '6379'),
  password: process.env.REDIS_PASSWORD,
  maxRetriesPerRequest: null,
  enableReadyCheck: false,
});

// Shared connection for blocking operations
const blockingConnection = connection.duplicate();

export function createQueue(name: string): Queue {
  return new Queue(name, {
    connection,
    defaultJobOptions: {
      removeOnComplete: { age: 86400, count: 1000 }, // Keep 24h or 1000 jobs
      removeOnFail: { age: 604800, count: 500 },     // Keep 7 days
      attempts: 3,
      backoff: { type: 'exponential', delay: 2000 },
    },
  });
}

export function createWorker(
  name: string,
  processor: (job: Job) => Promise<any>,
  concurrency = 5,
): Worker {
  return new Worker(name, processor, {
    connection,
    concurrency,
    limiter: {
      max: 100,
      duration: 60000, // 100 jobs per minute max
    },
  });
}

// Dead Letter Queue
export const deadLetterQueue = createQueue('dead-letter');
```

---

## Job Queues

### 1. Notification Queue

```typescript
// packages/jobs/src/queues/notification.queue.ts
import { Queue, Job } from 'bullmq';

export const notificationQueue = createQueue('notifications');

export interface NotificationJobData {
  userId: string;
  type: NotificationType;
  channel: NotificationChannel;
  variables: Record<string, string>;
  priority?: 'low' | 'normal' | 'high';
}

export function enqueueNotification(data: NotificationJobData): Promise<Job> {
  return notificationQueue.add('send', data, {
    priority: data.priority === 'high' ? 1 : data.priority === 'low' ? 3 : 2,
    attempts: 3,
    backoff: { type: 'exponential', delay: 1000 },
  });
}

// Worker
const notificationWorker = createWorker('notifications', async (job: Job<NotificationJobData>) => {
  const { userId, type, channel, variables } = job.data;

  await notificationService.send({
    userId,
    type,
    channel,
    variables,
  });

  return { delivered: true, channel };
}, 10);
```

### 2. Commission Queue

```typescript
// packages/jobs/src/queues/commission.queue.ts
export const commissionQueue = createQueue('commissions');

export interface CommissionJobData {
  orderId: string;
  subOrderId: string;
  storeId: string;
  amount: number;
  currency: string;
}

export function enqueueCommissionCalculation(data: CommissionJobData): Promise<Job> {
  return commissionQueue.add('calculate', data, {
    priority: 2,
    attempts: 3,
  });
}

// Worker
const commissionWorker = createWorker('commissions', async (job: Job<CommissionJobData>) => {
  const { orderId, subOrderId, storeId, amount, currency } = job.data;

  const commission = await commissionService.calculate(
    storeId,
    new Money(amount, currency),
  );

  await prisma.order.update({
    where: { id: subOrderId },
    data: {
      commission: commission.totalFee,
      sellerPayout: amount - commission.totalFee,
    },
  });

  // Emit event for payout processing
  await eventBus.emit('commission.calculated', {
    orderId,
    subOrderId,
    storeId,
    commission: commission.totalFee,
    payout: amount - commission.totalFee,
  });

  return commission;
});
```

### 3. Payout Queue

```typescript
// packages/jobs/src/queues/payout.queue.ts
export const payoutQueue = createQueue('payouts');

export interface PayoutJobData {
  storeId: string;
  sellerId: string;
  amount: number;
  currency: string;
  orderId: string;
}

export function enqueuePayout(data: PayoutJobData): Promise<Job> {
  return payoutQueue.add('process', data, {
    priority: 2,
    attempts: 5,
    backoff: { type: 'exponential', delay: 5000 },
  });
}

// Worker
const payoutWorker = createWorker('payouts', async (job: Job<PayoutJobData>) => {
  const { storeId, sellerId, amount, currency, orderId } = job.data;

  // Create payout record
  const payout = await prisma.sellerPayout.create({
    data: {
      storeId,
      orderId,
      amount,
      currency,
      status: 'PROCESSING',
    },
  });

  try {
    // Credit seller wallet
    await paymentService.creditWallet(sellerId, {
      amount: new Money(amount, currency),
      orderId,
      idempotencyKey: `payout:${payout.id}`,
      description: `Payout for order ${orderId}`,
    });

    await prisma.sellerPayout.update({
      where: { id: payout.id },
      data: { status: 'COMPLETED', completedAt: new Date() },
    });

    return { payoutId: payout.id, status: 'COMPLETED' };
  } catch (error) {
    await prisma.sellerPayout.update({
      where: { id: payout.id },
      data: { status: 'FAILED', error: error.message },
    });
    throw error;
  }
});
```

### 4. Reconciliation Queue

```typescript
// packages/jobs/src/queues/reconciliation.queue.ts
export const reconciliationQueue = createQueue('reconciliation');

export interface ReconciliationJobData {
  date: string; // YYYY-MM-DD
  type: 'daily' | 'weekly';
}

export function enqueueReconciliation(data: ReconciliationJobData): Promise<Job> {
  return reconciliationQueue.add('run', data, {
    priority: 3,
    attempts: 2,
    backoff: { type: 'fixed', delay: 30000 },
  });
}

// Worker
const reconciliationWorker = createWorker('reconciliation', async (job: Job<ReconciliationJobData>) => {
  const { date, type } = job.data;
  const startDate = new Date(date);
  const endDate = new Date(date);
  endDate.setDate(endDate.getDate() + 1);

  // Reconcile orders
  const orders = await prisma.order.findMany({
    where: {
      createdAt: { gte: startDate, lt: endDate },
      type: 'MASTER',
      status: { in: ['COMPLETED', 'DELIVERED'] },
    },
    include: { subOrders: true },
  });

  let totalRevenue = 0;
  let totalCommission = 0;
  let totalPayouts = 0;
  const discrepancies: string[] = [];

  for (const order of orders) {
    for (const subOrder of order.subOrders) {
      totalRevenue += Number(subOrder.subtotal);
      totalCommission += Number(subOrder.commission);
      totalPayouts += Number(subOrder.sellerPayout);

      // Verify math
      const expected = Number(subOrder.subtotal) - Number(subOrder.commission);
      if (Math.abs(expected - Number(subOrder.sellerPayout)) > 0.01) {
        discrepancies.push(`Order ${subOrder.id}: expected ${expected}, got ${subOrder.sellerPayout}`);
      }
    }
  }

  // Reconcile wallet balances
  const wallets = await prisma.wallet.findMany();
  for (const wallet of wallets) {
    const transactions = await prisma.walletTransaction.aggregate({
      where: { walletId: wallet.id },
      _sum: { amount: true },
    });

    const expectedBalance = Number(transactions._sum.amount || 0);
    if (Math.abs(expectedBalance - Number(wallet.balance)) > 0.01) {
      discrepancies.push(`Wallet ${wallet.id}: expected ${expectedBalance}, got ${wallet.balance}`);
    }
  }

  // Save reconciliation report
  const report = await prisma.reconciliationReport.create({
    data: {
      date: startDate,
      type,
      totalRevenue,
      totalCommission,
      totalPayouts,
      orderCount: orders.length,
      discrepancies,
      status: discrepancies.length > 0 ? 'DISCREPANCIES' : 'BALANCED',
    },
  });

  if (discrepancies.length > 0) {
    await notificationService.send({
      userId: process.env.ADMIN_USER_ID,
      type: 'SYSTEM',
      channel: 'EMAIL',
      variables: {
        date,
        discrepancies: discrepancies.join('\n'),
        totalRevenue: totalRevenue.toString(),
      },
    });
  }

  return report;
});
```

### 5. Cleanup Queue

```typescript
// packages/jobs/src/queues/cleanup.queue.ts
export const cleanupQueue = createQueue('cleanup');

export function enqueueCleanup(type: CleanupType): Promise<Job> {
  return cleanupQueue.add(type, { type }, {
    priority: 4,
    attempts: 2,
  });
}

type CleanupType = 'expired_holds' | 'old_sessions' | 'expired_otps' | 'old_notifications' | 'expired_stock_holds';

// Worker
const cleanupWorker = createWorker('cleanup', async (job: Job<{ type: CleanupType }>) => {
  switch (job.data.type) {
    case 'expired_holds':
      return await cleanupExpiredEscrowHolds();
    case 'old_sessions':
      return await cleanupOldSessions();
    case 'expired_otps':
      return await cleanupExpiredOTPs();
    case 'old_notifications':
      return await cleanupOldNotifications();
    case 'expired_stock_holds':
      return await cleanupExpiredStockHolds();
  }
});

async function cleanupExpiredEscrowHolds(): Promise<number> {
  const result = await prisma.escrowHold.deleteMany({
    where: {
      status: 'HELD',
      releaseAt: { lt: new Date(Date.now() - 30 * 24 * 60 * 60 * 1000) }, // 30 days old
    },
  });
  return result.count;
}

async function cleanupOldSessions(): Promise<number> {
  const result = await prisma.session.deleteMany({
    where: { expiresAt: { lt: new Date() } },
  });
  return result.count;
}

async function cleanupExpiredOTPs(): Promise<number> {
  // Redis TTL handles OTP cleanup automatically
  return 0;
}

async function cleanupOldNotifications(): Promise<number> {
  const cutoff = new Date(Date.now() - 90 * 24 * 60 * 60 * 1000); // 90 days
  const result = await prisma.notification.deleteMany({
    where: {
      createdAt: { lt: cutoff },
      channel: { not: 'IN_APP' },
    },
  });
  return result.count;
}

async function cleanupExpiredStockHolds(): Promise<number> {
  const result = await prisma.stockHold.deleteMany({
    where: {
      confirmed: false,
      expiresAt: { lt: new Date() },
    },
  });
  return result.count;
}
```

---

## Cron Jobs

```typescript
// packages/jobs/src/schedulers/index.ts
import { QueueScheduler } from 'bullmq';

// Notification queue scheduler
const notificationScheduler = new QueueScheduler('notifications', { connection });
const commissionScheduler = new QueueScheduler('commissions', { connection });
const payoutScheduler = new QueueScheduler('payouts', { connection });
const cleanupScheduler = new QueueScheduler('cleanup', { connection });
const reconciliationScheduler = new QueueScheduler('reconciliation', { connection });

// Schedule recurring jobs
export function setupCronJobs() {
  // Release expired escrow holds - every hour
  cleanupQueue.add('expired_holds', { type: 'expired_holds' }, {
    repeat: { cron: '0 * * * *' }, // Every hour
    jobId: 'cron:escrow-cleanup',
  });

  // Release expired stock holds - every 5 minutes
  cleanupQueue.add('expired_stock_holds', { type: 'expired_stock_holds' }, {
    repeat: { cron: '*/5 * * * *' }, // Every 5 minutes
    jobId: 'cron:stock-hold-cleanup',
  });

  // Cleanup old sessions - daily at 3 AM
  cleanupQueue.add('old_sessions', { type: 'old_sessions' }, {
    repeat: { cron: '0 3 * * *' },
    jobId: 'cron:session-cleanup',
  });

  // Cleanup old notifications - weekly on Sunday at 4 AM
  cleanupQueue.add('old_notifications', { type: 'old_notifications' }, {
    repeat: { cron: '0 4 * * 0' },
    jobId: 'cron:notification-cleanup',
  });

  // Daily reconciliation - daily at 2 AM
  reconciliationQueue.add('daily', {
    date: new Date().toISOString().split('T')[0],
    type: 'daily',
  }, {
    repeat: { cron: '0 2 * * *' },
    jobId: 'cron:daily-reconciliation',
  });

  // Weekly reconciliation - Monday at 1 AM
  reconciliationQueue.add('weekly', {
    date: new Date().toISOString().split('T')[0],
    type: 'weekly',
  }, {
    repeat: { cron: '0 1 * * 1' },
    jobId: 'cron:weekly-reconciliation',
  });

  // Process pending payouts - every 30 minutes
  payoutQueue.add('batch', { type: 'batch_process' }, {
    repeat: { cron: '*/30 * * * *' },
    jobId: 'cron:batch-payouts',
  });

  // Update product ratings - daily at 5 AM
  commissionQueue.add('update-ratings', { type: 'update-ratings' }, {
    repeat: { cron: '0 5 * * *' },
    jobId: 'cron:update-ratings',
  });
}
```

---

## Retry Strategies

| Job Type        | Max Retries | Backoff Type  | Initial Delay | Max Delay   |
|-----------------|-------------|---------------|---------------|-------------|
| Notification    | 3           | Exponential   | 1s            | 15s         |
| Commission      | 3           | Exponential   | 2s            | 30s         |
| Payout          | 5           | Exponential   | 5s            | 5min        |
| Reconciliation  | 2           | Fixed         | 30s           | 30s         |
| Cleanup         | 2           | Fixed         | 10s           | 10s         |

```typescript
// Custom retry logic
function getRetryDelay(attemptsMade: number, type: string): number {
  const strategies: Record<string, number[]> = {
    notification: [1000, 5000, 15000],
    commission: [2000, 10000, 30000],
    payout: [5000, 15000, 60000, 120000, 300000],
  };

  const delays = strategies[type] || [2000];
  return delays[Math.min(attemptsMade, delays.length - 1)];
}
```

---

## Dead Letter Queue

```typescript
// Failed jobs after all retries move to DLQ
export async function moveToDeadLetterQueue(job: Job, error: Error): Promise<void> {
  await deadLetterQueue.add('failed-job', {
    originalQueue: job.queueName,
    jobId: job.id,
    data: job.data,
    error: error.message,
    stack: error.stack,
    failedAt: new Date(),
    attemptsMade: job.attemptsMade,
  });

  // Alert admin for critical failures
  if (['payouts', 'commissions'].includes(job.queueName)) {
    await notificationService.send({
      userId: process.env.ADMIN_USER_ID,
      type: 'SYSTEM',
      channel: 'EMAIL',
      variables: {
        queue: job.queueName,
        jobId: job.id,
        error: error.message,
      },
    });
  }
}

// DLQ worker - for manual review
const dlqWorker = createWorker('dead-letter', async (job: Job) => {
  const { originalQueue, jobId, data, error } = job.data;

  logger.error('Job moved to DLQ', {
    queue: originalQueue,
    jobId,
    error,
    data,
  });

  // Store for admin review
  await prisma.failedJob.create({
    data: {
      queue: originalQueue,
      jobId,
      payload: data,
      error,
      status: 'PENDING_REVIEW',
    },
  });
});
```

---

## Monitoring

```typescript
// packages/jobs/src/monitoring.ts
export async function getQueueStats(): Promise<QueueStats[]> {
  const queues = [
    notificationQueue,
    commissionQueue,
    payoutQueue,
    reconciliationQueue,
    cleanupQueue,
    deadLetterQueue,
  ];

  const stats: QueueStats[] = [];

  for (const queue of queues) {
    const [waiting, active, completed, failed, delayed] = await Promise.all([
      queue.getWaitingCount(),
      queue.getActiveCount(),
      queue.getCompletedCount(),
      queue.getFailedCount(),
      queue.getDelayedCount(),
    ]);

    stats.push({
      name: queue.name,
      waiting,
      active,
      completed,
      failed,
      delayed,
      total: waiting + active + completed + failed + delayed,
    });
  }

  return stats;
}

export async function getJobHistory(
  queueName: string,
  status: 'completed' | 'failed' | 'active' | 'waiting',
  limit = 50,
): Promise<JobInfo[]> {
  const queue = getQueueByName(queueName);
  const jobs = await queue.getJobs([status], 0, limit);

  return jobs.map(job => ({
    id: job.id,
    name: job.name,
    data: job.data,
    timestamp: job.timestamp,
    processedOn: job.processedOn,
    finishedOn: job.finishedOn,
    attemptsMade: job.attemptsMade,
    failedReason: job.failedReason,
  }));
}
```

### Health Check

```typescript
export async function checkJobSystemHealth(): Promise<HealthStatus> {
  try {
    // Check Redis connection
    await connection.ping();

    // Check queue stats
    const stats = await getQueueStats();
    const dlqCount = stats.find(s => s.name === 'dead-letter')?.failed || 0;

    // Alert if DLQ has too many jobs
    if (dlqCount > 100) {
      logger.warn('Dead letter queue growing', { count: dlqCount });
    }

    // Check for stuck jobs (active > 5 minutes)
    const stuckJobs = stats.filter(s => s.active > 0 && s.name !== 'dead-letter');

    return {
      status: dlqCount > 500 ? 'degraded' : 'healthy',
      redis: 'connected',
      queues: stats,
      stuckJobs: stuckJobs.length,
      timestamp: new Date().toISOString(),
    };
  } catch (error) {
    return {
      status: 'unhealthy',
      error: error.message,
      timestamp: new Date().toISOString(),
    };
  }
}
```

### API Endpoints for Monitoring

| Method | Endpoint                    | Description                | Auth  |
|--------|-----------------------------|----------------------------|-------|
| GET    | /admin/jobs/stats           | Get all queue stats        | Admin |
| GET    | /admin/jobs/:queue          | Get queue details          | Admin |
| GET    | /admin/jobs/:queue/history  | Get job history            | Admin |
| POST   | /admin/jobs/:queue/:id/retry| Retry failed job           | Admin |
| DELETE | /admin/jobs/:queue/:id      | Remove job                 | Admin |
| GET    | /admin/jobs/dlq             | List dead letter jobs      | Admin |
| POST   | /admin/jobs/dlq/:id/retry   | Retry DLQ job              | Admin |
| GET    | /admin/jobs/health          | Job system health check    | Admin |
