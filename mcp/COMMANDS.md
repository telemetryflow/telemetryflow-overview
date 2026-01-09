# TFO-MCP Commands Reference

> Complete CLI and MCP protocol commands reference for TelemetryFlow MCP Server

---

## Table of Contents

- [Overview](#overview)
- [CLI Commands](#cli-commands)
- [MCP Protocol Methods](#mcp-protocol-methods)
- [Built-in Tools](#built-in-tools)
- [Resource Operations](#resource-operations)
- [Prompt Operations](#prompt-operations)
- [Session Management](#session-management)
- [Examples](#examples)

---

## Overview

TFO-MCP provides two interfaces for interaction:

1. **CLI Commands** - Command-line interface for server management
2. **MCP Protocol Methods** - JSON-RPC methods for client communication

### Command Architecture

```mermaid
flowchart TB
    subgraph CLI["CLI Interface"]
        RUN["tfo-mcp run"]
        VERSION["tfo-mcp version"]
        VALIDATE["tfo-mcp validate"]
        HELP["tfo-mcp help"]
    end

    subgraph MCP["MCP Protocol"]
        INIT["initialize"]
        TOOLS["tools/*"]
        RESOURCES["resources/*"]
        PROMPTS["prompts/*"]
        LOGGING["logging/*"]
    end

    subgraph Server["TFO-MCP Server"]
        HANDLER["Command Handler"]
    end

    CLI --> HANDLER
    MCP --> HANDLER

    style CLI fill:#e3f2fd,stroke:#2196f3
    style MCP fill:#e8f5e9,stroke:#4caf50
    style Server fill:#fff3e0,stroke:#ff9800
```

---

## CLI Commands

### Command Structure

```mermaid
flowchart LR
    subgraph Structure["Command Structure"]
        BIN["tfo-mcp"]
        CMD["command"]
        FLAGS["--flags"]
        ARGS["arguments"]
    end

    BIN --> CMD --> FLAGS --> ARGS

    style Structure fill:#e3f2fd,stroke:#2196f3
```

### Available Commands

| Command    | Description              | Usage                      |
| ---------- | ------------------------ | -------------------------- |
| `run`      | Start the MCP server     | `tfo-mcp run [flags]`      |
| `version`  | Show version information | `tfo-mcp version`          |
| `validate` | Validate configuration   | `tfo-mcp validate [flags]` |
| `help`     | Show help information    | `tfo-mcp help [command]`   |

### run Command

Start the MCP server.

```mermaid
flowchart TB
    subgraph Run["tfo-mcp run"]
        PARSE["Parse Flags"]
        LOAD["Load Config"]
        INIT["Initialize Server"]
        START["Start Listening"]
        SERVE["Serve Requests"]
    end

    PARSE --> LOAD --> INIT --> START --> SERVE

    style Run fill:#e8f5e9,stroke:#4caf50
```

**Usage:**

```bash
# Basic usage
tfo-mcp run

# With custom config
tfo-mcp run --config /path/to/config.yaml

# With debug logging
tfo-mcp run --log-level debug

# With specific transport
tfo-mcp run --transport stdio
```

**Flags:**

| Flag          | Short | Type     | Default       | Description                             |
| ------------- | ----- | -------- | ------------- | --------------------------------------- |
| `--config`    | `-c`  | string   | "config.yaml" | Configuration file path                 |
| `--log-level` | `-l`  | string   | "info"        | Log level (trace/debug/info/warn/error) |
| `--transport` | `-t`  | string   | "stdio"       | Transport type                          |
| `--timeout`   |       | duration | "30s"         | Request timeout                         |

### version Command

Display version and build information.

```bash
tfo-mcp version
```

**Output:**

```
TFO-MCP - TelemetryFlow MCP Server
Version:    1.1.2
Go Version: go1.24
Build Date: 2024-01-15
Git Commit: abc1234
Platform:   darwin/arm64
```

### validate Command

Validate the configuration file.

```mermaid
flowchart TB
    subgraph Validate["tfo-mcp validate"]
        READ["Read Config"]
        PARSE["Parse YAML"]
        CHECK["Validate Fields"]
        REPORT["Report Results"]
    end

    READ --> PARSE --> CHECK --> REPORT

    style Validate fill:#fff3e0,stroke:#ff9800
```

**Usage:**

```bash
# Validate default config
tfo-mcp validate

# Validate specific file
tfo-mcp validate --config /path/to/config.yaml

# Verbose output
tfo-mcp validate --verbose
```

**Flags:**

| Flag        | Short | Type   | Default       | Description             |
| ----------- | ----- | ------ | ------------- | ----------------------- |
| `--config`  | `-c`  | string | "config.yaml" | Configuration file path |
| `--verbose` | `-v`  | bool   | false         | Verbose output          |

---

## MCP Protocol Methods

### Protocol Overview

```mermaid
flowchart TB
    subgraph Client["MCP Client"]
        REQ["JSON-RPC Request"]
    end

    subgraph Server["TFO-MCP Server"]
        ROUTER["Method Router"]

        subgraph Handlers["Method Handlers"]
            H1["initialize"]
            H2["tools/list"]
            H3["tools/call"]
            H4["resources/list"]
            H5["resources/read"]
            H6["prompts/list"]
            H7["prompts/get"]
            H8["logging/setLevel"]
        end
    end

    subgraph Response["Response"]
        RES["JSON-RPC Response"]
    end

    REQ --> ROUTER
    ROUTER --> H1 & H2 & H3 & H4 & H5 & H6 & H7 & H8
    Handlers --> RES

    style Client fill:#e3f2fd,stroke:#2196f3
    style Server fill:#fff3e0,stroke:#ff9800
    style Response fill:#e8f5e9,stroke:#4caf50
```

### Method Categories

```mermaid
flowchart LR
    subgraph Methods["MCP Methods"]
        direction TB
        LIFECYCLE["Lifecycle<br/>initialize, shutdown"]
        TOOLS["Tools<br/>tools/list, tools/call"]
        RESOURCES["Resources<br/>resources/list, resources/read"]
        PROMPTS["Prompts<br/>prompts/list, prompts/get"]
        LOGGING["Logging<br/>logging/setLevel"]
    end

    style LIFECYCLE fill:#e1bee7,stroke:#9c27b0
    style TOOLS fill:#e3f2fd,stroke:#2196f3
    style RESOURCES fill:#e8f5e9,stroke:#4caf50
    style PROMPTS fill:#fff3e0,stroke:#ff9800
    style LOGGING fill:#f5f5f5,stroke:#9e9e9e
```

### initialize

Initialize the MCP session.

```mermaid
sequenceDiagram
    participant Client
    participant Server

    Client->>Server: initialize
    Note right of Server: Create session<br/>Register capabilities
    Server-->>Client: InitializeResult
    Client->>Server: notifications/initialized
    Server-->>Client: (acknowledged)
```

**Request:**

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "initialize",
  "params": {
    "protocolVersion": "2024-11-05",
    "capabilities": {},
    "clientInfo": {
      "name": "my-client",
      "version": "1.0.0"
    }
  }
}
```

**Response:**

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "protocolVersion": "2024-11-05",
    "capabilities": {
      "tools": {},
      "resources": {},
      "prompts": {},
      "logging": {}
    },
    "serverInfo": {
      "name": "tfo-mcp",
      "version": "1.1.2"
    }
  }
}
```

### tools/list

List available tools.

**Request:**

```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "method": "tools/list",
  "params": {}
}
```

**Response:**

```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "result": {
    "tools": [
      {
        "name": "claude_conversation",
        "description": "Have a conversation with Claude AI",
        "inputSchema": {
          "type": "object",
          "properties": {
            "message": {
              "type": "string",
              "description": "The message to send to Claude"
            },
            "model": {
              "type": "string",
              "description": "Claude model to use"
            }
          },
          "required": ["message"]
        }
      }
    ]
  }
}
```

### tools/call

Execute a tool.

```mermaid
sequenceDiagram
    participant Client
    participant Server
    participant Tool
    participant Claude as Claude API

    Client->>Server: tools/call
    Server->>Tool: Execute
    alt Tool is claude_conversation
        Tool->>Claude: API Request
        Claude-->>Tool: API Response
    end
    Tool-->>Server: Result
    Server-->>Client: CallToolResult
```

**Request:**

```json
{
  "jsonrpc": "2.0",
  "id": 3,
  "method": "tools/call",
  "params": {
    "name": "claude_conversation",
    "arguments": {
      "message": "What is the capital of France?",
      "model": "claude-sonnet-4-20250514"
    }
  }
}
```

**Response:**

```json
{
  "jsonrpc": "2.0",
  "id": 3,
  "result": {
    "content": [
      {
        "type": "text",
        "text": "The capital of France is Paris."
      }
    ],
    "isError": false
  }
}
```

### resources/list

List available resources.

**Request:**

```json
{
  "jsonrpc": "2.0",
  "id": 4,
  "method": "resources/list",
  "params": {}
}
```

**Response:**

```json
{
  "jsonrpc": "2.0",
  "id": 4,
  "result": {
    "resources": [
      {
        "uri": "file:///project/README.md",
        "name": "README.md",
        "description": "Project readme file",
        "mimeType": "text/markdown"
      }
    ]
  }
}
```

### resources/read

Read a resource.

**Request:**

```json
{
  "jsonrpc": "2.0",
  "id": 5,
  "method": "resources/read",
  "params": {
    "uri": "file:///project/README.md"
  }
}
```

**Response:**

```json
{
  "jsonrpc": "2.0",
  "id": 5,
  "result": {
    "contents": [
      {
        "uri": "file:///project/README.md",
        "mimeType": "text/markdown",
        "text": "# Project README\n..."
      }
    ]
  }
}
```

### prompts/list

List available prompts.

**Request:**

```json
{
  "jsonrpc": "2.0",
  "id": 6,
  "method": "prompts/list",
  "params": {}
}
```

**Response:**

```json
{
  "jsonrpc": "2.0",
  "id": 6,
  "result": {
    "prompts": [
      {
        "name": "code_review",
        "description": "Review code for best practices",
        "arguments": [
          {
            "name": "code",
            "description": "Code to review",
            "required": true
          },
          {
            "name": "language",
            "description": "Programming language",
            "required": false
          }
        ]
      }
    ]
  }
}
```

### prompts/get

Get a specific prompt.

**Request:**

```json
{
  "jsonrpc": "2.0",
  "id": 7,
  "method": "prompts/get",
  "params": {
    "name": "code_review",
    "arguments": {
      "code": "func main() { fmt.Println(\"Hello\") }",
      "language": "go"
    }
  }
}
```

**Response:**

```json
{
  "jsonrpc": "2.0",
  "id": 7,
  "result": {
    "description": "Review code for best practices",
    "messages": [
      {
        "role": "user",
        "content": {
          "type": "text",
          "text": "Please review this go code for best practices:\n\nfunc main() { fmt.Println(\"Hello\") }"
        }
      }
    ]
  }
}
```

### logging/setLevel

Set the logging level.

**Request:**

```json
{
  "jsonrpc": "2.0",
  "id": 8,
  "method": "logging/setLevel",
  "params": {
    "level": "debug"
  }
}
```

**Response:**

```json
{
  "jsonrpc": "2.0",
  "id": 8,
  "result": {}
}
```

---

## Built-in Tools

### Tool Overview

```mermaid
flowchart TB
    subgraph Tools["Built-in Tools"]
        direction TB
        AI["AI Tools"]
        FILE["File Tools"]
        SYSTEM["System Tools"]
    end

    subgraph AITools["AI Tools"]
        CLAUDE["claude_conversation"]
    end

    subgraph FileTools["File Tools"]
        READ["read_file"]
        WRITE["write_file"]
        LIST["list_directory"]
        SEARCH["search_files"]
    end

    subgraph SystemTools["System Tools"]
        EXEC["execute_command"]
        INFO["system_info"]
        ECHO["echo"]
    end

    AI --> AITools
    FILE --> FileTools
    SYSTEM --> SystemTools

    style AI fill:#e1bee7,stroke:#9c27b0
    style FILE fill:#e3f2fd,stroke:#2196f3
    style SYSTEM fill:#e8f5e9,stroke:#4caf50
```

### claude_conversation

Interact with Claude AI.

```mermaid
sequenceDiagram
    participant Client
    participant Tool as claude_conversation
    participant Claude as Claude API

    Client->>Tool: message, model, system_prompt
    Tool->>Tool: Build request
    Tool->>Claude: CreateMessage
    Claude-->>Tool: Response
    Tool->>Tool: Extract text
    Tool-->>Client: Text result
```

**Parameters:**

| Name            | Type   | Required | Description                          |
| --------------- | ------ | -------- | ------------------------------------ |
| `message`       | string | Yes      | Message to send to Claude            |
| `model`         | string | No       | Claude model (default: config value) |
| `system_prompt` | string | No       | System prompt for context            |
| `max_tokens`    | int    | No       | Maximum response tokens              |
| `temperature`   | float  | No       | Response temperature (0-1)           |

**Example:**

```json
{
  "name": "claude_conversation",
  "arguments": {
    "message": "Explain recursion in programming",
    "model": "claude-sonnet-4-20250514",
    "system_prompt": "You are a helpful programming tutor",
    "max_tokens": 1000
  }
}
```

### read_file

Read file contents.

**Parameters:**

| Name       | Type   | Required | Description                    |
| ---------- | ------ | -------- | ------------------------------ |
| `path`     | string | Yes      | File path to read              |
| `encoding` | string | No       | File encoding (default: utf-8) |

**Example:**

```json
{
  "name": "read_file",
  "arguments": {
    "path": "/project/main.go"
  }
}
```

### write_file

Write content to a file.

**Parameters:**

| Name          | Type   | Required | Description               |
| ------------- | ------ | -------- | ------------------------- |
| `path`        | string | Yes      | File path to write        |
| `content`     | string | Yes      | Content to write          |
| `create_dirs` | bool   | No       | Create parent directories |

**Example:**

```json
{
  "name": "write_file",
  "arguments": {
    "path": "/project/output.txt",
    "content": "Hello, World!",
    "create_dirs": true
  }
}
```

### list_directory

List directory contents.

**Parameters:**

| Name             | Type   | Required | Description          |
| ---------------- | ------ | -------- | -------------------- |
| `path`           | string | Yes      | Directory path       |
| `recursive`      | bool   | No       | List recursively     |
| `include_hidden` | bool   | No       | Include hidden files |

**Example:**

```json
{
  "name": "list_directory",
  "arguments": {
    "path": "/project",
    "recursive": true
  }
}
```

### search_files

Search for files matching a pattern.

**Parameters:**

| Name      | Type   | Required | Description                      |
| --------- | ------ | -------- | -------------------------------- |
| `pattern` | string | Yes      | Search pattern (glob or regex)   |
| `path`    | string | No       | Base path (default: current dir) |
| `type`    | string | No       | Pattern type: "glob" or "regex"  |

**Example:**

```json
{
  "name": "search_files",
  "arguments": {
    "pattern": "*.go",
    "path": "/project",
    "type": "glob"
  }
}
```

### execute_command

Execute a shell command.

```mermaid
flowchart TB
    subgraph Security["Security Checks"]
        VALIDATE["Validate Command"]
        SANITIZE["Sanitize Input"]
        TIMEOUT["Apply Timeout"]
    end

    subgraph Execution["Command Execution"]
        EXEC["Execute"]
        CAPTURE["Capture Output"]
    end

    VALIDATE --> SANITIZE --> TIMEOUT --> EXEC --> CAPTURE

    style Security fill:#ffcdd2,stroke:#f44336
    style Execution fill:#e8f5e9,stroke:#4caf50
```

**Parameters:**

| Name          | Type     | Required | Description        |
| ------------- | -------- | -------- | ------------------ |
| `command`     | string   | Yes      | Command to execute |
| `args`        | []string | No       | Command arguments  |
| `timeout`     | string   | No       | Execution timeout  |
| `working_dir` | string   | No       | Working directory  |

**Example:**

```json
{
  "name": "execute_command",
  "arguments": {
    "command": "go",
    "args": ["build", "-o", "app"],
    "timeout": "60s",
    "working_dir": "/project"
  }
}
```

### system_info

Get system information.

**Parameters:**

| Name      | Type     | Required | Description                             |
| --------- | -------- | -------- | --------------------------------------- |
| `include` | []string | No       | Info to include (os, cpu, memory, disk) |

**Example:**

```json
{
  "name": "system_info",
  "arguments": {
    "include": ["os", "cpu", "memory"]
  }
}
```

**Response:**

```json
{
  "os": {
    "platform": "darwin",
    "architecture": "arm64",
    "version": "14.0"
  },
  "cpu": {
    "cores": 8,
    "model": "Apple M1"
  },
  "memory": {
    "total": "16GB",
    "available": "8GB"
  }
}
```

### echo

Echo back the input (useful for testing).

**Parameters:**

| Name      | Type   | Required | Description     |
| --------- | ------ | -------- | --------------- |
| `message` | string | Yes      | Message to echo |

**Example:**

```json
{
  "name": "echo",
  "arguments": {
    "message": "Hello, TFO-MCP!"
  }
}
```

---

## Resource Operations

### Resource Types

```mermaid
flowchart TB
    subgraph Resources["Resource Types"]
        FILE["File Resources<br/>file://"]
        HTTP["HTTP Resources<br/>http://, https://"]
        CUSTOM["Custom Resources<br/>custom://"]
    end

    subgraph Operations["Operations"]
        LIST["List"]
        READ["Read"]
        SUBSCRIBE["Subscribe"]
    end

    FILE --> Operations
    HTTP --> Operations
    CUSTOM --> Operations

    style FILE fill:#e3f2fd,stroke:#2196f3
    style HTTP fill:#e8f5e9,stroke:#4caf50
    style CUSTOM fill:#fff3e0,stroke:#ff9800
```

### Resource URI Schemes

| Scheme      | Description       | Example                        |
| ----------- | ----------------- | ------------------------------ |
| `file://`   | Local file system | `file:///path/to/file.txt`     |
| `http://`   | HTTP resource     | `http://api.example.com/data`  |
| `https://`  | HTTPS resource    | `https://api.example.com/data` |
| `custom://` | Custom handler    | `custom://myresource/id`       |

---

## Prompt Operations

### Prompt Lifecycle

```mermaid
stateDiagram-v2
    [*] --> Registered: Register Prompt
    Registered --> Listed: prompts/list
    Registered --> Retrieved: prompts/get
    Retrieved --> Executed: Use with Claude
    Executed --> Retrieved: Reuse
    Registered --> Unregistered: Unregister
    Unregistered --> [*]
```

### Built-in Prompts

| Prompt         | Description           | Arguments                    |
| -------------- | --------------------- | ---------------------------- |
| `code_review`  | Code review assistant | code, language               |
| `explain_code` | Code explanation      | code, language, detail_level |
| `debug_help`   | Debugging assistant   | error_message, code_context  |
| `write_docs`   | Documentation writer  | code, doc_type               |

---

## Session Management

### Session Lifecycle

```mermaid
stateDiagram-v2
    [*] --> Created: Start Server
    Created --> Initializing: initialize
    Initializing --> Ready: initialized
    Ready --> Ready: Handle Requests
    Ready --> Closing: shutdown
    Closing --> Closed: Complete
    Closed --> [*]

    Ready --> Error: Error Occurred
    Error --> Ready: Recover
    Error --> Closed: Fatal Error
```

### Session States

| State        | Description                      | Valid Operations |
| ------------ | -------------------------------- | ---------------- |
| Created      | Session created, not initialized | initialize       |
| Initializing | Processing initialize request    | -                |
| Ready        | Session ready for requests       | All MCP methods  |
| Closing      | Session shutting down            | -                |
| Closed       | Session terminated               | -                |

---

## Examples

### Complete Session Example

```mermaid
sequenceDiagram
    participant Client
    participant Server

    Note over Client,Server: Session Initialization
    Client->>Server: initialize
    Server-->>Client: InitializeResult
    Client->>Server: notifications/initialized

    Note over Client,Server: List Available Tools
    Client->>Server: tools/list
    Server-->>Client: ToolListResult

    Note over Client,Server: Call Claude Tool
    Client->>Server: tools/call (claude_conversation)
    Server-->>Client: CallToolResult

    Note over Client,Server: Read a Resource
    Client->>Server: resources/read
    Server-->>Client: ResourceReadResult

    Note over Client,Server: Session Shutdown
    Client->>Server: shutdown
    Server-->>Client: (acknowledged)
```

### CLI Usage Examples

```bash
# Start server with default config
tfo-mcp run

# Start with debug logging
tfo-mcp run --log-level debug

# Start with custom config
tfo-mcp run --config /etc/tfo-mcp/production.yaml

# Validate configuration
tfo-mcp validate --verbose

# Show version
tfo-mcp version

# Get help
tfo-mcp help
tfo-mcp help run
```

### JSON-RPC Examples

```bash
# Initialize session
echo '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2024-11-05","clientInfo":{"name":"test","version":"1.0.0"}}}' | tfo-mcp run

# List tools
echo '{"jsonrpc":"2.0","id":2,"method":"tools/list","params":{}}' | tfo-mcp run

# Call Claude conversation tool
echo '{"jsonrpc":"2.0","id":3,"method":"tools/call","params":{"name":"claude_conversation","arguments":{"message":"Hello!"}}}' | tfo-mcp run
```

---

## Related Documentation

- [Architecture Guide](ARCHITECTURE.md)
- [Configuration Guide](CONFIGURATION.md)
- [Development Guide](DEVELOPMENT.md)
- [Troubleshooting Guide](TROUBLESHOOTING.md)

---

<div align="center">

**[Back to Documentation Index](README.md)**

</div>
