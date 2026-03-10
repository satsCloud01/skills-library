---
name: deploy-ec2
description: "End-to-end deployment workflow: git push, SSH, Docker rebuild, health check, and rollback procedure"
category: deployment
difficulty: intermediate
tags: [deploy, ec2, docker, ssh, rollback]
stack: [git, docker, ssh, bash]
---

# EC2 Deployment

You are a deployment engineer. Follow this deployment workflow:

## Pre-Deploy Checklist

1. All tests pass locally: `PYTHONPATH=src .venv/bin/pytest tests/ -v`
2. Frontend builds: `cd frontend && npm run build`
3. Changes committed and pushed to GitHub
4. No secrets in committed files

## Deployment Steps

```bash
# 1. Push to GitHub
git add -A && git commit -m "feat: description" && git push origin main

# 2. SSH to EC2
ssh -i ~/.ssh/consolidated-apps-key.pem ubuntu@<ip>

# 3. Pull latest code
cd ~/apps/<app-name>
git pull origin main

# 4. Rebuild container
sudo docker compose build <service-name>

# 5. Deploy with zero-ish downtime
sudo docker compose up -d <service-name>

# 6. Verify health
sleep 5
curl -s http://localhost:<port>/health | python3 -m json.tool

# 7. Check logs for errors
sudo docker compose logs --tail=20 <service-name>
```

## Rollback Procedure

```bash
# If deployment fails:
# 1. Check what went wrong
sudo docker compose logs --tail=50 <service-name>

# 2. Revert to previous git commit
cd ~/apps/<app-name>
git log --oneline -5  # find last good commit
git checkout <good-commit-sha>

# 3. Rebuild and restart
sudo docker compose build <service-name>
sudo docker compose up -d <service-name>

# 4. Verify recovery
curl -s http://localhost:<port>/health
```

## Multi-App Deployment (batch)

```bash
# Deploy all apps that changed
APPS=("ai-guardian" "data-guardian" "archsmith")
for app in "${APPS[@]}"; do
  echo "Deploying $app..."
  cd ~/apps/$app && git pull
  sudo docker compose build $app
  sudo docker compose up -d $app
  sleep 3
  echo "✓ $app deployed"
done
```

## Rules
- Always pull before build — never build stale code
- Health check after every deploy — don't assume success
- Check logs for startup errors — Python import errors are common
- Keep previous Docker image until verified (don't prune immediately)
- If disk is low: `sudo docker system prune -af` BEFORE building
- Never force-push to main — creates confusion on EC2 pull
- Document deployment in audit log or commit message
