---
name: session-only-api-key
description: "React Context pattern for session-only AI API key management — key lives in state only, never persisted to localStorage, sessionStorage, cookies, or any server"
category: ui
difficulty: intermediate
tags: [react, context, api-key, security, BYOK, session]
stack: [react, vite]
---

# Session-Only API Key Management

Keeps AI API keys **in memory only** (React state). Key is gone when the tab closes — never written to disk or any browser storage.

## Pattern

```jsx
// APIKeyContext.jsx
import { createContext, useContext, useState, useRef } from 'react'
import APIKeyPrompt from '../components/APIKeyPrompt'

const APIKeyContext = createContext(null)

export function APIKeyProvider({ children }) {
  const [apiKey, setApiKey] = useState('')
  const [provider, setProvider] = useState('Anthropic')
  const [promptOpen, setPromptOpen] = useState(false)
  const [pendingFeature, setPendingFeature] = useState('')
  const resolveRef = useRef(null)

  // Returns a Promise resolving to the key (or null if dismissed)
  const requireKey = (featureName = 'AI features') => {
    if (apiKey) return Promise.resolve(apiKey)
    return new Promise(resolve => {
      resolveRef.current = resolve
      setPendingFeature(featureName)
      setPromptOpen(true)
    })
  }

  const handleSubmit = (key, prov) => {
    setApiKey(key)
    setProvider(prov)
    setPromptOpen(false)
    resolveRef.current?.(key)
  }

  const handleClose = () => {
    setPromptOpen(false)
    resolveRef.current?.(null)
  }

  const clearKey = () => {
    setApiKey('')
    setProvider('Anthropic')
  }

  return (
    <APIKeyContext.Provider value={{ apiKey, provider, requireKey, clearKey, hasKey: !!apiKey }}>
      {children}
      {promptOpen && (
        <APIKeyPrompt
          featureName={pendingFeature}
          onSubmit={handleSubmit}
          onClose={handleClose}
        />
      )}
    </APIKeyContext.Provider>
  )
}

export function useAPIKey() {
  return useContext(APIKeyContext)
}
```

## Usage in a Feature

```jsx
const { requireKey } = useAPIKey()

async function handleAIFeature() {
  const key = await requireKey('Semantic suggestions')
  if (!key) return  // user dismissed the prompt

  const res = await fetch('/api/ai/suggest', {
    headers: { 'X-AI-Key': key }
  })
  // ...
}
```

## Prompt Component (APIKeyPrompt.jsx)

Full-screen modal with:
- Security warning banner (session-only, never stored)
- Provider selector (Anthropic / OpenAI / Google)
- Masked password input with show/hide toggle
- Link to get API key for each provider

## Backend — Never Store the Key

```python
# FastAPI — receive key per-request, never persist
@router.post("/ai/suggest")
async def ai_suggest(
    request: SuggestRequest,
    x_ai_key: str | None = Header(default=None)
):
    if not x_ai_key:
        return mock_response()  # graceful fallback

    client = anthropic.AsyncAnthropic(api_key=x_ai_key)
    # use and discard — never write to DB or logs
```

## Security Properties

- Key never touches `localStorage`, `sessionStorage`, cookies, or IndexedDB
- Key never written to server database or logs
- Tab close = key gone (browser GC clears React state)
- No server-side session storage
- Mock fallback means platform works without any key
