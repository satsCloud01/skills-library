---
name: deploy-s3-static
description: "Deploy static HTML/CSS/JS to S3 with CloudFront CDN, cache invalidation, and custom domain"
category: deployment
difficulty: beginner
tags: [s3, cloudfront, static-site, cdn, deployment]
stack: [aws-s3, cloudfront, route53]
---

# S3 Static Site Deployment

You are a static site deployment specialist.

## Deploy Command

```bash
# Upload to S3
aws s3 cp file.html s3://bucket-name/index.html --content-type "text/html"

# Invalidate CloudFront cache
aws cloudfront create-invalidation --distribution-id EXXXXXXXX --paths "/*"
```

## Full Site Deploy (directory)

```bash
# Sync entire directory
aws s3 sync ./dist/ s3://bucket-name/ \
  --delete \
  --cache-control "public, max-age=31536000" \
  --exclude "index.html"

# Upload index.html with short cache
aws s3 cp ./dist/index.html s3://bucket-name/index.html \
  --content-type "text/html" \
  --cache-control "public, max-age=300"

# Invalidate
aws cloudfront create-invalidation --distribution-id EXXXXXXXX --paths "/*"
```

## S3 Bucket Config

- Static website hosting: enabled
- Index document: `index.html`
- Error document: `index.html` (for SPA routing)
- Bucket policy: CloudFront OAI access only (not public)

## CloudFront Settings

- Origin: S3 bucket (use OAI, not public URL)
- Default root object: `index.html`
- Custom error pages: 403/404 → `/index.html` (SPA)
- SSL: ACM certificate (us-east-1 for CloudFront)
- Price class: Use Only NA and Europe (cheapest)

## Rules
- Always invalidate after deploy — CloudFront caches aggressively
- Set short cache on `index.html` (5 min), long cache on assets (1 year)
- Use `--delete` with sync to remove old files
- Verify with `curl -I https://domain.link` after invalidation
- Cost: ~$0.50/month for low-traffic static sites
