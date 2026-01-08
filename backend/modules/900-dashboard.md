# Module: 900-dashboard (Dashboard Management)

- **Module**: `900-dashboard`
- **Category**: Backend / Business Modules
- **Status**: Production Ready
- **Priority:** 🔥 HIGH - Core Visualization
- **Version**: 1.1.2-CE

---

## Module Overview

```mermaid
graph TB
    subgraph "900-dashboard Module"
        A[Dashboard Management<br/>Custom & Template-based]
        B[Widget System<br/>Metrics/Logs/Traces]
        C[Template Library<br/>Pre-built Dashboards]
        D[Real-time Updates<br/>WebSocket Support]
    end

    subgraph "Key Features"
        E[✅ Drag & Drop Layout]
        F[✅ 6 Built-in Templates]
        G[✅ Widget Customization]
        H[✅ Favorite Dashboards]
    end

    A --> E
    B --> F
    C --> G
    D --> H

    style A fill:#ff6b6b
    style B fill:#4ecdc4
    style C fill:#45b7d1
    style D fill:#f9ca24
```

**Purpose:** Complete dashboard management system with templates, widgets, and real-time visualizations for metrics, logs, and traces.

**Location:** `backend/src/modules/900-dashboard/`

---

## Architecture

### Module Structure

```mermaid
graph TB
    subgraph "presentation/ - REST API"
        C1[DashboardsController<br/>/api/v2/dashboards]
        C2[DashboardTemplatesController<br/>/api/v2/dashboard-templates]
        C3[WidgetsController<br/>/api/v2/widgets]
    end

    subgraph "application/ - CQRS"
        CMD1[CreateDashboardCommand]
        CMD2[CloneDashboardFromTemplateCommand]
        CMD3[AddWidgetCommand]
        QRY1[ListDashboardsQuery]
        H1[Command/Query Handlers]
    end

    subgraph "domain/ - Business Logic"
        AGG1[Dashboard Aggregate]
        AGG2[DashboardTemplate Aggregate]
        AGG3[Widget Aggregate]
        VO1[Value Objects:<br/>DashboardId, WidgetType]
    end

    subgraph "infrastructure/ - Storage"
        REPO1[DashboardRepository<br/>PostgreSQL]
        REPO2[TemplateRepository<br/>PostgreSQL]
        MAP1[Dashboard Mappers]
    end

    C1 --> CMD1
    C1 --> QRY1
    CMD1 --> AGG1
    QRY1 --> REPO1
    AGG1 --> REPO1

    style C1 fill:#ff6b6b
    style CMD1 fill:#f9ca24
    style AGG1 fill:#4ecdc4
    style REPO1 fill:#6c5ce7
```

---

## Dashboard Creation Flow

### Complete Dashboard Lifecycle

```mermaid
sequenceDiagram
    participant User as Frontend User
    participant API as DashboardsController
    participant Handler as CreateDashboardHandler
    participant Agg as Dashboard Aggregate
    participant Repo as DashboardRepository
    participant Event as EventBus
    participant Cache as CacheService

    User->>API: POST /dashboards<br/>{name, templateId}
    API->>Handler: Execute CreateDashboardCommand

    alt From Template
        Handler->>Repo: Load template by ID
        Repo-->>Handler: Return DashboardTemplate

        Handler->>Agg: Dashboard.createFromTemplate()<br/>Clone widgets & layout
    else Custom Dashboard
        Handler->>Agg: Dashboard.create()<br/>Empty dashboard
    end

    Agg->>Agg: Validate Business Rules<br/>- Name not empty<br/>- Valid workspace
    Agg->>Agg: Publish DashboardCreated event

    Handler->>Repo: Save dashboard<br/>with all widgets
    Handler->>Event: Publish event to bus
    Handler->>Cache: Invalidate dashboard list cache

    Handler-->>API: Return dashboard ID
    API-->>User: 201 {id, name, widgets}

    Event->>Event: Trigger downstream processors<br/>- Audit logging<br/>- Analytics tracking
```

