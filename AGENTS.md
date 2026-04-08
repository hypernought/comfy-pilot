# AGENTS.md - Agentic Coding Guidelines for Comfy Pilot

This file provides guidelines for agents working on the Comfy Pilot codebase.

## Project Overview

Comfy Pilot is an MCP server that lets any MCP client (OpenCode, Claude Code, OpenClaw) see and edit ComfyUI workflows. This is a fork of [ConstantineB6/comfy-pilot](https://github.com/ConstantineB6/comfy-pilot) with platform-neutral `/mcp/` endpoints.

### Repository Structure
```
comfy-pilot/
├── mcp_server.py      # MCP server (stdio transport)
├── __init__.py        # Plugin backend (REST endpoints: /mcp/*)
├── js/claude-code.js   # Frontend (workflow sync, MCP indicator)
├── mcp_config.json   # MCP configuration template
├── tests/            # Unit tests
├── pyproject.toml    # Project config (pytest, poetry)
└── README.md         # Fork docs (links to original)
```

## Build/Test Commands

### Running Tests
```bash
# Run all tests
pytest

# Run a single test file
pytest tests/test_edit_graph.py

# Run a specific test class
pytest tests/test_edit_graph.py::TestEditGraphInputParsing

# Run a specific test
pytest tests/test_edit_graph.py::TestEditGraphInputParsing::test_json_string_list

# Run with verbose output
pytest -v tests/test_edit_graph.py
```

### Installation
```bash
# Install as ComfyUI custom node
comfy node install comfy-pilot

# Or via ComfyUI Manager (search "Comfy Pilot")
```

## Code Style Guidelines

### Python Style
- Use **type hints** for function parameters and return types
- Use **4 spaces** for indentation (not tabs)
- Use **snake_case** for function/variable names, **PascalCase** for classes
- Maximum line length: 100 characters
- Add docstrings for public functions (Google-style)

### Imports
- Standard library imports first, then third-party, then local
- Group by: `json`, `socket`, `sys`, `urllib`, `typing`, `unittest` (stdlib)
- Use explicit relative imports for local modules

### Error Handling
- Return error dictionaries `{"error": "message"}` for API failures
- Catch specific exceptions (e.g., `urllib.error.URLError`)
- Provide descriptive error messages with context

### Naming
- Functions: `get_workflow()`, `edit_graph()`, `get_node_types()`
- Constants: `COMFYUI_URL`, `CACHE_TTL = 300`
- Classes: No classes currently in mcp_server.py (module uses functions)

### Code Organization
- Keep functions focused and small (< 50 lines)
- Group related functions into sections with comments
- Use `send_graph_command()` for frontend communication
- Use `make_request()` for ComfyUI API calls

## Key Patterns

### MCP Tool Functions
Each tool (e.g., `get_node_types`, `edit_graph`) should:
1. Accept typed parameters
2. Return string results (compact TOON-like format) or error dicts
3. Handle missing/invalid inputs gracefully

### Caching
- Use `_object_info_cache` with TTL for node type info
- Invalidate cache on errors

### Testing
- Mock `get_object_info_cached`, `send_graph_command`, `get_workflow`
- Use `@pytest.fixture` for mock setup
- Tests live in `tests/` directory

## Special Guidelines (from CLAUDE.md)

### Node Creation
- Always assign descriptive **titles** (e.g., "Positive Prompt" not "CLIPTextEncode")
- Use `edit_graph` with `place_in_view: true` for disconnected nodes
- Batch multiple operations in single `edit_graph` call

### Node Layout
- Position: Loaders left (pos_x ~100-300), processing middle (400-700), output right (800+)
- Minimum 20px padding between nodes
- Add 200px extra vertical spacing below Load 3D & Animation nodes

### Searching
- Use minimal search first (`get_node_types(search=["camera"])`)
- Request detailed fields (`inputs`, `outputs`) only for specific nodes you'll use

### Preview Results
- Connect outputs to Preview Image nodes for image data
- Run lightweight workflows automatically; ask before GPU-intensive ones

## Testing Best Practices

1. **Single test execution**: Use `pytest tests/test_edit_graph.py::TestClassName::test_name`
2. **Isolate tests**: Each test should be independent
3. **Mock external calls**: Patch network functions to avoid real HTTP requests
4. **Test input parsing**: Cover JSON string, list, dict variants
5. **Test validation**: Unknown node types, missing required fields

## Additional Resources

- `CLAUDE.md` - ComfyUI MCP plugin guidelines
- `README.md` - Project documentation (links to original)
- `.claude/skills/comfy-nodes/` - ComfyUI node development skills

## Fork Notes

This is a fork of the original Comfy Pilot. Key differences:
- Uses `/mcp/` endpoints instead of `/claude-code/` for platform neutrality
- Removed embedded terminal (use external MCP client instead)
- MCP indicator in UI shows connection status

## Development Workflow

1. Make changes in this repo (C:\Users\takad\comfy-pilot-mcp)
2. Push to remote: `git add -A && git commit -m "description" && git push`
3. Pull in ComfyUI custom_nodes: `cd U:\ComfyUI\custom_nodes\comfy-pilot && git pull`
4. Restart ComfyUI to apply changes