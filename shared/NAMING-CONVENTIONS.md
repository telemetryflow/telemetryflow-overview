# Naming Conventions

- **Version**: 1.1.2-CE
- **Standard**: TypeScript + NestJS + Go
- **Status**: ✅ Enforced

---

## Table of Contents

1. [Overview](#overview)
2. [File Naming](#file-naming)
3. [Variable Naming](#variable-naming)
4. [Function Naming](#function-naming)
5. [Class Naming](#class-naming)
6. [Database Naming](#database-naming)
7. [API Naming](#api-naming)
8. [Module Naming](#module-naming)
9. [Go Language Conventions](#go-language-conventions)

---

## Overview

Consistent naming conventions improve **code readability**, **maintainability**, and **collaboration**. TelemetryFlow follows industry-standard conventions for TypeScript and NestJS.

---

## File Naming

### Backend (TypeScript/NestJS)

**Pattern:** `{name}.{type}.ts`

| File Type | Pattern | Example |
|-----------|---------|---------|
| **Controller** | `{module}.controller.ts` | `user.controller.ts` |
| **Service** | `{module}.service.ts` | `user.service.ts` |
| **Repository** | `{module}.repository.ts` | `user.repository.ts` |
| **Entity** | `{name}.entity.ts` | `User.entity.ts` |
| **DTO** | `{action}-{name}.dto.ts` | `create-user.dto.ts` |
| **Guard** | `{name}.guard.ts` | `auth.guard.ts` |
| **Interceptor** | `{name}.interceptor.ts` | `logging.interceptor.ts` |
| **Middleware** | `{name}.middleware.ts` | `cors.middleware.ts` |
| **Decorator** | `{name}.decorator.ts` | `current-user.decorator.ts` |
| **Module** | `{module}.module.ts` | `user.module.ts` |
| **Test** | `{name}.spec.ts` | `user.service.spec.ts` |
| **E2E Test** | `{name}.e2e-spec.ts` | `user.e2e-spec.ts` |

**DDD Files:**

| File Type | Pattern | Example |
|-----------|---------|---------|
| **Aggregate** | `{Name}.ts` or `{Name}.aggregate.ts` | `User.ts` |
| **Value Object** | `{Name}Id.ts` or `{Name}.ts` | `UserId.ts` |
| **Command** | `{Action}{Name}.command.ts` | `CreateUser.command.ts` |
| **Query** | `Get{Name}.query.ts` | `GetUser.query.ts` |
| **Handler** | `{CommandName}.handler.ts` | `CreateUser.handler.ts` |
| **Event** | `{Name}{Action}Event.ts` | `UserCreatedEvent.ts` |
| **Mapper** | `{name}-response.mapper.ts` | `user-response.mapper.ts` |

### Frontend (Vue 3)

| File Type | Pattern | Example |
|-----------|---------|---------|
| **Component** | `{Name}.vue` | `UserCard.vue` |
| **View** | `{Name}.vue` | `Dashboard.vue` |
| **Store** | `{module}-store.ts` | `user-store.ts` |
| **Composable** | `use{Name}.ts` | `useAuth.ts` |
| **API Client** | `{module}-api.ts` | `user-api.ts` |
| **Repository** | `{module}-repository.ts` | `user-repository.ts` |
| **Type** | `{Name}.ts` | `User.ts` |
| **Test** | `{name}.spec.ts` | `UserCard.spec.ts` |

---

## Variable Naming

### TypeScript Variables

**Use `camelCase` for variables and constants:**

```typescript
// ✅ Good
const userId = 'user_123';
const userName = 'John Doe';
const isActive = true;
const totalCount = 100;

// ❌ Bad
const user_id = 'user_123';
const UserName = 'John Doe';
const is_active = true;
```

**Use `SCREAMING_SNAKE_CASE` for constants:**

```typescript
// ✅ Good
const MAX_RETRY_ATTEMPTS = 3;
const DEFAULT_PAGE_SIZE = 20;
const API_BASE_URL = 'http://localhost:3100';

// ❌ Bad
const maxRetryAttempts = 3;
const default_page_size = 20;
```

**Use descriptive names:**

```typescript
// ✅ Good
const userRepository: UserRepository
const createdAt: Date
const isEmailVerified: boolean

// ❌ Bad
const repo: any
const dt: Date
const flag: boolean
```

---

## Function Naming

### TypeScript Functions

**Use `camelCase` and verb prefixes:**

```typescript
// ✅ Good - CRUD operations
async function createUser(data: CreateUserDto): Promise<User> {}
async function getUser(id: string): Promise<User> {}
async function updateUser(id: string, data: UpdateUserDto): Promise<User> {}
async function deleteUser(id: string): Promise<void> {}

// ✅ Good - Query operations
async function findUserById(id: string): Promise<User | null> {}
async function findAllUsers(filters: UserFilters): Promise<User[]> {}
async function findActiveUsers(): Promise<User[]> {}

// ✅ Good - Boolean checks
function isUserActive(user: User): boolean {}
function hasPermission(user: User, permission: string): boolean {}
function canAccessResource(user: User, resource: Resource): boolean {}

// ✅ Good - Actions
async function sendEmail(to: string, subject: string): Promise<void> {}
async function validateApiKey(apiKey: string): Promise<boolean> {}
async function transformMetric(metric: RawMetric): Promise<Metric> {}

// ❌ Bad
async function user(id: string): Promise<User> {}
async function getAll(): Promise<User[]> {}
function active(user: User): boolean {}
```

### Verb Prefixes

| Prefix | Meaning | Example |
|--------|---------|---------|
| **get** | Retrieve single item | `getUser()`, `getUserById()` |
| **find** | Search/filter items | `findAll()`, `findByEmail()` |
| **create** | Create new item | `createUser()`, `createOrganization()` |
| **update** | Modify existing item | `updateUser()`, `updateProfile()` |
| **delete** | Remove item | `deleteUser()`, `softDelete()` |
| **remove** | Remove from collection | `removePermission()` |
| **add** | Add to collection | `addRole()`, `addMember()` |
| **is/has/can** | Boolean check | `isActive()`, `hasPermission()`, `canAccess()` |
| **validate** | Validation | `validateEmail()`, `validateApiKey()` |
| **transform** | Data transformation | `transformToDto()`, `mapToEntity()` |
| **send** | Send data/message | `sendEmail()`, `sendNotification()` |
| **fetch** | External data retrieval | `fetchFromApi()`, `fetchMetrics()` |

---

## Class Naming

### TypeScript Classes

**Use `PascalCase`:**

```typescript
// ✅ Good - Services
export class UserService {}
export class AuthService {}
export class EmailService {}

// ✅ Good - Controllers
export class UserController {}
export class MetricController {}

// ✅ Good - Repositories
export class UserRepository {}
export class MetricRepository {}

// ✅ Good - Entities/Aggregates
export class User {}
export class Organization {}
export class Metric {}

// ✅ Good - Value Objects
export class UserId {}
export class EmailAddress {}
export class TenantId {}

// ✅ Good - DTOs
export class CreateUserDto {}
export class UpdateUserDto {}
export class UserResponseDto {}

// ✅ Good - Commands
export class CreateUserCommand {}
export class UpdateUserCommand {}

// ✅ Good - Queries
export class GetUserQuery {}
export class GetUsersQuery {}

// ✅ Good - Handlers
export class CreateUserHandler {}
export class GetUserHandler {}

// ✅ Good - Events
export class UserCreatedEvent {}
export class UserUpdatedEvent {}

// ❌ Bad
export class userService {}
export class user_repository {}
export class CREATE_USER_DTO {}
```

---

## Database Naming

### PostgreSQL Tables

**Use `snake_case` for tables and columns:**

#### Table Names
- **Pattern:** Plural nouns in `snake_case`
- **Examples:** `users`, `organizations`, `workspaces`, `tenants`, `alert_rules`, `api_keys`

#### Primary Keys
- **Pattern:** `{entity}_id` (singular entity name + _id)
- **Type:** UUID (preferred) or BIGINT
- **Examples:**
  - Table `users` → Primary key `user_id`
  - Table `organizations` → Primary key `organization_id`
  - Table `alert_rules` → Primary key `alert_rule_id`
  - Table `api_keys` → Primary key `api_key_id`

**❌ AVOID:** Generic `id` without entity name prefix

#### Foreign Keys
- **Pattern:** `{referenced_entity}_id`
- **Examples:**
  - Reference to users → `user_id`
  - Reference to organizations → `organization_id`
  - Reference to workspaces → `workspace_id`
  - Reference to tenants → `tenant_id`

#### Boolean Columns
- **Pattern:** `is_{property}`, `has_{property}`, `{property}_enabled`
- **Examples:**
  - `is_active`, `is_system`, `is_public`, `is_default`, `is_favorite`
  - `has_permission`, `has_access`
  - `mfa_enabled`, `email_verified`, `sso_enabled`

**❌ AVOID:** `status_active`, `active`, `enabled` without prefix

#### Timestamp Columns
- **Pattern:** `{action}_at` or `{state}_at`
- **Type:** TIMESTAMP WITH TIME ZONE (timestamptz)
- **Standard Columns:**
  - `created_at` - Record creation timestamp
  - `updated_at` - Last update timestamp
  - `deleted_at` - Soft deletion timestamp (NULL if not deleted)
- **Action-specific:**
  - `last_login_at`, `last_seen_at`, `last_heartbeat_at`
  - `activated_at`, `deactivated_at`
  - `expires_at`, `revoked_at`, `cancelled_at`
  - `sent_at`, `delivered_at`, `read_at`
  - `scheduled_at`, `executed_at`, `completed_at`

**❌ AVOID:** `date_activation`, `create_date`, `last_login_date`

#### Complete Table Example

```sql
-- ✅ Good - Follows ALL conventions
CREATE TABLE users (
  -- Primary Key: {entity}_id pattern
  user_id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),

  -- Foreign Keys: {referenced_entity}_id
  organization_id UUID NOT NULL REFERENCES organizations(organization_id),
  workspace_id UUID REFERENCES workspaces(workspace_id),
  tenant_id UUID REFERENCES tenants(tenant_id),

  -- Basic fields: snake_case
  email VARCHAR(255) NOT NULL UNIQUE,
  first_name VARCHAR(100),
  last_name VARCHAR(100),
  password_hash VARCHAR(255),

  -- Boolean fields: is_{property} pattern
  is_active BOOLEAN DEFAULT true,
  is_system BOOLEAN DEFAULT false,
  mfa_enabled BOOLEAN DEFAULT false,
  email_verified BOOLEAN DEFAULT false,

  -- Timestamp fields: {action}_at pattern
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  deleted_at TIMESTAMP WITH TIME ZONE,
  last_login_at TIMESTAMP WITH TIME ZONE,

  -- Audit fields
  created_by UUID REFERENCES users(user_id),
  updated_by UUID REFERENCES users(user_id),
  deleted_by UUID REFERENCES users(user_id)
);

CREATE TABLE organizations (
  -- Primary Key
  organization_id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),

  -- Foreign Keys
  region_id UUID REFERENCES regions(region_id),

  -- Fields
  organization_code VARCHAR(50) UNIQUE NOT NULL,
  organization_name VARCHAR(255) NOT NULL,

  -- Boolean fields
  is_active BOOLEAN DEFAULT true,

  -- Timestamps
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  deleted_at TIMESTAMP WITH TIME ZONE
);

CREATE TABLE alert_rules (
  -- Primary Key: Multi-word entities still use singular + _id
  alert_rule_id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),

  -- Foreign Keys
  workspace_id UUID NOT NULL REFERENCES workspaces(workspace_id),
  tenant_id UUID NOT NULL REFERENCES tenants(tenant_id),
  created_by UUID REFERENCES users(user_id),

  -- Fields
  name VARCHAR(255) NOT NULL,
  description TEXT,
  severity VARCHAR(50),
  threshold FLOAT,

  -- Boolean fields
  is_active BOOLEAN DEFAULT true,

  -- Timestamps
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  deleted_at TIMESTAMP WITH TIME ZONE,
  last_evaluated_at TIMESTAMP WITH TIME ZONE,
  last_triggered_at TIMESTAMP WITH TIME ZONE
);

-- ❌ Bad - Violates multiple conventions
CREATE TABLE AlertRule (
  -- ❌ Generic id without entity prefix
  id UUID PRIMARY KEY,

  -- ❌ camelCase instead of snake_case
  workspaceId UUID,
  tenantId UUID,

  -- ❌ PascalCase column names
  Name VARCHAR(255),
  Description TEXT,

  -- ❌ Boolean without is_ prefix
  active BOOLEAN,
  status_active BOOLEAN,

  -- ❌ Timestamp without _at suffix
  date_creation TIMESTAMP,
  last_trigger TIMESTAMP,
  create_date TIMESTAMP
);
```

#### Junction Tables (Many-to-Many)
- **Pattern:** `{entity1}_{entity2}` (alphabetical order)
- **Examples:**
  - `role_permissions` (not `permission_roles`)
  - `user_roles` (not `role_users`)
  - `group_users`

```sql
-- ✅ Good - Junction table
CREATE TABLE role_permissions (
  role_id UUID NOT NULL REFERENCES roles(role_id) ON DELETE CASCADE,
  permission_id UUID NOT NULL REFERENCES permissions(permission_id) ON DELETE CASCADE,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  created_by UUID REFERENCES users(user_id),
  PRIMARY KEY (role_id, permission_id)
);

CREATE TABLE user_roles (
  user_id UUID NOT NULL REFERENCES users(user_id) ON DELETE CASCADE,
  role_id UUID NOT NULL REFERENCES roles(role_id) ON DELETE CASCADE,
  workspace_id UUID REFERENCES workspaces(workspace_id),
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  created_by UUID REFERENCES users(user_id),
  PRIMARY KEY (user_id, role_id, workspace_id)
);
```

### ClickHouse Tables

**Use `snake_case`, prefix with `telemetry_`:**

```sql
-- ✅ Good
CREATE TABLE telemetry_metrics (
  timestamp DateTime64(3),
  metric_name LowCardinality(String),
  metric_id UUID,
  tenant_id UUID,
  value Float64,
  created_at DateTime DEFAULT now()
) ENGINE = MergeTree()
ORDER BY (tenant_id, metric_name, timestamp);

CREATE TABLE telemetry_logs (
  timestamp DateTime64(3),
  severity_text LowCardinality(String),
  log_body String,
  tenant_id UUID
) ENGINE = MergeTree()
ORDER BY (tenant_id, timestamp);

-- ❌ Bad
CREATE TABLE Metrics (
  Timestamp DateTime64(3),
  MetricName String,
  TenantId UUID
);
```

### Indexes and Constraints

```sql
-- ✅ Good - Indexes
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_organization ON users(organization_id);
CREATE INDEX idx_metrics_tenant_time ON telemetry_metrics(tenant_id, timestamp);

-- ✅ Good - Constraints
ALTER TABLE users ADD CONSTRAINT fk_users_organization
  FOREIGN KEY (organization_id) REFERENCES organizations(organization_id);

ALTER TABLE users ADD CONSTRAINT uk_users_email UNIQUE (email);

-- ❌ Bad
CREATE INDEX UserEmail ON users(email);
CREATE INDEX idx1 ON users(organization_id);
```

---

## Migration & Seed Naming

### Migration Files

**Pattern:** `{sequence}-{description}.ts`

#### Naming Convention:
- **Sequence:** 3-digit number (001, 002, 003)
- **Description:** kebab-case, descriptive action
- **Class Name:** PascalCase with timestamp

```typescript
// ✅ Good - Migration file naming
// File: backend/src/modules/100-core/infrastructure/persistence/postgres/migrations/001-create-users-table.ts
export class CreateUsersTable1732200000000 implements MigrationInterface {
  name = 'CreateUsersTable1732200000000';

  public async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`
      CREATE TABLE users (
        user_id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
        email VARCHAR(255) NOT NULL UNIQUE,
        first_name VARCHAR(100),
        last_name VARCHAR(100),
        is_active BOOLEAN DEFAULT true,
        created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
        updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
        deleted_at TIMESTAMP WITH TIME ZONE
      )
    `);
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`DROP TABLE users`);
  }
}

// File: 002-add-mfa-fields-to-users.ts
export class AddMfaFieldsToUsers1732201000000 implements MigrationInterface {
  name = 'AddMfaFieldsToUsers1732201000000';

  public async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`
      ALTER TABLE users
      ADD COLUMN mfa_enabled BOOLEAN DEFAULT false,
      ADD COLUMN mfa_secret VARCHAR(255)
    `);
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`
      ALTER TABLE users
      DROP COLUMN mfa_enabled,
      DROP COLUMN mfa_secret
    `);
  }
}

// File: 003-create-organizations-table.ts
// File: 004-add-soft-deletion-to-users.ts
// File: 005-create-alert-rules-table.ts

// ❌ Bad - Migration file naming
// CreateUsersTable.ts (missing sequence)
// 1-users.ts (unclear description)
// create_users_table.ts (snake_case instead of kebab-case)
// AddMFA.ts (unclear, missing context)
```

#### Migration Action Patterns:

| Pattern | Example |
|---------|---------|
| `create-{table}` | `001-create-users-table.ts` |
| `add-{field}-to-{table}` | `002-add-mfa-fields-to-users.ts` |
| `remove-{field}-from-{table}` | `003-remove-deprecated-fields-from-users.ts` |
| `rename-{old}-to-{new}` | `004-rename-status-to-is-active.ts` |
| `update-{table}-{description}` | `005-update-users-add-indexes.ts` |
| `drop-{table}` | `006-drop-deprecated-sessions.ts` |
| `alter-{table}-{description}` | `007-alter-users-change-email-length.ts` |

#### Module-Specific Migration Numbering:

**Standard: Hyphenated Module-Number Prefix (REQUIRED)**

Use the module number followed by a hyphen and sequence number. This provides crystal-clear module identification and prevents numbering conflicts.

**Pattern:** `{module-number}-{sequence}-{function_migration}.ts`

**Format:**
- **module-number**: The module's numeric identifier (100, 200, 300, etc.)
- **sequence**: 3-digit sequence number (001, 002, 010, etc.)
- **function_migration**: Descriptive action in kebab-case

```
100-core/migrations/
├── 100-001-create-iam-tables.ts
├── 100-002-create-organization-table.ts
├── 100-003-add-soft-deletion.ts
├── 100-999-fix-organization-is-active-column.ts

200-auth/migrations/
├── 200-001-create-auth-sessions-table.ts
├── 200-002-add-mfa-fields-to-users.ts
├── 200-003-create-trusted-devices-table.ts
├── 200-999-rename-id-to-session-id.ts

300-api-keys/migrations/
├── 300-001-create-api-keys-table.ts
├── 300-002-add-rotation-support.ts
├── 300-999-rename-id-to-api-key-id.ts

400-telemetry/migrations/
├── 400-001-create-telemetry-tables.ts
├── 400-999-rename-id-columns.ts

600-alerts/migrations/
├── 600-001-create-alert-rules-table.ts
├── 600-002-add-alert-fatigue-prevention.ts
├── 600-999-rename-id-columns.ts

900-dashboard/migrations/
├── 900-001-create-dashboard-templates.ts
├── 900-002-create-dashboards.ts
├── 900-003-create-widgets.ts

1000-subscription/migrations/
├── 1000-001-create-subscription-tables.ts
├── 1000-999-rename-id-columns.ts

1100-agents/migrations/
├── 1100-001-create-agents-table.ts
├── 1100-002-add-soft-deletion-to-agents.ts
├── 1100-999-rename-status-and-date-columns.ts

1500-retention/migrations/
├── 1500-001-create-retention-policies-table.ts
├── 1500-999-rename-id-to-retention-policy-id.ts
```

**Benefits:**
- ✅ **Crystal-clear module identification** - Module number is explicit
- ✅ **No numbering conflicts** - Each module has isolated sequence
- ✅ **Perfect sorting** - Files group by module naturally
- ✅ **Easy to search** - Can grep for "200-" to find all auth migrations
- ✅ **Scalable** - Supports up to 999 migrations per module

**Examples:**
```typescript
// Module 200 (auth), sequence 010
// File: 200-999-rename-id-to-session-id.ts
export class RenameIdToSessionId1734134400000 implements MigrationInterface {
  name = 'RenameIdToSessionId1734134400000';

  public async up(queryRunner: QueryRunner): Promise<void> {
    // Migration code
  }
}

// Module 600 (alerts), sequence 010
// File: 600-999-rename-id-columns.ts
export class RenameIdColumns1734134700000 implements MigrationInterface {
  name = 'RenameIdColumns1734134700000';

  public async up(queryRunner: QueryRunner): Promise<void> {
    // Migration code
  }
}
```

**❌ AVOID:**
- Generic numbering: `001-create-table.ts` (no module context)
- Concatenated: `100001-create-table.ts` (hard to read)
- Wrong separator: `100_001_create_table.ts` (use hyphens)

### Seed Files

**Pattern:** `{unixtimestamp}-seed-{module_name}-{submodule_or_function}.ts`

#### Naming Convention:
- **Unix Timestamp:** 13-digit millisecond timestamp for chronological ordering
- **Module Name:** Short module identifier (core, auth, api-keys, telemetry, etc.)
- **Submodule/Function:** Specific domain area or functional component

**Why Unix Timestamps?**
- ✅ Guaranteed unique ordering
- ✅ Chronological execution sequence
- ✅ No conflicts between developers
- ✅ Self-documenting creation time

```typescript
// ✅ Good - Seed file naming with unix timestamps

// Module: 100-core
// File: 1704240000101-seed-core-iam-permissions.ts
export class SeedCoreIamPermissions1704240000101 implements Seeder {
  public async run(dataSource: DataSource): Promise<void> {
    const permissionRepository = dataSource.getRepository(Permission);

    const permissions = [
      { permission_id: uuid(), name: 'users:read', description: 'Read users' },
      { permission_id: uuid(), name: 'users:write', description: 'Create/update users' },
      { permission_id: uuid(), name: 'users:delete', description: 'Delete users' },
    ];

    await permissionRepository.save(permissions);
  }
}

// File: 1704240000102-seed-core-iam-roles.ts
export class SeedCoreIamRoles1704240000102 implements Seeder {
  // Seed default roles: super_admin, admin, developer, viewer, demo
}

// File: 1704240000103-seed-core-tenancy-regions.ts
export class SeedCoreTenancyRegions1704240000103 implements Seeder {
  // Seed regions: us-east, eu-west, ap-south
}

// Module: 200-auth
// File: 1704240100201-seed-auth-default-users.ts
export class SeedAuthDefaultUsers1704240100201 implements Seeder {
  // Seed default users for testing
}

// File: 1704240100202-seed-auth-test-sessions.ts
export class SeedAuthTestSessions1704240100202 implements Seeder {
  // Seed auth sessions for development
}

// Module: 600-alerts
// File: 1704240600601-seed-alerts-default-rules.ts
export class SeedAlertsDefaultRules1704240600601 implements Seeder {
  // Seed 33 production-ready alert rules
}

// Module: 900-dashboard
// File: 1704240900901-seed-dashboard-system-monitoring-template.ts
export class SeedDashboardSystemMonitoringTemplate1704240900901 implements Seeder {
  // Seed system monitoring dashboard template
}

// File: 1704240900902-seed-dashboard-apm-template.ts
export class SeedDashboardApmTemplate1704240900902 implements Seeder {
  // Seed APM dashboard template
}

// ❌ Bad - Seed file naming
// permissions.seed.ts (missing timestamp and module context)
// seed-data.ts (too generic, no timestamp)
// default-roles.ts (missing seed prefix and timestamp)
// 001-permissions.ts (sequential number instead of timestamp)
// seed-core-permissions.ts (missing timestamp)
```

#### Seed File Naming Patterns by Module:

| Module | Pattern Example | Purpose |
|--------|----------------|---------|
| **100-core** | `1704240000101-seed-core-iam-permissions.ts` | IAM permissions |
| | `1704240000102-seed-core-iam-roles.ts` | IAM roles |
| | `1704240000103-seed-core-tenancy-regions.ts` | Multi-tenancy regions |
| **200-auth** | `1704240100201-seed-auth-default-users.ts` | Default test users |
| | `1704240100202-seed-auth-mfa-secrets.ts` | MFA test data |
| **300-api-keys** | `1704240300301-seed-api-keys-test-keys.ts` | Test API keys |
| **600-alerts** | `1704240600601-seed-alerts-default-rules.ts` | Default alert rules |
| **900-dashboard** | `1704240900901-seed-dashboard-system-monitoring.ts` | Dashboard templates |

#### Generating Unix Timestamps:

```bash
# Generate current unix timestamp (milliseconds)
date +%s%3N

# Generate timestamp for specific date
date -d "2025-01-15 10:00:00" +%s%3N

# In Node.js
Date.now()  // Returns millisecond timestamp
```

#### Seed File Class Naming:

**Pattern:** `Seed{ModuleName}{Purpose}{Timestamp}`

```typescript
// File: 1704240000101-seed-core-iam-permissions.ts
export class SeedCoreIamPermissions1704240000101 implements Seeder {
  // Class name matches file purpose
}

// File: 1704240900901-seed-dashboard-system-monitoring-template.ts
export class SeedDashboardSystemMonitoringTemplate1704240900901 implements Seeder {
  // Class name is PascalCase version of file purpose
}
```

---

## API Naming

### REST API Endpoints

**Use `kebab-case`, plural nouns:**

```http
# ✅ Good - Resource collections
GET    /api/v1/users
POST   /api/v1/users
GET    /api/v1/users/:id
PATCH  /api/v1/users/:id
DELETE /api/v1/users/:id

# ✅ Good - Nested resources
GET    /api/v1/organizations/:orgId/workspaces
GET    /api/v1/workspaces/:workspaceId/tenants

# ✅ Good - Actions (non-CRUD)
POST   /api/v1/users/:id/activate
POST   /api/v1/users/:id/deactivate
POST   /api/v1/api-keys/:id/revoke

# ✅ Good - Queries
GET    /api/v1/metrics?metric_name=cpu_usage&start_time=1699564800
GET    /api/v1/logs?severity=ERROR&limit=100

# ❌ Bad
GET    /api/v1/getUsers
GET    /api/v1/user
POST   /api/v1/createUser
GET    /api/v1/Users
```

### Query Parameters

**Use `snake_case`:**

```http
# ✅ Good
GET /api/v1/metrics?metric_name=cpu_usage&start_time=1699564800&end_time=1699651200

# ❌ Bad
GET /api/v1/metrics?metricName=cpu_usage&startTime=1699564800&endTime=1699651200
```

### Request/Response Bodies

**Use `snake_case` for JSON keys:**

```json
// ✅ Good
{
  "user_id": "user_123",
  "email": "john@example.com",
  "first_name": "John",
  "last_name": "Doe",
  "is_active": true,
  "created_at": "2025-12-12T10:00:00Z"
}

// ❌ Bad
{
  "userId": "user_123",
  "Email": "john@example.com",
  "firstName": "John",
  "isActive": true
}
```

---

## Module Naming

### Backend Modules

**Use numbers + kebab-case:**

```
modules/
├── 100-core/              # IAM, Multi-tenancy, RBAC
├── 200-auth/              # Authentication
├── 300-api-keys/          # API Keys
├── 400-telemetry/         # OTLP Ingestion
├── 500-monitoring/        # Uptime Monitors
├── 600-alerts/            # Alert Rules
├── 700-sso/               # Single Sign-On
├── 800-audit/             # Audit Logging
├── 900-dashboard/         # Dashboard Templates
├── 1000-subscription/     # Subscription Management
├── 1100-agents/           # Agent Management
├── 1200-status-page/      # Status Pages
├── 1300-export/           # Data Export
├── 1400-query-builder/    # Visual Query Builder
└── 1500-retention-policy/ # Data Retention
```

### Frontend Modules

**Use kebab-case:**

```
modules/
├── auth/
├── telemetry/
├── iam/
├── monitoring/
└── api-keys/
```

---

## Go Language Conventions

> Based on [Effective Go](https://go.dev/doc/effective_go), [Go Code Review Comments](https://github.com/golang/go/wiki/CodeReviewComments), and [Google Go Style Guide](https://google.github.io/styleguide/go/).

### Go File Naming

**Use `snake_case` for file names:**

| File Type | Pattern | Example |
|-----------|---------|---------|
| **Main Package** | `main.go` | `main.go` |
| **Package File** | `{feature}.go` | `order.go`, `user.go` |
| **Test File** | `{feature}_test.go` | `order_test.go` |
| **Platform-Specific** | `{feature}_{os}.go` | `file_linux.go`, `file_windows.go` |
| **Architecture-Specific** | `{feature}_{arch}.go` | `asm_amd64.go` |
| **Platform + Arch** | `{feature}_{os}_{arch}.go` | `syscall_linux_amd64.go` |

**DDD/Clean Architecture Files:**

| File Type | Pattern | Example |
|-----------|---------|---------|
| **Entity** | `{entity}.go` | `order.go`, `user.go` |
| **Repository Interface** | `{entity}_repository.go` | `order_repository.go` |
| **Repository Impl** | `{entity}_repository_{driver}.go` | `order_repository_postgres.go` |
| **Service** | `{entity}_service.go` | `order_service.go` |
| **Handler** | `{entity}_handler.go` | `order_handler.go` |
| **DTO** | `{entity}_dto.go` | `order_dto.go` |
| **Mapper** | `{entity}_mapper.go` | `order_mapper.go` |

---

### Go Package Naming

**Use short, lowercase, single-word names:**

```go
// ✅ Good - Short, lowercase, no underscores
package order
package user
package config
package http
package grpc
package postgres
package telemetry

// ✅ Good - Domain packages
package domain     // Domain entities and value objects
package repository // Repository interfaces
package service    // Business logic
package handler    // HTTP/gRPC handlers
package dto        // Data Transfer Objects
package mapper     // Entity-DTO mappers

// ❌ Bad - Multi-word, underscores, camelCase
package orderService
package order_service
package OrderService
package userManagement
package util       // Too generic - be specific
package common     // Too generic - be specific
package helpers    // Too generic - be specific
package misc       // Too vague
```

**Package Path Conventions:**

```
github.com/telemetryflow/order-service/
├── cmd/
│   └── order-service/
│       └── main.go              # package main
├── internal/
│   ├── domain/
│   │   ├── order/
│   │   │   ├── order.go         # package order
│   │   │   ├── order_status.go
│   │   │   └── order_item.go
│   │   └── customer/
│   │       └── customer.go      # package customer
│   ├── application/
│   │   └── order/
│   │       ├── service.go       # package order
│   │       ├── command.go
│   │       └── query.go
│   ├── infrastructure/
│   │   ├── persistence/
│   │   │   └── postgres/
│   │   │       └── order_repository.go  # package postgres
│   │   └── http/
│   │       └── order_handler.go         # package http
│   └── config/
│       └── config.go            # package config
├── pkg/
│   ├── safefile/
│   │   └── safefile.go          # package safefile
│   └── validator/
│       └── validator.go         # package validator
└── tests/
    ├── unit/
    │   └── order_test.go
    ├── integration/
    │   └── order_integration_test.go
    └── e2e/
        └── order_e2e_test.go
```

---

### Go Variable Naming

**Use `camelCase` for variables, `MixedCaps` for exported:**

```go
// ✅ Good - Local variables (camelCase)
var orderID string
var userName string
var isActive bool
var totalCount int
var httpClient *http.Client
var dbConnection *sql.DB

// ✅ Good - Exported variables (MixedCaps/PascalCase)
var DefaultTimeout = 30 * time.Second
var MaxRetryAttempts = 3

// ✅ Good - Short names for short scopes
for i := 0; i < len(items); i++ { }
for _, v := range values { }
if err != nil { return err }

// ✅ Good - Acronyms: ALL CAPS or all lowercase
var userID string     // Not userId
var httpURL string    // Not httpUrl
var xmlParser Parser  // Not XmlParser
var apiKey string     // Not APIkey (consistent casing)
var urlPath string    // Not URLPath in local scope

// ❌ Bad - snake_case, wrong acronym casing
var user_id string
var user_name string
var httpUrl string    // Should be httpURL
var UserId string     // Should be UserID (exported)
```

**Common Acronym Conventions:**

| Acronym | Exported | Unexported |
|---------|----------|------------|
| **ID** | `UserID`, `OrderID` | `userID`, `orderID` |
| **URL** | `ImageURL`, `APIURL` | `imageURL`, `apiURL` |
| **HTTP** | `HTTPClient`, `HTTPServer` | `httpClient`, `httpServer` |
| **API** | `APIKey`, `APIVersion` | `apiKey`, `apiVersion` |
| **JSON** | `JSONEncoder` | `jsonEncoder` |
| **XML** | `XMLParser` | `xmlParser` |
| **SQL** | `SQLQuery` | `sqlQuery` |
| **TCP/UDP** | `TCPConn`, `UDPAddr` | `tcpConn`, `udpAddr` |
| **UUID** | `UserUUID` | `userUUID` |
| **GRPC** | `GRPCServer` | `grpcServer` |
| **OTLP** | `OTLPExporter` | `otlpExporter` |

---

### Go Constant Naming

**Use `MixedCaps` (NOT SCREAMING_SNAKE_CASE):**

```go
// ✅ Good - Go style constants
const (
    MaxRetryAttempts = 3
    DefaultPageSize  = 20
    DefaultTimeout   = 30 * time.Second

    // Grouped related constants
    StatusPending   = "pending"
    StatusActive    = "active"
    StatusCompleted = "completed"
    StatusCancelled = "cancelled"
)

// ✅ Good - Unexported constants
const (
    maxBufferSize    = 1024
    defaultBatchSize = 100
    retryDelay       = 5 * time.Second
)

// ✅ Good - iota for enumerations
type OrderStatus int

const (
    OrderStatusPending OrderStatus = iota
    OrderStatusConfirmed
    OrderStatusShipped
    OrderStatusDelivered
    OrderStatusCancelled
)

// String method for OrderStatus
func (s OrderStatus) String() string {
    return [...]string{
        "pending",
        "confirmed",
        "shipped",
        "delivered",
        "cancelled",
    }[s]
}

// ❌ Bad - SCREAMING_SNAKE_CASE (not Go style)
const MAX_RETRY_ATTEMPTS = 3
const DEFAULT_PAGE_SIZE = 20
const STATUS_PENDING = "pending"
```

---

### Go Function and Method Naming

**Use `MixedCaps`, verb prefixes, descriptive names:**

```go
// ✅ Good - Exported functions (PascalCase)
func CreateOrder(ctx context.Context, req CreateOrderRequest) (*Order, error) {}
func GetOrderByID(ctx context.Context, id string) (*Order, error) {}
func ListOrders(ctx context.Context, filter OrderFilter) ([]*Order, error) {}
func UpdateOrder(ctx context.Context, id string, req UpdateOrderRequest) (*Order, error) {}
func DeleteOrder(ctx context.Context, id string) error {}

// ✅ Good - Unexported functions (camelCase)
func validateOrder(order *Order) error {}
func calculateTotal(items []OrderItem) decimal.Decimal {}
func formatOrderID(id string) string {}

// ✅ Good - Boolean functions (use Is, Has, Can, Should)
func IsValidEmail(email string) bool {}
func HasPermission(user *User, permission string) bool {}
func CanAccessResource(user *User, resource *Resource) bool {}
func ShouldRetry(err error) bool {}

// ✅ Good - Constructors (New prefix)
func NewOrderService(repo OrderRepository) *OrderService {}
func NewHTTPClient(opts ...ClientOption) *Client {}
func NewConfig(path string) (*Config, error) {}

// ✅ Good - Factory functions for interfaces
func NewRepository(db *sql.DB) Repository {}  // Returns interface

// ✅ Good - Methods on types
func (o *Order) Total() decimal.Decimal {}
func (o *Order) AddItem(item OrderItem) {}
func (o *Order) Cancel() error {}
func (o *Order) IsCompleted() bool {}

// ❌ Bad - Poor naming
func order(id string) (*Order, error) {}       // Not descriptive
func GetAll() ([]*Order, error) {}              // Missing context
func DoOrder(req Request) error {}              // Vague verb
func ProcessIt(data interface{}) error {}       // "It" is unclear
func HandleStuff(ctx context.Context) error {}  // "Stuff" is vague
```

**Function Verb Prefixes:**

| Prefix | Purpose | Example |
|--------|---------|---------|
| **Get** | Retrieve single item | `GetOrderByID()`, `GetUser()` |
| **List** | Retrieve collection | `ListOrders()`, `ListUsers()` |
| **Find** | Search with criteria | `FindByEmail()`, `FindActive()` |
| **Create** | Create new item | `CreateOrder()`, `CreateUser()` |
| **Update** | Modify existing | `UpdateOrder()`, `UpdateProfile()` |
| **Delete** | Remove item | `DeleteOrder()`, `DeleteUser()` |
| **New** | Constructor | `NewService()`, `NewClient()` |
| **Parse** | Parse input | `ParseConfig()`, `ParseJSON()` |
| **Format** | Format output | `FormatDate()`, `FormatMoney()` |
| **Validate** | Validation | `ValidateEmail()`, `ValidateOrder()` |
| **Is/Has/Can** | Boolean check | `IsValid()`, `HasAccess()`, `CanRetry()` |
| **Must** | Panics on error | `MustParse()`, `MustCompile()` |
| **With** | Builder pattern | `WithTimeout()`, `WithRetry()` |

---

### Go Interface Naming

**Single-method interfaces: use verb + "er" suffix:**

```go
// ✅ Good - Single method interfaces (verb + er)
type Reader interface {
    Read(p []byte) (n int, err error)
}

type Writer interface {
    Write(p []byte) (n int, err error)
}

type Closer interface {
    Close() error
}

type Stringer interface {
    String() string
}

type Validator interface {
    Validate() error
}

// ✅ Good - Combined interfaces
type ReadWriter interface {
    Reader
    Writer
}

type ReadWriteCloser interface {
    Reader
    Writer
    Closer
}

// ✅ Good - Domain interfaces (descriptive names)
type OrderRepository interface {
    Create(ctx context.Context, order *Order) error
    GetByID(ctx context.Context, id string) (*Order, error)
    List(ctx context.Context, filter OrderFilter) ([]*Order, error)
    Update(ctx context.Context, order *Order) error
    Delete(ctx context.Context, id string) error
}

type OrderService interface {
    CreateOrder(ctx context.Context, req CreateOrderRequest) (*Order, error)
    GetOrder(ctx context.Context, id string) (*Order, error)
    CancelOrder(ctx context.Context, id string) error
}

type MetricExporter interface {
    Export(ctx context.Context, metrics []Metric) error
    Shutdown(ctx context.Context) error
}

// ✅ Good - Handler interfaces
type Handler interface {
    ServeHTTP(w http.ResponseWriter, r *http.Request)
}

type MessageHandler interface {
    Handle(ctx context.Context, msg Message) error
}

// ❌ Bad - "I" prefix (not Go style)
type IOrderRepository interface {}  // Don't use I prefix
type IUserService interface {}      // Don't use I prefix

// ❌ Bad - "Interface" suffix
type OrderRepositoryInterface interface {}  // Redundant
type UserServiceInterface interface {}      // Redundant
```

---

### Go Struct Naming

**Use `MixedCaps`, descriptive names:**

```go
// ✅ Good - Domain entities
type Order struct {
    ID          string          `json:"id"`
    CustomerID  string          `json:"customer_id"`
    Items       []OrderItem     `json:"items"`
    Status      OrderStatus     `json:"status"`
    TotalAmount decimal.Decimal `json:"total_amount"`
    CreatedAt   time.Time       `json:"created_at"`
    UpdatedAt   time.Time       `json:"updated_at"`
}

type OrderItem struct {
    ProductID   string          `json:"product_id"`
    ProductName string          `json:"product_name"`
    Quantity    int             `json:"quantity"`
    UnitPrice   decimal.Decimal `json:"unit_price"`
}

// ✅ Good - Value Objects
type Money struct {
    Amount   decimal.Decimal
    Currency string
}

type Address struct {
    Street     string
    City       string
    State      string
    PostalCode string
    Country    string
}

// ✅ Good - DTOs (Request/Response)
type CreateOrderRequest struct {
    CustomerID string             `json:"customer_id" validate:"required,uuid"`
    Items      []OrderItemRequest `json:"items" validate:"required,min=1"`
}

type OrderResponse struct {
    ID          string              `json:"id"`
    CustomerID  string              `json:"customer_id"`
    Items       []OrderItemResponse `json:"items"`
    TotalAmount string              `json:"total_amount"`
    Status      string              `json:"status"`
    CreatedAt   string              `json:"created_at"`
}

// ✅ Good - Configuration
type Config struct {
    Server   ServerConfig   `yaml:"server"`
    Database DatabaseConfig `yaml:"database"`
    Redis    RedisConfig    `yaml:"redis"`
    OTEL     OTELConfig     `yaml:"otel"`
}

type ServerConfig struct {
    Host         string        `yaml:"host" env:"SERVER_HOST" default:"0.0.0.0"`
    Port         int           `yaml:"port" env:"SERVER_PORT" default:"8080"`
    ReadTimeout  time.Duration `yaml:"read_timeout" default:"30s"`
    WriteTimeout time.Duration `yaml:"write_timeout" default:"30s"`
}

// ✅ Good - Options pattern
type ClientOption func(*Client)

type Client struct {
    baseURL    string
    httpClient *http.Client
    timeout    time.Duration
    retries    int
}

func WithTimeout(d time.Duration) ClientOption {
    return func(c *Client) {
        c.timeout = d
    }
}

func WithRetries(n int) ClientOption {
    return func(c *Client) {
        c.retries = n
    }
}

// ❌ Bad - Hungarian notation, poor naming
type TOrder struct {}       // T prefix
type OrderStruct struct {}  // Struct suffix
type order_data struct {}   // snake_case
```

---

### Go Error Naming

**Use `Err` prefix for sentinel errors, descriptive messages:**

```go
// ✅ Good - Sentinel errors (package-level)
var (
    ErrNotFound       = errors.New("not found")
    ErrAlreadyExists  = errors.New("already exists")
    ErrInvalidInput   = errors.New("invalid input")
    ErrUnauthorized   = errors.New("unauthorized")
    ErrForbidden      = errors.New("forbidden")
    ErrInternal       = errors.New("internal error")
)

// ✅ Good - Domain-specific errors
var (
    ErrOrderNotFound     = errors.New("order not found")
    ErrOrderCancelled    = errors.New("order already cancelled")
    ErrInsufficientStock = errors.New("insufficient stock")
    ErrPaymentFailed     = errors.New("payment failed")
)

// ✅ Good - Custom error types
type ValidationError struct {
    Field   string
    Message string
}

func (e *ValidationError) Error() string {
    return fmt.Sprintf("validation error on field %s: %s", e.Field, e.Message)
}

type NotFoundError struct {
    Entity string
    ID     string
}

func (e *NotFoundError) Error() string {
    return fmt.Sprintf("%s with id %s not found", e.Entity, e.ID)
}

// ✅ Good - Error wrapping
func GetOrder(ctx context.Context, id string) (*Order, error) {
    order, err := repo.GetByID(ctx, id)
    if err != nil {
        if errors.Is(err, sql.ErrNoRows) {
            return nil, fmt.Errorf("order %s: %w", id, ErrNotFound)
        }
        return nil, fmt.Errorf("get order %s: %w", id, err)
    }
    return order, nil
}

// ❌ Bad - Poor error naming
var NotFoundErr = errors.New("not found")  // Should be ErrNotFound
var ORDER_ERROR = errors.New("error")      // SCREAMING_SNAKE_CASE
var error1 = errors.New("error")           // Non-descriptive
```

---

### Go Test Naming

**Use `Test` prefix, descriptive names with underscores:**

```go
// ✅ Good - Test function naming
func TestOrderService_CreateOrder(t *testing.T) {}
func TestOrderService_CreateOrder_Success(t *testing.T) {}
func TestOrderService_CreateOrder_InvalidInput(t *testing.T) {}
func TestOrderService_CreateOrder_DuplicateOrder(t *testing.T) {}

// ✅ Good - Table-driven tests
func TestCalculateTotal(t *testing.T) {
    tests := []struct {
        name     string
        items    []OrderItem
        expected decimal.Decimal
        wantErr  bool
    }{
        {
            name:     "single item",
            items:    []OrderItem{{Quantity: 1, UnitPrice: decimal.NewFromInt(100)}},
            expected: decimal.NewFromInt(100),
            wantErr:  false,
        },
        {
            name:     "multiple items",
            items:    []OrderItem{
                {Quantity: 2, UnitPrice: decimal.NewFromInt(50)},
                {Quantity: 1, UnitPrice: decimal.NewFromInt(75)},
            },
            expected: decimal.NewFromInt(175),
            wantErr:  false,
        },
        {
            name:     "empty items",
            items:    []OrderItem{},
            expected: decimal.Zero,
            wantErr:  false,
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got, err := calculateTotal(tt.items)
            if (err != nil) != tt.wantErr {
                t.Errorf("calculateTotal() error = %v, wantErr %v", err, tt.wantErr)
                return
            }
            if !got.Equal(tt.expected) {
                t.Errorf("calculateTotal() = %v, want %v", got, tt.expected)
            }
        })
    }
}

// ✅ Good - Benchmark naming
func BenchmarkOrderService_CreateOrder(b *testing.B) {}
func BenchmarkCalculateTotal(b *testing.B) {}

// ✅ Good - Example naming (for documentation)
func ExampleOrderService_CreateOrder() {}
func ExampleNewOrderService() {}

// ❌ Bad - Poor test naming
func Test1(t *testing.T) {}                    // Non-descriptive
func TestCreateOrderWorks(t *testing.T) {}     // Not structured
func Testcreateorder(t *testing.T) {}          // Wrong casing
```

---

### Go Project Structure

**Standard Go project layout:**

```
project-name/
├── cmd/                          # Main applications
│   ├── api/
│   │   └── main.go               # API server entry point
│   └── worker/
│       └── main.go               # Background worker entry point
├── internal/                     # Private application code
│   ├── config/
│   │   └── config.go
│   ├── domain/                   # Domain layer (entities, value objects)
│   │   ├── order/
│   │   │   ├── order.go
│   │   │   ├── order_item.go
│   │   │   ├── order_status.go
│   │   │   └── repository.go     # Repository interface
│   │   └── customer/
│   │       └── customer.go
│   ├── application/              # Application layer (use cases)
│   │   └── order/
│   │       ├── service.go
│   │       ├── create_order.go
│   │       ├── get_order.go
│   │       └── dto.go
│   ├── infrastructure/           # Infrastructure layer
│   │   ├── persistence/
│   │   │   └── postgres/
│   │   │       ├── order_repository.go
│   │   │       └── migrations/
│   │   ├── http/
│   │   │   ├── router.go
│   │   │   ├── middleware/
│   │   │   └── handler/
│   │   │       └── order_handler.go
│   │   └── grpc/
│   │       └── order_server.go
│   └── telemetry/
│       ├── metrics.go
│       ├── tracing.go
│       └── logging.go
├── pkg/                          # Public libraries
│   ├── safefile/
│   │   └── safefile.go
│   └── validator/
│       └── validator.go
├── api/                          # API definitions
│   ├── openapi/
│   │   └── openapi.yaml
│   └── proto/
│       └── order.proto
├── configs/                      # Configuration files
│   ├── config.yaml
│   └── config.example.yaml
├── scripts/                      # Build/deploy scripts
│   ├── build.sh
│   └── migrate.sh
├── tests/                        # Additional test files
│   ├── e2e/
│   └── integration/
├── Makefile
├── Dockerfile
├── go.mod
├── go.sum
└── README.md
```

---

### Go Documentation Comments

**Use complete sentences, start with the name:**

```go
// ✅ Good - Package documentation
// Package order provides order management functionality for the
// TelemetryFlow e-commerce platform. It includes entities, repositories,
// and services for creating, updating, and querying orders.
package order

// ✅ Good - Type documentation
// Order represents a customer order in the system. It contains
// all order details including items, status, and timestamps.
type Order struct {
    // ID is the unique identifier for the order.
    ID string

    // CustomerID is the ID of the customer who placed the order.
    CustomerID string

    // Items contains all items in this order.
    Items []OrderItem

    // Status represents the current state of the order.
    Status OrderStatus

    // CreatedAt is when the order was created.
    CreatedAt time.Time
}

// ✅ Good - Function documentation
// NewOrder creates a new Order with the given customer ID and items.
// It validates the input and returns an error if the order is invalid.
// The order is created with StatusPending and current timestamp.
func NewOrder(customerID string, items []OrderItem) (*Order, error) {
    // implementation
}

// ✅ Good - Method documentation
// AddItem adds an item to the order. It returns an error if the order
// is already completed or cancelled.
func (o *Order) AddItem(item OrderItem) error {
    // implementation
}

// ✅ Good - Constant documentation
// OrderStatus represents the current state of an order.
type OrderStatus int

const (
    // OrderStatusPending indicates the order is awaiting processing.
    OrderStatusPending OrderStatus = iota

    // OrderStatusConfirmed indicates the order has been confirmed.
    OrderStatusConfirmed

    // OrderStatusShipped indicates the order has been shipped.
    OrderStatusShipped

    // OrderStatusDelivered indicates the order has been delivered.
    OrderStatusDelivered

    // OrderStatusCancelled indicates the order has been cancelled.
    OrderStatusCancelled
)

// ❌ Bad - Poor documentation
// This is the order struct
type Order struct {}

// creates order
func CreateOrder() {}

// order status
const StatusPending = 0
```

---

### Go JSON/YAML Tags

**Use `snake_case` for JSON, consistent naming:**

```go
// ✅ Good - JSON tags with snake_case
type Order struct {
    ID          string          `json:"id"`
    CustomerID  string          `json:"customer_id"`
    OrderNumber string          `json:"order_number"`
    TotalAmount decimal.Decimal `json:"total_amount"`
    IsActive    bool            `json:"is_active"`
    CreatedAt   time.Time       `json:"created_at"`
    UpdatedAt   time.Time       `json:"updated_at"`
    DeletedAt   *time.Time      `json:"deleted_at,omitempty"`
}

// ✅ Good - YAML tags for config
type Config struct {
    Server   ServerConfig   `yaml:"server"`
    Database DatabaseConfig `yaml:"database"`
    LogLevel string         `yaml:"log_level" env:"LOG_LEVEL" default:"info"`
}

// ✅ Good - Multiple tags
type User struct {
    ID        string    `json:"id" db:"user_id" validate:"required,uuid"`
    Email     string    `json:"email" db:"email" validate:"required,email"`
    IsActive  bool      `json:"is_active" db:"is_active"`
    CreatedAt time.Time `json:"created_at" db:"created_at"`
}

// ✅ Good - Omitempty for optional fields
type UpdateOrderRequest struct {
    Status      *OrderStatus `json:"status,omitempty"`
    Notes       *string      `json:"notes,omitempty"`
    ShippingAddr *Address    `json:"shipping_address,omitempty"`
}

// ❌ Bad - camelCase in JSON (not API standard)
type Order struct {
    ID         string `json:"id"`
    CustomerId string `json:"customerId"`  // Should be customer_id
    OrderNum   string `json:"orderNum"`    // Should be order_num
    IsActive   bool   `json:"isActive"`    // Should be is_active
}
```

---

### Go Makefile Targets

**Use lowercase, hyphen-separated names:**

```makefile
# ✅ Good - Makefile target naming
.PHONY: build run test lint clean

# Build targets
build:                    ## Build the application
build-linux:              ## Build for Linux
build-darwin:             ## Build for macOS
build-windows:            ## Build for Windows
build-all:                ## Build for all platforms

# Run targets
run:                      ## Run the application
run-dev:                  ## Run in development mode
run-docker:               ## Run in Docker

# Test targets
test:                     ## Run all tests
test-unit:                ## Run unit tests
test-integration:         ## Run integration tests
test-e2e:                 ## Run end-to-end tests
test-coverage:            ## Run tests with coverage

# Lint targets
lint:                     ## Run linter
lint-fix:                 ## Run linter and fix issues
fmt:                      ## Format code
fmt-check:                ## Check code formatting
vet:                      ## Run go vet

# Database targets
db-migrate:               ## Run database migrations
db-migrate-down:          ## Rollback migrations
db-seed:                  ## Seed database

# Docker targets
docker-build:             ## Build Docker image
docker-push:              ## Push Docker image
docker-compose-up:        ## Start Docker Compose
docker-compose-down:      ## Stop Docker Compose

# Clean targets
clean:                    ## Clean build artifacts
clean-all:                ## Clean all generated files

# CI targets
ci-build:                 ## CI build
ci-test:                  ## CI test
ci-lint:                  ## CI lint

# ❌ Bad - Inconsistent naming
Build:                    # Capitalized
runApplication:           # camelCase
test_unit:                # snake_case
TestIntegration:          # PascalCase
```

---

## Best Practices Summary

### TypeScript/NestJS Conventions

| Context | Convention | Example |
|---------|------------|---------|
| **TypeScript Variables** | camelCase | `userId`, `userName` |
| **TypeScript Constants** | SCREAMING_SNAKE_CASE | `MAX_RETRY_ATTEMPTS` |
| **TypeScript Functions** | camelCase | `createUser()`, `findAll()` |
| **TypeScript Classes** | PascalCase | `UserService`, `CreateUserDto` |
| **Files** | kebab-case.type.ts | `user.controller.ts` |
| **Vue Components** | PascalCase | `UserCard.vue` |
| **Vue Stores** | kebab-case-store | `user-store.ts` |

### Go Conventions

| Context | Convention | Example |
|---------|------------|---------|
| **Go Package Names** | lowercase, single-word | `order`, `config`, `http` |
| **Go File Names** | snake_case | `order_service.go`, `order_test.go` |
| **Go Variables (local)** | camelCase | `orderID`, `userName` |
| **Go Variables (exported)** | MixedCaps | `DefaultTimeout`, `MaxRetries` |
| **Go Constants** | MixedCaps (NOT SCREAMING) | `MaxRetryAttempts`, `StatusPending` |
| **Go Functions (exported)** | MixedCaps/PascalCase | `CreateOrder()`, `GetByID()` |
| **Go Functions (unexported)** | camelCase | `validateOrder()`, `formatID()` |
| **Go Interfaces** | verb+er suffix | `Reader`, `Writer`, `OrderRepository` |
| **Go Structs** | MixedCaps | `Order`, `OrderItem`, `CreateOrderRequest` |
| **Go Errors** | Err prefix | `ErrNotFound`, `ErrInvalidInput` |
| **Go Tests** | Test prefix + underscores | `TestOrderService_CreateOrder` |
| **Go JSON Tags** | snake_case | `json:"user_id"`, `json:"created_at"` |
| **Go Makefile Targets** | lowercase, hyphen-separated | `build-linux`, `test-unit` |

### Shared Conventions (All Languages)

| Context | Convention | Example |
|---------|------------|---------|
| **Database Tables** | snake_case, plural | `users`, `telemetry_metrics` |
| **Database Columns** | snake_case | `user_id`, `created_at` |
| **Primary Keys** | {entity}_id | `user_id`, `order_id` |
| **Boolean Columns** | is_{property} | `is_active`, `is_deleted` |
| **Timestamp Columns** | {action}_at | `created_at`, `updated_at` |
| **API Endpoints** | kebab-case, plural | `/api/v1/users`, `/api/v1/orders` |
| **API Query Params** | snake_case | `?metric_name=cpu_usage` |
| **JSON Keys (API)** | snake_case | `"user_id": "123"` |

---

## Related Documentation

- **[02-DDD-CQRS.md](../backend/02-DDD-CQRS.md)** - DDD patterns
- **[03-MODULE-STRUCTURE.md](../backend/03-MODULE-STRUCTURE.md)** - Module organization
- **[DATABASE-SCHEMA.md](./DATABASE-SCHEMA.md)** - Database schema

---

- **Last Updated:** December 26, 2025
- **Maintained By:** DevOpsCorner Indonesia
