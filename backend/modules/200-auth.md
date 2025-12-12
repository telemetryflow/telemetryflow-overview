# Module: 200-auth (Authentication & Authorization)

- **Module**: `200-auth`
- **Category**: Backend / Business Modules
- **Status**: Production Ready
- **Priority:** 🔥 CRITICAL - Security Foundation
- **Version**: 3.10.0

---

## Module Overview

```mermaid
graph TB
    subgraph "200-auth Module"
        A[JWT Authentication<br/>Access + Refresh Tokens]
        B[Multi-Factor Auth<br/>TOTP-based MFA]
        C[Session Management<br/>Redis-backed Sessions]
        D[Password Security<br/>Argon2id Hashing]
    end

    subgraph "Key Features"
        E[✅ JWT with 15min Expiry]
        F[✅ TOTP MFA Support]
        G[✅ Account Lockout Protection]
        H[✅ Token Blacklisting]
    end

    A --> E
    B --> F
    C --> G
    D --> H

    style A fill:#ff6b6b
    style B fill:#4ecdc4
    style C fill:#45b7d1
    style D fill:#f9ca24
```

**Purpose:** Comprehensive authentication and authorization system with JWT, MFA, session management, and security controls.

**Location:** `backend/src/modules/200-auth/`

---

## Architecture

### Module Structure

```mermaid
graph TB
    subgraph "presentation/ - REST API"
        C1[AuthController<br/>/api/v2/auth/*]
        C2[SettingsController<br/>/api/v2/auth/settings/*]
        G1[JwtAuthGuard]
        G2[SessionGuard]
    end

    subgraph "application/ - CQRS"
        CMD1[LoginCommand]
        CMD2[RegisterCommand]
        CMD3[ValidateMfaCommand]
        QRY1[GetAuthStatusQuery]
        H1[Command/Query Handlers]
    end

    subgraph "domain/ - Business Logic"
        AGG1[Auth Aggregate]
        AGG2[AuthSession Aggregate]
        AGG3[MFASecret Aggregate]
        SVC1[Password Policy Service]
        SVC2[Account Lockout Service]
        SVC3[Token Blacklist Service]
    end

    subgraph "infrastructure/ - Storage"
        REPO1[AuthRepository<br/>PostgreSQL]
        REPO2[SessionRepository<br/>Redis]
        MAP1[Auth Mappers]
    end

    C1 --> CMD1
    CMD1 --> AGG1
    AGG1 --> SVC1
    AGG1 --> SVC2
    AGG1 --> REPO1

    G1 --> SVC3

    style C1 fill:#ff6b6b
    style CMD1 fill:#f9ca24
    style AGG1 fill:#4ecdc4
    style REPO1 fill:#6c5ce7
```

---

## Authentication Flows

### Complete Login Flow

```mermaid
sequenceDiagram
    participant Client as Client App
    participant API as AuthController
    participant Handler as LoginHandler
    participant Policy as PasswordPolicyService
    participant Lockout as AccountLockoutService
    participant User as UserRepository
    participant JWT as JWT Service
    participant Redis as Redis (Session)
    participant Audit as AuditLog

    Client->>API: POST /auth/login<br/>{email, password}
    API->>Handler: Execute LoginCommand

    Handler->>Lockout: Check if account is locked
    alt Account Locked
        Lockout-->>API: 401 Account Locked
        API-->>Client: Error: Too many attempts
    end

    Handler->>User: Find user by email
    alt User Not Found
        Handler->>Policy: Simulate password check<br/>(timing attack prevention)
        Handler-->>API: 401 Invalid Credentials
    end

    Handler->>Policy: Verify password (Argon2id)
    alt Password Invalid
        Handler->>Lockout: Increment failed attempts<br/>Lock after 5 failures
        Handler->>Audit: Log failed login attempt
        Handler-->>API: 401 Invalid Credentials
    else Password Valid
        Handler->>Lockout: Reset failed attempts

        alt MFA Enabled
            Handler->>JWT: Generate temp token (5min)
            Handler-->>Client: 200 {tempToken, mfaRequired: true}

            Client->>API: POST /auth/mfa/validate<br/>{tempToken, code}
            API->>Handler: Execute ValidateMfaCommand
            Handler->>Handler: Validate TOTP code

            alt Invalid MFA Code
                Handler-->>Client: 401 Invalid MFA Code
            end
        end

        Handler->>JWT: Generate access token (15min)<br/>Generate refresh token (7d)
        Handler->>Redis: Store session<br/>session:{userId}
        Handler->>Audit: Log successful login
        Handler-->>Client: 200 {accessToken, refreshToken, expiresIn}
    end
```

### Registration Flow

