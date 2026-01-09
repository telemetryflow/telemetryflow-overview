# TelemetryFlow MCP Server Documentation

- **Version:** 1.1.2
- **MCP Protocol:** 2024-11-05
- **Last Updated:** January 2026
- **Status:** Production Ready

---

## Documentation Index

| Document                                 | Description                                           |
| ---------------------------------------- | ----------------------------------------------------- |
| [ARCHITECTURE.md](ARCHITECTURE.md)       | System architecture, DDD patterns, component overview |
| [ERD.md](ERD.md)                         | Entity Relationship Diagrams, database schemas        |
| [DFD.md](DFD.md)                         | Data Flow Diagrams, state machines, sequence diagrams |
| [CONFIGURATION.md](CONFIGURATION.md)     | Complete configuration reference                      |
| [COMMANDS.md](COMMANDS.md)               | CLI commands and usage                                |
| [DEVELOPMENT.md](DEVELOPMENT.md)         | Development guide, coding standards, testing          |
| [INSTALLATION.md](INSTALLATION.md)       | Installation and deployment guide                     |
| [TROUBLESHOOTING.md](TROUBLESHOOTING.md) | Common issues and solutions                           |

---

## Quick Links

### Getting Started

- [Installation Guide](INSTALLATION.md)
- [Configuration Reference](CONFIGURATION.md)
- [CLI Commands](COMMANDS.md)

### Architecture

- [System Architecture](ARCHITECTURE.md#system-architecture)
- [DDD Patterns](ARCHITECTURE.md#domain-driven-design)
- [CQRS Implementation](ARCHITECTURE.md#cqrs-pattern)
- [MCP Protocol Flow](ARCHITECTURE.md#mcp-protocol-flow)

### Diagrams

- [Entity Relationship Diagrams](ERD.md)
- [Data Flow Diagrams](DFD.md)
- [State Machines](DFD.md#state-diagrams)
- [Sequence Diagrams](DFD.md#sequence-diagrams)

### Development

- [Development Setup](DEVELOPMENT.md#setup)
- [Coding Standards](DEVELOPMENT.md#coding-standards)
- [Testing Guide](DEVELOPMENT.md#testing)
- [Contributing](DEVELOPMENT.md#contributing)

---

## Component Overview

```mermaid
graph TB
    subgraph "Documentation Structure"
        README[README.md<br/>Project Overview]
        ARCH[ARCHITECTURE.md<br/>System Design]
        ERD[ERD.md<br/>Entity Diagrams]
        DFD[DFD.md<br/>Data Flow]
        CONFIG[CONFIGURATION.md<br/>Config Reference]
        CMD[COMMANDS.md<br/>CLI Usage]
        DEV[DEVELOPMENT.md<br/>Dev Guide]
        INSTALL[INSTALLATION.md<br/>Setup Guide]
        TROUBLE[TROUBLESHOOTING.md<br/>Problem Solving]
    end

    README --> INSTALL
    INSTALL --> CONFIG
    CONFIG --> CMD
    README --> ARCH
    ARCH --> ERD
    ARCH --> DFD
    ERD --> DEV
    DFD --> DEV
    DEV --> TROUBLE

    style README fill:#E1BEE7,stroke:#7B1FA2,stroke-width:2px
    style ARCH fill:#C8E6C9,stroke:#388E3C
    style ERD fill:#FFF3E0,stroke:#FF9800
    style DFD fill:#FCE4EC,stroke:#E91E63
    style CONFIG fill:#BBDEFB,stroke:#1976D2
```

---

## Version History

| Version | Date         | Changes                               |
| ------- | ------------ | ------------------------------------- |
| 1.1.2   | January 2026 | Initial release with full MCP support |

---

## Additional Resources

- **Main README**: [../README.md](../README.md)
- **GitHub Repository**: [devopscorner/telemetryflow](https://github.com/devopscorner/telemetryflow)
- **MCP Specification**: [modelcontextprotocol.io](https://modelcontextprotocol.io)
- **Anthropic Claude API**: [docs.anthropic.com](https://docs.anthropic.com)
