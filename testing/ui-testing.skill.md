---
name: ui-testing
description: "Frontend testing with Vitest, React Testing Library, component rendering, user interactions, and snapshot tests"
category: testing
difficulty: intermediate
tags: [vitest, react-testing-library, component-testing, snapshot, jest]
stack: [react-18, typescript, vitest, testing-library]
---

# UI Testing

You are a frontend testing expert. When writing UI tests:

## Setup

```bash
npm install -D vitest @testing-library/react @testing-library/jest-dom @testing-library/user-event jsdom
```

```typescript
// vite.config.ts
export default defineConfig({
  test: {
    environment: 'jsdom',
    globals: true,
    setupFiles: './src/test/setup.ts',
  },
})

// src/test/setup.ts
import '@testing-library/jest-dom'
```

## Component Test Pattern

```tsx
import { render, screen, waitFor } from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import { BrowserRouter } from 'react-router-dom'
import Dashboard from '../pages/Dashboard'

// Wrapper for components that need router
function renderWithRouter(ui: React.ReactElement) {
  return render(<BrowserRouter>{ui}</BrowserRouter>)
}

describe('Dashboard', () => {
  beforeEach(() => {
    vi.restoreAllMocks()
  })

  it('renders loading state', () => {
    renderWithRouter(<Dashboard />)
    expect(screen.getByText(/loading/i)).toBeInTheDocument()
  })

  it('renders data after fetch', async () => {
    vi.spyOn(global, 'fetch').mockResolvedValueOnce({
      ok: true,
      json: () => Promise.resolve({ total: 42 }),
    } as Response)

    renderWithRouter(<Dashboard />)
    await waitFor(() => {
      expect(screen.getByText('42')).toBeInTheDocument()
    })
  })

  it('handles user interaction', async () => {
    const user = userEvent.setup()
    renderWithRouter(<Dashboard />)

    const button = screen.getByRole('button', { name: /filter/i })
    await user.click(button)
    expect(screen.getByRole('dialog')).toBeInTheDocument()
  })
})
```

## Test Categories

1. **Render tests**: Component mounts without crashing
2. **Content tests**: Text, headings, labels are present
3. **Interaction tests**: Click, type, select — verify state changes
4. **API integration**: Mock fetch, verify loading → data flow
5. **Error states**: Mock failed fetch, verify error UI
6. **Accessibility**: Check roles, labels, keyboard navigation

## Rules
- Use `screen.getByRole` over `getByTestId` — tests should mirror user behavior
- Use `userEvent` over `fireEvent` — more realistic interactions
- Mock `fetch` not the API module — tests the full chain
- `waitFor` for async state updates — never `setTimeout`
- One assertion per test ideal, but related assertions OK
- Run: `npx vitest run` (CI) or `npx vitest` (watch mode)
