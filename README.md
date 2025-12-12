# TelemetryFlow Platform - Overview Documentation

<div align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://github.com/telemetryflow/.github/blob/main/docs/assets/tfo-logo-dark.svg">
    <source media="(prefers-color-scheme: light)" srcset="https://github.com/telemetryflow/.github/blob/main/docs/assets/tfo-logo-light.svg">
    <img src="https://github.com/telemetryflow/.github/blob/main/docs/assets/tfo-logo-light.svg" alt="TelemetryFlow Logo" width="80%">
  </picture>

  <h3>Enterprise-Grade Observability Platform for Modern Cloud Infrastructure</h3>

  <p>
    <strong>100% OpenTelemetry Compliant</strong> • Built with <strong>DDD/CQRS</strong> • Production-Ready
  </p>

  [![Version](https://img.shields.io/badge/version-3.10.0-blue.svg)](../CHANGELOG.md)
  [![License](https://img.shields.io/badge/license-Apache--2.0-green.svg)](../LICENSE)
  [![NestJS](https://img.shields.io/badge/NestJS-10.x-E0234E?logo=nestjs)](https://nestjs.com/)
  [![Vue](https://img.shields.io/badge/Vue-3.x-4FC08D?logo=vue.js)](https://vuejs.org/)
  [![TypeScript](https://img.shields.io/badge/TypeScript-5.7+-3178C6?logo=typescript)](https://www.typescriptlang.org/)
  [![ClickHouse](https://img.shields.io/badge/ClickHouse-23+-FFCC00?logo=clickhouse)](https://clickhouse.com/)
  [![OpenTelemetry](https://img.shields.io/badge/OTLP-100%25%20Compliant-success?logo=opentelemetry)](https://opentelemetry.io/)
  [![DDD](https://img.shields.io/badge/Architecture-DDD%2FCQRS-blueviolet)](docs-ddd-backend/)
  [![RBAC](https://img.shields.io/badge/Security-5--Tier%20RBAC-red)](../backend/src/modules/iam/)

</div>

- **Version:** 3.10.0
- **Status:** Production Ready
- **License:** Apache 2.0
- **Built by:** DevOpsCorner Indonesia
- **Last Updated:** December 12, 2025

---

## Table of Contents

1. [What is TelemetryFlow?](#what-is-telemetryflow)
2. [Documentation Structure](#documentation-structure)
3. [Quick Start](#quick-start)
4. [Key Features](#key-features)
5. [Architecture Overview](#architecture-overview)
6. [Technology Stack](#technology-stack)
7. [Module Overview](#module-overview)
8. [Contributing](#contributing)

---

## What is TelemetryFlow?

**TelemetryFlow** is an **enterprise-grade observability platform** that provides complete telemetry collection, storage, and visualization capabilities. It is a **100% OpenTelemetry Protocol (OTLP) compliant** platform designed to be an open-source alternative to commercial observability solutions like Datadog, New Relic, or Dynatrace.

### Problem It Solves

| Problem | TelemetryFlow Solution |
|---------|----------------------|
| **Fragmented Observability** | Organizations use separate tools for metrics (Prometheus), logs (ELK), and traces (Jaeger). TelemetryFlow unifies all three signals in one platform. |
| **Vendor Lock-in** | By being OTLP-compliant, it works with any OpenTelemetry SDK or Collector, providing vendor-neutral observability. |
| **Multi-Tenancy Complexity** | Provides hierarchical tenant isolation (Region → Organization → Workspace → Tenant) with automatic data segregation. |
| **High Cost** | Open-source platform that can be self-hosted, eliminating per-GB pricing of commercial solutions. |
| **Compliance Requirements** | Built-in audit logging, GDPR compliance, regional data segregation, and soft deletion for compliance. |

### Core Capabilities

- **📊 Unified Telemetry Collection** - Metrics, Logs, and Traces in one platform
- **🔌 100% OTLP Compliant** - Works with any OpenTelemetry SDK
- **🏢 Enterprise Multi-Tenancy** - Hierarchical isolation with Region → Org → Workspace → Tenant
- **🚨 Advanced Alerting** - 33 production-ready alert rules with fatigue prevention
- **📈 Real-time Dashboards** - 6 pre-configured templates with 12+ widget types
- **🔐 Enterprise Security** - JWT, MFA, SSO (Google/GitHub/Azure/Okta), RBAC, API keys
- **⚡ High Performance** - Multi-level caching, queue-based processing, ClickHouse optimization
- **📋 Compliance Ready** - Audit logging, GDPR, SOC2, HIPAA support

---

## Documentation Structure

This documentation is organized into the following sections:

```
.
├── README.md                          # This file - Platform overview
├── architecture/
│   ├── 01-SYSTEM-ARCHITECTURE.md      # High-level system architecture
│   ├── 02-DATA-FLOW.md                # How data flows through the system
│   ├── 03-MULTI-TENANCY.md            # Multi-tenancy architecture
│   ├── 04-SECURITY.md                 # Security architecture
│   └── 05-PERFORMANCE.md              # Performance optimizations
├── backend/
│   ├── 00-BACKEND-OVERVIEW.md         # Backend architecture overview
│   ├── 01-TECH-STACK.md               # Technology stack details
│   ├── 02-DDD-CQRS.md                 # Domain-Driven Design & CQRS patterns
│   ├── 03-MODULE-STRUCTURE.md         # Standard module structure
│   ├── modules/
│   │   ├── 100-core.md                # Core IAM module
│   │   ├── 200-auth.md                # Authentication module
│   │   ├── 300-api-keys.md            # API keys module
│   │   ├── 400-telemetry.md           # Telemetry ingestion module
│   │   ├── 500-monitoring.md          # Uptime monitoring module
│   │   ├── 600-alerts.md              # Alerting module
│   │   ├── 700-sso.md                 # Single Sign-On module
│   │   ├── 800-audit.md               # Audit logging module
│   │   ├── 900-dashboard.md           # Dashboard module
│   │   ├── 1000-subscription.md       # Subscription module
│   │   ├── 1100-agents.md             # Agent management module
│   │   ├── 1200-status-page.md        # Status page module
│   │   ├── 1300-export.md             # Data export module
│   │   ├── 1400-query-builder.md      # Query builder module
│   │   └── 1500-retention-policy.md   # Retention policy module
│   └── shared/
│       ├── logger.md                  # Logger module
│       ├── cache.md                   # Cache module
│       ├── queue.md                   # Queue module
│       ├── messaging.md               # Messaging module
│       ├── email.md                   # Email module
│       └── cors.md                    # CORS module
├── frontend/
│   ├── 00-FRONTEND-OVERVIEW.md        # Frontend architecture overview
│   ├── 01-TECH-STACK.md               # Vue 3, Vite, TypeScript
│   ├── 02-MODULE-STRUCTURE.md         # Frontend module organization
│   ├── 03-STATE-MANAGEMENT.md         # Pinia stores and composition
│   ├── 04-ROUTING.md                  # Vue Router configuration
│   └── 05-VISUALIZATION.md            # ECharts integration
├── shared/
│   ├── API-REFERENCE.md               # REST API documentation
│   ├── OTLP-INGESTION.md              # OTLP ingestion guide
│   ├── DATABASE-SCHEMA.md             # PostgreSQL + ClickHouse schemas
│   └── NAMING-CONVENTIONS.md          # Coding standards
└── deployment/
    ├── DOCKER-COMPOSE.md              # Docker deployment
    ├── KUBERNETES.md                  # Kubernetes deployment
    ├── CONFIGURATION.md               # Environment configuration
    └── PRODUCTION-CHECKLIST.md        # Production deployment guide
```

---

## Quick Start

### Prerequisites

- **Node.js** 18+ (20.x recommended)
- **PostgreSQL** 15+
- **ClickHouse** 23+
- **Redis** 7+
- **Docker** & **Docker Compose** (for local development)

### Local Development Setup

```bash
# 1. Clone the repository
git clone https://github.com/telemetryflow/telemetryflow-platform.git
cd telemetryflow-platform

# 2. Start infrastructure services (PostgreSQL, ClickHouse, Redis)
cd backend
pnpm docker:up

# 3. Install backend dependencies
pnpm install

# 4. Run database migrations
pnpm migration:run

# 5. Start backend development server
pnpm dev

# 6. In a new terminal, start frontend
cd ../frontend
pnpm install
pnpm dev
```

### Access the Platform

- **Frontend:** http://localhost:5173
- **Backend API:** http://localhost:3100/api/v2
- **API Documentation:** http://localhost:3100/api/docs
- **Health Check:** http://localhost:3100/health

### Default Credentials

```bash
# Super Administrator
Email: super.administrator@telemetryflow.id
Password: SuperAdmin@123456

# Administrator
Email: admin.telemetryflow@telemetryflow.id
Password: Admin@123456

# Developer
Email: developer.telemetryflow@telemetryflow.id
Password: Developer@123456

# Viewer
Email: viewer.telemetryflow@telemetryflow.id
Password: Viewer@123456

# Demo
Email: demo.telemetryflow@telemetryflow.id
Password: Demo@123456
```

> References:
> [06-RBAC-SYSTEM-PLATFORM.md](./architecture/06-RBAC-SYSTEM-PLATFORM.md)

---

## Key Features

### 1. Unified Telemetry Collection (OTLP Compliant)

**Metrics**
- Time-series storage in ClickHouse
- Support for: Gauges, Counters, Histograms, Summaries
- Exemplars for metric-trace correlation
- Pre-aggregation tables for 50-90% query speedup
- Custom aggregation functions (sum, avg, min, max, percentiles)

**Logs**
- Structured logging with full-text search
- Severity levels: DEBUG, INFO, WARN, ERROR, FATAL
- Trace context propagation (traceId, spanId)
- Real-time log streaming via WebSocket
- High-cardinality attribute indexing

**Traces**
- Distributed tracing with span visualization
- Service dependency mapping
- Critical path analysis
- Trace-log correlation
- Span attribute search

**OTLP Endpoints**
```
POST /api/v2/otlp/metrics   # Ingest OTLP metrics
POST /api/v2/otlp/logs      # Ingest OTLP logs
POST /api/v2/otlp/traces    # Ingest OTLP traces
```

**OTLP Ingestion Flow:**

```mermaid
sequenceDiagram
    participant CLIENT as OTEL Client
    participant OTLP as OTLP Controller
    participant AUTH as API Key Auth
    participant TRANS as Transformer
    participant QUEUE as BullMQ Queue
    participant WORKER as Queue Worker
    participant CH as ClickHouse

    CLIENT->>OTLP: POST /v1/metrics
    OTLP->>AUTH: Validate API Key
    AUTH->>AUTH: Check Argon2id Hash

    alt Valid API Key
        AUTH-->>OTLP: Authorized
        OTLP->>TRANS: Transform OTLP
        TRANS->>TRANS: Extract attributes
        TRANS->>QUEUE: Enqueue Job
        QUEUE-->>OTLP: Job ID
        OTLP-->>CLIENT: 200 OK

        QUEUE->>WORKER: Process Job
        WORKER->>WORKER: Batch 10K rows
        WORKER->>CH: INSERT telemetry
        CH-->>WORKER: Success
        WORKER->>QUEUE: Complete Job
    else Invalid API Key
        AUTH-->>CLIENT: 401 Unauthorized
    end

    Note over WORKER,CH: Async processing
```

### 2. Multi-Tenancy Architecture

**Hierarchical Isolation:**

```mermaid
graph TD
    REGION[Region<br/>Geographic Isolation<br/>us-east, eu-west, ap-south<br/>GDPR Compliance]

    REGION --> ORG1[Organization 1<br/>Company A<br/>Multi-org Support]
    REGION --> ORG2[Organization 2<br/>Company B]

    ORG1 --> WS1[Workspace 1<br/>Team: Backend<br/>Logical Separation]
    ORG1 --> WS2[Workspace 2<br/>Team: Frontend]

    WS1 --> T1[Tenant: Production<br/>Environment Level<br/>Data Isolation]
    WS1 --> T2[Tenant: Staging]
    WS1 --> T3[Tenant: Development]

    WS2 --> T4[Tenant: Production]
    WS2 --> T5[Tenant: Development]

    style REGION fill:#FF6B6B
    style ORG1 fill:#4ECDC4
    style ORG2 fill:#4ECDC4
    style WS1 fill:#45B7D1
    style WS2 fill:#45B7D1
    style T1 fill:#96CEB4
    style T2 fill:#96CEB4
    style T3 fill:#96CEB4
    style T4 fill:#96CEB4
    style T5 fill:#96CEB4
```

**Features:**
- Automatic tenant context injection
- All queries filtered by workspace_id and tenant_id
- ClickHouse partitioning by tenant
- Cross-tenant data isolation guaranteed
- Resource quotas per workspace
- Regional data segregation for compliance

### 3. Authentication & Security

**5-Tier RBAC System:**
- **Super Administrator** - Global platform management
- **Administrator** - Organization-level management
- **Developer** - Write access to telemetry
- **Viewer** - Read-only dashboard access
- **Demo** - Limited demo access

**API Key Authentication (OTLP):**
- AWS-style dual-key system (tfk-*/tfs-*)
- Argon2id hashing (OWASP-recommended)
- Permission-based access: `metrics:write`, `logs:write`, `traces:write`
- Automatic key rotation with zero-downtime
- Rate limiting: 1000 req/min per key

**Authentication Methods:**
- JWT with refresh tokens
- Multi-Factor Authentication (TOTP)
- SSO providers: Google, GitHub, Azure AD, Okta
- SAML 2.0 and OIDC support

**Authentication Flow:**

```mermaid
sequenceDiagram
    participant USER as User
    participant FE as Frontend
    participant API as Backend API
    participant GUARD as Auth Guard
    participant JWT as JWT Service
    participant DB as PostgreSQL
    participant MFA as MFA Service

    USER->>FE: Login
    FE->>API: POST /auth/login
    API->>DB: Find user by email
    DB-->>API: User found

    alt Password Valid
        API->>API: Verify Argon2id

        alt MFA Enabled
            API->>MFA: Generate TOTP
            MFA-->>USER: Send MFA code
            USER->>FE: Enter MFA code
            FE->>API: POST /auth/verify-mfa
            API->>MFA: Validate TOTP

            alt MFA Valid
                MFA-->>API: Valid
                API->>JWT: Generate tokens
                JWT-->>API: access + refresh
                API-->>FE: 200 + tokens
                FE->>FE: Store tokens
                FE-->>USER: Logged in
            else MFA Invalid
                MFA-->>USER: 401 Invalid MFA
            end
        else No MFA
            API->>JWT: Generate tokens
            JWT-->>API: access + refresh
            API-->>FE: 200 + tokens
            FE-->>USER: Logged in
        end
    else Password Invalid
        API-->>USER: 401 Invalid credentials
    end

    Note over FE,API: Subsequent requests
    USER->>FE: Access dashboard
    FE->>API: GET /api/v1/metrics
    API->>GUARD: Validate JWT
    GUARD->>GUARD: Verify signature
    GUARD->>GUARD: Check expiration
    GUARD-->>API: Valid
    API-->>FE: 200 + data
```

### 4. Advanced Alerting

**33 Default Production-Ready Rules:**
- Kubernetes (pod crashes, OOMKilled, restarts)
- VMs (CPU, memory, disk, network)
- Redis (memory usage, evictions, slowlog)
- Load Balancers (5xx errors, target health)
- Databases (connection pool, slow queries)

**Fatigue Prevention:**
- Cooldown periods (5-60 minutes)
- Rate limiting (max 10 alerts/hour per rule)
- Deduplication (fingerprint-based)
- Auto-resolution after conditions clear

**Notification Channels (8):**
- Email, Slack, PagerDuty, Webhook, Microsoft Teams, Discord, Telegram, SMS (Twilio)

### 5. Dashboards & Visualization

**6 Pre-configured Templates:**
1. **System Monitoring** - CPU, memory, disk, network
2. **Application Performance Monitoring (APM)** - Response times, throughput, errors, traces
3. **Logs Explorer** - Advanced log filtering and analysis
4. **Infrastructure Monitoring** - Containers, Kubernetes, cloud resources
5. **Network Monitoring** - Bandwidth, latency, packet loss
6. **Custom Metrics Dashboard** - Flexible custom metrics

**12+ Widget Types:**
- line_chart, bar_chart, area_chart, pie_chart, donut_chart
- table, gauge, stat, heatmap, graph (network diagram)
- text panel, logs viewer

**Features:**
- Drag-and-drop dashboard builder
- Real-time updates via WebSocket
- Template variables for dynamic parameterization
- Clone from templates
- Export/import dashboards

### 6. Performance Optimizations

**Multi-Level Cache:**
- L1: In-memory cache (60s TTL)
- L2: Redis cache (30min TTL)
- 60-80% cache hit rate

**Message Queues:**
- 5 BullMQ queues: OTLP, Alerts, Aggregation, Cleanup, Notifications
- Async processing with retries
- Job prioritization

**Database Optimizations:**
- 20 ClickHouse indexes (bloom filter, minmax, set)
- 10-50x faster searches
- Partitioning by tenant and timestamp
- Data compression (50-90% space savings)

---

## Architecture Overview

### High-Level Architecture

```mermaid
graph TB
    subgraph "External Clients"
        OTEL[OpenTelemetry SDK/Collector<br/>gRPC/HTTP]
        BROWSER[Web Browser]
        SSO[SSO Providers<br/>Google/GitHub/Azure/Okta]
    end

    subgraph "TelemetryFlow Platform"
        subgraph "Frontend Layer"
            FE[Frontend<br/>Vue 3 + Vite + TypeScript<br/>Port 3101]
        end

        subgraph "Backend Layer"
            API[Backend API<br/>NestJS + TypeScript<br/>Port 3100]
            OTLP_EP[OTLP Endpoint<br/>gRPC Port 4317]
        end

        subgraph "Data Layer"
            PG[(PostgreSQL 15+<br/>Metadata & RBAC<br/>Users, Tenants, Configs)]
            CH[(ClickHouse 23+<br/>Time-Series Telemetry<br/>Metrics, Logs, Traces)]
            REDIS[(Redis 7+<br/>Cache & Queue<br/>L2 Cache, BullMQ)]
        end

        subgraph "Processing Layer"
            QUEUE[BullMQ Queues<br/>OTLP, Alerts, Aggregation]
            NATS[NATS<br/>Event Streaming<br/>Real-time Events]
        end

        subgraph "Integration Layer"
            EMAIL[Email Service<br/>Nodemailer]
            NOTIF[Notification Services<br/>Slack, PagerDuty, Teams]
        end
    end

    BROWSER -->|HTTPS| FE
    FE -->|REST API| API
    OTEL -->|OTLP/gRPC| OTLP_EP
    SSO -.->|OAuth2/OIDC| API

    API -->|Queries| PG
    API -->|Telemetry Queries| CH
    API -->|Cache/Queue| REDIS
    OTLP_EP -->|Enqueue| QUEUE

    QUEUE -->|Process| CH
    QUEUE -->|Alert Check| API
    API -->|Publish Events| NATS

    API -->|Send Email| EMAIL
    API -->|Send Notification| NOTIF

    style FE fill:#42b983
    style API fill:#e34c26
    style PG fill:#336791
    style CH fill:#ffcc01
    style REDIS fill:#d82c20
    style QUEUE fill:#cf1f1f
    style NATS fill:#27aae1
```

### Backend Architecture (DDD + CQRS)

```mermaid
graph TD
    subgraph "Presentation Layer"
        CTRL[Controllers HTTP Endpoints]
        DTO[DTOs Request Response]
        GUARD[Guards Auth RBAC]
        DEC[Decorators CurrentUser TenantContext]
        INT[Interceptors Logging Transform]
    end

    subgraph "Application Layer CQRS"
        CMD[Commands CreateUser IngestMetric]
        QRY[Queries GetUser QueryMetrics]
        HANDLER[Handlers Business Logic]
        SVC[Services Application Services]
        EVENT[Event Bus Domain Events]
    end

    subgraph "Domain Layer DDD"
        AGG[Aggregates User Tenant Metric]
        VO[Value Objects UserId TenantId Email]
        DEVT[Domain Events UserCreated MetricIngested]
        PORT[Repository Ports Interfaces]
    end

    subgraph "Infrastructure Layer"
        ORM[TypeORM PostgreSQL]
        CH_SVC[ClickHouse Client Time-Series DB]
        CACHE[Redis Client L2 Cache]
        QUEUE_SVC[BullMQ Job Queues]
        EXT[External APIs Email SSO Notifications]
    end

    CTRL --> DTO
    CTRL --> GUARD
    CTRL --> DEC
    CTRL --> INT
    CTRL --> CMD
    CTRL --> QRY

    CMD --> HANDLER
    QRY --> HANDLER
    HANDLER --> SVC
    HANDLER --> EVENT

    SVC --> AGG
    SVC --> VO
    SVC --> DEVT
    SVC --> PORT

    PORT -.implements.-> ORM
    PORT -.implements.-> CH_SVC
    PORT -.implements.-> CACHE
    PORT -.implements.-> QUEUE_SVC
    PORT -.implements.-> EXT

    EVENT --> QUEUE_SVC

    style CTRL fill:#4CAF50
    style CMD fill:#2196F3
    style QRY fill:#2196F3
    style AGG fill:#FF9800
    style ORM fill:#336791
    style CH_SVC fill:#ffcc01
```

### Deployment Architecture

```mermaid
graph TB
    subgraph "Production Environment"
        subgraph "Load Balancer Layer"
            LB[Load Balancer<br/>Nginx/HAProxy<br/>SSL Termination]
        end

        subgraph "Application Layer"
            FE1[Frontend Instance 1<br/>Nginx + Vue 3]
            FE2[Frontend Instance 2<br/>Nginx + Vue 3]
            API1[Backend Instance 1<br/>NestJS]
            API2[Backend Instance 2<br/>NestJS]
            API3[Backend Instance 3<br/>NestJS]
        end

        subgraph "Data Layer"
            PG_PRIMARY[(PostgreSQL Primary<br/>Write/Read)]
            PG_REPLICA[(PostgreSQL Replica<br/>Read Only)]
            CH_CLUSTER[(ClickHouse Cluster<br/>3 Shards, 2 Replicas)]
            REDIS_CLUSTER[(Redis Cluster<br/>3 Masters, 3 Replicas)]
        end

        subgraph "Message Layer"
            NATS_CLUSTER[NATS Cluster<br/>3 Nodes]
        end

        subgraph "Monitoring Layer"
            PROM[Prometheus]
            GRAF[Grafana]
            ALERT[Alertmanager]
        end
    end

    subgraph "External Services"
        S3[S3/Object Storage<br/>Backups]
        EMAIL_SVC[Email Service<br/>SendGrid/SES]
        SSO_SVC[SSO Providers<br/>Google/GitHub/Okta]
    end

    LB -->|Round Robin| FE1
    LB -->|Round Robin| FE2
    LB -->|Round Robin| API1
    LB -->|Round Robin| API2
    LB -->|Round Robin| API3

    FE1 --> API1
    FE2 --> API2

    API1 --> PG_PRIMARY
    API2 --> PG_REPLICA
    API3 --> PG_PRIMARY

    API1 --> CH_CLUSTER
    API2 --> CH_CLUSTER
    API3 --> CH_CLUSTER

    API1 --> REDIS_CLUSTER
    API2 --> REDIS_CLUSTER
    API3 --> REDIS_CLUSTER

    API1 --> NATS_CLUSTER
    API2 --> NATS_CLUSTER
    API3 --> NATS_CLUSTER

    PG_PRIMARY -.->|Replication| PG_REPLICA
    PG_PRIMARY -->|Backup| S3

    API1 --> EMAIL_SVC
    API1 --> SSO_SVC

    PROM -->|Scrape| API1
    PROM -->|Scrape| API2
    PROM -->|Scrape| API3
    GRAF --> PROM
    PROM --> ALERT

    style LB fill:#FF6B6B
    style FE1 fill:#42b983
    style FE2 fill:#42b983
    style API1 fill:#e34c26
    style API2 fill:#e34c26
    style API3 fill:#e34c26
    style PG_PRIMARY fill:#336791
    style CH_CLUSTER fill:#ffcc01
    style REDIS_CLUSTER fill:#d82c20
```

For detailed architecture documentation, see:
- [System Architecture](./architecture/01-SYSTEM-ARCHITECTURE.md)
- [Data Flow](./architecture/02-DATA-FLOW.md)
- [Multi-Tenancy](./architecture/03-MULTI-TENANCY.md)
- [Security](./architecture/04-SECURITY.md)
- [Performance](./architecture/05-PERFORMANCE.md)

---

## Technology Stack

### Backend

| Category | Technology | Version | Purpose |
|----------|-----------|---------|---------|
| **Framework** | NestJS | 10.x | Enterprise Node.js framework |
| **Language** | TypeScript | 5.7+ | Type-safe development |
| **Runtime** | Node.js | 18-20.x | JavaScript runtime |
| **Metadata DB** | PostgreSQL | 15+ | Relational data storage |
| **Telemetry DB** | ClickHouse | 23+ | Time-series data storage |
| **Cache & Queue** | Redis | 7+ | Caching and job queues |
| **ORM** | TypeORM | 0.3.x | PostgreSQL migrations |
| **Queue** | BullMQ | 5.x | Async job processing |
| **Messaging** | NATS | 2.x | Event streaming (optional) |
| **Telemetry** | OpenTelemetry | 0.208+ | Self-instrumentation |
| **Auth** | Passport JWT | Latest | Authentication |
| **Validation** | class-validator | Latest | DTO validation |
| **Hashing** | Argon2 | Latest | Password hashing |

### Frontend

| Category | Technology | Version | Purpose |
|----------|-----------|---------|---------|
| **Framework** | Vue | 3.5.24 | Progressive JavaScript framework |
| **Build Tool** | Vite | 7.2.4 | Lightning-fast HMR |
| **Language** | TypeScript | 5.8.3 | Type-safe development |
| **UI Library** | Naive UI | 2.43.2 | Vue 3 component library |
| **CSS Engine** | UnoCSS | 66.5.9 | Atomic CSS |
| **State** | Pinia | 3.0.4 | Vue 3 state management |
| **Router** | Vue Router | 4.6.3 | Official Vue router |
| **Charts** | ECharts | 6.0.0 | 80+ chart types |
| **HTTP Client** | Axios | 1.13.2 | REST API calls |
| **WebSocket** | Socket.IO | 4.8.1 | Real-time updates |

---

## Module Overview

### Module Dependencies

```mermaid
graph LR
    subgraph "Shared Modules (Foundation)"
        LOGGER[Logger]
        CACHE[Cache]
        QUEUE[Queue]
        MSG[Messaging]
        EMAIL[Email]
        CORS[CORS]
        CH[ClickHouse]
        OTEL[OpenTelemetry]
    end

    subgraph "Core Modules"
        CORE[100-Core<br/>IAM, Multi-tenancy]
        AUTH[200-Auth<br/>JWT, MFA, Sessions]
        APIKEY[300-API Keys<br/>OTLP Auth]
        SSO[700-SSO<br/>OAuth Providers]
    end

    subgraph "Telemetry Modules"
        TELEM[400-Telemetry<br/>OTLP Ingestion]
        MONITOR[500-Monitoring<br/>Uptime Checks]
        ALERT[600-Alerts<br/>Alert Rules]
    end

    subgraph "Dashboard Modules"
        DASH[900-Dashboard<br/>Templates & Widgets]
        QUERY[1400-Query Builder<br/>Visual Queries]
        EXPORT[1300-Export<br/>CSV, JSON, Parquet]
    end

    subgraph "Management Modules"
        AUDIT[800-Audit<br/>Compliance Logging]
        SUB[1000-Subscription<br/>Billing]
        AGENT[1100-Agents<br/>Agent Management]
        STATUS[1200-Status Page<br/>Public Status]
        RETENTION[1500-Retention<br/>Data Lifecycle]
    end

    CORE --> LOGGER
    CORE --> CACHE
    CORE --> QUEUE

    AUTH --> CORE
    AUTH --> EMAIL
    AUTH --> CACHE

    APIKEY --> CORE
    APIKEY --> CACHE

    SSO --> AUTH
    SSO --> CORE

    TELEM --> APIKEY
    TELEM --> QUEUE
    TELEM --> CH
    TELEM --> OTEL

    MONITOR --> CORE
    MONITOR --> ALERT
    MONITOR --> QUEUE

    ALERT --> CORE
    ALERT --> EMAIL
    ALERT --> MSG
    ALERT --> QUEUE

    DASH --> CORE
    DASH --> CH
    DASH --> CACHE

    QUERY --> CORE
    QUERY --> CH
    QUERY --> CACHE

    EXPORT --> CORE
    EXPORT --> CH
    EXPORT --> QUEUE

    AUDIT --> CORE
    AUDIT --> LOGGER
    AUDIT --> QUEUE

    SUB --> CORE
    SUB --> EMAIL

    AGENT --> CORE
    AGENT --> MONITOR

    STATUS --> CORE
    STATUS --> MONITOR

    RETENTION --> CORE
    RETENTION --> CH
    RETENTION --> QUEUE

    style CORE fill:#4CAF50
    style AUTH fill:#2196F3
    style TELEM fill:#FF9800
    style DASH fill:#9C27B0
    style LOGGER fill:#607D8B
    style CACHE fill:#607D8B
    style QUEUE fill:#607D8B
    style CH fill:#ffcc01
```

### Backend Modules (15)

| Module | Name | Purpose | Status |
|--------|------|---------|--------|
| **100** | Core | IAM, Multi-tenancy, RBAC | ✅ Production |
| **200** | Auth | Authentication, JWT, MFA | ✅ Production |
| **300** | API Keys | OTLP API key authentication | ✅ Production |
| **400** | Telemetry | OTLP ingestion (metrics/logs/traces) | ✅ Production |
| **500** | Monitoring | Uptime monitors, agent management | ✅ Production |
| **600** | Alerts | Alert rules, notifications, history | ✅ Production |
| **700** | SSO | Google, GitHub, Azure, Okta | ✅ Production |
| **800** | Audit | Audit logging, compliance | ✅ Production |
| **900** | Dashboard | Dashboard templates, widgets | ✅ Production |
| **1000** | Subscription | Billing, subscription management | ✅ Production |
| **1100** | Agents | Agent deployment, heartbeat | ✅ Production |
| **1200** | Status Page | Public/private status pages | ✅ Production |
| **1300** | Export | Data export (CSV, JSON, Parquet) | ✅ Production |
| **1400** | Query Builder | Visual query builder | ✅ Production |
| **1500** | Retention Policy | Data retention management | ✅ Production |

### Shared Modules (10)

| Module | Purpose | Key Features |
|--------|---------|--------------|
| **logger** | Logging service | Winston, multiple transports, trace propagation |
| **cache** | Multi-level cache | L1 (memory) + L2 (Redis), 60-80% hit rate |
| **queue** | Job queue | BullMQ, 5 queues, async processing |
| **messaging** | Event streaming | NATS pub/sub, optional |
| **email** | Email service | Nodemailer, Handlebars templates |
| **cors** | CORS configuration | Database-driven validation |
| **clickhouse** | ClickHouse service | Shared ClickHouse client |
| **otel** | OpenTelemetry | Self-instrumentation |
| **ui** | Web UI | EJS templates, static assets |
| **platform** | Platform utilities | Common helpers |

---

## Contributing

### Development Workflow

1. **Fork the repository**
2. **Create a feature branch** - `git checkout -b feature/my-feature`
3. **Follow naming conventions** - See [NAMING-CONVENTIONS.md](./shared/NAMING-CONVENTIONS.md)
4. **Write tests** - Maintain 88%+ coverage
5. **Run linting** - `pnpm lint`
6. **Build successfully** - `pnpm build` (must show 0 errors)
7. **Submit pull request**

### Coding Standards

- **Backend:** NestJS best practices, DDD patterns, CQRS
- **Frontend:** Vue 3 Composition API, TypeScript strict mode
- **Testing:** Unit tests (88%+), integration tests, e2e tests
- **Documentation:** JSDoc for all public APIs
- **Commit Messages:** Conventional Commits format

### Module Development Guide

When creating a new module, follow the standard structure:

```
{module-number}-{module-name}/
├── {module-number}-{module-name}.module.ts
├── README.md
├── application/
│   ├── commands/
│   ├── queries/
│   ├── handlers/
│   ├── dto/
│   └── services/
├── domain/
│   ├── aggregates/
│   ├── entities/
│   ├── events/
│   ├── repositories/
│   ├── services/
│   └── value-objects/
├── infrastructure/
│   ├── persistence/
│   │   ├── postgres/
│   │   └── clickhouse/
│   ├── messaging/
│   └── services/
├── presentation/
│   ├── controllers/
│   ├── dto/
│   ├── guards/
│   └── decorators/
└── tests/
    ├── unit/
    ├── integration/
    └── e2e/
```

---

## Support & Resources

### Documentation

- **Architecture:** [./architecture/](./architecture/)
- **Backend:** [./backend/](./backend/)
- **Frontend:** [./frontend/](./frontend/)
- **API Reference:** [./shared/API-REFERENCE.md](./shared/API-REFERENCE.md)
- **Deployment:** [./deployment/](./deployment/)

### Community

- **GitHub:** https://github.com/telemetryflow/telemetryflow-platform
- **Issues:** https://github.com/telemetryflow/telemetryflow-platform/issues
- **Discussions:** https://github.com/telemetryflow/telemetryflow-platform/discussions

### Statistics

| Metric | Count |
|--------|-------|
| Backend Modules | 15 |
| Frontend Modules | 5+ |
| CQRS Handlers | 40+ |
| API Endpoints | 120+ |
| Database Tables | 50+ |
| Lines of Code | 110,000+ |
| Test Cases | 280+ |
| Test Coverage | 88-92% |
| Documentation Pages | 203+ |
| Version | 3.10.0 |

---

## License

Apache License 2.0 - See [LICENSE](./LICENSE) for details.

---

## Acknowledgments

Built with ❤️ by **DevOpsCorner Indonesia**

- **Status:** ✅ Production Ready (Zero Build Errors)
- **Last Updated:** December 12, 2025
