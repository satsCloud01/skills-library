---
name: react-page
description: "Creates production-ready React page components with data fetching, loading states, and responsive layout"
category: ui
difficulty: intermediate
tags: [react, typescript, page, routing, data-fetching]
stack: [react-18, typescript, vite, tailwind, react-router-dom]
---

# React Page Builder

You are an expert React developer. When creating a new page component:

## Structure

```
src/pages/PageName.tsx
```

## Implementation Pattern

1. **Imports**: React hooks, API client, UI components, icons (lucide-react)
2. **State**: Typed state with `useState<T | null>(null)` + `loading` + `error`
3. **Data fetch**: `useEffect` with async IIFE, `.finally(() => setLoading(false))`
4. **Loading**: Skeleton or spinner, never blank screen
5. **Error**: Inline error banner with retry action
6. **Layout**: `<div className="space-y-6">` wrapper, `page-title` heading

## Code Template

```tsx
import { useEffect, useState } from 'react'
import { api } from '../lib/api'
import { Icon } from 'lucide-react'

interface PageData { /* typed shape */ }

export default function PageName() {
  const [data, setData] = useState<PageData | null>(null)
  const [loading, setLoading] = useState(true)
  const [error, setError] = useState('')

  useEffect(() => {
    setLoading(true)
    api.feature.list()
      .then(setData)
      .catch(e => setError(e.message))
      .finally(() => setLoading(false))
  }, [])

  if (loading) return <div className="p-6 text-slate-400">Loading…</div>
  if (error) return <div className="p-6 text-red-400">{error}</div>

  return (
    <div className="space-y-6">
      <h1 className="page-title"><Icon className="w-6 h-6" /> Page Title</h1>
      {/* KPI cards: grid grid-cols-2 lg:grid-cols-4 gap-4 */}
      {/* Data table or content */}
    </div>
  )
}
```

## Rules
- Always type API responses with TypeScript interfaces
- Use `clsx` for conditional classes, never string concatenation
- Wrap long lists in responsive grids: `grid-cols-1 md:grid-cols-2 lg:grid-cols-3`
- Use lucide-react icons, never emoji in UI
- Register route in App.tsx inside `<AppLayout>` wrapper
- Add to Sidebar NAV array with icon and label
- Never fetch inside render — always in useEffect or event handlers
- Handle empty state: show placeholder when data array is empty
