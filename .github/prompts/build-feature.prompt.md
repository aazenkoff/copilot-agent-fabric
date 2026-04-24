---
description: "Prompt for building a new feature end-to-end"
---

# Build Feature

## Context
I need to implement a new feature: **{{FEATURE_NAME}}**. The tasks can be delegated to different agents based on their expertise. The feature should be implemented according to the requirements and best practices.

## Requirements
{{FEATURE_REQUIREMENTS}}

## Workflow Note
- Use `/agent` to select the role for each step.
- For multi-step coordination, you can select the repo's **orchestrator** first and have it manage the workflow.
- Use `/fleet` only if some steps can run in parallel.
- Use `@` only for files and paths, not for agent selection.

## Steps
1. **Enter the Context Engineering Pipeline** — start with `/context-engineering` (`context-engineering.prompt.md`). The orchestrator will triage the request and either bypass (trivial) or run the full R→D→P→I pipeline. All non-trivial feature work enters here first.
2. **Code Investigator** — (CE Design phase) analyze the requirements and design the feature architecture.
3. **Orchestrator (optional)** — coordinate the overall plan and route work to the right specialists.
4. **Framework/engine specialist** (for Unity gameplay or systems, use **Unity Gameplay Developer**) or **Code Writer** — implement the feature on a `copilot/feat/<slug>` branch.
5. **Tester** — write tests for the new feature on the same branch.
6. **Code Reviewer** — review the implementation.
7. **Documenter** — update documentation.
8. **Implementing agent** — push the branch and create a PR via `gh pr create`, then report the PR URL for manual review and merge.

