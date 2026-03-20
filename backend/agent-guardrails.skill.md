---
name: agent-guardrails
description: "Real-time guardrail evaluation for AI agent actions — policy-as-code with configurable strictness, auto-remediation, and confidence thresholds"
category: backend
difficulty: advanced
tags: [guardrails, agent, policy-as-code, safety, compliance, FastAPI]
stack: [fastapi, sqlalchemy, anthropic]
---

# Agent Guardrails

Evaluates agent actions against configurable safety and compliance rules before execution. Returns ALLOW/BLOCK/WARN decisions with triggered rule details.

## Guardrail Categories

| Category | Examples |
|---|---|
| data_privacy | PII access, GDPR retention, data masking |
| access_control | Role permissions, cross-domain access |
| data_quality | Schema validation, null thresholds |
| compliance | Regulatory rules (GDPR, SOX, HIPAA) |
| operational | Rate limits, resource quotas |
| security | Injection prevention, auth checks |
| cost | Token budget, API rate limits |
| ethical | Bias detection, fairness checks |

## SQLAlchemy Model

```python
class GuardrailRule(Base):
    __tablename__ = "guardrail_rules"

    id = Column(Integer, primary_key=True)
    name = Column(String, nullable=False)
    category = Column(String, nullable=False)
    severity = Column(String, default="medium")     # critical/high/medium/low
    action = Column(String, default="BLOCK")        # BLOCK/WARN/ALLOW
    condition = Column(String)                       # rule expression
    confidence_threshold = Column(Float, default=0.8)
    auto_remediate = Column(Boolean, default=False)
    enabled = Column(Boolean, default=True)
```

## Evaluation Engine

```python
async def evaluate(action: dict, rules: list[GuardrailRule], api_key: str | None = None) -> EvalResult:
    triggered = []

    for rule in rules:
        if not rule.enabled:
            continue
        if await matches(rule, action):
            triggered.append(rule)

    if not triggered:
        return EvalResult(decision="ALLOW", triggered_rules=[])

    # Highest severity wins
    severity_order = {"critical": 4, "high": 3, "medium": 2, "low": 1}
    worst = max(triggered, key=lambda r: severity_order.get(r.severity, 0))

    decision = worst.action  # BLOCK/WARN
    risk_score = severity_order[worst.severity] / 4.0

    # Auto-remediate if configured and not critical
    if worst.auto_remediate and worst.severity != "critical":
        remediated = await auto_remediate(action, worst)
        if remediated:
            return EvalResult(decision="ALLOW", remediated=True, triggered_rules=triggered)

    # HITL required for high risk
    if risk_score >= 0.8 and decision == "BLOCK":
        hitl_task = await create_hitl_task(action, triggered, risk_score)
        return EvalResult(decision="BLOCK", hitl_task_id=hitl_task.id, triggered_rules=triggered)

    return EvalResult(decision=decision, risk_score=risk_score, triggered_rules=triggered)
```

## FastAPI Endpoint

```python
@router.post("/evaluate")
async def evaluate_action(
    request: EvaluateRequest,
    x_ai_key: str | None = Header(default=None),
    db: AsyncSession = Depends(get_db)
):
    rules = await db.execute(select(GuardrailRule).where(GuardrailRule.enabled == True))
    result = await evaluate(request.action, rules.scalars().all(), api_key=x_ai_key)
    return result
```

## Mode Configuration

```python
GUARDRAIL_MODES = {
    "strict":      {"confidence_threshold": 0.6, "auto_remediate": False},
    "standard":    {"confidence_threshold": 0.8, "auto_remediate": True},
    "permissive":  {"confidence_threshold": 0.95, "auto_remediate": True},
}
```
