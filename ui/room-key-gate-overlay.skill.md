---
name: room-key-gate-overlay
description: "Client-side room key gate overlay with session management, admin panel, identity capture, and request workflow — vanilla JS for embedding in any HTML page"
category: ui
difficulty: intermediate
tags: [gate, overlay, access-control, admin-panel, session, vanilla-js, room-key]
stack: [javascript, html, css]
---

# Room Key Gate Overlay — Client-Side Access Control

You are an expert frontend engineer building a client-side gate overlay that protects web pages behind room keys. The gate communicates with a serverless API and provides an admin panel for key management.

## Reference Implementation

Canonical implementation: `/room-key-api/client/` (GitHub: satsCloud01/room-key-api)
Production: embedded in `my-solution-registry.satszone.link`

## Architecture

```
HTML Page
  ├── gate.css          — dark-theme styles
  ├── snippet.html      — overlay + admin modal markup
  └── gate.js           — session management + admin panel logic
        │
        ▼
  API (Lambda)
    POST /auth/validate
    POST /auth/request
    POST /auth/identity
    GET  /admin/*
```

## Session Management Pattern

```javascript
var API_ENDPOINT = 'https://your-api.execute-api.region.amazonaws.com/prod';
var SESSION_KEY = 'srk_session';
var SESSION_TTL_MS = 15 * 60 * 1000; // 15 minutes

var _adminKey = null;   // in-memory only — NEVER persisted
var _sessionKey = null;
var _expiryTimer = null;

function saveSession(isAdmin, keyValue) {
  var payload = {
    expires: Date.now() + SESSION_TTL_MS,
    is_admin: !!isAdmin,
    key_value: keyValue || '',
  };
  try { localStorage.setItem(SESSION_KEY, JSON.stringify(payload)); } catch (e) {}
}

function loadSession() {
  try {
    var raw = localStorage.getItem(SESSION_KEY);
    if (!raw) return null;
    var s = JSON.parse(raw);
    if (Date.now() > s.expires) {
      localStorage.removeItem(SESSION_KEY);
      return null;
    }
    return s;
  } catch (e) { return null; }
}

function startExpiryTimer(msRemaining) {
  if (_expiryTimer) clearTimeout(_expiryTimer);
  _expiryTimer = setTimeout(lockPage, msRemaining);
}
```

## Gate Unlock Flow

```javascript
function submitGateKey() {
  var input = document.getElementById('gateInput');
  var key = input ? input.value.trim() : '';
  if (!key) { showError('Please enter a room key.'); return; }

  fetch(API_ENDPOINT + '/auth/validate', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ key: key }),
  })
    .then(function(r) { return r.json(); })
    .then(function(data) {
      if (data.valid) {
        unlockPage(data.is_admin, key, data);
      } else if (data.expired) {
        showError('Key expired. <a href="#" onclick="showRequestView();return false;">Request a new one</a>.');
      } else if (data.rate_limited) {
        showError('Daily limit reached. Resets at midnight UTC.');
      } else {
        showError('Invalid or revoked key.');
      }
    })
    .catch(function() { showError('Network error. Please try again.'); });
}

function unlockPage(isAdmin, keyValue, apiData) {
  _sessionKey = keyValue;
  saveSession(isAdmin, keyValue);

  // Fade out overlay
  var overlay = document.getElementById('gateOverlay');
  if (overlay) {
    overlay.classList.add('gate-hidden');
    setTimeout(function() { overlay.style.display = 'none'; }, 380);
  }

  // Show admin FAB for admin keys
  if (isAdmin) {
    _adminKey = keyValue;
    var fab = document.getElementById('adminFab');
    if (fab) fab.style.display = 'inline-flex';
  }

  startExpiryTimer(SESSION_TTL_MS);

  // Expiry warning banner
  if (apiData && apiData.expiry_warning) {
    showExpiryWarning(apiData.hours_until_expiry);
  }

  // Identity capture (non-admin, once per session)
  if (!isAdmin && !sessionStorage.getItem('srk_id_shown')) {
    sessionStorage.setItem('srk_id_shown', '1');
    setTimeout(function() { showIdentityBanner(); }, 2000);
  }
}

function lockPage() {
  _adminKey = null;
  _sessionKey = null;
  localStorage.removeItem(SESSION_KEY);
  // Show overlay, reset form
  var overlay = document.getElementById('gateOverlay');
  if (overlay) {
    overlay.style.display = 'flex';
    requestAnimationFrame(function() { overlay.classList.remove('gate-hidden'); });
  }
}
```

## Auto-Restore on Page Load

```javascript
(function checkExistingSession() {
  var session = loadSession();
  if (session) {
    _sessionKey = session.key_value || null;
    var msRemaining = Math.max(0, session.expires - Date.now());
    var overlay = document.getElementById('gateOverlay');
    if (overlay) {
      overlay.classList.add('gate-hidden');
      setTimeout(function() { overlay.style.display = 'none'; }, 380);
    }
    if (session.is_admin) {
      var fab = document.getElementById('adminFab');
      if (fab) fab.style.display = 'inline-flex';
      // _adminKey NOT restored — admin must re-enter after refresh
    }
    startExpiryTimer(msRemaining);
  }
})();
```

