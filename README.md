# AI Agent Management Environment

A structured workspace for defining, coordinating, and maintaining custom agents for GitHub Copilot CLI. This repository contains **only agent configuration** — no application source code should be stored here.

## 🏗️ Architecture

```
ai-env-public0/
├── .github/
│   ├── agents/                        # Agent definitions (who does the work)
│   │   ├── orchestrator.agent.md      # 🎯 Optional coordinator for complex workflows
│   │   ├── agent-creator.agent.md     # 🆕 Agent & skill creator
│   │   ├── code-writer.agent.md       # ✍️  Code implementation
│   │   ├── code-reviewer.agent.md     # 🔍 Code review & refactoring
│   │   ├── documenter.agent.md        # 📝 Documentation
│   │   ├── devops.agent.md            # ⚙️  Infrastructure
│   │   ├── researcher.agent.md        # 🔬 Research & analysis
│   │   ├── tester.agent.md            # 🧪 Testing
│   │   └── code-investigator.agent.md # 🔎 Bug investigation
│   ├── skills/                        # Skill definitions (reusable capabilities)
│   │   ├── file-operations.skill.md   # 📂 Read, write, search files
│   │   ├── terminal-commands.skill.md # 💻 Execute shell commands
│   │   ├── code-generation.skill.md   # 🏗️ Generate code
│   │   ├── code-analysis.skill.md     # 🔎 Analyze code structure
│   │   ├── web-search.skill.md        # 🌐 Search the web
│   │   ├── security-audit.skill.md    # 🛡️ Security scanning
│   │   ├── dependency-management.skill.md # 📦 Manage packages
│   │   ├── markdown-generation.skill.md   # 📄 Generate Markdown
│   │   ├── docker.skill.md            # 🐳 Container management
│   │   └── ci-cd.skill.md             # 🔄 CI/CD pipelines
│   ├── agents-config/                 # Configuration & metadata
│   │   ├── registry.yaml              # Central agent & skill registry
│   │   ├── agent-template.md          # Template for new agents
│   │   └── skill-template.md          # Template for new skills
│   ├── prompts/                       # Reusable prompt templates
│   │   ├── build-feature.prompt.md
│   │   ├── review-code.prompt.md
│   │   ├── create-agent.prompt.md
│   │   ├── setup-infra.prompt.md
│   │   └── refactor.prompt.md
│   └── copilot-instructions.md        # Global instructions
├── docs/                              # Documentation
│   ├── architecture.md                # System architecture
│   ├── agent-guide.md                 # How to use agents & skills
│   ├── docker-mcp-guide.md            # Docker MCP server usage
│   ├── figma-mcp-guide.md             # Figma MCP server usage
└── README.md                          # This file
```

## 🚀 Quick Start

### Prerequisites

