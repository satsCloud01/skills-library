---
name: learning-academy
description: "Build a complete Learning Academy app — structured lessons, interactive playgrounds with live code execution, agent simulator, design patterns gallery, quizzes, research papers, glossary, hands-on labs, and architecture diagrams"
category: backend
difficulty: advanced
tags: [academy, learning, playground, lessons, quiz, simulator, patterns, papers, labs, glossary, duckdb, agentic-ai]
stack: [FastAPI, React, Vite, Tailwind CSS, Recharts, DuckDB, PyArrow, Python 3.12]
---

# Learning Academy — Full-Stack Builder

You are building a **Learning Academy** application — a self-contained, full-stack educational platform that teaches any technical topic through structured lessons, real interactive playgrounds, simulations, quizzes, and curated research papers. This skill produces production-ready apps matching the SatsLearning Academy and Duck Analytics reference implementations.

## What You Will Build

A FastAPI + React application with **up to 15 pages** and **up to 12 backend routers**, combining:

1. **Structured Learning** — Multi-step lesson wizard with category sidebar, progress tracking, and mixed content types (text, code, diagrams, comparisons, sequence diagrams, state machines)
2. **Real Playgrounds** — Live code execution environments (SQL playground, code sandbox, API tester) where users run actual queries/code against a real engine (DuckDB, Python, etc.)
3. **Agent Simulator** — Step-by-step visualization of agentic loops (ReAct: Thought → Action → Observation → Answer) with multiple scenarios
4. **Design Patterns Gallery** — Visual catalog of design patterns with flow diagrams, complexity ratings, and when-to-use guidance
5. **Knowledge Quiz** — Multiple-choice assessment with instant feedback, explanations, and score breakdown by category
6. **Research Papers Library** — Curated papers/specs with metadata, key insights, and optional embedded reader
7. **Hands-On Labs** — Guided exercises with starter code, progressive hints, and full solutions
8. **Glossary** — Searchable term dictionary with related-term cross-references
9. **Architecture Diagrams** — Interactive node-edge diagrams showing how concepts connect
10. **Performance Benchmarks** — Real benchmarking tools comparing approaches (e.g., Arrow IPC vs JSON)
11. **Data Pipeline Visualizer** — Upload → Ingest → Transform → Analyze with stage-by-stage animation
12. **Dashboard** — Progress overview with completion stats, quiz accuracy, and per-topic breakdown

---

## Architecture

```
┌──────────────────────────────────────────────────────────┐
│                    React 18 + Vite                       │
│  Landing │ Learn │ Playground │ Simulator │ Patterns     │
│  Quiz │ Papers │ Labs │ Glossary │ Architecture │ Bench  │
│  Dashboard │ Pipeline │ Settings                         │
└────────────────────────┬─────────────────────────────────┘
                         │ /api/*
┌────────────────────────┴─────────────────────────────────┐
│                  FastAPI (Python 3.12)                    │
│  routers: lessons, simulator, quiz, papers, playground,  │
│  labs, glossary, architecture, patterns, bench, pipeline,│
│  settings                                                │
│                                                          │
│  services: playground_service, bench_service,            │
│  pipeline_service (optional per topic)                   │
│                                                          │
│  data: lessons.py (all curriculum content)               │
│  db: SQLite or DuckDB (progress, history, benchmarks)    │
└──────────────────────────────────────────────────────────┘
```

### Two Content Strategies

**Strategy A — Static Content (Agentic AI Academy pattern)**
- All lessons, quizzes, papers, patterns stored in Python dicts/lists
- No database (or SQLite for progress tracking only)
- Zero external API calls at runtime
- Best for: conceptual topics (Agentic AI, cloud architecture, protocols)

**Strategy B — Live Engine (Duck Analytics pattern)**
- Real query engine (DuckDB, SQLite, or Python sandbox) for playground
- Database stores query history, benchmark results, user snippets
- Demo data auto-seeded on startup
- Best for: data/SQL topics, coding topics, anything requiring live execution

**Choose based on the topic.** Many academies combine both — static lessons + a live playground.

---

## Project Structure

