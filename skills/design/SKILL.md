---
name: design
description: "P3 UI design generation via Google Stitch MCP with prompt engineering"
---

# /forge:design — P3 AI-Generated Design via Stitch

## Overview

Generate production-quality UI designs using Google Stitch MCP, then export HTML + screenshots for development reference. Claude's role is to craft precise prompts for Stitch — NOT to design the UI itself.

This phase runs in parallel with P2 (Architect).

## CRITICAL: MCP Permission Constraint

**Stitch MCP tools CANNOT be called from sub-agents** (Claude Code security restriction). ALL Stitch MCP calls MUST be executed directly in the main conversation:
- `create_project` — main conversation
- `generate_screen_from_text` — main conversation
- `get_screen_code` — main conversation
- `get_screen_image` — main conversation

Do NOT delegate any Stitch MCP call to an Agent. The orchestrator (`/forge:build`) must execute P3 Design inline, not as a parallel agent.

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
- Tone: [2-3 adjectives, e.g., "professional, modern, developer-friendly"]

## Pages to Generate
| # | Page | Route | Purpose | Key Content |
|---|------|-------|---------|-------------|
| 1 | Homepage | / | Landing page | Hero, CTA, categories, featured items |
| 2 | ... | ... | ... | ... |
```

**Keep the brief SHORT.** It's for Claude's own planning, not for Stitch. Each page's actual Stitch prompt will be crafted individually in Step 4.

### Step 3: Create Stitch Project

Call Stitch MCP to create a project:

```
Tool: create_project
Input: { "name": "[Product Name] Design" }
```

Record the `projectId` for subsequent calls.

### Step 4: Generate Screens via Stitch

**Generation strategy: Homepage first, then derive.**

**Step 4a: Generate Homepage first (establishes the design style)**

```
Tool: generate_screen_from_text
Input: {
  "projectId": "<project-id>",
  "prompt": "<homepage prompt>",
  "deviceType": "DESKTOP",
  "modelId": "GEMINI_3_PRO"
}
```

Wait for homepage to complete. This establishes the visual language (colors, fonts, spacing, component styles) for the entire project.

**Step 4b: Generate remaining pages, referencing the established style**

For each subsequent page, add this line to the prompt:
`"Maintain the same visual style, colors, and typography as the other screens in this project."`

```
Tool: generate_screen_from_text
Input: {
  "projectId": "<project-id>",
  "prompt": "<page prompt>\n\nMaintain the same visual style, colors, and typography as the other screens in this project.",
  "deviceType": "DESKTOP",
  "modelId": "GEMINI_3_PRO"
}
```

**Step 4c: Consistency check — if any page looks off, use edit_screens:**

```
Tool: edit_screens
Input: {
  "projectId": "<project-id>",
  "selectedScreenIds": ["<inconsistent-screen-id>"],
  "prompt": "Adjust this screen to match the visual style of the other screens in the project.",
  "modelId": "GEMINI_3_PRO"
}
```

**Prompt Philosophy — CRITICAL:**

Stitch is a professional AI design tool powered by Gemini 2.5 Pro. It makes BETTER design decisions than Claude. Your prompts should act as a **product brief**, NOT a **design specification**.

**DO: Tell Stitch WHAT the product needs (product context)**
**DON'T: Tell Stitch HOW to design it (visual specifics)**

| Claude provides (WHAT) | Stitch decides (HOW) |
|------------------------|---------------------|
| Product name and what it does | Color palette |
| Target audience | Typography choices |
| Page content and features | Layout details and spacing |
| Example/realistic data | Component styling |
| Brand tone (1-2 words) | Visual hierarchy |
| | Theme (dark/light) |
| | Animations and polish |

**Each prompt should include:**
1. **Product context** — One line: what the product is and who it's for
2. **Page purpose** — What this specific page does
3. **Content** — What information appears, with realistic example data
4. **Tone** — 2-3 adjectives (e.g., "professional, developer-focused, modern")

**Do NOT include in prompts:**
- Specific hex colors (e.g., `#6C3DE8`)
- Font names (e.g., `Inter`, `JetBrains Mono`)
- CSS values (e.g., `rounded-lg`, `shadow-md`, `padding: 24px`)
- Layout grid specifics (e.g., `grid-cols-4`, `max-width: 1200px`)
- Component implementation details
- Theme preference (dark/light) — let Stitch decide

**Example prompt for a Dashboard page:**

```
ClawToolkit — an AI agent tool marketplace for developers.

Dashboard page showing a developer's tool usage and spending.

Content:
- 4 summary stats: Total API Calls (12,847), Credits Spent (4,230 CP),
  Credit Balance (15,770 CP), Active API Keys (3)
- Usage trend chart: daily API calls over last 30 days
- Table of tools used with columns: Tool Name, Calls, Cost, Avg Latency, Last Used
  (show 5 rows of realistic data)

Tone: professional, developer-focused, modern.
```

