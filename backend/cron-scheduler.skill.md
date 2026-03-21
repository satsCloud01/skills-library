---
name: Cron Scheduler
description: Cron-based and interval-based task scheduling for agents and pipelines using pure Python asyncio
category: backend
tags: [cron, scheduling, asyncio, background-tasks, automation]
stack: [python, fastapi]
---

# Cron Scheduler

Schedule agents and pipelines to run on cron expressions or fixed intervals, using a pure-Python asyncio loop (no APScheduler dependency).

## Cron Parser

```python
SHORTCUTS = {
    "@hourly": "0 * * * *",
    "@daily": "0 0 * * *",
    "@weekly": "0 0 * * 1",
    "@monthly": "0 0 1 * *",
    "@yearly": "0 0 1 1 *",
}

def parse_cron(expr) -> tuple[set, set, set, set, set]:
    """Returns (minutes, hours, days, months, weekdays) as sets."""
    # Supports: *, */N, N-M, N, comma-separated

def next_cron_time(expr, after=None) -> datetime:
    """Minute-by-minute scan up to 1 year to find next match."""
```

## Scheduler Service (Singleton)

```python
class SchedulerService:
    _instance = None
    _task = None

    async def start(self):
        self._task = asyncio.create_task(self._loop())

    async def _loop(self):
        while True:
            await self._check_schedules()  # query DB for due schedules
            await asyncio.sleep(30)        # poll every 30 seconds

    async def _execute_schedule(self, db, schedule, now):
        if schedule["target_type"] == "agent":
            # Create run record, launch agent in background task
            asyncio.create_task(self._run_agent_bg(...))
        elif schedule["target_type"] == "pipeline":
            # Create pipeline_run record
        # Update next_run_at, last_run_at, run_count
```

## Schedule Types

| Type | Config | Next Run Calculation |
|------|--------|---------------------|
| `cron` | `cron_expression` (5-field) | Parse and scan forward |
| `interval` | `interval_seconds` | `now + timedelta(seconds=N)` |

## Human-Readable Display

```python
def cron_to_readable(expr) -> str:
    # "0 9 * * *" -> "Daily at 9:00 AM"
    # "*/15 * * * *" -> "Every 15 minutes"
    # "0 0 * * 1" -> "Weekly on Mon"
```

## Key Patterns

- Pure asyncio — no external scheduler library needed
- Singleton pattern with `get_instance()` classmethod
- Background task execution: agent runs don't block the scheduler loop
- DB-persisted schedules survive restarts (re-scanned on next poll)
- Schedule metadata: `last_run_at`, `next_run_at`, `run_count` updated atomically
- Supports both agent and pipeline targets
