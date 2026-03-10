---
name: contract-testing
description: "API contract testing to verify frontend-backend schema agreement using OpenAPI spec validation and Pact-style contracts"
category: testing
difficulty: advanced
tags: [contract, api-contract, pact, openapi, schema-validation]
stack: [python-3.12, typescript, openapi, pydantic]
---

# Contract Testing

You are an API contract testing specialist. Ensure frontend and backend agree on API shapes:

## Strategy

```
Frontend TypeScript interfaces  ←→  Backend Pydantic schemas  ←→  OpenAPI spec
```

## 1. Schema Snapshot Tests (Backend)

```python
# tests/test_contracts.py
import pytest
from pydantic import TypeAdapter
from appname.schemas import ItemResponse, ItemCreate, DashboardSummary

pytestmark = pytest.mark.asyncio

EXPECTED_ITEM_FIELDS = {"id", "name", "description", "status", "created_at"}
EXPECTED_DASHBOARD_FIELDS = {"total_models", "active_count", "draft_count"}

def test_item_response_shape():
    adapter = TypeAdapter(ItemResponse)
    schema = adapter.json_schema()
    assert set(schema["properties"].keys()) >= EXPECTED_ITEM_FIELDS

def test_item_create_required():
    adapter = TypeAdapter(ItemCreate)
    schema = adapter.json_schema()
    assert "name" in schema.get("required", [])

def test_dashboard_summary_shape():
    adapter = TypeAdapter(DashboardSummary)
    schema = adapter.json_schema()
    assert set(schema["properties"].keys()) >= EXPECTED_DASHBOARD_FIELDS
```

## 2. OpenAPI Spec Validation

```python
# tests/test_openapi.py
import pytest
from httpx import AsyncClient

pytestmark = pytest.mark.asyncio

async def test_openapi_spec_available(client: AsyncClient):
    r = await client.get("/openapi.json")
    assert r.status_code == 200
    spec = r.json()
    assert "paths" in spec
    assert "/api/items" in spec["paths"]
    assert "get" in spec["paths"]["/api/items"]

async def test_openapi_response_schemas(client: AsyncClient):
    r = await client.get("/openapi.json")
    spec = r.json()
    schemas = spec.get("components", {}).get("schemas", {})
    assert "ItemResponse" in schemas
    assert "ItemCreate" in schemas
```

## 3. Frontend Type Guard Tests

```typescript
// src/test/contracts.test.ts
import { describe, it, expect } from 'vitest'

interface ItemResponse {
  id: number
  name: string
  status: string
  created_at: string
}

function isItemResponse(obj: unknown): obj is ItemResponse {
  if (!obj || typeof obj !== 'object') return false
  const o = obj as Record<string, unknown>
  return typeof o.id === 'number'
    && typeof o.name === 'string'
    && typeof o.status === 'string'
    && typeof o.created_at === 'string'
}

describe('API contracts', () => {
  it('validates ItemResponse shape', () => {
    const mock = { id: 1, name: 'test', status: 'Draft', created_at: '2026-01-01T00:00:00' }
    expect(isItemResponse(mock)).toBe(true)
  })

  it('rejects invalid ItemResponse', () => {
    expect(isItemResponse({ id: 'bad' })).toBe(false)
  })
})
```

## Rules
- Contract tests run in CI — break the build on schema drift
- Test field names AND types, not just presence
- Keep frontend interfaces and backend schemas in sync manually (or auto-generate)
- Test required vs optional fields explicitly
- Run contract tests before integration tests in pipeline
- When schema changes: update contract tests FIRST, then implementation
