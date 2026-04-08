# Comfy Pilot

[![Stars](https://img.shields.io/github/stars/hypernought/comfy-pilot)](https://github.com/hypernought/comfy-pilot/stargazers)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![ComfyUI Registry](https://img.shields.io/badge/ComfyUI-Registry-blue)](https://registry.comfy.org/publishers/constantine/nodes/comfy-pilot)

Talk to your ComfyUI workflows. Comfy Pilot gives any MCP-compatible client (OpenCode, Claude Code, OpenClaw, etc.) direct access to see, edit, and run your workflows — with an embedded terminal right inside ComfyUI.

![Comfy Pilot](thumbnail.jpg)

## Why?

Building ComfyUI workflows means manually searching for nodes, dragging connections, and tweaking values one at a time. With Comfy Pilot, you just describe what you want:

- *"Build me an SDXL workflow with ControlNet"* — Claude creates all the nodes, connects them, and sets the parameters
- *"Look at the output and increase the detail"* — Claude sees your generated image and adjusts the workflow
- *"Download the FLUX schnell model and set up a workflow for it"* — Claude downloads the model and builds a workflow from scratch

No copy-pasting node names. No hunting through menus. Just say what you want.

## Installation

**CLI (Recommended):**
```bash
comfy node install comfy-pilot
```

**ComfyUI Manager:**
1. Open ComfyUI
2. Click **Manager** → **Install Custom Nodes**
3. Search for "Comfy Pilot"
4. Click **Install**
5. Restart ComfyUI

**Git Clone:**
```bash
cd ~/Documents/ComfyUI/custom_nodes && git clone https://github.com/hypernought/comfy-pilot.git
```

**Using the Fork**: This fork uses `/mcp/` endpoints for OpenCode/OpenClaw compatibility. If using the original [ConstantineB6/comfy-pilot](https://github.com/ConstantineB6/comfy-pilot), it uses `/claude-code/` endpoints instead.

Claude Code CLI will be installed automatically if not found.

## Requirements

- ComfyUI
- Python 3.8+

## Features

- **MCP Server** - Gives any MCP client direct access to view, edit, and run your ComfyUI workflows
- **Embedded Terminal** - Full xterm.js terminal running Claude Code right inside ComfyUI
- **Image Viewing** - Claude can see outputs from Preview Image and Save Image nodes
- **Graph Editing** - Create, delete, move, and connect nodes programmatically

## Demo

https://github.com/user-attachments/assets/325b1194-2334-48a1-94c3-86effd1fef02

## Fork

This is a fork of [Comfy Pilot](https://github.com/ConstantineB6/comfy-pilot) with platform-neutral MCP support.

### Differences from Original

- **Generic MCP Endpoints**: The original uses `/claude-code/` endpoints which are tied to the Claude Code platform. This fork uses `/mcp/` endpoints for platform neutrality, allowing any MCP client (OpenCode, Claude Code, OpenClaw) to connect without platform-specific assumptions.

- **Platform-Agnostic Naming**: Refactored MCP tool names and configuration to work with any MCP-compatible client, not just Claude Code.

- **Extended MCP Setup Docs**: Added setup instructions for OpenCode, Claude Code, and OpenClaw in a unified document.

## Usage

1. Restart ComfyUI after installation
2. The floating terminal appears in the top-right corner (Unix/Linux only)
3. Configure your MCP client (see below)
4. Ask your MCP client to help with your workflow:
   - "What nodes are in my current workflow?"
   - "Add a KSampler node connected to my checkpoint loader"
   - "Run the workflow up to node 5"

## MCP Setup

The MCP server works with any MCP-compatible client. Configure it manually:

### OpenCode

Add to your `settings.json`:
```json
{
  "mcpServers": {
    "comfyui": {
      "command": "python",
      "args": ["U:/ComfyUI/custom_nodes/comfy-pilot/mcp_server.py"]
    }
  }
}
```

### Claude Code

Add to `~/.claude/settings.json`:
```json
{
  "mcpServers": {
    "comfyui": {
      "command": "python3",
      "args": ["/path/to/comfy-pilot/mcp_server.py"]
    }
  }
}
```

Or run: `claude mcp add comfyui python3 /path/to/mcp_server.py`

### OpenClaw

Add to your MCP configuration:
```json
{
  "mcpServers": {
    "comfyui": {
      "command": "python",
      "args": ["/path/to/mcp_server.py"]
    }
  }
}
```

**Note:** Ensure Python is in your PATH or use the full path to your Python executable.

## MCP Tools

The MCP server provides these tools to your MCP client:

| Tool | Description |
|------|-------------|
| `get_workflow` | Get the current workflow from the browser |
| `summarize_workflow` | Human-readable workflow summary |
| `get_node_types` | Search available node types with filtering |
| `get_node_info` | Get detailed info about a specific node type |
| `get_status` | Queue status, system stats, and execution history |
| `run` | Run workflow (optionally up to a specific node) or interrupt |
| `edit_graph` | Batch create, delete, move, connect, and configure nodes |
| `view_image` | View images from Preview Image / Save Image nodes |
| `search_custom_nodes` | Search ComfyUI Manager registry for custom nodes |
| `install_custom_node` | Install a custom node from the registry |
| `uninstall_custom_node` | Uninstall a custom node |
| `update_custom_node` | Update a custom node to latest version |
| `download_model` | Download models from Hugging Face, CivitAI, or direct URLs |

### Example: Creating Nodes

```
Create a KSampler and connect it to my checkpoint loader
```

Claude will use `edit_graph` to:
1. Create the KSampler node
2. Connect the MODEL output from CheckpointLoader to KSampler's model input
3. Position it appropriately in the graph

### Example: Viewing Images

```
Look at the preview image and describe what you see
```

Claude will use `view_image` to fetch and analyze the image output.

### Example: Downloading Models

```
Download the FLUX.1 schnell model for me
```

Claude will use `download_model` to download from Hugging Face to your ComfyUI models folder. Supports:
- Hugging Face (including gated models with token auth)
- CivitAI
- Direct download URLs

## Terminal Controls

- **Drag** title bar to move
- **Drag** bottom-right corner to resize
- **−** Minimize
- **×** Close
- **↻** Reconnect session

## Architecture

```
┌─────────────────────────────────────────────────────┐
│  Browser (ComfyUI)                                  │
│  ┌─────────────────┐  ┌──────────────────────────┐  │
│  │  xterm.js       │  │  Workflow State          │  │
│  │  Terminal       │  │  (synced to backend)     │  │
│  └────────┬────────┘  └────────────┬─────────────┘  │
│           │ WebSocket              │ REST API       │
└───────────┼────────────────────────┼────────────────┘
            │                        │
            ▼                        ▼
┌─────────────────────────────────────────────────────┐
│  ComfyUI Server                                     │
│  ┌─────────────────┐  ┌──────────────────────────┐  │
│  │  PTY Process    │  │  Plugin Endpoints        │  │
│  │  (claude CLI)   │  │  /mcp/*                  │  │
│  └─────────────────┘  └──────────────────────────┘  │
└─────────────────────────────────────────────────────┘
            │                        │
            │                        ▼
            │           ┌──────────────────────────┐
            └──────────▶│  MCP Server              │
                        │  (stdio transport)       │
                        └──────────────────────────┘
```

## Files

- `__init__.py` - Plugin backend: WebSocket terminal, REST endpoints
- `js/claude-code.js` - Frontend: xterm.js terminal, workflow sync
- `mcp_server.py` - MCP server for any MCP-compatible client
- `mcp_config.json` - MCP configuration template
- `CLAUDE.md` - Instructions for AI assistants when working with ComfyUI

## Troubleshooting

### "Command 'claude' not found"

Install Claude Code CLI:

**macOS / Linux / WSL:**
```bash
curl -fsSL https://claude.ai/install.sh | bash
```

**Windows (PowerShell):**
```powershell
irm https://claude.ai/install.ps1 | iex
```

**Windows (CMD):**
```cmd
curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd && del install.cmd
```

### MCP server not connecting

The plugin auto-configures MCP on startup. Check ComfyUI console for errors, or manually add to `~/.claude.json`:

```json
{
  "mcpServers": {
    "comfyui": {
      "command": "python3",
      "args": ["/path/to/comfy-pilot/mcp_server.py"]
    }
  }
}
```

### Terminal disconnected

Click the ↻ button to reconnect, or check ComfyUI console for errors.

## License

MIT
