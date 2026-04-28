---
description: "Break a plan, spec, or PRD into independently grabbable GitHub issues using vertical tracer-bullet slices."
name: To Issues
---

# To Issues Skill

> Adapted from `mattpocock/skills` at `383b6a06d59c4ce0ffcb14112bfd91265a86cf91` (MIT). See `docs/imported-skills.md`.

## Capabilities
- **Vertical slicing** — split work into thin end-to-end increments rather than horizontal layer tickets.
- **Dependency mapping** — identify blockers so issues can be created in dependency order.
- **HITL/AFK classification** — mark which slices require human decisions and which autonomous agents can implement.
- **Issue creation** — create concise GitHub issues with acceptance criteria when the target repo workflow supports it.

## Workflow
1. Use the current conversation, PRD, plan, or referenced GitHub issue as source material.
2. Explore the repo if needed to understand current architecture.
3. Draft tracer-bullet issues with title, type, blockers, and user stories covered.
4. Ask the user to approve granularity and dependencies.
5. Create issues in blocker-first order and reference parent/blocking issues where applicable.

## When to Use
- Turning a plan or PRD into implementation tickets.
- Preparing parallelizable work for multiple agents.

