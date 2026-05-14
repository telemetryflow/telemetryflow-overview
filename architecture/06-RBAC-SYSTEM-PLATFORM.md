# 5-Tier RBAC System - Platform Documentation

- **Date**: May 14, 2026
- **Status**: ✅ Complete
- **Version**: 4.0 (Complete Platform + Core Integration)

---

## 📋 Overview

TelemetryFlow implements a **5-tier Role-Based Access Control (RBAC) system** with hierarchical permissions and organizational scoping. This document provides comprehensive visualization and comparison of all roles, permissions, and access patterns.

**This document combines:**

- ✅ Platform-level RBAC architecture and visualization
- ✅ Core implementation details from backend modules
- ✅ Complete permission matrices and comparison tables
- ✅ Security features and multi-tenancy isolation
- ✅ Implementation notes and best practices

---

## 🎯 Role Hierarchy

### Visual Hierarchy

```mermaid
graph TD
    A[Tier 1: Super Administrator] --> B[Tier 2: Administrator]
    B --> C[Tier 3: Developer]
    C --> D[Tier 4: Viewer]
    C --> E[Tier 5: Demo]

    A1[Global Scope<br/>All Organizations<br/>All Regions] -.-> A
    B1[Organization Scope<br/>Single Organization<br/>Multiple Regions] -.-> B
    C1[Organization Scope<br/>Single Organization] -.-> C
    D1[Organization Scope<br/>Read-Only] -.-> D
    E1[Demo Org Only<br/>Auto-cleanup 6h] -.-> E

    style A fill:#ff6b6b,stroke:#c92a2a,stroke-width:3px,color:#fff
    style B fill:#fa5252,stroke:#e03131,stroke-width:2px,color:#fff
    style C fill:#ffd43b,stroke:#fab005,stroke-width:2px,color:#000
    style D fill:#51cf66,stroke:#37b24d,stroke-width:2px,color:#fff
    style E fill:#74c0fc,stroke:#339af0,stroke-width:2px,color:#fff
```

### Permission Flow Diagram

```mermaid
flowchart LR
    subgraph Global["🌍 Global (Super Admin)"]
        G1[All Organizations]
        G2[All Regions]
        G3[Platform Management]
        G4[System Admin]
    end

    subgraph Org["🏢 Organization (Admin/Dev/Viewer)"]
        O1[Single Organization]
        O2[Multiple Workspaces]
        O3[Multiple Tenants]
        O4[User Management]
    end

    subgraph Demo["🔬 Demo Environment (Demo)"]
        D1[Demo Organization]
        D2[Demo Workspace]
        D3[Demo Tenant]
        D4[Auto-cleanup 6h]
    end

    Global --> Org
    Org --> Demo

    style Global fill:#ff6b6b,stroke:#c92a2a,stroke-width:2px,color:#fff
    style Org fill:#ffd43b,stroke:#fab005,stroke-width:2px,color:#000
    style Demo fill:#74c0fc,stroke:#339af0,stroke-width:2px,color:#fff
```

### Detailed Hierarchy Diagram

```mermaid
flowchart TD
    T1["<b>Tier 1: Super Administrator</b><br/>🌍 Scope: All Organizations, All Regions<br/>🔑 Permissions: 60+ (100%)<br/>🎯 Use: Platform management, System maintenance"]

    T2["<b>Tier 2: Administrator</b><br/>🏢 Scope: Single Organization, Multiple Regions<br/>🔑 Permissions: 55+ (92%)<br/>🎯 Use: Organization admin, Team leads"]

    T3["<b>Tier 3: Developer</b><br/>💻 Scope: Single Organization<br/>🔑 Permissions: 40+ (67%)<br/>🎯 Use: Developers, DevOps engineers, QA"]

    T4["<b>Tier 4: Viewer</b><br/>👁️ Scope: Organization<br/>🔑 Permissions: 17 (28%)<br/>🎯 Use: Read-only access"]

    T5["<b>Tier 5: Demo</b><br/>🔬 Scope: Demo Org ONLY<br/>🔑 Permissions: 40+ (67%)<br/>🎯 Use: Trials, demos<br/>⏰ Auto-cleanup: 6 hours"]

    T1 --> T2
    T2 --> T3
    T3 --> T4
    T3 --> T5

    style T1 fill:#ff6b6b,stroke:#c92a2a,stroke-width:3px,color:#fff
    style T2 fill:#fa5252,stroke:#e03131,stroke-width:2px,color:#fff
    style T3 fill:#ffd43b,stroke:#fab005,stroke-width:2px,color:#000
    style T4 fill:#51cf66,stroke:#37b24d,stroke-width:2px,color:#fff
    style T5 fill:#74c0fc,stroke:#339af0,stroke-width:2px,color:#fff
```

---

## 🔐 Role Details

### Tier 1: Super Administrator

**Scope**: 🌍 Global (all organizations, regions, workspaces, tenants)

**Description**: Can manage all the SaaS Platform across all organizations and regions

**Permission Count**: **60+** (100% of all permissions)

#### Permissions Breakdown

| Category          | Permissions                            | Description                                    |
| ----------------- | -------------------------------------- | ---------------------------------------------- |
| **Platform**      | platform:\*                            | Full platform management                       |
| **IAM**           | iam:_, users:_, roles:_, permissions:_ | All identity and access operations             |
| **Organizations** | organizations:\*                       | All organization operations                    |
| **Regions**       | regions:\*                             | All region operations                          |
| **Workspaces**    | workspaces:\*                          | All workspace operations                       |
| **Tenants**       | tenants:\*                             | All tenant operations                          |
| **Telemetry**     | metrics:_, logs:_, traces:\*           | All observability operations                   |
| **Dashboards**    | dashboards:\*                          | All dashboard operations                       |
| **Alerts**        | alerts:_, alert-rule-groups:_          | All alerting operations                        |
| **Monitoring**    | monitoring:\*                          | All monitoring operations                      |
| **Agents**        | agents:\*                              | All agent operations                           |
| **Uptime**        | uptime:\*                              | All uptime monitoring operations               |
| **DB Monitoring** | monitoring:db                          | Database Monitoring with QAN (9 DB collectors) |
| **Audit**         | audit-logs:read, audit-logs:export     | Audit log access                               |
| **System**        | system:\*                              | System administration                          |