### Widget Management Flow

```mermaid
sequenceDiagram
    participant User as Frontend User
    participant API as DashboardsController
    participant Handler as AddWidgetHandler
    participant Agg as Dashboard Aggregate
    participant Widget as Widget Aggregate
    participant Repo as DashboardRepository

    User->>API: POST /dashboards/:id/widgets<br/>{type, config, position}
    API->>Handler: Execute AddWidgetCommand

    Handler->>Repo: Load dashboard by ID
    Repo-->>Handler: Return Dashboard

    Handler->>Widget: Widget.create()<br/>{type, configuration}
    Widget->>Widget: Validate configuration<br/>- Required fields<br/>- Valid metric/log query

    Handler->>Agg: dashboard.addWidget(widget)
    Agg->>Agg: Validate layout<br/>- No overlapping widgets<br/>- Within grid bounds

    Agg->>Agg: Update layout<br/>Publish WidgetAdded event

    Handler->>Repo: Update dashboard
    Handler-->>API: Return widget details
    API-->>User: 201 {widget}
```

---

## Domain Model

### Dashboard Aggregate

```typescript
// domain/aggregates/Dashboard.aggregate.ts
export class Dashboard extends AggregateRoot {
  private readonly _id: DashboardId;
  private _name: string;
  private _description?: string;
  private readonly _workspaceId: WorkspaceId;
  private readonly _tenantId: TenantId;
  private readonly _createdBy: UserId;
  private _widgets: Widget[] = [];
  private _layout: DashboardLayout;
  private _isFavorite: boolean = false;
  private _templateId?: DashboardTemplateId;

  static create(
    name: string,
    workspaceId: WorkspaceId,
    tenantId: TenantId,
    createdBy: UserId,
    description?: string,
    templateId?: DashboardTemplateId
  ): Dashboard {
    // Business Rule: Name cannot be empty
    if (!name || name.trim() === '') {
      throw new DomainError('Dashboard name cannot be empty');
    }

    const dashboard = new Dashboard({
      id: DashboardId.create(),
      name,
      workspaceId,
      tenantId,
      createdBy,
      description,
      templateId,
      widgets: [],
      layout: DashboardLayout.createDefault(),
      createdAt: new Date(),
      updatedAt: new Date(),
    });

    dashboard.apply(new DashboardCreated(dashboard));
    return dashboard;
  }

  static createFromTemplate(
    template: DashboardTemplate,
    workspaceId: WorkspaceId,
    tenantId: TenantId,
    createdBy: UserId
  ): Dashboard {
    const dashboard = Dashboard.create(
      template.name,
      workspaceId,
      tenantId,
      createdBy,
      template.description,
      template.id
    );

    // Clone all widgets from template
    template.widgets.forEach(templateWidget => {
      const widget = Widget.createFromTemplate(templateWidget);
      dashboard.addWidget(widget);
    });

    dashboard._layout = DashboardLayout.create(template.layout);
    return dashboard;
  }

  addWidget(widget: Widget): void {
    // Business Rule: Widget position must not overlap with existing widgets
    if (this._layout.hasOverlap(widget.position)) {
      throw new DomainError('Widget position overlaps with existing widget');
    }

    // Business Rule: Widget must be within grid bounds
    if (!this._layout.isWithinBounds(widget.position)) {
      throw new DomainError('Widget position is outside grid bounds');
    }

    this._widgets.push(widget);
    this._layout.addWidget(widget.id, widget.position);
    this.apply(new WidgetAdded(this._id, widget.id));
  }

  removeWidget(widgetId: WidgetId): void {
    const widgetIndex = this._widgets.findIndex(w => w.id.equals(widgetId));
    if (widgetIndex === -1) {
      throw new DomainError('Widget not found');
    }

    this._widgets.splice(widgetIndex, 1);
    this._layout.removeWidget(widgetId);
    this.apply(new WidgetRemoved(this._id, widgetId));
  }

  toggleFavorite(): void {
    this._isFavorite = !this._isFavorite;
  }
}
```

