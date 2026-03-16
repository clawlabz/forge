# /forge:design — P3 AI-Generated Design via Stitch

## Overview

Generate production-quality UI designs using Google Stitch MCP, then export HTML + screenshots for development reference. Claude's role is to craft precise prompts for Stitch — NOT to design the UI itself.

This phase runs in parallel with P2 (Architect).

## Usage

```
/forge:design
```

## Prerequisites

- P1 PRD must be approved (`docs/forge/P1-prd.md`)
- Stitch MCP server configured (see Provider Configuration below)

## Provider Configuration

Default provider: `stitch` (via `@_davideast/stitch-mcp`)

```json
// Claude Code MCP configuration
{
  "mcpServers": {
    "stitch": {
      "command": "npx",
      "args": ["@_davideast/stitch-mcp", "proxy"]
    }
  }
}
```

**Authentication**: `STITCH_API_KEY` environment variable (obtain from stitch.withgoogle.com)

**Fallback**: If Stitch MCP is unavailable, fall back to `manual_spec` provider (Claude generates a design spec document instead). Check `forge.config.yaml` providers.design.type.

## Process

### Step 1: Read Context

Read these files:
- `docs/forge/P1-prd.md` — approved PRD (page list, features, user personas)
- `docs/forge/P0-brainstorm-report.md` — competitor analysis (for design differentiation)
- `forge.config.yaml` — design provider configuration

### Step 2: Design Brief Preparation

Before calling Stitch, prepare a structured design brief. This is the MOST IMPORTANT step — Stitch's output quality depends entirely on prompt quality.

Analyze the PRD and determine:

1. **Product type** — SaaS dashboard? Consumer app? Marketplace? Landing page?
2. **Target audience** — Developers? Enterprises? General consumers?
3. **Emotional tone** — Professional, playful, minimal, bold, warm, technical?
4. **Brand direction** — Color preferences, competitor differentiation, visual identity
5. **Key pages** — Full list from PRD with purpose and priority

Write the design brief to `docs/forge/P3-design-brief.md`:

```markdown
# Design Brief: [Product Name]

## Product Identity
- Type: [SaaS / marketplace / tool / ...]
- Audience: [developers / enterprises / consumers]
- Tone: [professional + modern / playful + colorful / ...]
- Differentiator: [what makes this visually distinct from competitors]

## Brand Direction
- Primary color preference: [color or "let Stitch decide"]
- Style references: [e.g., "Vercel's clean design meets GitHub's developer focus"]
- Must avoid: [e.g., "generic corporate blue", "template aesthetics"]

## Pages to Generate
| # | Page | Route | Purpose | Key Elements |
|---|------|-------|---------|-------------|
| 1 | Homepage | / | Landing + hero | Hero, CTA, features, categories |
| 2 | ... | ... | ... | ... |

## Technical Constraints
- Framework: [Next.js + Tailwind CSS]
- Must support: dark mode, responsive (mobile-first), WCAG AA
- Component library: [shadcn/ui base]
```

### Step 3: Create Stitch Project

Call Stitch MCP to create a project:

```
Tool: create_project
Input: { "name": "[Product Name] Design" }
```

Record the `projectId` for subsequent calls.

### Step 4: Generate Screens via Stitch

For EACH page in the design brief, call Stitch MCP with a carefully crafted prompt:

```
Tool: generate_screen_from_text
Input: {
  "projectId": "<project-id>",
  "prompt": "<detailed page prompt>"
}
```

**Prompt Engineering Guidelines — CRITICAL:**

Each prompt MUST include:
1. **Page purpose** — What this page does, who uses it
2. **Layout structure** — Header/sidebar/main/footer, grid layout
3. **Content sections** — Ordered list of what appears on the page
4. **Interactive elements** — Buttons, forms, tabs, modals
5. **Data display** — Tables, charts, cards, lists with example data
6. **Visual style** — Color scheme, typography feel, spacing preference
7. **States** — Loading, empty, error states if applicable
8. **Responsive note** — Mobile-first, breakpoint behavior

**Example prompt for a Dashboard page:**

```
Design a developer dashboard for an AI agent tool marketplace called "ClawToolkit".

Style: Dark theme, professional developer aesthetic. Primary color: deep purple (#6C3DE8).
Clean, spacious layout with subtle borders. Similar feel to Vercel or Linear dashboards.

Layout:
- Top navigation bar with logo "ClawToolkit" (left), nav links (center), user avatar (right)
- Main content area with max-width 1200px, centered

Content sections (top to bottom):
1. Summary cards row (4 cards): "Total Calls: 12,847", "CP Spent: 4,230", "CP Balance: 15,770", "API Keys: 3"
   - Each card has icon, label, and large number
2. Usage chart: Line chart showing daily API calls over last 30 days, purple gradient fill
3. Tool breakdown table: columns [Tool Name, Calls, CP Spent, Avg Latency, Last Used]
   - Show 5 rows of realistic example data
   - Sortable column headers

Colors: Dark background (#0F172A), card backgrounds (#1E293B), purple accents (#6C3DE8),
gray text hierarchy (white headings, #94A3B8 body, #64748B secondary)

Typography: Clean sans-serif (Inter), monospace for numbers/code

Must include: hover states on cards, active nav item indicator, responsive mobile layout
```

