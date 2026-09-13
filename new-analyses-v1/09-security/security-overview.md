# YemenMart Security Overview

## Table of Contents

1. [Security Architecture](#1-security-architecture)
2. [Authentication Security](#2-authentication-security)
3. [Authorization Security](#3-authorization-security)
4. [Data Protection](#4-data-protection)
5. [API Security](#5-api-security)
6. [Payment Security](#6-payment-security)
7. [Database Security](#7-database-security)
8. [File Upload Security](#8-file-upload-security)
9. [Infrastructure Security](#9-infrastructure-security)
10. [Monitoring and Logging](#10-monitoring-and-logging)
11. [Incident Response](#11-incident-response)
12. [Threat Model Summary](#12-threat-model-summary)
13. [Security Test Points](#13-security-test-points)
14. [Acceptance Criteria](#14-acceptance-criteria)

---

## 1. Security Architecture

### 1.1 Security Layers

```
┌─────────────────────────────────────────────────────┐
│                   CLIENT LAYER                       │
│  HTTPS │ CSP │ HSTS │ Anti-Fraud │ Bot Detection    │
├─────────────────────────────────────────────────────┤
│                   API GATEWAY                        │
│  Rate Limiting │ WAF │ DDoS Protection │ CORS       │
├─────────────────────────────────────────────────────┤
│                 APPLICATION LAYER                    │
│  JWT Auth │ RBAC │ Input Validation │ CSRF          │
├─────────────────────────────────────────────────────┤
│                  SERVICE LAYER                       │
│  Business Logic │ Idempotency │ Audit Logging       │
├─────────────────────────────────────────────────────┤
│                  DATA LAYER                          │
│  Parameterized Queries │ Encryption │ Backup         │
├─────────────────────────────────────────────────────┤
│               INFRASTRUCTURE LAYER                   │
│  VPC │ Firewall │ Secret Management │ Monitoring     │
└─────────────────────────────────────────────────────┘
```

### 1.2 Security Principles

| Principle | Implementation |
|-----------|---------------|
| Least Privilege | Role-based access with minimal permissions |
| Defense in Depth | Multiple security layers for every asset |
| Separation of Duties | Different roles for creation, approval, execution |
| Fail Secure | Default deny, explicit allow |
| Complete Mediation | Every request validated at each layer |
| Economy of Mechanism | Simple, auditable security controls |
| Open Design | Security through proper configuration, not obscurity |

### 1.3 Trust Boundaries

```
Internet ──[HTTPS]──> CDN/WAF ──> API Gateway ──> Auth Middleware ──> Business Logic ──> Database
                        │              │              │
                        ▼              ▼              ▼
                   DDoS Shield    Rate Limiter    Permission Check
                   Bot Detection  IP Blocking     Audit Logging
```

---

## 2. Authentication Security

### 2.1 JWT Configuration

```json
{
  "algorithm": "RS256",
  "access_token_expiry": "15m",
  "refresh_token_expiry": "7d",
  "issuer": "yemenmart.auth",
  "audience": "yemenmart.api"
}
```

**Security Requirements:**
- RS256 asymmetric signing (private key on server, public key for verification)
- Short-lived access tokens (15 minutes)
- Refresh tokens stored in HTTP-only, Secure, SameSite=Strict cookies
- Token rotation on refresh
- Token revocation capability via blocklist

### 2.2 Password Security

| Requirement | Value |
|-------------|-------|
| Algorithm | bcrypt |
| Cost Factor | 12 |
| Minimum Length | 8 characters |
| Complexity | Uppercase, lowercase, number, special character |
| History Check | Last 5 passwords prevented |
| Lockout Policy | 5 failed attempts → 15 minute lockout |

```typescript
// Password hashing
const HASH_ROUNDS = 12;
const hash = await bcrypt.hash(password, HASH_ROUNDS);

// Password validation
const MIN_LENGTH = 8;
const PATTERNS = {
  uppercase: /[A-Z]/,
  lowercase: /[a-z]/,
  number: /[0-9]/,
  special: /[!@#$%^&*(),.?":{}|<>]/
};
```

### 2.3 OTP Security

| Parameter | Value |
|-----------|-------|
| Code Length | 6 digits |
| Expiry | 5 minutes |
| Max Attempts | 3 |
| Cooldown | 60 seconds between requests |
| Purpose Isolation | Separate codes for login, registration, reset |
| Phone Verification | Required before account activation |

**Anti-Brute-Force:**
- Rate limit: 3 OTP requests per phone per 10 minutes
- Progressive delay after failed attempts
- Lock phone number after 5 failed OTP attempts in 1 hour

### 2.4 Session Management

```
Token Lifecycle:
  Access Token (15min) ──[expired]──> Use Refresh Token ──> New Access Token
                              │
                              ├──[revoked]──> Re-authenticate
                              │
                              └──[stolen]──> Detect anomaly ──> Revoke all sessions
```

**Security Controls:**
- Maximum 5 active sessions per user
- Session invalidation on password change
- Device fingerprinting for anomaly detection
- IP-based session binding (optional)

---

## 3. Authorization Security

### 3.1 Role-Based Access Control (RBAC)

```yaml
Roles:
  super_admin:
    permissions: ["*"]
    description: Full system access
    
  admin:
    permissions:
      - users:read, users:update
      - vendors:read, vendors:update, vendors:approve_kyc
      - products:read, products:update, products:delete
      - orders:read, orders:update
      - finance:read
      - reports:read
      - support:read, support:update
    description: Platform administration
    
  vendor:
    permissions:
      - products:create, products:read, products:update (own)
      - orders:read (own), orders:update (own)
      - inventory:read, inventory:update (own)
      - store:read, store:update (own)
      - finance:read (own)
      - reviews:read (own products)
    description: Vendor store management
    
  customer:
    permissions:
      - products:read
      - orders:create, orders:read (own)
      - cart:read, cart:update (own)
      - wishlist:read, wishlist:update (own)
      - reviews:create (verified purchases)
      - support:create
    description: End customer operations
```

### 3.2 Resource Ownership Verification

```typescript
// Every resource access must verify ownership
async function authorizeResourceAccess(
  userId: string,
  role: string,
  resourceType: string,
  resourceId: string,
  action: string
): Promise<boolean> {
  // Super admin bypasses ownership check
  if (role === 'super_admin') return true;
  
  // Admin has broad access but not ownership resources
  if (role === 'admin') return checkAdminPermissions(resourceType, action);
  
  // Vendor and customer must own the resource
  const ownership = await verifyOwnership(userId, resourceType, resourceId);
  return ownership.isValid && hasPermission(role, resourceType, action);
}
```

### 3.3 Permission Matrix

| Resource | super_admin | admin | vendor | customer |
|----------|:-----------:|:-----:|:------:|:--------:|
| users.read | Full | Own | - | - |
| users.update | Full | Own | - | Own |
| vendors.read | Full | Full | Own | Public |
| vendors.update | Full | Full | Own | - |
| products.read | Full | Full | Own | Public |
| products.create | Full | Yes | Own | - |
| products.update | Full | Yes | Own | - |
| products.delete | Full | Yes | Own | - |
| orders.read | Full | Full | Own | Own |
| orders.update | Full | Yes | Own | Own |
| inventory.read | Full | Full | Own | - |
| inventory.update | Full | Yes | Own | - |
| finance.read | Full | Full | Own | - |
| finance.update | Full | Yes | - | - |
| reviews.read | Full | Full | Own products | Public |
| reviews.create | - | - | - | Verified |
| support.read | Full | Full | Own tickets | Own tickets |
| coupons.create | Full | Yes | Own | - |

---

## 4. Data Protection

### 4.1 Data Classification

| Classification | Examples | Protection Level |
|---------------|----------|-----------------|
| **Public** | Product names, prices, store names | Integrity only |
| **Internal** | Analytics, aggregate statistics | Access control |
| **Confidential** | Email, phone, addresses | Encryption + Access control |
| **Restricted** | Passwords, payment data, KYC docs | Strong encryption + Audit + Access control |

### 4.2 Encryption Standards

| Data Type | At Rest | In Transit | Key Management |
|-----------|---------|------------|----------------|
| Passwords | bcrypt (cost 12) | N/A | N/A |
| PII (email, phone) | AES-256-GCM | TLS 1.3 | HSM / KMS |
| Payment data | AES-256-GCM | TLS 1.3 | HSM / KMS |
| KYC documents | AES-256-GCM | TLS 1.3 | HSM / KMS |
| Session tokens | N/A | TLS 1.3 | N/A |
| API keys | AES-256-GCM | TLS 1.3 | Vault |

### 4.3 Data Masking Rules

| Field | Masking Pattern | Example |
|-------|----------------|---------|
| Phone | +967 XX XXX XXXX | +967 77 123 4567 |
| Email | j***@example.com | john@example.com |
| Card Number | ****-****-****-1234 | 4111-****-****-1234 |
| National ID | XXXXXXXXX1234 | 12345678901234 |
| IP Address | 192.168.x.x | 192.168.1.x |

### 4.4 Data Retention

| Data Type | Retention Period | Deletion Method |
|-----------|-----------------|-----------------|
| Active user accounts | Indefinite | Soft delete |
| Deleted accounts | 30 days | Hard delete with overwrite |
| Session tokens | 7 days | Auto-expire |
| OTP codes | 24 hours | Auto-purge |
| Audit logs | 2 years | Archive then delete |
| Transaction records | 7 years | Archive |
| KYC documents | 5 years after account closure | Secure wipe |
| Support tickets | 1 year after resolution | Anonymize |
| Marketing consent | Until withdrawn | Delete on request |

### 4.5 GDPR / Privacy Compliance

- **Right to Access:** Users can export all personal data
- **Right to Erasure:** Account deletion with 30-day grace period
- **Right to Rectification:** Profile update without audit trail gaps
- **Right to Portability:** Data export in JSON format
- **Consent Management:** Granular opt-in/opt-out for marketing
- **Data Processing Agreements:** Required for all third-party processors

---

## 5. API Security

### 5.1 Rate Limiting

```yaml
Rate Limit Tiers:
  anonymous:
    requests: 60
    window: 1 minute
    burst: 10
    
  authenticated:
    requests: 300
    window: 1 minute
    burst: 50
    
  vendor:
    requests: 600
    window: 1 minute
    burst: 100
    
  admin:
    requests: 1200
    window: 1 minute
    burst: 200

Endpoint-Specific Limits:
  /api/v1/auth/login: 5 requests per minute per IP
  /api/v1/auth/otp: 3 requests per minute per phone
  /api/v1/orders: 30 requests per minute per user
  /api/v1/products: 100 requests per minute per user
```

### 5.2 CORS Configuration

```typescript
const corsOptions = {
  origin: [
    'https://yemenmart.com',
    'https://www.yemenmart.com',
    'https://admin.yemenmart.com'
  ],
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH'],
  allowedHeaders: ['Content-Type', 'Authorization', 'X-Request-ID'],
  exposedHeaders: ['X-RateLimit-Limit', 'X-RateLimit-Remaining'],
  credentials: true,
  maxAge: 86400
};
```

### 5.3 Security Headers

```
Content-Security-Policy: default-src 'self'; script-src 'self' 'nonce-{random}'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; font-src 'self' https://fonts.gstatic.com; connect-src 'self' https://api.yemenmart.com; frame-ancestors 'none'; base-uri 'self'; form-action 'self'
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 0
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: camera=(), microphone=(), geolocation=(self), payment=(self)
X-Permitted-Cross-Domain-Policies: none
```

### 5.4 Request Validation

```typescript
// Input validation schema
const createProductSchema = z.object({
  name_ar: z.string().min(1).max(300).trim(),
  name_en: z.string().min(1).max(300).trim(),
  category_id: z.string().uuid(),
  description_ar: z.string().max(5000).optional(),
  description_en: z.string().max(5000).optional(),
  variants: z.array(z.object({
    sku: z.string().min(1).max(50).regex(/^[A-Z0-9-]+$/),
    price: z.number().positive().max(999999),
    stock_quantity: z.number().int().min(0),
    attributes: z.record(z.string()).optional()
  })).min(1).max(50)
}).strict();
```

### 5.5 Request ID Tracking

```typescript
// Every request gets a unique ID for tracing
app.use((req, res, next) => {
  req.id = req.headers['x-request-id'] || uuid();
  res.setHeader('X-Request-ID', req.id);
  next();
});
```

---

## 6. Payment Security

### 6.1 Idempotency

```typescript
// Idempotency key for all financial operations
interface IdempotencyRecord {
  key: string;
  user_id: string;
  endpoint: string;
  response_status: number;
  response_body: any;
  created_at: Date;
  expires_at: Date;
}

// Every payment endpoint requires idempotency key
app.post('/api/v1/wallet/topup', requireIdempotencyKey, async (req, res) => {
  const idempotencyKey = req.headers['idempotency-key'];
  
  // Check if request was already processed
  const existing = await db.idempotencyKeys.findOne({ key: idempotencyKey });
  if (existing) {
    return res.status(existing.response_status).json(existing.response_body);
  }
  
  // Process and store result
  const result = await processTopup(req.body);
  await db.idempotencyKeys.create({
    key: idempotencyKey,
    user_id: req.user.id,
    endpoint: '/api/v1/wallet/topup',
    response_status: result.status,
    response_body: result.body,
    expires_at: new Date(Date.now() + 24 * 60 * 60 * 1000)
  });
  
  return res.status(result.status).json(result.body);
});
```

### 6.2 Atomic Operations

```sql
-- Wallet transfer with row-level locking
BEGIN;

-- Lock source wallet
SELECT balance FROM wallets
WHERE id = $1 AND status = 'active'
FOR UPDATE;

-- Verify sufficient balance
DO $$
BEGIN
  IF (SELECT balance FROM wallets WHERE id = $1) < $2 THEN
    RAISE EXCEPTION 'Insufficient balance';
  END IF;
END $$;

-- Debit source
UPDATE wallets
SET balance = balance - $2,
    updated_at = NOW()
WHERE id = $1;

-- Credit destination
UPDATE wallets
SET balance = balance + $2,
    updated_at = NOW()
WHERE id = $3;

-- Record transactions
INSERT INTO wallet_transactions (id, wallet_id, type, amount, balance_after, reference_type, reference_id)
VALUES
  (gen_random_uuid(), $1, 'debit', -$2, (SELECT balance FROM wallets WHERE id = $1), 'transfer', $4),
  (gen_random_uuid(), $3, 'credit', $2, (SELECT balance FROM wallets WHERE id = $3), 'transfer', $4);

COMMIT;
```

### 6.3 Double-Spend Prevention

```typescript
// Optimistic locking for concurrent operations
async function debitWallet(
  walletId: string,
  amount: number,
  reference: string
): Promise<WalletTransaction> {
  const maxRetries = 3;
  
  for (let attempt = 0; attempt < maxRetries; attempt++) {
    const wallet = await db.wallets.findOne({ id: walletId });
    
    if (wallet.balance < amount) {
      throw new InsufficientBalanceError();
    }
    
    const result = await db.wallets.updateOne(
      { id: walletId, balance: wallet.balance },
      {
        $inc: { balance: -amount },
        $set: { updated_at: new Date() }
      }
    );
    
    if (result.modifiedCount === 1) {
      return db.walletTransactions.create({
        wallet_id: walletId,
        type: 'debit',
        amount: -amount,
        balance_after: wallet.balance - amount,
        reference
      });
    }
    
    // Retry on conflict
    await sleep(50 * Math.pow(2, attempt));
  }
  
  throw new ConcurrencyConflictError();
}
```

### 6.4 Payment Provider Webhook Security

```typescript
// Webhook signature verification
function verifyWebhookSignature(
  payload: Buffer,
  signature: string,
  secret: string
): boolean {
  const expectedSignature = crypto
    .createHmac('sha256', secret)
    .update(payload)
    .digest('hex');
  
  return crypto.timingSafeEqual(
    Buffer.from(signature),
    Buffer.from(expectedSignature)
  );
}

// Process webhook
app.post('/webhooks/payment', async (req, res) => {
  const signature = req.headers['x-webhook-signature'];
  
  if (!verifyWebhookSignature(req.rawBody, signature, process.env.WEBHOOK_SECRET)) {
    logger.warn('Invalid webhook signature', { ip: req.ip });
    return res.status(401).json({ error: 'Invalid signature' });
  }
  
  // Deduplicate webhook
  const eventId = req.body.event_id;
  if (await isWebhookProcessed(eventId)) {
    return res.status(200).json({ received: true });
  }
  
  await processPaymentEvent(req.body);
  await markWebhookProcessed(eventId);
  
  return res.status(200).json({ received: true });
});
```

---

## 7. Database Security

### 7.1 SQL Injection Prevention

```typescript
// WRONG - vulnerable to SQL injection
const query = `SELECT * FROM users WHERE id = '${userId}'`;

// CORRECT - parameterized query
const result = await db.query(
  'SELECT * FROM users WHERE id = $1',
  [userId]
);

// CORRECT - using ORM
const user = await User.findOne({ where: { id: userId } });
```

### 7.2 Query Parameterization

```typescript
// All queries must use parameterized statements
class ProductRepository {
  async search(filters: ProductFilters): Promise<Product[]> {
    const conditions: string[] = ['p.status = $1'];
    const params: any[] = ['active'];
    let paramIndex = 2;
    
    if (filters.categoryId) {
      conditions.push(`p.category_id = $${paramIndex}`);
      params.push(filters.categoryId);
      paramIndex++;
    }
    
    if (filters.minPrice) {
      conditions.push(`pv.price >= $${paramIndex}`);
      params.push(filters.minPrice);
      paramIndex++;
    }
    
    if (filters.search) {
      conditions.push(`(
        p.name_ar ILIKE $${paramIndex} OR 
        p.name_en ILIKE $${paramIndex}
      )`);
      params.push(`%${filters.search}%`);
      paramIndex++;
    }
    
    const query = `
      SELECT p.*, pv.price, pv.sku
      FROM products p
      JOIN product_variants pv ON pv.product_id = p.id
      WHERE ${conditions.join(' AND ')}
      ORDER BY p.score DESC
      LIMIT $${paramIndex} OFFSET $${paramIndex + 1}
    `;
    
    params.push(filters.limit || 20, filters.offset || 0);
    
    return db.query(query, params);
  }
}
```

### 7.3 Database Access Control

```yaml
Database Users:
  app_readonly:
    permissions: SELECT
    tables: all
    connection_limit: 20
    
  app_readwrite:
    permissions: SELECT, INSERT, UPDATE, DELETE
    tables: all
    connection_limit: 10
    
  app_admin:
    permissions: ALL
    tables: schema_migrations, audit_logs
    connection_limit: 2
    
  backup_user:
    permissions: SELECT
    tables: all
    connection_limit: 1
    access_hours: "02:00-04:00"
```

### 7.4 Data at Rest Encryption

```sql
-- PostgreSQL TDE (Transparent Data Encryption)
-- Encrypted tablespace for sensitive data
CREATE TABLESPACE sensitive_data
  LOCATION '/encrypted/tablespace'
  WITH (encryption = 'aes-256');

-- Sensitive tables in encrypted tablespace
ALTER TABLE users SET TABLESPACE sensitive_data;
ALTER TABLE wallets SET TABLESPACE sensitive_data;
ALTER TABLE kyc_documents SET TABLESPACE sensitive_data;
```

---

## 8. File Upload Security

### 8.1 Allowed File Types

```yaml
Product Images:
  allowed_types:
    - image/jpeg
    - image/png
    - image/webp
  max_size: 5MB
  max_dimensions: 4000x4000
  required: false
  
KYC Documents:
  allowed_types:
    - application/pdf
    - image/jpeg
    - image/png
  max_size: 10MB
  required: false
  
Profile Avatars:
  allowed_types:
    - image/jpeg
    - image/png
    - image/webp
  max_size: 2MB
  max_dimensions: 500x500
  
Support Attachments:
  allowed_types:
    - image/jpeg
    - image/png
    - application/pdf
  max_size: 5MB
  
Banner Images:
  allowed_types:
    - image/jpeg
    - image/png
    - image/webp
  max_size: 2MB
  dimensions:
    desktop: 1920x600
    mobile: 750x400
```

### 8.2 Upload Pipeline

```typescript
async function processUpload(file: UploadedFile, context: UploadContext) {
  // Step 1: Validate file type (magic bytes, not just extension)
  const fileType = await FileType.fromBuffer(file.buffer);
  if (!context.allowedTypes.includes(fileType.mime)) {
    throw new ValidationError('Invalid file type');
  }
  
  // Step 2: Validate file size
  if (file.size > context.maxSize) {
    throw new ValidationError('File too large');
  }
  
  // Step 3: Scan for malware
  const scanResult = await antivirusScanner.scan(file.buffer);
  if (scanResult.infected) {
    logger.warn('Malware detected', { filename: file.originalname, virus: scanResult.virus });
    throw new SecurityError('File rejected');
  }
  
  // Step 4: Strip EXIF data (prevent metadata leakage)
  const cleanBuffer = await sharp(file.buffer)
    .rotate() // Auto-rotate based on EXIF
    .withMetadata({ orientation: undefined })
    .toBuffer();
  
  // Step 5: Generate secure filename
  const safeFilename = generateSecureFilename(file.originalname);
  const filePath = `${context.folder}/${safeFilename}`;
  
  // Step 6: Upload to secure storage
  await storage.upload(filePath, cleanBuffer, {
    contentType: fileType.mime,
    acl: 'private',
    serverSideEncryption: 'AES256'
  });
  
  // Step 7: Generate signed URL for access
  const signedUrl = await storage.getSignedUrl(filePath, {
    expiresIn: 3600,
    responseContentType: fileType.mime
  });
  
  return { filePath, signedUrl, fileType: fileType.mime };
}
```

### 8.3 File Content Validation

```typescript
// Image validation with sharp
async function validateImage(buffer: Buffer): Promise<ImageValidation> {
  const metadata = await sharp(buffer).metadata();
  
  // Check for reasonable dimensions
  if (metadata.width > 4000 || metadata.height > 4000) {
    throw new ValidationError('Image dimensions too large');
  }
  
  if (metadata.width < 100 || metadata.height < 100) {
    throw new ValidationError('Image dimensions too small');
  }
  
  // Check for suspicious format combinations
  if (metadata.format === 'jpeg' && metadata.channels === 4) {
    throw new ValidationError('Invalid JPEG format');
  }
  
  // Check for potentially malicious content
  if (buffer.includes(Buffer.from('JavaScript'))) {
    throw new SecurityError('Suspicious file content');
  }
  
  return { valid: true, metadata };
}
```

### 8.4 Access Control for Files

```typescript
// Signed URL generation with access control
async function getSignedUrl(
  userId: string,
  filePath: string
): Promise<string> {
  const file = await db.files.findOne({ path: filePath });
  
  if (!file) throw new NotFoundError('File not found');
  
  // Check access permissions
  const hasAccess = await checkFileAccess(userId, file);
  if (!hasAccess) throw new ForbiddenError('Access denied');
  
  // Generate time-limited signed URL
  return storage.getSignedUrl(filePath, {
    expiresIn: 3600, // 1 hour
    conditions: {
      'Content-Type': file.content_type
    }
  });
}
```

---

## 9. Infrastructure Security

### 9.1 Network Security

```yaml
VPC Configuration:
  vpc_cidr: 10.0.0.0/16
  
  Public Subnets:
    - 10.0.1.0/24 (AZ-a)
    - 10.0.2.0/24 (AZ-b)
    Resources: Load Balancer, NAT Gateway
    
  Private Subnets:
    - 10.0.10.0/24 (AZ-a)
    - 10.0.20.0/24 (AZ-b)
    Resources: Application Servers, Workers
    
  Database Subnets:
    - 10.0.100.0/24 (AZ-a)
    - 10.0.200.0/24 (AZ-b)
    Resources: PostgreSQL, Redis

Security Groups:
  alb-sg:
    ingress:
      - 80/tcp from 0.0.0.0/0
      - 443/tcp from 0.0.0.0/0
    egress: all
    
  app-sg:
    ingress:
      - 8080/tcp from alb-sg
    egress:
      - 5432/tcp to db-sg
      - 6379/tcp to redis-sg
      - 443/tcp to 0.0.0.0/0 (external APIs)
      
  db-sg:
    ingress:
      - 5432/tcp from app-sg
    egress: none
    
  redis-sg:
    ingress:
      - 6379/tcp from app-sg
    egress: none
```

### 9.2 HTTPS Configuration

```yaml
TLS Configuration:
  minimum_version: TLSv1.2
  preferred_version: TLSv1.3
  
  cipher_suites:
    - TLS_AES_256_GCM_SHA384
    - TLS_CHACHA20_POLY1305_SHA256
    - TLS_AES_128_GCM_SHA256
    - ECDHE-RSA-AES256-GCM-SHA384
    - ECDHE-RSA-AES128-GCM-SHA256
    
  certificate:
    provider: Let's Encrypt
    auto_renewal: true
    key_type: ECDSA P-256
    
  hsts:
    max_age: 31536000
    include_subdomains: true
    preload: true
```

### 9.3 Secrets Management

```yaml
Secret Storage:
  provider: HashiCorp Vault
  authentication: AppRole
  secret_engine: kv-v2
  
Secret Rotation:
  database_password: every 30 days
  api_keys: every 90 days
  jwt_signing_key: every 180 days
  webhook_secrets: every 90 days
  
Access Policy:
  app_role:
    capabilities: read
    paths:
      - secret/data/yemenmart/app/*
  admin_role:
    capabilities: read, list, create, update, delete
    paths:
      - secret/data/yemenmart/*
```

### 9.4 Container Security

```yaml
Docker Security:
  base_image: node:20-alpine
  run_as_user: node
  read_only_filesystem: true
  no_new_privileges: true
  
  capabilities:
    drop: [ALL]
    add: []
    
  resource_limits:
    memory: 512MB
    cpus: "0.5"
    
  health_check:
    path: /health
    interval: 30s
    timeout: 5s
    retries: 3
```

---

## 10. Monitoring and Logging

### 10.1 Security Events to Log

```yaml
Authentication Events:
  - login_success
  - login_failure
  - logout
  - password_change
  - password_reset_request
  - password_reset_complete
  - otp_sent
  - otp_verified
  - otp_failed
  - session_created
  - session_destroyed
  - account_locked
  - account_unlocked

Authorization Events:
  - access_denied
  - permission_change
  - role_change
  - privilege_escalation_attempt

Data Events:
  - pii_access
  - pii_export
  - pii_deletion
  - bulk_data_export
  - schema_change

Payment Events:
  - payment_initiated
  - payment_completed
  - payment_failed
  - refund_initiated
  - refund_completed
  - wallet_topup
  - wallet_transfer
  - escrow_held
  - escrow_released

System Events:
  - config_change
  - secret_accessed
  - backup_completed
  - backup_failed
  - service_restart
  - deployment
```

### 10.2 Log Structure

```json
{
  "timestamp": "2026-01-15T10:30:00.000Z",
  "level": "INFO",
  "event": "login_success",
  "service": "auth-service",
  "trace_id": "abc-123-def-456",
  "user_id": "uuid-or-null",
  "ip_address": "192.168.1.100",
  "user_agent": "Mozilla/5.0...",
  "resource": "/api/v1/auth/login",
  "method": "POST",
  "status_code": 200,
  "duration_ms": 150,
  "details": {
    "phone": "+967771234567",
    "otp_purpose": "login",
    "session_id": "session-uuid"
  },
  "security_context": {
    "risk_score": 0.1,
    "geo_location": "Sana'a, Yemen",
    "device_fingerprint": "fp-hash",
    "is_vpn": false
  }
}
```

### 10.3 Alert Rules

```yaml
Critical Alerts (P1):
  - condition: failed_logins > 10 in 5 minutes from same IP
    action: block_ip, notify_secops
    
  - condition: sql_injection_attempt detected
    action: block_request, alert_secops
    
  - condition: unauthorized_admin_access
    action: block_session, alert_secops
    
  - condition: data_export > 1000 records in 1 hour
    action: alert_secops, review
    
  - condition: payment_anomaly detected
    action: hold_transaction, alert_finance

Warning Alerts (P2):
  - condition: rate_limit_exceeded > 5 times in 10 minutes
    action: temporary_block, log
    
  - condition: invalid_file_upload > 3 times in 5 minutes
    action: temporary_block, log
    
  - condition: api_error_rate > 5% in 15 minutes
    action: alert_devops
    
  - condition: database_connection_pool > 80%
    action: alert_devops

Info Alerts (P3):
  - condition: new_admin_login from unusual location
    action: log, notify_admin
    
  - condition: large_order > $1000
    action: log, notify_finance
```

### 10.4 Log Retention and Storage

| Log Type | Retention | Storage | Archival |
|----------|-----------|---------|----------|
| Security events | 2 years | Hot (30 days) → Warm (90 days) → Cold (2 years) | S3 Glacier |
| Audit logs | 7 years | Hot (30 days) → Warm (1 year) → Cold (7 years) | S3 Glacier |
| Application logs | 90 days | Hot (30 days) → Warm (90 days) | Delete |
| Access logs | 1 year | Hot (30 days) → Warm (1 year) | S3 Standard |
| Error logs | 6 months | Hot (30 days) → Warm (6 months) | Delete |

---

## 11. Incident Response

### 11.1 Incident Classification

| Severity | Description | Response Time | Resolution Target |
|----------|-------------|---------------|-------------------|
| **P1 - Critical** | Active breach, data leak, payment fraud | 15 minutes | 4 hours |
| **P2 - High** | Service compromise, unauthorized access | 30 minutes | 8 hours |
| **P3 - Medium** | Vulnerability exploitation attempt | 2 hours | 24 hours |
| **P4 - Low** | Suspicious activity, failed attempts | 24 hours | 72 hours |

### 11.2 Response Procedures

```yaml
Detection:
  - Automated monitoring alerts
  - User reports
  - Third-party notifications
  - Security audit findings

Triage:
  - Verify the incident
  - Classify severity
  - Assign response team
  - Create incident ticket

Containment:
  - Isolate affected systems
  - Revoke compromised credentials
  - Block malicious IPs
  - Preserve evidence

Eradication:
  - Remove malicious code
  - Patch vulnerabilities
  - Update access controls
  - Clear compromised sessions

Recovery:
  - Restore from clean backups
  - Re-enable services
  - Verify system integrity
  - Monitor for recurrence

Post-Incident:
  - Conduct post-mortem
  - Update security controls
  - Document lessons learned
  - Notify affected users (if required)
  - File regulatory reports (if required)
```

### 11.3 Communication Templates

```yaml
Internal Notification:
  template: |
    [SECURITY INCIDENT - {severity}]
    Time: {timestamp}
    Summary: {summary}
    Impact: {impact}
    Response: {current_actions}
    Next Steps: {next_steps}
    
External Notification (if P1/P2):
  template: |
    Subject: Security Notice - YemenMart
    
    Dear {user_name},
    
    We are writing to inform you of a security incident that may have affected your account.
    
    What happened: {description}
    What information was involved: {data_types}
    What we are doing: {actions_taken}
    What you can do: {user_actions}
    
    We sincerely apologize for any inconvenience.
```

---

## 12. Threat Model Summary

### 12.1 Threat Categories (55 Threats)

#### Authentication Threats (T01-T08)

| ID | Threat | Risk Level | Mitigation |
|----|--------|-----------|------------|
| T01 | Credential stuffing attack | High | Rate limiting, CAPTCHA, account lockout |
| T02 | Brute force OTP | High | Rate limiting, progressive delay, lockout |
| T03 | JWT token theft | High | Short expiry, refresh rotation, HTTPS only |
| T04 | Session hijacking | Medium | Secure cookies, session binding, anomaly detection |
| T05 | Password spray attack | High | Account lockout, breach detection, MFA |
| T06 | SIM swapping for OTP | Medium | Device binding, anomaly detection |
| T07 | Token replay attack | Medium | Short expiry, nonce validation |
| T08 | Weak password policy | Medium | Complexity requirements, breach database check |

#### Authorization Threats (T09-T15)

| ID | Threat | Risk Level | Mitigation |
|----|--------|-----------|------------|
| T09 | Horizontal privilege escalation | High | Ownership verification, resource-level auth |
| T10 | Vertical privilege escalation | Critical | RBAC enforcement, audit logging |
| T11 | IDOR (Insecure Direct Object Reference) | High | UUID resources, ownership checks |
| T12 | Missing function-level access control | High | Middleware auth checks on every route |
| T13 | JWT claim manipulation | Critical | Server-side validation, signing verification |
| T14 | Role parameter tampering | High | Server-side role assignment only |
| T15 | API endpoint bypass | Medium | Centralized auth middleware, route guards |

#### Data Threats (T16-T24)

| ID | Threat | Risk Level | Mitigation |
|----|--------|-----------|------------|
| T16 | SQL injection | Critical | Parameterized queries, ORM, input validation |
| T17 | NoSQL injection | High | Input validation, parameterized queries |
| T18 | XSS (Cross-Site Scripting) | High | Output encoding, CSP, input sanitization |
| T19 | CSRF (Cross-Site Request Forgery) | Medium | CSRF tokens, SameSite cookies |
| T20 | Data exfiltration | Critical | Encryption, access control, monitoring |
| T21 | PII exposure in logs | High | Log sanitization, masked fields |
| T22 | Backup data exposure | High | Encrypted backups, access control |
| T23 | Metadata leakage | Medium | EXIF stripping, header removal |
| T24 | Cache poisoning | Medium | Cache control headers, validation |

#### Payment Threats (T25-T33)

| ID | Threat | Risk Level | Mitigation |
|----|--------|-----------|------------|
| T25 | Double-spending | Critical | Atomic operations, optimistic locking |
| T26 | Payment amount tampering | Critical | Server-side price calculation |
| T27 | Webhook spoofing | High | Signature verification, IP whitelist |
| T28 | Refund fraud | High | Approval workflow, audit trail |
| T29 | Wallet balance manipulation | Critical | Row-level locking, transaction isolation |
| T30 | Idempotency key abuse | Medium | Rate limiting, key validation |
| T31 | Currency manipulation | High | Server-side currency validation |
| T32 | Escrow bypass | Critical | Automated escrow on order confirmation |
| T33 | Commission evasion | Medium | Automated commission calculation |

#### API Threats (T34-T40)

| ID | Threat | Risk Level | Mitigation |
|----|--------|-----------|------------|
| T34 | DDoS attack | High | CDN, rate limiting, auto-scaling |
| T35 | API abuse/scraping | Medium | Rate limiting, bot detection |
| T36 | Request smuggling | Medium | HTTP/2, proper parsing |
| T37 | GraphQL introspection abuse | Medium | Disable in production |
| T38 | CORS bypass | Medium | Strict origin validation |
| T39 | SSL/TLS downgrade | Medium | HSTS, strong cipher suites |
| T40 | API key leakage | High | Key rotation, vault storage |

#### Infrastructure Threats (T41-T47)

| ID | Threat | Risk Level | Mitigation |
|----|--------|-----------|------------|
| T41 | Container escape | Critical | Non-root user, capabilities drop |
| T42 | Secret leakage | Critical | Vault, encrypted at rest |
| T43 | Supply chain attack | High | Dependency scanning, lock files |
| T44 | DNS hijacking | High | DNSSEC, monitoring |
| T45 | Man-in-the-middle | High | TLS 1.3, certificate pinning |
| T46 | Server compromise | Critical | Hardening, monitoring, minimal images |
| T47 | Cloud misconfiguration | High | Infrastructure as code, scanning |

#### Business Logic Threats (T48-T55)

| ID | Threat | Risk Level | Mitigation |
|----|--------|-----------|------------|
| T48 | Coupon abuse | Medium | Usage limits, fraud detection |
| T49 | Price manipulation | High | Server-side validation |
| T50 | Review fraud | Medium | Verified purchase only, AI detection |
| T51 | Vendor collusion | Medium | Monitoring, rate limiting |
| T52 | Loyalty points exploitation | Medium | Point earning limits, monitoring |
| T53 | Return fraud | Medium | Return history tracking, limits |
| T54 | Fake KYC documents | High | Manual review, document verification |
| T55 | Account farming | Medium | Device fingerprinting, rate limiting |

### 12.2 Risk Matrix

```
Impact ↑
  High   │ T10 T16 T20 T25 T26 T29 T32 T42 T46
         │ T13 T33
         │
  Medium │ T01 T02 T03 T09 T11 T12 T17 T18
         │ T27 T28 T34 T37 T38 T43 T47 T49
         │
  Low    │ T04 T06 T07 T08 T19 T21 T22 T23
         │ T30 T31 T36 T39 T40 T44 T45
         │
  Info   │ T05 T14 T15 T24 T35 T41 T48 T50
         │ T51 T52 T53 T54 T55
         │
         └──────────────────────────────────→ Likelihood
              Low         Medium        High
```

---

## 13. Security Test Points

### 13.1 Authentication Tests (STP-01 to STP-05)

| ID | Test Point | Expected Result |
|----|-----------|-----------------|
| STP-01 | Attempt login with invalid credentials | 401 error, no user existence revealed |
| STP-02 | Attempt login with correct credentials from new device | OTP verification required |
| STP-03 | Submit OTP with expired code | 400 error, code expired |
| STP-04 | Submit OTP with wrong code (3 attempts) | Account temporarily locked |
| STP-05 | Attempt to use expired JWT token | 401 error, token expired |

### 13.2 Authorization Tests (STP-06 to STP-10)

| ID | Test Point | Expected Result |
|----|-----------|-----------------|
| STP-06 | Access vendor resource as customer | 403 forbidden |
| STP-07 | Access other user's order | 403 forbidden |
| STP-08 | Attempt admin action as vendor | 403 forbidden |
| STP-09 | Access soft-deleted resource | 404 not found |
| STP-10 | Attempt SQL injection in auth field | Input rejected, logged |

### 13.3 Data Protection Tests (STP-11 to STP-15)

| ID | Test Point | Expected Result |
|----|-----------|-----------------|
| STP-11 | Check password stored as hash | bcrypt hash, not plaintext |
| STP-12 | Verify PII encrypted in database | AES-256 encrypted columns |
| STP-13 | Check API response doesn't expose sensitive data | No passwords, tokens, internal IDs |
| STP-14 | Verify logs don't contain PII | Masked phone, email in logs |
| STP-15 | Test data export includes only user's data | Scoped to authenticated user |

### 13.4 Payment Tests (STP-16 to STP-20)

| ID | Test Point | Expected Result |
|----|-----------|-----------------|
| STP-16 | Attempt double wallet topup with same idempotency key | Second request returns cached result |
| STP-17 | Attempt wallet debit with insufficient funds | 400 error, balance check enforced |
| STP-18 | Submit order with manipulated price | Server recalculates, uses DB price |
| STP-19 | Attempt concurrent wallet debit | Optimistic locking prevents double-spend |
| STP-20 | Verify escrow created on order confirmation | Atomic transaction with order |

### 13.5 API Security Tests (STP-21 to STP-26)

| ID | Test Point | Expected Result |
|----|-----------|-----------------|
| STP-21 | Send request without CORS origin | Request blocked |
| STP-22 | Send request with invalid origin | Request blocked |
| STP-23 | Test rate limiting (61 requests/min for anon) | 429 error after limit |
| STP-24 | Upload file exceeding size limit | 413 error |
| STP-25 | Upload file with invalid type | 400 error |
| STP-26 | Check security headers in response | All required headers present |

---

## 14. Acceptance Criteria

### 14.1 Authentication Criteria (AC-01 to AC-10)

| ID | Acceptance Criterion |
|----|---------------------|
| AC-01 | Passwords are hashed with bcrypt (cost factor >= 12) before storage |
| AC-02 | JWT tokens expire within 15 minutes for access tokens |
| AC-03 | Refresh tokens are stored in HTTP-only, Secure, SameSite cookies |
| AC-04 | OTP codes expire after 5 minutes and are single-use |
| AC-05 | Account locks after 5 failed login attempts for 15 minutes |
| AC-06 | Password change invalidates all active sessions |
| AC-07 | Login from new device requires OTP verification |
| AC-08 | Session tokens are cryptographically random (>= 128 bits) |
| AC-09 | Password reset link expires after 1 hour |
| AC-10 | Failed authentication attempts are logged with IP and timestamp |

### 14.2 Authorization Criteria (AC-11 to AC-18)

| ID | Acceptance Criterion |
|----|---------------------|
| AC-11 | Every API endpoint has explicit role-based access control |
| AC-12 | Resource ownership is verified before any read/write operation |
| AC-13 | Users cannot access resources belonging to other users |
| AC-14 | Vendors can only manage their own store and products |
| AC-15 | Admin actions require admin or super_admin role |
| AC-16 | Role changes are logged and require appropriate permissions |
| AC-17 | API keys are scoped to specific permissions |
| AC-18 | Default deny: all resources require explicit access rules |

### 14.3 Data Protection Criteria (AC-19 to AC-28)

| ID | Acceptance Criterion |
|----|---------------------|
| AC-19 | All PII is encrypted at rest using AES-256-GCM |
| AC-20 | All data in transit uses TLS 1.3 |
| AC-21 | API responses never expose password hashes or internal IDs |
| AC-22 | Logs are sanitized of PII before storage |
| AC-23 | File uploads are validated by magic bytes, not extension |
| AC-24 | Uploaded images have EXIF data stripped |
| AC-25 | SQL queries use parameterized statements exclusively |
| AC-26 | User data export is scoped to the requesting user only |
| AC-27 | Data retention policies are enforced automatically |
| AC-28 | Backup data is encrypted with separate keys |

### 14.4 Payment Security Criteria (AC-29 to AC-38)

| ID | Acceptance Criterion |
|----|---------------------|
| AC-29 | All financial transactions are atomic with proper isolation |
| AC-30 | Wallet balance cannot go below zero |
| AC-31 | Price is always calculated server-side from database |
| AC-32 | Idempotency keys prevent duplicate transactions |
| AC-33 | Payment webhooks are verified by signature |
| AC-34 | Escrow is created automatically on order confirmation |
| AC-35 | Commission is calculated automatically on delivery |
| AC-36 | Refunds require approval workflow |
| AC-37 | Double-spending is prevented by optimistic locking |
| AC-38 | All payment events are audit-logged with full trace |

### 14.5 API Security Criteria (AC-39 to AC-48)

| ID | Acceptance Criterion |
|----|---------------------|
| AC-39 | Rate limits are enforced per user and per IP |
| AC-40 | CORS is configured to allow only trusted origins |
| AC-41 | Security headers are set on all responses |
| AC-42 | File upload size limits are enforced |
| AC-43 | Request body size limits are enforced (10MB max) |
| AC-44 | Input validation is performed on all endpoints |
| AC-45 | SQL injection attempts are blocked and logged |
| AC-46 | XSS attempts are prevented by output encoding |
| AC-47 | CSRF protection is enabled for state-changing operations |
| AC-48 | API versioning prevents breaking changes |

### 14.6 Infrastructure Security Criteria (AC-49 to AC-55)

| ID | Acceptance Criterion |
|----|---------------------|
| AC-49 | HTTPS is enforced with HSTS headers |
| AC-50 | Secrets are stored in a vault, not in code or config files |
| AC-51 | Database connections use SSL/TLS |
| AC-52 | Containers run as non-root user |
| AC-53 | Security scans are run on every deployment |
| AC-54 | Dependencies are scanned for known vulnerabilities |
| AC-55 | Infrastructure changes are tracked in version control |

### 14.7 Monitoring and Response Criteria (AC-56 to AC-62)

| ID | Acceptance Criterion |
|----|---------------------|
| AC-56 | All authentication events are logged |
| AC-57 | All authorization failures are logged |
| AC-58 | Critical security alerts trigger within 5 minutes |
| AC-59 | Audit logs are tamper-proof (append-only) |
| AC-60 | Security incidents are classified and responded to within SLA |
| AC-61 | P1 incidents are escalated within 15 minutes |
| AC-62 | Post-incident reviews are conducted for P1/P2 incidents |

---

## Appendix A: Security Checklist Summary

```
Authentication          [██████████] 10/10 criteria
Authorization           [██████████] 8/8 criteria
Data Protection         [████████████████] 10/10 criteria
Payment Security        [████████████████████████████████] 10/10 criteria
API Security            [████████████████████████████████] 10/10 criteria
Infrastructure          [██████████████] 7/7 criteria
Monitoring              [██████████████████████] 7/7 criteria
─────────────────────────────────────────────
Total                   [████████████████████████████████████████████████████████████████] 62/62 criteria
```

## Appendix B: Compliance Mapping

| Standard | Relevant Controls | Status |
|----------|-------------------|--------|
| OWASP Top 10 2021 | All 10 categories addressed | Compliant |
| PCI-DSS v4.0 | Payment card data handling | Compliant |
| GDPR | Data protection, user rights | Compliant |
| SOC 2 Type II | Security, availability, confidentiality | In Progress |
| ISO 27001 | Information security management | Planned |
