# Contributing to Awesome AIBuildSpace

Thank you for contributing! This community is only as good as what we put into it together.

## Table of Contents

- [What We're Looking For](#what-were-looking-for)
- [Submitting a Resource](#submitting-a-resource)
- [Submitting a Risk Registry Entry](#submitting-a-risk-registry-entry)
- [Submitting a Skill](#submitting-a-skill)
- [Submitting a CLAUDE.md Template](#submitting-a-claudemd-template)
- [Submitting a Showcase](#submitting-a-showcase)
- [Quality Standards](#quality-standards)
- [Code of Conduct](#code-of-conduct)

---

## What We're Looking For

**Most valuable (please submit these):**
- 🔴 Risk Registry entries — real failures, gotchas, and mitigations
- 🧠 Agent Skills — SKILL.md files you use in production, with example output
- 📝 CLAUDE.md templates — real configs that work in the field
- 🪝 Hooks — tested, working hooks with clear documentation
- 🚀 Showcases — real projects with honest "what failed" sections

**Also welcome:**
- Plugin packages (bundled skills + hooks + MCP)
- Tools and integrations (with risk level noted)
- Learning resources you've actually used
- MCP server configurations
- Corrections to existing entries

**Not a fit:**
- Untested or theoretical resources
- Pure promotional content with no learnings
- Resources without any risk/limitation notes

---

## Submitting a Resource

1. **Fork** this repository
2. **Add your entry** in the appropriate section of `README.md`
3. Follow the table format:
   ```
   | [Tool Name](url) | One-line description | Risk Level | Key gotcha or note |
   ```
4. **Open a Pull Request** with:
   - Title: `Add: [resource name]`
   - Description: How you use it and what to watch out for

### Risk Levels
- 🟢 **Low** — well-tested, stable, minimal surprises
- 🟡 **Medium** — works well but has known edge cases
- 🔴 **High** — powerful but requires careful use; read the docs

---

## Submitting a Risk Registry Entry

This is the most valuable contribution you can make. Every entry saves other builders hours of debugging.

1. Open `RISK_REGISTRY.md`
2. Copy the entry template below
3. Fill it out completely — especially the mitigation
4. Open a PR with title: `Risk: [short description]`

### Entry Template

```markdown
---

### RISK-XXX — [Short title]

**Severity:** 🔴 High / 🟡 Medium / 🟢 Low
**Discovered by:** @your-github-handle
**Date:** YYYY-MM
**Affects:** Claude Code vX.X+ / Clawdata / MCP / [component]
**Status:** Active / Mitigated / Under investigation

#### Description

[2-4 sentences describing what happens and why it's a problem.]

#### How to Reproduce

1. [Step 1]
2. [Step 2]
3. [Expected vs. actual behavior]

#### Impact

[What breaks? What's the blast radius? Data loss risk? Cost risk?]

#### Mitigation

[Concrete steps to avoid or recover from this. Code examples welcome.]

#### Related

- [Reddit discussion link]
- [GitHub issue link]
- [Related RISK entries]
```

---

## Submitting a Skill

Agent Skills are the highest-value contribution for day-to-day builders. A skill you actually use is worth more than ten theoretical ones.

1. Create a `SKILL.md` file following the format in the [Skills & Slash Commands](README.md#skills--slash-commands) section
2. Include at least one real example of the skill's output
3. Set `disable-model-invocation: true` in the frontmatter if the skill has side effects
4. Open a PR with title: `Skill: [skill-name]`

Skills submitted to this repo are added to the [Community Skills Gallery](README.md#-community-skills-gallery) with a link to their source.

---

## Submitting a CLAUDE.md Template

1. Create a `CLAUDE.md` file for a specific project type (web app, data pipeline, skills project, etc.)
2. Include a comment block at the top explaining:
   - What type of project it's designed for
   - Key decisions and why
   - What to customize before using
3. Open a PR with title: `Template: [use case name]` — we'll add it to a `templates/` directory in the repo

---

## Submitting a Showcase

Share something you built with Claude Code — the good, the bad, and the broken.

Add an entry to the Showcases section of `README.md`:

```markdown
| [Project Name](url) | What it does | What failed / what I learned |
```

The "what failed" column is not optional. It's the point.

---

## Quality Standards

Every entry must include:
- ✅ A working, valid URL
- ✅ An honest description (not marketing copy)
- ✅ At minimum one note about limitations, risks, or gotchas
- ✅ Date of last verified working (for tools/integrations)

Entries will be removed if:
- The URL goes dead and no replacement is found within 30 days
- The resource is purely promotional
- The risk/limitation note is missing or vague ("use carefully" is not a mitigation)

---

## Code of Conduct

- Be honest about what works and what doesn't
- Credit original authors
- No promotional language — describe, don't sell
- Respond constructively to PR feedback
- If you disagree with a risk assessment, explain why with evidence

---

Questions? Open an issue or [post on r/AIBuildSpace](https://reddit.com/r/AIBuildSpace).