```mermaid
sequenceDiagram
    participant Client as Client App
    participant API as AuthController
    participant Handler as RegisterHandler
    participant Policy as PasswordPolicyService
    participant User as UserRepository
    participant Email as EmailService
    participant Audit as AuditLog

    Client->>API: POST /auth/register<br/>{email, password, firstName, lastName}
    API->>Handler: Execute RegisterCommand

    Handler->>Policy: Validate password strength<br/>- Min 12 characters<br/>- Uppercase/lowercase<br/>- Numbers & symbols

    alt Password Too Weak
        Handler-->>Client: 400 Password does not meet requirements
    end

    Handler->>User: Check if email exists
    alt Email Already Exists
        Handler-->>Client: 409 Email already registered
    end

    Handler->>Policy: Hash password (Argon2id)
    Handler->>User: Create user entity<br/>email_verified: false
    Handler->>User: Generate verification token<br/>expires in 24 hours

    Handler->>Email: Send verification email<br/>with token link
    Handler->>Audit: Log registration event

    Handler-->>Client: 201 {message: "Check email for verification"}

    Note over Client,Audit: User clicks email link

    Client->>API: POST /auth/verify-email<br/>{token}
    API->>Handler: Execute VerifyEmailCommand
    Handler->>User: Validate token & mark verified
    Handler-->>Client: 200 {success: true}
```

### Token Refresh Flow

```mermaid
sequenceDiagram
    participant Client as Client App
    participant API as AuthController
    participant Handler as RefreshTokenHandler
    participant JWT as JWT Service
    participant Blacklist as TokenBlacklistService
    participant Redis as Redis (Session)

    Client->>API: POST /auth/refresh<br/>{refreshToken}
    API->>Handler: Execute RefreshTokenCommand

    Handler->>JWT: Verify refresh token signature
    alt Invalid Signature
        Handler-->>Client: 401 Invalid Token
    end

    Handler->>Blacklist: Check if token is blacklisted
    alt Token Blacklisted
        Handler-->>Client: 401 Token Revoked
    end

    Handler->>JWT: Extract userId from token
    Handler->>Redis: Validate session exists

    alt Session Expired
        Handler-->>Client: 401 Session Expired
    end

    Handler->>JWT: Generate new access token (15min)
    Handler->>JWT: Generate new refresh token (7d)
    Handler->>Blacklist: Blacklist old refresh token
    Handler->>Redis: Update session timestamp

    Handler-->>Client: 200 {accessToken, refreshToken, expiresIn: 900}
```

---

## Domain Model

### Auth Aggregate

```typescript
// domain/aggregates/Auth.aggregate.ts
export class Auth extends AggregateRoot {
  private readonly _id: AuthId;
  private readonly _userId: UserId;
  private _credentials: Credentials;  // Value Object with password hash
  private _mfaEnabled: boolean;
  private _mfaSecret?: MFASecret;
  private _emailVerified: boolean;
  private _verificationToken?: string;
  private _verificationTokenExpiry?: Date;

  static create(
    userId: UserId,
    email: string,
    password: string,
    passwordPolicyService: PasswordPolicyService
  ): Auth {
    // Business Rule: Password must meet strength requirements
    const validationResult = passwordPolicyService.validatePassword(password);
    if (!validationResult.isValid) {
      throw new DomainError(validationResult.errors.join(', '));
    }

    // Business Rule: Email must be unique (checked in handler)
    const hashedPassword = await passwordPolicyService.hashPassword(password);
    const credentials = Credentials.create(email, hashedPassword);

    const auth = new Auth({
      id: AuthId.create(),
      userId,
      credentials,
      mfaEnabled: false,
      emailVerified: false,
    });

    auth.apply(new AuthCreated(auth));
    return auth;
  }

  validatePassword(
    plainPassword: string,
    passwordPolicyService: PasswordPolicyService
  ): boolean {
    return passwordPolicyService.verifyPassword(
      plainPassword,
      this._credentials.passwordHash
    );
  }

  enableMfa(secret: MFASecret): void {
    this._mfaEnabled = true;
    this._mfaSecret = secret;
    this.apply(new MfaEnabled(this._userId));
  }

  verifyEmail(token: string): void {
    // Business Rule: Token must match and not be expired
    if (this._verificationToken !== token) {
      throw new DomainError('Invalid verification token');
    }
    if (!this._verificationTokenExpiry || new Date() > this._verificationTokenExpiry) {
      throw new DomainError('Verification token expired');
    }

    this._emailVerified = true;
    this._verificationToken = undefined;
    this._verificationTokenExpiry = undefined;
  }
}
```

### Security Services

