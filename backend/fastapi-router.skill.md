---
name: fastapi-router
description: "Creates FastAPI routers with async endpoints, dependency injection, pagination, filtering, and proper error handling"
category: backend
difficulty: intermediate
tags: [fastapi, router, api, crud, async]
stack: [python-3.12, fastapi, sqlalchemy-2.0, pydantic-v2]
---

# FastAPI Router Builder

You are an expert FastAPI developer. When creating routers:

## File Placement

```
backend/src/appname/routers/feature.py
```

## Router Template

```python
from fastapi import APIRouter, Depends, HTTPException, Query
from sqlalchemy import select, func, or_
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy.orm import selectinload

from appname.database import get_db
from appname.models import Item
from appname.schemas import ItemCreate, ItemUpdate, ItemResponse

router = APIRouter(prefix="/api/items", tags=["items"])


@router.get("")
async def list_items(
    search: str = Query("", max_length=200),
    status: str = Query(""),
    skip: int = Query(0, ge=0),
    limit: int = Query(50, ge=1, le=200),
    db: AsyncSession = Depends(get_db),
):
    q = select(Item)
    if search:
        q = q.where(or_(
            Item.name.ilike(f"%{search}%"),
            Item.description.ilike(f"%{search}%"),
        ))
    if status:
        q = q.where(Item.status == status)
    q = q.order_by(Item.created_at.desc()).offset(skip).limit(limit)
    result = await db.execute(q)
    return [ItemResponse.model_validate(r) for r in result.scalars().all()]


@router.get("/{item_id}")
async def get_item(item_id: int, db: AsyncSession = Depends(get_db)):
    q = select(Item).where(Item.id == item_id).options(selectinload(Item.versions))
    result = await db.execute(q)
    item = result.scalar_one_or_none()
    if not item:
        raise HTTPException(404, "Item not found")
    return item


@router.post("", status_code=201)
async def create_item(body: ItemCreate, db: AsyncSession = Depends(get_db)):
    item = Item(**body.model_dump())
    db.add(item)
    await db.commit()
    await db.refresh(item)
    return item


@router.put("/{item_id}")
async def update_item(item_id: int, body: ItemUpdate, db: AsyncSession = Depends(get_db)):
    q = select(Item).where(Item.id == item_id)
    result = await db.execute(q)
    item = result.scalar_one_or_none()
    if not item:
        raise HTTPException(404, "Item not found")
    for k, v in body.model_dump(exclude_unset=True).items():
        setattr(item, k, v)
    await db.commit()
    await db.refresh(item)
    return item


@router.delete("/{item_id}", status_code=204)
async def delete_item(item_id: int, db: AsyncSession = Depends(get_db)):
    q = select(Item).where(Item.id == item_id)
    result = await db.execute(q)
    item = result.scalar_one_or_none()
    if not item:
        raise HTTPException(404, "Item not found")
    await db.delete(item)
    await db.commit()
```

## Registration in main.py

```python
from appname.routers import feature
app.include_router(feature.router)
```

## Rules
- All endpoints async with `AsyncSession`
- Use `Depends(get_db)` — never create sessions manually
- Use `selectinload()` for relationships to avoid N+1
- Pagination: `skip`/`limit` with Query validators
- Search: `ilike` for case-insensitive partial match
- Return 201 for POST, 204 for DELETE
- Use `model_dump(exclude_unset=True)` for partial updates
- Raise `HTTPException(404)` not bare `Exception`
- Tag routers for OpenAPI grouping
- Prefix: `/api/{plural-noun}` (e.g., `/api/models`, `/api/policies`)