#### Use Cases

```mermaid
mindmap
  root((Super Admin))
    Platform Management
      Multi-organization oversight
      System configuration
      Global settings
    DevOps Operations
      Infrastructure management
      Deployment pipelines
      Resource provisioning
    Compliance & Security
      Audit log review
      Security policies
      Access control setup
    System Maintenance
      Database management
      Backup & restore
      Performance tuning
```

**Typical Users**:

- Platform administrators
- DevOps team managing infrastructure
- System maintenance engineers
- Security administrators

---

### Tier 2: Administrator

**Scope**: 🏢 Organization-scoped (single organization, multiple regions)

**Description**: Can manage all permissions within their organization across multiple regions

**Permission Count**: **55+** (92% of all permissions)

#### Permissions Breakdown

| Category          | Permissions                              | Description                                    |
| ----------------- | ---------------------------------------- | ---------------------------------------------- |
| **Platform**      | ❌ None                                  | No platform management                         |
| **Organizations** | organizations:read, organizations:update | Read/Update only (no create/delete)            |
| **Users**         | users:\*                                 | Full user CRUD within organization             |
| **Roles**         | roles:\*                                 | Full role CRUD within organization             |
| **Permissions**   | permissions:read                         | Read-only permission viewing                   |
| **Tenants**       | tenants:\*                               | Full tenant CRUD                               |
| **Workspaces**    | workspaces:\*                            | Full workspace CRUD                            |
| **Regions**       | regions:read                             | Read-only region access                        |
| **Telemetry**     | metrics:_, logs:_, traces:\*             | All observability operations                   |
| **Dashboards**    | dashboards:\*                            | All dashboard operations                       |
| **Alerts**        | alerts:_, alert-rule-groups:_            | All alerting operations                        |
| **Monitoring**    | monitoring:\*                            | All monitoring operations                      |
| **Agents**        | agents:\*                                | All agent operations                           |
| **Uptime**        | uptime:\*                                | All uptime operations                          |
| **DB Monitoring** | monitoring:db                            | Database Monitoring with QAN (9 DB collectors) |
| **Audit**         | audit-logs:read, audit-logs:export       | Audit log read/export                          |
| **System**        | ❌ None                                  | No system administration                       |

#### Capabilities vs Restrictions

```mermaid
pie title Administrator Capabilities
    "Full Access (IAM, Telemetry, Monitoring)" : 85
    "Limited Access (Org, Regions)" : 10
    "No Access (Platform, System)" : 5
```

**Typical Users**:

- Organization administrators
- Team leads
- Department managers
- Workspace owners

---

### Tier 3: Developer

**Scope**: 💻 Organization-scoped (single organization)

**Description**: Can create and update resources within their organization, but cannot delete

**Permission Count**: **40+** (67% of all permissions)

#### Permissions Breakdown

| Category          | Permissions                                                                | Description           |
| ----------------- | -------------------------------------------------------------------------- | --------------------- |
| **Organizations** | organizations:read                                                         | Read-only             |
| **Users**         | users:create, users:read, users:update                                     | No delete             |
| **Roles**         | roles:read                                                                 | Read-only             |
| **Permissions**   | permissions:read                                                           | Read-only             |
| **Tenants**       | tenants:create, tenants:read, tenants:update                               | No delete             |
| **Workspaces**    | workspaces:create, workspaces:read, workspaces:update                      | No delete             |
| **Regions**       | regions:read                                                               | Read-only             |
| **Metrics**       | metrics:read, metrics:write                                                | No delete             |
| **Logs**          | logs:read, logs:write                                                      | No delete             |
| **Traces**        | traces:read, traces:write                                                  | No delete             |
| **Dashboards**    | dashboards:create, dashboards:read, dashboards:update                      | No delete             |
| **Alerts**        | alerts:create, alerts:read, alerts:update                                  | No delete/acknowledge |
| **Alert Rules**   | alert-rule-groups:create, alert-rule-groups:read, alert-rule-groups:update | No delete             |
| **Agents**        | agents:create, agents:read, agents:update, agents:register                 | No delete             |
| **Uptime**        | uptime:create, uptime:read, uptime:update, uptime:check                    | No delete             |
| **Audit**         | audit-logs:read                                                            | Read-only             |

#### Developer Workflow

```mermaid
stateDiagram-v2
    [*] --> Create: Create Resources
    Create --> Read: Monitor & Debug
    Read --> Update: Modify Configuration
    Update --> Read: Verify Changes
    Read --> [*]: Task Complete

    Create --> Blocked: ❌ Cannot Delete
    Update --> Blocked: ❌ Cannot Manage Users
    Blocked --> RequestAdmin: Request Admin Help
    RequestAdmin --> [*]
```

**Typical Users**:

- Software developers
- DevOps engineers
- QA engineers
- Site reliability engineers (SRE)

---

### Tier 4: Viewer

**Scope**: 👁️ Organization-scoped (single organization)

**Description**: Read-only access to resources within their organization

**Permission Count**: **17** (28% of all permissions)

#### Permissions Breakdown

| Category                       | Permissions               | Description                 |
| ------------------------------ | ------------------------- | --------------------------- |
| **All Resources**              | \*:read                   | Read-only for all resources |
| **Uptime**                     | uptime:read, uptime:check | Can check uptime status     |
| **Write/Create/Update/Delete** | ❌ None                   | No modification allowed     |

#### Read-Only Access Pattern

