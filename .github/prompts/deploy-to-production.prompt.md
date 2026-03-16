---
description: "Prompt for deploying a service to production with pre-flight checks, rollback plan, and monitoring"
---

# Deploy to Production

## Context
Deploy **{{SERVICE_NAME}}** to production environment.

## Release Details
- **Version/Tag**: {{VERSION_OR_TAG}}
- **Environment**: {{TARGET_ENVIRONMENT}}
- **Deployment Strategy**: {{STRATEGY}} *(blue-green | canary | rolling)*

## Steps

### 1 — Pre-flight Checks (`@devops`)

Delegate to `@devops`:

> Verify production readiness for **{{SERVICE_NAME}}** version **{{VERSION_OR_TAG}}**:
>
> 1. CI pipeline passes on the release branch/tag — all tests green
> 2. No critical or high-severity vulnerabilities in dependency scan
> 3. Environment configuration is complete (secrets, env vars, feature flags)
> 4. Database migrations (if any) are tested and ready to apply
> 5. Rollback artifacts are available (previous version image/package)
> 6. Resource limits and scaling policies are configured
>
> Produce a pre-flight checklist with pass/fail status for each item.

---

### 2 — Deploy (`@devops`)

Delegate to `@devops`:

> Implement the **{{STRATEGY}}** deployment for **{{SERVICE_NAME}}**:
>
> - **Blue-green**: deploy to inactive environment, run health checks, switch traffic
> - **Canary**: deploy to canary instances (5–10% traffic), monitor error rate and latency
> - **Rolling**: update instances incrementally with health check gates between batches
>
> Include:
> 1. Deployment script or pipeline configuration
> 2. Health check validation between stages
> 3. Automatic rollback triggers (error rate > threshold, health check failures)
> 4. Rollback procedure documentation

---

### 3 — Smoke Tests (`@tester`)

Delegate to `@tester`:

> Run smoke tests against the deployed **{{TARGET_ENVIRONMENT}}** environment:
>
> 1. Health check endpoints return 200
> 2. Critical user flows complete successfully (login, core feature, data retrieval)
> 3. API response times are within SLO thresholds
> 4. External integrations are functional (payment, email, third-party APIs)
> 5. Report pass/fail for each check with response times

---

### 4 — Verify Monitoring (`@devops`)

Delegate to `@devops`:

> Using the `observability` skill, verify monitoring is active for the deployment:
>
> 1. Application metrics are flowing (request rate, error rate, latency percentiles)
> 2. Log aggregation is receiving new entries
> 3. Alerting rules are active for: error rate spike, latency degradation, resource exhaustion
> 4. Distributed tracing is capturing request flows
> 5. Dashboards reflect the new version in traffic

---

### 5 — Deployment Runbook (`@documenter`)

Delegate to `@documenter`:

> Create or update the deployment runbook for **{{SERVICE_NAME}}**:
>
> 1. Deployment procedure (step-by-step with commands)
> 2. Rollback procedure (step-by-step with commands)
> 3. Health check URLs and expected responses
> 4. Escalation contacts and communication channels
> 5. Post-deployment validation checklist
> 6. Known issues and workarounds for this release
