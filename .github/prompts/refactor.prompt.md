---
description: "Prompt for refactoring and cleaning up technical debt"
---

# Refactor Code

## Context
The following area needs refactoring: **{{TARGET_AREA}}**

## Reason
{{REASON_FOR_REFACTORING}}

## Constraints
- Must not change existing behavior
- Existing tests must continue to pass

## Workflow Note
- Use `/agent` to select the role for each step.
- For multi-step coordination, you can choose the orchestrator workflow.
- Use `@` only for file and path mentions.

## Steps
1. **Enter the Context Engineering Pipeline** — start with `/context-engineering` (`context-engineering.prompt.md`). Refactors typically **skip Phase 1 (Research)** and begin at **Phase 2 (Design)**; note this in the CE prompt. Enter at Phase 1 only if the refactor scope is unclear and codebase context is needed first.
2. **Tester** — (before any changes) verify existing test coverage and add tests for uncovered code paths to establish a safety net.
3. **Code Reviewer** — (CE Design / Plan phases) identify code smells (duplication, long methods, deep nesting, tight coupling) and produce a prioritized refactoring plan.
4. **Framework/engine specialist** (for Unity gameplay or systems, use **Unity Gameplay Developer**) or **Code Writer** — implement the refactoring changes from the plan, one pattern at a time (extract method, rename, decompose, etc.).
5. **Tester** — verify all tests still pass after each refactoring step.
6. **Code Reviewer** — perform a final review of the refactored code for quality, readability, and adherence to the plan.

