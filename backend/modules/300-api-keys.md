# Module 300: API Keys

- **Module**: `300-api-keys`
- **Category**: Backend / Business Modules
- **Status**: Production Ready
- **Priority:** 🔥 CRITICAL - Core Platform Functionality
- **Version**: 1.1.1-CE

---

## Overview

The **API Keys module** provides **programmatic authentication** for applications and services to access the TelemetryFlow platform. It enables:

- **Dual-key system**: Separate KeyId (`tfk-*`) and KeySecret (`tfs-*`)
- **API key rotation**: Zero-downtime key rotation with grace periods
- **Scope-based access control**: Tenant and workspace level permissions
- **Argon2id hashing**: Secure secret storage
- **Revocation and expiration**: Lifecycle management

This module is critical for OTLP ingestion agents, monitoring scripts, and third-party integrations.

---

## Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [Key Generation System](#key-generation-system)
3. [Domain Model](#domain-model)
4. [Database Schema](#database-schema)
5. [API Endpoints](#api-endpoints)
6. [Authentication Flow](#authentication-flow)
7. [Key Rotation](#key-rotation)
8. [Configuration](#configuration)
9. [Security Considerations](#security-considerations)

---

## Architecture Overview

```mermaid
graph TB
    subgraph "API Key Module Architecture"
        API[API Layer<br/>Controllers]
        APP[Application Layer<br/>Commands/Queries]
        DOM[Domain Layer<br/>ApiKey Aggregate]
        INF[Infrastructure Layer<br/>Repository]
    end

    subgraph "Authentication Flow"
        REQ[HTTP Request<br/>Authorization: Bearer tfk-xxx.tfs-yyy]
        GUARD[ApiKeyGuard]
        VERIFY[Verify Service]
        HASH[Hash Comparison<br/>Argon2id]
    end

    subgraph "Persistence"
        PG[(PostgreSQL<br/>API Keys)]
        CACHE[(Redis<br/>Key Cache)]
    end

    REQ --> GUARD
    GUARD --> VERIFY
    VERIFY --> HASH
    HASH --> PG
    VERIFY --> CACHE

    API --> APP
    APP --> DOM
    DOM --> INF
    INF --> PG
```

**Key Principles:**
- **Domain-Driven Design**: ApiKey aggregate with rotation and revocation logic
- **Security First**: Never store plaintext secrets
- **Zero-downtime rotation**: Grace period for key migration
- **Audit trail**: Log all key usage and lifecycle events

---

## Key Generation System

### Dual-Key System

API authentication requires **both** a KeyId and a KeySecret:

```
Authorization: Bearer tfk-a1b2c3d4e5f6.tfs-x1y2z3a4b5c6d7e8f9
                      └─ KeyId ─┘ └─── KeySecret ────┘
```

| Component | Prefix | Length | Purpose | Storage |
|-----------|--------|--------|---------|---------|
| **KeyId** | `tfk-` | 20 chars | Public identifier | Plaintext in DB |
| **KeySecret** | `tfs-` | 40 chars | Secret credential | Hashed (Argon2id) |

### Key Generation Algorithm

```typescript
// /backend/src/modules/300-api-keys/domain/value-objects/KeyId.ts

export class KeyId extends ValueObject<string> {
  private static readonly PREFIX = 'tfk-';
  private static readonly LENGTH = 16; // hex chars after prefix

  static generate(): KeyId {
    const randomHex = randomBytes(this.LENGTH / 2).toString('hex');
    const value = `${this.PREFIX}${randomHex}`;
    return new KeyId(value);
  }

  validate(value: string): void {
    if (!value.startsWith(this.PREFIX)) {
      throw new Error('Invalid KeyId prefix');
    }
    if (value.length !== this.PREFIX.length + this.LENGTH) {
      throw new Error('Invalid KeyId length');
    }
  }
}
```

```typescript
// /backend/src/modules/300-api-keys/domain/value-objects/KeySecret.ts

export class KeySecret extends ValueObject<string> {
  private static readonly PREFIX = 'tfs-';
  private static readonly LENGTH = 32; // hex chars after prefix

  static generate(): { secret: KeySecret; hash: string } {
    const randomHex = randomBytes(this.LENGTH / 2).toString('hex');
    const value = `${this.PREFIX}${randomHex}`;

    // Hash immediately with Argon2id
    const hash = argon2.hash(value, {
      type: argon2.argon2id,
      memoryCost: 2 ** 16, // 64 MB
      timeCost: 3,
      parallelism: 1,
    });

    return {
      secret: new KeySecret(value),
      hash,
    };
  }

  async verify(hash: string): Promise<boolean> {
    return argon2.verify(hash, this.value);
  }
}
```

---

## Domain Model

### ApiKey Aggregate

```typescript
// /backend/src/modules/300-api-keys/domain/aggregates/ApiKey.ts

export type ApiKeyStatus = 'active' | 'rotated' | 'revoked' | 'expired';

export interface ApiKeyProps {
  keyId: KeyId;
  secretHash: string; // Argon2id hash
  name: string;
  scope: KeyScope;
  permissions: ApiKeyPermissions;
  tenantId: TenantId;
  workspaceId: WorkspaceId;
  createdBy: UserId;
  status: ApiKeyStatus;
  lastUsedAt: Date | null;
  usageCount: number;
  expiresAt: Date | null;
  gracePeriodExpiresAt: Date | null; // For rotation
  revokedAt: Date | null;
  revokedBy: UserId | null;
  createdAt: Date;
}

export class ApiKey extends AggregateRoot<ApiKeyId> {
  private _props: ApiKeyProps;

  // Factory method
  static create(params: {
    name: string;
    secretHash: string;
    scope: KeyScope;
    permissions: ApiKeyPermissions;
    tenantId: string;
    workspaceId: string;
    createdBy: string;
    expiresAt?: Date;
  }): { apiKey: ApiKey; keyId: KeyId } {
    const apiKeyId = ApiKeyId.generate();
    const keyId = KeyId.generate();

    const apiKey = new ApiKey(apiKeyId, {
      keyId,
      secretHash: params.secretHash,
      name: params.name,
      scope: params.scope,
      permissions: params.permissions,
      tenantId: new TenantId(params.tenantId),
      workspaceId: new WorkspaceId(params.workspaceId),
      createdBy: new UserId(params.createdBy),
      status: 'active',
      lastUsedAt: null,
      usageCount: 0,
      expiresAt: params.expiresAt ?? null,
      gracePeriodExpiresAt: null,
      revokedAt: null,
      revokedBy: null,
      createdAt: new Date(),
    });

    apiKey.addDomainEvent(new ApiKeyCreatedEvent(apiKeyId, keyId, params.name, params.tenantId));
    return { apiKey, keyId };
  }

  // Rotate key (generate new KeyId, mark old as rotated)
  rotate(newSecretHash: string, rotatedBy: string, gracePeriodMinutes: number = 30): { newKeyId: KeyId } {
    if (this._props.status !== 'active') {
      throw new Error('Can only rotate active keys');
    }

    const newKeyId = KeyId.generate();
    const gracePeriodExpiresAt = new Date(Date.now() + gracePeriodMinutes * 60 * 1000);

    // Mark current key as rotated
    this._props.status = 'rotated';
    this._props.gracePeriodExpiresAt = gracePeriodExpiresAt;

    this.addDomainEvent(
      new ApiKeyRotatedEvent(
        this.id,
        this._props.keyId,
        newKeyId,
        new UserId(rotatedBy),
        gracePeriodExpiresAt,
      )
    );

    return { newKeyId };
  }

  // Revoke key immediately
  revoke(revokedBy: string): void {
    if (this._props.status === 'revoked') {
      throw new Error('Key already revoked');
    }

    this._props.status = 'revoked';
    this._props.revokedAt = new Date();
    this._props.revokedBy = new UserId(revokedBy);

    this.addDomainEvent(new ApiKeyRevokedEvent(this.id, this._props.keyId, new UserId(revokedBy)));
  }

  // Check if key is usable
  isUsable(): boolean {
    // Active keys: check not expired
    if (this._props.status === 'active') {
      return !this.isExpired();
    }

    // Rotated keys: usable during grace period
    if (this._props.status === 'rotated' && this._props.gracePeriodExpiresAt) {
      return new Date() < this._props.gracePeriodExpiresAt && !this.isExpired();
    }

    return false;
  }

  private isExpired(): boolean {
    if (!this._props.expiresAt) return false;
    return new Date() > this._props.expiresAt;
  }

  // Record usage
  recordUsage(): void {
    this._props.lastUsedAt = new Date();
    this._props.usageCount += 1;
  }

  // Check permission
  hasPermission(permission: string): boolean {
    return this._props.permissions.has(permission);
  }

  // Scope matching
  matchesScope(tenantId: string, workspaceId?: string): boolean {
    if (this._props.scope.type === 'tenant') {
      return this._props.tenantId.equals(new TenantId(tenantId));
    }

    if (this._props.scope.type === 'workspace') {
      return this._props.workspaceId.equals(new WorkspaceId(workspaceId ?? ''));
    }

    return false;
  }

  // Getters
  get keyId(): KeyId { return this._props.keyId; }
  get secretHash(): string { return this._props.secretHash; }
  get status(): ApiKeyStatus { return this._props.status; }
  get tenantId(): TenantId { return this._props.tenantId; }
  get workspaceId(): WorkspaceId { return this._props.workspaceId; }
}
```

### Value Objects

```typescript
// /backend/src/modules/300-api-keys/domain/value-objects/KeyScope.ts

export type KeyScopeType = 'tenant' | 'workspace';

export interface KeyScopeProps {
  type: KeyScopeType;
  resourceId: string; // tenant_id or workspace_id
}

export class KeyScope extends ValueObject<KeyScopeProps> {
  static forTenant(tenantId: string): KeyScope {
    return new KeyScope({ type: 'tenant', resourceId: tenantId });
  }

  static forWorkspace(workspaceId: string): KeyScope {
    return new KeyScope({ type: 'workspace', resourceId: workspaceId });
  }

  get type(): KeyScopeType {
    return this.props.type;
  }

  get resourceId(): string {
    return this.props.resourceId;
  }
}
```

```typescript
// /backend/src/modules/300-api-keys/domain/value-objects/ApiKeyPermissions.ts

export class ApiKeyPermissions extends ValueObject<Set<string>> {
  static create(permissions: string[]): ApiKeyPermissions {
    return new ApiKeyPermissions(new Set(permissions));
  }

  has(permission: string): boolean {
    return this.value.has(permission);
  }

  toArray(): string[] {
    return Array.from(this.value);
  }

  // Common permission sets
  static readOnly(): ApiKeyPermissions {
    return new ApiKeyPermissions(new Set([
      'metrics:read:tenant',
      'logs:read:tenant',
      'traces:read:tenant',
    ]));
  }

  static fullAccess(): ApiKeyPermissions {
    return new ApiKeyPermissions(new Set([
      'metrics:read:tenant',
      'metrics:write:tenant',
      'logs:read:tenant',
      'logs:write:tenant',
      'traces:read:tenant',
      'traces:write:tenant',
      'alerts:read:tenant',
      'alerts:write:tenant',
    ]));
  }

  static ingestionOnly(): ApiKeyPermissions {
    return new ApiKeyPermissions(new Set([
      'metrics:write:tenant',
      'logs:write:tenant',
      'traces:write:tenant',
    ]));
  }
}
```

---

## Database Schema

### API Keys Table

```sql
-- /backend/src/modules/300-api-keys/infrastructure/persistence/postgres/entities/ApiKey.entity.ts

CREATE TABLE api_keys (
  api_key_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

  -- Key Identifiers
  key_id VARCHAR(50) UNIQUE NOT NULL, -- tfk-xxx (public)
  secret_hash TEXT NOT NULL,          -- Argon2id hash (never plaintext!)

  -- Metadata
  name VARCHAR(255) NOT NULL,
  description TEXT,

  -- Scope
  scope_type VARCHAR(20) NOT NULL, -- 'tenant' | 'workspace'
  tenant_id UUID REFERENCES tenants(tenant_id) ON DELETE CASCADE,
  workspace_id UUID REFERENCES workspaces(workspace_id) ON DELETE CASCADE,

  -- Permissions (array of permission strings)
  permissions TEXT[] DEFAULT '{}',

  -- Ownership
  created_by UUID REFERENCES users(user_id),

  -- Status
  status VARCHAR(20) NOT NULL DEFAULT 'active', -- 'active', 'rotated', 'revoked', 'expired'

  -- Usage Tracking
  last_used_at TIMESTAMP,
  usage_count BIGINT DEFAULT 0,

  -- Lifecycle
  expires_at TIMESTAMP,
  grace_period_expires_at TIMESTAMP, -- For rotation
  revoked_at TIMESTAMP,
  revoked_by UUID REFERENCES users(user_id),

  -- Timestamps
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),

  -- Constraints
  CHECK (scope_type IN ('tenant', 'workspace')),
  CHECK (status IN ('active', 'rotated', 'revoked', 'expired'))
);

-- Indexes
CREATE UNIQUE INDEX idx_api_keys_key_id ON api_keys(key_id);
CREATE INDEX idx_api_keys_tenant ON api_keys(tenant_id);
CREATE INDEX idx_api_keys_workspace ON api_keys(workspace_id);
CREATE INDEX idx_api_keys_status ON api_keys(status);
CREATE INDEX idx_api_keys_created_by ON api_keys(created_by);

-- Performance: Index for authentication lookups
CREATE INDEX idx_api_keys_active_lookup ON api_keys(key_id, status)
  WHERE status IN ('active', 'rotated');
```

### API Key Usage Logs Table

```sql
CREATE TABLE api_key_usage_logs (
  log_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  api_key_id UUID REFERENCES api_keys(api_key_id) ON DELETE CASCADE,

  -- Request Info
  endpoint VARCHAR(255),
  http_method VARCHAR(10),
  ip_address INET,
  user_agent TEXT,

  -- Response
  status_code INTEGER,
  response_time_ms INTEGER,

  -- Tenant Context
  tenant_id UUID,

  -- Timestamp
  created_at TIMESTAMP DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_api_key_usage_logs_api_key ON api_key_usage_logs(api_key_id);
CREATE INDEX idx_api_key_usage_logs_created_at ON api_key_usage_logs(created_at);

-- Partition by month for performance
CREATE TABLE api_key_usage_logs_2025_12 PARTITION OF api_key_usage_logs
  FOR VALUES FROM ('2025-12-01') TO ('2026-01-01');
```

---

## API Endpoints

### API Key Management

| Method | Endpoint | Description | Required Permission |
|--------|----------|-------------|---------------------|
| `POST` | `/api/v1/api-keys` | Create new API key | `api-keys:write:tenant` |
| `GET` | `/api/v1/api-keys` | List API keys (paginated) | `api-keys:read:tenant` |
| `GET` | `/api/v1/api-keys/:id` | Get API key details | `api-keys:read:tenant` |
| `PATCH` | `/api/v1/api-keys/:id` | Update API key name/description | `api-keys:write:tenant` |
| `DELETE` | `/api/v1/api-keys/:id` | Revoke API key | `api-keys:write:tenant` |
| `POST` | `/api/v1/api-keys/:id/rotate` | Rotate API key | `api-keys:write:tenant` |
| `GET` | `/api/v1/api-keys/:id/usage` | Get usage statistics | `api-keys:read:tenant` |

### Create API Key

**Request:**
```http
POST /api/v1/api-keys
Authorization: Bearer <JWT>
Content-Type: application/json

{
  "name": "Production Ingestion",
  "description": "OTLP ingestion from prod servers",
  "scope": {
    "type": "tenant",
    "tenantId": "uuid"
  },
  "permissions": [
    "metrics:write:tenant",
    "logs:write:tenant",
    "traces:write:tenant"
  ],
  "expiresAt": "2026-12-31T23:59:59Z"
}
```

**Response:**
```json
{
  "apiKeyId": "uuid",
  "keyId": "tfk-a1b2c3d4e5f6",
  "keySecret": "tfs-x1y2z3a4b5c6d7e8f9",
  "name": "Production Ingestion",
  "scope": {
    "type": "tenant",
    "tenantId": "uuid"
  },
  "permissions": [
    "metrics:write:tenant",
    "logs:write:tenant",
    "traces:write:tenant"
  ],
  "status": "active",
  "expiresAt": "2026-12-31T23:59:59Z",
  "createdAt": "2025-12-12T10:00:00Z"
}
```

**⚠️ IMPORTANT**: The `keySecret` is **only returned once** during creation. Store it securely.

### Rotate API Key

**Request:**
```http
POST /api/v1/api-keys/:id/rotate
Authorization: Bearer <JWT>
Content-Type: application/json

{
  "gracePeriodMinutes": 30
}
```

**Response:**
```json
{
  "apiKeyId": "uuid",
  "oldKeyId": "tfk-a1b2c3d4e5f6",
  "newKeyId": "tfk-g7h8i9j0k1l2",
  "newKeySecret": "tfs-m3n4o5p6q7r8s9t0u1",
  "gracePeriodExpiresAt": "2025-12-12T10:30:00Z",
  "message": "Old key will work until grace period expires"
}
```

---

## Authentication Flow

### API Key Authentication

```mermaid
sequenceDiagram
    participant C as Client
    participant G as ApiKeyGuard
    participant V as ApiKeyVerifyService
    participant DB as PostgreSQL
    participant CACHE as Redis Cache
    participant H as Handler

    C->>G: Request + Authorization: Bearer tfk-xxx.tfs-yyy
    G->>G: Extract KeyId + KeySecret

    G->>V: Verify(keyId, keySecret)

    V->>CACHE: GET api_key:{keyId}

    alt Cache Hit
        CACHE-->>V: Cached ApiKey
    else Cache Miss
        V->>DB: SELECT api_key WHERE key_id = ?
        DB-->>V: ApiKey Entity
        V->>CACHE: SET api_key:{keyId} (TTL: 15min)
    end

    V->>V: Check if key.isUsable()

    alt Key Revoked/Expired
        V-->>G: AuthenticationException
        G-->>C: 401 Unauthorized
    end

    V->>V: Verify Secret (Argon2id)

    alt Secret Mismatch
        V-->>G: AuthenticationException
        G-->>C: 401 Unauthorized
    end

    V->>V: Check Permissions

    alt Missing Permission
        V-->>G: ForbiddenException
        G-->>C: 403 Forbidden
    end

    V->>DB: UPDATE usage_count, last_used_at
    V->>DB: INSERT INTO api_key_usage_logs

    V-->>G: Verified + Tenant Context
    G->>H: Forward Request with Context
    H-->>C: Response
```

### Usage in OTLP Ingestion

```bash
# OpenTelemetry Collector configuration
exporters:
  otlphttp:
    endpoint: https://telemetryflow.id/v1/otlp
    headers:
      Authorization: Bearer tfk-a1b2c3d4e5f6.tfs-x1y2z3a4b5c6d7e8f9
```

```bash
# cURL example
curl -X POST https://telemetryflow.id/v1/otlp/metrics \
  -H "Authorization: Bearer tfk-a1b2c3d4e5f6.tfs-x1y2z3a4b5c6d7e8f9" \
  -H "Content-Type: application/json" \
  -d '{...}'
```

---

## Key Rotation

### Rotation Flow

```mermaid
graph TD
    A[Active Key<br/>tfk-old] -->|Rotate| B[Rotated Key<br/>tfk-old<br/>Grace Period: 30min]
    B -->|Grace Period Active| C[New Key Created<br/>tfk-new<br/>Status: Active]
    B -->|Grace Period Expires| D[Expired<br/>tfk-old<br/>No longer usable]

    C -->|Continue Using| E[Production Traffic]

    style A fill:#4caf50
    style B fill:#ff9800
    style C fill:#4caf50
    style D fill:#f44336
```

### Implementation

```typescript
// /backend/src/modules/300-api-keys/application/commands/RotateApiKeyCommand.ts

@CommandHandler(RotateApiKeyCommand)
export class RotateApiKeyCommandHandler {
  async execute(command: RotateApiKeyCommand): Promise<RotateApiKeyResult> {
    const apiKey = await this.apiKeyRepository.findById(command.apiKeyId);

    if (!apiKey) {
      throw new NotFoundException('API key not found');
    }

    // Generate new secret
    const { secret: newSecret, hash: newSecretHash } = KeySecret.generate();

    // Rotate (returns new KeyId)
    const { newKeyId } = apiKey.rotate(
      newSecretHash,
      command.userId,
      command.gracePeriodMinutes ?? 30,
    );

    // Save updated old key (now "rotated")
    await this.apiKeyRepository.save(apiKey);

    // Create new API key with same permissions
    const { apiKey: newApiKey } = ApiKey.create({
      name: `${apiKey.name} (Rotated)`,
      secretHash: newSecretHash,
      scope: apiKey.scope,
      permissions: apiKey.permissions,
      tenantId: apiKey.tenantId.value,
      workspaceId: apiKey.workspaceId.value,
      createdBy: command.userId,
      expiresAt: apiKey.expiresAt,
    });

    await this.apiKeyRepository.save(newApiKey);

    // Publish event
    await this.eventBus.publish(
      new ApiKeyRotatedEvent(apiKey.id, apiKey.keyId, newKeyId, command.userId)
    );

    return {
      oldKeyId: apiKey.keyId,
      newKeyId,
      newKeySecret: newSecret, // Only returned here!
      gracePeriodExpiresAt: apiKey.gracePeriodExpiresAt,
    };
  }
}
```

### Best Practices

1. **Plan rotation ahead**: Notify users 7 days before rotation
2. **Use grace periods**: Default 30 minutes allows gradual migration
3. **Monitor usage**: Track which keys are still using old credentials
4. **Automate rotation**: Rotate keys every 90 days
5. **Emergency revocation**: Immediate revoke if compromised (no grace period)

---

## Configuration

### Environment Variables

```bash
# API Key Settings
API_KEY_DEFAULT_EXPIRATION_DAYS=365    # 1 year
API_KEY_MAX_EXPIRATION_DAYS=730        # 2 years max
API_KEY_ROTATION_GRACE_PERIOD=30       # Grace period (minutes)
API_KEY_CACHE_TTL=900                  # Cache TTL (15 minutes)

# Argon2id Hashing
ARGON2_MEMORY_COST=65536               # 64 MB
ARGON2_TIME_COST=3                     # 3 iterations
ARGON2_PARALLELISM=1                   # Single thread

# Usage Logging
API_KEY_LOG_USAGE=true                 # Enable usage logging
API_KEY_USAGE_LOG_RETENTION_DAYS=90    # Keep logs for 90 days

# Rate Limiting (per API key)
API_KEY_RATE_LIMIT_PER_MINUTE=1000     # 1000 requests/minute
API_KEY_RATE_LIMIT_PER_HOUR=50000      # 50K requests/hour
```

---

## Security Considerations

### 1. Never Store Plaintext Secrets

```typescript
// ❌ BAD: Storing plaintext
await db.insert({ key_id: 'tfk-xxx', secret: 'tfs-yyy' });

// ✅ GOOD: Hash with Argon2id
const secretHash = await argon2.hash('tfs-yyy', {
  type: argon2.argon2id,
  memoryCost: 2 ** 16,
  timeCost: 3,
});
await db.insert({ key_id: 'tfk-xxx', secret_hash: secretHash });
```

### 2. Only Return Secret Once

The `keySecret` must **only** be returned during creation or rotation. After that, it cannot be retrieved.

### 3. Use HTTPS Only

API keys transmitted over HTTP are vulnerable to interception. **Always use HTTPS**.

### 4. Implement Rate Limiting

```typescript
// Per-key rate limiting
@UseGuards(ApiKeyGuard, ApiKeyRateLimitGuard)
@Post('metrics')
async ingestMetrics(@ApiKeyContext() apiKey: ApiKey) {
  // Rate limit: 1000 req/min per key
}
```

### 5. Audit All Key Operations

```typescript
// Log key creation, rotation, revocation
await this.auditLogService.log({
  action: 'api_key.created',
  resource: 'api_key',
  resourceId: apiKey.id,
  userId: command.userId,
  tenantId: command.tenantId,
  metadata: {
    keyId: apiKey.keyId,
    scope: apiKey.scope,
  },
});
```

### 6. Automatic Expiration

```sql
-- Cron job to mark expired keys
UPDATE api_keys
SET status = 'expired'
WHERE status = 'active'
  AND expires_at < NOW();

-- Mark rotated keys past grace period as expired
UPDATE api_keys
SET status = 'expired'
WHERE status = 'rotated'
  AND grace_period_expires_at < NOW();
```

---

## Best Practices

### 1. Use Descriptive Names

```typescript
// ❌ BAD
name: "Key 1"

// ✅ GOOD
name: "Production OTLP Ingestion - us-west-2"
```

### 2. Minimal Permissions

```typescript
// ❌ BAD: Full access for ingestion
permissions: ApiKeyPermissions.fullAccess()

// ✅ GOOD: Ingestion-only
permissions: ApiKeyPermissions.ingestionOnly()
```

### 3. Scope to Tenant

```typescript
// ❌ BAD: Workspace-wide access
scope: KeyScope.forWorkspace(workspaceId)

// ✅ GOOD: Tenant-specific
scope: KeyScope.forTenant(tenantId)
```

### 4. Set Expiration

```typescript
// ❌ BAD: No expiration
expiresAt: null

// ✅ GOOD: 1 year expiration
expiresAt: new Date(Date.now() + 365 * 24 * 60 * 60 * 1000)
```

### 5. Rotate Regularly

- **Automated rotation**: Every 90 days
- **Manual rotation**: On employee departure, suspected compromise
- **Emergency revocation**: Immediately if leaked

---

## Related Documentation

- [Module 200: Auth](./200-auth.md) - JWT authentication
- [Module 400: Telemetry](./400-telemetry.md) - OTLP ingestion using API keys
- [Module 800: Audit](./800-audit.md) - Audit logging for key operations
- [Architecture: Security](../../04-SECURITY.md) - Security implementation details

---

- **Last Updated**: December 12, 2025
- **Maintained By**: DevOpsCorner Indonesia
