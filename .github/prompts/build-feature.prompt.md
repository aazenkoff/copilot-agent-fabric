---
description: "Prompt for building a new feature end-to-end"
---

# Build Feature

## Context
I need to implement a new feature: **{{FEATURE_NAME}}**. The tasks can be delegated to different agents based on their expertise. The feature should be implemented according to the requirements and best practices.

## Requirements
{{FEATURE_REQUIREMENTS}}

## Steps
1. @code-investigator — analyze the requirements and design the feature architecture
2. @orchestrator — break down the feature into tasks and assign them to the appropriate agents
3. @code-writer or an appropriate specialized agent — implement the feature on a `copilot/feat/<slug>` branch
4. @tester — write tests for the new feature (on the same branch)
5. @code-reviewer — review the implementation
6. @documenter — update documentation
7. @code-writer or implementing agent — push the branch and create a PR via `gh pr create`, report the PR URL to the user for manual review and merge

