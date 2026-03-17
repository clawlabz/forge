---
description: Run the full Forge pipeline from idea to deployed product
---

# /forge:build — Full Pipeline Orchestrator

Run the complete Forge pipeline from idea to deployed product.

## Usage

```
/forge:build "A real-time collaborative whiteboard for remote teams"
```

## What This Command Does

Orchestrates the entire product manufacturing pipeline:
P0 Brainstorm → P1 PRD → P2 Architect → P3 Design → P4 Develop → P5 Test → P6 Verify → P7 Deploy

Pauses at 3 approval gates (PRD, Architecture, Design) and resumes after human approval.

## How It Works

Read the full orchestration logic from the `forge` skill (`skills/forge/SKILL.md`), then execute it step by step. If `.forge/state.yaml` exists, resume from the last checkpoint instead of starting over.

## CRITICAL: MCP Tool Usage

Stitch MCP and other MCP tools CANNOT be called from sub-agents (permission restriction). When the pipeline reaches P3 Design:
- Do NOT delegate Stitch MCP calls to a sub-agent
- Execute all `generate_screen_from_text`, `get_screen_code`, `get_screen_image` calls directly in the main conversation
- Sub-agents can still be used for non-MCP tasks (code review, testing, etc.)
