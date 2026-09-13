# YemenMart Deployment Strategy

## Table of Contents

1. [Environment Strategy](#1-environment-strategy)
2. [Infrastructure Requirements](#2-infrastructure-requirements)
3. [CI/CD Pipeline](#3-cicd-pipeline)
4. [Deployment Process](#4-deployment-process)
5. [Monitoring & Alerting](#5-monitoring--alerting)
6. [Backup & Recovery](#6-backup--recovery)
7. [Security](#7-security)
8. [Rollback Procedures](#8-rollback-procedures)

---

## 1. Environment Strategy

### Development (Local)

| Aspect | Detail |
|--------|--------|
| **Purpose** | Individual developer work, feature building, bug fixes |
| **Infrastructure** | Docker Compose (single host) |
| **Database** | PostgreSQL 15 container with seed data |
| **Cache** | Redis 7 container |
| **Search** | Elasticsearch 8 container |
| **Storage** | MinIO container (S3-compatible) |
| **URL** | `http://localhost:3000` |
| **Data** | Synthetic test data, reset on `docker compose down -v` |

**Local Setup Commands:**

```bash
# Clone and start
git clone https://github.com/yemenmart/backend.git
cd backend
cp .env.example .env
docker compose up -d
docker compose exec api python manage.py migrate
docker compose exec api python manage.py seed_data
```

### Testing (QA)

| Aspect | Detail |
|--------|--------|
| **Purpose** | QA validation, automated test runs, exploratory testing |
| **Infrastructure** | Single VM (4 vCPU, 8 GB RAM) |
| **Database** | PostgreSQL 15 (dedicated instance) |
| **Cache** | Redis 7 (single node) |
| **Search** | Elasticsearch 8 (single node) |
| **Storage** | MinIO (single node) |
| **URL** | `https://qa.yemenmart.com` |
| **Data** | Anonymized production snapshot, refreshed weekly |
| **Deploy Trigger** | Push to `develop` branch |

### Staging (Pre-Production)

| Aspect | Detail |
|--------|--------|
| **Purpose** | Final validation before production, load testing, UAT |
| **Infrastructure** | Mirrors production topology (scaled down) |
| **Database** | PostgreSQL 15 (primary + read replica) |
| **Cache** | Redis 7 (sentinel, 2 nodes) |
| **Search** | Elasticsearch 8 (single node) |
| **Storage** | MinIO (single node) |
| **URL** | `https://staging.yemenmart.com` |
| **Data** | Full anonymized production snapshot, refreshed daily |
| **Deploy Trigger** | Merge to `main` branch or release tag |
| **Gate** | All QA tests passing, load test results acceptable |

### Production

| Aspect | Detail |
|--------|--------|
| **Purpose** | Live system serving real users |
| **Infrastructure** | Full scaled architecture (see §2) |
| **Database** | PostgreSQL 15 (primary + read replica + daily backups) |
| **Cache** | Redis 7 (3-node cluster) |
| **Search** | Elasticsearch 8 (3-node cluster) |
| **Storage** | MinIO/S3 (distributed, versioned) |
| **CDN** | Cloudflare |
| **URL** | `https://yemenmart.com` |
| **Deploy Trigger** | Manual promotion from staging via CI/CD approval gate |
| **SLA** | 99.9% uptime, <200ms p95 API latency |

### Environment Variable Matrix

| Variable | Dev | QA | Staging | Production |
|----------|-----|-----|---------|------------|
| `DEBUG` | `true` | `false` | `false` | `false` |
| `DATABASE_URL` | localhost | qa-db.yemenmart | staging-db.yemenmart | prod-db.yemenmart |
| `REDIS_URL` | localhost | qa-redis.yemenmart | staging-redis.yemenmart | prod-redis.yemenmart |
| `ELASTICSEARCH_URL` | localhost | qa-es.yemenmart | staging-es.yemenmart | prod-es.yemenmart |
| `S3_ENDPOINT` | localhost (minio) | qa-minio.yemenmart | staging-minio.yemenmart | prod-s3.amazonaws.com |
| `SENTRY_DSN` | (disabled) | qa-dsn | staging-dsn | prod-dsn |
| `LOG_LEVEL` | `DEBUG` | `INFO` | `INFO` | `WARNING` |
| `ALLOWED_HOSTS` | `localhost` | `qa.yemenmart.com` | `staging.yemenmart.com` | `yemenmart.com, www.yemenmart.com` |

---

## 2. Infrastructure Requirements

### Server Architecture

```
                    ┌─────────────┐
                    │  Cloudflare  │
                    │  (CDN/WAF)  │
                    └──────┬──────┘
                           │
                    ┌──────┴──────┐
                    │   Load      │
                    │  Balancer   │
                    │  (HAProxy/  │
                    │   Nginx)    │
                    └──────┬──────┘
                           │
              ┌────────────┴────────────┐
              │                         │
       ┌──────┴──────┐          ┌──────┴──────┐
       │  API Server │          │  API Server │
       │     #1      │          │     #2      │
       │  (4 vCPU,   │          │  (4 vCPU,   │
       │   8GB RAM)  │          │   8GB RAM)  │
       └──────┬──────┘          └──────┬──────┘
              │                         │
       ┌──────┴─────────────────────────┴──────┐
       │                                        │
  ┌────┴────┐  ┌──────────┐  ┌──────────┐  ┌──┴──────┐
  │PostgreSQL│  │  Redis   │  │Elastic-  │  │  MinIO  │
  │ Primary  │  │ Cluster  │  │ search   │  │ /S3     │
  │    +     │  │ (3 nodes)│  │ Cluster  │  │ Storage │
  │ Replica  │  └──────────┘  │ (3 nodes)│  └─────────┘
  └─────────┘                 └──────────┘
```

### Server Specifications

#### API Servers (2x)

| Component | Specification |
|-----------|---------------|
| **OS** | Ubuntu 22.04 LTS |
| **CPU** | 4 vCPU (AMD EPYC / Intel Xeon) |
| **RAM** | 8 GB |
| **Disk** | 50 GB SSD (NVMe) |
| **Network** | 1 Gbps |
| **Runtime** | Python 3.11+ / Node.js 20 LTS |
| **App Server** | Gunicorn (4 workers) / Uvicorn (4 workers) |
| **Reverse Proxy** | Nginx 1.24+ |

**Software on each API server:**

```bash
# System packages
apt update && apt install -y \
  python3.11 python3.11-venv python3-pip \
  nginx certbot python3-certbot-nginx \
  prometheus-node-exporter \
  filebeat

# Application
python3.11 -m venv /opt/yemenmart/venv
source /opt/yemenmart/venv/bin/activate
pip install -r requirements/production.txt
```

**Nginx Configuration:**

```nginx
upstream yemenmart_api {
    least_conn;
    server api-server-1:8000 weight=5;
    server api-server-2:8000 weight=5;
    keepalive 32;
}

server {
    listen 80;
    server_name yemenmart.com www.yemenmart.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name yemenmart.com www.yemenmart.com;

    ssl_certificate /etc/letsencrypt/live/yemenmart.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yemenmart.com/privkey.pem;

    # Security headers
    add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload";
    add_header X-Content-Type-Options nosniff;
    add_header X-Frame-Options DENY;
    add_header X-XSS-Protection "1; mode=block";

    location /api/ {
        proxy_pass http://yemenmart_api;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_connect_timeout 5s;
        proxy_read_timeout 30s;
        proxy_send_timeout 30s;
    }

    location /static/ {
        alias /opt/yemenmart/static/;
        expires 30d;
        add_header Cache-Control "public, immutable";
    }

    location /health {
        access_log off;
        return 200 '{"status":"healthy"}';
        add_header Content-Type application/json;
    }
}
```

**Gunicorn Configuration:**

```python
# gunicorn.conf.py
import multiprocessing

bind = "0.0.0.0:8000"
workers = multiprocessing.cpu_count() * 2 + 1
worker_class = "uvicorn.workers.UvicornWorker"
timeout = 30
keepalive = 5
max_requests = 1000
max_requests_jitter = 50
accesslog = "/var/log/yemenmart/access.log"
errorlog = "/var/log/yemenmart/error.log"
loglevel = "warning"
```

### Database: PostgreSQL 15

| Node | Role | Specification |
|------|------|---------------|
| **Primary** | Read/Write | 8 vCPU, 32 GB RAM, 500 GB SSD |
| **Replica** | Read-only | 4 vCPU, 16 GB RAM, 500 GB SSD |

**postgresql.conf (Primary):**

```ini
# Memory
shared_buffers = 8GB
effective_cache_size = 24GB
work_mem = 64MB
maintenance_work_mem = 2GB

# Write Ahead Log
wal_level = replica
max_wal_senders = 5
wal_keep_size = 1GB
archive_mode = on
archive_command = 'test ! -f /archive/%f && cp %p /archive/%f'

# Checkpoints
checkpoint_timeout = 10min
checkpoint_completion_target = 0.9
max_wal_size = 4GB

# Connections
max_connections = 200

# Logging
log_min_duration_statement = 500
log_checkpoints = on
log_connections = on
log_disconnections = on
log_lock_waits = on
```

**Replication Setup:**

```sql
-- On primary
CREATE ROLE replicator WITH REPLICATION LOGIN PASSWORD 'secure_password';
-- pg_hba.conf addition
-- host replication replicator replica_ip/32 scram-sha-256

-- On replica
SELECT pg_basebackup(
  host = 'primary_ip',
  user = 'replicator',
  checkpoint = 'fast'
);
```

### Cache: Redis 7+ (Cluster)

| Node | Role | Specification |
|------|------|---------------|
| **Node 1** | Master (slots 0-5460) | 2 vCPU, 4 GB RAM |
| **Node 2** | Master (slots 5461-10922) | 2 vCPU, 4 GB RAM |
| **Node 3** | Master (slots 10923-16383) | 2 vCPU, 4 GB RAM |

**redis.conf:**

```ini
maxmemory 3gb
maxmemory-policy allkeys-lru
appendonly yes
appendfsync everysec
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
requirepass <secure_password>
```

**Cluster Setup:**

```bash
redis-cli --cluster create \
  node1:6379 node2:6379 node3:6379 \
  --cluster-replicas 0 \
  -a <secure_password>
```

### Search: Elasticsearch 8

| Node | Role | Specification |
|------|------|---------------|
| **Node 1** | Master + Data | 4 vCPU, 16 GB RAM, 200 GB SSD |
| **Node 2** | Data | 4 vCPU, 16 GB RAM, 200 GB SSD |
| **Node 3** | Data + Ingest | 4 vCPU, 16 GB RAM, 200 GB SSD |

**elasticsearch.yml:**

```yaml
cluster.name: yemenmart-search
node.name: es-node-1
path.data: /data/elasticsearch
path.logs: /var/log/elasticsearch
network.host: 0.0.0.0
discovery.seed_hosts: ["es-node-1", "es-node-2", "es-node-3"]
cluster.initial_master_nodes: ["es-node-1", "es-node-2", "es-node-3"]
xpack.security.enabled: true
xpack.security.transport.ssl.enabled: true
```

### Storage: MinIO/S3

| Aspect | Detail |
|--------|--------|
| **Product** | MinIO (self-hosted) or AWS S3 |
| **Buckets** | `yemenmart-products`, `yemenmart-uploads`, `yemenmart-backups` |
| **Lifecycle** | Versioning enabled, 90-day expiration on multipart uploads |
| **CORS** | Configured for `yemenmart.com` and `*.yemenmart.com` |

**MinIO Docker Compose (standalone):**

```yaml
minio:
  image: minio/minio:latest
  command: server /data --console-address ":9001"
  environment:
    MINIO_ROOT_USER: ${MINIO_ROOT_USER}
    MINIO_ROOT_PASSWORD: ${MINIO_ROOT_PASSWORD}
  volumes:
    - minio_data:/data
  ports:
    - "9000:9000"
    - "9001:9001"
```

### CDN: Cloudflare

| Setting | Value |
|---------|-------|
| **Plan** | Pro or Business |
| **SSL Mode** | Full (Strict) |
| **Minification** | JS, CSS, HTML enabled |
| **Caching** | Standard (aggressive for static assets) |
| **Page Rules** | Cache everything for `/static/*` |
| **WAF** | Enabled, OWASP ruleset |
| **DDoS** | Enabled (L3/L4/L7) |
| **Rate Limiting** | 100 req/min per IP for `/api/auth/*` |
| **Bot Management** | Enabled |

---

## 3. CI/CD Pipeline

### GitHub Actions Workflow

**File: `.github/workflows/deploy.yml`**

```yaml
name: YemenMart CI/CD Pipeline

on:
  push:
    branches: [develop, main]
  pull_request:
    branches: [develop, main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  # ─── Stage 1: Build & Validate ───────────────────────────
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'
          cache: 'pip'

      - name: Install dependencies
        run: |
          pip install -r requirements/production.txt
          pip install -r requirements/dev.txt

      - name: Run Linter (Ruff)
        run: ruff check .

      - name: Run Formatter Check
        run: ruff format --check .

      - name: Run Type Checker
        run: mypy .

  # ─── Stage 2: Test ───────────────────────────────────────
  test:
    runs-on: ubuntu-latest
    needs: build
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_DB: yemenmart_test
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
      redis:
        image: redis:7
        ports:
          - 6379:6379
      elasticsearch:
        image: elasticsearch:8.11.0
        env:
          discovery.type: single-node
          xpack.security.enabled: "false"
        ports:
          - 9200:9200
    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'
          cache: 'pip'

      - name: Install dependencies
        run: |
          pip install -r requirements/production.txt
          pip install -r requirements/dev.txt

      - name: Run Database Migrations
        env:
          DATABASE_URL: postgresql://test:test@localhost:5432/yemenmart_test
        run: python manage.py migrate

      - name: Run Unit Tests
        env:
          DATABASE_URL: postgresql://test:test@localhost:5432/yemenmart_test
          REDIS_URL: redis://localhost:6379
          ELASTICSEARCH_URL: http://localhost:9200
        run: pytest tests/unit/ -v --tb=short

      - name: Run Integration Tests
        env:
          DATABASE_URL: postgresql://test:test@localhost:5432/yemenmart_test
          REDIS_URL: redis://localhost:6379
          ELASTICSEARCH_URL: http://localhost:9200
        run: pytest tests/integration/ -v --tb=short

      - name: Run API Tests
        run: pytest tests/api/ -v --tb=short

      - name: Generate Coverage Report
        run: pytest --cov=app --cov-report=xml --cov-report=html

      - name: Upload Coverage
        uses: codecov/codecov-action@v3
        with:
          file: coverage.xml
          flags: unittests

  # ─── Stage 3: Security Scan ──────────────────────────────
  security:
    runs-on: ubuntu-latest
    needs: build
    steps:
      - uses: actions/checkout@v4

      - name: Run SAST (Semgrep)
        uses: semgrep/semgrep-action@v1
        with:
          config: p/python

      - name: Run Dependency Scan (Safety)
        run: |
          pip install safety
          safety check -r requirements/production.txt

      - name: Run Secret Scan (Gitleaks)
        uses: gitleaks/gitleaks-action@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      - name: Run Container Scan (Trivy)
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          scan-ref: '.'
          severity: 'CRITICAL,HIGH'

  # ─── Stage 4: Build & Push Container ─────────────────────
  docker:
    runs-on: ubuntu-latest
    needs: [test, security]
    if: github.ref == 'refs/heads/main' || github.ref == 'refs/heads/develop'
    permissions:
      contents: read
      packages: write
    outputs:
      image_tag: ${{ steps.meta.outputs.version }}
    steps:
      - uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Log in to Container Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=sha,prefix=

      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  # ─── Stage 5: Deploy to QA ──────────────────────────────
  deploy-qa:
    runs-on: ubuntu-latest
    needs: docker
    if: github.ref == 'refs/heads/develop'
    environment: QA
    steps:
      - name: Deploy to QA
        run: |
          ssh ${{ secrets.QA_SERVER }} \
            "docker pull ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:develop && \
             cd /opt/yemenmart && \
             docker compose -f docker-compose.qa.yml up -d"

      - name: Run Smoke Tests
        run: |
          sleep 30
          curl -sf https://qa.yemenmart.com/api/health || exit 1
          curl -sf https://qa.yemenmart.com/api/v1/status || exit 1

      - name: Notify QA Team
        uses: slackapi/slack-github-action@v1
        with:
          payload: |
            {"text": "QA deploy complete: ${{ github.sha }}"}
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}

  # ─── Stage 6: Deploy to Staging ─────────────────────────
  deploy-staging:
    runs-on: ubuntu-latest
    needs: docker
    if: github.ref == 'refs/heads/main'
    environment: Staging
    steps:
      - name: Deploy to Staging
        run: |
          ssh ${{ secrets.STAGING_SERVER }} \
            "docker pull ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:main && \
             cd /opt/yemenmart && \
             docker compose -f docker-compose.staging.yml up -d"

      - name: Run Migration
        run: |
          ssh ${{ secrets.STAGING_SERVER }} \
            "cd /opt/yemenmart && docker compose exec api python manage.py migrate --no-input"

      - name: Run Smoke Tests
        run: |
          sleep 30
          curl -sf https://staging.yemenmart.com/api/health || exit 1
          curl -sf https://staging.yemenmart.com/api/v1/status || exit 1

      - name: Run Load Tests
        run: |
          pip install locust
          locust -f tests/load/locustfile.py \
            --host=https://staging.yemenmart.com \
            --users=100 --spawn-rate=10 --run-time=5m \
            --headless --csv=results

      - name: Notify Staging Complete
        uses: slackapi/slack-github-action@v1
        with:
          payload: |
            {"text": "Staging deploy complete: ${{ github.sha }}"}
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}

  # ─── Stage 7: Deploy to Production ──────────────────────
  deploy-production:
    runs-on: ubuntu-latest
    needs: docker
    if: github.ref == 'refs/heads/main'
    environment: Production
    steps:
      - name: Create Deployment Record
        run: |
          echo "Deploying ${{ github.sha }} to production"
          echo "Triggered by: ${{ github.actor }}"

      - name: Deploy to Production (Blue-Green)
        run: |
          # Tag current live as 'previous'
          ssh ${{ secrets.PROD_SERVER }} \
            "cd /opt/yemenmart && \
             docker compose -f docker-compose.prod.yml pull api && \
             docker compose -f docker-compose.prod.yml up -d --no-deps api"

      - name: Run Migration
        run: |
          ssh ${{ secrets.PROD_SERVER }} \
            "cd /opt/yemenmart && \
             docker compose -f docker-compose.prod.yml exec -T api python manage.py migrate --no-input"

      - name: Health Check
        run: |
          for i in $(seq 1 10); do
            STATUS=$(curl -s -o /dev/null -w "%{http_code}" https://yemenmart.com/api/health)
            if [ "$STATUS" = "200" ]; then
              echo "Health check passed"
              exit 0
            fi
            echo "Attempt $i: status $STATUS, retrying in 10s..."
            sleep 10
          done
          echo "Health check failed"
          exit 1

      - name: Notify Production Deploy
        uses: slackapi/slack-github-action@v1
        with:
          payload: |
            {"text": "PRODUCTION deploy complete: ${{ github.sha }} by ${{ github.actor }}"}
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}
```

### Pipeline Summary

```
┌─────────┐    ┌──────┐    ┌──────────┐    ┌─────────┐    ┌───────────┐    ┌──────────┐    ┌────────────┐
│  Build   │───▶│ Test │───▶│ Security │───▶│ Docker  │───▶│ Deploy QA │───▶│ Staging  │───▶│ Production │
│ (lint,   │    │(unit,│    │(SAST,    │    │ (build, │    │ (auto)    │    │ (auto)   │    │ (manual    │
│  type)   │    │ int) │    │ deps)    │    │  push)  │    │           │    │          │    │  approval) │
└─────────┘    └──────┘    └──────────┘    └─────────┘    └───────────┘    └──────────┘    └────────────┘
                                                                │                │               │
                                                              (develop)       (main)         (main +
                                                                                             approval)
```

---

## 4. Deployment Process

### Blue-Green Deployment

**Strategy:** Two identical environments (Blue and Green). Only one serves production traffic at a time.

**Implementation:**

```yaml
# docker-compose.blue-green.yml
version: '3.8'

services:
  # Blue environment (current live)
  api-blue:
    image: ${REGISTRY}/yemenmart:${BLUE_TAG}
    environment:
      - APP_COLOR=blue
      - DATABASE_URL=${DATABASE_URL}
    deploy:
      replicas: 2

  # Green environment (next deployment)
  api-green:
    image: ${REGISTRY}/yemenmart:${GREEN_TAG}
    environment:
      - APP_COLOR=green
      - DATABASE_URL=${DATABASE_URL}
    deploy:
      replicas: 0  # Scaled down until swap

  nginx:
    image: nginx:alpine
    volumes:
      - ./nginx/blue-green.conf:/etc/nginx/nginx.conf:ro
    ports:
      - "80:80"
      - "443:443"
```

**Switch Procedure:**

```bash
#!/bin/bash
# swap-environments.sh

CURRENT=$(docker compose ps api-blue --format '{{.Status}}' | head -1)
if echo "$CURRENT" | grep -q "Up"; then
  LIVE="blue"
  DEPLOY="green"
else
  LIVE="blue"
  DEPLOY="green"
fi

echo "Swapping: $LIVE → $DEPLOY"

# Scale up new environment
docker compose up -d --scale api-$DEPLOY=2 --scale api-$LIVE=0 api-$DEPLOY

# Update nginx upstream
sed -i "s/api-$LIVE/api-$DEPLOY/g" nginx/blue-green.conf
docker compose exec nginx nginx -s reload

echo "Swap complete. $DEPLOY is now live."
```

### Zero-Downtime Migrations

**Principle:** Never lock tables in production. Use backward-compatible migrations.

**Strategy:**

1. **Expand schema** (add columns/tables) — non-breaking
2. **Deploy code** that writes to new columns + reads from old
3. **Backfill data** — background job
4. **Deploy code** that reads from new columns
5. **Contract schema** (remove old columns) — non-breaking

**Migration Safety Checklist:**

```markdown
- [ ] Migration is reversible
- [ ] Does not lock tables for >1s
- [ ] New column has DEFAULT or is nullable
- [ ] Backfill script provided for data migrations
- [ ] Tested on production-size dataset (1M+ rows)
- [ ] Rollback tested
```

**Alembic/Django Configuration:**

```python
# Django settings
MIGRATION_MODULES = {
    'products': 'apps.products.migrations',
}

# For large tables, use online schema changes
# pip install django-online-migrations
DATABASE_ROUTERS = ['routers.PrimaryReplicaRouter']
```

**Online Schema Change Example:**

```python
# migrations/0045_add_price_cents.py
from django.db import migrations

class Migration(migrations.Migration):
    atomic = False  # Non-atomic for online changes

    dependencies = [
        ('products', '0044_previous'),
    ]

    operations = [
        migrations.AddField(
            model_name='product',
            name='price_cents',
            field=models.BigIntegerField(null=True),
        ),
    ]
```

### Feature Flags

**Implementation: Using Django-Waffle or Unleash**

```python
# Feature flag checks in code
from waffle import flag_is_active

def product_list_view(request):
    if flag_is_active(request, 'new_product_search'):
        return new_search_implementation(request)
    return legacy_search_implementation(request)

# API endpoint for frontend
@api_view(['GET'])
def feature_flags(request):
    return Response({
        'new_checkout': flag_is_active(request, 'new_checkout'),
        'dark_mode': flag_is_active(request, 'dark_mode'),
        'arabic_only': flag_is_active(request, 'arabic_only'),
    })
```

**Flag Lifecycle:**

```
Created → Development → QA → Staging → Production (1%) → Production (100%) → Permanent
```

### Canary Releases

**Strategy:** Roll out to 5% → 25% → 50% → 100% of users over 24-48 hours.

**Nginx Canary Configuration:**

```nginx
upstream yemenmart_backend {
    server api-stable:8000 weight=95;
    server api-canary:8000 weight=5;
}

# Monitor canary metrics
# If error rate >1% or p95 >500ms, auto-rollback
```

**Canary Monitoring Script:**

```python
# canary_monitor.py
import requests
import time

CANARY_THRESHOLD_ERROR_RATE = 0.01
CANARY_THRESHOLD_LATENCY = 0.5  # seconds

def check_canary():
    # Query Prometheus for canary metrics
    error_rate = query_prometheus(
        'rate(http_requests_total{version="canary",status=~"5.."}[5m]) / '
        'rate(http_requests_total{version="canary"}[5m])'
    )
    p95_latency = query_prometheus(
        'histogram_quantile(0.95, rate(http_request_duration_seconds_bucket{version="canary"}[5m]))'
    )

    if error_rate > CANARY_THRESHOLD_ERROR_RATE:
        rollback_canary()
        alert("Canary rolled back: error rate exceeded threshold")
        return False

    if p95_latency > CANARY_THRESHOLD_LATENCY:
        rollback_canary()
        alert("Canary rolled back: latency exceeded threshold")
        return False

    return True
```

---

## 5. Monitoring & Alerting

### Application Monitoring: Prometheus + Grafana

**Prometheus Configuration:**

```yaml
# prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

alerting:
  alertmanagers:
    - static_configs:
        - targets: ['alertmanager:9093']

rule_files:
  - 'rules/*.yml'

scrape_configs:
  - job_name: 'yemenmart-api'
    static_configs:
      - targets: ['api-server-1:8000', 'api-server-2:8000']
    metrics_path: '/metrics'

  - job_name: 'postgres'
    static_configs:
      - targets: ['postgres-primary:9187']

  - job_name: 'redis'
    static_configs:
      - targets: ['redis-cluster:9121']

  - job_name: 'elasticsearch'
    static_configs:
      - targets: ['elasticsearch:9114']

  - job_name: 'nginx'
    static_configs:
      - targets: ['nginx:9113']
```

**Key Metrics (RED Method):**

| Metric | Description | Alert Threshold |
|--------|-------------|-----------------|
| **Rate** | Requests per second | Drop >50% from baseline |
| **Errors** | Error rate (5xx) | >1% over 5 min |
| **Duration** | Request latency p95 | >500ms over 5 min |

**Prometheus Alert Rules:**

```yaml
# rules/yemenmart.yml
groups:
  - name: yemenmart
    rules:
      - alert: HighErrorRate
        expr: rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m]) > 0.01
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High error rate on {{ $labels.instance }}"
          description: "Error rate is {{ $value | humanizePercentage }}"

      - alert: HighLatency
        expr: histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m])) > 0.5
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High latency on {{ $labels.instance }}"

      - alert: DatabaseConnectionPoolExhausted
        expr: pg_stat_activity_count > pg_settings_max_connections * 0.8
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "Database connection pool nearly exhausted"

      - alert: RedisMemoryHigh
        expr: redis_memory_used_bytes / redis_memory_max_bytes > 0.85
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Redis memory usage above 85%"

      - alert: ElasticsearchClusterRed
        expr: elasticsearch_cluster_health_status{color="red"} == 1
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Elasticsearch cluster status is RED"
```

**Grafana Dashboards:**

| Dashboard | Panels |
|-----------|--------|
| **API Overview** | Request rate, error rate, latency p50/p95/p99, active connections |
| **Database** | Connections, query duration, cache hit ratio, replication lag, deadlocks |
| **Redis** | Memory usage, hit rate, connected clients, command latency, cluster status |
| **Elasticsearch** | Index rate, search latency, cluster health, JVM heap, disk usage |
| **Infrastructure** | CPU, memory, disk I/O, network I/O per server |

### Logging: ELK Stack

**Filebeat Configuration (on each server):**

```yaml
# filebeat.yml
filebeat.inputs:
  - type: filestream
    paths:
      - /var/log/yemenmart/access.log
    parsers:
      - ndjson:
          target: ""
    fields:
      service: yemenmart-api
      environment: production

  - type: filestream
    paths:
      - /var/log/yemenmart/error.log
    parsers:
      - ndjson:
          target: ""
    fields:
      service: yemenmart-api
      environment: production

output.elasticsearch:
  hosts: ["elasticsearch:9200"]
  index: "yemenmart-%{+yyyy.MM.dd}"

setup.template.name: "yemenmart"
setup.template.pattern: "yemenmart-*"
```

**Structured Logging Format:**

```json
{
  "timestamp": "2026-09-13T10:30:00Z",
  "level": "info",
  "service": "yemenmart-api",
  "trace_id": "abc123",
  "method": "GET",
  "path": "/api/v1/products",
  "status": 200,
  "duration_ms": 45,
  "user_id": "usr_12345",
  "request_id": "req_xyz789"
}
```

**Kibana Saved Views:**

| View | Description |
|------|-------------|
| **Error Dashboard** | All 5xx errors, grouped by endpoint and error type |
| **Slow Queries** | Requests >500ms, SQL queries >100ms |
| **Security Events** | Failed logins, rate limit hits, suspicious patterns |
| **User Activity** | Audit trail for order/payment changes |

### APM: Sentry

**Configuration:**

```python
# settings.py
import sentry_sdk
from sentry_sdk.integrations.django import DjangoIntegration
from sentry_sdk.integrations.redis import RedisIntegration
from sentry_sdk.integrations.postgres import PostgresIntegration

sentry_sdk.init(
    dsn="https://xxx@sentry.io/yyy",
    environment="production",
    traces_sample_rate=0.1,  # 10% of transactions
    profiles_sample_rate=0.1,
    integrations=[
        DjangoIntegration(),
        RedisIntegration(),
        PostgresIntegration(),
    ],
    beforeSend=filter_pii,  # Strip sensitive data
)

def filter_pii(event, hint):
    if 'exception' in event:
        for exc in event['exception'].get('values', []):
            for frame in exc.get('stacktrace', {}).get('frames', []):
                frame.pop('vars', None)
    return event
```

**Sentry Alert Rules:**

| Alert | Condition | Action |
|-------|-----------|--------|
| New Error | First occurrence of error type | Slack #alerts, email on-call |
| Regression | Error reappears after being resolved | Slack #alerts |
| Performance | Transaction duration >2s p95 | Slack #performance |
| Release Health | Crash-free rate <95% | Slack #releases, page on-call |

### Uptime Monitoring: UptimeRobot

| Check | URL | Interval | Alert |
|-------|-----|----------|-------|
| **Homepage** | `https://yemenmart.com` | 1 min | Email + SMS + Slack |
| **API Health** | `https://yemenmart.com/api/health` | 1 min | Email + SMS + Slack |
| **API Products** | `https://yemenmart.com/api/v1/products` | 5 min | Email + Slack |
| **SSL Certificate** | `yemenmart.com` | 1 day | Email (30 days before expiry) |
| **DNS Check** | `yemenmart.com` | 5 min | Email + SMS |

---

## 6. Backup & Recovery

### Database Backups

**Strategy:**

| Type | Frequency | Retention | Storage |
|------|-----------|-----------|---------|
| **Full Backup** | Daily 02:00 UTC | 30 days | S3 + local |
| **WAL Archiving** | Continuous | 7 days | S3 |
| **Logical Dump** | Weekly (Sunday) | 90 days | S3 + offsite |
| **Snapshot** | On each migration | 7 days | S3 |

**Backup Script:**

```bash
#!/bin/bash
# backup-database.sh

set -euo pipefail

TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/backups/postgres"
S3_BUCKET="s3://yemenmart-backups/database"

# Full physical backup
pg_basebackup \
  -h $PGHOST \
  -U $PGUSER \
  -D "${BACKUP_DIR}/full_${TIMESTAMP}" \
  -Ft \
  -z \
  -P \
  --wal-method=stream

# Compress
tar -czf "${BACKUP_DIR}/full_${TIMESTAMP}.tar.gz" \
  -C "${BACKUP_DIR}" "full_${TIMESTAMP}"

# Upload to S3
aws s3 cp "${BACKUP_DIR}/full_${TIMESTAMP}.tar.gz" \
  "${S3_BUCKET}/full/${TIMESTAMP}.tar.gz" \
  --storage-class STANDARD_IA

# Cleanup old backups
find ${BACKUP_DIR} -name "*.tar.gz" -mtime +30 -delete

# Verify backup integrity
pg_restore -l "${BACKUP_DIR}/full_${TIMESTAMP}/backup_manifest" > /dev/null

echo "Backup complete: full_${TIMESTAMP}"
```

**Cron Schedule:**

```cron
# Daily full backup at 02:00 UTC
0 2 * * * /opt/yemenmart/scripts/backup-database.sh >> /var/log/yemenmart/backup.log 2>&1

# Weekly logical dump on Sunday at 03:00 UTC
0 3 * * 0 pg_dump -Fc yemenmart > /backups/postgres/logical_$(date +\%Y\%m\%d).dump

# Clean old WAL archives daily at 04:00 UTC
0 4 * * * find /archive/wal -mtime +7 -delete
```

### WAL Archiving (Point-in-Time Recovery)

**postgresql.conf:**

```ini
wal_level = replica
archive_mode = on
archive_command = 'aws s3 cp %p s3://yemenmart-backups/wal/%f'
archive_timeout = 300  # Force archive every 5 minutes
```

**Recovery Procedure:**

```bash
# 1. Stop PostgreSQL
systemctl stop postgresql

# 2. Restore base backup
rm -rf /var/lib/postgresql/15/main/*
tar -xzf /backups/postgres/full_YYYYMMDD.tar.gz -C /var/lib/postgresql/15/main/

# 3. Configure recovery
cat > /var/lib/postgresql/15/main/postgresql.auto.conf << EOF
restore_command = 'aws s3 cp s3://yemenmart-backups/wal/%f %p'
recovery_target_time = '2026-09-13 10:30:00 UTC'
recovery_target_action = 'promote'
EOF

touch /var/lib/postgresql/15/main/recovery.signal

# 4. Start PostgreSQL (enters recovery mode)
systemctl start postgresql

# 5. Verify
psql -c "SELECT pg_is_in_recovery();"
# Should return: false
```

### RPO & RTO Targets

| Metric | Target | Strategy |
|--------|--------|----------|
| **RPO** (Recovery Point Objective) | 5 minutes | WAL archiving every 5 min, streaming replication |
| **RTO** (Recovery Time Objective) | 1 hour | Automated failover, pre-staged infrastructure |

**RPO Verification:**

```bash
# Check replication lag
psql -c "SELECT now() - pg_last_xact_replay_timestamp() AS replication_lag;"
# Must be <5 minutes
```

**RTO Verification:**

```bash
# Time backup restoration test (monthly)
time (
  pg_basebackup ... # simulate restore
  pg_wal_replay ...
) 
# Must complete in <60 minutes
```

### Backup Testing Schedule

| Test | Frequency | Duration | Success Criteria |
|------|-----------|----------|------------------|
| **Restore Test** | Monthly | ~30 min | DB restored, queries work |
| **Full DR Drill** | Quarterly | ~2 hours | Complete failover successful |
| **WAL Replay Test** | Monthly | ~15 min | PITR to specific timestamp |
| **S3 Restore Test** | Monthly | ~20 min | Files accessible, checksums valid |

---

## 7. Security

### HTTPS Everywhere

**Certificate Management:**

```bash
# Initial setup with Certbot
certbot certonly --nginx -d yemenmart.com -d www.yemenmart.com

# Auto-renewal cron
0 12 * * * certbot renew --quiet --post-hook "systemctl reload nginx"
```

**HSTS Configuration:**

```nginx
add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload" always;
```

**Certificate Monitoring:**

```python
# Alert when certificate expires in <14 days
import ssl
import socket
from datetime import datetime

def check_cert_expiry(domain):
    context = ssl.create_default_context()
    with socket.create_connection((domain, 443)) as sock:
        with context.wrap_socket(sock, server_hostname=domain) as ssock:
            cert = ssock.getpeercert()
            expiry = datetime.strptime(cert['notAfter'], '%b %d %H:%M:%S %Y %Z')
            days_left = (expiry - datetime.now()).days
            if days_left < 14:
                alert(f"SSL cert for {domain} expires in {days_left} days")
            return days_left
```

### Secrets Management

**Strategy: HashiCorp Vault (or AWS Secrets Manager)**

```python
# settings.py - Fetch secrets from Vault
import hvac

client = hvac.Client(url='https://vault.yemenmart.com:8200')
client.token = os.environ['VAULT_TOKEN']

# Read secrets
db_secret = client.secrets.kv.v2.read_secret_version(path='database/prod')
redis_secret = client.secrets.kv.v2.read_secret_version(path='redis/prod')
api_secret = client.secrets.kv.v2.read_secret_version(path='api-keys/prod')

DATABASE_URL = db_secret['data']['data']['url']
REDIS_URL = redis_secret['data']['data']['url']
SECRET_KEY = api_secret['data']['data']['django-secret']
```

**Secret Rotation Schedule:**

| Secret | Rotation Frequency | Method |
|--------|-------------------|--------|
| Database password | 90 days | Automated via Vault |
| API keys | 90 days | Manual + Vault |
| Django SECRET_KEY | On compromise | Manual |
| Redis password | 90 days | Automated via Vault |
| JWT signing key | 180 days | Manual + Vault |
| SSL certificates | Auto (90 days) | Certbot |

### WAF Rules (Cloudflare)

**Custom Rules:**

```
# Block SQL injection attempts
(http.request.uri contains "union" and http.request.uri contains "select") or
(http.request.uri contains "1=1" and http.request.uri contains "or") or
(http.request.uri contains "drop table")

# Block XSS attempts
(http.request.uri contains "<script") or
(http.request.uri contains "javascript:")

# Block path traversal
(http.request.uri contains "../") or (http.request.uri contains "..%2f")

# Rate limit authentication endpoints
(http.request.uri.path eq "/api/auth/login" and cf.bot_management.score lt 30)
```

**Rate Limiting Rules:**

| Endpoint | Limit | Window | Action |
|----------|-------|--------|--------|
| `/api/auth/login` | 5 requests | 1 minute | Block 10 min |
| `/api/auth/register` | 3 requests | 1 hour | Block 1 hour |
| `/api/orders` | 10 requests | 1 minute | Challenge |
| `/api/*` (general) | 100 requests | 1 minute | Throttle |

### DDoS Protection

**Layer 3/4:** Cloudflare always-on DDoS protection
**Layer 7:** Cloudflare WAF + rate limiting

**Additional Measures:**

```nginx
# Nginx rate limiting
limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;
limit_req_zone $binary_remote_addr zone=auth:10m rate=1r/s;

server {
    location /api/ {
        limit_req zone=api burst=20 nodelay;
    }

    location /api/auth/ {
        limit_req zone=auth burst=5 nodelay;
    }
}
```

### Security Audit Checklist

```markdown
## Pre-Deployment Security Review

- [ ] No secrets in code or environment files
- [ ] All API endpoints require authentication
- [ ] Input validation on all user-facing endpoints
- [ ] SQL injection protection (ORM usage verified)
- [ ] XSS protection (escape output, CSP headers)
- [ ] CSRF protection enabled
- [ ] Rate limiting configured on auth endpoints
- [ ] HTTPS enforced, HSTS enabled
- [ ] Security headers configured
- [ ] Dependency vulnerabilities checked (safety, trivy)
- [ ] Container images scanned for CVEs
- [ ] Database access restricted to app servers only
- [ ] Redis/Elasticsearch not exposed to public
- [ ] Logging excludes PII
- [ ] Backup encryption verified
```

---

## 8. Rollback Procedures

### Decision Matrix

| Issue | Severity | Action | Timeline |
|-------|----------|--------|----------|
| Cosmetic bug | Low | Hotfix in next release | Next sprint |
| Feature broken | Medium | Feature flag disable | <15 minutes |
| API errors >5% | High | Application rollback | <30 minutes |
| Data corruption | Critical | DB rollback + app rollback | <1 hour |
| Full outage | Critical | Full rollback + DR activation | <1 hour |

### Database Migration Rollback

**Before Deployment:**

```bash
# Create pre-migration backup
pg_dump -Fc yemenmart > /backups/pre_migration_$(date +%Y%m%d_%H%M%S).dump

# Record current migration state
python manage.py showmigrations > /backups/migration_state_$(date +%Y%m%d_%H%M%S).txt
```

**Rollback Migration:**

```bash
# Option 1: Rollback last migration
python manage.py migrate app_name previous_migration_name

# Option 2: Rollback multiple migrations
python manage.py migrate app_name 0044  # Rollback to migration 0044

# Option 3: Restore from backup (nuclear option)
systemctl stop yemenmart
pg_restore -d yemenmart /backups/pre_migration_YYYYMMDD.dump --clean
systemctl start yemenmart
```

**Safe Rollback Checklist:**

```markdown
## Migration Rollback Readiness

- [ ] Migration is reversible (has `reverse()` method)
- [ ] No data loss on rollback
- [ ] Code is backward-compatible with rolled-back schema
- [ ] Feature flags can disable new functionality
- [ ] Backup created before migration applied
- [ ] Rollback tested in staging
```

### Application Rollback

**Docker Rollback:**

```bash
# List recent images
docker images yemenmart-api --format "{{.Tag}} {{.CreatedAt}}"

# Rollback to previous version
PREVIOUS_TAG="v1.2.3"
docker pull ${REGISTRY}/yemenmart:${PREVIOUS_TAG}
docker compose -f docker-compose.prod.yml up -d --no-deps api

# Verify health
curl -sf https://yemenmart.com/api/health
```

**Git Rollback:**

```bash
# Revert last commit
git revert HEAD
git push origin main

# Revert to specific commit
git revert abc1234
git push origin main
```

**Rollback Script:**

```bash
#!/bin/bash
# rollback.sh

set -euo pipefail

PREVIOUS_TAG=${1:?Usage: ./rollback.sh <previous-tag>}

echo "=== YEMENMART ROLLBACK ==="
echo "Rolling back to: ${PREVIOUS_TAG}"
echo "Timestamp: $(date -u +%Y-%m-%dT%H:%M:%SZ)"

# 1. Pull previous image
echo "Pulling image..."
docker pull ${REGISTRY}/yemenmart:${PREVIOUS_TAG}

# 2. Run pre-rollback migration if needed
echo "Running rollback migrations..."
docker compose -f docker-compose.prod.yml exec api python manage.py migrate --reverse

# 3. Deploy previous version
echo "Deploying previous version..."
PREVIOUS_TAG=${PREVIOUS_TAG} docker compose -f docker-compose.prod.yml up -d api

# 4. Wait for health check
echo "Waiting for health check..."
for i in $(seq 1 20); do
    if curl -sf https://yemenmart.com/api/health > /dev/null; then
        echo "Health check passed"
        break
    fi
    if [ $i -eq 20 ]; then
        echo "HEALTH CHECK FAILED - MANUAL INTERVENTION REQUIRED"
        exit 1
    fi
    sleep 5
done

# 5. Verify no errors
sleep 30
ERROR_RATE=$(curl -s "http://prometheus:9090/api/v1/query?query=rate(http_requests_total{status=~'5..'}[5m])" | jq '.data.result[0].value[1]')
echo "Current error rate: ${ERROR_RATE}"

# 6. Notify team
curl -X POST ${SLACK_WEBHOOK} \
    -H 'Content-type: application/json' \
    -d "{\"text\":\"ROLLBACK COMPLETE: ${PREVIOUS_TAG} is now live. Triggered by: $(whoami)\"}"

echo "=== ROLLBACK COMPLETE ==="
```

### Communication Plan

**Rollback Notification Template:**

```
INCIDENT: YemenMart Rollback
SEVERITY: [P1/P2/P3]
AFFECTED: [Service/Feature]
DURATION: [Start time] - [End time]
ACTION: Rolling back from [bad version] to [good version]
IMPACT: [User-facing impact description]
NEXT STEPS:
1. Monitor for 30 minutes
2. Investigate root cause
3. Fix in development
4. Re-deploy when ready

ON-CALL: [Name]
UPDATED: [Timestamp]
```

**Communication Channels:**

| Channel | Audience | When |
|---------|----------|------|
| **#incidents** (Slack) | Engineering team | All rollbacks |
| **#status** (Slack) | All employees | P1/P2 incidents |
| **Status page** | Customers | Service degradation |
| **Email** | Stakeholders | P1 incidents |
| **SMS** | On-call engineers | P1 incidents |

**Status Page Updates:**

```
[Investigating] We are aware of issues with [service]. 
Our team is actively working on a resolution.

[Identified] The issue has been identified. 
We are rolling back to a previous stable version.

[Monitoring] The rollback is complete. 
We are monitoring for stability.

[Resolved] The issue has been resolved. 
All services are operating normally.
```

### Post-Rollback Review

```markdown
## Rollback Review Template

**Date:** YYYY-MM-DD
**Rollback by:** [Name]
**Duration:** [X minutes]
**Trigger:** [What caused the rollback]

### Root Cause
- [Description of what went wrong]

### Timeline
- HH:MM - Deploy started
- HH:MM - Issue detected
- HH:MM - Rollback initiated
- HH:MM - Rollback complete
- HH:MM - Services verified healthy

### Impact
- Users affected: [Number/Percentage]
- Revenue impact: [Estimated]
- Data loss: [Yes/No, details]

### Action Items
- [ ] Fix root cause in development
- [ ] Add test coverage for edge case
- [ ] Update monitoring/alerts
- [ ] Update runbook
- [ ] Schedule post-mortem meeting
```

---

## Appendix

### Quick Reference Commands

```bash
# Health check
curl -sf https://yemenmart.com/api/health

# View logs
docker compose -f docker-compose.prod.yml logs -f api

# Check database replication
psql -c "SELECT now() - pg_last_xact_replay_timestamp() AS lag;"

# Check Redis cluster
redis-cli -c cluster info

# Trigger manual backup
/opt/yemenmart/scripts/backup-database.sh

# Force deployment
gh workflow run deploy.yml -f ref=main

# Emergency rollback
./rollback.sh v1.2.3
```

### Contact List

| Role | Name | Phone | Email |
|------|------|-------|-------|
| **DevOps Lead** | TBD | TBD | TBD |
| **Backend Lead** | TBD | TBD | TBD |
| **Database Admin** | TBD | TBD | TBD |
| **Security Lead** | TBD | TBD | TBD |
| **On-Call Engineer** | TBD | TBD | TBD |

---

*Last updated: 2026-09-13*
*Version: 1.0*
*Owner: DevOps Team*
