---
name: room-key-api
description: "Serverless room-key access control API with DynamoDB, SES approval workflow, rate limiting, key expiry, and admin CRUD — deployed via AWS SAM"
category: backend
difficulty: intermediate
tags: [aws, lambda, dynamodb, ses, api-gateway, sam, access-control, room-key, rate-limiting, approval-workflow]
stack: [python-3.12, aws-lambda, dynamodb, ses, api-gateway, sam]
---

# Room Key API — Serverless Access Control

You are an expert backend engineer building a serverless access-control API using AWS Lambda + DynamoDB + SES + API Gateway, deployed via SAM. This pattern protects web apps behind time-limited, rate-limited keys with an email-driven approval workflow and a 7-tab admin panel.

## Reference Implementation

Canonical implementation: `/room-key-api/` (GitHub: satsCloud01/room-key-api)
Production: `https://nl0iqz9cxd.execute-api.us-east-1.amazonaws.com/prod`

## Architecture

```
Client (gate.js / gate-auth.html)
    │
    ▼
API Gateway (REST, /prod stage, CORS locked)
    │
    ▼
Lambda (Python 3.12, single handler, 16 routes)
    │
    ├── DynamoDB — Keys, Requests, AccessLog, Identities (4 tables)
    ├── SES — approval emails, key delivery, denial notifications
    ├── Cost Explorer — MTD, prev month, 6-month trend, forecast
    ├── ECR — repository + image browser
    └── EC2 — instance list + start/stop/reboot control
```

## SAM Template Pattern

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31

Parameters:
  AllowedOrigin:
    Type: String
    Default: 'https://your-domain.com'

Globals:
  Function:
    Runtime: python3.12
    Timeout: 30
    MemorySize: 128
    Environment:
      Variables:
        KEYS_TABLE: !Ref KeysTable
        ALLOWED_ORIGIN: !Ref AllowedOrigin

Resources:
  KeysTable:
    Type: AWS::DynamoDB::Table
    Properties:
      # NEVER hardcode TableName — let CloudFormation auto-name
      BillingMode: PAY_PER_REQUEST
      AttributeDefinitions:
        - AttributeName: key_value
          AttributeType: S
      KeySchema:
        - AttributeName: key_value
          KeyType: HASH

  ApiFunction:
    Type: AWS::Serverless::Function
    Properties:
      # NEVER hardcode FunctionName
      CodeUri: lambda/
      Handler: handler.handler
      Policies:
        - DynamoDBCrudPolicy:
            TableName: !Ref KeysTable
        - DynamoDBCrudPolicy:
            TableName: YourRequestsTable
        - DynamoDBCrudPolicy:
            TableName: YourAccessLogTable
        - DynamoDBCrudPolicy:
            TableName: YourIdentitiesTable
        - Version: '2012-10-17'
          Statement:
            - Effect: Allow
              Action: [ses:SendEmail]
              Resource: '*'
            - Effect: Allow
              Action: [ce:GetCostAndUsage, ce:GetCostForecast]
              Resource: '*'
            - Effect: Allow
              Action: [ecr:DescribeRepositories, ecr:ListImages, ecr:DescribeImages]
              Resource: '*'
            - Effect: Allow
              Action: [ec2:DescribeInstances, ec2:StartInstances, ec2:StopInstances, ec2:RebootInstances]
              Resource: '*'
      Events:
        AuthValidate:
          Type: Api
          Properties:
            Path: /auth/validate
            Method: POST
            RestApiId: !Ref Api
        AuthProxy:
          Type: Api
          Properties:
            Path: /auth/{proxy+}
            Method: ANY
            RestApiId: !Ref Api
        AdminProxy:
          Type: Api
          Properties:
            Path: /admin/{proxy+}
            Method: ANY
            RestApiId: !Ref Api
        OptionsAny:
          Type: Api
          Properties:
            Path: /{proxy+}
            Method: OPTIONS
            RestApiId: !Ref Api

  Api:
    Type: AWS::Serverless::Api
    Properties:
      StageName: prod
      Cors:
        AllowOrigin: !Sub "'${AllowedOrigin}'"
        AllowHeaders: "'Content-Type,X-Admin-Key'"
        AllowMethods: "'GET,POST,PUT,DELETE,OPTIONS'"
