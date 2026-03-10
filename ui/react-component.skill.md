---
name: react-component
description: "Creates reusable React UI components with typed props, variants, and composition patterns"
category: ui
difficulty: intermediate
tags: [react, typescript, component, reusable, composition]
stack: [react-18, typescript, tailwind, clsx]
---

# React Component Builder

You are an expert React component designer. When creating reusable components:

## File Placement

- `src/components/ui/` — generic (Badge, Modal, Card, StatCard, Tabs)
- `src/components/layout/` — structural (Sidebar, AppLayout, Header)
- `src/components/` — domain-specific (ModelCard, PolicyRow)

## Implementation Pattern

```tsx
import clsx from 'clsx'
import { ReactNode } from 'react'

interface ComponentProps {
  variant?: 'primary' | 'success' | 'warning' | 'error'
  size?: 'sm' | 'md' | 'lg'
  className?: string
  children: ReactNode
}

export default function Component({ variant = 'primary', size = 'md', className, children }: ComponentProps) {
  return (
    <div className={clsx(
      'base-classes',
      { 'variant-a': variant === 'primary', 'variant-b': variant === 'success' },
      { 'text-sm': size === 'sm', 'text-base': size === 'md' },
      className
    )}>
      {children}
    </div>
  )
}
```

## Common Component Patterns

### StatCard (KPI)
```tsx
interface StatCardProps { label: string; value: string | number; icon: LucideIcon; trend?: number }
```

### Modal
```tsx
interface ModalProps { isOpen: boolean; onClose: () => void; title: string; children: ReactNode }
// Use portal, backdrop click to close, Escape key handler
```

### Badge
```tsx
interface BadgeProps { variant: 'active' | 'draft' | 'archived'; children: ReactNode }
```

### Tabs
```tsx
interface TabsProps { tabs: { key: string; label: string; content: ReactNode }[]; defaultTab?: string }
```

## Rules
- Always accept `className` prop for composition override
- Use `clsx` for conditional classes — never template literals
- Props interface defined in same file, exported if shared
- Default variant/size via destructuring defaults
- Never use `any` — type all props explicitly
- Prefer composition (children) over configuration (many props)
- Keep components under 80 lines — extract sub-components if larger
- Use `React.memo` only when measured performance issue exists
- No inline styles — only Tailwind utility classes
- Accessible: use semantic HTML, aria labels, keyboard handlers
