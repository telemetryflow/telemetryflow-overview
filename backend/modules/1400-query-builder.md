# Module 1400: Visual Query Builder

- **Module**: `1400-query-builder`
- **Category**: Backend / Business Modules
- **Status**: Production Ready
- **Priority:** 🔥 HIGH - Core Platform Functionality
- **Version**: 3.10.0

---

## Overview

The **Visual Query Builder module** provides a **no-code interface** for creating complex telemetry queries. Features:

- **Drag-and-drop interface**: Build queries visually
- **PromQL support**: Prometheus-style queries
- **Query templates**: Pre-built query patterns
- **Real-time preview**: See results as you build
- **Save and share**: Reusable query definitions

---

## Query Structure

```typescript
export interface QueryDefinition {
  id: string;
  name: string;
  dataType: 'metrics' | 'logs' | 'traces';

  filters: QueryFilter[];
  aggregation?: Aggregation;
  groupBy?: string[];
  timeRange: TimeRange;

  limit?: number;
  orderBy?: OrderBy;
}

export interface QueryFilter {
  field: string;
  operator: 'equals' | 'contains' | 'gt' | 'lt' | 'in';
  value: any;
}
```

---

## API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/v1/queries` | Create query |
| `GET` | `/api/v1/queries` | List saved queries |
| `POST` | `/api/v1/queries/:id/execute` | Execute query |

---

- **Last Updated**: December 12, 2025
- **Maintained By**: DevOpsCorner Indonesia
