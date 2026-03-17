# Global Copilot Instructions

## Default Agent: Orchestrator

**All user requests are handled by the `orchestrator` agent by default.**

When a user sends a message, the Orchestrator receives it first and decides how to handle it:
- Simple, single-domain tasks → delegate directly to the best-suited specialist agent
- Complex, multi-step tasks → break down and dispatch multiple agents (in parallel where possible)
- Infrastructure tasks → delegate to `devops`
- Code writing → delegate to `code-writer` or `game-developer`
- Testing → delegate to `tester`
- Reviews → delegate to `code-reviewer`
- Documentation → delegate to `documenter`

The Orchestrator **never does the work itself** — it always delegates to the right specialist.

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

## Repository Protection Rules
- This repository is an **agent-config-only** workspace. It must NOT contain application source code, project files, or third-party dependencies.
- **Only the `agent-creator` agent is authorized to make changes** to files in this repository.
- All changes **must be submitted via Pull Request** — direct commits to the main branch are prohibited.
- Other agents (code-writer, tester, devops, etc.) must **never** modify files in this repository directly.
- If a non-agent-creator agent needs a repo change, it must request the change through the orchestrator, who will delegate to `agent-creator`.

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
- The Orchestrator is responsible for populating the registry at the start of a session if it is not already set.
