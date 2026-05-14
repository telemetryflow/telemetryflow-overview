# TFO-OTEL-Agent Documentation

- **Version:** 1.4.0
- **Last Updated:** May 2026
- **Component:** TelemetryFlow Agent (Infrastructure Agent)
- **Agent Version:** v1.2.0
- **Go Version:** 1.26+
- **OpenTelemetry SDK:** v1.43.0
- **Status:** Production Ready

---

## Table of Contents

1. [Overview](#overview)
2. [Features](#features)
3. [Installation](#installation)
4. [Configuration](#configuration)
5. [CLI Commands](#cli-commands)
6. [Auto-Registration](#auto-registration)
7. [Deployment Patterns](#deployment-patterns)
8. [Monitoring](#monitoring)
9. [High Availability](#high-availability)
10. [Troubleshooting](#troubleshooting)
11. [Best Practices](#best-practices)

---

## Overview

**TFO-Agent v1.2.0** is an enterprise-grade telemetry collection agent built on **OpenTelemetry Go SDK v1.43.0**. It consolidates multiple monitoring tools into a single Go binary, replacing Prometheus, kube-state-metrics, node-exporter, FluentBit, and cAdvisor.

The agent provides comprehensive infrastructure monitoring with **15+ built-in collectors** covering databases, containers, Kubernetes, eBPF, and system metrics, plus **39+ third-party integrations**.

### Key Characteristics

- **Custom Go Implementation**: Built with Go 1.26+, custom Cobra CLI
- **Lightweight**: ~50MB RAM, <1% CPU (idle), ~5% CPU (peak)
- **15+ Collectors**: System, Node Exporter, Kubernetes, cAdvisor, Docker, eBPF, 9 databases
- **39+ Integrations**: AWS, GCP, Azure, Proxmox, VMware, Cisco, MQTT, Datadog, Splunk
- **Disk-Backed Buffer**: Persistent queue for network outage resilience
- **Cross-Platform**: Linux, macOS, and Windows support

### Architecture Role

```mermaid
graph LR
    APP[Applications] -->|OTLP| AGENT[TFO-Agent v1.2.0]
    INFRA[Infrastructure<br/>DB, K8s, Docker, eBPF] -->|Native Collectors| AGENT
    LOGS[Log Files] -->|File Collector| AGENT

    AGENT -->|OTLP HTTP| COLLECTOR[TFO-Collector v1.2.1]
    AGENT -.->|Direct Mode| TFO[TelemetryFlow API]

    style AGENT fill:#FFE082,stroke:#F57C00,color:#000
    style COLLECTOR fill:#81C784,stroke:#388E3C,color:#000
    style TFO fill:#64B5F6,stroke:#1976D2,color:#fff
```

### Collectors Overview

```mermaid
graph TB
    subgraph Agent["TFO-Agent (Go 1.26, OTEL SDK v1.43.0)"]
        NE_MOD["Node Exporter<br/>CPU, Memory, DiskIO,<br/>Filesystem, Network"]
        K8S_MOD["Kubernetes<br/>Nodes, Pods, Deployments,<br/>Services, HPA, PDB"]
        CAD_MOD["cAdvisor / Docker<br/>Container CPU, Memory,<br/>Network, Filesystem<br/>32 per-container metrics"]
        DB_MOD["Databases (9)<br/>MySQL, PostgreSQL, MongoDB,<br/>MSSQL, ClickHouse, CockroachDB,<br/>Aurora, TimescaleDB, SQLite3"]
        EBPF_MOD["eBPF (28 metrics)<br/>Syscalls, Network,<br/>File I/O, Scheduler"]
        SYS_MOD["System<br/>CPU, Memory, Disk, Network"]
    end

    Agent -->|"OTLP"| PLATFORM["TFO Platform"]

    style Agent fill:#e8f5e9,stroke:#2e7d32,color:#000
```

### Project Structure

```
telemetryflow-agent/
├── cmd/tfo-agent/           # CLI entry point
│   └── main.go              # Cobra CLI with banner
├── internal/                # Core implementation (DDD)
│   ├── agent/               # Core agent lifecycle
│   ├── buffer/              # Disk-backed retry buffer
│   ├── collector/           # Metric collectors
│   │   ├── aurora/          # Amazon Aurora collector
│   │   ├── cadvisor/        # Docker containers (cAdvisor)
│   │   ├── clickhouse/      # ClickHouse DB collector
│   │   ├── cockroachdb/     # CockroachDB collector
│   │   ├── docker/          # Docker metrics collector
│   │   ├── ebpf/            # eBPF kernel-level metrics
│   │   ├── kubernetes/      # Kubernetes metrics
│   │   ├── mongodb/         # MongoDB collector
│   │   ├── mssql/           # Microsoft SQL Server
│   │   ├── mysql/           # MySQL / MariaDB collector
│   │   ├── nodeexporter/    # Node exporter metrics
│   │   ├── postgresql/      # PostgreSQL collector
│   │   ├── sqlite3/         # SQLite3 collector
│   │   ├── system/          # System metrics collector
│   │   └── timescaledb/     # TimescaleDB collector
│   ├── config/              # Configuration management
│   ├── exporter/            # OTLP data exporters (SDK v1.43.0)
│   └── version/             # Version and banner info
├── pkg/                     # LEGO Building Blocks
│   ├── api/                 # HTTP API client
│   ├── banner/              # Startup banner
│   ├── config/              # Config loader utilities
│   └── plugin/              # Plugin registry system
├── tests/                   # Comprehensive testing (DDD)
│   ├── unit/                # Unit tests
│   ├── integration/         # Integration tests
│   ├── e2e/                 # End-to-end tests
│   ├── mocks/               # Mock implementations
│   └── fixtures/            # Test fixtures
├── docs/                    # Documentation
├── configs/                 # Configuration templates
├── deploy/                  # Helm charts & K8s manifests
│   ├── helm/                # Helm chart
│   └── kubernetes/          # K8s manifests
├── scripts/                 # Build/install scripts
├── build/                   # Build output
├── Makefile
├── Dockerfile
├── docker-compose.yml
├── .env.example
└── README.md
```

---

## Features

### OpenTelemetry SDK v1.43.0

- **OTEL Native**: Built on OTEL Go SDK v1.43.0
- **OTLP Export**: OpenTelemetry Protocol for metrics, logs, and traces
- **Multi-Signal Support**: Metrics, logs, and traces collection

### Database Collectors (9 Databases)

| Database                  | Metrics                                | Features              |
| ------------------------- | -------------------------------------- | --------------------- |
| **MySQL/MariaDB/Percona** | InnoDB, replication, Galera            | Query Analytics (QAN) |
| **PostgreSQL**            | pg_stat_statements, activity, bgwriter | Query Analytics (QAN) |
| **Amazon Aurora**         | 60+ CloudWatch metrics via AWS SDK     | Performance Insights  |
| **MongoDB**               | Server status, replica set, sharding   | Profiler, QAN         |
| **MSSQL**                 | Wait stats, perf counters, query store | Index usage, QAN      |
| **ClickHouse**            | System metrics, query log              | Performance analysis  |
| **CockroachDB**           | Runtime metrics, SQL stats             | Range metrics, QAN    |
| **TimescaleDB**           | Hypertable stats, compression          | Chunk metrics         |
| **SQLite3**               | Database file stats, query performance | WAL metrics           |

### eBPF Metrics (28 Kernel-Level Metrics)

7 categories of kernel-level observability:

| Category      | Metrics | Description                          |
| ------------- | ------- | ------------------------------------ |
| **Syscalls**  | 6       | System call tracing and counts       |
| **Network**   | 5       | TCP/UDP connections, packet analysis |
| **File I/O**  | 5       | File read/write operations, latency  |
| **Scheduler** | 4       | CPU scheduling, context switches     |
| **Memory**    | 4       | Page faults, allocation tracking     |
| **Process**   | 2       | Process creation, execution          |
| **Security**  | 2       | Security event monitoring            |

### Docker / cAdvisor Monitoring

32 per-container metrics:

| Category       | Metrics                              |
| -------------- | ------------------------------------ |
| **CPU**        | Usage, throttling, periods, total    |
| **Memory**     | Usage, working set, RSS, page faults |
| **Network**    | Bytes/segments sent/received, errors |
| **Filesystem** | Usage, limit, available, inodes      |

### Kubernetes Monitoring

| Resource        | Metrics                                    |
| --------------- | ------------------------------------------ |
| **Nodes**       | CPU, memory, disk, network, conditions     |
| **Pods**        | Status, resource requests/limits, restarts |
| **Deployments** | Replicas, available, updated               |
| **Services**    | Endpoints, ports, selectors                |
| **HPA**         | Current/desired replicas, metrics          |
| **PDB**         | Disruptions allowed, current/desired       |
| **Ingresses**   | Rules, TLS, load balancer status           |

### 39+ Third-Party Integrations

| Category           | Integrations                      |
| ------------------ | --------------------------------- |
| **Cloud**          | AWS, GCP, Azure, Alibaba Cloud    |
| **APM**            | Datadog, Dynatrace, New Relic     |
| **Observability**  | Prometheus, Splunk, Elasticsearch |
| **Infrastructure** | Proxmox, VMware, Nutanix          |
| **Network**        | Cisco, SNMP, MQTT                 |
| **Streaming**      | Kafka, RabbitMQ, NATS             |

### Agent Lifecycle (Backend Integration)

- **Agent Registration**: Auto-register with TelemetryFlow backend
- **Heartbeat Monitoring**: Regular health checks to backend
- **Health Status Sync**: Report agent health and system info
- **Activation/Deactivation**: Remote agent control from backend

### LEGO Building Blocks

| Block        | Description                           |
| ------------ | ------------------------------------- |
| `pkg/banner` | ASCII art startup banner              |
| `pkg/config` | Flexible configuration loader         |
| `pkg/plugin` | Plugin registry for extensibility     |
| `pkg/api`    | HTTP client for backend communication |

---

## Installation

### Method 1: From Source

```bash
git clone https://github.com/telemetryflow/telemetryflow-agent.git
cd telemetryflow-agent
make build
./build/tfo-agent --help
```

### Method 2: Docker Compose (Recommended)

```bash
cp .env.example .env
vim .env
docker-compose up -d --build
docker-compose logs -f tfo-agent
```

### Method 3: Docker Directly

```bash
docker build \
  --build-arg VERSION=1.2.0 \
  --build-arg GIT_COMMIT=$(git rev-parse --short HEAD) \
  --build-arg GIT_BRANCH=$(git rev-parse --abbrev-ref HEAD) \
  --build-arg BUILD_TIME=$(date -u '+%Y-%m-%dT%H:%M:%SZ') \
  -t telemetryflow/telemetryflow-agent:1.2.0 .

docker run -d --name tfo-agent \
  -p 4317:4317 -p 4318:4318 -p 13133:13133 \
  -v /path/to/config.yaml:/etc/tfo-agent/tfo-agent.yaml:ro \
  -v /var/lib/tfo-agent:/var/lib/tfo-agent \
  telemetryflow/telemetryflow-agent:1.2.0
```

### Method 4: Kubernetes (DaemonSet)

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: observability

---
apiVersion: v1
kind: ConfigMap
metadata:
  name: tfo-agent-config
  namespace: observability
data:
  tfo-agent.yaml: |
    agent:
      description: "TelemetryFlow Agent - K8s"
      tags:
        environment: "production"
        cluster: "main"

    collectors:
      system:
        enabled: true
        interval: 30s
        cpu: { enabled: true }
        memory: { enabled: true }
        disk: { enabled: true }
        network: { enabled: true }

      kubernetes:
        enabled: true
        interval: 30s

      cadvisor:
        enabled: true
        interval: 30s

      ebpf:
        enabled: false

    receivers:
      otlp:
        enabled: true
        protocols:
          grpc: { enabled: true, endpoint: "0.0.0.0:4317" }
          http: { enabled: true, endpoint: "0.0.0.0:4318" }

    processors:
      batch:
        enabled: true
        send_batch_size: 8192
        timeout: 200ms
      memory_limiter:
        enabled: true
        limit_percentage: 80

    exporter:
      otlp:
        enabled: true
        endpoint: "http://tfo-collector.observability.svc.cluster.local:4317"
        compression: "gzip"

    buffer:
      enabled: true
      path: "/var/lib/tfo-agent/buffer"
      max_size_mb: 100

    heartbeat:
      enabled: true
      interval: 60s

    extensions:
      health_check:
        enabled: true
        endpoint: "0.0.0.0:13133"

---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: tfo-agent
  namespace: observability
  labels:
    app: tfo-agent
    version: "1.2.0"
spec:
  selector:
    matchLabels:
      app: tfo-agent
  template:
    metadata:
      labels:
        app: tfo-agent
        version: "1.2.0"
    spec:
      serviceAccountName: tfo-agent
      hostNetwork: true
      dnsPolicy: ClusterFirstWithHostNet

      containers:
        - name: tfo-agent
          image: telemetryflow/telemetryflow-agent:1.2.0
          imagePullPolicy: IfNotPresent
          args:
            - start
            - --config=/etc/tfo-agent/tfo-agent.yaml

          env:
            - name: HOSTNAME
              valueFrom:
                fieldRef:
                  fieldPath: spec.nodeName
            - name: HOST_IP
              valueFrom:
                fieldRef:
                  fieldPath: status.hostIP
            - name: K8S_NODE_NAME
              valueFrom:
                fieldRef:
                  fieldPath: spec.nodeName
            - name: TELEMETRYFLOW_API_ENDPOINT
              value: "https://api.telemetryflow.id"
            - name: TELEMETRYFLOW_API_KEY_ID
              valueFrom:
                secretKeyRef:
                  name: telemetryflow-secrets
                  key: api-key-id
            - name: TELEMETRYFLOW_API_KEY_SECRET
              valueFrom:
                secretKeyRef:
                  name: telemetryflow-secrets
                  key: api-key-secret

          ports:
            - name: otlp-grpc
              containerPort: 4317
              hostPort: 4317
            - name: otlp-http
              containerPort: 4318
              hostPort: 4318
            - name: health
              containerPort: 13133

          livenessProbe:
            httpGet: { path: /, port: 13133 }
            initialDelaySeconds: 30
            periodSeconds: 30

          readinessProbe:
            httpGet: { path: /, port: 13133 }
            initialDelaySeconds: 10
            periodSeconds: 10

          resources:
            requests: { memory: "128Mi", cpu: "100m" }
            limits: { memory: "256Mi", cpu: "500m" }

          volumeMounts:
            - name: config
              mountPath: /etc/tfo-agent
            - name: buffer-storage
              mountPath: /var/lib/tfo-agent

      volumes:
        - name: config
          configMap:
            name: tfo-agent-config
        - name: buffer-storage
          hostPath:
            path: /var/lib/tfo-agent
            type: DirectoryOrCreate

---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tfo-agent
  namespace: observability

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: tfo-agent
rules:
  - apiGroups: [""]
    resources:
      - nodes
      - nodes/proxy
      - services
      - endpoints
      - pods
    verbs: ["get", "list", "watch"]
  - apiGroups: ["apps"]
    resources:
      - deployments
      - replicasets
      - daemonsets
      - statefulsets
    verbs: ["get", "list", "watch"]
  - apiGroups: ["autoscaling"]
    resources: ["horizontalpodautoscalers"]
    verbs: ["get", "list", "watch"]
  - apiGroups: ["policy"]
    resources: ["poddisruptionbudgets"]
    verbs: ["get", "list", "watch"]
  - apiGroups: ["networking.k8s.io"]
    resources: ["ingresses"]
    verbs: ["get", "list", "watch"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: tfo-agent
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: tfo-agent
subjects:
  - kind: ServiceAccount
    name: tfo-agent
    namespace: observability
```

### Method 5: Systemd Service

```bash
sudo make install
sudo mkdir -p /etc/tfo-agent /var/lib/tfo-agent/buffer /var/log/tfo-agent
sudo cp configs/tfo-agent.yaml /etc/tfo-agent/
sudo useradd -r -s /bin/false telemetryflow
sudo chown -R telemetryflow:telemetryflow /etc/tfo-agent /var/lib/tfo-agent /var/log/tfo-agent
```

```ini
# /etc/systemd/system/tfo-agent.service
[Unit]
Description=TelemetryFlow Agent
After=network.target

[Service]
Type=simple
User=telemetryflow
Group=telemetryflow
ExecStart=/usr/local/bin/tfo-agent start --config /etc/tfo-agent/tfo-agent.yaml
Restart=always
RestartSec=5
NoNewPrivileges=true
PrivateTmp=true
ProtectSystem=strict
ProtectHome=true
ReadWritePaths=/var/lib/tfo-agent

[Install]
WantedBy=multi-user.target
```

---

## Configuration

### Configuration File Locations

1. Path specified via `--config` flag
2. `./configs/tfo-agent.yaml` (current directory)
3. `~/.tfo-agent/tfo-agent.yaml` (user home)
4. `/etc/tfo-agent/tfo-agent.yaml` (system)

### Minimal Configuration

```yaml
agent:
  description: "TelemetryFlow Agent"

collectors:
  system:
    enabled: true
    interval: 60s

exporter:
  otlp:
    enabled: true
    endpoint: "http://tfo-collector:4317"
```

### Production Configuration

```yaml
# TelemetryFlow Agent - Production Configuration

agent:
  id: ""
  hostname: ""
  description: "Production Agent"
  tags:
    environment: "production"
    datacenter: "dc1"

# Collectors
collectors:
  system:
    enabled: true
    interval: 30s
    cpu: { enabled: true, per_cpu: true }
    memory: { enabled: true }
    disk: { enabled: true, mount_points: ["/", "/data"] }
    network: { enabled: true, interfaces: ["eth0", "ens*"] }

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

  # Database Collectors (enable as needed)
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
    cluster_id: "my-cluster"

  timescaledb:
    enabled: false
    dsn: "postgresql://user:password@localhost:5432/timeseries"

  sqlite3:
    enabled: false
    path: "/path/to/database.db"

  logs:
    enabled: true
    paths: ["/var/log/app/*.log"]
    exclude_paths: ["/var/log/*.gz"]

# Receivers
receivers:
  otlp:
    enabled: true
    protocols:
      grpc: { enabled: true, endpoint: "0.0.0.0:4317" }
      http: { enabled: true, endpoint: "0.0.0.0:4318" }

# Processors
processors:
  batch:
    enabled: true
    send_batch_size: 8192
    timeout: 200ms

  memory_limiter:
    enabled: true
    limit_percentage: 80
    spike_limit_percentage: 25

  resource_detection:
    enabled: true
    detectors: [env, system, docker]

# Exporter
exporter:
  otlp:
    enabled: true
    endpoint: "http://tfo-collector:4317"
    compression: "gzip"
    retry:
      enabled: true
      initial_interval: 5s
      max_interval: 30s
    queue:
      enabled: true
      queue_size: 5000

# Buffer
buffer:
  enabled: true
  path: "/var/lib/tfo-agent/buffer"
  max_size_mb: 500
  flush_interval: 5s

# Heartbeat
heartbeat:
  enabled: true
  interval: 60s
  timeout: 10s

# Extensions
extensions:
  health_check:
    enabled: true
    endpoint: "0.0.0.0:13133"

# Logging
logging:
  level: "info"
  format: "json"
```

---

## CLI Commands

### Start Agent

```bash
tfo-agent start
tfo-agent start --config /path/to/config.yaml
TELEMETRYFLOW_LOG_LEVEL=debug tfo-agent start
```

### Validate Configuration

```bash
tfo-agent config validate --config /path/to/config.yaml
```

### Show Version

```bash
tfo-agent version

# Output:
# TelemetryFlow Agent
# Version:    1.2.0
# Git Commit: abc1234
# Git Branch: main
# Build Time: 2026-05-14T00:00:00Z
# Go Version: go1.26
# OTEL SDK:   v1.43.0
```

---

## Auto-Registration

```mermaid
sequenceDiagram
    participant Agent as TFO-Agent
    participant Backend as TelemetryFlow Backend
    participant DB as PostgreSQL

    Agent->>Agent: Start up
    Agent->>Agent: Load config
    Agent->>Backend: POST /api/v2/agents/register
    Backend->>DB: INSERT INTO agents
    DB-->>Backend: Agent ID
    Backend-->>Agent: 201 Created {agent_id}

    loop Every 60 seconds
        Agent->>Backend: POST /api/v2/agents/:id/heartbeat
        Backend->>DB: UPDATE agents SET last_seen_at
        DB-->>Backend: OK
        Backend-->>Agent: 200 OK
    end
```

### Registration Payload

```json
{
  "agent_id": "auto-generated-uuid",
  "version": "1.2.0",
  "hostname": "prod-node-01",
  "ip_address": "10.0.1.15",
  "capabilities": [
    "otlp_grpc",
    "otlp_http",
    "system_metrics",
    "kubernetes",
    "cadvisor",
    "ebpf",
    "mysql",
    "postgresql",
    "mongodb"
  ],
  "resource_attributes": {
    "os.type": "linux",
    "os.description": "Ubuntu 22.04",
    "host.arch": "amd64"
  },
  "tags": {
    "environment": "production",
    "datacenter": "dc1"
  }
}
```

---

## Deployment Patterns

### Pattern 1: Per-Host Agent

```mermaid
graph TB
    subgraph HOST[Physical/Virtual Host]
        AGENT[TFO-Agent v1.2.0]
        APP1[App 1] -->|OTLP| AGENT
        APP2[App 2] -->|OTLP| AGENT
        DB[(Database)] -->|Native Collector| AGENT
    end

    COLLECTOR[TFO-Collector v1.2.1]
    AGENT -->|OTLP HTTP| COLLECTOR

    style AGENT fill:#FFE082,stroke:#F57C00,color:#000
    style COLLECTOR fill:#81C784,stroke:#388E3C,color:#000
```

### Pattern 2: Kubernetes DaemonSet

See [Kubernetes Installation](#method-4-kubernetes-daemonset) for full manifest.

### Pattern 3: Sidecar Container

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-application
spec:
  template:
    spec:
      containers:
        - name: app
          image: my-app:latest
          env:
            - name: OTEL_EXPORTER_OTLP_ENDPOINT
              value: "http://localhost:4318"

        - name: tfo-agent
          image: telemetryflow/telemetryflow-agent:1.2.0
          args: ["start", "--config=/etc/tfo-agent/config.yaml"]
          ports:
            - containerPort: 4317
            - containerPort: 4318
          resources:
            requests: { memory: "64Mi", cpu: "50m" }
            limits: { memory: "128Mi", cpu: "200m" }
```

### Pattern 4: Edge Gateway

```yaml
buffer:
  enabled: true
  path: "/var/lib/tfo-agent/buffer"
  max_size_mb: 500

exporter:
  otlp:
    enabled: true
    endpoint: "https://cloud-collector:4317"
    retry:
      enabled: true
      initial_interval: 10s
      max_interval: 300s
      max_elapsed_time: 3600s
```

---

## Monitoring

### System Metrics

| Metric                      | Type    | Description              |
| --------------------------- | ------- | ------------------------ |
| `system.cpu.usage`          | gauge   | CPU usage percentage     |
| `system.cpu.cores`          | gauge   | Number of CPU cores      |
| `system.memory.total`       | gauge   | Total memory (bytes)     |
| `system.memory.used`        | gauge   | Used memory (bytes)      |
| `system.memory.usage`       | gauge   | Memory usage percentage  |
| `system.disk.total`         | gauge   | Total disk space (bytes) |
| `system.disk.used`          | gauge   | Used disk space (bytes)  |
| `system.disk.usage`         | gauge   | Disk usage percentage    |
| `system.network.bytes_sent` | counter | Total bytes sent         |
| `system.network.bytes_recv` | counter | Total bytes received     |

### Database Metrics (Per Collector)

| Collector  | Example Metrics                                             |
| ---------- | ----------------------------------------------------------- |
| MySQL      | `mysql.innoDB.buffer_pool.size`, `mysql.connections.active` |
| PostgreSQL | `pg.stat.statements.calls`, `pg.bgwriter.checkpoints`       |
| MongoDB    | `mongodb.server.connections`, `mongodb.replication.lag`     |
| MSSQL      | `mssql.wait_stats.wait_time_ms`, `mssql.perf.counter`       |
| ClickHouse | `clickhouse.query.count`, `clickhouse.part.count`           |

### eBPF Metrics

| Metric                            | Category  | Description                |
| --------------------------------- | --------- | -------------------------- |
| `ebpf.syscall.count`              | Syscalls  | System call counts by type |
| `ebpf.network.tcp.connections`    | Network   | Active TCP connections     |
| `ebpf.fileio.read.bytes`          | File I/O  | Bytes read from files      |
| `ebpf.scheduler.context_switches` | Scheduler | Context switch count       |

### Health Check

```bash
curl http://localhost:13133/
```

---

## High Availability

### Persistent Buffer

```yaml
buffer:
  enabled: true
  path: "/var/lib/tfo-agent/buffer"
  max_size_mb: 500
```

### Retry Configuration

```yaml
exporter:
  otlp:
    retry:
      enabled: true
      initial_interval: 5s
      max_interval: 30s
      max_elapsed_time: 300s
```

### Graceful Shutdown

- **SIGINT/SIGTERM**: Flush buffers and exit
- **SIGHUP**: Reload configuration (hot reload)

---

## Troubleshooting

### Agent Not Starting

```bash
docker logs tfo-agent
sudo journalctl -u tfo-agent -f
tfo-agent config validate --config /etc/tfo-agent/tfo-agent.yaml
```

### Agent Not Registering

```bash
curl -v https://api.telemetryflow.id/health
echo $TELEMETRYFLOW_API_KEY_ID
echo $TELEMETRYFLOW_API_KEY_SECRET
```

### High Memory Usage

```yaml
processors:
  memory_limiter:
    limit_percentage: 60
  batch:
    send_batch_size: 256
```

### Buffer Filling Up

```bash
du -sh /var/lib/tfo-agent/buffer
```

---

## Best Practices

1. **Enable Memory Limiter**: `limit_percentage: 80`
2. **Configure Disk Buffer**: `max_size_mb: 500`
3. **Use Compression**: `compression: "gzip"` (~70% bandwidth reduction)
4. **Enable Heartbeat**: `interval: 60s`
5. **Use Environment Variables for Secrets**: `${TFO_API_KEY}`
6. **Enable Relevant Collectors Only**: Disable unused database/eBPF collectors

---

## Development

```bash
make help
make build              # Build agent
make build-all          # Build for all platforms
make run                # Build and run
make dev                # Run with go run
make test               # Run tests
make test-coverage      # Run with coverage
make lint               # Run linter
make docker-build       # Build Docker image
```

---

## Links

- **Website**: [https://telemetryflow.id](https://telemetryflow.id)
- **Documentation**: [https://docs.telemetryflow.id](https://docs.telemetryflow.id)
- **Repository**: [https://github.com/telemetryflow/telemetryflow-agent](https://github.com/telemetryflow/telemetryflow-agent)
- **Developer**: [DevOpsCorner Indonesia](https://devopscorner.id)

---

**Version:** 1.4.0 | **Component:** TFO-Agent v1.2.0 | **OTEL SDK:** v1.43.0 | **Last Updated:** May 2026
