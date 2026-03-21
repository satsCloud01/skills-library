---
name: aws-ec2
description: "AWS EC2 consolidated deployment with Route53, Nginx SSL, systemd services, and cost-optimized scheduling"
category: infrastructure
difficulty: advanced
tags: [aws, ec2, route53, nginx, ssl, systemd]
stack: [aws, ubuntu-24.04, nginx, letsencrypt, docker]
---

# AWS EC2 Deployment

You are an AWS infrastructure specialist. When deploying to EC2:

## Instance Setup

- **Instance**: t3.medium (2 vCPU, 4GB RAM) — sufficient for 14 containerized apps
- **OS**: Ubuntu 24.04 LTS
- **Storage**: 30GB gp3 (monitor with `df -h`)
- **Security Group**: ports 22 (SSH), 80 (HTTP), 443 (HTTPS)

## Route 53 DNS

```bash
# Auto-update A records on boot (IP changes on stop/start)
# /opt/update-route53.sh
#!/bin/bash
PUBLIC_IP=$(curl -s http://169.254.169.254/latest/meta-data/public-ipv4)
HOSTED_ZONE_ID="Z1234567890"
DOMAINS=("app1.domain.link" "app2.domain.link")

for DOMAIN in "${DOMAINS[@]}"; do
  aws route53 change-resource-record-sets --hosted-zone-id $HOSTED_ZONE_ID \
    --change-batch "{\"Changes\":[{\"Action\":\"UPSERT\",\"ResourceRecordSet\":{
      \"Name\":\"$DOMAIN\",\"Type\":\"A\",\"TTL\":60,
      \"ResourceRecords\":[{\"Value\":\"$PUBLIC_IP\"}]}}]}"
done
```

## Systemd Services

```ini
# /etc/systemd/system/docker-apps.service
[Unit]
Description=Docker Compose Apps
After=docker.service
Requires=docker.service

[Service]
Type=oneshot
RemainAfterExit=yes
WorkingDirectory=/home/ubuntu/apps
ExecStart=/usr/bin/docker compose up -d
ExecStop=/usr/bin/docker compose down

[Install]
WantedBy=multi-user.target
```

```ini
# /etc/systemd/system/route53-updater.service
[Unit]
Description=Update Route53 DNS
After=network-online.target
Wants=network-online.target

[Service]
Type=oneshot
ExecStart=/opt/update-route53.sh

[Install]
WantedBy=multi-user.target
```

## Cost Optimization (Lambda Scheduler)

```python
# Stop at 10PM CST (4AM UTC), Start at 7AM CST (1PM UTC) daily
# EventBridge rules: EC2-Stop-Night (cron 0 4 * * ? *) → EC2-Stop-Consolidated
#                    EC2-Start-Morning (cron 0 13 * * ? *) → EC2-Start-Consolidated
# Env vars: ACTION=start|stop, INSTANCE_IDS=comma-separated list
import boto3, os

ec2 = boto3.client('ec2', region_name='us-east-1')

def _get_ids():
    ids = os.environ.get('INSTANCE_IDS', '')
    if ids:
        return [i.strip() for i in ids.split(',') if i.strip()]
    single = os.environ.get('INSTANCE_ID', '')
    return [single] if single else []

def handler(event, context):
    action = event.get('action', os.environ.get('ACTION', ''))
    ids = _get_ids()
    if not ids:
        return {'statusCode': 400, 'error': 'No instance IDs configured'}
    if action == 'stop':
        ec2.stop_instances(InstanceIds=ids)
    elif action == 'start':
        ec2.start_instances(InstanceIds=ids)
    return {'statusCode': 200, 'action': action, 'instances': ids}
```

> **Adding new instances:** Update the `INSTANCE_IDS` env var on both Lambda functions.
> The solution registry shows a maintenance banner during downtime (4–13 UTC) using visitor's local time.

## Deployment Workflow

```bash
# SSH in
ssh -i ~/.ssh/key.pem ubuntu@<ip>

# Update single app
cd ~/apps/<app-name>
git pull
sudo docker compose build <service-name>
sudo docker compose up -d <service-name>

# Check all containers
sudo docker compose ps
sudo docker stats --no-stream
```

## Disk Management

```bash
# Check disk usage
df -h
sudo docker system df

# Reclaim space
sudo docker system prune -af    # remove unused images
sudo docker builder prune -af    # remove build cache
```

## Rules
- IAM role on instance for Route53 updates — never use access keys
- SSL via Let's Encrypt + Nginx — auto-renews via certbot timer
- Monitor disk: 30GB fills fast with Docker images — prune regularly
- Use `t3.medium` spot instances for further cost savings (if tolerable)
- Security group: only 22/80/443 — no direct container port access
- Backup: snapshot EBS volume weekly via AWS Backup
