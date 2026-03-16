---
description: "Prompt for reviewing code quality, security, architecture, and performance"
---

# Review Code

## Context
Please review the following code for quality, security, and best practices.

## Files to Review
{{FILES_OR_PATHS}}

## Focus Areas
- [ ] Correctness
- [ ] Security vulnerabilities
- [ ] Performance
- [ ] Code style and readability
- [ ] Test coverage
- [ ] Error handling
- [ ] Architecture and design patterns
- [ ] API contracts (backwards compatibility, status codes, validation)
- [ ] Database queries (indexing, N+1, migration safety)
- [ ] Accessibility (semantic HTML, ARIA, keyboard navigation)

## Steps
1. @code-reviewer — perform a thorough code review covering correctness, security, readability, and structural improvements. Include the Extended Review Areas (API contract, database, accessibility) where applicable.
2. @code-reviewer — perform an architecture review: component boundaries, dependency direction, separation of concerns, and adherence to project patterns
3. @code-reviewer — perform a performance review: identify N+1 queries, missing caching opportunities, unnecessary re-renders, memory leaks, and unbounded data fetches
4. @tester — check if test coverage is adequate; flag untested critical paths and edge cases

