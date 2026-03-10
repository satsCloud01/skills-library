---
name: websocket
description: "WebSocket real-time communication with FastAPI, connection management, and React hooks"
category: integrations
difficulty: advanced
tags: [websocket, real-time, fastapi, streaming, bidirectional]
stack: [python-3.12, fastapi, react-18]
---

# WebSocket Integration

You are a real-time communication expert.

## FastAPI WebSocket Endpoint

```python
from fastapi import WebSocket, WebSocketDisconnect
from typing import dict

class ConnectionManager:
    def __init__(self):
        self.connections: dict[str, WebSocket] = {}

    async def connect(self, ws: WebSocket, client_id: str):
        await ws.accept()
        self.connections[client_id] = ws

    def disconnect(self, client_id: str):
        self.connections.pop(client_id, None)

    async def send(self, client_id: str, data: dict):
        ws = self.connections.get(client_id)
        if ws:
            await ws.send_json(data)

    async def broadcast(self, data: dict):
        for ws in self.connections.values():
            await ws.send_json(data)

manager = ConnectionManager()

@app.websocket("/ws/{client_id}")
async def websocket_endpoint(ws: WebSocket, client_id: str):
    await manager.connect(ws, client_id)
    try:
        while True:
            data = await ws.receive_json()
            # Process message
            await manager.broadcast({"from": client_id, **data})
    except WebSocketDisconnect:
        manager.disconnect(client_id)
```

## React WebSocket Hook

```tsx
function useWebSocket(url: string) {
  const [messages, setMessages] = useState<any[]>([])
  const [connected, setConnected] = useState(false)
  const wsRef = useRef<WebSocket | null>(null)

  useEffect(() => {
    const ws = new WebSocket(url)
    wsRef.current = ws

    ws.onopen = () => setConnected(true)
    ws.onclose = () => { setConnected(false); /* reconnect logic */ }
    ws.onmessage = (e) => setMessages(prev => [...prev, JSON.parse(e.data)])

    return () => ws.close()
  }, [url])

  const send = useCallback((data: any) => {
    wsRef.current?.send(JSON.stringify(data))
  }, [])

  return { messages, connected, send }
}
```

## Rules
- Always handle `WebSocketDisconnect` — clients drop unexpectedly
- Use JSON for message format — not raw text
- Implement reconnection with exponential backoff on client
- Heartbeat/ping every 30s to detect dead connections
- Clean up connections on disconnect — prevent memory leaks
- Use unique client IDs (UUID) not user-facing identifiers
