---
name: MCP Server Generation
description: Generate Model Context Protocol (MCP) servers with tool registration, manifest endpoint, and request handling
category: backend
tags: [mcp, ai-agents, protocols, tool-use]
stack: [python, fastapi]
---

# MCP Server Generation

Generate a compliant MCP (Model Context Protocol) server that allows AI agents to discover and use tools.

## Structure

```
mcp_server.py
├── MCPServer class
│   ├── register_tool(name, description, schema, handler)
│   ├── GET /mcp/manifest — returns tool catalog
│   └── POST /mcp/request — executes tool by name
```

## Key Patterns

- Tool registry as dict mapping name → {description, input_schema, handler}
- Manifest endpoint returns JSON with tools array (name, description, inputSchema)
- Request endpoint validates tool name, calls handler, returns result
- Error handling with proper MCP error codes

## Example

```python
class MCPServer:
    def __init__(self):
        self.tools = {}

    def register_tool(self, name, description, schema, handler):
        self.tools[name] = {"description": description, "input_schema": schema, "handler": handler}

    def manifest(self):
        return {"tools": [{"name": n, "description": t["description"], "inputSchema": t["input_schema"]} for n, t in self.tools.items()]}

    async def handle_request(self, tool_name, arguments):
        tool = self.tools.get(tool_name)
        if not tool:
            raise ValueError(f"Unknown tool: {tool_name}")
        return await tool["handler"](arguments)
```
