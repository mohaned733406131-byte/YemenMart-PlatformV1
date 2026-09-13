# Authentication Service

## Overview

YemenMart authentication uses **phone + password as the primary login method**. Phone + OTP (SMS/WhatsApp) is available as a secondary method for forgot password scenarios only. JWT tokens handle session state with short-lived access tokens and longer refresh tokens.

---

## Primary Login: Phone + Password

### Login Flow

```typescript
// packages/auth-module/src/auth.service.ts
@injectable()
export class AuthService implements IAuthService {
  constructor(
    @inject('PasswordService') private passwordService: PasswordService,
    @inject('TokenService') private tokenService: TokenService,
    @inject('LockoutService') private lockoutService: AccountLockoutService,
    @inject('PrismaClient') private prisma: PrismaClient,
  ) {}

  // PRIMARY: Phone + Password login
  async loginWithPassword(phone: string, password: string): Promise<AuthResult> {
    const isLocked = await this.lockoutService.isLocked(phone);
    if (isLocked) throw new AccountLockedError();

    const user = await this.prisma.user.findUnique({ where: { phone } });
    if (!user || !user.passwordHash) {
      throw new InvalidCredentialsError();
    }

    const valid = await this.passwordService.verifyPassword(password, user.passwordHash);
    if (!valid) {
      const status = await this.lockoutService.recordFailedAttempt(phone);
      if (status.locked) throw new AccountLockedError(status.lockoutExpiresIn);
      throw new InvalidCredentialsError();
    }

    await this.lockoutService.clearAttempts(phone);

    const tokens = await this.tokenService.generateTokens(user.id, user.role);
    return { user, ...tokens };
  }
```

  private maskPhone(phone: string): string {
    return phone.slice(0, 4) + '****' + phone.slice(-2);
  }
}
```

---

## JWT Token Generation

### Token Configuration

```typescript
// packages/auth-module/src/token.service.ts
const ACCESS_TOKEN_EXPIRY = '15m';
const REFRESH_TOKEN_EXPIRY = '7d';
const JWT_SECRET = process.env.JWT_SECRET;
const REFRESH_SECRET = process.env.JWT_REFRESH_SECRET;

interface TokenPayload {
  sub: string;       // User ID
  role: string;
  sessionId: string;
  iat?: number;
  exp?: number;
}
```

### Generate Tokens

```typescript
@injectable()
export class TokenService {
  constructor(
    @inject('PrismaClient') private prisma: PrismaClient,
  ) {}

  async generateTokens(userId: string, role: string): Promise<TokenPair> {
    const sessionId = uuidv4();

    // Create session record
    const session = await this.prisma.session.create({
      data: {
        userId,
        token: '', // Will be updated
        refreshToken: '', // Will be updated
        expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000), // 7 days
      },
    });

    const accessPayload: TokenPayload = {
      sub: userId,
      role,
      sessionId: session.id,
    };

    const refreshPayload: TokenPayload = {
      sub: userId,
      role,
      sessionId: session.id,
    };

    const token = jwt.sign(accessPayload, JWT_SECRET, {
      expiresIn: ACCESS_TOKEN_EXPIRY,
      issuer: 'yemenmart',
      audience: 'yemenmart-api',
    });

    const refreshToken = jwt.sign(refreshPayload, REFRESH_SECRET, {
      expiresIn: REFRESH_TOKEN_EXPIRY,
      issuer: 'yemenmart',
      audience: 'yemenmart-api',
    });

    // Update session with token hashes
    await this.prisma.session.update({
      where: { id: session.id },
      data: {
        token: await bcrypt.hash(token, 4), // Light hash for revocation check
        refreshToken: await bcrypt.hash(refreshToken, 4),
      },
    });

    // Enforce max 5 concurrent sessions per user
    await this.enforceSessionLimit(userId);

