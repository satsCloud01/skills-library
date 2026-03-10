# Skills Library

Reusable skill definitions for AI agents. Each skill encodes best practices from production projects and industry standards.

## Categories

| Folder | Skills | Purpose |
|--------|--------|---------|
| `ui/` | React component, page, styling, accessibility, tour | Frontend development |
| `backend/` | FastAPI router, models, schemas, services, auth, seed | Backend development |
| `integrations/` | API client, Claude AI, S3/AWS, WebSocket, OAuth | External system integration |
| `infrastructure/` | Docker, Nginx, EC2, Route53, CI/CD | Infrastructure & DevOps |
| `testing/` | Unit, E2E, contract, performance, accessibility | All testing disciplines |
| `deployment/` | Docker build, EC2 deploy, S3 static, rollback | Deployment automation |
| `documentation/` | C4 architecture, API spec, runbook, domain model | Technical documentation |
| `github/` | Commit, PR, branch, release, repo setup | Git & GitHub workflows |

## Skill File Format

```yaml
---
name: skill-name
description: "What this skill does"
category: ui | backend | testing | ...
difficulty: beginner | intermediate | advanced
tags: [relevant, tags]
stack: [react, fastapi, docker, ...]
---

# Skill instructions (markdown)
```

## Based On

- 14 production apps on satszone.link
- OWASP, WCAG 2.1, 12-Factor App, C4 Model standards
- SQLAlchemy 2.0+, FastAPI, React 18, Vite 5, Tailwind 3.4
