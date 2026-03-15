---
name: on-demand-deploy
description: "AWS infrastructure for on-demand EC2 deployment: Lambda launcher/status/cleanup, API Gateway, EventBridge TTL, IAM roles, and setup automation"
category: infrastructure
difficulty: advanced
tags: [aws, ec2, lambda, api-gateway, eventbridge, on-demand, auto-destroy, ttl, ecr, docker]
stack: [aws-lambda, api-gateway, ec2, ecr, eventbridge, iam, python]
---

# On-Demand Deploy Infrastructure

You are an infrastructure engineer building serverless on-demand deploy infrastructure on AWS. This pattern provisions EC2 instances via Lambda + API Gateway, runs Docker containers from ECR, provides status polling, and auto-terminates expired instances via EventBridge.

## Architecture

```
Client (browser)
  ├── POST /deploy          → API Gateway → Lambda (agent-launcher) → EC2 t3.small
  ├── GET /status/{id}      → API Gateway → Lambda (agent-status)   → describe instance
  └── POST /deploy (_kill)  → API Gateway → Lambda (agent-launcher) → terminate instance

EventBridge (rate: 5 min) → Lambda (agent-cleanup) → terminates expired instances
```

## Lambda: agent-launcher

Handles both deploy and kill operations.

```python
"""
Lambda: agent-launcher
Launches EC2 t3.small with Docker, pulls ECR image, runs the agent.
Also handles kill requests (terminate instance).
"""
import json
import boto3
import time
import os

ec2 = boto3.client('ec2', region_name='us-east-1')
ecr_uri = os.environ.get('ECR_URI', 'ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com')

# Agent configs: agent_id -> {image, port, name}
AGENTS = {
    'my-agent': {
        'image': f'{ecr_uri}/namespace/my-agent:latest',
        'port': 8008,
        'name': 'My Agent',
    },
}

SECURITY_GROUP_ID = os.environ.get('SECURITY_GROUP_ID', '')
SUBNET_ID = os.environ.get('SUBNET_ID', '')
KEY_NAME = os.environ.get('KEY_NAME', '')
AMI_ID = os.environ.get('AMI_ID', '')  # Amazon Linux 2023


def get_user_data(agent_config):
    """Bootstrap: install Docker, pull ECR image, run container, self-tag ready."""
    image = agent_config['image']
    port = agent_config['port']
    return f"""#!/bin/bash
set -ex
yum update -y
yum install -y docker
systemctl start docker
systemctl enable docker

# Login to ECR
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin {ecr_uri}

# Pull and run
docker pull {image}
docker run -d --name agent -p 80:{port} --restart unless-stopped {image}

# Signal ready via IMDSv2 self-tagging
TOKEN=$(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 60")
INSTANCE_ID=$(curl -s -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/instance-id)
aws ec2 create-tags --resources "$INSTANCE_ID" --tags Key=AgentStatus,Value=running --region us-east-1 || true
"""


def handler(event, context):
    if isinstance(event.get('body'), str):
        body = json.loads(event['body'])
    else:
        body = event.get('body') or event

    agent_id = body.get('agent_id', '')

    # Kill endpoint
    if agent_id == '_kill':
        instance_id = body.get('instance_id')
        if not instance_id:
            return response(400, {'error': 'instance_id required for kill'})
        try:
            # SECURITY: Only terminate agent-managed instances
            desc = ec2.describe_instances(InstanceIds=[instance_id])
            tags = {t['Key']: t['Value'] for t in desc['Reservations'][0]['Instances'][0].get('Tags', [])}
            if tags.get('ManagedBy') != 'agent-launcher':
                return response(403, {'error': 'Not an agent-managed instance'})
            ec2.terminate_instances(InstanceIds=[instance_id])
            return response(200, {'terminated': instance_id})
        except Exception as e:
            return response(500, {'error': str(e)})

    if agent_id not in AGENTS:
        return response(400, {'error': f'Unknown agent: {agent_id}'})

    agent_config = AGENTS[agent_id]

    try:
        result = ec2.run_instances(
            ImageId=AMI_ID,
            InstanceType='t3.small',
            KeyName=KEY_NAME,
            SecurityGroupIds=[SECURITY_GROUP_ID],
            SubnetId=SUBNET_ID,
            MinCount=1, MaxCount=1,
            UserData=get_user_data(agent_config),
            IamInstanceProfile={'Name': 'agent-ec2-profile'},
            TagSpecifications=[{
                'ResourceType': 'instance',
                'Tags': [
                    {'Key': 'Name', 'Value': f'agent-{agent_id}'},
                    {'Key': 'AgentId', 'Value': agent_id},
                    {'Key': 'AgentStatus', 'Value': 'provisioning'},
                    {'Key': 'TTL', 'Value': str(int(time.time()) + 1800)},
                    {'Key': 'ManagedBy', 'Value': 'agent-launcher'},
                ]
            }],
        )

        instance_id = result['Instances'][0]['InstanceId']

        # Wait for public IP (up to 60s)
        public_ip = None
        for _ in range(12):
            time.sleep(5)
            desc = ec2.describe_instances(InstanceIds=[instance_id])
            public_ip = desc['Reservations'][0]['Instances'][0].get('PublicIpAddress')
            if public_ip:
                break

        return response(200, {
            'instance_id': instance_id,
            'public_ip': public_ip,
            'port': 80,
            'url': f'http://{public_ip}' if public_ip else None,
            'agent_id': agent_id,
            'ttl_seconds': 1800,
        })
    except Exception as e:
        return response(500, {'error': str(e)})


def response(status, body):
    return {
        'statusCode': status,
        'headers': {
            'Content-Type': 'application/json',
            'Access-Control-Allow-Origin': '*',
            'Access-Control-Allow-Methods': 'POST,GET,OPTIONS',
            'Access-Control-Allow-Headers': 'Content-Type',
        },
        'body': json.dumps(body),
    }
```

