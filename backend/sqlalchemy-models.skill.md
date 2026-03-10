---
name: sqlalchemy-models
description: "Defines SQLAlchemy 2.0+ ORM models with typed columns, relationships, lifecycle hooks, and async database setup"
category: backend
difficulty: intermediate
tags: [sqlalchemy, orm, database, models, async]
stack: [python-3.12, sqlalchemy-2.0, aiosqlite, greenlet]
---

# SQLAlchemy 2.0+ Models

You are a database modeling expert. When defining models:

## Database Setup (database.py)

```python
from sqlalchemy.ext.asyncio import AsyncSession, create_async_engine, async_sessionmaker
from sqlalchemy.orm import DeclarativeBase

DATABASE_URL = "sqlite+aiosqlite:///./app.db"

engine = create_async_engine(DATABASE_URL, echo=False)
SessionLocal = async_sessionmaker(engine, class_=AsyncSession, expire_on_commit=False)

class Base(DeclarativeBase):
    pass

async def get_db():
    async with SessionLocal() as session:
        yield session

async def init_db():
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)
```

## Model Definitions (models.py)

```python
from datetime import datetime
from typing import Optional
from sqlalchemy import ForeignKey, Text, Boolean, DateTime, JSON, Index
from sqlalchemy.orm import Mapped, mapped_column, relationship
from appname.database import Base


class DataModel(Base):
    __tablename__ = "data_models"

    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str] = mapped_column(unique=True, index=True)
    display_name: Mapped[str]
    description: Mapped[str] = mapped_column(Text, default="")
    model_type: Mapped[str] = mapped_column(default="Relational")
    domain: Mapped[str] = mapped_column(default="", index=True)
    owner: Mapped[str] = mapped_column(default="")
    status: Mapped[str] = mapped_column(default="Draft")
    sensitivity: Mapped[str] = mapped_column(default="Internal")
    tags: Mapped[Optional[str]] = mapped_column(Text, nullable=True)
    created_at: Mapped[datetime] = mapped_column(default=datetime.utcnow)
    updated_at: Mapped[datetime] = mapped_column(default=datetime.utcnow, onupdate=datetime.utcnow)

    versions: Mapped[list["ModelVersion"]] = relationship(
        back_populates="model", cascade="all, delete-orphan",
        order_by="ModelVersion.version_number.desc()"
    )

    __table_args__ = (
        Index("ix_model_status_domain", "status", "domain"),
    )


class ModelVersion(Base):
    __tablename__ = "model_versions"

    id: Mapped[int] = mapped_column(primary_key=True)
    model_id: Mapped[int] = mapped_column(ForeignKey("data_models.id", ondelete="CASCADE"))
    version_number: Mapped[str]
    change_summary: Mapped[str] = mapped_column(Text, default="")
    created_at: Mapped[datetime] = mapped_column(default=datetime.utcnow)

    model: Mapped["DataModel"] = relationship(back_populates="versions")
```

## Pydantic Schemas (schemas.py)

```python
from pydantic import BaseModel, Field
from typing import Optional

class ItemCreate(BaseModel):
    name: str = Field(..., min_length=1, max_length=200)
    description: str = ""

class ItemUpdate(BaseModel):
    name: Optional[str] = None
    description: Optional[str] = None

class ItemResponse(ItemCreate):
    id: int
    status: str
    created_at: str

    class Config:
        from_attributes = True
```

## Rules
- Use `Mapped[type]` with `mapped_column()` — never legacy `Column()`
- Always `expire_on_commit=False` for async sessions
- `greenlet` must be in requirements.txt (macOS async SQLAlchemy)
- Use `cascade="all, delete-orphan"` for parent-child relationships
- Index frequently filtered columns: status, domain, created_at
- Use `Text` for descriptions, `JSON` for flexible data
- Timestamps: `default=datetime.utcnow` (not `datetime.utcnow()` — no parens)
- Naming: table = `snake_case` plural, model class = `PascalCase` singular
- ForeignKey ondelete: `CASCADE` for owned children, `SET NULL` for references
- Status values as string constants — not separate enum table
