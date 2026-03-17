---
description: "P3: Generate UI designs via Google Stitch MCP"
---

# /forge:design — P3 Visual Design via Stitch

Generate UI designs using Google Stitch MCP.

## Usage

```
/forge:design
```

## How It Works

Read the full design logic from the `design` skill (`skills/design/SKILL.md`), then execute it.

## CRITICAL: MCP Permission

Stitch MCP tools CANNOT be called from sub-agents. ALL Stitch MCP calls (create_project, generate_screen_from_text, get_screen_code, get_screen_image) MUST be executed directly in the main conversation, NOT delegated to agents.
