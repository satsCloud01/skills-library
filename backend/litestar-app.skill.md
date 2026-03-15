---
name: litestar-app
description: "Scaffolds a Litestar application with controllers, lifespan hooks, CORS, health endpoint, and binary response support"
category: backend
difficulty: intermediate
tags: [litestar, application, cors, lifespan, controllers, async]
stack: [python-3.12, litestar, uvicorn]
---

# Litestar Application Setup

You are a backend architect. When scaffolding a Litestar app:

## main.py

```python
from contextlib import asynccontextmanager
from collections.abc import AsyncGenerator
from litestar import Litestar, get
from litestar.config.cors import CORSConfig

from .database import get_connection
from .routers.example import ExampleController


@asynccontextmanager
async def lifespan(app: Litestar) -> AsyncGenerator[None, None]:
    get_connection()  # init DB + seed on startup
    yield


@get("/api/health")
async def health() -> dict:
    return {"status": "ok", "engine": "litestar"}


app = Litestar(
    route_handlers=[health, ExampleController],
    cors_config=CORSConfig(
        allow_origins=["http://localhost:5173"],
        allow_methods=["*"],
        allow_headers=["*"],
    ),
    lifespan=[lifespan],
)
```

## Controller Pattern

```python
from litestar import Controller, get, post
from dataclasses import dataclass


@dataclass
class CreateRequest:
    name: str
    value: str


class ExampleController(Controller):
    path = "/api/example"

    @get("/")
    async def list_all(self) -> list[dict]:
        return service.get_all()

    @post("/")
    async def create(self, data: CreateRequest) -> dict:
        return service.create(data.name, data.value)

    @get("/{item_id:int}")
    async def get_one(self, item_id: int) -> dict:
        return service.get_by_id(item_id)
```

## Binary Response (e.g., Arrow IPC)

```python
from litestar import Response

@post("/binary")
async def binary_response(self, data: QueryRequest) -> Response:
    raw_bytes = service.get_binary_data(data.query)
    return Response(
        content=raw_bytes,
        media_type="application/octet-stream",
        headers={"Content-Length": str(len(raw_bytes))},
    )
```

## File Upload (Multipart)

```python
from litestar.datastructures import UploadFile
from litestar.enums import RequestEncodingType
from litestar.params import Body

@post("/upload")
async def upload(
    self,
    data: UploadFile = Body(media_type=RequestEncodingType.MULTI_PART),
) -> dict:
    content = await data.read()
    return {"filename": data.filename, "size": len(content)}
```

## Key Differences from FastAPI

| Feature | FastAPI | Litestar |
|---------|---------|----------|
| Routing | `@app.get()` decorators | Controller classes |
| Validation | Pydantic models | Dataclasses or Pydantic |
| Lifespan | `@asynccontextmanager` on app | `lifespan=[...]` list |
| CORS | Middleware | `CORSConfig` object |
| Path params | `{id: int}` | `{id:int}` (no space) |

## Run

```bash
PYTHONPATH=src uvicorn appname.main:app --reload --port 8000
```
