---
name: Protocol Message Format Viewer
description: Side-by-side comparison UI for viewing protocol message formats (MCP, A2A, OpenAI, Anthropic)
category: ui
difficulty: intermediate
tags: [protocols, comparison, message-formats, mcp, a2a]
stack: [React, Tailwind CSS]
---

# Protocol Message Format Viewer

Build a tabbed side-by-side viewer for comparing protocol message formats with syntax-highlighted JSON and contextual annotations.

## Pattern Overview

When building educational or reference tools for AI protocols (MCP, A2A, OpenAI function calling, Anthropic tool use), users need to compare message structures across protocols. This pattern provides a tabbed panel where each tab shows a protocol's message format in syntax-highlighted JSON, with inline annotations explaining each field.

## Architecture

### State Shape

```jsx
const [selectedProtocols, setSelectedProtocols] = useState(['mcp', 'a2a']);
const [activeMessage, setActiveMessage] = useState('request'); // 'request' | 'response'
const [showAnnotations, setShowAnnotations] = useState(true);
```

### Protocol Data Structure

Define each protocol's message examples as structured data. Each protocol has request and response examples with field-level annotations.

```jsx
const PROTOCOLS = {
  mcp: {
    name: 'Model Context Protocol',
    shortName: 'MCP',
    color: 'indigo',
    request: {
      json: {
        jsonrpc: "2.0",
        method: "tools/call",
        params: {
          name: "web_search",
          arguments: { query: "Tokyo population 2026" }
        },
        id: 1
      },
      annotations: {
        jsonrpc: "JSON-RPC 2.0 envelope",
        method: "MCP method — tools/call invokes a registered tool",
        "params.name": "Tool name as registered in the server's tool list",
        "params.arguments": "Tool-specific parameters passed by the LLM",
      }
    },
    response: {
      json: {
        jsonrpc: "2.0",
        result: {
          content: [{ type: "text", text: "Tokyo population: 13.96 million" }]
        },
        id: 1
      },
      annotations: {
        "result.content": "Array of content blocks — supports text, image, resource types",
      }
    }
  },
  a2a: {
    name: 'Agent-to-Agent Protocol',
    shortName: 'A2A',
    color: 'emerald',
    request: {
      json: {
        jsonrpc: "2.0",
        method: "tasks/send",
        params: {
          id: "task-123",
          message: {
            role: "user",
            parts: [{ type: "text", text: "Find Tokyo population" }]
          }
        }
      },
      annotations: {
        method: "A2A method — tasks/send creates or continues a task",
        "params.id": "Task identifier for multi-turn conversations",
        "params.message.parts": "Multi-part message supporting text, file, and data parts",
      }
    },
    response: { /* ... */ }
  },
  // openai, anthropic...
};
```

### Side-by-Side Layout

The viewer renders two protocol panels side by side. Each panel has a tab bar for switching protocols and a toggle for request/response view.

```jsx
function ProtocolViewer() {
  return (
    <div className="flex gap-4">
      {selectedProtocols.map((proto, idx) => (
        <div key={proto} className="flex-1 border rounded-xl overflow-hidden">
          {/* Protocol tab bar */}
          <div className="flex bg-gray-50 border-b">
            {Object.entries(PROTOCOLS).map(([key, p]) => (
              <button
                key={key}
                onClick={() => {
                  const next = [...selectedProtocols];
                  next[idx] = key;
                  setSelectedProtocols(next);
                }}
                className={`px-4 py-2 text-sm font-medium ${
                  proto === key
                    ? `bg-${p.color}-100 text-${p.color}-800 border-b-2 border-${p.color}-500`
                    : 'text-gray-500 hover:text-gray-700'
                }`}
              >
                {p.shortName}
              </button>
            ))}
          </div>

          {/* JSON content */}
          <AnnotatedJson
            data={PROTOCOLS[proto][activeMessage]}
            color={PROTOCOLS[proto].color}
            showAnnotations={showAnnotations}
          />
        </div>
      ))}
    </div>
  );
}
```

### Syntax-Highlighted JSON with Annotations

Render JSON with color-coded syntax and optional inline annotations that explain each field.

```jsx
function AnnotatedJson({ data, color, showAnnotations }) {
  const jsonStr = JSON.stringify(data.json, null, 2);
  const lines = jsonStr.split('\n');

  return (
    <div className="p-4 font-mono text-sm bg-gray-900 text-gray-100 overflow-x-auto">
      {lines.map((line, i) => {
        // Extract field name from the line for annotation lookup
        const fieldMatch = line.match(/"(\w+)":/);
        const fieldName = fieldMatch ? fieldMatch[1] : null;
        const annotation = fieldName && showAnnotations
          ? data.annotations[fieldName]
          : null;

        return (
          <div key={i} className="flex group">
            <span className="text-gray-500 w-8 text-right mr-4 select-none">{i + 1}</span>
            <span className="flex-1">
              <SyntaxLine content={line} />
            </span>
            {annotation && (
              <span className={`ml-4 text-${color}-400 text-xs italic opacity-0 group-hover:opacity-100 transition-opacity`}>
                {annotation}
              </span>
            )}
          </div>
        );
      })}
    </div>
  );
}

function SyntaxLine({ content }) {
  // Apply basic JSON syntax highlighting
  const highlighted = content
    .replace(/"([^"]+)"(?=:)/g, '<span class="text-blue-300">"$1"</span>')    // keys
    .replace(/: "([^"]+)"/g, ': <span class="text-green-300">"$1"</span>')     // string values
    .replace(/: (\d+)/g, ': <span class="text-amber-300">$1</span>')          // numbers
    .replace(/: (true|false|null)/g, ': <span class="text-purple-300">$1</span>'); // literals

  return <span dangerouslySetInnerHTML={{ __html: highlighted }} />;
}
```

### Request/Response Toggle

A simple toggle button switches between request and response views for both panels simultaneously.

```jsx
<div className="flex justify-center gap-2 mb-4">
  {['request', 'response'].map(mode => (
    <button
      key={mode}
      onClick={() => setActiveMessage(mode)}
      className={`px-4 py-2 rounded-lg text-sm font-medium capitalize ${
        activeMessage === mode
          ? 'bg-indigo-600 text-white'
          : 'bg-gray-100 text-gray-600 hover:bg-gray-200'
      }`}
    >
      {mode}
    </button>
  ))}
</div>
```

## Key Design Decisions

- **Static data, no API needed**: Protocol examples are defined in the frontend as constants. No backend call is required since the message formats are fixed reference material.
- **Two-panel comparison**: Users always compare exactly two protocols side by side. This avoids complex multi-panel layouts while covering the most common comparison use case.
- **Hover annotations**: Annotations appear on hover to keep the view clean by default, but can be toggled always-on for study mode.
- **Dark background for JSON**: JSON panels use a dark theme (`bg-gray-900`) for readability and to visually distinguish code from surrounding UI.

## When to Use This Pattern

- Protocol reference documentation (MCP, A2A, OpenAI, Anthropic APIs)
- API format comparison tools
- Educational platforms teaching message serialization formats
- Developer tools comparing JSON schemas or API contracts
