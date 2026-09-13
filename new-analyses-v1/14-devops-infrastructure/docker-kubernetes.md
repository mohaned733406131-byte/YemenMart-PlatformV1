# Docker & Kubernetes Strategy - YemenMart

## Document Information
| Property | Value |
|----------|-------|
| Document ID | YEM-INF-DCRK-001 |
| Version | 1.0 |
| Status | Active |
| Last Updated | 2026-09-12 |

---

## 1. Container Strategy

### 1.1 Container Runtime
| Component | Selection | Version |
|-----------|-----------|---------|
| Container Runtime | containerd | 1.7.x |
| Orchestrator | Kubernetes (EKS) | 1.28 |
| Package Format | OCI | - |
| Image Registry | ECR | - |

### 1.2 Base Images
| Service Type | Base Image | Size | Security |
|-------------|------------|------|----------|
| Node.js API | node:20-alpine | ~150MB | Minimal attack surface |
| Python Service | python:3.12-slim | ~120MB | Minimal packages |
| Go Service | gcr.io/distroless/static | ~2MB | No shell |
| Nginx | nginx:1.25-alpine | ~25MB | Minimal |
| PostgreSQL | postgres:15-alpine | ~80MB | Official |

### 1.3 Image Standards
| Standard | Implementation |
|----------|---------------|
| Multi-stage Builds | Separate build and runtime stages |
| Non-root User | Run as non-root (UID 1001) |
| Read-only Filesystem | Read-only root filesystem |
| No Latest Tag | Always use specific versions |
| Image Scanning | Trivy scan before push |
| Image Signing | Cosign for image signing |
| Layer Optimization | Minimize layers, combine RUN commands |

---

## 2. Docker Configuration

### 2.1 Dockerfile Best Practices
| Practice | Implementation |
|----------|---------------|
| Official Base Images | Use official Docker Hub images |
| Specific Versions | pin image versions with digests |
| Multi-stage Builds | Reduce final image size |
| .dockerignore | Exclude .git, node_modules, tests |
| COPY vs ADD | Use COPY for local files |
| Cache Dependencies | Copy package*.json first |
| Security Scanning | Trivy scan in CI |
| Health Checks | Include HEALTHCHECK instruction |

### 2.2 Docker Compose (Local Development)
```yaml
version: '3.8'

services:
  api:
    build:
      context: .
      dockerfile: Dockerfile.dev
    ports:
      - "8080:8080"
    volumes:
      - ./src:/app/src
      - /app/node_modules
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgresql://postgres:password@db:5432/yemenmart
      - REDIS_URL=redis://redis:6379
    depends_on:
      - db
      - redis

  db:
    image: postgres:15-alpine
    ports:
      - "5432:5432"
    environment:
      POSTGRES_DB: yemenmart
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data

  elasticsearch:
    image: elasticsearch:8.11.0
    ports:
      - "9200:9200"
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
    volumes:
      - es_data:/usr/share/elasticsearch/data

  mailhog:
    image: mailhog/mailhog
    ports:
      - "1025:1025"
      - "8025:8025"

volumes:
  postgres_data:
  redis_data:
  es_data:
```

### 2.3 Environment Variables
| Variable | Source | Description |
|----------|--------|-------------|
| NODE_ENV | ConfigMap | Environment mode |
| DATABASE_URL | Secret | Database connection string |
| REDIS_URL | Secret | Redis connection string |
| JWT_SECRET | Secret | JWT signing key |
| API_KEY | Secret | External API key |
| LOG_LEVEL | ConfigMap | Logging level |
| CORS_ORIGIN | ConfigMap | Allowed origins |

---

## 3. Kubernetes Architecture

### 3.1 Cluster Configuration
| Setting | Value |
|---------|-------|
| Cluster Name | yemenmart-prod |
| Region | me-south-1 (Bahrain) |
| Kubernetes Version | 1.28 |
| Node AMI | Amazon Linux 2 |
| Pod CIDR | 10.0.0.0/16 |
| Service CIDR | 10.100.0.0/16 |
| DNS Provider | CoreDNS |
| CNI Plugin | Amazon VPC CNI |
| Storage Class | gp3 |

### 3.2 Node Groups
| Node Group | Instance | Min | Max | Labels | Taints |
|-----------|----------|-----|-----|--------|--------|
| system | m6i.xlarge | 3 | 5 | node-role: system | Dedicated |
| application | m6i.2xlarge | 6 | 50 | node-role: app | - |
| data | r6i.2xlarge | 3 | 15 | node-role: data | Dedicated |

