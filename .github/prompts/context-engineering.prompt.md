---
description: "Workflow template for the Context Engineering pipeline: Research → Design → Plan → Implement with quality gates and prototype-mode escape hatch."
---

# Context Engineering Pipeline

> **Canonical spec:** `.github/skills/context-engineering.skill.md`
> **Human guide:** `docs/context-engineering.md`

## Parameters

```yaml
FEATURE_REQUEST: "<describe the feature or change>"
mode: standard  # or: prototype
```

---

## Step 0 — Triage (orchestrator)

Classify `{{FEATURE_REQUEST}}`:

| Trigger | Classification | Next step |
|---------|---------------|-----------|
| Typo, comment, single-line, dep bump, docs-only | **Trivial** | Bypass pipeline → delegate directly |
| User says "prototype/spike/sketch/demo/poc" or `mode: prototype` | **Prototype** | Pipeline with auto-pass gates |
| Anything else involving production code | **Standard** | Full pipeline, strict gates |

If **Trivial**: skip to the relevant specialist agent. This prompt does not apply.

If **Prototype**: set `MODE=prototype` — all gates auto-pass with the banner:
```
> ⚠️ Prototype: Gate N auto-passed.
```

Announce the classification and mode to the user before proceeding.

---

## Step 1 — Research (no code)

**Dispatch:** `researcher` (+ `code-investigator` for codebase recon)

**Task:** produce `docs/context-eng/<slug>/research.md` in the target project repo using the research template from `context-engineering` skill.

Minimum sections required:
- Problem restatement & success criteria
- Affected codebase areas (file/symbol map)
- Existing patterns/conventions to follow
- Prior art / library options considered
- Constraints (perf, security, platform, deps)
- Open questions (answered or deferred with user acknowledgment)

Also create `docs/context-eng/<slug>/README.md` (index file) and `docs/context-eng/<slug>/gates.md` (empty gate log) at this point.

**Gate 1 — Research Approval**

Post the `research.md` content (or link the file path) and ask:

> "**Gate 1 — Research complete.** Does this research look accurate and sufficient? Reply **approved** to proceed to Design, or describe any changes needed."

- **Standard mode:** wait for explicit user approval before Step 2.
- **Prototype mode:** auto-pass; append banner to `gates.md`.

---

## Step 2 — Design (no code)

**Dispatch:** `code-investigator` as Design lead (use `diagram-authoring` skill)

**Task:** produce `docs/context-eng/<slug>/design.md` in the target project repo using the design template from `context-engineering` skill.

Required diagrams:
- Architecture diagram (Mermaid C4 component or graph)
- Data Flow Diagram (Mermaid)
- ≥1 Sequence Diagram for the primary interaction

In **prototype mode**: full diagrams optional; at minimum one sequence diagram for any cross-component flow.

**Gate 2 — Design Approval**

Post the `design.md` content (or link) and ask:

> "**Gate 2 — Design complete.** Does this architecture look correct? Any contracts or failure modes missing? Reply **approved** to proceed to Planning, or describe changes needed."

- **Standard mode:** wait for explicit user approval.
- **Prototype mode:** auto-pass; append banner to `gates.md`.

---

## Step 3 — Plan (no code)

**Dispatch:** `orchestrator` (self)

**Task:** produce `docs/context-eng/<slug>/plan.md` in the target project repo using the plan template from `context-engineering` skill.

Plan requirements:
- Ordered, atomic phases (each independently implementable and testable)
- Each phase: ID, title, goal, files to touch, acceptance criteria, suggested coder agent, risk level, dependencies
- No time estimates (repo convention)

**Gate 3 — Plan Approval** *(most important gate)*

Post the `plan.md` content (or link) and ask:

> "**Gate 3 — Plan complete.** This plan is the implementation contract. Does the phase order look correct? Are acceptance criteria testable? Reply **approved** to begin implementation, or describe changes needed."

- **Standard mode:** wait for explicit user approval.
- **Prototype mode:** auto-pass; append banner to `gates.md`.

> Post-Gate-3 changes require appending an `## Amendment` block to `plan.md` and re-seeking approval for affected phases.

---

## Step 4 — Implement (code allowed)

**Branch:** `copilot/feat/<slug>` in the target project repo.

For each phase in `plan.md`, in order:

### Per-Phase Loop

1. **Dispatch Coder** — specialist or `code-writer` (per `plan.md` "Suggested agent"):
   - Read `plan.md` phase for acceptance criteria
   - Read `design.md` for contracts and architecture constraints
   - Implement only this phase; follow `disciplined-coding` skill
   - Commit: `feat(<slug>): phase-<N> — <title>`

2. **Dispatch `code-reviewer`**:
   - Review against `design.md` contracts + this phase's acceptance criteria
   - Not just code-quality heuristics — verify the implementation matches the design

3. **Coder applies feedback** (if any).

4. **Dispatch `tester`**:
   - Write and run tests for this phase
   - In prototype mode: smoke-level tests are acceptable
   - Record test evidence (output summary or link)

5. **Append to `gates.md`:**
   ```markdown
   ### Phase <N> — <Title>
   - **Coder:** <summary of changes>
   - **Reviewer notes:** <summary or "no issues">
   - **Tester evidence:** <test output summary>
   - **Status:** ✅ passed | ❌ blocked
   ```

6. Proceed to next phase if status is ✅; block and report if ❌.

---

## Final Step — PR

After all phases pass:

1. Ensure all CE artifacts are committed on the branch:
   - `docs/context-eng/<slug>/README.md`
   - `docs/context-eng/<slug>/research.md`
   - `docs/context-eng/<slug>/design.md`
   - `docs/context-eng/<slug>/plan.md`
   - `docs/context-eng/<slug>/gates.md`

2. Open PR:
   ```bash
   gh pr create \
     --title "feat: <feature name> (CE pipeline)" \
     --body "Implements <feature> via the Context Engineering pipeline.
   
   CE artifacts: docs/context-eng/<slug>/
   
   Closes #<issue> (if applicable)"
   ```

3. Report PR URL to user. Do NOT merge.

---

## MemPalace — Phase Mirroring

After each phase completes, mirror the artifact:
```
mempalace_add_drawer(
  key="<project>/<slug>/<phase>",
  content=<artifact summary>,
  tags=["ce-pipeline", "<slug>", "<phase>"]
)
```

Before starting each phase, query for prior runs:
```
mempalace_search(query="<slug> <phase>", wing=<project>)
```
