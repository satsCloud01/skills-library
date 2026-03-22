---
name: universal-llm-dispatcher
description: "Universal multi-provider LLM dispatcher — routes requests to Anthropic, OpenAI, Google, Mistral, Groq, Together, or Ollama based on API key auto-detection or explicit provider selection"
category: backend
difficulty: advanced
tags: [python, llm, multi-provider, BYOK, anthropic, openai, google, mistral, groq, together, ollama, dispatcher]
stack: [python, fastapi, anthropic-sdk, openai-sdk, httpx]
---

# Universal LLM Dispatcher

Routes LLM requests to any of 7 providers with auto-detection from API key prefix. Supports both text and JSON response parsing. Mock fallback when no key provided.

## Provider Registry

```python
PROVIDERS = {
    "anthropic": {"name": "Anthropic", "models": ["claude-haiku-4-5-20251001", "claude-sonnet-4-20250514"], "default_model": "claude-haiku-4-5-20251001", "key_prefix": "sk-ant-"},
    "openai":    {"name": "OpenAI",    "models": ["gpt-4o-mini", "gpt-4o"], "default_model": "gpt-4o-mini", "key_prefix": "sk-"},
    "google":    {"name": "Google",    "models": ["gemini-2.0-flash", "gemini-2.5-pro"], "default_model": "gemini-2.0-flash", "key_prefix": "AI"},
    "mistral":   {"name": "Mistral",   "models": ["mistral-small-latest", "mistral-large-latest"], "default_model": "mistral-small-latest", "key_prefix": ""},
    "groq":      {"name": "Groq",      "models": ["llama-3.3-70b-versatile"], "default_model": "llama-3.3-70b-versatile", "key_prefix": "gsk_"},
    "together":  {"name": "Together",  "models": ["meta-llama/Llama-3.1-70B-Instruct-Turbo"], "default_model": "meta-llama/Llama-3.1-70B-Instruct-Turbo", "key_prefix": ""},
    "ollama":    {"name": "Ollama",    "models": ["llama3.2", "mistral"], "default_model": "llama3.2", "key_prefix": ""},
}
```

## Auto-Detection

```python
def detect_provider(api_key: str) -> str:
    if not api_key: return "mock"
    if api_key.startswith("sk-ant-"): return "anthropic"
    if api_key.startswith("gsk_"): return "groq"
    if api_key.startswith("sk-"): return "openai"
    if api_key.startswith("AI"): return "google"
    return "anthropic"
```

## Unified Call Interface

```python
async def call_llm(prompt, api_key="", provider="", model="", system="", max_tokens=2048, temperature=0.7) -> dict:
    """Returns {text, provider, model, input_tokens, output_tokens, latency_ms}"""

async def call_llm_json(prompt, api_key="", provider="", model="", system="", max_tokens=2048) -> dict:
    """Calls LLM and parses JSON from response. Returns {text, parsed, provider, ...}"""
```

## Key Design Decisions

- Keys NEVER stored server-side — passed per-request via header/body
- Auto-detect provider from key prefix when provider param not specified
- JSON parsing: strip markdown fences, extract `{...}` if full parse fails
- Mock fallback returns deterministic responses when no key provided
- Latency tracked per-call for usage logging
- Ollama uses local HTTP API at localhost:11434

## Reference Implementation

See: `prediction-intelligence/backend/src/predictor/llm_dispatcher.py`
