---
name: AWS ECR + EC2 Admin Dashboard
description: Backend endpoints and frontend panels for ECR image listing and EC2 instance management via Lambda
category: infrastructure
tags: [aws, ecr, ec2, lambda, admin, dashboard]
---

# AWS ECR + EC2 Admin Dashboard

Provide ECR image browsing and EC2 instance lifecycle management (start/stop/reboot/terminate) through a Lambda-backed admin API and vanilla JS frontend.

## Backend (Lambda + boto3)

### Required IAM Permissions

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ECRReadOnly",
      "Effect": "Allow",
      "Action": ["ecr:DescribeRepositories", "ecr:DescribeImages", "ecr:ListImages"],
      "Resource": "*"
    },
    {
      "Sid": "EC2InstanceManagement",
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeInstances",
        "ec2:StartInstances", "ec2:StopInstances",
        "ec2:RebootInstances", "ec2:TerminateInstances"
      ],
      "Resource": "*"
    }
  ]
}
```

### ECR Endpoint Response Shape

```json
{
  "registry_id": "123456789012",
  "repositories": [
    {
      "name": "my-app",
      "images": [
        {
          "tags": ["latest", "v1.2.3"],
          "digest": "sha256:abc...",
          "size_mb": 142.3,
          "pushed_at": "2026-03-15T10:30:00+00:00"
        }
      ]
    }
  ]
}
```

### EC2 Endpoint Response Shape

```json
{
  "instances": [
    {
      "instance_id": "i-0abc123",
      "name": "my-server",
      "state": "running",
      "type": "t3.medium",
      "az": "us-east-1a",
      "public_ip": "3.4.5.6",
      "private_ip": "10.0.1.5",
      "launch_time": "2026-03-01T08:00:00+00:00"
    }
  ]
}
```

### EC2 Actions

- `POST /admin/ec2-instances/{instance_id}/stop`
- `POST /admin/ec2-instances/{instance_id}/start`
- `POST /admin/ec2-instances/{instance_id}/reboot`
- `POST /admin/ec2-instances/{instance_id}/terminate`

All return `{"ok": true, "instance_id": "...", "action": "..."}`.

### Key Implementation Notes

- Filter out `terminated` instances from listings (they linger in AWS API)
- Sort instances: running first, then by name
- Use `imagePushedAt.isoformat()` for datetime serialization (boto3 returns datetime objects)
- Sort ECR repos alphabetically, images by push date descending
- Increase Lambda timeout to 30s (ECR describe_images can be slow with many repos)

## Frontend (Vanilla JS)

### EC2 Action Buttons

- **Running instances**: Stop (yellow), Reboot (blue), Terminate (red)
- **Stopped instances**: Start (green), Terminate (red)
- Terminate always requires explicit confirmation with instance ID shown
- Auto-refresh the panel 3 seconds after any action (allows AWS state to propagate)

### State Color Coding

| State | Color | Dot |
|-------|-------|-----|
| running | #22c55e | green |
| stopped | #f87171 | red |
| pending/stopping/other | #f59e0b | amber |
