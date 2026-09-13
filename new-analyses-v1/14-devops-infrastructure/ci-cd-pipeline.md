# CI/CD Pipeline - YemenMart

## Document Information
| Property | Value |
|----------|-------|
| Document ID | YEM-INF-CICD-001 |
| Version | 1.0 |
| Status | Active |
| Last Updated | 2026-09-12 |

---

## 1. Pipeline Overview

### 1.1 Pipeline Architecture
```
┌─────────────────────────────────────────────────────────────────┐
│                     GitHub Actions CI/CD                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐   │
│  │  Commit  │──▶│   Build  │──▶│   Test   │──▶│  Deploy  │   │
│  │          │   │          │   │          │   │          │   │
│  │ - Lint   │   │ - Docker │   │ - Unit   │   │ - Dev    │   │
│  │ - Format │   │ - Push   │   │ - Integ  │   │ - QA     │   │
│  │ - Check  │   │ - Sign   │   │ - E2E    │   │ - Stage  │   │
│  │          │   │          │   │ - Security│  │ - Prod   │   │
│  └──────────┘   └──────────┘   └──────────┘   └──────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 1.2 Pipeline Goals
| Goal | Target |
|------|--------|
| Build Time | < 10 minutes |
| Test Time | < 15 minutes |
| Deploy to Dev | < 5 minutes |
| Deploy to Production | < 15 minutes |
| Pipeline Success Rate | > 95% |
| Mean Time to Deploy | < 30 minutes |

---

## 2. GitHub Actions Configuration

### 2.1 Repository Structure
```
yemenmart/
├── .github/
│   ├── workflows/
│   │   ├── ci.yml              # Main CI pipeline
│   │   ├── cd-dev.yml          # Dev deployment
│   │   ├── cd-staging.yml      # Staging deployment
│   │   ├── cd-prod.yml         # Production deployment
│   │   ├── security-scan.yml   # Security scanning
│   │   └── release.yml         # Release automation
│   ├── actions/
│   │   ├── build/action.yml    # Build action
│   │   ├── test/action.yml     # Test action
│   │   └── deploy/action.yml   # Deploy action
│   └── dependabot.yml          # Dependency updates
├── docker/
│   ├── Dockerfile              # Main Dockerfile
│   ├── Dockerfile.dev          # Dev Dockerfile
│   └── docker-compose.yml      # Local development
├── helm/
│   ├── yemenmart/
│   │   ├── Chart.yaml
│   │   ├── values.yaml
│   │   └── templates/
│   └── environments/
│       ├── dev/
│       ├── staging/
│       └── prod/
└── terraform/
    ├── modules/
    └── environments/
```

### 2.2 Main CI Workflow
```yaml
# .github/workflows/ci.yml
name: CI Pipeline

on:
  push:
    branches: [main, develop, 'feature/*']
  pull_request:
    branches: [main, develop]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  lint-and-format:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npm run lint
      - run: npm run format:check
      - run: npm run typecheck

  unit-tests:
    runs-on: ubuntu-latest
    needs: lint-and-format
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npm run test:unit -- --coverage
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          token: ${{ secrets.CODECOV_TOKEN }}

  integration-tests:
    runs-on: ubuntu-latest
    needs: lint-and-format
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_DB: yemenmart_test
          POSTGRES_PASSWORD: test
        ports: ['5432:5432']
      redis:
        image: redis:7
        ports: ['6379:6379']
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npm run test:integration
        env:
          DATABASE_URL: postgresql://postgres:test@localhost:5432/yemenmart_test
          REDIS_URL: redis://localhost:6379

  security-scan:
    runs-on: ubuntu-latest
    needs: lint-and-format
    steps:
      - uses: actions/checkout@v4
      - name: Run Snyk security scan
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          scan-ref: '.'

  build:
    runs-on: ubuntu-latest
    needs: [unit-tests, integration-tests, security-scan]
    permissions:
      contents: read
      packages: write
    steps:
      - uses: actions/checkout@v4
      - name: Login to GHCR
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: |
            ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
            ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:latest
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

