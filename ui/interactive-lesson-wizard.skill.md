---
name: Interactive Lesson Wizard
description: Build a step-by-step lesson system with progress tracking, category navigation, and multiple content types
category: ui
difficulty: intermediate
tags: [learning, wizard, progress, lessons]
stack: [React, Tailwind CSS, FastAPI]
---

# Interactive Lesson Wizard

Build a multi-step lesson wizard with category-based navigation, progress tracking, and mixed content types (text, code, diagrams, quizzes).

## Pattern Overview

The lesson wizard pattern combines a persistent category sidebar with a main content area that renders different content types per lesson. Progress is tracked per-category and overall, giving learners a clear sense of advancement through the curriculum.

## Architecture

### State Shape

```jsx
const [categories, setCategories] = useState([]);      // [{name, lesson_count, order}]
const [lessons, setLessons] = useState([]);             // All lessons (summary)
const [activeCategory, setActiveCategory] = useState(null);
const [activeLesson, setActiveLesson] = useState(null); // Full lesson with content
const [completedLessons, setCompletedLessons] = useState(new Set());
```

### Layout Structure

```
+------------------+------------------------------------------+
| Category Sidebar | Lesson Content Area                       |
|                  |                                           |
| [x] Foundations  | Title + Key Concepts badges               |
|     6/6 done    |                                           |
| [ ] Protocols    | Content body (markdown/structured text)   |
|     3/13 done   |                                           |
| [ ] Patterns     | Optional: Inline diagram component        |
|     0/7 done    | (SequenceDiagram, StateMachine, etc.)     |
|                  |                                           |
| Progress: 20%   | [Previous]  [Mark Complete]  [Next]       |
+------------------+------------------------------------------+
```

### Category Sidebar

The sidebar fetches category summaries from `/api/lessons/categories/summary` and displays them as a vertical list. Each category shows its name, lesson count, and a progress indicator (completed / total). Clicking a category filters the lesson list and selects the first incomplete lesson.

```jsx
function CategorySidebar({ categories, activeCategory, completedLessons, onSelect }) {
  return (
    <div className="w-64 border-r border-gray-200 bg-gray-50 p-4">
      {categories.map(cat => {
        const done = lessons
          .filter(l => l.category === cat.name)
          .filter(l => completedLessons.has(l.id)).length;
        return (
          <button
            key={cat.name}
            onClick={() => onSelect(cat.name)}
            className={`w-full text-left p-3 rounded-lg mb-2 ${
              activeCategory === cat.name ? 'bg-indigo-100 border-indigo-300' : 'hover:bg-gray-100'
            }`}
          >
            <div className="font-medium">{cat.name}</div>
            <div className="text-sm text-gray-500">{done}/{cat.lesson_count} complete</div>
            <div className="mt-1 h-1 bg-gray-200 rounded">
              <div
                className="h-1 bg-indigo-500 rounded"
                style={{ width: `${(done / cat.lesson_count) * 100}%` }}
              />
            </div>
          </button>
        );
      })}
    </div>
  );
}
```

### Content Type Rendering

Each lesson has an optional `diagram_type` field that determines which visual component renders alongside the text content. This allows mixing content types without changing the page structure.

```jsx
const DIAGRAM_COMPONENTS = {
  sequence: SequenceDiagram,
  transport: TransportDiagram,
  ecosystem: McpEcosystem,
  state: StateMachine,
  agent: AgentCard,
  message: MessageFormats,
};

function LessonContent({ lesson }) {
  const DiagramComponent = lesson.diagram_type
    ? DIAGRAM_COMPONENTS[lesson.diagram_type]
    : null;

  return (
    <div className="flex-1 p-8 max-w-4xl">
      <h1 className="text-2xl font-bold mb-4">{lesson.title}</h1>

      {/* Key concept badges */}
      <div className="flex flex-wrap gap-2 mb-6">
        {lesson.key_concepts.map(c => (
          <span key={c} className="px-3 py-1 bg-indigo-50 text-indigo-700 rounded-full text-sm">
            {c}
          </span>
        ))}
      </div>

      {/* Main content */}
      <div className="prose prose-indigo">{lesson.content}</div>

      {/* Optional diagram */}
      {DiagramComponent && (
        <div className="mt-8 p-6 bg-gray-50 rounded-xl border">
          <DiagramComponent {...lesson.diagram_data} />
        </div>
      )}
    </div>
  );
}
```

### Progress Tracking

Progress is maintained in component state as a `Set<string>` of completed lesson IDs. The overall progress bar at the top shows `completedLessons.size / totalLessons`. Navigation buttons (Previous / Next) move through lessons within the active category, wrapping to the next category when the current one is exhausted.

### Backend API Pattern

The backend serves lesson data from static Python data structures. The list endpoint returns summaries (no content) for fast sidebar rendering. The detail endpoint returns full content including `diagram_data` for the selected lesson. This two-tier approach minimizes payload size while keeping the API simple.

```python
@router.get("/")
def list_lessons(category: str | None = None):
    result = LESSONS if not category else [l for l in LESSONS if l["category"] == category]
    return [{"id": l["id"], "title": l["title"], "category": l["category"],
             "order": l["order"], "key_concepts": l["key_concepts"],
             "diagram_type": l.get("diagram_type")} for l in result]

@router.get("/{lesson_id}")
def get_lesson(lesson_id: str):
    lesson = next((l for l in LESSONS if l["id"] == lesson_id), None)
    if not lesson:
        raise HTTPException(404, "Lesson not found")
    return lesson
```

## When to Use This Pattern

- Educational platforms with structured curricula
- Onboarding wizards with multiple sections
- Documentation viewers with chapter navigation
- Any multi-step content consumption flow where users need to track their progress across categories
