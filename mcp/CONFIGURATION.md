# TFO-MCP Configuration Guide

> Complete configuration reference for TelemetryFlow MCP Server

---

## Table of Contents

- [Overview](#overview)
- [Configuration Architecture](#configuration-architecture)
- [Configuration File](#configuration-file)
- [Environment Variables](#environment-variables)
- [Server Configuration](#server-configuration)
- [Claude API Configuration](#claude-api-configuration)
- [MCP Protocol Configuration](#mcp-protocol-configuration)
- [Logging Configuration](#logging-configuration)
- [Telemetry Configuration](#telemetry-configuration)
- [Security Configuration](#security-configuration)
- [Configuration Validation](#configuration-validation)
- [Configuration Examples](#configuration-examples)
- [Best Practices](#best-practices)

---

## Overview

TFO-MCP uses a hierarchical configuration system that supports multiple sources with precedence rules.

### Configuration Hierarchy

```mermaid
flowchart TD
    subgraph Sources["Configuration Sources"]
        ENV["Environment Variables<br/>(Highest Priority)"]
        CLI["CLI Flags"]
        FILE["Config File<br/>(config.yaml)"]
        DEFAULT["Default Values<br/>(Lowest Priority)"]
    end

    subgraph Loader["Configuration Loader"]
        VIPER["Viper Config Manager"]
        VALIDATE["Validation Layer"]
        MERGE["Configuration Merger"]
    end

    subgraph Output["Final Configuration"]
        CONFIG["Unified Config Object"]
    end

    ENV --> VIPER
    CLI --> VIPER
    FILE --> VIPER
    DEFAULT --> VIPER
    VIPER --> MERGE
    MERGE --> VALIDATE
    VALIDATE --> CONFIG

    style ENV fill:#e8f5e9,stroke:#4caf50
    style CONFIG fill:#e3f2fd,stroke:#2196f3
```

### Configuration Loading Process

```mermaid
sequenceDiagram
    participant Main as main.go
    participant Viper as Viper Manager
    participant File as Config File
    participant Env as Environment
    participant Validator as Validator

    Main->>Viper: Initialize()
    Viper->>Viper: Set defaults
    Viper->>File: Read config.yaml
    File-->>Viper: File contents
    Viper->>Env: Bind environment variables
    Env-->>Viper: Environment values
    Viper->>Viper: Merge configurations
    Viper->>Validator: Validate()
    Validator-->>Viper: Validation result
    Viper-->>Main: Config object
```

---

## Configuration Architecture

### Configuration Domains

```mermaid
flowchart TB
    subgraph Config["TFO-MCP Configuration"]
        SERVER["Server Config"]
        CLAUDE["Claude Config"]
        MCP["MCP Config"]
        LOGGING["Logging Config"]
        TELEMETRY["Telemetry Config"]
        SECURITY["Security Config"]
    end

    subgraph Server["Server Settings"]
        S1["Name"]
        S2["Version"]
        S3["Description"]
        S4["Timeout"]
    end

    subgraph Claude["Claude Settings"]
        C1["API Key"]
        C2["Model"]
        C3["Max Tokens"]
        C4["Temperature"]
        C5["Retry Config"]
    end

    subgraph MCPSettings["MCP Settings"]
        M1["Protocol Version"]
        M2["Capabilities"]
        M3["Transport"]
    end

    SERVER --> S1 & S2 & S3 & S4
    CLAUDE --> C1 & C2 & C3 & C4 & C5
    MCP --> M1 & M2 & M3

    style Config fill:#fff3e0,stroke:#ff9800
    style SERVER fill:#e3f2fd,stroke:#2196f3
    style CLAUDE fill:#fce4ec,stroke:#e91e63
    style MCP fill:#e8f5e9,stroke:#4caf50
```

### Configuration Structure

```mermaid
classDiagram
    class Config {
        +ServerConfig Server
        +ClaudeConfig Claude
        +MCPConfig MCP
        +LoggingConfig Logging
        +TelemetryConfig Telemetry
        +SecurityConfig Security
        +Validate() error
    }

    class ServerConfig {
        +string Name
        +string Version
        +string Description
        +Duration Timeout
    }

    class ClaudeConfig {
        +string APIKey
        +string Model
        +int MaxTokens
        +float64 Temperature
        +RetryConfig Retry
    }

    class MCPConfig {
        +string ProtocolVersion
        +CapabilitiesConfig Capabilities
        +TransportConfig Transport
    }

    class LoggingConfig {
        +string Level
        +string Format
        +string Output
        +bool Caller
    }

    class TelemetryConfig {
        +bool Enabled
        +string ServiceName
        +string Endpoint
        +float64 SampleRate
    }

    class SecurityConfig {
        +bool RateLimitEnabled
        +int RequestsPerMinute
        +[]string AllowedOrigins
    }

    Config --> ServerConfig
    Config --> ClaudeConfig
    Config --> MCPConfig
    Config --> LoggingConfig
    Config --> TelemetryConfig
    Config --> SecurityConfig
```

---

## Configuration File

### Default Configuration Location

```mermaid
flowchart LR
    subgraph Locations["Config File Locations"]
        CWD["./config.yaml<br/>(Current Directory)"]
        HOME["~/.tfo-mcp/config.yaml<br/>(Home Directory)"]
        ETC["/etc/tfo-mcp/config.yaml<br/>(System)"]
    end

    subgraph Priority["Search Order"]
        P1["1. Current Dir"]
        P2["2. Home Dir"]
        P3["3. System Dir"]
    end

    CWD --> P1
    HOME --> P2
    ETC --> P3

    style CWD fill:#e8f5e9,stroke:#4caf50
```

### Complete Configuration File

```yaml
# ==============================================================================
# TFO-MCP Configuration File
# ==============================================================================

# Server Configuration
server:
  name: "tfo-mcp"
  version: "1.1.2"
  description: "TelemetryFlow MCP Server - Claude AI Integration"
  timeout: 30s

# Claude API Configuration
claude:
  api_key: "" # Use TELEMETRYFLOW_MCP_CLAUDE_API_KEY environment variable
  base_url: "https://api.anthropic.com"
  model: "claude-sonnet-4-20250514"
  max_tokens: 4096
  temperature: 0.7
  top_p: 0.9
  top_k: 40
  retry:
    max_attempts: 3
    initial_delay: 1s
    max_delay: 30s
    multiplier: 2.0

# MCP Protocol Configuration
mcp:
  protocol_version: "2024-11-05"
  capabilities:
    tools: true
    resources: true
    prompts: true
    logging: true
  transport:
    type: "stdio"
    buffer_size: 65536

# Logging Configuration
logging:
  level: "info"
  format: "json"
  output: "stderr"
  caller: false
  timestamp_format: "2006-01-02T15:04:05.000Z07:00"

# Telemetry Configuration (OpenTelemetry)
telemetry:
  enabled: false
  service_name: "tfo-mcp"
  service_version: "1.1.2"
  environment: "development"
  endpoint: "localhost:4317"
  sample_rate: 1.0
  export_timeout: 30s

# Security Configuration
security:
  rate_limit:
    enabled: true
    requests_per_minute: 60
    burst_size: 10
  cors:
    enabled: false
    allowed_origins:
      - "*"
  api_key_validation: true
```

---

## Environment Variables

### Environment Variable Mapping

```mermaid
flowchart LR
    subgraph EnvVars["Environment Variables"]
        E1["TELEMETRYFLOW_MCP_CLAUDE_API_KEY"]
        E2["TELEMETRYFLOW_MCP_CLAUDE_MODEL"]
        E3["TELEMETRYFLOW_MCP_LOG_LEVEL"]
        E4["TELEMETRYFLOW_MCP_TELEMETRY_ENABLED"]
        E5["TELEMETRYFLOW_MCP_SERVER_TIMEOUT"]
    end

    subgraph Config["Configuration Fields"]
        C1["claude.api_key"]
        C2["claude.model"]
        C3["logging.level"]
        C4["telemetry.enabled"]
        C5["server.timeout"]
    end

    E1 --> C1
    E2 --> C2
    E3 --> C3
    E4 --> C4
    E5 --> C5

    style EnvVars fill:#fff3e0,stroke:#ff9800
    style Config fill:#e3f2fd,stroke:#2196f3
```

### Complete Environment Variables Reference

| Variable                               | Config Path                               | Type     | Default                     | Description               |
| -------------------------------------- | ----------------------------------------- | -------- | --------------------------- | ------------------------- |
| `TELEMETRYFLOW_MCP_CLAUDE_API_KEY`     | `claude.api_key`                          | string   | ""                          | Claude API key (required) |
| `TELEMETRYFLOW_MCP_CLAUDE_BASE_URL`    | `claude.base_url`                         | string   | "https://api.anthropic.com" | Claude API base URL       |
| `TELEMETRYFLOW_MCP_CLAUDE_MODEL`       | `claude.model`                            | string   | "claude-sonnet-4-20250514"  | Default Claude model      |
| `TELEMETRYFLOW_MCP_CLAUDE_MAX_TOKENS`  | `claude.max_tokens`                       | int      | 4096                        | Maximum response tokens   |
| `TELEMETRYFLOW_MCP_CLAUDE_TEMPERATURE` | `claude.temperature`                      | float    | 0.7                         | Response temperature      |
| `TELEMETRYFLOW_MCP_SERVER_NAME`        | `server.name`                             | string   | "tfo-mcp"                   | Server name               |
| `TELEMETRYFLOW_MCP_SERVER_TIMEOUT`     | `server.timeout`                          | duration | "30s"                       | Request timeout           |
| `TELEMETRYFLOW_MCP_LOG_LEVEL`          | `logging.level`                           | string   | "info"                      | Log level                 |
| `TELEMETRYFLOW_MCP_LOG_FORMAT`         | `logging.format`                          | string   | "json"                      | Log format                |
| `TELEMETRYFLOW_MCP_TELEMETRY_ENABLED`  | `telemetry.enabled`                       | bool     | false                       | Enable telemetry          |
| `TELEMETRYFLOW_MCP_TELEMETRY_ENDPOINT` | `telemetry.endpoint`                      | string   | "localhost:4317"            | OTLP endpoint             |
| `TELEMETRYFLOW_MCP_RATE_LIMIT_ENABLED` | `security.rate_limit.enabled`             | bool     | true                        | Enable rate limiting      |
| `TELEMETRYFLOW_MCP_RATE_LIMIT_RPM`     | `security.rate_limit.requests_per_minute` | int      | 60                          | Requests per minute       |

### Setting Environment Variables

```bash
# Required - Claude API Key
export TELEMETRYFLOW_MCP_CLAUDE_API_KEY="sk-ant-api03-..."

# Optional - Model selection
export TELEMETRYFLOW_MCP_CLAUDE_MODEL="claude-sonnet-4-20250514"

# Optional - Logging
export TELEMETRYFLOW_MCP_LOG_LEVEL="debug"
export TELEMETRYFLOW_MCP_LOG_FORMAT="text"

# Optional - Telemetry
export TELEMETRYFLOW_MCP_TELEMETRY_ENABLED="true"
export TELEMETRYFLOW_MCP_TELEMETRY_ENDPOINT="otel-collector:4317"

# Optional - Rate limiting
export TELEMETRYFLOW_MCP_RATE_LIMIT_ENABLED="true"
export TELEMETRYFLOW_MCP_RATE_LIMIT_RPM="120"
```

---

## Server Configuration

### Server Settings

```mermaid
flowchart TB
    subgraph ServerConfig["Server Configuration"]
        NAME["name: tfo-mcp"]
        VERSION["version: 1.1.2"]
        DESC["description: TelemetryFlow MCP Server"]
        TIMEOUT["timeout: 30s"]
    end

    subgraph Impact["Configuration Impact"]
        I1["Server Identification"]
        I2["Version Reporting"]
        I3["Client Info"]
        I4["Request Timeouts"]
    end

    NAME --> I1
    VERSION --> I2
    DESC --> I3
    TIMEOUT --> I4

    style ServerConfig fill:#e3f2fd,stroke:#2196f3
```

### Server Configuration Options

| Option        | Type     | Default                    | Description                |
| ------------- | -------- | -------------------------- | -------------------------- |
| `name`        | string   | "tfo-mcp"                  | Server identifier          |
| `version`     | string   | "1.1.2"                    | Server version             |
| `description` | string   | "TelemetryFlow MCP Server" | Human-readable description |
| `timeout`     | duration | "30s"                      | Default request timeout    |

### Server Configuration Example

```yaml
server:
  name: "tfo-mcp"
  version: "1.1.2"
  description: "TelemetryFlow MCP Server - Claude AI Integration"
  timeout: 30s
```

---

## Claude API Configuration

### Claude Configuration Flow

```mermaid
flowchart TB
    subgraph Input["Configuration Input"]
        API["API Key"]
        MODEL["Model Selection"]
        PARAMS["Parameters"]
    end

    subgraph Client["Claude Client"]
        VALIDATE["Validate Config"]
        INIT["Initialize Client"]
        RETRY["Setup Retry Logic"]
    end

    subgraph API_Call["API Call"]
        REQUEST["Build Request"]
        SEND["Send to Claude API"]
        RESPONSE["Process Response"]
    end

    API --> VALIDATE
    MODEL --> VALIDATE
    PARAMS --> VALIDATE
    VALIDATE --> INIT
    INIT --> RETRY
    RETRY --> REQUEST
    REQUEST --> SEND
    SEND --> RESPONSE

    style Input fill:#fff3e0,stroke:#ff9800
    style Client fill:#e3f2fd,stroke:#2196f3
    style API_Call fill:#e8f5e9,stroke:#4caf50
```

### Claude Configuration Options

| Option                | Type     | Default                     | Description                |
| --------------------- | -------- | --------------------------- | -------------------------- |
| `api_key`             | string   | ""                          | Claude API key (required)  |
| `base_url`            | string   | "https://api.anthropic.com" | API base URL               |
| `model`               | string   | "claude-sonnet-4-20250514"  | Default model              |
| `max_tokens`          | int      | 4096                        | Maximum response tokens    |
| `temperature`         | float    | 0.7                         | Response randomness (0-1)  |
| `top_p`               | float    | 0.9                         | Nucleus sampling threshold |
| `top_k`               | int      | 40                          | Top-k sampling             |
| `retry.max_attempts`  | int      | 3                           | Maximum retry attempts     |
| `retry.initial_delay` | duration | "1s"                        | Initial retry delay        |
| `retry.max_delay`     | duration | "30s"                       | Maximum retry delay        |
| `retry.multiplier`    | float    | 2.0                         | Backoff multiplier         |

### Supported Models

```mermaid
flowchart LR
    subgraph Models["Claude Models"]
        direction TB
        OPUS["claude-opus-4-20250514<br/>Most Capable"]
        SONNET["claude-sonnet-4-20250514<br/>Balanced"]
        SONNET35["claude-3-5-sonnet-20241022<br/>Fast & Capable"]
        HAIKU["claude-3-5-haiku-20241022<br/>Fastest"]
    end

    subgraph UseCase["Use Cases"]
        UC1["Complex Tasks"]
        UC2["General Purpose"]
        UC3["Code Generation"]
        UC4["Quick Responses"]
    end

    OPUS --> UC1
    SONNET --> UC2
    SONNET35 --> UC3
    HAIKU --> UC4

    style OPUS fill:#e1bee7,stroke:#9c27b0
    style SONNET fill:#c5cae9,stroke:#3f51b5
    style SONNET35 fill:#b2dfdb,stroke:#009688
    style HAIKU fill:#ffe0b2,stroke:#ff9800
```

### Claude Configuration Example

```yaml
claude:
  api_key: "" # Use TELEMETRYFLOW_MCP_CLAUDE_API_KEY env var
  base_url: "https://api.anthropic.com"
  model: "claude-sonnet-4-20250514"
  max_tokens: 4096
  temperature: 0.7
  top_p: 0.9
  top_k: 40
  retry:
    max_attempts: 3
    initial_delay: 1s
    max_delay: 30s
    multiplier: 2.0
```

---

## MCP Protocol Configuration

### MCP Capabilities

```mermaid
flowchart TB
    subgraph MCPConfig["MCP Configuration"]
        VERSION["protocol_version: 2024-11-05"]
        subgraph Capabilities["capabilities"]
            TOOLS["tools: true"]
            RESOURCES["resources: true"]
            PROMPTS["prompts: true"]
            LOGGING["logging: true"]
        end
        subgraph Transport["transport"]
            TYPE["type: stdio"]
            BUFFER["buffer_size: 65536"]
        end
    end

    style MCPConfig fill:#e8f5e9,stroke:#4caf50
    style Capabilities fill:#fff3e0,stroke:#ff9800
    style Transport fill:#e3f2fd,stroke:#2196f3
```

### MCP Configuration Options

| Option                   | Type   | Default      | Description                 |
| ------------------------ | ------ | ------------ | --------------------------- |
| `protocol_version`       | string | "2024-11-05" | MCP protocol version        |
| `capabilities.tools`     | bool   | true         | Enable tools capability     |
| `capabilities.resources` | bool   | true         | Enable resources capability |
| `capabilities.prompts`   | bool   | true         | Enable prompts capability   |
| `capabilities.logging`   | bool   | true         | Enable logging capability   |
| `transport.type`         | string | "stdio"      | Transport type              |
| `transport.buffer_size`  | int    | 65536        | Buffer size in bytes        |

### MCP Configuration Example

```yaml
mcp:
  protocol_version: "2024-11-05"
  capabilities:
    tools: true
    resources: true
    prompts: true
    logging: true
  transport:
    type: "stdio"
    buffer_size: 65536
```

---

## Logging Configuration

### Logging Levels

```mermaid
flowchart LR
    subgraph Levels["Log Levels"]
        TRACE["trace"]
        DEBUG["debug"]
        INFO["info"]
        WARN["warn"]
        ERROR["error"]
        FATAL["fatal"]
    end

    TRACE -->|includes| DEBUG
    DEBUG -->|includes| INFO
    INFO -->|includes| WARN
    WARN -->|includes| ERROR
    ERROR -->|includes| FATAL

    style TRACE fill:#e3f2fd,stroke:#2196f3
    style DEBUG fill:#e8f5e9,stroke:#4caf50
    style INFO fill:#fff3e0,stroke:#ff9800
    style WARN fill:#fff9c4,stroke:#ffc107
    style ERROR fill:#ffcdd2,stroke:#f44336
    style FATAL fill:#d32f2f,stroke:#b71c1c,color:#fff
```

### Logging Configuration Options

| Option             | Type   | Default  | Description                                   |
| ------------------ | ------ | -------- | --------------------------------------------- |
| `level`            | string | "info"   | Log level (trace/debug/info/warn/error/fatal) |
| `format`           | string | "json"   | Output format (json/text)                     |
| `output`           | string | "stderr" | Output destination (stderr/stdout/file path)  |
| `caller`           | bool   | false    | Include caller information                    |
| `timestamp_format` | string | RFC3339  | Timestamp format                              |

### Log Output Formats

```mermaid
flowchart TB
    subgraph JSON["JSON Format"]
        J1["{"]
        J2["  'level': 'info',"]
        J3["  'time': '2024-01-15T10:30:00Z',"]
        J4["  'message': 'Server started',"]
        J5["  'service': 'tfo-mcp'"]
        J6["}"]
    end

    subgraph Text["Text Format"]
        T1["2024-01-15T10:30:00Z"]
        T2["INF"]
        T3["Server started"]
        T4["service=tfo-mcp"]
    end

    style JSON fill:#e3f2fd,stroke:#2196f3
    style Text fill:#e8f5e9,stroke:#4caf50
```

### Logging Configuration Example

```yaml
logging:
  level: "info"
  format: "json"
  output: "stderr"
  caller: false
  timestamp_format: "2006-01-02T15:04:05.000Z07:00"
```

---

## Telemetry Configuration

### OpenTelemetry Integration

```mermaid
flowchart TB
    subgraph TFO_MCP["TelemetryFlow MCP"]
        TRACES["Traces"]
        METRICS["Metrics"]
        LOGS["Logs"]
    end

    subgraph OTEL["OpenTelemetry SDK"]
        TRACER["Tracer Provider"]
        METER["Meter Provider"]
        LOGGER["Logger Provider"]
        EXPORTER["OTLP Exporter"]
    end

    subgraph Backend["Observability Backend"]
        COLLECTOR["OTEL Collector"]
        JAEGER["Jaeger/Tempo"]
        PROM["Prometheus"]
        LOKI["Loki"]
    end

    TRACES --> TRACER
    METRICS --> METER
    LOGS --> LOGGER
    TRACER --> EXPORTER
    METER --> EXPORTER
    LOGGER --> EXPORTER
    EXPORTER --> COLLECTOR
    COLLECTOR --> JAEGER
    COLLECTOR --> PROM
    COLLECTOR --> LOKI

    style TFO_MCP fill:#e3f2fd,stroke:#2196f3
    style OTEL fill:#fff3e0,stroke:#ff9800
    style Backend fill:#e8f5e9,stroke:#4caf50
```

### Telemetry Configuration Options

| Option            | Type     | Default          | Description               |
| ----------------- | -------- | ---------------- | ------------------------- |
| `enabled`         | bool     | false            | Enable telemetry          |
| `service_name`    | string   | "tfo-mcp"        | Service name for traces   |
| `service_version` | string   | "1.1.2"          | Service version           |
| `environment`     | string   | "development"    | Deployment environment    |
| `endpoint`        | string   | "localhost:4317" | OTLP endpoint             |
| `sample_rate`     | float    | 1.0              | Trace sampling rate (0-1) |
| `export_timeout`  | duration | "30s"            | Export timeout            |

### Telemetry Configuration Example

```yaml
telemetry:
  enabled: true
  service_name: "tfo-mcp"
  service_version: "1.1.2"
  environment: "production"
  endpoint: "otel-collector:4317"
  sample_rate: 0.1 # 10% sampling
  export_timeout: 30s
```

---

## Security Configuration

### Security Architecture

```mermaid
flowchart TB
    subgraph Request["Incoming Request"]
        REQ["MCP Request"]
    end

    subgraph Security["Security Layer"]
        subgraph RateLimit["Rate Limiting"]
            RL1["Token Bucket"]
            RL2["Per-Minute Limit"]
            RL3["Burst Allowance"]
        end

        subgraph Validation["Validation"]
            V1["API Key Check"]
            V2["Input Validation"]
            V3["Schema Validation"]
        end

        subgraph CORS["CORS"]
            C1["Origin Check"]
            C2["Headers Validation"]
        end
    end

    subgraph Handler["Request Handler"]
        PROCESS["Process Request"]
    end

    REQ --> RateLimit
    RateLimit -->|Pass| Validation
    RateLimit -->|Reject| REJECT1["429 Too Many Requests"]
    Validation -->|Pass| CORS
    Validation -->|Reject| REJECT2["401 Unauthorized"]
    CORS -->|Pass| PROCESS
    CORS -->|Reject| REJECT3["403 Forbidden"]

    style Security fill:#ffcdd2,stroke:#f44336
    style RateLimit fill:#fff3e0,stroke:#ff9800
    style Validation fill:#e3f2fd,stroke:#2196f3
```

### Security Configuration Options

| Option                           | Type     | Default | Description             |
| -------------------------------- | -------- | ------- | ----------------------- |
| `rate_limit.enabled`             | bool     | true    | Enable rate limiting    |
| `rate_limit.requests_per_minute` | int      | 60      | Max requests per minute |
| `rate_limit.burst_size`          | int      | 10      | Burst allowance         |
| `cors.enabled`                   | bool     | false   | Enable CORS             |
| `cors.allowed_origins`           | []string | ["*"]   | Allowed origins         |
| `api_key_validation`             | bool     | true    | Validate API keys       |

### Security Configuration Example

```yaml
security:
  rate_limit:
    enabled: true
    requests_per_minute: 60
    burst_size: 10
  cors:
    enabled: false
    allowed_origins:
      - "https://example.com"
      - "https://app.example.com"
  api_key_validation: true
```

---

## Configuration Validation

### Validation Process

```mermaid
flowchart TB
    subgraph Input["Configuration Input"]
        CONFIG["Config Object"]
    end

    subgraph Validation["Validation Steps"]
        V1["Required Fields Check"]
        V2["Type Validation"]
        V3["Range Validation"]
        V4["Dependency Check"]
        V5["Security Check"]
    end

    subgraph Result["Validation Result"]
        SUCCESS["Valid Configuration"]
        FAILURE["Validation Errors"]
    end

    CONFIG --> V1
    V1 -->|Pass| V2
    V1 -->|Fail| FAILURE
    V2 -->|Pass| V3
    V2 -->|Fail| FAILURE
    V3 -->|Pass| V4
    V3 -->|Fail| FAILURE
    V4 -->|Pass| V5
    V4 -->|Fail| FAILURE
    V5 -->|Pass| SUCCESS
    V5 -->|Fail| FAILURE

    style Validation fill:#fff3e0,stroke:#ff9800
    style SUCCESS fill:#e8f5e9,stroke:#4caf50
    style FAILURE fill:#ffcdd2,stroke:#f44336
```

### Validation Rules

| Field                   | Rule        | Error Message                         |
| ----------------------- | ----------- | ------------------------------------- |
| `claude.api_key`        | Required    | "Claude API key is required"          |
| `claude.max_tokens`     | > 0         | "max_tokens must be positive"         |
| `claude.temperature`    | 0-1         | "temperature must be between 0 and 1" |
| `logging.level`         | Valid level | "invalid log level"                   |
| `telemetry.sample_rate` | 0-1         | "sample_rate must be between 0 and 1" |

### Validating Configuration

```bash
# Validate configuration file
tfo-mcp validate

# Validate with verbose output
tfo-mcp validate --verbose

# Validate specific config file
tfo-mcp validate --config /path/to/config.yaml
```

---

## Configuration Examples

### Development Configuration

```yaml
# Development configuration
server:
  name: "tfo-mcp-dev"
  version: "1.1.2"
  timeout: 60s

claude:
  model: "claude-3-5-haiku-20241022" # Fast model for development
  max_tokens: 2048
  temperature: 0.9

logging:
  level: "debug"
  format: "text" # Human-readable format
  caller: true

telemetry:
  enabled: false

security:
  rate_limit:
    enabled: false # Disable for development
```

### Production Configuration

```yaml
# Production configuration
server:
  name: "tfo-mcp"
  version: "1.1.2"
  timeout: 30s

claude:
  model: "claude-sonnet-4-20250514"
  max_tokens: 4096
  temperature: 0.7
  retry:
    max_attempts: 5
    initial_delay: 2s
    max_delay: 60s

logging:
  level: "info"
  format: "json"
  caller: false

telemetry:
  enabled: true
  service_name: "tfo-mcp"
  environment: "production"
  endpoint: "otel-collector.monitoring:4317"
  sample_rate: 0.1

security:
  rate_limit:
    enabled: true
    requests_per_minute: 120
    burst_size: 20
  api_key_validation: true
```

### Minimal Configuration

```yaml
# Minimal configuration - only required settings
claude:
  api_key: "" # Set via TELEMETRYFLOW_MCP_CLAUDE_API_KEY
```

### Docker Configuration

```yaml
# Docker-optimized configuration
server:
  name: "tfo-mcp"
  timeout: 30s

claude:
  api_key: "" # Injected via environment

logging:
  level: "info"
  format: "json"
  output: "stdout" # Docker-friendly

telemetry:
  enabled: true
  endpoint: "otel-collector:4317"
```

---

## Best Practices

### Configuration Best Practices

```mermaid
flowchart TB
    subgraph Secrets["Secret Management"]
        S1["Never commit API keys"]
        S2["Use environment variables"]
        S3["Use secret managers"]
    end

    subgraph Environment["Environment-Specific"]
        E1["Separate dev/prod configs"]
        E2["Use config profiles"]
        E3["Override via env vars"]
    end

    subgraph Validation["Configuration Validation"]
        V1["Validate on startup"]
        V2["Fail fast on errors"]
        V3["Log configuration state"]
    end

    subgraph Monitoring["Monitoring"]
        M1["Enable telemetry in prod"]
        M2["Set appropriate log levels"]
        M3["Configure rate limits"]
    end

    style Secrets fill:#ffcdd2,stroke:#f44336
    style Environment fill:#fff3e0,stroke:#ff9800
    style Validation fill:#e3f2fd,stroke:#2196f3
    style Monitoring fill:#e8f5e9,stroke:#4caf50
```

### Security Best Practices

1. **Never commit secrets**

   ```bash
   # Use .gitignore
   echo "config.local.yaml" >> .gitignore
   echo ".env" >> .gitignore
   ```

2. **Use environment variables for secrets**

   ```bash
   export TELEMETRYFLOW_MCP_CLAUDE_API_KEY="sk-ant-api03-..."
   ```

3. **Use secret managers in production**

   ```bash
   # AWS Secrets Manager
   TELEMETRYFLOW_MCP_CLAUDE_API_KEY=$(aws secretsmanager get-secret-value --secret-id tfo-mcp/api-key --query SecretString --output text)
   ```

4. **Enable rate limiting in production**
   ```yaml
   security:
     rate_limit:
       enabled: true
       requests_per_minute: 60
   ```

### Performance Best Practices

1. **Choose appropriate models**

   - Use Haiku for quick responses
   - Use Sonnet for balanced performance
   - Use Opus for complex tasks

2. **Configure appropriate timeouts**

   ```yaml
   server:
     timeout: 30s # Adjust based on workload
   ```

3. **Enable telemetry for monitoring**
   ```yaml
   telemetry:
     enabled: true
     sample_rate: 0.1 # 10% sampling to reduce overhead
   ```

---

## Related Documentation

- [Architecture Guide](ARCHITECTURE.md)
- [Commands Reference](COMMANDS.md)
- [Development Guide](DEVELOPMENT.md)
- [Installation Guide](INSTALLATION.md)
- [Troubleshooting Guide](TROUBLESHOOTING.md)

---

<div align="center">

**[Back to Documentation Index](README.md)**

</div>
