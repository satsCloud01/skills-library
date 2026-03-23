---
name: deploy-gcp-cloudrun
description: "Build and deploy containerized apps to GCP Cloud Run with source-based builds, IAM setup, and public URL"
category: deployment
difficulty: beginner
tags: [gcp, cloud-run, docker, container, deployment, google-cloud]
stack: [gcp-cloud-run, cloud-build, docker, flask, gunicorn]
---

# GCP Cloud Run Deployment

You are a GCP Cloud Run deployment specialist. You help users containerize and deploy applications to Google Cloud Run with minimal configuration.

## Prerequisites

- `gcloud` CLI installed and authenticated (`gcloud auth login`)
- A GCP project set (`gcloud config set project PROJECT_ID`)
- Cloud Run and Cloud Build APIs enabled (auto-enabled on first deploy)

## Quick Deploy (Source-based)

The simplest approach — GCP builds the container from source automatically:

```bash
cd /path/to/app
gcloud run deploy SERVICE_NAME \
  --source . \
  --region us-central1 \
  --allow-unauthenticated \
  --quiet
```

## App Scaffolding

### Python Flask (recommended for simple apps)

**main.py**
```python
import os
from flask import Flask

app = Flask(__name__)

@app.route("/")
def hello():
    return "<h1>Hello World</h1>"

if __name__ == "__main__":
    port = int(os.environ.get("PORT", 8080))
    app.run(host="0.0.0.0", port=port)
```

**requirements.txt**
```
flask==3.1.0
gunicorn==23.0.0
```

**Dockerfile**
```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
CMD exec gunicorn --bind :$PORT --workers 1 --threads 2 main:app
```

### Node.js Express

**index.js**
```javascript
const express = require('express');
const app = express();
const port = process.env.PORT || 8080;

app.get('/', (req, res) => res.send('<h1>Hello World</h1>'));
app.listen(port, () => console.log(`Listening on port ${port}`));
```

**Dockerfile**
```dockerfile
FROM node:20-slim
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
CMD ["node", "index.js"]
```

## IAM Permissions Fix

New GCP projects often need permissions granted to the default compute service account. If you see `PERMISSION_DENIED` on first deploy:

```bash
PROJECT_ID=$(gcloud config get-value project)
PROJECT_NUM=$(gcloud projects describe $PROJECT_ID --format='value(projectNumber)')

gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member="serviceAccount:${PROJECT_NUM}-compute@developer.gserviceaccount.com" \
  --role="roles/storage.objectViewer" --quiet

gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member="serviceAccount:${PROJECT_NUM}-compute@developer.gserviceaccount.com" \
  --role="roles/cloudbuild.builds.builder" --quiet

gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member="serviceAccount:${PROJECT_NUM}-compute@developer.gserviceaccount.com" \
  --role="roles/run.admin" --quiet

gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member="serviceAccount:${PROJECT_NUM}-compute@developer.gserviceaccount.com" \
  --role="roles/iam.serviceAccountUser" --quiet
```

Then retry the deploy.

## Configuration Options

| Flag | Description | Example |
|------|-------------|---------|
| `--region` | Deployment region | `us-central1`, `europe-west1` |
| `--allow-unauthenticated` | Public access (no auth) | — |
| `--memory` | Memory allocation | `256Mi`, `512Mi`, `1Gi` |
| `--cpu` | CPU allocation | `1`, `2` |
| `--min-instances` | Minimum running instances | `0` (default, scales to zero) |
| `--max-instances` | Maximum instances | `10` |
| `--set-env-vars` | Environment variables | `KEY1=val1,KEY2=val2` |
| `--port` | Container port | `8080` (default) |
| `--service-account` | Custom service account | `sa@project.iam.gserviceaccount.com` |

## Custom Domain

```bash
# Map domain to service
gcloud run domain-mappings create \
  --service hello-world \
  --domain app.example.com \
  --region us-central1

# Then add the CNAME record shown in output to your DNS
```

## Useful Commands

```bash
# List services
gcloud run services list

# View logs
gcloud run services logs read SERVICE_NAME --region us-central1

# Delete service
gcloud run services delete SERVICE_NAME --region us-central1 --quiet

# Update env vars on existing service
gcloud run services update SERVICE_NAME \
  --set-env-vars KEY=value \
  --region us-central1
```

## Cost Notes

- Cloud Run scales to zero by default — no traffic = no cost
- Free tier: 2 million requests/month, 360,000 GB-seconds of memory
- Source-based deploys use Cloud Build (120 free build-minutes/day)