### 2.3 Deployment Workflows
```yaml
# .github/workflows/cd-prod.yml
name: Deploy to Production

on:
  workflow_run:
    workflows: ["CI Pipeline"]
    types: [completed]
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    if: ${{ github.event.workflow_run.conclusion == 'success' }}
    environment: production
    steps:
      - uses: actions/checkout@v4
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::ACCOUNT:role/GitHubActions
          aws-region: me-south-1
      - name: Deploy with ArgoCD
        uses: argoproj/argo-cd-action@v2
        with:
          server: https://argocd.yemenmart.com
          auth-token: ${{ secrets.ARGOCD_TOKEN }}
          command: argo app sync yemenmart-prod
```

---

## 3. Build Process

### 3.1 Docker Build
| Stage | Purpose | Base Image |
|-------|---------|------------|
| Dependencies | Install npm dependencies | node:20-alpine |
| Build | TypeScript compilation | Dependencies stage |
| Production | Final production image | node:20-alpine |

### 3.2 Dockerfile
```dockerfile
# Stage 1: Dependencies
FROM node:20-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# Stage 2: Build
FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 3: Production
FROM node:20-alpine AS production
WORKDIR /app
RUN addgroup -g 1001 -S appgroup && \
    adduser -S appuser -u 1001 -G appgroup
COPY --from=deps /app/node_modules ./node_modules
COPY --from=build /app/dist ./dist
COPY --from=build /app/package.json ./
USER appuser
EXPOSE 8080
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:8080/health || exit 1
CMD ["node", "dist/index.js"]
```

### 3.3 Build Optimization
| Strategy | Implementation |
|----------|---------------|
| Multi-stage Build | Separate build and runtime stages |
| Layer Caching | Cache dependencies layer |
| Docker BuildKit | Parallel builds |
| .dockerignore | Exclude unnecessary files |
| Image Scanning | Trivy scan before push |

---

## 4. Testing Strategy

### 4.1 Test Types in Pipeline
| Type | Trigger | Duration | Coverage Gate |
|------|---------|----------|---------------|
| Lint | Every push | < 1 min | 100% pass |
| Unit Tests | Every push | < 5 min | > 80% |
| Integration Tests | Every PR | < 10 min | > 70% |
| E2E Tests | Nightly | < 30 min | > 60% |
| Security Scan | Every PR | < 5 min | 0 critical |
| Performance | Weekly | < 1 hour | No regression |

### 4.2 Test Reporting
| Report | Tool | Format |
|--------|------|--------|
| Coverage | Istanbul/nyc | HTML + LCOV |
| Unit Results | Jest/pytest | JUnit XML |
| Integration Results | Jest/pytest | JUnit XML |
| E2E Results | Cypress | HTML + Video |
| Security Results | Snyk/Trivy | JSON + Summary |

---

## 5. Deployment Strategy

