# Stitch — Design Provider (Default)

## Configuration

```yaml
providers:
  design:
    type: "stitch"
    config:
      mcp_package: "@_davideast/stitch-mcp"
```

### Claude Code MCP Setup

```json
{
  "mcpServers": {
    "stitch": {
      "command": "npx",
      "args": ["@_davideast/stitch-mcp", "proxy"]
    }
  }
}
```

## Required Credentials

| Credential | Environment Variable | How to Get |
|-----------|---------------------|------------|
| Stitch API Key | `STITCH_API_KEY` | stitch.withgoogle.com → Settings → API Key |

## MCP Tools Available

| Tool | Purpose | Input | Output |
|------|---------|-------|--------|
| `create_project` | Create a new design project | `{ name }` | `{ projectId }` |
| `generate_screen_from_text` | Generate UI screen from text prompt | `{ projectId, prompt }` | `{ screenId }` |
| `list_projects` | List all projects | — | `[{ id, name }]` |
| `list_screens` | List screens in a project | `{ projectId }` | `[{ id, name }]` |
| `get_project` | Get project details | `{ projectId }` | `{ id, name, screens }` |
| `get_screen` | Get screen details | `{ projectId, screenId }` | `{ id, name }` |
| `get_screen_code` | Get HTML code for a screen | `{ projectId, screenId }` | HTML string |
| `get_screen_image` | Get screenshot (base64) | `{ projectId, screenId }` | base64 PNG |
| `build_site` | Build multi-page site from screens | `{ projectId, routes: [{ screenId, route }] }` | HTML per route |

## Output Formats

| Format | Use Case | Storage |
|--------|----------|---------|
| **HTML code** | Development reference — extract layout, styles, structure | `.forge/design-html/<page>.html` (gitignored) |
| **Screenshots (PNG)** | Approval review, documentation | `docs/forge/screenshots/<page>.png` (committed) |
| **Astro site** | Design preview deployment (optional) | `.forge/design-site/` (gitignored) |

## Workflow in Forge

```
1. Create Stitch project
2. For each PRD page:
   a. Craft detailed prompt (layout, content, colors, interactions)
   b. generate_screen_from_text → screenId
   c. get_screen_code → save HTML
   d. get_screen_image → save screenshot
3. Write design spec referencing screenshots
4. Push screenshots to GitHub for approval
5. P4 developers reference HTML for implementation
```

## Prompt Best Practices

Each prompt should include:
- **Product name and context** (what the product does)
- **Visual style** (dark/light, color scheme, reference apps)
- **Layout** (header/sidebar/main/footer structure)
- **Content sections** (ordered, with example data)
- **Interactive elements** (buttons, forms, tables with states)
- **Typography and spacing** preferences
- **Responsive notes** (mobile behavior)

## When Stitch is Unavailable

If `STITCH_API_KEY` is not set or Stitch service is down, the design skill automatically falls back to `manual_spec` provider — Claude generates a text-based design specification document instead.

## Notes

- Stitch is powered by Gemini 2.5 Pro
- Generated HTML is for REFERENCE only — not production code
- Developers extract patterns and adapt to React/Next.js + Tailwind
- Regenerate any unsatisfactory screen with improved prompts (max 3 attempts)
- Dark mode: specify in prompts or generate separate dark-theme screens