### Widget Types

```mermaid
classDiagram
    class Widget {
        -WidgetId id
        -WidgetType type
        -WidgetConfiguration config
        -WidgetPosition position
        +create() Widget$
        +updateConfiguration(config) void
    }

    class WidgetType {
        <<enumeration>>
        TIME_SERIES_CHART
        LOG_STREAM
        TRACE_WATERFALL
        METRIC_VALUE
        GAUGE
        TABLE
        HEATMAP
    }

    class WidgetConfiguration {
        -metricName string
        -query string
        -timeRange string
        -aggregation string
        +validate() boolean
    }

    class WidgetPosition {
        -x number
        -y number
        -width number
        -height number
        +overlaps(other) boolean
    }

    Widget --> WidgetType
    Widget --> WidgetConfiguration
    Widget --> WidgetPosition

    style Widget fill:#4ecdc4
    style WidgetType fill:#ff6b6b
    style WidgetConfiguration fill:#f9ca24
```

---

## Built-in Dashboard Templates

```mermaid
graph TB
    subgraph "6 Built-in Templates"
        T1[System Monitoring<br/>CPU, Memory, Disk]
        T2[Application Performance<br/>Response Time, Throughput]
        T3[Logs Explorer<br/>Log Stream & Filters]
        T4[Infrastructure<br/>Network, Containers]
        T5[Network Monitoring<br/>Traffic, Latency]
        T6[Custom Metrics<br/>User-defined Metrics]
    end

    subgraph "Template Features"
        F1[Pre-configured Widgets]
        F2[Optimized Layout]
        F3[Best Practice Queries]
    end

    T1 --> F1
    T2 --> F2
    T3 --> F3

    style T1 fill:#ff6b6b
    style T2 fill:#4ecdc4
    style T3 fill:#45b7d1
    style T4 fill:#f9ca24
    style T5 fill:#6c5ce7
    style T6 fill:#a29bfe
```

### Template Details

**1. System Monitoring Template**
```json
{
  "name": "System Monitoring",
  "description": "Monitor system-level metrics",
  "widgets": [
    {
      "type": "TIME_SERIES_CHART",
      "config": {
        "title": "CPU Usage",
        "metricName": "system.cpu.utilization",
        "aggregation": "avg",
        "timeRange": "1h"
      },
      "position": {"x": 0, "y": 0, "width": 6, "height": 4}
    },
    {
      "type": "TIME_SERIES_CHART",
      "config": {
        "title": "Memory Usage",
        "metricName": "system.memory.usage",
        "aggregation": "avg",
        "timeRange": "1h"
      },
      "position": {"x": 6, "y": 0, "width": 6, "height": 4}
    },
    {
      "type": "GAUGE",
      "config": {
        "title": "Disk Usage",
        "metricName": "system.disk.usage",
        "unit": "%",
        "thresholds": {"warning": 70, "critical": 90}
      },
      "position": {"x": 0, "y": 4, "width": 4, "height": 3}
    }
  ]
}
```

**2. Application Performance Template**
- Response Time (P50, P95, P99)
- Request Throughput
- Error Rate
- Active Connections

**3. Logs Explorer Template**
- Live Log Stream
- Log Level Distribution
- Error Log Table
- Search & Filter Panel

---

## Database Schema

### Dashboards Table (PostgreSQL)

