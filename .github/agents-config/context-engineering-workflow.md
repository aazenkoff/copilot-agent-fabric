# Context Engineering Workflow

> This file is referenced by `.github/agents/orchestrator.agent.md`.
> Canonical pipeline spec: `.github/skills/context-engineering.skill.md`
> Workflow prompt: `.github/prompts/context-engineering.prompt.md`

## Default Workflow: Context Engineering Pipeline

For any **non-trivial code change**, the orchestrator runs the CE pipeline before delegating implementation work.

### Triviality Classifier

Classify the user's request before dispatching any agent:

| Verdict | Criteria | Action |
|---------|----------|--------|
| **Trivial** | Typo · comment-only · single-line tweak · dep bump · docs-only edit | Skip pipeline — delegate directly via Delegation Decision Tree |
| **Prototype** | User says "prototype / spike / sketch / demo / poc" or request contains `mode: prototype` | Run pipeline with **relaxed gates** (auto-pass with banner) |
| **Standard** | Any production code change not in the above two categories | Run **full pipeline** with strict gates |

Announce the classification to the user before proceeding.

### Prototype-Mode Detection

Activate prototype mode when any of these tokens appear in the user's message or prompt:
`prototype`, `spike`, `sketch`, `demo`, `throwaway`, `poc`, `mode: prototype`

In prototype mode, all gates auto-pass and a banner is appended to `gates.md`:
```
> ⚠️ Prototype: Gate N auto-passed — [condensed artifact note].
```

### Per-Phase Agent Dispatch Table

| Phase | Lead Agent | Support Agents | Artifact |
|-------|-----------|----------------|---------|
| 0 — Triage | `orchestrator` | — | classification announcement |
| 1 — Research | `researcher` | `code-investigator` (codebase recon) | `research.md` |
| 2 — Design | `code-investigator` | `documenter` (diagram polish, optional) | `design.md` |
| 3 — Plan | `orchestrator` | — | `plan.md` |
| 4 — Implement (per phase) | `orchestrator` (sequences) | Coder + `code-reviewer` + `tester` | code + `gates.md` |

Coder selection per plan phase (use `plan.md` "Suggested agent" field):
- Unity project → `unity-gameplay-developer` or matching Unity specialist
- PixiJS project → `pixijs-prototype-specialist` or `pixijs-architect`
- General → `code-writer`

### Gate Enforcement Protocol

```
1. orchestrator: post artifact content or file path
2. orchestrator: "Gate N complete. Approve to proceed, or describe changes."
3. user:         "approved" | "changes: ..."
4.   if approved  → proceed to next phase
5.   if changes   → dispatch lead agent to revise → return to step 1
```

**Never auto-approve in strict mode.** Never proceed to the next phase without explicit user approval.

**Gate 3 (Plan) is the most important gate.** An approved plan is the implementation contract. Post-Gate-3 changes require a plan amendment.

### Trigger: When to Enter CE Pipeline

Check `.github/prompts/` first:
- Feature work → `context-engineering.prompt.md` is the entry point (supersedes `build-feature.prompt.md` for Standard/Prototype requests)
- Refactor work → CE pipeline starting at **Phase 2 — Design** (skip Research unless the refactor scope is unclear)
- Bug fix with known root cause → CE pipeline starting at **Phase 3 — Plan**
- Bug fix with unknown root cause → full pipeline from Phase 1

### Artifact Paths

All artifacts live in the **target project repo**:
```
docs/context-eng/<feature-slug>/
├── README.md   (index + gate log summary)
├── research.md
├── design.md
├── plan.md
└── gates.md
```

Create the directory and all files at the start of Phase 1.
