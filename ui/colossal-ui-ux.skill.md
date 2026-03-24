---
name: colossal-ui-ux
description: "Generates premium editorial-style web pages inspired by Colossal.com — cinematic heroes, numbered sections, bold statistics, species/product cards, before/after sliders, process flowcharts, partner grids, and immersive scroll storytelling. Outputs self-contained HTML."
version: 1.0.0
category: ui
difficulty: intermediate
tags: [html, css, editorial, immersive, storytelling, landing-page, cinematic, scroll, dark-theme]
stack: [html5, css3, vanilla-js, google-fonts]
---

# Colossal UI/UX — Cinematic Editorial Page Generator

Generate premium, immersive single-file HTML pages inspired by the **Colossal Biosciences** design language: bold cinematic heroes, numbered editorial sections, dramatic statistics, progressive disclosure, and scroll-driven storytelling.

## When to Use

Trigger when the user asks for: "cinematic page", "editorial layout", "immersive landing page", "storytelling page", "magazine-style page", "colossal style", "bold hero page", "statistics showcase", "species page", "process flowchart page", or any request emphasizing dramatic visual storytelling with numbered sections.

---

## Design System Reference

### Fonts
- **Display**: `Space Grotesk` (weights: 500, 700) — hero headlines, section numbers, statistics
- **Body**: `Inter` (weights: 300, 400, 500, 600) — body text, labels, navigation
- **Serif accent**: `Playfair Display` (weights: 700, 900; italic 400) — pull quotes, testimonial text
- Import:
```
https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@500;700&family=Inter:wght@300;400;500;600&family=Playfair+Display:ital,wght@0,700;0,900;1,400&display=swap
```

### Color Palette (CSS Variables)
```css
:root {
  /* Core */
  --bg-primary: #0a0a0a;          /* page background — near-black */
  --bg-surface: #111111;          /* cards, panels */
  --bg-elevated: #1a1a1a;         /* hover states, elevated cards */
  --bg-section-alt: #050505;      /* alternate section background */

  /* Brand */
  --brand-purple: #7a00df;        /* primary accent — CTAs, highlights */
  --brand-purple-light: #9b30ff;  /* hover state */
  --brand-purple-glow: rgba(122, 0, 223, 0.15); /* glow effects */
  --brand-cyan: #0693e3;          /* secondary accent — links, data */
  --brand-green: #00c853;         /* conservation/success accent */

  /* Text */
  --text-primary: #ffffff;        /* headlines, primary text */
  --text-secondary: #b0b0b0;      /* body text */
  --text-muted: #666666;          /* labels, captions, section numbers */
  --text-accent: #7a00df;         /* highlighted text */

  /* Borders & Effects */
  --border: rgba(255, 255, 255, 0.08);
  --border-hover: rgba(255, 255, 255, 0.15);
  --glow-shadow: 0 0 40px rgba(122, 0, 223, 0.2);
  --card-shadow: 0 4px 24px rgba(0, 0, 0, 0.4);

  /* Spacing scale */
  --space-xs: 0.5rem;    /* 8px */
  --space-sm: 1rem;      /* 16px */
  --space-md: 1.5rem;    /* 24px */
  --space-lg: 3rem;      /* 48px */
  --space-xl: 5rem;      /* 80px */
  --space-2xl: 8rem;     /* 128px */

  /* Typography scale */
  --text-xs: 0.75rem;    /* 12px — labels */
  --text-sm: 0.875rem;   /* 14px — captions */
  --text-base: 1rem;     /* 16px — body */
  --text-lg: 1.125rem;   /* 18px — lead body */
  --text-xl: 1.5rem;     /* 24px — subheads */
  --text-2xl: 2.5rem;    /* 40px — section titles */
  --text-3xl: clamp(2.5rem, 5vw, 4.5rem); /* hero headline */
  --text-stat: clamp(3rem, 8vw, 7rem);    /* giant statistics */
}
```

### Layout Tokens
- **Max content width**: 1200px centered
- **Wide content width**: 1400px (for grids, partner logos)
- **Section padding**: `var(--space-2xl) var(--space-lg)` (128px vertical, 48px horizontal)
- **Card border-radius**: 0px (sharp, editorial) OR 4px (subtle softness)
- **Section dividers**: 1px solid `var(--border)` or none (rely on spacing)

---

## Page Architecture

Every generated page MUST follow this section flow (include/skip based on content):

### 1. Navigation Bar (sticky)
```
┌─────────────────────────────────────────────────────┐
│  LOGO          01 About  02 Impact  03 Process  ... │
│                                          [CTA btn]  │
└─────────────────────────────────────────────────────┘
```
- Fixed top, `backdrop-filter: blur(12px)`, `background: rgba(10,10,10,0.85)`
- Numbered nav items: `<span class="nav-number">01</span> Label`
- Nav number in `--text-muted`, label in `--text-secondary`, active in `--text-primary`
- Hamburger menu on mobile (< 768px)
- CTA button: pill shape, `--brand-purple` background

