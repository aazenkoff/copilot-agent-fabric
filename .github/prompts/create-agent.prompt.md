---
description: "Prompt for creating a new specialized agent"
---

# Create New Agent

## Agent Details
- **Name**: {{AGENT_NAME}}
- **Purpose**: {{AGENT_PURPOSE}}
- **Category**: {{CATEGORY}} (meta | development | quality | documentation | operations | analysis)

## Workflow Note
- Use `/agent` to select **agent-creator** for the repo changes.
- Keep repository edits with **agent-creator**; other agents may be consulted, but they should not modify this repo directly.
- Use `@` only for file/path mentions.

## Steps
1. **Agent Creator** — create the agent definition file at `.github/agents/{{AGENT_NAME}}.agent.md`.
2. **Agent Creator** — register the agent in `.github/agents-config/registry.yaml`.
3. **Agent Creator** — update the orchestrator's agent table in `.github/agents/orchestrator.agent.md`.
4. **Agent Creator** — update the README if the new agent should be listed for users.
