---
description: "Create and manage CI/CD pipelines, GitHub Actions workflows, and automated deployment configurations."
name: CI/CD
---

# CI/CD Skill

## Capabilities
- **GitHub Actions** — create and manage workflow files
- **Pipeline design** — build → test → lint → deploy stages
- **Environment management** — staging, production, preview
- **Secret management** — use GitHub Secrets for sensitive values
- **Matrix builds** — test across multiple versions/platforms

## Best Practices
1. Fail fast — run linting and unit tests before expensive steps.
2. Cache dependencies (node_modules, pip cache) for faster builds.
3. Use specific action versions (e.g., `actions/checkout@v4`, not `@latest`).
4. Store secrets in GitHub Secrets, never in workflow files.
5. Use environment protection rules for production deployments.
6. Add status badges to the README.
7. Keep workflows DRY — use reusable workflows and composite actions.

## Standard Pipeline Stages
```
┌─────────┐   ┌──────┐   ┌──────┐   ┌────────┐   ┌────────┐
│ Install  │→  │ Lint │→  │ Test │→  │ Build  │→  │ Deploy │
└─────────┘   └──────┘   └──────┘   └────────┘   └────────┘
```

## Workflow File Location
All workflows go in `.github/workflows/` with descriptive names:
- `ci.yml` — main CI pipeline
- `deploy.yml` — deployment pipeline
- `release.yml` — release automation
- `pr-checks.yml` — pull request validation

## When to Use
- Setting up automated testing for a repository.
- Creating deployment pipelines.
- Automating release processes.
- Adding quality gates to pull requests.