```
{project-name}/
├── start.sh                    # Launch both servers
├── backend/
│   ├── requirements.txt
│   ├── src/{package}/
│   │   ├── __init__.py
│   │   ├── main.py             # FastAPI app, CORS, lifespan, router registration
│   │   ├── database.py         # SQLite/DuckDB connection (if needed)
│   │   ├── models.py           # Pydantic response models
│   │   ├── data/
│   │   │   ├── __init__.py
│   │   │   └── lessons.py      # All curriculum content (LESSONS, QUIZ_QUESTIONS, PAPERS, etc.)
│   │   ├── routers/
│   │   │   ├── __init__.py
│   │   │   ├── lessons.py      # GET /api/lessons/, GET /api/lessons/{id}
│   │   │   ├── simulator.py    # GET /api/simulator/scenarios, POST /api/simulator/step
│   │   │   ├── quiz.py         # GET /api/quiz/questions, POST /api/quiz/check
│   │   │   ├── papers.py       # GET /api/papers/
│   │   │   ├── playground.py   # POST /api/playground/execute, GET /api/playground/examples
│   │   │   ├── labs.py         # GET /api/labs/, GET /api/labs/{id}
│   │   │   ├── glossary.py     # GET /api/glossary/
│   │   │   ├── architecture.py # GET /api/architecture/diagrams
│   │   │   ├── patterns.py     # GET /api/patterns/
│   │   │   ├── bench.py        # POST /api/bench/run (optional — for live-engine topics)
│   │   │   ├── pipeline.py     # POST /api/pipeline/upload (optional — for data topics)
│   │   │   └── settings.py     # GET/PUT /api/settings/
│   │   └── services/
│   │       ├── __init__.py
│   │       ├── playground_service.py  # Live execution engine
│   │       └── bench_service.py       # Benchmark runner (optional)
│   └── tests/
│       ├── conftest.py
│       ├── test_health.py
│       ├── test_lessons.py
│       ├── test_simulator.py
│       ├── test_quiz.py
│       ├── test_papers.py
│       ├── test_playground.py
│       ├── test_labs.py
│       └── test_glossary.py
├── frontend/
│   ├── package.json
│   ├── vite.config.js
│   ├── tailwind.config.js
│   ├── postcss.config.js
│   ├── index.html
│   └── src/
│       ├── main.jsx
│       ├── App.jsx              # React Router with all routes
│       ├── lib/
│       │   └── api.js           # API client wrapper
│       ├── components/
│       │   ├── layout/
│       │   │   └── Sidebar.jsx  # Dark sidebar with nav links + progress
│       │   ├── GuidedTour.jsx   # Multi-step onboarding tour
│       │   └── diagrams/        # Reusable diagram components
│       │       ├── SequenceDiagram.jsx
│       │       ├── FlowDiagram.jsx
│       │       ├── StateMachine.jsx
│       │       └── PatternFlow.jsx
│       └── pages/
│           ├── Landing.jsx
│           ├── Dashboard.jsx
│           ├── Learn.jsx         # Lesson wizard with sidebar
│           ├── Playground.jsx    # Live code execution
│           ├── Simulator.jsx     # Agent/process simulation
│           ├── Patterns.jsx      # Pattern gallery
│           ├── Quiz.jsx          # Knowledge assessment
│           ├── Papers.jsx        # Research library
│           ├── Labs.jsx          # Hands-on exercises
│           ├── LabDetail.jsx     # Single lab with hints/solution
│           ├── Glossary.jsx      # Searchable terms
│           ├── Architecture.jsx  # Interactive diagrams
│           ├── Bench.jsx         # Performance benchmarks (optional)
│           ├── Pipeline.jsx      # Data pipeline viz (optional)
│           └── Settings.jsx
└── docs/
    ├── architecture.md
    ├── domain-model.md
    ├── api-spec.md
    └── constraints.md
```

---

## Backend Implementation

### main.py — FastAPI App

