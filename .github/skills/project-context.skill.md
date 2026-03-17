---
description: "Read and write the persistent project registry so agents know which projects exist and where they live on disk."
name: project-context
---

# Project Context Skill

## Purpose
Maintain a persistent, named list of the user's projects and their local paths. Any agent that needs to work with code should check this registry first.

## When to Use
- Before working on any code task (writing, testing, reviewing, investigating, deploying)
- When the user mentions a project by name but no path is known
- When navigating the file system to find source code

## How to Use

### 1. Check Memory First
Before asking the user, always check if a project registry already exists in memory (subject: `project-registry`). If it exists, parse the stored fact and use those paths.

### 2. Ask the User (if no registry exists)
If no project registry is found in memory, ask the user using `ask_user`:
- Ask for a named list of projects and their local paths.
- Example: "Please provide your projects as a list, e.g.: `my-api=/Users/me/Develop/my-api, frontend=/Users/me/Develop/frontend`"
- Accept freeform text input (`allow_freeform: true`).

### 3. Store the Registry
After collecting project info, call `store_memory` with:
- `subject`: `project-registry`
- `category`: `user_preferences`
- `fact`: `Projects: name1=/path/to/project1, name2=/path/to/project2`
- `reason`: `Agents need to know which projects exist and their paths to work effectively without asking the user repeatedly.`
- `citations`: `User input: provided by user`

### 4. Use the Registry
When delegating or starting work on a project:
- Parse the stored fact to extract project names and paths.
- Pass the relevant project path as context when delegating to specialist agents.
- If the user refers to a project by name, look it up in the registry.

## Updating the Registry
If the user provides a new project or updates a path, call `store_memory` again with the full updated list (overwrite, don't append).

## Best Practices
- Only ask once per session — check memory before prompting.
- If only one project is in the registry and the task doesn't specify which project, use it automatically.
- If multiple projects exist and the task doesn't specify which, ask the user to pick one using `ask_user` with `choices` populated from the registry.
