---
name: A2A Handler Generation
description: Generate Google Agent-to-Agent (A2A) protocol handlers with agent card discovery and task lifecycle management
category: backend
tags: [a2a, ai-agents, protocols, google]
stack: [python, fastapi]
---

# A2A Handler Generation

Generate a handler implementing the Google Agent-to-Agent (A2A) protocol for agent interoperability.

## Structure

```
a2a_handler.py
├── GET /.well-known/agent.json — agent card (discovery)
├── POST /a2a/tasks — create task
├── GET /a2a/tasks/{id} — get task status
└── POST /a2a/tasks/{id}/complete — mark complete with result
```

## Agent Card

```json
{
  "name": "my_agent",
  "description": "Agent description",
  "capabilities": ["tool_use", "reasoning"],
  "endpoint": "https://agent.example.com/a2a",
  "version": "1.0.0"
}
```

## Task Lifecycle

1. **created** → task received
2. **running** → agent processing
3. **completed** → result available
4. **failed** → error occurred

## Key Patterns

- Agent card at well-known URL for discovery
- Task objects with id, status, input, output
- Async task execution with status polling
- Standard error responses with codes
