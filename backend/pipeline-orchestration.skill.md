---
name: Pipeline Orchestration
description: Compose agents into DAG pipelines with conditional routing, parallel fan-out, transforms, and human review gates
category: backend
tags: [pipelines, dag, orchestration, conditional-routing, agent-chaining]
stack: [python, fastapi]
---

# Pipeline Orchestration

Execute multi-agent workflows as directed acyclic graphs (DAGs) with conditional branching, parallel execution, and data transforms.

## Pipeline Config Schema

```python
pipeline_config = {
    "nodes": [
        {"id": "n1", "type": "agent", "label": "Email Triage", "agent_id": "..."},
        {"id": "n2", "type": "condition", "label": "Urgency Check",
         "config": {"field": "status", "operator": "equals", "value": "urgent"}},
        {"id": "n3", "type": "agent", "label": "Escalation Handler"},
        {"id": "n4", "type": "agent", "label": "Standard Response"},
    ],
    "edges": [
        {"source": "n1", "target": "n2"},
        {"source": "n2", "target": "n3", "label": "true"},
        {"source": "n2", "target": "n4", "label": "false"},
    ],
}
```

## Node Types

| Type | Behavior |
|------|----------|
| `agent` | Invokes an agent, passes input, returns output |
| `condition` | Evaluates `field operator value`, routes to `true`/`false` edge |
| `parallel` | Fan-out: runs all children concurrently |
| `merge` | Fan-in: collects outputs from parallel branches |
| `transform` | Remaps fields via `config.mapping` dict |
| `human_review` | Pauses pipeline, creates review queue entry |

## Execution Engine

```python
class PipelineRuntime:
    async def execute(self, pipeline_config, input_data, db=None, pipeline_run_id=None):
        # 1. Build adjacency list from edges
        # 2. Find root nodes (no incoming edges)
        # 3. Topological walk: execute each node, follow edges
        # 4. Condition nodes route based on output["branch"]
        # 5. Parallel nodes fan-out to all children
        # 6. Human review nodes pause with status "pending_review"
        # 7. Collect leaf node outputs as final result
        return {"status": "completed", "output": {...}, "steps": [...], "duration_ms": 42.5}
```

## Condition Operators

- `equals` — exact string match
- `contains` — substring match
- `not_equals` — inverse match

## Step Persistence

Each node execution is saved as a `pipeline_run_step` row with input, output, status, timing, and error fields. Steps are persisted incrementally so partial progress survives failures.

## Key Patterns

- Config-driven: no code changes needed to rewire pipelines
- Edges carry labels for conditional routing (`"true"`, `"false"`)
- Pipeline templates provide starter configs per industry (banking, HR, insurance, etc.)
- Duplicate endpoint lets users clone and modify existing pipelines
- Status tracking: `draft` -> `active` -> per-run `running`/`completed`/`failed`/`paused`
