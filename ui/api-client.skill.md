---
name: api-client
description: "Creates typed TypeScript API client with error handling, namespaced methods, and header injection"
category: ui
difficulty: intermediate
tags: [api, fetch, typescript, http-client, error-handling]
stack: [typescript, react-18]
---

# Typed API Client

You are a frontend API integration expert. Build API clients following this pattern:

## File: `src/lib/api.ts`

```tsx
const BASE = '/api'

async function request<T>(method: string, path: string, body?: unknown, headers?: Record<string, string>): Promise<T> {
  const opts: RequestInit = {
    method,
    headers: { 'Content-Type': 'application/json', ...headers },
  }
  if (body) opts.body = JSON.stringify(body)

  const r = await fetch(`${BASE}${path}`, opts)
  if (!r.ok) {
    const err = await r.json().catch(() => ({ detail: r.statusText }))
    throw new Error(err.detail || `API error ${r.status}`)
  }
  return r.json()
}

const get  = <T>(path: string, headers?: Record<string, string>) => request<T>('GET', path, undefined, headers)
const post = <T>(path: string, body: unknown, headers?: Record<string, string>) => request<T>('POST', path, body, headers)
const put  = <T>(path: string, body: unknown, headers?: Record<string, string>) => request<T>('PUT', path, body, headers)
const del  = <T>(path: string, headers?: Record<string, string>) => request<T>('DELETE', path, undefined, headers)
```

## Namespaced API Object

```tsx
export const api = {
  dashboard: {
    summary:  () => get<DashboardSummary>('/dashboard/summary'),
    activity: () => get<ActivityPoint[]>('/dashboard/activity'),
  },
  models: {
    list:   (params?: Record<string, string>) => get<Model[]>('/models?' + new URLSearchParams(params)),
    get:    (id: number) => get<ModelDetail>(`/models/${id}`),
    create: (body: ModelCreate) => post<Model>('/models', body),
    update: (id: number, body: ModelUpdate) => put<Model>(`/models/${id}`, body),
    delete: (id: number) => del<void>(`/models/${id}`),
  },
}
```

## API Key Header Injection

```tsx
// For AI features — key from localStorage context
const withAiKey = (apiKey: string) => ({ 'X-API-Key': apiKey })

export const aiApi = {
  suggest: (prompt: string, apiKey: string) =>
    post<AiSuggestion>('/ai/suggest', { prompt }, withAiKey(apiKey)),
}
```

## Rules
- Every response type has a TypeScript interface
- Error messages extracted from `detail` field (FastAPI convention)
- Base URL uses Vite proxy (`/api` → backend) — never hardcode `localhost`
- File uploads: use `FormData`, omit `Content-Type` header (browser sets boundary)
- Query params via `URLSearchParams` — never manual string building
- Export types alongside api object for page imports