```mermaid
graph LR
    A[Viewer Login] --> B{Access Request}
    B -->|Read Operations| C[✅ Allowed]
    B -->|Write Operations| D[❌ Denied]
    B -->|Delete Operations| E[❌ Denied]
    B -->|Create Operations| F[❌ Denied]

    C --> G[View Dashboards]
    C --> H[Query Metrics]
    C --> I[Read Logs]
    C --> J[View Traces]

    style C fill:#51cf66,stroke:#37b24d
    style D fill:#ff6b6b,stroke:#c92a2a
    style E fill:#ff6b6b,stroke:#c92a2a
    style F fill:#ff6b6b,stroke:#c92a2a
```

**Typical Users**:

- Business analysts
- Stakeholders
- External auditors
- Read-only monitoring users
- Customer support (viewing only)

---

### Tier 5: Demo (NEW)

**Scope**: 🔬 Demo Organization ONLY (org-demo, ws-demo, tn-demo)

**Description**: Developer access limited to Demo Organization, Demo Workspace, and Demo Tenant only

**Permission Count**: **40+** (67% - same as Developer)

**Auto-Cleanup**: ⏰ Every 6 hours

#### Permissions Breakdown

| Category                      | Permissions        | Restrictions                  |
| ----------------------------- | ------------------ | ----------------------------- |
| **All Developer Permissions** | Same as Tier 3     | 🔒 Demo Org ONLY              |
| **Organization Access**       | organizations:read | 🔒 `org-demo` only            |
| **Workspace Access**          | workspaces:\*      | 🔒 `ws-demo` only             |
| **Tenant Access**             | tenants:\*         | 🔒 `tn-demo` only             |
| **Data Retention**            | All operations     | ⏰ Auto-deleted every 6 hours |
| **Production Access**         | ❌ None            | 🔒 Cannot access production   |

#### Demo Environment Isolation

```mermaid
graph TD
    subgraph Production["🔒 Production Environment"]
        P1[Org: TelemetryFlow]
        P2[Workspace: Production]
        P3[Tenant: Prod-001]
    end

    subgraph Demo["🔬 Demo Environment"]
        D1[Org: org-demo]
        D2[Workspace: ws-demo]
        D3[Tenant: tn-demo]
        D4[Auto-cleanup: 6h]
    end

    Demo_User[Demo User] -->|✅ Allowed| Demo
    Demo_User -->|❌ Denied| Production

    style Production fill:#ff6b6b,stroke:#c92a2a,stroke-width:2px
    style Demo fill:#74c0fc,stroke:#339af0,stroke-width:2px
```

#### Demo Data Lifecycle

```mermaid
gantt
    title Demo Environment Data Lifecycle (6-hour cycle)
    dateFormat HH:mm
    axisFormat %H:%M

    section Data Phase
    Demo data created    :active, data, 00:00, 06:00
    User experiments     :active, exp, 00:30, 05:30

    section Cleanup Phase
    Auto-cleanup         :crit, cleanup, 06:00, 06:01

    section Reset Phase
    Fresh seed data      :done, seed, 06:01, 06:05
    Ready for demo       :milestone, ready, 06:05, 0m
```

**Typical Users**:

- Product demonstration accounts
- Trial users (free tier)
- Training environments
- Feature testing
- Customer evaluations
- Proof-of-concept (POC) users

---

## 📊 Permission Comparison Matrices

### Complete Permission Matrix

| Permission Category       | Super Admin    | Administrator   | Developer                      | Viewer          | Demo                           |
| ------------------------- | -------------- | --------------- | ------------------------------ | --------------- | ------------------------------ |
| **Scope**                 | 🌍 Global      | 🏢 Organization | 💻 Organization                | 👁️ Organization | 🔬 Demo Org Only               |
| **Permission Count**      | 60+ (100%)     | 55+ (92%)       | 40+ (67%)                      | 17 (28%)        | 40+ (67%)                      |
| **Platform Management**   | ✅ Full        | ❌ None         | ❌ None                        | ❌ None         | ❌ None                        |
| **System Administration** | ✅ Full        | ❌ None         | ❌ None                        | ❌ None         | ❌ None                        |
| **Organization CRUD**     | ✅ Full        | 📖 Read/Update  | 📖 Read                        | 📖 Read         | 📖 Read (Demo only)            |
| **User Management**       | ✅ Full        | ✅ Full         | 📝 Create/Read/Update          | 📖 Read         | 📝 Create/Read/Update          |
| **Role Management**       | ✅ Full        | ✅ Full         | 📖 Read                        | 📖 Read         | 📖 Read                        |
| **Permission Mgmt**       | ✅ Full        | 📖 Read         | 📖 Read                        | 📖 Read         | 📖 Read                        |
| **Tenant Management**     | ✅ Full        | ✅ Full         | 📝 Create/Read/Update          | 📖 Read         | 📝 Create/Read/Update          |
| **Workspace Mgmt**        | ✅ Full        | ✅ Full         | 📝 Create/Read/Update          | 📖 Read         | 📝 Create/Read/Update          |
| **Region Management**     | ✅ Full        | 📖 Read         | 📖 Read                        | 📖 Read         | 📖 Read                        |
| **Metrics**               | ✅ Full        | ✅ Full         | 📊 Read/Write                  | 📖 Read         | 📊 Read/Write                  |
| **Logs**                  | ✅ Full        | ✅ Full         | 📊 Read/Write                  | 📖 Read         | 📊 Read/Write                  |
| **Traces**                | ✅ Full        | ✅ Full         | 📊 Read/Write                  | 📖 Read         | 📊 Read/Write                  |
| **Dashboards**            | ✅ Full        | ✅ Full         | 📝 Create/Read/Update          | 📖 Read         | 📝 Create/Read/Update          |
| **Alerts**                | ✅ Full        | ✅ Full         | 📝 Create/Read/Update          | 📖 Read         | 📝 Create/Read/Update          |
| **Alert Rule Groups**     | ✅ Full        | ✅ Full         | 📝 Create/Read/Update          | 📖 Read         | 📝 Create/Read/Update          |
| **Agents**                | ✅ Full        | ✅ Full         | 📝 Create/Read/Update/Register | 📖 Read         | 📝 Create/Read/Update/Register |
| **Uptime Monitoring**     | ✅ Full        | ✅ Full         | 📝 Create/Read/Update/Check    | 📖 Read/Check   | 📝 Create/Read/Update/Check    |
| **Audit Logs**            | ✅ Read/Export | ✅ Read/Export  | 📖 Read                        | 📖 Read         | 📖 Read                        |
| **Delete Operations**     | ✅ Yes         | ✅ Yes          | ❌ No                          | ❌ No           | ❌ No                          |
| **Export Operations**     | ✅ Yes         | ✅ Yes          | ❌ No                          | ❌ No           | ❌ No                          |
| **Data Retention**        | ♾️ Permanent   | ♾️ Permanent    | ♾️ Permanent                   | ♾️ Permanent    | ⏰ 6 hours                     |
| **Multi-Org Access**      | ✅ Yes         | ❌ No           | ❌ No                          | ❌ No           | ❌ No                          |