## Lambda: agent-status

```python
"""
Lambda: agent-status
Returns instance state, public IP, agent readiness, and TTL.
"""
import json
import boto3

ec2 = boto3.client('ec2', region_name='us-east-1')


def handler(event, context):
    params = event.get('pathParameters') or {}
    query = event.get('queryStringParameters') or {}
    instance_id = params.get('instance_id') or query.get('instance_id')

    if not instance_id:
        return response(400, {'error': 'instance_id required'})

    try:
        desc = ec2.describe_instances(InstanceIds=[instance_id])
        inst = desc['Reservations'][0]['Instances'][0]
        state = inst['State']['Name']
        public_ip = inst.get('PublicIpAddress')
        tags = {t['Key']: t['Value'] for t in inst.get('Tags', [])}

        return response(200, {
            'instance_id': instance_id,
            'state': state,
            'public_ip': public_ip,
            'url': f'http://{public_ip}' if public_ip else None,
            'agent_id': tags.get('AgentId', ''),
            'agent_status': tags.get('AgentStatus', 'unknown'),
            'ttl': int(tags.get('TTL', 0)),
        })
    except Exception as e:
        return response(500, {'error': str(e)})


def response(status, body):
    return {
        'statusCode': status,
        'headers': {
            'Content-Type': 'application/json',
            'Access-Control-Allow-Origin': '*',
            'Access-Control-Allow-Methods': 'GET,OPTIONS',
            'Access-Control-Allow-Headers': 'Content-Type',
        },
        'body': json.dumps(body),
    }
```

## Lambda: agent-cleanup

```python
"""
Lambda: agent-cleanup
EventBridge scheduled (every 5 min). Terminates expired agent instances.
SAFETY: Only targets instances tagged ManagedBy=agent-launcher.
"""
import json
import boto3
import time

ec2 = boto3.client('ec2', region_name='us-east-1')


def handler(event, context):
    now = int(time.time())
    terminated = []

    try:
        result = ec2.describe_instances(Filters=[
            {'Name': 'tag:ManagedBy', 'Values': ['agent-launcher']},
            {'Name': 'instance-state-name', 'Values': ['running', 'pending']},
        ])

        for reservation in result['Reservations']:
            for inst in reservation['Instances']:
                tags = {t['Key']: t['Value'] for t in inst.get('Tags', [])}
                ttl = int(tags.get('TTL', 0))
                instance_id = inst['InstanceId']

                if ttl > 0 and now > ttl:
                    ec2.terminate_instances(InstanceIds=[instance_id])
                    terminated.append(instance_id)

    except Exception as e:
        return {'statusCode': 500, 'body': json.dumps({'error': str(e)})}

    return {
        'statusCode': 200,
        'body': json.dumps({
            'checked_at': now,
            'terminated': terminated,
            'count': len(terminated),
        }),
    }
```

