---
name: cloud-comparison-calculator
description: "Build a weighted comparison calculator with adjustable sliders, presets, and bar chart visualization"
category: ui
difficulty: intermediate
tags: [calculator, comparison, weighted-scoring, sliders, recharts]
stack: [React, Tailwind, Recharts]
---

# Weighted Comparison Calculator

You are a frontend engineer building an interactive weighted comparison calculator. This pattern works for cloud provider comparison, tool evaluation, vendor scoring, or any multi-criteria decision.

## Architecture

Single React page component with all logic self-contained (no backend needed).

```
src/pages/Calculator.jsx
```

## Data Model

```javascript
// Define criteria (decision dimensions)
const CRITERIA = [
  "Service Breadth",
  "Enterprise Governance",
  "Data & AI",
  "Developer Experience",
  "Hybrid/Multi-Cloud",
  "Cost Optimization",
  "Security & Compliance",
  "Operational Maturity"
];

// Baseline ratings per provider (0-5 scale, expert-curated)
const PROVIDER_RATINGS = {
  "AWS":   [5, 4, 4, 4, 3, 4, 5, 5],
  "Azure": [4, 5, 5, 4, 5, 4, 5, 4],
  "GCP":   [4, 3, 5, 5, 4, 4, 4, 4]
};

// Presets (pre-configured weight profiles)
const PRESETS = {
  "Balanced":            [5, 5, 5, 5, 5, 5, 5, 5],
  "Microsoft Enterprise":[3, 8, 6, 4, 7, 5, 8, 5],
  "Data/AI First":       [4, 3, 9, 7, 4, 5, 5, 4],
  "Ops Simplicity":      [4, 4, 4, 5, 3, 6, 5, 8]
};
```

## Scoring Algorithm

```javascript
const calculateScore = (weights, ratings) => {
  const totalWeight = weights.reduce((sum, w) => sum + w, 0);
  if (totalWeight === 0) return 0;
  const weightedSum = weights.reduce((sum, w, i) => sum + w * ratings[i], 0);
  return (weightedSum / totalWeight).toFixed(2);
};
```

## UI Components

### Weight Sliders
```jsx
{CRITERIA.map((criterion, i) => (
  <div key={criterion} className="flex items-center gap-4">
    <span className="w-48 text-sm">{criterion}</span>
    <input
      type="range" min="0" max="10" value={weights[i]}
      onChange={(e) => updateWeight(i, parseInt(e.target.value))}
      className="flex-1"
    />
    <span className="w-8 text-center font-mono">{weights[i]}</span>
  </div>
))}
```

### Preset Buttons
```jsx
{Object.entries(PRESETS).map(([name, values]) => (
  <button key={name} onClick={() => setWeights([...values])}
    className="px-3 py-1 rounded bg-gray-700 hover:bg-gray-600 text-sm">
    {name}
  </button>
))}
```

### Results Bar Chart (Recharts)
```jsx
<BarChart data={providers.map(p => ({ name: p, score: calculateScore(weights, PROVIDER_RATINGS[p]) }))}>
  <XAxis dataKey="name" />
  <YAxis domain={[0, 5]} />
  <Bar dataKey="score" fill="#00d4ff" />
</BarChart>
```

## Customization Points

- **Criteria**: Change to match your comparison domain (tools, vendors, frameworks)
- **Providers**: Any set of options being compared (2-6 recommended)
- **Rating scale**: 0-5 default, adjustable
- **Weight scale**: 0-10 default slider range
- **Presets**: Domain-specific weight profiles for quick switching
- **Colors**: Per-provider colors in the bar chart for visual distinction