**Legend**:

- ✅ Full = Full CRUD access
- 📝 Create/Read/Update = No delete
- 📊 Read/Write = No delete
- 📖 Read = Read-only
- ❌ None = No access

### CRUD Operations Matrix

```mermaid
graph TD
    subgraph Operations["CRUD Operations by Role"]
        direction LR
        C[Create] --> R[Read]
        R --> U[Update]
        U --> D[Delete]
    end

    SA[Super Admin] -.->|✅ All| C
    SA -.->|✅ All| R
    SA -.->|✅ All| U
    SA -.->|✅ All| D

    Admin[Administrator] -.->|✅ Most| C
    Admin -.->|✅ All| R
    Admin -.->|✅ Most| U
    Admin -.->|✅ Most| D

    Dev[Developer] -.->|✅ Limited| C
    Dev -.->|✅ All| R
    Dev -.->|✅ Limited| U
    Dev -.->|❌ None| D

    View[Viewer] -.->|❌ None| C
    View -.->|✅ All| R
    View -.->|❌ None| U
    View -.->|❌ None| D

    Demo[Demo] -.->|✅ Limited| C
    Demo -.->|✅ Demo only| R
    Demo -.->|✅ Limited| U
    Demo -.->|❌ None| D

    style SA fill:#ff6b6b,color:#fff
    style Admin fill:#fa5252,color:#fff
    style Dev fill:#ffd43b,color:#000
    style View fill:#51cf66,color:#fff
    style Demo fill:#74c0fc,color:#fff
```

### Scope & Access Comparison

| Aspect                | Super Admin           | Administrator       | Developer        | Viewer           | Demo                   |
| --------------------- | --------------------- | ------------------- | ---------------- | ---------------- | ---------------------- |
| **Geographic Scope**  | 🌍 All Regions        | 🌍 Multiple Regions | 🗺️ Single Region | 🗺️ Single Region | 📍 Demo Region Only    |
| **Org Access**        | 🏢 All Organizations  | 🏢 Single Org       | 🏢 Single Org    | 🏢 Single Org    | 🏢 Demo Org ONLY       |
| **Workspace Access**  | 📁 All Workspaces     | 📁 All in Org       | 📁 All in Org    | 📁 All in Org    | 📁 Demo Workspace ONLY |
| **Tenant Access**     | 🏷️ All Tenants        | 🏷️ All in Org       | 🏷️ All in Org    | 🏷️ All in Org    | 🏷️ Demo Tenant ONLY    |
| **Cross-Org Access**  | ✅ Yes                | ❌ No               | ❌ No            | ❌ No            | ❌ No                  |
| **Production Access** | ✅ Yes                | ✅ Yes              | ✅ Yes           | ✅ Yes           | ❌ No (Demo only)      |
| **Multi-Tenancy**     | ✅ All Tenants        | ✅ Org Tenants      | ✅ Org Tenants   | ✅ Org Tenants   | 🔒 Demo Tenant         |
| **Data Isolation**    | 🔓 None (Full access) | 🔒 Org-level        | 🔒 Org-level     | 🔒 Org-level     | 🔒🔒 Demo-level        |

### Permission Count Breakdown

```mermaid
pie title Permission Distribution Across Roles
    "Super Admin (60+)" : 60
    "Administrator (55+)" : 55
    "Developer (40+)" : 40
    "Demo (40+)" : 40
    "Viewer (17)" : 17
```

### Access Level Comparison

| Access Level          | Super Admin | Administrator | Developer  | Viewer     | Demo       |
| --------------------- | ----------- | ------------- | ---------- | ---------- | ---------- |
| **Global Platform**   | ⭐⭐⭐⭐⭐  | ⚫⚫⚫⚫⚫    | ⚫⚫⚫⚫⚫ | ⚫⚫⚫⚫⚫ | ⚫⚫⚫⚫⚫ |
| **Organization Mgmt** | ⭐⭐⭐⭐⭐  | ⭐⭐⭐⭐⚫    | ⭐⚫⚫⚫⚫ | ⭐⚫⚫⚫⚫ | ⭐⚫⚫⚫⚫ |
| **User Management**   | ⭐⭐⭐⭐⭐  | ⭐⭐⭐⭐⭐    | ⭐⭐⭐⚫⚫ | ⭐⚫⚫⚫⚫ | ⭐⭐⭐⚫⚫ |
| **Resource CRUD**     | ⭐⭐⭐⭐⭐  | ⭐⭐⭐⭐⭐    | ⭐⭐⭐⚫⚫ | ⭐⚫⚫⚫⚫ | ⭐⭐⭐⚫⚫ |
| **Telemetry Write**   | ⭐⭐⭐⭐⭐  | ⭐⭐⭐⭐⭐    | ⭐⭐⭐⭐⚫ | ⚫⚫⚫⚫⚫ | ⭐⭐⭐⭐⚫ |
| **Telemetry Read**    | ⭐⭐⭐⭐⭐  | ⭐⭐⭐⭐⭐    | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Delete Operations** | ⭐⭐⭐⭐⭐  | ⭐⭐⭐⭐⭐    | ⚫⚫⚫⚫⚫ | ⚫⚫⚫⚫⚫ | ⚫⚫⚫⚫⚫ |
| **System Admin**      | ⭐⭐⭐⭐⭐  | ⚫⚫⚫⚫⚫    | ⚫⚫⚫⚫⚫ | ⚫⚫⚫⚫⚫ | ⚫⚫⚫⚫⚫ |

