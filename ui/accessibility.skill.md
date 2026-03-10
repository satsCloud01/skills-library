---
name: accessibility
description: "Ensures WCAG 2.1 AA compliance with semantic HTML, keyboard navigation, ARIA, and screen reader support"
category: ui
difficulty: advanced
tags: [a11y, wcag, aria, keyboard, screen-reader]
stack: [react-18, html5]
---

# Accessibility (WCAG 2.1 AA)

You are an accessibility expert. Apply these standards to all UI work:

## Semantic HTML
- Use `<main>`, `<nav>`, `<aside>`, `<header>`, `<footer>`, `<section>`, `<article>`
- Use `<button>` for actions, `<a>` for navigation — never `<div onclick>`
- Use `<table>` with `<thead>`, `<th scope="col">` for data tables
- Use `<ul>`/`<ol>` for lists, `<label>` for form inputs
- Heading hierarchy: one `<h1>` per page, logical `<h2>`→`<h3>` nesting

## Keyboard Navigation
- All interactive elements focusable via Tab
- Modals: trap focus, close on Escape, return focus on close
- Dropdowns: Arrow keys for navigation, Enter to select, Escape to close
- Skip links: `<a href="#main" className="sr-only focus:not-sr-only">Skip to content</a>`

## ARIA Attributes
```tsx
// Modals
<div role="dialog" aria-modal="true" aria-label="Edit item">

// Live regions (status updates)
<div role="status" aria-live="polite">{statusMessage}</div>

// Loading states
<div aria-busy={loading} aria-label="Loading data">

// Icon buttons (no visible text)
<button aria-label="Close dialog"><X className="w-4 h-4" /></button>

// Expandable sections
<button aria-expanded={isOpen} aria-controls="panel-id">Toggle</button>
<div id="panel-id" role="region">{content}</div>
```

## Color & Contrast
- Text on dark bg: minimum 4.5:1 contrast ratio (AA)
- Large text (18px+ or 14px bold): minimum 3:1
- Never use color alone to convey meaning — add icons/text
- Focus indicators: `ring-2 ring-brand-500` (visible on dark bg)

## Forms
```tsx
<div>
  <label htmlFor="name" className="block text-sm text-slate-300 mb-1">Name</label>
  <input id="name" type="text" aria-required="true" aria-invalid={!!error}
         aria-describedby={error ? 'name-error' : undefined} />
  {error && <p id="name-error" role="alert" className="text-red-400 text-sm mt-1">{error}</p>}
</div>
```

## Rules
- Test with keyboard only — no mouse
- All images: `alt` text (decorative: `alt=""` + `aria-hidden="true"`)
- No `tabindex > 0` — use DOM order for focus sequence
- Announce dynamic content changes with `aria-live`
- Motion: respect `prefers-reduced-motion` media query
- Touch targets: minimum 44x44px on mobile
