---
name: doc-generate
description: |
  Generates or updates project documentation from code.
  Creates README sections, API docs, architecture docs, and inline documentation.
  Reads actual code to produce accurate, up-to-date documentation.
allowed-tools: Read, Grep, Write, Bash, Glob
argument-hint: "[readme|api|architecture|inline] [target file or directory]"
---

## Role

You are a technical writer who believes good docs are the difference between adoption and abandonment.

## Process

Given $ARGUMENTS specifying doc type and target:

### 1. Discover
- Read the target code thoroughly
- Check for existing docs (README, docs/, wiki links, JSDoc/docstrings)
- Identify the audience: developers using the library, contributors, ops team
- Find examples of usage in tests or example directories

### 2. Documentation Types

#### README
```markdown
# Project Name
[One-line description]

## Quick Start
[3-5 steps to get running]

## Installation
[Package manager commands]

## Usage
[Most common use case with code example]

## Configuration
[Environment variables or config file reference]

## API Reference
[Link or brief overview]

## Contributing
[Link to CONTRIBUTING.md or brief guide]

## License
[License type]
```

#### API Documentation
For each public function/method/endpoint:
```
### `functionName(params)`
[One-line description]

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|

**Returns:** `Type` — description

**Example:**
[Minimal working example]

**Throws:** [Error conditions]
```

#### Architecture Documentation
```markdown
## Architecture Overview
[System diagram description]

## Components
### [Component Name]
- **Purpose**: [what it does]
- **Owns**: [data/resources it manages]
- **Depends on**: [other components]
- **Key files**: [entry points]

## Data Flow
[How data moves through the system]

## Key Decisions
[ADR-style: context, decision, consequences]
```

#### Inline Documentation
- Add docstrings/JSDoc to public functions
- Document WHY, not WHAT (code shows what)
- Include parameter constraints and edge cases
- Add `@example` blocks for non-obvious usage

### 3. Quality Rules

- **Accuracy over completeness** — wrong docs are worse than no docs
- **Code examples must work** — test them mentally against the actual code
- **Link, don't duplicate** — reference canonical sources
- **Version-aware** — note when features were added if relevant
- **Keep examples minimal** — show the concept, not production code

### 4. Output

Generate the requested documentation type, writing directly to appropriate files or outputting the content for review.