```sql
CREATE TABLE dashboards (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(255) NOT NULL,
  description TEXT,

  -- Multi-Tenancy
  organization_id UUID,
  workspace_id UUID NOT NULL,
  tenant_id UUID,

  -- Template Reference
  template_id UUID,

  -- Layout Configuration
  layout JSONB NOT NULL DEFAULT '{
    "grid": {"columns": 12, "rowHeight": 30},
    "widgets": []
  }'::jsonb,

  -- User Preferences
  is_favorite BOOLEAN DEFAULT false,
  is_public BOOLEAN DEFAULT false,

  -- Ownership
  created_by UUID NOT NULL,

  -- Timestamps
  created_at TIMESTAMP NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP NOT NULL DEFAULT NOW(),
  last_viewed_at TIMESTAMP,

  CONSTRAINT fk_workspace FOREIGN KEY (workspace_id)
    REFERENCES workspaces(id) ON DELETE CASCADE,
  CONSTRAINT fk_template FOREIGN KEY (template_id)
    REFERENCES dashboard_templates(id) ON DELETE SET NULL,
  CONSTRAINT fk_created_by FOREIGN KEY (created_by)
    REFERENCES users(id) ON DELETE CASCADE
);

-- Indexes
CREATE INDEX idx_dashboards_workspace ON dashboards(workspace_id);
CREATE INDEX idx_dashboards_tenant ON dashboards(tenant_id);
CREATE INDEX idx_dashboards_created_by ON dashboards(created_by);
CREATE INDEX idx_dashboards_is_favorite ON dashboards(is_favorite);
CREATE INDEX idx_dashboards_template ON dashboards(template_id);
```

### Widgets Table (PostgreSQL)

```sql
CREATE TABLE widgets (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  dashboard_id UUID NOT NULL,

  -- Widget Type
  type VARCHAR(50) NOT NULL,  -- TIME_SERIES_CHART, LOG_STREAM, etc.

  -- Widget Configuration
  title VARCHAR(255) NOT NULL,
  configuration JSONB NOT NULL,  -- Widget-specific config

  -- Layout Position
  position JSONB NOT NULL DEFAULT '{
    "x": 0, "y": 0, "width": 6, "height": 4
  }'::jsonb,

  -- Display Settings
  refresh_interval INT DEFAULT 30,  -- seconds
  show_legend BOOLEAN DEFAULT true,

  -- Timestamps
  created_at TIMESTAMP NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP NOT NULL DEFAULT NOW(),

  CONSTRAINT fk_dashboard FOREIGN KEY (dashboard_id)
    REFERENCES dashboards(id) ON DELETE CASCADE
);

-- Indexes
CREATE INDEX idx_widgets_dashboard ON widgets(dashboard_id);
CREATE INDEX idx_widgets_type ON widgets(type);
```

### Dashboard Templates Table (PostgreSQL)

```sql
CREATE TABLE dashboard_templates (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(255) NOT NULL UNIQUE,
  description TEXT,
  category VARCHAR(100),  -- system, application, infrastructure

  -- Template Content
  layout JSONB NOT NULL,
  widgets JSONB NOT NULL,  -- Pre-configured widgets

  -- Metadata
  is_system BOOLEAN DEFAULT false,  -- Built-in templates
  icon VARCHAR(100),
  tags JSONB DEFAULT '[]'::jsonb,

  -- Usage Tracking
  usage_count INT DEFAULT 0,

  -- Timestamps
  created_at TIMESTAMP NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP NOT NULL DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_templates_category ON dashboard_templates(category);
CREATE INDEX idx_templates_is_system ON dashboard_templates(is_system);
```

---

## Widget Configuration Examples

### Time Series Chart Widget

```json
{
  "type": "TIME_SERIES_CHART",
  "title": "HTTP Request Rate",
  "configuration": {
    "metricName": "http_requests_total",
    "aggregation": "rate",
    "interval": "1m",
    "timeRange": "1h",
    "filters": {
      "service_name": "api-gateway",
      "status_code": "200"
    },
    "visualization": {
      "chartType": "line",
      "colors": ["#4ecdc4"],
      "yAxis": {
        "label": "Requests/sec",
        "min": 0
      }
    }
  },
  "position": {
    "x": 0,
    "y": 0,
    "width": 6,
    "height": 4
  },
  "refreshInterval": 30
}
```

### Log Stream Widget

