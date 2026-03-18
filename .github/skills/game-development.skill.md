---
description: "Coordinate the full PixiJS game-development agent team to build, fix, and test a game end-to-end across design, architecture, implementation, backend, and QA."
name: Game Development
---

# Game Development Skill

This skill activates the full game-development agent pipeline. It is used by the `orchestrator` whenever a task spans multiple game-development domains: design, scene architecture, implementation, art generation, backend wiring, testing, and quality review.

## Capabilities

- **Build new game features** — design, architect, implement, and test any feature from mechanic spec to working code
- **Fix gameplay bugs** — investigate root cause, apply fix, validate with tests, and run quality gate
- **Add or regenerate art assets** — generate UI art, backgrounds, icons, and textures using the `openai-image-generation` skill
- **Redesign screens or game flows** — update scene graphs, layouts, and transitions end-to-end
- **Add narrative or level content** — integrate lore, dialogue, level flow, and encounter pacing into the game
- **Run a full QA pass** — test coverage, visual QA, build validation, and code review across all changed components
- **Coordinate multi-agent parallel work** — dispatch design, art, and backend agents simultaneously to maximise throughput

## Team Composition

| Agent | Role in game development |
|-------|--------------------------|
| `game-designer` | Writes the GDD, mechanic specs, player experience goals, progression tuning, and design acceptance criteria |
| `narrative-designer` | Authors lore, dialogue scripts, branching narrative structure, and story beats |
| `level-designer` | Defines level flow, encounter pacing, spatial readability plans, and zone briefs |
| `pixijs-architect` | Designs PixiJS scene graphs, layer boundaries, entity-component patterns, state machines, and asset management strategy |
| `pixijs-prototype-specialist` | Implements PixiJS scenes, generates art assets (via `openai-image-generation`), and runs visual QA against reference screenshots |
| `code-writer` | Wires backend APIs, data models, game-state persistence, and server-side logic |
| `tester` | Writes and runs unit, integration, and e2e tests to validate gameplay loops and feature behaviour |
| `code-reviewer` | Enforces quality gates: security, performance, correctness, and code standards before any PR is merged |
| `orchestrator` | Coordinates all agents, tracks phase completion, synthesises outputs, and manages PRs |

## Workflow

Work proceeds in **six sequential phases**. Each phase has an explicit exit gate that must pass before the next phase begins. Phases 2 and 3 may run **in parallel** when the architecture spec is complete.

### Phase 0 — Design
**Owner:** `game-designer` (+ `narrative-designer`, `level-designer` as needed)

