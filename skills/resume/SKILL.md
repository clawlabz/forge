---
name: resume
description: "Resume interrupted Forge pipeline from last checkpoint via state file"
---

# /forge:resume — Resume Interrupted Forge Pipeline

## Overview

Resume a Forge pipeline from the last checkpoint. Reads `.forge/state.yaml` to determine where to pick up.

## Process

### Step 1: Locate State File

Look for `.forge/state.yaml` in the current working directory.
- If not found → tell user: "No Forge state found. Run `/forge:init` to start a new project."
- If found → read and parse it

### Step 2: Display Status Summary

Output a clear summary of the pipeline state:

```
📋 Forge Pipeline Status: <project name>

  Started: <date>
  Current: <current phase> — <status>

  ✅ P0 Brainstorm    — completed
  ✅ P1 PRD           — approved
  🔴 P2 Architect     — awaiting approval
  ⬜ P3 Design        — not started
  ⬜ P4 Develop       — not started
  ⬜ P5 Test          — not started
  ⬜ P6 Verify        — not started
  ⬜ P7 Deploy        — not started

  Summary: <state.summary field>
```

Use these status icons:
- ✅ completed / approved
- 🟡 running / in_progress
- 🔴 awaiting_approval / rejected
- ⬜ not_started
- ❌ failed

### Step 3: Determine Resume Action

Based on current state:

| Status | Action |
|--------|--------|
| `awaiting_approval` | Show the approval doc URL, ask user if approved or needs changes |
| `running` | Resume execution from the last completed sub-task |
| `rejected` | Show rejection reason, re-run the phase with modifications |
| `failed` | Show error, attempt to fix and retry |
| `completed` (all phases) | Tell user the pipeline is complete |

### Step 4: Resume Execution

- Update `state.yaml` status to `running`
- Invoke the appropriate `/forge:<phase>` skill
- If the phase was in the middle of sub-tasks (e.g., P4 develop phase 2 of 4), pass the sub-phase state so it can skip completed work

### Step 5: Handle Approval Gates

If the current phase is `awaiting_approval`:
1. Display the document URL: "Pending approval: <url>"
2. Ask: "Has this been approved? (yes / no / changes needed)"
3. If yes → update state to `approved`, advance to next phase
4. If no/changes → update state to `rejected` with reason, re-run phase

## Important

- ALWAYS read state.yaml before taking any action
- NEVER skip phases or approval gates
- If state.yaml seems corrupted or inconsistent, warn the user and ask how to proceed
- Update state.yaml after every significant action