### 2. Hero Section (full viewport)
```
┌─────────────────────────────────────────────────────┐
│                                                     │
│         [SECTION 00 label]                          │
│         MASSIVE HEADLINE                            │
│         SPANNING MULTIPLE LINES                     │
│                                                     │
│         Subtitle text in lighter weight             │
│                                                     │
│         [Learn More →]                              │
│                                                     │
│         ↓ scroll indicator                          │
└─────────────────────────────────────────────────────┘
```
- Full viewport height: `min-height: 100vh`
- Background: CSS gradient or solid dark with subtle pattern
- If user provides image/video URL, use as background with dark overlay `rgba(0,0,0,0.55)`
- Headline: `Space Grotesk 700`, `var(--text-3xl)`, uppercase, letter-spacing: -0.02em
- Subtitle: `Inter 300`, `var(--text-lg)`, `--text-secondary`, max-width: 600px
- Scroll indicator: animated chevron at bottom center
- Fade-in animation on load (CSS keyframes)

### 3. Statistics Band
```
┌─────────────────────────────────────────────────────┐
│   30,000          150            1,000,000           │
│   species/year    per day        threatened           │
│   [supporting     [supporting    [supporting          │
│    text]           text]          text]                │
└─────────────────────────────────────────────────────┘
```
- Dark background `var(--bg-section-alt)`
- Numbers: `Space Grotesk 700`, `var(--text-stat)`, `--text-primary`
- Labels: `Inter 400`, `var(--text-sm)`, uppercase, letter-spacing 3px, `--text-muted`
- Supporting text: `Inter 300`, `var(--text-sm)`, `--text-secondary`
- CSS counter animation on scroll (Intersection Observer)
- Flex row, evenly spaced, border-right dividers between stats

### 4. Numbered Content Sections
```
┌─────────────────────────────────────────────────────┐
│  SECTION 01                                          │
│  ─────────                                           │
│  Section Title Here                                  │
│                                                      │
│  Body text with generous line-height (1.8) and      │
│  max-width: 700px for readability.                  │
│                                                      │
│  [Discover More →]                                   │
└─────────────────────────────────────────────────────┘
```
- Section label: `var(--text-xs)`, uppercase, letter-spacing 4px, `--text-muted`
- Thin line under label: 60px wide, 2px, `--brand-purple`
- Title: `Space Grotesk 700`, `var(--text-2xl)`, `--text-primary`
- Body: `Inter 400`, `var(--text-base)`, `--text-secondary`, line-height 1.8, max-width 700px
- CTA link: `--brand-purple`, no underline, arrow suffix `→`, hover underline
- Alternate layout: odd sections left-aligned, even sections right-aligned (on desktop)
- Fade-up animation on scroll entry

### 5. Split Content (Image + Text)
```
┌──────────────────┬──────────────────────────────────┐
│                  │  SECTION 02                       │
│   [IMAGE]        │  Title                            │
│                  │  Body text...                     │
│                  │  [CTA →]                          │
└──────────────────┴──────────────────────────────────┘
```
- 50/50 grid on desktop, stacked on mobile
- Image side: `object-fit: cover`, full height of section
- Text side: vertically centered, padded `var(--space-xl)`
- Alternate: even sections flip image to right

### 6. Card Grid (Species / Products / Features)
```
┌────────────┐  ┌────────────┐  ┌────────────┐
│ [corner    │  │            │  │            │
│  icon]     │  │  IMAGE     │  │            │
│            │  │            │  │            │
├────────────┤  ├────────────┤  ├────────────┤
│ Title      │  │ Title      │  │ Title      │
│ Subtitle   │  │ Subtitle   │  │ Subtitle   │
│ [Explore →]│  │ [Explore →]│  │ [Explore →]│
└────────────┘  └────────────┘  └────────────┘
```
- Grid: `repeat(auto-fit, minmax(340px, 1fr))`, gap `var(--space-md)`
- Card: `--bg-surface`, border `var(--border)`, sharp corners (0px radius)
- Corner icon overlay: small badge top-left over image
- Hover: `translateY(-6px)`, border becomes `--border-hover`, `--card-shadow`
- Image ratio: 16:9 via `aspect-ratio: 16/9`, `object-fit: cover`
- Title: `Space Grotesk 700`, `var(--text-xl)`, `--text-primary`
- Subtitle: `Inter 400`, `var(--text-sm)`, `--text-secondary`

