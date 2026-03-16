---
name: svg-logo-generator
description: "Generate animated SVG logos with orbital rings, pillar labels, monograms, and brand taglines — navy/gold enterprise aesthetic, embeddable inline or standalone"
category: ui
difficulty: intermediate
tags: [logo, svg, branding, animation, identity, orbital, monogram, enterprise]
stack: [html, css, svg]
---

# SVG Logo Generator

You are a brand identity designer specializing in animated SVG logos. When asked to create a logo, build a self-contained SVG with CSS animations that works both inline in HTML and as a standalone `.svg` file.

## When to Use

- Creating brand logos, favicons, or hero-section identity marks
- Animated SVG logos for web pages (no image exports needed)
- Enterprise/premium brand aesthetics with navy + gold palette
- Logos that need to embed pillar/domain labels around a central mark

## Design System

### Color Palette (Navy + Gold — Enterprise Premium)

```
Primary Navy:  #0a1628  (backgrounds, fills)
Gold Accent:   #c8a96e  (strokes, text, highlights)
Light Gold:    #e8c98e  (gradient stops, emphasis)
Deep Gold:     #a8894e  (gradient stops, depth)
```

### Animation Keyframes

Always include these inside an SVG `<style>` block or `<defs>`:

```css
@keyframes orbitSpin { from { transform: rotate(0deg); } to { transform: rotate(360deg); } }
@keyframes pulse { 0%,100% { opacity: 0.4; } 50% { opacity: 1; } }
@keyframes nodePulse { 0%,100% { r: 4; } 50% { r: 6; } }
```

### Fonts

- **Monogram / Brand name**: `'Playfair Display', Georgia, serif` — elegant, premium
- **Labels / Taglines**: `'Inter', system-ui, sans-serif` — clean, modern

## Logo Archetypes

### 1. Orbital Monogram (Recommended for personal/platform brands)

A bold single letter centered in a circle, orbited by animated elliptical rings with pillar labels at cardinal positions.

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="240" height="235" viewBox="0 0 240 235">
  <defs>
    <linearGradient id="goldGrad" x1="0%" y1="0%" x2="100%" y2="100%">
      <stop offset="0%" stop-color="#e8c98e"/>
      <stop offset="50%" stop-color="#c8a96e"/>
      <stop offset="100%" stop-color="#a8894e"/>
    </linearGradient>
    <style>
      @keyframes orbitSpin { from { transform: rotate(0deg); } to { transform: rotate(360deg); } }
      @keyframes pulse { 0%,100% { opacity: 0.4; } 50% { opacity: 1; } }
      .orbit-ring { animation: orbitSpin 8s linear infinite; }
      .pulse-dot { animation: pulse 2s ease-in-out infinite; }
    </style>
  </defs>

  <!-- Pillar labels at cardinal positions -->
  <text x="120" y="18" text-anchor="middle" fill="#c8a96e" font-family="system-ui,sans-serif" font-size="9" font-weight="500" letter-spacing="2">PILLAR_1</text>
  <text x="215" y="98" text-anchor="middle" fill="#c8a96e" font-family="system-ui,sans-serif" font-size="9" font-weight="500" letter-spacing="2">PILLAR_2</text>
  <text x="120" y="178" text-anchor="middle" fill="#c8a96e" font-family="system-ui,sans-serif" font-size="9" font-weight="500" letter-spacing="2">PILLAR_3</text>
  <text x="22" y="98" text-anchor="middle" fill="#c8a96e" font-family="system-ui,sans-serif" font-size="8" font-weight="500" letter-spacing="1.5">PILLAR_4</text>

  <!-- Pulsing connector dots -->
  <circle cx="120" cy="24" r="3" fill="#c8a96e" class="pulse-dot"/>
  <circle cx="205" cy="93" r="3" fill="#c8a96e" class="pulse-dot" style="animation-delay:0.5s"/>
  <circle cx="120" cy="165" r="3" fill="#c8a96e" class="pulse-dot" style="animation-delay:1s"/>
  <circle cx="35" cy="93" r="3" fill="#c8a96e" class="pulse-dot" style="animation-delay:1.5s"/>

  <!-- Faint connecting lines to center -->
  <line x1="120" y1="27" x2="120" y2="55" stroke="rgba(200,169,110,0.15)" stroke-width="1"/>
  <line x1="202" y1="93" x2="170" y2="93" stroke="rgba(200,169,110,0.15)" stroke-width="1"/>
  <line x1="120" y1="162" x2="120" y2="133" stroke="rgba(200,169,110,0.15)" stroke-width="1"/>
  <line x1="38" y1="93" x2="70" y2="93" stroke="rgba(200,169,110,0.15)" stroke-width="1"/>

  <!-- Orbital ring 1 (clockwise) -->
  <g class="orbit-ring" style="transform-origin: 120px 93px;">
    <ellipse cx="120" cy="93" rx="70" ry="26" fill="none" stroke="url(#goldGrad)" stroke-width="1.5" transform="rotate(-30 120 93)" opacity="0.5"/>
    <circle cx="165" cy="55" r="4" fill="#e8c98e"/>
  </g>

  <!-- Orbital ring 2 (counter-clockwise, slower) -->
  <g class="orbit-ring" style="transform-origin: 120px 93px; animation-duration:12s; animation-direction:reverse;">
    <ellipse cx="120" cy="93" rx="70" ry="26" fill="none" stroke="url(#goldGrad)" stroke-width="1.5" transform="rotate(30 120 93)" opacity="0.35"/>
    <circle cx="75" cy="55" r="3" fill="#c8a96e"/>
  </g>

  <!-- Inner circle with monogram -->
  <circle cx="120" cy="93" r="38" fill="rgba(10,22,40,0.9)" stroke="#c8a96e" stroke-width="2"/>
  <text x="120" y="105" text-anchor="middle" fill="url(#goldGrad)" font-family="Georgia,serif" font-size="42" font-weight="700">X</text>

  <!-- Brand name + tagline -->
  <text x="120" y="200" text-anchor="middle" fill="#e8c98e" font-family="Georgia,serif" font-size="18" font-weight="600" letter-spacing="4">BRAND NAME</text>
  <text x="120" y="215" text-anchor="middle" fill="#c8a96e" font-family="system-ui,sans-serif" font-size="9" font-weight="400" font-style="italic" letter-spacing="2">Tagline goes here</text>
