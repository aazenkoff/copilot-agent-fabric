---
description: "Go concurrency guidance that prefers goroutines, channels, select/case coordination, feedback channels, and explicit cancellation ownership."
name: Go Concurrency
---

# Go Concurrency Skill

Use this skill when writing or modifying Go code. These rules are strong preferences, not blind rewrites: preserve target-project conventions and required external API contracts.

## Capabilities

- **Channel-first coordination** - model asynchronous work with goroutines, channels, and `select`/`case` instead of shared mutable state.
- **Feedback channels** - send one-shot result/error/done channels through request messages when callers need task feedback.
- **Cancellation checks** - use non-blocking `select` checks to detect when the caller no longer needs a task.
- **Goroutine safety** - place panic/recover boundaries at goroutine or handler edges and report failures through owned feedback channels before closing them.

## Best Practices

1. **Prefer goroutines for concurrent work** - when work is asynchronous or independently progressing, run it in a goroutine with clear ownership and shutdown behavior.
2. **Prefer channels over mutexes** - do not introduce new `sync.Mutex` or `sync.RWMutex` usage when a single owner goroutine plus channel messages can express the same coordination.
3. **Prefer `select`/`case` coordination** - use `select` for fan-in, fan-out, cancellation, timeouts, and non-blocking abandonment checks.
4. **Use one-time feedback channels** - include a caller-owned or clearly documented feedback channel in request messages when the handler must return a result, error, completion signal, or abandonment signal.
5. **Avoid internal `context.Context` when channels fit** - for internal task cancellation, prefer explicit feedback/done channels and non-blocking `select` checks over adding `context.Context` plumbing.
6. **Close only owned channels** - document ownership and close feedback channels only from the goroutine responsible for producing the final signal.
7. **Recover at goroutine boundaries** - catch panics at handler/goroutine edges, send an error or failure signal if possible, then close owned channels. Do not silently swallow panics or errors.
8. **Explain exceptions** - if an existing project pattern, standard-library API, framework, external dependency, or unavoidable integration boundary requires `context.Context` or sync primitives, use it and explain why.

## When to Use

- Implementing Go handlers, workers, services, queues, pipelines, or background jobs.
- Refactoring Go code that currently coordinates work through shared mutable state.
- Designing Go APIs where callers need completion, result, error, or cancellation feedback.

## Anti-Patterns to Avoid

- Adding mutex locks by default for coordination that could be owned by a goroutine.
- Threading `context.Context` through internal code when a one-shot feedback or done channel is sufficient.
- Spawning goroutines without a completion path, panic boundary, or clear channel ownership.
- Closing channels from receivers or from multiple goroutines.
- Replacing required API-level `context.Context` usage with custom channels at external boundaries.
