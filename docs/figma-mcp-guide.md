# Figma MCP Server Guide

## Overview

The Figma MCP (Model Context Protocol) server enables **design-to-code workflows** through Copilot CLI. It provides tools for extracting design context, capturing screenshots, managing Code Connect mappings, working with design variables, and generating diagrams in FigJam.

Key capabilities:
- **Design Context Extraction** — get reference code, screenshots, and metadata from Figma designs
- **Asset & Variable Export** — extract colors, fonts, spacings, and downloadable assets
- **Code Connect** — map Figma components to code components bidirectionally
- **Visual QA** — capture Figma screenshots for side-by-side comparison with browser output
- **FigJam Diagrams** — generate flowcharts, sequence diagrams, gantt charts, and state diagrams

## Setup

### Installation

The Figma MCP server is configured in `~/.copilot/mcp-config.json`:

```json
{
  "mcpServers": {
    "figma": {
      "command": "npx",
      "args": ["-y", "@anthropic/figma-mcp@latest"]
    }
  }
}
```

> **Note**: The exact package name may vary. Check the [MCP server registry](https://modelcontextprotocol.io/) for the latest Figma MCP package.

### Prerequisites
- A Figma account with API access
- Node.js (for npx)
- GitHub Copilot CLI configured
- Authenticated Figma session (verify with `figma-whoami`)

## Available Tools

### Design Context & Screenshots

| Tool | Description | Key Parameters |
|------|-------------|----------------|
| `figma-get_design_context` | **Primary design-to-code tool.** Returns reference code, screenshot, and contextual metadata | `nodeId`, `fileKey` |
| `figma-get_screenshot` | Generate a screenshot for a given Figma node | `nodeId`, `fileKey` |
| `figma-get_metadata` | Get node structure in XML format (IDs, types, names, positions, sizes) | `nodeId`, `fileKey` |
| `figma-get_variable_defs` | Get design variable definitions (colors, fonts, sizes, spacings) | `nodeId`, `fileKey` |

### Code Connect

| Tool | Description |
|------|-------------|
| `figma-get_code_connect_map` | Get existing node→component mappings |
| `figma-get_code_connect_suggestions` | AI-suggested Code Connect mappings for a node |
| `figma-send_code_connect_mappings` | Save multiple approved Code Connect mappings in bulk |
| `figma-add_code_connect_map` | Map a single Figma node to a code component |

### FigJam

| Tool | Description |
|------|-------------|
| `figma-get_figjam` | Generate UI code from FigJam nodes (FigJam files only) |
| `figma-generate_diagram` | Create Mermaid.js diagrams in FigJam (flowcharts, sequence, gantt, state) |

### Utility

| Tool | Description |
|------|-------------|
| `figma-create_design_system_rules` | Generate design system rules for the current repo |
| `figma-whoami` | Check the authenticated Figma user and verify permissions |

## URL Parsing

All Figma MCP tools require a `fileKey` and `nodeId`. Extract these from Figma URLs:

### Design Files

```
https://figma.com/design/:fileKey/:fileName?node-id=1-2
```
- **fileKey** = `:fileKey` (the path segment after `/design/`)
- **nodeId** = `1:2` (convert `1-2` from the URL to `1:2` for API calls)

### Branch Files

```
https://figma.com/design/:fileKey/branch/:branchKey/:fileName
```
- **fileKey** = use `:branchKey` (not the main fileKey)

### FigJam Files

```
https://figma.com/board/:fileKey/:fileName?node-id=1-2
```
- Same extraction pattern as design files, but use `figma-get_figjam` instead of `figma-get_design_context`

### Figma Make Files

```
https://figma.com/make/:makeFileKey/:makeFileName
```
- **fileKey** = `:makeFileKey`

### Quick Reference

| URL Pattern | fileKey Source | Tool |
|-------------|---------------|------|
| `/design/:key/...` | `:key` | `figma-get_design_context` |
| `/design/:key/branch/:branch/...` | `:branch` | `figma-get_design_context` |
| `/board/:key/...` | `:key` | `figma-get_figjam` |
| `/make/:key/...` | `:key` | `figma-get_design_context` |

## Usage Patterns

### Design-to-Code Workflow

The standard workflow for implementing a UI from a Figma design:

1. **Extract design context**: Call `figma-get_design_context` with the `nodeId` and `fileKey` from the Figma URL
2. **Review the output**: Examine the returned reference code, screenshot, and metadata
3. **Adapt to your stack**: Modify the reference code to match your project's framework, component library, and conventions
4. **Download assets**: Use the returned asset URLs to download images, icons, and other resources
5. **Visual QA**: Compare your implementation against the Figma design (see below)

### Visual QA Loop

Combine Figma MCP with Chrome DevTools MCP for pixel-perfect comparison:

1. **Capture browser screenshot**: `chrome-devtools-take_screenshot` of your implementation
2. **Capture Figma screenshot**: `figma-get_screenshot` of the design node
3. **Compare side-by-side**: Identify visual discrepancies
4. **Iterate**: Adjust CSS/layout and repeat until the implementation matches the design

### Code Connect Setup

Establish and maintain mappings between Figma components and code components:

1. **Get suggestions**: Call `figma-get_code_connect_suggestions` with a root node to get AI-suggested mappings
2. **Review mappings**: Examine the suggested `nodeId → component` mappings
3. **Save approved mappings**: Call `figma-send_code_connect_mappings` with the approved mappings
4. **Add individual mappings**: Use `figma-add_code_connect_map` for one-off mappings

### Design System Extraction

Extract design tokens from Figma for use in CSS custom properties or theme files:

1. **Get variable definitions**: Call `figma-get_variable_defs` with a node to retrieve colors, fonts, sizes, and spacings
2. **Map to tokens**: Convert Figma variables to CSS custom properties, SCSS variables, or theme tokens
3. **Generate rules**: Call `figma-create_design_system_rules` to produce design system rules for the repo

### Exploring a Figma File

When you don't know the file structure yet:

1. **Get metadata first**: Call `figma-get_metadata` with the page node (e.g., `0:1`) to see the full tree structure
2. **Identify target nodes**: Browse the XML output for the node names and IDs you need
3. **Get design context**: Call `figma-get_design_context` on specific nodes of interest

### FigJam Diagram Generation

Create architecture or workflow diagrams in FigJam:

1. **Write Mermaid syntax**: Prepare a flowchart, sequence diagram, gantt chart, or state diagram
2. **Generate the diagram**: Call `figma-generate_diagram` with the Mermaid syntax
3. **Share the link**: The returned URL can be opened directly in Figma

## Best Practices

- **Always prefer `figma-get_design_context`** — it's the primary design-to-code tool; use `figma-get_metadata` only when you need to explore file structure
- **Extract `fileKey` and `nodeId` from URLs first** — parse Figma URLs before calling any tool
- **Use `1:2` format for nodeId** — Figma URLs use `1-2` but API calls expect `1:2`
- **Maintain Code Connect mappings** — keep design-to-code mappings up to date as components evolve
- **Combine with Chrome DevTools MCP** — use both servers together for visual QA workflows
- **Check authentication first** — run `figma-whoami` if you encounter permission errors
- **Use `excludeScreenshot` sparingly** — screenshots provide essential visual context; only exclude to save tokens
- **Don't use `figma-get_figjam` on design files** — it only works on FigJam board files

## Which Agents Use Figma MCP?

| Agent | Use Case |
|-------|----------|
| **PixiJS Prototype Specialist** | Figma-to-PixiJS prototype workflows, asset extraction, visual QA loops |
| **Technical Artist** | Asset export workflows, render-pipeline guidance, and visual fidelity handoff |
| **Code Writer** | Design-to-code implementation, component scaffolding from designs |
| **Code Reviewer** | Visual QA comparison against design specifications |
| **Documenter** | FigJam diagram generation for architecture documentation |

## Troubleshooting

### Permission errors
- Run `figma-whoami` to verify authentication status
- Ensure your Figma account has access to the target file
- Check that the MCP server is properly configured in `~/.copilot/mcp-config.json`

### Node not found
- Verify the `nodeId` format: use `1:2` (colon-separated), not `1-2` (dash-separated)
- Use `figma-get_metadata` to explore the file structure and find valid node IDs
- Make sure the `fileKey` is correct (for branch files, use the branch key)

### Empty design context
- Try `figma-get_metadata` first to verify the node exists and has content
- The node may be empty or hidden in the Figma file
- Check that you're using the correct `fileKey` for branch files

### FigJam tools returning errors on design files
- `figma-get_figjam` only works on FigJam files (`/board/` URLs), not design files (`/design/` URLs)
- Use `figma-get_design_context` for standard Figma design files

### MCP server not starting
- Verify Node.js is installed: `node --version`
- Ensure `~/.copilot/mcp-config.json` has the correct configuration
- Restart Copilot CLI to reload MCP server configuration

### Large files or slow responses
- Use `excludeScreenshot: true` to reduce response size when screenshots aren't needed
- Target specific nodes rather than entire pages
- Use `figma-get_metadata` to narrow down which nodes to fetch
