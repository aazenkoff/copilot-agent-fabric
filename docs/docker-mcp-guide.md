# Docker MCP Server Guide

## Overview

The Docker MCP (Model Context Protocol) server provides secure Docker container execution and file operations through Copilot CLI. It enables agents to interact with Docker containers directly during task execution.

## Setup

### Installation

The Docker MCP server is configured in `~/.copilot/mcp-config.json`:

```json
{
  "mcpServers": {
    "docker": {
      "command": "npx",
      "args": ["-y", "docker-mcp-server@latest"]
    }
  }
}
```

### Prerequisites
- Docker installed and running
- Node.js (for npx)
- GitHub Copilot CLI configured

## Available Tools

### Command Execution
| Tool | Description |
|------|-------------|
| `execute_command` | Run shell commands inside containers |
| `check_process` | Monitor background processes |
| `send_input` | Send input to running interactive processes |

### File Operations
| Tool | Description |
|------|-------------|
| `file_read` | Read files from containers |
| `file_write` | Write files to containers |
| `file_edit` | Modify existing files in containers |
| `file_ls` | List directory contents in containers |
| `file_grep` | Search file contents in containers |

## Quick Reference

### Running Commands
```
execute_command(container_name, "npm test")
```

### Reading Files
```
file_read(container_name, "/app/config.json")
```

### Writing Files
```
file_write(container_name, "/app/config.json", content)
```

### Checking Processes
```
check_process(container_name, process_id)
```

## Usage Patterns

### Container Debugging
1. List running containers
2. Execute diagnostic commands (`ps`, `top`, `netstat`)
3. Read log files
4. Check configuration files

### Database Operations
1. Connect to database container
2. Execute SQL queries
3. Export/import data
4. Check database status

### Application Deployment
1. Build application in container
2. Run database migrations
3. Start application services
4. Verify health checks

## Best Practices

- **Use specific container names** — avoid ambiguous references
- **Handle errors** — always check command exit codes
- **Clean up** — remove temporary files and stop unused containers
- **Security** — never pass secrets as command-line arguments; use environment variables
- **Timeouts** — set appropriate timeouts for long-running operations

## Which Agents Use Docker MCP?

| Agent | Use Case |
|-------|----------|
| **DevOps** | Container management, CI/CD, infrastructure |
| **Tester** | Running tests in isolated containers |

## Troubleshooting

### MCP server not starting
- Verify Docker is running: `docker info`
- Check Node.js is installed: `node --version`
- Ensure `~/.copilot/mcp-config.json` has the correct configuration

### Connection issues
- Restart the MCP server by restarting Copilot CLI
- Check Docker daemon connectivity
- Verify the container exists and is running

### Permission errors
- Ensure the Docker socket is accessible
- Check container user permissions for file operations
