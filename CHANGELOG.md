<div align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://github.com/telemetryflow/.github/raw/main/docs/assets/tfo-logo-dark.svg">
    <source media="(prefers-color-scheme: light)" srcset="https://github.com/telemetryflow/.github/raw/main/docs/assets/tfo-logo-light.svg">
    <img src="https://github.com/telemetryflow/.github/raw/main/docs/assets/tfo-logo-light.svg" alt="TelemetryFlow Logo" width="80%">
  </picture>

  <h3>TelemetryFlow Platform - Overview Documentation</h3>

[![Version](https://img.shields.io/badge/version-1.1.2--CE-orange.svg)](CHANGELOG.md)
[![License](https://img.shields.io/badge/license-Apache--2.0-green.svg)](LICENSE)
[![NestJS](https://img.shields.io/badge/NestJS-11.x-E0234E?logo=nestjs)](https://nestjs.com/)
[![Vue](https://img.shields.io/badge/Vue-3.5.24-4FC08D?logo=vue.js)](https://vuejs.org/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.9.3-3178C6?logo=typescript)](https://www.typescriptlang.org/)
[![ClickHouse](https://img.shields.io/badge/ClickHouse-23+-FFCC00?logo=clickhouse)](https://clickhouse.com/)
[![OpenTelemetry](https://img.shields.io/badge/OTLP-100%25%20Compliant-success?logo=opentelemetry)](https://opentelemetry.io/)
[![DDD](https://img.shields.io/badge/Architecture-DDD%2FCQRS-blueviolet)](backend/)
[![RBAC](https://img.shields.io/badge/Security-5--Tier%20RBAC-red)](architecture/06-RBAC-SYSTEM-PLATFORM.md)

<p align="center">
  <strong>TelemetryFlow Platform</strong> is an enterprise-grade observability platform providing complete telemetry collection, storage, and visualization capabilities. 100% OpenTelemetry Protocol (OTLP) compliant with unified metrics, logs, and traces collection.
</p>

</div>

---

# Changelog

All notable changes to **TelemetryFlow Platform Overview Documentation** will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.1.2] - 2026-01-09

### Summary

**Documentation Repository Alignment** - Updated repository branding and documentation to accurately reflect the TelemetryFlow Platform Overview Documentation repository. Improved consistency across CONTRIBUTING, SECURITY, and CHANGELOG files.

**Key Highlights:**

- Updated branding from "TelemetryFlow Core" to "TelemetryFlow Platform - Overview Documentation"
- Aligned CONTRIBUTING.md with platform documentation standards
- Updated SECURITY.md with current version and contact information
- Cleaned up CHANGELOG to reflect documentation-specific changes

### Changed

#### CONTRIBUTING.md

- Updated project structure to reflect documentation repository layout
- Added documentation-specific contribution guidelines
- Updated development workflow for documentation contributions
- Added proper references to architecture, backend, frontend, and deployment docs

#### SECURITY.md

- Updated version to 1.1.2-CE
- Updated security contact information
- Refreshed security best practices section
- Updated last modified date to January 2026

#### CHANGELOG.md

- Restructured to focus on documentation changes
- Removed code-specific changelog entries (IAM module, test coverage, etc.)
- Added proper versioning for documentation releases

---

## [1.1.1] - 2026-01-01

### Summary

**Documentation Structure Enhancement** - Comprehensive reorganization of platform documentation with improved navigation and content structure.

**Key Highlights:**

- Reorganized documentation into clear sections (architecture, backend, frontend, deployment)
- Added 15+ module documentation files
- Enhanced shared services documentation
- Added OpenTelemetry collector documentation

### Added

#### Architecture Documentation

- [01-SYSTEM-ARCHITECTURE.md](architecture/01-SYSTEM-ARCHITECTURE.md) - High-level system architecture
- [02-DATA-FLOW.md](architecture/02-DATA-FLOW.md) - Data flow through the system
- [03-MULTI-TENANCY.md](architecture/03-MULTI-TENANCY.md) - Multi-tenancy architecture
- [04-SECURITY.md](architecture/04-SECURITY.md) - Security architecture
- [05-PERFORMANCE.md](architecture/05-PERFORMANCE.md) - Performance optimizations
- [06-RBAC-SYSTEM-PLATFORM.md](architecture/06-RBAC-SYSTEM-PLATFORM.md) - 5-Tier RBAC system

#### Backend Documentation

- [00-BACKEND-OVERVIEW.md](backend/00-BACKEND-OVERVIEW.md) - Backend architecture overview
- [01-TECH-STACK.md](backend/01-TECH-STACK.md) - Technology stack details
- [02-DDD-CQRS.md](backend/02-DDD-CQRS.md) - Domain-Driven Design & CQRS patterns
- [03-MODULE-STRUCTURE.md](backend/03-MODULE-STRUCTURE.md) - Standard module structure

#### Backend Module Documentation

- [100-core.md](backend/modules/100-core.md) - Core IAM module
- [200-auth.md](backend/modules/200-auth.md) - Authentication module
- [300-api-keys.md](backend/modules/300-api-keys.md) - API keys module
- [400-telemetry.md](backend/modules/400-telemetry.md) - Telemetry ingestion module
- [500-monitoring.md](backend/modules/500-monitoring.md) - Uptime monitoring module
- Additional module documentation (600-900 series)

#### Backend Shared Services Documentation

- Cache service documentation
- Logging service documentation
- Queue service documentation
- OpenTelemetry integration documentation

#### Frontend Documentation

- [00-FRONTEND-OVERVIEW.md](frontend/00-FRONTEND-OVERVIEW.md) - Frontend architecture overview
- [01-TECH-STACK.md](frontend/01-TECH-STACK.md) - Vue.js technology stack
- [02-MODULE-STRUCTURE.md](frontend/02-MODULE-STRUCTURE.md) - Frontend module structure
- [03-STATE-MANAGEMENT.md](frontend/03-STATE-MANAGEMENT.md) - Pinia state management
- [04-ROUTING.md](frontend/04-ROUTING.md) - Vue Router configuration
- [05-VISUALIZATION.md](frontend/05-VISUALIZATION.md) - Data visualization components

#### Deployment Documentation

- [CONFIGURATION.md](deployment/CONFIGURATION.md) - Environment configuration guide
- [DOCKER-COMPOSE.md](deployment/DOCKER-COMPOSE.md) - Docker Compose deployment
- [KUBERNETES.md](deployment/KUBERNETES.md) - Kubernetes deployment guide
- [PRODUCTION-CHECKLIST.md](deployment/PRODUCTION-CHECKLIST.md) - Production readiness checklist

#### OpenTelemetry Collector Documentation

- [tfo-otel/](tfo-otel/) - TelemetryFlow OpenTelemetry Collector configuration

### Changed

- Improved README.md with clearer navigation structure
- Enhanced cross-references between documentation sections
- Updated Mermaid diagrams for better visualization

---

## [1.1.0] - 2025-12-15

### Summary

**Initial Documentation Release** - First release of the TelemetryFlow Platform Overview Documentation repository.

**Key Highlights:**

- Complete platform overview documentation
- Architecture documentation with 35+ Mermaid diagrams
- Backend and frontend technology stack documentation
- Deployment guides for Docker and Kubernetes

### Added

#### Platform Overview

- README.md with comprehensive platform introduction
- Problem/solution documentation
- Core capabilities overview
- Technology stack summary

#### Documentation Structure

- Organized directory structure for scalable documentation
- Consistent formatting and styling across all documents
- Cross-referenced navigation between sections

#### Architecture

- System architecture diagrams
- Data flow documentation
- Multi-tenancy design patterns
- Security architecture overview
- Performance optimization guides
- 5-Tier RBAC system documentation

#### Backend

- NestJS backend architecture
- DDD/CQRS pattern implementation guides
- Module structure standards
- 15+ module documentation files

#### Frontend

- Vue.js 3.5 frontend architecture
- State management with Pinia
- Routing configuration
- Visualization component library

#### Deployment

- Docker Compose configuration
- Kubernetes deployment manifests
- Production checklist
- Environment configuration guide

### Technical Details

**Documentation Statistics:**

- Total Markdown Files: 40+
- Mermaid Diagrams: 35+
- Code Examples: 100+
- API Endpoints Documented: 50+

**Coverage:**

- Architecture: Complete system design documentation
- Backend: All 15+ modules documented
- Frontend: All major components documented
- Deployment: Docker, Kubernetes, and production guides

---

## [1.0.0] - 2025-12-01

### Summary

**Repository Creation** - Initial creation of the TelemetryFlow Platform Overview Documentation repository.

### Added

- Initial repository structure
- Apache 2.0 license
- Basic README.md placeholder
- CONTRIBUTING.md guidelines
- SECURITY.md policy

---

## Documentation Guidelines

### Versioning

This documentation repository follows semantic versioning:
- **Major (X.0.0)**: Significant restructuring or new major sections
- **Minor (0.X.0)**: New documentation files or substantial updates
- **Patch (0.0.X)**: Corrections, clarifications, and minor updates

### Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for documentation contribution guidelines.

### Security

See [SECURITY.md](SECURITY.md) for security policy and vulnerability reporting.

---

## Release Information

- **Current Version**: 1.1.2-CE
- **Release Date**: January 9th, 2026
- **Status**: Production Documentation
- **License**: Apache-2.0

## Related Repositories

| Repository                                                                          | Description                    |
| ----------------------------------------------------------------------------------- | ------------------------------ |
| [telemetryflow-platform](https://github.com/telemetryflow/telemetryflow-platform)   | Main platform implementation   |
| [telemetryflow-collector](https://github.com/telemetryflow/telemetryflow-collector) | OpenTelemetry collector (Go)   |
| [telemetryflow-agent](https://github.com/telemetryflow/telemetryflow-agent)         | Telemetry agent (Go)           |
| [telemetryflow-go-sdk](https://github.com/telemetryflow/telemetryflow-go-sdk)       | Go SDK for telemetry           |
| [order-service](https://github.com/telemetryflow/order-service)                     | Example microservice           |

## Contributors

- DevOpsCorner Indonesia Team

## Acknowledgments

Documentation for [TelemetryFlow Platform](https://github.com/telemetryflow/telemetryflow-platform) - Enterprise Telemetry & Observability Platform.

---

Built with love by DevOpsCorner Indonesia