### 5.1 Deployment Environments
| Environment | Trigger | Strategy | Approval |
|------------|---------|----------|----------|
| Development | Push to develop | Rolling update | Automatic |
| QA | Push to release/* | Rolling update | Automatic |
| Staging | Push to main | Rolling update | Automatic |
| Production | Manual trigger | Blue-Green | Required |

### 5.2 Deployment Process
| Step | Action | Validation |
|------|--------|------------|
| 1 | Build Docker image | Image scan pass |
| 2 | Push to ECR | Image available |
| 3 | Update Helm values | Values valid |
| 4 | ArgoCD sync | Sync successful |
| 5 | Health check | All pods healthy |
| 6 | Smoke tests | Tests pass |
| 7 | Monitor | No errors for 5 min |

### 5.3 Rollback Strategy
| Scenario | Method | Time |
|----------|--------|------|
| Failed deployment | ArgoCD rollback | < 1 min |
| Health check failure | Auto-rollback | < 5 min |
| Error rate spike | Manual rollback | < 5 min |
| Data issue | Point-in-time recovery | < 15 min |

---

## 6. GitOps Configuration

### 6.1 ArgoCD Setup
| Component | Configuration |
|-----------|--------------|
| Repository | GitHub (yemenmart-infra) |
| Sync Policy | Automated, self-heal |
| Auto-prune | Disabled (manual) |
| Server | https://argocd.yemenmart.com |

### 6.2 Helm Charts
| Chart | Version | Description |
|-------|---------|-------------|
| yemenmart | 1.x | Main application chart |
| yemenmart-infra | 1.x | Infrastructure chart |
| yemenmart-monitoring | 1.x | Monitoring stack chart |

### 6.3 Values Management
| Environment | File | Overrides |
|-------------|------|-----------|
| Dev | values-dev.yaml | Replicas, resources |
| QA | values-qa.yaml | Replicas, resources |
| Staging | values-staging.yaml | Production mirror |
| Production | values-prod.yaml | Full configuration |

---

## 7. Security in Pipeline

### 7.1 Security Gates
| Gate | Tool | Block Criteria |
|------|------|---------------|
| SAST | SonarQube | Critical/High issues |
| DAST | OWASP ZAP | Critical vulnerabilities |
| Dependency | Snyk | Critical CVEs |
| Container | Trivy | Critical vulnerabilities |
| Secret | GitGuardian | Any secret detected |
| License | FOSSA | Copyleft licenses |

### 7.2 Secret Management
| Secret Type | Storage | Rotation |
|------------|---------|----------|
| AWS Credentials | GitHub OIDC | Automatic |
| Database Passwords | AWS Secrets Manager | 90 days |
| API Keys | AWS Secrets Manager | 90 days |
| TLS Certificates | ACM | Automatic |
| GitHub Token | GitHub Secrets | Manual |

---

## 8. Release Management

### 8.1 Versioning Strategy
| Type | Format | Example |
|------|--------|---------|
| Semantic Version | MAJOR.MINOR.PATCH | 1.2.3 |
| Pre-release | MAJOR.MINOR.PATCH-rc.N | 1.2.3-rc.1 |
| Build Metadata | MAJOR.MINOR.PATCH+SHA | 1.2.3+abc123 |

### 8.2 Release Process
| Step | Action | Owner |
|------|--------|-------|
| 1 | Create release branch | Developer |
| 2 | Bump version | Release Manager |
| 3 | Update CHANGELOG | Release Manager |
| 4 | Create PR | Developer |
| 5 | Review + Approve | Team Lead |
| 6 | Merge to main | Release Manager |
| 7 | Create GitHub Release | Release Manager |
| 8 | Deploy to staging | Automated |
| 9 | QA validation | QA Team |
| 10 | Deploy to production | Release Manager |

### 8.3 Release Schedule
| Release Type | Frequency | Example |
|-------------|-----------|---------|
| Patch | As needed | Bug fixes |
| Minor | Bi-weekly | New features |
| Major | Quarterly | Breaking changes |
| Hotfix | Immediate | Critical issues |

---

## 9. Pipeline Monitoring

### 9.1 Pipeline Metrics
| Metric | Target | Alert |
|--------|--------|-------|
| Pipeline Success Rate | > 95% | < 90% |
| Average Build Time | < 10 min | > 15 min |
| Average Test Time | < 15 min | > 20 min |
| Deployment Frequency | Daily | < Weekly |
| Lead Time | < 1 week | > 2 weeks |
| Change Failure Rate | < 5% | > 10% |

### 9.2 Pipeline Notifications
| Event | Channel | Recipients |
|-------|---------|------------|
| Pipeline Failure | Slack | Team channel |
| Deploy Success | Slack | Team channel |
| Security Issue | Slack + Email | Security team |
| Release Created | Slack + Email | All stakeholders |

---

## 10. Disaster Recovery

### 10.1 Pipeline DR
| Scenario | Recovery Method | RTO |
|----------|----------------|-----|
| GitHub Down | Mirror to GitLab | 1 hour |
| Actions Down | Self-hosted runners | 30 min |
| ECR Down | S3 backup | 1 hour |
| ArgoCD Down | kubectl apply | 15 min |

### 10.2 Backup Strategy
| Component | Backup Method | Frequency | Retention |
|-----------|--------------|-----------|-----------|
| GitHub Repo | GitHub Mirror | Real-time | Indefinite |
| Actions Cache | S3 | Daily | 30 days |
| Docker Images | ECR Lifecycle | Per push | 30 versions |
| Helm Charts | Git | Real-time | Indefinite |
| Terraform State | S3 + DynamoDB | Real-time | Indefinite |
