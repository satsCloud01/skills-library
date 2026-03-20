---
name: circuit-breaker
description: "Circuit breaker pattern for data pipelines and agent actions — CLOSED/OPEN/HALF_OPEN state machine protecting downstream systems"
category: backend
difficulty: intermediate
tags: [circuit-breaker, resilience, pipeline, state-machine, FastAPI]
stack: [fastapi, sqlalchemy]
---

# Circuit Breaker Pattern

Implements the CLOSED → OPEN → HALF_OPEN → CLOSED state machine to protect downstream systems from cascading failures.

## States

| State | Meaning | Requests |
|---|---|---|
| CLOSED | Normal operation | All pass through |
| OPEN | Failure threshold exceeded | All blocked, fast-fail |
| HALF_OPEN | Testing recovery | One probe request allowed |

## SQLAlchemy Model

```python
class CircuitBreaker(Base):
    __tablename__ = "circuit_breakers"

    id = Column(Integer, primary_key=True)
    name = Column(String, nullable=False)
    pipeline_id = Column(Integer, ForeignKey("pipelines.id"), nullable=True)
    state = Column(String, default="CLOSED")           # CLOSED/OPEN/HALF_OPEN
    failure_threshold = Column(Integer, default=5)     # failures before OPEN
    success_threshold = Column(Integer, default=2)     # successes to close from HALF_OPEN
    timeout_seconds = Column(Integer, default=60)      # OPEN → HALF_OPEN after this
    failure_count = Column(Integer, default=0)
    success_count = Column(Integer, default=0)
    last_failure_at = Column(DateTime, nullable=True)
    opened_at = Column(DateTime, nullable=True)
```

## State Transition Logic

```python
async def record_failure(breaker: CircuitBreaker, db: AsyncSession):
    breaker.failure_count += 1
    breaker.last_failure_at = datetime.utcnow()

    if breaker.state == "CLOSED" and breaker.failure_count >= breaker.failure_threshold:
        breaker.state = "OPEN"
        breaker.opened_at = datetime.utcnow()
    elif breaker.state == "HALF_OPEN":
        breaker.state = "OPEN"  # probe failed, reopen

    await db.commit()

async def record_success(breaker: CircuitBreaker, db: AsyncSession):
    if breaker.state == "HALF_OPEN":
        breaker.success_count += 1
        if breaker.success_count >= breaker.success_threshold:
            breaker.state = "CLOSED"
            breaker.failure_count = 0
            breaker.success_count = 0
    await db.commit()

async def check_state(breaker: CircuitBreaker, db: AsyncSession) -> str:
    if breaker.state == "OPEN":
        elapsed = (datetime.utcnow() - breaker.opened_at).seconds
        if elapsed >= breaker.timeout_seconds:
            breaker.state = "HALF_OPEN"
            await db.commit()
    return breaker.state

def is_allowed(breaker: CircuitBreaker) -> bool:
    return breaker.state in ("CLOSED", "HALF_OPEN")
```

## FastAPI Endpoints

```python
@router.post("/{id}/trip")
async def trip_breaker(id: int, db=Depends(get_db)):
    """Manually open a circuit breaker."""
    breaker = await db.get(CircuitBreaker, id)
    breaker.state = "OPEN"
    breaker.opened_at = datetime.utcnow()
    await db.commit()
    return {"state": "OPEN"}

@router.post("/{id}/reset")
async def reset_breaker(id: int, db=Depends(get_db)):
    """Manually reset to CLOSED."""
    breaker = await db.get(CircuitBreaker, id)
    breaker.state = "CLOSED"
    breaker.failure_count = 0
    breaker.success_count = 0
    await db.commit()
    return {"state": "CLOSED"}
```