    return {
      accessToken: token,
      refreshToken,
      expiresIn: 900, // 15 minutes in seconds
      tokenType: 'Bearer',
    };
  }

  async verifyAccessToken(token: string): Promise<TokenPayload> {
    const payload = jwt.verify(token, JWT_SECRET, {
      issuer: 'yemenmart',
      audience: 'yemenmart-api',
    }) as TokenPayload;

    // Check session still exists and not revoked
    const session = await this.prisma.session.findUnique({
      where: { id: payload.sessionId },
    });

    if (!session || session.expiresAt < new Date()) {
      throw new SessionExpiredError();
    }

    return payload;
  }

  async refreshTokens(refreshToken: string): Promise<TokenPair> {
    const payload = jwt.verify(refreshToken, REFRESH_SECRET, {
      issuer: 'yemenmart',
      audience: 'yemenmart-api',
    }) as TokenPayload;

    // Verify session
    const session = await this.prisma.session.findUnique({
      where: { id: payload.sessionId },
      include: { user: true },
    });

    if (!session || session.expiresAt < new Date()) {
      throw new SessionExpiredError();
    }

    // Rotate refresh token
    await this.prisma.session.delete({ where: { id: session.id } });

    return this.generateTokens(payload.sub, payload.role);
  }

  async revokeSession(sessionId: string): Promise<void> {
    await this.prisma.session.delete({ where: { id: sessionId } });
  }

  async revokeAllSessions(userId: string): Promise<void> {
    await this.prisma.session.deleteMany({ where: { userId } });
  }

  private async enforceSessionLimit(userId: string, maxSessions = 5): Promise<void> {
    const sessions = await this.prisma.session.findMany({
      where: { userId },
      orderBy: { createdAt: 'desc' },
    });

    if (sessions.length > maxSessions) {
      const toDelete = sessions.slice(maxSessions);
      await this.prisma.session.deleteMany({
        where: { id: { in: toDelete.map(s => s.id) } },
      });
    }
  }
}
```

---

## Password Hashing (BCrypt)

```typescript
// packages/auth-module/src/password.service.ts
const BCRYPT_ROUNDS = 12;

export class PasswordService {
  async hashPassword(password: string): Promise<string> {
    return bcrypt.hash(password, BCRYPT_ROUNDS);
  }

  async verifyPassword(password: string, hash: string): Promise<boolean> {
    return bcrypt.compare(password, hash);
  }

