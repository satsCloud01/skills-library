---
name: guided-tour
description: "Implements multi-step guided tours with SVG spotlight overlay, viewport tracking, and localStorage persistence"
category: ui
difficulty: advanced
tags: [tour, onboarding, overlay, ux, walkthrough]
stack: [react-18, typescript, tailwind]
---

# Guided Tour System

You are a UX engineer. Implement guided tours using this proven pattern:

## Architecture

```
src/components/TourOverlay.tsx   — SVG mask cutout + tooltip positioning
src/hooks/useTour.ts             — Step state machine + localStorage
src/pages/Landing.tsx            — Tour steps definition + trigger button
```

## Tour Steps Definition

```tsx
const TOUR_STEPS = [
  { target: 'hero-section', title: 'Welcome', description: 'Start here...', position: 'bottom' as const },
  { target: 'features-grid', title: 'Features', description: 'Explore...', position: 'top' as const },
  // target = data-tour="value" attribute on DOM element
]
```

## useTour Hook

```tsx
const STORAGE_KEY = 'app_tour_seen'

export function useTour(steps: TourStep[]) {
  const [active, setActive] = useState(false)
  const [step, setStep] = useState(0)

  const start = () => { setStep(0); setActive(true) }
  const next  = () => step < steps.length - 1 ? setStep(s => s + 1) : finish()
  const prev  = () => step > 0 && setStep(s => s - 1)
  const finish = () => { setActive(false); localStorage.setItem(STORAGE_KEY, 'true') }

  return { active, step, currentStep: steps[step], start, next, prev, finish, totalSteps: steps.length }
}
```

## TourOverlay Component

Key implementation details:
1. **SVG mask**: Full-viewport SVG with `<rect fill="rgba(0,0,0,0.7)">` + cutout `<rect rx="8">` using mask
2. **Positioning**: `getBoundingClientRect()` on `[data-tour="${target}"]` element
3. **Viewport tracking**: `ResizeObserver` + `scroll` listener to reposition on changes
4. **Scroll into view**: `element.scrollIntoView({ behavior: 'smooth', block: 'center' })`
5. **Tooltip**: Absolute-positioned card with arrow, respects `position` prop
6. **Controls**: Back / Next / Skip buttons + step counter (`2 of 7`)

## Streamlit Tour (Python apps)

```python
if 'tour_step' not in st.session_state:
    st.session_state.tour_step = 0
    st.session_state.tour_active = False

TOUR_STEPS = [
    {"title": "Welcome", "content": "Start here...", "icon": "👋"},
]

if st.session_state.tour_active:
    step = TOUR_STEPS[st.session_state.tour_step]
    st.info(f"**{step['icon']} {step['title']}** — {step['content']}")
    col1, col2, col3 = st.columns([1, 1, 1])
    with col1: st.button("← Back", disabled=st.session_state.tour_step == 0)
    with col2: st.button("Skip Tour")
    with col3: st.button("Next →")
```

## Rules
- Always persist tour completion in localStorage/session_state
- Add `data-tour="step-id"` attributes to target elements
- Tour button visible on Landing page: "Take a Tour" with Sparkles icon
- 6-9 steps max — keep descriptions under 2 sentences
- Include "Try this" examples where possible
- Smooth scroll to off-screen elements before showing tooltip
- Z-index: overlay at 9999, tooltip at 10000
