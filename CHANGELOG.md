# Changelog

All notable changes to this list will be documented here.

## [Unreleased]

## [0.4.0] — 2026-03

### Added
- RISK-008: Hook RCE via malicious project files — CVE-2025-59536 / CVE-2026-21852 (Check Point Research)
- All entries now include "Models" field referencing Opus 4.6, Sonnet 4.6, Haiku 4.5
- RISK_REGISTRY.md index now includes Models column
- Recent sources added to all entries (2025–2026 GitHub issues, CVEs, real incidents)

### Changed
- RISK-001 expanded to cover MCP tool definition bloat as a second cause of context exhaustion (documented 2025)
- RISK-002 updated with real Feb 2026 incident: fabricated claims published to 8+ platforms (GitHub Issue #27430)
- RISK-006 updated with $5,623 real incident (Jul 2025) and double-billing bug (GitHub Issue #23315, Feb 2026)
- RISK-007 updated with Supabase/Cursor credential exfiltration incident (mid-2025) and 8,000+ exposed MCP servers report (Feb 2026)
- RISK-004 updated with March 2026 open issues around hook reliability (hooks silently not firing)
- RISK registry badge updated to 8 entries

## [0.3.0] — 2026-03

### Added
- RISK-006: Cost runaway in agentic loops with no spend guard
- RISK-007: Prompt injection via MCP responses and file contents
- Animated header and footer to RISK_REGISTRY.md
- Per-entry structured metadata table (severity, status, affects, first reported)

### Changed
- RISK-003 generalized from fictional "Clawdata" product to any MCP/data pipeline
- RISK-002 retitled from "tool results silently dropped" to "hallucinated references" — more accurately describes the user-visible failure
- RISK_REGISTRY.md index expanded with Affects column
- RISK-003 dead link removed (pointed to non-existent anthropics/clawdata repo)

### Fixed
- RISK-003 referenced a fictional Anthropic product and a dead GitHub link

## [0.2.0] — 2026-03

### Added
- Claude Code CLI deep-dive sections: Skills, Plugins, Hooks, CLAUDE.md
- Agent Skills / OpenClaw / Cowork section with platform compatibility table and packaging guide
- Community Skills Gallery
- `templates/SKILL-starter.md` — annotated SKILL.md starter template with all frontmatter fields documented
- `templates/CLAUDE-skills-project.md` — CLAUDE.md template for projects building Agent Skills or plugins
- Top Repos & References section featuring VoltAgent/awesome-openclaw-skills (5,400+ skills), VoltAgent/awesome-agent-skills, and other top community repos
- Animated header (capsule-render wave) and typing SVG to README
- Learning path navigation (Getting Started paths by experience level)
- Full resource index table (All Resources)
- Contributing section expanded with Skill and Plugin submission guidance

### Changed
- README fully restructured as a knowledge directory with topic-based navigation
- LICENSE corrected from MIT to CC0 to match stated license throughout repo

### Fixed
- LICENSE/README conflict: LICENSE file was MIT, all README references were CC0

## [0.1.0] — 2026-03

### Added
- Initial repository structure
- README with full category index
- RISK_REGISTRY.md seeded with 5 entries (RISK-001 through RISK-005)
- CONTRIBUTING.md with PR templates and quality standards
- CLAUDE.md template library: General, Web App, Data Pipeline
- Official resources section
- MCP servers section
