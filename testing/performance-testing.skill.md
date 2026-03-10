---
name: performance-testing
description: "Load testing, response time benchmarks, and performance profiling for FastAPI backends using Locust and pytest-benchmark"
category: testing
difficulty: advanced
tags: [performance, load-testing, locust, benchmark, latency]
stack: [python-3.12, locust, pytest-benchmark]
---

# Performance Testing

You are a performance engineering expert. When testing application performance:

## 1. Response Time Tests (pytest)

```python
# tests/test_performance.py
import pytest
import time

pytestmark = pytest.mark.asyncio

async def test_dashboard_response_time(client):
    start = time.perf_counter()
    r = await client.get("/api/dashboard/summary")
    elapsed = time.perf_counter() - start
    assert r.status_code == 200
    assert elapsed < 0.5, f"Dashboard took {elapsed:.3f}s (limit: 0.5s)"

async def test_list_endpoint_response_time(client):
    start = time.perf_counter()
    r = await client.get("/api/models?limit=50")
    elapsed = time.perf_counter() - start
    assert r.status_code == 200
    assert elapsed < 1.0, f"List took {elapsed:.3f}s (limit: 1.0s)"

async def test_search_response_time(client):
    start = time.perf_counter()
    r = await client.get("/api/models?search=customer")
    elapsed = time.perf_counter() - start
    assert r.status_code == 200
    assert elapsed < 0.5
```

## 2. Load Testing (Locust)

```python
# locustfile.py
from locust import HttpUser, task, between

class AppUser(HttpUser):
    wait_time = between(1, 3)
    host = "http://localhost:8000"

    @task(3)
    def dashboard(self):
        self.client.get("/api/dashboard/summary")

    @task(2)
    def list_models(self):
        self.client.get("/api/models?limit=20")

    @task(1)
    def get_model(self):
        self.client.get("/api/models/1")

    @task(1)
    def search(self):
        self.client.get("/api/models?search=finance")
```

```bash
# Run: 50 users, spawn 10/sec
locust -f locustfile.py --headless -u 50 -r 10 --run-time 60s
```

## 3. Database Query Profiling

```python
# Enable SQLAlchemy query logging in test
import logging
logging.getLogger("sqlalchemy.engine").setLevel(logging.INFO)

async def test_no_n_plus_one(client, caplog):
    with caplog.at_level(logging.INFO, logger="sqlalchemy.engine"):
        await client.get("/api/models/1")
    query_count = sum(1 for r in caplog.records if "SELECT" in r.message)
    assert query_count <= 3, f"N+1 detected: {query_count} queries for single item"
```

## Performance Budgets

| Endpoint | P95 Target | Max Queries |
|----------|-----------|-------------|
| GET /health | < 50ms | 0 |
| GET /dashboard/summary | < 500ms | 5 |
| GET /items (list) | < 500ms | 2 |
| GET /items/:id | < 200ms | 3 |
| POST /items | < 300ms | 2 |
| GET /items?search= | < 500ms | 2 |

## Rules
- Set response time budgets per endpoint — test against them
- Profile N+1 queries with SQLAlchemy logging
- Load test with realistic user behavior (weighted tasks)
- Test with seeded data (not empty DB)
- Monitor memory: `tracemalloc` for leak detection
- Run performance tests in CI but with lower thresholds (CI is slower)
