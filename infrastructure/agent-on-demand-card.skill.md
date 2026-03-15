---
name: agent-on-demand-card
description: "Create on-demand agent cards with real AWS deploy, progress animations, health checks, kill switch, and auto-destroy TTL"
category: infrastructure
difficulty: advanced
tags: [agent-card, on-demand-deploy, ec2-launch, auto-destroy, deploy-modal]
stack: [aws-lambda, api-gateway, ecr, ec2, javascript]
---

# Agent On-Demand Card

You are an infrastructure engineer building on-demand deployable agent cards for a solution registry. Each agent card allows users to deploy a full application to a fresh EC2 instance with one click, monitor progress, open the live app, and kill it when done.

## Architecture

```
Registry HTML → API Gateway (HTTP API)
                  ├── POST /deploy     → Lambda (agent-launcher) → EC2 t3.small
                  ├── GET /status/{id} → Lambda (agent-status)   → describe instance
                  └── POST /deploy (_kill) → Lambda (terminate)

EventBridge (every 5 min) → Lambda (agent-cleanup) → terminates expired instances
```

## Agent Card HTML Structure

Each deployable agent card must include:

```html
<div class="app-card" style="position:relative">
  <div class="card-top">
    <div class="app-icon" style="background:linear-gradient(135deg,#6366f1,#8b5cf6)">EMOJI</div>
    <div class="card-title">
      <h3>Agent Name</h3>
      <div class="domain">Short Description</div>
    </div>
    <span class="badge-ondemand">&#9889; On-Demand</span>
  </div>
  <div class="card-body">
    <p>Full description...</p>
    <div><strong>Capabilities:</strong>
      <ul style="list-style:none;padding:0;display:grid;grid-template-columns:1fr 1fr;gap:.25rem">
        <li>&#10004; Capability 1</li>
        <li>&#10004; Capability 2</li>
      </ul>
    </div>
    <div class="tags">
      <span class="tag t-py">Python 3.12</span>
      <span class="tag t-react">React 18</span>
      <!-- etc -->
    </div>
  </div>
  <!-- Status bar ABOVE footer -->
  <div class="card-deploy-status" id="AGENT_ID_Status"></div>
  <div class="card-footer" id="AGENT_ID_Footer">
    <button class="btn-deploy" id="AGENT_ID_DeployBtn" onclick="deployAgent('AGENT_ID')">&#9889; Deploy Now</button>
    <span id="AGENT_ID_KillWrap" style="display:none">
      <button class="btn-kill" onclick="killInstance(event)">&#10006; Kill</button>
    </span>
    <a class="btn btn-ghost" href="GITHUB_URL" target="_blank">GitHub</a>
    <button class="btn btn-stack" onclick="toggleStack(this)">Tech Stack</button>
  </div>
</div>
```

## Required CSS Classes

```css
/* Card status bar */
.card-deploy-status { display:none; margin-top:.5rem; padding:.5rem .8rem; border-radius:8px; font-size:.78rem; text-align:center; }
.card-deploy-status.deploying { display:block; background:rgba(99,102,241,.1); border:1px solid rgba(99,102,241,.2); color:var(--accent); }
.card-deploy-status.live { display:block; background:rgba(34,197,94,.1); border:1px solid rgba(34,197,94,.25); color:#22c55e; }

/* Kill button */
.btn-kill { display:inline-flex; align-items:center; gap:.3rem; padding:.3rem .7rem; border-radius:6px; font-size:.72rem; font-weight:600; border:1px solid rgba(239,68,68,.3); background:rgba(239,68,68,.08); color:#ef4444; cursor:pointer; }

/* On-demand badge */
.badge-ondemand { font-size:.65rem; padding:.2rem .5rem; border-radius:5px; background:rgba(6,182,212,.12); color:#22d3ee; font-weight:600; }

/* Deploy modal animations */
.deploy-fun-emoji { font-size:2.5rem; display:inline-block; animation: bounce-emoji 1.4s ease-in-out infinite; }
.deploy-spinner { display:inline-block; width:18px; height:18px; border:2px solid var(--border); border-top-color:var(--accent); border-radius:50%; animation: spin-slow .8s linear infinite; }
```

## Deploy Modal