```

## Lambda Handler — Key Validation with Rate Limiting + Expiry

```python
import json, os, uuid, secrets, string
from datetime import datetime, timezone, timedelta
import boto3

KEYS_TABLE = os.environ.get('KEYS_TABLE', 'RegistryKeys')
KEY_TTL_DAYS = int(os.environ.get('KEY_TTL_DAYS', '5'))
RATE_LIMIT_PER_DAY = int(os.environ.get('RATE_LIMIT_PER_DAY', '30'))
EXPIRY_WARN_HOURS = int(os.environ.get('EXPIRY_WARN_HOURS', '24'))
ALLOWED_ORIGIN = os.environ.get('ALLOWED_ORIGIN', '*')

dynamodb = boto3.resource('dynamodb')

def now_utc(): return datetime.now(timezone.utc)
def today_str(): return now_utc().strftime('%Y-%m-%d')

def generate_key():
    return 'srk_' + ''.join(secrets.choice(string.ascii_letters + string.digits) for _ in range(32))

def make_response(status, body):
    return {
        'statusCode': status,
        'headers': {
            'Content-Type': 'application/json',
            'Access-Control-Allow-Origin': ALLOWED_ORIGIN,
            'Access-Control-Allow-Headers': 'Content-Type,X-Admin-Key',
            'Access-Control-Allow-Methods': 'GET,POST,PUT,DELETE,OPTIONS',
        },
        'body': json.dumps(body, default=str),
    }

def validate_key(key):
    table = dynamodb.Table(KEYS_TABLE)
    item = table.get_item(Key={'key_value': key}).get('Item')
    if not item or not item.get('is_active'):
        return {'valid': False, 'can_request': True}

    # Expiry check
    if not item.get('is_admin') and item.get('expires_at'):
        expires = datetime.fromisoformat(item['expires_at'])
        if expires < now_utc():
            return {'valid': False, 'expired': True}
        hours_left = (expires - now_utc()).total_seconds() / 3600
        expiry_warning = hours_left < EXPIRY_WARN_HOURS
    else:
        hours_left = None
        expiry_warning = False

    # Rate limit check (skip for admin)
    if not item.get('is_admin'):
        today = today_str()
        count = int(item.get('rate_limit_count', 0))
        if item.get('rate_limit_date') == today and count >= RATE_LIMIT_PER_DAY:
            return {'valid': False, 'rate_limited': True}
        new_count = (count + 1) if item.get('rate_limit_date') == today else 1
        table.update_item(
            Key={'key_value': key},
            UpdateExpression='SET rate_limit_count = :c, rate_limit_date = :d',
            ExpressionAttributeValues={':c': new_count, ':d': today})

    # Log access
    _log_access(key, item.get('key_id', ''))

    return {
        'valid': True,
        'is_admin': bool(item.get('is_admin')),
        'label': item.get('label', ''),
        'expiry_warning': expiry_warning,
        'hours_remaining': round(hours_left) if hours_left else None,
    }
```

## Email-Driven Approval Workflow

```python
APPROVAL_TTL_HOURS = 48
ses = boto3.client('ses', region_name='us-east-1')

