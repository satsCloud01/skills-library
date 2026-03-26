---
name: ambient-background
description: Ambient Background Animations — floating particles, orb drifts, aurora sweeps, film grain, scroll-fade, and card glow effects for premium dark-themed UIs
trigger: When the user wants to add ambient/atmospheric background animations, floating particles, glowing orbs, aurora effects, film grain, or premium visual polish to a dark-themed UI
category: ui
---

# Ambient Background Animations

Add premium ambient background effects to dark-themed UIs. Includes floating canvas particles, CSS-animated orb drifts, aurora sweeps, film grain overlays, scroll-triggered fade-ins, and hover glow effects.

## CSS Animations (add to global CSS)

```css
/* Floating orb — slow drifting radial glow */
@keyframes orbDrift {
  0%   { transform: translate(0, 0) scale(1); opacity: 0.12; }
  25%  { transform: translate(60px, -40px) scale(1.15); opacity: 0.18; }
  50%  { transform: translate(-30px, 50px) scale(0.9); opacity: 0.1; }
  75%  { transform: translate(-50px, -20px) scale(1.1); opacity: 0.16; }
  100% { transform: translate(0, 0) scale(1); opacity: 0.12; }
}

/* Gentle breathing pulse */
@keyframes gentlePulse {
  0%, 100% { transform: scale(1); opacity: 0.06; }
  50% { transform: scale(1.3); opacity: 0.14; }
}

/* Horizontal aurora color band */
@keyframes auroraSweep {
  0%   { transform: translateX(-100%) skewX(-15deg); opacity: 0; }
  20%  { opacity: 0.06; }
  50%  { opacity: 0.04; }
  80%  { opacity: 0.06; }
  100% { transform: translateX(200%) skewX(-15deg); opacity: 0; }
}

/* Film grain noise */
@keyframes grainShift {
  0%, 100% { transform: translate(0, 0); }
  10% { transform: translate(-2%, -2%); }
  30% { transform: translate(1%, 3%); }
  50% { transform: translate(-3%, 1%); }
  70% { transform: translate(3%, -1%); }
  90% { transform: translate(1%, 2%); }
}

/* Scroll fade-in */
.fade-in-section {
  opacity: 0; transform: translateY(30px);
  transition: opacity 0.8s ease, transform 0.8s ease;
}
.fade-in-section.visible { opacity: 1; transform: translateY(0); }

/* Card hover glow */
.card-glow { transition: box-shadow 0.4s ease, transform 0.3s ease; }
.card-glow:hover {
  box-shadow: 0 0 40px rgba(42,106,96,.12), 0 20px 50px rgba(0,0,0,.4);
  transform: translateY(-3px);
}

/* Gold text shimmer */
@keyframes textShimmer {
  0% { background-position: -200% center; }
  100% { background-position: 200% center; }
}
.text-shimmer {
  background: linear-gradient(90deg, #C4BAA8 0%, #E2B96F 25%, #C8923A 50%, #E2B96F 75%, #C4BAA8 100%);
  background-size: 200% auto;
  -webkit-background-clip: text; background-clip: text;
  -webkit-text-fill-color: transparent;
  animation: textShimmer 6s linear infinite;
}

/* Breathing border */
@keyframes borderBreathe {
  0%, 100% { border-color: rgba(255,255,255,.06); }
  50% { border-color: rgba(42,106,96,.25); }
}
.border-breathe { animation: borderBreathe 4s ease-in-out infinite; }
```

## CSS Classes for Sections

