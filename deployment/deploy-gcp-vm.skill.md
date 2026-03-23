---
name: deploy-gcp-vm
description: "Deploy a satszone.link app to GCP free tier e2-micro VM with Docker, Nginx, SSL, Route 53 DNS, and room key gate"
category: deployment
difficulty: intermediate
tags: [gcp, deployment, docker, nginx, free-tier, route53]
stack: [gcp, docker, nginx, python-3.12, certbot, aws-route53]
---

# GCP VM Deployment (Free Tier e2-micro)

You are deploying a satszone.link app to a GCP free tier e2-micro instance. The app runs in Docker behind Nginx with SSL, DNS via AWS Route 53, and room key gate authentication.

## Architecture

```
Browser → Route 53 (AWS) → GCP Static IP → Nginx (SSL + auth_request)
  → gate-auth-server.py (validates cookie via AWS Lambda API)
  → Docker container (app on localhost:PORT)
```

Key: **GCP hosts the app, AWS provides DNS + auth**. The gate-auth-server on GCP calls the same Lambda API used by all EC2 apps.

## Prerequisites

1. GCP project with Compute Engine API enabled
2. `gcloud` CLI configured locally
3. AWS CLI configured (for Route 53 + CORS updates)
4. App has a working `Dockerfile`

## Step 1: Provision GCP e2-micro VM

Free tier eligible zones: `us-east1`, `us-west1`, `us-central1`. Zones often exhaust — try multiple.

```bash
gcloud compute instances create APP-gcp \
  --zone=us-east1-d \
  --machine-type=e2-micro \
  --image-family=ubuntu-2204-lts \
  --image-project=ubuntu-os-cloud \
  --boot-disk-size=30GB \
  --boot-disk-type=pd-standard \
  --tags=http-server,https-server
```

## Step 2: Firewall Rules (once per project)

```bash
gcloud compute firewall-rules create allow-http --allow=tcp:80 --target-tags=http-server --direction=INGRESS
gcloud compute firewall-rules create allow-https --allow=tcp:443 --target-tags=https-server --direction=INGRESS
```

## Step 3: Reserve Static IP (free when attached to running VM)

```bash
gcloud compute addresses create APP-ip --region=us-east1
GCP_IP=$(gcloud compute addresses describe APP-ip --region=us-east1 --format='value(address)')

# Swap ephemeral for static
gcloud compute instances delete-access-config APP-gcp --zone=us-east1-d --access-config-name="external-nat"
gcloud compute instances add-access-config APP-gcp --zone=us-east1-d --address=$GCP_IP
```

## Step 4: Add CORS Origin to API Gateway

**CRITICAL: Without this, the room key gate shows "Connection error".**

```bash
# Check current origins
aws apigatewayv2 get-api --api-id dr9yrgyzg2 --query "CorsConfiguration.AllowOrigins"

# Add new origin (include ALL existing + new)
aws apigatewayv2 update-api --api-id dr9yrgyzg2 --cors-configuration '{
  "AllowHeaders": ["content-type", "x-admin-key"],
  "AllowMethods": ["*"],
  "AllowOrigins": [ ...existing..., "https://APP.satszone.link" ],
  "MaxAge": 3600
}'
```

## Step 5: Deploy App via Docker

```bash
# Create tarball locally (exclude heavy dirs)
cd /path/to/app
tar czf /tmp/app.tar.gz --exclude='.venv' --exclude='node_modules' --exclude='__pycache__' --exclude='.git' --exclude='*.db' --exclude='product-video' .

# Upload and build on GCP
gcloud compute scp /tmp/app.tar.gz APP-gcp:/tmp/ --zone=us-east1-d
gcloud compute ssh APP-gcp --zone=us-east1-d --command='
sudo apt update -qq && sudo apt install -y -qq docker.io nginx certbot python3-certbot-nginx
sudo systemctl enable docker && sudo systemctl start docker

# Swap (e2-micro only has 1GB RAM)
sudo fallocate -l 2G /swapfile && sudo chmod 600 /swapfile
sudo mkswap /swapfile && sudo swapon /swapfile
echo "/swapfile none swap sw 0 0" | sudo tee -a /etc/fstab

# Build and run
sudo mkdir -p /opt/APP && cd /opt/APP && sudo tar xzf /tmp/app.tar.gz
sudo docker build -t APP .
sudo docker run -d --restart always -p PORT:PORT --name APP APP
'
```

