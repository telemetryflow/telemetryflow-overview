# Module Structure Guide (LEGO Pattern)

- **Version:** 1.0.0-CE
- **Last Updated:** December 12, 2025
- **Status:** ✅ Complete

---

## Table of Contents

1. [Introduction](#introduction)
2. [LEGO Pattern Philosophy](#lego-pattern-philosophy)
3. [Standard Module Structure](#standard-module-structure)
4. [Layer Breakdown](#layer-breakdown)
5. [File Naming Conventions](#file-naming-conventions)
6. [Module Template](#module-template)
7. [Creating a New Module](#creating-a-new-module)
8. [Module Communication](#module-communication)
9. [Testing Structure](#testing-structure)
10. [Best Practices](#best-practices)

---

## Introduction

TelemetryFlow uses a **standardized module structure** called the **LEGO Pattern** to ensure consistency, maintainability, and scalability across all 15 business modules.

### Why LEGO Pattern?

```mermaid
graph LR
    subgraph "LEGO Pattern Benefits"
        A[Consistency<br/>All modules follow same structure]
        B[Predictability<br/>Easy to navigate any module]
        C[Reusability<br/>Copy-paste template for new modules]
        D[Scalability<br/>Easy to extract to microservices]
    end

    subgraph "Developer Experience"
        E[✅ Faster Onboarding<br/>Learn once, apply everywhere]
        F[✅ Reduced Cognitive Load<br/>Know where everything is]
        G[✅ Better Code Review<br/>Standard patterns]
        H[✅ Easier Refactoring<br/>Clear boundaries]
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

## LEGO Pattern Philosophy

### Building Block Metaphor

```mermaid
graph TB
    subgraph "LEGO Blocks = Modules"
        L1[Module 100-core<br/>IAM Foundation]
        L2[Module 200-auth<br/>Authentication]
        L3[Module 400-telemetry<br/>OTLP Ingestion]
        L4[Module 600-alerts<br/>Alert Rules]
    end

    subgraph "Each Block Has Same Structure"
        S1[Presentation Layer]
        S2[Application Layer]
        S3[Domain Layer]
        S4[Infrastructure Layer]
    end

    subgraph "Blocks Connect via Interfaces"
        I1[Domain Events]
        I2[Repository Interfaces]
        I3[Public APIs]
    end

    L1 --> S1
    L2 --> S1
    L3 --> S1
    L4 --> S1

    S1 --> I1
    S2 --> I2
    S3 --> I3

    style L3 fill:#ff6b6b
    style S1 fill:#4ecdc4
    style I1 fill:#f9ca24
```

**Key Principles:**
1. ✅ **Uniform Structure** - Every module looks the same
2. ✅ **Clear Boundaries** - Explicit interfaces between layers
3. ✅ **Loose Coupling** - Modules communicate via events
4. ✅ **High Cohesion** - Related code stays together

---

## Standard Module Structure

### Complete Folder Structure

```
400-telemetry/
├── 400-telemetry.module.ts              # Module definition & registration
│
├── presentation/                         # Layer 1: HTTP/REST Interface
│   ├── controllers/
│   │   ├── metrics.controller.ts
│   │   ├── logs.controller.ts
│   │   └── traces.controller.ts
│   ├── dtos/
│   │   ├── ingest-metrics.dto.ts
│   │   ├── query-metrics.dto.ts
│   │   └── metric-filters.dto.ts
│   ├── guards/
│   │   └── api-key-auth.guard.ts
│   └── decorators/
│       └── tenant-context.decorator.ts
│
├── application/                          # Layer 2: Business Workflows
│   ├── commands/
│   │   ├── ingest-metrics-from-otlp.command.ts
│   │   └── delete-old-metrics.command.ts
│   ├── queries/
│   │   ├── get-metric-timeseries.query.ts
│   │   ├── get-metric-names.query.ts
│   │   └── get-logs-paginated.query.ts
│   ├── handlers/
│   │   ├── ingest-metrics-from-otlp.handler.ts
│   │   ├── get-metric-timeseries.handler.ts
│   │   └── index.ts
│   ├── services/
│   │   ├── metrics.service.ts
│   │   ├── logs.service.ts
│   │   └── aggregation.service.ts
│   └── events/
│       └── metric-ingested.handler.ts
│
├── domain/                               # Layer 3: Business Logic
│   ├── aggregates/
│   │   ├── Metric.aggregate.ts
│   │   ├── Log.aggregate.ts
│   │   └── Trace.aggregate.ts
│   ├── value-objects/
│   │   ├── MetricName.vo.ts
│   │   ├── MetricValue.vo.ts
│   │   ├── MetricType.vo.ts
│   │   ├── Timestamp.vo.ts
│   │   └── TenantContext.vo.ts
│   ├── events/
│   │   ├── MetricIngested.event.ts
│   │   ├── MetricValueUpdated.event.ts
│   │   └── ExemplarLinked.event.ts
│   ├── repositories/
│   │   ├── metric.repository.interface.ts
│   │   ├── log.repository.interface.ts
│   │   └── trace.repository.interface.ts
│   └── services/
│       └── metric-validator.service.ts
│
└── infrastructure/                       # Layer 4: External Systems
    ├── persistence/
    │   ├── typeorm/
    │   │   └── entities/
    │   │       └── (if needed for metadata)
    │   └── clickhouse/
    │       ├── schemas/
    │       │   ├── 001-metrics.schema.ts
    │       │   ├── 002-logs.schema.ts
    │       │   └── 003-traces.schema.ts
    │       ├── repositories/
    │       │   ├── metric.repository.ts
    │       │   ├── log.repository.ts
    │       │   └── trace.repository.ts
    │       └── migrations/
    │           ├── 001-create-tables.ts
    │           └── 002-query-optimization.ts
    ├── mappers/
    │   ├── metric.mapper.ts
    │   ├── log.mapper.ts
    │   └── otlp-to-domain.mapper.ts
    └── clients/
        └── otlp-transformer.client.ts
```

### Visual Structure

```mermaid
graph TB
    subgraph "400-telemetry Module"
        Root[400-telemetry.module.ts<br/>Root Module File]

        subgraph "presentation/ - Layer 1"
            C[controllers/<br/>REST Endpoints]
            D[dtos/<br/>Validation]
            G[guards/<br/>Auth]
            Dec[decorators/<br/>Metadata]
        end

        subgraph "application/ - Layer 2"
            CMD[commands/<br/>Write Ops]
            QRY[queries/<br/>Read Ops]
            H[handlers/<br/>CQRS]
            SVC[services/<br/>Orchestration]
        end

        subgraph "domain/ - Layer 3"
            AGG[aggregates/<br/>Entities]
            VO[value-objects/<br/>Immutable]
            EV[events/<br/>Domain Events]
            REPO[repositories/<br/>Interfaces]
        end

        subgraph "infrastructure/ - Layer 4"
            PERS[persistence/<br/>TypeORM + ClickHouse]
            MAP[mappers/<br/>Data Transformation]
            CLIENT[clients/<br/>External APIs]
        end
    end

    Root --> C
    C --> CMD
    CMD --> AGG
    AGG --> PERS
    PERS --> MAP

    style Root fill:#4ecdc4
    style CMD fill:#ff6b6b
    style AGG fill:#f9ca24
    style PERS fill:#6c5ce7
```

---

## Layer Breakdown

### Layer 1: Presentation

**Responsibility:** HTTP request handling, validation, authentication

```mermaid
graph LR
    subgraph "Presentation Layer Components"
        A[Controllers]
        B[DTOs]
        C[Guards]
        D[Interceptors]
        E[Decorators]
    end

    subgraph "Responsibilities"
        F[HTTP Request Parsing]
        G[Input Validation]
        H[Authentication Check]
        I[Response Serialization]
    end

    A --> F
    B --> G
    C --> H
    D --> I

    style A fill:#4ecdc4
    style B fill:#45b7d1
    style C fill:#f9ca24
```

**Example Controller:**
```typescript
// presentation/controllers/metrics.controller.ts
@Controller('api/v2/telemetry/metrics')
@UseGuards(JwtAuthGuard, TenantContextGuard)
export class MetricsController {
  constructor(
    private readonly commandBus: CommandBus,
    private readonly queryBus: QueryBus,
  ) {}

  @Post()
  @ApiOperation({ summary: 'Ingest metrics via OTLP' })
  async ingestMetrics(
    @Body() dto: IngestMetricsDto,
    @TenantContext() tenantContext: TenantContext,
  ): Promise<{ accepted: boolean }> {
    const command = new IngestMetricsFromOtlpCommand(dto, tenantContext);
    await this.commandBus.execute(command);
    return { accepted: true };
  }

  @Get()
  @ApiOperation({ summary: 'Query metrics time series' })
  async queryMetrics(
    @Query() dto: QueryMetricsDto,
    @TenantContext() tenantContext: TenantContext,
  ): Promise<MetricTimeSeries> {
    const query = new GetMetricTimeSeriesQuery(dto, tenantContext);
    return this.queryBus.execute(query);
  }
}
```

### Layer 2: Application

**Responsibility:** Business workflow orchestration, CQRS implementation

```mermaid
graph TB
    subgraph "Application Layer Components"
        A[Commands<br/>Write Intentions]
        B[Queries<br/>Read Requests]
        C[Handlers<br/>Execute Logic]
        D[Services<br/>Complex Workflows]
    end

    subgraph "Responsibilities"
        E[✅ Orchestrate Domain Logic]
        F[✅ Transaction Management]
        G[✅ Event Publishing]
        H[✅ Error Handling]
    end

    A --> E
    C --> F
    C --> G
    C --> H

    style A fill:#ff6b6b
    style B fill:#4ecdc4
    style C fill:#45b7d1
```

**Example Command Handler:**
```typescript
// application/handlers/ingest-metrics-from-otlp.handler.ts
@CommandHandler(IngestMetricsFromOtlpCommand)
export class IngestMetricsFromOtlpHandler
  implements ICommandHandler<IngestMetricsFromOtlpCommand> {

  constructor(
    private readonly queueService: QueueService,
    private readonly logger: WinstonLogger,
  ) {}

  async execute(command: IngestMetricsFromOtlpCommand): Promise<void> {
    this.logger.info('Ingesting metrics from OTLP', {
      tenantId: command.tenantContext.tenantId.value,
      metricCount: command.metrics.length,
    });

    // Queue for async processing (prevents HTTP timeout)
    await this.queueService.addJob('otlp-ingestion', {
      type: 'metrics',
      data: command.metrics,
      tenantContext: command.tenantContext,
    });
  }
}
```

### Layer 3: Domain

**Responsibility:** Business rules, domain logic, pure functions

```mermaid
graph TB
    subgraph "Domain Layer Components"
        A[Aggregates<br/>Business Entities]
        B[Value Objects<br/>Validated Types]
        C[Domain Events<br/>State Changes]
        D[Repository Interfaces<br/>Data Ports]
        E[Domain Services<br/>Cross-Aggregate Logic]
    end

    subgraph "Characteristics"
        F[✅ Pure Business Logic]
        G[✅ No Infrastructure Dependencies]
        H[✅ Highly Testable]
        I[✅ Framework Agnostic]
    end

    A --> F
    B --> G
    C --> H
    E --> I

    style A fill:#f9ca24
    style B fill:#4ecdc4
    style C fill:#ff6b6b
```

**Example Aggregate:**
```typescript
// domain/aggregates/Metric.aggregate.ts
export class Metric extends AggregateRoot {
  private readonly _id: MetricId;
  private readonly _metricName: MetricName;
  private _value: MetricValue;

  static create(props: MetricProps): Metric {
    // Business Rule: Counter metrics must be monotonic
    if (props.metricType.isCounter() && props.isMonotonic === false) {
      throw new DomainError('Counter metrics must be monotonic');
    }

    const metric = new Metric(props);
    metric.apply(new MetricIngested(metric));
    return metric;
  }

  updateValue(newValue: MetricValue): void {
    // Business Rule: Counters cannot decrease
    if (this._metricType.isCounter() && newValue.isLessThan(this._value)) {
      throw new DomainError('Counter values cannot decrease');
    }
    this._value = newValue;
    this.apply(new MetricValueUpdated(this));
  }
}
```

### Layer 4: Infrastructure

**Responsibility:** External systems, database, third-party APIs

```mermaid
graph TB
    subgraph "Infrastructure Layer Components"
        A[persistence/<br/>Database Access]
        B[mappers/<br/>Data Transformation]
        C[clients/<br/>External APIs]
        D[migrations/<br/>Schema Evolution]
    end

    subgraph "Technologies"
        E[TypeORM<br/>PostgreSQL]
        F[ClickHouse Client<br/>Time-Series]
        G[HTTP Clients<br/>REST APIs]
        H[Redis<br/>Cache]
    end

    A --> E
    A --> F
    C --> G
    A --> H

    style A fill:#6c5ce7
    style B fill:#4ecdc4
```

**Example Repository:**
```typescript
// infrastructure/persistence/clickhouse/repositories/metric.repository.ts
@Injectable()
export class MetricRepository implements IMetricRepository {
  constructor(
    private readonly clickhouse: ClickHouseService,
    private readonly cache: CacheService,
    private readonly mapper: MetricMapper,
  ) {}

  async save(metric: Metric): Promise<void> {
    const schema = this.mapper.toClickHouseSchema(metric);
    await this.clickhouse.insert('telemetry_metrics', [schema]);
    await this.cache.invalidate(`metrics:${metric.tenantId}`);
  }

  async findById(id: MetricId): Promise<Metric | null> {
    const row = await this.clickhouse.query(
      'SELECT * FROM telemetry_metrics WHERE id = {id:String}',
      { id: id.value },
    );
    return row ? this.mapper.toDomain(row) : null;
  }
}
```

---

## File Naming Conventions

### Naming Patterns

```mermaid
graph TB
    subgraph "File Naming Convention"
        A[Entity Type]
        B[Purpose]
        C[Suffix]
        D[Extension]
    end

    subgraph "Examples"
        E[metric.aggregate.ts<br/>Entity: metric<br/>Type: aggregate]
        F[metric-name.vo.ts<br/>Entity: metric-name<br/>Type: value-object]
        G[ingest-metrics.command.ts<br/>Action: ingest-metrics<br/>Type: command]
        H[get-metrics.query.ts<br/>Action: get-metrics<br/>Type: query]
        I[metrics.controller.ts<br/>Resource: metrics<br/>Type: controller]
    end

    A --> E
    B --> F
    C --> G
    D --> H

    style E fill:#f9ca24
    style F fill:#4ecdc4
    style G fill:#ff6b6b
    style H fill:#45b7d1
```

### File Suffix Guide

| Suffix | Usage | Example |
|--------|-------|---------|
| `.aggregate.ts` | Aggregate Roots | `Metric.aggregate.ts` |
| `.vo.ts` | Value Objects | `MetricName.vo.ts` |
| `.event.ts` | Domain Events | `MetricIngested.event.ts` |
| `.command.ts` | CQRS Commands | `IngestMetrics.command.ts` |
| `.query.ts` | CQRS Queries | `GetMetrics.query.ts` |
| `.handler.ts` | Command/Query/Event Handlers | `IngestMetrics.handler.ts` |
| `.controller.ts` | REST Controllers | `metrics.controller.ts` |
| `.dto.ts` | Data Transfer Objects | `ingest-metrics.dto.ts` |
| `.service.ts` | Application/Domain Services | `metrics.service.ts` |
| `.repository.ts` | Repository Implementations | `metric.repository.ts` |
| `.interface.ts` | Repository Interfaces | `metric.repository.interface.ts` |
| `.mapper.ts` | Data Mappers | `metric.mapper.ts` |
| `.guard.ts` | Auth Guards | `api-key-auth.guard.ts` |
| `.decorator.ts` | Custom Decorators | `tenant-context.decorator.ts` |
| `.schema.ts` | Database Schemas | `001-metrics.schema.ts` |
| `.module.ts` | NestJS Modules | `400-telemetry.module.ts` |

---

## Module Template

### Complete Module File

```typescript
// 400-telemetry/400-telemetry.module.ts
import { Module } from '@nestjs/common';
import { CqrsModule } from '@nestjs/cqrs';

// Presentation Layer
import { MetricsController } from './presentation/controllers/metrics.controller';
import { LogsController } from './presentation/controllers/logs.controller';
import { TracesController } from './presentation/controllers/traces.controller';

// Application Layer - Commands
import { IngestMetricsFromOtlpHandler } from './application/handlers/ingest-metrics-from-otlp.handler';
import { DeleteOldMetricsHandler } from './application/handlers/delete-old-metrics.handler';

// Application Layer - Queries
import { GetMetricTimeSeriesHandler } from './application/handlers/get-metric-timeseries.handler';
import { GetMetricNamesHandler } from './application/handlers/get-metric-names.handler';
import { GetLogsPaginatedHandler } from './application/handlers/get-logs-paginated.handler';

// Application Layer - Event Handlers
import { MetricIngestedHandler } from './application/events/metric-ingested.handler';

// Application Layer - Services
import { MetricsService } from './application/services/metrics.service';
import { LogsService } from './application/services/logs.service';
import { AggregationService } from './application/services/aggregation.service';

// Infrastructure Layer
import { MetricRepository } from './infrastructure/persistence/clickhouse/repositories/metric.repository';
import { LogRepository } from './infrastructure/persistence/clickhouse/repositories/log.repository';
import { TraceRepository } from './infrastructure/persistence/clickhouse/repositories/trace.repository';
import { MetricMapper } from './infrastructure/mappers/metric.mapper';
import { OtlpTransformer } from './infrastructure/clients/otlp-transformer.client';

// Shared Modules
import { CacheModule } from '@/shared/cache/cache.module';
import { QueueModule } from '@/shared/queue/queue.module';
import { ClickHouseModule } from '@/shared/clickhouse/clickhouse.module';

const CommandHandlers = [
  IngestMetricsFromOtlpHandler,
  DeleteOldMetricsHandler,
];

const QueryHandlers = [
  GetMetricTimeSeriesHandler,
  GetMetricNamesHandler,
  GetLogsPaginatedHandler,
];

const EventHandlers = [
  MetricIngestedHandler,
];

const ApplicationServices = [
  MetricsService,
  LogsService,
  AggregationService,
];

const Repositories = [
  MetricRepository,
  LogRepository,
  TraceRepository,
];

const Mappers = [
  MetricMapper,
];

const Clients = [
  OtlpTransformer,
];

@Module({
  imports: [
    CqrsModule,
    CacheModule,
    QueueModule,
    ClickHouseModule,
  ],
  controllers: [
    MetricsController,
    LogsController,
    TracesController,
  ],
  providers: [
    ...CommandHandlers,
    ...QueryHandlers,
    ...EventHandlers,
    ...ApplicationServices,
    ...Repositories,
    ...Mappers,
    ...Clients,
  ],
  exports: [
    ...Repositories,
    ...ApplicationServices,
  ],
})
export class TelemetryModule {}
```

---

## Creating a New Module

### Step-by-Step Guide

```mermaid
flowchart TD
    A[1. Create Module Directory<br/>mkdir 1600-new-feature] --> B[2. Create LEGO Structure<br/>presentation/application/domain/infrastructure]
    B --> C[3. Create Module File<br/>1600-new-feature.module.ts]
    C --> D[4. Define Domain Models<br/>Aggregates + Value Objects]
    D --> E[5. Create Repository Interface<br/>domain/repositories/]
    E --> F[6. Implement Commands/Queries<br/>application/commands, queries]
    F --> G[7. Create Handlers<br/>application/handlers]
    G --> H[8. Implement Repository<br/>infrastructure/persistence]
    H --> I[9. Create Controllers<br/>presentation/controllers]
    I --> J[10. Register in App Module<br/>app.module.ts]
    J --> K[11. Write Tests<br/>Unit + Integration]
    K --> L[✅ Module Complete]

    style A fill:#4ecdc4
    style D fill:#f9ca24
    style G fill:#ff6b6b
    style L fill:#27ae60
```

### CLI Commands

```bash
# 1. Create module directory
mkdir -p src/modules/1600-new-feature

# 2. Create LEGO structure
mkdir -p src/modules/1600-new-feature/presentation/{controllers,dtos,guards,decorators}
mkdir -p src/modules/1600-new-feature/application/{commands,queries,handlers,services,events}
mkdir -p src/modules/1600-new-feature/domain/{aggregates,value-objects,events,repositories,services}
mkdir -p src/modules/1600-new-feature/infrastructure/{persistence,mappers,clients}

# 3. Create module file
touch src/modules/1600-new-feature/1600-new-feature.module.ts

# 4. Create test directory
mkdir -p src/modules/1600-new-feature/__tests__/{unit,integration,e2e}
```

---

## Module Communication

### Event-Driven Communication (Preferred)

```mermaid
sequenceDiagram
    participant M1 as Module A<br/>400-telemetry
    participant Bus as EventBus<br/>@nestjs/cqrs
    participant M2 as Module B<br/>600-alerts
    participant M3 as Module C<br/>800-audit

    M1->>M1: Business Logic Executes
    M1->>Bus: Publish MetricIngested Event

    par Parallel Event Handling
        Bus->>M2: Handle MetricIngested
        M2->>M2: Evaluate Alert Rules
        M2->>Bus: Publish AlertTriggered Event

        Bus->>M3: Handle MetricIngested
        M3->>M3: Log Audit Entry
    end

    Note over M1,M3: Modules are decoupled<br/>No direct dependencies
```

### Direct Communication (Discouraged)

```mermaid
graph LR
    A[Module A] -->|Direct Import ❌| B[Module B]
    A -->|Domain Events ✅| C[Event Bus]
    C --> B

    style A fill:#e74c3c
    style C fill:#27ae60
```

---

## Testing Structure

### Test Organization

```
400-telemetry/
├── __tests__/
│   ├── unit/
│   │   ├── domain/
│   │   │   ├── Metric.aggregate.spec.ts
│   │   │   └── MetricName.vo.spec.ts
│   │   ├── application/
│   │   │   └── IngestMetrics.handler.spec.ts
│   │   └── infrastructure/
│   │       └── metric.repository.spec.ts
│   ├── integration/
│   │   ├── metrics.controller.spec.ts
│   │   └── metric-ingestion.spec.ts
│   └── e2e/
│       └── otlp-ingestion.e2e.spec.ts
```

### Test Pyramid

```mermaid
graph TB
    subgraph "Test Pyramid"
        E2E[E2E Tests - 10%<br/>Full API flow]
        Integration[Integration Tests - 30%<br/>Database + Services]
        Unit[Unit Tests - 60%<br/>Domain + Application]
    end

    E2E --> Integration
    Integration --> Unit

    style Unit fill:#27ae60
    style Integration fill:#f9ca24
    style E2E fill:#ff6b6b
```

---

## Best Practices

### ✅ Do's

```typescript
// ✅ Use Value Objects for validation
const metricName = MetricName.create('cpu_usage_percent');

// ✅ Emit domain events
metric.apply(new MetricIngested(metric));

// ✅ Use repository interfaces in domain
interface IMetricRepository {
  save(metric: Metric): Promise<void>;
}

// ✅ Keep controllers thin
@Post()
async create(@Body() dto: CreateDto) {
  const command = new CreateCommand(dto);
  return this.commandBus.execute(command);
}

// ✅ Use CQRS for complex operations
@CommandHandler(IngestMetricsCommand)
class IngestMetricsHandler { }

// ✅ Separate read and write models
class GetMetricsQuery { }
class IngestMetricsCommand { }
```

### ❌ Don'ts

```typescript
// ❌ Don't use primitives everywhere (primitive obsession)
class Metric {
  constructor(public name: string) {} // Use MetricName.vo.ts
}

// ❌ Don't put business logic in controllers
@Post()
async create(@Body() dto: CreateDto) {
  if (dto.value < 0) throw new Error(); // Move to domain
  await this.repository.save(dto);
}

// ❌ Don't directly import other modules
import { AlertService } from '../600-alerts'; // Use events instead

// ❌ Don't mix layers
class Metric {
  async save(clickhouse: ClickHouse) { } // Infrastructure in domain!
}

// ❌ Don't skip validation
const metric = new Metric(userInput); // Use factory with validation
```

---

## Summary

```mermaid
graph TB
    subgraph "LEGO Pattern Summary"
        A[Standardized Structure<br/>4 Layers in Every Module]
        B[Clear Boundaries<br/>Layer Dependencies Point Inward]
        C[Event-Driven Communication<br/>Loose Coupling Between Modules]
        D[Testable Architecture<br/>Pure Domain Logic]
    end

    subgraph "Results"
        E[✅ Consistent Codebase]
        F[✅ Easy Navigation]
        G[✅ Scalable Design]
        H[✅ Maintainable Code]
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

## Related Documentation

- [Backend Overview](./00-BACKEND-OVERVIEW.md) - Architecture principles
- [DDD & CQRS](./02-DDD-CQRS.md) - Domain-driven design patterns
- [System Architecture](../architecture/01-SYSTEM-ARCHITECTURE.md) - High-level overview

---

- **File Location:** `./backend/03-MODULE-STRUCTURE.md`
- **Maintained By:** DevOpsCorner Indonesia
- **Last Updated:** December 12, 2025
