---
name: nginx-room-key-gate
description: "Deploy Nginx auth_request room key gate to EC2 or GCP instances protecting satszone.link apps with cookie-based authentication"
category: infrastructure
difficulty: intermediate
tags: [nginx, auth, security, gate, room-key, ec2, gcp, deployment]
stack: [nginx, python-3.12, systemd, aws-ec2, gcp, route53]
---

# Nginx Room Key Gate Deployment

You are an infrastructure engineer deploying the SatsZone room key gate. This gate protects all apps behind a room key prompt using Nginx `auth_request` + a local Python auth server.

## Architecture

- **Auth server**: Python HTTP server on `127.0.0.1:9401` — validates `sz_room_key` cookie against gate API
- **Gate API**: `https://dr9yrgyzg2.execute-api.us-east-1.amazonaws.com/auth/validate`
- **Cookie**: `sz_room_key` — 24h expiry, SameSite=Lax, Secure
- **Cache**: Valid keys cached 5 min in-memory
- **Gate HTML**: Served from `/var/www/gate/gate-auth.html` (NEVER `/home/ubuntu/` — www-data can't access home dirs)

## Instances

### AWS EC2
| Instance | IP | SSH Key | Apps |
|----------|-----|---------|------|
| consolidated-apps | 3.82.56.250 | `~/.ssh/consolidated-apps-key.pem` | 15 apps (main) |
| Consolidated Apps 2 | 100.53.181.201 | `~/.ssh/sentinel-ai-key.pem` | sentinel-ai, agentkraft, cip, agentsubstrate |
| agentsubstrate (dedicated) | 3.88.105.213 | `~/.ssh/sentinel-ai-key.pem` | agentsubstrate |

### GCP
| Instance | IP | Zone | Apps |
|----------|-----|------|------|
| agentsubstrate-gcp | 34.73.223.187 | us-east1-d | agentsubstrate (GCP copy) |

## Step 1: Create Gate HTML File

Create `/tmp/gate-auth.html`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8"/>
<meta name="viewport" content="width=device-width,initial-scale=1.0"/>
<title>Access Required — SatsZone</title>
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700;800&display=swap" rel="stylesheet"/>
<style>
*{box-sizing:border-box;margin:0;padding:0}
body{font-family:Inter,system-ui,sans-serif;background:#f5f7fb;color:#152040;min-height:100vh;display:flex;align-items:center;justify-content:center}
.gate-card{background:#fff;border:1px solid rgba(30,45,82,.1);border-radius:20px;padding:2.8rem 2.5rem 2.4rem;width:100%;max-width:420px;text-align:center;box-shadow:0 24px 64px rgba(0,0,0,.08)}
.gate-lock{font-size:2.4rem;margin-bottom:1.1rem;display:inline-flex;align-items:center;justify-content:center;width:72px;height:72px;border-radius:18px;background:linear-gradient(135deg,rgba(30,58,110,.12),rgba(200,169,110,.1));border:1px solid rgba(30,58,110,.2)}
.gate-card h2{font-size:1.35rem;font-weight:800;margin-bottom:.45rem;background:linear-gradient(135deg,#152040,#1e3a6e);-webkit-background-clip:text;-webkit-text-fill-color:transparent;background-clip:text}
.gate-card p{color:#6b7490;font-size:.9rem;margin-bottom:1.6rem}
.gate-input-row{display:flex;gap:.5rem;margin-bottom:.75rem}
.gate-input-row input{flex:1;background:#edf0f7;border:1px solid rgba(30,45,82,.1);border-radius:10px;padding:.65rem 1rem;color:#152040;font-size:.95rem;font-family:'SF Mono','Fira Code',monospace;outline:none;transition:border-color .2s}
.gate-input-row input:focus{border-color:#1e3a6e}
.gate-input-row input::placeholder{color:#6b7490}
.gate-submit{background:linear-gradient(135deg,#152040,#1e3a6e);border:none;border-radius:10px;padding:.65rem 1.2rem;color:#fff;font-size:.9rem;font-weight:700;cursor:pointer;white-space:nowrap;transition:opacity .2s}
.gate-submit:hover{opacity:.88}
.gate-submit:disabled{opacity:.55;cursor:not-allowed}
.gate-error{color:#d44040;font-size:.82rem;min-height:1.2em;transition:opacity .2s}
.gate-footer{margin-top:1.4rem;color:#6b7490;font-size:.75rem}
.gate-back{display:inline-block;margin-top:1rem;color:#1e3a6e;text-decoration:none;font-size:.82rem;font-weight:600}
.gate-back:hover{text-decoration:underline}
</style>
</head>
<body>
<div class="gate-card">
  <div class="gate-lock">&#128274;</div>
  <h2>Access Required</h2>
  <p>This application is private. Enter your room key to continue.</p>
  <div class="gate-input-row">
    <input type="password" id="gateInput" placeholder="Enter room key…" autocomplete="off"
           onkeydown="if(event.key==='Enter')submitKey()"/>
    <button class="gate-submit" id="gateBtn" onclick="submitKey()">Unlock</button>
  </div>
  <div id="gateError" class="gate-error"></div>
  <a class="gate-back" href="https://my-solution-registry.satszone.link">← Back to Solution Registry</a>
  <div class="gate-footer">
    &#128274; Keys are time-limited &amp; rate-limited for security.<br/>
    By entering a key you acknowledge this is a monitored system.
  </div>
</div>
<script>
var API='https://dr9yrgyzg2.execute-api.us-east-1.amazonaws.com';
function submitKey(){
  var key=document.getElementById('gateInput').value.trim();
  var err=document.getElementById('gateError');
  if(!key){err.textContent='Please enter a key.';return;}
  document.getElementById('gateBtn').disabled=true;
  err.textContent='';
  fetch(API+'/auth/validate',{method:'POST',headers:{'Content-Type':'application/json'},body:JSON.stringify({key:key})})
  .then(function(r){return r.json()})
  .then(function(d){
    if(d.valid){
      var expires=new Date(Date.now()+24*60*60*1000).toUTCString();
      document.cookie='sz_room_key='+encodeURIComponent(key)+';path=/;expires='+expires+';SameSite=Lax;Secure';
      location.reload();
    } else {
      err.textContent=d.message||'Invalid key. Please try again.';
      document.getElementById('gateBtn').disabled=false;
    }
  })
  .catch(function(){
    err.textContent='Connection error. Please try again.';
    document.getElementById('gateBtn').disabled=false;
  });
}
</script>
</body>
</html>
```

## Step 2: Create Auth Server

Create `/tmp/gate-auth-server.py`:

```python
#!/usr/bin/env python3
"""Nginx auth_request validator. Checks sz_room_key cookie against gate API."""
import http.server, http.cookies, urllib.request, json, time, threading

GATE_API = "https://dr9yrgyzg2.execute-api.us-east-1.amazonaws.com/auth/validate"
_cache = {}
_lock = threading.Lock()
CACHE_TTL = 300

class AuthHandler(http.server.BaseHTTPRequestHandler):
    def do_GET(self):
        cookie_header = self.headers.get("Cookie", "")
        cookies = http.cookies.SimpleCookie()
        try: cookies.load(cookie_header)
        except: pass
        key = cookies.get("sz_room_key")
        if not key:
            self.send_response(401); self.end_headers(); return
        key_val = key.value
        with _lock:
            if key_val in _cache and _cache[key_val] > time.time():
                self.send_response(200); self.end_headers(); return
        try:
            req = urllib.request.Request(GATE_API, data=json.dumps({"key": key_val}).encode(),
                headers={"Content-Type": "application/json"}, method="POST")
            with urllib.request.urlopen(req, timeout=5) as resp:
                data = json.loads(resp.read())
            if data.get("valid"):
                with _lock: _cache[key_val] = time.time() + CACHE_TTL
                self.send_response(200)
            else: self.send_response(401)
        except: self.send_response(401)
        self.end_headers()
    def log_message(self, format, *args): pass

if __name__ == "__main__":
    server = http.server.HTTPServer(("127.0.0.1", 9401), AuthHandler)
    print("Auth server on 127.0.0.1:9401")
    server.serve_forever()
```

## Step 3: Create Systemd Service

Create `/tmp/gate-auth.service`:

```ini
[Unit]
Description=SatsZone Room Key Auth Server
After=network.target

[Service]
Type=simple
User=ubuntu
ExecStart=/usr/bin/python3 /home/ubuntu/gate/gate-auth-server.py
Restart=always
RestartSec=3

[Install]
WantedBy=multi-user.target
```

## Step 4: Create Nginx Snippet

Create `/tmp/gate-auth-snippet.conf`:

```nginx
# Room key gate - internal endpoints only
# IMPORTANT: Do NOT put auth_request or error_page here — those go in each server block
location = /_auth {
    internal;
    proxy_pass http://127.0.0.1:9401;
    proxy_pass_request_body off;
    proxy_set_header Content-Length "";
    proxy_set_header Cookie $http_cookie;
}

location = /_gate {
    internal;
    root /var/www/gate;
    try_files /gate-auth.html =404;
}
```

## Step 5: Deploy to Instance (EC2 or GCP)

For each instance:

```bash
# Upload files (EC2: use scp -i $KEY ubuntu@$IP, GCP: use gcloud compute scp)
scp -i $KEY /tmp/gate-auth.html /tmp/gate-auth-server.py /tmp/gate-auth-snippet.conf /tmp/gate-auth.service ubuntu@$IP:/tmp/

# Install on server
ssh -i $KEY ubuntu@$IP << 'REMOTE'
set -e

# Gate files
mkdir -p /home/ubuntu/gate
cp /tmp/gate-auth.html /home/ubuntu/gate/
cp /tmp/gate-auth-server.py /home/ubuntu/gate/
chmod +x /home/ubuntu/gate/gate-auth-server.py

# CRITICAL: Gate HTML must be in /var/www/gate/ with www-data ownership
# /home/ubuntu/ is NOT readable by nginx worker (www-data)
sudo mkdir -p /var/www/gate
sudo cp /home/ubuntu/gate/gate-auth.html /var/www/gate/
sudo chown -R www-data:www-data /var/www/gate

# Nginx snippet
sudo cp /tmp/gate-auth-snippet.conf /etc/nginx/snippets/gate-auth.conf

# Systemd service
sudo cp /tmp/gate-auth.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable gate-auth
sudo systemctl restart gate-auth
REMOTE
```

## Step 6: Update Nginx Server Blocks

### Pattern A — React/Static apps (serve dist + proxy /api):

```nginx
server {
    server_name APP.satszone.link;
    root /home/ubuntu/apps/APP/frontend/dist;
    index index.html;
    include /etc/nginx/snippets/gate-auth.conf;
    location / {
        auth_request /_auth;
        error_page 401 =200 /_gate;
        try_files $uri $uri/ /index.html;
    }
    location /api/ {
        proxy_pass http://127.0.0.1:PORT;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
    listen 443 ssl;
    ssl_certificate /etc/letsencrypt/live/APP.satszone.link/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/APP.satszone.link/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;
}
server { listen 80; server_name APP.satszone.link; return 301 https://$host$request_uri; }
```

### Pattern B — Proxy-only apps (Docker containers):

**CRITICAL: `error_page` goes at SERVER level, not inside location block.**

```nginx
server {
    server_name APP.satszone.link;
    include /etc/nginx/snippets/gate-auth.conf;
    error_page 401 =200 /_gate;
    location / {
        auth_request /_auth;
        proxy_pass http://127.0.0.1:PORT;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
    location /api/ {
        proxy_pass http://127.0.0.1:PORT;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
    listen 443 ssl;
    ssl_certificate /etc/letsencrypt/live/APP.satszone.link/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/APP.satszone.link/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;
}
server { listen 80; server_name APP.satszone.link; return 301 https://$host$request_uri; }
```

### Pattern C — Streamlit apps (add /_stcore bypass):

Same as Pattern B but add:

```nginx
    location /_stcore/ {
        proxy_pass http://127.0.0.1:PORT;
        proxy_set_header Host $host;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
```

### Pattern D — WebSocket apps (messenger):

Same as Pattern B but add to `location /`:

```nginx
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
```

### Pattern E — GCP / Non-EC2 Docker apps (with gate.js stripping + no-cache):

**Use this for ANY app deployed outside the EC2 consolidated instance (GCP, other clouds).**
Apps embed a client-side `gate.js` that creates a SECOND gate overlay. On EC2 this works because
gate.js and the Nginx gate share localStorage/cookies. On non-EC2 hosts, only the Nginx gate should
be active — gate.js must be stripped to avoid a duplicate broken overlay.

**CRITICAL differences from Pattern B:**
1. `sub_filter` strips the `gate.js` script URL to prevent the client-side gate overlay
2. `proxy_set_header Accept-Encoding ""` disables gzip so `sub_filter` can match text
3. `Cache-Control: no-store` prevents browser from caching gate page and serving stale 304s after auth passes
4. Must add the new subdomain to API Gateway CORS AllowOrigins (see GCP Checklist below)

```nginx
server {
    server_name APP.satszone.link;
    include /etc/nginx/snippets/gate-auth.conf;
    error_page 401 =200 /_gate;

    # CRITICAL: Prevent browser caching gate page vs app page (causes 304 stale responses)
    add_header Cache-Control "no-store, no-cache, must-revalidate" always;
    add_header Pragma "no-cache" always;
    expires 0;

    client_max_body_size 50M;

    location / {
        auth_request /_auth;
        proxy_pass http://127.0.0.1:PORT;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_read_timeout 300s;
        proxy_connect_timeout 60s;
        proxy_send_timeout 300s;

        # CRITICAL: Disable compression so sub_filter can match
        proxy_set_header Accept-Encoding "";

        # CRITICAL: Strip client-side gate.js — Nginx auth_request handles auth server-side
        # Without this, the app shows TWO gate overlays (Nginx + gate.js) and gate.js fails
        # because it uses localStorage sessions, not the sz_room_key cookie
        sub_filter_once on;
        sub_filter "my-solution-registry.satszone.link/gate.js" "about:blank";
    }

    location /api/ {
        proxy_pass http://127.0.0.1:PORT;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Allow caching for immutable static assets (JS/CSS bundles with hashed names)
    location /assets/ {
        auth_request /_auth;
        proxy_pass http://127.0.0.1:PORT;
        proxy_set_header Host $host;
        expires 7d;
        add_header Cache-Control "public, immutable";
    }

    listen 443 ssl;
    ssl_certificate /etc/letsencrypt/live/APP.satszone.link/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/APP.satszone.link/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;
}
server {
    if ($host = APP.satszone.link) { return 301 https://$host$request_uri; }
    listen 80; server_name APP.satszone.link; return 404;
}
```

## Step 7: Test and Reload

```bash
# Always backup first
sudo cp /etc/nginx/sites-enabled/apps /etc/nginx/sites-enabled/apps.bak

# Test config
sudo nginx -t

# Reload (NOT restart — reload is zero-downtime)
sudo systemctl reload nginx

# Clean up backups after confirming
sudo rm -f /etc/nginx/sites-enabled/*.bak
```

## Step 8: Verify

```bash
# Check single app
curl -s --max-time 8 "https://APP.satszone.link" | grep -c "Access Required"
# Should return > 0

# Check all apps at once
for name in dataanalyzer data-guardian semantic-modeler satshub sentinel-ai ai-guardian agentfier agent-control agentsubstrate agentkraft cip prediction archsmith cloud-deployer policyguard cortex legal messenger sats-recruitment agentsubstrate-gcp; do
  has_gate=$(curl -s --max-time 8 "https://$name.satszone.link" | grep -c 'Access Required' 2>/dev/null)
  [ "$has_gate" -gt 0 ] && echo "OK   $name" || echo "MISS $name"
done
```

## GCP Deployment Checklist

When deploying a new app to GCP (or any non-EC2 host), follow these EXTRA steps on top of the standard gate deployment:

### 1. Add subdomain to API Gateway CORS AllowOrigins

The Room Key Lambda API (API Gateway `dr9yrgyzg2`) has an explicit CORS AllowOrigins list.
**If the new subdomain is not in this list, the browser gate.js/gate-auth.html will get "Connection error"** because the CORS preflight fails.

```bash
# Get current origins
aws apigatewayv2 get-api --api-id dr9yrgyzg2 --query "CorsConfiguration.AllowOrigins" --output json

# Add the new origin — include ALL existing origins plus the new one
aws apigatewayv2 update-api --api-id dr9yrgyzg2 --cors-configuration '{
  "AllowHeaders": ["content-type", "x-admin-key"],
  "AllowMethods": ["*"],
  "AllowOrigins": [
    ... existing origins ...,
    "https://NEWAPP.satszone.link"
  ],
  "MaxAge": 3600
}'
```

### 2. Use Nginx Pattern E (not Pattern B)

Pattern E includes three critical fixes for non-EC2 hosts:
- **`sub_filter`**: Strips `gate.js` URL → prevents duplicate client-side gate overlay
- **`Accept-Encoding ""`**: Disables gzip so `sub_filter` can pattern-match the HTML
- **`Cache-Control: no-store`**: Prevents browser from caching gate HTML and showing stale 304 after auth passes

### 3. Add swap on GCP e2-micro

```bash
sudo fallocate -l 2G /swapfile && sudo chmod 600 /swapfile
sudo mkswap /swapfile && sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

### 4. Reserve a static IP

GCP ephemeral IPs change on reboot. Reserve a static IP:
```bash
gcloud compute addresses create APP-ip --region=REGION
```

## New App Checklist (Any Cloud)

When adding a new app:

1. **DNS**: Create A record in Route53 (zone Z047244933J6LNZKU0UTJ) pointing to the instance IP
2. **CORS**: Add `https://NEWAPP.satszone.link` to API Gateway CORS AllowOrigins (API ID: `dr9yrgyzg2`)
3. **SSL**: `sudo certbot --nginx -d NEWAPP.satszone.link --non-interactive --agree-tos -m admin@satszone.link`
4. **Nginx config**: Use Pattern A-D (EC2) or **Pattern E** (GCP/non-EC2)
5. **Test**: `sudo nginx -t && sudo systemctl reload nginx`
6. **Verify**: `curl -s "https://NEWAPP.satszone.link" | grep -c "Access Required"`

## Gotchas

1. **www-data permissions**: Gate HTML in `/home/ubuntu/` returns 404. MUST be in `/var/www/gate/` with www-data ownership.
2. **error_page placement**: For `proxy_pass` locations → server level. For `try_files` locations → can be in location block.
3. **No duplicate directives**: `auth_request` and `error_page` go in server blocks only, NOT in the snippet.
4. **DNS verification**: Always `dig +short APP.satszone.link` and compare with actual instance IP.
5. **SSL certs needed before HTTPS**: Run certbot AFTER DNS points to the right IP.
6. **/api/ routes stay unprotected**: Intentional — SPA backends need to work after cookie unlock.
7. **Auth server must be running**: `sudo systemctl status gate-auth` — it caches valid keys for 5 min.
8. **Cookie domain**: The `sz_room_key` cookie is per-subdomain. Unlocking one app doesn't unlock others.
9. **CORS on non-EC2 (GCP etc.)**: New subdomains MUST be added to API Gateway CORS AllowOrigins or gate.js/gate-auth.html will show "Connection error" / "Network error".
10. **Browser caching on non-EC2**: Use `Cache-Control: no-store` (Pattern E) to prevent 304 stale responses after auth state changes.
11. **Duplicate gate overlay on non-EC2**: Apps embed `gate.js` (client-side dark-themed gate) which conflicts with Nginx gate. Use `sub_filter` (Pattern E) to strip it. Without `Accept-Encoding ""`, sub_filter won't work on gzipped responses.
12. **GCP zone exhaustion**: `e2-micro` free tier is often exhausted in popular zones. Try multiple zones: `us-east1-b`, `us-east1-c`, `us-east1-d`.