  validatePasswordStrength(password: string): ValidationResult {
    const errors: string[] = [];

    if (password.length < 8) errors.push('Password must be at least 8 characters');
    if (password.length > 128) errors.push('Password must be at most 128 characters');
    if (!/[A-Z]/.test(password)) errors.push('Must contain at least one uppercase letter');
    if (!/[a-z]/.test(password)) errors.push('Must contain at least one lowercase letter');
    if (!/[0-9]/.test(password)) errors.push('Must contain at least one digit');
    if (!/[!@#$%^&*(),.?":{}|<>]/.test(password)) errors.push('Must contain at least one special character');

    // Check common passwords
    if (this.isCommonPassword(password)) {
      errors.push('Password is too common');
    }

    return { valid: errors.length === 0, errors };
  }

  private isCommonPassword(password: string): boolean {
    const common = [
      'password', '12345678', 'qwerty123', 'letmein', 'admin',
      'welcome', 'monkey', 'dragon', 'login', 'princess',
    ];
    return common.includes(password.toLowerCase());
  }
}
```

### Password History (Last 3 Blocked)

```typescript
// packages/auth-module/src/password-history.service.ts
@injectable()
export class PasswordHistoryService {
  constructor(
    @inject('PrismaClient') private prisma: PrismaClient,
  ) {}

  async isPasswordReused(userId: string, newPasswordHash: string): Promise<boolean> {
    const recentPasswords = await this.prisma.passwordHistory.findMany({
      where: { userId },
      orderBy: { createdAt: 'desc' },
      take: 3,
    });

    for (const entry of recentPasswords) {
      const match = await bcrypt.compare(newPasswordHash, entry.passwordHash);
      if (match) return true;
    }

    return false;
  }

  async recordPassword(userId: string, passwordHash: string): Promise<void> {
    await this.prisma.passwordHistory.create({
      data: { userId, passwordHash },
    });

    // Keep only last 10 entries
    const count = await this.prisma.passwordHistory.count({ where: { userId } });
    if (count > 10) {
      const oldest = await this.prisma.passwordHistory.findMany({
        where: { userId },
        orderBy: { createdAt: 'asc' },
        take: count - 10,
      });
      await this.prisma.passwordHistory.deleteMany({
        where: { id: { in: oldest.map(p => p.id) } },
      });
    }
  }
}
```

---

## Session Management

### Max 5 Concurrent Sessions

```typescript
// Already implemented in TokenService.enforceSessionLimit()
// When a 6th session is created, the oldest session is revoked.

// Session list endpoint
async getUserSessions(userId: string): Promise<SessionInfo[]> {
  return this.prisma.session.findMany({
    where: { userId },
    select: {
      id: true,
      ipAddress: true,
      userAgent: true,
      createdAt: true,
      expiresAt: true,
    },
    orderBy: { createdAt: 'desc' },
  });
}

// Revoke specific session
async revokeSession(userId: string, sessionId: string): Promise<void> {
  const session = await this.prisma.session.findUnique({ where: { id: sessionId } });
  if (!session || session.userId !== userId) {
    throw new SessionNotFoundError();
  }
  await this.prisma.session.delete({ where: { id: sessionId } });
}
```

---

## Account Lockout

### 5 Failed Attempts → 15 Minute Lockout

```typescript
// packages/auth-module/src/lockout.service.ts
@injectable()
export class AccountLockoutService {
  constructor(
    @inject('RedisClient') private redis: Redis,
  ) {}

  private MAX_ATTEMPTS = 5;
  private LOCKOUT_DURATION = 900; // 15 minutes in seconds

  async recordFailedAttempt(identifier: string): Promise<LockoutStatus> {
    const key = `lockout:${identifier}`;
    const attempts = await this.redis.incr(key);
    await this.redis.expire(key, this.LOCKOUT_DURATION);

    const remaining = this.MAX_ATTEMPTS - attempts;

    if (remaining <= 0) {
      await this.redis.setex(`locked:${key}`, this.LOCKOUT_DURATION, '1');
      return { locked: true, remainingAttempts: 0, lockoutExpiresIn: this.LOCKOUT_DURATION };
    }

    return { locked: false, remainingAttempts: remaining, lockoutExpiresIn: 0 };
  }

  async isLocked(identifier: string): Promise<boolean> {
    const locked = await this.redis.get(`locked:lockout:${identifier}`);
    return locked === '1';
  }

  async clearAttempts(identifier: string): Promise<void> {
    await this.redis.del(`lockout:${identifier}`);
  }

  async getRemainingLockoutTime(identifier: string): Promise<number> {
    const ttl = await this.redis.ttl(`locked:lockout:${identifier}`);
    return Math.max(0, ttl);
  }
}
```

---

## Login Flow

```typescript
// packages/auth-module/src/auth.service.ts
@injectable()
export class AuthService implements IAuthService {
  constructor(
    @inject('OtpService') private otpService: OtpService,
    @inject('TokenService') private tokenService: TokenService,
    @inject('PasswordService') private passwordService: PasswordService,
    @inject('LockoutService') private lockoutService: AccountLockoutService,
    @inject('PrismaClient') private prisma: PrismaClient,
  ) {}

  // PRIMARY: Phone + Password login (all users)
  async loginWithPassword(phone: string, password: string): Promise<AuthResult> {
    const isLocked = await this.lockoutService.isLocked(phone);
    if (isLocked) throw new AccountLockedError();

    const user = await this.prisma.user.findUnique({ where: { phone } });
    if (!user || !user.passwordHash) {
      throw new InvalidCredentialsError();
    }

    const valid = await this.passwordService.verifyPassword(password, user.passwordHash);
    if (!valid) {
      const status = await this.lockoutService.recordFailedAttempt(phone);
      if (status.locked) throw new AccountLockedError(status.lockoutExpiresIn);
      throw new InvalidCredentialsError();
    }

    await this.lockoutService.clearAttempts(phone);

    const tokens = await this.tokenService.generateTokens(user.id, user.role);
    return { user, ...tokens };
  }

  // Forgot password via OTP (secondary method for password recovery)
  async forgotPasswordViaOtp(phone: string): Promise<ForgotPasswordResult> {
    const user = await this.prisma.user.findUnique({ where: { phone } });
    if (!user) {
      // Don't reveal whether phone exists
      return { message: 'If the phone number exists, a verification code has been sent' };
    }

    // Send OTP via SMS/WhatsApp
    await this.otpService.requestPasswordResetOtp(phone);

    return { message: 'If the phone number exists, a verification code has been sent' };
  }

  // Reset password with OTP verification
  async resetPasswordWithOtp(phone: string, otpCode: string, newPassword: string): Promise<void> {
    // Verify OTP first
    const verification = await this.otpService.verifyPasswordResetOtp(phone, otpCode);
    if (!verification.userId) throw new InvalidOtpError();

    // Validate password strength
    const validation = this.passwordService.validatePasswordStrength(newPassword);
    if (!validation.valid) throw new WeakPasswordError(validation.errors);

    // Hash new password
    const newHash = await this.passwordService.hashPassword(newPassword);

    // Update password
    await this.prisma.user.update({
      where: { id: verification.userId },
      data: { passwordHash: newHash },
    });

    // Invalidate all existing sessions
    await this.tokenService.revokeAllSessions(verification.userId);
  }

    const newHash = await this.passwordService.hashPassword(newPassword);

    // Check password history
    const reused = await this.passwordHistoryService.isPasswordReused(payload.sub, newHash);
    if (reused) throw new PasswordReusedError();

    // Update password
    await this.prisma.user.update({
      where: { id: payload.sub },
      data: { passwordHash: newHash },
    });

    // Record in history
    await this.passwordHistoryService.recordPassword(payload.sub, newHash);

    // Clear reset token
    await this.redis.del(`reset:${payload.sub}`);

    // Revoke all sessions (force re-login)
    await this.tokenService.revokeAllSessions(payload.sub);
  }

  // Register new user
  async register(input: RegisterInput): Promise<AuthResult> {
    const existing = await this.prisma.user.findUnique({
      where: { phone: input.phone },
    });
    if (existing) throw new UserAlreadyExistsError();

    const passwordHash = await this.passwordService.hashPassword(input.password);

    const user = await this.prisma.user.create({
      data: {
        phone: input.phone,
        email: input.email,
        firstName: input.firstName,
        lastName: input.lastName,
        passwordHash,
        role: input.role || 'BUYER',
        status: 'ACTIVE',
      },
    });

    const tokens = await this.tokenService.generateTokens(user.id, user.role);
    return { user, ...tokens };
  }
}
```

---

## API Endpoints

```typescript
// packages/auth-module/src/auth.controller.ts
@injectable()
export class AuthController {
  constructor(@inject('AuthService') private authService: IAuthService) {}

