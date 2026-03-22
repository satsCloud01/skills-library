---
name: knowledge-graph-builder
description: "AI-powered knowledge graph builder with entity extraction from documents, full CRUD on nodes/edges, and React Flow visualization"
category: backend
difficulty: intermediate
tags: [python, knowledge-graph, entity-extraction, LLM, react-flow, CRUD, fastapi, NLP]
stack: [python, fastapi, sqlalchemy, react, react-flow]
---

# Knowledge Graph Builder

Extracts entities and relationships from seed documents using LLM, builds an interactive knowledge graph with full manual CRUD.

## Pipeline

```
Upload Seeds (text/file) → AI Entity Extraction → Knowledge Graph
                                                    ↕ Manual CRUD
                                              React Flow Visualization
```

## Entity Types

| Type | Color | Example |
|------|-------|---------|
| entity | #06b6d4 | Generic entity |
| concept | #8b5cf6 | "Inflation Rate" |
| event | #f59e0b | "Trade Agreement" |
| location | #10b981 | "European Union" |
| organization | #3b82f6 | "Federal Reserve" |
| person | #ec4899 | "Dr. Sarah Chen" |

## Backend Endpoints

```python
# Seed Document CRUD
POST /graph/seeds              # Add text seed
POST /graph/seeds/upload       # Upload file (multipart/form-data)
GET  /graph/seeds/{id}         # Get full content
PUT  /graph/seeds/{id}         # Edit content/metadata
DELETE /graph/seeds/{id}       # Remove seed

# Node CRUD
POST /graph/nodes              # Create node manually
GET  /graph/nodes/{id}         # Get node detail
PUT  /graph/nodes/{id}         # Update label/type/description/position
DELETE /graph/nodes/{id}       # Delete node (cascades connected edges)

# Edge CRUD
POST /graph/edges              # Create edge between nodes
PUT  /graph/edges/{id}         # Update relation/weight
DELETE /graph/edges/{id}       # Delete edge

# AI Build
POST /graph/build              # Extract entities from all seeds via LLM
```

## AI Extraction Prompt Pattern

```python
prompt = """Analyze documents and extract a knowledge graph.
Return JSON:
{
  "entities": [{"label": "...", "type": "person|org|concept|...", "description": "..."}],
  "relations": [{"source": "label", "target": "label", "relation": "verb"}]
}"""
```

## Delete Cascade

When deleting a node, all connected edges (source or target) are automatically removed.

## Reference Implementation

See: `prediction-intelligence/backend/src/predictor/routers/graph.py`
