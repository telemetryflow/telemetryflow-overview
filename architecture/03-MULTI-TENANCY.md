# Multi-Tenancy Architecture

- **Version:** 3.10.0
- **Last Updated:** December 12, 2025
- **Status:** ✅ Production Ready

---

## Table of Contents

1. [Overview](#overview)
2. [Hierarchical Tenant Model](#hierarchical-tenant-model)
3. [Tenant Context Propagation](#tenant-context-propagation)
4. [Database-Level Isolation](#database-level-isolation)
5. [Query Filtering](#query-filtering)
6. [ClickHouse Partitioning](#clickhouse-partitioning)
7. [Security Considerations](#security-considerations)
8. [Performance Impact](#performance-impact)

---

## Overview

TelemetryFlow implements a **4-level hierarchical multi-tenancy** model designed for enterprise SaaS:

```mermaid
graph TD
    A[Region<br/>Geographic Isolation] --> B[Organization<br/>Company Level]
    B --> C[Workspace<br/>Team/Project Level]
    C --> D[Tenant<br/>Environment Level]
    D --> E[Users & Data]

    style A fill:#e1f5ff
    style B fill:#b3e5fc
    style C fill:#81d4fa
    style D fill:#4fc3f7
    style E fill:#29b6f6
```

**Isolation Guarantees:**
- ✅ Data never crosses tenant boundaries
- ✅ Queries automatically scoped to tenant context
- ✅ Database-level partitioning for performance
- ✅ Regional compliance (GDPR, data residency)

---

## Hierarchical Tenant Model

### Entity Hierarchy

```mermaid
erDiagram
    REGION ||--o{ ORGANIZATION : "contains"
    ORGANIZATION ||--o{ WORKSPACE : "contains"
    WORKSPACE ||--o{ TENANT : "contains"
    TENANT ||--o{ USER : "assigned to"
    TENANT ||--o{ API_KEY : "authenticates"
    TENANT ||--o{ TELEMETRY_DATA : "owns"

    REGION {
        uuid region_id PK
        string name "US-East, EU-West, APAC"
        string code "us-east-1, eu-west-1"
        string data_location "North America, Europe, Asia"
        boolean is_active
    }

    ORGANIZATION {
        uuid organization_id PK
        uuid region_id FK
        string name "Acme Corp"
        string code "acme"
        jsonb settings
        boolean is_active
    }

    WORKSPACE {
        uuid workspace_id PK
        uuid organization_id FK
        string name "Production, Staging, Dev"
        string code "prod, staging, dev"
        jsonb datasource_config
        boolean is_active
    }

    TENANT {
        uuid tenant_id PK
        uuid workspace_id FK
        string name "app-server, database, cache"
        string code "app-srv, db, cache"
        string domain "app.acme.com"
        jsonb settings
        boolean is_active
    }

    USER {
        uuid user_id PK
        uuid organization_id FK
        uuid tenant_id FK "Primary tenant"
        string email
        string role "admin, developer, viewer"
    }

    API_KEY {
        uuid api_key_id PK
        uuid workspace_id FK
        uuid tenant_id FK
        string key_id "tfk-xxx"
        string secret_hash
    }

    TELEMETRY_DATA {
        string workspace_id FK
        string tenant_id FK
        datetime timestamp
        string metric_name
        float64 value
    }
```

### Hierarchy Flow

```mermaid
flowchart LR
    A[👤 User Login] --> B{Belongs to Region?}
    B -->|Yes| C[Select Organization]
    C --> D[Select Workspace]
    D --> E[Select Tenant]
    E --> F[Access Telemetry Data]

    B -->|No| G[❌ Access Denied]

    style F fill:#90EE90
    style G fill:#FFB6C1
```

### Hierarchy Examples

**Example 1: E-commerce Platform**
```
Region: US-East-1
└─ Organization: ShopCo Inc
   ├─ Workspace: Production
   │  ├─ Tenant: web-frontend
   │  ├─ Tenant: api-backend
   │  └─ Tenant: payment-service
   └─ Workspace: Staging
      ├─ Tenant: web-frontend
      └─ Tenant: api-backend
```

**Example 2: Multi-Customer MSP**
```
Region: EU-West-1
├─ Organization: Customer-A (ACME Corp)
│  └─ Workspace: Production
│     ├─ Tenant: application
│     └─ Tenant: infrastructure
└─ Organization: Customer-B (TechCo)
   └─ Workspace: Production
      ├─ Tenant: microservice-1
      └─ Tenant: microservice-2
```

---

## Tenant Context Propagation

### Context Extraction Flow

```mermaid
sequenceDiagram
    participant Client as Client<br/>(SDK/Browser)
    participant Gateway as API Gateway
    participant Guard as Auth Guard
    participant Context as Tenant Context
    participant Handler as Request Handler
    participant DB as Database

    Client->>Gateway: Request with Authentication
    Note over Client,Gateway: JWT Token OR API Key

    Gateway->>Guard: Extract & Validate Auth

    alt JWT Token
        Guard->>Guard: Verify JWT signature
        Guard->>Context: Extract from payload<br/>workspaceId, tenantId
    else API Key
        Guard->>Guard: Validate API key
        Guard->>Context: Extract from headers<br/>X-Workspace-Id, X-Tenant-Id
    else OTLP Resource Attributes
        Guard->>Context: Extract from OTLP<br/>telemetryflow.workspace.id
    end

    Context->>Context: Create TenantContext VO
    Context->>Handler: Inject via @CurrentTenantContext

    Handler->>DB: Query with tenant filter
    Note over Handler,DB: WHERE tenant_id = ?<br/>AND workspace_id = ?

    DB-->>Handler: Tenant-scoped results
    Handler-->>Client: Response
```

### Tenant Context Value Object

```mermaid
classDiagram
    class TenantContext {
        +string workspaceId
        +string tenantId
        +string organizationId
        +string source
        +fromJwt(payload) TenantContext$
        +fromHeaders(headers) TenantContext$
        +fromResourceAttributes(attrs) TenantContext$
        +toWhereClause() string
        +validate() void
    }

    class JwtPayload {
        +string sub
        +string email
        +string workspaceId
        +string tenantId
        +string organizationId
        +string[] roles
    }

    class OtlpResourceAttributes {
        +string telemetryflow.workspace.id
        +string telemetryflow.tenant.id
        +string telemetryflow.organization.id
        +string service.name
    }

    TenantContext ..> JwtPayload : extracts from
    TenantContext ..> OtlpResourceAttributes : extracts from
```

### Context Extraction Priority

```mermaid
graph TD
    A[Incoming Request] --> B{Authentication Type}

    B -->|JWT Token| C[Extract from JWT Payload]
    B -->|API Key| D[Extract from HTTP Headers]
    B -->|OTLP Ingestion| E[Extract from Resource Attributes]

    C --> F{Has workspaceId & tenantId?}
    D --> G{Has X-Workspace-Id & X-Tenant-Id?}
    E --> H{Has telemetryflow.workspace.id?}

    F -->|Yes| I[Priority 1: JWT Payload]
    G -->|Yes| J[Priority 2: HTTP Headers]
    H -->|Yes| K[Priority 3: OTLP Attributes]

    F -->|No| L[Use Default Context]
    G -->|No| J
    H -->|No| K

    I --> M[Create TenantContext]
    J --> M
    K --> M
    L --> M

    M --> N[Validate Context]
    N --> O{Valid?}
    O -->|Yes| P[Inject into Request]
    O -->|No| Q[❌ 403 Forbidden]

    style P fill:#90EE90
    style Q fill:#FFB6C1
```

---

## Database-Level Isolation

### PostgreSQL Tenant Filtering

```mermaid
flowchart TD
    A[Repository Method] --> B[Build Query]
    B --> C{Requires Tenant Scope?}

    C -->|Yes| D[Add WHERE Clause]
    D --> E["WHERE tenant_id = $1<br/>AND workspace_id = $2"]

    C -->|No| F[Public Query<br/>e.g., regions]

    E --> G[Execute Query]
    F --> G

    G --> H[Return Results]

    style E fill:#FFE4B5
    style F fill:#E0E0E0
```

**Example Query Pattern:**
```typescript
// UserRepository.findByTenant()
async findByTenant(tenantContext: TenantContext): Promise<User[]> {
  return this.repository.find({
    where: {
      tenant_id: tenantContext.tenantId,
      workspace: { workspace_id: tenantContext.workspaceId },
      is_deleted: false,
    },
    relations: ['workspace', 'organization'],
  });
}
```

### ClickHouse Tenant Filtering

```mermaid
flowchart TD
    A[Telemetry Query] --> B[Build ClickHouse Query]
    B --> C[Add Tenant Filters]

    C --> D["WHERE workspace_id = {workspaceId:String}<br/>AND tenant_id = {tenantId:String}"]

    D --> E[Apply Time Range]
    E --> F["AND timestamp >= {start:DateTime64}<br/>AND timestamp <= {end:DateTime64}"]

    F --> G[Apply Other Filters]
    G --> H["AND service_name = {service:String}<br/>AND metric_name = {metric:String}"]

    H --> I{Use Index?}
    I -->|Yes| J[Bloom Filter Index<br/>on tenant_id]
    I -->|Yes| K[MinMax Index<br/>on timestamp]

    J --> L[Execute Query]
    K --> L

    L --> M[Return Results]

    style D fill:#FFE4B5
    style J fill:#90EE90
    style K fill:#90EE90
```

**Example ClickHouse Query:**
```sql
SELECT
  service_name,
  metric_name,
  avg(value) as avg_value,
  toStartOfHour(timestamp) as hour
FROM metrics_v3
WHERE workspace_id = {workspaceId:String}
  AND tenant_id = {tenantId:String}
  AND timestamp >= {startTime:DateTime64(9)}
  AND timestamp <= {endTime:DateTime64(9)}
  AND metric_name = {metricName:String}
GROUP BY service_name, metric_name, hour
ORDER BY hour DESC
LIMIT 1000
```

---

## Query Filtering

### Automatic Tenant Injection

```mermaid
sequenceDiagram
    participant C as Controller
    participant D as @CurrentTenantContext
    participant Q as QueryBus
    participant H as Query Handler
    participant R as Repository
    participant DB as Database

    C->>D: Extract tenant context
    D-->>C: TenantContext{workspaceId, tenantId}

    C->>Q: Dispatch Query<br/>with TenantContext
    Q->>H: Execute(query, tenantContext)

    H->>R: findByFilters(filters, tenantContext)

    Note over R: Auto-inject tenant filters

    R->>DB: SELECT * FROM table<br/>WHERE tenant_id = ?<br/>AND workspace_id = ?<br/>AND ... other filters

    DB-->>R: Filtered results
    R-->>H: Domain models
    H-->>Q: DTOs
    Q-->>C: Response
```

### Repository Pattern

```mermaid
classDiagram
    class IMetricRepository {
        <<interface>>
        +findById(id, tenantContext) Metric
        +findByFilters(filters, tenantContext) Metric[]
        +save(metric, tenantContext) void
        +delete(id, tenantContext) void
    }

    class MetricRepository {
        -clickhouseService: ClickHouseService
        +findByFilters(filters, context) Metric[]
    }

    class TenantContext {
        +workspaceId: string
        +tenantId: string
        +organizationId: string
        +toWhereClause() string
    }

    class MetricQueryFilters {
        +tenantContext: TenantContext
        +serviceName: string
        +metricName: string
        +startTime: Date
        +endTime: Date
    }

    IMetricRepository <|.. MetricRepository : implements
    MetricRepository ..> TenantContext : uses
    MetricQueryFilters *-- TenantContext : contains
```

---

## ClickHouse Partitioning

### Partitioning Strategy

```mermaid
graph TD
    A[Metrics Table] --> B{Partition By}

    B --> C[Time-based Partitioning]
    C --> D["PARTITION BY toYYYYMMDD(timestamp)"]

    D --> E[Daily Partitions]
    E --> F["2025-12-01<br/>2025-12-02<br/>2025-12-03"]

    B --> G[NOT Partitioned by Tenant]
    G --> H[Reason: Too many partitions<br/>100+ tenants × 90 days = 9000 partitions]

    style C fill:#90EE90
    style G fill:#FFB6C1
```

### Partition Pruning

```mermaid
flowchart LR
    A[Query with<br/>Time Range] --> B[Partition Pruning]

    B --> C{Date in Range?}

    C -->|2025-12-01| D[✅ Read Partition]
    C -->|2025-12-02| E[✅ Read Partition]
    C -->|2025-12-03| F[✅ Read Partition]
    C -->|2025-11-30| G[❌ Skip Partition]
    C -->|2025-12-04| H[❌ Skip Partition]

    D --> I[Filter by tenant_id<br/>using Bloom Filter]
    E --> I
    F --> I

    I --> J[Return Results]

    style D fill:#90EE90
    style E fill:#90EE90
    style F fill:#90EE90
    style G fill:#E0E0E0
    style H fill:#E0E0E0
```

### Index-Based Tenant Filtering

```mermaid
graph TD
    A[Query Execution] --> B[Partition Selection<br/>by timestamp]

    B --> C[Index Filtering]

    C --> D[Bloom Filter: tenant_id]
    C --> E[Bloom Filter: workspace_id]
    C --> F[Bloom Filter: service_name]

    D --> G{Match tenant_id?}
    E --> H{Match workspace_id?}
    F --> I{Match service_name?}

    G -->|Yes| J[Include Granule]
    G -->|No| K[Skip Granule<br/>10-50x faster]

    H -->|Yes| J
    H -->|No| K

    I -->|Yes| J
    I -->|No| K

    J --> L[Read Data]
    L --> M[Return Results]

    style J fill:#90EE90
    style K fill:#FFB6C1
```

### Table Schema with Tenant Columns

```sql
CREATE TABLE metrics_v3 (
    timestamp DateTime64(9),
    metric_name String,
    service_name String,

    -- Multi-tenancy columns
    organization_id String DEFAULT '',
    workspace_id String DEFAULT '',
    tenant_id String DEFAULT '',

    resource_attributes Map(String, String),
    attributes Map(String, String),
    value Float64,
    unit String,
    metric_type Enum8('gauge'=1, 'counter'=2, 'histogram'=3, 'summary'=4),

    INDEX idx_tenant_id tenant_id TYPE bloom_filter GRANULARITY 1,
    INDEX idx_workspace_id workspace_id TYPE bloom_filter GRANULARITY 1,
    INDEX idx_organization_id organization_id TYPE bloom_filter GRANULARITY 1,
    INDEX idx_service_name service_name TYPE bloom_filter GRANULARITY 1,
    INDEX idx_metric_name metric_name TYPE bloom_filter GRANULARITY 1,
    INDEX idx_timestamp timestamp TYPE minmax GRANULARITY 1
)
ENGINE = MergeTree()
PARTITION BY toYYYYMMDD(timestamp)
ORDER BY (service_name, metric_name, timestamp)
TTL toDateTime(timestamp) + INTERVAL 90 DAY;
```

---

## Security Considerations

### Tenant Isolation Guarantees

```mermaid
graph TD
    A[Security Layer 1:<br/>Authentication] --> B[JWT Validation<br/>API Key Validation]

    B --> C[Security Layer 2:<br/>Authorization]
    C --> D[RBAC Permission Check]

    D --> E[Security Layer 3:<br/>Context Extraction]
    E --> F[Extract workspaceId<br/>Extract tenantId]

    F --> G[Security Layer 4:<br/>Query Filtering]
    G --> H[PostgreSQL: WHERE tenant_id = ?]
    G --> I[ClickHouse: WHERE tenant_id = ?]

    H --> J[Security Layer 5:<br/>Result Validation]
    I --> J

    J --> K{Results belong to<br/>correct tenant?}
    K -->|Yes| L[✅ Return Data]
    K -->|No| M[❌ Security Violation<br/>Log & Alert]

    style L fill:#90EE90
    style M fill:#FF6B6B
```

### Cross-Tenant Access Prevention

```mermaid
sequenceDiagram
    participant U1 as User (Tenant A)
    participant API as API Gateway
    participant Auth as Auth Service
    participant DB as Database
    participant Audit as Audit Log

    U1->>API: Request data for Tenant B
    API->>Auth: Validate JWT
    Auth->>Auth: Extract tenant context<br/>tenantId = A

    API->>DB: Query with tenantId = A
    Note over API,DB: User CANNOT specify<br/>different tenant ID

    alt Attempt to bypass tenant filter
        U1->>API: Request with modified tenant_id
        API->>Auth: Validate tenant context
        Auth->>Audit: ⚠️ Log security violation
        Auth-->>U1: ❌ 403 Forbidden
    end

    DB-->>API: Data for Tenant A only
    API-->>U1: ✅ Authorized data
```

### Audit Trail

```mermaid
stateDiagram-v2
    [*] --> Request: User Action
    Request --> Extract: Extract Tenant Context
    Extract --> Validate: Validate Permissions

    Validate --> Execute: Authorized
    Validate --> Denied: Unauthorized

    Execute --> Log: Query Database
    Denied --> LogViolation: Security Event

    Log --> [*]: Audit success
    LogViolation --> [*]: Audit violation

    note right of Log
        Audit Fields
        user_id
        tenant_id
        workspace_id
        action
        resource
        timestamp
        ip_address
        result
    end note
```

---

## Performance Impact

### Query Performance Comparison

```mermaid
graph LR
    subgraph "Without Tenant Filtering"
        A1[Full Table Scan] --> B1[1M rows scanned]
        B1 --> C1[500ms query time]
    end

    subgraph "With Tenant Filtering"
        A2[Bloom Filter Index] --> B2[10k rows scanned<br/>100x reduction]
        B2 --> C2[5ms query time<br/>100x faster]
    end

    style C1 fill:#FFB6C1
    style C2 fill:#90EE90
```

### Index Hit Rate

```mermaid
pie title Tenant Filter Performance
    "Index Hit" : 95
    "Index Miss" : 5
```

### Partition Pruning Impact

| Query Type | Without Pruning | With Pruning | Improvement |
|------------|----------------|--------------|-------------|
| **Single Day** | 90 partitions | 1 partition | 90x faster |
| **1 Week** | 90 partitions | 7 partitions | 12.8x faster |
| **1 Month** | 90 partitions | 30 partitions | 3x faster |

### Cache Strategy per Tenant

```mermaid
graph TD
    A[Query Request] --> B{Check L1 Cache}

    B -->|Hit| C[Return from Memory]
    B -->|Miss| D{Check L2 Cache Redis}

    D -->|Hit| E[Return from Redis]
    D -->|Miss| F[Query Database]

    F --> G[Cache L2 tenant scoped]
    G --> H[Cache L1]
    H --> I[Return Results]

    C --> I
    E --> H

    style C fill:#90EE90
    style E fill:#FFD700
    style F fill:#FFA07A
```

---

## Implementation Checklist

### ✅ Multi-Tenancy Implementation

- [x] **Hierarchical Model**
  - [x] Region → Organization → Workspace → Tenant entities
  - [x] Foreign key relationships enforced
  - [x] Soft delete for compliance

- [x] **Context Propagation**
  - [x] TenantContext value object
  - [x] @CurrentTenantContext decorator
  - [x] Extraction from JWT payload
  - [x] Extraction from API key headers
  - [x] Extraction from OTLP resource attributes

- [x] **Database Isolation**
  - [x] PostgreSQL: WHERE tenant_id = ? filters
  - [x] ClickHouse: tenant_id columns with indexes
  - [x] Bloom filter indexes on tenant columns
  - [x] Query parameterization (prevent injection)

- [x] **Security**
  - [x] Tenant context validation
  - [x] Cross-tenant access prevention
  - [x] Audit logging for all operations
  - [x] Security violation detection

- [x] **Performance**
  - [x] Multi-level caching per tenant
  - [x] Partition pruning by time
  - [x] Bloom filter indexes
  - [x] Query optimization

---

## Best Practices

### 1. Always Use Tenant Context

```typescript
// ❌ BAD: No tenant context
async getMetrics() {
  return this.metricRepository.find();
}

// ✅ GOOD: Tenant context injected
async getMetrics(@CurrentTenantContext() context: TenantContext) {
  return this.metricRepository.findByTenant(context);
}
```

### 2. Validate Tenant Access

```typescript
// ✅ Always validate tenant context
if (!tenantContext || !tenantContext.tenantId) {
  throw new ForbiddenException('Tenant context required');
}

// ✅ Validate entity belongs to tenant
const dashboard = await this.findById(id);
if (dashboard.tenant_id !== tenantContext.tenantId) {
  throw new ForbiddenException('Access denied');
}
```

### 3. Use Parameterized Queries

```typescript
// ❌ BAD: SQL injection risk
const query = `SELECT * FROM metrics WHERE tenant_id = '${tenantId}'`;

// ✅ GOOD: Parameterized query
const query = `
  SELECT * FROM metrics
  WHERE tenant_id = {tenantId:String}
  AND workspace_id = {workspaceId:String}
`;
const params = { tenantId, workspaceId };
```

### 4. Cache Per Tenant

```typescript
// ✅ Tenant-scoped cache keys
const cacheKey = `tenant:${tenantId}:metrics:${queryHash}`;
const cached = await this.cache.get(cacheKey);
```

---

## Next Steps

- [Data Flow Architecture](./02-DATA-FLOW.md)
- [Security Architecture](./04-SECURITY.md)
- [Performance Optimizations](./05-PERFORMANCE.md)
- [Database Schema](../shared/DATABASE-SCHEMA.md)

---

- **Last Updated:** December 12, 2025
- **Maintained By:** DevOpsCorner Indonesia
