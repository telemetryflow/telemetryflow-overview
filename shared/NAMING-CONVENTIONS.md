# Naming Conventions

- **Version**: 3.10.0
- **Standard**: TypeScript + NestJS
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

```sql
-- ✅ Good - Tables
CREATE TABLE users (
  user_id UUID PRIMARY KEY,
  email VARCHAR(255) NOT NULL,
  first_name VARCHAR(100),
  last_name VARCHAR(100),
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  deleted_at TIMESTAMP
);

CREATE TABLE organizations (
  organization_id UUID PRIMARY KEY,
  organization_code VARCHAR(50) UNIQUE,
  organization_name VARCHAR(255),
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP DEFAULT NOW()
);

-- ❌ Bad
CREATE TABLE Users (
  UserId UUID PRIMARY KEY,
  Email VARCHAR(255),
  firstName VARCHAR(100),
  isActive BOOLEAN
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

## Best Practices Summary

| Context | Convention | Example |
|---------|------------|---------|
| **TypeScript Variables** | camelCase | `userId`, `userName` |
| **TypeScript Constants** | SCREAMING_SNAKE_CASE | `MAX_RETRY_ATTEMPTS` |
| **TypeScript Functions** | camelCase | `createUser()`, `findAll()` |
| **TypeScript Classes** | PascalCase | `UserService`, `CreateUserDto` |
| **Files** | kebab-case.type.ts | `user.controller.ts` |
| **Database Tables** | snake_case | `users`, `telemetry_metrics` |
| **Database Columns** | snake_case | `user_id`, `created_at` |
| **API Endpoints** | kebab-case | `/api/v1/users` |
| **API Parameters** | snake_case | `?metric_name=cpu_usage` |
| **JSON Keys** | snake_case | `"user_id": "123"` |
| **Vue Components** | PascalCase | `UserCard.vue` |
| **Vue Stores** | kebab-case-store | `user-store.ts` |

---

## Related Documentation

- **[02-DDD-CQRS.md](../backend/02-DDD-CQRS.md)** - DDD patterns
- **[03-MODULE-STRUCTURE.md](../backend/03-MODULE-STRUCTURE.md)** - Module organization
- **[DATABASE-SCHEMA.md](./DATABASE-SCHEMA.md)** - Database schema

---

- **Last Updated:** December 12, 2025
- **Maintained By:** DevOpsCorner Indonesia
