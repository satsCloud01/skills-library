---
name: oauth-auth
description: "Authentication patterns: API key headers, JWT tokens, OAuth2 flows, and session management"
category: integrations
difficulty: advanced
tags: [auth, oauth, jwt, api-key, security]
stack: [python-3.12, fastapi, pyjwt]
---

# Authentication & Authorization

You are a security integration expert.

## Pattern 1: API Key via Header (Current Standard)

```python
# User stores key in browser localStorage
# Frontend sends via header — never persisted server-side
from fastapi import Header, HTTPException

async def require_api_key(x_api_key: str = Header(..., alias="X-API-Key")):
    if not x_api_key or len(x_api_key) < 10:
        raise HTTPException(401, "Valid API key required")
    return x_api_key

@router.post("/ai/suggest")
async def suggest(body: Request, api_key: str = Depends(require_api_key)):
    # Use api_key to call external service
    pass
```

## Pattern 2: JWT Token Auth

```python
import jwt
from datetime import datetime, timedelta

SECRET_KEY = "your-secret-key"
ALGORITHM = "HS256"

def create_token(user_id: str, expires_hours: int = 24) -> str:
    payload = {
        "sub": user_id,
        "exp": datetime.utcnow() + timedelta(hours=expires_hours),
        "iat": datetime.utcnow(),
    }
    return jwt.encode(payload, SECRET_KEY, algorithm=ALGORITHM)

def verify_token(token: str) -> dict:
    try:
        return jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
    except jwt.ExpiredSignatureError:
        raise HTTPException(401, "Token expired")
    except jwt.InvalidTokenError:
        raise HTTPException(401, "Invalid token")

# FastAPI dependency
async def get_current_user(authorization: str = Header(...)):
    if not authorization.startswith("Bearer "):
        raise HTTPException(401, "Bearer token required")
    token = authorization[7:]
    payload = verify_token(token)
    return payload["sub"]
```

## Pattern 3: Room Key (Static Token with Expiry)

```python
# Pre-generated keys stored in DynamoDB/SQLite
# Validated on each request, rate-limited, time-limited
async def validate_room_key(key: str) -> dict:
    record = await db.get_key(key)
    if not record or not record.is_active:
        return {"valid": False}
    if record.expires_at < datetime.utcnow():
        return {"valid": False, "expired": True}
    return {"valid": True, "is_admin": record.is_admin}
```

## Frontend Key Storage

```tsx
// localStorage for persistence across tabs
const setKey = (key: string) => localStorage.setItem('app_key', key)
const getKey = () => localStorage.getItem('app_key') || ''
const clearKey = () => localStorage.removeItem('app_key')

// Send with every request
headers: { 'X-API-Key': getKey() }
```

## Rules
- Never store API keys server-side — browser only
- JWT: short expiry (1-24h), refresh tokens for long sessions
- Hash passwords with bcrypt — never store plaintext
- Rate limit auth endpoints (prevent brute force)
- Return 401 (unauthenticated) vs 403 (unauthorized) correctly
- Log auth failures for security monitoring
- HTTPS only — never send credentials over HTTP
