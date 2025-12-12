# Module: 100-core (IAM & Multi-Tenancy Foundation)

- **Module**: `100-core`
- **Category**: Backend / Business Modules
- **Status**: Production Ready
- **Priority:** 🔥 CRITICAL - Platform Foundation
- **Version**: 3.10.0

---

## Module Overview

```mermaid
graph TB
    subgraph "100-core Module"
        A[IAM<br/>Identity & Access]
        B[Multi-Tenancy<br/>Isolation]
        C[RBAC<br/>Role-Based Access]
        D[Hierarchical Model<br/>Region→Org→Workspace→Tenant]
    end

    subgraph "Core Entities"
        E[User]
        F[Organization]
        G[Workspace]
        H[Tenant]
        I[Role]
        J[Permission]
    end

    A --> E
    B --> F
    B --> G
    B --> H
    C --> I
    C --> J

    style A fill:#4ecdc4
    style B fill:#ff6b6b
    style C fill:#f9ca24
    style D fill:#6c5ce7
```

**Purpose:** Foundation module providing Identity & Access Management (IAM), multi-tenancy hierarchy, and role-based access control (RBAC) for the entire platform.

**Location:** `backend/src/modules/100-core/`

---

## Multi-Tenancy Hierarchy

### 4-Level Tenant Model

```mermaid
graph TD
    A[Region<br/>Geographic Isolation] -->|Contains| B[Organization<br/>Company/Entity]
    B -->|Contains| C[Workspace<br/>Project/Team]
    C -->|Contains| D[Tenant<br/>Environment]
    D -->|Assigned to| E[Users & Data]

    A1[Examples:<br/>us-east, eu-west, ap-south] -.-> A
    B1[Examples:<br/>Acme Corp, XYZ Inc] -.-> B
    C1[Examples:<br/>Production Team, Dev Team] -.-> C
    D1[Examples:<br/>prod, staging, dev] -.-> D

    style A fill:#4ecdc4
    style B fill:#45b7d1
    style C fill:#f9ca24
    style D fill:#ff6b6b
```

**Hierarchy Levels:**

| Level | Purpose | Scope | Example |
|-------|---------|-------|---------|
| **Region** | Geographic/regulatory isolation | Infrastructure | `us-east-1`, `eu-central-1` |
| **Organization** | Company/legal entity | Billing, SSO config | `acme-corp`, `devopscorner` |
| **Workspace** | Project/team grouping | Collaboration | `production-ops`, `dev-team` |
| **Tenant** | Environment isolation | Data segregation | `prod`, `staging`, `dev` |

---

## Core Entities

### Entity Relationship Diagram

```mermaid
erDiagram
    REGION ||--o{ ORGANIZATION : contains
    ORGANIZATION ||--o{ WORKSPACE : contains
    ORGANIZATION ||--o{ USER : employs
    WORKSPACE ||--o{ TENANT : contains
    WORKSPACE ||--o{ USER : "has members"
    TENANT ||--o{ USER : "assigned to"

    USER ||--o{ USER_ROLE : has
    ROLE ||--o{ USER_ROLE : assigned
    ROLE ||--o{ ROLE_PERMISSION : has
    PERMISSION ||--o{ ROLE_PERMISSION : granted

    USER ||--o{ USER_GROUP : "member of"
    GROUP ||--o{ USER_GROUP : contains
    GROUP ||--o{ GROUP_ROLE : has
    ROLE ||--o{ GROUP_ROLE : assigned

    REGION {
        uuid id PK
        string name
        string code
        boolean is_active
        timestamp created_at
    }

    ORGANIZATION {
        uuid id PK
        uuid region_id FK
        string name
        string slug
        timestamp created_at
    }

    WORKSPACE {
        uuid id PK
        uuid organization_id FK
        string name
        string slug
        timestamp created_at
    }

    TENANT {
        uuid id PK
        uuid workspace_id FK
        string name
        string environment
        timestamp created_at
    }

    USER {
        uuid id PK
        uuid organization_id FK
        string email UK
        string password_hash
        string first_name
        string last_name
        boolean is_active
        timestamp last_login
    }

    ROLE {
        uuid id PK
        string name UK
        string description
        int level
    }

    PERMISSION {
        uuid id PK
        string resource
        string action
        string scope
    }
```

---

## IAM Implementation

### User Management

