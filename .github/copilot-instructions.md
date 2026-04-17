# Global Copilot Instructions

## Recommended Coordination Pattern

This repository defines a custom **orchestrator** agent as the preferred coordinator for complex, multi-step work. In GitHub Copilot CLI, users browse and select agents with `/agent`.

- Select **orchestrator** with `/agent` when you want one agent to coordinate specialist work.
- Select a specialist directly with `/agent` for focused single-domain tasks.
- For Unity implementation work, prefer Unity specialists over `code-writer`; use `unity-gameplay-developer` for gameplay/system work that should be validated live with Unity MCP.
- Use `/fleet` when parallel subagent execution would help.
- Use `@` to mention files and paths for context, not to invoke custom agents.

The orchestrator is a **repo convention and selectable workflow**, not an automatic CLI default.

---

## Project Overview
This workspace is an **AI Agent Management Environment** — a structured system for creating, orchestrating, and managing AI agents via GitHub Copilot CLI.

## Conventions
- All agent definitions live in `.github/agents/` as `<name>.agent.md` files.
- Shared prompts and templates live in `.github/prompts/`.
- Agent configuration and metadata live in `.github/agents-config/`.
- Skill definitions live in `.github/skills/` as `<name>.skill.md` files.
- Scripts for lifecycle management live in `scripts/`.
- Documentation lives in `docs/`.

## Documentation Access
- If MCP Ref is available, agents may use it to retrieve documentation sources.

## MemPalace (Persistent Memory)

Agents with the `mempalace-memory` skill have access to cross-session persistent memory via the MemPalace MCP server. See `docs/mempalace-guide.md` for the full integration guide and `.github/skills/mempalace-memory.skill.md` for the skill definition.

**Quick protocol:**
1. **Before starting work** — call `mempalace_search` with task-relevant keywords.
2. **After significant work** — call `mempalace_add_drawer` to store findings.
3. **For entity relationships** — use `mempalace_kg_query` / `mempalace_kg_add`.
4. **At session end** — call `mempalace_diary_write` to record what happened.

## Repository Protection Rules
- This repository is an **agent-config-only** workspace. It must NOT contain application source code, project files, or third-party dependencies.
- **Only the `agent-creator` agent is authorized to make changes** to files in this repository.
- All changes **must be submitted via Pull Request** — direct commits to the main branch are prohibited.
- Other agents (code-writer, tester, devops, etc.) must **never** modify files in this repository directly.
- If another agent identifies a needed repo change, it should stop and request the `agent-creator` workflow rather than editing directly.

## Agent Naming
- Use lowercase kebab-case for agent file names: `my-agent.agent.md`
- Use lowercase kebab-case for skill file names: `my-skill.skill.md`
- Use clear, descriptive names that reflect the agent's or skill's purpose.

## Agent Design Principles
1. **Single Responsibility** — each agent should have one clear purpose.
2. **Composability** — agents should be able to delegate to other agents and use skills.
3. **Observability** — agents should log decisions and actions.
4. **Guardrails** — agents must operate within defined boundaries.
5. **Idempotency** — repeated runs should produce consistent results.

## Model Preferences

When the orchestrator delegates work to **coding agents**, it must use `model: claude-sonnet-4.6`. This is the standard model for all code-writing, code-review, testing, and specialist implementation agents in this environment.

- **Coding agents** (require `claude-sonnet-4.6`): `code-writer`, `code-reviewer`, `code-investigator`, `tester`, and all engine/platform specialists (`pixijs-*`, `unity-*`, `godot-*`, `unreal-*`, `blender-addon-engineer`, `roblox-systems-scripter`, `game-audio-engineer`, `technical-artist`)
- **Non-coding agents** (no model override required): `researcher`, `documenter`, `game-designer`, `level-designer`, `narrative-designer`, `roblox-experience-designer`, `roblox-avatar-creator`

## PixiJS Reference

For any PixiJS task in this environment, treat `docs/pixijs-llms-reference.md` as required local guidance. It points to the official PixiJS LLM reference at https://pixijs.com/llms-full.txt and summarizes the v8-specific rules agents should follow here.

Before writing or reviewing PixiJS code:

1. Read `docs/pixijs-llms-reference.md`.
2. Prefer PixiJS v8 patterns such as async `Application` initialization, `app.canvas`, `Assets.load`, container-based scene organization, and ticker-driven updates.
3. Do not guess older PixiJS APIs unless the target project already uses them.

## Token Efficiency

All agents must minimize token consumption. Every token spent on unnecessary output is a token not available for useful work.

### Reading Files
- Use `view_range` or `grep` to read only the lines you need. Never read an entire large file when a targeted range or search will do.
- Do not re-read files that are already in your context window.
- Prefer `grep` with a glob pattern to narrow search scope before reading matches.

### Tool Calls
- Batch all independent tool calls into a single response. If you need to read 3 files, make 3 `view` calls in one turn — not 3 separate turns.
- Chain shell commands with `&&` instead of making separate calls.
- Suppress verbose output: use `--quiet`, `--no-pager`, pipe to `head` or `grep` when full output is not needed.

### Output
- Do not echo file contents back to the user — they already see tool output.
- Do not narrate what you are about to do. Just do it.
- Do not explain obvious steps or repeat the user's request back to them.
- Prefer short, direct answers. Elaborate only when the user asks for detail.
- When generating code, skip boilerplate comments like `// Constructor` or `// Import dependencies`. Only comment non-obvious logic.

### Investigation
- Stop investigating once you have enough information to act. Do not exhaustively search every file for completeness.
- If a search returns clear results, act on them immediately rather than running follow-up searches to confirm.
- Use the most specific search tool available: code intelligence > LSP > glob > grep > shell commands.

## Skill Design Principles
1. **Reusability** — skills are shared building blocks any agent can use.
2. **Atomicity** — each skill does one thing well.
3. **Declarative** — skills describe *what* to do, not *which agent* does it.
4. **Composable** — skills can be combined in prompt templates and agent instructions.

## Project Registry

Agents use a persistent **project registry** to remember which projects exist and where they live on disk.

- The registry is stored via `store_memory` with subject `project-registry`.
- Format: `Projects: name1=/path/to/project1, name2=/path/to/project2`
- Any agent that needs to work with code should check this registry **before** asking the user.
- The `project-context` skill defines the full read/write pattern.
- When using the orchestrator workflow, the orchestrator should populate the registry at the start of a session if it is not already set.

## Project Copilot Instructions

Every project has a `.github/copilot-instructions.md` file **inside the project repo** that GitHub Copilot automatically loads when working in that project.

These files are **mandatory context** for all agents working on code tasks. Before starting any coding, testing, reviewing, or infrastructure work on a project:

1. Look up the project path via the Project Registry (see above).
2. Read `<project-path>/.github/copilot-instructions.md`.
3. Use the architecture, conventions, types, and commands documented there.

### Creating Instructions
When a project is added to the registry, use the `agent-creator` workflow to create `.github/copilot-instructions.md` inside the target project repo via PR. Use the project's existing README and source structure as the basis.

### Enforcement
- Agents must **not** ask the user for information already in the instructions file.
- Agents must **not** start coding work without loading the instructions first.
- If no instructions file exists, agents should suggest creating one before proceeding.
