---
name: runbook
description: "Operational runbook covering local setup, production deployment, monitoring, troubleshooting, and incident response"
category: documentation
difficulty: intermediate
tags: [runbook, operations, troubleshooting, monitoring, deployment]
stack: [docker, aws, nginx, systemd]
---

# Operational Runbook

You are an SRE documentation expert. When writing runbooks:

## Structure

```markdown
# Runbook: {App Name}

## Quick Start (Local)

```bash
git clone https://github.com/org/repo.git
cd repo
./start.sh
# Frontend: http://localhost:5173
# Backend:  http://localhost:8000
# API Docs: http://localhost:8000/docs
```

## Production Environment

| Property | Value |
|----------|-------|
| URL | https://app.satszone.link |
| Host | EC2 i-xxx (t3.medium) |
| Container port | 8010 |
| Docker service | app-name |
| Log group | docker compose logs app-name |

## Health Checks

```bash
# Local
curl http://localhost:8000/health

# Production
curl https://app.satszone.link/health
```

## Common Operations

### Deploy Update
```bash
ssh -i ~/.ssh/key.pem ubuntu@<ip>
cd ~/apps/app-name && git pull
sudo docker compose build app-name
sudo docker compose up -d app-name
```

### View Logs
```bash
sudo docker compose logs -f app-name --tail=100
```

### Restart Service
```bash
sudo docker compose restart app-name
```

### Check Disk Space
```bash
df -h
sudo docker system df
```

## Troubleshooting

### Container won't start
1. Check logs: `sudo docker compose logs app-name --tail=50`
2. Common: `ModuleNotFoundError` → check `PYTHONPATH=/app/src` in Dockerfile
3. Common: port conflict → check `sudo docker compose ps`

### No disk space
1. `sudo docker system prune -af`
2. `sudo docker builder prune -af`
3. Check db files: `du -sh ~/apps/*/backend/*.db`

### SSL certificate expired
```bash
sudo certbot renew
sudo nginx -s reload
```

### App returns 502/504
1. Container crashed → `sudo docker compose up -d app-name`
2. Port mismatch → verify nginx upstream port matches docker-compose port
3. Memory exceeded → `sudo docker stats --no-stream`

## Incident Response
1. **Detect**: Health check fails or user report
2. **Assess**: Check logs, container status, disk, memory
3. **Mitigate**: Restart container or rollback to previous commit
4. **Resolve**: Fix root cause, deploy, verify
5. **Document**: Note cause and fix for future reference
```

## Rules
- Every app needs a runbook
- Include exact commands — no "run the appropriate command"
- Cover the 5 most common failure scenarios
- Include health check URLs
- Document all environment variables
- Keep updated when infrastructure changes
