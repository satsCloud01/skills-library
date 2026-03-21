---
name: Human Review Queue
description: Human-in-the-loop approval workflows with approve, reject, escalate, and reassign actions plus audit trail
category: backend
tags: [human-in-the-loop, approval, review-queue, audit-trail, governance]
stack: [python, fastapi]
---

# Human Review Queue

Pause agent/pipeline execution at designated steps, queue outputs for human review, and resume or reject based on reviewer action.

## Review Lifecycle

```
Agent runs -> hits human_review step -> run paused (status: pending_review)
    -> Review item created in review_queue (status: pending)
        -> Reviewer approves  -> run resumes (status: completed)
        -> Reviewer rejects   -> run fails (status: failed, error logged)
        -> Reviewer escalates -> reassigned to senior reviewer
        -> Reviewer reassigns -> routed to different reviewer
```

## API Endpoints

| Method | Path | Action |
|--------|------|--------|
| `GET` | `/api/reviews/` | List reviews (filter by status, agent_id) |
| `GET` | `/api/reviews/stats` | Aggregate counts (pending, approved, rejected, escalated) |
| `GET` | `/api/reviews/{id}` | Detail view with run steps and audit history |
| `POST` | `/api/reviews/{id}/approve` | Approve and resume the paused run |
| `POST` | `/api/reviews/{id}/reject` | Reject and fail the paused run |
| `POST` | `/api/reviews/{id}/escalate` | Escalate to another reviewer |
| `POST` | `/api/reviews/{id}/reassign` | Reassign to a named reviewer |

## Action Schemas

```python
class ReviewAction(BaseModel):
    notes: Optional[str] = None

class EscalateAction(BaseModel):
    notes: Optional[str] = None
    escalate_to: Optional[str] = None

class ReassignAction(BaseModel):
    reviewer: str
    notes: Optional[str] = None
```

## Audit Trail

Every review action inserts an `audit_logs` row:

```python
await db.execute(
    "INSERT INTO audit_logs (id, agent_id, run_id, action, details, created_at) VALUES (...)",
    # action: review_approved | review_rejected | review_escalated | review_reassigned
    # details: JSON with review_id, notes, escalate_to
)
```

## Key Patterns

- Review detail endpoint joins run data, run steps, and audit history in one response
- Status guards prevent double-action (only `pending`/`escalated` can be approved/rejected)
- Approve updates both `review_queue` and `runs`/`run_steps` tables atomically
- Stats endpoint powers dashboard widgets (pending count, today's approvals/rejections)
- Audit trail provides full traceability for compliance
