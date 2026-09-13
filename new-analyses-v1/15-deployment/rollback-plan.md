# Rollback Plan — YemenMart v2

**Document Version:** 1.0
**Date:** 2026-09-13
**Scope:** Production rollback procedures for application, database, configuration, and infrastructure changes

---

## 1. Rollback Decision Framework

### 1.1 Rollback Triggers

| Trigger | Threshold | Action | Authority |
|---------|-----------|--------|-----------|
| Error rate spike | > 5% (5-min window) | Immediate rollback | On-call engineer |
| P95 latency degradation | > 500ms (10-min window) | Investigate, rollback if > 15 min | On-call engineer |
| Payment failures | > 10 failures/min | Immediate rollback | On-call engineer |
| Data integrity error | Any detected | Immediate rollback + incident | Engineering lead |
| Security vulnerability | Critical/High | Immediate rollback + patch | Security team |
| Feature regression | Customer-reported critical bug | Rollback within 1 hour | Product + Engineering |
| Infrastructure failure | Service unavailable > 5 min | Failover + rollback | SRE team |

### 1.2 Rollback Authority

| Severity | Decision Maker | Max Rollback Time |
|----------|---------------|-------------------|
| P1 Critical | On-call engineer (auto-approve) | < 5 minutes |
| P2 High | Engineering lead | < 15 minutes |
| P3 Medium | Engineering manager | < 1 hour |
| P4 Low | Scheduled deployment window | Next deployment |

### 1.3 Rollback vs. Hotfix Decision

| Scenario | Decision | Reasoning |
|----------|----------|-----------|
| Bug in new code | Rollback + hotfix | Clean revert, fix forward |
| Bad database migration | Rollback + fix migration | Data integrity first |
| Infrastructure misconfig | Config rollback + fix | No code change needed |
| Third-party service outage | Wait / degrade gracefully | Rollback won't help |
| Performance regression | Rollback + optimize | User experience critical |

---

## 2. Pre-Rollback Checklist

### 2.1 Assessment (2 minutes)

- [ ] Identify the failing component (application, database, infrastructure)
- [ ] Determine rollback scope (full / partial / single service)
- [ ] Check if database migration is involved
- [ ] Verify current deployment version and target rollback version
- [ ] Notify team via #incidents Slack channel
- [ ] Document rollback reason in incident ticket

### 2.2 Communication (3 minutes)

- [ ] Post status update to status page (status.yemenmart.com)
- [ ] Notify customer support team via #support-alerts
- [ ] Alert engineering team via #engineering-alerts
- [ ] If payment affected: notify finance team
- [ ] If vendor-facing: notify vendor success team

### 2.3 Backup Verification (1 minute)

- [ ] Verify latest database backup exists and is restorable
- [ ] Confirm application artifact (Docker image) for previous version is available
- [ ] Verify configuration snapshots are current
- [ ] Check Redis cache state (snapshot if needed)

---

## 3. Application Rollback

### 3.1 Kubernetes Deployment Rollback

**Trigger:** Application code deployment causes errors or degradation.

```bash
# Check current deployment status
kubectl get deployments -n yemenmart
kubectl rollout history deployment/api-server -n yemenmart

# Rollback to previous revision
kubectl rollout undo deployment/api-server -n yemenmart

# Rollback to specific revision
kubectl rollout undo deployment/api-server --to-revision=N -n yemenmart

# Monitor rollback progress
kubectl rollout status deployment/api-server -n yemenmart

# Verify pods are running
kubectl get pods -n yemenmart -l app=api-server
```

### 3.2 ArgoCD Rollback

**Trigger:** GitOps deployment requires rollback.

```bash
# List application history
argocd app list
argocd app history yemenmart-api

# Rollback to previous version
argocd app rollback yemenmart-api

# Rollback to specific commit
argocd app rollback yemenmart-api <commit-sha>

# Sync to target state
argocd app sync yemenmart-api
```

### 3.3 Docker Image Rollback

**Trigger:** Container image issue.

```bash
# Find current image tag
kubectl get deployment api-server -n yemenmart -o jsonpath='{.spec.template.spec.containers[0].image}'

# Update to previous image
kubectl set image deployment/api-server \
  api-server=yemenmart/api-server:PREVIOUS_TAG \
  -n yemenmart

# Alternative: Edit deployment directly
kubectl edit deployment api-server -n yemenmart
# Change image tag to previous version
```

### 3.4 Blue-Green Rollback

