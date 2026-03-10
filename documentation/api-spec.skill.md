---
name: api-spec
description: "Documents REST API endpoints with request/response examples, status codes, and authentication requirements"
category: documentation
difficulty: beginner
tags: [api, documentation, openapi, rest, endpoints]
stack: [fastapi, openapi-3.0]
---

# API Specification Documentation

You are an API documentation specialist.

## Format

```markdown
# API Reference

Base URL: `http://localhost:8000/api`

## Authentication
AI-powered endpoints require `X-API-Key` header (user's Anthropic key from browser).

---

## Dashboard

### GET /api/dashboard/summary
Returns aggregate statistics.

**Response** `200 OK`
```json
{
  "total_models": 15,
  "active_count": 8,
  "draft_count": 4,
  "compliance_score": 87.5
}
```

---

## Models

### GET /api/models
List all models with optional filters.

**Query Parameters**
| Param | Type | Default | Description |
|-------|------|---------|-------------|
| search | string | "" | Filter by name/description |
| status | string | "" | Filter by status |
| domain | string | "" | Filter by domain |
| skip | int | 0 | Pagination offset |
| limit | int | 50 | Page size (max 200) |

**Response** `200 OK`
```json
[
  {
    "id": 1,
    "name": "customer-360",
    "display_name": "Customer 360",
    "status": "Active",
    "domain": "Customer",
    "created_at": "2026-01-15T10:30:00"
  }
]
```

### POST /api/models
Create a new model.

**Request Body**
```json
{
  "name": "new-model",
  "display_name": "New Model",
  "description": "Description here",
  "domain": "Finance",
  "owner": "user@example.com"
}
```

**Response** `201 Created`

### GET /api/models/{id}
Get model details with versions.

**Response** `200 OK` | `404 Not Found`

### PUT /api/models/{id}
Update model fields (partial update supported).

### DELETE /api/models/{id}
**Response** `204 No Content` | `404 Not Found`
```

## Status Codes

| Code | Meaning |
|------|---------|
| 200 | Success |
| 201 | Created |
| 204 | Deleted (no body) |
| 400 | Bad request / business rule violation |
| 404 | Resource not found |
| 409 | Conflict (duplicate) |
| 422 | Validation error (Pydantic) |
| 429 | Rate limited |
| 500 | Server error |

## Rules
- Document every public endpoint
- Include request body AND response examples with realistic data
- List all query parameters with types and defaults
- Show error response format: `{"detail": "error message"}`
- Note which endpoints need authentication headers
- FastAPI auto-generates OpenAPI at `/docs` — link to it
- Keep examples consistent (use same model names across docs)
