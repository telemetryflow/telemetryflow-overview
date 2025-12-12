# Data Flow Architecture

- **Version:** 1.0.0-CE
- **Last Updated:** December 12, 2025
- **Status:** ✅ Production Ready

---

## Table of Contents

1. [Overview](#overview)
2. [OTLP Ingestion Flow](#otlp-ingestion-flow)
3. [Query Flow](#query-flow)
4. [Authentication Flow](#authentication-flow)
5. [Multi-Tenant Data Isolation](#multi-tenant-data-isolation)
6. [Caching Strategy](#caching-strategy)
7. [Queue Processing](#queue-processing)
8. [WebSocket Real-Time Updates](#websocket-real-time-updates)

---

## Overview

TelemetryFlow implements a comprehensive data flow architecture that handles:
- **Ingestion**: 15,000+ OTLP metrics/logs/traces per second
- **Storage**: Dual database strategy (PostgreSQL + ClickHouse)
- **Caching**: Multi-level (L1 in-memory + L2 Redis)
- **Async Processing**: BullMQ with 5 specialized queues
- **Real-time**: WebSocket for live dashboard updates

```mermaid
graph TB
    subgraph "External Sources"
        A[OTLP Clients]
        B[User Browser]
        C[SSO Providers]
    end

    subgraph "Ingress Layer"
        D[API Gateway / Load Balancer]
    end

    subgraph "Backend Services"
        E[NestJS Backend]
        F[Authentication Service]
        G[OTLP Controller]
        H[Query Service]
    end

    subgraph "Data Processing"
        I[BullMQ Workers]
        J[Cache Service]
    end

    subgraph "Data Storage"
        K[PostgreSQL<br/>Metadata]
        L[ClickHouse<br/>Telemetry]
        M[Redis<br/>Cache & Queue]
    end

    A -->|OTLP gRPC/HTTP| D
    B -->|HTTPS| D
    C -->|OAuth2/OIDC| D

    D --> E
    E --> F
    E --> G
    E --> H

    F --> K
    G --> I
    H --> J

    J --> L
    J --> K
    I --> L

    J --> M
    I --> M
```

---

## OTLP Ingestion Flow

### Complete Ingestion Pipeline

```mermaid
sequenceDiagram
    participant Client as OTLP Client<br/>(Collector/SDK)
    participant API as OTLP Controller
    participant Guard as ApiKeyAuthGuard
    participant Queue as BullMQ Queue
    participant Worker as OTLP Processor
    participant DB as ClickHouse
    participant Cache as Redis Cache

    Client->>API: POST /api/v2/otlp/metrics<br/>Headers: X-API-Key-ID, X-API-Key-Secret
    API->>Guard: Validate API Key
    Guard->>Cache: Check cache for API key
    Cache-->>Guard: Cache miss or hit

    alt Cache Miss
        Guard->>DB: Query PostgreSQL for API key
        DB-->>Guard: API key + permissions
        Guard->>Cache: Store in cache (5min TTL)
    end

    Guard->>Guard: Verify Argon2id hash
    Guard->>Guard: Check permissions (metrics:write)
    Guard->>Guard: Check rate limit (1000 req/min)
    Guard-->>API: ✅ Authorized + Rate limit info

    API->>API: Extract tenant context<br/>(headers or OTLP attributes)
    API->>API: Transform OTLP to internal format<br/>(parse resourceMetrics, convert timestamps)
    API->>Queue: Add job to OTLP_INGESTION queue
    Queue-->>API: Job ID
    API-->>Client: 200 OK + Job ID<br/>(async response)

    Queue->>Worker: Job picked up (10 concurrent)
    Worker->>Worker: Parse job data<br/>(metrics/logs/traces)
    Worker->>DB: Bulk insert into ClickHouse<br/>(JSONEachRow format, async insert)
    DB-->>Worker: Inserted count
    Worker-->>Queue: Job complete
```

### OTLP Transformation Process

```mermaid
graph LR
    subgraph "OTLP Input Format"
        A1[resourceMetrics]
        A2[resourceLogs]
        A3[resourceSpans]
    end

    subgraph "Transformation Layer"
        B1[transformOtlpMetrics]
        B2[transformOtlpLogs]
        B3[transformOtlpTraces]
    end

    subgraph "Internal Format"
        C1[Simple Metrics<br/>gauge, counter]
        C2[Histogram Metrics<br/>with buckets]
        C3[Log Records<br/>with trace context]
        C4[Spans<br/>with events & links]
    end

    subgraph "Queue Jobs"
        D1[OTLP_INGESTION Job]
    end

    subgraph "ClickHouse Tables"
        E1[metrics_v3]
        E2[metric_histograms]
        E3[logs_v3]
        E4[traces_v3]
        E5[spans_v3]
    end

    A1 --> B1
    A2 --> B2
    A3 --> B3

    B1 --> C1
    B1 --> C2
    B2 --> C3
    B3 --> C4

    C1 --> D1
    C2 --> D1
    C3 --> D1
    C4 --> D1

    D1 --> E1
    D1 --> E2
    D1 --> E3
    D1 --> E4
    D1 --> E5
```

### Data Transformation Details

**Metrics Transformation:**
```typescript
// OTLP Protocol
resourceMetrics[i].scopeMetrics[j].metrics[k].sum.dataPoints[m]
  → {
      timestamp: nanoToIso(dataPoint.timeUnixNano),
      metric_name: metric.name,
      value: dataPoint.asDouble || dataPoint.asInt,
      temporality: sum.aggregationTemporality (delta/cumulative),
      workspace_id: extractedContext.workspaceId,
      tenant_id: extractedContext.tenantId,
      exemplar_trace_id: dataPoint.exemplars[0]?.traceId,
      resource_attributes: { "service.name": "myapp", ... },
      attributes: { "method": "GET", ... }
    }
```

**Logs Transformation:**
```typescript
// OTLP Protocol
resourceLogs[i].scopeLogs[j].logRecords[k]
  → {
      timestamp: nanoToIso(logRecord.timeUnixNano),
      observed_timestamp: nanoToIso(logRecord.observedTimeUnixNano),
      severity_text: logRecord.severityText,
      severity_number: logRecord.severityNumber,
      body: extractBodyFrom(logRecord.body),  // stringValue, intValue, etc.
      trace_id: logRecord.traceId,
      span_id: logRecord.spanId,
      trace_flags: logRecord.flags,
      workspace_id: extractedContext.workspaceId,
      tenant_id: extractedContext.tenantId,
      service_name: resource.attributes["service.name"],
      log_attributes: { "http.status_code": 200, ... }
    }
```

**Traces Transformation:**
```typescript
// OTLP Protocol
resourceSpans[i].scopeSpans[j].spans[k]
  → {
      timestamp: nanoToIso(span.startTimeUnixNano),
      trace_id: span.traceId,
      span_id: span.spanId,
      parent_span_id: span.parentSpanId,
      span_name: span.name,
      span_kind: span.kind,  // CLIENT, SERVER, PRODUCER, CONSUMER
      duration_nano: span.endTimeUnixNano - span.startTimeUnixNano,
      status_code: span.status.code,
      workspace_id: extractedContext.workspaceId,
      tenant_id: extractedContext.tenantId,
      events: span.events.map(...),  // Nested events
      links: span.links.map(...),    // Nested links
      span_attributes: { "http.method": "GET", ... }
    }
```

---

## Query Flow

### Dashboard Query Execution

```mermaid
sequenceDiagram
    participant User as User Browser
    participant API as Query Controller
    participant Auth as JwtAuthGuard
    participant Cache as Cache Service
    participant CQRS as QueryBus
    participant Handler as GetMetrics Handler
    participant Repo as MetricRepository
    participant CH as ClickHouse

    User->>API: GET /api/v2/telemetry/metrics<br/>Authorization: Bearer <jwt>
    API->>Auth: Validate JWT
    Auth->>Auth: Verify signature & expiry
    Auth->>Cache: Check token blacklist
    Cache-->>Auth: Not blacklisted
    Auth->>Auth: Extract tenant context<br/>(from JWT payload)
    Auth-->>API: ✅ Authorized + Tenant context

    API->>Cache: Check query cache<br/>Key: metrics:{tenantId}:{queryHash}

    alt Cache Hit (L1 or L2)
        Cache-->>API: Cached result
        API-->>User: 200 OK + Metrics data<br/>(X-Cache: HIT)
    else Cache Miss
        API->>CQRS: Dispatch GetMetricsByFiltersQuery
        CQRS->>Handler: Execute query
        Handler->>Handler: Apply tenant filter<br/>(workspace_id, tenant_id)
        Handler->>Repo: findByFilters(filters)
        Repo->>CH: SELECT * FROM metrics_v3<br/>WHERE tenant_id = ? AND timestamp >= ?<br/>ORDER BY timestamp DESC
        CH-->>Repo: Query results (with indexes)
        Repo-->>Handler: Domain models
        Handler->>Handler: Map to DTOs
        Handler-->>CQRS: DTO array
        CQRS-->>API: Query result
        API->>Cache: Store result (30min TTL)
        API-->>User: 200 OK + Metrics data<br/>(X-Cache: MISS)
    end
```

### ClickHouse Query Optimization Flow

```mermaid
graph TD
    A[Query Request] --> B{Check Cache}
    B -->|HIT| C[Return Cached]
    B -->|MISS| D[Build ClickHouse Query]

    D --> E[Apply Filters]
    E --> E1[Tenant: workspace_id, tenant_id]
    E --> E2[Time Range: timestamp >= ? AND timestamp <= ?]
    E --> E3[Service: service_name = ?]
    E --> E4[Metric Name: metric_name = ?]

    E1 --> F[Index Selection]
    E2 --> F
    E3 --> F
    E4 --> F

    F --> F1[Bloom Filter<br/>tenant_id, service_name, metric_name]
    F --> F2[MinMax Index<br/>timestamp]
    F --> F3[Partition Pruning<br/>toYYYYMMDD timestamp]

    F1 --> G[Execute Query]
    F2 --> G
    F3 --> G

    G --> H{Use Materialized View?}
    H -->|Yes| I[mv_service_list<br/>mv_metric_list<br/>100x faster]
    H -->|No| J[Raw Table Query]

    I --> K[Return Results]
    J --> K
    K --> L[Cache Result 30min]
    L --> M[Return to Client]
```

---

## Authentication Flow

### JWT Authentication with MFA

```mermaid
sequenceDiagram
    participant User as User Browser
    participant API as Auth Controller
    participant Service as Auth Service
    participant Lockout as Account Lockout
    participant MFA as MFA Service
    participant DB as PostgreSQL
    participant Cache as Redis

    User->>API: POST /api/v2/auth/login<br/>{email, password}
    API->>Service: validateUser(email, password)
    Service->>Lockout: isAccountLocked(userId)
    Lockout->>Cache: Check lock status (Redis)
    Cache-->>Lockout: Not locked
    Lockout-->>Service: Continue

    Service->>DB: Find user by email
    DB-->>Service: User entity
    Service->>Service: Compare password<br/>(bcrypt, 10 rounds)

    alt Password Incorrect
        Service->>Lockout: recordFailedAttempt(userId)
        Lockout->>Lockout: Increment failed_attempts
        alt failed_attempts >= 5
            Lockout->>DB: Set locked_until (15min)
            Lockout->>Cache: Set lock in Redis
            Lockout-->>Service: Account Locked
            Service-->>API: 403 Forbidden
            API-->>User: Account locked for 15 minutes
        else failed_attempts < 5
            Service-->>API: 401 Unauthorized
            API-->>User: Invalid credentials<br/>(4 attempts remaining)
        end
    else Password Correct
        Service->>Lockout: clearFailedAttempts(userId)
        Lockout->>DB: Clear attempts

        alt MFA Not Enabled
            Service->>Service: Generate JWT tokens
            Service->>Cache: Store session (Redis)
            Service-->>API: Access + Refresh tokens
            API-->>User: 200 OK + Tokens
        else MFA Enabled
            Service->>Service: Generate temp token (5min)
            Service-->>API: Temp token + mfaRequired: true
            API-->>User: 200 OK + Temp token + MFA prompt

            User->>API: POST /api/v2/auth/validate-mfa<br/>{tempToken, mfaCode}
            API->>Service: validateMfaAndLogin(tempToken, code)
            Service->>Service: Verify temp token signature
            Service->>MFA: validateTotp(secret, code)
            MFA->>MFA: speakeasy.verify()<br/>(30s window, ±1 step tolerance)

            alt TOTP Valid
                MFA-->>Service: ✅ Valid
                Service->>Service: Generate full access token
                Service->>Cache: Store session
                Service-->>API: Access + Refresh tokens
                API-->>User: 200 OK + Tokens
            else TOTP Invalid
                MFA-->>Service: ❌ Invalid
                Service-->>API: 401 Unauthorized
                API-->>User: Invalid MFA code
            end
        end
    end
```

### SSO Authentication Flow (OAuth2/OIDC)

```mermaid
sequenceDiagram
    participant User as User Browser
    participant Frontend as Frontend
    participant Backend as SSO Controller
    participant DB as PostgreSQL
    participant Provider as SSO Provider<br/>(Google/GitHub/Azure)

    User->>Frontend: Click "Login with Google"
    Frontend->>Backend: GET /api/v2/sso/google/login
    Backend->>DB: Fetch SSO config<br/>(client_id, scopes)
    DB-->>Backend: SSO config
    Backend->>Backend: Generate authorization URL<br/>+ state token (CSRF protection)
    Backend-->>Frontend: Redirect URL
    Frontend-->>Provider: Redirect to OAuth consent screen

    User->>Provider: Approve permissions
    Provider-->>Frontend: Redirect to callback URL<br/>/api/v2/sso/google/callback<br/>?code=xxx&state=yyy

    Frontend->>Backend: GET /api/v2/sso/google/callback<br/>?code=xxx&state=yyy
    Backend->>Backend: Verify state token (CSRF)
    Backend->>Provider: Exchange code for access token<br/>POST /oauth/token
    Provider-->>Backend: Access token + ID token
    Backend->>Provider: Fetch user profile<br/>GET /userinfo
    Provider-->>Backend: User profile {email, name, picture}

    Backend->>DB: Find or create user<br/>(by email)
    DB-->>Backend: User entity
    Backend->>Backend: Generate JWT tokens
    Backend->>Backend: Store session (Redis)
    Backend-->>Frontend: Redirect to app<br/>with tokens in query/cookie
    Frontend-->>User: Logged in successfully
```

---

## Multi-Tenant Data Isolation

### Tenant Context Propagation

```mermaid
graph TB
    A[HTTP Request] --> B{Authentication Type}

    B -->|JWT| C[JWT Guard]
    B -->|API Key| D[ApiKey Guard]
    B -->|Session| E[Session Guard]

    C --> F[Extract from JWT Payload]
    F --> F1[payload.workspaceId]
    F --> F2[payload.tenantId]

    D --> G[Extract from Headers/OTLP]
    G --> G1[X-Workspace-Id header]
    G --> G2[X-Tenant-Id header]
    G --> G3[telemetryflow.workspace.id attribute]
    G --> G4[telemetryflow.tenant.id attribute]

    E --> H[Extract from Session]
    H --> H1[session.workspaceId]
    H --> H2[session.tenantId]

    F1 --> I[TenantContext VO]
    F2 --> I
    G1 --> I
    G2 --> I
    G3 --> I
    G4 --> I
    H1 --> I
    H2 --> I

    I --> J[CurrentTenantContext Decorator]
    J --> K[Controller Method]
    K --> L[CQRS Command/Query]
    L --> M[Repository Layer]

    M --> N[PostgreSQL Query]
    N --> N1[WHERE tenant_id]

    M --> O[ClickHouse Query]
    O --> O1[WHERE workspace_id AND tenant_id]

    N1 --> P[Results Scoped to Tenant]
    O1 --> P
```

### Database-Level Tenant Filtering

**PostgreSQL** (Metadata):
```sql
-- User query (automatically filtered)
SELECT * FROM users
WHERE tenant_id = $1  -- Injected by repository
  AND organization_id = $2
  AND is_deleted = FALSE;

-- Workspace query
SELECT * FROM workspaces
WHERE organization_id = $1;

-- Permission query
SELECT p.* FROM permissions p
  INNER JOIN role_permissions rp ON p.permission_id = rp.permission_id
  INNER JOIN user_roles ur ON rp.role_id = ur.role_id
WHERE ur.user_id = $1
  AND ur.tenant_id = $2;  -- Tenant-scoped RBAC
```

**ClickHouse** (Telemetry):
```sql
-- Metrics query (tenant-filtered with indexes)
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
GROUP BY service_name, metric_name, hour
ORDER BY hour DESC;
-- Indexes: bloom filter on workspace_id, tenant_id
--          minmax on timestamp
--          partition pruning on toYYYYMMDD(timestamp)
```

---

## Caching Strategy

### Multi-Level Cache Architecture

```mermaid
graph TD
    A[Query Request] --> B{Check L1 Cache<br/>in-memory, 60s TTL}

    B -->|HIT| C[Return from L1<br/>sub-millisecond latency]

    B -->|MISS| D{Check L2 Cache<br/>Redis, 30min TTL}

    D -->|HIT| E[Return from L2<br/>1-5ms latency]
    E --> F[Populate L1 Cache]

    D -->|MISS| G[Execute Database Query<br/>ClickHouse/PostgreSQL]
    G --> H[Query Result<br/>50-500ms latency]
    H --> I[Populate L2 Cache]
    I --> J[Populate L1 Cache]
    J --> K[Return to Client]

    F --> C

    style C fill:#90EE90
    style E fill:#FFD700
    style H fill:#FFA07A
```

### Cache Invalidation Strategy

```mermaid
sequenceDiagram
    participant User as User Action
    participant API as API Controller
    participant Service as Domain Service
    participant Cache as Cache Service
    participant DB as Database

    User->>API: POST /api/v2/dashboards<br/>(Create dashboard)
    API->>Service: CreateDashboardCommand
    Service->>DB: INSERT INTO dashboards
    DB-->>Service: Dashboard created

    Service->>Cache: Invalidate pattern<br/>dashboard:{tenantId}:*
    Cache->>Cache: L2: SCAN dashboard:tenant123:*<br/>DEL matching keys
    Cache->>Cache: L1: Clear matching entries
    Cache-->>Service: Cache cleared

    Service-->>API: Dashboard DTO
    API-->>User: 201 Created

    Note over Cache: Future GET requests will fetch fresh data
```

### Cache Key Strategy

| Resource | L1 TTL | L2 TTL | Key Pattern | Invalidation Trigger |
|----------|--------|--------|-------------|----------------------|
| **Metrics** | 1min | 30min | `metrics:{tenant}:{query_hash}` | New data ingested (every 10min) |
| **Logs** | 1min | 5min | `logs:{tenant}:{query_hash}` | New data ingested (every 1min) |
| **Traces** | 1min | 30min | `trace:{tenant}:{traceId}` | New spans ingested |
| **Dashboards** | 5min | 1hr | `dashboard:{tenant}:{dashboardId}` | Dashboard updated |
| **Users** | 5min | 30min | `user:{userId}` | User profile updated |
| **Permissions** | 5min | 5min | `rbac:permissions:{userId}` | Role/permission changed |
| **API Keys** | 1min | 5min | `apikey:{keyId}` | API key rotated/revoked |

---

## Queue Processing

### BullMQ Queue Architecture

```mermaid
graph TB
    subgraph "Controllers"
        A1[OTLP Controller]
        A2[Alert Controller]
        A3[Cron Jobs]
    end

    subgraph "Queue Service"
        B1[OTLP_INGESTION<br/>Priority: HIGH]
        B2[ALERT_EVALUATION<br/>Priority: HIGH]
        B3[AGGREGATION<br/>Priority: MEDIUM]
        B4[CLEANUP<br/>Priority: LOW]
        B5[NOTIFICATION<br/>Priority: MEDIUM]
    end

    subgraph "Workers 10 concurrent"
        C1[OTLP Processor]
        C2[Alert Processor]
        C3[Aggregation Processor]
        C4[Cleanup Processor]
        C5[Notification Processor]
    end

    subgraph "Backends"
        D1[ClickHouse]
        D2[PostgreSQL]
        D3[Redis]
        D4[External APIs<br/>Slack, Email, Webhook]
    end

    A1 --> B1
    A2 --> B2
    A3 --> B3
    A3 --> B4

    B1 --> C1
    B2 --> C2
    B3 --> C3
    B4 --> C4
    B2 --> C5

    C1 --> D1
    C2 --> D1
    C2 --> D2
    C3 --> D1
    C4 --> D1
    C5 --> D4

    D3 -.->|Queue Backend| B1
    D3 -.->|Queue Backend| B2
    D3 -.->|Queue Backend| B3
    D3 -.->|Queue Backend| B4
    D3 -.->|Queue Backend| B5
```

### Job Retry Strategy

```mermaid
graph LR
    A[Job Added to Queue] --> B{Execute Job}

    B -->|Success| C[Mark Complete<br/>Remove after 1hr]

    B -->|Failure| D{Attempts < Max?}

    D -->|Yes| E[Calculate Backoff<br/>Exponential: attempt^2 * base]
    E --> F[Wait Backoff Time]
    F --> B

    D -->|No| G[Mark Failed<br/>Keep 24-48hrs<br/>Send alert]

    C --> H[Job Complete]
    G --> I[Manual Retry or<br/>Dead Letter Queue]
```

**Backoff Timing Examples:**

| Queue | Attempt 1 | Attempt 2 | Attempt 3 | Attempt 4 | Attempt 5 |
|-------|-----------|-----------|-----------|-----------|-----------|
| **OTLP** | 1s | 4s | 9s | - | - |
| **Alert** | 2s | 8s | 18s | - | - |
| **Aggregation** | 5s | 20s | 45s | - | - |
| **Cleanup** | 10s | 40s | - | - | - |
| **Notification** | 3s | 12s | 27s | 48s | 75s |

---

## WebSocket Real-Time Updates

### WebSocket Event Flow

```mermaid
sequenceDiagram
    participant Browser as Frontend
    participant Gateway as Telemetry Gateway<br/>(WebSocket)
    participant Worker as OTLP Worker
    participant CH as ClickHouse
    participant Cache as Redis Pub/Sub

    Browser->>Gateway: Connect WebSocket<br/>auth: JWT token
    Gateway->>Gateway: Validate JWT
    Gateway->>Gateway: Join room: workspace:{id}:tenant:{id}
    Gateway-->>Browser: Connected

    Worker->>CH: Insert metrics batch
    CH-->>Worker: Inserted 1000 metrics
    Worker->>Cache: PUBLISH metrics:new<br/>{tenantId, count}
    Cache->>Gateway: Subscriber receives event
    Gateway->>Gateway: Filter by tenant/workspace
    Gateway-->>Browser: emit('metrics:new', data)
    Browser->>Browser: Update dashboard in real-time

    Note over Browser,Gateway: Live updates without polling
```

### WebSocket Event Types

| Event | Direction | Payload | Frequency |
|-------|-----------|---------|-----------|
| **metrics:new** | Server → Client | `{count, timestamp, services[]}` | Every 10s (batch) |
| **logs:new** | Server → Client | `{logs[], count}` | Real-time |
| **alert:triggered** | Server → Client | `{ruleId, severity, message}` | Real-time |
| **dashboard:updated** | Server → Client | `{dashboardId}` | On update |
| **subscribe:dashboard** | Client → Server | `{dashboardId}` | On dashboard open |
| **unsubscribe:dashboard** | Client → Server | `{dashboardId}` | On dashboard close |

---

## Performance Metrics

| Flow | Target Latency | Actual (p95) | Cache Hit Rate |
|------|----------------|--------------|----------------|
| **OTLP Ingestion** | < 200ms | 150ms | N/A (async) |
| **Query (cached)** | < 10ms | 5ms | 75% (L1+L2) |
| **Query (uncached)** | < 500ms | 300ms | 25% |
| **Authentication** | < 100ms | 80ms | 60% (permissions) |
| **WebSocket Latency** | < 50ms | 30ms | N/A |
| **Queue Processing** | < 2s | 1.5s | N/A |

---

## Data Retention and Cleanup

```mermaid
graph LR
    A[Cron Job<br/>Daily at 2 AM] --> B[CLEANUP Queue]
    B --> C[Cleanup Processor]

    C --> D{Data Type}

    D -->|Metrics| E1[Delete > 90 days<br/>ALTER TABLE metrics_v3 DELETE]
    D -->|Logs| E2[Delete > 30 days<br/>ALTER TABLE logs_v3 DELETE]
    D -->|Traces| E3[Delete > 30 days<br/>ALTER TABLE traces_v3 DELETE]
    D -->|Audit Logs| E4[Delete > 365 days<br/>PostgreSQL]

    E1 --> F[ClickHouse Mutation]
    E2 --> F
    E3 --> F
    E4 --> G[PostgreSQL DELETE]

    F --> H[Background Processing<br/>Low priority]
    G --> H
    H --> I[Cleanup Complete<br/>Log metrics]
```

---

## Next Steps

- [Multi-Tenancy Architecture](./03-MULTI-TENANCY.md)
- [Security Architecture](./04-SECURITY.md)
- [Performance Optimizations](./05-PERFORMANCE.md)
- [Backend Tech Stack](../backend/01-TECH-STACK.md)

---

- **Last Updated:** December 12, 2025
- **Maintained By:** DevOpsCorner Indonesia