**Trigger:** Blue-green deployment switch needs reversal.

```bash
# Check which environment is live
kubectl get svc api-server -n yemenmart -o jsonpath='{.spec.selector.version}'

# Switch traffic back to previous environment
kubectl patch svc api-server -n yemenmart \
  -p '{"spec":{"selector":{"version":"blue"}}}'

# Verify traffic routing
kubectl get endpoints api-server -n yemenmart
```

### 3.5 Canary Rollback

**Trigger:** Canary deployment showing issues.

```bash
# Remove canary weight from Ingress
kubectl annotate ingress api-ingress -n yemenmart \
  nginx.ingress.kubernetes.io/canary-weight- --overwrite

# Or set weight to 0
kubectl annotate ingress api-ingress -n yemenmart \
  nginx.ingress.kubernetes.io/canary-weight="0"

# Remove canary deployment
kubectl delete deployment api-server-canary -n yemenmart
```

---

## 4. Database Rollback

### 4.1 Migration Reversal (Forward Fix)

**Preferred approach:** Write a new migration that reverses the problematic change.

```bash
# Generate reversal migration
npx prisma migrate diff \
  --from-schema-datamodel prisma/schema.prisma \
  --to-schema-datamodel prisma/schema.prisma.prev \
  --script > migrations/reverse_migration.sql

# Review the generated SQL
cat migrations/reverse_migration.sql

# Apply in a transaction
psql $DATABASE_URL <<EOF
BEGIN;
-- Apply reversal migration
\i migrations/reverse_migration.sql
-- Verify
SELECT verify_migration_integrity();
COMMIT;
EOF
```

### 4.2 Database Point-in-Time Recovery

**Trigger:** Data corruption or destructive migration.

```bash
# Restore to specific point in time (RDS)
aws rds restore-db-instance-to-point-in-time \
  --source-db-instance-identifier yemenmart-prod \
  --target-db-instance-identifier yemenmart-restore \
  --restore-time "2026-09-13T10:00:00Z" \
  --db-instance-class db.r6g.xlarge \
  --vpc-security-group-ids sg-xxxxx

# Wait for restore to complete
aws rds wait db-instance-available \
  --db-instance-identifier yemenmart-restore

# Verify data integrity
psql -h yemenmart-restore.yemenmart.com -U ym_admin -d yemenmart <<EOF
SELECT verify_data_integrity();
SELECT count(*) FROM orders WHERE created_at > '2026-09-13T10:00:00Z';
EOF
```

### 4.3 Schema Rollback (Without Data Loss)

**Trigger:** Schema change breaks application compatibility.

```sql
-- Step 1: Create backup of affected table
CREATE TABLE orders_backup_20260913 AS SELECT * FROM orders;

-- Step 2: Apply schema change
ALTER TABLE orders DROP COLUMN IF EXISTS new_column;
ALTER TABLE orders ALTER COLUMN status TYPE varchar(30);

-- Step 3: Verify application compatibility
-- (Run smoke tests against restored schema)

-- Step 4: If issues, restore from backup
-- DROP TABLE orders;
-- ALTER TABLE orders_backup_20260913 RENAME TO orders;
```

### 4.4 Migration Rollback Script Template

```sql
-- File: migrations/YYYYMMDD_HHMMSS_rollback.sql
-- Purpose: Rollback migration YYYYMMDD_HHMMSS

BEGIN;

-- Log rollback action
INSERT INTO audit_logs (action, resource_type, resource_id, new_values, created_at)
VALUES ('migration_rollback', 'migration', 'YYYYMMDD_HHMMSS',
        '{"reason": "Caused API errors", "rolled_back_by": "admin"}', now());

-- Reverse DDL changes
ALTER TABLE orders DROP COLUMN IF EXISTS priority_flag;
ALTER TABLE orders ALTER COLUMN total_amount TYPE bigint;

-- Reverse DML changes
UPDATE products SET status = 'active' WHERE status = 'pending_review_v2';

-- Verify rollback
DO $$
BEGIN
    IF EXISTS (SELECT 1 FROM information_schema.columns
               WHERE table_name = 'orders' AND column_name = 'priority_flag') THEN
        RAISE EXCEPTION 'Rollback failed: column still exists';
    END IF;
END $$;

COMMIT;
```

---

## 5. Configuration Rollback

### 5.1 Environment Variable Rollback

**Trigger:** Configuration change causes service degradation.

