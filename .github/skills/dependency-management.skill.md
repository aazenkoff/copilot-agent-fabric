---
description: "Manage project dependencies — add, update, remove packages and validate for security vulnerabilities."
name: Dependency Management
---

# Dependency Management Skill

## Capabilities
- **Add dependencies** — install new packages with correct version constraints
- **Update dependencies** — upgrade to latest compatible versions
- **Remove dependencies** — clean up unused packages
- **Audit dependencies** — check for CVEs and security issues
- **Lock files** — ensure reproducible installs

## Best Practices
1. Always use exact or caret version ranges (not wildcard).
2. Check for CVEs before adding a new dependency.
3. Prefer well-maintained packages with active communities.
4. Minimize the number of dependencies — avoid "left-pad" scenarios.
5. Keep lock files (package-lock.json, poetry.lock, etc.) in version control.
6. Review transitive dependencies, not just direct ones.

## Package Manager Detection
Detect the project's package manager from:
- `package.json` / `package-lock.json` → npm/yarn/pnpm
- `requirements.txt` / `pyproject.toml` → pip/poetry
- `pom.xml` / `build.gradle` → Maven/Gradle
- `go.mod` → Go modules
- `Cargo.toml` → Cargo (Rust)
- `Gemfile` → Bundler (Ruby)

## When to Use
- Adding a new library or framework to the project.
- Upgrading dependencies for security patches.
- Cleaning up unused dependencies.
- Setting up a new project.

