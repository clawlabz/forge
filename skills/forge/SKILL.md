---
name: forge
description: "Main orchestrator — runs full P0-P7 pipeline with approval gates"
---

# /forge — Main Orchestrator

## Overview

The main entry point for the Forge pipeline. Accepts a product idea and orchestrates the entire P0→P7 flow automatically, pausing only at approval gates.

## Usage

```
/forge "A real-time collaborative whiteboard for remote teams"
```

## Orchestration Flow

```
INPUT: idea string
│
├─ Check for existing .forge/state.yaml
│  ├─ Found → invoke /forge:resume
│  └─ Not found → continue
│
├─ Step 1: Initialize
│  └─ Create project structure (inline, not full /forge:init)
│     - forge.config.yaml (from preset or interactive)
│     - .forge/state.yaml
│     - docs/forge/
│     - knowledge/
│
├─ Step 2: P0 Brainstorm
│  └─ Invoke /forge:brainstorm with the idea
│  └─ If result is KILL → stop, inform user
│  └─ If result is PIVOT → show suggestion, ask user, then continue with modified idea
│  └─ If result is GO → continue
│  └─ Update state → P0 completed
│
├─ Step 3: P1 PRD
│  └─ Invoke /forge:prd
│  └─ Self-review 2 rounds (via prd-reviewer agent)
│  └─ Push to GitHub
│  └─ 🔴 APPROVAL GATE: output link, STOP and wait
│  └─ On approval → update state, continue
│  └─ On rejection → re-run with feedback
│
├─ Step 4: P2 + P3 in Parallel (mixed execution)
│  ├─ Launch P2 Architect as background Agent (subagent_type="general-purpose"):
│  │   "Read skills/architect/SKILL.md, then execute the architecture phase."
│  │   NOTE: Use "general-purpose" or "everything-claude-code:architect", NOT "forge:architect"
│  │
│  └─ Simultaneously, execute P3 Design INLINE in main conversation:
│     Call Stitch MCP tools directly: create_project, generate_screen_from_text, etc.
│     (MCP tools require main conversation permissions, cannot run in sub-agent)
│
│  └─ Wait for both to complete
│  └─ 🔴 APPROVAL GATE: output BOTH links together, wait for approval
│     - Architecture: https://github.com/<org>/<repo>/blob/main/docs/forge/P2-architecture.md
│     - Design: https://github.com/<org>/<repo>/blob/main/docs/forge/P3-design-spec.md
│  └─ Both approved → continue to P4
│
├─ Step 5: P4 Develop
│  └─ Invoke /forge:develop
│  └─ Reads P2 dev plan + P3 design spec
│  └─ Executes phase by phase with checkpoints
│  └─ Update state after each sub-phase
│
├─ Step 6: P5 Test
│  └─ Invoke /forge:test
│  └─ Run full test suite
│  └─ Auto-fix failures and retest
│
├─ Step 7: P6 Verify (3 rounds)
│  └─ Invoke /forge:verify
│  └─ 3 rounds of multi-agent review + fix
│
├─ Step 8: P7 Deploy
│  └─ Invoke /forge:deploy
│  └─ Git push, DB migrate, deploy, DNS, analytics, knowledge base
│  └─ Output final project summary
│
└─ DONE: Output delivery report with all links
```

## State Management

At every phase transition, update `.forge/state.yaml`:
1. Set previous phase status to `completed`
2. Set current phase status to `running`
3. Update `current_phase` field
4. Update `summary` field with human-readable progress

## Approval Gate Protocol

When hitting an approval gate:

1. Ensure the document is pushed to GitHub
2. Get the raw URL: `https://github.com/<org>/<repo>/blob/main/docs/forge/<doc>.md`
3. Output to user:
   ```
   🔴 审批门控

   请审批 <document type>: <GitHub URL>

   回复 "approved" 继续，或提出修改意见。
   ```
4. Wait for user response
5. If approved → update state, continue
6. If changes requested → record feedback, re-run phase, increment attempt counter

## Model Routing

Before invoking each phase skill, check `forge.config.yaml` models section:
- If the platform supports model switching (Claude Code Agent tool `model` param) → use configured model
- If not supported → use current session model (fallback)
- Log which model was used in `.forge/logs/`

## Error Recovery

If any phase fails:
1. Update state to `failed` with error details
2. Attempt auto-recovery (e.g., build-error-resolver for build failures)
3. If auto-recovery fails → update state, inform user, suggest `/forge:resume` after fixing

## Parallel Execution

**P2 + P3 run in parallel** with mixed execution:
- **P2 Architect**: Launched as a **background sub-agent** (no MCP needed)
- **P3 Design**: Executed **inline in main conversation** (Stitch MCP requires main session)
- Both run simultaneously — P2 in background agent, P3 in foreground
- Wait for both to complete, then present both for approval together

In P4, independent development tasks within the same phase are parallelized via multiple Agent invocations.

In P6, multiple review dimensions run as parallel Agents.

## Agent Type Rules

When using the Agent tool to delegate work:
- Use `subagent_type="general-purpose"` for most tasks
- Or use `subagent_type="everything-claude-code:architect"` for architecture work
- **NEVER use forge skill/command names as agent types** (e.g., "forge:architect" is NOT a valid agent type)
- Forge skills (`forge:arch-reviewer`, `forge:prd-reviewer`, `forge:design-reviewer`, `forge:verify-checker`) ARE valid agent types
- Pass the skill content as context in the agent prompt, e.g.: "Read skills/architect/SKILL.md and execute..."

## Important Rules

- NEVER skip an approval gate
- ALWAYS update state.yaml before and after each phase
- ALWAYS push documents to GitHub before requesting approval
- If state.yaml exists when /forge is called, ALWAYS resume (don't restart)
- Each phase skill is responsible for its own output documents
- NEVER call Stitch MCP tools from a sub-agent — always in main conversation
