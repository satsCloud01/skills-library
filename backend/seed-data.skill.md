---
name: seed-data
description: "Creates realistic seed data for demos using Faker with idempotent seeding and domain-appropriate content"
category: backend
difficulty: beginner
tags: [seed, faker, demo-data, database, fixtures]
stack: [python-3.12, faker, sqlalchemy-2.0]
---

# Seed Data Generator

You are a data generation expert. When creating seed data:

## Pattern

```python
from faker import Faker
from sqlalchemy import select, func
from sqlalchemy.ext.asyncio import AsyncSession

fake = Faker()
Faker.seed(42)  # reproducible data

async def seed_if_empty(db: AsyncSession):
    from appname.models import Model, Item

    count = await db.scalar(select(func.count()).select_from(Model))
    if count and count > 0:
        return  # idempotent — skip if data exists

    models = []
    for i in range(15):
        models.append(Model(
            name=f"model-{i+1}",
            display_name=fake.catch_phrase(),
            description=fake.paragraph(nb_sentences=3),
            domain=fake.random_element(["Finance", "Risk", "Operations", "Customer"]),
            owner=fake.email(),
            status=fake.random_element(["Draft", "Review", "Approved", "Active"]),
            sensitivity=fake.random_element(["Restricted", "Confidential", "Internal"]),
        ))
    db.add_all(models)
    await db.commit()
```

## Rules
- Always check count before seeding — must be idempotent
- Use `Faker.seed(42)` for reproducible data across runs
- 10-25 records is enough for demo — not too sparse, not overwhelming
- Use domain-appropriate values (bank terms for finance apps, etc.)
- Include variety in status/type fields to show all UI states
- Seed relationships: if Model has Versions, seed both
- Call from `lifespan` in main.py — auto-seeds on first run
- Never seed in production — guard with env var if needed
