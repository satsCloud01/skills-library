---
name: product-bio
description: Generate a single-page product bio / landing page HTML file for any software product. Use when the user asks to "create a product bio", "generate a landing page", "make a product page", "build a product showcase", or wants a standalone HTML page that presents a product professionally. Outputs a self-contained HTML file matching the Agent Substrate design system.
version: 1.0.0
---

# ProductBio — Single-Page Product Bio Generator

Generate a polished, self-contained single-file HTML product bio page that matches the **Agent Substrate** design language.

## Design System Reference

### Fonts
- **Primary**: `Inter` (weights: 300–900) — all body text, labels, stats, descriptions
- **Serif accent**: `Playfair Display` (weights: 400, 700, 900; italic 400) — hero titles, section titles, stat values, tenet headings
- Import: `https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800;900&family=Playfair+Display:ital,wght@0,400;0,700;0,900;1,400&display=swap`
- Use class `.serif` for Playfair Display elements

### Color Palette (CSS Variables)
```css
:root {
  --bg: #f5f7fb;           /* page background */
  --bg-alt: #edf0f7;       /* alternate background */
  --bg-card: #ffffff;       /* card background */
  --navy: #152040;          /* primary brand — headings, hero, kernel */
  --navy-dark: #0f1729;
  --navy-mid: #1e2d52;
  --navy-light: #2a3a66;
  --accent: #1e3a6e;        /* section labels, links */
  --gold: #c8a96e;          /* premium accent */
  --gold-light: #e8c98e;
  --gold-deep: #a8894e;
  --text: #1a2040;          /* body text */
  --text-muted: #6b7490;    /* descriptions, secondary text */
  --indigo: #4f56c2;        /* phase: Ideate */
  --violet: #7c5cbf;        /* phase: Build */
  --amber: #d48f0b;         /* phase: Govern */
  --red: #d44040;           /* phase: Protect */
  --green: #0ea572;         /* phase: Deploy */
  --cyan: #0899b5;          /* phase: Intelligence */
  --pink: #c94088;          /* phase: Iterate */
  --border: rgba(30,45,82,0.1);
}
```

### Layout Patterns
- **Max width**: 1400px centered with `margin: 0 auto`
- **Section padding**: `100px 40px`
- **Border radius**: 14–16px for cards, 10–12px for smaller elements, 20px for pills/badges
- **Card shadows**: `0 2px 12px rgba(0,0,0,0.04)` → hover: `0 8px 24px rgba(0,0,0,0.06)`
- **Hover transitions**: `transform 0.3s` with `translateY(-4px)` or `scale(1.01–1.05)`

### Section Anatomy
Every content section follows this structure:
1. **Section label**: 12px uppercase, letter-spacing 4px, color `--accent`
2. **Section title**: Playfair Display, clamp(32px, 4vw, 52px), weight 800, color `--navy`
3. **Section description**: 18px, color `--text-muted`, max-width 700px, line-height 1.7
4. **Gold line divider**: 80px × 3px, gradient from `--accent` to `--indigo`, margin `24px 0 40px`
5. **Content grid**: `grid-template-columns: repeat(auto-fit, minmax(300-380px, 1fr))`, gap 20-24px

### Hero Pattern
- Full viewport height (`min-height: 100vh`), centered content
- Subtle grid background: 60px grid lines at 4% opacity
- Radial gradient overlays for depth
- Title: Playfair Display, clamp(48px, 6vw, 86px), weight 900
- Tagline: Playfair Display italic, clamp(18px, 2.2vw, 28px)
- Stat cards in flex row: Playfair serif numbers (36px, weight 800) + uppercase labels (12px, letter-spacing 2px)
- Scroll hint: 13px uppercase, letter-spacing 3px, bounce animation

### Card Patterns
- **Why/Problem cards**: White bg, 1px border, 16px radius, 32px padding, large ghost numbers (48px, 8% opacity)
- **Tenet/Principle cards**: Same base, with small uppercase label (11px, letter-spacing 3px) + Playfair h3
- **Module cards**: Tinted background (4% phase color), 1px phase border, 3px top color bar, emoji icon, capability bullet list with colored dots

### CTA Section
- Centered text, 120px vertical padding
- Radial gradient background
- Button: navy bg, white text, 18px weight 700, 14px radius, 18px 56px padding, hover scale(1.05) + deeper shadow

### Footer
- Centered, 40px padding, 1px top border
- Brand name in Playfair Display 16px weight 700
- Tagline in 13px muted text

### Responsive
- 768px breakpoint: collapse grids to single column, reduce swimlane label width

### Section Dividers
- `height: 1px; background: linear-gradient(90deg, transparent, rgba(30,58,110,0.1), transparent)`

## Generation Instructions

When the user asks for a ProductBio, gather these inputs (ask if not provided):

1. **Product name** — the title
2. **Tagline** — one-line italic subtitle
3. **Description** — 1-2 sentence overview
4. **Key stats** — 3-5 metrics (number + label)
5. **Problem statements** — 3-6 "why this matters" cards
6. **Architecture / layers** — optional layered architecture diagram
7. **Features / capabilities** — grouped by category with bullet points
8. **Design principles / tenets** — 4-6 guiding principles
9. **CTA text and link** — call to action
10. **Video URL** — optional product tour video
11. **Production URL** — optional live demo link

Then generate a **single self-contained HTML file** (`<product-name>-bio.html`) that:

- Uses the exact CSS variables, fonts, and patterns above
- Includes all styles inline in a `<style>` block (no external CSS)
- Is fully responsive
- Has smooth hover transitions on all interactive elements
- Uses the section-label → section-title → gold-line → content-grid pattern for every section
- Includes the `gate.js` script: `<script src="https://my-solution-registry.satszone.link/gate.js"></script>`
- Outputs to the project root directory

### File Output
Write the file to: `<working-directory>/<product-name-kebab>-bio.html`

### Deployment
After generating, offer to deploy to S3:
```bash
aws s3 cp <file> s3://my-solution-registry.satszone.link/<filename> --content-type "text/html"
aws cloudfront create-invalidation --distribution-id E2R00426B8QGNB --paths "/<filename>"
```
