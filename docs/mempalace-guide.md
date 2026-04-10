# MemPalace Integration Guide

MemPalace gives agents persistent memory across sessions. Knowledge is stored as **drawers** organized into **wings** (projects) and **rooms** (topics), with a **knowledge graph** for entity relationships.

## Architecture

```mermaid
graph TB
    subgraph "Copilot CLI Session"
        O[Orchestrator] --> CW[Code Writer]
        O --> CI[Code Investigator]
        O --> UGD[Unity Gameplay Dev]
        O --> T[Tester]
    end

    subgraph "MemPalace MCP Server"
        MP[mempalace.mcp]
        DB[(ChromaDB Vector Store)]
        KG[(Knowledge Graph)]
        MP --> DB
        MP --> KG
    end

    CW -->|search / store| MP
    CI -->|search / store| MP
    UGD -->|search / store| MP
    T -->|search / store| MP
    O -->|search / diary| MP
```

## How It Works

1. **Before starting a task**, agents call `mempalace_search` to check for existing knowledge (past bug fixes, architecture decisions, gotchas).
2. **During work**, agents call `mempalace_kg_query` to understand entity relationships.
3. **After completing significant work**, agents call `mempalace_add_drawer` to store findings and `mempalace_kg_add` for new relationships.
4. **At session end**, agents call `mempalace_diary_write` to record a summary.

## Palace Structure

### Wings (Projects)

| Wing | Description |
|------|-------------|
| `tower-defense` | Tower Defense Unity game — architecture, economy, balance, bots, enemies, bugs, UI, editor tools |
| `scary-hotel` | ScaryHotel Unity game — architecture, NavMesh, SceneBootstrapper |
| `agent-fabric` | This repo — agent definitions, skills, conventions |
| `tooling` | Cross-project — environment, MCP config, project registry |

### Standard Rooms

| Room | Content |
|------|---------|
| `architecture` | Core systems, patterns, data flow |
| `economy` | Currency, costs, resource systems |
| `balance` | Tuning values, enemy stats, wave composition |
| `gameplay` | Mechanics, combat, interactions |
| `bots` | AI behavior, roles, pathfinding |
| `enemies` | Enemy types, spawning, behavior |
| `bugs` | Critical bugs and their fixes |
| `ui` | HUD, menus, input handling |
| `editor` | Editor tools, bootstrappers |
| `art` | Visual style, sprite generation |
| `timeline` | Development chronology |
| `environment` | Dev machine versions and paths |
| `mcp` | MCP server configuration |
| `agents` | Agent definitions and conventions |

## Which Agents Use It

The `mempalace-memory` skill is assigned to agents that benefit from cross-session knowledge:

| Agent | How It Uses MemPalace |
|-------|----------------------|
| **orchestrator** | Reads diary on session start; searches before delegating |
| **code-writer** | Searches for patterns and gotchas before implementing |
| **code-investigator** | Searches for known bugs; stores root cause findings |
| **code-reviewer** | Checks known patterns and past issues during review |
| **tester** | Searches for known failure patterns; stores test findings |
| **unity-gameplay-developer** | Searches Unity-specific gotchas; stores gameplay findings |
| **game-designer** | Searches balance data and past tuning decisions |

## MCP Configuration

MemPalace runs as a stdio MCP server. Config is in `~/.copilot/mcp-config.json`:

```json
{
  "mcpServers": {
    "mempalace": {
      "type": "stdio",
      "command": "C:\\Users\\User\\AppData\\Local\\...\\python.exe",
      "args": ["-m", "mempalace.mcp", "--palace-dir", "C:\\Users\\User\\.mempalace"]
    }
  }
}
```

The palace data lives at `~/.mempalace/` using ChromaDB for vector storage.

## Knowledge Graph

The KG stores typed relationships between entities:

```
Tower Defense → uses_engine → Unity6-URP-2D
Tower Defense → has_system → BFS grid pathfinding
Tower Defense → has_system → dual-currency-coins-energy
Tower Defense → has_system → bot-ai-builder-defender-runner
ScaryHotel → uses_engine → Unity6-URP-3D
ScaryHotel → reimplements → Tower Defense
```

Query with `mempalace_kg_query(entity="Tower Defense")` to see all relationships.

**KG naming rules:** Use kebab-case identifiers. No parentheses, backslashes, or special characters.

## Adding New Knowledge

### When to Store

- ✅ Bug root cause + fix that took real investigation
- ✅ Architecture decision with non-obvious rationale
- ✅ Integration gotcha (e.g., "URP 2D doesn't render sprites under mesh renderers")
- ✅ Balance values that took iteration
- ❌ Routine code changes
- ❌ Info already in `copilot-instructions.md`
- ❌ Temporary debugging notes

### Example

```
mempalace_add_drawer(
  wing="tower-defense",
  room="bugs",
  content="Door pass-through bug: enemies walked through doors because NavMesh obstacle carving radius was 0.1. Fixed by setting to 0.5 matching door width.",
  source_file="Assets/Scripts/DoorController.cs"
)
```
