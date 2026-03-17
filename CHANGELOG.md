# Changelog

## 0.5.0 (2026-03-17)

### Design
- **Stitch as default design provider** — Google Stitch MCP generates UI designs
- Homepage-first generation strategy for visual consistency
- Screenshots downloaded to Git for approval (Stitch project URLs are private)
- Prompts are product briefs, not design specs — let Stitch make visual decisions

### Pipeline Flow
- P2 (agent, background) + P3 (main conversation, Stitch MCP) run in parallel
- P4→P5→P6→P7 fully automatic — no pause, no skip, no user confirmation
- P6 Verify enforces exactly 3 rounds with report files
- P7 Deploy auto-executes all scripts — never outputs manual TODO list

### Quality
- PRD and Architecture: self-review must complete BEFORE approval gate
- Parallel development: conditional policy (≥3 independent tasks → must parallelize)

### Plugin
- Commands renamed: `/forge:build` (main), `/forge:init`, `/forge:resume`, etc.
- Frontmatter on all commands, skills, and agents
- Plugin validation passes with zero warnings
- Agent type rules documented (skills ≠ agent types)

## 0.1.0 (2026-03-15)

- Initial release
- 11 skills, 4 agents, 7 templates, 2 presets
- Provider architecture with pluggable services
- State management with interrupt/resume support
