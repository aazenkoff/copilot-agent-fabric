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
1. @tester — verify existing test coverage; add tests for any uncovered code paths to establish a safety net
2. @code-reviewer — identify code smells (duplication, long methods, deep nesting, tight coupling) and produce a prioritized refactoring plan
3. @code-writer — implement the refactoring changes from the plan, one pattern at a time (extract method, rename, decompose, etc.)
4. @tester — verify all tests still pass after each refactoring step
5. @code-reviewer — final review of the refactored code for quality, readability, and adherence to the plan