```mermaid
classDiagram
    class PasswordPolicyService {
        +validatePassword(password) ValidationResult
        +hashPassword(password) Promise~string~
        +verifyPassword(plain, hash) Promise~boolean~
    }

    class AccountLockoutService {
        -maxAttempts: 5
        -lockoutDuration: 900s
        +recordFailedAttempt(userId) void
        +resetFailedAttempts(userId) void
        +isLocked(userId) Promise~boolean~
    }

    class TokenBlacklistService {
        +blacklistToken(token, expiresIn) Promise~void~
        +isBlacklisted(token) Promise~boolean~
        +cleanup() Promise~void~
    }

    class MFAService {
        +generateSecret(userId) MFASecret
        +validateTOTP(secret, code) boolean
        +generateQRCode(secret) string
    }

    style PasswordPolicyService fill:#4ecdc4
    style AccountLockoutService fill:#ff6b6b
    style TokenBlacklistService fill:#f9ca24
```

---

## Security Features

### Password Policy

```mermaid
flowchart TD
    A[Password Input] --> B{Length >= 12?}
    B -->|No| Z[Reject ❌]
    B -->|Yes| C{Has Uppercase?}
    C -->|No| Z
    C -->|Yes| D{Has Lowercase?}
    D -->|No| Z
    D -->|Yes| E{Has Number?}
    E -->|No| Z
    E -->|Yes| F{Has Symbol?}
    F -->|No| Z
    F -->|Yes| G{Not Common Password?}
    G -->|No| Z
    G -->|Yes| H[Hash with Argon2id ✅]
    H --> I[Store Hash]

    style I fill:#27ae60
    style Z fill:#e74c3c
```

**Password Requirements:**
- ✅ Minimum 12 characters
- ✅ At least 1 uppercase letter
- ✅ At least 1 lowercase letter
- ✅ At least 1 number
- ✅ At least 1 special character
- ✅ Not in common password list
- ✅ Hashed using Argon2id (OWASP recommended)

**Argon2id Configuration:**
```typescript
{
  type: argon2.argon2id,
  memoryCost: 65536,  // 64 MB
  timeCost: 3,        // 3 iterations
  parallelism: 4      // 4 threads
}
```

### Account Lockout

```mermaid
stateDiagram-v2
    [*] --> Active: Initial State
    Active --> Attempt1: Failed Login (1/5)
    Attempt1 --> Attempt2: Failed Login (2/5)
    Attempt2 --> Attempt3: Failed Login (3/5)
    Attempt3 --> Attempt4: Failed Login (4/5)
    Attempt4 --> Locked: Failed Login (5/5)

    Locked --> Active: 15 minutes elapsed
    Locked --> Active: Admin unlock

    Attempt1 --> Active: Successful Login
    Attempt2 --> Active: Successful Login
    Attempt3 --> Active: Successful Login
    Attempt4 --> Active: Successful Login

    Active: Failed Attempts: 0
    Locked: Account Locked<br/>Duration: 15 min
```

**Lockout Rules:**
- 🔒 Lock after 5 failed attempts
- ⏱️ Lockout duration: 15 minutes (900 seconds)
- ✅ Reset counter on successful login
- 🔓 Admin can manually unlock

### Token Security

| Token Type | Lifetime | Storage | Revocation |
|------------|----------|---------|------------|
| **Access Token** | 15 minutes | Client (memory) | Blacklist on logout |
| **Refresh Token** | 7 days | Client (secure cookie) | Blacklist on refresh |
| **Verification Token** | 24 hours | Database | Single-use |
| **Reset Token** | 1 hour | Database | Single-use |
| **MFA Temp Token** | 5 minutes | Client | Blacklist after MFA |

---

## Database Schema

### Auth Table (PostgreSQL)

```sql
CREATE TABLE auth (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL UNIQUE,
  email VARCHAR(255) NOT NULL UNIQUE,
  password_hash VARCHAR(255) NOT NULL,  -- Argon2id hash

  -- Email Verification
  email_verified BOOLEAN NOT NULL DEFAULT false,
  verification_token VARCHAR(255),
  verification_token_expiry TIMESTAMP,

  -- Password Reset
  reset_token VARCHAR(255),
  reset_token_expiry TIMESTAMP,

  -- MFA
  mfa_enabled BOOLEAN NOT NULL DEFAULT false,
  mfa_secret VARCHAR(255),  -- Encrypted TOTP secret

  -- Security
  failed_login_attempts INT NOT NULL DEFAULT 0,
  locked_until TIMESTAMP,
  last_login_at TIMESTAMP,
  last_login_ip VARCHAR(45),

  -- Password Management
  password_changed_at TIMESTAMP,
  password_expires_at TIMESTAMP,
  must_change_password BOOLEAN DEFAULT false,

  -- Timestamps
  created_at TIMESTAMP NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP NOT NULL DEFAULT NOW(),

  CONSTRAINT fk_user FOREIGN KEY (user_id)
    REFERENCES users(id) ON DELETE CASCADE
);

-- Indexes
CREATE UNIQUE INDEX idx_auth_email ON auth(email);
CREATE UNIQUE INDEX idx_auth_user_id ON auth(user_id);
CREATE INDEX idx_auth_verification_token ON auth(verification_token);
CREATE INDEX idx_auth_reset_token ON auth(reset_token);
```

