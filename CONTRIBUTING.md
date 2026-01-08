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

# Contributing to TelemetryFlow Platform Overview Documentation

Thank you for your interest in contributing to TelemetryFlow Platform! This document provides guidelines and information for contributors.

## Table of Contents

- [Code of Conduct](#code-of-conduct)
- [Getting Started](#getting-started)
- [Development Setup](#development-setup)
- [Project Structure](#project-structure)
- [Module Standardization](#module-standardization)
- [Making Changes](#making-changes)
- [Testing](#testing)
- [Submitting Changes](#submitting-changes)
- [Style Guide](#style-guide)
- [Architecture Guidelines](#architecture-guidelines)
- [Release Process](#release-process)

## Code of Conduct

By participating in this project, you agree to maintain a respectful and inclusive environment. We expect all contributors to:

- Be respectful and constructive in discussions
- Welcome newcomers and help them learn
- Focus on what is best for the community
- Show empathy towards other community members

## Getting Started

### Prerequisites

- Node.js 18+ or higher
- pnpm 8+ (preferred package manager)
- Docker & Docker Compose
- Git

### Fork and Clone

1. Fork the repository on GitHub
2. Clone your fork locally:

```bash
git clone https://github.com/YOUR_USERNAME/telemetryflow-overview.git
cd telemetryflow-overview
```

3. Add the upstream remote:

```bash
git remote add upstream https://github.com/telemetryflow/telemetryflow-overview.git
```

## Development Setup

### Install Dependencies

```bash
# Install dependencies
pnpm install

# Configure environment
cp .env.example .env
# Edit .env with your configuration

# Generate secrets
pnpm generate:secrets

# Start infrastructure
docker-compose --profile core up -d

# Run migrations and seeds
pnpm db:migrate:seed

# Start development server
pnpm dev
```

### Verify Setup

```bash
# Run tests
pnpm test

# Run linter
pnpm lint

# Check health endpoint
curl http://localhost:3000/health

# Access Swagger UI
open http://localhost:3000/api
```

### IDE Setup

We recommend using an IDE with TypeScript support:

- **VS Code** with TypeScript and NestJS extensions
- **WebStorm** by JetBrains
- **Vim/Neovim** with TypeScript LSP

## Project Structure

TelemetryFlow Platform follows Domain-Driven Design (DDD) and CQRS patterns. This overview documentation is organized as follows:

```
telemetryflow-overview/
├── README.md                          # Platform overview
├── CHANGELOG.md                       # Version history
├── CONTRIBUTING.md                    # Contribution guidelines
├── SECURITY.md                        # Security policy
├── LICENSE                            # Apache 2.0 license
├── architecture/                      # Architecture documentation
│   ├── 01-SYSTEM-ARCHITECTURE.md      # High-level system architecture
│   ├── 02-DATA-FLOW.md                # How data flows through the system
│   ├── 03-MULTI-TENANCY.md            # Multi-tenancy architecture
│   ├── 04-SECURITY.md                 # Security architecture
│   ├── 05-PERFORMANCE.md              # Performance optimizations
│   └── 06-RBAC-SYSTEM-PLATFORM.md     # 5-Tier RBAC system
├── backend/                           # Backend documentation
│   ├── 00-BACKEND-OVERVIEW.md         # Backend architecture overview
│   ├── 01-TECH-STACK.md               # Technology stack details
│   ├── 02-DDD-CQRS.md                 # Domain-Driven Design & CQRS patterns
│   ├── 03-MODULE-STRUCTURE.md         # Standard module structure
│   └── modules/                       # Module-specific documentation
├── frontend/                          # Frontend documentation
│   ├── 00-FRONTEND-OVERVIEW.md        # Frontend architecture overview
│   ├── 01-TECH-STACK.md               # Vue 3 + TypeScript stack
│   └── components/                    # Component documentation
├── deployment/                        # Deployment guides
│   ├── 01-DOCKER.md                   # Docker deployment
│   ├── 02-KUBERNETES.md               # Kubernetes deployment
│   └── 03-PRODUCTION.md               # Production guidelines
├── shared/                            # Shared documentation
│   ├── 01-API-CONVENTIONS.md          # API conventions
│   └── 02-CODING-STANDARDS.md         # Coding standards
└── tfo-otel/                          # OpenTelemetry documentation
    ├── 01-OTEL-OVERVIEW.md            # OTEL architecture
    └── 02-COLLECTOR-CONFIG.md         # Collector configuration
```

### Module Structure (DDD)

Each module follows the standardized DDD structure:

```
src/modules/{module}/
├── domain/                     # Business logic layer
│   ├── aggregates/             # Domain aggregates
│   ├── entities/               # Domain entities
│   ├── value-objects/          # Value objects
│   ├── events/                 # Domain events
│   ├── repositories/           # Repository interfaces
│   └── services/               # Domain services
├── application/                # Use cases layer (CQRS)
│   ├── commands/               # Write operations
│   ├── queries/                # Read operations
│   ├── handlers/               # Command/Query handlers
│   └── dto/                    # Application DTOs
├── infrastructure/             # Technical implementation
│   ├── persistence/            # Database layer
│   └── messaging/              # Event processors
├── presentation/               # API layer
│   ├── controllers/            # REST controllers
│   ├── dto/                    # Request/Response DTOs
│   ├── guards/                 # Authorization guards
│   └── decorators/             # Custom decorators
├── __tests__/                  # Module tests
├── docs/                       # Module documentation
└── {module}.module.ts          # NestJS module
```

## Module Standardization

TelemetryFlow Platform follows strict module standardization guidelines to ensure consistency, quality, and maintainability. All modules must pass comprehensive quality gates before being considered complete.

### Standardization Specifications

The project includes detailed standardization specifications for all modules:

- **IAM Module**: `.kiro/specs/iam-module-standardization/`
- **Audit Module**: `.kiro/specs/audit-module-standardization/`
- **Auth Module**: `.kiro/specs/auth-module-standardization/`
- **Cache Module**: `.kiro/specs/cache-module-standardization/`

Each specification includes:
- `requirements.md` - 8 major requirements with 80 acceptance criteria using EARS patterns
- `design.md` - DDD architecture, components, interfaces, and 8 correctness properties
- `tasks.md` - 52-60 detailed implementation tasks with checkpoints

### Quality Gates

All modules must pass these standardization gates:

#### Gate 1: Documentation (100% Complete)
- ✅ Root README.md (500+ lines) with comprehensive sections
- ✅ Complete documentation structure (INDEX.md, ERD.mermaid.md, DFD.mermaid.md)
- ✅ Testing documentation (TESTING.md, TEST_PATTERNS.md)
- ✅ API documentation (openapi.yaml, endpoints.md, authentication.md)

#### Gate 2: Test Coverage (≥90%)
- ✅ Domain layer: ≥95% coverage (business logic is critical)
- ✅ Application layer: ≥90% coverage (use cases and handlers)
- ✅ Infrastructure layer: ≥85% coverage (database and external integrations)
- ✅ Presentation layer: ≥85% coverage (controllers and DTOs)
- ✅ Overall module: ≥90% coverage

#### Gate 3: DDD Structure (100% Compliant)
- ✅ Proper domain/application/infrastructure/presentation layering
- ✅ Standardized file naming and organization
- ✅ Barrel exports (index.ts) in all directories
- ✅ TypeScript path mapping for clean imports

#### Gate 4: Database Patterns (100% Compliant)
- ✅ Migration naming: `{timestamp}-{Description}.ts`
- ✅ Seed naming: `{timestamp}-seed-{module}-{entity}.ts`
- ✅ Environment variables (no hardcoded values)
- ✅ Proper foreign keys and indexes

#### Gate 5: API Standards (100% Compliant)
- ✅ Swagger decorators on all endpoints
- ✅ Validation decorators on all DTOs
- ✅ Permission guards on protected endpoints
- ✅ REST conventions and error handling

#### Gate 6: Build & Quality (0 Errors)
- ✅ `pnpm build` succeeds with 0 errors
- ✅ `pnpm lint` succeeds with 0 errors
- ✅ `pnpm test` succeeds with 0 failures
- ✅ Coverage thresholds met

### Working with Specifications

#### Viewing Specifications
```bash
# View IAM module requirements
cat .kiro/specs/iam-module-standardization/requirements.md

# View design document
cat .kiro/specs/iam-module-standardization/design.md

# View implementation tasks
cat .kiro/specs/iam-module-standardization/tasks.md
```

#### Implementation Workflow
1. **Review Requirements**: Read the requirements.md file for acceptance criteria
2. **Study Design**: Review the design.md file for architecture and components
3. **Follow Tasks**: Implement tasks from tasks.md in order
4. **Validate Quality**: Ensure all quality gates pass
5. **Update Documentation**: Keep documentation current with changes

#### Property-Based Testing
Each module includes 8 correctness properties for comprehensive testing:
1. **Idempotency**: Operations produce same result when repeated
2. **Consistency**: Data remains consistent across operations
3. **Validation**: All inputs are properly validated
4. **Authorization**: Access control is enforced
5. **Persistence**: Data is correctly saved and retrieved
6. **Event Handling**: Domain events are properly published
7. **Error Handling**: Errors are handled gracefully
8. **Performance**: Operations meet performance requirements

### Standardization Tools

#### Coverage Enforcement
```bash
# Check test coverage
pnpm test:cov

# Coverage thresholds are enforced in jest.config.js
{
  "coverageThreshold": {
    "global": { "branches": 90, "functions": 90, "lines": 90, "statements": 90 },
    "src/modules/*/domain/**": { "branches": 95, "functions": 95, "lines": 95, "statements": 95 }
  }
}
```

#### Quality Validation
```bash
# Run all quality checks
pnpm lint && pnpm test && pnpm build

# Check for circular dependencies
pnpm madge --circular src/

# Validate naming conventions
pnpm validate:naming
```

#### Documentation Generation
```bash
# Generate module documentation
pnpm generate:docs --module iam

# Generate API documentation
pnpm generate:api-docs

# Export OpenAPI specification
./scripts/export-swagger-docs.sh
```

## Making Changes

### Branch Naming

Use descriptive branch names:

- `feature/add-user-management-api`
- `fix/role-permission-validation`
- `docs/update-api-documentation`
- `refactor/simplify-domain-events`
- `standardization/iam-module-quality-gates`

### Creating a Branch

```bash
# Sync with upstream
git fetch upstream
git checkout main
git merge upstream/main

# Create feature branch
git checkout -b feature/your-feature-name
```

### Commit Messages

Follow conventional commits format:

```
type(scope): short description

Longer description if needed.

Fixes #123
```

**Types:**
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `test`: Adding or updating tests
- `refactor`: Code refactoring
- `chore`: Maintenance tasks
- `standardization`: Module standardization work

**Examples:**

```
feat(iam): add user role assignment API

Implement role assignment functionality with proper validation
and audit logging for the IAM module.

Fixes #45
```

```
standardization(audit): implement quality gates for audit module

Added comprehensive test coverage, documentation, and validation
to meet module standardization requirements.

- Added 95% test coverage for domain layer
- Created complete API documentation
- Implemented property-based testing
- Added automated quality validation

Closes #67
```

## Testing

### Running Tests

```bash
# Run all tests
pnpm test

# Run tests with coverage
pnpm test:cov

# Run tests in watch mode
pnpm test:watch

# Run specific test file
pnpm test src/modules/iam/domain/aggregates/User.spec.ts

# Run tests matching a pattern
pnpm test --testNamePattern="User"

# Run BDD API tests
pnpm test:bdd
```

### Writing Tests

Follow these guidelines for tests:

1. **Unit Tests**: Test individual functions and methods in isolation
2. **Integration Tests**: Test interactions between layers
3. **E2E Tests**: Test complete user workflows
4. **Property-Based Tests**: Test correctness properties

Example test structure:

```typescript
import { User } from '../User';
import { Email } from '../../value-objects/Email';
import { UserCreatedEvent } from '../../events/UserCreated.event';

describe('User Aggregate', () => {
  describe('create', () => {
    it('should create user with valid email', () => {
      // Arrange
      const email = Email.create('test@example.com');

      // Act
      const user = User.create(email);

      // Assert
      expect(user.email).toEqual(email);
      expect(user.isActive).toBe(true);
      expect(user.getUncommittedEvents()).toHaveLength(1);
      expect(user.getUncommittedEvents()[0]).toBeInstanceOf(UserCreatedEvent);
    });

    it('should throw error for invalid email', () => {
      // Act & Assert
      expect(() => Email.create('invalid-email')).toThrow();
    });
  });
});
```

### Test Coverage Requirements

We maintain high test coverage standards:

- **Domain layer**: 95%+ (business logic is critical)
- **Application layer**: 90%+ (use cases and handlers)
- **Infrastructure layer**: 85%+ (database and external integrations)
- **Presentation layer**: 85%+ (controllers and DTOs)
- **Overall**: 90%+

### Property-Based Testing

Each module must implement 8 correctness properties:

```typescript
describe('User Aggregate Properties', () => {
  it('should maintain idempotency', () => {
    // Test that operations produce same result when repeated
  });

  it('should maintain consistency', () => {
    // Test that data remains consistent across operations
  });

  it('should validate all inputs', () => {
    // Test that all inputs are properly validated
  });

  // ... 5 more properties
});
```

## Submitting Changes

### Before Submitting

1. **Run tests**: `pnpm test`
2. **Run linter**: `pnpm lint`
3. **Check build**: `pnpm build`
4. **Check coverage**: `pnpm test:cov`
5. **Update documentation** if needed
6. **Add tests** for new functionality
7. **Validate standardization** if working on modules

### Pull Request Process

1. Push your branch to your fork:

```bash
git push origin feature/your-feature-name
```

2. Create a Pull Request on GitHub

3. Fill in the PR template with:
   - Description of changes
   - Related issue numbers
   - Testing performed
   - Standardization checklist (if applicable)

4. Wait for review and address feedback

### PR Title Format

Use the same format as commit messages:

```
feat(iam): add user role assignment API
fix(auth): handle JWT token expiration
docs(api): update endpoint documentation
standardization(audit): implement quality gates
```

## Style Guide

### TypeScript Style

Follow NestJS and TypeScript best practices:

- Use `prettier` and `eslint` for code formatting and linting
- Use type annotations for all public functions and methods
- Use decorators for NestJS components (@Controller, @Injectable, etc.)
- Prefer interfaces over types for object shapes

### Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| Modules | PascalCase | `IAMModule`, `AuditModule` |
| Classes | PascalCase | `User`, `UserController`, `CreateUserHandler` |
| Interfaces | PascalCase with I prefix | `IUserRepository`, `ILogger` |
| Functions | camelCase | `createUser`, `validateEmail` |
| Constants | UPPER_SNAKE_CASE | `DEFAULT_PAGE_SIZE`, `MAX_RETRY_ATTEMPTS` |
| Files | PascalCase with suffix | `User.entity.ts`, `CreateUser.handler.ts` |

### Error Handling

- Use custom exceptions for domain-specific errors
- Always include helpful error messages
- Use proper HTTP status codes in controllers

```typescript
// Good
export class UserNotFoundError extends Error {
  constructor(userId: string) {
    super(`User with ID ${userId} not found`);
    this.name = 'UserNotFoundError';
  }
}

// Controller error handling
@Get(':id')
async getUser(@Param('id') id: string): Promise<UserResponseDto> {
  try {
    const query = new GetUserQuery(id);
    return await this.queryBus.execute(query);
  } catch (error) {
    if (error instanceof UserNotFoundError) {
      throw new NotFoundException(error.message);
    }
    throw error;
  }
}
```

### Documentation

- Document all public classes, methods, and interfaces
- Use JSDoc comments for TypeScript
- Include type information in documentation

```typescript
/**
 * Creates a new user in the system.
 *
 * @param email - The user's email address
 * @param organizationId - The organization the user belongs to
 * @returns Promise that resolves when user is created
 * @throws {DomainError} When email is invalid or already exists
 *
 * @example
 * ```typescript
 * const command = new CreateUserCommand('user@example.com', 'org-123');
 * await this.commandBus.execute(command);
 * ```
 */
async execute(command: CreateUserCommand): Promise<void> {
  // Implementation...
}
```

## Architecture Guidelines

When contributing, maintain the architectural principles:

### Domain-Driven Design

1. **Aggregates**: Business entities that encapsulate business logic
2. **Value Objects**: Immutable objects that validate on creation
3. **Domain Events**: Represent important business occurrences
4. **Repository Interfaces**: Define how aggregates are persisted

### CQRS Pattern

1. **Commands**: Represent intentions to change state
2. **Queries**: Represent requests for data
3. **Handlers**: Execute commands and queries
4. **DTOs**: Data transfer objects for API communication

### Clean Architecture Principles

1. **Dependency Direction**: Dependencies point inward (Domain ← Application ← Infrastructure ← Presentation)
2. **Layer Isolation**: Each layer has specific responsibilities
3. **Interface Segregation**: Use small, focused interfaces
4. **Dependency Inversion**: Depend on abstractions, not implementations

### Adding New Features

When adding new features to modules:

1. **Start with Domain**: Define aggregates, value objects, and events
2. **Add Use Cases**: Create commands/queries in application layer
3. **Implement Infrastructure**: Add repositories and external integrations
4. **Expose via API**: Create controllers and DTOs in presentation layer
5. **Add Comprehensive Tests**: Unit, integration, and E2E tests
6. **Update Documentation**: Keep all documentation current
7. **Validate Standardization**: Ensure quality gates are met

### Module Standardization

When working on module standardization:

1. **Follow Specifications**: Use the detailed specs in `.kiro/specs/`
2. **Implement Quality Gates**: Ensure all 6 gates pass
3. **Property-Based Testing**: Implement all 8 correctness properties
4. **Documentation First**: Create comprehensive documentation
5. **Automated Validation**: Use tools to enforce standards
6. **Continuous Improvement**: Update standards based on learnings

## Release Process

Releases follow semantic versioning (SemVer):

- **MAJOR**: Breaking API changes
- **MINOR**: New features, backward compatible
- **PATCH**: Bug fixes, backward compatible

### CI/CD Workflows

The project uses GitHub Actions for CI/CD:

| Workflow       | Trigger           | Purpose                                 |
| -------------- | ----------------- | --------------------------------------- |
| `ci.yml`       | Push/PR           | Lint, test, build verification          |
| `docker.yml`   | Push to main/tags | Build Docker images                     |
| `release.yml`  | Tags (v*.*.*)     | Create GitHub release                   |

### Changelog

Update CHANGELOG.md with your changes under the "Unreleased" section.

### Creating a Release

1. Update version in `package.json`
2. Update version badge in `README.md` and `CONTRIBUTING.md`
3. Move "Unreleased" section in CHANGELOG.md to new version
4. Create and push tag:

```bash
git tag v1.2.0
git push origin v1.2.0
```

GitHub Actions will automatically:

- Run tests
- Build Docker images
- Create GitHub release

## Getting Help

- **Questions**: Open a GitHub Discussion
- **Bugs**: Open a GitHub Issue with the bug template
- **Features**: Open a GitHub Issue with the feature request template
- **Security**: Email security@devopscorner.id (do not open public issues)
- **Standardization**: Review specs in `.kiro/specs/` directory

## Recognition

Contributors are recognized in:

- GitHub Contributors page
- CHANGELOG.md for significant contributions
- README.md acknowledgments section

Thank you for contributing to TelemetryFlow Platform!

---

Built with care by the **DevOpsCorner Indonesia** community