## Setup Script

Automates creation of all AWS resources. Run once per account/region.

```bash
#!/bin/bash
set -euo pipefail
REGION="us-east-1"
ACCOUNT_ID="YOUR_ACCOUNT_ID"
VPC_ID="YOUR_VPC_ID"
SUBNET_ID="YOUR_SUBNET_ID"

# 1. Security Group (inbound 80, 443, 22)
SG_ID=$(aws ec2 create-security-group \
  --group-name sats-agent-sg \
  --description "On-demand agent instances" \
  --vpc-id "$VPC_ID" \
  --query 'GroupId' --output text 2>/dev/null || \
  aws ec2 describe-security-groups --filters "Name=group-name,Values=sats-agent-sg" \
    --query 'SecurityGroups[0].GroupId' --output text)

for PORT in 80 443 22; do
  aws ec2 authorize-security-group-ingress --group-id "$SG_ID" \
    --protocol tcp --port $PORT --cidr 0.0.0.0/0 2>/dev/null || true
done

# 2. Key Pair
aws ec2 create-key-pair --key-name sats-agent-key \
  --query 'KeyMaterial' --output text > /tmp/sats-agent-key.pem 2>/dev/null && \
  chmod 600 /tmp/sats-agent-key.pem || echo "Key pair exists"

# 3. IAM Role for EC2 (ECR pull + self-tagging)
aws iam create-role --role-name agent-ec2-role \
  --assume-role-policy-document '{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Principal":{"Service":"ec2.amazonaws.com"},"Action":"sts:AssumeRole"}]}' 2>/dev/null || true

aws iam put-role-policy --role-name agent-ec2-role \
  --policy-name agent-ec2-permissions \
  --policy-document '{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Action":["ecr:GetAuthorizationToken","ecr:BatchCheckLayerAvailability","ecr:GetDownloadUrlForLayer","ecr:BatchGetImage"],"Resource":"*"},{"Effect":"Allow","Action":["ec2:CreateTags"],"Resource":"*"}]}'

aws iam create-instance-profile --instance-profile-name agent-ec2-profile 2>/dev/null || true
aws iam add-role-to-instance-profile --instance-profile-name agent-ec2-profile \
  --role-name agent-ec2-role 2>/dev/null || true

# 4. IAM Role for Lambda
aws iam create-role --role-name agent-launcher-role \
  --assume-role-policy-document '{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Principal":{"Service":"lambda.amazonaws.com"},"Action":"sts:AssumeRole"}]}' 2>/dev/null || true

aws iam put-role-policy --role-name agent-launcher-role \
  --policy-name agent-launcher-permissions \
  --policy-document '{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Action":["ec2:RunInstances","ec2:DescribeInstances","ec2:TerminateInstances","ec2:CreateTags","ec2:DescribeTags","iam:PassRole"],"Resource":"*"},{"Effect":"Allow","Action":["logs:CreateLogGroup","logs:CreateLogStream","logs:PutLogEvents"],"Resource":"arn:aws:logs:*:*:*"}]}'

# 5. Find AMI
AMI_ID=$(aws ec2 describe-images --owners amazon \
  --filters "Name=name,Values=al2023-ami-2023*-x86_64" "Name=state,Values=available" \
  --query 'sort_by(Images, &CreationDate)[-1].ImageId' --output text)

sleep 10  # IAM propagation

# 6. Lambda functions (zip and create/update)
ROLE_ARN="arn:aws:iam::${ACCOUNT_ID}:role/agent-launcher-role"
ENV_VARS="Variables={SECURITY_GROUP_ID=$SG_ID,SUBNET_ID=$SUBNET_ID,AMI_ID=$AMI_ID,ECR_URI=${ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com}"

for FUNC in launcher status cleanup; do
  zip -j /tmp/${FUNC}.zip infra/${FUNC}.py
  HANDLER="${FUNC}.handler"
  TIMEOUT=60; [ "$FUNC" = "launcher" ] && TIMEOUT=120
  MEM=128; [ "$FUNC" = "launcher" ] && MEM=256

  aws lambda create-function --function-name "agent-${FUNC}" \
    --runtime python3.12 --handler "$HANDLER" --role "$ROLE_ARN" \
    --zip-file "fileb:///tmp/${FUNC}.zip" --timeout $TIMEOUT --memory-size $MEM \
    --environment "$ENV_VARS" 2>/dev/null || \
  aws lambda update-function-code --function-name "agent-${FUNC}" \
    --zip-file "fileb:///tmp/${FUNC}.zip" > /dev/null
done

# 7. API Gateway (HTTP API with CORS)
API_ID=$(aws apigatewayv2 create-api --name AgentDeployAPI \
  --protocol-type HTTP \
  --cors-configuration AllowOrigins='*',AllowMethods='POST,GET,OPTIONS',AllowHeaders='Content-Type' \
  --query 'ApiId' --output text 2>/dev/null || \
  aws apigatewayv2 get-apis --query "Items[?Name=='AgentDeployAPI'].ApiId|[0]" --output text)

# Create integrations and routes
for FUNC in launcher status; do
  LAMBDA_ARN="arn:aws:lambda:${REGION}:${ACCOUNT_ID}:function:agent-${FUNC}"
  INT_ID=$(aws apigatewayv2 create-integration --api-id "$API_ID" \
    --integration-type AWS_PROXY --integration-uri "$LAMBDA_ARN" \
    --payload-format-version "2.0" --query 'IntegrationId' --output text 2>/dev/null || echo "exists")

  if [ "$FUNC" = "launcher" ]; then
    aws apigatewayv2 create-route --api-id "$API_ID" --route-key "POST /deploy" \
      --target "integrations/$INT_ID" 2>/dev/null || true
  else
    aws apigatewayv2 create-route --api-id "$API_ID" --route-key "GET /status/{instance_id}" \
      --target "integrations/$INT_ID" 2>/dev/null || true
  fi

  aws lambda add-permission --function-name "agent-${FUNC}" \
    --statement-id apigateway-invoke --action lambda:InvokeFunction \
    --principal apigateway.amazonaws.com \
    --source-arn "arn:aws:execute-api:${REGION}:${ACCOUNT_ID}:${API_ID}/*" 2>/dev/null || true
done

aws apigatewayv2 create-stage --api-id "$API_ID" --stage-name prod --auto-deploy 2>/dev/null || true
API_ENDPOINT=$(aws apigatewayv2 get-api --api-id "$API_ID" --query 'ApiEndpoint' --output text)

# 8. EventBridge cleanup schedule (every 5 min)
CLEANUP_ARN="arn:aws:lambda:${REGION}:${ACCOUNT_ID}:function:agent-cleanup"
aws events put-rule --name agent-cleanup-schedule --schedule-expression "rate(5 minutes)" --state ENABLED 2>/dev/null || true
aws events put-targets --rule agent-cleanup-schedule --targets "Id=cleanup,Arn=$CLEANUP_ARN" 2>/dev/null || true
aws lambda add-permission --function-name agent-cleanup \
  --statement-id eventbridge-invoke --action lambda:InvokeFunction \
  --principal events.amazonaws.com \
  --source-arn "arn:aws:events:${REGION}:${ACCOUNT_ID}:rule/agent-cleanup-schedule" 2>/dev/null || true

echo "API Endpoint: ${API_ENDPOINT}/prod"
```

