---
description: "Query and store persistent project knowledge using the MemPalace MCP server for cross-session memory."
name: mempalace-memory
---

# MemPalace Memory Skill

## Purpose
Give agents persistent, searchable memory across sessions. MemPalace stores knowledge as **drawers** organized into **wings** (projects) and **rooms** (topics), with a **knowledge graph** for entity relationships and a **diary** for session journaling.

## When to Use

### Read (before starting work)
- Before investigating a bug — search for known bugs and past fixes.
- Before implementing a feature — check for architecture decisions and conventions.
- Before reviewing code — recall project patterns and gotchas.
- When the user asks "what do you know about X" — search, don't guess.

### Write (after completing work)
- After fixing a critical bug — store the root cause and solution.
- After making an architecture decision — store the rationale.
- After discovering a gotcha or pattern — store it so future sessions know.
- At the end of a significant session — write a diary entry summarizing what happened.

## Available MCP Tools

| Tool | Purpose | When |
|------|---------|------|
| `mempalace_search` | Semantic search across all drawers | Before any task — check what's already known |
| `mempalace_kg_query` | Query entity relationships | When you need facts about a project, system, or concept |
| `mempalace_kg_timeline` | Chronological fact history | When you need to understand how something evolved |
| `mempalace_add_drawer` | Store new knowledge | After completing significant work |
| `mempalace_kg_add` | Add entity relationship | When you discover a new project/system relationship |
| `mempalace_kg_invalidate` | Mark a fact as no longer true | When architecture changes, bugs are fixed, etc. |
| `mempalace_diary_write` | Write agent session diary | At the end of significant sessions |
| `mempalace_diary_read` | Read past diary entries | At session start to recall what happened recently |
| `mempalace_status` | Palace overview | To check what wings/rooms exist |
| `mempalace_list_wings` | List all wings | To see available project wings |
| `mempalace_check_duplicate` | Check if content exists | Before adding a drawer to avoid duplicates |

## Wing and Room Convention

Wings map to **projects**. Rooms map to **topic areas** within a project.

| Wing | Project |
|------|---------|
| `tower-defense` | Tower Defense Unity game |
| `scary-hotel` | ScaryHotel Unity game |
| `agent-fabric` | This agent management repo |
| `tooling` | Cross-project environment and tools |

Standard rooms (use these consistently):

| Room | Content |
|------|---------|
| `architecture` | Core systems, patterns, data flow |
| `economy` | Currency, costs, resource systems |
| `balance` | Tuning values, stats, wave composition |
| `gameplay` | Mechanics, combat, interactions |
| `bots` | AI behavior, roles, pathfinding |
| `enemies` | Enemy types, spawning, behavior |
| `bugs` | Critical bugs found and fixed |
| `ui` | HUD, menus, input handling |
| `editor` | Editor tools, bootstrappers |
| `art` | Visual style, sprite generation |
| `timeline` | Development chronology |
| `environment` | Dev machine, versions, paths |
| `mcp` | MCP server configuration |
| `projects` | Project registry |
| `agents` | Agent definitions and conventions |

## How to Use

### 1. Search Before Acting

```
mempalace_search(query="defeat flow freeze bug", wing="tower-defense")
```

Always check for existing knowledge before investigating from scratch. If the palace has the answer, use it. If not, proceed with normal investigation and store findings afterward.

### 2. Store Significant Findings

```
mempalace_add_drawer(
  wing="tower-defense",
  room="bugs",
  content="BeginNewGame must ONLY call SceneManager.LoadScene — no manual cleanup. BFS re-computation on half-destroyed objects caused editor freezes during defeat flow.",
  source_file="Assets/Scripts/Game.cs"
)
```

**What to store:**
- Bug root causes and their fixes
- Architecture decisions and rationale
- Gotchas that aren't obvious from the code
- Balance/tuning values that took iteration to find
- Integration patterns (e.g., "URP 2D doesn't render SpriteRenderers under MeshRenderers")

**What NOT to store:**
- Routine code changes
- Temporary debugging notes
- Information that's already in `copilot-instructions.md`

### 3. Update the Knowledge Graph

```
mempalace_kg_add(
  subject="Tower Defense",
  predicate="has_system",
  object="dual-currency-coins-energy"
)
```

Use kebab-case identifiers for objects. Avoid parentheses, backslashes, or special characters in KG values — they are rejected.

### 4. Write Diary Entries

At the end of a significant session, summarize what happened:

```
mempalace_diary_write(
  agent_name="unity-gameplay-developer",
  entry="Implemented wave 6 boss with phased behavior. Fixed turret targeting priority to prefer bosses. Tuned boss HP to 12000 after three playtest rounds.",
  topic="wave-system"
)
```

### 5. Check for Duplicates

Before adding a drawer, optionally check if similar content already exists:

```
mempalace_check_duplicate(content="...", threshold=0.85)
```

## Protocol for Agents

1. **On task start** — call `mempalace_search` with keywords relevant to the task. Check if past sessions already solved a similar problem.
2. **During investigation** — call `mempalace_kg_query` on entities you're working with (project names, system names) to understand relationships.
3. **After significant work** — call `mempalace_add_drawer` to store findings. Call `mempalace_kg_add` if you discovered new entity relationships.
4. **On session end** — call `mempalace_diary_write` to record what happened.

## Best Practices

- **Verify before trusting** — palace content may be outdated. Cross-reference with actual code.
- **Be specific** — "Game.cs line 296 defeat timer uses Update not coroutine" is better than "defeat flow works a certain way."
- **One concept per drawer** — don't dump everything into one drawer. Separate concerns.
- **Use source_file** — always include the relevant file path when storing code knowledge.
- **Invalidate stale facts** — if you find something in the palace that's no longer true, call `mempalace_kg_invalidate`.
