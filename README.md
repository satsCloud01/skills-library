# Skills Library

Reusable skill definitions for AI agents. Each skill encodes best practices from production projects and industry standards.

**105 skills** across 12 categories.

## Categories

| Folder | Skills | Purpose |
|--------|--------|---------|
| `ui/` | 24 | Frontend development — React components, pages, styling, accessibility, tours, video players, design patterns |
| `backend/` | 26 | Backend development — FastAPI, routers, models, agents, pipelines, scheduling, human review |
| `testing/` | 11 | All testing — unit, E2E, contract, performance, QA, visual QA, security audit |
| `planning/` | 5 | Planning & review — CEO review, design review, eng review, auto-review pipeline, office hours |
| `deployment/` | 7 | Deployment — EC2, S3, GCP, canary monitoring, deploy config, land-and-deploy |
| `infrastructure/` | 8 | Infrastructure & DevOps — Docker, Nginx, EC2, ECR, on-demand deploy |
| `integrations/` | 6 | External systems — API clients, S3, WebSocket, OAuth, second opinion |
| `github/` | 6 | Git & GitHub — commits, PRs, repo setup, shipping, retrospectives |
| `documentation/` | 4 | Technical docs — C4 architecture, API spec, runbook, release docs |
| `safety/` | 4 | Safety guardrails — careful mode, freeze/unfreeze scope, guard mode |
| `browser/` | 3 | Browser automation — headless Chromium, Chrome connect, cookie import |
| `debugging/` | 1 | Debugging — root cause investigation with 4-phase methodology |

## Skill File Format

```yaml
---
name: skill-name
description: "What this skill does"
category: ui | backend | testing | ...
tags: [relevant, tags]
---

# Skill instructions (markdown)
```

## New Skills (from gstack)

The following 27 skills were extracted from the [gstack](https://github.com/satsCloud01/gstack) framework, cleaned of framework-specific references, and reorganized:

| Skill | Category | Description |
|-------|----------|-------------|
| `pr-review` | github | Pre-landing PR review — SQL safety, LLM trust boundaries, side effects |
| `ship-workflow` | github | End-to-end ship: tests → review → version bump → changelog → PR |
| `eng-retro` | github | Weekly engineering retrospective with trend tracking |
| `qa-test-fix` | testing | Full QA test-fix-verify loop (Quick/Standard/Exhaustive) |
| `qa-report` | testing | Report-only QA — structured bug reports, never modifies code |
| `performance-benchmark` | testing | Performance regression detection (Core Web Vitals, bundles) |
| `visual-qa` | testing | Designer's eye QA — spacing, hierarchy, visual consistency |
| `security-audit` | testing | CSO-mode security audit — OWASP, STRIDE, supply chain |
| `canary-monitor` | deployment | Post-deploy canary monitoring with severity escalation |
| `land-and-deploy` | deployment | Merge PR → wait CI → verify production health |
| `deploy-config` | deployment | Configure deploy settings (Fly, Render, Vercel, Netlify, etc.) |
| `release-docs` | documentation | Post-ship docs update — README, ARCHITECTURE, CHANGELOG |
| `auto-review-pipeline` | planning | Auto-run CEO + design + eng reviews sequentially |
| `ceo-review` | planning | CEO/founder-mode plan review — 10-star product thinking |
| `design-plan-review` | planning | Designer's eye plan review (pre-implementation) |
| `eng-review` | planning | Eng manager-mode plan review — architecture, edge cases |
| `office-hours` | planning | YC Office Hours — startup and builder brainstorming modes |
| `design-system` | ui | Design consultation — full design system proposal |
| `careful-mode` | safety | Warns before destructive commands (rm -rf, DROP, force-push) |
| `freeze-scope` | safety | Restrict file edits to a specific directory |
| `unfreeze-scope` | safety | Clear the freeze boundary |
| `guard-mode` | safety | Full safety: careful + freeze combined |
| `root-cause-debug` | debugging | Systematic 4-phase debugging with root cause enforcement |
| `headless-browser` | browser | Fast headless Chromium (~100ms/cmd) for QA and dogfooding |
| `chrome-connect` | browser | Launch visible Chrome with live activity side panel |
| `browser-cookies` | browser | Import browser cookies for authenticated QA testing |
| `second-opinion` | integrations | Independent code review via OpenAI Codex (review/challenge/consult) |

## Based On

- 15 production apps on satszone.link
- [gstack](https://github.com/satsCloud01/gstack) builder framework
- OWASP, WCAG 2.1, 12-Factor App, C4 Model standards
- SQLAlchemy 2.0+, FastAPI, React 18, Vite 5, Tailwind 3.4
