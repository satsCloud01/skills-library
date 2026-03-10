---
name: docker-build
description: "Multi-stage Docker builds for FastAPI+React apps with optimized layers, security hardening, and production config"
category: infrastructure
difficulty: intermediate
tags: [docker, multi-stage, containerization, production, optimization]
stack: [docker, python-3.12, node-20, nginx]
---

# Docker Build

You are a container engineering expert. When building Docker images:

## Multi-Stage Dockerfile (FastAPI + React)

```dockerfile
# ── Stage 1: Frontend build ──
FROM node:20-slim AS frontend-build
WORKDIR /app/frontend
COPY frontend/package*.json ./
RUN npm ci --no-audit --no-fund
COPY frontend/ ./
RUN npm run build

# ── Stage 2: Production ──
FROM python:3.12-slim
WORKDIR /app

# System deps
RUN apt-get update && apt-get install -y --no-install-recommends \
    gcc libffi-dev && \
    rm -rf /var/lib/apt/lists/*

# Python deps (cached layer)
COPY backend/requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt

# App code
COPY backend/src/ ./src/

# Frontend static files
COPY --from=frontend-build /app/frontend/dist ./src/appname/static/

# Environment
ENV PYTHONPATH=/app/src
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

EXPOSE 8000
CMD ["uvicorn", "appname.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

## CPU-Only PyTorch (for t3.medium / no GPU)

```dockerfile
# Install CPU-only torch FIRST to avoid pulling CUDA
RUN pip install --no-cache-dir torch --index-url https://download.pytorch.org/whl/cpu
RUN pip install --no-cache-dir -r requirements.txt
```

## Streamlit Apps

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt
COPY src/ ./src/
ENV PYTHONPATH=/app/src
EXPOSE 8501
CMD ["streamlit", "run", "src/appname/app.py", \
     "--server.port=8501", "--server.address=0.0.0.0", \
     "--server.headless=true", "--browser.gatherUsageStats=false"]
```

## .dockerignore

```
**/.venv
**/__pycache__
**/node_modules
**/.git
**/.env
**/dist
*.pyc
.pytest_cache
```

## Optimization Rules
- Order layers: system deps → pip install → COPY code (maximize cache hits)
- `--no-cache-dir` on pip — saves 30-50% image size
- `npm ci` not `npm install` — deterministic, faster
- `python:3.12-slim` not `python:3.12` — saves ~600MB
- Multi-stage: frontend build artifacts only, no node_modules in final image
- Remove apt cache: `rm -rf /var/lib/apt/lists/*`
- Set `PYTHONDONTWRITEBYTECODE=1` — no .pyc files
- Set `PYTHONUNBUFFERED=1` — real-time logging

## Security Rules
- Never COPY `.env` files into image
- Don't run as root in production: `RUN useradd -m app && USER app`
- No secrets in build args — use runtime env vars
- Pin base image versions for reproducibility
- Scan images: `docker scout cves <image>`
