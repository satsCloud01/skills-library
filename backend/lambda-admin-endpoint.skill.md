---
name: Lambda Admin API Endpoint
description: Add new admin-authenticated endpoints to an AWS Lambda handler with boto3 service integration
category: backend
tags: [lambda, aws, admin-api, boto3, api-gateway]
---

# Lambda Admin API Endpoint

Add new admin-authenticated API endpoints to an existing AWS Lambda handler that uses API Gateway HTTP API with a `$default` catch-all route.

## Pattern

The handler uses path-based routing inside a single Lambda function. Admin endpoints are protected by an `X-Admin-Key` header validated against DynamoDB.

### Structure

```python
# 1. Add boto3 client at module level (cold-start friendly)
new_service = boto3.client('service_name', region_name='us-east-1')

# 2. Add route inside the admin-authenticated block
# GET /admin/{resource}
if method == 'GET' and path == '/admin/{resource}':
    try:
        # Call AWS service
        resp = new_service.describe_things()
        items = []
        for item in resp.get('things', []):
            items.append({
                'id': item['id'],
                'name': item.get('name', ''),
                # ... map fields
            })
        return make_response(200, {'items': items})
    except Exception as e:
        return make_response(500, {'error': str(e)})

# POST /admin/{resource}/{id}/{action} (for mutations)
if method == 'POST' and path.startswith('/admin/{resource}/'):
    parts = path.split('/')
    if len(parts) == 5:
        resource_id = parts[3]
        action = parts[4]
        try:
            if action == 'start':
                new_service.start_thing(ThingIds=[resource_id])
            elif action == 'stop':
                new_service.stop_thing(ThingIds=[resource_id])
            else:
                return make_response(400, {'error': 'Invalid action: ' + action})
            return make_response(200, {'ok': True, 'id': resource_id, 'action': action})
        except Exception as e:
            return make_response(500, {'error': str(e)})
```

### IAM Permissions

Always add the minimum required IAM permissions as an inline policy on the Lambda execution role:

```bash
aws iam put-role-policy --role-name {LambdaRoleName} \
  --policy-name {ServiceName}Access \
  --policy-document '{
    "Version": "2012-10-17",
    "Statement": [{
      "Effect": "Allow",
      "Action": ["service:DescribeThings", "service:StartThing", "service:StopThing"],
      "Resource": "*"
    }]
  }'
```

### Deployment

```bash
# Package and deploy
cd lambda/ && zip -j /tmp/handler.zip handler.py
aws lambda update-function-code --function-name {FunctionName} \
  --zip-file fileb:///tmp/handler.zip --region us-east-1

# Increase timeout if new service calls are slow
aws lambda update-function-configuration --function-name {FunctionName} --timeout 30
```

### Checklist

- [ ] boto3 client created at module level
- [ ] Route added inside admin auth guard block
- [ ] IAM inline policy added to Lambda role
- [ ] Lambda timeout increased if needed
- [ ] Route comment updated in handler docstring
- [ ] API Gateway uses `$default` route (no route changes needed) OR new routes added
- [ ] Error handling returns `make_response(500, {'error': str(e)})`
