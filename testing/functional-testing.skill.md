---
name: functional-testing
description: "Functional tests validating business logic, state machines, workflows, and domain rules end-to-end"
category: testing
difficulty: intermediate
tags: [functional, business-logic, workflow, state-machine, validation]
stack: [python-3.12, pytest, pytest-asyncio]
---

# Functional Testing

You are a quality assurance expert. When writing functional tests:

## What to Test

Functional tests validate **business rules**, not just HTTP status codes:

1. **State machine transitions**: Draft → Review → Approved → Active → Deprecated
2. **Business validation**: Duplicate names, required fields, domain constraints
3. **Workflow logic**: Approval chains, cascading updates, notifications
4. **Data integrity**: Relationships, cascading deletes, referential integrity
5. **Authorization rules**: Who can do what in which state

## State Machine Tests

```python
pytestmark = pytest.mark.asyncio

async def test_valid_status_transition(client):
    # Create in Draft
    r = await client.post("/api/models", json={"name": "sm-test", "status": "Draft"})
    model_id = r.json()["id"]

    # Draft → Review (valid)
    r = await client.put(f"/api/models/{model_id}", json={"status": "Review"})
    assert r.status_code == 200
    assert r.json()["status"] == "Review"

    # Review → Approved (valid)
    r = await client.put(f"/api/models/{model_id}", json={"status": "Approved"})
    assert r.status_code == 200

async def test_invalid_status_transition(client):
    r = await client.post("/api/models", json={"name": "sm-invalid", "status": "Draft"})
    model_id = r.json()["id"]

    # Draft → Active (invalid — must go through Review)
    r = await client.put(f"/api/models/{model_id}", json={"status": "Active"})
    assert r.status_code == 400
```

## Business Rule Tests

```python
async def test_duplicate_name_rejected(client):
    await client.post("/api/models", json={"name": "unique-name"})
    r = await client.post("/api/models", json={"name": "unique-name"})
    assert r.status_code == 409  # or 400

async def test_cascading_delete(client):
    # Create parent with children
    r = await client.post("/api/models", json={"name": "parent"})
    model_id = r.json()["id"]
    await client.post(f"/api/models/{model_id}/versions", json={"version": "1.0"})

    # Delete parent — children should be deleted
    await client.delete(f"/api/models/{model_id}")
    r = await client.get(f"/api/models/{model_id}/versions")
    assert r.status_code == 404 or len(r.json()) == 0

async def test_search_filters_correctly(client):
    r = await client.get("/api/models?domain=Finance&status=Active")
    data = r.json()
    for item in data:
        assert item["domain"] == "Finance"
        assert item["status"] == "Active"
```

## Workflow Tests

```python
async def test_full_lifecycle(client):
    """Test complete item lifecycle: create → update → review → approve → deprecate"""
    # Create
    r = await client.post("/api/models", json={"name": "lifecycle-test"})
    assert r.status_code == 201
    item_id = r.json()["id"]

    # Update description
    r = await client.put(f"/api/models/{item_id}", json={"description": "Updated"})
    assert r.json()["description"] == "Updated"

    # Submit for review
    r = await client.put(f"/api/models/{item_id}", json={"status": "Review"})
    assert r.json()["status"] == "Review"

    # Approve
    r = await client.put(f"/api/models/{item_id}", json={"status": "Approved"})
    assert r.json()["status"] == "Approved"

    # Verify appears in approved list
    r = await client.get("/api/models?status=Approved")
    names = [m["name"] for m in r.json()]
    assert "lifecycle-test" in names
```

## Rules
- Test business rules, not implementation details
- Cover all valid AND invalid state transitions
- Test edge cases: empty strings, max lengths, boundary values
- Test workflows end-to-end (create → use → archive)
- Verify data consistency after operations
- Test with realistic seed data, not minimal fixtures
- Group related tests with descriptive class names
