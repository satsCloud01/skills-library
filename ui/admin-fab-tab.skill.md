---
name: Admin FAB Tab Panel
description: Add a new tab with data panel to a floating admin button modal in vanilla HTML/JS
category: ui
tags: [admin-panel, fab, tabs, vanilla-js, dashboard]
---

# Admin FAB Tab Panel

Add a new tab section to an existing floating action button (FAB) admin modal panel using vanilla HTML/JS. Follows the pattern used in the Sats Solution Registry admin panel.

## Steps

### 1. Add Tab Button

In the `.admin-tabs` container, add a new tab button:

```html
<button id="tab-{name}" class="admin-tab" onclick="switchTab('{name}')">
  {emoji} {Label}
</button>
```

### 2. Add Panel Container

In the `.admin-body` container, add the hidden panel div:

```html
<div id="panel-{name}" style="display:none"></div>
```

### 3. Register in switchTab

Add the tab name to the tab list array and add a loader call:

```javascript
// In the forEach array, add '{name}'
['keys','requests','log','identities','costs','{name}'].forEach(function(t) { ... });

// Add loader trigger
if (tab === '{name}') load{Name}();
```

### 4. Implement Load + Render Functions

```javascript
function load{Name}() {
  var panel = document.getElementById('panel-{name}');
  if (!panel) return;
  panel.innerHTML = '<div class="admin-status">Loading {label}...</div>';
  adminFetch('GET', '/admin/{endpoint}')
    .then(function(data) { render{Name}(panel, data); })
    .catch(function(e) {
      panel.innerHTML = '<div class="admin-status" style="color:#f87171">'
        + escHtml(e.message) + '</div>';
    });
}

function render{Name}(panel, data) {
  var items = data.items || [];
  if (!items.length) {
    panel.innerHTML = '<div class="admin-empty">No {label} found.</div>';
    return;
  }

  // Summary cards row (reuse costCard helper)
  var cards = '<div style="display:grid;grid-template-columns:repeat(auto-fit,minmax(150px,1fr));gap:1rem;margin-bottom:1.5rem">'
    + costCard('Total', items.length, '', '#6c63ff')
    + '</div>';

  // Data table
  var rows = items.map(function(item) {
    return '<tr>'
      + '<td>' + escHtml(item.name) + '</td>'
      + '<td style="color:var(--muted)">' + escHtml(item.status) + '</td>'
      + '<td style="text-align:right">' + actionButtons(item) + '</td>'
      + '</tr>';
  }).join('');

  var table = '<table class="key-table"><thead><tr>'
    + '<th>Name</th><th>Status</th><th style="text-align:right">Actions</th>'
    + '</tr></thead><tbody>' + rows + '</tbody></table>';

  panel.innerHTML = cards + table;
}
```

### 5. Action Buttons Pattern (for mutation actions)

```javascript
function {name}Action(id, action) {
  if (!confirm(action + ' ' + id + '?')) return;
  adminFetch('POST', '/admin/{endpoint}/' + id + '/' + action)
    .then(function() {
      setTimeout(load{Name}, 3000); // refresh after action propagates
    })
    .catch(function(e) { alert(e.message); });
}
```

### Panel Width

If adding more than 5 tabs, widen `.admin-panel` max-width (e.g., from 780px to 920px) to prevent tab wrapping issues.

### Styling Notes

- Summary cards: use the existing `costCard(title, value, subtitle, color)` helper
- Tables: use `.key-table` class for consistent styling
- Status dots: `<span style="display:inline-block;width:8px;height:8px;border-radius:50%;background:{color}"></span>`
- Action buttons: inline `style` with border + color matching the action severity
- Empty states: `<div class="admin-empty">...</div>`
- Loading states: `<div class="admin-status">Loading...</div>`
- Error states: `<div class="admin-status" style="color:#f87171">...</div>`
