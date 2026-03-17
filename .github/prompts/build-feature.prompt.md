---
description: "Prompt for building a new feature end-to-end"
---

# Build Feature

## Context
I need to implement a new feature: **{{FEATURE_NAME}}**. The tasks can be delegated to different agents based on their expertise. The feature should be implemented according to the requirements and best practices.

## Requirements
{{FEATURE_REQUIREMENTS}}

## Steps
1. **Code Investigator** — analyze the requirements and design the feature architecture.
2. **Orchestrator (optional)** — coordinate the overall plan and route work to the right specialists.
3. **Code Writer** or another suitable specialist — implement the feature on a `copilot/feat/<slug>` branch.
4. **Tester** — write tests for the new feature on the same branch.
5. **Code Reviewer** — review the implementation.
6. **Documenter** — update documentation.
7. **Implementing agent** — push the branch and create a PR via `gh pr create`, then report the PR URL for manual review and merge.
