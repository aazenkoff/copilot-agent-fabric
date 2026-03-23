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
1. **Code Investigator** — analyze the requirements and design the feature architecture.
2. **Orchestrator (optional)** — coordinate the overall plan and route work to the right specialists.
3. **Framework/engine specialist** (for Unity gameplay or systems, use **Unity Gameplay Developer**) or **Code Writer** — implement the feature on a `copilot/feat/<slug>` branch.
4. **Tester** — write tests for the new feature on the same branch.
5. **Code Reviewer** — review the implementation.
6. **Documenter** — update documentation.
7. **Implementing agent** — push the branch and create a PR via `gh pr create`, then report the PR URL for manual review and merge.
