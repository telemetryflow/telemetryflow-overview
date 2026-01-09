# TFO-MCP Git Workflow

> Git workflow and branching strategy for TelemetryFlow MCP Server

---

## Table of Contents

- [Overview](#overview)
- [Branching Strategy](#branching-strategy)
- [Branch Types](#branch-types)
- [Workflow Diagrams](#workflow-diagrams)
- [Commit Guidelines](#commit-guidelines)
- [Pull Request Process](#pull-request-process)
- [Release Process](#release-process)
- [Hotfix Process](#hotfix-process)
- [Best Practices](#best-practices)

---

## Overview

TFO-MCP follows a Git Flow-inspired branching strategy optimized for continuous delivery.

### Workflow Philosophy

```mermaid
flowchart LR
    subgraph Philosophy["Workflow Principles"]
        STABLE["Main is Always<br/>Deployable"]
        FEATURE["Features in<br/>Isolation"]
        REVIEW["All Changes<br/>Reviewed"]
        CI["Continuous<br/>Integration"]
    end

    STABLE --> FEATURE --> REVIEW --> CI --> STABLE

    style Philosophy fill:#e3f2fd,stroke:#2196f3
```

---

## Branching Strategy

### Branch Hierarchy

```mermaid
gitGraph
    commit id: "initial" tag: "v1.0.0"
    branch develop
    checkout develop
    commit id: "dev-1"
    branch feature/new-tool
    checkout feature/new-tool
    commit id: "feat-1"
    commit id: "feat-2"
    checkout develop
    merge feature/new-tool
    commit id: "dev-2"
    branch release/1.1.0
    checkout release/1.1.0
    commit id: "rel-1"
    checkout main
    merge release/1.1.0 tag: "v1.1.0"
    checkout develop
    merge release/1.1.0
    commit id: "dev-3"
    branch feature/claude-streaming
    checkout feature/claude-streaming
    commit id: "stream-1"
    checkout develop
    merge feature/claude-streaming
    checkout main
    branch hotfix/critical-bug
    commit id: "hotfix-1"
    checkout main
    merge hotfix/critical-bug tag: "v1.1.1"
    checkout develop
    merge hotfix/critical-bug
```

### Branch Overview

```mermaid
flowchart TB
    subgraph Branches["Branch Types"]
        MAIN["main<br/>Production Ready"]
        DEVELOP["develop<br/>Integration"]
        FEATURE["feature/*<br/>New Features"]
        RELEASE["release/*<br/>Release Prep"]
        HOTFIX["hotfix/*<br/>Critical Fixes"]
    end

    FEATURE -->|"Merge"| DEVELOP
    DEVELOP -->|"Branch"| RELEASE
    RELEASE -->|"Merge"| MAIN
    RELEASE -->|"Back-merge"| DEVELOP
    HOTFIX -->|"Merge"| MAIN
    HOTFIX -->|"Back-merge"| DEVELOP

    style MAIN fill:#c8e6c9,stroke:#388e3c,stroke-width:2px
    style DEVELOP fill:#bbdefb,stroke:#1976d2,stroke-width:2px
    style FEATURE fill:#fff3e0,stroke:#ff9800
    style RELEASE fill:#e1bee7,stroke:#7b1fa2
    style HOTFIX fill:#ffcdd2,stroke:#c62828
```

---

## Branch Types

### Main Branch

```mermaid
flowchart TB
    subgraph Main["main Branch"]
        DESC["Always production-ready<br/>Protected branch<br/>Requires PR approval<br/>Tagged with version"]
    end

    RELEASE -->|"Merge"| Main
    HOTFIX -->|"Merge"| Main

    style Main fill:#c8e6c9,stroke:#388e3c
```

**Rules:**

- Always deployable
- Protected - no direct commits
- Requires PR with approval
- All commits tagged with version

### Develop Branch

```mermaid
flowchart TB
    subgraph Develop["develop Branch"]
        DESC["Integration branch<br/>Feature target<br/>CI/CD runs<br/>Nightly builds"]
    end

    FEATURE -->|"Merge"| Develop
    Develop -->|"Branch"| RELEASE

    style Develop fill:#bbdefb,stroke:#1976d2
```

**Rules:**

- Integration branch for features
- CI runs on every push
- Nightly builds for testing
- Base for feature branches

### Feature Branches

```mermaid
flowchart TB
    subgraph Feature["feature/* Branches"]
        DESC["New functionality<br/>Bug fixes<br/>Improvements<br/>Short-lived"]
    end

    DEVELOP -->|"Branch from"| Feature
    Feature -->|"Merge to"| DEVELOP

    style Feature fill:#fff3e0,stroke:#ff9800
```

**Naming Convention:**

```
feature/<ticket-id>-<short-description>
feature/add-search-tool
feature/TFO-123-claude-streaming
feature/improve-error-handling
```

**Rules:**

- Branch from `develop`
- Merge back to `develop`
- Delete after merge
- Max lifetime: 2 weeks

### Release Branches

```mermaid
flowchart TB
    subgraph Release["release/* Branches"]
        DESC["Version preparation<br/>Bug fixes only<br/>Documentation updates<br/>Version bumps"]
    end

    DEVELOP -->|"Branch from"| Release
    Release -->|"Merge to"| MAIN
    Release -->|"Back-merge"| DEVELOP

    style Release fill:#e1bee7,stroke:#7b1fa2
```

**Naming Convention:**

```
release/<version>
release/1.1.0
release/1.2.0-rc1
```

**Rules:**

- Branch from `develop`
- Only bug fixes allowed
- Merge to `main` and back to `develop`
- Create version tag on merge

### Hotfix Branches

```mermaid
flowchart TB
    subgraph Hotfix["hotfix/* Branches"]
        DESC["Critical production fixes<br/>Branch from main<br/>Merge to main + develop<br/>Immediate release"]
    end

    MAIN -->|"Branch from"| Hotfix
    Hotfix -->|"Merge to"| MAIN
    Hotfix -->|"Back-merge"| DEVELOP

    style Hotfix fill:#ffcdd2,stroke:#c62828
```

**Naming Convention:**

```
hotfix/<version>-<description>
hotfix/1.1.1-session-timeout
hotfix/1.1.2-api-auth-fix
```

**Rules:**

- Branch from `main`
- Critical fixes only
- Merge to `main` and `develop`
- Increment patch version

---

## Workflow Diagrams

### Feature Development Flow

```mermaid
sequenceDiagram
    participant Dev as Developer
    participant Feature as feature/branch
    participant Develop as develop
    participant CI as CI/CD

    Dev->>Feature: Create branch from develop
    Dev->>Feature: Implement feature

    loop Development
        Dev->>Feature: Commit changes
        Feature->>CI: Run tests
        CI-->>Dev: Test results
    end

    Dev->>Feature: Push branch
    Feature->>CI: Full CI pipeline
    CI-->>Dev: Pipeline status

    Dev->>Develop: Create Pull Request
    Develop->>Dev: Code review

    alt Approved
        Develop->>Develop: Merge PR
        Develop->>Feature: Delete branch
    else Changes Requested
        Dev->>Feature: Address feedback
        Dev->>Develop: Update PR
    end
```

### Release Flow

```mermaid
sequenceDiagram
    participant Dev as Developer
    participant Release as release/x.y.z
    participant Main as main
    participant Develop as develop
    participant CI as CI/CD

    Dev->>Release: Create from develop
    Dev->>Release: Version bump

    loop Stabilization
        Dev->>Release: Bug fixes only
        Release->>CI: Run tests
    end

    Dev->>Main: Create PR to main
    Main->>Dev: Approval required

    Main->>Main: Merge release
    Main->>Main: Create tag v.x.y.z

    Main->>CI: Build release
    CI->>CI: Create artifacts
    CI->>CI: Publish release

    Dev->>Develop: Back-merge release
    Dev->>Release: Delete branch
```

### Hotfix Flow

```mermaid
sequenceDiagram
    participant Dev as Developer
    participant Hotfix as hotfix/x.y.z
    participant Main as main
    participant Develop as develop
    participant CI as CI/CD

    Note over Main: Critical bug in production!

    Dev->>Hotfix: Create from main
    Dev->>Hotfix: Fix bug
    Dev->>Hotfix: Bump patch version

    Hotfix->>CI: Run tests
    CI-->>Dev: Tests pass

    Dev->>Main: Create PR
    Main->>Dev: Emergency review

    Main->>Main: Merge hotfix
    Main->>Main: Create tag

    Main->>CI: Build hotfix release
    CI->>CI: Deploy to production

    Dev->>Develop: Back-merge hotfix
    Dev->>Hotfix: Delete branch
```

---

## Commit Guidelines

### Conventional Commits

```mermaid
flowchart LR
    subgraph Format["Commit Format"]
        TYPE["type"]
        SCOPE["(scope)"]
        COLON[":"]
        DESC["description"]
    end

    TYPE --> SCOPE --> COLON --> DESC

    style Format fill:#e3f2fd,stroke:#2196f3
```

**Format:**

```
<type>(<scope>): <description>

[optional body]

[optional footer(s)]
```

### Commit Types

| Type       | Description        | Example                                       |
| ---------- | ------------------ | --------------------------------------------- |
| `feat`     | New feature        | `feat(tools): add file search tool`           |
| `fix`      | Bug fix            | `fix(session): handle timeout correctly`      |
| `docs`     | Documentation      | `docs: update API reference`                  |
| `style`    | Formatting         | `style: fix indentation`                      |
| `refactor` | Code restructuring | `refactor(handlers): simplify error handling` |
| `test`     | Tests              | `test: add session handler tests`             |
| `chore`    | Maintenance        | `chore: update dependencies`                  |
| `perf`     | Performance        | `perf(claude): optimize token counting`       |
| `ci`       | CI/CD              | `ci: add release workflow`                    |
| `build`    | Build system       | `build: update Dockerfile`                    |
| `revert`   | Revert commit      | `revert: feat(tools): add file search tool`   |

### Commit Message Examples

```mermaid
flowchart TB
    subgraph Examples["Commit Examples"]
        E1["feat(tools): add new search_files tool<br/><br/>Implements glob and regex pattern matching<br/>for searching files in directories.<br/><br/>Closes #123"]
        E2["fix(session): resolve timeout issue<br/><br/>Session timeout was not being reset after<br/>successful requests, causing premature<br/>session termination.<br/><br/>Fixes #456"]
        E3["docs: update installation guide<br/><br/>Add Docker and Homebrew installation<br/>instructions."]
    end

    style E1 fill:#c8e6c9,stroke:#388e3c
    style E2 fill:#ffcdd2,stroke:#c62828
    style E3 fill:#e3f2fd,stroke:#2196f3
```

---

## Pull Request Process

### PR Workflow

```mermaid
flowchart TB
    subgraph Create["1. Create PR"]
        C1["Push branch"]
        C2["Open PR"]
        C3["Fill template"]
    end

    subgraph Review["2. Review"]
        R1["Automated checks"]
        R2["Code review"]
        R3["Address feedback"]
    end

    subgraph Merge["3. Merge"]
        M1["Approval required"]
        M2["Squash or merge"]
        M3["Delete branch"]
    end

    Create --> Review --> Merge

    style Create fill:#e3f2fd,stroke:#2196f3
    style Review fill:#fff3e0,stroke:#ff9800
    style Merge fill:#c8e6c9,stroke:#388e3c
```

### PR Checklist

```mermaid
flowchart TB
    subgraph Checklist["PR Checklist"]
        CHECK1["✓ Tests pass"]
        CHECK2["✓ Linting passes"]
        CHECK3["✓ Documentation updated"]
        CHECK4["✓ Changelog updated"]
        CHECK5["✓ No merge conflicts"]
        CHECK6["✓ Reviewed by peer"]
    end

    style Checklist fill:#e8f5e9,stroke:#4caf50
```

### PR Template

```markdown
## Description

Brief description of changes.

## Type of Change

- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update

## Related Issues

Fixes #(issue number)

## Testing

- [ ] Unit tests added
- [ ] Integration tests added
- [ ] Manual testing performed

## Checklist

- [ ] My code follows the style guidelines
- [ ] I have performed a self-review
- [ ] I have commented hard-to-understand code
- [ ] I have updated the documentation
- [ ] My changes generate no new warnings
- [ ] I have added tests proving my fix/feature works
- [ ] All tests pass locally
```

---

## Release Process

### Version Numbering

```mermaid
flowchart LR
    subgraph SemVer["Semantic Versioning"]
        MAJOR["MAJOR<br/>Breaking changes"]
        MINOR["MINOR<br/>New features"]
        PATCH["PATCH<br/>Bug fixes"]
    end

    subgraph Example["Example: 1.1.2"]
        M["1"]
        N["1"]
        P["2"]
    end

    MAJOR --> M
    MINOR --> N
    PATCH --> P

    style SemVer fill:#e3f2fd,stroke:#2196f3
```

### Release Steps

```mermaid
flowchart TB
    subgraph Prepare["1. Prepare"]
        P1["Create release branch"]
        P2["Bump version"]
        P3["Update CHANGELOG"]
        P4["Final testing"]
    end

    subgraph Release["2. Release"]
        R1["Merge to main"]
        R2["Create git tag"]
        R3["Build artifacts"]
        R4["GitHub release"]
    end

    subgraph Post["3. Post-Release"]
        O1["Back-merge to develop"]
        O2["Delete release branch"]
        O3["Announce release"]
    end

    Prepare --> Release --> Post

    style Prepare fill:#e3f2fd,stroke:#2196f3
    style Release fill:#c8e6c9,stroke:#388e3c
    style Post fill:#fff3e0,stroke:#ff9800
```

### Release Commands

```bash
# Create release branch
git checkout develop
git pull origin develop
git checkout -b release/1.2.0

# Bump version in files
# Update CHANGELOG.md
# Commit changes
git commit -m "chore(release): prepare v1.2.0"

# Merge to main
git checkout main
git pull origin main
git merge --no-ff release/1.2.0
git tag -a v1.2.0 -m "Release v1.2.0"
git push origin main --tags

# Back-merge to develop
git checkout develop
git merge --no-ff release/1.2.0
git push origin develop

# Delete release branch
git branch -d release/1.2.0
git push origin --delete release/1.2.0
```

---

## Hotfix Process

### Hotfix Steps

```mermaid
flowchart TB
    subgraph Identify["1. Identify"]
        I1["Critical bug found"]
        I2["Create hotfix branch"]
    end

    subgraph Fix["2. Fix"]
        F1["Implement fix"]
        F2["Bump patch version"]
        F3["Test thoroughly"]
    end

    subgraph Deploy["3. Deploy"]
        D1["Merge to main"]
        D2["Tag and release"]
        D3["Back-merge to develop"]
    end

    Identify --> Fix --> Deploy

    style Identify fill:#ffcdd2,stroke:#c62828
    style Fix fill:#fff3e0,stroke:#ff9800
    style Deploy fill:#c8e6c9,stroke:#388e3c
```

### Hotfix Commands

```bash
# Create hotfix from main
git checkout main
git pull origin main
git checkout -b hotfix/1.1.3-critical-bug

# Fix the bug
# Bump version
git commit -m "fix(critical): resolve production issue"

# Merge to main
git checkout main
git merge --no-ff hotfix/1.1.3-critical-bug
git tag -a v1.1.3 -m "Hotfix v1.1.3"
git push origin main --tags

# Back-merge to develop
git checkout develop
git merge --no-ff hotfix/1.1.3-critical-bug
git push origin develop

# Delete hotfix branch
git branch -d hotfix/1.1.3-critical-bug
```

---

## Best Practices

### Do's and Don'ts

```mermaid
flowchart TB
    subgraph Do["Do"]
        D1["✓ Write clear commit messages"]
        D2["✓ Keep PRs small and focused"]
        D3["✓ Update documentation"]
        D4["✓ Run tests before pushing"]
        D5["✓ Review your own PR first"]
    end

    subgraph Dont["Don't"]
        N1["✗ Force push to shared branches"]
        N2["✗ Commit directly to main"]
        N3["✗ Merge without review"]
        N4["✗ Leave branches stale"]
        N5["✗ Skip the changelog"]
    end

    style Do fill:#c8e6c9,stroke:#388e3c
    style Dont fill:#ffcdd2,stroke:#c62828
```

### Branch Hygiene

```bash
# List merged branches
git branch --merged

# Delete merged local branches
git branch --merged | grep -v '\*\|main\|develop' | xargs -n 1 git branch -d

# Delete remote tracking branches
git fetch --prune

# Find stale branches (older than 2 weeks)
git for-each-ref --sort=committerdate refs/heads/ --format='%(committerdate:short) %(refname:short)'
```

### Useful Git Aliases

```bash
# Add to ~/.gitconfig
[alias]
    co = checkout
    br = branch
    ci = commit
    st = status
    lg = log --oneline --graph --decorate
    last = log -1 HEAD
    unstage = reset HEAD --
    amend = commit --amend
    pf = push --force-with-lease
    sync = !git fetch origin && git rebase origin/develop
```

---

## Related Documentation

- [Git Hooks](githooks/README.md)
- [Contributing Guide](../CONTRIBUTING.md)
- [Development Guide](DEVELOPMENT.md)

---

<div align="center">

**[Back to Documentation Index](README.md)**

</div>
