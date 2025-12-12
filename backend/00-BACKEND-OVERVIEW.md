# Backend Architecture Overview

- **Version:** 1.0.0-CE
- **Last Updated:** December 12, 2025
- **Status:** ✅ Complete

---

## Table of Contents

1. [Introduction](#introduction)
2. [Technology Stack](#technology-stack)
3. [Architecture Principles](#architecture-principles)
4. [Module Organization](#module-organization)
5. [Shared Modules](#shared-modules)
6. [Layer Architecture](#layer-architecture)
7. [Module Structure (LEGO Pattern)](#module-structure-lego-pattern)
8. [Module Dependencies](#module-dependencies)
9. [Getting Started](#getting-started)

---

## Introduction

The TelemetryFlow backend is built as a **modular monolith** using **NestJS 11.x**, implementing **Domain-Driven Design (DDD)** and **CQRS** patterns. The architecture is designed for:

- **Modularity:** 15 bounded contexts (business modules) + 10 shared modules
- **Scalability:** Each module can be extracted into a microservice
- **Maintainability:** Clear separation of concerns with 4-layer architecture
- **Observability:** Built-in OpenTelemetry instrumentation
- **Multi-Tenancy:** Tenant isolation at database and application layers

### Key Characteristics

```mermaid
graph LR
    subgraph "Backend Characteristics"
        A[Modular Monolith<br/>15 Business Modules]
        B[Domain-Driven Design<br/>DDD + CQRS]
        C[Event-Driven<br/>EventBus + BullMQ]
        D[Multi-Tenant<br/>Workspace + Tenant Isolation]
    end

    subgraph "Benefits"
        E[Easy to Scale<br/>Extract to Microservices]
        F[High Cohesion<br/>Low Coupling]
        G[Testable<br/>Unit + Integration Tests]
        H[Auditable<br/>Event Sourcing]
    end

    A --> E
    B --> F
    C --> G
    D --> H

    style A fill:#4ecdc4
    style B fill:#45b7d1
    style C fill:#f9ca24
    style D fill:#ff6b6b
```

---

## Technology Stack

### Core Framework

```mermaid
graph TB
    subgraph "Runtime & Framework"
        A[Node.js 18-20.x Runtime]
        B[NestJS 11.x Framework]
        C[TypeScript 5.9.3]
    end

    subgraph "ORM & Database Clients"
        D[TypeORM 0.3.x PostgreSQL]
        E[ClickHouse Client 1.x]
        F[ioredis 5.x Redis]
    end

    subgraph "Observability"
        G[OpenTelemetry API 1.x]
        H[Winston 3.x Logging]
        I[BullMQ 5.x Queue]
    end

    A --> B
    B --> C
    B --> D
    B --> E
    B --> F
    B --> G
    G --> H
    B --> I

    style B fill:#4ecdc4
    style C fill:#45b7d1
    style G fill:#f9ca24
```

### Key Dependencies

| Category | Technology | Version | Purpose |
|----------|------------|---------|---------|
| **Framework** | NestJS | 11.x | Enterprise application framework |
| **Language** | TypeScript | 5.9.3 | Type-safe development |
| **Runtime** | Node.js | 18-20.x | JavaScript runtime |
| **PostgreSQL ORM** | TypeORM | 0.3.x | Relational data persistence |
| **ClickHouse** | @clickhouse/client | 1.x | Time-series data client |
| **Redis** | ioredis | 5.x | Cache & queue backend |
| **Queue** | BullMQ | 5.x | Async job processing |
| **Validation** | class-validator | 0.14.x | DTO validation |
| **Transformation** | class-transformer | 0.5.x | DTO transformation |
| **CQRS** | @nestjs/cqrs | 11.x | Command/Query pattern |
| **Scheduling** | @nestjs/schedule | 4.x | Cron jobs |
| **Rate Limiting** | @nestjs/throttler | 6.x | API rate limiting |
| **Authentication** | @nestjs/jwt | 11.x | JWT token management |
| **Passport** | @nestjs/passport | 11.x | Auth strategies |
| **Logging** | winston | 3.x | Structured logging |
| **Telemetry** | @opentelemetry/api | 1.x | Observability |

---

## Architecture Principles

### 1. Modular Monolith Architecture

```mermaid
graph TB
    subgraph "Application Structure"
        App[App Module<br/>Root Module]

        subgraph "15 Business Modules - Bounded Contexts"
            M1[100-core<br/>IAM & Multi-Tenancy]
            M2[200-auth<br/>Authentication]
            M3[300-api-keys<br/>API Keys]
            M4[400-telemetry<br/>OTLP Ingestion]
            M5[500-monitoring<br/>Uptime Monitors]
            M6[600-alerts<br/>Alert Rules]
            M7[700-sso<br/>Single Sign-On]
            M8[800-audit<br/>Audit Logging]
            M9[900-dashboard<br/>Dashboards]
            M10[1000-subscription<br/>Billing]
            M11[1100-agents<br/>Agents]
            M12[1200-status-page<br/>Status Pages]
            M13[1300-export<br/>Data Export]
            M14[1400-query-builder<br/>Visual Query Builder]
            M15[1500-retention-policy<br/>Data Retention]
        end

        subgraph "10 Shared Modules - Infrastructure"
            S1[logger<br/>Winston Logging]
            S2[cache<br/>Multi-Level Cache]
            S3[queue<br/>BullMQ Queues]
            S4[clickhouse<br/>ClickHouse Client]
            S5[email<br/>Email Service]
            S6[cors<br/>CORS Config]
            S7[otel<br/>OpenTelemetry]
            S8[ui<br/>UI Templates]
            S9[platform<br/>Platform Utils]
            S10[messaging<br/>NATS Events]
        end
    end

    App --> M1
    App --> M2
    App --> M3
    App --> M4
    App --> M5
    App --> M6
    App --> M7
    App --> M8
    App --> M9
    App --> M10
    App --> M11
    App --> M12
    App --> M13
    App --> M14
    App --> M15

    M1 -.-> S1
    M2 -.-> S2
    M4 -.-> S3
    M4 -.-> S4
    M6 -.-> S5
    M8 -.-> S7

    style App fill:#4ecdc4
    style M1 fill:#45b7d1
    style M4 fill:#ff6b6b
    style M6 fill:#f9ca24
    style S2 fill:#6c5ce7
    style S3 fill:#e74c3c
```

### 2. Domain-Driven Design (DDD)

**Core Concepts:**

```mermaid
classDiagram
    class Aggregate {
        +AggregateRoot root
        +Entity[] entities
        +ValueObject[] valueObjects
        +validate()
        +apply(DomainEvent)
    }

    class Entity {
        +UUID id
        +created_at
        +updated_at
    }

    class ValueObject {
        +immutable properties
        +equals(other)
    }

    class DomainEvent {
        +UUID eventId
        +timestamp
        +aggregateId
        +payload
    }

    class Repository {
        <<interface>>
        +find(id)
        +save(aggregate)
        +delete(id)
    }

    class DomainService {
        +businessLogic()
    }

    Aggregate *-- Entity
    Aggregate *-- ValueObject
    Aggregate --> DomainEvent
    Repository --> Aggregate
    DomainService --> Aggregate

    style Aggregate fill:#4ecdc4
    style Entity fill:#45b7d1
    style ValueObject fill:#f9ca24
    style DomainEvent fill:#ff6b6b
```

**Example Aggregates:**

| Module | Aggregates | Value Objects |
|--------|-----------|---------------|
| **100-core** | User, Organization, Workspace, Tenant, Role | Email, TenantId, WorkspaceId, UserId |
| **400-telemetry** | Metric, Log, Trace, Span | MetricName, Timestamp, Attributes |
| **600-alerts** | AlertRule, Notification, Incident | Threshold, Channel, Severity |
| **900-dashboard** | Dashboard, Widget | Layout, Query, Visualization |

### 3. CQRS (Command Query Responsibility Segregation)

```mermaid
graph LR
    subgraph "Write Model - Commands"
        C1[Command<br/>IngestMetricsCommand]
        C2[CommandHandler<br/>Business Logic]
        C3[Domain Events<br/>MetricIngested]
        C4[Write to DB<br/>ClickHouse]
    end

    subgraph "Read Model - Queries"
        Q1[Query<br/>GetMetricsByFiltersQuery]
        Q2[QueryHandler<br/>Optimized Reads]
        Q3[Read from DB<br/>+ Cache]
    end

    C1 --> C2
    C2 --> C3
    C3 --> C4

    Q1 --> Q2
    Q2 --> Q3

    C3 -.->|Event triggers| Q2

    style C1 fill:#ff6b6b
    style Q1 fill:#4ecdc4
```

**Benefits:**
- ✅ Separate write and read optimization
- ✅ Independent scaling of commands and queries
- ✅ Event sourcing for audit trails
- ✅ Clearer business logic separation

### 4. Event-Driven Architecture

```mermaid
sequenceDiagram
    participant CMD as Command
    participant Handler as CommandHandler
    participant Domain as Domain Aggregate
    participant Bus as EventBus
    participant H1 as Event Handler 1
    participant H2 as Event Handler 2
    participant H3 as Event Handler 3

    CMD->>Handler: Execute Command
    Handler->>Domain: Apply Business Rules
    Domain->>Domain: Validate & Update State
    Domain->>Bus: Publish Domain Event

    Bus->>H1: Alert Evaluation
    Bus->>H2: Aggregation Update
    Bus->>H3: Audit Logging

    Note over Bus,H3: Handlers execute asynchronously in parallel
```

---

## Module Organization

### Module Numbering Convention

TelemetryFlow uses a **numerical prefix** for module organization:

```mermaid
graph TB
    subgraph "100-299: Core & Auth"
        M100[100-core<br/>Foundation: IAM, Multi-Tenancy]
        M200[200-auth<br/>Authentication & Authorization]
        M300[300-api-keys<br/>API Key Management]
    end

    subgraph "400-699: Business Features"
        M400[400-telemetry<br/>Metrics, Logs, Traces]
        M500[500-monitoring<br/>Uptime & Health Checks]
        M600[600-alerts<br/>Alert Rules & Notifications]
    end

    subgraph "700-999: Integrations"
        M700[700-sso<br/>OAuth/OIDC SSO]
        M800[800-audit<br/>Audit Trail & Compliance]
        M900[900-dashboard<br/>Visualization & Dashboards]
    end

    subgraph "1000+: Advanced Features"
        M1000[1000-subscription<br/>Billing & Plans]
        M1100[1100-agents<br/>Agent Management]
        M1200[1200-status-page<br/>Public Status Pages]
        M1300[1300-export<br/>Data Export]
        M1400[1400-query-builder<br/>Visual Query Builder]
        M1500[1500-retention-policy<br/>Data Lifecycle]
    end

    style M100 fill:#4ecdc4
    style M400 fill:#ff6b6b
    style M600 fill:#f9ca24
    style M900 fill:#6c5ce7
```

### Module Categories

```mermaid
pie title Module Distribution by Category
    "Core & Auth (3)" : 3
    "Telemetry & Monitoring (3)" : 3
    "Integrations (3)" : 3
    "Advanced Features (6)" : 6
```

### Core Module Hierarchy

```mermaid
graph TB
    Core[100-core<br/>IAM & Multi-Tenancy]
    Auth[200-auth<br/>Authentication]
    APIKeys[300-api-keys<br/>API Key Auth]
    Telemetry[400-telemetry<br/>OTLP Ingestion]
    Monitoring[500-monitoring<br/>Uptime Monitors]
    Alerts[600-alerts<br/>Alert Rules]
    Dashboard[900-dashboard<br/>Dashboards]

    Core --> Auth
    Core --> APIKeys
    Auth --> Telemetry
    Auth --> Monitoring
    Auth --> Alerts
    Auth --> Dashboard

    Telemetry --> Alerts
    Telemetry --> Dashboard

    style Core fill:#4ecdc4
    style Auth fill:#45b7d1
    style Telemetry fill:#ff6b6b
    style Alerts fill:#f9ca24
```

---

## Shared Modules

Shared modules provide **infrastructure services** used across all business modules.

```mermaid
graph TB
    subgraph "Shared Infrastructure Modules"
        subgraph "Data Layer"
            CH[clickhouse<br/>ClickHouse Client]
            Cache[cache<br/>L1 + L2 Cache]
            Queue[queue<br/>BullMQ Jobs]
        end

        subgraph "Communication"
            Email[email<br/>Nodemailer]
            Messaging[messaging<br/>NATS Events]
        end

        subgraph "Observability"
            Logger[logger<br/>Winston + OTEL]
            OTEL[otel<br/>OpenTelemetry]
        end

        subgraph "Utilities"
            CORS[cors<br/>CORS Config]
            UI[ui<br/>UI Templates]
            Platform[platform<br/>Platform Utils]
        end
    end

    subgraph "Business Modules"
        M1[400-telemetry]
        M2[600-alerts]
        M3[900-dashboard]
    end

    M1 --> CH
    M1 --> Queue
    M1 --> Logger

    M2 --> Email
    M2 --> Queue
    M2 --> Logger

    M3 --> Cache
    M3 --> Logger

    style CH fill:#6c5ce7
    style Cache fill:#4ecdc4
    style Queue fill:#f9ca24
    style Logger fill:#45b7d1
```

### Shared Module Summary

| Module | Purpose | Key Features |
|--------|---------|--------------|
| **logger** | Structured logging with OTEL | Winston 3.x, JSON format, trace correlation |
| **cache** | Multi-level caching | L1 in-memory (60s), L2 Redis (30min) |
| **queue** | Async job processing | BullMQ, 5 queues, retry strategies |
| **clickhouse** | ClickHouse client | Query builder, connection pooling |
| **email** | Email notifications | Nodemailer, template engine |
| **cors** | CORS configuration | Environment-based CORS rules |
| **otel** | OpenTelemetry instrumentation | Auto-instrumentation, exporters |
| **ui** | UI template rendering | Handlebars templates |
| **platform** | Platform utilities | Common helpers, constants |
| **messaging** | Event streaming | NATS messaging system |

---

## Layer Architecture

TelemetryFlow implements a **4-layer architecture** per module:

```mermaid
graph TB
    subgraph "1. Presentation Layer"
        P1[Controllers<br/>REST Endpoints]
        P2[DTOs<br/>Request/Response Validation]
        P3[Guards<br/>Auth & Permissions]
        P4[Interceptors<br/>Logging & Tracing]
    end

    subgraph "2. Application Layer"
        A1[Commands<br/>Write Operations]
        A2[Queries<br/>Read Operations]
        A3[Handlers<br/>Business Orchestration]
        A4[EventBus<br/>Domain Events]
    end

    subgraph "3. Domain Layer"
        D1[Aggregates<br/>Business Entities]
        D2[Value Objects<br/>Immutable Types]
        D3[Domain Events<br/>State Changes]
        D4[Repository Interfaces<br/>Ports]
    end

    subgraph "4. Infrastructure Layer"
        I1[TypeORM Entities<br/>PostgreSQL]
        I2[ClickHouse Adapters<br/>Time-series]
        I3[Repository Implementations<br/>Adapters]
        I4[External Services<br/>Email, Slack, etc.]
    end

    P1 --> A1
    P1 --> A2
    A1 --> D1
    A2 --> D1
    D1 --> I1
    D1 --> I2
    D4 --> I3

    style P1 fill:#4ecdc4
    style A1 fill:#45b7d1
    style D1 fill:#f9ca24
    style I1 fill:#ff6b6b
```

### Dependency Flow

```mermaid
flowchart TD
    A[Presentation Layer] -->|Depends on| B[Application Layer]
    B -->|Depends on| C[Domain Layer]
    D[Infrastructure Layer] -->|Implements| C

    Note1[⚠️ Domain Layer<br/>is INDEPENDENT<br/>No outer layer dependencies]

    style C fill:#f9ca24
    style Note1 fill:#e74c3c
```

**Key Principle:** Dependencies point **INWARD** toward the domain layer. The domain layer has **zero dependencies** on outer layers.

---

## Module Structure (LEGO Pattern)

Each module follows a **standardized structure** (LEGO pattern) for consistency:

```mermaid
graph TB
    subgraph "Module: 400-telemetry"
        Root[400-telemetry.module.ts<br/>Module Definition]

        subgraph "presentation/"
            Controllers[controllers/<br/>REST Endpoints]
            DTOs[dtos/<br/>Validation Schemas]
            Guards[guards/<br/>Auth Guards]
        end

        subgraph "application/"
            Commands[commands/<br/>Write Operations]
            Queries[queries/<br/>Read Operations]
            Handlers[handlers/<br/>CQRS Handlers]
            Services[services/<br/>Application Services]
        end

        subgraph "domain/"
            Aggregates[aggregates/<br/>Business Entities]
            ValueObjects[value-objects/<br/>Immutable Types]
            Events[events/<br/>Domain Events]
            Repositories[repositories/<br/>Port Interfaces]
        end

        subgraph "infrastructure/"
            Persistence[persistence/<br/>TypeORM + ClickHouse]
            Mappers[mappers/<br/>Entity ↔ Domain]
            Clients[clients/<br/>External Services]
        end
    end

    Root --> Controllers
    Root --> Commands
    Root --> Aggregates
    Root --> Persistence

    style Root fill:#4ecdc4
    style Commands fill:#ff6b6b
    style Aggregates fill:#f9ca24
    style Persistence fill:#6c5ce7
```

### Folder Structure Example

```
400-telemetry/
├── 400-telemetry.module.ts          # Module definition
├── presentation/                     # Layer 1: HTTP/REST
│   ├── controllers/
│   │   ├── metrics.controller.ts
│   │   ├── logs.controller.ts
│   │   └── traces.controller.ts
│   ├── dtos/
│   │   ├── ingest-metrics.dto.ts
│   │   └── query-metrics.dto.ts
│   └── guards/
│       └── api-key-auth.guard.ts
├── application/                      # Layer 2: Orchestration
│   ├── commands/
│   │   └── ingest-metrics.command.ts
│   ├── queries/
│   │   └── get-metrics.query.ts
│   ├── handlers/
│   │   ├── ingest-metrics.handler.ts
│   │   └── get-metrics.handler.ts
│   └── services/
│       └── telemetry.service.ts
├── domain/                           # Layer 3: Business Logic
│   ├── aggregates/
│   │   ├── metric.aggregate.ts
│   │   └── log.aggregate.ts
│   ├── value-objects/
│   │   ├── metric-name.vo.ts
│   │   └── timestamp.vo.ts
│   ├── events/
│   │   └── metric-ingested.event.ts
│   └── repositories/
│       └── metric.repository.interface.ts
└── infrastructure/                   # Layer 4: External Systems
    ├── persistence/
    │   ├── typeorm/
    │   │   └── entities/
    │   └── clickhouse/
    │       ├── schemas/
    │       │   └── metrics.schema.ts
    │       └── repositories/
    │           └── metric.repository.ts
    └── mappers/
        └── metric.mapper.ts
```

---

## Module Dependencies

### Dependency Graph

```mermaid
graph TB
    Core[100-core<br/>IAM Foundation]
    Auth[200-auth<br/>Authentication]
    APIKeys[300-api-keys<br/>API Keys]
    Telemetry[400-telemetry<br/>OTLP Ingestion]
    Monitoring[500-monitoring<br/>Uptime Monitors]
    Alerts[600-alerts<br/>Alert Rules]
    SSO[700-sso<br/>Single Sign-On]
    Audit[800-audit<br/>Audit Logging]
    Dashboard[900-dashboard<br/>Dashboards]
    Sub[1000-subscription<br/>Billing]
    Agents[1100-agents<br/>Agent Management]
    Status[1200-status-page<br/>Status Pages]
    Export[1300-export<br/>Data Export]
    Query[1400-query-builder<br/>Query Builder]
    Retention[1500-retention-policy<br/>Data Retention]

    Core --> Auth
    Core --> APIKeys
    Core --> Sub

    Auth --> Telemetry
    Auth --> Monitoring
    Auth --> Alerts
    Auth --> SSO
    Auth --> Audit
    Auth --> Dashboard
    Auth --> Agents
    Auth --> Status
    Auth --> Export
    Auth --> Query

    Telemetry --> Alerts
    Telemetry --> Dashboard
    Telemetry --> Export
    Telemetry --> Retention

    Monitoring --> Alerts
    Monitoring --> Status

    Alerts --> Status

    style Core fill:#4ecdc4
    style Auth fill:#45b7d1
    style Telemetry fill:#ff6b6b
    style Alerts fill:#f9ca24
    style Dashboard fill:#6c5ce7
```

### Module Communication

**Modules communicate via:**

1. **Domain Events (EventBus)** - Preferred for async communication
2. **Shared Interfaces** - For direct method calls (discouraged)
3. **API Calls** - For cross-boundary communication

```mermaid
sequenceDiagram
    participant M1 as 400-telemetry<br/>Module
    participant Bus as EventBus<br/>NestJS CQRS
    participant M2 as 600-alerts<br/>Module
    participant M3 as 800-audit<br/>Module

    M1->>M1: Ingest Metric
    M1->>Bus: Publish MetricIngested Event

    Bus->>M2: Handle MetricIngested
    M2->>M2: Evaluate Alert Rules

    Bus->>M3: Handle MetricIngested
    M3->>M3: Log Audit Entry

    Note over M1,M3: Loose coupling via events<br/>Modules don't know each other
```

---

## Getting Started

### Prerequisites

- **Node.js** 18.x or 20.x
- **pnpm** 8.x or higher
- **PostgreSQL** 15+
- **ClickHouse** 23+
- **Redis** 7+

### Installation

```bash
# Clone repository
git clone https://github.com/telemetryflow/telemetryflow.git

# Navigate to backend
cd telemetryflow/backend

# Install dependencies
pnpm install

# Copy environment variables
cp .env.example .env

# Run database migrations
pnpm migration:run

# Start development server
pnpm start:dev
```

### Project Scripts

```bash
# Development
pnpm start:dev          # Start dev server with hot reload
pnpm start:debug        # Start with debugging

# Build
pnpm build              # Build for production
pnpm start:prod         # Start production server

# Testing
pnpm test               # Run unit tests
pnpm test:e2e           # Run end-to-end tests
pnpm test:cov           # Generate coverage report

# Database
pnpm migration:generate # Generate new migration
pnpm migration:run      # Run pending migrations
pnpm migration:revert   # Revert last migration

# Linting
pnpm lint               # Run ESLint
pnpm format             # Format code with Prettier
```

### Environment Variables

```env
# Application
NODE_ENV=development
PORT=3000
API_PREFIX=api/v2

# Database - PostgreSQL
DB_HOST=localhost
DB_PORT=5432
DB_USERNAME=postgres
DB_PASSWORD=password
DB_DATABASE=telemetryflow

# Database - ClickHouse
CLICKHOUSE_HOST=localhost
CLICKHOUSE_PORT=8123
CLICKHOUSE_DATABASE=telemetry
CLICKHOUSE_USERNAME=default
CLICKHOUSE_PASSWORD=

# Redis
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_PASSWORD=

# JWT
JWT_SECRET=your-secret-key-change-in-production
JWT_EXPIRATION=15m
REFRESH_TOKEN_EXPIRATION=7d

# Rate Limiting
THROTTLE_TTL=60
THROTTLE_LIMIT=100
THROTTLE_INGESTION_LIMIT=1000

# OpenTelemetry
OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4318
OTEL_SERVICE_NAME=telemetryflow-backend
```

---

## Module Development Guide

### Creating a New Module

```bash
# 1. Create module directory
mkdir -p src/modules/1600-new-feature

# 2. Create LEGO structure
mkdir -p src/modules/1600-new-feature/{presentation,application,domain,infrastructure}
mkdir -p src/modules/1600-new-feature/presentation/{controllers,dtos,guards}
mkdir -p src/modules/1600-new-feature/application/{commands,queries,handlers,services}
mkdir -p src/modules/1600-new-feature/domain/{aggregates,value-objects,events,repositories}
mkdir -p src/modules/1600-new-feature/infrastructure/{persistence,mappers,clients}

# 3. Create module file
touch src/modules/1600-new-feature/1600-new-feature.module.ts
```

### Module Template

```typescript
// 1600-new-feature.module.ts
import { Module } from '@nestjs/common';
import { CqrsModule } from '@nestjs/cqrs';

// Presentation Layer
import { NewFeatureController } from './presentation/controllers/new-feature.controller';

// Application Layer
import { CreateFeatureHandler } from './application/handlers/create-feature.handler';
import { GetFeatureHandler } from './application/handlers/get-feature.handler';

// Infrastructure Layer
import { FeatureRepository } from './infrastructure/persistence/typeorm/repositories/feature.repository';

const CommandHandlers = [CreateFeatureHandler];
const QueryHandlers = [GetFeatureHandler];
const EventHandlers = [];

@Module({
  imports: [CqrsModule],
  controllers: [NewFeatureController],
  providers: [
    ...CommandHandlers,
    ...QueryHandlers,
    ...EventHandlers,
    FeatureRepository,
  ],
  exports: [FeatureRepository],
})
export class NewFeatureModule {}
```

---

## Testing Strategy

```mermaid
graph TB
    subgraph "Test Pyramid"
        E2E[E2E Tests<br/>10%<br/>Full API tests]
        Integration[Integration Tests<br/>30%<br/>Database + Services]
        Unit[Unit Tests<br/>60%<br/>Pure functions + Logic]
    end

    subgraph "Test Types"
        T1[Unit Tests<br/>Jest + ts-jest]
        T2[Integration Tests<br/>Testcontainers]
        T3[E2E Tests<br/>Supertest]
    end

    Unit --> T1
    Integration --> T2
    E2E --> T3

    style Unit fill:#4ecdc4
    style Integration fill:#45b7d1
    style E2E fill:#f9ca24
```

### Test Coverage Goals

| Layer | Target Coverage | Current |
|-------|----------------|---------|
| **Domain Layer** | 90% | 85% |
| **Application Layer** | 80% | 75% |
| **Infrastructure Layer** | 70% | 65% |
| **Presentation Layer** | 60% | 55% |
| **Overall** | 75% | 70% |

---

## Best Practices

### 1. Module Independence

✅ **Good:**
```typescript
// Module communicates via events
this.eventBus.publish(new MetricIngested(metric));
```

❌ **Bad:**
```typescript
// Direct cross-module dependency
import { AlertService } from '../600-alerts/application/services/alert.service';
```

### 2. Domain-First Development

✅ **Good:**
```typescript
// Start with domain aggregate
class Metric {
  validate() {
    if (this.value < 0) throw new InvalidMetricError();
  }
}
```

❌ **Bad:**
```typescript
// Start with database schema
@Entity()
class MetricEntity {
  @Column() value: number;
}
```

### 3. Use Value Objects

✅ **Good:**
```typescript
class Email {
  private readonly value: string;

  constructor(email: string) {
    if (!this.isValid(email)) throw new InvalidEmailError();
    this.value = email;
  }
}
```

❌ **Bad:**
```typescript
// Primitive obsession
function createUser(email: string) {
  // No validation
}
```

---

## Performance Considerations

```mermaid
graph LR
    subgraph "Performance Optimizations"
        C[Caching<br/>L1 + L2]
        Q[Queue<br/>BullMQ Async]
        I[Indexes<br/>20+ ClickHouse]
        P[Pooling<br/>Connection Reuse]
    end

    subgraph "Results"
        R1[Cache Hit Rate: 75-85%]
        R2[Async Processing: 1000 jobs/sec]
        R3[Query Latency: 50-200ms p95]
        R4[API Latency: 100-300ms p95]
    end

    C --> R1
    Q --> R2
    I --> R3
    P --> R4

    style C fill:#4ecdc4
    style Q fill:#f9ca24
    style I fill:#6c5ce7
    style P fill:#e74c3c
```

---

## Documentation Index

### Architecture Docs
- [System Architecture](../architecture/01-SYSTEM-ARCHITECTURE.md) - High-level platform overview
- [Data Flow](../architecture/02-DATA-FLOW.md) - Request/response flows
- [Multi-Tenancy](../architecture/03-MULTI-TENANCY.md) - Tenant isolation
- [Security](../architecture/04-SECURITY.md) - Auth, RBAC, audit
- [Performance](../architecture/05-PERFORMANCE.md) - Optimization strategies

### Backend Docs
- **00-BACKEND-OVERVIEW.md** ← You are here
- [01-TECH-STACK.md](./01-TECH-STACK.md) - Technology stack details
- [02-DDD-CQRS.md](./02-DDD-CQRS.md) - DDD/CQRS implementation
- [03-MODULE-STRUCTURE.md](./03-MODULE-STRUCTURE.md) - LEGO pattern guide

### Module Docs
- [100-core](./modules/100-core.md) - IAM & Multi-Tenancy
- [200-auth](./modules/200-auth.md) - Authentication
- [400-telemetry](./modules/400-telemetry.md) - OTLP Ingestion
- [600-alerts](./modules/600-alerts.md) - Alert Rules
- [900-dashboard](./modules/900-dashboard.md) - Dashboards

---

- **File Location:** `./backend/00-BACKEND-OVERVIEW.md`
- **Maintained By:** DevOpsCorner Indonesia
- **Last Updated:** December 12, 2025