```typescript
// domain/aggregates/User.aggregate.ts
export class User extends AggregateRoot {
  private readonly _id: UserId;
  private readonly _email: Email;        // Value Object with validation
  private _passwordHash: PasswordHash;
  private _isActive: boolean;
  private readonly _roles: Role[];
  private readonly _permissions: Permission[];

  static create(props: CreateUserProps): User {
    // Business Rule: Email must be unique
    const email = Email.create(props.email);

    // Business Rule: Password must meet security requirements
    const passwordHash = PasswordHash.fromPlaintext(
      props.password,
      { minLength: 12, requireSpecialChar: true }
    );

    const user = new User({ email, passwordHash, ...props });
    user.apply(new UserCreated(user));
    return user;
  }

  assignRole(role: Role): void {
    // Business Rule: Cannot assign duplicate roles
    if (this._roles.some(r => r.id.equals(role.id))) {
      throw new DomainError('User already has this role');
    }
    this._roles.push(role);
    this.apply(new RoleAssignedToUser(this._id, role.id));
  }

  hasPermission(permission: Permission): boolean {
    // Check direct permissions
    if (this._permissions.some(p => p.equals(permission))) {
      return true;
    }
    // Check role-based permissions
    return this._roles.some(role => role.hasPermission(permission));
  }
}
```

### Role-Based Access Control (RBAC)

```mermaid
graph TB
    subgraph "5 Default Roles"
        R1[Super Admin<br/>Level 1<br/>All Permissions]
        R2[Administrator<br/>Level 2<br/>Org Management]
        R3[Developer<br/>Level 3<br/>Read/Write Data]
        R4[Viewer<br/>Level 4<br/>Read-Only]
        R5[Demo<br/>Level 5<br/>Limited Read]
    end

    subgraph "Permission Structure"
        P1[Resource:Action:Scope<br/>metrics:read:tenant]
        P2[metrics:write:workspace]
        P3[users:manage:organization]
        P4[dashboards:create:workspace]
    end

    R1 --> P1
    R1 --> P2
    R1 --> P3
    R1 --> P4
    R2 --> P2
    R2 --> P3
    R2 --> P4
    R3 --> P1
    R3 --> P2
    R3 --> P4
    R4 --> P1
    R5 --> P1

    style R1 fill:#e74c3c
    style R2 fill:#f39c12
    style R3 fill:#27ae60
    style R4 fill:#4ecdc4
```

**Role Hierarchy:**

| Role | Level | Permissions | Use Case |
|------|-------|-------------|----------|
| **Super Admin** | 1 | All (`*:*:*`) | Platform administrators |
| **Administrator** | 2 | Organization management | Org owners, billing admins |
| **Developer** | 3 | Data read/write, dashboard create | Engineers, DevOps |
| **Viewer** | 4 | Read-only access | Stakeholders, managers |
| **Demo** | 5 | Limited read (sample data only) | Trial users, demos |

---

## Permission System

### Permission Format

```
resource:action:scope

Examples:
- metrics:read:tenant      → Read metrics in current tenant
- logs:write:workspace     → Write logs in current workspace
- users:manage:organization → Manage users in organization
- dashboards:delete:tenant  → Delete dashboards in tenant
- alerts:create:workspace   → Create alerts in workspace
```

### Permission Check Flow

```mermaid
sequenceDiagram
    participant User as User Request
    participant Guard as PermissionsGuard
    participant Cache as Redis Cache
    participant DB as PostgreSQL
    participant Decision as Authorization Decision

    User->>Guard: HTTP Request with JWT
    Guard->>Guard: Extract User ID from JWT
    Guard->>Cache: Check Permission Cache<br/>Key: perms:userId

    alt Cache Hit
        Cache-->>Guard: Return Cached Permissions ⚡<br/>~1ms
    else Cache Miss
        Guard->>DB: Query User Roles & Permissions
        DB-->>Guard: Return Roles + Permissions
        Guard->>Cache: Store in Cache (5min TTL)
    end

    Guard->>Decision: Check Required Permission<br/>Against User Permissions

    alt Has Permission
        Decision-->>Guard: Authorized ✅
        Guard-->>User: Continue to Controller
    else Missing Permission
        Decision-->>Guard: Forbidden ❌
        Guard-->>User: 403 Forbidden
    end
```

**Performance:**
- Cache Hit: ~1ms
- Cache Miss: ~50ms (includes DB query)
- Cache TTL: 5 minutes

---

## Tenant Context Propagation

### Context Extraction

```mermaid
flowchart TD
    A[HTTP Request] --> B{Check JWT Token}
    B -->|Valid JWT| C[Extract User ID & Tenant Context]
    B -->|Invalid/Missing| D[Check API Key]

    C --> E[Tenant Context from JWT:<br/>workspace_id, tenant_id]
    D -->|Valid API Key| F[Tenant Context from API Key:<br/>workspace_id, tenant_id]
    D -->|Invalid| G[401 Unauthorized]

    E --> H[Inject into Request Context]
    F --> H

    H --> I[Available in All Layers:<br/>@TenantContext decorator]
    I --> J[Automatic DB Filtering]

    style E fill:#27ae60
    style F fill:#4ecdc4
    style G fill:#e74c3c
```

