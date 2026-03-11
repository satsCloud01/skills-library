---
name: Decision Intelligence Interview
description: Guided business-first interview that translates non-technical answers into technical specifications
category: backend
tags: [decision-intelligence, guided-interview, no-code, business-logic]
stack: [python, fastapi]
---

# Decision Intelligence Interview

A pattern for translating business user answers into technical specifications through guided questions.

## Flow

1. **Define questions** — business-oriented, with predefined options
2. **Collect answers** — show impact of each choice in plain language
3. **Derive spec** — rule-based translation of answers to technical config

## Question Structure

```python
GUIDED_QUERIES = [
    {
        "question": "What business problem are you trying to solve?",
        "hint": "Describe the challenge in your own words",
        "category": "problem",
        "options": ["Customer support", "Data analysis", "Automation", ...]
    },
    ...
]
```

## Impact Feedback

After each answer, show the user what their choice means:

```python
def _impact_for_answer(category, answer):
    return {
        "what_this_means": "Human-friendly explanation",
        "agent_will": "What the system will do based on this choice"
    }
```

## Blueprint Derivation

```python
def _derive_blueprint(answers: dict) -> dict:
    # Rule-based: keyword matching on answers → tools, framework, capabilities
    # e.g. "knowledge base" in data_sources → add knowledge_base_search tool
    # e.g. "step-by-step" in behavior → plan-execute framework
    return {"tools": [...], "framework": "...", "capabilities": [...]}
```

## Key Patterns

- No technical jargon in questions — pure business language
- Each answer immediately shows its impact
- Deterministic rule-based derivation (no AI call needed)
- Answers stored per session for audit trail