**Legend**: ⭐ = Has access, ⚫ = No access

---

## 🔄 Role Assignment

### Default Users

| User                        | Email                                          | Password            | Role                | Organization  | Tenant        |
| --------------------------- | ---------------------------------------------- | ------------------- | ------------------- | ------------- | ------------- |
| Super Administrator         | `superadmin.telemetryflow@telemetryflow.id`    | `SuperAdmin@123456` | super_administrator | All           | All           |
| Administrator TelemetryFlow | `administrator.telemetryflow@telemetryflow.id` | `Admin@123456`      | administrator       | TelemetryFlow | TelemetryFlow |
| Developer TelemetryFlow     | `developer.telemetryflow@telemetryflow.id`     | `Developer@123456`  | developer           | TelemetryFlow | TelemetryFlow |
| Viewer TelemetryFlow        | `viewer.telemetryflow@telemetryflow.id`        | `Viewer@123456`     | viewer              | TelemetryFlow | TelemetryFlow |
| Demo TelemetryFlow          | `demo.telemetryflow@telemetryflow.id`          | `Demo@123456`       | demo                | Demo Org      | Demo Tenant   |

### Role Assignment Flow

```mermaid
sequenceDiagram
    participant Admin as Administrator
    participant System as TelemetryFlow System
    participant User as New User
    participant DB as PostgreSQL Database

    Admin->>System: Create user with role
    System->>System: Validate role assignment
    System->>System: Check organization scope
    System->>DB: Save user with role
    DB-->>System: Confirm creation
    System->>User: Send invitation email
    User->>System: Accept & login
    System->>System: Load role permissions
    System->>System: Apply tenant context
    System-->>User: Grant access based on role
```

### Role Migration Path

```mermaid
stateDiagram-v2
    direction LR

    [*] --> Demo: Trial User
    Demo --> Viewer: Upgrade (Read-only)
    Viewer --> Developer: Upgrade (Write access)
    Developer --> Administrator: Promotion (Team lead)
    Administrator --> SuperAdmin: Promotion (Platform admin)

    Demo --> [*]: Trial expires

    note right of Demo
        6-hour data retention
        Demo org only
    end note

    note right of SuperAdmin
        Requires special approval
        Full platform access
    end note
```

---

## 🛡️ Security Features

### Multi-Tenancy Isolation

```mermaid
graph TD
    subgraph Global["🌍 Global Isolation"]
        G1[Region Isolation]
        G2[Organization Isolation]
    end

    subgraph Org["🏢 Organization Isolation"]
        O1[Workspace Isolation]
        O2[Tenant Isolation]
    end

    subgraph Demo["🔬 Demo Isolation"]
        D1[Complete Separation]
        D2[Auto-cleanup]
        D3[No Production Access]
    end

    Global --> Org
    Org --> Demo

    style Global fill:#ff6b6b,stroke:#c92a2a,color:#fff
    style Org fill:#ffd43b,stroke:#fab005,color:#000
    style Demo fill:#74c0fc,stroke:#339af0,color:#fff
```

#### Isolation Layers

| Layer              | Super Admin       | Administrator     | Developer         | Viewer            | Demo              |
| ------------------ | ----------------- | ----------------- | ----------------- | ----------------- | ----------------- |
| **Region**         | ✅ All regions    | ✅ Org regions    | ✅ Org regions    | ✅ Org regions    | 🔒 Demo region    |
| **Organization**   | ✅ All orgs       | 🔒 Single org     | 🔒 Single org     | 🔒 Single org     | 🔒 Demo org ONLY  |
| **Workspace**      | ✅ All workspaces | ✅ Org workspaces | ✅ Org workspaces | ✅ Org workspaces | 🔒 Demo workspace |
| **Tenant**         | ✅ All tenants    | ✅ Org tenants    | ✅ Org tenants    | ✅ Org tenants    | 🔒 Demo tenant    |
| **Data Isolation** | 🔓 None           | 🔒 Org-level      | 🔒 Org-level      | 🔒 Org-level      | 🔒🔒 Demo-level   |

### Demo Environment Protection

```mermaid
timeline
    title Demo Environment Cleanup Cycle
    section Hour 0
        Demo user creates data : Dashboards : Metrics : Alerts
    section Hour 3
        Data accumulates : More experiments : Testing features
    section Hour 6
        Auto-cleanup triggered : All data deleted : Fresh seed loaded
    section Hour 6+
        Environment ready : New demo session : Repeat cycle
```

#### Demo Protection Mechanisms

| Protection               | Status    | Description                            |
| ------------------------ | --------- | -------------------------------------- |
| **Data Isolation**       | ✅ Active | Cannot access production organizations |
| **Workspace Lock**       | ✅ Active | Cannot access production workspaces    |
| **Tenant Lock**          | ✅ Active | Cannot access production tenants       |
| **Auto-cleanup**         | ✅ Active | Data deleted every 6 hours             |
| **Separate Domain**      | ✅ Active | `demo.telemetryflow.id`                |
| **Read-only Production** | ✅ Active | No read access to production           |
| **Rate Limiting**        | ✅ Active | API rate limits enforced               |
| **Resource Quotas**      | ✅ Active | Limited metrics, logs, traces          |

### Permission Enforcement Flow