## AWS Resources Created

| Resource | Name | Purpose |
|----------|------|---------|
| Security Group | `sats-agent-sg` | Inbound 80, 443, 22 for agent instances |
| Key Pair | `sats-agent-key` | SSH access for debugging |
| IAM Role | `agent-ec2-role` | EC2: ECR pull + self-tagging |
| Instance Profile | `agent-ec2-profile` | Attaches EC2 role to instances |
| IAM Role | `agent-launcher-role` | Lambda: EC2 + logs permissions |
| Lambda | `agent-launcher` | Deploy + kill endpoint |
| Lambda | `agent-status` | Instance state polling |
| Lambda | `agent-cleanup` | TTL-based auto-termination |
| API Gateway | `AgentDeployAPI` | HTTP API with CORS |
| EventBridge Rule | `agent-cleanup-schedule` | Triggers cleanup every 5 min |

## Tag-Based Safety

All agent instances are tagged with `ManagedBy=agent-launcher`. This tag is the safety boundary:

- **Kill endpoint**: Verifies `ManagedBy=agent-launcher` before terminating — returns 403 otherwise
- **Cleanup Lambda**: Filters `tag:ManagedBy=agent-launcher` — never touches untagged instances
- **TTL tag**: Unix timestamp, checked by cleanup — instances auto-destroyed after 30 min

