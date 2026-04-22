# Context Engineering Pipeline

> **TL;DR** — Before writing any production code, run Research → Design → Plan through quality gates. Then implement phase-by-phase. This prevents "AI slop" and produces reviewable, auditable artifacts.

## What Is Context Engineering?

Context Engineering (CE) is a **4-phase engineering pipeline** for AI-assisted development. Instead of asking an AI agent to "just build the feature," you front-load the work into three artifact-driven, gate-controlled phases before a single line of production code is written.

The pipeline is based on empirical data: skipping Research/Design/Plan phases results in approximately 1.7× more defects, 8× more performance issues, and 41% more unnecessary complexity.

## The 4 Phases

```
Phase 0  Triage      — classify the request (Trivial / Prototype / Standard)
Phase 1  Research    — gather context; produce research.md
         ── Gate 1: user approves ──
Phase 2  Design      — architecture + diagrams + contracts; produce design.md
         ── Gate 2: user approves ──
Phase 3  Plan        — ordered atomic phases with acceptance criteria; produce plan.md
         ── Gate 3: user approves (most important!) ──
Phase 4  Implement   — Coder → Reviewer → Tester per plan phase; produce gates.md
         ── Final: PR opened, URL reported ──
```

### Phase 0 — Triage

The **Orchestrator** classifies every incoming request before dispatching any agent:

| Verdict | Criteria |
|---------|----------|
| **Trivial** | Typo, comment-only, single-line fix, dep bump, docs-only |
| **Prototype** | User says "prototype / spike / sketch / poc / demo", or `mode: prototype` in prompt |
| **Standard** | Any other production code change |

Trivial changes bypass the pipeline entirely. Standard and prototype requests enter the pipeline.

### Phase 1 — Research (no code)

**Lead agent:** `researcher` (+ `code-investigator` for deep codebase recon)

Produces **`docs/context-eng/<slug>/research.md`** covering:
- Problem restatement and measurable success criteria
- Map of affected files, modules, and symbols
- Existing patterns and conventions to follow
- Prior art, library options, and tradeoffs considered
- Constraints (performance, security, platform, dependencies)
- Open questions (answered or explicitly deferred)

### Phase 2 — Design (no code)

**Lead agent:** `code-investigator`

Produces **`docs/context-eng/<slug>/design.md`** covering:
- Architecture diagram (Mermaid C4 component or graph)
- Data Flow Diagram (Mermaid)
- Sequence diagram(s) for key interactions
- Public contracts: types, function signatures, API shapes
- State model where relevant
- Failure modes and rollback story
- Explicit non-goals

### Phase 3 — Plan (no code)

**Lead agent:** `orchestrator`

Produces **`docs/context-eng/<slug>/plan.md`** — an ordered list of atomic implementation phases, each with:
- ID, title, one-sentence goal
- Files to touch
- Testable acceptance criteria
- Suggested coder agent (e.g., `code-writer`, `unity-gameplay-developer`)
- Risk level (low / medium / high) and blast radius note
- Dependencies on previous phases

**Gate 3 is the most important gate.** An approved plan is the implementation contract. Changes after Gate 3 require a plan amendment.

### Phase 4 — Implement (code allowed)

For each phase in `plan.md`, the orchestrator runs a **Coder → Reviewer → Tester** loop:

1. **Coder** (specialist or `code-writer`) implements the phase against its acceptance criteria.
2. **`code-reviewer`** reviews against `design.md` contracts + acceptance criteria.
3. Coder applies feedback.
4. **`tester`** writes and runs tests; result is recorded as evidence.
5. Phase result appended to **`docs/context-eng/<slug>/gates.md`**.
6. Commit with message `feat(<slug>): phase-<N> — <title>`.

After all phases pass: a single PR is opened containing code + all CE artifacts.

## Artifact Layout

All CE artifacts live **in the target project repo** (not this fabric repo):

```
docs/context-eng/<feature-slug>/
├── README.md        # index: slug, mode, status, gate log summary
├── research.md      # Phase 1 output
├── design.md        # Phase 2 output (Mermaid diagrams included)
├── plan.md          # Phase 3 output (ordered phases + acceptance criteria)
└── gates.md         # running append-only gate log
```

