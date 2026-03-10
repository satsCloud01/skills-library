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
# Stop at 10PM CST (4AM UTC), Start at 7AM CST (1PM UTC) Mon-Fri
# EventBridge rules → Lambda functions
import boto3

def stop_handler(event, context):
    boto3.client('ec2', region_name='us-east-1').stop_instances(
        InstanceIds=['i-0123456789abcdef0'])

def start_handler(event, context):
    boto3.client('ec2', region_name='us-east-1').start_instances(
        InstanceIds=['i-0123456789abcdef0'])
```

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
