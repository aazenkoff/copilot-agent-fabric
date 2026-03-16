---
description: "Prompt for implementing authentication (registration, login, tokens, security)"
---

# Setup Authentication

## Context
Implement authentication for: **{{PROJECT_OR_SERVICE}}**

## Requirements
{{AUTH_REQUIREMENTS}}

## Steps

### 1 — Research Auth Strategy (`@researcher`)

Delegate to `@researcher`:

> Evaluate authentication strategies for **{{PROJECT_OR_SERVICE}}**:
>
> - **JWT** — stateless tokens, suitable for APIs and SPAs
> - **OAuth2/OIDC** — delegated auth, social login, enterprise SSO
> - **Session-based** — server-side sessions, traditional web apps
> - **SAML** — enterprise federation, legacy systems
>
> Consider:
> - Application type (SPA, mobile, server-rendered, API-only)
> - User base (consumer, enterprise, internal)
> - Compliance requirements (GDPR, SOC2, HIPAA)
> - Token storage and refresh strategy
>
> Produce a recommendation with security trade-offs.

---

### 2 — Implement Authentication (`@code-writer`)

Branch: `copilot/feat/{{PROJECT_OR_SERVICE}}-auth`

Delegate to `@code-writer`:

> Implement the authentication system based on the research from Step 1:
>
> 1. **Registration** — input validation, email verification, duplicate detection
> 2. **Login** — credential verification, rate limiting on failed attempts
> 3. **Password handling** — bcrypt/argon2 hashing, minimum complexity rules
> 4. **Token management** — access token issuance, refresh token rotation, secure storage
> 5. **Session management** — logout, token revocation, session invalidation
> 6. **Middleware/guards** — route protection, role-based access control
> 7. **Configuration** — secret keys via environment variables, configurable token TTLs

---

### 3 — Security Review (`@code-reviewer`)

Delegate to `@code-reviewer`:

> Perform a security-focused review of the auth implementation on branch `copilot/feat/{{PROJECT_OR_SERVICE}}-auth`:
>
> Verify against OWASP Authentication guidelines:
> - Password storage uses strong adaptive hashing (bcrypt/argon2, not MD5/SHA)
> - Tokens are signed and have appropriate expiry
> - Refresh tokens are rotated on use and revocable
> - No credentials in logs, URLs, or error messages
> - Rate limiting on login and registration endpoints
> - CSRF protection for session-based auth
> - Secure cookie flags (HttpOnly, Secure, SameSite) where applicable
> - Input validation on all auth endpoints

---

### 4 — Test (`@tester`)

Delegate to `@tester` (same branch):

> Write comprehensive auth tests:
>
> 1. **Happy path** — registration → login → authenticated request → logout
> 2. **Invalid credentials** — wrong password, non-existent user, locked account
> 3. **Token lifecycle** — token expiry, refresh flow, revocation
> 4. **Brute force protection** — rate limiting triggers after N failed attempts
> 5. **Edge cases** — duplicate registration, password reset flow, concurrent sessions
> 6. **Security** — SQL injection in login fields, XSS in user input, CSRF tokens

---

### 5 — Document Auth Flow (`@documenter`)

Delegate to `@documenter` (same branch):

> Create authentication documentation:
>
> 1. Auth flow sequence diagram (Mermaid) — registration, login, token refresh
> 2. API reference — auth endpoints with request/response examples
> 3. Security considerations and configuration guide
> 4. Integration guide for frontend clients (token storage, refresh strategy)
> 5. Troubleshooting common auth issues

Push the branch and open a PR:

```bash
git push -u origin copilot/feat/{{PROJECT_OR_SERVICE}}-auth
gh pr create --title "feat: authentication for {{PROJECT_OR_SERVICE}}" \
  --body "## Summary\nAuthentication system with registration, login, tokens, and security review."
```

Report the PR URL.
