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

## Steps
1. @tester — verify existing test coverage (add tests if missing)
2. @code-reviewer — identify code smells, plan and implement refactoring changes
3. @tester — verify all tests still pass
4. @code-reviewer — final review of the refactored code

