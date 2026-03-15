---
name: design-patterns-gallery
description: Visual pattern gallery with category filters, flow diagrams, detail panels, and quick reference table
category: ui
tags: [react, patterns, gallery, flow-visualization, design-patterns]
---

# Design Patterns Gallery

A filterable gallery page for displaying design patterns, architecture patterns, or best practices. Each pattern shows a flow visualization (step boxes with arrows), complexity badge, and expandable detail panel with "When to Use" and "Real Examples."

## When to Use

- Learning platforms teaching architecture or design patterns
- Documentation sites with pattern catalogs
- Any app that catalogs reusable solutions with visual flows

## Page Structure

```
┌─────────────────────────────────────────────────┐
│  🧩 Design Patterns                             │
│  [All] [Category A] [Category B] [Category C]  │
├─────────────────────┬───────────────────────────┤
│ Pattern Card        │ Pattern Card              │
│ Name [Complexity]   │ Name [Complexity]         │
│ Description...      │ Description...            │
│ [Step] → [Step] → . │ [Step] → [Step] → ...    │
├─────────────────────┴───────────────────────────┤
│ Detail Panel (expanded when selected)           │
│ ┌─────────────┬─────────────────────────┐       │
│ │ When to Use │ Real Examples           │       │
│ │ Description │ [Tag] [Tag] [Tag]       │       │
│ ├─────────────┴─────────────────────────┤       │
│ │ Flow: [Step] → [Step] → [Step] → ... │       │
│ └───────────────────────────────────────┘       │
├─────────────────────────────────────────────────┤
│ Quick Reference Table                            │
│ Pattern | Category | Complexity | Best For       │
│ ─────── | ──────── | ────────── | ──────────     │
│ Name    | Cat A    | [Basic]    | Use case       │
└─────────────────────────────────────────────────┘
```

## Backend Pattern

```python
PATTERNS = [
    {
        "name": "Pattern Name",
        "category": "Category",
        "complexity": "Basic",  # Basic | Intermediate | Advanced
        "technologies": ["tech1", "tech2"],  # optional
        "description": "What this pattern does and why.",
        "flow": ["Step 1", "Step 2", "Step 3", "Step N"],
        "when": "When to use this pattern",
        "examples": ["Real Example 1", "Real Example 2"],
    },
]

@router.get("/")
async def list_patterns():
    return PATTERNS
```

## Frontend Pattern

```jsx
const COMPLEXITY_COLORS = {
  Basic: 'bg-green-100 text-green-700',
  Intermediate: 'bg-yellow-100 text-yellow-700',
  Advanced: 'bg-red-100 text-red-700',
};

// Flow visualization: horizontal step boxes with arrows
{p.flow.map((step, i) => (
  <div key={i} className="flex items-center gap-1.5">
    <span className="text-xs px-2 py-1 rounded bg-gray-100">{step}</span>
    {i < p.flow.length - 1 && <span className="text-gray-400">→</span>}
  </div>
))}

// Detail panel: shown below the grid when a pattern is selected
// Quick reference table: always visible at bottom
```

## Content Guidelines

| Aspect | Recommendation |
|--------|----------------|
| Patterns count | 8-12 patterns |
| Categories | 3-5 categories |
| Flow steps | 4-6 steps per pattern |
| Examples | 2-3 real-world examples |
| Complexity | Mix of Basic/Intermediate/Advanced |

## Styling Rules

- Grid: `grid-cols-1 md:grid-cols-2 gap-4`
- Selected: `border-accent-500 ring-2 ring-accent-200 bg-accent-50`
- Complexity badges: green/yellow/red
- Flow steps: `bg-gray-100 text-gray-700 rounded px-2 py-1`
- Detail panel flow: `bg-white border shadow-sm rounded-lg px-3 py-2`
- Quick reference: standard table with `hover:bg-gray-50`