## Step 6: Deploy Room Key Gate

Follow the `nginx-room-key-gate` skill, Step 5. Then use **Nginx Pattern E** (not Pattern B):

### Nginx Pattern E — GCP/Non-EC2 (REQUIRED for all non-EC2 deployments)

Three critical differences from EC2 Pattern B:
1. **`sub_filter`** strips `gate.js` URL → prevents duplicate client-side gate overlay
2. **`Accept-Encoding ""`** disables gzip so sub_filter can match the HTML
3. **`Cache-Control: no-store`** prevents browser caching gate page as 304 after auth passes

```nginx
server {
    server_name APP.satszone.link;
    include /etc/nginx/snippets/gate-auth.conf;
    error_page 401 =200 /_gate;

    # CRITICAL: Prevent browser caching gate vs app (304 stale response bug)
    add_header Cache-Control "no-store, no-cache, must-revalidate" always;
    add_header Pragma "no-cache" always;
    expires 0;

    client_max_body_size 50M;

    location / {
        auth_request /_auth;
        proxy_pass http://127.0.0.1:PORT;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_read_timeout 300s;

        # CRITICAL: Disable gzip so sub_filter can match
        proxy_set_header Accept-Encoding "";

        # CRITICAL: Strip client-side gate.js to prevent duplicate gate overlay
        sub_filter_once on;
        sub_filter "my-solution-registry.satszone.link/gate.js" "about:blank";
    }

    location /api/ {
        proxy_pass http://127.0.0.1:PORT;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location /assets/ {
        auth_request /_auth;
        proxy_pass http://127.0.0.1:PORT;
        proxy_set_header Host $host;
        expires 7d;
        add_header Cache-Control "public, immutable";
    }
}
```

## Step 7: Route 53 DNS + SSL

```bash
# DNS
aws route53 change-resource-record-sets --hosted-zone-id Z047244933J6LNZKU0UTJ --change-batch '{
  "Changes": [{"Action":"UPSERT","ResourceRecordSet":{"Name":"APP.satszone.link","Type":"A","TTL":300,"ResourceRecords":[{"Value":"GCP_IP"}]}}]
}'

# Wait for propagation, then SSL
sleep 30
gcloud compute ssh APP-gcp --zone=us-east1-d --command='
sudo certbot --nginx -d APP.satszone.link --non-interactive --agree-tos -m admin@satszone.link
'
```

## Step 8: Verify

```bash
# Gate should appear
curl -s https://APP.satszone.link | grep -c "Access Required"
# Should be > 0
```

## GCP-Specific Gotchas

1. **Zone exhaustion**: `e2-micro` often unavailable. Try `us-east1-b`, `us-east1-c`, `us-east1-d` in sequence.
2. **Private repos**: `git clone` won't work on GCP without SSH keys. Use `tar + scp` instead.
3. **1GB RAM**: Always add 2GB swap. Docker builds may fail without it.
4. **Static IP cost**: Free only when attached to a running VM. Detached IPs cost ~$3/mo.
5. **CORS**: Must add subdomain to API Gateway `dr9yrgyzg2` CORS AllowOrigins BEFORE deploying.
6. **gate.js overlay**: Use Pattern E `sub_filter` or users see TWO gate prompts (Nginx + client-side).
7. **304 caching**: Use Pattern E `Cache-Control: no-store` or browser caches gate page forever.
8. **Repo cloning**: GCP VM has no GitHub SSH keys — tar the app locally and SCP it.

## Automated Deploy Script

For AgentSubstrate, a reusable deploy script exists at:
`agentic-data-os/deploy/deploy-gcp.sh <GCP_IP> <DOMAIN> <PORT> [ZONE]`

It handles: CORS update, packages, Docker build, gate auth, Nginx Pattern E, Route 53, SSL, verification.
