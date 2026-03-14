---
description: "Prompt for creating a new specialized agent"
---

# Create New Agent

## Agent Details
- **Name**: {{AGENT_NAME}}
- **Purpose**: {{AGENT_PURPOSE}}
- **Category**: {{CATEGORY}} (meta | development | quality | documentation | operations | analysis)

## Steps
1. @agent-creator — create the agent definition file at `.github/agents/{{AGENT_NAME}}.agent.md`
2. Register the agent in `.github/agents-config/registry.yaml`
3. Update the orchestrator's agent table in `.github/agents/orchestrator.agent.md`
4. @documenter — update the README with the new agent

