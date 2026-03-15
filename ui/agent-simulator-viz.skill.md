---
name: Agent Simulator Visualization
description: Visualize AI agent ReAct loops with step-by-step thought, action, observation rendering
category: ui
difficulty: advanced
tags: [agent, simulator, react-loop, visualization]
stack: [React, Tailwind CSS, FastAPI]
---

# Agent Simulator Visualization

Build a step-by-step agent simulator that renders ReAct (Reason + Act) execution traces with color-coded phases and animated transitions.

## Pattern Overview

The simulator displays pre-scripted agent execution scenarios one step at a time. Each step belongs to a phase — Thought, Action, Observation, or Answer — rendered with a distinct color and icon. Users advance through steps manually or via auto-play, watching the agent's reasoning unfold like a debugger stepping through code.

## Phase Color System

| Phase | Background | Border | Icon | Purpose |
|---|---|---|---|---|
| **Thought** | `bg-purple-50` | `border-purple-300` | Brain | Agent's internal reasoning |
| **Action** | `bg-green-50` | `border-green-300` | Zap | Tool invocation with parameters |
| **Observation** | `bg-blue-50` | `border-blue-300` | Eye | Tool result / external data |
| **Answer** | `bg-amber-50` | `border-amber-300` | CheckCircle | Final synthesized response |

## Architecture

### State Shape

```jsx
const [scenarios, setScenarios] = useState([]);        // Available scenarios
const [activeScenario, setActiveScenario] = useState(null);
const [visibleSteps, setVisibleSteps] = useState([]);  // Steps revealed so far
const [currentStep, setCurrentStep] = useState(0);     // Current step index
const [isPlaying, setIsPlaying] = useState(false);     // Auto-play mode
```

### Step Rendering

Each step is rendered as a card with phase-specific styling. The card animates in from below using CSS transitions when it first appears.

```jsx
const PHASE_STYLES = {
  thought:     { bg: 'bg-purple-50', border: 'border-purple-300', icon: Brain,       label: 'Thought' },
  action:      { bg: 'bg-green-50',  border: 'border-green-300',  icon: Zap,         label: 'Action' },
  observation: { bg: 'bg-blue-50',   border: 'border-blue-300',   icon: Eye,         label: 'Observation' },
  answer:      { bg: 'bg-amber-50',  border: 'border-amber-300',  icon: CheckCircle, label: 'Answer' },
};

function StepCard({ step, isNew }) {
  const style = PHASE_STYLES[step.phase];
  const Icon = style.icon;

  return (
    <div className={`p-4 rounded-lg border-l-4 ${style.bg} ${style.border}
      transition-all duration-500 ${isNew ? 'animate-slideUp opacity-0' : 'opacity-100'}`}>
      <div className="flex items-center gap-2 mb-2">
        <Icon className="w-5 h-5" />
        <span className="font-semibold text-sm uppercase tracking-wide">{style.label}</span>
        <span className="text-xs text-gray-400 ml-auto">Step {step.step_number}</span>
      </div>
      <p className="text-gray-700">{step.content}</p>

      {/* Tool details for action steps */}
      {step.tool && (
        <div className="mt-3 p-3 bg-white/60 rounded font-mono text-sm">
          <div className="text-green-700 font-semibold">{step.tool}()</div>
          <pre className="text-gray-600 mt-1">{JSON.stringify(step.tool_input, null, 2)}</pre>
        </div>
      )}

      {/* Tool output for observation steps */}
      {step.tool_output && (
        <div className="mt-3 p-3 bg-white/60 rounded font-mono text-sm">
          <pre className="text-blue-700">{JSON.stringify(step.tool_output, null, 2)}</pre>
        </div>
      )}
    </div>
  );
}
```

### Step Advancement

The simulator advances one step at a time by calling `POST /api/simulator/step` with the current scenario ID and step number. The backend returns the next step and a progress fraction.

```jsx
async function advanceStep() {
  const res = await fetch('/api/simulator/step', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      scenario_id: activeScenario.id,
      current_step: currentStep,
    }),
  });
  const data = await res.json();
  setVisibleSteps(prev => [...prev, data.step]);
  setCurrentStep(data.step.step_number);
  if (data.is_last) setIsPlaying(false);
}
```

### Auto-Play Mode

When auto-play is enabled, a `useEffect` with `setInterval` calls `advanceStep()` every 1.5 seconds. The interval is cleared when the last step is reached or the user pauses.

```jsx
useEffect(() => {
  if (!isPlaying) return;
  const timer = setInterval(advanceStep, 1500);
  return () => clearInterval(timer);
}, [isPlaying, currentStep]);
```

### Progress Bar

A horizontal progress bar at the top shows how far through the scenario the user has advanced.

```jsx
<div className="h-2 bg-gray-200 rounded-full mb-6">
  <div
    className="h-2 bg-indigo-500 rounded-full transition-all duration-300"
    style={{ width: `${(currentStep / activeScenario.total_steps) * 100}%` }}
  />
</div>
```

### Scenario Selector

A card grid lets the user pick from available scenarios before starting. Each card shows the scenario title, description, agent type, and total step count.

### CSS Animation

```css
@keyframes slideUp {
  from { transform: translateY(20px); opacity: 0; }
  to   { transform: translateY(0);    opacity: 1; }
}
.animate-slideUp {
  animation: slideUp 0.5s ease-out forwards;
}
```

## Backend Pattern

The backend stores pre-scripted scenarios as static Python data. The step endpoint is stateless — it simply looks up the next step by index.

```python
@router.post("/step")
def simulator_step(body: StepRequest):
    scenario = next((s for s in SCENARIOS if s["id"] == body.scenario_id), None)
    if not scenario:
        raise HTTPException(404, "Scenario not found")
    next_idx = body.current_step  # current_step is 0-indexed position
    if next_idx >= len(scenario["steps"]):
        raise HTTPException(400, "No more steps")
    step = scenario["steps"][next_idx]
    return {
        "scenario_id": body.scenario_id,
        "step": step,
        "is_last": next_idx == len(scenario["steps"]) - 1,
        "progress": (next_idx + 1) / len(scenario["steps"]),
    }
```

## When to Use This Pattern

- Visualizing agent reasoning traces (ReAct, Chain-of-Thought, Tree-of-Thought)
- Debugging agent executions step by step
- Educational tools that teach how AI agents work internally
- Demo environments showing agent capabilities to stakeholders
