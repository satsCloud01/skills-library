---
name: fastapi-app
description: "Scaffolds FastAPI application with lifespan, CORS, static serving, health endpoint, and router registration"
category: backend
difficulty: intermediate
tags: [fastapi, application, cors, lifespan, middleware]
stack: [python-3.12, fastapi, uvicorn]
---

# FastAPI Application Setup

You are a backend architect. When scaffolding a FastAPI app:

## main.py

```python
from contextlib import asynccontextmanager
from pathlib import Path

from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from fastapi.staticfiles import StaticFiles

from appname.database import init_db, SessionLocal
from appname.seed import seed_if_empty


@asynccontextmanager
async def lifespan(app: FastAPI):
    await init_db()
    async with SessionLocal() as db:
        await seed_if_empty(db)
    yield


app = FastAPI(
    title="App Name",
    version="1.0.0",
    lifespan=lifespan,
)

app.add_middleware(
    CORSMiddleware,
    allow_origins=[
        "http://localhost:5173",
        "http://localhost:5174",
    ],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Register routers
from appname.routers import dashboard, models, settings
app.include_router(dashboard.router)
app.include_router(models.router)
app.include_router(settings.router)


@app.get("/health")
async def health():
    return {"status": "ok", "version": "1.0.0"}


# Serve frontend static files in production
static_dir = Path(__file__).parent / "static"
if static_dir.exists():
    app.mount("/", StaticFiles(directory=str(static_dir), html=True), name="static")
```

## Seed Pattern (seed.py)

```python
from sqlalchemy import select, func
from sqlalchemy.ext.asyncio import AsyncSession
from appname.models import Item

async def seed_if_empty(db: AsyncSession):
    count = await db.scalar(select(func.count()).select_from(Item))
    if count and count > 0:
        return

    items = [
        Item(name="example-1", display_name="Example One", ...),
        # ... more seed data
    ]
    db.add_all(items)
    await db.commit()
```

## Rules
- Use `lifespan` context manager — not deprecated `@app.on_event`
- CORS: list explicit dev origins, never `["*"]` in production
- Health endpoint at `/health` — returns version for monitoring
- Static files mount LAST (catch-all route)
- Seed: check if data exists before inserting (idempotent)
- Run with: `PYTHONPATH=src uvicorn appname.main:app --reload --port 8000`
- Requirements: fastapi, uvicorn[standard], sqlalchemy, aiosqlite, greenlet, pydantic