```python
from contextlib import asynccontextmanager
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from fastapi.staticfiles import StaticFiles
import os

@asynccontextmanager
async def lifespan(app: FastAPI):
    # Initialize DB / seed demo data here if needed
    yield
    # Cleanup connections here if needed

app = FastAPI(title="{App Name} Academy", lifespan=lifespan)

app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:{FRONTEND_PORT}"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Import and register routers
from .routers import lessons, simulator, quiz, papers, playground, labs, glossary, architecture, patterns, settings
app.include_router(lessons.router, prefix="/api/lessons", tags=["lessons"])
app.include_router(simulator.router, prefix="/api/simulator", tags=["simulator"])
app.include_router(quiz.router, prefix="/api/quiz", tags=["quiz"])
app.include_router(papers.router, prefix="/api/papers", tags=["papers"])
app.include_router(playground.router, prefix="/api/playground", tags=["playground"])
app.include_router(labs.router, prefix="/api/labs", tags=["labs"])
app.include_router(glossary.router, prefix="/api/glossary", tags=["glossary"])
app.include_router(architecture.router, prefix="/api/architecture", tags=["architecture"])
app.include_router(patterns.router, prefix="/api/patterns", tags=["patterns"])
app.include_router(settings.router, prefix="/api/settings", tags=["settings"])

# Optional: bench, pipeline routers for live-engine topics
# app.include_router(bench.router, prefix="/api/bench", tags=["bench"])
# app.include_router(pipeline.router, prefix="/api/pipeline", tags=["pipeline"])

@app.get("/api/health")
def health():
    return {
        "status": "ok",
        "app": "{App Name} Academy",
        "lessons": len(LESSONS),
        "categories": len(set(l["category"] for l in LESSONS)),
    }

# Static files (production: Vite build output)
if os.path.isdir("/app/static"):
    app.mount("/", StaticFiles(directory="/app/static", html=True), name="static")
```

### data/lessons.py — Curriculum Content

This is the heart of the academy. All content lives in Python data structures. A typical academy has **20-40 lessons** across **5-8 categories**.

```python
LESSONS = [
    {
        "id": 1,
        "category": "Foundations",          # Group lessons by topic
        "title": "What is {Topic}?",
        "type": "intro",                    # intro | concept | deep-dive | summary
        "description": "...",
        "keyPoints": ["Point 1", "Point 2", "Point 3"],
        # Optional rich content (include what's relevant):
        "code": "# Python/SQL code example\n...",
        "codeLanguage": "python",
        "comparison": {                     # Side-by-side comparison table
            "title": "X vs Y",
            "headers": ["Feature", "X", "Y"],
            "rows": [["Speed", "Fast", "Slow"], ...]
        },
        "diagram": {                        # Flow/architecture diagram
            "type": "flow",                 # flow | architecture | hierarchy
            "nodes": [{"id": "n1", "label": "Step 1", "color": "#6366f1"}],
            "edges": [{"from": "n1", "to": "n2", "label": "triggers"}]
        },
        "sequenceDiagram": {                # Protocol/interaction diagrams
            "title": "Request Flow",
            "actors": ["Client", "Server", "DB"],
            "steps": [
                {"from": "Client", "to": "Server", "label": "POST /api/query"},
                {"from": "Server", "to": "DB", "label": "SELECT ..."},
            ]
        },
        "realWorld": [                      # Real-world usage examples
            {"name": "Netflix", "description": "Uses this for ...", "patterns": ["pattern1"]}
        ],
    },
    # ... 20-40 more lessons
]

QUIZ_QUESTIONS = [
    {
        "id": 1,
        "category": "Foundations",
        "question": "What is the primary purpose of ...?",
        "options": ["Option A", "Option B", "Option C", "Option D"],
        "correct": 0,                       # Index of correct answer
        "explanation": "Option A is correct because ..."
    },
    # ... 10-30 questions (at least 2 per category)
]

PAPERS = [
    {
        "id": "react-2022",
        "title": "ReAct: Synergizing Reasoning and Acting",
        "authors": "Yao et al.",
        "year": 2022,
        "category": "Foundations",
        "url": "https://arxiv.org/abs/2210.03629",
        "summary": "Introduces the ReAct paradigm...",
        "keyInsight": "Interleaving reasoning and action outperforms..."
    },
    # ... 10-20 papers
]

SCENARIOS = [
    {
        "id": "scenario-1",
        "title": "Scenario Name",
        "task": "Description of what the agent/process does",
        "steps": [
            {"type": "thought", "content": "I need to..."},
            {"type": "action", "content": "Calling tool...", "tool": "web_search", "input": {"query": "..."}},
            {"type": "observation", "content": "Result: ..."},
            {"type": "answer", "content": "Final answer: ..."},
        ]
    },
    # ... 3-5 scenarios with 6-8 steps each
]

PATTERNS = [
    {
        "id": "pattern-1",
        "name": "Pattern Name",
        "category": "Architecture",         # Architecture | Data Processing | Governance | etc.
        "complexity": "intermediate",       # basic | intermediate | advanced
        "description": "...",
        "whenToUse": "Use when you need...",
        "flow": [                           # Visual step sequence
            {"step": "Input", "color": "#6366f1"},
            {"step": "Process", "color": "#10b981"},
            {"step": "Output", "color": "#f59e0b"},
        ],
        "realWorld": ["Company A uses this for...", "Company B ..."],
        "techStack": ["tech1", "tech2"],
    },
    # ... 8-12 patterns
]

GLOSSARY_TERMS = [
    {
        "id": "term-1",
        "term": "Term Name",
        "definition": "Clear 1-2 sentence definition...",
        "technology": "category-name",      # Maps to lesson categories
        "related": ["term-2", "term-3"],    # Cross-references
    },
    # ... 30-50 terms
]

LABS = [
    {
        "id": "lab-1",
        "title": "Build X from Scratch",
        "description": "In this lab you will...",
        "difficulty": "beginner",
        "technology": "category-name",
        "instructions": "Step 1: ...\nStep 2: ...",
        "starterCode": "# Fill in the blanks\ndef process():\n    pass",
        "hints": [
            "Hint 1: Start by importing...",
            "Hint 2: Use the X method to...",
        ],
        "solution": "# Complete solution\ndef process():\n    return do_thing()",
    },
    # ... 4-6 labs
]

ARCHITECTURE_DIAGRAMS = [
    {
        "id": "overview",
        "title": "System Overview",
        "description": "How all components connect",
        "nodes": [
            {"id": "n1", "label": "Component A", "type": "system", "x": 100, "y": 50},
            {"id": "n2", "label": "Component B", "type": "service", "x": 300, "y": 50},
        ],
        "edges": [
            {"from": "n1", "to": "n2", "label": "REST API", "style": "solid"},
        ]
    },
    # ... 2-4 diagrams
]
```

