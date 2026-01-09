# TFO-MCP Entity Relationship Diagrams

> Entity Relationship Diagrams for TelemetryFlow MCP Server

---

## Table of Contents

- [Overview](#overview)
- [Complete Domain ERD](#complete-domain-erd)
- [Session Aggregate ERD](#session-aggregate-erd)
- [Conversation Aggregate ERD](#conversation-aggregate-erd)
- [Value Objects ERD](#value-objects-erd)
- [Database Schema](#database-schema)

---

## Overview

This document provides Entity Relationship Diagrams that describe the data structures and relationships within the TFO-MCP Server.

```mermaid
flowchart LR
    subgraph ERD["Entity Relationship Diagrams"]
        DOMAIN["Domain ERD"]
        SESSION["Session ERD"]
        CONV["Conversation ERD"]
        VO["Value Objects"]
        DB["Database Schema"]
    end

    DOMAIN --> |"Describes"| STRUCT["Domain Structure"]
    SESSION --> |"Details"| SAGG["Session Aggregate"]
    CONV --> |"Details"| CAGG["Conversation Aggregate"]
    VO --> |"Defines"| TYPES["Value Types"]
    DB --> |"Maps to"| TABLES["PostgreSQL Tables"]

    style DOMAIN fill:#e3f2fd,stroke:#2196f3
    style SESSION fill:#e8f5e9,stroke:#4caf50
    style CONV fill:#fff3e0,stroke:#ff9800
    style VO fill:#fce4ec,stroke:#e91e63
    style DB fill:#f3e5f5,stroke:#9c27b0
```

---

## Complete Domain ERD

```mermaid
erDiagram
    SESSION ||--o{ CONVERSATION : contains
    SESSION ||--o{ TOOL : registers
    SESSION ||--o{ RESOURCE : registers
    SESSION ||--o{ PROMPT : registers
    CONVERSATION ||--o{ MESSAGE : contains
    MESSAGE ||--o{ CONTENT_BLOCK : contains
    TOOL ||--o{ TOOL_RESULT : produces
    RESOURCE ||--o{ RESOURCE_CONTENT : provides

    SESSION {
        uuid id PK
        string client_name
        string client_version
        string protocol_version
        enum state
        datetime created_at
        datetime updated_at
    }

    CONVERSATION {
        uuid id PK
        uuid session_id FK
        string model
        string system_prompt
        enum status
        int max_tokens
        float temperature
        datetime created_at
    }

    MESSAGE {
        uuid id PK
        uuid conversation_id FK
        enum role
        datetime created_at
    }

    CONTENT_BLOCK {
        uuid id PK
        uuid message_id FK
        enum type
        text content
        string mime_type
    }

    TOOL {
        string name PK
        string description
        json input_schema
        boolean enabled
    }

    TOOL_RESULT {
        uuid id PK
        string tool_name FK
        json content
        boolean is_error
        datetime executed_at
    }

    RESOURCE {
        string uri PK
        string name
        string description
        string mime_type
        bigint size
    }

    RESOURCE_CONTENT {
        uuid id PK
        string uri FK
        text content
        string blob
    }

    PROMPT {
        string name PK
        string description
        json arguments
    }
```

---

## Session Aggregate ERD

```mermaid
erDiagram
    SESSION_AGGREGATE ||--|| SESSION_ID : has
    SESSION_AGGREGATE ||--|| CLIENT_INFO : has
    SESSION_AGGREGATE ||--|| CAPABILITIES : has
    SESSION_AGGREGATE ||--o{ TOOL_ENTITY : contains
    SESSION_AGGREGATE ||--o{ RESOURCE_ENTITY : contains
    SESSION_AGGREGATE ||--o{ PROMPT_ENTITY : contains
    SESSION_AGGREGATE ||--o{ CONVERSATION_AGGREGATE : manages

    SESSION_AGGREGATE {
        SessionID id
        SessionState state
        ClientInfo client
        Capabilities capabilities
        datetime created_at
    }

    SESSION_ID {
        uuid value
    }

    CLIENT_INFO {
        string name
        string version
    }

    CAPABILITIES {
        boolean tools
        boolean resources
        boolean prompts
        boolean logging
    }

    TOOL_ENTITY {
        string name
        string description
        JSONSchema input_schema
        ToolHandler handler
        boolean enabled
    }

    RESOURCE_ENTITY {
        ResourceURI uri
        string name
        MimeType mime_type
        ResourceReader reader
    }

    PROMPT_ENTITY {
        string name
        string description
        PromptArgument[] arguments
        PromptGenerator generator
    }

    CONVERSATION_AGGREGATE {
        ConversationID id
        Model model
        Message[] messages
        ConversationStatus status
    }
```

---

## Conversation Aggregate ERD

```mermaid
erDiagram
    CONVERSATION_AGGREGATE ||--|| CONVERSATION_ID : has
    CONVERSATION_AGGREGATE ||--|| MODEL : uses
    CONVERSATION_AGGREGATE ||--o{ MESSAGE_ENTITY : contains
    CONVERSATION_AGGREGATE ||--o{ TOOL_BINDING : uses
    MESSAGE_ENTITY ||--|| MESSAGE_ID : has
    MESSAGE_ENTITY ||--|| ROLE : has
    MESSAGE_ENTITY ||--o{ CONTENT_BLOCK : contains

    CONVERSATION_AGGREGATE {
        ConversationID id
        SessionID session_id
        Model model
        SystemPrompt system_prompt
        ConversationStatus status
        int max_tokens
        float temperature
    }

    CONVERSATION_ID {
        uuid value
    }

    MODEL {
        string value
    }

    MESSAGE_ENTITY {
        MessageID id
        Role role
        ContentBlock[] content
        datetime created_at
    }

    MESSAGE_ID {
        uuid value
    }

    ROLE {
        enum value "user|assistant"
    }

    CONTENT_BLOCK {
        ContentType type
        string text
        string data
        string mime_type
    }

    TOOL_BINDING {
        string tool_name
        boolean enabled
    }
```

---

## Value Objects ERD

```mermaid
erDiagram
    VALUE_OBJECTS ||--o{ IDENTIFIERS : contains
    VALUE_OBJECTS ||--o{ CONTENT_TYPES : contains
    VALUE_OBJECTS ||--o{ MCP_TYPES : contains

    IDENTIFIERS {
        SessionID session_id "uuid"
        ConversationID conversation_id "uuid"
        MessageID message_id "uuid"
        ToolID tool_id "uuid"
        ResourceID resource_id "uuid"
        PromptID prompt_id "uuid"
        RequestID request_id "uuid"
    }

    CONTENT_TYPES {
        ContentType type "text|image|resource"
        Role role "user|assistant"
        Model model "claude-*"
        MimeType mime_type "text/*"
        TextContent text "string"
        SystemPrompt system "string"
    }

    MCP_TYPES {
        JSONRPCVersion version "2.0"
        MCPMethod method "string"
        MCPCapability capability "string"
        MCPLogLevel log_level "enum"
        MCPProtocolVersion protocol "2024-11-05"
        MCPErrorCode error_code "int"
    }
```

---

## Database Schema

### PostgreSQL Schema (GORM Models)

The PostgreSQL database stores transactional data for MCP sessions, conversations, and registered components.

```mermaid
erDiagram
    sessions ||--o{ conversations : has
    sessions ||--o{ resource_subscriptions : has
    sessions ||--o{ tool_executions : logs
    conversations ||--o{ messages : contains
    conversations ||--o{ tool_executions : logs

    sessions {
        uuid id PK
        varchar(20) state
        varchar(255) client_name
        varchar(50) client_version
        varchar(20) protocol_version
        jsonb capabilities
        jsonb server_info
        varchar(20) log_level
        timestamptz created_at
        timestamptz updated_at
        timestamptz closed_at
    }

    conversations {
        uuid id PK
        uuid session_id FK
        varchar(50) model
        text system_prompt
        varchar(20) status
        int max_tokens
        decimal temperature
        int total_input_tokens
        int total_output_tokens
        jsonb metadata
        timestamptz created_at
        timestamptz updated_at
        timestamptz closed_at
    }

    messages {
        uuid id PK
        uuid conversation_id FK
        varchar(20) role
        jsonb content
        int input_tokens
        int output_tokens
        varchar(50) stop_reason
        jsonb metadata
        timestamptz created_at
    }

    tools {
        uuid id PK
        varchar(255) name UK
        text description
        jsonb input_schema
        varchar(50) category
        text[] tags
        boolean is_enabled
        int timeout_seconds
        jsonb metadata
        timestamptz created_at
        timestamptz updated_at
    }

    resources {
        uuid id PK
        varchar(2048) uri UK
        varchar(2048) uri_template
        varchar(255) name
        text description
        varchar(255) mime_type
        boolean is_template
        jsonb metadata
        timestamptz created_at
        timestamptz updated_at
    }

    prompts {
        uuid id PK
        varchar(255) name UK
        text description
        jsonb arguments
        text template
        jsonb metadata
        timestamptz created_at
        timestamptz updated_at
    }

    resource_subscriptions {
        uuid id PK
        uuid session_id FK
        varchar(2048) resource_uri
        timestamptz created_at
    }

    tool_executions {
        uuid id PK
        uuid session_id FK
        uuid conversation_id FK
        varchar(255) tool_name
        jsonb input_params
        jsonb result
        boolean is_error
        varchar(255) error_message
        int duration_ms
        timestamptz executed_at
    }

    api_keys {
        uuid id PK
        varchar(255) key_hash UK
        varchar(255) name
        text description
        text[] scopes
        boolean is_active
        int rate_limit_per_minute
        int rate_limit_per_hour
        timestamptz last_used_at
        timestamptz expires_at
        timestamptz created_at
        timestamptz updated_at
    }

    schema_migrations {
        varchar(255) version PK
        timestamptz applied_at
    }
```

### ClickHouse Analytics Schema

The ClickHouse database stores high-volume analytics data with time-series optimizations.

```mermaid
erDiagram
    tool_call_analytics {
        datetime64 timestamp
        uuid session_id
        uuid conversation_id
        string tool_name
        uint64 duration_ms
        uint8 is_error
        string error_message
        uint32 input_size
        uint32 output_size
        string metadata
    }

    api_request_analytics {
        datetime64 timestamp
        uuid request_id
        uuid session_id
        uuid conversation_id
        string model
        uint32 input_tokens
        uint32 output_tokens
        uint32 total_tokens
        uint64 duration_ms
        uint16 status_code
        uint8 is_error
        uint8 is_streaming
        string stop_reason
        string metadata
    }

    session_analytics {
        datetime64 timestamp
        uuid session_id
        string event_type
        string client_name
        string client_version
        string protocol_version
        uint64 duration_ms
        uint32 message_count
        uint32 tool_call_count
        uint64 total_tokens
        uint32 error_count
        string metadata
    }

    error_analytics {
        datetime64 timestamp
        uuid session_id
        uuid conversation_id
        string error_type
        string error_code
        string error_message
        string stack_trace
        string context
        string metadata
    }

    token_usage_hourly {
        datetime hour
        string model
        uint64 input_tokens
        uint64 output_tokens
        uint64 total_tokens
        uint64 request_count
    }

    tool_usage_hourly {
        datetime hour
        string tool_name
        uint64 call_count
        uint64 error_count
        uint64 total_duration_ms
        uint64 min_duration_ms
        uint64 max_duration_ms
    }

    latency_percentiles_hourly {
        datetime hour
        string model
        float64 p50_ms
        float64 p90_ms
        float64 p95_ms
        float64 p99_ms
        float64 avg_ms
        uint64 request_count
    }
```

### ClickHouse Table Engines

| Table                        | Engine             | Partitioning | TTL      |
| ---------------------------- | ------------------ | ------------ | -------- |
| `tool_call_analytics`        | MergeTree          | Monthly      | 90 days  |
| `api_request_analytics`      | MergeTree          | Monthly      | 90 days  |
| `session_analytics`          | MergeTree          | Monthly      | 180 days |
| `error_analytics`            | MergeTree          | Monthly      | 30 days  |
| `token_usage_hourly`         | SummingMergeTree   | Monthly      | -        |
| `tool_usage_hourly`          | SummingMergeTree   | Monthly      | -        |
| `latency_percentiles_hourly` | ReplacingMergeTree | Monthly      | -        |

### Materialized Views

```mermaid
flowchart LR
    subgraph "Source Tables"
        API[api_request_analytics]
        TOOL[tool_call_analytics]
    end

    subgraph "Materialized Views"
        MV_TOKEN[mv_token_usage_hourly]
        MV_TOOL[mv_tool_usage_hourly]
    end

    subgraph "Aggregation Tables"
        TOKEN[token_usage_hourly]
        TOOL_AGG[tool_usage_hourly]
    end

    API --> MV_TOKEN
    MV_TOKEN --> TOKEN

    TOOL --> MV_TOOL
    MV_TOOL --> TOOL_AGG

    style MV_TOKEN fill:#E1BEE7,stroke:#7B1FA2
    style MV_TOOL fill:#E1BEE7,stroke:#7B1FA2
```

---

## Related Documentation

- [Data Flow Diagrams](DFD.md)
- [Architecture Guide](ARCHITECTURE.md)
- [Development Guide](DEVELOPMENT.md)

---

<div align="center">

**[Back to Documentation Index](README.md)**

</div>
