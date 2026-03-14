---
description: "Generate code following project patterns, language conventions, and industry best practices."
name: Code Generation
---

# Code Generation Skill

## Capabilities
- **Generate functions/classes** — write new code modules
- **Implement patterns** — apply design patterns appropriate to the language
- **Scaffold projects** — create boilerplate and project structure
- **Generate boilerplate** — create repetitive but necessary code

## Best Practices
1. Follow the existing project's coding style and conventions.
2. Use meaningful, descriptive names for variables, functions, and classes.
3. Include error handling for all external interactions.
4. Add JSDoc/docstrings/comments for public APIs.
5. Prefer composition over inheritance.
6. Keep functions small and focused (single responsibility).
7. Use dependency injection where appropriate.

## Language-Aware Rules
- Detect the project language from existing files (package.json, requirements.txt, pom.xml, etc.).
- Follow the idiomatic patterns for that language.
- Use the project's existing formatter/linter configuration.

## When to Use
- Implementing new features or endpoints.
- Creating utility functions or helper modules.
- Scaffolding new components or services.