### Routers — API Endpoints

Each router follows a minimal pattern. Here are the key ones:

#### routers/lessons.py
```python
from fastapi import APIRouter, HTTPException
from ..data.lessons import LESSONS

router = APIRouter()

@router.get("/")
def list_lessons(category: str | None = None):
    result = LESSONS if not category else [l for l in LESSONS if l["category"] == category]
    return [{"id": l["id"], "title": l["title"], "category": l["category"],
             "type": l.get("type", "concept")} for l in result]

@router.get("/categories/summary")
def category_summary():
    cats = {}
    for l in LESSONS:
        c = l["category"]
        cats[c] = cats.get(c, 0) + 1
    return [{"name": k, "lesson_count": v} for k, v in cats.items()]

@router.get("/{lesson_id}")
def get_lesson(lesson_id: int):
    lesson = next((l for l in LESSONS if l["id"] == lesson_id), None)
    if not lesson:
        raise HTTPException(404, "Lesson not found")
    return lesson
```

#### routers/simulator.py
```python
from fastapi import APIRouter, HTTPException
from pydantic import BaseModel
from ..data.lessons import SCENARIOS

router = APIRouter()

class StepRequest(BaseModel):
    scenario_id: str
    step: int

@router.get("/scenarios")
def list_scenarios():
    return [{"id": s["id"], "title": s["title"], "task": s["task"],
             "step_count": len(s["steps"])} for s in SCENARIOS]

@router.get("/scenarios/{scenario_id}")
def get_scenario(scenario_id: str):
    s = next((s for s in SCENARIOS if s["id"] == scenario_id), None)
    if not s:
        raise HTTPException(404, "Scenario not found")
    return s

@router.post("/step")
def advance_step(req: StepRequest):
    s = next((s for s in SCENARIOS if s["id"] == req.scenario_id), None)
    if not s:
        raise HTTPException(404, "Scenario not found")
    if req.step >= len(s["steps"]):
        raise HTTPException(400, "No more steps")
    return s["steps"][req.step]
```

