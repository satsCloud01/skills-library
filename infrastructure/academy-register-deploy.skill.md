---
name: academy-register-deploy
description: "Register a new Learning Academy app in the agent-launcher Lambda, push Docker image to ECR, add on-demand card to Solution Registry, and verify deploy"
category: infrastructure
difficulty: intermediate
tags: [academy, learning, lambda, ecr, on-demand, agent-launcher, solution-registry]
stack: [aws-lambda, ecr, docker, fastapi, react, vite]
---

# Academy Register & Deploy

You are a deployment engineer registering a new Learning Academy application in the on-demand agent launcher infrastructure. This skill covers the full pipeline: building and pushing the Docker image, registering the agent in the Lambda launcher, adding the on-demand card to the Solution Registry HTML, and verifying the deploy flow works end-to-end.

## Prerequisites

- App has a working `Dockerfile` (multi-stage: Node frontend + Python backend)
- App has a `/api/health` endpoint
- AWS CLI configured with ECR, Lambda, and EC2 permissions
- Solution Registry HTML at `/Users/Sats/Documents/TechnicalPlayGround/CodexFolder/deployed-apps.html`

## Step 1: Create ECR Repository (if not exists)

```bash
ECR_URI="506334647682.dkr.ecr.us-east-1.amazonaws.com"
AGENT_ID="my-academy-app"        # kebab-case, e.g. sats-learning-academy
ECR_REPO="satszone/${AGENT_ID}"

aws ecr create-repository --repository-name "$ECR_REPO" --region us-east-1 2>/dev/null || echo "Repo exists"
```

## Step 2: Build & Push Docker Image

```bash
APP_DIR="/Users/Sats/Documents/TechnicalPlayGround/CodexFolder/${AGENT_ID}"
cd "$APP_DIR"

# Login to ECR
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin "$ECR_URI"

# Build (ensure .dockerignore excludes frontend/node_modules, backend/.venv, __pycache__)
docker build --platform linux/amd64 -t "${ECR_REPO}:latest" .

# Tag and push
docker tag "${ECR_REPO}:latest" "${ECR_URI}/${ECR_REPO}:latest"
docker push "${ECR_URI}/${ECR_REPO}:latest"
```

### Dockerfile Pattern (Academy Apps)

Academy apps follow this standard multi-stage pattern:

```dockerfile
# Stage 1: Build frontend
FROM node:20-slim AS frontend
WORKDIR /app/frontend
COPY frontend/package.json frontend/package-lock.json ./
RUN npm ci
COPY frontend/ ./
RUN npm run build

# Stage 2: Python runtime
FROM python:3.12-slim
WORKDIR /app
COPY backend/requirements.txt ./requirements.txt
RUN pip install --no-cache-dir -r requirements.txt
COPY backend/src/ ./src/
COPY --from=frontend /app/frontend/dist ./static

# Mount static files from FastAPI
RUN echo '\nfrom fastapi.staticfiles import StaticFiles\nimport os\nif os.path.isdir("/app/static"):\n    app.mount("/", StaticFiles(directory="/app/static", html=True), name="static")' >> /app/src/MODULE_NAME/main.py

ENV PYTHONPATH=/app/src
EXPOSE APP_PORT
CMD ["uvicorn", "MODULE_NAME.main:app", "--host", "0.0.0.0", "--port", "APP_PORT"]
```

**Important:** Replace `MODULE_NAME` with the Python package name (e.g. `academy`) and `APP_PORT` with the backend port.

### Required .dockerignore

```
frontend/node_modules
backend/.venv
backend/__pycache__
**/__pycache__
*.pyc
.git
```

Node modules symlinks created on macOS break on Linux Docker builds — the `.dockerignore` is mandatory.

## Step 3: Register Agent in Lambda Launcher

The agent-launcher Lambda (`agent-launcher`) contains an `AGENTS` dict that maps `agent_id` to ECR image, port, and display name. To register a new academy app:

### 3a. Download current Lambda code

```bash
mkdir -p /tmp/lambda-download
aws lambda get-function --function-name agent-launcher --region us-east-1 \
  --query 'Code.Location' --output text | \
  xargs curl -s -o /tmp/lambda-download/launcher.zip

cd /tmp/lambda-download && unzip -o launcher.zip -d launcher/
```

### 3b. Add agent entry to AGENTS dict

Edit `/tmp/lambda-download/launcher/launcher.py` and add a new entry to the `AGENTS` dict:

```python
AGENTS = {
    # ... existing agents ...
    'my-academy-app': {
        'image': f'{ecr_uri}/satszone/my-academy-app:latest',
        'port': APP_PORT,       # The port in the Dockerfile CMD (e.g. 8025)
        'name': 'My Academy App Display Name',
    },
}
```

### 3c. Deploy updated Lambda

```bash
cd /tmp/lambda-download/launcher
zip -j /tmp/lambda-download/launcher-updated.zip launcher.py
aws lambda update-function-code \
  --function-name agent-launcher \
  --zip-file fileb:///tmp/lambda-download/launcher-updated.zip \
  --region us-east-1
```

### 3d. Verify registration

```bash
sleep 3
curl -s https://t695k3ysm1.execute-api.us-east-1.amazonaws.com/prod/deploy \
  -X POST -H 'Content-Type: application/json' \
  -d '{"agent_id":"my-academy-app"}'
```

Expected: JSON with `instance_id`, `public_ip`, `url`. If you get `{"error": "Unknown agent: ..."}`, the Lambda update hasn't propagated — wait and retry.

**Important:** Terminate the test instance immediately after verification:

