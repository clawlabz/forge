---
description: Run the full Forge pipeline from idea to deployed product
---

# /forge — Full Pipeline Orchestrator

Run the complete Forge pipeline from idea to deployed product.

## Usage

```
/forge "A real-time collaborative whiteboard for remote teams"
```

## What This Command Does

Orchestrates the entire product manufacturing pipeline:
P0 Brainstorm → P1 PRD → P2 Architect → P3 Design → P4 Develop → P5 Test → P6 Verify → P7 Deploy

Pauses at 3 approval gates (PRD, Architecture, Design) and resumes after human approval.

## How It Works

Read the full orchestration logic from the `forge` skill (`skills/forge/SKILL.md`), then execute it step by step. If `.forge/state.yaml` exists, resume from the last checkpoint instead of starting over.