```mermaid
flowchart TD
    A[User Request] --> B{Authentication}
    B -->|Valid JWT| C{Load User Role}
    B -->|Invalid| X[❌ 401 Unauthorized]

    C --> D{Load Permissions}
    D --> E{Check Permission}

    E -->|Has Permission| F{Check Tenant Context}
    E -->|No Permission| Y[❌ 403 Forbidden]

    F -->|Valid Tenant| G{Check Organization}
    F -->|Invalid Tenant| Z[❌ 403 Forbidden - Wrong Tenant]

    G -->|Valid Org| H{Special: Demo Role?}
    G -->|Invalid Org| W[❌ 403 Forbidden - Wrong Org]

    H -->|Yes, Demo Role| I{Is Demo Org?}
    H -->|No, Regular Role| J[✅ Allow Access]

    I -->|Yes| J
    I -->|No| V[❌ 403 Forbidden - Demo Only]

    J --> K[Execute Request]
    K --> L[Log Audit Event]
    L --> M[Return Response]

    style X fill:#ff6b6b,stroke:#c92a2a,color:#fff
    style Y fill:#ff6b6b,stroke:#c92a2a,color:#fff
    style Z fill:#ff6b6b,stroke:#c92a2a,color:#fff
    style W fill:#ff6b6b,stroke:#c92a2a,color:#fff
    style V fill:#ff6b6b,stroke:#c92a2a,color:#fff
    style J fill:#51cf66,stroke:#37b24d,color:#fff
    style M fill:#51cf66,stroke:#37b24d,color:#fff
```

### Audit Logging

```mermaid
graph LR
    A[User Action] --> B[Permission Check]
    B --> C[Action Executed]
    C --> D[Audit Log Created]

    D --> E[Log Details]
    E --> E1[User ID]
    E --> E2[Role]
    E --> E3[Action]
    E --> E4[Resource]
    E --> E5[Timestamp]
    E --> E6[Organization]
    E --> E7[Tenant]
    E --> E8[Success/Failure]

    style D fill:#ffd43b,stroke:#fab005,color:#000
```

---

## 📝 Implementation Notes

### Architecture Overview

```mermaid
graph TD
    subgraph Backend["Backend (NestJS)"]
        A[Auth Module] --> B[IAM Module]
        B --> C[Role Service]
        C --> D[Permission Service]
        D --> E[Guards]
    end

    subgraph Database["PostgreSQL"]
        F[users Table]
        G[roles Table]
        H[permissions Table]
        I[role_permissions Table]
    end

    subgraph Seeding["Seed Data"]
        J[roles.seed.ts]
        K[users.seed.ts]
        L[demo-cleanup.seed.ts]
    end

    E --> F
    C --> G
    D --> H
    D --> I

    J --> G
    K --> F
    L --> F

    style Backend fill:#ffd43b,stroke:#fab005,color:#000
    style Database fill:#74c0fc,stroke:#339af0,color:#fff
    style Seeding fill:#51cf66,stroke:#37b24d,color:#fff
```

### Role Seeding Process

```mermaid
sequenceDiagram
    participant Seed as Seed Script
    participant RoleService as Role Service
    participant PermService as Permission Service
    participant DB as PostgreSQL

    Seed->>RoleService: Create Super Admin role
    RoleService->>PermService: Get all 60+ permissions
    PermService-->>RoleService: Return permissions
    RoleService->>DB: Save role with permissions

    Seed->>RoleService: Create Administrator role
    RoleService->>PermService: Get 55+ permissions (no platform/system)
    PermService-->>RoleService: Return permissions
    RoleService->>DB: Save role with permissions

    Seed->>RoleService: Create Developer role
    RoleService->>PermService: Get 40+ permissions (no delete)
    PermService-->>RoleService: Return permissions
    RoleService->>DB: Save role with permissions

    Seed->>RoleService: Create Viewer role
    RoleService->>PermService: Get 17 read permissions
    PermService-->>RoleService: Return permissions
    RoleService->>DB: Save role with permissions

    Seed->>RoleService: Create Demo role
    RoleService->>PermService: Get 40+ permissions (same as Developer)
    PermService-->>RoleService: Return permissions
    RoleService->>DB: Save role with demo restrictions
```

### User Assignment Process

```mermaid
flowchart TD
    A[Start: Create User] --> B{Select Role}

    B -->|Super Admin| C[Assign Global Scope]
    B -->|Administrator| D[Assign Organization Scope]
    B -->|Developer| E[Assign Organization Scope]
    B -->|Viewer| F[Assign Organization Scope]
    B -->|Demo| G[Assign Demo Org Scope]

    C --> H[Load 60+ permissions]
    D --> I[Load 55+ permissions]
    E --> J[Load 40+ permissions]
    F --> K[Load 17 permissions]
    G --> L[Load 40+ permissions + Demo restrictions]

    H --> M[Save User]
    I --> M
    J --> M
    K --> M
    L --> M

    M --> N[User Ready]
```

### Demo Cleanup Process

```mermaid
sequenceDiagram
    participant Cron as Cron Job (Every 6h)
    participant Cleanup as Cleanup Service
    participant DB as PostgreSQL
    participant ClickHouse as ClickHouse
    participant Seed as Seed Service

    Cron->>Cleanup: Trigger cleanup
    Cleanup->>DB: Delete demo users (except default)
    Cleanup->>DB: Delete demo dashboards
    Cleanup->>DB: Delete demo alerts
    Cleanup->>ClickHouse: Delete demo metrics
    Cleanup->>ClickHouse: Delete demo logs
    Cleanup->>ClickHouse: Delete demo traces

    Cleanup->>Seed: Re-seed demo data
    Seed->>DB: Create fresh demo user
    Seed->>DB: Create sample dashboards
    Seed->>ClickHouse: Insert sample metrics

    Seed-->>Cleanup: Seeding complete
    Cleanup-->>Cron: Cleanup complete
```

### Permission Expansion

**Before** (Wildcards):

```typescript
permissions: ["metrics:*", "logs:*"];
```

**After** (Explicit):

```typescript
permissions: [
  "metrics:read",
  "metrics:write",
  "metrics:delete",
  "metrics:export",
  "logs:read",
  "logs:write",
  "logs:delete",
  "logs:export",
];
```

**Why?**

- ✅ Explicit permissions for clarity
- ✅ Better security auditing
- ✅ Easier permission debugging
- ✅ No unexpected wildcard expansion

---

## 🎯 Best Practices

### Security Best Practices