#### routers/quiz.py
```python
from fastapi import APIRouter, HTTPException
from pydantic import BaseModel
from ..data.lessons import QUIZ_QUESTIONS

router = APIRouter()

class CheckRequest(BaseModel):
    question_id: int
    selected: int

@router.get("/questions")
def list_questions():
    return [{"id": q["id"], "category": q["category"], "question": q["question"],
             "options": q["options"]} for q in QUIZ_QUESTIONS]

@router.post("/check")
def check_answer(req: CheckRequest):
    q = next((q for q in QUIZ_QUESTIONS if q["id"] == req.question_id), None)
    if not q:
        raise HTTPException(404, "Question not found")
    return {
        "correct": req.selected == q["correct"],
        "correct_answer": q["correct"],
        "explanation": q["explanation"],
    }
```

#### routers/playground.py — Live Code Execution

For **live-engine topics** (data, SQL, coding), the playground executes real code:

```python
from fastapi import APIRouter, HTTPException
from pydantic import BaseModel

router = APIRouter()

# Preloaded examples per technology/category
EXAMPLES = {
    "sql": [
        {"id": "ex1", "title": "Basic SELECT", "code": "SELECT * FROM demo_sales LIMIT 10;",
         "language": "sql", "description": "Query the sales table"},
        # ... more examples
    ],
    "python": [
        {"id": "ex2", "title": "Arrow Basics", "code": "import pyarrow as pa\n...",
         "language": "python", "description": "Create an Arrow table"},
    ],
}

class ExecuteRequest(BaseModel):
    code: str
    language: str = "sql"

@router.get("/examples/{technology}")
def get_examples(technology: str):
    return EXAMPLES.get(technology, [])

@router.post("/execute")
def execute_code(req: ExecuteRequest):
    """Execute code in a sandboxed environment. Adapt to your engine."""
    try:
        if req.language == "sql":
            # Use DuckDB, SQLite, or other SQL engine
            result = playground_service.execute_sql(req.code)
        elif req.language == "python":
            # Use restricted exec or subprocess
            result = playground_service.execute_python(req.code)
        else:
            raise HTTPException(400, f"Unsupported language: {req.language}")
        return result
    except Exception as e:
        return {"error": str(e)}
```

For **static-content topics**, the playground just displays preloaded code with syntax highlighting (no execution).

---

## Frontend Implementation

### App.jsx — Routes

```jsx
import { BrowserRouter, Routes, Route } from 'react-router-dom';
import Sidebar from './components/layout/Sidebar';
import Landing from './pages/Landing';
import Learn from './pages/Learn';
import Playground from './pages/Playground';
import Simulator from './pages/Simulator';
import Patterns from './pages/Patterns';
import Quiz from './pages/Quiz';
import Papers from './pages/Papers';
import Labs from './pages/Labs';
import LabDetail from './pages/LabDetail';
import Glossary from './pages/Glossary';
import Architecture from './pages/Architecture';
import Dashboard from './pages/Dashboard';
import Settings from './pages/Settings';

export default function App() {
  return (
    <BrowserRouter>
      <div className="flex h-screen bg-gray-50">
        <Sidebar />
        <main className="flex-1 overflow-auto">
          <Routes>
            <Route path="/" element={<Landing />} />
            <Route path="/dashboard" element={<Dashboard />} />
            <Route path="/learn" element={<Learn />} />
            <Route path="/playground" element={<Playground />} />
            <Route path="/simulator" element={<Simulator />} />
            <Route path="/patterns" element={<Patterns />} />
            <Route path="/quiz" element={<Quiz />} />
            <Route path="/papers" element={<Papers />} />
            <Route path="/labs" element={<Labs />} />
            <Route path="/labs/:id" element={<LabDetail />} />
            <Route path="/glossary" element={<Glossary />} />
            <Route path="/architecture" element={<Architecture />} />
            <Route path="/settings" element={<Settings />} />
          </Routes>
        </main>
      </div>
    </BrowserRouter>
  );
}
```

### Landing.jsx — Hero Page

The landing page must:
1. Show a compelling hero with the academy topic and value proposition
2. Display feature cards linking to each section (Learn, Playground, Simulator, etc.)
3. Show key stats (lesson count, category count, quiz questions, papers)
4. Include a health badge showing backend status
5. Auto-start a guided tour on first visit
6. List the topic categories with descriptions

### Learn.jsx — Lesson Wizard