### 7. Process Flowchart (Numbered Steps)
```
  ①─────────②─────────③─────────④
  Step 1     Step 2     Step 3     Step 4
  Desc       Desc       Desc       Desc
```
- Horizontal on desktop (flex), vertical on mobile
- Step number: circle, 48px, `--brand-purple` border, `Space Grotesk 700`
- Connecting line: 2px dashed `var(--border)`
- Step title: `Inter 600`, `var(--text-base)`, `--text-primary`
- Step description: `Inter 400`, `var(--text-sm)`, `--text-secondary`
- Active/completed steps: filled circle `--brand-purple`, white number

### 8. Before/After Comparison
```
┌────────────────────────────────────────────┐
│   BEFORE          │|│         AFTER         │
│                   │|│                        │
│                   │|│                        │
│              [drag handle]                   │
└────────────────────────────────────────────┘
```
- Container: `position: relative`, `overflow: hidden`
- Two images stacked, top image clipped via `clip-path: inset(0 50% 0 0)`
- Draggable divider with vertical line and circle handle
- JS: mousedown/touchstart listener updates clip-path percentage
- Labels: positioned top-left and top-right, uppercase, `--text-muted`

### 9. Testimonial / Expert Quotes
```
┌─────────────────────────────────────────────────────┐
│  "Quote text in Playfair Display italic..."          │
│                                                      │
│  — Name, Title                                       │
│    Organization                                      │
└─────────────────────────────────────────────────────┘
```
- Quote text: `Playfair Display italic 400`, `var(--text-xl)`, `--text-primary`
- Opening quote mark: decorative `"`, `--brand-purple`, `font-size: 6rem`, `opacity: 0.3`
- Attribution: `Inter 500`, `var(--text-sm)`, `--text-muted`
- If multiple quotes: horizontal carousel with dot indicators

### 10. Partner / Logo Grid
```
┌─────────────────────────────────────────────────────┐
│  [logo] [logo] [logo] [logo] [logo] [logo]          │
│  [logo] [logo] [logo] [logo] [logo] [logo]          │
└─────────────────────────────────────────────────────┘
```
- Grid: `repeat(auto-fit, minmax(140px, 1fr))`, gap `var(--space-lg)`
- Logos: grayscale by default (`filter: grayscale(1) brightness(0.7)`)
- Hover: full color (`filter: none`), `transition: 0.3s`
- If no logos provided, render partner names in styled text boxes

### 11. Newsletter / CTA Band
```
┌─────────────────────────────────────────────────────┐
│  Let's build something extraordinary.                │
│  [email input_______________] [Subscribe]            │
└─────────────────────────────────────────────────────┘
```
- Background: subtle gradient `--bg-primary` → `--bg-surface`
- Input: transparent border-bottom style, `--text-primary`
- Button: `--brand-purple`, pill shape, hover glow

### 12. Footer
```
┌─────────────────────────────────────────────────────┐
│  LOGO                                                │
│                                                      │
│  Col 1 Links    Col 2 Links    Col 3 Links   Social │
│                                                      │
│  © 2026 Brand. All rights reserved.                  │
└─────────────────────────────────────────────────────┘
```
- Multi-column flex layout, border-top `var(--border)`
- Link style: `--text-muted`, hover `--text-primary`
- Social icons: SVG inline, 20px, `--text-muted`, hover `--brand-purple`

---

## CSS Animation Library

Include these CSS keyframes in every generated page:

```css
@keyframes fadeInUp {
  from { opacity: 0; transform: translateY(30px); }
  to { opacity: 1; transform: translateY(0); }
}
@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}
@keyframes slideInLeft {
  from { opacity: 0; transform: translateX(-40px); }
  to { opacity: 1; transform: translateX(0); }
}
@keyframes countUp {
  from { opacity: 0; transform: scale(0.8); }
  to { opacity: 1; transform: scale(1); }
}
@keyframes pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.5; }
}
@keyframes scrollDown {
  0% { transform: translateY(0); opacity: 1; }
  100% { transform: translateY(12px); opacity: 0; }
}

.animate-on-scroll {
  opacity: 0;
  transform: translateY(30px);
  transition: opacity 0.8s ease, transform 0.8s ease;
}
.animate-on-scroll.visible {
  opacity: 1;
  transform: translateY(0);
}
```

## JavaScript — Scroll Animations & Interactions

Include this script block at the end of `<body>`:

