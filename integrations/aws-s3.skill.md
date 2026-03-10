---
name: aws-s3
description: "S3 integration for file storage, static hosting, presigned URLs, and lifecycle policies"
category: integrations
difficulty: intermediate
tags: [aws, s3, storage, presigned-url, static-hosting]
stack: [python-3.12, boto3, aws-s3]
---

# AWS S3 Integration

You are an AWS storage specialist.

## boto3 Client Pattern

```python
import boto3
from botocore.exceptions import ClientError

s3 = boto3.client('s3', region_name='us-east-1')

# Upload file
def upload_file(file_path: str, bucket: str, key: str, content_type: str = None):
    extra = {}
    if content_type:
        extra['ContentType'] = content_type
    s3.upload_file(file_path, bucket, key, ExtraArgs=extra)

# Upload bytes
def upload_bytes(data: bytes, bucket: str, key: str, content_type: str):
    s3.put_object(Bucket=bucket, Key=key, Body=data, ContentType=content_type)

# Download
def download_file(bucket: str, key: str, local_path: str):
    s3.download_file(bucket, key, local_path)

# Presigned URL (temporary access)
def get_presigned_url(bucket: str, key: str, expires_in: int = 3600) -> str:
    return s3.generate_presigned_url('get_object',
        Params={'Bucket': bucket, 'Key': key}, ExpiresIn=expires_in)

# List objects
def list_objects(bucket: str, prefix: str = '') -> list[dict]:
    resp = s3.list_objects_v2(Bucket=bucket, Prefix=prefix)
    return resp.get('Contents', [])

# Delete
def delete_object(bucket: str, key: str):
    s3.delete_object(Bucket=bucket, Key=key)
```

## Static Website Hosting

```python
# Enable static hosting
s3.put_bucket_website(
    Bucket=bucket,
    WebsiteConfiguration={
        'IndexDocument': {'Suffix': 'index.html'},
        'ErrorDocument': {'Key': 'index.html'},
    }
)
```

## Rules
- Use IAM roles on EC2, not access keys
- Set `ContentType` explicitly — S3 doesn't always guess correctly
- Presigned URLs for temporary access — default 1 hour
- Use `--no-cache-dir` in uploads for frequently updated files
- Enable versioning for important buckets
- Lifecycle rules: move to Glacier after 90 days for archival
- Never make buckets public — use CloudFront OAI
