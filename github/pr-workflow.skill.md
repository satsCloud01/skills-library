---
name: pr-workflow
description: "Pull request workflow with templates, review checklists, CI checks, and merge strategies"
category: github
difficulty: intermediate
tags: [github, pull-request, code-review, ci, merge]
stack: [github, git, gh-cli]
---

# Pull Request Workflow

You are a code review and collaboration expert.

## Creating a PR

```bash
# Create branch
git checkout -b feat/feature-name

# Work, commit, push
git add specific-files.py
git commit -m "feat(scope): description"
git push -u origin feat/feature-name

# Create PR via CLI
gh pr create --title "feat: short description" --body "$(cat <<'EOF'
## Summary
- What changed and why

## Changes
- `file1.py`: Added new router for feature X
- `Component.tsx`: New UI for feature X

## Test plan
- [ ] All existing tests pass
- [ ] New tests added for feature X
- [ ] Manual smoke test on local

🤖 Generated with [Claude Code](https://claude.com/claude-code)
EOF
)"
```

## PR Template

```markdown
## Summary
{1-3 bullet points describing what and why}

## Changes
{List of files changed with brief description}

## Test Plan
- [ ] Unit tests pass
- [ ] E2E tests pass
- [ ] Manual testing completed
- [ ] No security vulnerabilities introduced

## Screenshots
{If UI changes, include before/after}
```

## Review Checklist

### Code Quality
- [ ] Follows existing patterns in codebase
- [ ] No unnecessary complexity or over-engineering
- [ ] Error handling for edge cases
- [ ] No hardcoded values that should be configurable

### Security
- [ ] No secrets or API keys committed
- [ ] Input validation on API boundaries
- [ ] No SQL injection (using ORM properly)
- [ ] No XSS vulnerabilities

### Testing
- [ ] New code has test coverage
- [ ] Existing tests still pass
- [ ] Edge cases tested

### Documentation
- [ ] API changes documented
- [ ] README updated if needed

## Merge Strategy

- **Squash merge** for feature branches (clean history)
- **Merge commit** for release branches (preserve history)
- Delete branch after merge

## Commands

```bash
# List open PRs
gh pr list

# View PR details
gh pr view 123

# Check PR status
gh pr checks 123

# Merge PR
gh pr merge 123 --squash --delete-branch
```

## Rules
- PRs should be reviewable in < 30 minutes (limit scope)
- One logical change per PR — split large features
- Always include test plan
- Link related issues: `Fixes #123`
- Don't force-push to shared branches
- Respond to review comments within 24 hours