```css
/* Floating orbs (apply to any section) */
.ambient-orbs { position: relative; overflow: hidden; }
.ambient-orbs::before, .ambient-orbs::after {
  content: ''; position: absolute; border-radius: 50%;
  pointer-events: none; filter: blur(80px);
}
.ambient-orbs::before {
  width: 400px; height: 400px;
  background: radial-gradient(circle, rgba(42,106,96,.25) 0%, transparent 70%);
  top: -10%; right: -5%;
  animation: orbDrift 20s ease-in-out infinite;
}
.ambient-orbs::after {
  width: 350px; height: 350px;
  background: radial-gradient(circle, rgba(200,146,58,.2) 0%, transparent 70%);
  bottom: -8%; left: -3%;
  animation: orbDrift 25s ease-in-out infinite reverse;
}

/* Film grain overlay */
.ambient-grain { position: relative; }
.ambient-grain::after {
  content: ''; position: absolute; inset: 0;
  pointer-events: none; opacity: 0.03; z-index: 1;
  background-image: url("data:image/svg+xml,...feTurbulence...");
  background-size: 200px;
  animation: grainShift 4s steps(6) infinite;
}

/* Aurora sweep */
.ambient-aurora { position: relative; overflow: hidden; }
.ambient-aurora::before {
  content: ''; position: absolute; top: 20%; left: 0;
  width: 100%; height: 120px;
  background: linear-gradient(90deg, transparent, rgba(42,106,96,.12), rgba(200,146,58,.08), transparent);
  pointer-events: none;
  animation: auroraSweep 18s ease-in-out infinite;
}
```

## React: FloatingParticles Component

```jsx
import { useEffect, useRef } from 'react'

export function FloatingParticles({ count = 25, color = 'rgba(200,146,58,0.3)' }) {
  const ref = useRef(null)
  useEffect(() => {
    const canvas = ref.current
    const ctx = canvas.getContext('2d')
    const resize = () => { canvas.width = canvas.offsetWidth; canvas.height = canvas.offsetHeight }
    resize(); window.addEventListener('resize', resize)
    const particles = Array.from({ length: count }, () => ({
      x: Math.random() * canvas.width, y: Math.random() * canvas.height,
      r: Math.random() * 1.8 + 0.5, vy: -(Math.random() * 0.3 + 0.1),
      vx: (Math.random() - 0.5) * 0.2, opacity: Math.random() * 0.5 + 0.1,
    }))
    let id
    const draw = () => {
      ctx.clearRect(0, 0, canvas.width, canvas.height)
      for (const p of particles) {
        p.x += p.vx; p.y += p.vy
        if (p.y < -10) { p.y = canvas.height + 10; p.x = Math.random() * canvas.width }
        ctx.beginPath(); ctx.arc(p.x, p.y, p.r, 0, Math.PI * 2)
        ctx.fillStyle = color.replace(/[\d.]+\)$/, `${p.opacity})`); ctx.fill()
      }
      id = requestAnimationFrame(draw)
    }
    draw()
    return () => { cancelAnimationFrame(id); window.removeEventListener('resize', resize) }
  }, [count, color])
  return <canvas ref={ref} className="absolute inset-0 w-full h-full pointer-events-none" />
}
```

## React: FadeInOnScroll Component

```jsx
export function FadeInOnScroll({ children, delay = 0 }) {
  const ref = useRef(null)
  useEffect(() => {
    const el = ref.current
    const obs = new IntersectionObserver(([e]) => {
      if (e.isIntersecting) { setTimeout(() => el.classList.add('visible'), delay); obs.unobserve(el) }
    }, { threshold: 0.15 })
    obs.observe(el)
    return () => obs.disconnect()
  }, [delay])
  return <div ref={ref} className="fade-in-section">{children}</div>
}
```

## Usage

```jsx
{/* Section with teal orbs + aurora + grain */}
<Section ambient="orbs-teal aurora grain">
  <FadeInOnScroll>
    <h2>Content fades in on scroll</h2>
  </FadeInOnScroll>
</Section>

{/* Hero with floating particles */}
<section className="relative ambient-grain">
  <FloatingParticles count={30} color="rgba(226,185,111,0.35)" />
  <h1 className="text-shimmer">Shimmering heading</h1>
</section>

{/* Card with hover glow */}
<div className="card-glow border-breathe">
  Card content
</div>
```
