---
name: backend-e2e
description: "End-to-end API tests with in-memory SQLite, async httpx client, seeded fixtures, and full CRUD coverage"
category: testing
difficulty: intermediate
tags: [pytest, e2e, api-testing, httpx, async]
stack: [python-3.12, pytest, pytest-asyncio, httpx, aiosqlite]
---

# Backend E2E Testing

You are a test engineering expert. When writing API tests:

## Test Setup (conftest.py)

```python
import pytest
import pytest_asyncio
from httpx import AsyncClient, ASGITransport
from sqlalchemy.ext.asyncio import create_async_engine, async_sessionmaker, AsyncSession

TEST_DB_URL = "sqlite+aiosqlite:///:memory:"
_test_engine = create_async_engine(TEST_DB_URL, echo=False)
_TestSession = async_sessionmaker(_test_engine, class_=AsyncSession, expire_on_commit=False)

async def override_get_db():
    async with _TestSession() as session:
        yield session

# Patch BEFORE importing app
import appname.database as _db
_db.engine = _test_engine
_db.SessionLocal = _TestSession
_db.get_db = override_get_db

from appname.database import get_db

@pytest_asyncio.fixture(scope="session")
async def client():
    from appname.models import Base  # import after patch
    from appname.main import app

    async with _test_engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)

    async with _TestSession() as db:
        from appname.seed import seed_if_empty
        await seed_if_empty(db)

    app.dependency_overrides[get_db] = override_get_db
    transport = ASGITransport(app=app)
    async with AsyncClient(transport=transport, base_url="http://test") as ac:
        yield ac
```

## Test Patterns

```python
import pytest
pytestmark = pytest.mark.asyncio

# Health check
async def test_health(client):
    r = await client.get("/health")
    assert r.status_code == 200
    assert r.json()["status"] == "ok"

# List endpoint
async def test_list_items(client):
    r = await client.get("/api/items")
    assert r.status_code == 200
    data = r.json()
    assert isinstance(data, list)
    assert len(data) > 0

# Create
async def test_create_item(client):
    r = await client.post("/api/items", json={"name": "test", "description": "Test item"})
    assert r.status_code == 201
    assert r.json()["name"] == "test"

# Get by ID
async def test_get_item(client):
    r = await client.get("/api/items/1")
    assert r.status_code == 200
    assert "name" in r.json()

# Not found
async def test_get_missing(client):
    r = await client.get("/api/items/99999")
    assert r.status_code == 404

# Update
async def test_update_item(client):
    r = await client.put("/api/items/1", json={"name": "updated"})
    assert r.status_code == 200
    assert r.json()["name"] == "updated"

# Delete
async def test_delete_item(client):
    create = await client.post("/api/items", json={"name": "to-delete"})
    item_id = create.json()["id"]
    r = await client.delete(f"/api/items/{item_id}")
    assert r.status_code == 204

# Search/filter
async def test_search(client):
    r = await client.get("/api/items?search=test")
    assert r.status_code == 200

# Pagination
async def test_pagination(client):
    r = await client.get("/api/items?skip=0&limit=5")
    assert r.status_code == 200
    assert len(r.json()) <= 5

# Validation
async def test_invalid_create(client):
    r = await client.post("/api/items", json={})
    assert r.status_code == 422
```

## Run Command
```bash
cd backend && PYTHONPATH=src .venv/bin/pytest tests/ -v
```

## Rules
- In-memory SQLite — never touch real database
- Patch database module BEFORE importing app
- `scope="session"` fixture — seed once, share across tests
- Test every endpoint: happy path + error cases
- Test 404, 422 (validation), search, pagination
- Assert status codes AND response body shape
- Name tests: `test_{action}_{subject}` (e.g., `test_create_model`)
- Aim for 100+ tests across all routers
- No mocking unless testing external API calls