```mermaid
mindmap
  root((RBAC Best Practices))
    Principle of Least Privilege
      Assign minimum required permissions
      Start with Viewer, upgrade as needed
      Review permissions quarterly
    Role Separation
      Use appropriate role for each user
      No shared accounts
      Separate prod and demo users
    Demo Isolation
      Keep demo users in demo org only
      Never grant production access
      Monitor demo usage patterns
    Regular Audits
      Review role assignments monthly
      Check audit logs weekly
      Identify permission anomalies
    Password Security
      Enforce 12+ character passwords
      Require uppercase + special chars
      Rotate passwords every 90 days
    MFA Enforcement
      Mandatory for Super Admin
      Mandatory for Administrator
      Optional but recommended for Developer
```

### Role Assignment Decision Tree

```mermaid
flowchart TD
    A[New User Needs Access] --> B{What is their primary role?}

    B -->|Platform Management| C[Super Administrator]
    B -->|Org Management| D[Administrator]
    B -->|Development Work| E[Developer]
    B -->|Monitoring Only| F[Viewer]
    B -->|Trial/Demo| G[Demo]

    C --> H{Do they need global access?}
    H -->|Yes| I[✅ Assign Super Admin]
    H -->|No| J[⚠️ Reconsider - Use Administrator instead]

    D --> K{Manage users and resources?}
    K -->|Yes| L[✅ Assign Administrator]
    K -->|No| M[⚠️ Reconsider - Use Developer instead]

    E --> N{Need to delete resources?}
    N -->|Yes| O[⚠️ Reconsider - Use Administrator instead]
    N -->|No| P[✅ Assign Developer]

    F --> Q{Need to create/modify?}
    Q -->|Yes| R[⚠️ Reconsider - Use Developer instead]
    Q -->|No| S[✅ Assign Viewer]

    G --> T{Production access needed?}
    T -->|Yes| U[⚠️ Reconsider - Use Developer instead]
    T -->|No| V[✅ Assign Demo]
```

### Permission Review Checklist

| Check                       | Frequency | Owner         | Action                         |
| --------------------------- | --------- | ------------- | ------------------------------ |
| **Review role assignments** | Monthly   | Administrator | Remove inactive users          |
| **Audit permission usage**  | Quarterly | Super Admin   | Identify over-privileged users |
| **Check demo cleanup**      | Weekly    | DevOps        | Verify auto-cleanup working    |
| **Review audit logs**       | Weekly    | Security Team | Identify suspicious activity   |
| **Password rotation**       | 90 days   | All users     | Enforce password changes       |
| **MFA status check**        | Monthly   | Administrator | Ensure MFA enabled for admins  |
| **Demo org isolation**      | Daily     | System        | Verify no production access    |

---

## 🔄 Core Implementation Details

### Database Structure

| Database       | Purpose           | Tables                                                                                                       |
| -------------- | ----------------- | ------------------------------------------------------------------------------------------------------------ |
| **PostgreSQL** | IAM data storage  | users, roles, permissions, role_permissions, user_roles, organizations, tenants, workspaces, regions, groups |
| **ClickHouse** | Audit log storage | audit_logs, audit_logs_stats, audit_logs_user_activity                                                       |

### Telemetry Components

| Component              | Version             | Key Features                                                                                                                                           |
| ---------------------- | ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **TFO-Collector**      | v1.2.1 (OCB-native) | OTEL Core v1.58.0, Contrib v0.152.0, 4 custom TFO components (`tfootlp`, `tfo`, `tfoauth`, `tfoidentity`)                                              |
| **TFO-Agent**          | v1.2.0 (Go 1.26)    | OTEL SDK v1.43.0, 15+ collectors, 39+ integrations, eBPF (28 metrics), Docker/cAdvisor (32 per-container metrics), DB Monitoring QAN (9 DB collectors) |
| **Component Registry** | 3 registries        | Graph (260+), Stat Panel (158), DataTable (41) with composable bridge pattern                                                                          |
| **TFQL**               | Query Language      | Unified query interface translating to PromQL, ClickHouse SQL, and Elasticsearch DSL                                                                   |

### Seed Files

| Order | File                                          | Purpose                                  | Records                  |
| ----- | --------------------------------------------- | ---------------------------------------- | ------------------------ |
| 1     | `1704240000001-seed-iam-roles-permissions.ts` | Create 5 roles with explicit permissions | 5 roles, 22+ permissions |
| 2     | `1704240000002-seed-auth-test-users.ts`       | Create test users for each role          | 5 users                  |
| 3     | `1704240000003-seed-groups.ts`                | Create user groups                       | 4 groups                 |

### Permission Enforcement Components

| Component        | Location                          | Purpose                                             |
| ---------------- | --------------------------------- | --------------------------------------------------- |
| **Guards**       | `@RequirePermissions()` decorator | Checks user permissions before controller execution |
| **Decorators**   | `@CurrentUser()`                  | Extracts authenticated user from request            |
| **Repositories** | Auto-scoping queries              | Automatically filters by organization/tenant        |
| **Audit**        | ClickHouse service                | Logs all actions to audit_logs table                |

### Multi-Tenancy Implementation

| Level            | Implementation        | Example                         |
| ---------------- | --------------------- | ------------------------------- |
| **Organization** | Query filter          | `WHERE organizationId = :orgId` |
| **Tenant**       | Repository scoping    | `findByTenant(tenantId)`        |
| **Workspace**    | Resource filtering    | `WHERE workspaceId = :wsId`     |
| **Demo**         | Separate organization | `organizationId = 'org-demo'`   |

### Security Request Flow