```bash
# View current ConfigMap
kubectl get configmap api-config -n yemenmart -o yaml

# Rollback to previous version
kubectl rollout undo configmap/api-config -n yemenmart

# Or apply saved configuration
kubectl apply -f configs/api-config-backup.yaml -n yemenmart

# Restart pods to pick up new config
kubectl rollout restart deployment/api-server -n yemenmart
```

### 5.2 Secrets Rollback

```bash
# View secret history (if using sealed-secrets or external-secrets)
kubectl get secret api-secrets -n yemenmart -o yaml

# Rollback sealed secret
kubectl apply -f secrets/api-secrets-backup.yaml -n yemenmart

# Restart affected deployments
kubectl rollout restart deployment/api-server -n yemenmart
```

### 5.3 Feature Flag Rollback

**Immediate rollback without deployment:**

```bash
# Disable feature via API
curl -X PUT https://api.yemenmart.com/api/v1/admin/settings/feature-flags \
  -H "Authorization: Bearer $ADMIN_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "flags": {
      "new_checkout_flow": false,
      "wallet_v2": false,
      "search_v2": false
    }
  }'

# Or via Redis directly
redis-cli DEL feature_flag:new_checkout_flow
redis-cli SET feature_flag:wallet_v2 "false"
```

### 5.4 Nginx/ALB Configuration Rollback

```bash
# ALB listener rule rollback
aws elbv2 modify-rule \
  --rule-arn $RULE_ARN \
  --actions '[{
    "Type": "forward",
    "TargetGroupArn": "arn:aws:elasticloadbalancing:...targetgroup/api-previous/..."
  }]'

# Nginx config rollback (if using ConfigMap)
kubectl apply -f nginx/nginx-config-backup.yaml -n yemenmart
kubectl rollout restart deployment/nginx -n yemenmart
```

### 5.5 CDN Configuration Rollback

```bash
# CloudFront distribution rollback
aws cloudfront create-invalidation \
  --distribution-id $DISTRIBUTION_ID \
  --paths "/*"

# WAF rule rollback
aws wafv2 update-web-acl \
  --name yemenmart-waf \
  --scope REGIONAL \
  --id $WAF_ACL_ID \
  --lock-token $LOCK_TOKEN \
  --default-action '{"Allow":{}}' \
  --rules file://waf-rules-backup.json
```

---

## 6. Infrastructure Rollback

### 6.1 Terraform Rollback

```bash
# Check current state
terraform state list
terraform plan -out=rollback.tfplan

# Apply previous state
terraform apply rollback.tfplan

# Or target specific resource
terraform apply -target=aws_instance.api_server[0] rollback.tfplan

# Import drifted resources
terraform import aws_instance.api_server[0] i-xxxxx
```

### 6.2 ECS Service Rollback

```bash
# Check service history
aws ecs describe-services \
  --cluster yemenmart-prod \
  --services api-server

# Update to previous task definition
aws ecs update-service \
  --cluster yemenmart-prod \
  --service api-server \
  --task-definition yemenmart-api:PREVIOUS_REVISION \
  --force-new-deployment

# Wait for stability
aws ecs wait services-stable \
  --cluster yemenmart-prod \
  --services api-server
```

---

## 7. Post-Rollback Verification

### 7.1 Automated Checks (Within 5 minutes)

- [ ] All health checks passing (Kubernetes / ECS)
- [ ] Error rate returned to baseline (< 0.1%)
- [ ] P95 latency within SLA (< 200ms)
- [ ] Payment processing functioning (test transaction)
- [ ] Search functionality working (test query)
- [ ] Mobile app connectivity confirmed

### 7.2 Smoke Test Suite

```bash
# Run smoke tests
npm run test:smoke:production

# Key checks:
# 1. Authentication flow (register, login, token refresh)
# 2. Product listing and search
# 3. Cart operations (add, update, remove)
# 4. Checkout flow (payment intent creation)
# 5. Order listing
# 6. Wallet balance check
# 7. Notification delivery
# 8. Admin dashboard access
```

### 7.3 Manual Verification

| Check | Owner | Sign-off |
|-------|-------|----------|
| User login works | QA Engineer | [ ] |
| Product search returns results | QA Engineer | [ ] |
| Cart add/remove works | QA Engineer | [ ] |
| Payment processing works | QA Engineer | [ ] |
| Order placement works | QA Engineer | [ ] |
| Wallet top-up works | QA Engineer | [ ] |
| Admin dashboard loads | Admin | [ ] |
| Vendor portal loads | Vendor Manager | [ ] |
| Mobile app connects | Mobile Engineer | [ ] |
| RTL layout renders correctly | QA Engineer | [ ] |

