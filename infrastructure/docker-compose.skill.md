---
name: docker-compose
description: "Docker Compose orchestration for multi-app deployments with Nginx reverse proxy, health checks, and restart policies"
category: infrastructure
difficulty: intermediate
tags: [docker-compose, orchestration, nginx, multi-app, reverse-proxy]
stack: [docker-compose, nginx, ubuntu]
---

# Docker Compose Orchestration

You are a DevOps engineer. When orchestrating containers:

## docker-compose.yml (Multi-App)

```yaml
version: "3.8"

services:
  app-name:
    build:
      context: ./app-name
      dockerfile: Dockerfile
    container_name: app-name
    restart: unless-stopped
    ports:
      - "8010:8000"
    environment:
      - PYTHONPATH=/app/src
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 15s
    mem_limit: 512m
    cpus: 0.5

  another-app:
    build: ./another-app
    container_name: another-app
    restart: unless-stopped
    ports:
      - "8011:8000"
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
```

## Nginx Reverse Proxy Config

```nginx
server {
    listen 443 ssl http2;
    server_name app.satszone.link;

    ssl_certificate     /etc/letsencrypt/live/app.satszone.link/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/app.satszone.link/privkey.pem;

    location / {
        proxy_pass http://127.0.0.1:8010;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

server {
    listen 80;
    server_name app.satszone.link;
    return 301 https://$host$request_uri;
}
```

## Common Commands

```bash
# Build and start all
sudo docker compose up -d --build

# Rebuild single service
sudo docker compose build app-name && sudo docker compose up -d app-name

# View logs
sudo docker compose logs -f app-name --tail=50

# Check health
sudo docker compose ps

# Restart service
sudo docker compose restart app-name

# Prune old images (reclaim disk)
sudo docker system prune -af
```

## Port Mapping Convention

| Service | Internal | External |
|---------|----------|----------|
| App 1 | 8000 | 8010 |
| App 2 | 8000 | 8011 |
| App 3 | 8000 | 8012 |
| Streamlit apps | 8501 | 801x |

## Rules
- `restart: unless-stopped` — survives reboots but respects manual stops
- Health checks on every service — enables `docker compose ps` monitoring
- Memory limits: 512m for most apps, 1g for AI/ML apps
- One Nginx per host — handles SSL termination for all subdomains
- Let's Encrypt: `certbot --nginx -d app.satszone.link`
- Systemd service: `docker-apps.service` to start compose on boot
- Never use `docker compose down` in production unless replacing — use `restart`