</svg>
```

**Key rules for orbital logos:**
- `transform-origin` on orbital `<g>` elements MUST match the center of the circle (e.g., `120px 93px`) — otherwise orbits drift off-center
- Use different `animation-duration` and `animation-direction: reverse` on the second ring for visual depth
- Place a small `<circle>` on each orbital path to create a "satellite" effect

### 2. Grid Constellation (Recommended for portfolio/registry brands)

A dot matrix where selected dots are highlighted and connected, representing a catalog of items.

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="240" height="180" viewBox="0 0 240 180">
  <defs>
    <style>
      @keyframes nodePulse { 0%,100% { r: 4; } 50% { r: 6; } }
      .active-node { animation: nodePulse 2s ease-in-out infinite; }
    </style>
  </defs>

  <!-- Grid: 5 columns x 3 rows, spacing 30px -->
  <!-- Active nodes (gold, animated) represent items in the catalog -->
  <circle cx="60" cy="40" r="4" fill="#c8a96e" class="active-node"/>
  <!-- Inactive nodes (dim) represent empty slots -->
  <circle cx="120" cy="40" r="4" fill="rgba(200,169,110,0.2)"/>

  <!-- Connection lines between active nodes -->
  <line x1="60" y1="40" x2="90" y2="40" stroke="rgba(200,169,110,0.25)" stroke-width="1"/>

  <!-- Brand text below grid -->
  <text x="120" y="140" text-anchor="middle" fill="#e8c98e" font-family="Georgia,serif" font-size="16" font-weight="600" letter-spacing="2">REGISTRY NAME</text>
  <text x="120" y="158" text-anchor="middle" fill="#8a8070" font-family="system-ui,sans-serif" font-size="8" letter-spacing="3">N ITEMS — ONE ECOSYSTEM</text>
</svg>
```

### 3. Hexagonal Shield (Recommended for favicons and compact marks)

```svg
<polygon points="100,25 162,62.5 162,137.5 100,175 38,137.5 38,62.5"
         fill="#0a1628" stroke="url(#goldGrad)" stroke-width="2.5"/>
<!-- Monogram centered inside -->
<text x="100" y="108" text-anchor="middle" fill="url(#goldGrad)"
      font-family="Georgia,serif" font-size="30" font-weight="700">SZ</text>
```

## Embedding Guidelines

### Inline in HTML (hero section)
Wrap in a `<div>` with `margin-bottom` and set a smaller `width`/`height` on the `<svg>` while keeping the full `viewBox`:
```html
<div style="margin-bottom:1.2rem;">
  <svg width="180" height="176" viewBox="0 0 240 235" style="display:inline-block;">
    <!-- ... full SVG contents ... -->
  </svg>
</div>
```

### Standalone `.svg` file
Include `xmlns="http://www.w3.org/2000/svg"` on the root `<svg>` element. Animations and `<style>` blocks work in all modern browsers.

### Namespace collisions
When embedding multiple SVGs in one page, prefix gradient/filter IDs to avoid collisions (e.g., `szGoldGrad` instead of `goldGrad`).

## Customization Checklist

When generating a logo for a user:
1. Ask for or infer: brand name, monogram letter(s), tagline, and 3-4 pillar/domain labels
2. Replace placeholder text in the archetype template
3. Adjust `viewBox` height if tagline wraps to multiple lines (add ~15px per extra line)
4. For light backgrounds, swap `fill="rgba(10,22,40,0.9)"` to `fill="rgba(245,247,250,0.95)"` and use darker gold tones
5. Save both a standalone `.svg` and show the inline embed snippet