## EC2 UserData Bootstrap Flow

1. Install and start Docker
2. ECR login via `aws ecr get-login-password`
3. Pull Docker image from ECR
4. Run container mapping port 80 → app port
5. Self-tag `AgentStatus=running` via IMDSv2 (token-based metadata)

## Adding a New Agent

1. Push Docker image to ECR: `ACCOUNT.dkr.ecr.REGION.amazonaws.com/namespace/agent-name:latest`
2. Add entry to `AGENTS` dict in `launcher.py`:
   ```python
   'agent-name': {
       'image': f'{ecr_uri}/namespace/agent-name:latest',
       'port': APP_PORT,
       'name': 'Agent Display Name',
   },
   ```
3. Update Lambda: `zip -j /tmp/launcher.zip launcher.py && aws lambda update-function-code --function-name agent-launcher --zip-file fileb:///tmp/launcher.zip`
4. **Update the Solution Registry tour** — see "Post-Deploy: Update Solution Registry Tour" below

## Post-Deploy: Update Solution Registry Tour

**MANDATORY** after every new app, agent, or academy deployment. The "Take the Tour" on the Solution Registry must always reflect the complete portfolio.

1. Open `/Users/Sats/Documents/TechnicalPlayGround/CodexFolder/deployed-apps.html`
2. Find `var TOUR_STEPS = [` and add a new step in the correct category section:
   ```javascript
   {
     icon: 'EMOJI', title: 'App Name',
     subtitle: 'Category · Short tagline',
     gradient: 'linear-gradient(135deg,#COLOR1,#COLOR2)', url: 'https://SUBDOMAIN.satszone.link',
     desc: 'Full description...',
     features: ['Feature 1', 'Feature 2', 'Feature 3', 'Tests · Key differentiator']
   },
   ```
3. Update the **Welcome step** (first) — counts and 6-category breakdown
4. Update the **Finale step** (last) — total count, subtitle, and features
5. Push and deploy:
   ```bash
   cp deployed-apps.html /Users/Sats/Documents/TechnicalPlayGround/CodexFolder/solution-registry/index.html
   cd /Users/Sats/Documents/TechnicalPlayGround/CodexFolder/solution-registry
   git add index.html && git commit -m "feat: add APP_NAME to tour" && git push origin main
   aws s3 cp /Users/Sats/Documents/TechnicalPlayGround/CodexFolder/deployed-apps.html s3://my-solution-registry.satszone.link/index.html --content-type "text/html"
   aws cloudfront create-invalidation --distribution-id E2R00426B8QGNB --paths "/*"
   ```

## API Reference

### POST /deploy (launch)
```json
// Request
{"agent_id": "my-agent"}

// Response
{"instance_id": "i-xxx", "public_ip": "1.2.3.4", "port": 80, "url": "http://1.2.3.4", "agent_id": "my-agent", "ttl_seconds": 1800}
```

### POST /deploy (kill)
```json
// Request
{"agent_id": "_kill", "instance_id": "i-xxx"}

// Response
{"terminated": "i-xxx"}
```

### GET /status/{instance_id}
```json
// Response
{"instance_id": "i-xxx", "state": "running", "public_ip": "1.2.3.4", "url": "http://1.2.3.4", "agent_id": "my-agent", "agent_status": "running", "ttl": 1773450263}
```

## Docker Image Requirements

- Multi-stage build recommended: Node (frontend) + Python (backend)
- Must have `.dockerignore` excluding `frontend/node_modules` (broken symlinks on Linux)
- Health endpoint at `/api/health` (not `/` — that serves the SPA)
- Frontend served via `StaticFiles(directory="static", html=True)` mounted at `/`
- Single port exposed, mapped to port 80 on EC2

## Key Gotchas

1. **IMDSv2 required**: Use token-based metadata calls, not plain `curl`
2. **IAM propagation**: Wait ~10s after creating roles before creating Lambdas
3. **ec2:CreateTags condition**: Don't restrict by `ResourceTag/ManagedBy` on new instances — the tag doesn't exist yet at creation time
4. **Mixed content**: Don't health-check HTTP apps from HTTPS pages — use the status API instead
5. **Node modules symlinks**: macOS creates symlinks in `node_modules` that break in Linux Docker — always use `.dockerignore`
