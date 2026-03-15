---
name: quiz-assessment
description: "Implements interactive knowledge assessment with server-side scoring, per-category breakdown, and Recharts visualization"
category: ui
difficulty: intermediate
tags: [quiz, assessment, scoring, recharts, education]
stack: [react-18, fastapi, tailwind, recharts]
---

# Quiz Assessment System

You are a frontend + backend engineer. Implement interactive knowledge quizzes with this proven pattern:

## Architecture

```
Backend:
  routers/quiz.py              — GET /questions (no answers), POST /check (scoring)
  data/lessons.py              — QUIZ_QUESTIONS list with correct index + explanation

Frontend:
  src/pages/Quiz.jsx           — Question flow, option selection, results dashboard
  src/lib/api.js               — checkAnswer(questionId, selected) POST call
```

## Backend: Server-Side Scoring

Questions endpoint returns options WITHOUT correct answers (prevents cheating):

```python
from fastapi import APIRouter
from pydantic import BaseModel

router = APIRouter()

QUIZ_QUESTIONS = [
    {
        "id": 1,
        "category": "Fundamentals",
        "question": "Which device operates at Layer 2?",
        "options": ["Hub", "Switch", "Router", "Access Point"],
        "correct": 1,          # Server-only — never sent to client
        "explanation": "..."   # Server-only — sent after answer
    },
]

@router.get("/questions")
def get_questions():
    return [
        {"id": q["id"], "category": q["category"], "question": q["question"], "options": q["options"]}
        for q in QUIZ_QUESTIONS
    ]

class AnswerCheck(BaseModel):
    question_id: int
    selected: int

@router.post("/check")
def check_answer(answer: AnswerCheck):
    for q in QUIZ_QUESTIONS:
        if q["id"] == answer.question_id:
            return {
                "correct": answer.selected == q["correct"],
                "correct_index": q["correct"],
                "explanation": q["explanation"],
            }
    return {"correct": False, "correct_index": -1, "explanation": "Question not found"}
```

## Frontend: Quiz Flow

```jsx
import { useState, useEffect } from 'react';
import { BarChart, Bar, XAxis, YAxis, Tooltip, ResponsiveContainer, Cell } from 'recharts';
import { api } from '../lib/api';

export default function Quiz() {
  const [questions, setQuestions] = useState([]);
  const [currentIdx, setCurrentIdx] = useState(0);
  const [selected, setSelected] = useState(null);
  const [result, setResult] = useState(null);
  const [answers, setAnswers] = useState([]);
  const [showResults, setShowResults] = useState(false);

  useEffect(() => { api.getQuizQuestions().then(setQuestions); }, []);

  const handleSelect = async (optIdx) => {
    if (result) return;
    setSelected(optIdx);
    const res = await api.checkAnswer(questions[currentIdx].id, optIdx);
    setResult(res);
    setAnswers(prev => [...prev, { ...res, category: questions[currentIdx].category }]);
  };

  const nextQuestion = () => {
    if (currentIdx < questions.length - 1) {
      setCurrentIdx(currentIdx + 1);
      setSelected(null);
      setResult(null);
    } else {
      setShowResults(true);
    }
  };

  // Results view with per-category Recharts bar chart
  if (showResults) {
    const correct = answers.filter(a => a.correct).length;
    const catScores = {};
    answers.forEach(a => {
      if (!catScores[a.category]) catScores[a.category] = { correct: 0, total: 0 };
      catScores[a.category].total++;
      if (a.correct) catScores[a.category].correct++;
    });
    const chartData = Object.entries(catScores).map(([cat, s]) => ({
      name: cat, score: Math.round((s.correct / s.total) * 100)
    }));
    // Render BarChart with chartData...
  }
}
```

## Key Design Decisions

1. **Server-side scoring** — correct answers never sent to client until after submission
2. **Per-category breakdown** — group results by category for targeted study recommendations
3. **Recharts visualization** — bar chart showing score percentage per category
4. **Progressive reveal** — one question at a time with immediate feedback
5. **Option highlighting** — correct = green, wrong = red, others = dimmed after answer
6. **Explanation** — server returns why the answer is correct, shown after each question

## Option Styling Pattern

```jsx
let style = 'border-slate-700/50 hover:border-accent/40 text-slate-300'; // default
if (result) {
  if (i === result.correct_index) style = 'border-emerald-500/50 bg-emerald-500/10 text-emerald-400';
  else if (i === selected && !result.correct) style = 'border-rose-500/50 bg-rose-500/10 text-rose-400';
  else style = 'border-slate-800 text-slate-500'; // dimmed
}
```

## Retake Flow

Reset state: `setCurrentIdx(0); setSelected(null); setResult(null); setAnswers([]); setShowResults(false);`
