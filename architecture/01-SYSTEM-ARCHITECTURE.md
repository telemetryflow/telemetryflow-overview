# TelemetryFlow System Architecture

- **Version:** 1.0.0-CE
- **Last Updated:** December 12, 2025

---

## Table of Contents

1. [High-Level Architecture](#high-level-architecture)
2. [Architecture Principles](#architecture-principles)
3. [Component Overview](#component-overview)
4. [Layer Architecture (DDD)](#layer-architecture-ddd)
5. [CQRS Pattern](#cqrs-pattern)
6. [Event-Driven Architecture](#event-driven-architecture)
7. [Database Architecture](#database-architecture)
8. [Caching Strategy](#caching-strategy)
9. [Queue Architecture](#queue-architecture)
10. [Scalability Considerations](#scalability-considerations)

---

## High-Level Architecture

```mermaid
graph TB
    subgraph "External Systems"
        OTLP[OpenTelemetry Clients<br/>SDK/Collector]
        SSO[SSO Login<br/>Google/GitHub/Azure/Okta]
        Browser[Web Browser<br/>Vue 3 SPA]
        Notif[Notification Services<br/>Slack/PagerDuty/Email/Teams]
    end

    subgraph "Platform Core - NestJS Backend"
        Controllers[Controllers<br/>REST API]
        AppLayer[Application Layer<br/>Commands/Queries]
        DomainLayer[Domain Layer<br/>Aggregates/Events]
        InfraLayer[Infrastructure Layer<br/>TypeORM/Repositories]

        Controllers --> AppLayer
        AppLayer --> DomainLayer
        DomainLayer --> InfraLayer
    end

    subgraph "Data Stores"
        PG[(PostgreSQL<br/>Metadata)]
        CH[(ClickHouse<br/>Telemetry)]
        Redis[(Redis<br/>Cache/Queue)]

        PGData[• Users<br/>• Organizations<br/>• Workspaces<br/>• Alert Rules<br/>• Dashboards]
        CHData[• Metrics<br/>• Logs<br/>• Traces<br/>• Alert History<br/>• Audit Logs]
        RedisData[• L2 Cache<br/>• BullMQ Jobs<br/>• Sessions]

        PG -.-> PGData
        CH -.-> CHData
        Redis -.-> RedisData
    end

    subgraph "Async Processing - BullMQ Queues"
        Q1[Queue 1: OTLP Ingestion<br/>Metrics/Logs/Traces processing]
        Q2[Queue 2: Alerts<br/>Alert rule evaluation]
        Q3[Queue 3: Aggregation<br/>Hourly/daily rollups]
        Q4[Queue 4: Cleanup<br/>Data retention enforcement]
        Q5[Queue 5: Notifications<br/>Multi-channel alert delivery]
    end

    OTLP -->|OTLP Protocol| Controllers
    SSO -->|OAuth| Controllers
    Browser -->|HTTPS| Controllers
    Browser <-->|WebSocket| Controllers
    Notif <-.->|Webhook| InfraLayer

    InfraLayer <--> PG
    InfraLayer <--> CH
    InfraLayer <--> Redis

    InfraLayer --> Q1
    InfraLayer --> Q2
    InfraLayer --> Q3
    InfraLayer --> Q4
    InfraLayer --> Q5

    style Controllers fill:#4ecdc4
    style AppLayer fill:#45b7d1
    style DomainLayer fill:#f9ca24
    style InfraLayer fill:#ff6b6b
    style PG fill:#95a5a6
    style CH fill:#6c5ce7
    style Redis fill:#e74c3c
```

---

## Architecture Principles

### 1. Domain-Driven Design (DDD)

**Why DDD?**
- Complex business logic organized around domain concepts
- Clear boundaries between modules (15 bounded contexts)
- Ubiquitous language across team (Metric, Log, Trace, Tenant, etc.)
- Separation of concerns (domain logic vs infrastructure)

**Key DDD Components:**
- **Aggregates:** User, Organization, Workspace, Tenant, Metric, Log, Trace
- **Value Objects:** Email, TenantId, MetricName, TraceContext, Timestamp
- **Domain Events:** UserCreated, MetricIngested, AlertTriggered
- **Repositories:** Abstraction over data access (ports & adapters)
- **Domain Services:** Business logic that doesn't belong to a single entity

### 2. CQRS (Command Query Responsibility Segregation)

**Why CQRS?**
- Separate write operations (Commands) from read operations (Queries)
- Optimize reads independently from writes
- Enable event sourcing for audit trails
- Scale reads and writes independently

**Benefits:**
- **Performance:** Read-optimized models in ClickHouse, write-optimized in PostgreSQL
- **Complexity Management:** Clear separation of business logic
- **Scalability:** Independent scaling of command and query sides
- **Auditability:** Every command produces events for audit log

### 3. Event-Driven Architecture

**Why Event-Driven?**
- Loose coupling between modules
- Async processing for heavy operations
- Real-time updates to UI via WebSocket
- Audit trail of all system changes

**Event Types:**
- **Domain Events:** Business state changes (UserCreated, MetricIngested)
- **Integration Events:** Cross-module communication (AlertTriggered → NotificationSent)
- **System Events:** Infrastructure events (CacheMiss, QueueFull)

### 4. Microservices-Ready Modularity

**Why Modular Monolith?**
- Start as monolith for simplicity
- Each module is independently deployable
- Clear module boundaries (15 modules)
- Easy to extract to microservices later

**Module Independence:**
- Shared-nothing architecture
- Communication via events only
- No direct cross-module imports (only via public APIs)
- Each module has its own database schema

---

## Component Overview

### Frontend (Vue 3 SPA)

```mermaid
graph TB
    subgraph "Vue 3 Single Page App"
        VueStack[Technology Stack]
        VueFeatures[• Vue 3.5.24 + TypeScript 5.8.3<br/>• Vite 7.2.4 Build tool<br/>• Pinia 3.0.4 State management<br/>• Vue Router 4.6.3 Routing<br/>• Naive UI 2.43.2 Component library<br/>• ECharts 6.0.0 Visualization<br/>• Axios 1.13.2 HTTP client<br/>• Socket.IO 4.8.1 WebSocket]
    end

    subgraph "API Gateway"
        NestJS[NestJS Backend API]
    end

    VueStack -->|REST API HTTPS| NestJS
    NestJS -->|WebSocket WSS| VueStack

    style VueStack fill:#4ecdc4
    style NestJS fill:#45b7d1
```

### Backend (NestJS)

```mermaid
graph TB
    subgraph "NestJS Backend"
        Framework[Framework: NestJS 10.x + TypeScript 5.7+<br/>Runtime: Node.js 18-20.x]

        subgraph "15 Main Modules - Bounded Contexts"
            M100[100-core<br/>IAM, Multi-tenancy, RBAC]
            M200[200-auth<br/>Authentication, JWT, MFA]
            M300[300-api-keys<br/>API key authentication]
            M400[400-telemetry<br/>OTLP ingestion engine]
            M500[500-monitoring<br/>Uptime monitors]
            M600[600-alerts<br/>Alert rules & notifications]
            M700[700-sso<br/>Google/GitHub/Azure SSO]
            M800[800-audit<br/>Audit logging]
            M900[900-dashboard<br/>Dashboard templates]
            M1000[1000-subscription<br/>Billing management]
            M1100[1100-agents<br/>Agent management]
            M1200[1200-status-page<br/>Status pages]
            M1300[1300-export<br/>Data export]
            M1400[1400-query-builder<br/>Visual query builder]
            M1500[1500-retention-policy<br/>Data retention]
        end

        subgraph "10 Shared Modules"
            Shared[logger, cache, queue, messaging, email<br/>cors, clickhouse, otel, ui, platform]
        end
    end

    style M400 fill:#ff6b6b
    style M100 fill:#4ecdc4
    style M200 fill:#45b7d1
    style M600 fill:#f9ca24
    style Shared fill:#95a5a6
```

### Data Stores

```mermaid
graph LR
    subgraph "PostgreSQL 15+"
        PGFeatures[• Relational data<br/>• ACID compliance<br/>• Strong consistency<br/>• TypeORM ORM]
        PGTables[Tables 30+:<br/>• users<br/>• organizations<br/>• workspaces<br/>• tenants<br/>• roles<br/>• permissions<br/>• alert_rules<br/>• dashboards<br/>• widgets<br/>• api_keys]
    end

    subgraph "ClickHouse 23+"
        CHFeatures[• Time-series data<br/>• Columnar storage<br/>• 50-90% compression<br/>• 10-50x faster]
        CHTables[Tables 10+:<br/>• metrics<br/>• logs<br/>• traces<br/>• spans<br/>• exemplars<br/>• alert_history<br/>• audit_logs<br/>• uptime_checks<br/>• service_list<br/>• metric_list]
    end

    subgraph "Redis 7+"
        RedisFeatures[• L2 cache<br/>• BullMQ queues<br/>• Session storage<br/>• Pub/Sub]
        RedisKeys[Keys:<br/>• user:session:*<br/>• cache:*<br/>• bull:otlp:*<br/>• bull:alerts:*<br/>• bull:aggregation:*<br/>• bull:cleanup:*<br/>• bull:notify:*]
    end

    style PGFeatures fill:#4ecdc4
    style CHFeatures fill:#6c5ce7
    style RedisFeatures fill:#e74c3c
```

---

## Layer Architecture (DDD)

### 4-Layer Architecture

```mermaid
graph TB
    subgraph "1. PRESENTATION LAYER"
        P1[Responsibilities: HTTP handling, DTO validation, Auth guards]
        P2[• Controllers REST endpoints<br/>• DTOs Request/Response objects with validation<br/>• Guards AuthGuard, RolesGuard, PermissionsGuard<br/>• Decorators @Public, @Roles, @RequiredPermissions<br/>• Interceptors Logging, Tracing, Error handling<br/>• Filters Exception filters]
    end

    subgraph "2. APPLICATION LAYER"
        A1[Responsibilities: Orchestration, Business workflows]
        A2[• Commands Write operations - IngestMetricsCommand<br/>• Queries Read operations - GetMetricsByFiltersQuery<br/>• Handlers Execute commands/queries - 40+ handlers<br/>• Application Services Coordinate domain operations<br/>• DTOs Application-level data transfer objects<br/>• EventBus Publish domain events]
    end

    subgraph "3. DOMAIN LAYER"
        D1[Responsibilities: Business rules, Domain logic]
        D2[• Aggregates User, Metric, Log, Trace, AlertRule<br/>• Entities Mutable objects with identity<br/>• Value Objects Immutable concepts - Email, TenantId<br/>• Domain Events UserCreated, MetricIngested, AlertTriggered<br/>• Repository Interfaces Ports - IUserRepository<br/>• Domain Services Cross-aggregate business logic<br/>• Business Rules Validation, Invariants]
    end

    subgraph "4. INFRASTRUCTURE LAYER"
        I1[Responsibilities: External systems, Persistence, 3rd party APIs]
        I2[• TypeORM Entities PostgreSQL schema<br/>• Repository Implementations Adapters - UserRepository<br/>• ClickHouse Adapters Time-series persistence<br/>• Redis Cache Implementations<br/>• BullMQ Queue Processors<br/>• External Service Clients Email, Slack, PagerDuty<br/>• Mappers Domain ↔ Persistence transformation]
    end

    P2 -->|HTTP Request| A2
    A2 -->|Business Logic| D2
    D2 -->|Persistence| I2

    style P1 fill:#4ecdc4
    style A1 fill:#45b7d1
    style D1 fill:#f9ca24
    style I1 fill:#ff6b6b
```

### Dependency Rule

```mermaid
graph TD
    Presentation[Presentation Layer]
    Application[Application Layer]
    Domain[Domain Layer<br/>Core Business Logic]
    Infrastructure[Infrastructure Layer]

    Presentation --> Application
    Presentation -.-> Domain
    Application --> Domain
    Infrastructure --> Domain

    Note1[Rule: Dependencies point INWARD only<br/>Infrastructure knows Domain<br/>Domain knows NOTHING about outer layers]

    style Domain fill:#f9ca24
    style Presentation fill:#4ecdc4
    style Application fill:#45b7d1
    style Infrastructure fill:#ff6b6b
    style Note1 fill:#95a5a6
```

**Key Points:**
- **Domain Layer** is independent (no imports from outer layers)
- **Application Layer** depends only on Domain
- **Infrastructure Layer** implements Domain interfaces (Dependency Inversion)
- **Presentation Layer** uses Application services

---

## CQRS Pattern

### Command/Query Separation

```mermaid
graph TB
    subgraph "WRITE SIDE - Commands"
        W1[POST /api/v2/otlp/metrics]
        W2[IngestMetrics Command]
        W3[IngestMetrics Handler]
        W4[Domain Validation<br/>Business Rules]
        W5[QueueService<br/>BullMQ]
        W6[Async Processing<br/>Worker Thread]
        W7[MetricRepository<br/>ClickHouse]
        W8[Domain Event<br/>MetricIngested]
        W9[Event Handlers<br/>• Alert evaluation<br/>• Aggregation<br/>• Audit logging]

        W1 --> W2
        W2 --> W3
        W3 --> W4
        W4 --> W5
        W5 --> W6
        W6 --> W7
        W7 --> W8
        W8 --> W9
    end

    subgraph "READ SIDE - Queries"
        R1[GET /api/v2/telemetry/metrics?filters]
        R2[GetMetricsByFilters Query]
        R3[GetMetricsByFilters Handler]
        R4[ClickHouse<br/>Optimized Reads]

        R1 --> R2
        R2 --> R3
        R3 --> R4
    end

    style W1 fill:#ff6b6b
    style W5 fill:#f9ca24
    style W7 fill:#6c5ce7
    style R1 fill:#4ecdc4
    style R4 fill:#6c5ce7
```

### Command Examples

```typescript
// 1. Create Command
export class IngestMetricsFromOtlpCommand {
  constructor(
    public readonly otlpMetrics: OtlpMetrics,
    public readonly apiKey: ApiKey,
    public readonly tenantContext: TenantContext,
  ) {}
}

// 2. Command Handler
@CommandHandler(IngestMetricsFromOtlpCommand)
export class IngestMetricsFromOtlpHandler {
  async execute(command: IngestMetricsFromOtlpCommand): Promise<void> {
    // 1. Transform OTLP to domain model
    const metrics = this.transformOtlpToDomain(command.otlpMetrics);

    // 2. Validate business rules
    metrics.forEach(metric => metric.validate());

    // 3. Emit domain event
    this.eventBus.publish(new MetricIngested(metrics));

    // 4. Queue for async processing
    await this.queueService.addJob('otlp-ingestion', { metrics });
  }
}

// 3. Event Handler
@EventsHandler(MetricIngested)
export class AlertEvaluationHandler {
  async handle(event: MetricIngested): Promise<void> {
    // Evaluate alert rules in background
    await this.queueService.addJob('alert-evaluation', {
      metrics: event.metrics,
    });
  }
}
```

### Query Examples

```typescript
// 1. Create Query
export class GetMetricsByFiltersQuery {
  constructor(
    public readonly filters: MetricFilters,
    public readonly tenantContext: TenantContext,
  ) {}
}

// 2. Query Handler
@QueryHandler(GetMetricsByFiltersQuery)
export class GetMetricsByFiltersHandler {
  async execute(query: GetMetricsByFiltersQuery): Promise<Metric[]> {
    // 1. Check cache first
    const cacheKey = this.buildCacheKey(query);
    const cached = await this.cacheService.get(cacheKey);
    if (cached) return cached;

    // 2. Query ClickHouse (read-optimized)
    const metrics = await this.metricRepository.findByFilters(
      query.filters,
      query.tenantContext,
    );

    // 3. Cache results
    await this.cacheService.set(cacheKey, metrics, 60); // 60s TTL

    return metrics;
  }
}
```

---

## Event-Driven Architecture

### Event Flow

```mermaid
sequenceDiagram
    participant API as POST /api/v2/otlp/metrics
    participant CMD as IngestMetricsCommand
    participant Handler as IngestMetricsHandler
    participant Event as Domain Event<br/>MetricIngested
    participant Bus as EventBus
    participant H1 as AlertEvaluationHandler
    participant H2 as AggregationHandler
    participant H3 as AuditLogHandler
    participant H4 as WebSocketHandler

    API->>CMD: Create Command
    CMD->>Handler: Execute
    Handler->>Event: Emit Event
    Event->>Bus: EventBus.publish()

    Bus->>H1: 1. AlertEvaluationHandler<br/>Check if metric triggers alerts
    Bus->>H2: 2. AggregationHandler<br/>Update hourly/daily rollups
    Bus->>H3: 3. AuditLogHandler<br/>Log ingestion to ClickHouse
    Bus->>H4: 4. WebSocketHandler optional<br/>Push real-time update to frontend

    Note over Bus,H4: All handlers execute asynchronously<br/>in parallel
```

### Event Types

**Domain Events:**
- UserCreated, UserUpdated, UserDeleted
- OrganizationCreated, WorkspaceCreated, TenantCreated
- MetricIngested, LogIngested, TraceCreated
- AlertTriggered, AlertResolved
- DashboardCreated, WidgetAdded
- ApiKeyCreated, ApiKeyRotated, ApiKeyRevoked

**Integration Events:**
- NotificationSent (Email, Slack, PagerDuty)
- AggregationCompleted
- DataRetentionApplied
- CacheInvalidated

**System Events:**
- ServerStarted, ServerShutdown
- QueueFull, QueueDrained
- DatabaseConnectionLost, DatabaseConnectionRestored

---

## Database Architecture

### PostgreSQL (Relational Data)

**Purpose:** Store structured metadata requiring ACID transactions

**Schema Organization:**

```mermaid
graph TB
    subgraph "PostgreSQL Database"
        subgraph "IAM Schema - 100-core module"
            IAM[• users, organizations, workspaces, tenants<br/>• roles, permissions, user_roles, role_permissions<br/>• groups, user_groups<br/>• regions]
        end

        subgraph "Auth Schema - 200-auth module"
            Auth[• user_sessions, mfa_secrets, email_verifications<br/>• password_reset_tokens, refresh_tokens]
        end

        subgraph "API Keys Schema - 300-api-keys module"
            APIKeys[• api_keys, api_key_permissions<br/>• api_key_usage_logs]
        end

        subgraph "Alerts Schema - 600-alerts module"
            Alerts[• alert_rules, notification_groups<br/>• alert_rule_notification_groups]
        end

        subgraph "Dashboards Schema - 900-dashboard module"
            Dashboards[• dashboards, widgets, dashboard_templates]
        end

        subgraph "Monitoring Schema - 500-monitoring module"
            Monitoring[• uptime_monitors, agents]
        end

        subgraph "Subscription Schema - 1000-subscription module"
            Subscription[• subscriptions, billing_history]
        end
    end

    style IAM fill:#4ecdc4
    style Auth fill:#45b7d1
    style APIKeys fill:#f9ca24
    style Alerts fill:#ff6b6b
    style Dashboards fill:#6c5ce7
    style Monitoring fill:#e74c3c
    style Subscription fill:#95a5a6
```

**Key Features:**
- TypeORM for migrations and entity management
- Soft deletion with `deleted_at` column
- Audit fields: `created_at`, `updated_at`, `created_by`
- Multi-tenancy via `workspace_id` and `tenant_id` columns
- Indexes on foreign keys and query fields
- Partial indexes for soft-deleted records

### ClickHouse (Time-Series Data)

**Purpose:** Store high-volume telemetry data optimized for analytics

**Schema Organization:**

```mermaid
graph TB
    subgraph "ClickHouse Database"
        subgraph "Telemetry Tables - 400-telemetry module"
            Telem[• metrics - Time-series metrics<br/>• logs - Structured logs<br/>• traces - Distributed traces<br/>• spans - Individual trace spans<br/>• exemplars - Metric-trace correlation]
        end

        subgraph "Aggregation Tables - Pre-computed"
            Agg[• metrics_hourly - Hourly rollups<br/>• metrics_daily - Daily rollups<br/>• service_list_mv - Materialized view of services<br/>• metric_list_mv - Materialized view of metrics]
        end

        subgraph "Monitoring Tables - 500-monitoring module"
            Mon[• uptime_checks - Uptime check results<br/>• agent_heartbeats - Agent health data]
        end

        subgraph "Alert Tables - 600-alerts module"
            Alert[• alert_history - Alert trigger history 90d TTL]
        end

        subgraph "Audit Tables - 800-audit module"
            Audit[• audit_logs - Compliance audit trail]
        end
    end

    style Telem fill:#6c5ce7
    style Agg fill:#4ecdc4
    style Mon fill:#e74c3c
    style Alert fill:#ff6b6b
    style Audit fill:#f9ca24
```

**Key Features:**
- Columnar storage (50-90% compression)
- Partitioning by tenant_id and timestamp
- 20 optimized indexes (bloom filter, minmax, set)
- TTL-based automatic data cleanup
- Materialized views for common queries
- MergeTree engine family
- Horizontal scaling via sharding

### Redis (Cache & Queue)

**Purpose:** Ephemeral data, caching, and job queues

**Key Structure:**

```mermaid
graph TB
    subgraph "Redis Key-Value Store"
        subgraph "Cache Keys - cache module"
            Cache[• cache:metric:get:hash - Metric query cache<br/>• cache:log:get:hash - Log query cache<br/>• cache:trace:get:hash - Trace query cache<br/>• cache:dashboard:id - Dashboard data cache]
        end

        subgraph "Session Keys - 200-auth module"
            Session[• session:user:userId - Active sessions<br/>• session:token:token - Token validation]
        end

        subgraph "BullMQ Queue Keys - queue module"
            Queue[• bull:otlp:* - OTLP ingestion queue<br/>• bull:alerts:* - Alert evaluation queue<br/>• bull:aggregation:* - Data aggregation queue<br/>• bull:cleanup:* - Data retention queue<br/>• bull:notifications:* - Notification delivery queue]
        end
    end

    style Cache fill:#4ecdc4
    style Session fill:#45b7d1
    style Queue fill:#f9ca24
```

---

## Caching Strategy

### Multi-Level Cache (L1 + L2)

```mermaid
sequenceDiagram
    participant Request as HTTP Request
    participant L1 as L1 Cache In-Memory<br/>• TTL: 60s<br/>• Hit rate: 40-50%<br/>• Access: 0.1ms
    participant L2 as L2 Cache Redis<br/>• TTL: 30min<br/>• Hit rate: 20-30%<br/>• Access: 1-5ms
    participant DB as Database<br/>PostgreSQL/ClickHouse<br/>• Access: 10-500ms

    Request->>L1: Query Data

    alt L1 Cache Hit
        L1-->>Request: Return Data ⚡<br/>Instant response 0.1ms
    else L1 Cache Miss
        L1->>L2: Query L2 Cache

        alt L2 Cache Hit
            L2-->>L1: Return Data<br/>Fast response 1-5ms
            L1->>L1: Populate L1 Cache
            L1-->>Request: Return Data
        else L2 Cache Miss
            L2->>DB: Query Database<br/>Slower response 10-500ms
            DB-->>L2: Return Fresh Data
            L2->>L2: Populate L2 Cache
            L2-->>L1: Return Data
            L1->>L1: Populate L1 Cache
            L1-->>Request: Return Data
        end
    end

    Note over L1,DB: Cache Hit Rates:<br/>• L1 Hit: 40-50% → Instant<br/>• L2 Hit: 20-30% → Fast<br/>• Database: 20-30% → Slower<br/>• Combined: 60-80%
```

### Cache Invalidation Strategies

1. **TTL-Based:** Automatic expiration after TTL
2. **Event-Based:** Invalidate on domain events (e.g., MetricIngested)
3. **Pattern-Based:** Invalidate all keys matching pattern (e.g., `cache:metric:*`)
4. **Manual:** Admin endpoint `/api/v2/admin/cache/invalidate`

---

## Queue Architecture

### BullMQ Job Queues

```mermaid
sequenceDiagram
    participant API as API Endpoint<br/>POST /otlp/metrics
    participant Redis as Redis Job Queue<br/>Queue: otlp-ingestion
    participant Worker as BullMQ Worker Process<br/>Separate thread/process
    participant CH as ClickHouse
    participant Bus as Domain Event Bus

    API->>Redis: Add Job<br/>Metrics payload
    Note over Redis: Job 1: Metrics payload<br/>Job 2: Metrics payload<br/>Job 3: Metrics payload<br/>...

    Redis->>Worker: Worker pulls job
    Worker->>Worker: 1. Validate metrics
    Worker->>Worker: 2. Transform to domain model
    Worker->>CH: 3. Save to ClickHouse
    CH-->>Worker: Saved
    Worker->>Worker: 4. Emit domain event
    Worker->>Redis: 5. Mark job complete

    Worker->>Bus: Publish Domain Event
    Bus->>Bus: • Alert evaluation<br/>• Aggregation update<br/>• Audit logging

    Note over API,Bus: Async processing prevents<br/>HTTP timeout on heavy operations
```

### Queue Configuration

| Queue | Purpose | Concurrency | Priority | Retry |
|-------|---------|-------------|----------|-------|
| **otlp-ingestion** | Metrics/logs/traces ingestion | 10 workers | High | 3 attempts |
| **alerts** | Alert rule evaluation | 5 workers | High | 5 attempts |
| **aggregation** | Hourly/daily rollups | 2 workers | Medium | 3 attempts |
| **cleanup** | Data retention enforcement | 1 worker | Low | 3 attempts |
| **notifications** | Multi-channel delivery | 5 workers | High | 5 attempts |

**Benefits:**
- **Async Processing:** Don't block API responses
- **Retry Logic:** Auto-retry failed jobs with exponential backoff
- **Rate Limiting:** Prevent overwhelming downstream systems
- **Prioritization:** Critical jobs processed first
- **Monitoring:** Admin endpoint for queue stats

---

## Scalability Considerations

### Horizontal Scaling

```mermaid
graph TB
    LB[Load Balancer<br/>Nginx/HAProxy]

    subgraph "Application Layer - Horizontally Scaled"
        N1[NestJS Instance 1]
        N2[NestJS Instance 2]
        N3[NestJS Instance 3]
    end

    subgraph "Data Layer - Distributed"
        PG[(PostgreSQL<br/>Primary + Replicas)]
        CH[(ClickHouse<br/>Sharded)]
        RD[(Redis<br/>Cluster)]
    end

    LB --> N1
    LB --> N2
    LB --> N3

    N1 --> PG
    N1 --> CH
    N1 --> RD

    N2 --> PG
    N2 --> CH
    N2 --> RD

    N3 --> PG
    N3 --> CH
    N3 --> RD

    style LB fill:#4ecdc4
    style N1 fill:#45b7d1
    style N2 fill:#45b7d1
    style N3 fill:#45b7d1
    style PG fill:#95a5a6
    style CH fill:#6c5ce7
    style RD fill:#e74c3c
```

**Scaling Strategies:**
1. **API Layer:** Add more NestJS instances behind load balancer
2. **Database:** PostgreSQL read replicas, ClickHouse sharding
3. **Cache:** Redis Cluster for distributed caching
4. **Queue:** BullMQ workers scale independently
5. **Frontend:** CDN + multiple origin servers

### Performance Targets

| Metric | Target | Current |
|--------|--------|---------|
| API Response Time (p95) | < 200ms | ~150ms |
| OTLP Ingestion Throughput | 100k metrics/sec | 80k metrics/sec |
| Dashboard Load Time | < 1s | ~800ms |
| Cache Hit Rate | 70-80% | 60-80% |
| Query Performance (ClickHouse) | < 100ms (p95) | ~80ms |
| Concurrent Users | 1000+ | Tested up to 500 |

---

## Security Architecture

See [04-SECURITY.md](./04-SECURITY.md) for detailed security architecture.

**Key Security Layers:**
1. **Network Layer:** HTTPS, WSS, CORS
2. **Authentication:** JWT, MFA, SSO, API keys
3. **Authorization:** RBAC, Permission-based access
4. **Data Layer:** Encryption at rest, TLS in transit
5. **Audit:** Complete audit trail in ClickHouse

---

**Next:** [02-DATA-FLOW.md](./02-DATA-FLOW.md) - Detailed data flow documentation
