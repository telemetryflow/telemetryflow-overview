# TFO-OTEL Architecture

- **Version:** 1.1.1-CE
- **Last Updated:** December 13, 2025
- **Status:** ✅ Production Ready

---

## Table of Contents

1. [Overview](#overview)
2. [System Architecture](#system-architecture)
3. [Component Architecture](#component-architecture)
4. [Data Flow](#data-flow)
5. [Processing Pipeline](#processing-pipeline)
6. [Multi-Tenancy](#multi-tenancy)
7. [Deployment Patterns](#deployment-patterns)
8. [Scalability](#scalability)

---

## Overview

The TelemetryFlow OTEL Architecture provides a robust, scalable, and multi-tenant observability infrastructure based on OpenTelemetry standards.

### Design Principles

1. **Standard Compliance:** 100% OpenTelemetry Protocol (OTLP) compatibility
2. **Multi-Tenancy:** Built-in workspace and tenant isolation
3. **High Availability:** Stateless, horizontally scalable design
4. **Performance:** Optimized for high-throughput, low-latency operation
5. **Flexibility:** Pluggable receivers, processors, and exporters

---

## System Architecture

### Tier-Based Architecture

```mermaid
graph TB
    subgraph "Tier 1: Data Sources"
        subgraph "Applications"
            APP1[Node.js App<br/>OTEL SDK]
            APP2[Java App<br/>OTEL SDK]
            APP3[Python App<br/>OTEL SDK]
        end

        subgraph "Infrastructure"
            NODE[node-exporter<br/>Prometheus]
            K8S[kube-state-metrics<br/>Prometheus]
            DB[postgres-exporter<br/>Prometheus]
        end

        subgraph "Logs"
            SYSLOG[Syslog]
            FLUENT[Fluentd/Fluent Bit]
            WINSTON[Winston Logger]
        end
    end

    subgraph "Tier 2: Edge Collection (TFO-OTEL-Agent)"
        AGENT1[Agent - Host 1<br/>:4317, :4318]
        AGENT2[Agent - Host 2<br/>:4317, :4318]
        AGENT3[Agent - Host N<br/>:4317, :4318]
    end

    subgraph "Tier 3: Aggregation (TFO-OTEL-Collector)"
        LB[Load Balancer]
        COLL1[Collector 1<br/>:4317, :4318]
        COLL2[Collector 2<br/>:4317, :4318]
        COLL3[Collector 3<br/>:4317, :4318]
    end

    subgraph "Tier 4: Backends"
        TFO[TelemetryFlow Platform<br/>ClickHouse + PostgreSQL]
        PROM[Prometheus<br/>Time Series DB]
        LOKI[Grafana Loki<br/>Log Aggregation]
        OS[OpenSearch<br/>Full-Text Search]
    end

    APP1 & APP2 & APP3 -->|OTLP| AGENT1
    NODE & K8S & DB -->|Prometheus| AGENT2
    SYSLOG & FLUENT & WINSTON -->|OTLP Logs| AGENT3

    AGENT1 & AGENT2 & AGENT3 -->|OTLP HTTP| LB
    LB --> COLL1
    LB --> COLL2
    LB --> COLL3

    COLL1 & COLL2 & COLL3 -->|OTLP HTTP| TFO
    COLL1 & COLL2 & COLL3 -->|Remote Write| PROM
    COLL1 & COLL2 & COLL3 -->|Push API| LOKI
    COLL1 & COLL2 & COLL3 -->|Bulk API| OS

    style AGENT1 fill:#FFE082,stroke:#F57C00,color:#000
    style AGENT2 fill:#FFE082,stroke:#F57C00,color:#000
    style AGENT3 fill:#FFE082,stroke:#F57C00,color:#000
    style COLL1 fill:#81C784,stroke:#388E3C,color:#000
    style COLL2 fill:#81C784,stroke:#388E3C,color:#000
    style COLL3 fill:#81C784,stroke:#388E3C,color:#000
    style TFO fill:#64B5F6,stroke:#1976D2,color:#fff
```

---

## Component Architecture

### TFO-OTEL-Agent Architecture

```mermaid
graph TB
    subgraph "TFO-OTEL-Agent"
        subgraph "Receivers"
            R_OTLP[OTLP Receiver<br/>:4317, :4318]
            R_PROM[Prometheus Receiver<br/>Scraper]
            R_HOST[Host Metrics Receiver<br/>CPU, Memory, Disk]
        end

        subgraph "Processors"
            P_BATCH[Batch Processor<br/>1000 items, 10s]
            P_MEM[Memory Limiter<br/>512MB]
            P_ATTR[Attributes Processor<br/>Add Context]
            P_RES[Resource Detection<br/>Host, Docker, K8s]
        end

        subgraph "Exporters"
            E_COLL[OTLP Exporter<br/>to Collector]
            E_FILE[File Exporter<br/>Backup]
            E_LOG[Logging Exporter<br/>Debug]
        end

        subgraph "Extensions"
            EXT_HEALTH[Health Check<br/>:13133]
            EXT_PPROF[pprof<br/>:1777]
            EXT_ZPAGE[zPages<br/>:55679]
        end
    end

    R_OTLP & R_PROM & R_HOST --> P_MEM
    P_MEM --> P_RES
    P_RES --> P_ATTR
    P_ATTR --> P_BATCH
    P_BATCH --> E_COLL
    P_BATCH -.->|Fallback| E_FILE
    P_BATCH -.->|Debug| E_LOG

    style R_OTLP fill:#E1F5FE
    style P_BATCH fill:#FFF9C4
    style E_COLL fill:#C8E6C9
```

### TFO-OTEL-Collector Architecture

```mermaid
graph TB
    subgraph "TFO-OTEL-Collector"
        subgraph "Receivers"
            RC_OTLP[OTLP Receiver<br/>:4317, :4318]
            RC_PROM[Prometheus Receiver<br/>Scraper]
            RC_FLUENT[FluentForward Receiver<br/>:8006]
        end

        subgraph "Processors - Metrics Pipeline"
            PM_BATCH[Batch Processor]
            PM_MEM[Memory Limiter<br/>2GB]
            PM_ATTR[Attributes Processor<br/>Workspace/Tenant]
            PM_FILTER[Filter Processor<br/>Drop Internal]
            PM_TRANS[Transform Processor<br/>Metric Rename]
        end

        subgraph "Processors - Logs Pipeline"
            PL_BATCH[Batch Processor]
            PL_MEM[Memory Limiter]
            PL_ATTR[Attributes Processor]
            PL_PARSE[Log Parser<br/>JSON, Regex]
        end

        subgraph "Processors - Traces Pipeline"
            PT_BATCH[Batch Processor]
            PT_MEM[Memory Limiter]
            PT_ATTR[Attributes Processor]
            PT_SAMPLE[Tail Sampling<br/>10% rate]
        end

        subgraph "Exporters"
            EM_TFO[OTLP/HTTP<br/>TelemetryFlow]
            EM_PROM[Prometheus<br/>:8889]
            EM_LOKI[Loki<br/>Push API]
            EM_OS[OpenSearch<br/>Bulk API]
            EM_LOG[Logging]
        end
    end

    RC_OTLP --> PM_BATCH
    RC_PROM --> PM_BATCH
    PM_BATCH --> PM_MEM --> PM_ATTR --> PM_FILTER --> PM_TRANS
    PM_TRANS --> EM_TFO
    PM_TRANS --> EM_PROM
    PM_TRANS --> EM_LOG

    RC_OTLP --> PL_BATCH
    RC_FLUENT --> PL_BATCH
    PL_BATCH --> PL_MEM --> PL_ATTR --> PL_PARSE
    PL_PARSE --> EM_TFO
    PL_PARSE --> EM_LOKI
    PL_PARSE --> EM_OS
    PL_PARSE --> EM_LOG

    RC_OTLP --> PT_BATCH
    PT_BATCH --> PT_MEM --> PT_ATTR --> PT_SAMPLE
    PT_SAMPLE --> EM_TFO
    PT_SAMPLE --> EM_LOG

    style RC_OTLP fill:#E1F5FE
    style PM_BATCH fill:#FFF9C4
    style PL_BATCH fill:#FFF9C4
    style PT_BATCH fill:#FFF9C4
    style EM_TFO fill:#C8E6C9
```

---

## Data Flow

### Metrics Data Flow

```mermaid
sequenceDiagram
    participant App as Application<br/>(OTEL SDK)
    participant Agent as TFO-OTEL-Agent
    participant Collector as TFO-OTEL-Collector
    participant TFO as TelemetryFlow API
    participant CH as ClickHouse

    Note over App: Generate Metrics
    App->>App: Create MetricData
    App->>App: Add Resource Attributes

    App->>Agent: OTLP/gRPC ExportMetricsServiceRequest
    Note over Agent: Batch Processing (10s or 1000 items)
    Agent->>Agent: Add Host Metadata
    Agent->>Agent: Add Workspace/Tenant IDs

    Agent->>Collector: OTLP/HTTP POST /v1/metrics
    Note over Collector: Process & Enrich
    Collector->>Collector: Filter Internal Metrics
    Collector->>Collector: Transform Metric Names
    Collector->>Collector: Validate Multi-Tenant Context

    Collector->>TFO: OTLP/HTTP POST /api/v1/metrics<br/>Headers: X-Workspace-Id, X-Tenant-Id
    TFO->>TFO: Validate Request
    TFO->>TFO: Extract Tenant Context
    TFO->>TFO: Transform OTLP → Internal Format
    TFO->>CH: INSERT INTO metrics_v3
    CH-->>TFO: Success
    TFO-->>Collector: 200 OK
    Collector-->>Agent: 200 OK
    Agent-->>App: ExportMetricsServiceResponse
```

### Logs Data Flow

```mermaid
sequenceDiagram
    participant Winston as Winston Logger
    participant Agent as TFO-OTEL-Agent
    participant Collector as TFO-OTEL-Collector
    participant Loki as Grafana Loki
    participant TFO as TelemetryFlow API

    Winston->>Winston: logger.info(message, context)
    Winston->>Winston: Format as OTLP LogRecord

    Winston->>Agent: OTLP/gRPC ExportLogsServiceRequest
    Agent->>Agent: Batch Logs (10s or 1000 items)
    Agent->>Agent: Add Trace Context (if available)

    Agent->>Collector: OTLP/HTTP POST /v1/logs
    Collector->>Collector: Parse Log Body (JSON/Text)
    Collector->>Collector: Extract Structured Fields
    Collector->>Collector: Add Loki Labels

    par Parallel Export
        Collector->>Loki: POST /loki/api/v1/push
        Loki-->>Collector: 204 No Content
    and
        Collector->>TFO: OTLP/HTTP POST /api/v1/logs
        TFO-->>Collector: 200 OK
    end

    Collector-->>Agent: 200 OK
    Agent-->>Winston: ExportLogsServiceResponse
```

### Traces Data Flow

```mermaid
sequenceDiagram
    participant App as Application<br/>(Instrumented)
    participant Agent as TFO-OTEL-Agent
    participant Collector as TFO-OTEL-Collector
    participant TFO as TelemetryFlow API
    participant CH as ClickHouse

    App->>App: Start Span
    App->>App: Add Span Attributes
    App->>App: End Span

    App->>Agent: OTLP/gRPC ExportTraceServiceRequest
    Agent->>Agent: Batch Spans (10s or 1000 items)

    Agent->>Collector: OTLP/HTTP POST /v1/traces
    Note over Collector: Tail Sampling Decision
    Collector->>Collector: Check Sampling Policy:<br/>- Error traces: 100%<br/>- Slow traces (>1s): 100%<br/>- Normal traces: 10%

    alt Span Selected for Sampling
        Collector->>TFO: OTLP/HTTP POST /api/v1/traces
        TFO->>TFO: Validate & Transform
        TFO->>CH: INSERT INTO traces_v3
        CH-->>TFO: Success
        TFO-->>Collector: 200 OK
    else Span Dropped
        Note over Collector: Drop span (not sampled)
    end

    Collector-->>Agent: 200 OK
    Agent-->>App: ExportTraceServiceResponse
```

---

## Processing Pipeline

### Agent Pipeline Configuration

```yaml
service:
  pipelines:
    # Metrics Pipeline
    metrics:
      receivers: [otlp, prometheus, hostmetrics]
      processors:
        - memory_limiter
        - resourcedetection
        - attributes        # Add workspace/tenant IDs
        - batch
      exporters: [otlphttp/collector]

    # Logs Pipeline
    logs:
      receivers: [otlp]
      processors:
        - memory_limiter
        - resourcedetection
        - attributes
        - batch
      exporters: [otlphttp/collector]

    # Traces Pipeline
    traces:
      receivers: [otlp]
      processors:
        - memory_limiter
        - resourcedetection
        - attributes
        - batch
      exporters: [otlphttp/collector]
```

### Collector Pipeline Configuration

```yaml
service:
  pipelines:
    # Metrics Pipeline
    metrics:
      receivers: [otlp, prometheus]
      processors:
        - memory_limiter
        - resourcedetection
        - attributes
        - filter              # Drop internal metrics
        - transform           # Rename metrics
        - batch
      exporters: [otlphttp/telemetryflow, prometheus, logging]

    # Logs Pipeline
    logs:
      receivers: [otlp, fluentforward]
      processors:
        - memory_limiter
        - attributes
        - logstransform       # Parse JSON logs
        - batch
      exporters: [otlphttp/telemetryflow, loki, opensearch, logging]

    # Traces Pipeline
    traces:
      receivers: [otlp]
      processors:
        - memory_limiter
        - attributes
        - tail_sampling       # 10% sampling
        - batch
      exporters: [otlphttp/telemetryflow, logging]
```

---

## Multi-Tenancy

### Tenant Context Propagation

```mermaid
graph LR
    subgraph "Application Level"
        APP[Application] -->|1. Set Context| SDK[OTEL SDK]
    end

    subgraph "Resource Attributes"
        SDK -->|2. Add Attributes| RESOURCE[Resource:<br/>telemetryflow.workspace.id<br/>telemetryflow.tenant.id]
    end

    subgraph "Agent Level"
        RESOURCE -->|3. Forward| AGENT[TFO-OTEL-Agent]
        AGENT -->|4. Validate| AGENT
    end

    subgraph "Collector Level"
        AGENT -->|5. Enrich| COLLECTOR[TFO-OTEL-Collector]
        COLLECTOR -->|6. Add Headers| HEADERS[HTTP Headers:<br/>X-Workspace-Id<br/>X-Tenant-Id]
    end

    subgraph "Platform Level"
        HEADERS -->|7. Validate| TFO[TelemetryFlow API]
        TFO -->|8. Isolate| DB[(ClickHouse<br/>tenant_id column)]
    end

    style APP fill:#E1F5FE
    style AGENT fill:#FFE082
    style COLLECTOR fill:#81C784
    style TFO fill:#64B5F6,color:#fff
```

### Multi-Tenant Data Isolation

```mermaid
graph TB
    subgraph "OTEL Collector"
        RECV[OTLP Receiver]
        ATTR[Attributes Processor]
        EXPORT[OTLP Exporter]
    end

    subgraph "TelemetryFlow API"
        GUARD[Tenant Context Guard]
        VALIDATE[Validate Workspace/Tenant]
        TRANSFORM[Transform to Internal Format]
    end

    subgraph "ClickHouse Storage"
        TABLE[metrics_v3 Table]
        PART1[Partition: tenant_id=A<br/>workspace_id=W1]
        PART2[Partition: tenant_id=B<br/>workspace_id=W2]
        PART3[Partition: tenant_id=C<br/>workspace_id=W3]
    end

    RECV --> ATTR
    ATTR -->|Add tenant_id| EXPORT
    EXPORT -->|X-Tenant-Id: A| GUARD
    GUARD --> VALIDATE
    VALIDATE -->|Valid| TRANSFORM
    TRANSFORM --> TABLE
    TABLE --> PART1
    TABLE --> PART2
    TABLE --> PART3

    style GUARD fill:#F44336,color:#fff
    style PART1 fill:#4CAF50,color:#fff
    style PART2 fill:#2196F3,color:#fff
    style PART3 fill:#FF9800,color:#fff
```

---

## Deployment Patterns

### Pattern 1: Hub-and-Spoke (Recommended)

```mermaid
graph TB
    subgraph "Region: us-east-1"
        AGENT1[Agent - Host 1]
        AGENT2[Agent - Host 2]
        AGENT3[Agent - Host 3]
        COLL_EAST[Collector<br/>us-east-1<br/>3 replicas]
    end

    subgraph "Region: eu-west-1"
        AGENT4[Agent - Host 4]
        AGENT5[Agent - Host 5]
        COLL_EU[Collector<br/>eu-west-1<br/>3 replicas]
    end

    subgraph "Central"
        TFO[TelemetryFlow Platform<br/>Multi-Region]
    end

    AGENT1 & AGENT2 & AGENT3 --> COLL_EAST
    AGENT4 & AGENT5 --> COLL_EU
    COLL_EAST --> TFO
    COLL_EU --> TFO

    style AGENT1 fill:#FFE082
    style COLL_EAST fill:#81C784
    style COLL_EU fill:#81C784
    style TFO fill:#64B5F6,color:#fff
```

**Use When:**
- Multiple regions/datacenters
- Need centralized aggregation
- Want to optimize bandwidth

**Benefits:**
- Reduced backend connections
- Regional data aggregation
- Better resilience

### Pattern 2: Direct-to-Platform

```mermaid
graph TB
    AGENT1[Agent - Host 1]
    AGENT2[Agent - Host 2]
    AGENT3[Agent - Host 3]
    TFO[TelemetryFlow Platform]

    AGENT1 --> TFO
    AGENT2 --> TFO
    AGENT3 --> TFO

    style AGENT1 fill:#FFE082
    style TFO fill:#64B5F6,color:#fff
```

**Use When:**
- Small deployments (<50 hosts)
- Simple network topology
- Low data volume

**Benefits:**
- Simpler architecture
- Fewer components to manage
- Lower latency

### Pattern 3: Edge Buffering

```mermaid
graph TB
    subgraph "Edge Location (Intermittent Connectivity)"
        APP[Applications]
        AGENT[Agent with<br/>Persistent Queue<br/>10GB buffer]
    end

    subgraph "Cloud"
        COLLECTOR[Collector]
        TFO[TelemetryFlow]
    end

    APP -->|Always Available| AGENT
    AGENT -.->|When Connected| COLLECTOR
    COLLECTOR --> TFO

    style AGENT fill:#FFE082
    style COLLECTOR fill:#81C784
    style TFO fill:#64B5F6,color:#fff
```

**Use When:**
- Edge/IoT deployments
- Unreliable network
- Need data persistence

**Benefits:**
- Resilient to outages
- No data loss
- Automatic retry

---

## Scalability

### Horizontal Scaling

#### Agent Scaling

```
Scaling Model: Linear per host
Formula: 1 agent per host/VM/container

Capacity per Agent:
- 10,000 data points/sec
- 50MB RAM baseline
- +1MB per 1000 data points/sec

Example:
- 100 hosts × 1 agent = 100 agents
- Total capacity: 1M data points/sec
```

#### Collector Scaling

```
Scaling Model: Horizontal with load balancer
Formula: N collectors behind LB

Capacity per Collector:
- 100,000 data points/sec
- 512MB RAM baseline
- +100MB per 10,000 data points/sec

Example:
- 5 collectors × 100K = 500K data points/sec
- Total RAM: 5 × 512MB = 2.56GB
```

### Vertical Scaling

#### Resource Sizing

| Workload | Agent RAM | Agent CPU | Collector RAM | Collector CPU |
|----------|-----------|-----------|---------------|---------------|
| **Small** (<1K dps) | 50MB | 0.1 core | 256MB | 0.5 core |
| **Medium** (1K-10K dps) | 128MB | 0.5 core | 512MB | 1 core |
| **Large** (10K-100K dps) | 256MB | 1 core | 1GB | 2 cores |
| **Very Large** (>100K dps) | 512MB | 2 cores | 2GB | 4 cores |

```mermaid
graph LR
    subgraph "Small: 1K dps"
        S_AGENT[Agent<br/>50MB, 0.1 CPU]
        S_COLL[Collector<br/>256MB, 0.5 CPU]
    end

    subgraph "Medium: 10K dps"
        M_AGENT[Agent<br/>128MB, 0.5 CPU]
        M_COLL[Collector<br/>512MB, 1 CPU]
    end

    subgraph "Large: 100K dps"
        L_AGENT[Agent<br/>256MB, 1 CPU]
        L_COLL[Collector<br/>1GB, 2 CPU]
    end

    style S_AGENT fill:#C8E6C9
    style M_AGENT fill:#FFF9C4
    style L_AGENT fill:#FFCCBC
```

---

**Version:** 1.1.1-CE | **Maintained By:** DevOpsCorner Indonesia