The deploy modal shows real-time progress with 4 steps and fun animations:

```html
<div id="deployOverlay" class="deploy-overlay">
  <div class="deploy-modal">
    <h3 id="deployTitle">Deploying Agent...</h3>
    <div id="deploySteps">
      <div class="deploy-step active" id="ds1"><span class="step-icon">1</span> Provisioning EC2 instance</div>
      <div class="deploy-step" id="ds2"><span class="step-icon">2</span> Pulling Docker image from ECR</div>
      <div class="deploy-step" id="ds3"><span class="step-icon">3</span> Starting application</div>
      <div class="deploy-step" id="ds4"><span class="step-icon">4</span> Health check &amp; ready!</div>
    </div>
    <div id="deployFun" class="deploy-fun">
      <div class="deploy-fun-emoji" id="funEmoji">&#128640;</div>
      <div class="deploy-fun-text" id="funText">Warming up the engines...</div>
    </div>
    <div id="deployTimer"><span class="deploy-spinner"></span> Estimated: ~2 minutes</div>
    <div id="deployResult" style="display:none">
      <a id="deployUrl" class="deploy-open-btn disabled" href="#">Waiting for app...</a>
      <div>Auto-destroys in <span id="deployTTL">30:00</span></div>
    </div>
    <button onclick="closeDeployModal()">Close</button>
  </div>
</div>
```

## JavaScript Deploy Flow

### Key Rules
1. **One deploy at a time** — block if `_activeInstanceId` is set, show alert to kill first
2. **Background polling** — closing modal does NOT stop the deploy, polling continues
3. **Card updates live** — status bar and buttons update on the card itself
4. **Health check via status API** — do NOT use iframe/fetch to app URL (HTTPS→HTTP mixed content fails)
5. **Fallback readiness** — after EC2 running 90s+ without tag, assume ready
6. **Kill is safe** — only terminates instances tagged `ManagedBy=agent-launcher`
7. **Speech on deploy** — use Web Speech API for "Let me fire the engines up!" message

### Deploy Function Pattern

```javascript
var DEPLOY_API = 'https://YOUR_API_GATEWAY.execute-api.REGION.amazonaws.com/prod';

function deployAgent(agentId) {
  if (_activeInstanceId) {
    alert('Kill current instance first, then try again.');
    return;
  }
  // 1. Open modal, start fun animations, speak message
  // 2. POST /deploy with {agent_id}
  // 3. Poll GET /status/{instance_id} every 5s
  // 4. When EC2 state=running → mark step 2 done, update card to 'container-starting'
  // 5. When AgentStatus tag=running OR 90s elapsed → showDeployReady()
  // 6. showDeployReady: enable green button on modal + card, start TTL countdown
}
```

### Card State Machine

| State | Deploy Button | Status Bar | Kill |
|-------|--------------|------------|------|
| idle | "Deploy Now" (enabled) | hidden | hidden |
| deploying | "Deploying..." (disabled) | spinner + "Launching..." | visible |
| container-starting | "Starting..." (disabled) | spinner + IP | visible |
| live | "Open App" (green, opens URL) | green dot + URL + TTL | visible |
| killed/reset | "Deploy Now" (enabled) | hidden | hidden |
| error | "Deploy Now" (enabled) | warning + "Try again" | hidden |

### Fun Animation Messages (cycle every 3.5s)

```javascript
var FUN_STAGES = [
  { emoji: '🚀', text: 'Warming up the engines...' },
  { emoji: '📦', text: 'Unpacking containers like a pro...' },
  { emoji: '☕', text: 'Brewing a fresh instance...' },
  { emoji: '🧱', text: 'Stacking the layers...' },
  { emoji: '🎨', text: 'Painting the pixels...' },
  { emoji: '🔧', text: 'Tightening the bolts...' },
  { emoji: '🤖', text: 'Teaching the agent some tricks...' },
  { emoji: '🌟', text: 'Almost there, hang tight...' },
  { emoji: '🎬', text: 'Rehearsing the final scene...' },
  { emoji: '💡', text: 'Connecting the neurons...' },
  { emoji: '🚚', text: 'Delivering your app fresh...' },
  { emoji: '🎉', text: 'The confetti is almost ready...' },
];
```