The lesson wizard is the core learning experience. It must have:
- **Left sidebar**: Category list with progress bars (completed/total per category)
- **Main content area**: Renders the active lesson with mixed content types
- **Content type rendering**: Based on lesson fields, render key concepts, comparison tables, code blocks with syntax highlighting, flow diagrams, sequence diagrams, state machines, real-world examples
- **Navigation**: Previous/Next buttons with lesson counter (e.g., "12/37")
- **Progress tracking**: Set of completed lesson IDs (localStorage or API)

### Playground.jsx — Live Code Execution

For live-engine topics:
- **Left panel**: Schema browser (tables, columns, types) + file upload area
- **Main editor**: Code editor (CodeMirror or textarea with monospace) + sample query dropdown
- **Results panel**: Table view of results with row count and execution time
- **History tab**: Previous executions (clickable to reload)

For static topics:
- **Left panel**: Technology filter tabs
- **Main area**: Preloaded code examples with syntax highlighting and copy button
- **Save snippet**: Save custom snippets to database/localStorage

### Simulator.jsx — Agent/Process Simulation

- Scenario selector (3-5 scenarios)
- Step-by-step playback with Next Step / Run All / Reset controls
- Color-coded steps: Thought (purple), Action (green), Observation (blue), Answer (amber)
- Progress bar showing step completion
- Summary stats on completion

### Quiz.jsx — Knowledge Assessment

- 10-30 multiple-choice questions (randomized or sequential)
- Instant feedback with correct answer and explanation
- Score breakdown by category (Recharts bar chart)
- Overall percentage with performance message
- Restart button

### Papers.jsx — Research Library

- Left panel: Filterable list by category
- Right panel: Paper details (title, authors, year, summary, key insight)
- Optional: Embedded iframe reader for PDFs/specs
- Category filter buttons

---

## API Client (lib/api.js)

```javascript
const BASE = '/api';

async function request(path) {
  const res = await fetch(`${BASE}${path}`);
  if (!res.ok) throw new Error(`API error: ${res.status}`);
  return res.json();
}

async function post(path, body) {
  const res = await fetch(`${BASE}${path}`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(body),
  });
  if (!res.ok) throw new Error(`API error: ${res.status}`);
  return res.json();
}

const api = {
  health: () => request('/health'),
  // Lessons
  getLessons: (category) => request(`/lessons/${category ? `?category=${category}` : ''}`),
  getLesson: (id) => request(`/lessons/${id}`),
  getCategories: () => request('/lessons/categories/summary'),
  // Simulator
  getScenarios: () => request('/simulator/scenarios'),
  getScenario: (id) => request(`/simulator/scenarios/${id}`),
  simulateStep: (scenarioId, step) => post('/simulator/step', { scenario_id: scenarioId, step }),
  // Quiz
  getQuizQuestions: () => request('/quiz/questions'),
  checkAnswer: (questionId, selected) => post('/quiz/check', { question_id: questionId, selected }),
  // Papers
  getPapers: () => request('/papers/'),
  // Playground
  getExamples: (tech) => request(`/playground/examples/${tech}`),
  executeCode: (code, language) => post('/playground/execute', { code, language }),
  // Labs
  getLabs: () => request('/labs/'),
  getLab: (id) => request(`/labs/${id}`),
  // Glossary
  getGlossary: (tech) => request(`/glossary/${tech ? `?technology=${tech}` : ''}`),
  // Architecture
  getDiagrams: () => request('/architecture/diagrams'),
  // Patterns
  getPatterns: () => request('/patterns/'),
  // Settings
  getSettings: () => request('/settings/'),
};

export default api;
```

---

## Guided Tour (GuidedTour.jsx)

Every academy must include a **10-15 step guided tour** that auto-starts on first visit:

1. Welcome — introduce the academy topic
2. Learn — show the lesson wizard
3. Playground — demonstrate live code execution
4. Simulator — show how to step through scenarios
5. Patterns — browse the pattern gallery
6. Quiz — take a knowledge assessment
7. Papers — explore research papers
8. Labs — try hands-on exercises
9. Glossary — search key terms
10. Dashboard — track your progress

Store `{app}_tour_seen` in localStorage to show only once. Include "Skip Tour" and "Restart Tour" buttons.

---

## Content Volume Guidelines

