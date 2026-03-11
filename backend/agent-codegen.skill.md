---
name: Agent Code Generation
description: Generate complete AI agent Python packages from a spec (tools, framework, protocols) ready to deploy
category: backend
tags: [ai-agents, code-generation, mcp, a2a, deployment]
stack: [python]
---

# Agent Code Generation

Generate a complete, deployable AI agent Python package from a specification object.

## Input Spec

```python
class AgentSpec(BaseModel):
    name: str
    description: str
    tools: list[str]       # e.g. ["web_search", "database_query"]
    framework: str         # "react-loop" or "plan-execute"
    use_mcp: bool
    use_a2a: bool
```

## Generated File Tree

```
{name}/
├── agent.py           # Main agent loop (ReAct or Plan-Execute)
├── tools.py           # Tool implementations
├── config.py          # AgentConfig dataclass
├── prompts.py         # System prompts
├── __init__.py
├── __main__.py        # CLI entry point
├── mcp_server.py      # MCP server (if use_mcp)
├── a2a_handler.py     # A2A handler (if use_a2a)
├── requirements.txt
├── Dockerfile
├── README.md
├── ARCHITECTURE.md
├── API.md
└── DEPLOYMENT.md
```

## Frameworks

- **ReAct Loop**: Think → Act → Observe cycle, good for quick responses
- **Plan-Execute**: Plan all steps first, then execute sequentially, good for complex research

## Key Patterns

- Template-based generation (no AI call needed — instant)
- Tools mapped from string names to implementation stubs
- Generated agents are standalone — no runtime dependency on the platform
- Includes deployment configs for Docker, Kubernetes, and Serverless