```mermaid
graph LR
    A[Request] --> B{Authentication}
    B -->|Valid JWT| C{Authorization}
    B -->|Invalid| X[401 Unauthorized]

    C --> D{Role Check}
    D -->|Has Role| E{Permission Check}
    D -->|No Role| Y[403 Forbidden]

    E -->|Has Permission| F{Scope Check}
    E -->|No Permission| Y

    F -->|Organization Match| G{Tenant Filter}
    F -->|No Match| Y

    G -->|Allowed| H[Execute Query]
    G -->|Denied| Y

    H --> I[Audit Log]
    I --> J[Response]

    style A fill:#e3f2fd
    style B fill:#fff3e0
    style C fill:#fff3e0
    style D fill:#f3e5f5
    style E fill:#f3e5f5
    style F fill:#e8f5e9
    style G fill:#e8f5e9
    style H fill:#e1f5fe
    style I fill:#fce4ec
    style J fill:#e8f5e9
    style X fill:#ffebee
    style Y fill:#ffebee
```

---

## 🚀 Usage & Testing

### Login Credentials

```bash
# Super Administrator
Email: super.administrator@telemetryflow.id
Password: SuperAdmin@123456

# Administrator
Email: admin.telemetryflow@telemetryflow.id
Password: Admin@123456

# Developer
Email: developer.telemetryflow@telemetryflow.id
Password: Developer@123456

# Viewer
Email: viewer.telemetryflow@telemetryflow.id
Password: Viewer@123456

# Demo
Email: demo.telemetryflow@telemetryflow.id
Password: Demo@123456
```

⚠️ **Security Warning:** Change these default passwords immediately in production!

### API Access

All users can access the API at: **http://localhost:3000/api/v2**

Use Swagger UI at **http://localhost:3000/docs** to test different permission levels and see which endpoints are accessible for each role.

### Testing Permissions

1. **Login with different roles** to see permission differences
2. **Try CRUD operations** to verify access control
3. **Check Swagger UI** for available endpoints per role
4. **Review audit logs** to track all actions

### Seeding RBAC System

The 5-tier RBAC system is automatically seeded when running:

```bash
# Seed IAM data only
pnpm db:seed:iam

# Seed all data (PostgreSQL + ClickHouse)
pnpm db:seed

# Run migrations + seeds
pnpm db:migrate:seed

# Full bootstrap (dependencies, Docker, migrations, seeds)
bash scripts/bootstrap.sh --dev
```

---

## 📊 Additional Permission Details

### Multi-Tenancy Isolation Features

| Feature                | Description                         | Implementation                                 |
| ---------------------- | ----------------------------------- | ---------------------------------------------- |
| **Organization-level** | Data isolated by organization       | Query filters: `WHERE organizationId = :orgId` |
| **Tenant-level**       | Query filtering per tenant          | Automatic tenant scoping in repositories       |
| **Workspace-level**    | Resource scoping per workspace      | Workspace-based access control                 |
| **Demo isolation**     | Complete separation from production | Separate organization with auto-cleanup        |

### Demo Environment Protection Details

| Feature                  | Description                         | Status         |
| ------------------------ | ----------------------------------- | -------------- |
| **Auto-cleanup**         | Data deleted every 6 hours          | ✅ Implemented |
| **Production isolation** | Cannot access production orgs       | ✅ Enforced    |
| **Workspace isolation**  | Cannot access production workspaces | ✅ Enforced    |
| **Tenant isolation**     | Cannot access production tenants    | ✅ Enforced    |
| **Separate domain**      | `demo.telemetryflow.id`             | ✅ Configured  |

### Permission Enforcement Layers

| Layer              | Mechanism                         | Description                                  |
| ------------------ | --------------------------------- | -------------------------------------------- |
| **Authentication** | JWT tokens                        | Validates user identity                      |
| **Authorization**  | `@RequirePermissions()` decorator | Checks user permissions before execution     |
| **Scoping**        | Query filters                     | Automatically filters by organization/tenant |
| **Audit**          | ClickHouse audit_logs             | Logs all actions with user context           |
| **Validation**     | Guards & Interceptors             | Validates request data and permissions       |

---

## 📚 Related Documentation

- **IAM Module**: `/backend/src/modules/iam/`
- **Auth Module**: `/backend/src/modules/auth/`
- **Role Seed**: `/backend/src/database/seeds/roles.seed.ts`
- **User Seed**: `/backend/src/database/seeds/users.seed.ts`
- **Demo Cleanup**: `/backend/src/database/seeds/demo-cleanup.seed.ts`
- **Guards**: `/backend/src/modules/iam/guards/`
- **Permissions**: `/backend/src/modules/iam/domain/permissions/`

---

## ✅ Summary

### RBAC System Highlights

```mermaid
pie title Permission Distribution
    "Super Admin (100%)" : 100
    "Administrator (92%)" : 92
    "Developer (67%)" : 67
    "Viewer (28%)" : 28
    "Demo (67% with restrictions)" : 67
```

### Key Features

| Feature                  | Status      | Description                                                        |
| ------------------------ | ----------- | ------------------------------------------------------------------ |
| **5-Tier Hierarchy**     | ✅ Complete | Super Admin → Admin → Developer → Viewer, Demo                     |
| **60+ Permissions**      | ✅ Complete | Granular permission system                                         |
| **Multi-Tenancy**        | ✅ Complete | Organization, Workspace, Tenant isolation                          |
| **Demo Environment**     | ✅ Complete | Isolated demo org with auto-cleanup                                |
| **Audit Logging**        | ✅ Complete | All actions logged for compliance                                  |
| **MFA Support**          | ✅ Complete | Optional MFA for all roles (Argon2id hashed backup codes)          |
| **Auto-Cleanup**         | ✅ Complete | Demo data deleted every 6 hours                                    |
| **Explicit Permissions** | ✅ Complete | No wildcards, all explicit                                         |
| **API Key Auth**         | ✅ Complete | AWS-style dual keys (tfk-_/tfs-_) with Argon2id hashing            |
| **DB Monitoring**        | ✅ Complete | QAN with 9 DB collectors for PostgreSQL, MySQL, MongoDB, etc.      |
| **TFQL**                 | ✅ Complete | Unified query language (PromQL, ClickHouse SQL, Elasticsearch DSL) |

---

- **Document**: 06-RBAC-SYSTEM-PLATFORM.md
- **Version**: 4.0 (Complete Platform + Core Integration)
- **Date**: 2026-05-14
- **Status**: ✅ Complete