### 3.3 Namespace Strategy
| Namespace | Purpose | Resource Quota |
|-----------|---------|---------------|
| default | System workloads | Default |
| yemenmart-prod | Production apps | High |
| yemenmart-staging | Staging apps | Medium |
| yemenmart-qa | QA apps | Low |
| monitoring | Monitoring stack | Medium |
| logging | Logging stack | Medium |
| argocd | GitOps | Low |

---

## 4. Deployment Manifests

### 4.1 Deployment Template
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: yemenmart-api
  namespace: yemenmart-prod
  labels:
    app: yemenmart-api
    version: v1.0.0
spec:
  replicas: 3
  selector:
    matchLabels:
      app: yemenmart-api
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: yemenmart-api
        version: v1.0.0
    spec:
      serviceAccountName: yemenmart-api
      securityContext:
        runAsNonRoot: true
        runAsUser: 1001
        fsGroup: 1001
      containers:
        - name: api
          image: yemenmart/api:1.0.0
          ports:
            - containerPort: 8080
              protocol: TCP
          resources:
            requests:
              cpu: 500m
              memory: 512Mi
            limits:
              cpu: 1000m
              memory: 1Gi
          livenessProbe:
            httpGet:
              path: /health/live
              port: 8080
            initialDelaySeconds: 30
            periodSeconds: 10
            timeoutSeconds: 5
            failureThreshold: 3
          readinessProbe:
            httpGet:
              path: /health/ready
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 5
            timeoutSeconds: 3
            failureThreshold: 3
          startupProbe:
            httpGet:
              path: /health/startup
              port: 8080
            initialDelaySeconds: 10
            periodSeconds: 5
            timeoutSeconds: 3
            failureThreshold: 30
          envFrom:
            - configMapRef:
                name: yemenmart-config
            - secretRef:
                name: yemenmart-secrets
          volumeMounts:
            - name: tmp
              mountPath: /tmp
      volumes:
        - name: tmp
          emptyDir: {}
      topologySpreadConstraints:
        - maxSkew: 1
          topologyKey: topology.kubernetes.io/zone
          whenUnsatisfiable: DoNotSchedule
          labelSelector:
            matchLabels:
              app: yemenmart-api
```

### 4.2 Service Configuration
```yaml
apiVersion: v1
kind: Service
metadata:
  name: yemenmart-api
  namespace: yemenmart-prod
  labels:
    app: yemenmart-api
spec:
  type: ClusterIP
  ports:
    - port: 80
      targetPort: 8080
      protocol: TCP
      name: http
  selector:
    app: yemenmart-api
```

### 4.3 Ingress Configuration
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: yemenmart-api
  namespace: yemenmart-prod
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:me-south-1:ACCOUNT:certificate/CERT_ID
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTPS":443}]'
    alb.ingress.kubernetes.io/ssl-redirect: '443'
    alb.ingress.kubernetes.io/healthcheck-path: /health
    alb.ingress.kubernetes.io/healthcheck-interval-seconds: '15'
    alb.ingress.kubernetes.io/healthcheck-timeout-seconds: '5'
    alb.ingress.kubernetes.io/healthy-threshold-count: '2'
    alb.ingress.kubernetes.io/unhealthy-threshold-count: '3'
spec:
  rules:
    - host: api.yemenmart.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: yemenmart-api
                port:
                  number: 80
```

---

## 5. Horizontal Pod Autoscaler

### 5.1 HPA Configuration
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: yemenmart-api
  namespace: yemenmart-prod
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: yemenmart-api
  minReplicas: 3
  maxReplicas: 20
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
        - type: Percent
          value: 100
          periodSeconds: 60
        - type: Pods
          value: 4
          periodSeconds: 60
      selectPolicy: Max
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
        - type: Percent
          value: 10
          periodSeconds: 120
      selectPolicy: Min
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 60
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 70
    - type: Pods
      pods:
        metric:
          name: http_requests_per_second
        target:
          type: AverageValue
          averageValue: "1000"
```

### 5.2 Scaling Rules
| Metric | Scale Up | Scale Down | Cooldown |
|--------|----------|------------|----------|
| CPU > 60% | +100% or +4 pods | - | 60s |
| CPU < 30% | - | -10% every 2min | 300s |
| Memory > 70% | +100% or +4 pods | - | 60s |
| RPS > 1000/pod | +50% | - | 60s |
| Latency > 500ms | +50% | - | 120s |

---

## 6. Resource Management

### 6.1 Resource Quotas
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: yemenmart-quota
  namespace: yemenmart-prod
spec:
  hard:
    requests.cpu: "40"
    requests.memory: 80Gi
    limits.cpu: "80"
    limits.memory: 160Gi
    pods: "100"
    services: "20"
    persistentvolumeclaims: "10"
```

### 6.2 Limit Ranges
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: yemenmart-limits
  namespace: yemenmart-prod