```bash
curl -s https://t695k3ysm1.execute-api.us-east-1.amazonaws.com/prod/deploy \
  -X POST -H 'Content-Type: application/json' \
  -d '{"agent_id":"_kill","instance_id":"INSTANCE_ID_FROM_ABOVE"}'
```

## Step 4: Add On-Demand Card to Solution Registry

Add the agent card to `deployed-apps.html` in the Learning Academy section (`section-learning`).

### 4a. Add card prefix mapping

In the `CARD_PREFIX` JavaScript object, add the new agent:

```javascript
var CARD_PREFIX = {
  // ... existing entries ...
  'my-academy-app': 'PREFIX',   // 2-3 letter prefix for DOM IDs
};
```

### 4b. Add HTML card

Follow the on-demand card pattern (see `agent-on-demand-card` skill). Key elements:
- Card with `onclick="deployAgent('my-academy-app')"`
- Status div: `id="PREFIXStatus"`
- Footer div: `id="PREFIXFooter"`
- Deploy button: `id="PREFIXDeployBtn"`
- Kill wrap: `id="PREFIXKillWrap"`
- Badge: `<span class="badge-ondemand">&#9889; On-Demand</span>`
- GitHub link: `https://github.com/satsCloud01/REPO_NAME`

### 4c. Deploy updated registry

```bash
aws s3 cp deployed-apps.html s3://my-solution-registry.satszone.link/index.html --content-type "text/html"
aws cloudfront create-invalidation --distribution-id E2R00426B8QGNB --paths "/*"
```

## Step 5: Update Solution Registry Tour

The "Take the Tour" must include every new academy. Add a tour step in the `/* ── Learning Academy ── */` section of `TOUR_STEPS`:

```javascript
{
  icon: 'EMOJI', title: 'Academy Name',
  subtitle: 'Learning Academy \u00b7 Topic tagline',
  gradient: 'linear-gradient(135deg,#COLOR1,#COLOR2)', url: null,
  desc: 'Description of the academy content...',
  features: [
    'Lesson count and categories',
    'Interactive features (simulators, playground, etc.)',
    'Quiz details and scoring',
    'On-demand deploy: click Deploy Now to spin up a live instance for 30 minutes'
  ]
},
```

Then update the **Welcome step** (first entry) and **Finale step** (last entry) counts to reflect the new total. Deploy to S3 and push to GitHub:

```bash
cp deployed-apps.html /Users/Sats/Documents/TechnicalPlayGround/CodexFolder/solution-registry/index.html
cd /Users/Sats/Documents/TechnicalPlayGround/CodexFolder/solution-registry
git add index.html && git commit -m "feat: add ACADEMY_NAME to tour" && git push origin main
aws s3 cp /Users/Sats/Documents/TechnicalPlayGround/CodexFolder/deployed-apps.html s3://my-solution-registry.satszone.link/index.html --content-type "text/html"
aws cloudfront create-invalidation --distribution-id E2R00426B8QGNB --paths "/*"
```

## Step 6: End-to-End Verification

1. Open `https://my-solution-registry.satszone.link`
2. Click **Take the Tour** — verify the new academy step appears in the correct position
3. Navigate to **Learning Academy** section
4. Click **Deploy Now** on the new card
5. Verify the 4-step deploy modal completes:
   - Step 1: EC2 provisioned
   - Step 2: Docker image pulled
   - Step 3: Container started
   - Step 4: Health check passes
6. Click **Open App** — verify the app loads
7. Click **Kill** to terminate the instance

## Academy App Conventions

| Convention | Value |
|-----------|-------|
| ECR namespace | `satszone/` |
| Docker port | Matches `start.sh` backend port |
| EC2 port mapping | Always `80 → app_port` |
| Health endpoint | `/api/health` |
| TTL | 30 minutes (auto-destroy) |
| Instance type | `t3.small` |
| Frontend build | Vite → `dist/` → FastAPI `StaticFiles` |
| Static mount | Appended to `main.py` via Dockerfile `RUN echo` |

## Troubleshooting

| Issue | Cause | Fix |
|-------|-------|-----|
| `Unknown agent` error | Agent not in Lambda AGENTS dict | Re-download, add entry, redeploy Lambda |
| Health check never passes | App crashes on startup | SSH to instance, `docker logs agent` |
| Mixed content warning | HTTPS registry fetching HTTP app | Status API polls instead of direct fetch |
| Docker build fails on Linux | macOS node_modules symlinks | Add `frontend/node_modules` to `.dockerignore` |
| Static files 404 | `RUN echo` didn't append correctly | Check `main.py` for duplicate mount or syntax error |
| Container starts but no response | Wrong port in AGENTS dict | Verify port matches Dockerfile `EXPOSE`/`CMD` |

## Complete Checklist

- [ ] App has Dockerfile with multi-stage build
- [ ] `.dockerignore` excludes node_modules and .venv
- [ ] `/api/health` endpoint exists
- [ ] ECR repo created (`satszone/AGENT_ID`)
- [ ] Docker image built for `linux/amd64` and pushed to ECR
- [ ] Lambda `AGENTS` dict updated with correct image, port, name
- [ ] Lambda redeployed and verified
- [ ] Test instance launched and terminated
- [ ] On-demand card added to Solution Registry HTML
- [ ] `CARD_PREFIX` mapping added in JavaScript
- [ ] Tour step added to `TOUR_STEPS` in Learning Academy section
- [ ] Welcome and Finale tour steps updated with new counts
- [ ] Registry deployed to S3 + CloudFront invalidated
- [ ] Registry pushed to GitHub (satsCloud01/solution-registry)
- [ ] End-to-end deploy test from registry UI
- [ ] Tour verified (new step appears in correct position)