### Usage in Controllers

```typescript
// Presentation layer - controller
@Get()
@UseGuards(JwtAuthGuard, TenantContextGuard)
@RequirePermissions('metrics:read:tenant')
async query(
  @TenantContext() tenantContext: TenantContext,
  @Query() filters: QueryDto,
) {
  const query = new GetMetricsQuery(filters, tenantContext);
  return this.queryBus.execute(query);
}
```

### Automatic Database Filtering

```typescript
// Infrastructure layer - repository
async findAll(tenantContext: TenantContext): Promise<Metric[]> {
  // Automatic tenant filtering
  return this.clickhouse.query(
    `SELECT * FROM telemetry_metrics
     WHERE tenant_id = {tenantId:String}
     AND workspace_id = {workspaceId:String}`,
    {
      tenantId: tenantContext.tenantId.value,
      workspaceId: tenantContext.workspaceId.value,
    }
  );
}
```

---

## API Endpoints

### User Management

```mermaid
graph LR
    subgraph "User Endpoints"
        A[POST /api/v2/users<br/>Create User]
        B[GET /api/v2/users<br/>List Users]
        C[GET /api/v2/users/:id<br/>Get User]
        D[PUT /api/v2/users/:id<br/>Update User]
        E[DELETE /api/v2/users/:id<br/>Delete User]
    end

    subgraph "Required Permissions"
        F[user:write]
        G[user:read]
        H[user:read]
        I[user:write]
        J[user:delete]
    end

    A --> F
    B --> G
    C --> H
    D --> I
    E --> J

    style A fill:#ff6b6b
    style F fill:#e74c3c
```

### Role Management

| Endpoint | Method | Permission | Description |
|----------|--------|------------|-------------|
| `/api/v2/roles` | POST | `role:write` | Create new role |
| `/api/v2/roles` | GET | `role:read` | List all roles |
| `/api/v2/roles/:id` | GET | `role:read` | Get role details |
| `/api/v2/roles/:id` | PUT | `role:write` | Update role |
| `/api/v2/roles/:id` | DELETE | `role:delete` | Delete role |
| `/api/v2/users/:id/roles` | POST | `user:write` | Assign role to user |
| `/api/v2/users/:id/roles/:roleId` | DELETE | `user:write` | Revoke role from user |

### Permission Management

| Endpoint | Method | Permission | Description |
|----------|--------|------------|-------------|
| `/api/v2/permissions` | GET | `permission:read` | List all permissions |
| `/api/v2/users/:id/permissions` | GET | `user:read` | Get user permissions |
| `/api/v2/users/:id/permissions` | POST | `user:write` | Grant permission |
| `/api/v2/users/:id/permissions/:permId` | DELETE | `user:write` | Revoke permission |

---

## Database Schema

### PostgreSQL Tables

```sql
-- Users table
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id UUID NOT NULL REFERENCES organizations(id),
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  first_name VARCHAR(100),
  last_name VARCHAR(100),
  is_active BOOLEAN DEFAULT true,
  is_email_verified BOOLEAN DEFAULT false,
  last_login TIMESTAMP,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  deleted_at TIMESTAMP
);

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_organization ON users(organization_id);
CREATE INDEX idx_users_active ON users(is_active) WHERE deleted_at IS NULL;

-- Roles table
CREATE TABLE roles (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(100) UNIQUE NOT NULL,
  description TEXT,
  level INTEGER NOT NULL, -- 1-5 (lower = more powerful)
  is_system BOOLEAN DEFAULT false,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Permissions table
CREATE TABLE permissions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  resource VARCHAR(100) NOT NULL,
  action VARCHAR(50) NOT NULL,
  scope VARCHAR(50) NOT NULL,
  description TEXT,
  UNIQUE(resource, action, scope)
);

-- User-Role junction
CREATE TABLE user_roles (
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  role_id UUID REFERENCES roles(id) ON DELETE CASCADE,
  assigned_at TIMESTAMP DEFAULT NOW(),
  assigned_by UUID REFERENCES users(id),
  PRIMARY KEY (user_id, role_id)
);

-- Role-Permission junction
CREATE TABLE role_permissions (
  role_id UUID REFERENCES roles(id) ON DELETE CASCADE,
  permission_id UUID REFERENCES permissions(id) ON DELETE CASCADE,
  PRIMARY KEY (role_id, permission_id)
);
```

