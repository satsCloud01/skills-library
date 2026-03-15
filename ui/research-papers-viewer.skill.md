---
name: research-papers-viewer
description: Split-panel research papers viewer with category filters, metadata cards, and embedded iframe reader
category: ui
tags: [react, papers, viewer, iframe, split-panel, research]
---

# Research Papers Viewer

A split-panel UI component for browsing and reading research papers, specifications, and technical references. Left panel shows a filterable list of papers; right panel embeds the paper in an iframe with metadata header, key insight card, and "Open Original" fallback.

## When to Use

- Learning platforms with curated reference materials
- Documentation hubs with external specs/papers
- Any app that needs to display a list of linked resources with inline previewing

## Page Structure

```
┌─────────────────────────────────────────────────┐
│  📄 Research Papers & References                │
│  [All] [Category A] [Category B] [Category C]  │
├──────────────────────┬──────────────────────────┤
│ Paper List (420px)   │ Reader Pane (flex-1)     │
│                      │ ┌────────────────────┐   │
│ ┌──────────────────┐ │ │ Category · Year    │   │
│ │ [Cat] · 2024     │ │ │ Title              │   │
│ │ Paper Title      │ │ │ Authors            │   │
│ │ Authors          │ │ │ [Open Original ↗]  │   │
│ │ Summary...       │ │ ├────────────────────┤   │
│ └──────────────────┘ │ │ Key Insight        │   │
│                      │ ├────────────────────┤   │
│ ┌──────────────────┐ │ │ Summary            │   │
│ │ [Cat] · 2023     │ │ ├────────────────────┤   │
│ │ Another Paper    │ │ │                    │   │
│ │ Authors          │ │ │   <iframe>         │   │
│ │ Summary...       │ │ │   embedded reader  │   │
│ └──────────────────┘ │ │                    │   │
│                      │ └────────────────────┘   │
└──────────────────────┴──────────────────────────┘
```

## Backend Pattern

```python
# Router: papers.py
PAPERS = [
    {
        "id": "unique-slug",
        "title": "Paper Title",
        "authors": "Author et al. (Organization)",
        "year": 2024,
        "category": "Category Name",
        "url": "https://arxiv.org/abs/...",  # External URL for iframe + fallback
        "summary": "2-3 sentence summary of the paper.",
        "keyInsight": "One-sentence takeaway that captures the paper's core contribution.",
    },
    # ... 15-20 papers recommended
]

@router.get("/")
async def list_papers():
    return [{"id": p["id"], "title": p["title"], "authors": p["authors"],
             "year": p["year"], "category": p["category"], "summary": p["summary"]}
            for p in PAPERS]

@router.get("/{paper_id}")
async def get_paper(paper_id: str):
    for p in PAPERS:
        if p["id"] == paper_id:
            return p
    return {"error": "Not found"}
```

## Frontend Pattern

```jsx
// Key behaviors:
// 1. Left panel width shrinks from flex-1 to 420px when reader opens
// 2. Category filter pills with active state
// 3. Selected paper highlighted with ring + accent background
// 4. Reader pane: header → key insight → summary → iframe
// 5. iframe sandbox="allow-scripts allow-same-origin allow-popups"
// 6. Fallback message at bottom: "click Open Original if iframe doesn't load"

const CAT_COLORS = {
  'Category A': 'bg-blue-100 text-blue-700',
  'Category B': 'bg-purple-100 text-purple-700',
  // ... one color per category
};

export default function Papers() {
  const [papers, setPapers] = useState([]);
  const [selected, setSelected] = useState(null);
  const [detail, setDetail] = useState(null);
  const [filter, setFilter] = useState('All');
  const [readerUrl, setReaderUrl] = useState(null);

  // List shrinks when reader is open
  // <div className={`${readerUrl && selected ? 'w-[420px]' : 'flex-1 max-w-5xl mx-auto'} ...`}>

  // Iframe with fallback
  // <iframe src={readerUrl} sandbox="allow-scripts allow-same-origin allow-popups" />
  // <div className="absolute bottom-4 ...">If paper doesn't load, click Open Original</div>
}
```

## Content Guidelines

| Aspect | Recommendation |
|--------|----------------|
| Papers count | 15-20 curated references |
| Categories | 4-6 distinct categories |
| Summary length | 2-3 sentences |
| Key insight | 1 sentence, the "so what?" |
| URL types | arXiv, official specs, blog posts, GitHub repos |
| Year range | Include foundational + recent (e.g., 2020-2025) |

## Styling Rules

- Category badges: colored pills with `text-[10px] px-2 py-0.5 rounded-full`
- Selected card: `border-accent-500 bg-accent-50 ring-2 ring-accent-200`
- Key insight card: accent background (`bg-accent-50`)
- Reader panel: `flex-1 border-l flex flex-col bg-white`
- Close button + "Open Original" in header