spec:
  limits:
    - default:
        cpu: 1000m
        memory: 1Gi
      defaultRequest:
        cpu: 250m
        memory: 256Mi
      type: Container
```

---

## 7. Pod Disruption Budgets

### 7.1 PDB Configuration
```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: yemenmart-api
  namespace: yemenmart-prod
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: yemenmart-api
```

### 7.2 PDB Policy
| Service | Min Available | Max Unavailable |
|---------|--------------|-----------------|
| API Gateway | 2 | - |
| Auth Service | 2 | - |
| Product Service | 2 | - |
| Order Service | 2 | - |
| Payment Service | 2 | - |
| Search Service | 2 | - |

---

## 8. Network Policies

### 8.1 Default Deny
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
  namespace: yemenmart-prod
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress
```

### 8.2 Allow Rules
| Rule | From | To | Ports |
|------|------|----|-------|
| ALB to App | ALB namespace | App pods | 8080 |
| App to DB | App pods | Data subnet | 5432 |
| App to Redis | App pods | Data subnet | 6379 |
| App to ES | App pods | Data subnet | 9200 |
| App to App | App pods | App pods | 8080 |

---

## 9. Storage

### 9.1 Persistent Volumes
| Component | Storage Class | Size | Access Mode |
|-----------|--------------|------|-------------|
| PostgreSQL | gp3 | 100GB | ReadWriteOnce |
| Redis | gp3 | 16GB | ReadWriteOnce |
| Elasticsearch | gp3 | 500GB | ReadWriteOnce |
| Application | - | - | - |

### 9.2 Storage Classes
| Name | Provisioner | Type | Encryption |
|------|-------------|------|------------|
| gp3 | ebs.csi.aws.com | SSD | Enabled |
| gp2 | ebs.csi.aws.com | SSD | Enabled |
| io1 | ebs.csi.aws.com | SSD | Enabled |

---

## 10. Monitoring and Observability

### 10.1 Metrics Collection
| Source | Tool | Metrics |
|--------|------|---------|
| Kubernetes | kube-state-metrics | Cluster state |
| Nodes | node-exporter | System metrics |
| Pods | cAdvisor | Container metrics |
| Application | Prometheus client | Custom metrics |
| Ingress | ALB metrics | Traffic metrics |

### 10.2 Logging
| Component | Method | Destination |
|-----------|--------|-------------|
| Application Logs | stdout/stderr | Fluentd |
| Container Logs | /var/log | Fluentd |
| Audit Logs | kube-audit | S3 |
| Ingress Logs | ALB access logs | S3 |

### 10.3 Alerting Rules
| Alert | Condition | Severity |
|-------|-----------|----------|
| PodCrashLooping | restart > 5 in 10min | Critical |
| PodNotReady | not ready > 5min | Warning |
| HighCPU | CPU > 80% for 5min | Warning |
| HighMemory | Memory > 85% for 5min | Warning |
| PVCNearFull | Usage > 80% | Warning |
| NodeNotReady | not ready > 5min | Critical |

---

## 11. Security

### 11.1 Pod Security
| Control | Implementation |
|---------|---------------|
| Security Context | Non-root, read-only filesystem |
| AppArmor | Default profile |
| Seccomp | Runtime default |
| Capabilities | Drop all, add only needed |
| Image Policy | Signed images only |

### 11.2 RBAC
| Role | Permissions | Subjects |
|------|------------|----------|
| admin | Full cluster access | Platform team |
| developer | Deploy, view logs | Development team |
| viewer | View only | QA team |
| ci-cd | Deploy to namespaces | GitHub Actions |

### 11.3 Secret Management
| Secret Type | Storage | Rotation |
|------------|---------|----------|
| Database Credentials | AWS Secrets Manager | 90 days |
| API Keys | AWS Secrets Manager | 90 days |
| TLS Certificates | ACM | Automatic |
| Docker Registry | ECR | Automatic |

---

## 12. Disaster Recovery

### 12.1 Backup Strategy
| Component | Method | Frequency | Retention |
|-----------|--------|-----------|-----------|
| etcd | Velero | Daily | 30 days |
| Persistent Volumes | Velero | Daily | 30 days |
| Kubernetes Resources | Velero | Daily | 30 days |
| Helm Values | Git | Real-time | Indefinite |

### 12.2 Recovery Procedures
| Scenario | Method | RTO |
|----------|--------|-----|
| Pod Failure | Auto-restart | < 1 min |
| Node Failure | Pod reschedule | < 5 min |
| AZ Failure | Cross-AZ failover | < 15 min |
| Region Failure | DR region | < 1 hour |
| Complete Loss | Rebuild from backup | < 4 hours |