`<feature-slug>` is a short, lowercase, hyphenated name, e.g., `user-auth`, `payment-webhook`, `multiplayer-sync`.

## Gate Semantics

A **gate** is a checkpoint between phases. Gates enforce "no code before design."

### Strict mode (standard)

```
orchestrator: [posts artifact content or file path]
orchestrator: "Gate N complete. Approve to proceed, or describe changes."
user:         "approved"  ← explicit, typed approval required
orchestrator: [proceeds to next phase]
```

The orchestrator **never** auto-approves in strict mode. It waits for the user's "approved" message.

If changes are requested, the lead agent revises the artifact and the gate re-opens.

### Prototype mode (relaxed)

All gates auto-pass immediately with a banner in `gates.md`:
```
> ⚠️ Prototype: Gate N auto-passed — [condensed artifact note].
```

Research and Design may be condensed. Full diagrams are optional (but at least one sequence diagram is required for cross-component flows). Tests can be smoke-level.

## Prototype Mode

Prototype mode is designed for throwaway experiments, spikes, and demos where speed matters more than rigor.

### How to activate

Say any of: `prototype`, `spike`, `sketch`, `demo`, `poc`, `throwaway`, or include `mode: prototype` in your prompt.

### What changes

| Aspect | Standard | Prototype |
|--------|----------|-----------|
| Gates | Manual user approval | Auto-pass with banner |
| Research | Full template | Bullet-list summary |
| Design | All diagrams required | ≥1 sequence diagram (cross-component flows) |
| Plan | Full phase list + AC | Abbreviated |
| Tests | Full suite | Smoke-level |

Everything still gets an entry in `gates.md` — the record is always maintained.

## How to Invoke

1. Select the **Orchestrator** agent with `/agent`.
2. Describe your feature request.
3. The orchestrator classifies it and either:
   - Bypasses the pipeline (trivial) and routes directly.
   - Announces Standard or Prototype mode and starts Phase 1.
4. Alternatively, use the prompt template directly: `/context-engineering`.

## Worked Example

**Feature request:** "Add rate limiting to the public API endpoints."

### Phase 0 — Triage
> Standard (non-trivial production change). Branch: `copilot/feat/rate-limiting`.

### Phase 1 — Research
`researcher` scans the codebase and produces:
- **Problem:** public endpoints have no rate limiting; risk of abuse.
- **Success criteria:** 429 returned after N req/min per IP; headers per RFC 6585.
- **Affected areas:** `src/api/middleware/`, `src/routes/`.
- **Existing patterns:** Express middleware chain in `src/api/app.ts`.
- **Options considered:** `express-rate-limit` (adopt), custom Redis sliding window (reject — overkill for MVP).
- **Constraints:** must not break auth-exempt health check endpoint.

Gate 1 → user: "approved"

### Phase 2 — Design
`code-investigator` produces `design.md`:
- Component diagram: Client → Nginx → RateLimiter middleware → Route handler.
- DFD: IP extraction → Redis lookup → allow/block decision.
- Sequence: happy path (allowed) + rate-limit exceeded (429 with headers).
- Contracts: `RateLimitConfig { windowMs, max, skipRoutes }`.
- Failure modes: Redis unavailable → fail-open (log + allow).

Gate 2 → user: "approved"

### Phase 3 — Plan
`orchestrator` produces `plan.md` with 3 phases:
- Phase 1: install `express-rate-limit`, configure Redis store.
- Phase 2: apply middleware to all public routes; skip health check.
- Phase 3: add integration tests (429 behavior, header values, bypass list).

Gate 3 → user: "approved"

### Phase 4 — Implement
Orchestrator runs Coder → Reviewer → Tester for each phase, appending results to `gates.md`. Final PR includes all code and CE artifacts.

---

## Related Files

| File | Purpose |
|------|---------|
| `.github/skills/context-engineering.skill.md` | Canonical pipeline spec, templates, gate protocol |
| `.github/skills/diagram-authoring.skill.md` | Mermaid diagram authoring guidance |
| `.github/prompts/context-engineering.prompt.md` | Workflow prompt template for the orchestrator |
| `.github/agents-config/context-engineering-workflow.md` | Orchestrator dispatch table and gate protocol reference |