### Session Table (Redis)

```
Key Pattern: session:{userId}:{sessionId}

Value (JSON):
{
  "userId": "uuid",
  "sessionId": "uuid",
  "accessToken": "jwt-token",
  "refreshToken": "jwt-token",
  "ipAddress": "192.168.1.1",
  "userAgent": "Mozilla/5.0...",
  "createdAt": "2025-01-15T10:00:00Z",
  "lastAccessedAt": "2025-01-15T10:30:00Z",
  "expiresAt": "2025-01-22T10:00:00Z"
}

TTL: 7 days (604800 seconds)
```

---

## API Endpoints

### Endpoint Summary

| Endpoint | Method | Auth | Description |
|----------|--------|------|-------------|
| `/auth/login` | POST | Public | Email/password login |
| `/auth/register` | POST | Public | Create account |
| `/auth/verify-email` | POST | Public | Verify email with token |
| `/auth/forgot-password` | POST | Public | Request password reset |
| `/auth/reset-password` | POST | Public | Reset password with token |
| `/auth/mfa/validate` | POST | Public | Complete MFA challenge |
| `/auth/refresh` | POST | Public | Refresh access token |
| `/auth/logout` | POST | JWT | Invalidate session |
| `/auth/me` | GET | JWT | Get current user |
| `/auth/change-password` | POST | JWT | Change password |
| `/auth/password/status` | GET | JWT | Check password expiry |
| `/auth/settings/mfa/enable` | POST | JWT | Enable MFA |
| `/auth/settings/mfa/disable` | POST | JWT | Disable MFA |

---

## Configuration

### Environment Variables

```bash
# JWT Configuration
JWT_SECRET=your-secret-key-min-32-chars
JWT_ACCESS_TOKEN_EXPIRY=15m
JWT_REFRESH_TOKEN_EXPIRY=7d

# Password Policy
PASSWORD_MIN_LENGTH=12
PASSWORD_REQUIRE_UPPERCASE=true
PASSWORD_REQUIRE_LOWERCASE=true
PASSWORD_REQUIRE_NUMBER=true
PASSWORD_REQUIRE_SYMBOL=true
PASSWORD_EXPIRY_DAYS=90

# Account Lockout
ACCOUNT_LOCKOUT_ATTEMPTS=5
ACCOUNT_LOCKOUT_DURATION=900  # seconds

# MFA
MFA_ISSUER=TelemetryFlow
MFA_WINDOW=1  # Allow 1 step before/after current time
```

---

## API Examples

### Login

```bash
curl -X POST http://localhost:3000/api/v2/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "user@example.com",
    "password": "SecurePassword123!"
  }'

# Response:
{
  "accessToken": "eyJhbGc...",
  "refreshToken": "eyJhbGc...",
  "expiresIn": 900,
  "mfaRequired": false
}
```

### Register

```bash
curl -X POST http://localhost:3000/api/v2/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "email": "newuser@example.com",
    "password": "SecurePassword123!",
    "first_name": "John",
    "last_name": "Doe"
  }'

# Response:
{
  "message": "Registration successful. Please check your email to verify your account."
}
```

---

## Related Modules

```mermaid
graph TB
    A[200-auth<br/>Authentication]

    A -->|Creates| B[100-core<br/>Users & IAM]
    A -->|Uses| C[shared/cache<br/>Session Cache]
    A -->|Uses| D[shared/email<br/>Verification Emails]
    A -->|Logs to| E[800-audit<br/>Security Events]
    A -->|Integrates| F[700-sso<br/>SSO Providers]

    style A fill:#ff6b6b
    style B fill:#4ecdc4
    style E fill:#f9ca24
```

---

## Testing

### Unit Tests
- `Auth.aggregate.spec.ts` - Domain logic
- `PasswordPolicyService.spec.ts` - Password validation
- `AccountLockoutService.spec.ts` - Lockout logic

### Integration Tests
- `auth-flow.spec.ts` - Complete login flow
- `mfa-flow.spec.ts` - MFA validation

### E2E Tests
- `authentication.e2e.spec.ts` - Full auth lifecycle

---

- **File Location:** `./backend/modules/200-auth.md`
- **Maintained By:** DevOpsCorner Indonesia
