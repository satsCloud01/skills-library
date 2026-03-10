---
name: repo-setup
description: "GitHub repository setup with README, license, gitignore, branch protection, and project structure"
category: github
difficulty: beginner
tags: [github, repository, setup, readme, gitignore]
stack: [git, github, gh-cli]
---

# Repository Setup

You are a project initialization expert.

## Create and Initialize

```bash
# Create repo
gh repo create org/repo-name --public --clone

# Or initialize existing project
cd project-name
git init
gh repo create org/repo-name --source=. --public --push
```

## Essential Files

### README.md
```markdown
# Project Name

> One-liner description

**Live Demo:** [https://app.satszone.link](https://app.satszone.link)

## What It Does
- Feature 1
- Feature 2
- Feature 3

## Stack
| Layer | Technology |
|-------|-----------|
| Frontend | React 18 · TypeScript · Vite · Tailwind |
| Backend | FastAPI · Python 3.12 · SQLAlchemy 2.0 |
| Database | SQLite (auto-created) |
| AI | Claude Haiku (API key via UI) |

## Quick Start
\`\`\`bash
./start.sh
# Frontend → http://localhost:5173
# Backend  → http://localhost:8000/docs
\`\`\`

## Tests
\`\`\`bash
cd backend && PYTHONPATH=src .venv/bin/pytest tests/ -v
\`\`\`

## Documentation
See [docs/](docs/) for C4 architecture, API spec, and deployment guide.
```

### .env.example
```
# Copy to .env and fill in values
# ANTHROPIC_API_KEY=sk-ant-...  # Optional: for AI features
```

### pyproject.toml
```toml
[project]
name = "app-name"
version = "1.0.0"
requires-python = ">=3.12"

[tool.setuptools.packages.find]
where = ["src"]

[tool.pytest.ini_options]
asyncio_mode = "auto"
```

## First Commit

```bash
git add .gitignore README.md pyproject.toml requirements.txt start.sh \
        backend/src/ frontend/src/ frontend/package.json docs/
git commit -m "feat: initial project setup"
git push -u origin main
```

## Rules
- README: what it does, how to run it, stack — in that order
- .env.example: document all env vars without real values
- .gitignore: Python + Node + IDE + OS files
- No empty commits or placeholder files
- Tag first working version: `git tag -a v1.0.0 -m "Initial release"`
- Set up branch protection on main if team project
