---
name: nonfunctional-testing
description: "Non-functional testing: security (OWASP), reliability, error handling, concurrency, and resource limits"
category: testing
difficulty: advanced
tags: [security, reliability, owasp, stress, resilience]
stack: [python-3.12, pytest, pytest-asyncio]
---

# Non-Functional Testing

You are a resilience and security testing expert.

## 1. Security Tests (OWASP Top 10)

```python
pytestmark = pytest.mark.asyncio

# SQL Injection
async def test_sql_injection_in_search(client):
    r = await client.get("/api/models?search='; DROP TABLE data_models; --")
    assert r.status_code == 200  # should not crash

# XSS prevention
async def test_xss_in_input(client):
    r = await client.post("/api/models", json={
        "name": "test-xss",
        "description": "<script>alert('xss')</script>"
    })
    assert r.status_code in (201, 422)
    if r.status_code == 201:
        assert "<script>" not in r.json().get("description", "")  # sanitized or escaped

# Path traversal
async def test_path_traversal(client):
    r = await client.get("/api/models/../../etc/passwd")
    assert r.status_code in (400, 404, 422)

# Mass assignment
async def test_mass_assignment(client):
    r = await client.post("/api/models", json={
        "name": "mass-assign", "is_admin": True, "role": "superuser"
    })
    if r.status_code == 201:
        data = r.json()
        assert data.get("is_admin") is not True

# Header security
async def test_no_sensitive_headers(client):
    r = await client.get("/health")
    assert "X-Powered-By" not in r.headers
    assert "Server" not in r.headers or "uvicorn" not in r.headers.get("Server", "").lower()
```

## 2. Error Handling & Resilience

```python
async def test_invalid_json_body(client):
    r = await client.post("/api/models", content=b"not json",
                          headers={"Content-Type": "application/json"})
    assert r.status_code == 422

async def test_missing_content_type(client):
    r = await client.post("/api/models", content=b'{"name":"test"}')
    assert r.status_code in (200, 201, 422)  # should handle gracefully

async def test_oversized_payload(client):
    huge = {"name": "x" * 100000, "description": "y" * 1000000}
    r = await client.post("/api/models", json=huge)
    assert r.status_code in (400, 413, 422)

async def test_empty_body(client):
    r = await client.post("/api/models", json={})
    assert r.status_code == 422  # validation error

async def test_concurrent_creates(client):
    import asyncio
    tasks = [client.post("/api/models", json={"name": f"concurrent-{i}"}) for i in range(20)]
    results = await asyncio.gather(*tasks)
    success = [r for r in results if r.status_code == 201]
    assert len(success) == 20  # all should succeed
```

## 3. Rate Limiting & Resource Tests

```python
async def test_rapid_requests(client):
    """Verify server handles burst traffic without errors"""
    for i in range(100):
        r = await client.get("/api/dashboard/summary")
        assert r.status_code == 200

async def test_pagination_limits(client):
    r = await client.get("/api/models?limit=99999")
    assert r.status_code in (200, 422)
    if r.status_code == 200:
        assert len(r.json()) <= 200  # server-side cap
```

## 4. CORS Tests

```python
async def test_cors_headers(client):
    r = await client.options("/api/models",
        headers={"Origin": "http://localhost:5173", "Access-Control-Request-Method": "GET"})
    assert "access-control-allow-origin" in r.headers

async def test_cors_rejects_unknown_origin(client):
    r = await client.options("/api/models",
        headers={"Origin": "http://evil.com", "Access-Control-Request-Method": "GET"})
    origin = r.headers.get("access-control-allow-origin", "")
    assert origin != "http://evil.com" or origin == "*"
```

## Rules
- Run security tests in CI — they should never regress
- Test with malicious inputs: SQL, XSS, path traversal, oversized payloads
- Test concurrent access — no race conditions
- Test error responses: valid JSON, no stack traces in production
- Test CORS: only allowed origins accepted
- Test graceful degradation when dependencies are down