**Generate ALL pages from the design brief.** For large projects (>8 pages), group into batches:
- Batch 1: Core pages (homepage, main feature pages)
- Batch 2: Auth pages (login, signup)
- Batch 3: Dashboard pages
- Batch 4: Secondary pages (settings, docs)

### Step 5: Export HTML + Screenshots

After all screens are generated, export both HTML and screenshots:

**For each screen:**

```
Tool: get_screen_code
Input: { "projectId": "<id>", "screenId": "<screen-id>" }
→ Save HTML to: .forge/design-html/<page-name>.html
```

```
Tool: get_screen_image
Input: { "projectId": "<id>", "screenId": "<screen-id>" }
→ Save screenshot to: docs/forge/screenshots/<page-name>.png
```

**Optionally build full site:**

```
Tool: build_site
Input: {
  "projectId": "<id>",
  "routes": [
    { "screenId": "<id>", "route": "/" },
    { "screenId": "<id>", "route": "/dashboard" },
    ...
  ]
}
```

### Step 6: Generate Design Spec Document

Based on Stitch's generated designs, extract and document the design system:

Write `docs/forge/P3-design-spec.md`:

```markdown
# Design Specification: [Product Name]

> Version: 1.0
> Date: [date]
> Status: Pending Approval
> Generated by: Google Stitch + Forge
> Stitch Project ID: [id]

## 1. Design Overview
- Style direction (extracted from generated designs)
- Screenshots of key pages (embedded or linked)

## 2. Design System (extracted from Stitch HTML)
### Colors
(Extract from generated HTML — primary, secondary, accent, backgrounds, text colors)
### Typography
(Extract font families, sizes, weights from HTML)
### Spacing & Layout
(Extract grid, padding, margin patterns)

## 3. Page Designs
### 3.1 [Page Name]
- Screenshot: docs/forge/screenshots/<page-name>.png
- HTML reference: .forge/design-html/<page-name>.html
- Key design decisions
- Responsive notes

### 3.2 [Page Name]
...

## 4. Development Reference
- All HTML files: `.forge/design-html/`
- All screenshots: `docs/forge/screenshots/`
- Stitch project: [link or ID for future edits]

## 5. Implementation Notes
- Extract Tailwind classes from HTML for component styling
- Use HTML structure as layout reference, NOT as production code
- Adapt to React/Next.js component architecture
- Ensure all interactive states are implemented (hover, focus, disabled)
```

### Step 7: Self-Review

Review the generated designs for:
- [ ] All pages from PRD are generated
- [ ] Visual consistency across pages (same color/font/spacing)
- [ ] Dark mode present (or prompt Stitch to regenerate with dark theme)
- [ ] Mobile responsiveness visible in designs
- [ ] Brand differentiation — doesn't look generic
- [ ] Key interactive elements are visible (buttons, forms, navigation)

If any page is unsatisfactory, **regenerate it** with an improved prompt.

### Step 8: Push & Request Approval

1. Git add screenshots + design spec (NOT the .forge/design-html/ — too large for git)
2. Commit and push
3. Output:
   ```
   🔴 审批门控

   请审批设计方案: https://github.com/<org>/<repo>/blob/main/docs/forge/P3-design-spec.md

   设计截图:
   - 首页: docs/forge/screenshots/homepage.png
   - Dashboard: docs/forge/screenshots/dashboard.png
   - [其他页面...]

   回复 "approved" 继续，或提出修改意见（我会重新生成对应页面）。
   ```

## Fallback: Manual Spec Provider

If Stitch MCP is unavailable (`forge.config.yaml` providers.design.type = `manual_spec`):

1. Skip Steps 3-5
2. Use `ui-ux-pro-max` skill + `frontend-design` skill to generate a text-based design specification
3. Define design system (colors, typography, spacing, components) in markdown
4. Describe page layouts with ASCII wireframes
5. No HTML or screenshots — the spec document IS the deliverable

## Skills & Tools Used

**Primary (Stitch provider):**
- Stitch MCP Server (`@_davideast/stitch-mcp`) — AI design generation
- `design-reviewer` agent — self-review

**Fallback (manual_spec provider):**
- `ui-ux-pro-max` — 50+ styles, 21 color palettes, 50+ font pairings
- `frontend-design` — high-quality UI component design
- `everything-claude-code:frontend-patterns` — frontend design patterns

## Important Rules

- Claude does NOT design the UI — Stitch generates it, Claude crafts the prompts
- Every page in the PRD MUST have a corresponding Stitch screen
- Prompt quality is everything — be specific about layout, content, colors, interactions
- If a generated design is poor, iterate the prompt and regenerate (max 3 attempts per page)
- HTML output is for REFERENCE only — developers adapt it to React/Next.js, not copy-paste
- Screenshots are the primary approval artifact — they go in the design spec
- Dark mode: generate both light and dark variants, or specify dark theme in prompts
- .forge/design-html/ is gitignored (large files) — only screenshots go to git
