---
name: styling-tailwind
description: "Applies consistent dark-theme Tailwind CSS styling with brand colors, responsive grids, and custom utility classes"
category: ui
difficulty: beginner
tags: [tailwind, css, dark-theme, responsive, styling]
stack: [tailwind-3.4, postcss, vite]
---

# Tailwind Styling System

You are a UI styling expert. Apply these established conventions:

## Color System (Dark Theme)

| Token | Class | Usage |
|-------|-------|-------|
| Background | `bg-slate-950` | Page background |
| Surface | `bg-slate-900` | Cards, panels |
| Border | `border-slate-800` | Card borders, dividers |
| Text primary | `text-slate-200` | Body text |
| Text muted | `text-slate-400` | Labels, secondary |
| Brand accent | `text-brand-400` / `bg-brand-600` | CTAs, active states |
| Success | `text-emerald-400` / `bg-emerald-900/30` | Status badges |
| Warning | `text-amber-400` / `bg-amber-900/30` | Alerts |
| Error | `text-red-400` / `bg-red-900/30` | Errors |

## Brand Colors (tailwind.config.js)

```js
colors: {
  brand: {
    50: '#eef2ff', 100: '#e0e7ff', 300: '#a5b4fc',
    400: '#818cf8', 600: '#4f46e5', 700: '#4338ca',
    800: '#3730a3', 900: '#312e81',
  }
}
```

## Custom Utility Classes (index.css)

```css
.card        { @apply bg-slate-900 border border-slate-800 rounded-lg p-4; }
.btn-primary { @apply bg-brand-600 hover:bg-brand-700 text-white px-4 py-2 rounded-lg
                      transition-colors font-medium text-sm; }
.btn-ghost   { @apply text-slate-400 hover:text-slate-200 hover:bg-slate-800 px-3 py-2
                      rounded-lg transition-colors text-sm; }
.page-title  { @apply text-2xl font-bold flex items-center gap-2; }
.section-title { @apply text-lg font-semibold flex items-center gap-2 text-slate-200; }
```

## Layout Patterns

- **Page wrapper**: `<div className="space-y-6">`
- **KPI row**: `grid grid-cols-2 lg:grid-cols-4 gap-4`
- **Content grid**: `grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4`
- **Sidebar layout**: `flex min-h-screen` → sidebar `w-60 fixed` + main `ml-60 flex-1`
- **Max width**: `max-w-screen-xl mx-auto`

## Interactive States

```
hover:bg-slate-800/60    — subtle row/card hover
focus:ring-2 focus:ring-brand-500 focus:outline-none  — focus rings
transition-colors duration-150  — smooth state changes
cursor-pointer  — clickable elements
```

## Rules
- Never use inline `style={}` — only Tailwind classes
- Dark theme only — no light mode toggle needed
- Use `gap-*` not margins between flex/grid children
- Responsive: mobile-first, add `md:` `lg:` breakpoints
- Font: Inter via `font-sans` (configured in tailwind.config.js)
- Border radius: `rounded-lg` for cards, `rounded` for buttons, `rounded-full` for badges
- Shadows: avoid — use borders for elevation in dark themes
