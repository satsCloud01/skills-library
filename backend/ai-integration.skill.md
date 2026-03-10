---
name: ai-integration
description: "Integrates Claude AI features with API key from browser, mock fallback, and structured prompt engineering"
category: backend
difficulty: advanced
tags: [claude, anthropic, ai, llm, prompt-engineering]
stack: [python-3.12, anthropic-sdk, fastapi]
---

# AI Integration (Claude)

You are an AI integration specialist. When adding Claude-powered features:

## Architecture

```
Browser localStorage → X-API-Key header → FastAPI router → Anthropic SDK → Response
                                                         ↓ (if no key)
                                                    Mock fallback
```

## API Key Flow

```python
# Router extracts key from header — never stored server-side
from fastapi import Header

@router.post("/ai/suggest")
async def ai_suggest(
    body: SuggestRequest,
    x_api_key: str = Header("", alias="X-API-Key"),
):
    if not x_api_key:
        return mock_suggestion(body)  # graceful fallback
    return await call_claude(body, x_api_key)
```

## Claude API Call Pattern

```python
import anthropic

async def call_claude(request, api_key: str):
    client = anthropic.Anthropic(api_key=api_key)

    message = client.messages.create(
        model="claude-haiku-4-5-20251001",  # fast + cheap for features
        max_tokens=1024,
        temperature=0.3,
        system="You are an expert in {domain}. Respond in JSON only.",
        messages=[
            {"role": "user", "content": build_prompt(request)}
        ],
    )

    text = message.content[0].text
    # Parse structured response
    return json.loads(text)
```

## Prompt Engineering

```python
def build_prompt(request):
    return f"""Analyze the following and provide suggestions.

Context:
{request.context}

Requirements:
- Return valid JSON array
- Each item: {{"suggestion": "...", "confidence": 0.0-1.0, "rationale": "..."}}
- Maximum 5 suggestions
- Be specific and actionable

Input:
{request.input_text}"""
```

## Mock Fallback

```python
def mock_suggestion(request):
    return {
        "suggestions": [
            {"suggestion": "Example suggestion", "confidence": 0.85, "rationale": "Mock response"},
        ],
        "is_mock": True,
        "message": "AI features require an API key. Set it in Settings.",
    }
```

## Rules
- API key from browser only — never in env vars, never persisted server-side
- Always provide mock fallback when key is missing
- Use Claude Haiku for UI features (fast, cheap) — Sonnet for complex analysis
- Temperature: 0.1-0.3 for structured/analytical, 0.5-0.7 for creative
- Parse JSON responses with try/except — LLMs can produce malformed JSON
- Set max_tokens appropriately — don't waste tokens
- Add `is_mock: true` flag so frontend can show "AI unavailable" badge
- Rate limit AI endpoints separately from CRUD endpoints
- Log token usage for cost tracking (don't log prompts — may contain sensitive data)