  // POST /auth/request-otp
  async requestOtp(req: Request, res: Response) {
    const { phone, channel } = req.body;
    const result = await this.authService.requestOtp(phone, channel);
    res.json({ success: true, data: result });
  }

  // POST /auth/verify-otp
  async verifyOtp(req: Request, res: Response) {
    const { phone, code } = req.body;
    const result = await this.authService.loginWithOtp(phone, code);
    res.json({ success: true, data: result });
  }

  // POST /auth/login
  async login(req: Request, res: Response) {
    const { email, password } = req.body;
    const result = await this.authService.loginWithPassword(email, password);
    res.json({ success: true, data: result });
  }

  // POST /auth/register
  async register(req: Request, res: Response) {
    const result = await this.authService.register(req.body);
    res.json({ success: true, data: result });
  }

  // POST /auth/refresh
  async refresh(req: Request, res: Response) {
    const { refreshToken } = req.body;
    const tokens = await this.authService.refreshTokens(refreshToken);
    res.json({ success: true, data: tokens });
  }

  // POST /auth/logout
  @AuthGuard()
  async logout(req: Request, res: Response) {
    await this.authService.revokeSession(req.user.sessionId);
    res.json({ success: true });
  }

  // POST /auth/forgot-password
  async forgotPassword(req: Request, res: Response) {
    const result = await this.authService.forgotPassword(req.body.email);
    res.json({ success: true, data: result });
  }

  // POST /auth/reset-password
  async resetPassword(req: Request, res: Response) {
    await this.authService.resetPassword(req.body.token, req.body.password);
    res.json({ success: true });
  }

  // GET /auth/sessions
  @AuthGuard()
  async getSessions(req: Request, res: Response) {
    const sessions = await this.authService.getUserSessions(req.user.sub);
    res.json({ success: true, data: sessions });
  }

  // DELETE /auth/sessions/:id
  @AuthGuard()
  async revokeSession(req: Request, res: Response) {
    await this.authService.revokeSession(req.user.sub, req.params.id);
    res.json({ success: true });
  }
}
```

---

## Security Summary

| Feature                 | Implementation                          |
|-------------------------|-----------------------------------------|
| OTP delivery            | SMS + WhatsApp, 5-min expiry            |
| OTP storage             | bcrypt hash in Redis                    |
| OTP attempts            | 3 max per phone, lockout after          |
| Access token            | JWT, 15-min expiry                      |
| Refresh token           | JWT, 7-day expiry, rotated on use       |
| Password hashing        | bcrypt, 12 rounds                       |
| Password history        | Last 3 blocked from reuse               |
| Password strength       | 8+ chars, upper, lower, digit, special  |
| Concurrent sessions     | Max 5 per user                          |
| Account lockout         | 5 failed attempts → 15-min lockout      |
| Rate limiting           | 1 OTP per 60s per phone                 |
| Session revocation      | Single or all per user                  |
