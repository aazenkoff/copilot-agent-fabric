# Agent Usage Guide

## Table of Contents
- [Getting Started](#getting-started)
- [Using Agents](#using-agents)
- [Understanding Skills](#understanding-skills)
- [Using Prompt Templates](#using-prompt-templates)
- [Creating Custom Agents](#creating-custom-agents)
- [Creating Custom Skills](#creating-custom-skills)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)

## Getting Started

### Prerequisites
- GitHub Copilot CLI installed and authenticated
- Access to a GitHub Copilot subscription with agent support
- This repository cloned locally

### How It Works
Each `.agent.md` file in `.github/agents/` defines a specialized custom agent for GitHub Copilot CLI. Use `/agent` to browse and select an agent. Use `@` to mention files or paths for context.

## Using Agents

### Selecting an Agent

Use `/agent` to browse and select the agent you want to work with, then send your task.

```text
/agent
# select code-writer
Create a Python function that validates email addresses in @src/utils.py
```

```text
/agent
# select tester
Write pytest tests for the email validator in @src/utils.py
```

```text
/agent
# select code-reviewer
Review @src/auth/handler.py for security issues
```

### File Mentions

Use `@` for files and paths only.

```text
Review @README.md and @docs/architecture.md for clarity
```

Do **not** use `@code-writer`, `@tester`, or similar syntax to invoke agents.

### Orchestrated Workflows

For complex tasks, this repo recommends selecting the **orchestrator** with `/agent` and using it as the coordinator.

```text
/agent
# select orchestrator
Build a user authentication system with JWT tokens based on @docs/requirements/auth.md
```

The orchestrator can:
1. Break down the task
2. Route work to specialized agents
3. Coordinate the workflow
4. Synthesize the results

This is a **repo workflow convention**, not an automatic CLI default.

### Parallel Subagents with `/fleet`

Use `/fleet` when the work naturally splits into independent tracks.

Good examples:
- Research + implementation planning
- Review + test-gap analysis
- Multiple independent investigation tasks

`/fleet` enables parallel subagent execution. It does **not** mean “use the orchestrator.” You can use `/fleet` with or without the orchestrator workflow.

## Understanding Skills

Skills are **reusable capabilities** defined in `.github/skills/`. They are the building blocks that agents use to accomplish tasks.

## Code Quality Workflow

All code changes follow this mandatory workflow:

**Developer Agent → Code Review → Fix Issues → Verify → Complete**

This ensures consistent code quality across the codebase.

### How It Works

1. A developer agent (code-writer, tester) writes code
2. The developer or coordinating agent requests a code review from the code-reviewer agent
3. The code-reviewer analyzes the code and provides feedback (including structural improvements)
4. The developer agent applies the suggested improvements
5. Tests are re-run to ensure nothing broke
6. Only then is the work marked as complete

### Agents vs Skills

Think of it like a team:
- **Agent** = a team member with a specific role (e.g., "Code Reviewer")
- **Skill** = a tool or technique they're trained in (e.g., "Security Audit")

Multiple agents can share the same skill. For example, both the **Code Reviewer** and **Researcher** agents use the `code-analysis` skill, but they use it differently based on their role.

### How Skills Work

Skills are **not invoked directly** by users. Instead:
1. A skill defines capabilities, best practices, and guidelines.
2. Agents reference skills via the registry (`registry.yaml`).
3. When an agent is selected, it applies its assigned skills' knowledge.

### Skill Categories

| Category | Skills | Purpose |
|----------|--------|---------|
| **Core** | file-operations, terminal-commands | Foundational capabilities every agent can use |
| **Development** | code-generation, code-analysis, dependency-management | Building and understanding code |
| **Quality** | security-audit | Ensuring security and compliance |
| **Documentation** | markdown-generation | Creating well-structured docs |
| **Operations** | docker, ci-cd | Infrastructure and deployment |
| **Research** | web-search | Information gathering |
| **Data** | database-operations | Schema design, migrations, query optimization |
| **API** | api-design | REST/GraphQL patterns, OpenAPI, versioning |
| **Frontend** | frontend-frameworks, landing-page-creation, pixijs | React/Vue/Angular patterns plus landing-page creation and PixiJS game/prototype patterns for responsive, conversion-aware UI |
| **Observability** | observability | Logging, metrics, distributed tracing, alerting |
| **Performance** | performance-optimization | Profiling, caching, load testing, benchmarking |
| **Testing** | testing-infrastructure | Test data, fixtures, Testcontainers, test pyramid |

## MCP Servers

**Model Context Protocol (MCP)** servers provide specialized tools that extend agent capabilities beyond the core Copilot CLI features.

### What are MCP Servers?

MCP servers are external processes that expose tools to Copilot CLI. They enable agents to:
- Interact with complex external systems (browsers, containers, databases)
- Perform specialized operations (performance profiling, container management)
- Access capabilities not available through standard CLI tools

### Available MCP Servers

#### Chrome DevTools MCP
**Purpose**: Browser automation, web testing, and performance analysis

**Use cases**:
- Automated UI testing
- Web scraping and data extraction
- Lighthouse audits for performance/accessibility
- Screenshot capture and visual testing

#### Docker MCP
**Purpose**: Container management and operations

**Capabilities**:
- **Command Execution**: Run commands inside containers
- **File Operations**: Read, write, edit files in containers
- **Process Management**: Check running processes, send input to interactive shells

**Use cases**:
- DevOps automation workflows
- Container debugging and inspection
- Isolated test environment setup
- Multi-container orchestration

#### Figma MCP
**Purpose**: Design-to-code workflows, asset extraction, visual QA, and Code Connect mapping

**Capabilities**:
- **Design Context**: Extract layout, styles, and reference code from Figma designs
- **Screenshots**: Capture Figma node screenshots for visual comparison
- **Code Connect**: Map Figma components to code components bidirectionally
- **Variables**: Extract design tokens (colors, fonts, spacings)
- **FigJam**: Generate diagrams using Mermaid.js syntax

**Use cases**:
- Building UI components from Figma designs
- Visual QA comparing implementation vs. design
- Extracting design tokens for theming
- Maintaining design-code mappings with Code Connect
- Creating architecture diagrams in FigJam

### How Agents Use MCP Servers

MCP tools are available to all agents, but certain agents are better suited to leverage specific servers:

| MCP Server | Primary Agents | When to Use |
|------------|----------------|-------------|
| **Chrome DevTools** | Tester, Researcher | UI testing, web analysis, performance audits |
| **Docker** | DevOps, Tester | Container management, isolated environments |
| **Figma** | PixiJS Prototype Specialist, Technical Artist, Code Writer, Documenter | Design-to-code, visual QA, asset extraction, diagram generation |

### Learn More

- **Docker MCP**: See [Docker MCP Guide](docker-mcp-guide.md)
- **Figma MCP**: See [Figma MCP Guide](figma-mcp-guide.md)
- **Configuration**: MCP servers are configured in `~/.copilot/mcp-config.json`

## Using Prompt Templates

Prompt templates in `.github/prompts/` are reusable workflows:

| Template | File | Use Case |
|----------|------|----------|
| Build Feature | `build-feature.prompt.md` | End-to-end feature development |
| Review Code | `review-code.prompt.md` | Comprehensive code review |
| Create Agent | `create-agent.prompt.md` | Add a new agent to the system |
| Setup Infra | `setup-infra.prompt.md` | Configure CI/CD and infrastructure |
| Refactor | `refactor.prompt.md` | Clean up technical debt |
| Setup Database | `setup-database.prompt.md` | Database infrastructure, schema, and data access layer |
| Deploy to Production | `deploy-to-production.prompt.md` | Production deployment with rollback and monitoring |
| Setup Auth | `setup-auth.prompt.md` | Authentication system (registration, login, tokens) |

These templates describe **roles and workflow steps**. Use `/agent` to select the needed agent for each step, or select the orchestrator if you want a single coordinating agent to manage the workflow.

## Creating Custom Agents

### Step-by-Step

1. **Copy the template**:
   Copy `.github/agents-config/agent-template.md` to `.github/agents/<your-agent>.agent.md`

2. **Define the agent**:
   - Set the YAML front matter (`name`, `description`)
   - Write clear responsibilities
   - Define guidelines and constraints
   - Specify the expected output format

3. **Register the agent**:
   Add an entry in `.github/agents-config/registry.yaml`

4. **Update the orchestrator**:
   Add the agent to the table in `.github/agents/orchestrator.agent.md`

### Agent Design Tips
- Keep the description concise — it's shown in agent selection UI
- Be specific about what the agent should and should NOT do
- Include examples of expected output format
- Define clear boundaries to prevent scope creep
- List which skills the agent needs in the registry

## Creating Custom Skills

### Step-by-Step

1. **Copy the template**:
   Copy `.github/agents-config/skill-template.md` to `.github/skills/<your-skill>.skill.md`

2. **Define the skill**:
   - Set the YAML front matter (`name`, `description`)
   - List the capabilities the skill provides
   - Write best practices and guidelines
   - Define when to use this skill

3. **Register the skill**:
   Add an entry in `.github/agents-config/registry.yaml` under the `skills:` section

4. **Assign to agents**:
   Add the skill name to the `skills:` list of relevant agents in the registry

### Skill Design Tips
- A skill should describe a *capability*, not a *role*
- Keep skills atomic — one skill = one type of capability
- Include "When to Use" to help agents and coordinators decide applicability
- Prefer reusing existing skills over creating overlapping ones
- If two skills overlap significantly, merge them

## Best Practices

1. **Use `/agent` deliberately** — choose the specialist or orchestrator that matches the job
2. **Use `@` only for files** — mention paths, specs, and diffs for context
3. **Use the orchestrator for multi-step work when helpful** — it is the repo's recommended coordinator
4. **Use `/fleet` for parallelizable work** — especially independent research, review, or investigation tracks
5. **Keep agents focused** — create new agents rather than overloading existing ones

## Troubleshooting

### Agent not responding as expected
- Check the agent file has valid YAML front matter
- Verify the agent is registered in `registry.yaml` with `status: active`
- Review the agent instructions for conflicting guidelines

### Agent not found in `/agent`
- Ensure the file is in `.github/agents/` with `.agent.md` extension
- Check the file name matches what you're expecting
- Confirm it is registered and active in `registry.yaml`

### Poor quality output
- Add more specific guidelines to the agent
- Include examples of desired output
- Add constraints to prevent common mistakes