| Content Type | Minimum | Recommended | Maximum |
|-------------|---------|-------------|---------|
| Lessons | 15 | 25-40 | 60 |
| Categories | 4 | 6-8 | 12 |
| Quiz Questions | 8 | 15-30 | 50 |
| Papers | 8 | 15-20 | 30 |
| Patterns | 5 | 8-12 | 20 |
| Glossary Terms | 20 | 30-50 | 80 |
| Labs | 3 | 5-6 | 10 |
| Simulator Scenarios | 2 | 4-5 | 8 |
| Architecture Diagrams | 2 | 3-4 | 6 |
| Playground Examples | 10 | 20-30 | 50 |

---

## start.sh Pattern

```bash
#!/bin/bash
set -e
DIR="$(cd "$(dirname "$0")" && pwd)"

# Backend
cd "$DIR/backend"
if [ ! -d .venv ]; then
    python3 -m venv .venv
    .venv/bin/pip install -r requirements.txt
fi
PYTHONPATH=src .venv/bin/uvicorn {package}.main:app --reload --port {BACKEND_PORT} &
BACKEND_PID=$!

# Frontend
cd "$DIR/frontend"
if [ ! -d node_modules ]; then
    npm install
fi
npm run dev &
FRONTEND_PID=$!

echo ""
echo "  Backend:  http://localhost:{BACKEND_PORT}"
echo "  Frontend: http://localhost:{FRONTEND_PORT}"
echo ""

trap "kill $BACKEND_PID $FRONTEND_PID 2>/dev/null" EXIT
wait
```

---

## Port Assignment

Pick unused ports. Current assignments in the portfolio:
- 8000-8007: Other apps
- 8008: Iceberg Academy
- 8009: Duck Analytics
- 8025: Agentic AI Academy
- Frontend ports: 5173-5181

---

## Testing Pattern

Write tests for every router. Minimum test count: **40+ tests**.

```python
# tests/conftest.py
import pytest
from httpx import AsyncClient, ASGITransport
from {package}.main import app

@pytest.fixture
async def client():
    transport = ASGITransport(app=app)
    async with AsyncClient(transport=transport, base_url="http://test") as c:
        yield c

# tests/test_lessons.py
@pytest.mark.anyio
async def test_list_lessons(client):
    r = await client.get("/api/lessons/")
    assert r.status_code == 200
    data = r.json()
    assert len(data) >= 15  # Minimum lesson count

@pytest.mark.anyio
async def test_get_lesson_detail(client):
    r = await client.get("/api/lessons/1")
    assert r.status_code == 200
    assert "title" in r.json()
    assert "category" in r.json()

@pytest.mark.anyio
async def test_lesson_not_found(client):
    r = await client.get("/api/lessons/9999")
    assert r.status_code == 404
```

---

## Dockerfile (Production)

```dockerfile
# Stage 1: Build frontend
FROM node:20-slim AS frontend
WORKDIR /app/frontend
COPY frontend/package.json frontend/package-lock.json ./
RUN npm ci
COPY frontend/ ./
RUN npm run build

# Stage 2: Python runtime
FROM python:3.12-slim
WORKDIR /app
COPY backend/requirements.txt ./requirements.txt
RUN pip install --no-cache-dir -r requirements.txt
COPY backend/src/ ./src/
COPY --from=frontend /app/frontend/dist ./static

ENV PYTHONPATH=/app/src
EXPOSE {BACKEND_PORT}
CMD ["uvicorn", "{package}.main:app", "--host", "0.0.0.0", "--port", "{BACKEND_PORT}"]
```

Add static file mounting in main.py (see `academy-register-deploy` skill for the exact pattern).

---

## Deployment

After the app is built and tested, use the `academy-register-deploy` skill to:
1. Build and push Docker image to ECR
2. Register in the agent-launcher Lambda
3. Add on-demand card to Solution Registry
4. **Add tour step to Solution Registry tour** (MANDATORY — see below)
5. Verify end-to-end deploy

### MANDATORY: Update Solution Registry Tour

Every new academy **must** be added to the `TOUR_STEPS` array in `deployed-apps.html`. The tour must always reflect the complete portfolio.

