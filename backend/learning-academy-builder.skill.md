---
name: learning-academy-builder
description: "Build a self-contained learning academy app with FastAPI + React, static lesson data, server-side quiz scoring, and localStorage progress tracking"
category: backend
difficulty: intermediate
tags: [academy, learning, lessons, quiz, static-data, education]
stack: [FastAPI, React, Tailwind, Recharts, Vite]
---

# Learning Academy Builder

You are a full-stack engineer building an interactive learning academy. Follow this proven pattern used across Agentic AI Academy, AWS Academy, Azure Academy, and Cloud Comparison Academy.

## Architecture Pattern

```
backend/src/<package>/
├── main.py                    # FastAPI app, CORS, router registration, health endpoint
├── data/
│   └── lessons.py             # LESSONS list + QUIZ_QUESTIONS list (Python dicts)
└── routers/
    ├── lessons.py             # GET /lessons/, GET /lessons/{id}, GET /categories/summary
    ├── quiz.py                # GET /questions (no answers), POST /check (server-side scoring)
    └── papers.py              # GET /papers/, GET /papers/{id}

frontend/src/
├── App.jsx                    # React Router v6
├── components/
│   ├── layout/Sidebar.jsx     # Fixed left nav
│   └── ui/TourOverlay.jsx     # Guided tour modal
├── hooks/useTour.js           # Tour state with localStorage
├── lib/api.js                 # Fetch wrapper
└── pages/
    ├── Landing.jsx            # Hero, stats, curriculum grid, tour trigger
    ├── Learn.jsx              # Split layout: category nav + lesson viewer + progress
    ├── Quiz.jsx               # Questions, per-category scoring, Recharts bar chart
    ├── Papers.jsx             # Reference docs with key_takeaways
    └── Settings.jsx           # Status, progress reset
```

## Key Design Decisions

1. **No database** -- All content as Python dicts in `data/lessons.py`. No SQLite, no ORM, no migrations.
2. **No AI API keys** -- Zero `.env` dependencies. 100% offline-capable after page load.
3. **No authentication** -- Public learning platform.
4. **No server-side persistence** -- Progress in browser localStorage only.
5. **Server-side quiz scoring** -- Never send correct answers to the client.

## Lesson Data Structure

```python
LESSONS = [
    {
        "id": 1,
        "category": "Category Name",
        "title": "Lesson Title",
        "type": "intro",  # intro | concept | summary | reference
        "description": "Main content paragraph...",
        "keyPoints": ["Point 1...", "Point 2..."],
        "comparison": {  # optional
            "title": "Comparison Title",
            "headers": ["Col1", "Col2", "Col3"],
            "rows": [["val1", "val2", "val3"]]
        },
        "diagram": {  # optional
            "type": "architecture",
            "nodes": [{"id": "n1", "label": "Service", "x": 200, "y": 100}],
            "edges": [{"from": "n1", "to": "n2", "label": "calls"}]
        },
        "code": "example code here",  # optional
        "codeLanguage": "python"       # optional
    }
]
```

## Quiz Router Pattern

```python
from fastapi import APIRouter, HTTPException
from pydantic import BaseModel

router = APIRouter(prefix="/api/quiz", tags=["quiz"])

class CheckAnswer(BaseModel):
    question_id: int
    selected: int

@router.get("/questions")
def get_questions():
    """Return questions WITHOUT correct answers."""
    return [
        {"id": q["id"], "category": q["category"],
         "question": q["question"], "options": q["options"]}
        for q in QUIZ_QUESTIONS
    ]

@router.post("/check")
def check_answer(payload: CheckAnswer):
    """Server-side scoring -- correct answer never sent to client upfront."""
    q = next((q for q in QUIZ_QUESTIONS if q["id"] == payload.question_id), None)
    if not q:
        raise HTTPException(404, "Question not found")
    return {
        "correct": payload.selected == q["correct"],
        "correct_index": q["correct"],
        "explanation": q["explanation"]
    }
```

## Frontend Progress Tracking

```javascript
// localStorage key pattern: "<app-slug>-completed"
const STORAGE_KEY = "my-academy-completed";

const markComplete = (lessonId) => {
  const completed = JSON.parse(localStorage.getItem(STORAGE_KEY) || "[]");
  if (!completed.includes(lessonId)) {
    completed.push(lessonId);
    localStorage.setItem(STORAGE_KEY, JSON.stringify(completed));
  }
};
```

## Port Convention

Assign unique ports in the Sats port range:
- Backend: 8030+ (e.g., 8030, 8032, 8034, 8036)
- Frontend: 5190+ (e.g., 5190, 5191, 5192, 5193)
- Docker: backend_port + 1

## Content Rendering

The Learn page should render all content types from each lesson:
- `description` as intro paragraph
- `keyPoints` as styled bullet list
- `comparison` as a responsive table
- `diagram` as a flow/architecture visualization
- `code` in a syntax-highlighted code block with `codeLanguage` hint