1. `game-designer` creates or updates the GDD, mechanic spec, and player experience goals.
2. `narrative-designer` adds story beats and dialogue structure if the feature involves narrative.
3. `level-designer` adds a level brief or encounter plan if the feature involves level content.
4. Deliverable: a reviewed, approved design spec document (stored in `plan.md` or the project's docs directory).

**Exit gate:** Design spec reviewed and approved by the orchestrator (or the user) before Phase 1 begins.

---

### Phase 1 — Architecture
**Owner:** `pixijs-architect`

1. `pixijs-architect` reads the design spec.
2. Designs the PixiJS scene graph, layer boundaries, state machines, and asset management strategy.
3. Documents the architecture in a brief technical spec (scene hierarchy, data flow, asset list).

**Exit gate:** Architecture spec reviewed against the design spec; no unresolved conflicts.

---

### Phase 2 — Implementation *(can run parallel with Phase 3)*
**Owner:** `pixijs-prototype-specialist`

1. Reads the architecture spec and design spec.
2. Generates required art assets using the `openai-image-generation` skill (see Art Style Anchor section below).
3. Implements PixiJS scenes, containers, interactions, and animations.
4. Runs visual QA: takes browser screenshots and compares against reference images or design intent.
5. Runs the frontend build gate:
   ```bash
   npm run typecheck && npm run build
   ```

**Exit gate:** Build passes (`npm run typecheck && npm run build` green) + visual QA screenshots match reference.

---

### Phase 3 — Backend *(can run parallel with Phase 2)*
**Owner:** `code-writer`

1. Reads the design spec and API contracts from the architecture spec.
2. Implements backend APIs, data models, game-state persistence, and any server-side logic.
3. Runs the server test gate:
   ```bash
   ./gradlew test
   ```
   *(adjust to the project's actual backend test command — e.g., `npm test`, `pytest`, `go test ./...`)*

**Exit gate:** All backend tests pass.

---

### Phase 4 — Testing & Quality Gate
**Owners:** `tester` then `code-reviewer`

1. `tester` writes or extends unit, integration, and e2e tests covering the new feature and its edge cases.
2. `tester` runs the full test suite and confirms all tests pass.
3. `code-reviewer` performs a full quality review: security, performance, correctness, and standards compliance.
4. `code-reviewer` must give a sign-off (explicit "approved" verdict) before the PR is created.

**Exit gate:** All tests pass + `code-reviewer` sign-off.

---

### Phase 5 — PR & Merge
**Owner:** `orchestrator`

1. Orchestrator confirms all phase exit gates are met.
2. Creates (or consolidates) the PR using `gh pr create` with a conventional commit message.
3. Reports the PR URL to the user.
4. Leaves the PR **open** — never auto-merges. The user reviews and merges.

---

## When to Use This Skill

Use the full game-development pipeline when the task touches **two or more** of: design, art, implementation, backend, or QA.

| Task type | When to use this skill |
|-----------|------------------------|
| New game feature (mechanic, screen, or flow) | ✅ Full pipeline — all phases |
| Visual redesign of an existing screen | ✅ Phases 1–5 (skip Phase 0 if design spec already exists) |
| Art pipeline task (new icons, backgrounds, textures) | ⚠️ Art-only subset — `pixijs-prototype-specialist` only |
| Gameplay bug fix | ⚠️ Bug-fix subset — `code-investigator` + `code-writer` + `tester` + `code-reviewer` |
| Narrative or dialogue content only | ⚠️ Content subset — `narrative-designer` + `code-writer` (if wiring needed) |
| Level design document | ⚠️ Design subset — `level-designer` only |
| Backend API or data model only | ⚠️ Backend subset — `code-writer` + `tester` + `code-reviewer` |
| Full game vertical slice or new game project | ✅ Full pipeline — all phases, all agents |

## Agent Selection Guidance

### Full team (all agents)
Use when building a complete new feature, redesigning a major screen, or starting a new game project.

### Art-only task
Only `pixijs-prototype-specialist` is needed. Skip Phases 0, 1, and 3.
Provide a clear art brief: subject, dimensions, style anchor, and target directory.

### Bug fix
1. `code-investigator` — root-cause analysis and investigation report
2. `code-writer` — apply the fix
3. `tester` — add a regression test
4. `code-reviewer` — quality gate

### Backend-only task
1. `code-writer` — implement API or data model changes
2. `tester` — validate with tests
3. `code-reviewer` — quality gate

### Design-only task
Dispatch `game-designer`, `narrative-designer`, or `level-designer` as appropriate.
No code agents needed unless the design immediately feeds into implementation.

## Quality Gates (Summary)

| Phase | Gate | Command / Criterion |
|-------|------|---------------------|
| Phase 0 — Design | Design spec approved | Reviewed by orchestrator or user |
| Phase 1 — Architecture | Architecture reviewed against spec | No unresolved conflicts |
| Phase 2 — Implementation | Frontend build + visual QA | `npm run typecheck && npm run build` ✅ + screenshot match |
| Phase 3 — Backend | Server tests | `./gradlew test` ✅ (or project equivalent) |
| Phase 4 — Testing | Full suite + reviewer sign-off | All tests green + `code-reviewer` approved |
| Phase 5 — PR | PR created, left open | PR URL reported to user |

## Art Style Anchor

All AI art generation tasks **must use the project's locked style anchor phrase** to maintain visual cohesion across all generated assets.

### Where to find the style anchor
1. Check `plan.md` in the project root — the style anchor is documented in the art/visual section.
2. Check the project's `.github/copilot-instructions.md` — it may contain a `## Art Style` section.
3. If no style anchor exists, ask the `game-designer` to define one and store it in `plan.md` before any art generation begins.

### Style anchor format
```
<theme>, <art style>, <color palette>, <mood/atmosphere>, "no text, no watermarks"
```

**Example:**
> `"dark knight RPG game UI, iron and steel aesthetic, dark fantasy, dark navy blue and steel grey, moody atmospheric lighting, no text, no watermarks"`

**Rule:** Never generate art without a style anchor. Inconsistent art styles across assets break visual cohesion and require regeneration — this costs more time than defining the anchor upfront.

## Best Practices

1. **Design first, build second** — never start implementation without a reviewed design spec. Undocumented assumptions cause rework across all phases.
2. **Art anchor before art generation** — lock the style anchor phrase before any `openai-image-generation` call. All assets in a project must share the same anchor.
3. **Run phases in order, parallelise safely** — Phases 2 and 3 can run in parallel only after Phase 1 (architecture) is complete. Never skip phases or run them out of order.
4. **Quality gate before proceeding** — each exit gate must be explicitly confirmed, not assumed. A "probably fine" is not a pass.
5. **One PR per feature** — keep all changes for a feature in a single branch and PR. Avoid partial PRs that leave the codebase in a broken state.
6. **Never merge your own PRs** — all PRs are left open for user review and merge. Report the PR URL to the user and stop.
7. **Propagate context forward** — each phase must receive the outputs of the previous phase as explicit context. Agents must not re-derive design decisions that have already been made.

## When to Use
- Building a new game feature end-to-end (mechanic, screen, flow, or system)
- Running a full visual redesign of an existing game screen
- Generating and integrating a batch of new art assets
- Fixing a gameplay or rendering bug with full test coverage
- Adding narrative content, dialogue, or level data to a game
- Starting a new PixiJS game project or vertical slice from scratch
- Performing a full QA and quality gate pass on a game feature branch
