---
name: protocol-deep-dive-content
description: "Structure protocol and reference specification content for learning apps with papers, key takeaways, and curated references"
category: backend
difficulty: intermediate
tags: [papers, specifications, references, learning, content-structure]
stack: [FastAPI, Python]
---

# Protocol & Reference Content Structuring

You are a content architect structuring protocol specifications, whitepapers, and reference documents for a learning platform. This pattern is used in the academy apps for curating papers and specifications.

## Paper Data Structure

```python
PAPERS = [
    {
        "id": 1,
        "title": "Model Context Protocol Specification",
        "authors": "Anthropic",
        "year": 2024,
        "category": "Context Protocols",
        "url": "https://spec.modelcontextprotocol.io/",
        "summary": "One-paragraph summary of what this spec covers and why it matters.",
        "key_takeaways": [
            "JSON-RPC 2.0 over stdio/SSE transport",
            "Three primitives: tools, resources, prompts",
            "Host-client-server architecture",
            "Capability negotiation during initialization",
            "Human-in-the-loop for sensitive operations"
        ],
        "related_topics": ["A2A Protocol", "JSON-RPC", "Tool Use"]
    }
]
```

## Papers Router

```python
from fastapi import APIRouter, HTTPException

router = APIRouter(prefix="/api/papers", tags=["papers"])

@router.get("/")
def list_papers():
    return PAPERS

@router.get("/{paper_id}")
def get_paper(paper_id: int):
    paper = next((p for p in PAPERS if p["id"] == paper_id), None)
    if not paper:
        raise HTTPException(404, "Paper not found")
    return paper
```

## Content Curation Guidelines

### Key Takeaways
- 4-6 bullet points per paper
- Each takeaway should be a concrete, memorable fact (not vague)
- Use specific terminology from the spec
- Include version numbers, protocol names, transport mechanisms

### Categories
Group papers by domain area. Examples:
- **Context Protocols**: MCP, A2A, tool-use specs
- **Security**: OAuth, trust frameworks, OWASP guides
- **Architecture**: Well-Architected Frameworks, CAF, landing zone guides
- **RPC/Transport**: JSON-RPC, gRPC, REST standards

### Related Topics
- Cross-reference to other papers and lesson categories
- Enable learners to follow connected learning paths

## Frontend Rendering Pattern

```jsx
// Papers page with expandable cards
{papers.map(paper => (
  <div key={paper.id} className="bg-gray-800 rounded-lg p-6">
    <h3 className="text-lg font-semibold">{paper.title}</h3>
    <div className="text-sm text-gray-400">{paper.authors} ({paper.year})</div>
    <p className="mt-2 text-gray-300">{paper.summary}</p>
    <div className="mt-3">
      <h4 className="text-sm font-medium">Key Takeaways</h4>
      <ul className="list-disc list-inside text-sm text-gray-300">
        {paper.key_takeaways.map((t, i) => <li key={i}>{t}</li>)}
      </ul>
    </div>
    <div className="mt-2 flex gap-2 flex-wrap">
      {paper.related_topics.map(t => (
        <span key={t} className="px-2 py-0.5 bg-gray-700 rounded text-xs">{t}</span>
      ))}
    </div>
    <a href={paper.url} target="_blank" className="text-blue-400 text-sm mt-2 inline-block">
      Read Original →
    </a>
  </div>
))}
```

## Content Volume Guidelines

| App Size | Papers | Lessons | Quiz Questions |
|----------|--------|---------|----------------|
| Small    | 8-12   | 20-28   | 10-15          |
| Medium   | 12-16  | 36-42   | 15-20          |
| Large    | 16-24  | 48-60   | 20-30          |