```json
{
  "type": "LOG_STREAM",
  "title": "Error Logs",
  "configuration": {
    "query": "severity:ERROR",
    "limit": 100,
    "timeRange": "15m",
    "autoScroll": true,
    "highlightKeywords": ["error", "exception", "failed"],
    "columns": ["timestamp", "service_name", "message", "trace_id"]
  },
  "position": {
    "x": 0,
    "y": 4,
    "width": 12,
    "height": 6
  },
  "refreshInterval": 10
}
```

### Gauge Widget

```json
{
  "type": "GAUGE",
  "title": "Memory Usage",
  "configuration": {
    "metricName": "system.memory.usage",
    "aggregation": "latest",
    "unit": "%",
    "thresholds": {
      "low": 50,
      "warning": 70,
      "critical": 90
    },
    "colors": {
      "low": "#27ae60",
      "warning": "#f39c12",
      "critical": "#e74c3c"
    }
  },
  "position": {
    "x": 0,
    "y": 0,
    "width": 3,
    "height": 3
  },
  "refreshInterval": 60
}
```

---

## Performance

### Caching Strategy

```mermaid
graph LR
    subgraph "Dashboard Load"
        A[Request Dashboard]
        B[Check Cache]
        C[Load from DB]
        D[Transform to DTO]
        E[Cache Result]
    end

    subgraph "Widget Data"
        F[Widget Queries]
        G[ClickHouse]
        H[Aggregate Results]
    end

    A --> B
    B -->|Cache Miss| C
    C --> D
    D --> E

    B -->|Cache Hit| D

    D --> F
    F --> G
    G --> H

    style B fill:#4ecdc4
    style E fill:#f9ca24
```

| Operation | Cache TTL | Performance |
|-----------|-----------|-------------|
| **Dashboard List** | 5 minutes | 10ms (cached) |
| **Dashboard Details** | 1 minute | 50ms (cached) |
| **Widget Data** | 30 seconds | 100-500ms |
| **Template List** | 1 hour | 5ms (cached) |

---

## API Examples

### Create Dashboard from Template

```bash
curl -X POST http://localhost:3000/api/v2/dashboards \
  -H "Authorization: Bearer <jwt-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "My System Dashboard",
    "templateId": "template-system-monitoring-uuid",
    "workspaceId": "workspace-uuid",
    "tenantId": "tenant-uuid"
  }'
```

### Add Widget to Dashboard

```bash
curl -X POST http://localhost:3000/api/v2/dashboards/:dashboardId/widgets \
  -H "Authorization: Bearer <jwt-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "type": "TIME_SERIES_CHART",
    "title": "Custom CPU Chart",
    "configuration": {
      "metricName": "system.cpu.utilization",
      "aggregation": "avg",
      "timeRange": "1h"
    },
    "position": {
      "x": 6,
      "y": 4,
      "width": 6,
      "height": 4
    }
  }'
```

### List Dashboards

```bash
curl -X GET "http://localhost:3000/api/v2/dashboards?workspaceId=workspace-uuid&favorite=true" \
  -H "Authorization: Bearer <jwt-token>"
```

---

## Related Modules

```mermaid
graph TB
    D[900-dashboard<br/>Dashboard Management]

    D -->|Queries| T[400-telemetry<br/>Metrics/Logs/Traces]
    D -->|Uses| C[shared/cache<br/>Dashboard Cache]
    D -->|Integrates| A[600-alerts<br/>Alert Widgets]
    D -->|Logs to| E[800-audit<br/>View Tracking]

    style D fill:#ff6b6b
    style T fill:#4ecdc4
    style C fill:#f9ca24
```

---

## Testing

### Unit Tests
- `Dashboard.aggregate.spec.ts` - Dashboard logic
- `Widget.aggregate.spec.ts` - Widget validation
- `DashboardLayout.vo.spec.ts` - Layout calculations

### Integration Tests
- `dashboard-creation.spec.ts` - Template cloning
- `widget-management.spec.ts` - Widget CRUD

### E2E Tests
- `dashboard-lifecycle.e2e.spec.ts` - Complete flow

---

- **File Location:** `./backend/modules/900-dashboard.md`
- **Maintained By:** DevOpsCorner Indonesia
