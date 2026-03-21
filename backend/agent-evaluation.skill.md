---
name: Agent Evaluation
description: Test and score AI agent quality using test cases with expected outputs and keyword-overlap scoring
category: backend
tags: [evaluation, testing, scoring, agent-quality, benchmarking]
stack: [python, fastapi]
---

# Agent Evaluation

Run test suites against agents, score outputs against expected results, and track quality over time.

## Evaluation Flow

1. **Create evaluation** — name it, link to an agent, attach test cases
2. **Run evaluation** — execute each test case against the agent, score results
3. **Review results** — per-case pass/fail with similarity scores, aggregate score

## Test Case Schema

```python
class EvalCreate(BaseModel):
    name: str
    agent_id: str
    test_cases: list[dict]  # [{"input": "...", "expected_output": "..."}, ...]
```

## Scoring Function

```python
def _score_output(expected: str, actual: str) -> float:
    """Keyword overlap scoring."""
    if expected == actual:
        return 1.0
    exp_words = set(expected.lower().split())
    act_words = set(actual.lower().split())
    overlap = exp_words & act_words
    return round(len(overlap) / len(exp_words), 2)

# Pass threshold: score >= 0.7
```

## Auto-Generated Test Cases

Generate plausible test cases from agent metadata without an AI API call:

```python
@router.post("/generate")
async def generate_test_cases(body: TestCaseGenerateRequest):
    # Template pools keyed by agent name keywords:
    # "email"/"triage" -> email classification cases
    # "kyc"/"onboarding" -> KYC verification cases
    # "claim" -> insurance claim cases
    # default -> generic request processing cases
    return {"test_cases": random.sample(pool, count)}
```

## Evaluation Lifecycle

| Status | Meaning |
|--------|---------|
| `pending` | Created, not yet run |
| `running` | Test cases being executed |
| `completed` | All cases scored, aggregate score computed |

## Key Patterns

- Deterministic scoring (keyword overlap) — no AI dependency for evaluation itself
- Agent history endpoint shows score trends over time per agent
- Score stored as percentage (0-100) for dashboard display
- Template-based test case generation covers domain-specific scenarios
- Results include per-case input, expected, actual, score, and pass/fail
