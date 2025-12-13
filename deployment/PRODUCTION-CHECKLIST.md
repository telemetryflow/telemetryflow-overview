# Production Deployment Checklist

- **Version**: 1.0.0-CE
- **Target**: Production Environment
- **Status**: ✅ Ready for Production

---

## Table of Contents

1. [Pre-Deployment Checklist](#pre-deployment-checklist)
2. [Infrastructure Setup](#infrastructure-setup)
3. [Security Hardening](#security-hardening)
4. [Database Setup](#database-setup)
5. [Application Configuration](#application-configuration)
6. [Monitoring & Observability](#monitoring--observability)
7. [Backup & Disaster Recovery](#backup--disaster-recovery)
8. [Performance Optimization](#performance-optimization)
9. [Deployment Procedures](#deployment-procedures)
10. [Post-Deployment Validation](#post-deployment-validation)
11. [Rollback Procedures](#rollback-procedures)
12. [Maintenance & Operations](#maintenance--operations)

---

## Pre-Deployment Checklist

### Documentation Review

- [ ] Review [DOCKER-COMPOSE.md](./DOCKER-COMPOSE.md) for deployment architecture
- [ ] Review [01-SYSTEM-ARCHITECTURE.md](../architecture/01-SYSTEM-ARCHITECTURE.md) for system design
- [ ] Review [04-SECURITY.md](../architecture/04-SECURITY.md) for security requirements
- [ ] Review [DATABASE-SCHEMA.md](../shared/DATABASE-SCHEMA.md) for database structure
- [ ] Review [CONFIGURATION.md](./CONFIGURATION.md) for environment variables

### Team Preparation

- [ ] DevOps team trained on deployment procedures
- [ ] Backend team familiar with application architecture
- [ ] Database team prepared for migration execution
- [ ] Security team reviewed security configurations
- [ ] Support team prepared for incident response

### Resource Planning

- [ ] Infrastructure capacity planned (CPU, RAM, Storage)
- [ ] Network bandwidth requirements calculated
- [ ] Database sizing completed (PostgreSQL + ClickHouse)
- [ ] Redis memory allocation planned
- [ ] Backup storage allocated

### Testing Completed

- [ ] All unit tests passing (Backend + Frontend)
- [ ] Integration tests passing
- [ ] E2E tests passing
- [ ] Load testing completed
- [ ] Security penetration testing completed
- [ ] UAT (User Acceptance Testing) completed

---

## Infrastructure Setup

### Server Requirements

**Minimum Production Specifications:**

```yaml
Backend Server:
  CPU: 4 cores (8 recommended)
  RAM: 8GB (16GB recommended)
  Storage: 100GB SSD
  Network: 1Gbps

PostgreSQL Server:
  CPU: 4 cores
  RAM: 16GB
  Storage: 500GB SSD (RAID 10)
  IOPS: 3000+

ClickHouse Server:
  CPU: 8 cores
  RAM: 32GB
  Storage: 1TB NVMe SSD
  IOPS: 10000+

Redis Server:
  CPU: 2 cores
  RAM: 8GB
  Storage: 50GB SSD

NATS Server:
  CPU: 2 cores
  RAM: 4GB
  Storage: 20GB SSD
```

### Infrastructure Checklist

- [ ] Servers provisioned and accessible
- [ ] SSH keys configured (no password authentication)
- [ ] Firewall rules configured
- [ ] Load balancer configured (if applicable)
- [ ] SSL/TLS certificates obtained and installed
- [ ] DNS records configured
- [ ] CDN configured (if applicable)
- [ ] Backup storage configured

### Network Configuration

- [ ] Private network for database communication
- [ ] Public network for API access
- [ ] VPN access for administrative tasks
- [ ] Network segmentation implemented
- [ ] DDoS protection enabled
- [ ] Rate limiting configured

### Port Configuration

**Required Open Ports:**

```yaml
External (Public):
  443: HTTPS API/Frontend
  4317: OTLP gRPC (telemetry ingestion)

Internal (Private Network Only):
  3100: Backend API
  3101: Frontend (Nginx)
  5432: PostgreSQL
  8123: ClickHouse HTTP
  9000: ClickHouse Native
  6379: Redis
  4222: NATS Client
  8222: NATS Monitoring
  9090: Prometheus (optional)
  3000: Grafana (optional)
```

---

## Security Hardening

### System Security

- [ ] Disable root SSH login
- [ ] Configure SSH key-based authentication only
- [ ] Configure fail2ban for SSH protection
- [ ] Enable automatic security updates
- [ ] Configure firewall (ufw/iptables)
- [ ] Disable unnecessary services
- [ ] Configure AppArmor/SELinux

### Application Security

- [ ] **JWT_SECRET** set to cryptographically random 256-bit key
- [ ] **SESSION_SECRET** set to cryptographically random 256-bit key
- [ ] **ENCRYPTION_KEY** set for data encryption (AES-256)
- [ ] **API_KEY_SALT** set for API key hashing
- [ ] All secrets stored in environment variables (never in code)
- [ ] Secrets rotation policy implemented
- [ ] Rate limiting enabled on all public endpoints
- [ ] CORS configured with strict origin whitelisting
- [ ] Helmet.js security headers enabled
- [ ] XSS protection enabled
- [ ] CSRF protection enabled for session-based auth
- [ ] Input validation on all API endpoints
- [ ] SQL injection prevention (parameterized queries)

### Database Security

**PostgreSQL:**
- [ ] Password authentication required
- [ ] SSL/TLS encryption enabled
- [ ] Row-level security (RLS) enabled for multi-tenancy
- [ ] Audit logging enabled
- [ ] Connection pooling configured (max_connections)
- [ ] Backup encryption enabled
- [ ] Regular security patches applied

**ClickHouse:**
- [ ] User authentication configured (not default user)
- [ ] Network access restricted to private network
- [ ] SSL/TLS encryption enabled
- [ ] Query logging enabled
- [ ] Data compression enabled
- [ ] Backup encryption enabled

**Redis:**
- [ ] Password authentication required (requirepass)
- [ ] Rename dangerous commands (FLUSHALL, FLUSHDB, CONFIG)
- [ ] Network access restricted to private network
- [ ] Persistence configured (AOF + RDB)
- [ ] Memory limits configured (maxmemory-policy)

### SSL/TLS Configuration

- [ ] SSL certificates installed (Let's Encrypt or commercial)
- [ ] TLS 1.2+ only (disable TLS 1.0/1.1)
- [ ] Strong cipher suites configured
- [ ] HSTS (HTTP Strict Transport Security) enabled
- [ ] Certificate auto-renewal configured
- [ ] SSL certificate monitoring enabled

### Secrets Management

```bash
# Generate secure secrets
openssl rand -base64 32  # JWT_SECRET
openssl rand -base64 32  # SESSION_SECRET
openssl rand -base64 32  # ENCRYPTION_KEY
openssl rand -base64 32  # API_KEY_SALT

# Never commit .env files to git
echo ".env" >> .gitignore
echo ".env.production" >> .gitignore
```

**Recommended Secrets Management Tools:**
- [ ] HashiCorp Vault (recommended for enterprise)
- [ ] AWS Secrets Manager
- [ ] Azure Key Vault
- [ ] Docker Secrets (for Docker Swarm)
- [ ] Kubernetes Secrets (for K8s deployments)

---

## Database Setup

### PostgreSQL Setup

**1. Installation & Configuration:**

```bash
# Install PostgreSQL 15
sudo apt update
sudo apt install postgresql-15 postgresql-contrib

# Configure PostgreSQL
sudo nano /etc/postgresql/15/main/postgresql.conf

# Recommended settings
max_connections = 200
shared_buffers = 4GB
effective_cache_size = 12GB
maintenance_work_mem = 1GB
checkpoint_completion_target = 0.9
wal_buffers = 16MB
default_statistics_target = 100
random_page_cost = 1.1
effective_io_concurrency = 200
work_mem = 10485kB
min_wal_size = 1GB
max_wal_size = 4GB
```

**2. Create Database & User:**

```sql
-- Create database
CREATE DATABASE telemetryflow;

-- Create user
CREATE USER telemetryflow_user WITH ENCRYPTED PASSWORD 'your_secure_password';

-- Grant privileges
GRANT ALL PRIVILEGES ON DATABASE telemetryflow TO telemetryflow_user;

-- Connect to database
\c telemetryflow

-- Grant schema privileges
GRANT ALL ON SCHEMA public TO telemetryflow_user;
```

**3. Run Migrations:**

```bash
# Backend migrations
cd backend
npm run migration:run

# Verify migrations
npm run migration:show
```

**4. Create Indexes:**

```sql
-- User lookup optimization
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_organization ON users(organization_id);
CREATE INDEX idx_users_active ON users(is_active) WHERE deleted_at IS NULL;

-- Multi-tenancy hierarchy
CREATE INDEX idx_tenants_workspace ON tenants(workspace_id);
CREATE INDEX idx_workspaces_org ON workspaces(organization_id);
CREATE INDEX idx_orgs_region ON organizations(region_id);

-- API keys
CREATE INDEX idx_api_keys_tenant ON api_keys(tenant_id);
CREATE INDEX idx_api_keys_hash ON api_keys(api_key_hash);

-- Audit logs
CREATE INDEX idx_audit_logs_tenant_time ON audit_logs(tenant_id, created_at DESC);
CREATE INDEX idx_audit_logs_user ON audit_logs(user_id);
```

**5. Enable Row-Level Security (RLS):**

```sql
-- Enable RLS on multi-tenant tables
ALTER TABLE api_keys ENABLE ROW LEVEL SECURITY;
ALTER TABLE dashboard_templates ENABLE ROW LEVEL SECURITY;
ALTER TABLE alert_rules ENABLE ROW LEVEL SECURITY;

-- Create RLS policies
CREATE POLICY tenant_isolation ON api_keys
  USING (tenant_id = current_setting('app.current_tenant_id')::uuid);

CREATE POLICY tenant_isolation ON dashboard_templates
  USING (tenant_id = current_setting('app.current_tenant_id')::uuid);
```

**PostgreSQL Checklist:**
- [ ] PostgreSQL 15+ installed
- [ ] Database created
- [ ] User created with secure password
- [ ] Migrations run successfully
- [ ] Indexes created
- [ ] Row-level security enabled
- [ ] Backup configured
- [ ] Monitoring configured

### ClickHouse Setup

**1. Installation:**

```bash
# Install ClickHouse
sudo apt-get install -y apt-transport-https ca-certificates dirmngr
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 8919F6BD2B48D754

echo "deb https://packages.clickhouse.com/deb stable main" | sudo tee \
    /etc/apt/sources.list.d/clickhouse.list
sudo apt-get update

sudo apt-get install -y clickhouse-server clickhouse-client

# Start ClickHouse
sudo service clickhouse-server start
```

**2. Configuration:**

```xml
<!-- /etc/clickhouse-server/config.xml -->
<clickhouse>
    <max_connections>1000</max_connections>
    <max_concurrent_queries>100</max_concurrent_queries>

    <!-- Memory limits -->
    <max_memory_usage>20000000000</max_memory_usage>  <!-- 20GB -->

    <!-- Compression -->
    <compression>
        <case>
            <method>zstd</method>
            <level>3</level>
        </case>
    </compression>

    <!-- Listen on all interfaces (behind firewall) -->
    <listen_host>0.0.0.0</listen_host>
</clickhouse>
```

**3. Create Database & Tables:**

```sql
-- Create database
CREATE DATABASE telemetryflow;

-- Metrics table
CREATE TABLE telemetryflow.telemetry_metrics (
  timestamp DateTime64(3) CODEC(Delta, ZSTD),
  metric_name LowCardinality(String),
  metric_id UUID,
  tenant_id UUID,
  workspace_id UUID,
  organization_id UUID,
  region_id UUID,
  value Float64 CODEC(Gorilla, ZSTD),
  unit String,
  labels Map(String, String),
  service_name LowCardinality(String),
  service_version LowCardinality(String),
  deployment_environment LowCardinality(String),
  created_at DateTime DEFAULT now()
)
ENGINE = MergeTree()
PARTITION BY toYYYYMM(timestamp)
ORDER BY (tenant_id, metric_name, timestamp)
TTL timestamp + INTERVAL 90 DAY
SETTINGS index_granularity = 8192;

-- Logs table
CREATE TABLE telemetryflow.telemetry_logs (
  timestamp DateTime64(3) CODEC(Delta, ZSTD),
  trace_id String,
  span_id String,
  tenant_id UUID,
  severity_text LowCardinality(String),
  severity_number UInt8,
  log_body String CODEC(ZSTD),
  attributes Map(String, String),
  service_name LowCardinality(String),
  created_at DateTime DEFAULT now()
)
ENGINE = MergeTree()
PARTITION BY toYYYYMM(timestamp)
ORDER BY (tenant_id, timestamp)
TTL timestamp + INTERVAL 90 DAY
SETTINGS index_granularity = 8192;

-- Traces table
CREATE TABLE telemetryflow.telemetry_traces (
  timestamp DateTime64(3) CODEC(Delta, ZSTD),
  trace_id String,
  span_id String,
  parent_span_id String,
  tenant_id UUID,
  span_name String,
  span_kind LowCardinality(String),
  start_time_unix_nano UInt64,
  end_time_unix_nano UInt64,
  duration_ms Float64,
  attributes Map(String, String),
  service_name LowCardinality(String),
  created_at DateTime DEFAULT now()
)
ENGINE = MergeTree()
PARTITION BY toYYYYMM(timestamp)
ORDER BY (tenant_id, trace_id, timestamp)
TTL timestamp + INTERVAL 90 DAY
SETTINGS index_granularity = 8192;
```

**4. Create Materialized Views:**

```sql
-- Metrics aggregation (1-minute granularity)
CREATE MATERIALIZED VIEW telemetryflow.metrics_1m
ENGINE = SummingMergeTree()
PARTITION BY toYYYYMM(timestamp)
ORDER BY (tenant_id, metric_name, toStartOfMinute(timestamp))
AS SELECT
  tenant_id,
  metric_name,
  toStartOfMinute(timestamp) as timestamp,
  avg(value) as value_avg,
  max(value) as value_max,
  min(value) as value_min,
  count() as count
FROM telemetryflow.telemetry_metrics
GROUP BY tenant_id, metric_name, toStartOfMinute(timestamp);

-- Metrics aggregation (1-hour granularity)
CREATE MATERIALIZED VIEW telemetryflow.metrics_1h
ENGINE = SummingMergeTree()
PARTITION BY toYYYYMM(timestamp)
ORDER BY (tenant_id, metric_name, toStartOfHour(timestamp))
AS SELECT
  tenant_id,
  metric_name,
  toStartOfHour(timestamp) as timestamp,
  avg(value) as value_avg,
  max(value) as value_max,
  min(value) as value_min,
  count() as count
FROM telemetryflow.telemetry_metrics
GROUP BY tenant_id, metric_name, toStartOfHour(timestamp);
```

**ClickHouse Checklist:**
- [ ] ClickHouse 23+ installed
- [ ] Database created
- [ ] Tables created with proper codecs
- [ ] Materialized views created
- [ ] TTL policies configured
- [ ] Partitioning configured
- [ ] Memory limits configured
- [ ] Backup configured

---

## Application Configuration

### Environment Variables

**Production .env File:**

```bash
# ============================================
# PRODUCTION ENVIRONMENT CONFIGURATION
# ============================================

# Application
NODE_ENV=production
APP_NAME=TelemetryFlow
APP_VERSION=1.0.0-CE
API_BASE_URL=https://api.telemetryflow.id
FRONTEND_URL=https://app.telemetryflow.id

# Server
BACKEND_PORT=3100
FRONTEND_PORT=80

# Security (CRITICAL: Generate new secrets!)
JWT_SECRET=<generate-with-openssl-rand-base64-32>
SESSION_SECRET=<generate-with-openssl-rand-base64-32>
ENCRYPTION_KEY=<generate-with-openssl-rand-base64-32>
API_KEY_SALT=<generate-with-openssl-rand-base64-32>

# JWT Configuration
JWT_ACCESS_TOKEN_EXPIRES_IN=15m
JWT_REFRESH_TOKEN_EXPIRES_IN=7d

# Session Configuration
SESSION_MAX_AGE=86400000  # 24 hours

# PostgreSQL
POSTGRES_HOST=postgres.internal.telemetryflow.id
POSTGRES_PORT=5432
POSTGRES_DB=telemetryflow
POSTGRES_USER=telemetryflow_user
POSTGRES_PASSWORD=<secure-postgres-password>
POSTGRES_SSL=true
POSTGRES_POOL_MIN=5
POSTGRES_POOL_MAX=50

# ClickHouse
CLICKHOUSE_HOST=http://clickhouse.internal.telemetryflow.id:8123
CLICKHOUSE_USER=telemetryflow_user
CLICKHOUSE_PASSWORD=<secure-clickhouse-password>
CLICKHOUSE_DATABASE=telemetryflow
CLICKHOUSE_MAX_INSERT_BLOCK_SIZE=1048576
CLICKHOUSE_ASYNC_INSERT=1
CLICKHOUSE_WAIT_FOR_ASYNC_INSERT=0

# Redis
REDIS_HOST=redis.internal.telemetryflow.id
REDIS_PORT=6379
REDIS_PASSWORD=<secure-redis-password>
REDIS_DB=0
REDIS_TTL=3600

# NATS
NATS_URL=nats://nats.internal.telemetryflow.id:4222
NATS_USER=telemetryflow_user
NATS_PASSWORD=<secure-nats-password>

# BullMQ
BULLMQ_REDIS_HOST=redis.internal.telemetryflow.id
BULLMQ_REDIS_PORT=6379
BULLMQ_REDIS_PASSWORD=<secure-redis-password>
BULLMQ_REDIS_DB=1

# Email (Production SMTP)
EMAIL_HOST=smtp.sendgrid.net
EMAIL_PORT=587
EMAIL_SECURE=true
EMAIL_USER=apikey
EMAIL_PASSWORD=<sendgrid-api-key>
EMAIL_FROM=noreply@telemetryflow.id

# Rate Limiting
RATE_LIMIT_WINDOW_MS=60000
RATE_LIMIT_MAX_REQUESTS=100

# CORS
CORS_ORIGINS=https://app.telemetryflow.id,https://telemetryflow.id
CORS_CREDENTIALS=true

# Logging
LOG_LEVEL=info
LOG_FORMAT=json
LOG_FILE_PATH=/var/log/telemetryflow/app.log

# OpenTelemetry (Self-Monitoring)
OTEL_ENABLED=true
OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4317
OTEL_SERVICE_NAME=telemetryflow-backend
OTEL_TRACES_EXPORTER=otlp
OTEL_METRICS_EXPORTER=otlp

# Health Check
HEALTH_CHECK_ENABLED=true
HEALTH_CHECK_PATH=/health

# Monitoring
PROMETHEUS_ENABLED=true
PROMETHEUS_PORT=9090

# Backup
BACKUP_ENABLED=true
BACKUP_S3_BUCKET=telemetryflow-backups
BACKUP_S3_REGION=us-east-1
BACKUP_S3_ACCESS_KEY=<aws-access-key>
BACKUP_S3_SECRET_KEY=<aws-secret-key>
```

**Configuration Checklist:**
- [ ] All environment variables set
- [ ] All secrets generated and secured
- [ ] Database credentials configured
- [ ] SMTP credentials configured
- [ ] SSL/TLS enabled for all services
- [ ] CORS origins whitelisted
- [ ] Rate limiting configured
- [ ] Logging configured
- [ ] Monitoring enabled

---

## Monitoring & Observability

### Application Monitoring

**1. Health Checks:**

```bash
# Backend health
curl https://api.telemetryflow.id/health

# Expected response:
{
  "status": "ok",
  "info": {
    "database": { "status": "up" },
    "clickhouse": { "status": "up" },
    "redis": { "status": "up" },
    "nats": { "status": "up" }
  },
  "error": {},
  "details": {
    "database": { "status": "up" },
    "clickhouse": { "status": "up" },
    "redis": { "status": "up" },
    "nats": { "status": "up" }
  }
}
```

**2. Prometheus Metrics:**

- [ ] Prometheus endpoint exposed (`/metrics`)
- [ ] Metrics scraping configured
- [ ] Custom application metrics tracked:
  - `http_requests_total`
  - `http_request_duration_ms`
  - `otlp_ingestion_total`
  - `otlp_ingestion_duration_ms`
  - `clickhouse_queries_total`
  - `clickhouse_query_duration_ms`
  - `redis_cache_hit_total`
  - `redis_cache_miss_total`
  - `queue_jobs_processed_total`
  - `queue_jobs_failed_total`

**3. OpenTelemetry Self-Monitoring:**

- [ ] OTEL collector configured
- [ ] Traces exported to backend
- [ ] Metrics exported to backend
- [ ] Logs exported to backend
- [ ] Service dependencies tracked

**4. Log Aggregation:**

- [ ] Structured JSON logging enabled
- [ ] Log levels configured (info, warn, error)
- [ ] Log rotation configured
- [ ] Centralized log aggregation (Loki/ELK/CloudWatch)
- [ ] Log retention policy configured
- [ ] Sensitive data redacted from logs

### Database Monitoring

**PostgreSQL:**
- [ ] Connection pool monitoring
- [ ] Query performance tracking (pg_stat_statements)
- [ ] Slow query logging enabled
- [ ] Replication lag monitoring (if applicable)
- [ ] Disk usage monitoring
- [ ] Backup success monitoring

**ClickHouse:**
- [ ] Query execution time monitoring
- [ ] Insert rate monitoring
- [ ] Disk usage per table
- [ ] Merge performance tracking
- [ ] Memory usage monitoring
- [ ] Part count monitoring

**Redis:**
- [ ] Memory usage monitoring
- [ ] Cache hit ratio tracking
- [ ] Connection count monitoring
- [ ] Persistence status monitoring
- [ ] Eviction rate tracking

### Alerting

- [ ] CPU usage > 80%
- [ ] Memory usage > 85%
- [ ] Disk usage > 80%
- [ ] Database connection pool exhaustion
- [ ] ClickHouse insert failures
- [ ] Redis connection failures
- [ ] NATS connection failures
- [ ] Application error rate > 1%
- [ ] API response time > 2s (p95)
- [ ] Failed background jobs
- [ ] SSL certificate expiration (< 30 days)
- [ ] Backup failures

---

## Backup & Disaster Recovery

### PostgreSQL Backup

**1. Automated Backups:**

```bash
#!/bin/bash
# /etc/cron.daily/postgres-backup.sh

BACKUP_DIR="/backups/postgres"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="$BACKUP_DIR/telemetryflow_$TIMESTAMP.sql.gz"

# Create backup
pg_dump -h postgres.internal.telemetryflow.id \
        -U telemetryflow_user \
        -d telemetryflow \
        | gzip > $BACKUP_FILE

# Upload to S3
aws s3 cp $BACKUP_FILE s3://telemetryflow-backups/postgres/

# Cleanup old backups (keep 7 days)
find $BACKUP_DIR -type f -mtime +7 -delete

# Log backup
echo "$(date): Backup completed: $BACKUP_FILE" >> /var/log/backups.log
```

**2. Point-in-Time Recovery:**

```bash
# Enable WAL archiving in postgresql.conf
wal_level = replica
archive_mode = on
archive_command = 'test ! -f /backups/wal_archive/%f && cp %p /backups/wal_archive/%f'
```

**PostgreSQL Backup Checklist:**
- [ ] Daily full backups scheduled
- [ ] WAL archiving enabled
- [ ] Backups stored off-site (S3/Azure/GCS)
- [ ] Backup encryption enabled
- [ ] Backup retention policy configured (30 days)
- [ ] Restore procedures tested
- [ ] Backup monitoring enabled

### ClickHouse Backup

**1. Automated Backups:**

```bash
#!/bin/bash
# /etc/cron.daily/clickhouse-backup.sh

BACKUP_DIR="/backups/clickhouse"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)

# Freeze tables (creates hardlinks)
clickhouse-client -q "ALTER TABLE telemetryflow.telemetry_metrics FREEZE"
clickhouse-client -q "ALTER TABLE telemetryflow.telemetry_logs FREEZE"
clickhouse-client -q "ALTER TABLE telemetryflow.telemetry_traces FREEZE"

# Copy frozen data
tar -czf $BACKUP_DIR/clickhouse_$TIMESTAMP.tar.gz \
    /var/lib/clickhouse/shadow/

# Upload to S3
aws s3 cp $BACKUP_DIR/clickhouse_$TIMESTAMP.tar.gz \
    s3://telemetryflow-backups/clickhouse/

# Cleanup shadow directory
rm -rf /var/lib/clickhouse/shadow/*

# Cleanup old backups (keep 30 days)
find $BACKUP_DIR -type f -mtime +30 -delete
```

**ClickHouse Backup Checklist:**
- [ ] Daily backups scheduled
- [ ] Backups stored off-site
- [ ] Backup encryption enabled
- [ ] Retention policy: 30 days
- [ ] Restore procedures tested

### Disaster Recovery Plan

**Recovery Time Objective (RTO):** 4 hours
**Recovery Point Objective (RPO):** 24 hours

**DR Checklist:**
- [ ] DR site identified and configured
- [ ] Database replication configured (optional)
- [ ] Backup restore procedures documented
- [ ] DR runbook created
- [ ] DR drills conducted quarterly
- [ ] Contact list for DR team maintained
- [ ] Escalation procedures documented

---

## Performance Optimization

### Backend Optimization

**1. Node.js Configuration:**

```bash
# Set Node.js memory limit
NODE_OPTIONS="--max-old-space-size=4096"

# Enable cluster mode (multiple workers)
PM2_INSTANCES=4
```

**2. Connection Pooling:**

```typescript
// PostgreSQL pool
{
  min: 5,
  max: 50,
  idle: 10000,
  acquire: 30000,
}

// Redis pool
{
  maxRetriesPerRequest: 3,
  enableReadyCheck: true,
  maxLoadingRetryTime: 10000,
}
```

**3. Caching Strategy:**

- [ ] L1 cache (in-memory): Frequently accessed data
- [ ] L2 cache (Redis): Shared cache across instances
- [ ] Cache invalidation on data changes
- [ ] Cache warm-up on startup
- [ ] Cache monitoring (hit ratio > 80%)

**Backend Optimization Checklist:**
- [ ] Cluster mode enabled (multiple workers)
- [ ] Connection pooling optimized
- [ ] Multi-level caching implemented
- [ ] Async operations for I/O-bound tasks
- [ ] Response compression enabled (gzip)
- [ ] Static asset caching configured

### Database Optimization

**PostgreSQL:**
- [ ] Indexes created on frequently queried columns
- [ ] Query optimization (EXPLAIN ANALYZE)
- [ ] Vacuum and analyze scheduled
- [ ] Connection pooling configured
- [ ] Prepared statements used
- [ ] Pagination implemented for large result sets

**ClickHouse:**
- [ ] Partitioning by time (monthly)
- [ ] Proper ordering key (tenant_id, metric_name, timestamp)
- [ ] Compression codecs (Delta, ZSTD, Gorilla)
- [ ] Materialized views for aggregations
- [ ] TTL for automatic data expiration
- [ ] Async inserts enabled
- [ ] Batch inserts (10,000+ rows)

**Redis:**
- [ ] Memory eviction policy configured (allkeys-lru)
- [ ] Maximum memory limit set
- [ ] Persistence configured (AOF + RDB)
- [ ] Key expiration policies set

### Frontend Optimization

- [ ] Code splitting enabled
- [ ] Lazy loading for routes
- [ ] Asset compression (Brotli/gzip)
- [ ] CDN configured for static assets
- [ ] Image optimization
- [ ] Browser caching configured
- [ ] Service worker for offline support

### Load Testing

**Apache Bench Example:**

```bash
# Test API endpoint (100 concurrent, 10000 requests)
ab -n 10000 -c 100 https://api.telemetryflow.id/api/v1/metrics

# Expected results:
# - Requests per second: > 1000
# - Time per request (mean): < 100ms
# - Failed requests: 0
```

**Load Testing Checklist:**
- [ ] API load testing completed
- [ ] OTLP ingestion load testing completed
- [ ] Database query performance tested
- [ ] Cache performance tested
- [ ] Peak load capacity identified
- [ ] Autoscaling thresholds configured

---

## Deployment Procedures

### Pre-Deployment Steps

1. **Code Freeze:**
   - [ ] Code freeze announced 24h before deployment
   - [ ] All PRs merged and tested
   - [ ] Release notes prepared

2. **Backup:**
   - [ ] Full database backup completed
   - [ ] Backup verified and tested

3. **Communication:**
   - [ ] Stakeholders notified
   - [ ] Maintenance window scheduled
   - [ ] Status page updated

### Deployment Steps

**Using Docker Compose:**

```bash
# 1. Pull latest code
cd /opt/telemetryflow
git pull origin main

# 2. Pull Docker images
docker-compose pull

# 3. Run database migrations (if needed)
docker-compose run --rm backend npm run migration:run

# 4. Stop services gracefully
docker-compose down --timeout 30

# 5. Start services
docker-compose up -d

# 6. Verify health
docker-compose ps
curl https://api.telemetryflow.id/health

# 7. Check logs
docker-compose logs -f backend
```

**Using Kubernetes:**

```bash
# 1. Apply new manifests
kubectl apply -f k8s/

# 2. Run migrations (job)
kubectl apply -f k8s/migration-job.yaml
kubectl wait --for=condition=complete job/migration-job

# 3. Rolling update
kubectl rollout restart deployment/backend
kubectl rollout restart deployment/frontend

# 4. Monitor rollout
kubectl rollout status deployment/backend
kubectl rollout status deployment/frontend

# 5. Verify pods
kubectl get pods -l app=telemetryflow

# 6. Check logs
kubectl logs -f deployment/backend
```

**Deployment Checklist:**
- [ ] Code pulled from version control
- [ ] Docker images pulled
- [ ] Database migrations run
- [ ] Services restarted
- [ ] Health checks passing
- [ ] Logs reviewed for errors
- [ ] Monitoring dashboards checked

### Zero-Downtime Deployment

**Strategy: Rolling Update**

```yaml
# docker-compose.yml
services:
  backend:
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
        order: start-first
      rollback_config:
        parallelism: 1
        delay: 5s
```

**Zero-Downtime Checklist:**
- [ ] Multiple replicas running (min 2)
- [ ] Load balancer configured
- [ ] Health checks configured
- [ ] Graceful shutdown enabled
- [ ] Database migrations backward-compatible
- [ ] Feature flags for breaking changes

---

## Post-Deployment Validation

### Application Validation

**1. Health Checks:**

```bash
# Backend health
curl https://api.telemetryflow.id/health
# Expected: 200 OK

# Frontend accessibility
curl https://app.telemetryflow.id
# Expected: 200 OK

# OTLP ingestion
curl https://api.telemetryflow.id:4317
# Expected: Connection accepted
```

**2. Functional Testing:**

```bash
# Test authentication
curl -X POST https://api.telemetryflow.id/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@telemetryflow.id","password":"password"}'

# Test metrics ingestion
curl -X POST https://api.telemetryflow.id/v1/metrics \
  -H "Content-Type: application/json" \
  -H "X-API-Key: your-api-key" \
  -d '{"resourceMetrics":[...]}'

# Test metrics query
curl https://api.telemetryflow.id/api/v1/metrics?metric_name=cpu_usage \
  -H "Authorization: Bearer <token>"
```

**3. Performance Validation:**

```bash
# Check response times
ab -n 100 -c 10 https://api.telemetryflow.id/api/v1/metrics

# Expected:
# - Mean time per request: < 200ms
# - Failed requests: 0
```

**4. Database Validation:**

```sql
-- PostgreSQL: Check row counts
SELECT 'users' as table, COUNT(*) as count FROM users
UNION ALL
SELECT 'tenants', COUNT(*) FROM tenants
UNION ALL
SELECT 'api_keys', COUNT(*) FROM api_keys;

-- ClickHouse: Check recent data
SELECT
  toStartOfHour(timestamp) as hour,
  COUNT(*) as metric_count
FROM telemetry_metrics
WHERE timestamp > now() - INTERVAL 1 HOUR
GROUP BY hour
ORDER BY hour DESC;
```

**Post-Deployment Checklist:**
- [ ] All health checks passing
- [ ] Authentication working
- [ ] OTLP ingestion working
- [ ] API endpoints responding correctly
- [ ] Frontend accessible
- [ ] Database connections stable
- [ ] Cache operational
- [ ] Background jobs processing
- [ ] Metrics being collected
- [ ] Logs being written
- [ ] No error spikes in logs
- [ ] Response times within SLA
- [ ] CPU/Memory usage normal
- [ ] Disk usage normal

### Monitoring Validation

- [ ] Prometheus scraping metrics
- [ ] Grafana dashboards displaying data
- [ ] Alerts configured and working
- [ ] Log aggregation working
- [ ] APM traces visible
- [ ] Uptime monitors green

### User Acceptance

- [ ] Key users notified of deployment
- [ ] UAT completed for new features
- [ ] No critical bugs reported
- [ ] User feedback collected
- [ ] Support team ready

---

## Rollback Procedures

### When to Rollback

Initiate rollback if:
- Critical bugs discovered affecting core functionality
- Data corruption detected
- Performance degradation > 50%
- Security vulnerability introduced
- Health checks failing for > 5 minutes
- Error rate > 5%

### Rollback Steps

**Docker Compose Rollback:**

```bash
# 1. Stop current services
docker-compose down

# 2. Checkout previous version
git checkout <previous-tag>

# 3. Pull previous images
docker-compose pull

# 4. Start services
docker-compose up -d

# 5. Verify health
curl https://api.telemetryflow.id/health

# 6. Rollback database (if needed)
docker-compose exec postgres psql -U telemetryflow_user -d telemetryflow
# Run rollback migration
```

**Kubernetes Rollback:**

```bash
# 1. Rollback deployment
kubectl rollout undo deployment/backend
kubectl rollout undo deployment/frontend

# 2. Check rollout status
kubectl rollout status deployment/backend

# 3. Verify pods
kubectl get pods -l app=telemetryflow

# 4. Check logs
kubectl logs -f deployment/backend
```

**Database Rollback:**

```bash
# PostgreSQL: Restore from backup
pg_restore -h postgres.internal.telemetryflow.id \
           -U telemetryflow_user \
           -d telemetryflow \
           /backups/postgres/telemetryflow_20251212_100000.sql.gz

# ClickHouse: Restore from backup
tar -xzf /backups/clickhouse/clickhouse_20251212_100000.tar.gz \
    -C /var/lib/clickhouse/
```

**Rollback Checklist:**
- [ ] Rollback decision made by authorized person
- [ ] Users notified of rollback
- [ ] Services rolled back to previous version
- [ ] Database rolled back (if needed)
- [ ] Health checks passing
- [ ] Functionality verified
- [ ] Incident report created
- [ ] Root cause analysis scheduled

---

## Maintenance & Operations

### Daily Operations

**Morning Checklist:**
- [ ] Check health dashboard
- [ ] Review error logs
- [ ] Check disk space
- [ ] Verify backups completed
- [ ] Review performance metrics
- [ ] Check for security alerts

**Weekly Operations:**
- [ ] Review slow query logs
- [ ] Analyze application performance trends
- [ ] Check SSL certificate expiration
- [ ] Review user feedback
- [ ] Update dependencies (security patches)
- [ ] Review and rotate access logs

**Monthly Operations:**
- [ ] Database vacuum and analyze (PostgreSQL)
- [ ] Review and optimize ClickHouse partitions
- [ ] Test disaster recovery procedures
- [ ] Review capacity planning
- [ ] Security audit
- [ ] Update documentation

### Scaling Procedures

**Horizontal Scaling (Add more instances):**

```bash
# Docker Compose
docker-compose up -d --scale backend=3

# Kubernetes
kubectl scale deployment backend --replicas=5
```

**Vertical Scaling (Increase resources):**

```yaml
# Update resource limits
services:
  backend:
    deploy:
      resources:
        limits:
          cpus: '4'
          memory: 16G
        reservations:
          cpus: '2'
          memory: 8G
```

**Database Scaling:**

```sql
-- PostgreSQL: Increase connection pool
ALTER SYSTEM SET max_connections = 300;
SELECT pg_reload_conf();

-- ClickHouse: Add more memory
-- Edit /etc/clickhouse-server/config.xml
<max_memory_usage>40000000000</max_memory_usage>
```

### Troubleshooting

**High CPU Usage:**
```bash
# Check processes
docker-compose top
kubectl top pods

# Identify slow queries (PostgreSQL)
SELECT pid, query, state, wait_event_type
FROM pg_stat_activity
WHERE state = 'active'
ORDER BY query_start;

# Identify slow queries (ClickHouse)
SELECT query, elapsed, rows_read, bytes_read
FROM system.processes
WHERE elapsed > 10
ORDER BY elapsed DESC;
```

**High Memory Usage:**
```bash
# Check memory usage
docker stats
kubectl top nodes

# Clear Redis cache (if needed)
redis-cli FLUSHDB
```

**Connection Issues:**
```bash
# Test database connectivity
docker-compose exec backend npx typeorm query "SELECT 1"

# Test Redis connectivity
docker-compose exec backend redis-cli -h redis ping

# Test ClickHouse connectivity
docker-compose exec backend curl http://clickhouse:8123/ping
```

**Disk Space Issues:**
```bash
# Check disk usage
df -h

# Find large files
du -h /var/lib/docker | sort -rh | head -20

# Cleanup Docker
docker system prune -a --volumes

# Cleanup old logs
find /var/log/telemetryflow -type f -mtime +7 -delete
```

### Incident Response

**Incident Severity Levels:**

| Severity | Description | Response Time | Example |
|----------|-------------|---------------|---------|
| **P0 - Critical** | Complete outage | < 15 minutes | Database down |
| **P1 - High** | Major feature broken | < 1 hour | OTLP ingestion failing |
| **P2 - Medium** | Degraded performance | < 4 hours | Slow API responses |
| **P3 - Low** | Minor issue | < 24 hours | UI bug |

**Incident Response Checklist:**
- [ ] Incident detected and classified
- [ ] On-call engineer notified
- [ ] Incident channel created (Slack/Teams)
- [ ] Status page updated
- [ ] Investigation started
- [ ] Mitigation applied
- [ ] Service restored
- [ ] Users notified
- [ ] Post-mortem scheduled

### Useful Commands

```bash
# Docker Compose
docker-compose ps                    # List services
docker-compose logs -f backend       # Follow logs
docker-compose exec backend bash     # Shell access
docker-compose restart backend       # Restart service
docker-compose down && docker-compose up -d  # Full restart

# Database
docker-compose exec postgres psql -U telemetryflow_user -d telemetryflow
docker-compose exec backend clickhouse-client

# Redis
docker-compose exec redis redis-cli

# Check resource usage
docker stats

# Cleanup
docker system prune -a
docker volume prune
```

---

## Security Incident Response

### Potential Security Incidents

- Unauthorized access attempts
- SQL injection attempts
- DDoS attacks
- Data breach
- Privilege escalation
- Malware detection

### Incident Response Steps

1. **Detect & Assess:**
   - [ ] Incident detected via monitoring/alerts
   - [ ] Severity assessed
   - [ ] Scope determined

2. **Contain:**
   - [ ] Block malicious IPs
   - [ ] Disable compromised accounts
   - [ ] Isolate affected systems

3. **Eradicate:**
   - [ ] Remove malicious code/access
   - [ ] Patch vulnerabilities
   - [ ] Rotate compromised credentials

4. **Recover:**
   - [ ] Restore from clean backups
   - [ ] Verify system integrity
   - [ ] Resume normal operations

5. **Post-Incident:**
   - [ ] Document incident
   - [ ] Conduct post-mortem
   - [ ] Implement preventive measures
   - [ ] Update security policies

---

## Compliance & Auditing

### Compliance Requirements

**SOC 2:**
- [ ] Audit logging enabled for all access
- [ ] Data encryption at rest and in transit
- [ ] Access controls implemented
- [ ] Incident response procedures documented

**ISO 27001:**
- [ ] Information security policy documented
- [ ] Risk assessment completed
- [ ] Security controls implemented
- [ ] Regular security audits conducted

**GDPR:**
- [ ] Data processing agreements in place
- [ ] User consent management implemented
- [ ] Data retention policies configured
- [ ] Right to erasure implemented
- [ ] Data breach notification procedures in place

### Audit Logging

**Events to Log:**
- Authentication attempts (success/failure)
- Authorization failures
- Data access (sensitive data)
- Configuration changes
- User account changes
- API key creation/revocation
- Data exports

**Audit Log Retention:**
- Authentication logs: 90 days
- Access logs: 90 days
- Audit logs: 365 days
- Security logs: 365 days

---

## Final Production Checklist

### Infrastructure ✅
- [ ] Servers provisioned and secured
- [ ] Firewall configured
- [ ] SSL/TLS certificates installed
- [ ] DNS configured
- [ ] Load balancer configured
- [ ] Backup storage configured

### Security ✅
- [ ] All secrets generated and secured
- [ ] SSH key-based authentication only
- [ ] Fail2ban configured
- [ ] HTTPS enforced
- [ ] Security headers enabled
- [ ] Rate limiting enabled
- [ ] CORS configured

### Database ✅
- [ ] PostgreSQL 15+ installed and configured
- [ ] ClickHouse 23+ installed and configured
- [ ] Redis 7+ installed and configured
- [ ] All migrations run successfully
- [ ] Indexes created
- [ ] Backups configured and tested

### Application ✅
- [ ] Backend deployed and running
- [ ] Frontend deployed and running
- [ ] Environment variables configured
- [ ] Health checks passing
- [ ] Background jobs processing
- [ ] OTLP ingestion working

### Monitoring ✅
- [ ] Prometheus scraping metrics
- [ ] Grafana dashboards configured
- [ ] Alerts configured
- [ ] Log aggregation configured
- [ ] Uptime monitoring enabled
- [ ] APM configured

### Backup & DR ✅
- [ ] Automated backups configured
- [ ] Backups tested and verified
- [ ] DR plan documented
- [ ] DR drills conducted

### Documentation ✅
- [ ] Deployment procedures documented
- [ ] Runbooks created
- [ ] Architecture documented
- [ ] API documentation published
- [ ] User guides available

### Compliance ✅
- [ ] Audit logging enabled
- [ ] Data retention policies configured
- [ ] Privacy policy published
- [ ] Terms of service published

---

## Support Contacts

### Emergency Contacts

| Role | Contact | Phone | Email |
|------|---------|-------|-------|
| **DevOps Lead** | John Doe | +1-555-0001 | john@telemetryflow.id |
| **Backend Lead** | Jane Smith | +1-555-0002 | jane@telemetryflow.id |
| **Database Admin** | Bob Johnson | +1-555-0003 | bob@telemetryflow.id |
| **Security Lead** | Alice Brown | +1-555-0004 | alice@telemetryflow.id |

### Escalation Path

1. **Level 1:** On-call engineer
2. **Level 2:** Team lead
3. **Level 3:** Engineering manager
4. **Level 4:** CTO

### Vendor Support

- **PostgreSQL:** Community / Enterprise Support
- **ClickHouse:** ClickHouse Inc. Support
- **Redis:** Redis Labs Support
- **AWS:** AWS Support (if using AWS)

---

## Related Documentation

- **[DOCKER-COMPOSE.md](./DOCKER-COMPOSE.md)** - Docker deployment guide
- **[KUBERNETES.md](./KUBERNETES.md)** - Kubernetes deployment guide
- **[CONFIGURATION.md](./CONFIGURATION.md)** - Environment configuration
- **[04-SECURITY.md](../architecture/04-SECURITY.md)** - Security architecture
- **[05-PERFORMANCE.md](../architecture/05-PERFORMANCE.md)** - Performance optimization

---

## Changelog

| Date | Version | Changes |
|------|---------|---------|
| 2025-12-12 | 1.0.0-CE | Initial production checklist |

---

**Document Status:** ✅ Production Ready
**Last Updated:** December 12, 2025
**Maintained By:** DevOpsCorner Indonesia
**Review Cycle:** Quarterly
