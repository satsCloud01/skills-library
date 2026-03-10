---
name: commit-conventions
description: "Git commit conventions with conventional commits, atomic changes, meaningful messages, and branch strategies"
category: github
difficulty: beginner
tags: [git, commit, conventional-commits, branching, workflow]
stack: [git, github]
---

# Git Commit Conventions

You are a version control expert. Follow these conventions:

## Conventional Commits

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

### Types

| Type | When |
|------|------|
| `feat` | New feature or capability |
| `fix` | Bug fix |
| `refactor` | Code restructuring (no behavior change) |
| `docs` | Documentation only |
| `test` | Adding or fixing tests |
| `chore` | Build, deps, config changes |
| `style` | Formatting, whitespace (no logic change) |
| `perf` | Performance improvement |
| `ci` | CI/CD pipeline changes |

### Examples

```bash
feat(dashboard): add real-time activity chart
fix(models): handle duplicate name validation on update
refactor(routers): extract shared pagination logic
docs: add C4 architecture diagrams
test(api): add e2e tests for model lifecycle
chore: upgrade fastapi to 0.115.0
```

## Commit Rules

1. **Atomic commits**: One logical change per commit
2. **Present tense**: "add feature" not "added feature"
3. **No period**: Don't end the subject line with a period
4. **Under 72 chars**: Subject line length limit
5. **Why not what**: Body explains motivation, not code diff
6. **Stage specific files**: `git add file1 file2` not `git add -A`
7. **Never commit**: `.env`, credentials, `node_modules`, `.venv`, `__pycache__`

## Branch Strategy

```
main          ← production-ready, always deployable
├── feat/xyz  ← feature branches (short-lived)
├── fix/xyz   ← bug fix branches
└── chore/xyz ← maintenance branches
```

## .gitignore Template

```
# Python
.venv/
__pycache__/
*.pyc
.pytest_cache/
*.db

# Node
node_modules/
dist/

# IDE
.vscode/
.idea/

# Environment
.env
.env.local

# OS
.DS_Store
Thumbs.db
```

## Rules
- Commit early, commit often — small atomic commits
- Never commit secrets — use `.env` + `.gitignore`
- Write messages for future you — be descriptive
- Squash WIP commits before merging to main
- Tag releases: `git tag -a v1.0.0 -m "Initial release"`