```js
// Intersection Observer for scroll animations
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      entry.target.classList.add('visible');
      // Counter animation for stat numbers
      if (entry.target.dataset.count) {
        animateCounter(entry.target);
      }
    }
  });
}, { threshold: 0.15 });

document.querySelectorAll('.animate-on-scroll').forEach(el => observer.observe(el));

// Counter animation
function animateCounter(el) {
  const target = parseInt(el.dataset.count.replace(/,/g, ''));
  const duration = 2000;
  const start = performance.now();
  const format = el.dataset.count.includes(',');
  function update(now) {
    const progress = Math.min((now - start) / duration, 1);
    const eased = 1 - Math.pow(1 - progress, 3);
    const current = Math.floor(eased * target);
    el.textContent = format ? current.toLocaleString() : current;
    if (progress < 1) requestAnimationFrame(update);
    else el.textContent = el.dataset.count;
  }
  requestAnimationFrame(update);
}

// Smooth scroll for nav links
document.querySelectorAll('a[href^="#"]').forEach(a => {
  a.addEventListener('click', e => {
    e.preventDefault();
    document.querySelector(a.getAttribute('href'))?.scrollIntoView({ behavior: 'smooth' });
  });
});

// Mobile nav toggle
const navToggle = document.querySelector('.nav-toggle');
const navMenu = document.querySelector('.nav-menu');
if (navToggle && navMenu) {
  navToggle.addEventListener('click', () => navMenu.classList.toggle('open'));
}
```

## Before/After Slider JS

If the page includes a before/after section, add:
```js
document.querySelectorAll('.ba-container').forEach(container => {
  const handle = container.querySelector('.ba-handle');
  const before = container.querySelector('.ba-before');
  let dragging = false;
  const move = (x) => {
    const rect = container.getBoundingClientRect();
    const pct = Math.max(0, Math.min(100, ((x - rect.left) / rect.width) * 100));
    before.style.clipPath = `inset(0 ${100 - pct}% 0 0)`;
    handle.style.left = pct + '%';
  };
  handle.addEventListener('mousedown', () => dragging = true);
  handle.addEventListener('touchstart', () => dragging = true);
  window.addEventListener('mouseup', () => dragging = false);
  window.addEventListener('touchend', () => dragging = false);
  window.addEventListener('mousemove', e => dragging && move(e.clientX));
  window.addEventListener('touchmove', e => dragging && move(e.touches[0].clientX));
});
```

---

## Responsive Breakpoints

```css
/* Tablet */
@media (max-width: 1024px) {
  .split-content { grid-template-columns: 1fr; }
  .stats-band { flex-direction: column; gap: var(--space-lg); }
  .process-flow { flex-direction: column; }
}
/* Mobile */
@media (max-width: 768px) {
  :root { --space-2xl: 4rem; --space-xl: 2.5rem; }
  .nav-menu { display: none; }
  .nav-menu.open { display: flex; flex-direction: column; }
  .card-grid { grid-template-columns: 1fr; }
  .hero h1 { font-size: clamp(2rem, 8vw, 3rem); }
  .stat-number { font-size: clamp(2.5rem, 10vw, 4rem); }
}
```

---

## Generation Rules

1. **Output**: Single self-contained `.html` file — all CSS in `<style>`, all JS before `</body>`
2. **No frameworks**: Pure HTML, CSS, JS — no Tailwind, React, or external dependencies (except Google Fonts)
3. **Section numbering**: Always use `00`, `01`, `02a`, `02b`, etc. labels above section titles
4. **Statistics**: Always animate counters on scroll — use `data-count` attribute
5. **Images**: Use user-provided URLs, or `placeholder` divs with gradient backgrounds as stand-ins: `background: linear-gradient(135deg, var(--bg-surface), var(--bg-elevated))`
6. **Dark theme only**: This design system is exclusively dark — never generate light backgrounds
7. **Whitespace**: Be generous — `padding: var(--space-2xl)` between major sections minimum
8. **Typography contrast**: Headlines BOLD and LARGE, body text light and readable
9. **Uppercase sparingly**: Section labels, nav items, stat labels — never body text
10. **Accessibility**: All images need alt text, focusable interactive elements, color contrast ≥ 4.5:1
11. **Scroll animations**: Every section gets `class="animate-on-scroll"` — observer triggers fade-in
12. **Performance**: Lazy-load images (`loading="lazy"`), minimal JS, CSS containment where helpful
13. **CTA style**: Text links with arrow `→` (editorial) OR pill buttons `--brand-purple` (action)
14. **Quotes**: Always use `Playfair Display italic` for testimonial/expert quotes
15. **Partner logos**: Grayscale by default, color on hover

---

## Adaptation Guidelines

When the user provides a topic/product/brand:
- Replace Colossal-specific content with user's domain
- Keep the numbered section structure and visual hierarchy
- Adapt the statistics band to domain-relevant numbers
- Swap species cards for product/feature/team cards as needed
- Maintain the cinematic, editorial tone in all generated copy
- If the user provides brand colors, override `--brand-purple` and `--brand-cyan` but keep the dark background system
- The before/after slider can be adapted for any comparison (old/new, problem/solution, before/after)
