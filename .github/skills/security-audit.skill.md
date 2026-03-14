---
description: "Check for security vulnerabilities, CVEs, exposed secrets, and security anti-patterns in code and dependencies."
name: Security Audit
---

# Security Audit Skill

## Capabilities
- **CVE scanning** — check dependencies for known vulnerabilities
- **Secret detection** — find hardcoded passwords, API keys, tokens
- **OWASP checks** — detect common web security issues
- **Auth review** — verify authentication and authorization patterns
- **Input validation** — check for injection vulnerabilities

## Security Checklist
1. **Dependencies** — are all dependencies free of known CVEs?
2. **Secrets** — are there hardcoded credentials, API keys, or tokens?
3. **Injection** — is user input properly sanitized (SQL, XSS, command)?
4. **Authentication** — are auth flows implemented securely?
5. **Authorization** — are access controls properly enforced?
6. **Data exposure** — is sensitive data logged or exposed in errors?
7. **HTTPS** — are all external communications encrypted?
8. **CORS** — are cross-origin policies properly configured?

## Severity Classification
- 🔴 **Critical** — actively exploitable, immediate fix required
- 🟠 **High** — significant risk, fix in current sprint
- 🟡 **Medium** — moderate risk, fix soon
- 🟢 **Low** — minimal risk, fix when convenient

## When to Use
- Before merging code to main/production branches.
- When adding new dependencies.
- During periodic security audits.
- When handling user authentication or sensitive data.