### 7.4 Monitoring Dashboard

Monitor the following for 30 minutes post-rollback:

| Metric | Alert Threshold | Dashboard |
|--------|----------------|-----------|
| Request rate | > 50% deviation | Grafana: API Overview |
| Error rate (5xx) | > 1% | Grafana: API Errors |
| P95 latency | > 200ms | Grafana: API Latency |
| CPU utilization | > 80% | Grafana: Infrastructure |
| Memory utilization | > 85% | Grafana: Infrastructure |
| Database connections | > 80% pool | Grafana: Database |
| Queue depth | > 5,000 | Grafana: RabbitMQ |
| Cache hit ratio | < 90% | Grafana: Redis |

---

## 8. Communication Plan

### 8.1 Rollback Notification Templates

**Internal (Slack #incidents):**

```
🚨 ROLLBACK INITIATED
Component: [API / Database / Configuration]
Version: [FROM] → [TO]
Reason: [Brief description]
Impact: [User-facing impact]
ETA: [Estimated resolution time]
Incident lead: [Name]
Status page: https://status.yemenmart.com
```

**External (Status Page):**

```
[Investigating] We are investigating reports of [issue].
Some users may experience [symptom]. We are working to resolve this.

[Identified] The issue has been identified. We are implementing a fix.

[Monitoring] A fix has been deployed. We are monitoring the situation.

[Resolved] The issue has been resolved. All services are operating normally.
```

**Vendor Notification (Email):**

```
Subject: YemenMart Service Update — [Date]

Dear [Vendor Name],

We experienced a temporary service issue that has been resolved.
Your orders and products are unaffected.

If you notice any issues, please contact vendor-support@yemenmart.com.

Best regards,
YemenMart Team
```

### 8.2 Communication Escalation

| Time Since Rollback | Action | Channel |
|--------------------|--------|---------|
| 0 min | Initial alert | Slack #incidents |
| 5 min | Status page update | status.yemenmart.com |
| 10 min | Customer support briefing | Slack #support-alerts |
| 15 min | Vendor notification (if affected) | Email |
| 30 min | Management briefing | Slack #leadership |
| 1 hr | Customer email (if major impact) | Email blast |
| 2 hr | Public social media update | Twitter, Facebook |

---

## 9. Incident Documentation

### 9.1 Rollback Log Template

```markdown
# Rollback Report — [Date]

## Summary
- **Incident ID:** INC-YYYY-MM-DD-XXX
- **Rollback Time:** HH:MM UTC
- **Duration:** X minutes
- **Severity:** P1/P2/P3/P4
- **Impact:** [Description of user impact]

## Timeline
- HH:MM — Deployment initiated
- HH:MM — Error rate spike detected
- HH:MM — Rollback decision made
- HH:MM — Rollback initiated
- HH:MM — Rollback completed
- HH:MM — Verification passed
- HH:MM — Incident resolved

## Root Cause
[Detailed root cause analysis]

## Changes Rolled Back
- [List of commits/changes]
- [Database migrations]
- [Configuration changes]

## Verification
- [x] Health checks passing
- [x] Error rate normalized
- [x] Latency within SLA
- [x] Smoke tests passing
- [x] Manual verification complete

## Follow-up Actions
- [ ] Fix root cause
- [ ] Add test coverage
- [ ] Update monitoring/alerts
- [ ] Update runbooks
- [ ] Post-mortem scheduled

## Lessons Learned
[Key takeaways and process improvements]
```

### 9.2 Post-Mortem Schedule

| Timeframe | Action | Owner |
|-----------|--------|-------|
| Within 24 hours | Incident timeline documented | Incident lead |
| Within 48 hours | Root cause analysis completed | Engineering lead |
| Within 1 week | Post-mortem meeting held | Engineering manager |
| Within 2 weeks | Action items assigned and tracked | Engineering manager |
| Within 1 month | All action items completed | Team |

---

## 10. Rollback Automation

### 10.1 Automated Rollback Script

```bash
#!/bin/bash
# File: scripts/rollback.sh
# Usage: ./rollback.sh [component] [version]

set -euo pipefail

COMPONENT=${1:-"api-server"}
VERSION=${2:-"previous"}
NAMESPACE="yemenmart"
SLACK_WEBHOOK="${SLACK_WEBHOOK_URL}"

log() { echo "[$(date -u +%Y-%m-%dT%H:%M:%SZ)] $*"; }

notify_slack() {
  curl -s -X POST "$SLACK_WEBHOOK" \
    -H "Content-Type: application/json" \
    -d "{\"text\":\"🔄 ROLLBACK: $COMPONENT to $VERSION by $(whoami)\"}"
}

notify_slack

case $COMPONENT in
  api-server)
    log "Rolling back api-server to $VERSION"
    if [ "$VERSION" = "previous" ]; then
      kubectl rollout undo deployment/api-server -n $NAMESPACE
    else
      kubectl set image deployment/api-server \
        api-server=yemenmart/api-server:$VERSION -n $NAMESPACE
    fi
    kubectl rollout status deployment/api-server -n $NAMESPACE --timeout=300s
    ;;

  worker)
    log "Rolling back worker to $VERSION"
    kubectl rollout undo deployment/worker -n $NAMESPACE
    kubectl rollout status deployment/worker -n $NAMESPACE --timeout=300s
    ;;

  database)
    log "DATABASE ROLLBACK REQUIRES MANUAL APPROVAL"
    log "Run: npm run db:rollback:$VERSION"
    exit 1
    ;;

  config)
    log "Rolling back configuration"
    kubectl rollout undo configmap/app-config -n $NAMESPACE
    kubectl rollout restart deployment/api-server -n $NAMESPACE
    ;;

  *)
    log "Unknown component: $COMPONENT"
    exit 1
    ;;
esac

log "Rollback complete. Running smoke tests..."
npm run test:smoke:production

log "✅ Rollback and verification complete"
notify_slack
```

### 10.2 CI/CD Rollback Pipeline

```yaml
# .github/workflows/rollback.yml
name: Production Rollback
on:
  workflow_dispatch:
    inputs:
      component:
        description: 'Component to rollback'
        required: true
        type: choice
        options:
          - api-server
          - worker
          - all
      version:
        description: 'Target version (or "previous")'
        required: true
        default: 'previous'

jobs:
  rollback:
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/checkout@v4
      - name: Execute rollback
        run: ./scripts/rollback.sh ${{ inputs.component }} ${{ inputs.version }}
      - name: Run smoke tests
        run: npm run test:smoke:production
      - name: Notify team
        uses: slackapi/slack-github-action@v1
        with:
          payload: |
            {"text": "Rollback ${{ inputs.component }} to ${{ inputs.version }} completed"}
```

---

## 11. Rollback Testing

### 11.1 Regular Rollback Drills

| Frequency | Scope | Participants | Duration |
|-----------|-------|-------------|----------|
| Monthly | Application rollback | On-call team | 30 min |
| Quarterly | Full stack rollback | Engineering team | 2 hours |
| Bi-annually | DR site failover | All teams | 4 hours |
| Annually | Complete disaster recovery | All teams + management | 8 hours |

### 11.2 Rollback Test Checklist

- [ ] Previous version image/tag is available
- [ ] Database migration reversal exists
- [ ] Configuration backup is current
- [ ] Rollback script executes successfully
- [ ] Smoke tests pass after rollback
- [ ] Monitoring confirms metrics normalization
- [ ] Communication plan executes correctly
- [ ] Team members know their roles
- [ ] Rollback time meets SLA (< 15 min for P1)

### 11.3 Rollback Time Targets

| Component | Target Time | Maximum Time |
|-----------|-------------|--------------|
| Application (Kubernetes) | < 2 min | 5 min |
| Application (ECS) | < 3 min | 5 min |
| Configuration | < 1 min | 3 min |
| Database migration | < 10 min | 30 min |
| Database PITR | < 30 min | 60 min |
| Infrastructure (Terraform) | < 10 min | 30 min |
| Full stack | < 15 min | 60 min |

---

## 12. Appendix: Version Registry

| Version | Release Date | Image Tag | Migration | Rollback Notes |
|---------|-------------|-----------|-----------|----------------|
| v2.0.0 | 2026-09-01 | `v2.0.0` | `20260901_init` | Initial release |
| v2.0.1 | 2026-09-05 | `v2.0.1` | None | Bug fix only |
| v2.1.0 | 2026-09-10 | `v2.1.0` | `20260910_add_wallet` | Reversible migration |
| v2.1.1 | 2026-09-13 | `v2.1.1` | None | Bug fix only |

**Current production version:** v2.1.1
**Previous stable version:** v2.1.0
**Last known good version:** v2.1.1
