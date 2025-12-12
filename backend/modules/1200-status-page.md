# Module 1200: Status Page

- **Module**: `1200-status-page`
- **Category**: Backend / Business Modules
- **Status**: Production Ready
- **Priority:** 🔥 HIGH - Core Monitoring Functionality
- **Version**: 3.10.0

---

## Overview

The **Status Page module** provides **public-facing status pages** to communicate service health. Features:

- **Public status pages**: Shareable URLs for transparency
- **Monitor aggregation**: Display multiple monitor statuses
- **Incident management**: Create and manage incidents
- **Maintenance windows**: Schedule maintenance notifications
- **Custom branding**: Logo, colors, custom domain

---

## Domain Model

```typescript
// domain/aggregates/StatusPage.ts
export class StatusPage extends AggregateRoot<StatusPageId> {
  constructor(
    id: StatusPageId,
    public name: string,
    public slug: string,
    public description: string,
    public monitors: MonitorId[],
    public isPublic: boolean,
    public customDomain: string | null,
    public branding: StatusPageBranding,
    public organizationId: OrganizationId,
  ) {
    super(id);
  }

  static create(
    name: string,
    slug: string,
    organizationId: OrganizationId,
  ): StatusPage {
    const statusPage = new StatusPage(
      StatusPageId.create(),
      name,
      slug,
      '',
      [],
      true,
      null,
      StatusPageBranding.default(),
      organizationId,
    );

    statusPage.addDomainEvent(new StatusPageCreatedEvent(statusPage));
    return statusPage;
  }

  addMonitor(monitorId: MonitorId): void {
    if (!this.monitors.includes(monitorId)) {
      this.monitors.push(monitorId);
    }
  }

  removeMonitor(monitorId: MonitorId): void {
    this.monitors = this.monitors.filter(m => !m.equals(monitorId));
  }
}
```

---

## Database Schema

```sql
CREATE TABLE status_pages (
  status_page_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

  name VARCHAR(255) NOT NULL,
  slug VARCHAR(100) UNIQUE NOT NULL,
  description TEXT,

  is_public BOOLEAN DEFAULT true,
  custom_domain VARCHAR(255),

  -- Branding (JSON)
  branding JSONB DEFAULT '{}',

  -- Multi-tenancy
  organization_id UUID REFERENCES organizations(organization_id),
  workspace_id UUID REFERENCES workspaces(workspace_id),

  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  deleted_at TIMESTAMP
);

CREATE TABLE status_page_monitors (
  status_page_id UUID REFERENCES status_pages(status_page_id) ON DELETE CASCADE,
  monitor_id UUID REFERENCES monitors(monitor_id) ON DELETE CASCADE,
  display_order INTEGER DEFAULT 0,
  PRIMARY KEY (status_page_id, monitor_id)
);
```

---

## API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/v1/status-pages` | Create status page |
| `GET` | `/api/v1/status-pages` | List status pages |
| `GET` | `/api/v1/status-pages/:id` | Get status page |
| `GET` | `/status/:slug` | Public status page |

---

**Last Updated**: December 12, 2025
**Maintained By**: DevOpsCorner Indonesia
