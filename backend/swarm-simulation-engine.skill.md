---
name: swarm-simulation-engine
description: "Real round-by-round multi-agent swarm intelligence simulation engine — each round is a separate LLM call with context memory from previous rounds"
category: backend
difficulty: advanced
tags: [python, simulation, multi-agent, swarm-intelligence, round-by-round, LLM, prediction, fastapi]
stack: [python, fastapi, sqlalchemy, anthropic-sdk]
---

# Swarm Simulation Engine

Real multi-round simulation where autonomous agent personas interact, debate, and converge to form predictions. Each round is a separate LLM call with full memory of previous rounds.

## Architecture

```
SimulationRun (pending → running → paused → completed)
  └── Round 1: LLM call with agent descriptions
       └── Events: interaction, opinion_shift, conflict, consensus
       └── Agent Messages: direct quotes with sentiment
  └── Round 2: LLM call with Round 1 context
       └── Events build on previous dynamics
  └── Round N (final): LLM generates consensus summary
```

## Key Endpoints

```python
POST /simulation/create   # Create run (pending, not started)
POST /simulation/round     # Execute ONE round (real-time progression)
POST /simulation/run       # Execute ALL rounds in sequence
PUT  /simulation/runs/{id}/pause   # Pause mid-simulation
PUT  /simulation/runs/{id}/resume  # Resume paused simulation
DELETE /simulation/runs/{id}       # Delete run + cascade events/messages
```

## Round-by-Round Pattern

```python
# Each round builds on previous context
prev_events = db.query(SimulationEvent).filter(run_id=run.id).order_by(round_num)
prev_context = "\n".join(f"Round {e.round_num}: {e.description}" for e in prev_events[-10:])

prompt = f"""Simulate Round {next_round} of {total_rounds}.
AGENTS: {agents_desc}
PREVIOUS EVENTS: {prev_context}
{"FINAL round — work toward conclusion." if is_final else ""}
Return JSON: {{"events": [...], "final_consensus": "..."}}"""
```

## Event Types

| Type | Description |
|------|-------------|
| `interaction` | Agents exchange information |
| `opinion_shift` | Agent changes position based on evidence |
| `conflict` | Agents disagree on interpretation |
| `consensus` | Multiple agents align on a prediction |
| `event` | External event injected into simulation |

## Reference Implementation

See: `prediction-intelligence/backend/src/predictor/routers/simulation.py`