- [GitHub Copilot CLI](https://docs.github.com/en/copilot/using-github-copilot/using-github-copilot-in-the-command-line) installed and authenticated
- A GitHub Copilot subscription with agent support

### Using Agents in Copilot CLI

- Use `/agent` to browse and select an agent.
- Use `@` to mention files or paths for context, not to invoke agents.
- Use `/fleet` when you want parallel subagent execution for independent work.

Example flow:

```text
/agent
# select code-writer
Implement a login endpoint in @src/auth/
```

```text
/agent
# select tester
Write unit tests for @src/services/user-service.ts
```

### Recommended Coordination Pattern

For complex, multi-step work, this repo recommends selecting the **orchestrator** with `/agent` and using it as the coordinator.

```text
/agent
# select orchestrator
Build a user authentication system with JWT tokens for @docs/requirements/auth.md
```

This is a **repo convention**, not a Copilot CLI default. The orchestrator is a selectable custom agent that can break down work, route to specialists, and synthesize results when you choose to use it.

## 🎯 Available Agents

| Agent | Category | Key Skills |
|-------|----------|------------|
| **Orchestrator** | Meta | Coordination |
| **Agent Creator** | Meta | — |
| **Code Writer** | Development | code-generation, dependency-management, database-operations, api-design, frontend-frameworks, pixijs, performance-optimization |
| **Code Reviewer** | Quality | code-analysis, security-audit, api-design, database-operations, frontend-frameworks, pixijs |
| **Game Developer** | Development | code-generation, code-analysis, dependency-management, pixijs |
| **Tester** | Quality | code-generation, terminal-commands, testing-infrastructure, performance-optimization |
| **Documenter** | Documentation | markdown-generation |
| **DevOps** | Operations | docker, ci-cd, terminal-commands, observability |
| **Researcher** | Analysis | web-search, code-analysis |
| **Code Investigator** | Analysis | code-analysis, file-operations |

## 🛠️ Available Skills

| Skill | Purpose |
|-------|---------|
| file-operations | Read, write, search, navigate files |
| terminal-commands | Execute shell commands, run scripts |
| code-generation | Generate code following patterns |
| code-analysis | Analyze structure, detect issues |
| web-search | Search for docs and best practices |
| security-audit | Check vulnerabilities and secrets |
| dependency-management | Manage deps, check CVEs |
| markdown-generation | Generate Markdown with diagrams |
| docker | Dockerfiles, compose, containers |
| ci-cd | Pipelines and GitHub Actions |
| database-operations | Schema design, migrations, query optimization |
| api-design | REST/GraphQL patterns, OpenAPI, versioning |
| frontend-frameworks | React/Vue/Angular patterns, state management, a11y |
| pixijs | PixiJS game/prototype patterns for logical resolution, viewport fitting, safe areas, assets, and touch UI |
| observability | Logging, metrics, distributed tracing, alerting |
| performance-optimization | Profiling, caching, load testing, benchmarking |
| testing-infrastructure | Test data, fixtures, Testcontainers, test pyramid |

## 📋 Code Quality Workflow

All code changes follow a **mandatory code review cycle**:

**Developer Agent → Code Review → Fix Issues → Verify → Complete**

This ensures:
- ✅ Consistent code quality across all contributions
- 🎓 Continuous learning from code review feedback
- 🛡️ Issues caught before merge
- 📈 Improved long-term maintainability

See [Architecture Guide](docs/architecture.md#code-quality-workflow) for the detailed workflow diagram.

## 🔌 MCP Server Support

This environment supports [Model Context Protocol (MCP)](https://modelcontextprotocol.io/) servers for extended capabilities:

- **Chrome DevTools MCP** — Browser automation, UI testing, performance audits
- **Docker MCP** — Container management, isolated environments
- **Figma MCP** — Design-to-code workflows, asset extraction, visual QA, Code Connect

See [Docker MCP Guide](docs/docker-mcp-guide.md) and [Figma MCP Guide](docs/figma-mcp-guide.md) for setup and usage.

## 📖 Documentation

| Document | Description |
|----------|-------------|
| [Architecture](docs/architecture.md) | System design and coordination model |
| [Agent Guide](docs/agent-guide.md) | How to select, use, create, and customize agents |
| [Docker MCP Guide](docs/docker-mcp-guide.md) | Docker MCP server setup and reference |
| [Figma MCP Guide](docs/figma-mcp-guide.md) | Figma MCP server setup and reference |

## 🎨 Customization

### Adding a New Agent

Use `/agent` to select **agent-creator**, then ask for the new agent you want. If relevant, mention supporting files with `@`.

Or manually:
1. Copy `.github/agents-config/agent-template.md` → `.github/agents/<name>.agent.md`
2. Define responsibilities, guidelines, and output format
3. Register in `.github/agents-config/registry.yaml`
4. Update the orchestrator's agent table

### Adding a New Skill

1. Copy `.github/agents-config/skill-template.md` → `.github/skills/<name>.skill.md`
2. Define capabilities and best practices
3. Register in `.github/agents-config/registry.yaml`
4. Assign to relevant agents

See the [Agent Guide](docs/agent-guide.md) for detailed instructions.

## 📄 License

MIT