def handle_access_request(name, email, note):
    """POST /auth/request — visitor submits access request"""
    request_id = str(uuid.uuid4())
    approval_token = uuid.uuid4().hex
    denial_token = uuid.uuid4().hex
    now = now_utc()

    # Store request
    requests_table.put_item(Item={
        'request_id': request_id,
        'name': name, 'email': email, 'note': note,
        'status': 'pending',
        'created_at': now.isoformat(),
        'expires_at': (now + timedelta(hours=APPROVAL_TTL_HOURS)).isoformat(),
        'approval_token': approval_token,
        'denial_token': denial_token,
    })

    # Email admin with approve/deny links
    approve_url = f'{API_BASE_URL}/auth/approve/{approval_token}'
    deny_url = f'{API_BASE_URL}/auth/deny/{denial_token}'
    ses.send_email(
        Source=SENDER_EMAIL,
        Destination={'ToAddresses': [ADMIN_EMAIL]},
        Message={
            'Subject': {'Data': f'Access Request: {name}'},
            'Body': {'Html': {'Data': f'''
                <p><b>{name}</b> ({email}) requests access.</p>
                <p>Note: {note or "None"}</p>
                <p><a href="{approve_url}">Approve</a> | <a href="{deny_url}">Deny</a></p>
                <p><i>Links expire in {APPROVAL_TTL_HOURS}h</i></p>
            '''}},
        })
    return {'request_id': request_id}

def handle_approval(token):
    """GET /auth/approve/{token} — generates key and emails requester"""
    # Find request by token, check not expired
    # Generate key, email to requester
    # Update request status to 'approved'
    pass
```

## Route Summary

| Method | Path | Auth | Purpose |
|--------|------|------|---------|
| POST | `/auth/validate` | Public | Validate key, rate limit, expiry check |
| POST | `/auth/request` | Public | Submit access request |
| POST | `/auth/identity` | Public | Capture visitor identity |
| GET | `/auth/approve/{token}` | Token | Approve → generate + email key |
| GET | `/auth/deny/{token}` | Token | Deny → notify requester |
| GET | `/admin/keys` | Admin | List all keys |
| POST | `/admin/keys` | Admin | Generate key |
| PUT | `/admin/keys/{id}/rotate` | Admin | Rotate key value |
| PUT | `/admin/keys/{id}/toggle` | Admin | Enable/disable |
| DELETE | `/admin/keys/{id}` | Admin | Delete key |
| GET | `/admin/requests` | Admin | List requests |
| GET | `/admin/access-log` | Admin | Visit audit log |
| GET | `/admin/identities` | Admin | Visitor identities |
| GET | `/admin/aws-costs` | Admin | Cost Explorer analytics |
| GET | `/admin/ecr-images` | Admin | ECR image browser |
| GET | `/admin/ec2-instances` | Admin | EC2 list + control |
| POST | `/admin/ec2-instances/{id}/{action}` | Admin | Stop/start/reboot |

## DynamoDB Tables

| Table | PK | Purpose |
|-------|-----|---------|
| Keys | `key_value` | Room keys (admin + visitor) |
| Requests | `request_id` | Access request queue |
| AccessLog | `log_id` | Visit audit trail |
| Identities | `identity_id` | Captured visitor identities |

## Deployment Checklist

```bash
# 1. Deploy SAM stack
./deploy.sh

# 2. Bootstrap admin key
python3 init-admin.py

# 3. Update client API_ENDPOINT
# 4. Upload to S3 + invalidate CloudFront
```

## Rules

- NEVER hardcode DynamoDB TableName or Lambda FunctionName in SAM — use auto-naming to avoid orphan conflicts on redeployment
- ALWAYS use `!Ref` for table name env vars so they resolve to actual created names
- Admin key MUST be validated server-side on every `/admin/*` call
- CORS headers must be in BOTH API Gateway config AND Lambda responses
- SES sandbox requires both sender AND recipient to be verified — request production access for general use
- Rate limit resets by comparing `rate_limit_date` to today — no TTL or separate table needed
- Admin keys never expire and are protected from rotate/toggle/delete
- Use `{proxy+}` catch-all routes rather than defining each path individually
- Lambda timeout must be >= 30s — Cost Explorer and ECR API calls can be slow
- `ce:GetCostForecast` fails with insufficient history — always wrap in try/except
