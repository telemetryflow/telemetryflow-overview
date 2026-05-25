# TFO-OTEL Configuration Reference

- **Version:** 1.4.0
- **Last Updated:** May 2026
- **Component:** Configuration Guide
- **Collector:** OCB Native, OTEL Core v1.58.0, Contrib v0.152.0

---

## Table of Contents

1. [Overview](#overview)
2. [Configuration Structure](#configuration-structure)
3. [TFO Custom Components](#tfo-custom-components)
4. [Receivers](#receivers)
5. [Processors](#processors)
6. [Exporters](#exporters)
7. [Connectors](#connectors)
8. [Extensions](#extensions)
9. [Service Pipelines](#service-pipelines)
10. [Agent Collectors](#agent-collectors)
11. [Environment Variables](#environment-variables)
12. [Configuration Examples](#configuration-examples)
13. [Validation](#validation)
14. [Best Practices](#best-practices)

---

## Overview

TFO-OTEL uses YAML configuration files. The Collector uses **standard OTEL YAML format** (OCB native), while the Agent uses a **custom YAML format** with `enabled` flags.

```mermaid
graph TB
    CONFIG[Configuration File]

    CONFIG --> RECEIVERS[Receivers<br/>Data Sources]
    CONFIG --> PROCESSORS[Processors<br/>Data Transformation]
    CONFIG --> EXPORTERS[Exporters<br/>Data Destinations]
    CONFIG --> CONNECTORS[Connectors<br/>Derived Telemetry]
    CONFIG --> EXTENSIONS[Extensions<br/>Additional Services]
    CONFIG --> SERVICE[Service<br/>Pipeline Definition]

    style CONFIG fill:#E3F2FD,stroke:#1976D2,color:#000
    style RECEIVERS fill:#FFE082,stroke:#F57C00,color:#000
    style PROCESSORS fill:#81C784,stroke:#388E3C,color:#000
    style EXPORTERS fill:#64B5F6,stroke:#1976D2,color:#fff
    style CONNECTORS fill:#FF8A65,stroke:#D84315,color:#fff
    style EXTENSIONS fill:#CE93D8,stroke:#7B1FA2,color:#fff
```

---

## Configuration Structure

### Collector (Standard OTEL Format)

```yaml
receivers:
  <receiver-name>: <receiver-config>

processors:
  <processor-name>: <processor-config>

exporters:
  <exporter-name>: <exporter-config>

connectors:
  <connector-name>: <connector-config>

extensions:
  <extension-name>: <extension-config>

service:
  extensions: [<extension-list>]
  pipelines:
    traces:
      receivers: [<receiver-list>]
      processors: [<processor-list>]
      exporters: [<exporter-list>]
    metrics:
      receivers: [<receiver-list>]
      processors: [<processor-list>]
      exporters: [<exporter-list>]
    logs:
      receivers: [<receiver-list>]
      processors: [<processor-list>]
      exporters: [<exporter-list>]
```

---

## TFO Custom Components

### tfootlp (Receiver)

OTLP receiver with **v1/v2 dual endpoint support**:

```yaml
receivers:
  tfootlp:
    protocols:
      http:
        endpoint: "0.0.0.0:4318"
        cors:
          allowed_origins:
            - "https://app.telemetryflow.id"
          allowed_headers:
            - "X-API-Key"
            - "X-Workspace-Id"
            - "X-Tenant-Id"
      grpc:
        endpoint: "0.0.0.0:4317"
        max_recv_msg_size_mib: 4
```

| Endpoint       | Path                                    | Auth    | Description   |
| -------------- | --------------------------------------- | ------- | ------------- |
| v1 (Community) | `/v1/metrics`, `/v1/logs`, `/v1/traces` | Open    | Standard OTEL |
| v2 (Platform)  | `/v2/metrics`, `/v2/logs`, `/v2/traces` | tfoauth | TFO Platform  |

### tfo (Exporter)

Platform exporter with automatic authentication:

```yaml
exporters:
  tfo:
    endpoint: "http://telemetryflow-api:3000/api"
    auth:
      authenticator: tfoauth
    compression: gzip
    timeout: 30s
    retry_on_failure:
      enabled: true
      initial_interval: 5s
      max_interval: 30s
      max_elapsed_time: 300s
    sending_queue:
      enabled: true
      num_consumers: 10
      queue_size: 5000
```

### tfoauth (Extension)

Centralized API key management:

```yaml
extensions:
  tfoauth:
    api_key_id: "${env:TELEMETRYFLOW_API_KEY_ID}"
    api_key_secret: "${env:TELEMETRYFLOW_API_KEY_SECRET}"
    # AWS-style dual keys:
    #   tfk-live-abc123 (key ID)
    #   tfs-secret-xyz789 (key secret)
    # Platform verifies with Argon2id hash
```

### tfoidentity (Extension)

Collector identity and resource enrichment:

```yaml
extensions:
  tfoidentity:
    collector_id: "tfo-collector-prod-01"
    labels:
      environment: "production"
      region: "us-east-1"
      cluster: "main"
    # Adds to every telemetry item:
    #   collector.id = "tfo-collector-prod-01"
    #   collector.environment = "production"
    #   collector.region = "us-east-1"
```

---

## Receivers

### OTLP Receiver (Standard)

```yaml
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: "0.0.0.0:4317"
        max_recv_msg_size_mib: 4
      http:
        endpoint: "0.0.0.0:4318"
```

### Prometheus Receiver

```yaml
receivers:
  prometheus:
    config:
      scrape_configs:
        - job_name: "telemetryflow-api"
          scrape_interval: 15s
          static_configs:
            - targets: ["telemetryflow-api:3000"]
```

### Filelog Receiver

```yaml
receivers:
  filelog:
    include: ["/var/log/app/*.log"]
    operators:
      - type: json_parser
        timestamp:
          parse_from: attributes.time
```

### Kafka Receiver

```yaml
receivers:
  kafka:
    brokers: ["kafka-1:9092", "kafka-2:9092"]
    topic: telemetry-metrics
    auth:
      sasl:
        mechanism: SCRAM-SHA-512
```

---

## Processors

### Batch Processor

```yaml
processors:
  batch:
    send_batch_size: 8192
    timeout: 200ms
```

### Memory Limiter

```yaml
processors:
  memory_limiter:
    check_interval: 1s
    limit_mib: 2048
    spike_limit_mib: 512
```

### k8sattributes Processor

```yaml
processors:
  k8sattributes:
    auth_type: "serviceAccount"
    passthrough: false
    extract:
      metadata:
        - k8s.pod.name
        - k8s.namespace.name
        - k8s.node.name
        - k8s.deployment.name
    pod_association:
      - sources:
          - from: resource_attribute
            name: k8s.pod.ip
```

### Transform Processor

```yaml
processors:
  transform:
    error_mode: ignore
    metric_statements:
      - context: metric
        statements:
          - set(description, "") where name == ""
          - set(unit, "By") where name == "process.runtime.jvm.memory.usage"
```

### Attributes Processor

```yaml
processors:
  attributes:
    actions:
      - key: telemetryflow.workspace.id
        value: ${env:TELEMETRYFLOW_WORKSPACE_ID}
        action: upsert
      - key: telemetryflow.tenant.id
        value: ${env:TELEMETRYFLOW_TENANT_ID}
        action: upsert
```

### Filter Processor

```yaml
processors:
  filter:
    traces:
      exclude:
        match_type: strict
        span_names: ["/health", "/metrics"]
```

### Tail Sampling

```yaml
processors:
  tail_sampling:
    decision_wait: 10s
    policies:
      - name: error-policy
        type: status_code
        status_code:
          status_codes: [ERROR]
      - name: probabilistic-policy
        type: probabilistic
        probabilistic:
          sampling_percentage: 10
```

---

## Exporters

### tfo Exporter (TFO Platform)

```yaml
exporters:
  tfo:
    endpoint: "http://telemetryflow-api:3000/api"
    auth:
      authenticator: tfoauth
    compression: gzip
    retry_on_failure:
      enabled: true
      initial_interval: 5s
      max_interval: 30s
    sending_queue:
      enabled: true
      num_consumers: 10
      queue_size: 5000
```

### OTLP HTTP Exporter

```yaml
exporters:
  otlphttp:
    endpoint: "http://backend:3000/api"
    headers:
      X-Workspace-Id: "${env:TELEMETRYFLOW_WORKSPACE_ID}"
    compression: gzip
```

### Prometheus Exporter

```yaml
exporters:
  prometheus:
    endpoint: "0.0.0.0:8889"
    namespace: telemetryflow
```

### Loki Exporter

```yaml
exporters:
  loki:
    endpoint: "http://loki:3100/loki/api/v1/push"
    labels:
      resource:
        service.name: "service_name"
```

### Debug Exporter

```yaml
exporters:
  debug:
    verbosity: detailed
```

---

## Connectors

### spanmetrics Connector

Generates metrics from trace spans with exemplars:

```yaml
connectors:
  spanmetrics:
    histogram:
      explicit:
        buckets: [2ms, 6ms, 10ms, 50ms, 100ms, 250ms, 500ms, 1s, 5s, 10s]
    dimensions:
      - name: http.method
      - name: http.status_code
      - name: http.route
    aggregation_temporality: "AGGREGATION_TEMPORALITY_CUMULATIVE"
    metrics_flush_interval: 15s
```

### servicegraph Connector

Generates service dependency maps from traces:

```yaml
connectors:
  service_graph:
    latency_histogram_buckets: [100ms, 250ms, 500ms, 1s, 5s, 10s]
    dimensions: []
    store:
      ttl: 2s
      max_items: 200
```

---

## Extensions

### Health Check

```yaml
extensions:
  health_check:
    endpoint: "0.0.0.0:13133"
```

### File Storage (Persistent Queue)

```yaml
extensions:
  file_storage:
    directory: /var/lib/otelcol/queue
    timeout: 10s
    compaction:
      on_start: true
      on_rebound: true
```

### pprof (Profiling)

```yaml
extensions:
  pprof:
    endpoint: "0.0.0.0:1777"
```

---

## Service Pipelines

### Full Production Pipeline

```yaml
service:
  extensions: [tfoauth, tfoidentity, health_check]

  pipelines:
    traces:
      receivers: [tfootlp]
      processors: [k8sattributes, batch]
      exporters: [tfo, spanmetrics, servicegraph]

    metrics:
      receivers: [tfootlp]
      processors: [k8sattributes, transform, batch]
      exporters: [tfo, prometheus]

    logs:
      receivers: [tfootlp]
      processors: [k8sattributes, batch]
      exporters: [tfo]
```

### Processing Order

```
Receivers → Processor 1 → Processor 2 → ... → Exporters
```

Best practice order:

1. `memory_limiter` — First line of defense
2. `k8sattributes` — Add Kubernetes metadata
3. `transform` — Transform data
4. `batch` — Always last for efficiency

---

## Agent Collectors

TFO-Agent uses `enabled` flags for collector configuration:

```yaml
collectors:
  system:
    enabled: true
    interval: 30s
    cpu: { enabled: true }
    memory: { enabled: true }
    disk: { enabled: true }
    network: { enabled: true }

  nodeexporter:
    enabled: true
    interval: 30s

  kubernetes:
    enabled: true
    interval: 30s

  cadvisor:
    enabled: true
    interval: 30s

  docker:
    enabled: true
    interval: 30s

  ebpf:
    enabled: false

  mysql:
    enabled: false
    dsn: "user:password@tcp(localhost:3306)/"

  postgresql:
    enabled: false
    dsn: "postgresql://user:password@localhost:5432/postgres"

  mongodb:
    enabled: false
    uri: "mongodb://localhost:27017"

  mssql:
    enabled: false
    dsn: "sqlserver://user:password@localhost:1433"

  clickhouse:
    enabled: false
    dsn: "clickhouse://localhost:9000"

  cockroachdb:
    enabled: false
    dsn: "postgresql://user:password@localhost:26257/defaultdb"

  aurora:
    enabled: false
    region: "us-east-1"

  timescaledb:
    enabled: false
    dsn: "postgresql://user:password@localhost:5432/timeseries"

  sqlite3:
    enabled: false
    path: "/path/to/database.db"
```

---

## Environment Variables

### TelemetryFlow Context

| Variable                       | Required | Description                | Example                   |
| ------------------------------ | -------- | -------------------------- | ------------------------- |
| `TELEMETRYFLOW_API_KEY_ID`     | Yes      | API key ID (tfk-\*)        | `tfk-live-abc123`         |
| `TELEMETRYFLOW_API_KEY_SECRET` | Yes      | API key secret (tfs-\*)    | `tfs-secret-xyz789`       |
| `TELEMETRYFLOW_WORKSPACE_ID`   | Yes      | Workspace UUID             | `550e8400-...`            |
| `TELEMETRYFLOW_TENANT_ID`      | Yes      | Tenant UUID                | `660e8400-...`            |
| `TELEMETRYFLOW_API_ENDPOINT`   | Yes      | TelemetryFlow API endpoint | `http://tfo-api:3000/api` |

### Host Information

| Variable        | Description          |
| --------------- | -------------------- |
| `HOSTNAME`      | Host name            |
| `HOST_IP`       | Host IP address      |
| `K8S_NODE_NAME` | Kubernetes node name |

### Variable Substitution

```yaml
exporters:
  tfo:
    endpoint: ${env:TELEMETRYFLOW_API_ENDPOINT:-http://localhost:3000/api}
```

---

## Configuration Examples

### Example 1: Complete Collector (OCB Native)

```yaml
receivers:
  tfootlp:
    protocols:
      http:
        endpoint: "0.0.0.0:4318"
        cors:
          allowed_origins: ["https://app.telemetryflow.id"]
      grpc:
        endpoint: "0.0.0.0:4317"

  prometheus:
    config:
      scrape_configs:
        - job_name: "self-monitoring"
          scrape_interval: 30s
          static_configs:
            - targets: ["localhost:8888"]

processors:
  memory_limiter:
    check_interval: 1s
    limit_mib: 2048
    spike_limit_mib: 512

  k8sattributes:
    auth_type: "serviceAccount"
    extract:
      metadata:
        - k8s.pod.name
        - k8s.namespace.name
        - k8s.node.name

  transform:
    error_mode: ignore
    metric_statements:
      - context: metric
        statements:
          - set(description, "") where name == ""

  batch:
    send_batch_size: 8192
    timeout: 200ms

exporters:
  tfo:
    endpoint: "http://telemetryflow-api:3000/api"
    auth:
      authenticator: tfoauth
    compression: gzip
    retry_on_failure:
      enabled: true
      initial_interval: 5s
      max_interval: 30s
    sending_queue:
      enabled: true
      num_consumers: 10
      queue_size: 5000

  prometheus:
    endpoint: "0.0.0.0:8889"

connectors:
  spanmetrics:
    histogram:
      explicit:
        buckets: [2ms, 6ms, 10ms, 50ms, 100ms, 250ms, 500ms, 1s, 5s, 10s]
    dimensions:
      - name: http.method
      - name: http.status_code
    aggregation_temporality: "AGGREGATION_TEMPORALITY_CUMULATIVE"

  service_graph:
    latency_histogram_buckets: [100ms, 250ms, 500ms, 1s, 5s, 10s]

extensions:
  tfoauth:
    api_key_id: "${env:TELEMETRYFLOW_API_KEY_ID}"
    api_key_secret: "${env:TELEMETRYFLOW_API_KEY_SECRET}"

  tfoidentity:
    collector_id: "tfo-collector-prod"
    labels:
      environment: "production"

  health_check:
    endpoint: "0.0.0.0:13133"

service:
  extensions: [tfoauth, tfoidentity, health_check]

  pipelines:
    traces:
      receivers: [tfootlp]
      processors: [memory_limiter, k8sattributes, batch]
      exporters: [tfo, spanmetrics, servicegraph]

    metrics:
      receivers: [tfootlp, prometheus]
      processors: [memory_limiter, k8sattributes, transform, batch]
      exporters: [tfo, prometheus]

    logs:
      receivers: [tfootlp]
      processors: [memory_limiter, k8sattributes, batch]
      exporters: [tfo]
```

---

## Validation

```bash
# Validate config
tfo-collector validate --config /etc/tfo-collector/otel-collector.yaml

# Dry run
tfo-collector --config /etc/tfo-collector/otel-collector.yaml --dry-run
```

---

## Best Practices

1. **Always Use tfoauth**: Configure API key authentication for v2 endpoints
2. **Memory Management**: Set `limit_mib` to 80% of container memory
3. **Batching**: `send_batch_size: 8192`, `timeout: 200ms` for balanced performance
4. **Compression**: Always enable `compression: gzip` (70-90% reduction)
5. **Persistent Queue**: Configure `sending_queue` with `file_storage`
6. **Kubernetes Metadata**: Enable `k8sattributes` processor
7. **Derived Telemetry**: Enable `spanmetrics` + `servicegraph` connectors
8. **Collector Identity**: Configure `tfoidentity` for resource enrichment

---

**Version:** 1.4.0 | **Component:** Configuration Reference | **Last Updated:** May 2026