## Admin Panel — Tabbed Modal

```javascript
var _activeTab = 'keys';

function switchTab(tab) {
  _activeTab = tab;
  var tabs = ['keys', 'requests', 'log', 'identities', 'costs', 'ecr', 'ec2'];
  tabs.forEach(function(t) {
    var btn = document.getElementById('tab-' + t);
    var panel = document.getElementById('panel-' + t);
    if (btn) btn.classList.toggle('active', t === tab);
    if (panel) panel.style.display = (t === tab) ? '' : 'none';
  });
  // Load data for active tab
  var loaders = {
    keys: loadKeys, requests: loadRequests,
    log: loadAccessLog, identities: loadIdentities,
    costs: loadCosts, ecr: loadECR, ec2: loadEC2,
  };
  if (loaders[tab]) loaders[tab]();
}

function adminFetch(method, path, body) {
  var opts = {
    method: method,
    headers: { 'Content-Type': 'application/json', 'X-Admin-Key': _adminKey },
  };
  if (body) opts.body = JSON.stringify(body);
  return fetch(API_ENDPOINT + path, opts).then(function(r) {
    if (r.status === 401) throw new Error('Unauthorized — admin key rejected.');
    return r.json();
  });
}
```

## Request Access Flow

```javascript
var REQUEST_KEY = 'srk_request_pending';

function submitRequest() {
  var name = document.getElementById('reqName').value.trim();
  var email = document.getElementById('reqEmail').value.trim();
  var note = document.getElementById('reqNote').value.trim();

  fetch(API_ENDPOINT + '/auth/request', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ name: name, email: email, note: note }),
  })
    .then(function(r) { return r.json(); })
    .then(function(data) {
      if (data.request_id) {
        // Persist pending state across refreshes
        localStorage.setItem(REQUEST_KEY, JSON.stringify({
          request_id: data.request_id, submitted_at: Date.now()
        }));
        showRequestedView();
      }
    });
}
```

## CSS Design System (Dark Theme)

```css
:root {
  --bg: #0f1117;
  --surface: #1a1d27;
  --surface2: #21253a;
  --border: #2e3348;
  --accent: #6c63ff;
  --accent2: #00c6ff;
  --text: #e2e8f0;
  --muted: #8892a4;
  --green: #22c55e;
}

/* Gate overlay — covers entire page */
.gate-overlay {
  position: fixed; inset: 0;
  z-index: 9000;
  display: flex; align-items: center; justify-content: center;
  background: var(--bg);
  transition: opacity .38s ease;
}
.gate-overlay.gate-hidden { opacity: 0; pointer-events: none; }

/* Gate card */
.gate-card {
  max-width: 420px; width: 90%;
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: 20px;
  padding: 2.5rem;
  text-align: center;
}

/* Admin FAB */
.admin-fab {
  position: fixed; bottom: 1.5rem; right: 1.5rem;
  z-index: 8000;
  display: none; /* shown after admin unlock */
  background: linear-gradient(135deg, var(--accent), var(--accent2));
  border: none; border-radius: 50%;
  width: 52px; height: 52px;
  cursor: pointer;
}

/* Admin modal */
.admin-modal {
  position: fixed; inset: 0;
  z-index: 8500;
  display: flex; align-items: center; justify-content: center;
  background: rgba(0,0,0,.6);
}
.admin-body {
  max-width: 780px; width: 95%;
  max-height: 88vh; overflow-y: auto;
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: 16px;
}
```

## Embedding in Any HTML Page

```html
<!-- Before </body> -->
<link rel="stylesheet" href="gate.css" />
<!-- Paste snippet.html content (overlay + admin modal markup) -->
<script src="gate.js"></script>
```

## Three Gate Views

1. **Unlock View** — key input + Unlock button + "Request Access" link
2. **Request View** — name, email, reason fields + Send Request button
3. **Requested View** — confirmation message + "Back to unlock" link

## Rules

- Admin key MUST be held in JS memory only (`_adminKey` variable) — NEVER in localStorage, sessionStorage, or cookies
- Session key stored in localStorage with explicit TTL — auto-expires via `setTimeout`
- Gate overlay uses CSS transition (opacity) for smooth fade — use `requestAnimationFrame` for re-show
- Request pending state persisted in localStorage — prevents duplicate submissions across refreshes
- Identity capture banner is non-blocking, shown once per session via `sessionStorage`
- Admin panel uses tabbed modal pattern — each tab lazy-loads data on switch
- All admin API calls include `X-Admin-Key` header — validated server-side
- Error messages should distinguish: invalid key, expired key, rate-limited, network error
- Escape all user-generated content before inserting into innerHTML (XSS prevention)
- Click outside admin modal should close it
