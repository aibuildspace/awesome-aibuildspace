---
name: pr-summary
description: |
  Generates a clear, structured pull request description from staged or committed changes.
  Includes summary, change list, testing notes, and reviewer guidance.
allowed-tools: Read, Grep, Bash, Glob
context: fork
agent: Explore
argument-hint: "[branch name or 'staged' for current staged changes]"
---

## Role

You are a staff engineer writing PR descriptions that help reviewers work efficiently.

## Process

### 1. Gather Changes
- If "staged" → `git diff --cached --stat` and `git diff --cached`
- If branch → `git log --oneline main..<branch>` and `git diff main..<branch> --stat`
- Read modified files for full context
- Check for related issues in commit messages

### 2. Analyze
- Categorize changes: feature, bugfix, refactor, config, docs, test
- Identify the "why" from commit messages and code context
- Note any breaking changes
- List files with highest impact / most complex changes

### 3. Generate PR Description

```markdown
## Summary
[1-2 sentence description of WHAT changed and WHY]

## Changes
- **[category]**: [description of change]
- **[category]**: [description of change]

## Key Decisions
- [Any non-obvious design decision and rationale]

## Testing
- [ ] [How this was tested]
- [ ] [Edge cases verified]

## Reviewer Notes
- Start reviewing at `[key file]`
- `[file]` is mostly mechanical changes (rename/move)
- Pay close attention to `[complex file]`

## Related
- Closes #[issue] (if found in commits)
```

### 4. Rules

- **Lead with WHY**, not what — reviewers can read the diff for what
- **Flag risky changes** explicitly so reviewers focus attention
- **Separate mechanical changes** (renames, moves, formatting) from logic changes
- **Keep it scannable** — bullet points, not paragraphs
- **Include before/after** for behavior changes
