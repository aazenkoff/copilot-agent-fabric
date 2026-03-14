---
description: "Analyze code structure, detect patterns, find issues, and understand dependencies."
name: Code Analysis
---

# Code Analysis Skill

## Capabilities
- **Static analysis** — detect code smells, anti-patterns, complexity
- **Dependency mapping** — understand import/require graphs
- **Pattern detection** — identify design patterns in use
- **Error detection** — find potential bugs, type issues, null references
- **Complexity analysis** — assess cyclomatic complexity and maintainability

## Analysis Dimensions
1. **Correctness** — does the code behave as intended?
2. **Maintainability** — is the code easy to understand and change?
3. **Performance** — are there obvious performance bottlenecks?
4. **Security** — are there common vulnerability patterns?
5. **Testability** — is the code structured for easy testing?

## Output Format
When reporting analysis results, use severity levels:
- 🔴 **Critical** — must fix, blocks deployment
- 🟡 **Warning** — should fix, potential issue
- 🟢 **Info** — suggestion for improvement
- ⚪ **Note** — observation, no action needed

## When to Use
- Before refactoring, to understand current state.
- During code review, to find issues systematically.
- When onboarding to a new codebase.