---

## Domain Events

```mermaid
sequenceDiagram
    participant User as User Aggregate
    participant Bus as EventBus
    participant H1 as AuditLogHandler
    participant H2 as EmailNotificationHandler
    participant H3 as CacheInvalidationHandler

    User->>Bus: Publish UserCreated Event
    Bus->>H1: Handle Event
    H1->>H1: Log to Audit Trail

    Bus->>H2: Handle Event
    H2->>H2: Send Welcome Email

    Bus->>H3: Handle Event
    H3->>H3: Invalidate User Cache

    Note over User,H3: Event-driven architecture<br/>enables loose coupling
```

**Domain Events:**
- `UserCreated` - New user registered
- `UserUpdated` - User profile updated
- `UserDeleted` - User account deleted
- `RoleAssignedToUser` - Role granted
- `RoleRevokedFromUser` - Role removed
- `PermissionGranted` - Direct permission added
- `PermissionRevoked` - Permission removed
- `OrganizationCreated` - New organization
- `WorkspaceCreated` - New workspace
- `TenantCreated` - New tenant environment

---

## Security Features

### Password Security

```typescript
// Argon2id hashing with secure defaults
export class PasswordHash {
  static async fromPlaintext(password: string): Promise<PasswordHash> {
    // Validate password strength
    if (password.length < 12) {
      throw new Error('Password must be at least 12 characters');
    }

    // Argon2id parameters (OWASP recommended)
    const hash = await argon2.hash(password, {
      type: argon2.argon2id,
      memoryCost: 65536,  // 64 MB
      timeCost: 3,        // 3 iterations
      parallelism: 4,     // 4 threads
    });

    return new PasswordHash(hash);
  }

  async verify(plaintext: string): Promise<boolean> {
    return argon2.verify(this._hash, plaintext);
  }
}
```

### Account Lockout

```mermaid
stateDiagram-v2
    [*] --> Active: User Created
    Active --> LoginAttempt1: Failed Login
    LoginAttempt1 --> LoginAttempt2: Failed Login
    LoginAttempt2 --> LoginAttempt3: Failed Login
    LoginAttempt3 --> LoginAttempt4: Failed Login
    LoginAttempt4 --> LoginAttempt5: Failed Login
    LoginAttempt5 --> Locked: 5th Failed Login

    LoginAttempt1 --> Active: Successful Login
    LoginAttempt2 --> Active: Successful Login
    LoginAttempt3 --> Active: Successful Login
    LoginAttempt4 --> Active: Successful Login

    Locked --> Active: After 15 minutes
    Locked --> Active: Admin Unlock

    note right of Locked
        Account locked for 15 minutes
        after 5 failed attempts
    end note
```

---

## Performance Optimizations

### Permission Caching

```mermaid
graph LR
    subgraph "Permission Cache Strategy"
        A[User Login] --> B[Load Roles + Permissions]
        B --> C[Cache in Redis<br/>Key: perms:userId<br/>TTL: 5 minutes]
        C --> D[Subsequent Requests<br/>Read from Cache]
    end

    subgraph "Cache Invalidation"
        E[Role/Permission Change] --> F[Invalidate User Cache<br/>DEL perms:userId]
        F --> G[Next Request<br/>Reloads from DB]
    end

    style C fill:#27ae60
    style D fill:#4ecdc4
```

**Results:**
- Authorization check: 1ms (cached) vs 50ms (uncached)
- 50x performance improvement
- Minimal staleness (5min max)

---

## Testing

### Unit Tests
- `User.aggregate.spec.ts` - Business rule validation
- `Email.vo.spec.ts` - Email validation
- `PasswordHash.vo.spec.ts` - Password hashing

### Integration Tests
- `user-management.spec.ts` - CRUD operations
- `rbac.spec.ts` - Permission checking
- `tenant-isolation.spec.ts` - Multi-tenancy enforcement

---

## Related Modules

```mermaid
graph TB
    Core[100-core<br/>IAM Foundation]

    Core --> Auth[200-auth<br/>Authentication]
    Core --> Keys[300-api-keys<br/>API Key Auth]
    Core --> SSO[700-sso<br/>Single Sign-On]
    Core --> Audit[800-audit<br/>Audit Logging]

    Auth -.->|Uses| Core
    Keys -.->|Uses| Core
    SSO -.->|Uses| Core

    style Core fill:#4ecdc4
    style Auth fill:#45b7d1
```

---

- **File Location:** `./backend/modules/100-core.md`
- **Maintained By:** DevOpsCorner Indonesia