**Example prompt for a Homepage:**

```
ClawToolkit — an AI agent tool marketplace where developers publish and discover
tools that AI agents can call through a unified API.

Homepage / landing page.

Content:
- Hero section: headline about "one API key, all tools", call-to-action buttons
  for "Browse Tools" and "Publish a Tool"
- Category grid: 12 tool categories (Search & Web, Code Execution, Data & Analytics,
  Communication, File & Storage, AI & ML, Database, DevOps, Finance, Social & Media,
  Security, Utilities)
- Featured tools section: 4-6 tool cards showing name, description, price per call, usage count
- How it works: 3-step explanation (Discover → Connect → Pay-per-use)

Tone: professional, developer-focused, modern.
```

**Example prompt for Auth page:**

```
ClawToolkit — AI agent tool marketplace.

Sign in page. Options: GitHub OAuth button (primary) and email/password form.
Include link to sign up page.

Tone: clean, minimal, developer-friendly.
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

Based on Stitch's generated designs, write `docs/forge/P3-design-spec.md`.

**IMPORTANT**: The design spec is a SUMMARY extracted FROM Stitch's output — NOT Claude's own design ideas. Claude reads the generated HTML, extracts color values, fonts, spacing patterns, and documents them.

```markdown
# Design Specification: [Product Name]

> Version: 1.0
> Date: [date]
> Status: Pending Approval
> Generated by: Google Stitch + Forge
> Stitch Project: https://stitch.withgoogle.com/project/[projectId]

## 1. Page Designs

### Homepage
![Homepage](screenshots/homepage.png)
- Key elements: ...

### Dashboard
![Dashboard](screenshots/dashboard.png)
- Key elements: ...

### [Other Pages]
...

## 2. Design System (extracted from Stitch HTML)
### Colors
(Extract actual hex values from generated HTML)
### Typography
(Extract font families, sizes, weights)
### Spacing & Layout
(Extract grid, padding, margin patterns)

## 3. Development Reference
- **截图**: `docs/forge/screenshots/` (已提交到 Git)
- **HTML 文件**: `.forge/design-html/` (本地参考，gitignored)
- **Stitch Project ID**: [projectId] (用于后续编辑/变体生成)
- **实现方式**: 参考 Stitch HTML 的结构和样式，适配到 React/Next.js + Tailwind
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

1. For each generated screen, extract the `screenshot.downloadUrl` from Stitch response
2. Download each screenshot and save to `docs/forge/screenshots/<page-name>.png`
3. In the design spec, embed screenshots as GitHub-rendered images:
   ```markdown
   ### Homepage
   ![Homepage](screenshots/homepage.png)
   ```
4. Git add design spec + screenshots, commit and push
5. Output:
   ```
   🔴 审批门控

   请审批设计方案: https://github.com/<org>/<repo>/blob/main/docs/forge/P3-design-spec.md
   (文档内包含每个页面的截图，可直接在 GitHub 上查看)

   页面截图:
   | 页面 | 预览 |
   | Homepage | ![](docs/forge/screenshots/homepage.png) |
   | Dashboard | ![](docs/forge/screenshots/dashboard.png) |
   | ... | ... |

   回复 "approved" 继续，或指出需要修改的页面（我会重新生成）。
   ```

**关键**:
- 不要使用 `stitch.withgoogle.com/project/` 链接（项目是 PRIVATE，审批人无法访问）
- 必须下载截图到 `docs/forge/screenshots/` 并推送到 GitHub
- GitHub 会直接渲染图片，审批人在手机上打开 design-spec.md 就能看到所有页面设计

### 如何下载 Stitch 截图

Stitch `generate_screen_from_text` 返回的 response 包含:
```json
{
  "screenshot": {
    "downloadUrl": "https://lh3.googleusercontent.com/aida/..."
  },
  "htmlCode": {
    "downloadUrl": "https://contribution.usercontent.google.com/..."
  }
}
```

使用 `curl` 或 `fetch` 下载 `screenshot.downloadUrl` 保存为 PNG 文件:
```bash
curl -o docs/forge/screenshots/homepage.png "<downloadUrl>"
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

- Claude does NOT design the UI — Stitch generates it, Claude writes product briefs
- **Never specify colors, fonts, spacing, or CSS in prompts** — let Stitch make all visual decisions
- Prompts describe WHAT the page contains and WHO it's for, not HOW it should look
- The only visual guidance allowed: tone adjectives (e.g., "modern")
- Every page in the PRD MUST have a corresponding Stitch screen
- If a generated design is poor, adjust the CONTENT description, not add visual constraints
- HTML output is for REFERENCE only — developers adapt it to React/Next.js, not copy-paste
- Screenshots are the primary approval artifact — they go in the design spec
- .forge/design-html/ is gitignored (large files) — only screenshots go to git
