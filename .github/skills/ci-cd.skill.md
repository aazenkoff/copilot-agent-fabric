---
description: "Structured workflow for creating and managing CI/CD pipelines, GitHub Actions workflows, and deployment automation."
name: CI/CD
---

# CI/CD Skill

## Capabilities
- **GitHub Actions** — create and manage workflow files
- **Pipeline design** — build → test → lint → deploy stages
- **Environment management** — staging, production, preview
- **Secret management** — use GitHub Secrets for sensitive values
- **Matrix builds** — test across multiple versions/platforms

## Workflow

### Step 1 — Assess Requirements
Before writing any pipeline configuration:
- Identify the project's language, framework, and build tool
- Determine what needs to run: lint, test, build, deploy, release?
- Identify target environments (staging, production, preview)
- Check for existing CI configuration to extend rather than replace
- List secrets and credentials the pipeline will need

### Step 2 — Design the Pipeline
Structure stages to fail fast and minimize wasted compute:

```
┌─────────┐   ┌──────┐   ┌──────┐   ┌────────┐   ┌────────┐
│ Install  │→  │ Lint │→  │ Test │→  │ Build  │→  │ Deploy │
└─────────┘   └──────┘   └──────┘   └────────┘   └────────┘
```

- Run cheap checks first (lint, type-check) before expensive ones (test, build)
- Use parallelism where stages are independent
- Add matrix builds for multi-version/multi-platform support
- Define clear triggers: push, pull_request, release, schedule

### Step 3 — Implement Workflow Files
Create workflows in `.github/workflows/` with descriptive names:

| File | Purpose | Trigger |
|------|---------|---------|
| `ci.yml` | Main CI pipeline (lint + test + build) | push, pull_request |
| `deploy.yml` | Deployment to staging/production | push to main, workflow_dispatch |
| `release.yml` | Version tagging and release | release published |
| `pr-checks.yml` | PR-specific validation | pull_request |

Follow these rules when writing workflow files:
1. Pin action versions exactly (`actions/checkout@v4`, not `@latest`)
2. Cache dependencies (`actions/cache` for node_modules, pip, gradle, etc.)
3. Use GitHub Secrets for all credentials — never hardcode
4. Set timeouts on jobs to prevent runaway costs
5. Use `concurrency` groups to cancel superseded runs
6. Add `permissions` block to follow least-privilege principle

### Step 4 — Configure Environments
For deployment pipelines:
- Create GitHub Environments (staging, production) with protection rules
- Require manual approval for production deployments
- Set environment-specific secrets and variables
- Add deployment status notifications (Slack, email, etc.)

### Step 5 — Validate
After creating pipeline configuration:
- Push to a branch and verify the workflow triggers correctly
- Check that all stages pass on a clean run
- Verify caching is working (second run should be faster)
- Test failure scenarios (does a failing test block deployment?)
- Add status badges to the README

## Anti-Patterns to Avoid
- ❌ Using `@latest` or `@main` for action versions (supply chain risk)
- ❌ Storing secrets in workflow files or repository code
- ❌ Running expensive steps (full test suite, Docker build) before cheap checks (lint)
- ❌ Missing `concurrency` groups (wasted compute on superseded commits)
- ❌ No timeout on jobs (risk of infinite-running workflows)
- ❌ Duplicating logic across workflows instead of using reusable workflows
- ❌ Storing a kubeconfig with `127.0.0.1` or `localhost` as the Kubernetes API server address — GitHub Actions runners cannot reach loopback addresses on the remote machine (see [Kubernetes Deployment Pitfalls](#kubernetes-deployment-pitfalls))

## Kubernetes Deployment Pitfalls

### Kubeconfig using localhost/127.0.0.1 as Kubernetes API server address

**Problem**: When a kubeconfig is generated on a VPS or local cluster (e.g., k3s, microk8s, kind), the API server address is often set to `127.0.0.1:6443` or `localhost:6443`. If this kubeconfig is base64-encoded and stored as a GitHub Secret (`KUBECONFIG_B64`), the deploy job will fail with:

```
dial tcp 127.0.0.1:6443: connect: connection refused
```

GitHub Actions runners run on their own machines and cannot reach `127.0.0.1` on your remote VPS — that loopback address resolves to the runner itself, not your cluster.

**Fix — patch at deploy time with `sed`**: After decoding the kubeconfig secret in the workflow, rewrite the server address before invoking `kubectl`. This keeps the stored secret unchanged and avoids re-encoding the kubeconfig:

```bash
echo "${{ secrets.KUBECONFIG_B64 }}" | base64 -d > $HOME/.kube/config
sed -i 's|https://127.0.0.1:|https://<ACTUAL_VPS_IP>:|g' $HOME/.kube/config
sed -i 's|https://localhost:|https://<ACTUAL_VPS_IP>:|g' $HOME/.kube/config
chmod 600 $HOME/.kube/config
```

Replace `<ACTUAL_VPS_IP>` with the real public IP (or store it as a separate secret, e.g. `${{ secrets.VPS_IP }}`):

```bash
sed -i "s|https://127.0.0.1:|https://${{ secrets.VPS_IP }}:|g" $HOME/.kube/config
sed -i "s|https://localhost:|https://${{ secrets.VPS_IP }}:|g" $HOME/.kube/config
```

**Alternative — fix the kubeconfig at the source before encoding**: Update the cluster entry in the kubeconfig on the VPS, then re-encode and update the GitHub Secret:

```bash
kubectl config set-cluster <cluster-name> --server=https://<VPS_IP>:6443
cat ~/.kube/config | base64 | tr -d '\n'
# Then update the KUBECONFIG_B64 GitHub Secret with this new value
```

This approach is cleaner long-term but requires updating the secret whenever the kubeconfig changes.

## When to Use
- Setting up automated testing for a repository
- Creating deployment pipelines
- Automating release processes
- Adding quality gates to pull requests
- Configuring matrix builds for cross-platform testing