1. Open `/Users/Sats/Documents/TechnicalPlayGround/CodexFolder/deployed-apps.html`
2. Find `/* ── Learning Academy ── */` in `TOUR_STEPS` and add:
   ```javascript
   {
     icon: 'EMOJI', title: 'Academy Name',
     subtitle: 'Learning Academy \u00b7 Topic tagline',
     gradient: 'linear-gradient(135deg,#COLOR1,#COLOR2)', url: null,
     desc: 'Description of content...',
     features: [
       'Lesson count and categories',
       'Interactive features (simulators, playground, etc.)',
       'Quiz details and scoring',
       'On-demand deploy: click Deploy Now to spin up a live instance for 30 minutes'
     ]
   },
   ```
3. Update the **Welcome step** (first) and **Finale step** (last) with new counts
4. Push and deploy:
   ```bash
   cp deployed-apps.html /Users/Sats/Documents/TechnicalPlayGround/CodexFolder/solution-registry/index.html
   cd /Users/Sats/Documents/TechnicalPlayGround/CodexFolder/solution-registry
   git add index.html && git commit -m "feat: add ACADEMY_NAME to tour" && git push origin main
   aws s3 cp /Users/Sats/Documents/TechnicalPlayGround/CodexFolder/deployed-apps.html s3://my-solution-registry.satszone.link/index.html --content-type "text/html"
   aws cloudfront create-invalidation --distribution-id E2R00426B8QGNB --paths "/*"
   ```

---

## Rules

1. **All curriculum content goes in `data/lessons.py`** — never hardcode lesson text in routers or frontend
2. **Every lesson must have at least `id`, `category`, `title`, `description`, and `keyPoints`** — other fields are optional
3. **Quiz questions must include explanations** — not just correct/incorrect
4. **The landing page must show real stats** from the `/api/health` endpoint — not hardcoded numbers
5. **Progress tracking uses localStorage** on the frontend (Set of completed lesson IDs) unless you have a database
6. **Frontend uses plain JSX** (not TypeScript) with Tailwind CSS for styling
7. **No external AI API calls at runtime** unless the topic specifically requires it (e.g., AI tutor feature) — all content is pre-authored
8. **The guided tour auto-starts on first visit** and can be restarted from the sidebar/settings
9. **Code examples use dark terminal style** (`bg-gray-900 text-green-400 font-mono`)
10. **Playground examples are preloaded per technology** — users can also write custom code
11. **Minimum 40 backend tests** covering all routers
12. **Health endpoint returns lesson count and category count** — this is used by the landing page and Solution Registry
13. **Vite proxy config** must forward `/api` to the backend port
14. **The sidebar is dark themed** (gray-900 background) with active link highlighting
15. **Every page handles loading, error, and empty states** — use skeleton loaders for data-fetching pages

## Checklist

- [ ] Backend: main.py with lifespan, CORS, health, router registration
- [ ] Backend: data/lessons.py with LESSONS, QUIZ_QUESTIONS, PAPERS, SCENARIOS, PATTERNS, GLOSSARY_TERMS, LABS, ARCHITECTURE_DIAGRAMS
- [ ] Backend: All routers implemented (lessons, simulator, quiz, papers, playground, labs, glossary, architecture, patterns, settings)
- [ ] Backend: Playground service with live execution (if applicable)
- [ ] Backend: 40+ tests passing
- [ ] Frontend: Landing page with hero, feature cards, stats badge, guided tour trigger
- [ ] Frontend: Learn page with category sidebar, mixed content rendering, progress tracking
- [ ] Frontend: Playground with code editor/examples and execution/display
- [ ] Frontend: Simulator with step-by-step playback
- [ ] Frontend: Patterns gallery with flow diagrams
- [ ] Frontend: Quiz with instant feedback and score chart
- [ ] Frontend: Papers library with category filter
- [ ] Frontend: Labs with hints and solutions
- [ ] Frontend: Glossary with search and cross-references
- [ ] Frontend: Architecture diagrams (interactive nodes/edges)
- [ ] Frontend: Dashboard with progress overview
- [ ] Frontend: Guided tour (10+ steps, auto-start on first visit)
- [ ] Frontend: Dark sidebar with navigation and progress
- [ ] Frontend: API client (lib/api.js) covering all endpoints
- [ ] Dockerfile (multi-stage: Node frontend + Python backend)
- [ ] .dockerignore (node_modules, .venv, __pycache__)
- [ ] start.sh (launches both servers)
- [ ] docs/ (architecture.md, domain-model.md, api-spec.md, constraints.md)
