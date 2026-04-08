# Comfy Pilot Fork

See the original [Comfy Pilot](https://github.com/ConstantineB6/comfy-pilot) for full documentation.

## Changes Made

1. **Renamed endpoints** from `/claude-code/` to `/mcp/` for platform neutrality
2. **Added `mcp_config.json`** template for MCP configuration

## Why

The original uses `/claude-code/` endpoints which assume Claude Code as the client. This fork uses `/mcp/` so any MCP-compatible client (OpenCode, Claude Code, OpenClaw) can connect.

## License

MIT