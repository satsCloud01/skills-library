---
name: hitl-queue
description: "Human-in-the-Loop approval queue pattern — agents escalate high-risk actions for human review before execution"
category: backend
difficulty: intermediate
tags: [HITL, agent, approval, risk-scoring, FastAPI]
stack: [fastapi, sqlalchemy, react]
---

# Human-in-the-Loop (HITL) Queue

Agents pause execution on high-risk actions and create HITL tasks. Humans approve, reject, or escalate. Only after approval does the agent proceed.

## SQLAlchemy Model

```python
class HITLTask(Base):
    __tablename__ = "hitl_tasks"

    id = Column(Integer, primary_key=True)
    title = Column(String, nullable=False)
    agent = Column(String, nullable=False)          # which agent escalated
    action_type = Column(String, nullable=False)    # what action is pending
    priority = Column(String, default="medium")     # critical/high/medium/low
    status = Column(String, default="pending")      # pending/approved/rejected/escalated
    risk_score = Column(Float, default=0.0)         # 0.0–1.0
    context = Column(JSON, default={})              # action context for reviewer
    reviewer = Column(String, nullable=True)
    decision_note = Column(String, nullable=True)
    created_at = Column(DateTime, default=datetime.utcnow)
    resolved_at = Column(DateTime, nullable=True)
```

## FastAPI Router

```python
@router.post("/{task_id}/approve")
async def approve_task(task_id: int, note: str = "", db: AsyncSession = Depends(get_db)):
    task = await db.get(HITLTask, task_id)
    if not task or task.status != "pending":
        raise HTTPException(404)
    task.status = "approved"
    task.decision_note = note
    task.resolved_at = datetime.utcnow()
    await db.commit()
    # Trigger agent to resume execution
    return {"status": "approved", "task_id": task_id}

@router.post("/{task_id}/reject")
async def reject_task(task_id: int, reason: str, db: AsyncSession = Depends(get_db)):
    task = await db.get(HITLTask, task_id)
    task.status = "rejected"
    task.decision_note = reason
    task.resolved_at = datetime.utcnow()
    await db.commit()
    return {"status": "rejected", "task_id": task_id}
```

## Agent Integration Pattern

```python
async def execute_action(action: dict, risk_score: float) -> dict:
    if risk_score >= 0.8:
        # Create HITL task instead of executing
        task = await create_hitl_task(
            title=f"Approval required: {action['type']}",
            agent=action['agent'],
            risk_score=risk_score,
            context=action
        )
        return {"status": "hitl_required", "task_id": task.id}

    # Low-risk: execute directly
    return await run_action(action)
```

## Risk Score Factors

| Factor | Weight |
|---|---|
| Affects PII data | +0.4 |
| Mass delete (>10k rows) | +0.3 |
| Schema-level change | +0.3 |
| Cross-domain access | +0.2 |
| Production environment | +0.2 |
| Irreversible operation | +0.3 |