## Lambda Functions

### agent-launcher (POST /deploy)
- Receives `{agent_id}` or `{agent_id: '_kill', instance_id}`
- For deploy: looks up agent config (ECR image, port), launches t3.small with UserData
- UserData: installs Docker, pulls ECR image, runs container on port 80, self-tags `AgentStatus=running`
- Tags instance with: `Name`, `AgentId`, `AgentStatus=provisioning`, `TTL=now+1800`, `ManagedBy=agent-launcher`
- For kill: verifies `ManagedBy=agent-launcher` tag, then terminates
- Returns `{instance_id, public_ip, url, ttl_seconds}`

### agent-status (GET /status/{instance_id})
- Describes EC2 instance, returns state, public_ip, agent_status tag, TTL tag

### agent-cleanup (EventBridge every 5 min)
- Finds instances where `tag:ManagedBy=agent-launcher` AND `tag:TTL < now`
- Terminates only those — other instances are NEVER touched

## AWS Infrastructure Requirements

| Component | Details |
|-----------|---------|
| Security Group | Inbound: 80, 443, 22. Name: `sats-agent-sg` |
| IAM: Lambda role | ec2:RunInstances, DescribeInstances, TerminateInstances, CreateTags, iam:PassRole |
| IAM: EC2 role | ecr:GetAuthorizationToken, BatchGetImage, GetDownloadUrlForLayer, ec2:CreateTags |
| EC2 Instance Profile | `agent-ec2-profile` with EC2 role attached |
| ECR Repository | One per agent, e.g. `satszone/data-profiler` |
| API Gateway | HTTP API with CORS, two routes, auto-deploy stage |
| EventBridge | Rate(5 minutes) → agent-cleanup Lambda |
| Key Pair | For SSH access if debugging needed |

## Docker Image Requirements

- Multi-stage build: Node (frontend) + Python (backend)
- Must have `.dockerignore` excluding `frontend/node_modules` (broken symlinks on Linux)
- Frontend served via FastAPI `StaticFiles(directory="static", html=True)` mounted at `/`
- Health endpoint at `/api/health` (NOT `/` — that serves the SPA)
- Single port exposed (e.g. 8008), mapped to port 80 on EC2

## Security Rules

1. Kill endpoint verifies `ManagedBy=agent-launcher` tag before terminating
2. Cleanup Lambda ONLY targets `ManagedBy=agent-launcher` tagged instances
3. One deploy per user session — must kill before deploying another
4. TTL auto-destroys after 30 minutes
5. API keys (BYOK) stored in browser localStorage only, never server-side

## MANDATORY: Update Solution Registry Tour

After adding any new on-demand agent card, you **must** also add a matching tour step to the `TOUR_STEPS` array in `deployed-apps.html`. The tour must always reflect the complete portfolio.

1. Open `/Users/Sats/Documents/TechnicalPlayGround/CodexFolder/deployed-apps.html`
2. Find `var TOUR_STEPS = [` and add a step in the correct category section:
   ```javascript
   {
     icon: 'EMOJI', title: 'Agent Name',
     subtitle: 'Category · Short tagline',
     gradient: 'linear-gradient(135deg,#COLOR1,#COLOR2)', url: null,
     desc: 'Description...',
     features: ['Feature 1', 'Feature 2', 'Feature 3', 'On-demand deploy: click Deploy Now to spin up a live instance for 30 minutes']
   },
   ```
3. Update the **Welcome step** (first) and **Finale step** (last) with new counts
4. Copy to solution-registry repo, push to GitHub, deploy to S3, invalidate CloudFront:
   ```bash
   cp deployed-apps.html /Users/Sats/Documents/TechnicalPlayGround/CodexFolder/solution-registry/index.html
   cd /Users/Sats/Documents/TechnicalPlayGround/CodexFolder/solution-registry
   git add index.html && git commit -m "feat: add AGENT_NAME to tour" && git push origin main
   aws s3 cp /Users/Sats/Documents/TechnicalPlayGround/CodexFolder/deployed-apps.html s3://my-solution-registry.satszone.link/index.html --content-type "text/html"
   aws cloudfront create-invalidation --distribution-id E2R00426B8QGNB --paths "/*"
   ```
